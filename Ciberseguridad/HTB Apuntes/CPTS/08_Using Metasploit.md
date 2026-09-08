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
