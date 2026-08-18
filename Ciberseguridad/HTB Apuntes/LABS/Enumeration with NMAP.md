#lab
# EASY LAB
Our client wants to know if we can identify which operating system their provided machine is running on. Submit the OS name as the answer.

Ubuntu
## Attempts

## 1

```shell
sudo nmap 10.129.2.80 -n -Pn -sS -oN easy1.nmap
```
**Output**
```shell
# Nmap 7.94SVN scan initiated Mon Jan 12 13:13:42 2026 as: nmap -n -Pn -sS -oN easy1.nmap 10.129.2.80
Nmap scan report for 10.129.2.80
Host is up (0.31s latency).
Not shown: 869 closed tcp ports (reset), 128 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
10001/tcp open  scp-config

# Nmap done at Mon Jan 12 13:18:46 2026 -- 1 IP address (1 host up) scanned in 303.59 seconds
```

## 2

```shell
sudo nmap 10.129.2.80 -p22,80,1001 -O -Pn -n -oN easy1.1
```

8 alertas

## 3

```shell
sudo nmap 10.129.2.80 -sS -n -Pn --disable-arp-ping --packet-trace -sV -p22,80,10001
```

61 alertas

# MEDIUM LAB
After the configurations are transferred to the system, our client wants to know if it is possible to find out our target's DNS server version. Submit the DNS server version of the target as the answer.
## Attemps

```shell
sudo nmap 10.129.82.113 -p53 --packet-trace --disable-arp-ping -sU -sV -T2

# Mejora con -Pn
# Do not work with -n because it needs de DNS resolution to get the name
```
**Output**
```shell
PORT   STATE SERVICE VERSION
53/udp open  domain  (unknown banner: HTB{GoTtgUnyze9Psw4vGjcuMpHRp})
```

# HARD LAB
Now our client wants to know if it is possible to find out the version of the running services. Identify the version of service our client was talking about and submit the flag as the answer.

```shell
sudo nmap --packet-trace -Pn -sV --source-port 53 --disable-arp-ping 10.129.127.142 -oN hard1.try
```
**OUTPUT**
```shell
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http       Apache httpd 2.4.29 ((Ubuntu))
50000/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

By using -sV, Nmap returns that the host is down.

So, I tried with script `banner`

```shell
sudo nmap 10.129.127.142 -p50000 --script banner
```
**OUTPUT**
```shell
PORT      STATE    SERVICE
50000/tcp filtered ibm-db2
```

## Netcat

```shell
sudo nc -nv 10.129.127.142 50000 -p 53                                   
```
**OUTPUT**
``` 
Connection to 10.129.127.142 50000 port [tcp/*] succeeded!
220 HTB{kjnsdf2n982n1827eh76238s98di1w6}
```

