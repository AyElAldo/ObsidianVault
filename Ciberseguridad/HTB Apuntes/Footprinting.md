#ftp #ssh #smb #NFS #DNS #SMTP #IMAP #MySQL #MSSQL #OracleTNS #IPMI #RPC
# Layers

| **Layer**                | **Description**                                                                                        | **Information Categories**                                                                         |
| ------------------------ | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `1. Internet Presence`   | Identification of internet presence and externally accessible infrastructure.                          | Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances, Security Measures      |
| `2. Gateway`             | Identify the possible security measures to protect the company's external and internal infrastructure. | Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare                  |
| `3. Accessible Services` | Identify accessible interfaces and services that are hosted externally or internally.                  | Service Type, Functionality, Configuration, Port, Version, Interface                               |
| `4. Processes`           | Identify the internal processes, sources, and destinations associated with the services.               | PID, Processed Data, Tasks, Source, Destination                                                    |
| `5. Privileges`          | Identification of the internal permissions and privileges to the accessible services.                  | Groups, Users, Permissions, Restrictions, Environment                                              |
| `6. OS Setup`            | Identification of the internal components and systems setup.                                           | OS Type, Patch Level, Network config, OS Environment, Configuration files, sensitive private files |

`The goal of this layer is to identify all possible target systems and interfaces that can be tested.`
## Layer No.2: Gateway

`The goal is to understand what we are dealing with and what we have to watch out for.`
## Layer No.3: Accessible Services

`This layer aims to understand the reason and functionality of the target system and gain the necessary knowledge to communicate with it and exploit it for our purposes effectively.`
## Layer No.4: Processes

`The goal here is to understand these factors and identify the dependencies between them.`
## Layer No.5: Privileges

`It is crucial to identify these and understand what is and is not possible with these privileges.`
## Layer No.6: OS Setup

`The goal here is to see how the administrators manage the systems and what sensitive internal information we can glean from them.`

## Certificate Transparency

https://crt.sh/ : Certificate Search

```shell
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .

[
  {
    "issuer_ca_id": 23451835427,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "matomo.inlanefreight.com",
    "name_value": "matomo.inlanefreight.com",
    "id": 50815783237226155,
    "entry_timestamp": "2021-08-21T06:00:17.173",
    "not_before": "2021-08-21T05:00:16",
    "not_after": "2021-11-19T05:00:15",
    "serial_number": "03abe9017d6de5eda90"
  },
  {
    "issuer_ca_id": 6864563267,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "matomo.inlanefreight.com",
    "name_value": "matomo.inlanefreight.com",
    "id": 5081529377,
    "entry_timestamp": "2021-08-21T06:00:16.932",
    "not_before": "2021-08-21T05:00:16",
    "not_after": "2021-11-19T05:00:15",
    "serial_number": "03abe90104e271c98a90"
  },
  {
    "issuer_ca_id": 113123452,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "smartfactory.inlanefreight.com",
    "name_value": "smartfactory.inlanefreight.com",
    "id": 4941235512141012357,
    "entry_timestamp": "2021-07-27T00:32:48.071",
    "not_before": "2021-07-26T23:32:47",
    "not_after": "2021-10-24T23:32:45",
    "serial_number": "044bac5fcc4d59329ecbbe9043dd9d5d0878"
  },
  { ... SNIP ...
```

## Filtering Unique Subdomains

### [CRT.SH (Certificate Search)](https://crt.sh/)

We can search the platform on the Internet but the next section is a differente way to gather information by scripting.

```shell
# USE CASE AUTOMATE
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```
**OUTPUT**
```shell
account.ttn.inlanefreight.com
blog.inlanefreight.com
bots.inlanefreight.com
console.ttn.inlanefreight.com
```

Next, we can identify the hosts directly accessible from the Internet and not hosted by third-party providers. This is because we are not allowed to test the hosts without the permission of third-party providers.

### Shodan

[Shodan](https://www.shodan.io/) can be used to find devices and systems permanently connected to the Internet like `Internet of Things` (`IoT`). It searches the Internet for open TCP/IP ports and filters the systems according to specific terms and criteria. For example, open HTTP or HTTPS ports and other server ports for `FTP`, `SSH`, `SNMP`, `Telnet`, `RTSP`, or `SIP` are searched.

```shell
for i in $(cat ip-addresses.txt);do shodan host $i;done
```
**OUTPUT**
```shell
10.129.24.93
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-09-01T09:02:11.370085
Number of open ports:    2

Ports:
     80/tcp nginx 
    443/tcp nginx 
	
10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
...
```

### Dig

This command is used to send DNS requests.

- `A` records: We recognize the IP addresses that point to a specific (sub)domain through the A record. Here we only see one that we already know.
- `MX` records: The mail server records show us which mail server is responsible for managing the emails for the company. Since this is handled by google in our case, we should note this and skip it for now.
- `NS` records: These kinds of records show which name servers are used to resolve the FQDN to IP addresses. Most hosting providers use their own name servers, making it easier to identify the hosting provider.
- `TXT` records: this type of record often contains verification keys for different third-party providers and other security aspects of DNS, such as [SPF](https://datatracker.ietf.org/doc/html/rfc7208), [DMARC](https://datatracker.ietf.org/doc/html/rfc7489), and [DKIM](https://datatracker.ietf.org/doc/html/rfc6376), which are responsible for verifying and confirming the origin of the emails sent. Here we can already see some valuable information if we look closer at the results.

```shell
dig any inlanefreight.com
```
**OUTPUT**
```shell
; <<>> DiG 9.16.1-Ubuntu <<>> any inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52058
;; flags: qr rd ra; QUERY: 1, ANSWER: 17, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;inlanefreight.com.             IN      ANY

;; ANSWER SECTION:
inlanefreight.com.      300     IN      A       10.129.27.33
inlanefreight.com.      300     IN      A       10.129.95.250
inlanefreight.com.      3600    IN      MX      1 aspmx.l.google.com.
...
```

# Cloud Resources

We can use Google Dorks `inurl:` and `intext:` to narrow our search to specific terms like `inurl:amazonaws.com` or `blob.core.windows.net`.

Here are examples of third party providers that can tell us about the infrastructure of a company through keywords:  
- [domain.glass](https://domain.glass/) 
- [GrayHatWarfare](https://buckets.grayhatwarfare.com/) 

It is common that administrator can make mistakes like not filtering the content that will be public. A common active that tends to filter is the SSH Private Key of the devices.
# File Transfer Protocol (FTP)

> [!info] FTP Relevant Information
> [Link to commands](https://web.archive.org/web/20230326204635/https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/)  
> [Link to status codes](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes)
> > It runs in the Application Layer **TCP/IP**
> ---
> - **Active Mode**: The client stablishes the connection via TCP port 21 and specifies to the server the port where the client is gonna listen. But, if the client is being protected by a firewall, the server cannot stablish the connection.
> - **Passive Mode**: The server is listening and when the client asks it for passive mode, the server gives the client a port where the client has to start the connection. Through this way, the firewall won't block the connection.
> ---
> ## Features
> - `/etc/ftpusers`: This file is used to **deny certain users access** to the FTP service.
> - `/etc/vsftpd.conf`: One of the most common Linux FTP servers.
> ---
> ## Trivial File Transfer Protocol (TFTP)
>It uses UDP and doesn't require user authentication.
>
>| **TFTP Commands** | **Description** |
> | :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
> | `connect`         | Sets the remote host, and optionally the port, for file transfers.                                                                     |
> | `get`             | Transfers a file or set of files from the remote host to the local host.                                                               |
> | `put`             | Transfers a file or set of files from the local host onto the remote host.                                                             |
> | `quit`            | Exits tftp. |
> | `status`          | Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on. |
> | `verbose`         | Turns verbose mode, which displays additional information during file transfer, on or off. |
> ## **FTP Dangerous Settings (ANONYMOUS)**
>- `anonymous_enable=YES`_Allowing anonymous login?
>- `anon_upload_enable=YES` Allowing anonymous to upload files?
>- `anon_mkdir_write_enable=YES`Allowing anonymous to create new directories?
>- `no_anon_password=YES`Do not ask anonymous for password?
>- `anon_root=/home/username/ftp`Directory for anonymous.
>- `write_enable=YES`Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?
>## **Commands**
>- `debug`
>- `trace`
>- `status`
>
> ##  Misconfigurations
|**Setting**|**Description**|
|---|---|
|`dirmessage_enable=YES`|Show a message when they first enter a new directory?|
|`chown_uploads=YES`|Change ownership of anonymously uploaded files?|
|`chown_username=username`|User who is given ownership of anonymously uploaded files.|
|`local_enable=YES`|Enable local users to login?|
|`chroot_local_user=YES`|Place local users into their home directory?|
|`chroot_list_enable=YES`|Use a list of local users that will be placed in their home directory?|
>---
> ## Good Practice
> |**Setting**|**Description**|
|---|---|
|`hide_ids=YES`|All user and group information in directory listings will be displayed as "ftp".|
> This setting is a security feature to prevent local usernames from being revealed. With the usernames, we could attack the services like FTP and SSH and many others with a brute-force attack in theory.
> ## Offensive FTP
> Besides **GET** command, we can use **wget** to download all the files and folders at once.
> ```shell
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136 
>```

## FTP Scripts

Scripts are locate in:`/usr/share/nmap/scripts/`

 We can look for the FTP scripts with:
```shell
find / -type f -name ftp* 2>/dev/null | grep scripts
```
**OUTPUT**
```shell
ftp-anon.nse
ftp-bounce.nse
ftp-brute.nse
ftp-libopie.nse
ftp-proftpd-backdoor.nse
ftp-syst.nse
ftp-vsftpd-backdoor.nse
ftp-vuln-cve2010-4221.nse
tftp-enum.nse
tftp-version.nse
```

## TLS/SSL

Sometimes, the server can be run with SSL encryption. The good thing about using `openssl` is that we can see the SSL certificate, which can also be helpful.

```shell
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

## Lab

1. Which version of the FTP server is running on the target system? Submit the entire banner as the answer.

We can use nmap `sudo nmap <IP> -sV -sC -p21` or netcat `nc <ip>`

2. Enumerate the FTP server and find the flag.txt file. Submit the contents of it as the answer.

We connect to the FTP server and we log as `anonymous` user. 

![[Pasted image 20260115170447.png]]

---
#  Server Message Block (SMB)

> [!info] SMB Relevant Information 
> It is a client-server protocol that regulates access to files and entire directories and other network resources such as printers, routers, or interfaces released for the network.
> ## SAMBA
> There is an alternative implementation of the SMB server called Samba, which is developed for Unix-based operating systems. Samba implements the Common Internet File System (`CIFS`) network protocol. [CIFS](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs/934c2faa-54af-4526-ac74-6a24d126724e) is a dialect of SMB, meaning it is a specific implementation of the SMB protocol originally created by Microsoft. This allows Samba to communicate effectively with newer Windows systems. Therefore, it is often referred to as SMB/CIFS.
> ### **TCP Ports**
> - 137
> - 138
> - 139
> - **445 (CIFS)**
> ## **Features**
> `/etc/samba/smb.conf`
> ### **Versions**
> | **SMB Version** | **Supported**                       | **Features**                                                           |
| --------------- | ----------------------------------- | ---------------------------------------------------------------------- |
| CIFS            | Windows NT 4.0                      | Communication via NetBIOS interface                                    |
| SMB 1.0         | Windows 2000                        | Direct connection via TCP                                              |
| SMB 2.0         | Windows Vista, Windows Server 2008  | Performance upgrades, improved message signing, caching feature        |
| SMB 2.1         | Windows 7, Windows Server 2008 R2   | Locking mechanisms                                                     |
| SMB 3.0         | Windows 8, Windows Server 2012      | Multichannel connections, end-to-end encryption, remote storage access |
| SMB 3.0.2       | Windows 8.1, Windows Server 2012 R2 |                                                                        |
| SMB 3.1.1       | Windows 10, Windows Server 2016     | Integrity checking, AES-128 encryption                                 |
## Dangerous Settings

| **Setting**                 | **Description**                                                     |
| --------------------------- | ------------------------------------------------------------------- |
| `browseable = yes`          | Allow listing available shares in the current share?                |
| `read only = no`            | Forbid the creation and modification of files?                      |
| `writable = yes`            | Allow users to create and modify files?                             |
| `guest ok = yes`            | Allow connecting to the service without using a password?           |
| `enable privileges = yes`   | Honor privileges assigned to specific SID?                          |
| `create mask = 0777`        | What permissions must be assigned to the newly created files?       |
| `directory mask = 0777`     | What permissions must be assigned to the newly created directories? |
| `logon script = script.sh`  | What script needs to be executed on the user's login?               |
| `magic script = script.sh`  | Which script should be executed when the script gets closed?        |
| `magic output = script.out` | Where the output of the magic script needs to be stored?            |

> [!Info] Example with dangerous settings
> Connecting to the share. We can display a list (`-L`) of the server's shares with the `smbclient` command from our host. We use the so-called `null session` (`-N`), which is `anonymous` access without the input of existing users or valid passwords.
>```shell
smbclient -N -L //10.129.14.128
>```
>**OUTPUT**
>```shell
Sharename       Type      Comment
> ------------------------------------
print$          Disk      Printer Drivers
home            Disk      INFREIGHT Samba
dev             Disk      DEVenv
notes           Disk      CheckIT
IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
> ```
> Once we have our goal, we can access directly:
> ```shell
> smbclient //10.129.14.128/notes
> ```
> Then. we can use `ls` and `get` commands to exfiltrate our targets.
## SAMBA

`smbstatus`
```shell
sudo nmap 10.129.14.128 -sV -sC -p139,445
```
 
## Footprinting the Service
 
NMAP
```shell
sudo nmap 10.129.14.128 -sV -sC -p139,445
```
## RPC client

The `rpcclient` offers us many different requests with which we can execute specific functions on the SMB server to get information. A complete list of all these functions can be found on the [man page](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) of the rpcclient.

**ANONYMOUS**
```shell
rpcclient -U "" 10.129.14.128
```

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | **Enumerate all domains that are deployed in the network.**        |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | **Provides information about a specific user.**                    |
| `querygroup <RID>`        | **Provides information about a specific group.**                   |
## Enumeration 

```python
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

```python
samrdump.py 10.129.14.128 # Impacket --> samrdump.py
```

```python
smbmap -H 10.129.14.128
```

```python
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

> [!info]- SMB LAB SOLUTION
> Version
> ```python
> sudo nmap -sC -sV 10.129.202.5 -p137-139,445 
> smbclient -N -L \\10.129.2.54 ## List the shared folders
> smbclient -L -U '' //10.129.202.5 # No User
> enum4linux 10.129.202.5
> smbmap -H 10.129.202.5
>rpcclient 10.129.202.5 -U ''  ## --> netsharegetinfo
> ```

# NFS

> [!info]- NFS Relevant Information
> It is a network file system developed by Sun Microsystems and has the same purpose as SMB. Its purpose is to access file systems over a network as if they were local. However, it uses an entirely different protocol. [NFS](https://en.wikipedia.org/wiki/Network_File_System) is used between Linux and Unix systems. This means that NFS clients cannot communicate directly with SMB servers.
> ## **TCP/UDP Ports** 
> - 111 
> - 2049
> 
> |**Version**|**Features**|
|---|---|
|`NFSv2`|It is older but is supported by many systems and was initially operated entirely over UDP.|
|`NFSv3`|It has more features, including variable file size and better error reporting, but is not fully compatible with NFSv2 clients.|
|`NFSv4`|It includes Kerberos, works through firewalls and on the Internet, no longer requires portmappers, supports ACLs, applies state-based operations, and provides performance improvements and high security. It is also the first version to have a stateful protocol.|
> NFS is not difficult to configure because there are not as many options as FTP or SMB have. The `/etc/exports` file contains a table of physical filesystems on an NFS server accessible by the clients. The [NFS Exports Table](http://manpages.ubuntu.com/manpages/trusty/man5/exports.5.html) shows which options it accepts and thus indicates which options are available to us.
> ## ExportFS
> ```shell-session
>> echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
>> systemctl restart nfs-kernel-server 
>> exportfs
>```
>We have shared the folder `/mnt/nfs` to the subnet `10.129.14.0/24` with the setting shown above. This means that all hosts on the network will be able to mount this NFS share and inspect the contents of this folder.
## Dangerous Settings

| **Option**       | **Description**                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `rw`             | Read and write permissions.                                                                                          |
| `insecure`       | Ports above 1024 will be used.                                                                                       |
| `nohide`         | If another file system was mounted below an exported directory, this directory is exported by its own exports entry. |
| `no_root_squash` | All files created by root are kept with the UID/GID 0.                                                               |
## Footprinting the service

When footprinting NFS, the TCP ports `111` and `2049` are essential

```python
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
```

```python
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
```

> [!info]- NFS Enumeration
> ## Mounting NFS Share
>```shell
AyElAldo@htb[/htb]$ mkdir target-NFS
AyElAldo@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock # -t especifies the protocol used
AyElAldo@htb[/htb]$ cd target-NFS
AyElAldo@htb[/htb]$ tree .
>```
>## List Usernames & Group Names
>```shell-session
ls -l mnt/nfs/
>```
> ## List Contests with UIDs & GUIDs
> ```shell
ls -n mnt/nfs/
>```
>```shell-session
sudo umount ./target-NFS
>```

> [!tip] Tip
> For example, if we have access to the system via SSH and want to read files from another folder that a specific user can read, we would need to upload a shell to the NFS share that has the `SUID` of that user and then run the shell via the SSH user.

> [!info]- NFS LAB
> 1. Enumerate the NFS service and submit the contents of the flag.txt in the "nfs" share as the answer.
> 2. Enumerate the NFS service and submit the contents of the flag.txt in the "nfsshare" share as the answer.
> `showmount -e <IP>`
> `sudo mount 10.129.202.5:/ -t nfs ./nfs -o nolock`

# DNS

> [!info]- DNS Relevant Information
>  > **DNS** is a system for resolving computer names into IP addresses, and it does not have a central database.
>   The information is distributed over many thousands of name servers. Globally distributed DNS servers translate domain names into IP addresses and thus control which server a user can reach via a particular domain.
>   ## Port TCP
>   - Port 53
>   ## Details
> - `dig`: Command
> - `bind9`: Linux DNS server 
> 
|**Server Type**|**Description**|
|---|---|
|`DNS Root Server`|The root servers of the DNS are responsible for the top-level domains (`TLD`). As the last instance, they are only requested if the name server does not respond. Thus, a root server is a central interface between users and content on the Internet, as it links domain and IP address. The [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/) (`ICANN`) coordinates the work of the root name servers. There are `13` such root servers around the globe.|
|`Authoritative Nameserver`|Authoritative name servers hold authority for a particular zone. They only answer queries from their area of responsibility, and their information is binding. If an authoritative name server cannot answer a client's query, the root name server takes over at that point. Based on the country, company, etc., authoritative nameservers provide answers to recursive DNS nameservers, assisting in finding the specific web server(s).|
|`Non-authoritative Nameserver`|Non-authoritative name servers are not responsible for a particular DNS zone. Instead, they collect information on specific DNS zones themselves, which is done using recursive or iterative DNS querying.|
|`Caching DNS Server`|Caching DNS servers cache information from other name servers for a specified period. The authoritative name server determines the duration of this storage.|
|`Forwarding Server`|Forwarding servers perform only one function: they forward DNS queries to another DNS server.|
|`Resolver`|Resolvers are not authoritative DNS servers but perform name resolution locally in the computer or router.|
>A DNS query can therefore also be used, for example, to determine which computer serves as the e-mail server for the domain in question or what the domain's name servers are called.
>## DNS Records
>| **DNS Record** | **Description**                                                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`            | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                      |
| `AAAA`         | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                  |
| `MX`           | Returns the responsible mail servers as a result.                                                                                                                                                                                                 |
| `NS`           | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                              |
| `TXT`          | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam. |
| `CNAME`        | This record serves as an alias for another domain name. If you want the domain www.hackthebox.eu to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for www.hackthebox.eu.               |
| `PTR`          | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                     |
| `SOA`          | Provides information about the corresponding DNS zone and email address of the administrative contact.                                                                                                                                            |
> The `SOA` record is located in a domain's zone file and specifies who is responsible for the operation of the domain and how DNS information for the domain is managed.
>```python
>dig soa www.inlanefreight.com # Example
>```
>All DNS servers work with three different types of configuration files:
>1. local DNS configuration files
>2. zone files
>3. reverse name resolution files

## Default Configuration

The DNS server [Bind9](https://www.isc.org/bind/) is very often used on Linux-based distributions. Its local configuration file (`named.conf`) is roughly divided into two sections, firstly the options section for general settings and secondly the zone entries for the individual domains. The local configuration files are usually:

- `named.conf.local`
- `named.conf.options`
- `named.conf.log`

Global options are general and affect all zones. A zone option only affects the zone to which it is assigned. Options not listed in named.conf have default values. If an option is both global and zone-specific, then the zone option takes precedence.

## Local DNS Configuration

```python
cat /etc/bind/named.conf.local
```

In this file, we can define the different zones. These zones are divided into individual files, which in most cases are mainly intended for one domain only. Exceptions are ISP and public DNS servers. In addition, many different options extend or reduce the functionality. We can look these up on the [documentation](https://wiki.debian.org/Bind9) of Bind9.

```shell
cat /etc/bind/db.domain.com # Zone files
```

>[!note] 
>A zone file describes a zone completely. There must be precisely one `SOA` record and at least one `NS` record.
## Dangerous Settings

 A list of vulnerabilities targeting the BIND9 server can be found at [CVEdetails](https://www.cvedetails.com/product/144/ISC-Bind.html?vendor_id=64). In addition, SecurityTrails provides a short [list](https://web.archive.org/web/20250329174745/https://securitytrails.com/blog/most-popular-types-dns-attacks) of the most popular attacks on DNS servers.

|**Option**|**Description**|
|---|---|
|`allow-query`|Defines which hosts are allowed to send requests to the DNS server.|
|`allow-recursion`|Defines which hosts are allowed to send recursive requests to the DNS server.|
|`allow-transfer`|Defines which hosts are allowed to receive zone transfers from the DNS server.|
|`zone-statistics`|Collects statistical data of zones.|
## Footprinting the service
#### DIG - AXFR Zone Transfer

```python
dig axfr inlanefreight.htb @10.129.14.128
```
It returns all the subdomains, IP adresses and registers.
### DIG - AXFR Zone Transfer - Internal

```python
dig axfr internal.inlanefreight.htb @10.129.14.128
```
## Subdomain Brute Forcing

The individual `A` records with the hostnames can also be found out with the help of a brute-force attack bu using [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt).

```python
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

```python
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

> [!info]- DNS LAB
> - **Interact with the target DNS using its IP address and enumerate the FQDN of it for the "inlanefreight.htb" domain.**
> ```shell
> dig axfr inlanefreight.htb @10.129.116.138
> ```
> - **Identify if its possible to perform a zone transfer and submit the TXT record as the answer. (Format: HTB{...})**
> For a quicker work, I separated the domains in a differente file and do this:
> ```shell
> for i in $(cat subdominios); do dig axfr $i @10.129.116.138;done
> ```
> One of them will return a valid response.
> - **What is the FQDN of the host where the last octet ends with "x.x.x.203"?**
> Even if we use `dig axfr`, we cannot see the entire available subdomains or zones. That's why we use brute force.
> In order to answer this program, we need to try and error many times. I look for each domain and each file of the sexlists until the good one contained the correct subdomain `fierce-hostlist.txt`.
> ```shell
> # Many Attempts
> dnsenum --dnsserver 10.129.236.209 --enum -p 0 -s 0 -o subdom.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt  inlanefreight.htb
> # Good One
> for i in $(cat subdominios);do dnsenum --dnsserver 10.129.236.209 --enum -p 0 -s 0 -o recursive.subdom -f /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt $i; done
> ```

# SMTP

> [!info] SMTP Relevant Information
> The `Simple Mail Transfer Protocol` (`SMTP`) is a protocol for sending emails in an IP network. It can be used between an email client and an outgoing mail server or between two SMTP servers. SMTP is often combined with the IMAP or POP3 protocols, which can fetch emails and send emails
> ## **TCP Ports**
> - Port 25
> - Port 465 (Conexión cifrada)
> - Port 587 **STARTTLS** (newer servers)
> ## Details
> The authentication data is protected and no longer visible in plaintext over the network. At the beginning of the connection, authentication occurs when the client confirms its identity with a user name and password.
> SMTP works unencrypted; to keep the confidenciality the SMTP is used in conjunction with SSL/TLS encryption.
>
>| Client (`MUA`) | `➞` | Submission Agent (`MSA`) | `➞` | Open Relay (`MTA`) | `➞` | Mail Delivery Agent (`MDA`) | `➞` | Mailbox (`POP3`/`IMAP`) |
| -------------- | --- | ------------------------ | --- | ------------------ | --- | --------------------------- | --- | ----------------------- |
>## ESMPT
> When people talk about SMTP in general, they usually mean ESMTP. ESMTP uses TLS, which is done after the `EHLO` command by sending `STARTTLS`. This initializes the SSL-protected SMTP connection, and from this moment on, the entire connection is encrypted, and therefore more or less secure.
> ## Default Configuration
> ```shell
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
>```
## Commands

|**Command**|**Description**|
|---|---|
|`AUTH PLAIN`|AUTH is a service extension used to authenticate the client.|
|`HELO`|The client logs in with its computer name and thus starts the session.|
|`MAIL FROM`|The client names the email sender.|
|`RCPT TO`|The client names the email recipient.|
|`DATA`|The client initiates the transmission of the email.|
|`RSET`|The client aborts the initiated transmission but keeps the connection between client and server.|
|`VRFY`|The client checks if a mailbox is available for message transfer.|
|`EXPN`|The client also checks if a mailbox is available for messaging with this command.|
|`NOOP`|The client requests a response from the server to prevent disconnection due to time-out.|
|`QUIT`|The client terminates the session.|
>[!note] 
>We can use **telnet** to  inicialize a TCP connection with the SMTP Server.

## Dangerous Configurations

Whit this, the SMTP administrator allows all the network to have communication each other but it can be dangerous.
```shell
mynetworks = 0.0.0.0/0
```
## Footprinting the service

```shell
sudo nmap 10.129.14.128 -sC -sV -p25
```
### Nmap - Open Relay
However, we can also use the [smtp-open-relay](https://nmap.org/nsedoc/scripts/smtp-open-relay.html) NSE script to identify the target SMTP server as an open relay using 16 different tests.
```shell
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

>[!note] 
>It will show the vulnerabilities according to the tests. The more tests passed, the more unprotected the SMTP server is.
## SMTP LAB

> [!info]- SMTP LAB
> 1. **Enumerate the SMTP service and submit the banner, including its version as the answer.**
> ```shell
> sudo nmap <IP> -sV -sC -p25
> ```
> O usamos telnet o netcat
> 2. **Enumerate the SMTP service even further and find the username that exists on the system. Submit it as the answer.**
> ```shell
> # You have to be very patient, it could take a lot of time
> smtp-user-enum -t 10.129.54.96 -M VRFY -u robin -vv -w 15 
> ```

# IMAP / POP3

> [!info] IMAP/POP3 Relevant Information
> It is possible to access to emails from a mail server thanks to **Internet Message Access Protocol** (IMAP). It is a client-server protocol that allows synchronization accoss several independent clients. On the other hand, **Post Office Protocol** (POP3), once a client gets its emails, the server deletes the emails. So, if a client gets his emails form his PC, in his phone we are not be able to access.  
> Immediately after the connection is established, the user is authenticated by user name and password to the server. Access to the desired mailbox is only possible after successful authentication.  
> Without further measures, **IMAP** works unencrypted and transmits commands, emails, or usernames and passwords in plain text. Many email servers require establishing an encrypted IMAP session to ensure greater security in email traffic and prevent unauthorized access to mailboxes. **SSL/TLS** is usually used for this purpose.
> ## Services
> - `dovecot-imapd`
> - `dovecot-pop3d`
> ## Ports
> ### **IMAP**
> - 143
> - 993 (TLS)
> ### **POP3**
> - 110
> - 995 (TLS)
> ## IMAP Commands
> |**Command**|**Description**|
|---|---|
|`1 LOGIN username password`|User's login.|
|`1 LIST "" *`|Lists all directories.|
|`1 CREATE "INBOX"`|Creates a mailbox with a specified name.|
|`1 DELETE "INBOX"`|Deletes a mailbox.|
|`1 RENAME "ToRead" "Important"`|Renames a mailbox.|
|`1 LSUB "" *`|Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`.|
|`1 SELECT INBOX`|Selects a mailbox so that messages in the mailbox can be accessed.|
|`1 UNSELECT INBOX`|Exits the selected mailbox.|
|`1 FETCH <ID> all`|Retrieves data associated with a message in the mailbox.|
|`1 CLOSE`|Removes all messages with the `Deleted` flag set.|
|`1 LOGOUT`|Closes the connection with the IMAP server.|
>## POP3 Commands
>|   |   |
|---|---|
|`USER username`|Identifies the user.|
|`PASS password`|Authentication of the user using its password.|
|`STAT`|Requests the number of saved emails from the server.|
|`LIST`|Requests from the server the number and size of all emails.|
|`RETR id`|Requests the server to deliver the requested email by ID.|
|`DELE id`|Requests the server to delete the requested email by ID.|
|`CAPA`|Requests the server to display the server capabilities.|
|`RSET`|Requests the server to reset the transmitted information.|
|`QUIT`|Closes the connection with the POP3 server.|

## Dangerous Settings

| **Setting**               | **Description**                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `auth_debug`              | Enables all authentication debug logging.                                                 |
| `auth_debug_passwords`    | This setting adjusts log verbosity, the submitted passwords, and the scheme gets logged.  |
| `auth_verbose`            | Logs unsuccessful authentication attempts and their reasons.                              |
| `auth_verbose_passwords`  | Passwords used for authentication are logged and can also be truncated.                   |
| `auth_anonymous_username` | This specifies the username to be used when logging in with the ANONYMOUS SASL mechanism. |
## Footprinting the service

```shell
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC
```

```shell
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd # -k: Insecure

curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v
```

### OpenSSL TLS 

#### POP3

```shell
openssl s_client -connect 10.129.14.128:pop3s
```
#### IMAP

```shell
openssl s_client -connect 10.129.14.128:imaps
```

> [!info]- LAB SOLUTION IMAP/POP3
> ```shell
> # 1. Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer.
> # 2 What is the FQDN that the IMAP and POP3 servers are assigned to?
> sudo nmap <IP> -p110,143,993,995 
> # 3. Enumerate the IMAP service and submit the flag as the answer. (Format: HTB{...})
> nc -nv 10.129.5.103 143 
> # 4. What is the customized version of the POP3 server?
> nc -nv 10.129.5.103 110
> # 5. What is the admin email address?
> openssl s_client -connect <ip>:imaps
> 1 LOGIN robin robin
> 1 LIST "" *
>1 SELECT DEV.DEPARTMENT.INT
>1 FETCH 1 all
> ``` 
> ![[Pasted image 20260123214142.png]]
> ```shell
> # 6.Try to access the emails on the IMAP server and submit the flag as the answer. (Format: HTB{...})
> 1 FETCH 1 BODY.PEEK[TEXT]
> ```

# SNMP

> [!info] SNMP Relevant Information
> `Simple Network Management Protocol` ([SNMP](https://datatracker.ietf.org/doc/html/rfc1157)) was created to monitor network devices.
> For the SNMP client and server to exchange the respective values, the available SNMP objects must have unique addresses known on both sides. This addressing mechanism is an absolute prerequisite for successfully transmitting data and network monitoring using SNMP.
> ## Ports
> - 161 (UDP)
> - 162 (UDP)
> ## Daemon config
> ```shell
> cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
> ```
>System Object Identificator (OID):  `.1.3.6.1.2.1.1.5.0`

## Dangerous Settings

|**Settings**|**Description**|
|---|---|
|`rwuser noauth`|Provides access to the full OID tree without authentication.|
|`rwcommunity <community string> <IPv4 address>`|Provides access to the full OID tree regardless of where the requests were sent from.|
|`rwcommunity6 <community string> <IPv6 address>`|Same access as with `rwcommunity` with the difference of using IPv6.|
## Footprinting the service

We can use tools like `snmpwalk`, `onesixtyone`, and `braa`. `Snmpwalk` is used to query the OIDs with their information. `Onesixtyone` can be used to brute-force the names of the community strings since they can be named arbitrarily by the administrator.

### SNMPWALK
```shell
snmpwalk -v2c -c public 10.129.14.128 # Version 2c
```

Here we recognize some Python packages that have been installed on the system. If we do not know the community string, we can use `onesixtyone` and `SecLists` wordlists to identify these community strings.
### ONESIXTYONE
```shell
sudo apt install onesixtyone
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```
### BRAA
```shell
sudo apt install braa
braa <community string>@<IP>:.1.3.6.*   # Syntax
braa public@10.129.14.128:.1.3.6.*      # Example
```

> [!info]- SNMB Lab
> ```shell
> #If we don't know the community, we can use:
> onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.4.53
> # We use public 
> snmpwalk -v2c 10.129.4.53 -c public
> ```

# MySQL

> [!info] MySQL Relevant Information 
> `MySQL` is an open-source SQL relational database management system developed and supported by Oracle. A database is simply a structured collection of data organized for easy use and retrieval. The database system can quickly process large amounts of data with high performance.
> # Ports
> - 3306
> # Default Configuration
> ```shell
sudo apt install mysql-server -y
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
>``` 

## Dangerous Settings
|**Settings**|**Description**|
|---|---|
|`user`|Sets which user the MySQL service will run as.|
|`password`|Sets the password for the MySQL user.|
|`admin_address`|The IP address on which to listen for TCP/IP connections on the administrative network interface.|
|`debug`|This variable indicates the current debugging settings|
|`sql_warnings`|This variable controls whether single-row INSERT statements produce an information string if warnings occur.|
|`secure_file_priv`|This variable is used to limit the effect of data import and export operations.|
The settings `user`, `password`, and `admin_address` are security-relevant because the entries are made in plain text. Often, the rights for the configuration file of the MySQL server are not assigned correctly. If we get another way to read files or even a shell, we can see the file and the username and password for the MySQL server.

The `debug` and `sql_warnings` settings provide verbose information output in case of errors, which are essential for the administrator but should not be seen by others.
## Footprinting the service

```shell
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*
```

> [!warning]
>  As with all our scans, we must be careful with the results and manually confirm the information obtained because some of the information might turn out to be a false-positive.

```shell
mysql -u root -h 10.129.14.132
```

```shell
mysql -u root -pP4SSw0rd -h 10.129.14.128 # Password must be without space in the flag (-p)
```

If we look at the existing databases, we will see several already exist. The most important databases for the MySQL server are the `system schema` (`sys`) and `information schema` (`information_schema`). The system schema contains tables, information, and metadata necessary for management.

| **Command**                                          | **Description**                                                                                       |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `mysql -u <user> -p<password> -h <IP address>`       | Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password. |
| `show databases;`                                    | Show all databases.                                                                                   |
| `use <database>;`                                    | Select one of the existing databases.                                                                 |
| `show tables;`                                       | Show all available tables in the selected database.                                                   |
| `show columns from <table>;`                         | Show all columns in the selected table.                                                               |
| `select * from <table>;`                             | Show everything in the desired table.                                                                 |
| `select * from <table> where <column> = "<string>";` | Search for needed `string` in the desired table.                                                      |
|                                                      |                                                                                                       |
> [!info]- MySQL Lab
> ```shell
> # 1. Enumerate the MySQL server and determine the version in use
> nmap -sV -sC -p3306 <IP>
> ```
> ```shell
> # 2. During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"?
> mysql -u robin -probin -h 10.129.42.195
> ```
> ```sql
> # 2. Continuation
> show databases
> use myTable
> show columns FROM myTable
> SELECT email FROM myTable WHERE name = 'Otto Lang';
> ```
# MSSQL (Microsoft SQL)

>[!Info] MSSQL Relevant information
>[Microsoft SQL](https://www.microsoft.com/en-us/sql-server/sql-server-2019) (`MSSQL`) is Microsoft's SQL-based relational database management system. Unlike MySQL, which we discussed in the last section, MSSQL is closed source and was initially written to run on Windows operating systems. It is popular among database administrators and developers when building applications that run on Microsoft's .NET framework due to its strong native support for .NET.
>## Ports
>- 1433
>## MSSQL Clients
>[SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) (`SSMS`) comes as a feature that can be installed with the MSSQL install package or can be downloaded & installed separately.
> It doesn't only exist on the server hosting the database. This means we could come across a vulnerable system with SSMS with saved credentials that allow us to connect to the database.
>- mssql-cli
>- SQL Server PowerShell
>- HeidiSQL
>- SQLPro
>- **Impacket's mssqlclient.py**
>### MSSQLCLIENT.PY
>```shell
>locate mssqlclient
>```
>Authenticating with MSSQL will enable us to interact directly with databases through the SQL Database Engine.
>If we can guess or gain access to credentials, this allows us to remotely connect to the MSSQL server and start interacting with databases using T-SQL (`Transact-SQL`). 
>```shell
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
>```
>## MSSQL Databases
>|Default System Database|Description|
|---|---|
|`master`|Tracks all system information for an SQL server instance|
|`model`|Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database|
|`msdb`|The SQL Server Agent uses this database to schedule jobs & alerts|
|`tempdb`|Stores temporary objects|
|`resource`|Read-only database containing system objects included with SQL server|
## Dangerous Settings

This is not an extensive list because there are countless ways MSSQL databases can be configured by admins based on the needs of their respective organizations. We may benefit from looking into the following:

- MSSQL clients not using encryption to connect to the MSSQL server
- The use of self-signed certificates when encryption is being used. It is possible to spoof self-signed certificates
- The use of [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15)
- Weak & default `sa` credentials. Admins may forget to disable this account
## Footprinting the service

We can see the `hostname`, `database instance name`, `software version of MSSQL` and `named pipes are enabled`

```shell
# Using nmap
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

```shell
# Using Metasploit
scanner/mssql/mssql_ping
```

> [!info]- MSSQL LAB
> ```shell
> # 1. Enumerate the target using the concepts taught in this section. List the hostname of MSSQL server.
> msfconsole
> use  auxiliary/scanner/mssql/mssql_ping
> ```
> ```sql
> # 2. Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server.
> python3 mssqlclient.py backdoor@<IP> -windows-auth
> SELECT name FROM sys.databases
> # or
> enum_db
> ```
# Oracle TNS

> [!info] Oracle TNS Relevant Information 
> The `Oracle Transparent Network Substrate` (`TNS`) server is a communication protocol that facilitates communication between Oracle databases and applications over networks.
> Over time, TNS has been updated to support newer technologies, including `IPv6` and `SSL/TLS` encryption which makes it more suitable for the following purposes:
>## Ports
>- 1521
>## Default Configuration
>The TNS listener is configured to support various network protocols, including `TCP/IP`, `UDP`, `IPX/SPX`, and `AppleTalk`. The listener can also support multiple network interfaces and listen on specific IP addresses or all available network interfaces. By default, Oracle TNS can be remotely managed in `Oracle 8i`/`9i` but not in Oracle 10g/11g.
>The configuration files for Oracle TNS are called `tnsnames.ora` and `listener.ora` and are typically located in the `$ORACLE_HOME/network/admin` directory. The plain text file contains configuration information for Oracle database instances and other network services that use the TNS protocol.
>### Example TNSNAMES.ORA
>```txt
>ORCL =
  >(DESCRIPTION =
 >(ADDRESS_LIST =
  > (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
  >)
>    (CONNECT_DATA =
 >     (SERVER = DEDICATED)
  >    (SERVICE_NAME = orcl)
>    )
 > )
>```
>Here we can see a service called `ORCL`, which is listening on port `TCP/1521` on the IP address `10.129.11.102`. Clients should use the service name `orcl` when connecting to the service
>### Example LISTENER.ORA
>```txt
>SID_LIST_LISTENER =
 > (SID_LIST =
>    (SID_DESC =
>      (SID_NAME = PDB1)
>      (ORACLE_HOME = C:\oracle\product\19.0.0\dbhome_1)
>      (GLOBAL_DBNAME = PDB1)
>      (SID_DIRECTORY_LIST =
>        (SID_DIRECTORY =
>          (DIRECTORY_TYPE = TNS_ADMIN)
>          (DIRECTORY = C:\oracle\product\19.0.0\dbhome_1\network\admin)
>        )
>      )
  >  )
 > )
>
>LISTENER =
  >(DESCRIPTION_LIST =
>    (DESCRIPTION =
>      (ADDRESS = (PROTOCOL = TCP)(HOST = orcl.inlanefreight.htb)(PORT = 1521))
>      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
>    )
  >)
>
ADR_BASE_LISTENER = C:\oracle
>```

> [!Important] 
> In short, the client-side Oracle Net Services software uses the `tnsnames.ora` file to resolve service names to network addresses, while the listener process uses the `listener.ora` file to determine the services it should listen to and the behavior of the listener.
## Settings
| **Setting**          | **Description**                                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `DESCRIPTION`        | A descriptor that provides a name for the database and its connection type.                                              |
| `ADDRESS`            | The network address of the database, which includes the hostname and port number.                                        |
| `PROTOCOL`           | The network protocol used for communication with the server                                                              |
| `PORT`               | The port number used for communication with the server                                                                   |
| `CONNECT_DATA`       | Specifies the attributes of the connection, such as the service name or SID, protocol, and database instance identifier. |
| `INSTANCE_NAME`      | The name of the database instance the client wants to connect.                                                           |
| `SERVICE_NAME`       | The name of the service that the client wants to connect to.                                                             |
| `SERVER`             | The type of server used for the database connection, such as dedicated or shared.                                        |
| `USER`               | The username used to authenticate with the database server.                                                              |
| `PASSWORD`           | The password used to authenticate with the database server.                                                              |
| `SECURITY`           | The type of security for the connection.                                                                                 |
| `VALIDATE_CERT`      | Whether to validate the certificate using SSL/TLS.                                                                       |
| `SSL_VERSION`        | The version of SSL/TLS to use for the connection.                                                                        |
| `CONNECT_TIMEOUT`    | The time limit in seconds for the client to establish a connection to the database.                                      |
| `RECEIVE_TIMEOUT`    | The time limit in seconds for the client to receive a response from the database.                                        |
| `SEND_TIMEOUT`       | The time limit in seconds for the client to send a request to the database.                                              |
| `SQLNET.EXPIRE_TIME` | The time limit in seconds for the client to detect a connection has failed.                                              |
| `TRACE_LEVEL`        | The level of tracing for the database connection.                                                                        |
| `TRACE_DIRECTORY`    | The directory where the trace files are stored.                                                                          |
| `TRACE_FILE_NAME`    | The name of the trace file.                                                                                              |
| `LOG_FILE`           | The file where the log information is stored.                                                                            |
## Setting Up

```shell
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
sudo mkdir -p /opt/oracle
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.4.0.0.0dbru.zip
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.4.0.0.0dbru.zip
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH
source ~/.bashrc
cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
pip3 install cx_Oracle
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```

## Testing ODAT

Oracle Database Attacking Tool (`ODAT`) is an open-source penetration testing tool written in Python and designed to enumerate and exploit vulnerabilities in Oracle databases. It can be used to identify and exploit various security flaws in Oracle databases, including SQL injection, remote code execution, and privilege escalation.

```shell
./odat.py -h
```
## NMAP

```shell
sudo nmap -p1521 -sV 10.129.204.235 --open
```

In Oracle RDBMS, a System Identifier (`SID`) is a unique name that identifies a particular database instance. It can have multiple instances, each with its own System ID. There are various ways to enumerate, or better said, guess SIDs. Therefore we can use tools like `nmap`, `hydra`, `odat`, and others. Let us use `nmap` first.

```shell
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```
### ODAT
```shell
./odat.py all -s 10.129.204.235
```
### SQLPLUS
```shell
sqlplus scott/tiger@10.129.204.235/XE # XE: ENCONTRADA EN NMAP | SCOT:TIGER ENCONTRADA EN ODAT
```
**If you come across the following error `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`, please execute the below, taken from [here](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared).**
```shell
#### ONLY IF ERROR IN THE PREVIOUS
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```
## Oracle RDBMS - Database Enumeration

```shell
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```
## Oracle RDBMS - Extract Password Hashes

```sql
select name, password from sys.user$;
```

#### OR
Another option is to upload a web shell to the target. However, this requires the server to run a web server, and we need to know the exact location of the root directory for the webserver.

|**OS**|**Path**|
|---|---|
|Linux|`/var/www/html`|
|Windows|`C:\inetpub\wwwroot`|
## File Upload

```shell
echo "Oracle File Upload Test" > testing.txt

./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
# TESTING
curl -X GET http://10.129.204.235/testing.txt
```

> [!info]
> ```shell
> # FROM PWNBOX
> # 1. Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer.
> ./odat.py all -s <IP>
> sqlplus tiger/
> # Result:
> E066D214D5421CCC
> ```

# IPMI
> [!info] IPMI Relevant Information 
> [Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI_Basics) (`IPMI`) is a set of standardized specifications for hardware-based host management systems used for system management and monitoring. It acts as an autonomous subsystem and works independently of the host's BIOS, CPU, firmware, and underlying operating system.
> ## Ports
> - 623 (UDP)
> ## Uses
>  IPMI can also be used for remote upgrades to systems without requiring physical access to the target host. IPMI is typically used in three ways:
>- Before the OS has booted to modify BIOS settings
>- When the host is fully powered down
>- Access to a host after a system failure
>
>When not being used for these tasks, IPMI can monitor a range of different things such as system temperature, voltage, fan status, and power supplies. It can also be used for querying inventory information, reviewing hardware logs, and alerting using SNMP. **The host system can be powered off, but the IPMI module requires a power source and a LAN connection to work correctly.**
>The IPMI protocol was first published by Intel in 1998 and is now supported by over 200 system vendors, **including Cisco, Dell, HP, Supermicro, Intel, and more**. Systems using **IPMI version 2.0** can be administered via serial over LAN, giving sysadmins the ability to view serial console output in band. To function, IPMI requires the following components:
>- Baseboard Management Controller (BMC) - A micro-controller and essential component of an IPMI
>- Intelligent Chassis Management Bus (ICMB) - An interface that permits communication from one chassis to another
>- Intelligent Platform Management Bus (IPMB) - extends the BMC
>- IPMI Memory - stores things such as the system event log, repository store data, and more
>- Communications Interfaces - local system interfaces, serial and LAN interfaces, ICMB and PCI Management Bus
## Footprinting the service

```shell
# nmap
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

# metasploit
use auxiliary/scanner/ipmi/ipmi_version 
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include:

|Product|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|
## Dangerous Settings

During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place.
 These password hashes can then be cracked offline using a dictionary attack using `Hashcat` mode `7300`
 
 ```shell
 hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
 ```

Or using **Metasploit**:

```shell
use auxiliary/scanner/ipmi/ipmi_dumphashes 
```

>[!Important] 
>Experimenting with different word lists is crucial for obtaining the password from the acquired hash.

> [!info]- IPMI Lab Solution
> ```shell
> # 1. What username is configured for accessing the host via IPMI?
> # Metasploit 
> use auxiliary/scanner/ipmi/ipmi_dumphashes 
> ```
> ```shell
> # 2. What is the account's cleartext password?
>hashcat -m 7300 -a 0 hash_ipmi.txt /usr/share/wordlists/rockyou.txt  
>Answer: trinity
> ```

# SSH
[Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell) (`SSH`) enables two computers to establish an encrypted and direct connection within a possibly insecure network on the standard port `TCP 22`.
`SSH-2`, also known as SSH version 2, is a more advanced protocol than SSH version 1 in encryption, speed, stability, and security. For example, `SSH-1` is vulnerable to `MITM` attacks, whereas SSH-2 is not.

Authentication Methods:
1. Password authentication
2. Public-key authentication
3. Host-based authentication
4. Keyboard authentication
5. Challenge-response authentication
6. GSSAPI authentication
## Default Configuration

```shell
cat /etc/ssh/sshd_config  | grep -v "#" | sed -r '/^\s*$/d'
```
## Dangerous Settings
|**Setting**|**Description**|
|---|---|
|`PasswordAuthentication yes`|Allows password-based authentication.|
|`PermitEmptyPasswords yes`|Allows the use of empty passwords.|
|`PermitRootLogin yes`|Allows to log in as the root user.|
|`Protocol 1`|Uses an outdated version of encryption.|
|`X11Forwarding yes`|Allows X11 forwarding for GUI applications.|
|`AllowTcpForwarding yes`|Allows forwarding of TCP ports.|
|`PermitTunnel`|Allows tunneling.|
|`DebianBanner yes`|Displays a specific banner when logging in.|
## Footprinting the service
### SSH AUDIT
```shell-
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132
```
# Rsync

[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files. It can be used to copy files locally on a given machine and to/from remote hosts. It is highly versatile and well-known for its delta-transfer algorithm. This algorithm reduces the amount of data transmitted over the network when a version of the file already exists on the destination host. It does this by sending only the differences between the source files and the older version of the files that reside on the destination server. It is often used for backups and mirroring. It finds files that need to be transferred by looking at files that have changed in size or the last modified time.
## Port
- 873
## Footprinting
```shell
sudo nmap -sV -p 873 127.0.0.1
```
### Probing for accessible shares
```shell
nc -nv 127.0.0.1 873
```
### Enumerating an open share
```shell
rsync -av --list-only rsync://127.0.0.1/dev
```

# Windows Remote Management Protocols
## RDP
The [Remote Desktop Protocol](https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) (`RDP`) is a protocol developed by Microsoft for remote access to a computer running the Windows operating system. This protocol allows display and control commands to be transmitted via the GUI encrypted over IP networks. RDP works at the application layer in the TCP/IP reference model, typically utilizing TCP port 3389 as the transport protocol. However, the connectionless UDP protocol can use port 3389 also for remote administration.

RDP has handled [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (`TLS/SSL`) since Windows Vista, which means that all data, and especially the login process, is protected in the network by its good encryption. However, many Windows systems do not insist on this but still accept inadequate encryption via [RDP Security](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpbcgr/8e8b2cca-c1fa-456c-8ecb-a82fc60b2322).
## Footprinting the Service
### NMAP
```shell
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*
```

A Perl script named [rdp-sec-check.pl](https://github.com/CiscoCXSecurity/rdp-sec-check) has also been developed by [Cisco CX Security Labs](https://github.com/CiscoCXSecurity) that can unauthentically identify the security settings of RDP servers based on the handshakes.
#### RDP Security Check - Installation
```shell
sudo cpan
```
#### RDP Security Check
```shell
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl 10.129.201.248
```
#### Initiate an RDP Session
```shell
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248
```

## WinRM

The Windows Remote Management (`WinRM`) is a simple Windows integrated remote management protocol based on the command line. WinRM uses the Simple Object Access Protocol (`SOAP`) to establish connections to remote hosts and their applications. Therefore, WinRM must be explicitly enabled and configured starting with Windows 10. WinRM relies on `TCP` ports `5985` and `5986` for communication, with the last port `5986 using HTTPS`, as ports 80 and 443 were previously used for this task. However, since port 80 was mainly blocked for security reasons, the newer ports 5985 and 5986 are used today.
## Footprinting the Service

As we already know, WinRM uses TCP ports `5985` (`HTTP`) and `5986` (`HTTPS`) by default, which we can scan using Nmap. However, often we will see that only HTTP (`TCP 5985`) is used instead of HTTPS (`TCP 5986`).
### NMAP
```shell
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
```
If we want to find out whether one or more remote servers can be reached via WinRM, we can easily do this with the help of PowerShell. The [Test-WsMan](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/test-wsman?view=powershell-7.2) cmdlet is responsible for this, and the host's name in question is passed to it. In Linux-based environments, we can use the tool called [evil-winrm](https://github.com/Hackplayers/evil-winrm), another penetration testing tool designed to interact with WinRM.
```shell
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```
## WMI
`TCP` port `135`
Windows Management Instrumentation (`WMI`) is Microsoft's implementation and also an extension of the Common Information Model (`CIM`), core functionality of the standardized Web-Based Enterprise Management (`WBEM`) for the Windows platform. WMI allows read and write access to almost all settings on Windows systems.
### Footprinting the service
```shell
/usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
```
# LAB

> [!info] Labs Solution 
> ![[Ciberseguridad/HTB Apuntes/LABS/Footprinting|Footprinting]]

