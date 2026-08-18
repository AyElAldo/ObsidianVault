# HighLights

- [[#NMAP OPTIONS]]
- [[#UDP port scan]]
- [[#Default Scripts]]
- [[#Firewall and IDS/IPS Evasion]]

## Scan Techniques

- If our target sends a `SYN-ACK` flagged packet back to us, Nmap detects that the port is `open`.
- If the target responds with an `RST` flagged packet, it is an indicator that the port is `closed`.
- If Nmap does not receive a packet back, it will display it as `filtered`. Depending on the firewall configuration, certain packets may be dropped or ignored by the firewall.

## Host Discovery

- Para 192.168.3.2/24 --> ID red: 192.168.3.0/24
- Para 10.10.15.32/23 --> ID red: 10.10.14.0/23
### Scan Network Range



# NMAP OPTIONS

- `-sn`: No port scan
- `-oA <nombre_archivos>`: Stores the results in all formats starting 
	Formats:
	- **.nmap**
	- **.XML**
	- **.greapable**

- `-iL <lista_de_IPs>`: Performs defined scans against targets in provided list.
- `-PE`: Performs the ping scan by using 'ICMP Echo requests' against the target.
- `--packet-trace`: Shows all packets sent and received
- `--reason`: Displays the reason for specific result.
- `--disable-arp-ping`: Disables arp ping LOL
- `--top-ports=10`: Scans the specified top ports that have been defined as most frequent.
- `--max-retries`
- `--stats-every=5s`: Shows the progress of the scan every 5 seconds
- `-v/-vv/-vvv`: verbose


## SCAN Multiple IPs

```shell
# By writing multiple IPs
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5

# By range
sudo nmap -sn -oA tnet 10.129.2.18-20| grep for | cut -d" " -f5
```
# Port states

| **State**          | **Description**                                                                                                                                                                                         |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `open`             | This indicates that the connection to the scanned port has been established. These connections can be **TCP connections**, **UDP datagrams** as well as **SCTP associations**.                          |
| `closed`           | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an `RST` flag. This scanning method can also be used to determine if our target is alive or not. |
| `filtered`         | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.                  |
| `unfiltered`       | This state of a port only occurs during the **TCP-ACK** scan and means that the port is accessible, but it cannot be determined whether it is open or closed.                                           |
| `open\|filtered`   | If we do not get a response for a specific port, `Nmap` will set it to that state. This indicates that a firewall or packet filter may protect the port.                                                |
| `closed\|filtered` | This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.                                           |


- **(root)**`-sS`: It doesn't stablish the conection. It returns an output if received an `SYN-ACK`. (TCP)
- `-sY`: Makes a complete conection. 
- `-Pn`: Disables the ICMP echo requests
- `-n`: Disables DNS resolution
- `-sV`: Versions, service names and more details
- `-F`: Scan the top 100 ports

## UDP port scan

- `-sU`: UDP scan
- `-F`: Top 100 ports  

```shell
# Example
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason

PORT    STATE         SERVICE     REASON
138/udp open|filtered netbios-dgm no-response
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
```

# Saving the results

- `-oN (.nmap)`:  Normal
- `-oG (.gnmap)`: Grepable
- `-oX`: XML --> We can use the tool `xsltproc`

- `-oA`: Save the results in all formats

To convert the stored XML format to HTML, we use: 

```shell
xsltproc target.xml -o target.html
```

## **Respuestas**

Find all TCP ports on your target. Submit the total number of found TCP ports as the answer.
```shell
sudo nmap -p- 10.129.2.49 --reason -n -Pn -T4 --min-rate 5000 -sV
```

# Nmap Scripting Engine (NSE)

```shell
sudo nmap --script-updatedb # Updates the NSE database
```

|**Category**|**Description**|
|---|---|
|`auth`|Determination of authentication credentials.|
|`broadcast`|Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans.|
|`brute`|Executes scripts that try to log in to the respective service by brute-forcing with credentials.|
|`default`|Default scripts executed by using the `-sC` option.|
|`discovery`|Evaluation of accessible services.|
|`dos`|These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.|
|`exploit`|This category of scripts tries to exploit known vulnerabilities for the scanned port.|
|`external`|Scripts that use external services for further processing.|
|`fuzzer`|This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.|
|`intrusive`|Intrusive scripts that could negatively affect the target system.|
|`malware`|Checks if some malware infects the target system.|
|`safe`|Defensive scripts that do not perform intrusive and destructive access.|
|`version`|Extension for service detection.|
|`vuln`|Identification of specific vulnerabilities.|
## Default Scripts

- `-sC`: Default Scripts
## Specific Scripts Category

```bash
sudo nmap <target> --script <category>
```
## Defined Scripts

```bash
sudo nmap <target> --script <script-name>,<script-name>,...
```

## Agressive Option -A

This scans the target with multiple options as service detection (`-sV`), OS detection (`-O`), traceroute (`--traceroute`), and with the default NSE scripts (`-sC`).

```shell
# Example
sudo nmap 10.129.2.28 -p 80 -A

# Output

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: blog.inlanefreight.com
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), 
AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Netgear RAIDiator 4.2.28 (94%), 
Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

TRACEROUTE
HOP RTT      ADDRESS
1   11.91 ms 10.129.2.28
```
# Common Scripts

- `banner`: 
- `smpt-commands`: Shows us which commands we can use by interacting with the target SMTP server.
- `vuln`: Uses all related scripts from specified category.
-  `ftp*`: FTP scripts

> [!info]- FTP
> ![[Ciberseguridad/HTB Apuntes/Footprinting#FTP Scripts]]

# Performance

- `-T<0-5>`: How fast
- `--min-parallelism <number>`: Frequency
- `--max-rtt-timeout <time>`: Timeouts
- `--min-rate <number>`: Packets sent simultaneously
- `--max-retries <number>`: Number of retries

# Firewall and IDS/IPS Evasion

When firewalls reject package, they return:

- TCP -> RST
- ICMP -> Errors like:
	- Net Unreachable
	- Net Prohibited
	- Host Unreachable
	- Host Prohibited
	- Port Unreachable
	- Proto Unreachable

- `-sA`: Performs ACK scan on specified ports.

## Decoys

- `-D RND:5`: Generates five random IP addresses that indicates the source IP the connection comes from.
- `-S`: Sans the target by using differente source IP address.
### Change the IP source

```shell
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
```

|**Scanning Options**|**Description**|
|---|---|
|`10.129.2.28`|Scans the specified target.|
|`-n`|Disables DNS resolution.|
|`-Pn`|Disables ICMP Echo requests.|
|`-p 445`|Scans only the specified ports.|
|`-O`|Performs operation system detection scan.|
|`-S`|Scans the target by using different source IP address.|
|`10.129.2.200`|Specifies the source IP address.|
|`-e tun0`|Sends all requests through the specified interface.|
> [!info] Importante
> **Investigar Idle Scan**: Uso de máquinas zombies para ataques mas sofisticados. 

## DNS Proxying

We can confuse some tagets when changing out source port to DNS (port 53). Then, if the target is not set for this kind of evasion, the target will not reject our package.

```shell
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

# RESPONSE
SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).

PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

# Easy Lab

![[Enumeration with NMAP]]

# Medium Lab

1![[(Medium Lab) Firewall and IDS-IPS Evasion]]
# Hard Lab

![[(HardLab) Firewall and IDS-IPS Evasion]]