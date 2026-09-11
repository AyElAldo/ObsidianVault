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

|Password|MD5 Hash|
|---|---|
|123456|e10adc3949ba59abbe56e057f20f883e|
|12345|827ccb0eea8a706c4c34a16891f84e7b|
|123456789|25f9e794323b453885f5181f1b624d0b|
|password|5f4dcc3b5aa765d61d8327deb882cf99|
|iloveyou|f25a2fc72690b780b2a14e140ef6a9e0|
|princess|8afa847f50a716e64932d995c8e7435a|
|1234567|fcea920f7412b5da7be0cf42b8c93759|
|rockyou|f806fc5a2a0d5ba2471600758452799c|
|12345678|25d55ad283aa400af464c76d713c07ad|
|abc123|e99a18c428cb38d5f260853678922e03|
|...SNIP...|...SNIP...|