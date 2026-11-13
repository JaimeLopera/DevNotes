# 02 - Encabezados y texto

> [!info] ¿Qué es?
> Es la parte más básica de Markdown: títulos, párrafos y estilos de texto como negrita, cursiva o tachado.

## 1. Antes de empezar

Necesitas haber leído [[Markdown 01 - Fundamentos]]. Aquí solo se escribe texto normal con algunos símbolos.

## 2. Encabezados

Se crean con `#` al principio de la línea. **Más `#` = título más pequeño**. Hay 6 niveles.

`````markdown
# Título nivel 1
## Título nivel 2
### Título nivel 3
#### Título nivel 4
##### Título nivel 5
###### Título nivel 6
`````

> [!warning] Importante
> Tiene que haber un **espacio** entre los `#` y el texto. `#Título` no funciona en la mayoría de sitios.

> [!warning] Obsoleto / legado
> Existe otra forma antigua (estilo "Setext") que subraya el título con `===` o `---`:
>
> ```markdown
> Título nivel 1
> ==============
> ```
>
> Solo sirve para niveles 1 y 2. **Alternativa actual:** usa siempre `#`.

## 3. Párrafos

Un párrafo es un bloque de texto. Para separar dos párrafos, deja **una línea en blanco** entre ellos.

`````markdown
Este es el primer párrafo.

Este es el segundo párrafo.
`````

## 4. Saltos de línea

Si escribes dos líneas seguidas sin línea en blanco, muchos programas las unen en una sola. Para forzar un salto dentro del mismo párrafo hay tres formas:

| Forma | Cómo | Comentario |
| --- | --- | --- |
| Dos espacios | Terminar la línea con 2 espacios | Funciona, pero no se ven y a veces se borran |
| Barra invertida | Terminar la línea con `\` | Clara, pero no la soportan todos |
| Etiqueta HTML | Terminar con `<br>` | Funciona casi en todas partes |

`````markdown
Línea uno<br>
Línea dos
`````

> [!tip] En Obsidian
> Por defecto, un solo Enter ya crea salto de línea (se puede cambiar en ajustes).

## 5. Estilos de texto

| Estilo | Sintaxis | Resultado |
| --- | --- | --- |
| Cursiva | `*texto*` o `_texto_` | *texto* |
| Negrita | `**texto**` o `__texto__` | **texto** |
| Negrita y cursiva | `***texto***` | ***texto*** |
| Tachado | `~~texto~~` | ~~texto~~ |
| Código en línea | `` `texto` `` | `texto` |
| Resaltado (Obsidian) | `==texto==` | ==texto== |

> [!tip] Asterisco o guion bajo
> Dentro de una palabra usa **asteriscos**: `des*pués*` funciona, pero `des_pués_` suele fallar. Por eso es mejor usar siempre `*` y `**`.

## 6. Ejemplo práctico

`````markdown
# Mi proyecto

Este proyecto es **muy sencillo** y está *en desarrollo*.

## Estado

Ya funciona ~~el login antiguo~~ el nuevo login.
Falta añadir `config.json`.
`````

## 7. Buenas prácticas

- Usa **un solo `#`** (título nivel 1) por nota: es el título de la nota.
- No saltes niveles: después de `##` va `###`, no `####`.
- No uses negrita para hacer títulos; usa encabezados de verdad (así se genera el índice y la navegación).
- Escoge siempre el mismo estilo de negrita y cursiva.

## 8. Casos especiales

- **Negrita en medio de una palabra:** usa asteriscos (`**`).
- **Mostrar asteriscos tal cual:** escápalos: `\*`.
- **Texto que empieza con número y punto** (`2026. Fue un buen año`) puede convertirse en lista. Escápalo: `2026\. Fue un buen año`.
- **Varios espacios seguidos** se muestran como uno solo. Para espacios extra usa `&nbsp;`.

## 9. Resumen

- `#` a `######` crean títulos (con espacio después).
- Línea en blanco = párrafo nuevo.
- `**negrita**`, `*cursiva*`, `~~tachado~~`, `` `código` ``.
- Para saltos de línea: dos espacios, `\` o `<br>`.
- Un solo título nivel 1 por nota y sin saltar niveles.