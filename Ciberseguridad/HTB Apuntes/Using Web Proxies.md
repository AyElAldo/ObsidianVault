# Theory
With most applications heavily relying on back-end servers to process data, testing and securing the back-end servers is quickly becoming more important.
To capture the requests and traffic passing between applications and back-end servers and manipulate these types of requests for testing purposes, we need to use `Web Proxies`.
## What Are Web Proxies?

Web proxies are specialized tools that can be set up between a browser/mobile application and a back-end server to capture and view all the web requests being sent between both ends, essentially acting as man-in-the-middle (MITM) tools.
While other `Network Sniffing` applications, like Wireshark, operate by analyzing all local traffic to see what is passing through a network, Web Proxies mainly work with web ports such as, but not limited to, `HTTP/80` and `HTTPS/443`.
## Uses of Web Proxies

While the primary use of web proxies is to capture and replay HTTP requests, they have many other features that enable different uses for web proxies. The following list shows some of the other tasks we may use web proxies for:
- Web application vulnerability scanning
- Web fuzzing
- Web crawling
- Web application mapping
- Web request analysis
- Web configuration testing
- Code reviews
## Burp Suite

[Burp Suite (Burp)](https://portswigger.net/burp) -pronounced Burp Sweet- is the most common web proxy for web penetration testing. It has an excellent user interface for its various features and even provides a built-in Chromium browser to test web applications. Certain Burp features are only available in the commercial version `Burp Pro/Enterprise`, but even the free version is an extremely powerful testing tool to keep in our arsenal.

Some of the `paid-only` features are:

- Active web app scanner
- Fast Burp Intruder
- The ability to load certain Burp Extensions
## OWASP Zed Attack Proxy (ZAP)
[OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org/) is another common web proxy tool for web penetration testing. ZAP is a free and open-source project initiated by the [Open Web Application Security Project (OWASP)](https://owasp.org/) and maintained by the community. 
# Proxy Setup
## Installing CA Certificate
Another important step when using Burp Proxy/ZAP with our browser is installing the web proxy's CA Certificates. If we don't do this step, some HTTPS traffic may not get properly routed, or we may need to click `accept` every time Firefox needs to send an HTTPS request. We can install Burp's certificate once we select Burp as our proxy in `Foxy Proxy`, by browsing to `http://burp`, and downloading the certificate from there by clicking on `CA Certificate`.
## Manipulating Intercepted Requests

Once we intercept the request, it will remain hanging until we forward it, as we did above. We can examine the request, manipulate it to make any changes we want, and then send it to its destination. This helps us better understand what information a particular web application is sending in its web requests and how it may respond to any changes we make in that request.

There are numerous applications for this in Web Penetration Testing, such as testing for:

1. SQL injections
2. Command injections
3. Upload bypass
4. Authentication bypass
5. XSS
6. XXE
7. Error handling
8. Deserialization

>[!Example] Example
>When intercepting a Linux command within a parameter, we can close the command and injecting another one:
>- Original
>```shell
>ip=1
>```
>- Modified
>```shell
>ip=1;ls;cat flag.txt;
>```

## Automatic Request 

- Proxy>Proxy settings>HTTP match and replace rules
>[!Example] 
>```
>^User-Agent.*$ --> User-Agent: HackTheBox Agent 1.0`
>```
>We must enalble *regex match*.



