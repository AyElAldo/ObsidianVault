#CPTS #cybersecurity #HTB #hacking 

Let's put together everything we learned in this module and attack our first box without a guide.
# Tips

Remember that enumeration is an iterative process. After performing our `Nmap` port scans, make sure to perform detailed enumeration against all open ports based on what is running on the discovered ports. Follow the same process as we did with `Nibbles`:

- Enumeration/Scanning with `Nmap` - perform a quick scan for open ports followed by a full port scan
- Web Footprinting - check any identified web ports for running web applications, and any hidden files/directories. Some useful tools for this phase include `whatweb` and `Gobuster`
- If you identify the website URL, you can add it to your '/etc/hosts' file with the IP you get in the question below to load it normally, though this is unnecessary.
- After identifying the technologies in use, use a tool such as `Searchsploit` to find public exploits or search on Google for manual exploitation techniques
- After gaining an initial foothold, use the `Python3 pty` trick to upgrade to a pseudo TTY
- Perform manual and automated enumeration of the file system, looking for misconfigurations, services with known vulnerabilities, and sensitive data in cleartext such as credentials
- Organize this data offline to determine the various ways to escalate privileges to root on this target

There are two ways to gain a foothold—one using `Metasploit` and one via a manual process. Challenge ourselves to work through and gain an understanding of both methods.

There are two ways to escalate privileges to root on the target after obtaining a foothold. Make use of helper scripts such as [LinEnum](https://github.com/rebootuser/LinEnum) and [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) to assist you. Filter through the information searching for two well-known privilege escalation techniques.

Have fun, never stop learning, and do not forget to `think outside of the box`!

---
# Target: 10.129.164.230
### Spawn the target, gain a foothold and submit the contents of the user.txt flag.

I started with an `namp` enumeration:

```shell
# REMEMBER TO SAVE ALL FINDINGS
sudo nmap -Pn 10.129.164.230 -T4 -oA final_assesment

# Output
sudo nmap -Pn 10.129.164.230 -T4 -oA final_assesment                                                                                                             ─╯
[sudo] password for ayelaldo: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-20 11:05 CST
Nmap scan report for 10.129.164.230
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.86 seconds
```

Once we get this, we can enumerate more with the next command while trying fuzzing in the 80 port (web):

```shell
sudo nmap -Pn 10.129.164.230 -T4 -oA final_assesment_all_ports -p-

# Output                                                                                             ─╯
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-20 11:06 CST
Stats: 0:02:30 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 14.06% done; ETC: 11:23 (0:15:17 remaining)
Warning: 10.129.164.230 giving up on port because retransmission cap hit (6).
Stats: 0:05:27 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 28.41% done; ETC: 11:25 (0:13:44 remaining)
Stats: 0:18:28 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 99.41% done; ETC: 11:24 (0:00:07 remaining)
Nmap scan report for 10.129.164.230
Host is up (0.23s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
23918/tcp filtered unknown
42460/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 1136.60 seconds
```

The `fuzzing` enumeration that I did was:

```shell
# MEDIUM-SIZE SECLIST FIRST
ffuf -u http://10.129.164.230/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o med_directories -of all -t 200

# LARGE SECLIST THEN
ffuf -u http://10.129.164.230/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -o big_directories -of all -t 200
```

With both outputs we get the next directories:

```shell
data                    [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 244ms]
admin                   [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 3072ms]
backups                 [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 259ms]
plugins                 [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 6248ms]
theme                   [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 312ms]
server-status           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 229ms]
```

To get more information we can do a `recursive fuzzing`:

```shell
ffuf -u http://10.129.164.230/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -o recursion2-big -of all -t 200 -recursion -recursion-depth 3 -e .php,.html,.txt
```

#### Robots.txt

```txt
User-agent: *
Disallow: /admin/
```
#### 10.129.164.230/data/

```txt
Index of /data
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	- 	 
[DIR]	cache/	2024-03-12 13:05 	- 	 
[DIR]	other/	2024-03-12 13:05 	- 	 
[DIR]	pages/	2024-03-12 13:05 	- 	 
[DIR]	thumbs/	2018-09-07 17:58 	- 	 
[DIR]	uploads/	2018-09-07 17:58 	- 	 
[DIR]	users/	2024-03-12 13:05 	- 	 
Apache/2.4.41 (Ubuntu) Server at 10.129.164.230 Port 80
```

After looking for important information within `/data` folder we get the user `admin` in `/data/users/admin.xml`. This confirmed that the user exists and we can look for findings linked to the password

```txt
<item>
<USR>admin</USR>
<NAME/>
<PWD>d033e22ae348aeb5660fc2140aec35850c4da997</PWD>
<EMAIL>admin@gettingstarted.com</EMAIL>
<HTMLEDITOR>1</HTMLEDITOR>
<TIMEZONE/>
<LANG>en_US</LANG>
</item>
```

>[!Important] Important
>We have found the SHA1 admin's psswd above.

Now that we have the HASH of the password, we can crack it (case of low-sec password). 

```shell
echo d033e22ae348aeb5660fc2140aec35850c4da997 > hashadmin.txt
hashcat -m 100 hashadmin.txt /usr/share/wordlists/rockyou.txt 

# Output
Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 7 secs

d033e22ae348aeb5660fc2140aec35850c4da997:admin            
                                                          
Session..........: hashcat
Status...........: Cracked

```

Now we have the credentials, we can log on as `admin`

![[2_admin_login.png]]


![[2_theme_php_file.png]]

