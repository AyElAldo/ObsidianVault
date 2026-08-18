#HTB #CWES #web #cybersecurity 

Web fuzzing is a critical technique in web application security to identify vulnerabilities by testing various inputs. It involves automated testing of web applications by providing unexpected or random data to detect potential flaws that attackers could exploit.
## Fuzzing vs Brute Froce
- Fuzzing casts a wider net. It involves feeding the web application with unexpected inputs, including malformed data, invalid characters, and nonsensical combinations. The goal is to see how the application reacts to these strange inputs and uncover potential vulnerabilities in handling unexpected data. Fuzzing tools often leverage wordlists containing common patterns, mutations of existing parameters, or even random character sequences to generate a diverse set of payloads.
- Brute-forcing, on the other hand, is a more targeted approach. It focuses on systematically trying out many possibilities for a specific value, such as a password or an ID number. Brute-forcing tools typically rely on predefined lists or dictionaries (like password dictionaries) to guess the correct value through trial and error.

>[!Example] Example
>Here's an analogy to illustrate the difference: Imagine you're trying to open a locked door. Fuzzing would be like throwing everything you can find at the door - keys, screwdrivers, even a rubber duck - to see if anything unlocks it. Brute-forcing would be like trying every combination on a key ring until you find the one that opens the door.
### Use of fuzzing
- `Uncovering Hidden Vulnerabilities`: Fuzzing can uncover vulnerabilities that traditional security testing methods might miss. By bombarding a web application with unexpected and invalid inputs, fuzzing can trigger unexpected behaviors that reveal hidden flaws in the code.
- `Automating Security Testing`: Fuzzing automates generating and sending test inputs, saving valuable time and resources. This allows security teams to focus on analyzing results and addressing the vulnerabilities found.
- `Simulating Real-World Attacks`: Fuzzers can mimic attackers' techniques, helping you identify weaknesses before malicious actors exploit them. This proactive approach can significantly reduce the risk of a successful attack.
- `Strengthening Input Validation`: Fuzzing helps identify weaknesses in input validation mechanisms, which are crucial for preventing common vulnerabilities like `SQL injection` and `cross-site scripting` (`XSS`).
- `Improving Code Quality`: Fuzzing improves overall code quality by uncovering bugs and errors. Developers can use the feedback from fuzzing to write more robust and secure code.
- `Continuous Security`: Fuzzing can be integrated into the `software development lifecycle` (`SDLC`) as part of `continuous integration and continuous deployment` (`CI/CD`) pipelines, ensuring that security testing is performed regularly and vulnerabilities are caught early in the development process.
## Essential Concepts
Before we dive into the practical aspects of web fuzzing, it's important to understand some key concepts:

|Concept|Description|Example|
|---|---|---|
|`Wordlist`|A dictionary or list of words, phrases, file names, directory names, or parameter values used as input during fuzzing.|Generic: `admin`, `login`, `password`, `backup`, `config`  <br>Application-specific: `productID`, `addToCart`, `checkout`|
|`Payload`|The actual data sent to the web application during fuzzing. Can be a simple string, numerical value, or complex data structure.|`' OR 1=1 --` (for SQL injection)|
|`Response Analysis`|Examining the web application's responses (e.g., response codes, error messages) to the fuzzer's payloads to identify anomalies that might indicate vulnerabilities.|Normal: 200 OK  <br>Error (potential SQLi): 500 Internal Server Error with a database error message|
|`Fuzzer`|A software tool that automates generating and sending payloads to a web application and analyzing the responses.|`ffuf`, `wfuzz`, `Burp Suite Intruder`|
|`False Positive`|A result that is incorrectly identified as a vulnerability by the fuzzer.|A 404 Not Found error for a non-existent directory.|
|`False Negative`|A vulnerability that exists in the web application but is not detected by the fuzzer.|A subtle logic flaw in a payment processing function.|
|`Fuzzing Scope`|The specific parts of the web application that you are targeting with your fuzzing efforts.|Only fuzzing the login page or focusing on a particular API endpoint.|
# Setting Up (Go, Python and PIPX)

```shell
sudo apt install -y golang
sudo apt install -y python3 python3-pip
sudo apt install pipx
pipx ensurepath
sudo pipx ensurepath --global

go version
python3 --version
```
## FFUF

`FFUF` (`Fuzz Faster U Fool`) is a fast web fuzzer written in Go. It excels at quickly enumerating directories, files, and parameters within web applications. Its flexibility, speed, and ease of use make it a favorite among security professionals and enthusiasts.

```go
go install github.com/ffuf/ffuf/v2@latest
```
### Use Cases

|Use Case|Description|
|---|---|
|`Directory and File Enumeration`|Quickly identify hidden directories and files on a web server.|
|`Parameter Discovery`|Find and test parameters within web applications.|
|`Brute-Force Attack`|Perform brute-force attacks to discover login credentials or other sensitive information.|
## Gobuster

`Gobuster` is another popular web directory and file fuzzer. It's known for its speed and simplicity, making it a great choice for beginners and experienced users alike.

You can install `GoBuster` using the following command:
```go
go install github.com/OJ/gobuster/v3@latest
```
### Use Cases

|Use Case|Description|
|---|---|
|`Content Discovery`|Quickly scan and find hidden web content such as directories, files, and virtual hosts.|
|`DNS Subdomain Enumeration`|Identify subdomains of a target domain.|
|`WordPress Content Detection`|Use specific wordlists to find WordPress-related content.|
## FeroxBuster

`FeroxBuster` is a fast, recursive content discovery tool written in Rust. It's designed for brute-force discovery of unlinked content in web applications, making it particularly useful for identifying hidden directories and files. It's more of a "forced browsing" tool than a fuzzer like `ffuf`.

To install `FeroxBuster`, you can use the following command:

```shell
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | sudo bash -s $HOME/.local/bin
```
### Use Cases

|Use Case|Description|
|---|---|
|`Recursive Scanning`|Perform recursive scans to discover nested directories and files.|
|`Unlinked Content Discovery`|Identify content that is not linked within the web application.|
|`High-Performance Scans`|Benefit from Rust's performance to conduct high-speed content discovery.|
## wfuzz/wenum

`wenum` is an actively maintained fork of `wfuzz`, a highly versatile and powerful command-line fuzzing tool known for its flexibility and customization options. It's particularly well-suited for parameter fuzzing, allowing you to test a wide range of input values against web applications and uncover potential vulnerabilities in how they process those parameters.

If you are using a penetration testing Linux distribution like PwnBox or Kali, `wfuzz` may already be pre-installed, allowing you to use it right away if desired. However, there are currently complications when installing `wfuzz`, so you can substitute it with `wenum` instead. The commands are interchangeable, and they follow the same syntax, so you can simply replace `wenum` commands with `wfuzz` if necessary.

The following commands will use `pipx`, a tool for installing and managing Python applications in isolated environments, to install `wenum`. This ensures a clean and consistent environment for `wenum`, preventing any possible package conflicts:

```shell
pipx install git+https://github.com/WebFuzzForge/wenum 
pipx runpip wenum install setuptools
```
### Use Cases

|Use Case|Description|
|---|---|
|`Directory and File Enumeration`|Quickly identify hidden directories and files on a web server.|
|`Parameter Discovery`|Find and test parameters within web applications.|
|`Brute-Force Attack`|Perform brute-force attacks to discover login credentials or other sensitive information.|
## Wordlists
The tools we've discussed – `ffuf`, `wfuzz`, etc – don't have built-in wordlists, but they are designed to work seamlessly with external wordlist files. This flexibility allows you to use pre-existing wordlists or create your own to tailor your fuzzing efforts to specific targets and scenarios.

One of the most comprehensive and widely-used collections of wordlists is `SecLists`. This open-source project on GitHub ([https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)) provides a vast repository of wordlists for various security testing purposes, including directory and file fuzzing.

`SecLists` contains wordlists for:

- Common directory and file names
- Backup files
- Configuration files
- Vulnerable scripts
- And much more

The most commonly used wordlists for fuzzing web directories and files from `SecLists` are:

- `Discovery/Web-Content/common.txt`: This general-purpose wordlist contains a broad range of common directory and file names on web servers. It's an excellent starting point for fuzzing and often yields valuable results.
- `Discovery/Web-Content/directory-list-2.3-medium.txt`: This is a more extensive wordlist specifically focused on directory names. It's a good choice when you need a deeper dive into potential directories.
- `Discovery/Web-Content/raft-large-directories.txt`: This wordlist boasts a massive collection of directory names compiled from various sources. It's a valuable resource for thorough fuzzing campaigns.
- `Discovery/Web-Content/big.txt`: As the name suggests, this is a massive wordlist containing both directory and file names. It's useful when you want to cast a wide net and explore all possibilities.
# Actually Fuzzing
### ffuf

We will use `ffuf` for this fuzzing task. Here's how `ffuf` generally works:

1. `Wordlist`: You provide `ffuf` with a wordlist containing potential directory or file names.
2. `URL with FUZZ keyword`: You construct a URL with the `FUZZ` keyword as a placeholder where the wordlist entries will be inserted.
3. `Requests`: `ffuf` iterates through the wordlist, replacing the `FUZZ` keyword in the URL with each entry and sending HTTP requests to the target web server.
4. `Response Analysis`: `ffuf` analyzes the server's responses (status codes, content length, etc.) and filters the results based on your criteria.
## Directory fuzzing

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://IP:PORT/FUZZ
```

- `-w` (wordlist): Specifies the path to the wordlist we want to use. In this case, we're using a medium-sized directory list from SecLists.
- `-u` (URL): Specifies the base URL to fuzz. The `FUZZ` keyword acts as a placeholder where the fuzzer will insert words from the wordlist.
## File Fuzzing

While directory fuzzing focuses on finding folders, file fuzzing dives deeper into discovering specific files within those directories or even in the root of the web application. Web applications use various file types to serve content and perform different functions. Some common file extensions include:

- `.php`: Files containing PHP code, a popular server-side scripting language.
- `.html`: Files that define the structure and content of web pages.
- `.txt`: Plain text files, often storing simple information or logs.
- `.bak`: Backup files are created to preserve previous versions of files in case of errors or modifications.
- `.js`: Files containing JavaScript code add interactivity and dynamic functionality to web pages.

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://IP:PORT/w2ksvrus/FUZZ -e .php,.html,.txt,.bak,.js -v
```
## Exercise

```go
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://154.57.164.77:30463/webfuzzing_hidden_path/flag/FUZZ
```

```shell
This is the flag directory
```

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.77:30463/webfuzzing_hidden_path/flag/FUZZ -t 350 -e .php,.html,.txt,.bak,.js
```
# Recursive Fuzzing 
1. `Initial Fuzzing`:
    - The fuzzing process begins with the top-level directory, typically the web root (`/`).
    - The fuzzer starts sending requests based on the provided wordlist containing the potential directory and file names.
    - The fuzzer analyzes server responses, looking for successful results (e.g., HTTP 200 OK) that indicate the existence of a directory.
2. `Directory Discovery and Expansion`:
    - When a valid directory is found, the fuzzer doesn't just note it down. It creates a new branch for that directory, essentially appending the directory name to the base URL.
    - For example, if the fuzzer finds a directory named `admin` at the root level, it will create a new branch like `http://localhost/admin/`.
    - This new branch becomes the starting point for a fresh fuzzing process. The fuzzer will again iterate through the wordlist, appending each entry to the new branch's URL (e.g., `http://localhost/admin/FUZZ`).
3. `Iterative Depth`:
    - The process repeats for each discovered directory, creating further branches and expanding the fuzzing scope deeper into the web application's structure.
    - This continues until a specified depth limit is reached (e.g., a maximum of three levels deep) or no more valid directories are found.

```go
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -ic -v -u http://IP:PORT/FUZZ -e .html -recursion
```
### Be Responsible

While recursive fuzzing is a powerful technique, it can also be resource-intensive, especially on large web applications. Excessive requests can overwhelm the target server, potentially causing performance issues or triggering security mechanisms.

To mitigate these risks, `ffuf` provides options for fine-tuning the recursive fuzzing process:

- `-recursion-depth`: This flag allows you to set a maximum depth for recursive exploration. For example, `-recursion-depth 2` limits fuzzing to two levels deep (the starting directory and its immediate subdirectories).
- `-rate`: You can control the rate at which `ffuf` sends requests per second, preventing the server from being overloaded.
- `-timeout`: This option sets the timeout for individual requests, helping to prevent the fuzzer from hanging on unresponsive targets.
## Exercise

```go
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://154.57.164.83:30906/recursive_fuzz/FUZZ -t 400 -recursion -recursion-depth 2 -e .html -c
```

```go
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://154.57.164.83:30906/recursive_fuzz/level1/level2/level3/FUZZ -t 400 -e .html,.php,.txt,.bak,.js
```
# Parameter and Value Fuzzing

Building upon the discovery of hidden directories and files, we now delve into parameter and value fuzzing. This technique focuses on manipulating the parameters and their values within web requests to uncover vulnerabilities in how the application processes input.
Parameters are the messengers of the web, carrying vital information between your browser and the server that hosts the web application. They're like variables in programming, holding specific values that influence how the application behaves.
## GET Parameters: Openly Sharing Information

You'll often spot `GET` parameters right in the URL, following a question mark (`?`). Multiple parameters are strung together using ampersands (`&`). For example:

```http
https://example.com/search?query=fuzzing&category=security
```
In this URL:

- `query` is a parameter with the value "fuzzing"
- `category` is another parameter with the value "security"

`GET` parameters are like postcards – their information is visible to anyone who glances at the URL. They're primarily used for actions that don't change the server's state, like searching or filtering
## POST Parameters: Behind-the-Scenes Communication

While `GET` parameters are like open postcards, POST parameters are more like sealed envelopes, carrying their information discreetly within the body of the HTTP request. They are not visible directly in the URL, making them the preferred method for transmitting sensitive data like login credentials, personal information, or financial details.

When you submit a form or interact with a web page that uses POST requests, the following happens:

1. `Data Collection`: The information entered into the form fields is gathered and prepared for transmission.
2. `Encoding`: This data is encoded into a specific format, typically `application/x-www-form-urlencoded` or `multipart/form-data`:
    - `application/x-www-form-urlencoded`: This format encodes the data as key-value pairs separated by ampersands (`&`), similar to GET parameters but placed within the request body instead of the URL.
    - `multipart/form-data`: This format is used when submitting files along with other data. It divides the request body into multiple parts, each containing a specific piece of data or a file.
3. `HTTP Request`: The encoded data is placed within the body of an HTTP POST request and sent to the web server.
4. `Server-Side Processing`: The server receives the POST request, decodes the data, and processes it according to the application's logic.

Here's a simplified example of how a POST request might look when submitting a login form:
```http
POST /login HTTP/1.1 
Host: example.com 
Content-Type: application/x-www-form-urlencoded 

username=your_username&password=your_password
```
- `POST`: Indicates the HTTP method (POST).
- `/login`: Specifies the URL path where the form data is sent.
- `Content-Type`: Specifies how the data in the request body is encoded (`application/x-www-form-urlencoded` in this case).
- `Request Body`: Contains the encoded form data as key-value pairs (`username` and `password`).
## Why Parameters Matter for Fuzzing

Parameters are the gateways through which you can interact with a web application. By manipulating their values, you can test how the application responds to different inputs, potentially uncovering vulnerabilities. For instance:

- Altering a product ID in a shopping cart URL could reveal pricing errors or unauthorized access to other users' orders.
- Modifying a hidden parameter in a request might unlock hidden features or administrative functions.
- Injecting malicious code into a search query could expose vulnerabilities like Cross-Site Scripting (XSS) or SQL Injection (SQLi).

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.82:30933/get.php\?x\=FUZZ
```
### POST

Fuzzing POST parameters requires a slightly different approach than fuzzing GET parameters. Instead of appending values directly to the URL, we'll use `ffuf` to send the payloads within the request body. This enables us to test how the application handles data submitted through forms or other POST mechanisms.

Our target application also features a POST parameter named "`y`" within the `post.php` script. Let's probe it with `curl` to see its default behavior:

```shell
AyElAldo@htb[/htb]$ curl -d "" http://IP:PORT/post.php Invalid parameter value y:
```

The `-d` flag instructs `curl` to make a POST request with an empty body. The response tells us that the parameter `y` is expected but not provided.

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.82:30933/post.php -X POST -d "y=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -mc 200 
```

```shell
curl http://154.57.164.82:30933/post.php -d "y=SUNWmc"  
```
# Virtual Host and Subdomain Fuzzing

---

Both virtual hosting (vhosting) and subdomains play pivotal roles in organizing and managing web content.

Virtual hosting enables multiple websites or domains to be served from a single server or IP address. Each vhost is associated with a unique domain name or hostname. When a client sends an HTTP request, the web server examines the `Host` header to determine which vhost's content to deliver. This facilitates efficient resource utilization and cost reduction, as multiple websites can share the same server infrastructure.

Subdomains, on the other hand, are extensions of a primary domain name, creating a hierarchical structure within the domain. They are used to organize different sections or services within a website. For example, `blog.example.com` and `shop.example.com` are subdomains of the main domain `example.com`. Unlike vhosts, subdomains are resolved to specific IP addresses through DNS (Domain Name System) records.

| Feature        | Virtual Hosts                                                            | Subdomains                                                                  |
| -------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| Identification | Identified by the `Host` header in HTTP requests.                        | Identified by DNS records, pointing to specific IP addresses.               |
| Purpose        | Primarily used to host multiple websites on a single server.             | Used to organize different sections or services within a website.           |
| Security Risks | Misconfigured vhosts can expose internal applications or sensitive data. | Subdomain takeover vulnerabilities can occur if DNS records are mismanaged. |
## Gobuster

`Gobuster` is a versatile command-line tool renowned for its directory/file and DNS busting capabilities. It systematically probes target web servers or domains to uncover hidden directories, files, and subdomains, making it a valuable asset in security assessments and penetration testing.

`Gobuster's` flexibility extends to fuzzing for various types of content:

- `Directories`: Discover hidden directories on a web server.
- `Files`: Identify files with specific extensions (e.g., `.php`, `.txt`, `.bak`).
- `Subdomains`: Enumerate subdomains of a given domain.
- `Virtual Hosts (vhosts)`: Uncover hidden virtual hosts by manipulating the `Host` header.
### Gobuster VHost Fuzzing

While `gobuster` is primarily known for directory and file enumeration, its capabilities extend to virtual host (vhost) discovery, making it a valuable tool in assessing the security posture of a web server.

Let's dissect the `Gobuster` vhost fuzzing command:

```shell
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain
```
Using `ffuf`:
```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.82:31884 -H "Host: FUZZ.inlanefreight.htb"
```
### Gobuster Subdomain Fuzzing

While often associated with vhost and directory discovery, `Gobuster` also excels at subdomain enumeration, a crucial step in mapping the attack surface of a target domain. By systematically testing variations of potential subdomain names, `Gobuster` can uncover hidden or forgotten subdomains that might host valuable information or vulnerabilities.

Let's break down the `Gobuster` subdomain fuzzing command:
```shell
gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

- `gobuster dns`: Activates `Gobuster's` DNS fuzzing mode, directing it to focus on discovering subdomains.
- `-d inlanefreight.com`: Specifies the target domain (e.g., `inlanefreight.com`) for which you want to discover subdomains.
- `-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`: This points to the wordlist file that `Gobuster` will use to generate potential subdomain names. In this example, we're using a wordlist containing the top 5000 most common subdomains.
# Filtering Fuzzing Output

Web fuzzing tools like `gobuster`, `ffuf`, and `wfuzz` are designed to perform comprehensive scans, often generating a vast amount of data. Sifting through this output to identify the most relevant findings can be a daunting task. However, these tools offer powerful filtering mechanisms to streamline your analysis and focus on the results that matter most.

### Gobuster

`Gobuster` offers various filtering options depending on the module being run, to help you focus on specific responses and streamline your analysis. There is a small caveat, the `-s` and `-b` options are only available in the `dir` fuzzing mode.

|Flag|Description|Example Scenario|
|---|---|---|
|`-s` (include)|Include only responses with the specified status codes (comma-separated).|You're looking for redirects, so you filter for codes `301,302,307`|
|`-b` (exclude)|Exclude responses with the specified status codes (comma-separated).|The server returns many 404 errors. Exclude them with `-b 404`|
|`--exclude-length`|Exclude responses with specific content lengths (comma-separated, supports ranges).|You're not interested in 0-byte or 404-byte responses, so use `--exclude-length 0,404`|

By strategically combining these filtering options, you can tailor `Gobuster's` output to your specific needs and focus on the most relevant results for your security assessments.

``` shell
# Find directories with status codes 200 or 301, but exclude responses with a size of 0 (empty responses) 
gobuster dir -u http://example.com/ -w wordlist.txt -s 200,301 --exclude-length 0
```
## FFUF

`FFUF` offers a highly customizable filtering system, enabling precise control over the displayed output. This allows you to efficiently sift through potentially large amounts of data and focus on the most relevant findings. `FFUF's` filtering options are categorized into multiple types, each serving a specific purpose in refining your results.

|Flag|Description|Example Scenario|
|---|---|---|
|`-mc` (match code)|Include only responses that match the specified status codes. You can provide a single code, multiple codes separated by commas, or ranges of codes separated by hyphens (e.g., `200,204,301`, `400-499`). The default behavior is to match codes 200-299, 301, 302, 307, 401, 403, 405, and 500.|After fuzzing, you notice many 302 (Found) redirects, but you're primarily interested in 200 (OK) responses. Use `-mc 200` to isolate these.|
|`-fc` (filter code)|Exclude responses that match the specified status codes, using the same format as `-mc`. This is useful for removing common error codes like 404 Not Found.|A scan returns many 404 errors. Use `-fc 404` to remove them from the output.|
|`-fs` (filter size)|Exclude responses with a specific size or range of sizes. You can specify single sizes or ranges using hyphens (e.g., `-fs 0` for empty responses, `-fs 100-200` for responses between 100 and 200 bytes).|You suspect the interesting responses will be larger than 1KB. Use `-fs 0-1023` to filter out smaller responses.|
|`-ms` (match size)|Include only responses that match a specific size or range of sizes, using the same format as `-fs`.|You are looking for a backup file that you know is exactly 3456 bytes in size. Use `-ms 3456` to find it.|
|`-fw` (filter out number of words in response)|Exclude responses containing the specified number of words in the response.|You're filtering out a specific number of words from the responses. Use `-fw 219` to filter for responses containing that amount of words.|
|`-mw` (match word count)|Include only responses that have the specified amount of words in the response body.|You're looking for short, specific error messages. Use `-mw 5-10` to filter for responses with 5 to 10 words.|
|`-fl` (filter line)|Exclude responses with a specific number of lines or range of lines. For example, `-fl 5` will filter out responses with 5 lines.|You notice a pattern of 10-line error messages. Use `-fl 10` to filter them out.|
|`-ml` (match line count)|Include only responses that have the specified amount of lines in the response body.|You're looking for responses with a specific format, such as 20 lines. Use `-ml 20` to isolate them.|
|`-mt` (match time)|Include only responses that meet a specific time-to-first-byte (TTFB) condition. This is useful for identifying responses that are unusually slow or fast, potentially indicating interesting behavior.|The application responds slowly when processing certain inputs. Use `-mt >500` to find responses with a TTFB greater than 500 milliseconds.|

You can combine multiple filters. For example:

```shell
# Find directories with status code 200, based on the amount of words, and a response size greater than 500 bytes 
ffuf -u http://example.com/FUZZ -w wordlist.txt -mc 200 -fw 427 -ms >500 
# Filter out responses with status codes 404, 401, and 302 
ffuf -u http://example.com/FUZZ -w wordlist.txt -fc 404,401,302 
# Find backup files with the .bak extension and size between 10KB and 100KB
ffuf -u http://example.com/FUZZ.bak -w wordlist.txt -fs 0-10239 -ms 10240-102400
# Discover endpoints that take longer than 500ms to respond
ffuf -u http://example.com/FUZZ -w wordlist.txt -mt >500
```

## Exercise
### What flag do you find when successfully fuzzing the POST parameter?

```shell
ffuf -u http://154.57.164.64:30762/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200
# ffuf returns SUNwc with a 200 code
curl http://154.57.164.64:30762/post.php -X POST -d "y=SUNWmc"
```

### Fuzz the target system using directory-list-2.3-medium.txt, looking for a hidden directory. Once you have found the hidden directory, responsibly determine the validity of the vulnerability by analyzing the tar.gz file in the directory. Answer using the full Content-Length header, eg "Content-Length: 1337"

```shell
ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://154.57.164.71:31123/FUZZ -t 300 -fc 404
```

```shell
curl http://154.57.164.71:31123/ur-hiddenmember/backup.tar.gz -i
```
# Web APIs

| Feature            | Web Server                                                                                                                                                                                | API (Application Programming Interface)                                                                                                                                                                                                                                                                                  |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Purpose`          | Primarily designed to serve static content (HTML, CSS, images) and dynamic web pages (generated by server-side scripts).                                                                  | Primarily designed to provide a way for different software applications to communicate with each other, exchange data, and trigger actions.                                                                                                                                                                              |
| `Communication`    | Communicates with web browsers using the HTTP (Hypertext Transfer Protocol).                                                                                                              | Can use various protocols for communication, including HTTP, HTTPS, SOAP, and others, depending on the specific API.                                                                                                                                                                                                     |
| `Data Format`      | Primarily deals with HTML, CSS, JavaScript, and other web-related formats.                                                                                                                | Can exchange data in various formats, including JSON, XML, and others, depending on the API specification.                                                                                                                                                                                                               |
| `User Interaction` | Users interact with web servers directly through web browsers to view web pages and content.                                                                                              | Users typically do not interact with APIs directly; instead, applications use APIs to access data or functionality on behalf of the user.                                                                                                                                                                                |
| `Access`           | Web servers are usually publicly accessible over the internet.                                                                                                                            | APIs can be publicly accessible, private (for internal use only), or partner (accessible to specific partners or clients).                                                                                                                                                                                               |
| `Example`          | When you access a website like `https://www.example.com`, you are interacting with a web server that sends you the HTML, CSS, and JavaScript code to render the web page in your browser. | A weather app on your phone might use a weather API to fetch weather data from a remote server. The app then processes this data and displays it to you in a user-friendly format. You are not directly interacting with the API, but the app is using it behind the scenes to provide you with the weather information. |
# Why Fuzz APIs?

API fuzzing is crucial for several reasons:

- `Uncovering Hidden Vulnerabilities`: APIs often have hidden or undocumented endpoints and parameters that can be susceptible to attacks. Fuzzing helps uncover these hidden attack surfaces.
- `Testing Robustness`: Fuzzing assesses the API's ability to gracefully handle unexpected or malformed input, ensuring it doesn't crash or expose sensitive data.
- `Automating Security Testing`: Manual testing of all possible input combinations is infeasible. Fuzzing automates this process, saving time and effort.
- `Simulating Real-World Attacks`: Fuzzing can mimic the actions of malicious actors, allowing you to identify vulnerabilities before attackers exploit them.

## Types of API Fuzzing

There are 3 primary types of API fuzzing

1. `Parameter Fuzzing` - One of the primary techniques in API fuzzing, parameter fuzzing focuses on systematically testing different values for API parameters. This includes query parameters (appended to the API endpoint URL), headers (containing metadata about the request), and request bodies (carrying the data payload). By injecting unexpected or invalid values into these parameters, fuzzers can expose vulnerabilities like injection attacks (e.g., SQL injection, command injection), cross-site scripting (XSS), and parameter tampering.
2. `Data Format Fuzzing` - Web APIs frequently exchange data in structured formats like JSON or XML. Data format fuzzing specifically targets these formats by manipulating the structure, content, or encoding of the data. This can reveal vulnerabilities related to parsing errors, buffer overflows, or improper handling of special characters.
3. `Sequence Fuzzing` - APIs often involve multiple interconnected endpoints, where the order and timing of requests are crucial. Sequence fuzzing examines how an API responds to sequences of requests, uncovering vulnerabilities like race conditions, insecure direct object references (IDOR), or authorization bypasses. By manipulating the order, timing, or parameters of API calls, fuzzers can expose weaknesses in the API's logic and state management.
## Fuzzing the API

>[!Warning]
> It is recomended to create a virtual environmente before the next steps.

```shell
git clone https://github.com/PandaSt0rm/webfuzz_api.git 
cd webfuzz_api 
pip3 install -r requirements.txt

python3 api_fuzzer.py http://IP:PORT
```

In addition to discovering endpoints, fuzzing can be applied to the parameters these endpoints accept. By systematically injecting unexpected values into parameters, you can trigger errors, crashes, or unexpected behavior that could expose a wide range of vulnerabilities. For example, consider the following scenarios:

- `Broken Object-Level Authorization`: Fuzzing could reveal instances where manipulating parameter values can allow unauthorized access to specific objects or resources.
- `Broken Function Level Authorization`: Fuzzing might uncover cases where unauthorized function calls can be made by manipulating parameters, allowing attackers to perform actions they cannot.
- `Server-Side Request Forgery (SSRF)`: Injections of malicious values into parameters could trick the server into making unintended requests to internal or external resources, potentially exposing sensitive information or facilitating further attacks.
## Exercise
### What is the value returned by the endpoint that the api fuzzer has identified?

```python
python3 api_fuzzer.py http://154.57.164.82:30899
```

# Final Assessment

![[Ciberseguridad/HTB Apuntes/LABS/Web Fuzzing|Web Fuzzing]]

