# Cryptography
The study of writing of solving codes.
**PKI**: The implementation of cryptology on a computer network

- **Encyption**: Process of converting plaintext to crypo text

Privacy and Confidentiality --> Integruty --> Authenticity --> Non-Repudiation
## Ciphers
A secret way of writing a code.
- Algorithms --> Public
- Keys --> Usually Secret
### History
- Julio Cesar
- Transpositional cipher
- One-time pad cipher
	- Key: Ex. 10,4,3,11,18
	- Plain: Hi Bob
	- Cipher: RMEZT

- **Stream cipher** encrypts of plain text data one bit or byte at a time. Ex. **Fish** and **RC4** (faster)
- **Block cipher** encrypts plain text data in blocks. Ex. **AES, DES**.
## Keys 
- **Digital key**: Value that is used to perform cryptographic operations.
	- Digital Signature
	- **MAC**: It is a hash function that uses a digital key to ensure the integrity and authenticity of the message.
	- Encryption

>[!Important] 
>Larger keys encrypts small bits of data, while shorter encrypt bulk data.

`Strenght = Lenght + Complexity`

**BCRYPT** is a key stretching algorithm that hashes a password and adds salt.
### Symetric
- **EDS**: IBM --> Nist 1970s. 48-bit key to encrypt 64-bit blocks of data
- **3DES**: Encrypt data three times
- **IDEA**: James Massey in 1991. 128-bit symmetric key. Ciphers 64-bit block
- **AES**: 128-bit block usiang a 128, 192, 256-BIT KEY. NIST --> 2001
- **RC4, RC5, RC6**: Ron Rivest. 
	- RC4 stream cipher 40-bit to 2048-bit keys
	- RC5 block cipher 1994. Block and keys are configurables
	- RC6 128-bit block cipher. Was sedigned to meet the requirmetns of the AES
- **Blowfish**: 64-bit block symmetric cipher in 1993. variable length keys.
- **Twofish**: 128 bit block cipher, 128, 192,256 keys in 1998.

> [!Important]
> Symmetric is faster than asymmetric

## Asymmetric
- Encryption
- Key exchange
- Digital signature
### Algorithms
- **Diffie Helman**: 1976 --> key exchange protocol
- **Rivest, Shamir and Adleman (RSA)**: cipher that can encrypt, decrypt, sign, and verify
- **Elliptic curve cryptography**: Uses smaller keys
- **Pretty good privacy (PGP) and (GPG)**: 1991 -- popular email encryption tool. **GPG is the open source version**
- **Digitak Signing Algorithm (DSA)**: Digital signature. NIST in 1994
## Hashing and Digital Signatures
The process of converting data of an arbitrary size to a unique value of a fixed size.
- MD5
- SHA-1, SHA-2, SHA-3
- Microsoft LANMAN
- HAVEL
### Digital signature
Combination of hashing and asymmetric crypto.

- Input --> hashA --> asymmetric (private key) --> **digital signature**
### Verification
- Input --> hashB 
- Digital Signature --> asymmetric (public key of signer) --> Original HashA
- if HashA == HashB then **OK**
### Attacks
- Brute Force
- Birthday attack -->
## Public Key Infrastructure (PKI)
It is an ecosystem copmprised of policies, procedires, software, and hardware neede to create, distribute, store, use, and revoke digital certs.
- Certificate Authority (CA):
- Registration Authority (RA): 
- Directory Server 
- End entity
### Digital cert
Electronic doc issued and signed by a trust entity.
`X.509 v3`: Defines what content can be by certificate
It is composed by:
- Serial number
- Algorithms
- Validity period
- Issuer name: Trusted entity that issued the certificate
- Subject identity
- Key value
- Key ussage
### CA primary functions
- Cert entities
- Established an ecosystem of trust.

Types of CA environments
- *Hierarchical*: Root and sub CA
- **Latelal**:

>[!info] Definitions
>- CRL revokes and ARL lists revoked CA and cross certs.
>- PKI standards: X.500
>- RA --> Authorizes the CA to sign the cert by validating the data.

![[Pasted image 20260211231419.png]]
## Secure Network

> [!info] Firewalls
> ![[Getting Started in Cybersecurity#Firewalls]]

> [!info]- SASE 
> ![[Getting Started in Cybersecurity#Secure Access Service Edge (SASE)]]

> [!info] SD-WAN
> ![[Getting Started in Cybersecurity#Software Define WAN (SD-WAN)]]

> [!info] Sandbox 
> ![[Getting Started in Cybersecurity#Sandbox]]


### Secure Perimeter
Can filter traffic at different OSI layers
- **Physical:** MAC
- **Transport:** Packet filtering, NAT filtering
- **Application:** Proxy filter
#### Limitations
- Cloud
- BTOD
- IoT
### Zero Trust
It is a strategy.
- Never trust, always verify
- Least privilege
- Assume your network is breached
#### The never trust principle
- Identification process
- Context-based aspects
#### Principle of Least Privilege
- **Privilege acces management (PAM)**: 
- **Define the project surface:** Assess the network, devices.
- **Apply the kipling method:** Wonder who, why, whe, what, where and how when preparing to solve a problem
#### The assume of breach principle
- Prepare for the worst
- Mirosegmentations
#### Methods
- Access (**ZTA**)
	- Role based acces control
	- Endpoint agent for greater visibility and control of computer devices
	- ZTNA: Establishes a secure session automatically
## Centralized Security Network Management
Refers to the act of gathering through logs, SNMP. It centralizes the management.
### Benefits
- High-level view
- Device integration
- Reduction of manual tasks
- Easier maintenance
- Easier audits
## Network Segmentation

Divides a network to protect it.
- **Link data:** VLAN
- **Network layer:** ACL
- Application layer:  

Jump box: Unique Access point to manage internal
Bastion box: Device that Faces the internet
### Benefits
- Easier management
- Reduction in the number of network broadcasts
- Minimzation of network congestion
- A limiting of attacks to a specific segment
- Greater protection of vulnerable devices
- A reduction of the scope of devices affected by compliance
## Fortigate
It is a **NGFW**.
- ULTRA-FAST SECURITY THROUGHOUT THEIR NETWORK
- A CONSISTENT REAL TIME
- AN EXCELLENT USER EXPERIENCE
- OPERATIONAL EFFICIENCTY AND AUTOMATED WORKFLOW
## Fortigate platform structure
- NGFW, ANTIVIRUS, WEB FILTER IPS
- FortiOS
- Security Processing Units (SPUs)
## Mortigate models
- **Entry-level:** FG-80F, FWF-80F
- **Mid-range:** FG-100F, FG-1000F, FG-4200F
- **High-end:** FG-4800, FG-7081F, FG-7121F, FG-5114C
## Key Fortigate
- Firewall auth
- VPNs
- Security scanning such as antivirus, web, filtering, and application control.
- Monitoring and logging.
### Forti OS
Runs on fortigate and products. Helps to centralize the devices
# Secure Switching and porting
- Protect physicak switches locally
- Separate switches
- Limit flood attacks by limiting MAC addresses per port
- Using ACL
- Configure Sticky or static MAC
- Add port authentication **802.1x**
## Data link
Port, MAC, VLAN <-- CAM table
## Security Protocols
SSH
IPSec
HTTPS
TLS
## Common Network Threats
- **Spoofing**: The attacker impersonates and authorized devide
- **Hijacking**: Incercept a conection to discover, and potencially modify
- **Replay**: Hijacking but not in a real-time
- **Transitive attack**: 
- **DOS**:
	- **Floods**: Overwhelm
	- **Ping of death**: Buffer overflow because of the reassembly attempt
	- Teardrop attacks: 
# AUTHENTICATION AND VERIFICATION
## Auth Methods
- **Inherence**: Fingerprints, Irises, retinas, facial patterns
- **Possesion:** Something you have. Private key, tokens
- **Behavior:** Voice id, mouse use characteristics
- **Knowledge:** Q&A, psswd
## Single Sing-On (SSO)
One auth is enough
- Same Sign-On
- Components
	- Users
	- Service Provder
	- Identity provider
### SSO protocol
- OAuth
- Security Assertion Markup Language (SAML)
	- Authentication
	- Attribute
	- Authorization
![[SSO.png]]
## Auth Framework, protocols and tools
Auth framework is a squema or plan how entities prove their entities within a system.

- **Remote auth dial-in user service (RADIUS**): AAA protocol.
	- Uses **UDP**
	- Client-server
		- Access Request
		- Acces-Reject, Access-Accept or Access-challenge
	- User Credential repository
- **Lightweight Directory Access protocol (LDAP)-Compliant Directory**
	- Suplicant
	- Authentication Proxy (Firewall)
	- LDAP server
- **TACACS+**
	- SIMILAR TO RADIUS
	- Uses **TCP**
	- Encrypts all AAA protocols
### Auth methods Protocols
- **Password auth protocol (PAP)**: Two-way handshake
- **Challenge handshake auth protocol (CHAP)**: Hash, Three-Way Handshake, random string for auth
### Auth Framework
- **802.1x.** Port-based network acces control. It has two parts: **AUTH** and **Verification** in the auth sequence.
	- Client
	- Intermediary
	- Authentication server
- **Extensible Auth Protocol (EAP)**
	- PEAP
	- LEAP --> WEP keys
# Access control and methods
AC Restricts access to a resource.
- Simple
- Complex
## Common AC Methods
- **Mandatory SC (MAC):** Doesnt allow any actor to change the sec parametrs
	- Bnaks, militar, government
- **Discretionary Acess control (DAC)**:
	- Personal PC, offices
- **Lattice-bassed acces control (LBAC)**: Hierarchy location and resources in levels (**public, classified, secret, top secret**)
- **Rule-set-based access control (RSBAC)**: LIKE ACL
- **Role-based accesscontrol (RBAC)**: 
- **Attributed-based access control (ABAC)**:
	- Attributes
		- Personal
		- Resource
		- Environmental
![[AC_Methods.png]]
## Access Control Best practices
- Governance
	- Provisioning
		- Assign roles, create accounts
	- Auth
		- Validate identity
	- Authorization
		- Validate permissions
	- Self-service
		- Passwd change
		- Update account info
	- Deprovisioning
		- Remove acces, permissions

![[BestPractices.png]]
# Secure Remote access
- Data privacy
- Data Integrity
- Auth
- Authorization 
- Accounting
## Methods
VPN is a private connection across a public network.
- **TYPES**
	- Site-to-site
	- Secure remote access
- **COMPONENTS**
	- Client
	- Server
	- Protocols
## SSL VPN
`Application and transport layer`
Tech that supports and encrypt sessions.
Client --> Browser
- TLS
- **PROS**
	- Quick and easY access to network
	- USER IDENTIFICATION PERMITS MORE GRANULAR AUTHORIZATION
- **CONS**
	- Browser exclusive
	- Limited number of protocols
### Tunnel vs Portal/Web
SSL VPN software must be installed on the endpoint
## IPSec
`Network layer`
- Auth Header(AH)
- Encapsulating Sec Payload (ESP)

1. Key exchange
2. Data Assembly
3. Auth
4. Encryption
5. Transmission, decryption and verification
# ENDPOINT SECURE ACCESS

## IoT
- Personal
- 
- Industrial
Many devices une poor or no encryption
### Secure IoT
- Identify device
- Register its information
- Connect all unidentified devices to an isolated network and register them using identifying info
- Most IoT devices do not support the installing of traditional security sofware.

> [!info] ENDPOINT HARDENING TECHNIQUES 
> ![[Getting Started in Cybersecurity#Endpoint Hardening Techniques]]

[!info] ENDPOINT HARDENING TECHNIQUES 
> ![[Getting Started in Cybersecurity#Endpoint Monitoring]]
# Secure Data and Applications
- Data
	- In use
	- In transit
	- At rest
- Ensures:
	- Confidenciality
	- Integrity
	- Availability
	- Non-repudiation
## Types of data
- CUI: Classified
- PHI: Health
- Sensitive
- PII: Biometric 
- PCI: Payment
- IT Security Info
- Export control records
- Student Education Records
## Data risk
Accidental data exposure
Unintentional data sharing
## Data Lifecycle
Creation - Storage - Usage - Archiving - Destruction
## Data Protection
- Training Data Obfuscation
- Encryption
- Data Resiliency
- Destruction
- Data Loss Prevention
## Data Privacy
- Includes public expectation
- Control own personla data
### Laws and regulations
- Emplyees obligations
	- Undersatng the data privacy regulations followed by their organizations
	- Adhere to all policies and procedures related to data pricavy
- Governments
	- GDPR
	- ISO
	- NIST
	- SOC 2
	- PIPEDA
	- CCPA
	- HIPAA
	- PCI DSS

> [!Important] Fortigate filters and Fortiguard
> # Fortigate filters and Fortiguard
> Web filtering controlls and track what people visit
> - Prevent Users from inapropiate content
> - Block categorized content
>**FORTIGATE works with FORTIGUARD Categories**. 
>FORTIGUARD:
>- ALLOW
>- BLOCK
>- MONITOR
>	- IP, URL, destination, Fortigate logs
>- WARNING
>- AUTHENTICATE

## App Hardening
- Use specific tools
- Train employees
- **TECHNIQUES**:
	- Web browser
	- Cookies
	- Add-Ons
	- Opo-up blocker
	- Active content

## Intrusion Prevention System (IPS)
Identifies malicious packets. Inspects networks traffic at a granular level.

FortiGate IPS uses IPS sensors to detect malicious traffic. IPS sensors is a collections of signatures and filters
### Techniques
- Protocol decoders
- Signatures
![[FortigateIPS.png]]
## Actions of IPS
- **Default**: Fortiguards updates
- **Allow**:
- **Monitor**: Log the activitie
- **Block**: Drops traffic
- **Reset**: Reset the session whenever the signature is triggered
- **Quarantine**: Enable packet logging
# CLOUD AND VIRTUALIZATION
## VM risks
Areas to secure:
	- Hypervisor: Integrity is critical to ensure the sec in all the VMs
	- Hardware
## Secure VMs
- Patching OS
- Acces Management
- Firewalls virtually in the environment
- Cloud-network segmentation
- Others
	- Limit VM-host connectivity
	- Remove unnecessary pieces of virtual hardware
	- Avoid **virtualization sprawl**: Overwhelminmg the admin of the VMs
	- Restrict physical and administrative access
## Threats
- VM scape
- Data remanence
	- Overwriting
	- Degaussing
	- Crypto-erasing
	- Shredding
- Privilege Scalation
	- Principle of Least Privilege
- Live VM migration attacks
	- Encrypt the VMs and secure data channels used in the migration
## Common Cloud Threats
Biggest threats
- Setup
- Configuration
- Deployment
### Common
- Data Loss
- Connectivity
- Improper logging and monitoring
- RIghts and data ownership
- Auth and Authorization
- VM creep: Use frequent auditing of cloud-based virtual machines and data
- Misconfigured storage
### API and CORS Policies
- Encrypt
- Cross-site policies.
## Cloud Hosted Security
- CNFs
- WAFs
- Email Gateways
- Others
	- Centralized auth
	- Virus scanning
	- Secure browsing

### Cloud Based Auth
- IAM
- SSO
- 2FA
- Cloud access security borker (CASB)
- Cloud browser (RBI): All netwrok is controlled in a sandbox
- Cloud-based antivirus services and sandboxes
	- AI
	- Lists

- **SOCaaS**
- **SECaaS**: One vendor, many solutions

> [!Impotant] SASE
> **SASE** is a cloud architexture that mix **NETWORK + SECaaS**
> ## COMPONENTS
> - FWaaS
> - SWG
> - ZTNA
# Securing the Cloud
HOW THEY CAN WORK TOGHETHER

![[CloudSecure.png]]

## Connections
- Proxy: Control actions to the service acording to the authorization
- Reverse Proxy: Load balance connections. Such as a WAF. 
- **Cloud access security broker (CASB)**: It extends the enterprise's policies to the cloud. APIs. Prevent data loss.

### API
- **REST**: Quick configuration and simple monitoring. Use a REST API to rewuest information from the server or send basic configuration changes
- **SOAP**: Uses an XML achema to package the request as an HTTP PORT request

Securing API is critical. We can secure by using:
- HTTPS
- API gateway
	- Verify the auth
	- Enforce API squema
- VPNs
