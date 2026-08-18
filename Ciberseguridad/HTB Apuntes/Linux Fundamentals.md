#cybersecurity #code #HTB #linux 
## User Management

- `userdd`: Crea un nuevo usuario.
	- `-m, --create-home             create the user's home directory`
	- `-M, --no-create-home          do not create the user's home directory`
- `usermod`: Modifica la cuenta de un usuario
	- `-a, --append                  append the user to the supplemental GROUPS mentioned by the -G option without removing the user from other groups
	- `-L, --lock                    lock the user account`
	- `-r, --remove                  remove the user from only the supplemental GROUPS`
- `su`: Cambiar de usuario en el sistema Linux/Unix, permitiéndote ejecutar comandos como otro usuario
	- `-c, --command <command>         pass a single command to the shell with -c`
## Filter Contents

Comandos `more` y `less`: Ayudan a la visualización de archivos grandes de texto. 
- `more`: Al salir, se ve la página interactiva.
- `less`: Tiene mas opciones y al salir, no deja rastro de la página interactiva.
- `head`: Muestra las primeras líneas de un archivo
- `tail`: Muestra las últimas líneas de un archivo

Otras herramientas útiles que se puede combinar:
- `sort`: Ordena alfabéticamente las líneas.
	- `-u`: Quita repetidos.
	 **Ejemplo de uso:**
```shell
cat /etc/passwd | sort
```
- `grep`:  Busca patrones de los resultados.
	- `-v, --invert-match              Invert the sense of matching, to select non-matching lines.` 
```shell
cat /etc/passwd | grep "/bin/bash"
```
- `cut`: Cortar salidas
	- `-d, --delimiter`
	- `-f[numero]`: Muestra solo el fragmento \[numero] delimitado
```shell
cat /etc/passwd | grep -v nologin | cut -d ":" -f1 # Nombre de usuario

cut -d " " -f5-8 # Imprimir un rango de delimitadores
```
- `tr`: Sustituir valores de salida
```shell
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " "

# Output
root x 0 0 root /root /usr/bin/zsh
sync x 4 65534 sync /bin /bin/sync
```
- `column -t`
- `awk`: Permite cortar pero también usando cadenas y no solo caracteres
```shell
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | awk '{print $1, $NF}'
```
- `sed`: Sustituir salidas estándar
```shell
cat /etc/passwd | sed 's/ayelaldo/aldo/g'

# Original		          # Sustituido
ayelaldo:x:1000           aldo:x:1000:
```

### Respuestas complicadas
- How many services are listening on the target system on all interfaces? (Not on localhost and IPv4 only)
```shell
netstat -tuln | grep "LISTEN" | grep -v "127.0.0" | grep -v tcp6 | wc -l
```

## Regular Expressions

| Operador   | Descripción                                                                                                                                                                            |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **(a)**    | Los corchetes se utilizan para agrupar partes de una expresión regular.                                                                                                                |
| **[a-z]**  | Los corchetes se utilizan para definir clases de caracteres. Dentro de ellos, se puede especificar una lista de caracteres para buscar.                                                |
| **{1,10}** | Las llaves se utilizan para definir cuantificadores. Dentro de ellas, se puede especificar un número o un rango que indica la frecuencia con la que debe repetirse un patrón anterior. |
| **\|**     | También llamado operador OR y muestra resultados cuando una de las dos expresiones coincide.                                                                                           |
| **.***     | Funciona de manera similar a un operador AND mostrando resultados solo cuando ambas expresiones están presentes y coinciden en el orden especificado.                                  |
## Service and Process Management

Los demonios son identificados regularmente por la letra `d` al final de su nombre de programa, por ejemplo `sshd (SSH daemon)` o `systemd (init system)`.

### Acciones de servicios o procesos
1. Start/Restart a service/process
2. Stop a service/process
3. See what is/was happening with a service/process
4. Enable/Disable a service/process on boot
5. Find a service/process

> A todos los procesos de un sistema Linux se les asigna un *PID* y se pueden ver en el directorio */proc/*
### Ejemplo de Acciones en terminal

```shell
systemctl start ssh
systemctl status ssh
sudo systemctl enable ssh # Importante SUDO
```

O listar todos los servicios del sistema
```shell
systemctl list-units --type=service # Listar todos los servicios

journalctl -u ssh.service --no-pager # Ver información del servicio

```

### Formas de interactuar con los procesos

#### Todos:
```shell
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 2) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
3) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
4) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
5) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
6) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
7) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
8) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
9) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
10) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
11) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
12) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
13) SIGRTMAX-1  64) SIGRTMAX
```
#### Más comunes:

| **Signal** | **Description**                                                                                                          |
| ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| `1`        | `SIGHUP` - This is sent to a process when the terminal that controls it is closed.                                       |
| `2`        | `SIGINT` - Sent when a user presses `[Ctrl] + C` in the controlling terminal to interrupt a process.                     |
| `3`        | `SIGQUIT` - Sent when a user presses `[Ctrl] + D` to quit.                                                               |
| `9`        | `SIGKILL` - Immediately kill a process with no clean-up operations.                                                      |
| `15`       | `SIGTERM` - Program termination.                                                                                         |
| `19`       | `SIGSTOP` - Stop the program. It cannot be handled anymore.                                                              |
| `20`       | `SIGTSTP` - Sent when a user presses `[Ctrl] + Z` to request for a service to suspend. The user can handle it afterward. |
#### Ejemplos de uso

- Matar un proceso: 
```shell
kill 9 <PID>
```

- El atajo `Ctrl + Z` suspende los procesos y no se ejecutarán más. Para mantenerlos en segundo plano, debemos introducir el comando `bg` para poner el proceso en segundo plano.
	- Para ver las tareas suspendidas usamos el comando `jobs`
	- Para reanudar una tarea usamos el comando `bg`

> Otra opción es configurar automáticamente el proceso con un signo *AND (&)* al final del comando. De esta manera se puede seguir en la sesión mientras se ejecuta el comando anterior.

- **Foreground process (Primer plano)**
	 - Para devolver un proceso a primer plano se ejecuta el comando `fg <ID>`. 

### Execute Multiple Commands

Existen tres tipos:
- `;` 
```shell
echo '1'; ls MISSING_FILE; echo '3'
# Salida
1
ls: cannot access 'MISSING_FILE': No such file or directory
3
```
- `&&`
```shell
echo '1' && ls MISSING_FILE && echo '3'
# Salida
1
ls: cannot access 'MISSING_FILE': No such file or directory
```
- `|` : Depende de la salida de los comandos anteriores

## Task Scheduling

- \[Unit]
- \[Options]
- \[Install]

### Ejemplo

```shell
AyElAldo@htb[/htb]$ sudo mkdir /etc/systemd/system/mytimer.timer.d
AyElAldo@htb[/htb]$ sudo vim /etc/systemd/system/mytimer.timer
```

```shell
[Unit]
Description=My Timer

[Timer]
OnBootSec=3min
OnUnitActiveSec=1hour

[Install]
WantedBy=timers.target
```

```shell
sudo vim /etc/systemd/system/mytimer.service
```

```shell
sudo systemctl daemon-reload
```

```shell
sudo systemctl start mytimer.timer
sudo systemctl enable mytimer.timer
```

### Cron

Necesitamos almacenar las tareas en un archivo llamado **crontab** y luego decirle al demonio cuándo ejecutar las tareas.

| **Time Frame**         | **Description**                                                       |
| ---------------------- | --------------------------------------------------------------------- |
| Minutes (0-59)         | This specifies in which minute the task should be executed.           |
| Hours (0-23)           | This specifies in which hour the task should be executed.             |
| Days of month (1-31)   | This specifies on which day of the month the task should be executed. |
| Months (1-12)          | This specifies in which month the task should be executed.            |
| Days of the week (0-7) | This specifies on which day of the week the task should be executed.  |
## Web Servers

### cURL
cURL es una herramienta que nos permite transferir archivos desde la **shell** a través de protocolos como **HTTP, HTTPS, FTP, SFTP, FTPS** o **SCP**.

```shell
curl http://localhost/ # Regresa una salida estandar del source
```
### Wget

```shell
wget http://localhost/ # Descarga el source
```

### Levantar Web Servers

> Repositorio con varias opciones para levantar servers: https://gist.github.com/willurd/5720255

```shell
python -m http.server # python
php -S 127.0.0.1:8080 # php
http-server -p 8080 # npm ?
```

## File System Management

In Linux, files can be stored in one of several key types:

- Regular files
- Directories
- Symbolic links