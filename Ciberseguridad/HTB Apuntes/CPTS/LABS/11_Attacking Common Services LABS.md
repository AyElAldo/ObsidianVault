# Attacking Common Services - Easy

We were commissioned by the company Inlanefreight to conduct a penetration test against three different hosts to check the servers' configuration and security. We were informed that a flag had been placed somewhere on each server to prove successful access. These flags have the following format:

### You are targeting the inlanefreight.htb domain. Assess the target server and obtain the contents of the flag.txt file. Submit it as the answer.

```shell
sudo nmap -Pn -sV -sC -v 10.129.203.7 -T4 -oA easy
--
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Issuer: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: shaWithRSAEncryption
| Not valid before: 2022-04-21T19:27:17
| Not valid after:  2032-04-18T19:27:17
| MD5:   27ed:2da8:8b25:57e3:d2fc:c0c8:9f0b:55b0
|_SHA-1: 5018:d8d5:ba6b:5a1c:8df6:5969:45d7:fe06:3d32:7fad
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...
|   Help: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     214-The following commands are implemented
|     USER PASS ACCT QUIT PORT RETR
|     STOR DELE RNFR PWD CWD CDUP
|     NOOP TYPE MODE STRU
|     LIST NLST HELP FEAT UTF8 PASV
|     MDTM REST PBSZ PROT OPTS CCC
|     XCRC SIZE MFMT CLNT ABORT
|     HELP command successful
|   NULL, SMBProgNeg: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|   SSLSessionReq: 
|_    550 Too many connections, please try later...
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp   open  http          Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
|_http-server-header: Apache/2.4.53 (Win64) OpenSSL/1.1.1n PHP/7.4.29
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
| http-title: Welcome to XAMPP
|_Requested resource was http://10.129.203.7/dashboard/
443/tcp  open  https?
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Issuer: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: shaWithRSAEncryption
| Not valid before: 2022-04-21T19:27:17
| Not valid after:  2032-04-18T19:27:17
| MD5:   27ed:2da8:8b25:57e3:d2fc:c0c8:9f0b:55b0
|_SHA-1: 5018:d8d5:ba6b:5a1c:8df6:5969:45d7:fe06:3d32:7fad
| http-methods: 
|_  Supported Methods: GET
|_ssl-date: 2026-11-12T22:24:12+00:00; -1s from scanner time.
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
3306/tcp open  mysql         MariaDB 5.5.5-10.4.24
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.24-MariaDB
|   Thread ID: 10
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, DontAllowDatabaseTableColumn, SupportsCompression, ConnectWithDatabase, FoundRows, ODBCClient, Speaks41ProtocolOld, SupportsTransactions, Speaks41ProtocolNew, IgnoreSigpipes, LongColumnFlag, InteractiveClient, SupportsLoadDataLocal, IgnoreSpaceBeforeParenthesis, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: Y\@m%mw1)zNUA%3q`k{I
|_  Auth Plugin Name: mysql_native_password
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WIN-EASY
|   NetBIOS_Domain_Name: WIN-EASY
|   NetBIOS_Computer_Name: WIN-EASY
|   DNS_Domain_Name: WIN-EASY
|   DNS_Computer_Name: WIN-EASY
|   Product_Version: 10.0.17763
|_  System_Time: 2026-11-12T22:23:34+00:00
| ssl-cert: Subject: commonName=WIN-EASY
| Issuer: commonName=WIN-EASY
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-11-11T22:13:29
| Not valid after:  2027-05-13T22:13:29
| MD5:   9046:0bc1:5fd4:9bae:b046:79e9:3d0f:7be4
|_SHA-1: 42f8:a8b2:a30f:3b95:a6c0:f550:f26e:9e6f:b262:43f9
|_ssl-date: 2026-11-12T22:24:11+00:00; -2s from scanner time.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.95%I=7%D=11/12%Time=6AF63C0E%P=x86_64-pc-linux-gnu%r(NUL
SF:L,41,"220\x20Core\x20FTP\x20Server\x20Version\x202\.0,\x20build\x20725,
SF:\x2064-bit\x20Unregistered\r\n")%r(GenericLines,AD,"220\x20Core\x20FTP\
SF:x20Server\x20Version\x202\.0,\x20build\x20725,\x2064-bit\x20Unregistere
SF:d\r\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x20not\x20all
SF:owed\.\.\.\r\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x20n
SF:ot\x20allowed\.\.\.\r\n")%r(Help,17B,"220\x20Core\x20FTP\x20Server\x20V
SF:ersion\x202\.0,\x20build\x20725,\x2064-bit\x20Unregistered\r\n214-The\x
SF:20following\x20commands\x20are\x20implemented\r\n\x20\x20\x20\x20\x20US
SF:ER\x20\x20PASS\x20\x20ACCT\x20\x20QUIT\x20\x20PORT\x20\x20RETR\r\n\x20\
SF:x20\x20\x20\x20STOR\x20\x20DELE\x20\x20RNFR\x20\x20PWD\x20\x20\x20CWD\x
SF:20\x20\x20CDUP\r\n\x20\x20\x20\x20\x20MKD\x20\x20\x20RMD\x20\x20\x20NOO
SF:P\x20\x20TYPE\x20\x20MODE\x20\x20STRU\r\n\x20\x20\x20\x20\x20LIST\x20\x
SF:20NLST\x20\x20HELP\x20\x20FEAT\x20\x20UTF8\x20\x20PASV\r\n\x20\x20\x20\
SF:x20\x20MDTM\x20\x20REST\x20\x20PBSZ\x20\x20PROT\x20\x20OPTS\x20\x20CCC\
SF:r\n\x20\x20\x20\x20\x20XCRC\x20\x20SIZE\x20\x20MFMT\x20\x20CLNT\x20\x20
SF:ABORT\r\n214\x20\x20HELP\x20command\x20successful\r\n")%r(SSLSessionReq
SF:,2F,"550\x20Too\x20many\x20connections,\x20please\x20try\x20later\.\.\.
SF:\r\n")%r(SMBProgNeg,41,"220\x20Core\x20FTP\x20Server\x20Version\x202\.0
SF:,\x20build\x20725,\x2064-bit\x20Unregistered\r\n");
Service Info: Host: WIN-EASY; OS: Windows; CPE: cpe:/o:microsoft:windows
```

And then enumerate the SMTP service

```shell
smtp-user-enum -t 10.129.203.7 -U users.list -M RCPT -D inlanefreight.htb
---
10.129.203.7: fiona@inlanefreight.htb exists
```