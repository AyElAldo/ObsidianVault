# Interacting with Common Services

Vulnerabilities are commonly discovered by people who use and understand technology, a protocol, or a service. As we evolve in this field, we will find different services to interact with, and we will need to evolve and learn new technology constantly.

To be successful at attacking a service, we need to know its purpose, how to interact with it, what tools we can use, and what we can do with it. This section will focus on common services and how we can interact with them.
## File Share Services

A file sharing service is a type of service that provides, mediates, and monitors the transfer of computer files. Years ago, businesses commonly used only internal services for file sharing, such as SMB, NFS, FTP, TFTP, SFTP, but as cloud adoption grows, most companies now also have third-party cloud services such as Dropbox, Google Drive, OneDrive, SharePoint, or other forms of file storage such as AWS S3, Azure Blob Storage, or Google Cloud Storage. We will be exposed to a mixture of internal and external file-sharing services, and we need to be familiar with them.

This section will focus on internal services, but this may apply to cloud storage synced locally to servers and workstations.
## Server Message Block (SMB)

SMB is commonly used in Windows networks, and we will often find share folders in a Windows network. We can interact with SMB using the GUI, CLI, or tools. Let us cover some common ways of interacting with SMB using Windows & Linux.
#### Windows

There are different ways we can interact with a shared folder using Windows, and we will explore a couple of them. On Windows GUI, we can press `[WINKEY] + [R]` to open the Run dialog box and type the file share location, e.g.: `\\192.168.220.129\Finance\`

![Windows Server 2012 R2 desktop with Run dialog open, showing network path entry.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/116/windows_run_sharefolder2.jpg)

Suppose the shared folder allows anonymous authentication, or we are authenticated with a user who has privilege over that shared folder. In that case, we will not receive any form of authentication request, and it will display the content of the shared folder.

![File explorer open to network path \192.168.220.133\Finance showing Contracts folder.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/116/finance_share_folder2.jpg)

If we do not have access, we will receive an authentication request.

![Windows Security prompt for network credentials with fields for username, password, and domain.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/116/auth_request_share_folder2.jpg)

Windows has two command-line shells: the [Command shell](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands) and [PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/overview). Each shell is a software program that provides direct communication between us and the operating system or application, providing an environment to automate IT operations.

Let's discuss some commands to interact with file share using Command Shell (`CMD`) and `PowerShell`. The command [dir](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dir) displays a list of a directory's files and subdirectories.
#### Windows CMD - DIR

```powershell
dir \\192.168.220.129\Finance\
```
The command [net use](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/gg651155\(v=ws.11\)) connects a computer to or disconnects a computer from a shared resource or displays information about computer connections. We can connect to a file share with the following command and map its content to the drive letter `n`.

#### Windows CMD - Net Use

```powershell
net use n: \\192.168.220.129\Finance
```

We can also provide a username and password to authenticate to the share.

```powershell
net use n: \\192.168.220.129\Finance /user:plaintext Password123
```

With the shared folder mapped as the `n` drive, we can execute Windows commands as if this shared folder is on our local computer. Let's find how many files the shared folder and its subdirectories contain.
#### Windows CMD - DIR

We found 29,302 files. Let's walk through the command:

```powershell
dir n: /a-d /s /b | find /c ":\"
```

|**Syntax**|**Description**|
|---|---|
|`dir`|Application|
|`n:`|Directory or drive to search|
|`/a-d`|`/a` is the attribute and `-d` means not directories|
|`/s`|Displays files in a specified directory and all subdirectories|
|`/b`|Uses bare format (no heading information or summary)|
The following command `| find /c ":\\"` process the output of `dir n: /a-d /s /b` to count how many files exist in the directory and subdirectories. You can use `dir /?` to see the full help. Searching through 29,302 files is time consuming, scripting and command line utilities can help us speed up the search. With `dir` we can search for specific names in files such as:

- cred
- password
- users
- secrets
- key
- Common File Extensions for source code such as: .cs, .c, .go, .java, .php, .asp, .aspx, .html.

```powershell
dir n:\*cred* /s /b
dir n:\*secret* /s /b
```
#### Windows CMD - Findstr

```powershell
findstr /s /i cred n:\*.*
```
#### Windows PowerShell

PowerShell was designed to extend the capabilities of the Command shell to run PowerShell commands called `cmdlets`. Cmdlets are similar to Windows commands but provide a more extensible scripting language. We can run both Windows commands and PowerShell cmdlets in PowerShell, but the Command shell can only run Windows commands and not PowerShell cmdlets. Let's replicate the same commands now using Powershell.
#### Windows PowerShell

```powershell
Get-ChildItem \\192.168.220.129\Finance\
```

Instead of `net use`, we can use `New-PSDrive` in PowerShell.

```powershell
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

To provide a username and password with Powershell, we need to create a [PSCredential object](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential). It offers a centralized way to manage usernames, passwords, and credentials.
#### Windows PowerShell - PSCredential Object

```powershell
$username = 'plaintext'
$password = 'Password123'
$secpassword = ConvertTo-SecureString $password -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
```

In PowerShell, we can use the command `Get-ChildItem` or the short variant `gci` instead of the command `dir`.
#### Windows PowerShell - GCI

```powershell
N: 

PS N:\> (Get-ChildItem -File -Recurse | Measure-Object).Count
```

We can use the property `-Include` to find specific items from the directory specified by the Path parameter.

```powershell
Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
```

The `Select-String` cmdlet uses regular expression matching to search for text patterns in input strings and files. We can use `Select-String` similar to `grep` in UNIX or `findstr.exe` in Windows.

```powershell
Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

CLI enables IT operations to automate routine tasks like user account management, nightly backups, or interaction with many files. We can perform operations more efficiently by using scripts than the user interface or GUI.
#### Linux

Linux (UNIX) machines can also be used to browse and mount SMB shares. Note that this can be done whether the target server is a Windows machine or a Samba server. Even though some Linux distributions support a GUI, we will focus on Linux command-line utilities and tools to interact with SMB. Let's cover how to mount SMB shares to interact with directories and files locally.
#### Linux - Mount

```shell
sudo mkdir /mnt/Finance
sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

As an alternative, we can use a credential file.

```shell
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

The file `credentialfile` has to be structured like this:

As an alternative, we can use a credential file:

```shell
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile # The file contains credencials
```

>[!Note]
>We need to install `cifs-utils` to connect to an SMB share folder. To install it we can execute from the command line `sudo apt install cifs-utils`.
#### Linux - Find

```shell
find /mnt/Finance/ -name *cred*
```

Next, let's find files that contain the string `cred`:

```shell
grep -rn /mnt/Finance/ -ie cred
```
#### Linux - Install Evolution

```shell
sudo apt-get install evolution
```

>[!Note]
If an error appears when starting evolution indicating "bwrap: Can't create file at ...", use this command to start evolution `export WEBKIT_FORCE_SANDBOX=0 && evolution`.
# The Concept of Attacks
## Source

We can generalize `Source` as a source of information used for the specific task of a process. There are many different ways to pass information to a process. The graphic shows some of the most common examples of how information is passed to the processes.

| **Information Source** | **Description**                                                                                                                                                                                    |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Code`                 | This means that the already executed program code results are used as a source of information. These can come from different functions of a program.                                               |
| `Libraries`            | A library is a collection of program resources, including configuration data, documentation, help data, message templates, prebuilt code and subroutines, classes, values, or type specifications. |
| `Config`               | Configurations are usually static or prescribed values that determine how the process processes information.                                                                                       |
| `APIs`                 | The application programming interface (API) is mainly used as the interface of programs for retrieving or providing information.                                                                   |
| `User Input`           | If a program has a function that allows the user to enter specific values used to process the information accordingly, this is the manual entry of information by a person.                        |
## Processes

The `Process` is about processing the information forwarded from the source. These are processed according to the intended task determined by the program code. For each task, the developer specifies how the information is processed. This can occur using classes with different functions, calculations, and loops. The variety of possibilities for this is as diverse as the number of developers in the world. Accordingly, most of the vulnerabilities lie in the program code executed by the process.

| **Process Components** | **Description**                                                                                                                                                            |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `PID`                  | The Process-ID (PID) identifies the process being started or is already running. Running processes have already assigned privileges, and new ones are started accordingly. |
| `Input`                | This refers to the input of information that could be assigned by a user or as a result of a programmed function.                                                          |
| `Data processing`      | The hard-coded functions of a program dictate how the information received is processed.                                                                                   |
| `Variables`            | The variables are used as placeholders for information that different functions can further process during the task.                                                       |
| `Logging`              | During logging, certain events are documented and, in most cases, stored in a register or a file. This means that certain information remains in the system.               |
## Privileges

`Privileges` are present in any system that controls processes. These serve as a type of permission that determines what tasks and actions can be performed on the system. In simple terms, it can be compared to a bus ticket. If we use a ticket intended for a particular region, we will be able to use the bus, and otherwise, we will not. These privileges (or figuratively speaking, our tickets) can also be used for different means of transport, such as planes, trains, boats, and others. In computer systems, these privileges serve as control and segmentation of actions for which different permissions, controlled by the system, are needed. Therefore, the rights are checked based on this categorization when a process needs to fulfill its task. If the process satisfies these privileges and conditions, the system approves the action requested. We can divide these privileges into the following areas:

| **Privileges** | **Description**                                                                                                                                                                                           |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `System`       | These privileges are the highest privileges that can be obtained, which allow any system modification. In Windows, this type of privilege is called `SYSTEM`, and in Linux, it is called `root`.          |
| `User`         | User privileges are permissions that have been assigned to a specific user. For security reasons, separate users are often set up for particular services during the installation of Linux distributions. |
| `Groups`       | Groups are a categorization of at least one user who has certain permissions to perform specific actions.                                                                                                 |
| `Policies`     | Policies determine the execution of application-specific commands, which can also apply to individual or grouped users and their actions.                                                                 |
| `Rules`        | Rules are the permissions to perform actions handled from within the applications themselves.                                                                                                             |
## Destination

Every task has at least one purpose and goal that must be fulfilled. Logically, if any data set changes were missing or not stored or forwarded anywhere, the task would be generally unnecessary. The result of such a task is either stored somewhere or forwarded to another processing point. Therefore we speak here of the `Destination` where the changes will be made. Such processing points can point either to a local or remote process. Therefore, at the local level, local files or records may be modified by the process or be forwarded to other local services for further use. However, this does not exclude the possibility that the same process could reuse the resulting data too. If the process is completed with the data storage or its forwarding, the cycle leading to the task's completion is closed.

|**Destination**|**Description**|
|---|---|
|`Local`|The local area is the system's environment in which the process occurred. Therefore, the results and outcomes of a task are either processed further by a process that includes changes to data sets or storage of the data.|
|`Network`|The network area is mainly a matter of forwarding the results of a process to a remote interface. This can be an IP address and its services or even entire networks. The results of such processes can also influence the route under certain circumstances.|
This graphic breaks down the Log4j JNDI attack based on the `Concept of Attacks`.
#### Initiation of the Attack

| **Step** | **Log4j**                                                                                                                                                               | **Concept of Attacks - Category** |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `1.`     | The attacker manipulates the user agent with a JNDI lookup command.                                                                                                     | `Source`                          |
| `2.`     | The process misinterprets the assigned user agent, leading to the execution of the command.                                                                             | `Process`                         |
| `3.`     | The JNDI lookup command is executed with administrator privileges due to logging permissions.                                                                           | `Privileges`                      |
| `4.`     | This JNDI lookup command points to the server created and prepared by the attacker, which contains a malicious Java class containing commands designed by the attacker. | `Destination`                     |
#### Trigger Remote Code Execution

|**Step**|**Log4j**|**Concept of Attacks - Category**|
|---|---|---|
|`5.`|After the malicious Java class is retrieved from the attacker's server, it is used as a source for further actions in the following process.|`Source`|
|`6.`|Next, the malicious code of the Java class is read in, which in many cases has led to remote access to the system.|`Process`|
|`7.`|The malicious code is executed with administrator privileges due to logging permissions.|`Privileges`|
|`8.`|The code leads back over the network to the attacker with the functions that allow the attacker to control the system remotely.|`Destination`|
# Service Misconfigurations

Misconfigurations usually happen when a system administrator, technical support, or developer does not correctly configure the security framework of an application, website, desktop, or server leading to dangerous open pathways for unauthorized users. Let's explore some of the most typical misconfigurations of common services.
## Authentication

Nowadays, most software asks users to set up credentials upon installation, which is better than default credentials. However, keep in mind that we will still find vendors using default credentials, especially on older applications.

Even when the service does not have a set of default credentials, an administrator may use weak passwords or no passwords when setting up services with the idea that they will change the password once the service is set up and running.

As administrators, we need to define password policies that apply to software tested or installed in our environment. Administrators should be required to comply with a minimum password complexity to avoid user and passwords combinations such as:
#### Anonymous Authentication

Another misconfiguration that can exist in common services is anonymous authentication. The service can be configured to allow anonymous authentication, allowing anyone with network connectivity to the service without being prompted for authentication.
# Attacking FTP

The [File Transfer Protocol](https://en.wikipedia.org/wiki/File_Transfer_Protocol) (`FTP`) is a standard network protocol used to transfer files between computers. It also performs directory and files operations, such as changing the working directory, listing files, and renaming and deleting directories or files. By default, FTP listens on port `TCP/21`.
## Enumeration

`Nmap` default scripts `-sC` includes the [ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html) Nmap script which checks if a FTP server allows anonymous logins. The version enumeration flag `-sV` provides interesting information about FTP services, such as the FTP banner, which often includes the version name. We can use the `ftp` client or `nc` to interact with the FTP service. By default, FTP runs on TCP port 21.

```shell
sudo nmap -sC -sV -p 21 192.168.2.142
```
## Misconfigurations

As we discussed, anonymous authentication can be configured for different services such as FTP. To access with anonymous login, we can use the `anonymous` username and no password. This will be dangerous for the company if read and write permissions have not been set up correctly for the FTP service. Because with the anonymous login, the company could have stored sensitive information in a folder that the anonymous user of the FTP service could have access to.