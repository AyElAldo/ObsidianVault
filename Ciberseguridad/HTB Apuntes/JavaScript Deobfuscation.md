#cybersecurity #HTB #JavaScript #defensive #CWES

Without understanding what this code is doing, we may not know what exactly the code is doing, and hence may not be able to complete the red/blue team exercise.
## Code Obfuscation
Obfuscation is a technique used to make a script more difficult to read by humans but allows it to function the same from a technical point of view, though performance may be slower.
## Use Cases

There are many reasons why developers may consider obfuscating their code. One common reason is to hide the original code and its functions to prevent it from being reused or copied without the developer's permission, making it more difficult to reverse engineer the code's original functionality. Another reason is to provide a security layer when dealing with authentication or encryption to prevent attacks on vulnerabilities that may be found within the code.

>[!Important] It must be noted that doing authentication or encryption on the client-side is not recommended, as code is more prone to attacks this way.

The most common usage of obfuscation, however, is for malicious actions. It is common for attackers and malicious actors to obfuscate their malicious scripts to prevent Intrusion Detection and Prevention systems from detecting their scripts.
## Basic Obfuscation
### Minifying JavaScript code
A common way of reducing the readability of a snippet of JavaScript code while keeping it fully functional is JavaScript minification. `Code minification` means having the entire code in a single (often very long) line. `Code minification` is more useful for longer code, as if our code only consisted of a single line, it would not look much different when minified.

>[!Info] Minifier URL: https://javascript-minifier.com/
#### Example
**INPUT:**
```js
var numero = 10;
console.log(numero);
let numero2 = 12;
let num3 = numero + numero2;
console.log("SUMA: " + num3)
```
**Output**:
```js
var numero=10;console.log(numero);let numero2=12,num3=numero+numero2;console.log("SUMA: "+num3);
```
## Packing JavaScript code
Now, let us obfuscate our line of code to make it more obscure and difficult to read. First, we will try [BeautifyTools](http://beautifytools.com/javascript-obfuscator.php) to obfuscate our code.

>[!info] URL: http://beautifytools.com/javascript-obfuscator.php

#### Example
**INPUT:**
```js
var numero=10;console.log(numero);let numero2=12,num3=numero+numero2;console.log("SUMA: "+num3);
```
**Output**:
```js
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5 0=6;2.4(0);9 3=7,1=0+3;2.4("8: "+1);',10,10,'numero|num3|console|numero2|log|var|10|12|SUMA|let'.split('|'),0,{}))
```
## Advanced Obfuscation

**URL**: https://obfuscator.io/legacy-playground
#### Example
**INPUT:**
```js
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5 0=6;2.4(0);9 3=7,1=0+3;2.4("8: "+1);',10,10,'numero|num3|console|numero2|log|var|10|12|SUMA|let'.split('|'),0,{}))
```
**Output**:
```js
var _0x4fccbb=_0xab03;(function(_0x1aed86,_0x4db2cc){var _0x1b63f7=_0xab03,_0x4abc9c=_0x1aed86();while(!![]){try{var _0x4dd182=parseInt(_0x1b63f7(0x1d0))/0x1*(-parseInt(_0x1b63f7(0x1cc))/0x2)+parseInt(_0x1b63f7(0x1d6))/0x3*(-parseInt(_0x1b63f7(0x1d5))/0x4)+-parseInt(_0x1b63f7(0x1d1))/0x5*(-parseInt(_0x1b63f7(0x1d7))/0x6)+parseInt(_0x1b63f7(0x1cf))/0x7*(-parseInt(_0x1b63f7(0x1ca))/0x8)+-parseInt(_0x1b63f7(0x1c6))/0x9+parseInt(_0x1b63f7(0x1d3))/0xa*(-parseInt(_0x1b63f7(0x1d4))/0xb)+-parseInt(_0x1b63f7(0x1d2))/0xc*(-parseInt(_0x1b63f7(0x1cb))/0xd);if(_0x4dd182===_0x4db2cc)break;else _0x4abc9c['push'](_0x4abc9c['shift']());}catch(_0x16a27a){_0x4abc9c['push'](_0x4abc9c['shift']());}}}
...
var _0xb01ac5=_0x2fcf1e[0x0],_0x1441b2=_0x598f67+_0xb01ac5,_0x2d62a3=_0xab03['lMjpRW'][_0x1441b2];return!_0x2d62a3?(_0xab03b3=_0xab03['RastWB'](_0xab03b3),_0xab03['lMjpRW'][_0x1441b2]=_0xab03b3):_0xab03b3=_0x2d62a3,_0xab03b3;}function _0x2fcf(){var _0x505bcd=['mJjJzwHtwu4','C3bSAxq','nsaWpty7mI40kdaPoZKGmZ03lde9mcSZoZiUncGIodOGiISXktS','n2Hgwwnpva','mZe4nJv1rgj4CNO','nvzqwxHrCG','mte3ndC4mZjoCuDHte4','mte2otbWBwzzAxK','ndqXmw1fs2jOsq','mty0Aw1IqwPU','mJaWmtL3v2jsELG','ote4odq2s3z0wgz0','mtC3mJCXmM9owLjqCa','xhCR','CMvWBgfJzq','BNvTzxjVFg51Btn8y29UC29SzxXUDw1LCM8YFgXVz3X2yxj8mtb8mtj8u1vnqxXSzxq','ndmYntyYnffyDKfhAG','mJzls0TjuLC'];_0x2fcf=function(){return _0x505bcd;};return _0x2fcf();}
```
# Deobfuscation
Just as there are tools to obfuscate code automatically, there are tools to beautify and deobfuscate the code automatically.
## Beautify
We see that the current code we have is all written in a single line. This is known as `Minified JavaScript` code. In order to properly format the code, we need to `Beautify` our code. The most basic method for doing so is through our `Browser Dev Tools`.
For example, if we were using Firefox, we can open the browser debugger with `CTRL+SHIFT+Z`, and then click on our script `secret.js`. This will show the script in its original formatting, but we can click on the '`{ }`' button at the bottom, which will `Pretty Print` the script into its proper JavaScript formatting.

Furthermore, we can utilize many online tools or code editor plugins, like [Prettier](https://prettier.io/playground/) or [Beautifier](https://beautifier.io/). Let us copy the `secret.js` script:
![[prettier.png]]
## Unpack
We can find many good online tools to deobfuscate JavaScript code and turn it into something we can understand. One good tool is [UnPacker](https://matthewfl.com/unPacker.html). Let's try copying our above-obfuscated code and run it in UnPacker by clicking the `UnPack` button.

>[!Tip] Ensure you do not leave any empty lines before the script, as it may affect the deobfuscation process and give inaccurate results.

**URL:** https://matthewfl.com/unPacker.html
## Reverse Engineering

Though these tools are doing a good job so far in clearing up the code into something we can understand, once the code becomes more obfuscated and encoded, it would become much more difficult for automated tools to clean it up. This is especially true if the code was obfuscated using a custom obfuscation tool.

We would need to manually reverse engineer the code to understand how it was obfuscated and its functionality for such cases. If you are interested in knowing more about advanced JavaScript Deobfuscation and Reverse Engineering, you can check out the [Secure Coding 101](https://academy.hackthebox.com/module/details/38) module, which should thoroughly cover this topic.
# Web Requests
In the exercise, we send a POST request that returns a non-readeable string.
```shell
curl http://154.57.164.82:32196/serial.php/ -X POST -d "param1=prueba"
############################## OUTPUT
N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz% 
```
This is another important aspect of obfuscation that we referred to in `More Obfuscation` in the `Advanced Obfuscation` section. Many techniques can further obfuscate the code and make it less readable by humans and less detectable by systems. For that reason, you will very often find obfuscated code containing encoded text blocks that get decoded upon execution. We will cover 3 of the most commonly used text encoding methods:

- `base64`
- `hex`
- `rot13`
## Base64

`base64` encoding is usually used to reduce the use of special characters, as any characters encoded in `base64` would be represented in alpha-numeric characters, in addition to `+` and `/` only. Regardless of the input, even if it is in binary format, the resulting base64 encoded string would only use them.
#### Spotting Base64

`base64` encoded strings are easily spotted since they only contain alpha-numeric characters. However, the most distinctive feature of `base64` is its padding using `=` characters. The length of `base64` encoded strings has to be in a multiple of 4. If the resulting output is only 3 characters long, for example, an extra `=` is added as padding, and so on.

```shell
AyElAldo@htb[/htb]$ echo aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K | base64 -d
```
## Hex
Another common encoding method is `hex` encoding, which encodes each character into its `hex` order in the `ASCII` table. For example, `a` is `61` in hex, `b` is `62`, `c` is `63`, and so on. You can find the full `ASCII` table in Linux using the `man ascii` command.
#### Spotting Hex
Any string encoded in `hex` would be comprised of hex characters only, which are 16 characters only: 0-9 and a-f. That makes spotting `hex` encoded strings just as easy as spotting `base64` encoded strings.
#### Hex Encode
To encode any string into `hex` in Linux, we can use the `xxd -p` command:
```shell
AyElAldo@htb[/htb]$ echo https://www.hackthebox.eu/ | xxd -p
```
#### Hex Decode
To decode a `hex` encoded string, we can use the `xxd -p -r` command:
```shell
AyElAldo@htb[/htb]$ echo 68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r
```
## Caesar/Rot13
Another common -and very old- encoding technique is a Caesar cipher, which shifts each letter by a fixed number. For example, shifting by 1 character makes `a` become `b`, and `b` becomes `c`, and so on. Many variations of the Caesar cipher use a different number of shifts, the most common of which is `rot13`, which shifts each character 13 times forward.
#### Spotting Caesar/Rot13

Even though this encoding method makes any text looks random, it is still possible to spot it because each character is mapped to a specific character. For example, in `rot13`, `http://www` becomes `uggc://jjj`, which still holds some resemblances and may be recognized as such.
#### Rot13 Encode

There isn't a specific command in Linux to do `rot13` encoding. However, it is fairly easy to create our own command to do the character shifting:

```shell
AyElAldo@htb[/htb]$ echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m' uggcf://jjj.unpxgurobk.rh/
```
#### Rot13 Decode
We can use the same previous command to decode rot13 as well:
```shell
AyElAldo@htb[/htb]$ echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m' https://www.hackthebox.eu/
```
## Other Types of Encoding

There are hundreds of other encoding methods we can find online. Even though these are the most common, sometimes we will come across other encoding methods, which may require some experience to identify and decode.

`If you face any similar types of encoding, first try to determine the type of encoding, and then look for online tools to decode it.`

Some tools can help us automatically determine the type of encoding, like [Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier). Try the encoded strings above with [Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier), to see if it can correctly identify the encoding method.
# Exercise solution

```shell
curl -s http://154.57.164.63:30517/serial.php -X POST -d "serial=7h15_15_a_s3cr37_m3554g3"
```
# Skill Assessment

![[Ciberseguridad/HTB Apuntes/LABS/JavaScript Deobfuscation|JavaScript Deobfuscation]]