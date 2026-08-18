#cybersecurity #university #tools #laws 
##  Técnicas y métodos para la detección y prevención de amenazas a la ciberseguridad

### Métrica de explotabilidad

> Calculadora NIST para métricas de explotabilidad: https://nvd.nist.gov/vuln-metrics/cvss/v4-calculator

### Listas de control de accesos

Para poder permitir o bloquear el tráfico debemos conocer IP y puertos origen y destino.
#### Sintáxis de creación de ACLs:


```shell
access-list 101 permit tcp 192.168.1.0 0.0.0.255 any eq 80
```

En este ejemplo:
- _101_ es el número de la ACL,
- _permit_ es la acción,
- _tcp_ es el protocolo,
- _192.168.1.0 0.0.0.255_ define el rango de origen,
- _any_ es cualquier destino,
- _eq 80_ indica puerto 80 (HTTP).

#### **Ejemplo práctico:**

![[topologia.png]]

- Standard 

- Extended

Para este caso usamos extended

![[ACLs.png]]

![[ACLs_2.png]]

![[Pasted image 20251004122500.png]]
## Investigación profunda de malware

### Definiciones

- Punto de entrada: Dirección de la primera instrucción a ejecutar.

### Malwares Vistos

- Virus (Calc_Sality)
	- Se compararon hashes en los archivos corruptos (.exe y .dll)
	- Se comparó el original con el malicioso a nivel byte
- Gusano (Hiberium.exe)
	- 
- Dropper
- Downloader
- Scareware
- Backdoor
- Rootkit
- Keyloggers

# 6.1 Construcción y vigilancia de la ciberseguridad
## NIST

**NO ESPECIFICA COMO SE DEBE LOGRAR LOS RESULTADOS**


