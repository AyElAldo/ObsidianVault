### What is the IANA ID of the registrar of the inlanefreight.com domain?

```shell
whois inlanefregiht.com
```
### What http server software is powering the inlanefreight.htb site on the target system? Respond with the name of the software, not the version, e.g., Apache.

```shell
curl -I inlanefreight.htb
```

### What is the API key in the hidden admin directory that you have discovered on the target system?

```shell
gobuster vhost -u http://inlanefreight.htb:30843 -w /usr/share/seclists/Discovery/DNS/combined_subdomains.txt -t 500 --append-domain

---
Found: web1337.inlanefreight.htb:30843 Status: 200 [Size: 104]
```

>[!Warning] This one was tricky. TRY WITH MANY WORDLISTS

In the `robots.txt`, we have:

```txt
User-agent: *
Allow: /index.html
Allow: /index-2.html
Allow: /index-3.html
Disallow: /admin_h1dd3n
```

We navigate to: `http://web1337.inlanefreight.htb:30843/admin_h1dd3n/`

```shell
Welcome to web1337 admin site
The admin panel is currently under maintenance, but the API is still accessible with the key e963d863ee0e82ba7080fbf558ca0d3f
```

### After crawling the inlanefreight.htb domain on the target system, what is the email address you have found? Respond with the full email

```shell
gobuster vhost -u http://web1337.inlanefreight.htb:32190 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 --append-domain
```

We obtain:

Then we use `ReconSpider` to get the next answers:

```shell
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:30253
```

![[Pasted image 20260813225322.png]]
