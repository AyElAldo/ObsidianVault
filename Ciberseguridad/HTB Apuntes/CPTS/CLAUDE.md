# CLAUDE.md — HackerVault

`HackerVault.html` es una base de conocimiento de pentesting: un único archivo HTML, offline (funciona con `file://`, sin dependencias externas), con herramientas y comandos organizados por las 8 fases de la Cyber Kill Chain. Contenido en español. El archivo `Pentest-Knowledge-Base.html` es la versión antigua: NO tocarla.

## Cómo ampliar HackerVault.html (añadir herramientas desde los apuntes .md)

Cuando el usuario pida agregar herramientas nuevas (normalmente desde un apunte `.md` de esta carpeta):

1. Lee **solo** el/los archivo(s) `.md` que indique. No escanees todo el vault.
2. **No releas `HackerVault.html` entero** (son >100 KB). Para ubicar dónde insertar, usa `grep`/`Grep`:
   - último bloque de datos: busca `DATA.push` (inserta después del último).
   - versión: busca `const BASE_VERSION`.
3. Extrae solo las herramientas/comandos que **no estén ya** en la base (compara por `name`).
4. Añade **un único** bloque `DATA.push({...}, {...});` nuevo justo después del último `DATA.push` existente (antes del comentario `HELP_MAP`). No reescribas el archivo; usa `Edit` sobre esa zona.
5. **Sube `const BASE_VERSION` en 1** (p. ej. de `1` a `2`). Es imprescindible: el merge de `localStorage` solo incorpora las herramientas nuevas (por `name`) si la versión sube, y así no borra las ediciones locales del usuario.
6. Al terminar, di qué herramientas añadiste. No modifiques el diseño ni el motor.

## Formato de cada herramienta (objeto JS)

```
{
  name:"Nombre",
  phase:"recon|weapon|delivery|exploit|install|c2|actions|util",
  os:["Linux","Windows","Multiplataforma"],
  mitre:"TA0043 / T1595, T1046",
  approach:["reconocimiento","post-explotacion", ...],
  ambito:"red|host|ambas",
  source:"CPTS - ...",
  short:"Descripcion corta, 1-2 frases.",
  cmd:"comando principal con variables {{...}}",
  help:"comando -h / man comando",
  subs:[
    {title:"Subtitulo", short:"que hace", cmd:"comando", info:"HTML con <b>flags</b> explicados", examples:["ejemplo 1","ejemplo 2"], source:"CPTS - ..."}
  ]
}
```

### Reglas de contenido
- Variables globales que se sustituyen en vivo en los comandos (úsalas en `cmd`, `examples`, `subs[].cmd`):
  `{{RHOST}}` `{{RPORT}}` `{{LHOST}}` `{{LPORT}}` `{{DOMAIN}}` `{{TARGET_USER}}` `{{WORDLIST}}` `{{PIVOT_HOST}}`
- `phase`: fase de la Cyber Kill Chain (recon=Reconnaissance, weapon=Weaponization, delivery=Delivery, exploit=Exploitation, install=Installation, c2=Command&Control, actions=Actions on Objectives, util=Utilidades).
- `ambito`: `red` (sobre objetivos por red), `host` (enum/privesc/credenciales en un host ya comprometido) o `ambas`.
- `info` admite HTML; resalta flags y conceptos con `<b>...</b>`. Si necesitas la secuencia `</script>` dentro de un string, escríbela como `<\/script>` (si no, rompe el parser).
- Estilo del texto: español **sin tildes ni ñ** (igual que las entradas existentes: "Escaner", "evasion", "categoria"), para evitar problemas de codificacion.
- No dupliques herramientas ya presentes; si una existe pero con subcomandos nuevos, añade los `subs` nuevos a esa entrada.

## Diseño (no cambiar salvo que lo pidan)
Tema oscuro (negro `#050506` + acento lima `#e4f222`), colores de fase "Jewel Tones", pestañas de fase arriba (multi-seleccion), panel de variables a la izquierda ocultable, grid denso, copiar con animacion de contorno, paleta ⌘K, filtros SO/enfoque/Red-Host, CRUD + import/export JSON, persistencia en `localStorage` con merge por `BASE_VERSION`.
