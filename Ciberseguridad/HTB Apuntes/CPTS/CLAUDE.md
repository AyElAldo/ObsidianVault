# CLAUDE.md — HackerVault

`HackerVault.html` es una base de conocimiento de pentesting: un único archivo HTML, offline (funciona con `file://`, sin dependencias externas), con herramientas y comandos organizados por las 8 fases de la Cyber Kill Chain. Contenido en español. El archivo `Pentest-Knowledge-Base.html` es la versión antigua: NO tocarla.

## Cómo ampliar HackerVault.html (añadir herramientas desde los apuntes .md)

Cuando el usuario pida agregar herramientas nuevas (normalmente desde un apunte `.md` de esta carpeta):

1. Lee **solo** el/los archivo(s) `.md` que indique. No escanees todo el vault.
2. **No releas `HackerVault.html` entero** (son >100 KB). Para ubicar dónde insertar, usa `grep`/`Grep`:
   - último bloque de datos: busca `DATA.push` (inserta después del último).
   - versión: busca `const BASE_VERSION`.
3. Extrae solo las herramientas/comandos que **no estén ya** en la base (compara por `name`).
4. Añade **un único** bloque `DATA.push({...}, {...});` nuevo justo después del último `DATA.push` existente (antes del comentario `HELP_MAP`). No reescribas el archivo; usa `Edit` sobre esa zona.
5. **Sube `const BASE_VERSION` en 1** (p. ej. de `1` a `2`). Es imprescindible: el merge de `localStorage` solo incorpora las herramientas nuevas (por `name`) si la versión sube, y así no borra las ediciones locales del usuario.
6. Al terminar, di qué herramientas añadiste. No modifiques el diseño ni el motor.

## Formato de cada herramienta (objeto JS)

```
{
  name:"Nombre",
  phase:"recon|weapon|delivery|exploit|install|c2|actions|util",
  os:["Linux","Windows","Multiplataforma"],
  mitre:"TA0043 / T1595, T1046",
  approach:["reconocimiento","post-explotacion", ...],
  ambito:"red|host|ambas",
  source:"CPTS - ...",
  short:"Descripcion corta, 1-2 frases.",
  cmd:"comando principal con variables {{...}}",
  help:"comando -h / man comando",
  subs:[
    {title:"Subtitulo", short:"que hace", cmd:"comando", info:"HTML con <b>flags</b> explicados", examples:["ejemplo 1","ejemplo 2"], source:"CPTS - ..."}
  ]
}
```

### Reglas de contenido
- Variables globales que se sustituyen en vivo en los comandos (úsalas en `cmd`, `examples`, `subs[].cmd`):
  `{{RHOST}}` `{{RPORT}}` `{{LHOST}}` `{{LPORT}}` `{{DOMAIN}}` `{{TARGET_USER}}` `{{WORDLIST}}` `{{PIVOT_HOST}}`
- `phase`: fase de la Cyber Kill Chain (recon=Reconnaissance, weapon=Weaponization, delivery=Delivery, exploit=Exploitation, install=Installation, c2=Command&Control, actions=Actions on Objectives, util=Utilidades).
- `ambito`: `red` (sobre objetivos por red), `host` (enum/privesc/credenciales en un host ya comprometido) o `ambas`.
- `info` admite HTML; resalta flags y conceptos con `<b>...</b>`. Si necesitas la secuencia `</script>` dentro de un string, escríbela como `<\/script>` (si no, rompe el parser).
- Estilo del texto: español **sin tildes ni ñ** (igual que las entradas existentes: "Escaner", "evasion", "categoria"), para evitar problemas de codificacion.
- No dupliques herramientas ya presentes; si una existe pero con subcomandos nuevos, añade los `subs` nuevos a esa entrada.

## Diseño (no cambiar salvo que lo pidan)
Tema oscuro (negro `#050506` + acento lima `#e4f222`), colores de fase "Jewel Tones", pestañas de fase arriba (multi-seleccion), panel de variables a la izquierda ocultable, grid denso, copiar con animacion de contorno, paleta ⌘K, filtros SO/enfoque/Red-Host, CRUD + import/export JSON, persistencia en `localStorage` con merge por `BASE_VERSION`.

## Nivel de detalle exigido en TODAS las tarjetas (requerimiento del usuario, 2026-11-25)

El usuario pidio explicitamente llevar **todas** las tarjetas de HackerVault.html al mismo nivel de profundidad que las de **Chisel** y **Dnscat2** (fase `actions`/`c2`, anadidas 2026-11-24/25). Esto aplica tanto a las ~53 tarjetas ya existentes (pendientes de enriquecer) como a **cualquier herramienta nueva** que se agregue de ahora en adelante — no es opcional, es el estandar por defecto.

Una tarjeta al nivel esperado tiene:
- **3-5 `subs`** (no 1-2 genericos), cada uno cubriendo un paso real: instalacion/compilacion, los distintos modos de uso (ej. normal vs reverse), y troubleshooting especifico con errores conocidos y su causa.
- Al menos un `sub` o bloque `info` con **advertencias/recomendaciones explicitas** cuando el apunte original las menciona (ej: incompatibilidades de version, deteccion por EDR, limites de intentos/lockout, variantes obsoletas).
- `examples` reales (no arrays vacios) usando las variables globales `{{...}}`.
- `help` relleno cuando exista un `-h`/`man` real para esa herramienta.
- Recursos externos citados si el apunte los menciona (writeups, blogs, nombre de maquina de HTB, etc.), tal como se hizo con el post de Oxdf y el video de IppSec en Chisel.

### Importante: esto no es solo "agregar texto" — hay que releer el apunte `.md` origen
Para enriquecer una tarjeta ya existente hay que volver a leer la seccion correspondiente del `.md` fuente (ver `source:` del tool) y extraer matices, advertencias y pasos que quedaron fuera la primera vez — no inventar relleno ni expandir con generalidades. Si el apunte no tiene mas profundidad que la ya capturada, esta bien dejar la tarjeta como esta (no inflar artificialmente).

### Caveat tecnico: enriquecer una tarjeta EXISTENTE no se propaga solo con subir BASE_VERSION
El merge de `localStorage` (`loadData()`) solo **agrega** tools nuevas por `name` que el usuario todavia no tiene guardadas; **nunca sobreescribe** el contenido de una tarjeta cuyo `name` ya existe en su `localStorage`. Si el usuario ya abrio HackerVault.html antes, enriquecer una tarjeta existente en el codigo no le mostrara nada nuevo hasta que pulse **"↺ Restaurar base original"** (pierde sus ediciones locales) o borre el `localStorage` del archivo. Avisar de esto cada vez que se enriquezcan tarjetas que ya existian (no aplica a tarjetas nuevas, esas si aparecen solas con el merge).

### Checklist de progreso (marcar `[x]` al enriquecer; permite retomar si se pierde la sesion)

**Ya al nivel esperado:**
- [x] Dnscat2 (c2)
- [x] Chisel (actions)

**recon — completo (2026-11-25):**
- [x] Nmap (ya estaba al nivel, sin cambios)
- [x] WHOIS
- [x] dig / DNS
- [x] Subfinder / Amass
- [x] crt.sh (CT logs)
- [x] VHost Fuzzing
- [x] Shodan
- [x] Web Fingerprinting
- [x] DNSDumpster
- [x] theHarvester
- [x] Enum4linux-ng
- [x] smtp-user-enum
- [x] username-anarchy
- [x] ReconSpider
- [x] SMB Enumeration
- [x] Nessus / OpenVAS
- [x] searchsploit

**Pendientes — weapon:**
- [ ] MSFvenom
- [ ] Crunch
- [ ] CeWL
- [ ] Reglas de mutacion (Hashcat)
- [ ] JS Obfuscation
- [ ] Web Shells (preparacion)

**Pendientes — delivery:**
- [ ] PowerShell Downloads
- [ ] Descargas Linux (wget/curl)
- [ ] SMB File Transfer (impacket)
- [ ] Servidores HTTP rapidos
- [ ] LOLBins (certutil / bitsadmin)
- [ ] SCP

**Pendientes — exploit:**
- [ ] Metasploit (msfconsole)
- [ ] Hydra
- [ ] Medusa
- [ ] NetExec / CrackMapExec
- [ ] Evil-WinRM
- [ ] Attacking SQL (MSSQL/MySQL)
- [ ] Attacking FTP
- [ ] Responder
- [ ] SQL Injection (fundamentos)
- [ ] Cross-Site Scripting (XSS)
- [ ] Clickjacking (PoC)
- [ ] Kerbrute

**Pendientes — install:**
- [ ] Web Shell (acceso via navegador)
- [ ] Spawning de shell interactiva (TTY)

**Pendientes — c2:**
- [ ] Netcat Listener
- [ ] Penelope

**Pendientes — actions:**
- [ ] John the Ripper
- [ ] Hashcat
- [ ] Cracking de archivos/archives
- [ ] Dump de credenciales (SAM/NTDS)
- [ ] linPEAS
- [ ] winPEAS
- [ ] GTFOBins
- [ ] Pivoting / Tunneling
- [ ] PCredz
- [ ] Attacking LSASS
- [ ] Windows Credential Manager
- [ ] Pass-the-Hash (PtH)
- [ ] Pass-the-Ticket (PtT)
- [ ] Credential Hunting (Windows)
- [ ] Linux Auth & Cracking
- [ ] Pivoting (Windows / alternativas)

**Pendientes — util:**
- [ ] HackTricks
- [ ] SecLists
- [ ] Encoders / Decoders

Al terminar cada tanda de tarjetas, actualizar esta checklist en el mismo commit/edicion antes de seguir, y avisar al usuario que debe "Restaurar base original" si quiere ver las tarjetas existentes enriquecidas.
