#fortinet

 **Previous Courses**: [[Technical Introduction to Cybersecurity 3.0]] and [[Getting Started in Cybersecurity]]

![[Pasted image 20260304133421.png]]
## FortiGate Features
- Firewall auth, local and remote
- VPNs
- Security scanning such as antivirus, web filtering, and apllication control
- Monitoring and logging

VDOMs --> Virtual firewalls
Automation Stitches
- It is part of Fortinet security fabric
- iT GETS UPDATES FROM FortiGuard Labs

## Configuring System Settings and Basic Networking

Default admin: ***admin***
Create other admin asuers is recomendable.
### VLAN
- Isolation of network segments for security reasons
- Separating of traffic from different departments
## Routing
Default route: 0.0.0.0/0.0.0.0
# Firewall Policies

- Incoming and outgoing Interfaces
- Source: IP or user
- Destination: IP address or Internet Service
- Service: Destination port
- Schedule
## Fortigate
- Flow-based Inspection
- Proxy-Based Inspection: **Slower but safer**
### Auth
- Local password
- Remote password auth

>[!Note] Fortigate stores the local passwords in ForitAuth
### Configure Local Auth
1.  Create user account
2. Create a user group and add user to the group
3. Add the user group as the source for a firewall policy
4. Verify the configuration and monitor users.
### Configure Remote Auth
1. Add the remote server in Fortigate
2. Create a user group
3. Add the user group as the source for a firewall policy
4. Verify the configuration and monitor users.
LDAP Servers
# SSL Traffic
## Two different type of SSL Inspection in Fortigate
- **Certificate Inspection:** Inspects the SSL/TLS handshake 
	- Verifies the identity in each session. 
	- WEB FILTERING is the only security feature that can be used with SSL certificate Inspection 
- **Deep Inspection:** Decrypts incoming traffic
	- FG impersonates the recipian. It also protects from HTTPS, POP3S, IMAPS, and FTPS.

## Profiles of SSL inspectors
- Preloaded
	- certificate-inspector
		- Inspects TLS/SSL handshake
		- It causes certificate warnings only when FortiGate displays an encrypted replacement message
		- It is used only with web filtering
	- deep-inspector
		- It is used when applying all types of security scanning
		- Like MITM, it also causes certificate errors to appear in the browser
		- Decrypts the entire packet, scans it, and re-encrypts ir
	- no-inspector
- Cutom deep inspection
	- You can edit the preloaded profile

>[!Warning] CERTIFICATE WARNINGS appear when Forti encrypts traffic used a self-signed certificate
## Avoid Warnings
1. Download the Fortinet Cert and install it on all the workstations as a trusted root authority
2. Use a CA-Issued SSL Cert
# Blocking malware
FortiGuard Labs provides a database of signatures that allow FortiGate to identufy malware.
- Antivirus Scan: Signature
- Grayware Scan: Unwanted behaviour 
- ML/AI Scan: Detect Zero Dat attacks

## Configuration AV Profile
- Threat Windows Exe in Emial Attachments as Viruses
- Send files to FortiSandbox for inspection
- Include Mobile malware protection
- Quarantine
### Steps
1. Create an AV profile
2. Enable AV Scanning in the AV policy
3. Verify the configuration
4. Monitor AV protection

>[!Important] The deep inspection must be enable
# Control Web Access Using Web Filtering
Fortiguard can use differente filters

Each category contains websites that hace been assigned based on their domain contest.

fOrtiguard category filter. Not based on the URL.
- Allow
- Block 
- Monitor
- Warning 
- Authenticate
1. Validate Fortiguard sec sub license
2. Identify how FG categorizees the websitesConfgure a web filtering sec profile
3. Apply the web sec profile in the policy

# Fortigate IPS
To identify Fgate uses IPS Sensors. Some techniques overlap and complement each other
- Protocol Decoders: Identify traffic that does
- Signatures: Compares netwro ktraffic to a database of known threats, called signatures
## Configure
1. Select IPS sensor or create a custome one
2. Select the required signatures and filters for the sensor and decide whether you want to bloack
3. Apply the IPS sensor

### IPS Actions
- Default
- Allow
- Monitor
- Block
- Reset: Reset the session
- Quarantine
### Best practices
- Verify the IPS database is up to date
- Consider using the IPS sensors provided as initial templates for new custom ones
- Consider using the IPS both for incoming and outgoing traffic
- Ensure that SSL inspection is in place so that the IPS can examine all traffic
- Evaluate wheter you need to tune your IPS sensors 
# Controlling Applications Access

Helps imporve security and meet compliance standards in the traffic flow of network app
## Functions
	Monitoring
	Blocking
	Applying traffic shaping

## Configurations
1. Create an app control profile
2. Modify action in the app categories
3. Add the app control profile to a firewall policy
4. Verify the configuration
5. Monitor the logs regarding app access
# Creating IPSec VPN
IPSec suite of industry standard protocols ->this  secure connections are known as VPNs
## Features
- Data Auth
- Data Integrity
- Data Confidentiality
- Anti-replay protection
## Types of VPN
- Remote access VPN:
	- Allows a client device to connect to a remite network
	- The client always initiates the connection
- Site to site:
	- Allows networks in two different pgysicalocations to reach other securely between other FG devices, as well as with devices form other vendors.
	- Either side can initiate the connection
## Internet Key Exchange (IKE) Protocol
Is used to create the tunnels dynamically.
- IKEv1
	- Two phases
	- Must match:
		- IKE Mode
		- Auth
		- Encryption Algorithm
		- Hashing Algorithm
		- Diffie Helman Group
- IKEv2
	- Benefits_ fewer messages reduces the latency and the bandwidth
	- Better Reliability
	- Supports Extensible Auth Protocol (EAP)
	- Upport of PPK
	- Support asymmetric auth
	- Support of stronger algorithms
	- Better resilience agains DoS attacks.
	- **ENCAPSULATING SECURITY PAYLOAD (ESP)**
		- DES: Data Encryption Standard (weak)
		- 3DES: Slow permormance
		- AES: 
		- MD5: It is no longer recommended
		- SHA1: No longer recommended
		- SHA2: More secure than SHA1. 
## Best Practices
- Ensure your firewalls have the latest updates and sec patches installed
- Use encryption and hashing levels that meet your requirements
- Verify both peers support the ssame IPsec features
- Ensure the neede ports are open in all the firewalls in the traffic path
- Select the proper mode when using IKEv1

VPN Wizard
# Configuring Fortigate SSL VPN (Secure Socket Layer)
## Benefits
1. Use of common protocols HTTP/HTTPS (not blocked by firewalls)
2. Flexibility for client access: Web browser
3. Granular access: Admins select which resources the client are allowed to access
4. Integrity checks for Windows clients: This security feature ensures that remote devices connecting to the VPN with the policies of the organization. i.e. It can check if the client has an AV software installed and deny access if it doesn't
5. Cost effective: No additional license is required to use SSL VPN can be made available for download at no cost from the SSL portal.
## SSL VPN Functionality
- **Web mode**: 
	- Doesn't require any client software to be installed
	- Supports limited number of application an protocols. i.e. bookmarked, URLs, FTP servers, and so on.
- **Tunnel Mode**:
	- Provides full network access to remote users as if they were physically present on the corporate network
## Configuring 
1. Create the users and group that will be authorized to connect
2. Review, edit, or create SSL VPN portals
3. Configure the SSL VPN settings
4. Create a firewall policy to allow VPN traffic
## Best Practices
- Select the appropiate SSL VPN mode
- Reduce admin effort by using remote auth servers
- Use a valid SSL cert
- Use the principle of least priv when configuring firewall policies for VPN traffic
- Use the client integrity check
- If possible, do not allow connection
# Fortigate System Maintenance and Monitoring
## Functions
- Backing up conf
- Performing firmware Upgrades
	- (F)eatures
	- (M)ature
- Monitoring system performance
- Examining license
- Monitoring evento logs
# Configuring Fortinet Security Fabric
Enterprise arch.
Provides security:
- Integrated
- Automated
- Coordinated
## Benefits
A unified view of the entire network from a songle console

- MINIMAL OF TWO FORTIGATE FIREWALLS RUNNING IN NAT MODE
- ONE FORTIANALYZER OR A CLOUD LOGGING SOLUTION
## Configure
1. COnfigure one of the centralized logging platforms
2. On FG that will act as the root
	1. Enable the sec fabric
	2. Configure the Sec Fabric connector with the option Server as Fabric Root
	3. Configure logs to be sent to a supported logging platform
	4. Optionally, you can pre-authorize downstream devices
3. Join downstream devices to the Security Fabric
	1. Enable the Sec Fabric
	2. Configure the Sec Facrib connector with the option Join Existing Fabric
	3. Configure the address of the root firewall
4. On the root, authorize downstream devices
# High Availability
- Layer 1: Elects a device yo server as the primary
- Layer 2: Is selected by comparing the following parameters:
		Monitor Interfaces 
		Uptime 
		Priority 
		Serial Number 
- Layer 3:
		Monitor Interfaces --> fewest failed monitored interfaces
		Priority --> Highest priority value
		Uptime --> Highest HA uptime becomes primary
		Serial Number --> Unit with highest Serial Number becomes primary
## Protocols used in a HA Cluster
Core --> FGGCP
## Synchronized info
- Cluster conf settings (exceptions)
- FIB entried
- DHCP leases
- ARP table
- Fortiguard definitions
- IPsec tunnel SAs
- Info about current sessions
- Monitored interfaces
## Not Sync
Monitored Interfaces
Dashboard widgets
HA override conf
HA device priority value
HA management interface settings
In-band HA managemente interface
All license, except Fortitoken license
Cache
## HA modes
- Active-Passive
	- Only the device in the cluster processes traffic. The rest are on stand-by
- Active-Active
	- All devices in the cluster process traffic
## HA Requirements
- All clusters:
	- Model
	- Firmware version
	- Licensing
	- Hard drive conf
	- Operating mode
- During cluster creation:
	- HA group ID
	- Group name
	- Password
	- Heartbeat interface settings
## Best practices
- Use redundant, dedicated and matching heartbeat interfaces
- Use identucal connections for internal and external interfaces
- Match hardware and firmware
- Avoid making changes on secundary units
- Monitor critical interfaces
- Test Regulary

>[!Important] Session pick up synchronizes session in order to reduce the TCP sessions disruptions
# Configuring FortiLink

Protocol that allows adminstrators to manage FortiSwitch directly through the FortiGate interface without requirinh an additional license.
## Benefits
- Centralized Management
- Simplified Deployment
- Enhanced Security
- Scalability

FL uses LLDP, CAPWAP, HTTPS. Others like DHCP, DNS, NTP, SSH.
## Fortilink topologies
1. Single FG unit managing a stack of several FortiSwitch units
2. HA-mode FG units managing a stack of several FortiSwitch units
3. FortiLink mode ober a layer-3 network
4. Standalone FG unit with dual-homed FortiSwitch access
## Best practices
- Enable link aggregation to ensure redundancy and increase throughput between FG and FS.
- Keep firmware versions consistent across FG and FS to ensure compatibility
- Plan the use of VLANs and ports before deployment to minimize reconfiguration later
- Regulartly monitor switch status and logs to identify problems early.

>[!Important] HTTPS is the default protocol to send firmware images during upgrades. Fortilink y CAPWAP for heartbeat traffic









