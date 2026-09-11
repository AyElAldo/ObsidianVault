The `Metasploit Project` is a Ruby-based, modular penetration testing platform that enables you to write, test, and execute the exploit code. This exploit code can be custom-made by the user or taken from a database containing the latest already discovered and modularized exploits. The `Metasploit Framework` includes a suite of tools that you can use to test security vulnerabilities, enumerate networks, execute attacks, and evade detection. At its core, the `Metasploit Project` is a collection of commonly used tools that provide a complete environment for penetration testing and exploit development.

![Diagram of Metasploit Project showing two branches: Metasploit Framework, labeled as Open Source, Community Driven, Free; and Metasploit Pro, labeled as Commercial Use, Paid Subscription, Enterprise Oriented.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/39/S02_SS01.png)

The `modules` mentioned are actual exploit proof-of-concepts that have already been developed and tested in the wild and integrated within the framework to provide pentesters with ease of access to different attack vectors for different platforms and services.
## Metasploit Pro

`Metasploit` as a product is split into two versions. The `Metasploit Pro` version is different from the `Metasploit Framework` one with some additional features:

- Task Chains
- Social Engineering
- Vulnerability Validations
- GUI
- Quick Start Wizards
- Nexpose Integration

If you're more of a command-line user and prefer the extra features, the Pro version also contains its own console, much like `msfconsole`.
## Metasploit Framework Console

The `msfconsole` is probably the most popular interface to the `Metasploit Framework` `(MSF)`. It provides an "all-in-one" centralized console and allows you efficient access to virtually all options available in the `MSF`. `Msfconsole` may seem intimidating at first, but once you learn the syntax of the commands, you will learn to appreciate the power of utilizing this interface.

The features that `msfconsole` generally brings are the following:

- It is the only supported way to access most of the features within `Metasploit`
- Provides a console-based interface to the `Framework`
- Contains the most features and is the most stable `MSF` interface
- Full readline support, tabbing, and command completion
- Execution of external commands in `msfconsole`
## Understanding the Architecture
By default, all the base files related to Metasploit Framework can be found under `/usr/share/metasploit-framework` in our `ParrotOS Security` distro.
#### Data, Documentation, Lib

These are the base files for the Framework. The Data and Lib are the functioning parts of the msfconsole interface, while the Documentation folder contains all the technical details about the project.
#### Modules

The Modules detailed above are split into separate categories in this folder. We will go into detail about these in the next sections. They are contained in the following folders:
- auxiliary 
- encoders 
- evasion 
- exploits 
- nops 
- payloads 
- post
#### Plugins

Plugins offer the pentester more flexibility when using the `msfconsole` since they can easily be manually or automatically loaded as needed to provide extra functionality and automation during our assessment.
#### Scripts

Meterpreter functionality and other useful scripts.
#### Tools

Command-line utilities that can be called directly from the `msfconsole` menu.
# Introduction to MSFconsole

To start interacting with the Metasploit Framework, we need to type `msfconsole` in the terminal of our choice. Many security-oriented distributions such as Parrot Security and Kali Linux come with `msfconsole` preinstalled.
## MSF Engagement Structure

The MSF engagement structure can be divided into five main categories.

- Enumeration
- Preparation
- Exploitation
- Privilege Escalation
- Post-Exploitation

This division makes it easier for us to find and select the appropriate MSF features in a more structured way and to work with them accordingly. Each of these categories has different subcategories that are intended for specific purposes. These include, for example, Service Validation and Vulnerability Research.

It is therefore crucial that we familiarize ourselves with this structure. Therefore, we will look at this framework's components to better understand how they are related.

![Flowchart of Engagement Structure detailing steps: Enumeration with Service Validation and Vulnerability Research; Preparation with Code Auditing; Exploitation with Module Execution; Privilege Escalation; Post-Exploitation with Pivoting and Data Exfiltration.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/39/S04_SS03.png)
# Modules

As we mentioned previously, Metasploit `modules` are prepared scripts with a specific purpose and corresponding functions that have already been developed and tested in the wild. The `exploit` category consists of so-called proof-of-concept (`POCs`) that can be used to exploit existing vulnerabilities in a largely automated manner. Many people often think that the failure of the exploit disproves the existence of the suspected vulnerability. However, this is only proof that the Metasploit exploit does not work and not that the vulnerability does not exist.

Once we are in the `msfconsole`, we can select from an extensive list containing all the available Metasploit modules. Each of them is structured into folders, which will look like this:
#### Syntax

```shell
<No.> <type>/<os>/<service>/<name>
```
#### Type

The `Type` tag is the first level of segregation between the Metasploit `modules`. Looking at this field, we can tell what the piece of code for this module will accomplish. Some of these `types` are not directly usable as an `exploit` module would be, for example. However, they are set to introduce the structure alongside the interactable ones for better modularization. To explain better, here are the possible types that could appear in this field:

|**Type**|**Description**|
|---|---|
|`Auxiliary`|Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality.|
|`Encoders`|Ensure that payloads are intact to their destination.|
|`Exploits`|Defined as modules that exploit a vulnerability that will allow for the payload delivery.|
|`NOPs`|(No Operation code) Keep the payload sizes consistent across exploit attempts.|
|`Payloads`|Code runs remotely and calls back to the attacker machine to establish a connection (or shell).|
|`Plugins`|Additional scripts can be integrated within an assessment with `msfconsole` and coexist.|
|`Post`|Wide array of modules to gather information, pivot deeper, etc.|
#### OS

The `OS` tag specifies which operating system and architecture the module was created for. Naturally, different operating systems require different code to be run to get the desired results.

#### Service

The `Service` tag refers to the vulnerable service that is running on the target machine. For some modules, such as the `auxiliary` or `post` ones, this tag can refer to a more general activity such as `gather`, referring to the gathering of credentials, for example.

#### Name

Finally, the `Name` tag explains the actual action that can be performed using this module created for a specific purpose.
## Searching for Modules

Metasploit also offers a well-developed search function for the existing modules. With the help of this function, we can quickly search through all the modules using specific `tags` to find a suitable one for our target.
#### MSF - Search Function

```shell
help search 
```
#### MSF - Specific Search

```shell
search type:exploit platform:windows cve:2021 rank:excellent microsoft
```
#### MSF - Module Information

```shell
msf6 exploit(windows/smb/ms17_010_psexec) > info
```
#### MSF - Target Specification

```shell
set RHOSTS 10.10.10.40
```
#### MSF - Permanent Target Specification

```shell
setg RHOSTS 10.10.10.40
```
Finally, since we are about to use a TCP-based reverse shell (`/windows/meterpreter/reverse_tcp`) we need to specify to which IP address it needs to connect to in order to establish a connection. Therefore, we need to set `LHOST` to our own IP address like following:
# Targets

`Targets` are unique operating system identifiers taken from the versions of those specific operating systems which adapt the selected exploit module to run on that particular version of the operating system. The `show targets` command issued within an exploit module view will display all available vulnerable targets for that specific exploit, while issuing the same command in the root menu, outside of any selected exploit module, will let us know that we need to select an exploit module first.
## Selecting a Target

We can see that there is only one general type of target set for this type of exploit. What if we change the exploit module to something that needs more specific target ranges? The following exploit is aimed at:

- `MS12-063 Microsoft Internet Explorer execCommand Use-After-Free Vulnerability`

If we want to find out more about this specific module and what the vulnerability behind it does, we can use the `info` command. This command can help us out whenever we are unsure about the origins or functionality of different exploits or auxiliary modules. Keeping in mind that it is always considered best practice to audit our code for any artifact generation or 'additional features', the `info` command should be one of the first steps we take when using a new module. This way, we can familiarize ourselves with the exploit functionality while assuring a safe, clean working environment for both our clients and us.
#### MSF - Target Selection

```shell
msf6 exploit(windows/browser/ie_execcommand_uaf) > info
```

## Target Types

There is a large variety of target types. Every target can vary from another by service pack, OS version, and even language version. It all depends on the return address and other parameters in the target or within the exploit module.

The return address can vary because a particular language pack changes addresses, a different software version is available, or the addresses are shifted due to hooks. It is all determined by the type of return address required to identify the target. This address can be `jmp esp`, a jump to a specific register that identifies the target, or a `pop/pop/ret`. For more on the topic of return addresses, see the [Stack-Based Buffer Overflows on Windows x86](https://academy.hackthebox.com/module/89/section/931) module. Comments in the exploit module's code can help us determine what the target is defined by.
# Payloads

A `Payload` in Metasploit refers to a module that aids the exploit module in (typically) returning a shell to the attacker. The payloads are sent together with the exploit itself to bypass standard functioning procedures of the vulnerable service (`exploits job`) and then run on the target OS to typically return a reverse connection to the attacker and establish a foothold (`payload's job`).

There are three different types of payload modules in the Metasploit Framework: Singles, Stagers, and Stages. Using three typologies of payload interaction will prove beneficial to the pentester. It can offer the flexibility we need to perform certain types of tasks. Whether or not a payload is staged is represented by `/` in the payload name.

For example, `windows/shell_bind_tcp` is a single payload with no stage, whereas `windows/shell/bind_tcp` consists of a stager (`bind_tcp`) and a stage (`shell`).
#### Singles

A `Single` payload contains the exploit and the entire shellcode for the selected task. Inline payloads are by design more stable than their counterparts because they contain everything all-in-one. However, some exploits will not support the resulting size of these payloads as they can get quite large. `Singles` are self-contained payloads. They are the sole object sent and executed on the target system, getting us a result immediately after running. A Single payload can be as simple as adding a user to the target system or booting up a process.
#### Stagers

`Stager` payloads work together with `Stage` payloads to perform a specific task. A stager runs on the victim machine and initiates an outbound connection to the attacker's listener, setting up the communication channel over which the subsequent stage payload is delivered. Stagers are typically designed to be small and reliable. Metasploit will automatically select the most appropriate stager for a given scenario and fall back to a less-preferred one when necessary.

Windows NX vs. NO-NX Stagers

- Reliability issue for NX CPUs and DEP
- NX stagers are bigger (VirtualAlloc memory)
- Default is now NX + Win7 compatible
#### Stages

`Stages` are payload components that are downloaded by stager's modules. The various payload Stages provide advanced features with no size limits, such as Meterpreter, VNC Injection, and others. Payload stages automatically use middle stagers:

- A single `recv()` fails with large payloads
- The Stager receives the middle stager
- The middle Stager then performs a full download
- Also better for RWX
## Staged Payloads

A staged payload is, simply put, an `exploitation process` that is modularized and functionally separated to help segregate the different functions it accomplishes into different code blocks, each completing its objective individually but working on chaining the attack together. This will ultimately grant an attacker remote access to the target machine if all the stages work correctly.

The scope of this payload, as with any others, besides granting shell access to the target system, is to be as compact and inconspicuous as possible to aid with the Antivirus (`AV`) / Intrusion Prevention System (`IPS`) evasion as much as possible.
#### Meterpreter Payload

The `Meterpreter` payload is a specific type of multi-faceted payload that uses `DLL injection` to ensure the connection to the victim host is stable, hard to detect by simple checks, and persistent across reboots or system changes.
## Searching for Payloads

To select our first payload, we need to know what we want to do on the target machine. For example, if we are going for access persistence, we will probably want to select a Meterpreter payload.

As mentioned above, Meterpreter payloads offer us a significant amount of flexibility. Their base functionality is already vast and influential. We can automate and quickly deliver combined with plugins such as [GentilKiwi's Mimikatz Plugin](https://github.com/gentilkiwi/mimikatz) parts of the pentest while keeping an organized, time-effective assessment. To see all of the available payloads, use the `show payloads` command in `msfconsole`.
## Payload Types

The table below contains the most common payloads used for Windows machines and their respective descriptions.

| **Payload**                       | **Description**                                                        |
| --------------------------------- | ---------------------------------------------------------------------- |
| `generic/custom`                  | Generic listener, multi-use                                            |
| `generic/shell_bind_tcp`          | Generic listener, multi-use, normal shell, TCP connection binding      |
| `generic/shell_reverse_tcp`       | Generic listener, multi-use, normal shell, reverse TCP connection      |
| `windows/x64/exec`                | Executes an arbitrary command (Windows x64)                            |
| `windows/x64/loadlibrary`         | Loads an arbitrary x64 library path                                    |
| `windows/x64/messagebox`          | Spawns a dialog via MessageBox using a customizable title, text & icon |
| `windows/x64/shell_reverse_tcp`   | Normal shell, single payload, reverse TCP connection                   |
| `windows/x64/shell/reverse_tcp`   | Normal shell, stager + stage, reverse TCP connection                   |
| `windows/x64/shell/bind_ipv6_tcp` | Normal shell, stager + stage, IPv6 Bind TCP stager                     |
| `windows/x64/meterpreter/$`       | Meterpreter payload + varieties above                                  |
| `windows/x64/powershell/$`        | Interactive PowerShell sessions + varieties above                      |
| `windows/x64/vncinject/$`         | VNC Server (Reflective Injection) + varieties above                    |
## Exercise

### Exploit the Apache Druid service and find the flag.txt file. Submit the contents of this file as the answer.

```shell
sudo nmap 10.129.203.52 -p 8081,8082,8083,8888 -Pn -O --script banner 
# I found out that the service where Druid is running is on port 8888
msfconsole
serach druid

use exploit/linux/http/apache_druid_js_rce
# SET THE OPTIONS AND RUN
# WE GOT THE SHELL AND LOOKED FOR flag.txt
find / -name flag.txt 2>/dev/null
```

# Encoders

Over the 15 years of existence of the Metasploit Framework, `Encoders` have assisted with making payloads compatible with different processor architectures while at the same time helping with antivirus evasion. `Encoders` come into play with the role of changing the payload to run on different operating systems and architectures. These architectures include:

|`x64`|`x86`|`sparc`|`ppc`|`mips`|
|---|---|---|---|---|
Suppose we want to select an Encoder for an `existing payload`. Then, we can use the `show encoders` command within the `msfconsole` to see which encoders are available for our current `Exploit module + Payload` combination.
# Databases

`Databases` in `msfconsole` are used to keep track of your results. It is no mystery that during even more complex machine assessments, much less entire networks, things can get a little fuzzy and complicated due to the sheer amount of search results, entry points, detected issues, discovered credentials, etc.
## Setting up the Database

First, we must ensure that the PostgreSQL server is up and running on our host machine. To do so, input the following command:
#### PostgreSQL Status

```shell
sudo service postgresql status
```
#### Start PostgreSQL
```shell
sudo systemctl start postgresql
```

After starting PostgreSQL, we need to create and initialize the MSF database with `msfdb init`.
#### MSF - Initiate a Database

```shell
sudo msfdb init
```

Sometimes an error can occur if Metasploit is not up to date. This difference that causes the error can happen for several reasons. First, often it helps to update Metasploit again (`apt update`) to solve this problem. Then we can try to reinitialize the MSF database.

If the initialization is skipped and Metasploit tells us that the database is already configured, we can recheck the status of the database.

```shell
sudo msfdb status
```

If this error does not appear, which often happens after a fresh installation of Metasploit, then we will see the following when initializing the database:

```shell
sudo msfdb init
```

After the database has been initialized, we can start `msfconsole` and connect to the created database simultaneously.
#### MSF - Connect to the Initiated Database

```shell
sudo msfdb run
```
If, however, we already have the database configured and are not able to change the password to the MSF username, proceed with these commands:
#### MSF - Reinitiate the Database

```shell
msfdb reinit
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
sudo service postgresql restart
msfconsole -q

msf6 > db_status
```

Now, we are good to go. The `msfconsole` also offers integrated help for the database. This gives us a good overview of interacting with and using the database.
#### MSF - Database Options

```shell
help database
```
## Using the Database

With the help of the database, we can manage many different categories and hosts that we have analyzed. Alternatively, the information about them that we have interacted with using Metasploit. These databases can be exported and imported. This is especially useful when we have extensive lists of hosts, loot, notes, and stored vulnerabilities for these hosts. After confirming that the database is successfully connected, we can organize our `Workspaces`.
#### Workspaces

We can think of `Workspaces` the same way we would think of folders in a project. We can segregate the different scan results, hosts, and extracted information by IP, subnet, network, or domain.

To view the current Workspace list, use the `workspace` command. Adding a `-a` or `-d` switch after the command, followed by the workspace's name, will either `add` or `delete` that workspace to the database.
Notice that the default Workspace is named `default` and is currently in use according to the `*` symbol. Type the `workspace [name]` command to switch the presently used workspace. Looking back at our example, let us create a workspace for this assessment and select it.

```shell
# Create a workspace
workspace -a Target_1
# Switch to workspace
workspace Target_1
```
## Importing Scan Results

Next, let us assume we want to import a `Nmap scan` of a host into our Database's Workspace to understand the target better. We can use the `db_import` command for this. After the import is complete, we can check the presence of the host's information in our database by using the `hosts` and `services` commands. Note that the `.xml` file type is preferred for `db_import`.

```shell
# Stored Nmap Scan
cat Target.nmap
# Import scan
db_import Target.xml

# Then we can type the commands:
hosts

services
```
## Using Nmap Inside MSFconsole

Alternatively, we can use Nmap straight from msfconsole! To scan directly from the console without having to background or exit the process, use the `db_nmap` command.
#### MSF - Nmap

```shell
db_nmap -sV -sS 10.10.10.8
```
## Data Backup

After finishing the session, make sure to back up our data if anything happens with the PostgreSQL service. To do so, use the `db_export` command.

```shell
db_export -h
```

This data can be imported back to msfconsole later when needed. Other commands related to data retention are the extended use of `hosts`, `services`, and the `creds` and `loot` commands.
## Hosts

The `hosts` command displays a database table automatically populated with the host addresses, hostnames, and other information we find about these during our scans and interactions. For example, suppose `msfconsole` is linked with scanner plugins that can perform service and OS detection. In that case, this information should automatically appear in the table once the scans are completed through msfconsole. Again, tools like Nessus, NexPose, or Nmap will help us in these cases.

Hosts can also be manually added as separate entries in this table. After adding our custom hosts, we can also organize the format and structure of the table, add comments, change existing information, and more.
#### MSF - Stored Hosts

```shell
hosts -h
```
## Services

The `services` command functions the same way as the previous one. It contains a table with descriptions and information on services discovered during scans or interactions. In the same way as the command above, the entries here are highly customizable.
#### MSF - Stored Services of Hosts

```shell
services -h
```
## Credentials

The `creds` command allows you to visualize the credentials gathered during your interactions with the target host. We can also add credentials manually, match existing credentials with port specifications, add descriptions, etc.
#### MSF - Stored Credentials

```shell
creds -h
```
## Loot

The `loot` command works in conjunction with the command above to offer you an at-a-glance list of owned services and users. The loot, in this case, refers to hash dumps from different system types, namely hashes, passwd, shadow, and more.
# Plugins

The use of plugins makes a pentester's life even easier, bringing the functionality of well-known software into the `msfconsole` or Metasploit Pro environments. Whereas before, we needed to cycle between different software to import and export results, setting options and parameters over and over again, now, with the use of plugins, everything is automatically documented by msfconsole into the database we are using and hosts, services and vulnerabilities are made available at-a-glance for the user. [Plugins](https://web.archive.org/web/20240302133153/https://www.rubydoc.info/github/rapid7/metasploit-framework/Msf/Plugin) work directly with the API and can be used to manipulate the entire framework. They can be useful for automating repetitive tasks, adding new commands to the `msfconsole`, and extending the already powerful framework.
## Using Plugins

To start using a plugin, we will need to ensure it is installed in the correct directory on our machine. Navigating to `/usr/share/metasploit-framework/plugins`, which is the default directory for every new installation of `msfconsole`, should show us which plugins we have to our availability:

```shell
ls /usr/share/metasploit-framework/plugins
```
If the plugin is found here, we can fire it up inside `msfconsole` and will be met with the greeting output for that specific plugin, signaling that it was successfully loaded in and is now ready to use:
#### MSF - Load Nessus

```shell
load nessus
```

If the plugin is not installed correctly, we will receive the following error upon trying to load it.

```shell
load Plugin_That_Does_Not_Exist
```
## Installing new Plugins

New, more popular plugins are installed with each update of the Parrot OS distro as they are pushed out towards the public by their makers, collected in the Parrot update repo. To install new custom plugins not included in new updates of the distro, we can take the .rb file provided on the maker's page and place it in the folder at `/usr/share/metasploit-framework/plugins` with the proper permissions.

For example, let us try installing [DarkOperator's Metasploit-Plugins](https://github.com/darkoperator/Metasploit-Plugins.git). Then, following the link above, we get a couple of Ruby (`.rb`) files which we can directly place in the folder mentioned above.
#### Downloading MSF Plugins

```shell
git clone https://github.com/darkoperator/Metasploit-Plugins
ls Metasploit-Plugins
```
Here we can take the plugin `pentest.rb` as an example and copy it to `/usr/share/metasploit-framework/plugins`.
#### MSF - Copying Plugin to MSF

```shell
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```
Afterward, launch `msfconsole` and check the plugin's installation by running the `load` command. After the plugin has been loaded, the `help menu` at the `msfconsole` is automatically extended by additional functions.
#### MSF - Load Plugin

```shell
msfconsole -q
```

Many people write many different plugins for the Metasploit framework. They all have a specific purpose and can be an excellent help to save time after familiarizing ourselves with them. Check out the list of popular plugins below:

- nMap (pre-installed)	
- NexPose (pre-installed)	
- Nessus (pre-installed)
- Mimikatz (pre-installed V.1)	
- Stdapi (pre-installed)	
- RailgunPriv	
- Incognito (pre-installed)	
- Darkoperator's
## Mixins

The Metasploit Framework is written in Ruby, an object-oriented programming language. This plays a big part in what makes `msfconsole` excellent to use. Mixins are one of those features that, when implemented, offer a large amount of flexibility to both the creator of the script and the user.

Mixins are classes that act as methods for use by other classes without having to be the parent class of those other classes. Thus, it would be deemed inappropriate to call it inheritance but rather inclusion. They are mainly used when we:

1. Want to provide a lot of optional features for a class.
2. Want to use one particular feature for a multitude of classes.
# Sessions

MSFconsole can manage multiple modules at the same time. This is one of the many reasons it provides the user with so much flexibility. This is done with the use of `Sessions`, which creates dedicated control interfaces for all of your deployed modules.
## Using Sessions

While running any available exploits or auxiliary modules in msfconsole, we can background the session as long as they form a channel of communication with the target host. This can be done either by pressing the `[CTRL] + [Z]` key combination or by typing the `background` command in the case of Meterpreter stages. This will prompt us with a confirmation message. After accepting the prompt, we will be taken back to the msfconsole prompt (`msf6 >`) and will immediately be able to launch a different module.
#### Listing Active Sessions

We can use the `sessions` command to view our currently active sessions.

```shell
sessions
```
#### Interacting with a Session

You can use the `sessions -i [no.]` command to open up a specific session.

```shell
sessions -i 1
```

This is specifically useful when we want to run an additional module on an already exploited system with a formed, stable communication channel.

This can be done by backgrounding our current session, which is formed due to the success of the first exploit, searching for the second module we wish to run, and, if made possible by the type of module selected, selecting the session number on which the module should be run. This can be done from the second module's `show options` menu.
## Jobs

If, for example, we are running an active exploit under a specific port and need this port for a different module, we cannot simply terminate the session using `[CTRL] + [C]`. If we did that, we would see that the port would still be in use, affecting our use of the new module. So instead, we would need to use the `jobs` command to look at the currently active tasks running in the background and terminate the old ones to free up the port.
#### Viewing the Jobs Command Help Menu

We can view the help menu for this command, like others, by typing `jobs -h`.
#### Viewing the Exploit Command Help Menu

When we run an exploit, we can run it as a job by typing `exploit -j`. Per the help menu for the `exploit` command, adding `-j` to our command. Instead of just `exploit` or `run`, will "run it in the context of a job."
#### Listing Running Jobs

To list all running jobs, we can use the `jobs -l` command. To kill a specific job, look at the index no. of the job and use the `kill [index no.]` command. Use the `jobs -K` command to kill all running jobs.

```shell
exploit -h

exploit -j

jobs -l
```
## Exercise

### The target has a specific web application running that we can find by looking into the HTML source code. What is the name of that web application?

### Find the existing exploit in MSF and use it to get a shell on the target. What is the username of the user you obtained a shell with?

```shell
msfconsole
# In msfconsole
	use exploit/linux/http/elfinder_archive_cmd_injection
	# Set the options
	and RUN
```

www-data
### The target system has an old version of Sudo running. Find the relevant exploit and get root access to the target system. Find the flag.txt file and submit the contents of it as the answer.

I had to set to backgroud the current sessions

```shell
# Over the meterpreter session 
sysinfo # Ubuntu 20.04
shell
# Over the shell
	sudo -V # sudo 1.8.31
	exit
# Backgroud the meterpreter session to search and use another exploit
background
use exploit/linux/local/sudo_baron_samedit
# set options (we set session 1 in the options)
run
# We got a root session (WE ARE IN)
```