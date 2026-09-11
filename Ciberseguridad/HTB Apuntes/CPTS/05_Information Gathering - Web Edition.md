![[1_PenetrationTestingProcess.png]]
The primary goals of web reconnaissance include:

- `Identifying Assets`: Uncovering all publicly accessible components of the target, such as web pages, subdomains, IP addresses, and technologies used. This step provides a comprehensive overview of the target's online presence.
- `Discovering Hidden Information`: Locating sensitive information that might be inadvertently exposed, including backup files, configuration files, or internal documentation. These findings can reveal valuable insights and potential entry points for attacks.
- `Analysing the Attack Surface`: Examining the target's attack surface to identify potential vulnerabilities and weaknesses. This involves assessing the technologies used, configurations, and possible entry points for exploitation.
- `Gathering Intelligence`: Collecting information that can be leveraged for further exploitation or social engineering attacks. This includes identifying key personnel, email addresses, or patterns of behaviour that could be exploited.
## Types of Reconnaissance

Web reconnaissance encompasses two fundamental methodologies: `active` and `passive` reconnaissance. Each approach offers distinct advantages and challenges, and understanding their differences is crucial for adequate information gathering.
### Active Reconnaissance

In active reconnaissance, the attacker `directly interacts with the target system` to gather information. This interaction can take various forms:

| Technique                | Description                                                                                                                     | Example                                                                                                                                           | Tools                                                                   | Risk of Detection                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `Port Scanning`          | Identifying open ports and services running on the target.                                                                      | Using Nmap to scan a web server for open ports like 80 (HTTP) and 443 (HTTPS).                                                                    | **Nmap, Masscan, Unicornscan**                                          | High: Direct interaction with the target can trigger intrusion detection systems (IDS) and firewalls.              |
| `Vulnerability Scanning` | Probing the target for known vulnerabilities, such as outdated software or misconfigurations.                                   | Running Nessus against a web application to check for SQL injection flaws or cross-site scripting (XSS) vulnerabilities.                          | **Nessus, OpenVAS, Nikto**                                              | High: Vulnerability scanners send exploit payloads that security solutions can detect.                             |
| `Network Mapping`        | Mapping the target's network topology, including connected devices and their relationships.                                     | Using traceroute to determine the path packets take to reach the target server, revealing potential network hops and infrastructure.              | **Traceroute, Nmap**                                                    | Medium to High: Excessive or unusual network traffic can raise suspicion.                                          |
| `Banner Grabbing`        | Retrieving information from banners displayed by services running on the target.                                                | Connecting to a web server on port 80 and examining the HTTP banner to identify the web server software and version.                              | **Netcat, curl**                                                        | Low: Banner grabbing typically involves minimal interaction but can still be logged.                               |
| `OS Fingerprinting`      | Identifying the operating system running on the target.                                                                         | Using Nmap's OS detection capabilities (`-O`) to determine if the target is running Windows, Linux, or another OS.                                | **Nmap, Xprobe2**                                                       | Low: OS fingerprinting is usually passive, but some advanced techniques can be detected.                           |
| `Service Enumeration`    | Determining the specific versions of services running on open ports.                                                            | Using Nmap's service version detection (`-sV`) to determine if a web server is running Apache 2.4.50 or Nginx 1.18.0.                             | **Nmap**                                                                | Low: Similar to banner grabbing, service enumeration can be logged but is less likely to trigger alerts.           |
| `Web Spidering`          | Crawling the target website to identify web pages, directories, and files.                                                      | Running a web crawler like Burp Suite Spider or OWASP ZAP Spider to map out the structure of a website and discover hidden resources.             | **Burp Suite Spider, OWASP ZAP Spider, Scrapy (customisable)**          | Low to Medium: Can be detected if the crawler's behaviour is not carefully configured to mimic legitimate traffic. |
| Technique                | Description                                                                                                                     | Example                                                                                                                                           | Tools                                                                   | Risk of Detection                                                                                                  |
| `Search Engine Queries`  | Utilising search engines to uncover information about the target, including websites, social media profiles, and news articles. | Searching Google for "`[Target Name] employees`" to find employee information or social media profiles.                                           | Google, DuckDuckGo, Bing, and specialised search engines (e.g., Shodan) | Very Low: Search engine queries are normal internet activity and unlikely to trigger alerts.                       |
| `WHOIS Lookups`          | Querying WHOIS databases to retrieve domain registration details.                                                               | Performing a WHOIS lookup on a target domain to find the registrant's name, contact information, and name servers.                                | whois command-line tool, online WHOIS lookup services                   | Very Low: WHOIS queries are legitimate and do not raise suspicion.                                                 |
| `DNS`                    | Analysing DNS records to identify subdomains, mail servers, and other infrastructure.                                           | Using `dig` to enumerate subdomains of a target domain.                                                                                           | dig, nslookup, host, dnsenum, fierce, dnsrecon                          | Very Low: DNS queries are essential for internet browsing and are not typically flagged as suspicious.             |
| `Web Archive Analysis`   | Examining historical snapshots of the target's website to identify changes, vulnerabilities, or hidden information.             | Using the Wayback Machine to view past versions of a target website to see how it has changed over time.                                          | Wayback Machine                                                         | Very Low: Accessing archived versions of websites is a normal activity.                                            |
| `Social Media Analysis`  | Gathering information from social media platforms like LinkedIn, Twitter, or Facebook.                                          | Searching LinkedIn for employees of a target organisation to learn about their roles, responsibilities, and potential social engineering targets. | LinkedIn, Twitter, Facebook, specialised OSINT tools                    | Very Low: Accessing public social media profiles is not considered intrusive.                                      |
| `Code Repositories`      | Analysing publicly accessible code repositories like GitHub for exposed credentials or vulnerabilities.                         | Searching GitHub for code snippets or repositories related to the target that might contain sensitive information or code vulnerabilities.        | GitHub, GitLab                                                          | Very Low: Code repositories are meant for public access, and searching them is not suspicious.                     |
# WHOIS

WHOIS is a widely used query and response protocol designed to access databases that store information about registered internet resources. Primarily associated with domain names, WHOIS can also provide details about IP address blocks and autonomous systems. Think of it as a giant phonebook for the internet, letting you look up who owns or is responsible for various online assets.

```shell
whois inlanefreight.com [...] Domain Name: inlanefreight.com Registry Domain ID: 2420436757_DOMAIN_COM-VRSN Registrar WHOIS Server: whois.registrar.amazon Registrar URL: https://registrar.amazon.com Updated Date: 2023-07-03T01:11:15Z Creation Date: 2019-08-05T22:43:09Z [...]
```
Each WHOIS record typically contains the following information:

- `Domain Name`: The domain name itself (e.g., example.com)
- `Registrar`: The company where the domain was registered (e.g., GoDaddy, Namecheap)
- `Registrant Contact`: The person or organization that registered the domain.
- `Administrative Contact`: The person responsible for managing the domain.
- `Technical Contact`: The person handling technical issues related to the domain.
- `Creation and Expiration Dates`: When the domain was registered and when it's set to expire.
- `Name Servers`: Servers that translate the domain name into an IP address.
## Why WHOIS Matters for Web Recon

WHOIS data serves as a treasure trove of information for penetration testers during the reconnaissance phase of an assessment. It offers valuable insights into the target organisation's digital footprint and potential vulnerabilities:

- `Identifying Key Personnel`: WHOIS records often reveal the names, email addresses, and phone numbers of individuals responsible for managing the domain. This information can be leveraged for social engineering attacks or to identify potential targets for phishing campaigns.
- `Discovering Network Infrastructure`: Technical details like name servers and IP addresses provide clues about the target's network infrastructure. This can help penetration testers identify potential entry points or misconfigurations.
- `Historical Data Analysis`: Accessing historical WHOIS records through services like [WhoisFreaks](https://whoisfreaks.com/) can reveal changes in ownership, contact information, or technical details over time. This can be useful for tracking the evolution of the target's digital presence.
## Scenario 1: Phishing Investigation

An email security gateway flags a suspicious email sent to multiple employees within a company. The email claims to be from the company's bank and urges recipients to click on a link to update their account information. A security analyst investigates the email and begins by performing a WHOIS lookup on the domain linked in the email.

The WHOIS record reveals the following:

- `Registration Date`: The domain was registered just a few days ago.
- `Registrant`: The registrant's information is hidden behind a privacy service.
- `Name Servers`: The name servers are associated with a known bulletproof hosting provider often used for malicious activities.

This combination of factors raises significant red flags for the analyst. The recent registration date, hidden registrant information, and suspicious hosting strongly suggest a phishing campaign. The analyst promptly alerts the company's IT department to block the domain and warns employees about the scam.

Further investigation into the hosting provider and associated IP addresses may uncover additional phishing domains or infrastructure the threat actor uses.
## Scenario 2: Malware Analysis

A security researcher is analysing a new strain of malware that has infected several systems within a network. The malware communicates with a remote server to receive commands and exfiltrate stolen data. To gain insights into the threat actor's infrastructure, the researcher performs a WHOIS lookup on the domain associated with the command-and-control (C2) server.

The WHOIS record reveals:

- `Registrant`: The domain is registered to an individual using a free email service known for anonymity.
- `Location`: The registrant's address is in a country with a high prevalence of cybercrime.
- `Registrar`: The domain was registered through a registrar with a history of lax abuse policies.
## Scenario 3: Threat Intelligence Report

A cybersecurity firm tracks the activities of a sophisticated threat actor group known for targeting financial institutions. Analysts gather WHOIS data on multiple domains associated with the group's past campaigns to compile a comprehensive threat intelligence report.

By analysing the WHOIS records, analysts uncover the following patterns:

- `Registration Dates`: The domains were registered in clusters, often shortly before major attacks.
- `Registrants`: The registrants use various aliases and fake identities.
- `Name Servers`: The domains often share the same name servers, suggesting a common infrastructure.
- `Takedown History`: Many domains have been taken down after attacks, indicating previous law enforcement or security interventions.
## Using WHOIS
The WHOIS output for `facebook.com` reveals several key details:

1. `Domain Registration`:
    
    - `Registrar`: RegistrarSafe, LLC
    - `Creation Date`: 1997-03-29
    - `Expiry Date`: 2033-03-30
      
    These details indicate that the domain is registered with RegistrarSafe, LLC, and has been active for a considerable period, suggesting its legitimacy and established online presence. The distant expiry date further reinforces its longevity.
2. `Domain Owner`:
    
    - `Registrant/Admin/Tech Organization`: Meta Platforms, Inc.
    - `Registrant/Admin/Tech Contact`: Domain Admin
     
    This information identifies Meta Platforms, Inc. as the organization behind `facebook.com`, and "Domain Admin" as the point of contact for domain-related matters. This is consistent with the expectation that Facebook, a prominent social media platform, is owned by Meta Platforms, Inc.
3. `Domain Status`:
    - `clientDeleteProhibited`, `clientTransferProhibited`, `clientUpdateProhibited`, `serverDeleteProhibited`, `serverTransferProhibited`, and `serverUpdateProhibited`

# DNS

The `Domain Name System` (`DNS`) acts as the internet's GPS, guiding your online journey from memorable landmarks (domain names) to precise numerical coordinates (IP addresses). Much like how GPS translates a destination name into latitude and longitude for navigation, DNS translates human-readable domain names (like `www.example.com`) into the numerical IP addresses (like `192.0.2.1`) that computers use to communicate.

## How DNS Works

Imagine you want to visit a website like `www.example.com`. You type this friendly domain name into your browser, but your computer doesn't understand words – it speaks the language of numbers, specifically IP addresses. So, how does your computer find the website's IP address? Enter DNS, the internet's trusty translator.

![Flowchart showing two main sections: User Database and Data Collection. Includes steps like 'Check User', 'Send to Source Database', 'Store Data', and 'Send to UI System'.|1436](https://mermaid.ink/svg/pako:eNptkk1uwjAQha8y8rpcIItWkAAtUNQmlSrksDDxlEQQT-QfJIS4ex2nNG1arzx-n5-ex3NhBUlkEdtr0ZSwSnMFfhm36w425DTEVDfOou60do15XGJxMBCLosQtjEb3MLk8vcCMnJIP156ceA3WFIiYZ6ikgWSdwatDfQZLkKKh4wn1dnBngyZceuQxKYWFNS39jjtTWfyCvdsgb2t9c-wN4-CU_A7dy8mPjFOeYuG0qU4IK6KDa4bgLdjck9ZpZcC_20e7dWmYbRroGU-JLKxFjZCh7h88C_KCv62Sf9RFUJd87GxJurLCtsH-cssuUlfMu8blit2xGnUtKul_-NKKObMl1pizyG-l0Iec5erqOeEsZWdVsMhqh3dMk9uXLPoQR-Mr10hhMamE73L9fYqysqSfuwEKc3T9BOe0sj4)

1. `Your Computer Asks for Directions (DNS Query)`: When you enter the domain name, your computer first checks its memory (cache) to see if it remembers the IP address from a previous visit. If not, it reaches out to a DNS resolver, usually provided by your internet service provider (ISP).
2. `The DNS Resolver Checks its Map (Recursive Lookup)`: The resolver also has a cache, and if it doesn't find the IP address there, it starts a journey through the DNS hierarchy. It begins by asking a root name server, which is like the librarian of the internet.
3. `Root Name Server Points the Way`: The root server doesn't know the exact address but knows who does – the Top-Level Domain (TLD) name server responsible for the domain's ending (e.g., .com, .org). It points the resolver in the right direction.
4. `TLD Name Server Narrows It Down`: The TLD name server is like a regional map. It knows which authoritative name server is responsible for the specific domain you're looking for (e.g., `example.com`) and sends the resolver there.
5. `Authoritative Name Server Delivers the Address`: The authoritative name server is the final stop. It's like the street address of the website you want. It holds the correct IP address and sends it back to the resolver.
6. `The DNS Resolver Returns the Information`: The resolver receives the IP address and gives it to your computer. It also remembers it for a while (caches it), in case you want to revisit the website soon.
7. `Your Computer Connects`: Now that your computer knows the IP address, it can connect directly to the web server hosting the website, and you can start browsing.
### The Hosts File

The `hosts` file is a simple text file used to map hostnames to IP addresses, providing a manual method of domain name resolution that bypasses the DNS process. While DNS automates the translation of domain names to IP addresses, the `hosts` file allows for direct, local overrides. This can be particularly useful for development, troubleshooting, or blocking websites.
The `hosts` file is located in `C:\Windows\System32\drivers\etc\hosts` on Windows and in `/etc/hosts` on Linux and MacOS. Each line in the file follows the format:

```shell
# Windows
<IP Address> <Hostname> [<Alias> ...]

# Linux
127.0.0.1 localhost 
192.168.1.10 devserver.local
```

To edit the `hosts` file, open it with a text editor using administrative/root privileges. Add new entries as needed, and then save the file. The changes take effect immediately without requiring a system restart.
or blocking unwanted websites by redirecting their domains to a non-existent IP address:

```linux
0.0.0.0       unwanted-site.com
```

Think of the DNS process as a relay race. Your computer starts with the domain name and passes it along to the resolver. The resolver then passes the request to the root server, the TLD server, and finally, the authoritative server, each one getting closer to the destination. Once the IP address is found, it's relayed back down the chain to your computer, allowing you to access the website.

### Key DNS Concepts

In the `Domain Name System` (`DNS`), a `zone` is a distinct part of the domain namespace that a specific entity or administrator manages. Think of it as a virtual container for a set of domain names. For example, `example.com` and all its subdomains (like `mail.example.com` or `blog.example.com`) would typically belong to the same DNS zone.

The zone file, a text file residing on a DNS server, defines the resource records (discussed below) within this zone, providing crucial information for translating domain names into IP addresses.

To illustrate, here's a simplified example of what a zone file, for `example.com` might look like:

```dns-zone
$TTL 3600 ; Default Time-To-Live (1 hour)
@ IN SOA ns1.example.com. admin.example.com. ( 
			2024060401 ; Serial number (YYYYMMDDNN) 
			3600 ; Refresh interval 
			900 ; Retry interval 
			604800 ; Expire time 
			86400 ) ; Minimum TTL 
@ IN NS ns1.example.com. 
@ IN NS ns2.example.com. 
@ IN MX 10 mail.example.com. 
www IN A 192.0.2.1 
mail IN A 198.51.100.1 
ftp IN CNAME www.example.com.
```

This file defines the authoritative name servers (`NS` records), mail server (`MX` record), and IP addresses (`A` records) for various hosts within the `example.com` domain.

DNS servers store various resource records, each serving a specific purpose in the domain name resolution process. Let's explore some of the most common DNS concepts:

| DNS Concept                 | Description                                                                      | Example                                                                                                                                 |
| --------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `Domain Name`               | A human-readable label for a website or other internet resource.                 | `www.example.com`                                                                                                                       |
| `IP Address`                | A unique numerical identifier assigned to each device connected to the internet. | `192.0.2.1`                                                                                                                             |
| `DNS Resolver`              | A server that translates domain names into IP addresses.                         | Your ISP's DNS server or public resolvers like Google DNS (`8.8.8.8`)                                                                   |
| `Root Name Server`          | The top-level servers in the DNS hierarchy.                                      | There are 13 root servers worldwide, named A-M: `a.root-servers.net`                                                                    |
| `TLD Name Server`           | Servers responsible for specific top-level domains (e.g., .com, .org).           | [Verisign](https://en.wikipedia.org/wiki/Verisign) for `.com`, [PIR](https://en.wikipedia.org/wiki/Public_Interest_Registry) for `.org` |
| `Authoritative Name Server` | The server that holds the actual IP address for a domain.                        | Often managed by hosting providers or domain registrars.                                                                                |
| `DNS Record Types`          | Different types of information stored in DNS.                                    | A, AAAA, CNAME, MX, NS, TXT, etc.                                                                                                       |
***Records:***

| Record Type | Full Name                 | Description                                                                                                                                 | Zone File Example                                                                              |
| ----------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `A`         | Address Record            | Maps a hostname to its IPv4 address.                                                                                                        | `www.example.com.` IN A `192.0.2.1`                                                            |
| `AAAA`      | IPv6 Address Record       | Maps a hostname to its IPv6 address.                                                                                                        | `www.example.com.` IN AAAA `2001:db8:85a3::8a2e:370:7334`                                      |
| `CNAME`     | Canonical Name Record     | Creates an alias for a hostname, pointing it to another hostname.                                                                           | `blog.example.com.` IN CNAME `webserver.example.net.`                                          |
| `MX`        | Mail Exchange Record      | Specifies the mail server(s) responsible for handling email for the domain.                                                                 | `example.com.` IN MX 10 `mail.example.com.`                                                    |
| `NS`        | Name Server Record        | Delegates a DNS zone to a specific authoritative name server.                                                                               | `example.com.` IN NS `ns1.example.com.`                                                        |
| `TXT`       | Text Record               | Stores arbitrary text information, often used for domain verification or security policies.                                                 | `example.com.` IN TXT `"v=spf1 mx -all"` (SPF record)                                          |
| `SOA`       | Start of Authority Record | Specifies administrative information about a DNS zone, including the primary name server, responsible person's email, and other parameters. | `example.com.` IN SOA `ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400` |
| `SRV`       | Service Record            | Defines the hostname and port number for specific services.                                                                                 | `_sip._udp.example.com.` IN SRV 10 5 5060 `sipserver.example.com.`                             |
| `PTR`       | Pointer Record            | Used for reverse DNS lookups, mapping an IP address to a hostname.                                                                          | `1.2.0.192.in-addr.arpa.` IN PTR `www.example.com.`                                            |
The "`IN`" in the examples stands for "Internet." It's a class field in DNS records that specifies the protocol family. In most cases, you'll see "`IN`" used, as it denotes the Internet protocol suite (IP) used for most domain names.
## Why DNS Matters for Web Recon

DNS is not merely a technical protocol for translating domain names; it's a critical component of a target's infrastructure that can be leveraged to uncover vulnerabilities and gain access during a penetration test:

- `Uncovering Assets`: DNS records can reveal a wealth of information, including subdomains, mail servers, and name server records. For instance, a `CNAME` record pointing to an outdated server (`dev.example.com` CNAME `oldserver.example.net`) could lead to a vulnerable system.
- `Mapping the Network Infrastructure`: You can create a comprehensive map of the target's network infrastructure by analysing DNS data. For example, identifying the name servers (`NS` records) for a domain can reveal the hosting provider used, while an `A` record for `loadbalancer.example.com` can pinpoint a load balancer. This helps you understand how different systems are connected, identify traffic flow, and pinpoint potential choke points or weaknesses that could be exploited during a penetration test.
- `Monitoring for Changes`: Continuously monitoring DNS records can reveal changes in the target's infrastructure over time. For example, the sudden appearance of a new subdomain (`vpn.example.com`) might indicate a new entry point into the network, while a `TXT` record containing a value like `_1password=...` strongly suggests the organization is using 1Password, which could be leveraged for social engineering attacks or targeted phishing campaigns.

# Digging DNS
## DNS Tools

DNS reconnaissance involves utilizing specialized tools designed to query DNS servers and extract valuable information. Here are some of the most popular and versatile tools in the arsenal of web recon professionals:

|Tool|Key Features|Use Cases|
|---|---|---|
|`dig`|Versatile DNS lookup tool that supports various query types (A, MX, NS, TXT, etc.) and detailed output.|Manual DNS queries, zone transfers (if allowed), troubleshooting DNS issues, and in-depth analysis of DNS records.|
|`nslookup`|Simpler DNS lookup tool, primarily for A, AAAA, and MX records.|Basic DNS queries, quick checks of domain resolution and mail server records.|
|`host`|Streamlined DNS lookup tool with concise output.|Quick checks of A, AAAA, and MX records.|
|`dnsenum`|Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).|Discovering subdomains and gathering DNS information efficiently.|
|`fierce`|DNS reconnaissance and subdomain enumeration tool with recursive search and wildcard detection.|User-friendly interface for DNS reconnaissance, identifying subdomains and potential targets.|
|`dnsrecon`|Combines multiple DNS reconnaissance techniques and supports various output formats.|Comprehensive DNS enumeration, identifying subdomains, and gathering DNS records for further analysis.|
|`theHarvester`|OSINT tool that gathers information from various sources, including DNS records (email addresses).|Collecting email addresses, employee information, and other data associated with a domain from multiple sources.|
|`Online DNS Lookup Services`|User-friendly interfaces for performing DNS lookups.|Quick and easy DNS lookups, convenient when command-line tools are not available, checking for domain availability or basic information|
## The Domain Information Groper

The `dig` command (`Domain Information Groper`) is a versatile and powerful utility for querying DNS servers and retrieving various types of DNS records. Its flexibility and detailed and customizable output make it a go-to choice.
### Common dig Commands

| Command                         | Description                                                                                                                                                                                          |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dig domain.com`                | Performs a default A record lookup for the domain.                                                                                                                                                   |
| `dig domain.com A`              | Retrieves the IPv4 address (A record) associated with the domain.                                                                                                                                    |
| `dig domain.com AAAA`           | Retrieves the IPv6 address (AAAA record) associated with the domain.                                                                                                                                 |
| `dig domain.com MX`             | Finds the mail servers (MX records) responsible for the domain.                                                                                                                                      |
| `dig domain.com NS`             | Identifies the authoritative name servers for the domain.                                                                                                                                            |
| `dig domain.com TXT`            | Retrieves any TXT records associated with the domain.                                                                                                                                                |
| `dig domain.com CNAME`          | Retrieves the canonical name (CNAME) record for the domain.                                                                                                                                          |
| `dig domain.com SOA`            | Retrieves the start of authority (SOA) record for the domain.                                                                                                                                        |
| `dig @1.1.1.1 domain.com`       | Specifies a specific name server to query; in this case 1.1.1.1                                                                                                                                      |
| `dig +trace domain.com`         | Shows the full path of DNS resolution.                                                                                                                                                               |
| `dig -x 192.168.1.1`            | Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.                                                                     |
| `dig +short domain.com`         | Provides a short, concise answer to the query.                                                                                                                                                       |
| `dig +noall +answer domain.com` | Displays only the answer section of the query output.                                                                                                                                                |
| `dig domain.com ANY`            | Retrieves all available DNS records for the domain (Note: Many DNS servers ignore `ANY` queries to reduce load and prevent abuse, as per [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)). |

>[!Warning]
>Some servers can detect and block excessive DNS queries. Use caution and respect rate limits. Always obtain permission before performing extensive DNS reconnaissance on a target.

# Subdomains

When exploring DNS records, we've primarily focused on the main domain (e.g., `example.com`) and its associated information. However, beneath the surface of this primary domain lies a potential network of subdomains. These subdomains are extensions of the main domain, often created to organise and separate different sections or functionalities of a website. For instance, a company might use `blog.example.com` for its blog, `shop.example.com` for its online store, or `mail.example.com` for its email services.

## Why is this important for web reconnaissance?

Subdomains often host valuable information and resources that aren't directly linked from the main website. This can include:

- `Development and Staging Environments`: Companies often use subdomains to test new features or updates before deploying them to the main site. Due to relaxed security measures, these environments sometimes contain vulnerabilities or expose sensitive information.
- `Hidden Login Portals`: Subdomains might host administrative panels or other login pages that are not meant to be publicly accessible. Attackers seeking unauthorised access can find these as attractive targets.
- `Legacy Applications`: Older, forgotten web applications might reside on subdomains, potentially containing outdated software with known vulnerabilities.
- `Sensitive Information`: Subdomains can inadvertently expose confidential documents, internal data, or configuration files that could be valuable to attackers.

## Subdomain Enumeration

`Subdomain enumeration` is the process of systematically identifying and listing these subdomains. From a DNS perspective, subdomains are typically represented by `A` (or `AAAA` for IPv6) records, which map the subdomain name to its corresponding IP address. Additionally, `CNAME` records might be used to create aliases for subdomains, pointing them to other domains or subdomains. There are two main approaches to subdomain enumeration:

### 1. Active Subdomain Enumeration

This involves directly interacting with the target domain's DNS servers to uncover subdomains. One method is attempting a `DNS zone transfer`, where a misconfigured server might inadvertently leak a complete list of subdomains. However, due to tightened security measures, this is rarely successful.

A more common active technique is `brute-force enumeration`, which involves systematically testing a list of potential subdomain names against the target domain. Tools like `dnsenum`, `ffuf`, and `gobuster` can automate this process, using wordlists of common subdomain names or custom-generated lists based on specific patterns.

### 2. Passive Subdomain Enumeration

This relies on external sources of information to discover subdomains without directly querying the target's DNS servers. One valuable resource is `Certificate Transparency (CT) logs`, public repositories of SSL/TLS certificates. These certificates often include a list of associated subdomains in their Subject Alternative Name (SAN) field, providing a treasure trove of potential targets.

Another passive approach involves utilising `search engines` like Google or DuckDuckGo. By employing specialised search operators (e.g., `site:`), you can filter results to show only subdomains related to the target domain.

Additionally, various online databases and tools aggregate DNS data from multiple sources, allowing you to search for subdomains without directly interacting with the target.

Each of these methods has its strengths and weaknesses. Active enumeration offers more control and potential for comprehensive discovery but can be more detectable. Passive enumeration is stealthier but might not uncover all existing subdomains. Combining both approaches provides a more thorough and effective subdomain enumeration strategy.
# Subdomain Bruteforcing

`Subdomain Brute-Force Enumeration` is a powerful active subdomain discovery technique that leverages pre-defined lists of potential subdomain names. This approach systematically tests these names against the target domain to identify valid subdomains. By using carefully crafted wordlists, you can significantly increase the efficiency and effectiveness of your subdomain discovery efforts.

The process breaks down into four steps:

1. `Wordlist Selection`: The process begins with selecting a wordlist containing potential subdomain names. These wordlists can be:
    - `General-Purpose`: Containing a broad range of common subdomain names (e.g., `dev`, `staging`, `blog`, `mail`, `admin`, `test`). This approach is useful when you don't know the target's naming conventions.
    - `Targeted`: Focused on specific industries, technologies, or naming patterns relevant to the target. This approach is more efficient and reduces the chances of false positives.
    - `Custom`: You can create your own wordlist based on specific keywords, patterns, or intelligence gathered from other sources.
2. `Iteration and Querying`: A script or tool iterates through the wordlist, appending each word or phrase to the main domain (e.g., `example.com`) to create potential subdomain names (e.g., `dev.example.com`, `staging.example.com`).
3. `DNS Lookup`: A DNS query is performed for each potential subdomain to check if it resolves to an IP address. This is typically done using the A or AAAA record type.
4. `Filtering and Validation`: If a subdomain resolves successfully, it's added to a list of valid subdomains. Further validation steps might be taken to confirm the subdomain's existence and functionality (e.g., by attempting to access it through a web browser).

There are several tools available that excel at brute-force enumeration:

| **Herramienta**      | **Nivel de Uso Actual**            | **¿Para qué sirve MEJOR?**                                                       | **Fortalezas**                                                                                                                                           |
| -------------------- | ---------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Amass**            | ⭐⭐⭐⭐⭐ _(Líder en Pasivo)_          | Reconocimiento pasivo exhaustivo y mapeo de superficie de ataque.                | Consulta decenas de fuentes (OSINT, Certificate Transparency, APIs, WHOIS). Es el estándar de la industria respaldado por OWASP.                         |
| **Puredns**          | ⭐⭐⭐⭐⭐ _(Líder en Fuerza Bruta)_    | Fuerza bruta masiva y validación/resolución hiperrápida de subdominios.          | Utiliza `massdns` por debajo. Puede procesar listas gigantescas de diccionarios limpiando falsos positivos (wildcards) sin saturar los resolvers.        |
| **Assetfinder**      | ⭐⭐⭐⭐ _(Líder en Velocidad/Scrips)_ | Reconcimiento pasivo ultrarrápido y ligero en scripts de Bash.                   | Escrito en Go, no requiere configuración compleja. Ideal para una primera ráfaga rápida de subdominios.                                                  |
| **DNSRecon**         | ⭐⭐⭐ _(Uso Técnico Punctual)_       | Auditorías específicas de zonas DNS, transferencia de zona (`AXFR`) y registros. | Gran navaja suiza para inspeccionar la configuración DNS de un objetivo específico de forma directa.                                                     |
| **Dnsenum / Fierce** | ⭐⭐ _(Legacy / Históricos)_         | Prácticas de laboratorio, certificaciones clásicas (eJPT, OSCP tradicional).     | Fueron el estándar durante años, pero se han quedado obsoletas o muy lentas frente a volúmenes modernos de datos comparadas con herramientas en Go/Rust. |
### Ejemplo con DNSENUM

```shell
dnsenum -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --enum  inlanefreight.com
```

## What is a Zone Transfer

1. `Zone Transfer Request (AXFR)`: The secondary DNS server initiates the process by sending a zone transfer request to the primary server. This request typically uses the AXFR (Full Zone Transfer) type.
2. `SOA Record Transfer`: Upon receiving the request (and potentially authenticating the secondary server), the primary server responds by sending its Start of Authority (SOA) record. The SOA record contains vital information about the zone, including its serial number, which helps the secondary server determine if its zone data is current.
3. `DNS Records Transmission`: The primary server then transfers all the DNS records in the zone to the secondary server, one by one. This includes records like A, AAAA, MX, CNAME, NS, and others that define the domain's subdomains, mail servers, name servers, and other configurations.
4. `Zone Transfer Complete`: Once all records have been transmitted, the primary server signals the end of the zone transfer. This notification informs the secondary server that it has received a complete copy of the zone data.
5. `Acknowledgement (ACK)`: The secondary server sends an acknowledgement message to the primary server, confirming the successful receipt and processing of the zone data. This completes the zone transfer process.
## The Zone Transfer Vulnerability

While zone transfers are essential for legitimate DNS management, a misconfigured DNS server can transform this process into a significant security vulnerability. The core issue lies in the access controls governing who can initiate a zone transfer.

In the early days of the internet, allowing any client to request a zone transfer from a DNS server was common practice. This open approach simplified administration but opened a gaping security hole. It meant that anyone, including malicious actors, could ask a DNS server for a complete copy of its zone file, which contains a wealth of sensitive information.

The information gleaned from an unauthorised zone transfer can be invaluable to an attacker. It reveals a comprehensive map of the target's DNS infrastructure, including:

- `Subdomains`: A complete list of subdomains, many of which might not be linked from the main website or easily discoverable through other means. These hidden subdomains could host development servers, staging environments, administrative panels, or other sensitive resources.
- `IP Addresses`: The IP addresses associated with each subdomain, providing potential targets for further reconnaissance or attacks.
- `Name Server Records`: Details about the authoritative name servers for the domain, revealing the hosting provider and potential misconfigurations.

#### Exploiting Zone Transfers

You can use the `dig` command to request a zone transfer:

```shell
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

This command instructs `dig` to request a full zone transfer (`axfr`) from the DNS server responsible for `zonetransfer.me`. If the server is misconfigured and allows the transfer, you'll receive a complete list of DNS records for the domain, including all subdomains.
## Exercise

### After performing a zone transfer for the domain inlanefreight.htb on the target system, how many DNS records are retrieved from the target system's name server? Provide your answer as an integer, e.g, 123.

```shell
dig axfr @10.129.147.236 inlanefreight.htb
```
# Virtual Hosts
## How Virtual Hosts Work: Understanding VHosts and Subdomains

At the core of `virtual hosting` is the ability of web servers to distinguish between multiple websites or applications sharing the same IP address. This is achieved by leveraging the `HTTP Host` header, a piece of information included in every `HTTP` request sent by a web browser.

| **Criterio**        | **Subdominio**                                                                              | **Virtual Host (VHost)**                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Capa de trabajo** | **DNS** (Nivel de red / resolución de nombres).                                             | **Servidor Web** (Capa de aplicación / HTTP).                                            |
| **Dónde existe**    | Registrado en la zona DNS del servidor de nombres.                                          | Configurado en el archivo del servidor web (`sites-available`, `nginx.conf`).            |
| **Dirección IP**    | Puede apuntar a la **misma IP o a direcciones IP totalmente distintas** (otros servidores). | **Siempre apunta a la misma IP**, ya que el mismo servidor aloja múltiples sitios.       |
| **Enrutamiento**    | El cliente busca la IP asociada al nombre en el DNS.                                        | El servidor web examina la cabecera HTTP `Host:` para decidir qué carpeta/sitio mostrar. |
If a virtual host does not have a DNS record, you can still access it by modifying the `hosts` file on your local machine. The `hosts` file allows you to map a domain name to an IP address manually, bypassing DNS resolution.
Websites often have subdomains that are not public and won't appear in DNS records. These `subdomains` are only accessible internally or through specific configurations. `VHost fuzzing` is a technique to discover public and non-public `subdomains` and `VHosts` by testing various hostnames against a known IP address.

Virtual hosts can also be configured to use different domains, not just subdomains. For example:

```apache
# Example of name-based virtual host configuration in Apache 
<VirtualHost *:80> 
ServerName www.example1.com 
DocumentRoot /var/www/example1 
</VirtualHost> 

<VirtualHost *:80> 
ServerName www.example2.org 
DocumentRoot /var/www/example2 
</VirtualHost> 

<VirtualHost *:80> 
ServerName www.another-example.net 
DocumentRoot /var/www/another-example 
</VirtualHost>
```

![[Pasted image 20260809234940.png]]
### Types of Virtual Hosting

There are three primary types of virtual hosting, each with its advantages and drawbacks:

1. `Name-Based Virtual Hosting`: This method relies solely on the `HTTP Host header` to distinguish between websites. It is the most common and flexible method, as it doesn't require multiple IP addresses. It’s cost-effective, easy to set up, and supports most modern web servers. However, it requires the web server to support name-based `virtual hosting` and can have limitations with certain protocols like `SSL/TLS`.
2. `IP-Based Virtual Hosting`: This type of hosting assigns a unique IP address to each website hosted on the server. The server determines which website to serve based on the IP address to which the request was sent. It doesn't rely on the `Host header`, can be used with any protocol, and offers better isolation between websites. Still, it requires multiple IP addresses, which can be expensive and less scalable.
3. `Port-Based Virtual Hosting`: Different websites are associated with different ports on the same IP address. For example, one website might be accessible on port 80, while another is on port 8080. `Port-based virtual hosting` can be used when IP addresses are limited, but it’s not as common or user-friendly as `name-based virtual hosting` and might require users to specify the port number in the URL.

|Tool|Description|Features|
|---|---|---|
|[gobuster](https://github.com/OJ/gobuster)|A multi-purpose tool often used for directory/file brute-forcing, but also effective for virtual host discovery.|Fast, supports multiple HTTP methods, can use custom wordlists.|
|[Feroxbuster](https://github.com/epi052/feroxbuster)|Similar to Gobuster, but with a Rust-based implementation, known for its speed and flexibility.|Supports recursion, wildcard discovery, and various filters.|
|[ffuf](https://github.com/ffuf/ffuf)|Another fast web fuzzer that can be used for virtual host discovery by fuzzing the `Host` header.|Customizable wordlist input and filtering options.|
## Exercise
### Brute-force vhosts on the target system. What is the full subdomain that is prefixed with "web"? Answer using the full domain, e.g. "x.inlanefreight.htb"

```
gobuster vhost -u http://inlanefreight.htb:31453 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -t 10 --append-domain
```

>[!Note] Note: The `append-domian` flag is necessary in some versions of gobuster

> [!Tip] Tip
> To gather specific subdomain if knowing a substring
> ```
> grep -h ^web /usr/share/wordlists/seclists/Discovery/DNS/* | sort -u > web.txt
> ```
> and then
> ```
> gobuster vhost -u http://inlanefreight.htb:31453 -w web.txt --append-domain
> ```

# Certificate Transparency Logs
## What are Certificate Transparency Logs?

`Certificate Transparency` (`CT`) logs are public, append-only ledgers that record the issuance of SSL/TLS certificates. Whenever a Certificate Authority (CA) issues a new certificate, it must submit it to multiple CT logs. Independent organisations maintain these logs and are open for anyone to inspect.

Think of CT logs as a `global registry of certificates`. They provide a transparent and verifiable record of every SSL/TLS certificate issued for a website. This transparency serves several crucial purposes:
- `Early Detection of Rogue Certificates`: By monitoring CT logs, security researchers and website owners can quickly identify suspicious or misissued certificates. A rogue certificate is an unauthorized or fraudulent digital certificate issued by a trusted certificate authority. Detecting these early allows for swift action to revoke the certificates before they can be used for malicious purposes.
- `Accountability for Certificate Authorities`: CT logs hold CAs accountable for their issuance practices. If a CA issues a certificate that violates the rules or standards, it will be publicly visible in the logs, leading to potential sanctions or loss of trust.
- `Strengthening the Web PKI (Public Key Infrastructure)`: The Web PKI is the trust system underpinning secure online communication. CT logs help to enhance the security and integrity of the Web PKI by providing a mechanism for public oversight and verification of certificates.

In essence, CT logs provide a reliable and efficient way to discover subdomains without the need for exhaustive brute-forcing or relying on the completeness of wordlists.
## Searching CT Logs

There are two popular options for searching CT logs:

|Tool|Key Features|Use Cases|Pros|Cons|
|---|---|---|---|---|
|[crt.sh](https://crt.sh/)|User-friendly web interface, simple search by domain, displays certificate details, SAN entries.|Quick and easy searches, identifying subdomains, checking certificate issuance history.|Free, easy to use, no registration required.|Limited filtering and analysis options.|
|[Censys](https://search.censys.io/)|Powerful search engine for internet-connected devices, advanced filtering by domain, IP, certificate attributes.|In-depth analysis of certificates, identifying misconfigurations, finding related certificates and hosts.|Extensive data and filtering options, API access.|Requires registration (free tier available).|
### crt.sh lookup

While `crt.sh` offers a convenient web interface, you can also leverage its API for automated searches directly from your terminal. Let's see how to find all 'dev' subdomains on `facebook.com` using `curl` and `jq`:

```shell
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[] | select(.name_value | contains("dev")) | .name_value' | sort -u 

*.dev.facebook.com 
*.newdev.facebook.com
.
.
.secure.dev.facebook.com
```

# Fingerprinting

Fingerprinting focuses on extracting technical details about the technologies powering a website or web application. Similar to how a fingerprint uniquely identifies a person, the digital signatures of web servers, operating systems, and software components can reveal critical information about a target's infrastructure and potential security weaknesses. This knowledge empowers attackers to tailor attacks and exploit vulnerabilities specific to the identified technologies.

Fingerprinting serves as a cornerstone of web reconnaissance for several reasons:

- `Targeted Attacks`: By knowing the specific technologies in use, attackers can focus their efforts on exploits and vulnerabilities that are known to affect those systems. This significantly increases the chances of a successful compromise.
- `Identifying Misconfigurations`: Fingerprinting can expose misconfigured or outdated software, default settings, or other weaknesses that might not be apparent through other reconnaissance methods.
- `Prioritising Targets`: When faced with multiple potential targets, fingerprinting helps prioritise efforts by identifying systems more likely to be vulnerable or hold valuable information.
- `Building a Comprehensive Profile`: Combining fingerprint data with other reconnaissance findings creates a holistic view of the target's infrastructure, aiding in understanding its overall security posture and potential attack vectors.

|Tool|Description|Features|
|---|---|---|
|`Wappalyzer`|Browser extension and online service for website technology profiling.|Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more.|
|`BuiltWith`|Web technology profiler that provides detailed reports on a website's technology stack.|Offers both free and paid plans with varying levels of detail.|
|`WhatWeb`|Command-line tool for website fingerprinting.|Uses a vast database of signatures to identify various web technologies.|
|`Nmap`|Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting.|Can be used with scripts (NSE) to perform more specialised fingerprinting.|
|`Netcraft`|Offers a range of web security services, including website fingerprinting and security reporting.|Provides detailed reports on a website's technology, hosting provider, and security posture.|
|`wafw00f`|Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).|Helps determine if a WAF is present and, if so, its type and configuration.|
### Banner Grabbing

Our first step is to gather information directly from the web server itself. We can do this using the `curl` command with the `-I` flag (or `--head`) to fetch only the HTTP headers, not the entire page content.

```shell
curl -I inlanefreight.com
```

The output will include the server banner, revealing the web server software and version number

```shell
curl -I https://inlanefreight.com
```


### Wafw00f

`Web Application Firewalls` (`WAFs`) are security solutions designed to protect web applications from various attacks. Before proceeding with further fingerprinting, it's crucial to determine if `inlanefreight.com` employs a WAF, as it could interfere with our probes or potentially block our requests.

To detect the presence of a WAF, we'll use the `wafw00f` tool. To install `wafw00f`, you can use pip3:

 ```shell
 pip3 install git+https://github.com/EnableSecurity/wafw00f
 ```

Once it's installed, pass the domain you want to check as an argument to the tool:
The `wafw00f` scan on `inlanefreight.com` reveals that the website is protected by the `Wordfence Web Application Firewall` (`WAF`), developed by Defiant.

### Nikto

`Nikto` is a powerful open-source web server scanner. In addition to its primary function as a vulnerability assessment tool, `Nikto's` fingerprinting capabilities provide insights into a website's technology stack.

If you need to install it, you can run the following commands:

```shell
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto 
cd nikto/program 
chmod +x ./nikto.pl
```

To scan `inlanefreight.com` using `Nikto`, only running the fingerprinting modules, execute the following command:
```shell
nikto -h inlanefreight.com -Tuning b
```

The `-h` flag specifies the target host. The `-Tuning b` flag tells `Nikto` to only run the Software Identification modules.

`Nikto` will then initiate a series of tests, attempting to identify outdated software, insecure files or configurations, and other potential security risks.
## Exercise

Add the IP to `/etc/hosts`

```shell
nikto -h http://app.inlanefreight.local -Tuning b
```

# Crawling

`Crawling`, often called `spidering`, is the `automated process of systematically browsing the World Wide Web`. Similar to how a spider navigates its web, a web crawler follows links from one page to another, collecting information. These crawlers are essentially bots that use pre-defined algorithms to discover and index web pages, making them accessible through search engines or for other purposes like data analysis and web reconnaissance.

## How Web Crawlers Work

The basic operation of a web crawler is straightforward yet powerful. It starts with a seed URL, which is the initial web page to crawl. The crawler fetches this page, parses its content, and extracts all its links. It then adds these links to a queue and crawls them, repeating the process iteratively. Depending on its scope and configuration, the crawler can explore an entire website or even a vast portion of the web.

1. `Homepage`: You start with the homepage containing `link1`, `link2`, and `link3`.
2. `Visiting link1`: Visiting `link1` shows the homepage, `link2`, and also `link4` and `link5`.
3. `Continuing the Crawl`: The crawler continues to follow these links systematically, gathering all accessible pages and their links.
### Breadth-First Crawling

![Flowchart showing a Seed URL leading to Page 1, which branches to Page 2 and Page 3. Page 2 connects to Page 4 and Page 5, while Page 3 connects to Page 6 and Page 7.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/144/ig_crawling_1.png)

`Breadth-first crawling` prioritizes exploring a website's width before going deep. It starts by crawling all the links on the seed page, then moves on to the links on those pages, and so on. This is useful for getting a broad overview of a website's structure and content.

### Depth-First Crawling

![Flowchart showing a Seed URL leading to Page 1, then to Page 2. Page 2 connects to Page 3, which branches to Page 4 and Page 5.](https://mermaid.ink/svg/pako:eNo9zz0PgjAQBuC_0twsg18LgwlfGyYG4uQ5VHoC0RZS2sEQ_rsnTezU98mlvXeGZlAEMbRWjp0oKzSTf4RQEylxrUo0gk9yu8iWxPaOhoxCk4goOok06I41XSELsGfIVsgDHBjyFYoAR4YivCEEGtiAJqtlr3iZ-fclgutIE0LMVyXtCwHNwnPSu6H-mAZiZz1twA6-7SB-yvfEyY9KOsp7ySX0X0n1brDn0HWtvHwB2SFOww)

In contrast, `depth-first crawling` prioritizes depth over breadth. It follows a single path of links as far as possible before backtracking and exploring other paths. This can be useful for finding specific content or reaching deep into a website's structure.

The choice of strategy depends on the specific goals of the crawling process.
## Extracting Valuable Information

Crawlers can extract a diverse array of data, each serving a specific purpose in the reconnaissance process:

- `Links (Internal and External)`: These are the fundamental building blocks of the web, connecting pages within a website (`internal links`) and to other websites (`external links`). Crawlers meticulously collect these links, allowing you to map out a website's structure, discover hidden pages, and identify relationships with external resources.
- `Comments`: Comments sections on blogs, forums, or other interactive pages can be a goldmine of information. Users often inadvertently reveal sensitive details, internal processes, or hints of vulnerabilities in their comments.
- `Metadata`: Metadata refers to `data about data`. In the context of web pages, it includes information like page titles, descriptions, keywords, author names, and dates. This metadata can provide valuable context about a page's content, purpose, and relevance to your reconnaissance goals.
- `Sensitive Files`: Web crawlers can be configured to actively search for sensitive files that might be inadvertently exposed on a website. This includes `backup files` (e.g., `.bak`, `.old`), `configuration files` (e.g., `web.config`, `settings.php`), `log files` (e.g., `error_log`, `access_log`), and other files containing passwords, `API keys`, or other confidential information. Carefully examining the extracted files, especially backup and configuration files, can reveal a trove of sensitive information, such as `database credentials`, `encryption keys`, or even source code snippets.
# robots.txt

Imagine you're a guest at a grand house party. While you're free to mingle and explore, there might be certain rooms marked "Private" that you're expected to avoid. This is akin to how `robots.txt` functions in the world of web crawling. It acts as a virtual "`etiquette guide`" for bots, outlining which areas of a website they are allowed to access and which are off-limits.

## What is robots.txt?

Technically, `robots.txt` is a simple text file placed in the root directory of a website (e.g., `www.example.com/robots.txt`). It adheres to the Robots Exclusion Standard, guidelines for how web crawlers should behave when visiting a website. This file contains instructions in the form of "directives" that tell bots which parts of the website they can and cannot crawl.

### Understanding robots.txt Structure

The robots.txt file is a plain text document that lives in the root directory of a website. It follows a straightforward structure, with each set of instructions, or "record," separated by a blank line. Each record consists of two main components:

1. `User-agent`: This line specifies which crawler or bot the following rules apply to. A wildcard (`*`) indicates that the rules apply to all bots. Specific user agents can also be targeted, such as "Googlebot" (Google's crawler) or "Bingbot" (Microsoft's crawler).
2. `Directives`: These lines provide specific instructions to the identified user-agent.

Common directives include:

|Directive|Description|Example|
|---|---|---|
|`Disallow`|Specifies paths or patterns that the bot should not crawl.|`Disallow: /admin/` (disallow access to the admin directory)|
|`Allow`|Explicitly permits the bot to crawl specific paths or patterns, even if they fall under a broader `Disallow` rule.|`Allow: /public/` (allow access to the public directory)|
|`Crawl-delay`|Sets a delay (in seconds) between successive requests from the bot to avoid overloading the server.|`Crawl-delay: 10` (10-second delay between requests)|
|`Sitemap`|Provides the URL to an XML sitemap for more efficient crawling.|`Sitemap: https://www.example.com/sitemap.xml`|
### Why Respect robots.txt?

While robots.txt is not strictly enforceable (a rogue bot could still ignore it), most legitimate web crawlers and search engine bots will respect its directives. This is important for several reasons:

- `Avoiding Overburdening Servers`: By limiting crawler access to certain areas, website owners can prevent excessive traffic that could slow down or even crash their servers.
- `Protecting Sensitive Information`: Robots.txt can shield private or confidential information from being indexed by search engines.
- `Legal and Ethical Compliance`: In some cases, ignoring robots.txt directives could be considered a violation of a website's terms of service or even a legal issue, especially if it involves accessing copyrighted or private data.
### Analyzing robots.txt

Here's an example of a robots.txt file:

  User-agent: * Disallow: /admin/ Disallow: /private/ Allow: /public/  User-agent: Googlebot Crawl-delay: 10  Sitemap: https://www.example.com/sitemap.xml

This file contains the following directives:

- All user agents are disallowed from accessing the `/admin/` and `/private/` directories.
- All user agents are allowed to access the `/public/` directory.
- The `Googlebot` (Google's web crawler) is specifically instructed to wait 10 seconds between requests.
- The sitemap, located at `https://www.example.com/sitemap.xml`, is provided for easier crawling and indexing.

# FALTA COPIAR Y PEGAR RECONSPIDER

# LAB

> [!info]- LAB
![[Ciberseguridad/HTB Apuntes/CPTS/LABS/Information Gathering - Web Edition|Information Gathering - Web Edition]]


