# Firewalls
## Evolution
- **Packet Filter/stateless firewall**: Source, port, protocols
	- Implicit
	- Explicit
- **Stateful firewall**: Tracking the 5-tuple. Any packet that doesnt correspond with the output is denied.
	It dives into data payloads of the communications.
- **UTM/Application/Third Generation firewall**: It explores Data Payload, Statemful + applickation layer filter and com bines protection like Antivirus, IPS, VPN.
- **Next-generation firewall**: Opperates like airport security. See the if you are carrying any malicious item. Send malicious content to a sandbox. 
# Network Access Control (NAC)

User, device, location, OS. 

IEEE 802.1X
- Client device
- Authenticator
- Authenticator server

- Captive Portal
## NAC
NAC grants individuals access to sensitive information and apllication based on their need to know.

NAC enables an org to manage and authenticate temporary users and devices through a self-service portal. 

It integrates with other security point products such as switches and network solutions through the open/RESTful application programmiong interface or SSH
### Importance

- Improved security 
- Cost savings
- Automation
- Enhanced IT experiences
- Ease of control

> [!Important] FortiNAC
> NAC made by Forninet.
# Sandbox
It is an isolated virtual space that confines the actions of an application.
## Zero-Day attack
It is when a bad actor exploits an unknown vulnerability.

Sandboxing was created as a defense again zero-day attacks.
## Sandbox Generations
- **First Generation**: They were unable to share threat intelligence with other security devices.
- **Second Generation**: Allowing better threat intelligence sharing.
- **Third Generation**: Threat analysis standard for AI-driven attacks.

> [!Important] FortiSandbox
> Leverage AI/ML to identify advanced threats in real time. 
# Web Application Firewall (WAF)
It is a software that monitors HTTP/HTTPS traffic.
Thy can stop:
- SQL Injection
- XSS
- File Inclusion
- Security Misconfiguration

- **First Gen**: 
- **Second Gen**: Learned application behavior in order to evaluate if access attempts were normal or irregular.
- **Third Gen**: AI/ML

>[!Important] Fortiweb
>It provides real-time protection. In can be integrated with Fortigate and FortiSandbox
# Secure Email Gateway (SEG)
Tech solution design to proect organization from email-based threats.
- SPAM
- PHISHING

It protects helps with `Authentication and identity verification`, `Content Filter`, `SPAM`, `malware filtering`, `Encryption` and `Data Loss Prevention (DLP)` through:
- Keyword matching
- Regular expressions
- Deep packet inspections
- Context-aware analysis
## Sender Policy Framework
It verifies the sending server's IP address and prevents email spoofing and sender address forgery.
- **DKIM**: It helps to verify the authenticity of email messages and the legitimacy of the senders through a signature.
- **DMARC**: It builds on **SPF** and **DKIM** to allow email senders to specify their policies and how to handle messages that fail authentication.
 
>[!Important] Fortimail
>SEG includes support for **SPF**, **DKIM** and **DMARC**.
# Content filtering
It is the process to restrict access to objectionable email, webpages o other suspicious items.

- Search engine filters: Text and Images classification with AI.
- Email Filters: 
- DNS Based Content Filters: Check the website during the resolution.
- Web Filters: It caregorizes websites.It is similar DNS-Based content filter.
## Benefits
- Block access to sites 
- Identifies phishing 
- Increases the bandwith efficiency
- Can increase productivity
# WiFi

IEEE 802.11 stardards: A wireless technology employed for the local area networking of devices.
## Pros 
- Mobility
## Cons
- Security
## Sec Protocols
- WEP
- WPA v1 y v2
	- Advanced encryption Standard
	- Shared passphrase
- WPA v3
	- More secure handshake
## Best Practices
- Change SSID and default password
- Update firmware

> [!Important] Fortinet Wireless solution
> - FortiAP
> - FortiWiFi
> - FortiSwitch
> - Fortigate
# Endpoint Hardening Techniques

>[!note] 
>A network is only as secure as its most vulnerable endpoint.

With the spread of IoT, the number of endpoints to be secured is increasing.
## Hardening endpoints
- Company networks
- Personal environments
### Administrative Controls
- Passwords 
- User restrictions
- Principle of least privilege: Only oermissions they need
### Local Endpoint Protection
- **DLP**: Can detect if someone is sending important information over the network. Prevents or limits the use of attachable drives like USB or external hard deviced.
- **OS and startup hardening**: 
- **Firmware**: Responsible for reporting hardware connected to device. By restricting firmware so that it loads  only approved software.
- **Full disk encryption**: TPM stores the key to encrypt the entire disk
### Endpoint Maintenance
- **Backups**: Updating OS and firmware. 
- **Policy check**:
# Endpoint Monitoring

Endpoints solution helps manage and protect various types of endpoints
## Endpoint Monitoring
- **IDS**:
- **Endpoint Protection Platforms EPP**:
	- Verify versions of software and firmware
	- Scan the local system for viruses and malware
	- Enforces DLP
- **EDR**:
	- Suspicious connection, program, or behavior through IOC
	- Uses IA to predict and recognize suspicious files and programs.
	- Send alerts to other connected endpoints and allow them to block the suspicious behavior.
## Securing BYOD endpoints
- Isolated network 
- Register information
	- hostname
	- Serial number
	- MAC/static IP address
- Install security software
# Security Orchestration, Automation and Response (SOAR)

It connects tools and define workflows. Those are translated into a playbook. 
You can replace time-consuming activities with SOAR.
## Playbooks
Sometimes called workflows that do the repetitive tasts and are the key of automation capibility of SOAR. 
# Security Information and Event Management (SIEM)
Collect, normalize and store eventos and alerts.
- Monitoring for user-behavior anomalies
- Watching for IOC
- AI/ML
It enhaced security
Regulatory Compliance
## Complexity of SIEM
Insufficient number of qualified prefessionals
Increases complexity and causes a lack of network visibility
## SIEM evolution
Informaation platform --> Threat intelligence center --> A fully integrated and automated center for security and network operations.

- **What is one method that SIEM uses to analyze data?**
	Watch for known indicators of compromise (IoC)
- **Which feature provides SIEM greater visibility into the entire network?**
	Analyzing logs and alerts from a single-pane-of-glass
- **Which feature is a characteristic of later SIEMs?**
	User and entity behavior analytics (UEBA)
# Software Define WAN (SD-WAN)
It is a technology that uses software to centralize connection.
It monitors the permormance of connections.
## PROS
- **Centralized Orchestration**: Manages deployments, configurations, and operations.
- **Direct cloud access**: It enables direct access to critical cloud services for all users, regardless of their location.
- **Better app performance**: 
- **Increased Business Agility**: Network planners can deploy updates to all networks  simultaneously.
- **Cost saving**: Allows traffic to be rputed efficiently over multiple channels incluiding MPLS circuits and the public internet over LTE and boradband.
- **Improved security**: It is recommended to use NGFW, IPS, encryption, antivirus, and sandboxing capabilities.
## Evolution
- **Need for higher bandwidth**: Basid load-balancing
- **Need for higher performance**: Reduces delays associated with MPLS installation
- **Need for unifies infrastructure**: Has reduces operational cost and improved ease of network management  
## Secure SD-WAN
It is a combination of  afirewall and SD-WAN functions in one device.
# Zero Trust
- Least privilege access.

It assumes that every devide is a threat. 
## Componenets
- Client
- Proxy
- Authentication

## Dynamic access control 
- Based on:
	- User identity 
	- Policy
	- Device identity and risk profile
## ZTNA Workflow

EMS acts as the zena policy server.

1. The endpoint connects to the ZRTNA acces proxy
2. Fortigate challenges the ebdpoint for device idetification
3. Endpoint sends the device's certificate to FortiGate, whica had been issued by FortiClient EMS.
4. FortGate applied  the 
# Cloud security
- IaaS
	- Networking, Storage, Physical servers, Virtualization
- PaaS
	- ..., Operating system, Middleware
- SaaS
	- ..., Data, Applications

Basic sec options
# Secure Access Service Edge (SASE)
Allows orgs to apply secure access.
- SD-WAN
- Secure Web Gateway (**SWG**)
- Firewall As a Service (**FWaaS**)
- Cloud Access Security Broker (**CASB**)
- Zero Trust Network Access (ZTNA)
- Centralized Management 

> [!Important]FortiSASE
> Secure access service edge.
> - **SWG**
> - **ZTNA**
> - Next Generation Dual Mode **CASB**
> - Firewall as a Service (**FWaaS**)
> -