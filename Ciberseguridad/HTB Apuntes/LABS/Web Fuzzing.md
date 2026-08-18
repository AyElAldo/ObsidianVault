# Skills Assessment

To complete this Skills Assessment, you will need to apply the multitude of tools and techniques showcased throughout this module. All fuzzing can be completed using the `common.txt` SecLists Wordlist, found at `/usr/share/seclists/Discovery/Web-Content` on Pwnbox, or via the SecLists GitHub.

> [!Example] TARGET: 154.57.164.61:31667

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.61:31667/FUZZ -t 300 -recursion -e .php,.txt,.html
```

```html
http://154.57.164.61:31667/admin/panel.php
```

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.61:31667/admin/panel.php\?accessID\=FUZZ -t 300 -fs 58
```

```html
http://154.57.164.61:31667/admin/panel.php?accessID=getaccess
```

>[!Important]
>Add the domain to `/etc/hosts`

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.61:31667 -H "Host: FUZZ.fuzzing_fun.htb" -t 300 -mc 200
---
hidden                  [Status: 200, Size: 45, Words: 8, Lines: 1, Duration: 194ms]
```

>[!Important]
>Add the subdomain.domain.htb to `/etc/hosts`

```shell
curl http://fuzzing_fun.htb:31667/godeep -H "Host: hidden.fuzzing_fun.htb"  
---
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://hidden.fuzzing_fun.htb/godeep/">here</a>.</p>
<hr>
<address>Apache/2.4.61 (Debian) Server at hidden.fuzzing_fun.htb Port 80</address>
</body></html>
```

```shell
# FINAL command
ffuf -u http://hidden.fuzzing_fun.htb:31667/godeep/FUZZ -t 300 -w /usr/share/seclists/Discovery/Web-Content/common.txt -recursion
```

