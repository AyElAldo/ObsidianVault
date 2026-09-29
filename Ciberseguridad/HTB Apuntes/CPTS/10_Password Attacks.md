# Introduction

`Confidentiality`, `integrity`, and `availability` are at the core of every infosec practitioner's responsibilities. Without maintaining a balance among them, we cannot ensure the security of our enterprises. This balance is preserved by auditing and accounting for each file, object, and host in the environment; by validating that users have appropriate permissions (authorization) to access those resources; and by verifying each user's identity (authentication) before granting access. Most breaches can be traced back to the breakdown of one of these three principles.
## Authentication

Authentication, at its core, is the validation of your identity by presenting a combination of four factors to a validation mechanism. They are:

1. `Something you know`: a password, passcode, PIN, passphrase, etc.
2. `Something you have`: an ID card, smart card, trusted device/phone, etc.
3. `Something you are`: biometric characteristics such as fingerprint, face recognition, iris/retina, voice, etc.
4. `Somewhere you are`: geolocation, IP address, etc.
## The use of passwords

The most common and widely used authentication method is still the use of passwords. But what is a password? A password or passphrase can be generally defined as `a combination of letters, numbers, and symbols in a string for identity validation.` For example, if we work with passwords and take a standard 8-digit password that consists only of upper case letters and numbers, we would get a total of `36⁸` (`208,827,064,576`) possible passwords.
# Introduction to Password Cracking

Passwords are commonly `hashed` when stored, in order to provide some protection in the event they fall into the hands of an attacker. `Hashing` is a mathematical function which transforms an arbitrary number of input bytes into a (typically) fixed-size output; common examples of hash functions are `MD5`, and `SHA-256`.

Take the password `Soccer06!` for example. The corresponding `MD5` and `SHA-256` hashes can be generated with the following commands:

```shell
bmdyy@htb:~$ echo -n Soccer06! | md5sum
40291c1d19ee11a7df8495c4cccefdfa  -

bmdyy@htb:~$ echo -n Soccer06! | sha256sum
a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93  -
```

Hash functions are designed to work in `one direction`. This means it should not be possible to figure out what the original password was based on the hash alone. When attackers attempt to do this, it is called `password cracking`. Common techniques are to use `rainbow tables`, to perform `dictionary attacks`, and typically as a last resort, to perform `brute-force attacks`.
## Rainbow tables

Rainbow tables are large pre-compiled maps of input and output values for a given hash function. These can be used to very quickly identify the password if its corresponding hash has already been mapped.

| Password   | MD5 Hash                         |
| ---------- | -------------------------------- |
| 123456     | e10adc3949ba59abbe56e057f20f883e |
| 12345      | 827ccb0eea8a706c4c34a16891f84e7b |
| 123456789  | 25f9e794323b453885f5181f1b624d0b |
| password   | 5f4dcc3b5aa765d61d8327deb882cf99 |
| iloveyou   | f25a2fc72690b780b2a14e140ef6a9e0 |
| princess   | 8afa847f50a716e64932d995c8e7435a |
| 1234567    | fcea920f7412b5da7be0cf42b8c93759 |
| rockyou    | f806fc5a2a0d5ba2471600758452799c |
| 12345678   | 25d55ad283aa400af464c76d713c07ad |
| abc123     | e99a18c428cb38d5f260853678922e03 |
| ...SNIP... | ...SNIP...                       |
Because rainbow tables are such a powerful attack, `salting` is used. A `salt`, in cryptographic terms, is a random sequence of bytes added to a password before it is hashed. To maximize impact, salts should not be reused. 

A salt is not a secret value — when a system goes to check an authentication request, it needs to know what salt was used so that it can check if the password hash matches. For this reason, salts are typically prepended to corresponding hashes. The reason this technique works against rainbow tables is that even if the correct password has been mapped, the combination of salt and password has likely not (especially if the salt contains non-printable characters). To make rainbow tables effective again, an attacker would need to update their mapping to account for every possible salt. A salt consisting of `just one single byte` would mean the `15 billion` entries from before would have to be `3.84 trillion` (factor of 256).
## Brute-force attack

A `brute-force` attack involves attempting every possible combination of letters, numbers, and symbols until the correct password is discovered. Obviously, this can take a very long time—especially for long passwords—however shorter passwords (<9 characters) are viable targets, even on consumer hardware. Brute-forcing is the only password cracking technique that is `100% effective` - in that, given enough time, any password will be cracked with this technique.

**Note:** Brute-forcing speeds depend heavily on the hashing algorithm and hardware that is used. On a typical company laptop, a tool like `hashcat` might be able to guess over `five million` passwords per second when attacking MD5, while at the same time only managing `ten thousand` per second when targeting a DCC2 hash.
## Dictionary attack

A `dictionary` attack, otherwise known as a `wordlist` attack, is one of the most `efficient` techniques for cracking passwords, especially when operating under time-constraints as penetration testers usually do. Rather than attempting every possible combination of characters, a list containing statistically likely passwords is used. Well-known wordlists for password cracking are [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) and those included in [SecLists](https://github.com/danielmiessler/SecLists).
## Exercise

### What is the SHA1 hash for `Academy#2025`?
```shell
echo -n Academy#2025 | sha1sum

750fe4b402dc9f91cedf09b652543cd85406be8c
```
# Introduction to John The Ripper

[John the Ripper](https://github.com/openwall/john) (aka. `JtR` aka. `john`) is a well-known penetration testing tool used for cracking passwords through various attacks including brute-force and dictionary. It is open-source software initially developed for UNIX-based systems and was first released in 1996. It has become a staple of the security industry due to its various capabilities. The `"jumbo"` variant is recommended for our uses, as it has performance optimizations, additional features such as multilingual word lists, and support for 64-bit architectures.

## Cracking modes

#### Single crack mode

`Single crack mode` is a rule-based cracking technique that is most useful when targeting Linux credentials. It generates password candidates based on the victim's username, home directory name, and [GECOS](https://en.wikipedia.org/wiki/Gecos_field) values (full name, room number, phone number, etc.). These strings are run against a large set of rules that apply common string modifications seen in passwords (e.g. a user whose real name is `Bob Smith` might use `Smith1` as their password).

Based on the contents of the file, it can be inferred that the victim has the username `r0lf`, the real name `Rolf Sebastian`, and the home directory `/home/r0lf`. Single crack mode will use this information to generate candidate passwords and test them against the hash. We can run the attack with the following command:

```shell
john --single passwd
```
#### Wordlist mode

`Wordlist mode` is used to crack passwords with a dictionary attack, meaning it attempts all passwords in a supplied wordlist against the password hash. The basic syntax for the command is as follows:

```shell
john --wordlist=<wordlist_file> <hash_file>
```

The wordlist file (or files) used for cracking password hashes must be in plain text format, with one word per line. Multiple wordlists can be specified by separating them with a comma. Rules, either custom or built-in, can be specified by using the `--rules` argument. These can be applied to generate candidate passwords using transformations such as appending numbers, capitalizing letters and adding special characters.
#### Incremental mode

`Incremental mode` is a powerful, brute-force-style password cracking mode that generates candidate passwords based on a statistical model ([Markov chains](https://en.wikipedia.org/wiki/Markov_chain)). It is designed to test all character combinations defined by a specific character set, prioritizing more likely passwords based on training data.

The basic syntax is:

```shell
john --incremental <hash_file>
```

By default, JtR uses predefined incremental modes specified in its configuration file (`john.conf`), which define character sets and password lengths. You can customize these or define your own to target passwords that use special characters or specific patterns.

```shell
grep '# Incremental modes' -A 100 /etc/john/john.conf
```

**Note:** This mode can be resource-intensive and slow, especially for long or complex passwords. Customizing the character set and length can improve performance and focus the attack.
## Identifying hash formats

Sometimes, password hashes may appear in an unknown format, and even John the Ripper (JtR) may not be able to identify them with complete certainty. For example, consider the following hash:

```shell
hashid -j 193069ceb0461e1d40d216e32c79c704
```

Unfortunately, in our example it is still quite unclear what format the hash is in. This will sometimes be the case, and is simply one of the "problems" you will encounter as a pentester. Many times, the context of where the hash came from will be enough to make an educated case on the format.
## Cracking files

It is also possible to crack password-protected or encrypted files with JtR. Multiple `"2john"` tools come with JtR that can be used to process files and produce hashes compatible with JtR. The generalized syntax for these tools is:

```shell
<tool> <file_to_crack> > file.hash
```

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
## Exercise
### Use single-crack mode to crack r0lf's password.

```bash
# passwd 
r0lf:$6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/:0:0:Rolf Sebastian:/home/r0lf:/bin/bash
```
Based on Hashes.com we can get: $6$ues25dIanlctrWxg$nZHVz2z4kCy1760Ee28M1xtHdGoy0C2cYzZ8l2sVa1kIa8K9gAcdBP.GI6ng/qA4oaMrgElZ1Cb9OeXO4Fvy3/ - Possible algorithms: sha512crypt $6$, SHA512 (Unix)

But we just use single 

```shell
john --single passwd
```

NAITSABES
### Use wordlist-mode with rockyou.txt to crack the RIPEMD-128 password.

```shell
john --wordlist='rockyou.txt' -format=ripemd-128 passwd2
```
# Introduction to Hashcat

[Hashcat](https://hashcat.net/) is a well-known password cracking tool for Linux, Windows, and macOS. From 2009 until 2015 it was proprietary software, but has since been released as open-source. Featuring fantastic GPU support, it can be used to crack a large variety of hashes. Similar to JtR, hashcat supports multiple attack (cracking) modes which can be used to efficiently attack password hashes.

- `-a` is used to specify the `attack mode`
- `-m` is used to specify the `hash type`
- `<hashes>` is a either a hash string, or a file containing one or more password hashes of the same type
- `[wordlist, rule, mask, ...]` is a placeholder for additional arguments that depend on the attack mode
## Hash types

Hashcat supports hundreds of different hash types, each of which is assigned a ID. A list of associated IDs can be generated by running `hashcat --help`.

## Attack modes

Hashcat has many different attack mode, including `dictionary`, `mask`, `combinator`, and `association`. In this section we will go over the first two, as they are likely the most common ones that you will need to use.

#### Dictionary attack

[Dictionary attack](https://hashcat.net/wiki/doku.php?id=dictionary_attack) (`-a 0`) is, as the name suggests, a dictionary attack. The user provides password hashes and a wordlist as input, and Hashcat tests each word in the list as a potential password until the correct one is found or the list is exhausted.

A wordlist alone is often not enough to crack a password hash. As was the case with JtR, `rules` can be used to perform specific modifications to passwords to generate even more guesses. The rule files that come with hashcat are typically found under `/usr/share/hashcat/rules`

#### Mask attack

[Mask attack](https://hashcat.net/wiki/doku.php?id=mask_attack) (`-a 3`) is a type of brute-force attack in which the keyspace is explicitly defined by the user. For example, if we know that a password is eight characters long, rather than attempting every possible combination, we might define a mask that tests combinations of six letters followed by two numbers.

A mask is defined by combining a sequence of symbols, each representing a built-in or custom character set. Hashcat includes several built-in character sets:

| Symbol | Charset                             |
| ------ | ----------------------------------- |
| ?l     | abcdefghijklmnopqrstuvwxyz          |
| ?u     | ABCDEFGHIJKLMNOPQRSTUVWXYZ          |
| ?d     | 0123456789                          |
| ?h     | 0123456789abcdef                    |
| ?H     | 0123456789ABCDEF                    |
| ?s     | «space»!"#$%&'()*+,-./:;<=>?@[]^_`{ |
| ?a     | ?l?u?d?s                            |
| ?b     | 0x00 - 0xff                         |
### Use a dictionary attack to crack the first password hash. (Hash: e3e3ec5831ad5e7288241960e5d4fdb8)

```shell
hashcat -m 0 hash1 /usr/share/wordlists/rockyou.txt
```
### Use a dictionary attack with rules to crack the second password hash. (Hash: 1b0556a75770563578569ae21392630c)

```shell
hashcat -m 0 hash2 /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
### Use a mask attack to crack the third password hash. (Hash: 1e293d6912d074c0fd15844d803400dd)

```shell
hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```
# Writing Custom Wordlists and Rules

Many users create their passwords based on `simplicity rather than security`. To mitigate this human tendency (which often undermines security measures), password policies can be implemented on systems to enforce specific password requirements. For instance, a system might enforce the inclusion of uppercase letters, special characters, and numbers. Most password policies mandate a minimum length—typically eight characters—and require at least one character from each specified category.

We can use Hashcat to combine lists of potential names and labels with specific mutation rules to create custom wordlists. Hashcat uses a specific syntax to define characters, words, and their transformations. The complete syntax is documented in the official [Hashcat rule-based attack documentation](https://hashcat.net/wiki/doku.php?id=rule_based_attack), but the examples below are sufficient to understand how Hashcat mutates input words.

|**Function**|**Description**|
|---|---|
|`:`|Do nothing|
|`l`|Lowercase all letters|
|`u`|Uppercase all letters|
|`c`|Capitalize the first letter and lowercase others|
|`sXY`|Replace all instances of X with Y|
|`$!`|Add the exclamation character at the end|
Each rule is written on a new line and determines how a given word should be transformed. If we write the functions shown above into a file, it may look like this:

```shell
:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

We can use the following command to apply the rules in `custom.rule` to each word in `password.list` and store the mutated results in `mut_password.list`.

```shell
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

In this case, the single input word will produce fifteen mutated variants.
## Generating wordlists using CeWL

We can use a tool called [CeWL](https://github.com/digininja/CeWL) to scan potential words from a company's website and save them in a separate list. We can then combine this list with the desired rules to create a customized password list—one that has a higher probability of containing the correct password for an employee. We specify some parameters, like the depth to spider (`-d`), the minimum length of the word (`-m`), the storage of the found words in lowercase (`--lowercase`), as well as the file where we want to store the results (`-w`).

```shell
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```
## Exercise

For this sections exercise, imagine that we compromised the password hash of a `work email` belonging to `Mark White`. After performing a bit of OSINT, we have gathered the following information about Mark:

- He was born on `August 5, 1998`
- He works at `Nexura, Ltd.`
    - The company's password policy requires passwords to be at least 12 characters long, to contain at least one uppercase letter, at least one lowercase letter, at least one symbol and at least one number
- He lives in `San Francisco, CA, USA`
- He has a pet cat named `Bella`
- He has a wife named `Maria`
- He has a son named `Alex`
- He is a big fan of `baseball`

The password hash is: `97268a8ae45ac7d15c3cea4ce6ea550b`. Use the techniques covered in this section to generate a custom wordlist and ruleset targeting Mark specifically, and crack the password.
### What is Mark's password?

Create `mark.txt`

```shell
Mark
White
Nexura
SanFrancisco
Bella
Maria
Alex
baseball
1998
0805
080598
05081998
1998
```

Then, I created a wordlist with combinations and applying a rule according what the text said

`policy.rule` content:

```shell
$!
$#
$@
c $!
c $#
c $@
```

```shell
hashcat -m 0 hash.txt dict.txt -r policy.rule --stdout > final_wordlist.txt
```
 
 We get the Password `Baseball1998!`
# Cracking Protected Files

The use of file encryption is often neglected in both `private` and `professional` contexts. Even today, emails containing job applications, account statements, or contracts are frequently sent without encryption—sometimes in violation of legal regulations. For example, within the European Union, the [General Data Protection Regulation (GDPR)](https://gdpr-info.eu/) requires that personal data be encrypted both in transit and at rest. Nevertheless, it remains standard practice to discuss `confidential` topics or transmit `sensitive` data via email, which may be intercepted by attackers positioned to exploit these communication channels.

## Hunting for Encrypted Files

Many different extensions correspond to encrypted files—a useful reference list can be found on [FileInfo](https://fileinfo.com/filetypes/encoded). As an example, consider this command we might use to locate commonly encrypted files on a Linux system:

```shell
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
## Hunting for SSH keys

Certain files, such as SSH keys, do not have standard file extension. In cases like these, it may be possible to identify files by standard content such as header and footer values. For example, SSH private keys always begin with `-----BEGIN [...SNIP...] PRIVATE KEY-----`. We can use tools like `grep` to recursively search the file system for them during post-exploitation.

```shell
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

Some SSH keys are encrypted with a passphrase. With older PEM formats, it was possible to tell if an SSH key is encrypted based on the header, which contains the encryption method in use. Modern SSH keys, however, appear the same whether encrypted or not.
## Cracking encrypted SSH keys

As mentioned in a previous section, JtR has many different scripts for extracting hashes from files—which we can then proceed to crack. We can find these scripts on our system using the following command:

```shell
locate *2john*
```

For example, we could use the Python script `ssh2john.py` to acquire the corresponding hash for an encrypted SSH key, and then use JtR to try and crack it.

```shell
ssh2john.py SSH.private > ssh.hash
john --wordlist=rockyou.txt ssh.hash
```

We can then view the resulting hash:

```shell
john ssh.hash --show
```
## Cracking password-protected documents

Over the course of our careers, we are likely to encounter a wide variety of documents that are password-protected to restrict access to authorized individuals.

Today, most reports, documentation, and information sheets are commonly distributed as Microsoft Office documents or PDFs. John the Ripper (JtR) includes a Python script called `office2john.py`, which can be used to extract password hashes from all common Office document formats. These hashes can then be supplied to JtR or Hashcat for offline cracking. The cracking procedure remains consistent with other hash types.

```shell
office2john.py Protected.docx > protected-docx.hash
john --wordlist=rockyou.txt protected-docx.hash
john protected-docx.hash --show
```

The process for cracking PDF files is quite similar, as we simply swap out `office2john.py` for `pdf2john.py`.

```shell
df2john.py PDF.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
john pdf.hash --show
```
## Exercise
### Download the attached ZIP archive (cracking-protected-files.zip), and crack the file within. What is the password?

```shell
office2john Confidential.xlsx > xlsx.hash 
john --wordlist=/usr/share/wordlists/rockyou.txt xlsx.hash
john xlsx.hash --show
```
# Cracking Protected Archives

Besides standalone files, we will often run across `archives` and `compressed files`—such as ZIP files—which are protected with a password.

Let us assume the role of an employee at an administrative company and imagine that a client requests a summary of an analysis in various formats, such as Excel, PDF, Word, and a corresponding presentation. One approach would be to send these files individually. However, if we extend this scenario to a large organization managing multiple simultaneous projects, this method of file transfer can become cumbersome and may result in individual files being misplaced. In such cases, employees often rely on archive files, which allow them to organize necessary documents in a structured manner (typically using subfolders) before compressing them into a single, consolidated file.

There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.

A comprehensive list of archive file types can be found on [FileInfo](https://fileinfo.com/filetypes/compressed). Rather than typing them out manually, we can also query the data using a one-liner, apply filters as needed, and save the results to a file. At the time of writing, the website lists `365` archive file types.

```shell
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```
## Cracking ZIP files

The `ZIP` format is often heavily used in Windows environments to compress many files into one file. The process of cracking an encrypted ZIP file is similar to what we have seen already, except for using a different script to extract the hashes.

```shell
zip2john ZIP.zip > zip.hash
cat zip.hash
john --wordlist=rockyou.txt zip.hash
```
## Cracking OpenSSL encrypted GZIP files

It is not always immediately apparent whether a file is password-protected, particularly when the file extension corresponds to a format that does not natively support password protection. As previously discussed, `openssl` can be used to encrypt files in the `GZIP` format. To determine the actual format of a file, we can use the `file` command, which provides detailed information about its contents. For example:

```shell
file GZIP.gzip

# The HASH could be salted
```

When cracking OpenSSL encrypted files, we may encounter various challenges, including numerous false positives or complete failure to identify the correct password. To mitigate this, a more reliable approach is to use the `openssl` tool within a `for` loop that attempts to extract the contents directly, succeeding only if the correct password is found.

The following one-liner may produce several GZIP-related error messages, which can be safely ignored. If the correct password list is used, as in this example, we will see another file successfully extracted from the archive.

```shell
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```
## Cracking BitLocker-encrypted drives

[BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10) is a full-disk encryption feature developed by Microsoft for the Windows operating system. Available since Windows Vista, it uses the `AES` encryption algorithm with either 128-bit or 256-bit key lengths. If the password or PIN used for BitLocker is forgotten, decryption can still be performed using a recovery key—a 48-digit string generated during the setup process.

In enterprise environments, virtual drives are sometimes used to store personal information, documents, or notes on company-issued devices to prevent unauthorized access. To crack a BitLocker encrypted drive, we can use a script called `bitlocker2john` to [four different hashes](https://openwall.info/wiki/john/OpenCL-BitLocker): the first two correspond to the BitLocker password, while the latter two represent the recovery key. Because the recovery key is very long and randomly generated, it is generally not practical to guess—unless partial knowledge is available. Therefore, we will focus on cracking the password using the first hash (`$bitlocker$0$...`).

```shell
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
cat backup.hash
```

Once a hash is generated, either `JtR` or `hashcat` can be used to crack it. For this example, we will look at the procedure with `hashcat`. The hashcat mode associated with the `$bitlocker$0$...` hash is `-m 22100`. We supply the hash, specify the wordlist, and define the hash mode. Since this encryption uses strong AES encryption, cracking may take considerable time depending on hardware performance.

```shell
hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt
```
After successfully cracking the password, we can access the encrypted drive.
#### Mounting BitLocker-encrypted drives in Windows

The easiest method for mounting a BitLocker-encrypted virtual drive on Windows is to double-click the `.vhd` file. Since it is encrypted, Windows will initially show an error. After mounting, simply double-click the BitLocker volume to be prompted for the password.
#### Mounting BitLocker-encrypted drives in Linux (or macOS)

It is also possible to mount BitLocker-encrypted drives in Linux (or macOS). To do this, we can use a tool called [dislocker](https://github.com/Aorimn/dislocker). First, we need to install the package using `apt`:

```shell
sudo apt-get install dislocker
```

Next, we create two folders which we will use to mount the VHD.

```shell
sudo mkdir -p /media/bitlocker
sudo mkdir -p /media/bitlockermount
```

We then use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device), decrypt the drive using `dislocker`, and finally mount the decrypted volume:

```shell
sudo losetup -f -P Backup.vhd
sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

If everything was done correctly, we can now browse the files:

```shell
cd /media/bitlockermount/
ls -la
```

Once we have analyzed the files on the mounted drive, we can unmount it using the following commands:

```shell
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```

```shell
bitlocker2john -i Private.vhd > bitlocker.hashes 
grep "bitlocker\$0" bitlocker.hashes > bitlocker.hash
ohn bitlocker.hash --wordlist=/usr/share/wordlists/rockyou.txt
# francisco is the password
```

Now we go to our Windows Machine and mount the vhd and enter with the password `francisco`. We get the flag.txt.
# Network Services

During our penetration tests, every computer network we encounter will have services installed to manage, edit, or create content. All these services are hosted using specific permissions and are assigned to specific users. Apart from web applications, these services include (but are not limited to) `FTP`, `SMB`, `NFS`, `IMAP/POP3`, `SSH`, `MySQL/MSSQL`, `RDP`, `WinRM`, `VNC`, `Telnet`, `SMTP`, and `LDAP`.

Let us imagine that we want to manage a Windows server over the network. Accordingly, we need a service that allows us to access the system, execute commands on it, or access its contents via a GUI or the terminal. In this case, the most common services suitable for this are `RDP`, `WinRM`, and `SSH`. SSH is not as common on Windows, but it is the leading service for Linux-based systems.

All these services have an authentication mechanism using a username and password. Of course, these services can be modified and configured so that only predefined keys can be used for logging in, but they are configured with default settings in many cases.
## WinRM

[Windows Remote Management](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) (`WinRM`) is the Microsoft implementation of the [Web Services Management Protocol](https://docs.microsoft.com/en-us/windows/win32/winrm/ws-management-protocol) (`WS-Management`). It is a network protocol based on XML web services using the [Simple Object Access Protocol](https://docs.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary) (`SOAP`) used for remote management of Windows systems. It takes care of the communication between [Web-Based Enterprise Management](https://en.wikipedia.org/wiki/Web-Based_Enterprise_Management) (`WBEM`) and the [Windows Management Instrumentation](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) (`WMI`), which can call the [Distributed Component Object Model](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0) (`DCOM`).
