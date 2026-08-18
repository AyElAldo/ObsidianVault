#lab
# EASY LAB

We were commissioned by the company `Inlanefreight Ltd` to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.

The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.

Additionally, our teammates have found the following credentials "ceil:qwer1234", and they pointed out that some of the company's employees were talking about SSH keys on a forum.

The administrators have stored a `flag.txt` file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof.

- **Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer.**

```shell
sudo nmap --open 10.129.30.133 -Pn -T5 -p-
-----
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
53/tcp   open  domain
2121/tcp open  ccproxy-ftp
```

```shell
nc 10.129.30.133 2121 -v                                                  
------
Connection to 10.129.30.133 2121 port [tcp/iprop] succeeded!
220 ProFTPD Server (Ceil's FTP) [10.129.30.133]
```

```shell
ftp 10.129.30.133 2121                                                        
-----
Connected to 10.129.30.133.
user 220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.30.133]
```

![[Pasted image 20260207170027.png]]
## SSH PRIV KEY FOUND

![[Pasted image 20260207170010.png]]
## We use the priv key to get access via SSH

```shell
ssh ceil@<IP> -i id_rsa
```

![[Pasted image 20260207170746.png]]
# MEDIUM LAB
This second server is a server that everyone on the internal network has access to. In our discussion with our client, we pointed out that these servers are often one of the main targets for attackers and that this server should be added to the scope.

Our customer agreed to this and added this server to our scope. Here, too, the goal remains the same. We need to find out as much information as possible about this server and find ways to use it against the server itself. For the proof and protection of customer data, a user named `HTB` has been created. Accordingly, we need to obtain the credentials of this user as proof.

- **Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer:**

```shell
sudo nmap -p- 10.129.32.164 -T4 -Pn -oN Med1_scan.nmap
------
PORT      STATE SERVICE
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49679/tcp open  unknown
49680/tcp open  unknown
49681/tcp open  unknown

```
---
We collect only the port numbers and saving them in a txt file, for future actions.

```shell
cat Med1_scan.nmap | grep /tcp | cut -d '/' -f1 > Med2_ports.txt 
```
---
We discover the NFS server open, so we try to enumerate public o misconfigurations.
```shell
showmount -e 10.129.32.164 # 111,2049 NFS
----
/TechSupport (everyone)
```

We mount the directory in out machine:

```shell
sudo mount -t nfs 10.129.32.164:/TechSupport medium_target -o nolock 

sudo cd medium_target
```

After that, we found a file within the directory that contained a compromised conversation. In the conversación we can extract credentials.
```python
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"                                # <-- Credential: user: alex
 6    password="lol123!mD"                       # <-- Credential: password: lol123!mD
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
```

```mysql
# Credentials
alex
lol123!mD
```
We use this credentials in SMB 
```shell
smbclient -L //10.129.32.164 -U 'alex'
---
Password for [WORKGROUP\alex]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	devshare        Disk      
	IPC$            IPC       Remote IPC
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.32.164 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
Then, we log in devshare directory and get the file.
```shell
smbclient //10.129.32.164/devshare -U 'alex'
---
smb: \> ls
  .                                   D        0  Wed Nov 10 10:12:22 2021
  ..                                  D        0  Wed Nov 10 10:12:22 2021
  important.txt                       A       16  Wed Nov 10 10:12:55 2021

		6367231 blocks of size 4096. 2592487 blocks available
smb: \> get important.txt 
```

The file contains a text that could be a reference to credentials. Now we try RDP connection with the same credentials of Alex:
```shell
xfreerdp /u:alex /p:'lol123!mD' /v:10.129.32.164 
```

![[Pasted image 20260209004558.png]]

![[Pasted image 20260209011730.png]]
# HARD LAB
The third server is an MX and management server for the internal network. Subsequently, this server has the function of a backup server for the internal accounts in the domain. Accordingly, a user named `HTB` was also created here, whose credentials we need to access.

- **Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer.**

---

First reconanssence with Nmap

```shell
nmap -p- -Pn -sS -T4 -oA 1_hard_nmap 10.129.1.20
```

![[Pasted image 20260211135002.png]]
## Escaneo UDP
Enumeration with`onesixtyone`in order to find **community strings**. --> `backup` found.

![[Pasted image 20260211144004.png]]

```shell
snmpwalk -v2c -c backup 10.129.1.85
```
OUTPUT:
```shell
# IMPORTANT STRINGS FOUND IN THE OUTPUT
iso.3.6.1.2.1.1.4.0 = STRING: "Admin <tech@inlanefreight.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "NIXHARD"
iso.3.6.1.2.1.1.6.0 = STRING: "Inlanefreight"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
iso.3.6.1.2.1.25.1.7.1.2.1.4.6.66.65.67.75.85.80 = ""
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.3 = STRING: "chpasswd: (line 1, user tom) password not changed"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = STRING: "Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = No more variables left in this MIB View (It is past the end of the MIB tree)

```
We found credentials from an user. Even `tom` changed his password, he could use the same in other services. I tried with SSH but it is configured only with key access. So, I used `curl` to gather more info in `imap` protocol.
```shell
curl -k 'imaps://10.129.1.85' --user tom                                  ─╯
Enter host password for user 'tom':
* LIST (\HasNoChildren) "." Notes
* LIST (\HasNoChildren) "." Meetings
* LIST (\HasNoChildren \UnMarked) "." Important
* LIST (\HasNoChildren) "." INBOX
```

Once looged as tom in `IMAP`, I enumerated the emails.
![[Pasted image 20260211150635.png]]
And, after fetching the body of the INBOX mail we have a private key:
![[Pasted image 20260211150754.png]]
## SSH

![[Pasted image 20260211151003.png]]
After looking for important information within the files, the most important information was on the users file `/etc/passwd`. With the user `mysql` that could be another attack vector whether the password is the same. (**Spoiler: it was the same password**)
![[Pasted image 20260211152423.png]]
After logging in, I used basic enumeration to find the password. 

![[Pasted image 20260211152908.png]]

> [!Important] 
> Passwords in a database are typically hashed. Since the password we are looking for is weak, we can use a **custom dictionary** to crack it.


