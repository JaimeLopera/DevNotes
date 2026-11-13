# 11 - Referencia rápida de Markdown

> [!info] ¿Qué es?
> Chuleta con **toda la sintaxis** de la carpeta en un solo sitio. Para explicaciones, ve a cada nota.

## Texto

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| Título 1 a 6 | `#` a `######` | [[Markdown 02 - Encabezados y texto]] |
| Párrafo nuevo | Línea en blanco | |
| Salto de línea | `<br>` o `\` al final | |
| Cursiva | `*texto*` | |
| Negrita | `**texto**` | |
| Negrita y cursiva | `***texto***` | |
| Tachado | `~~texto~~` | |
| Resaltado | `==texto==` | Solo Obsidian |
| Escapar símbolo | `\*` | [[Markdown 01 - Fundamentos]] |
| Comentario | `<!-- texto -->` | |

## Listas

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| Viñetas | `- elemento` | [[Markdown 03 - Listas]] |
| Numerada | `1. elemento` | |
| Anidada | Sangría de 2 o 4 espacios | |
| Tarea pendiente | `- [ ] tarea` | |
| Tarea hecha | `- [x] tarea` | |

## Enlaces e imágenes

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| Enlace | `[texto](url)` | [[Markdown 04 - Enlaces e imágenes]] |
| Enlace con título | `[texto](url "título")` | |
| Enlace a apartado | `[texto](#apartado)` | |
| Enlace automático | `<https://...>` | |
| Referencia | `[texto][id]` y `[id]: url` | |
| Imagen | `![alt](ruta)` | |
| Imagen con enlace | `[![alt](ruta)](url)` | |

## Código

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| En línea | `` `código` `` | [[Markdown 05 - Código]] |
| Bloque | ` ```lenguaje ` ... ` ``` ` | |
| Bloque dentro de bloque | Más comillas fuera (4 o más) | |
| Cambios | ` ```diff ` | |
| Diagrama | ` ```mermaid ` | |

## Tablas

`````markdown
| Izquierda | Centro | Derecha |
| :--- | :---: | ---: |
| a | b | c |
`````

Más detalle en [[Markdown 06 - Tablas]].

## Citas, separadores y notas

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| Cita | `> texto` | [[Markdown 07 - Citas, separadores y notas al pie]] |
| Cita anidada | `>> texto` | |
| Línea horizontal | `---` | |
| Nota al pie | `[^1]` y `[^1]: texto` | |

## Obsidian

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| Enlace a nota | `[[Nota]]` | [[Markdown 08 - Markdown en Obsidian]] |
| Con alias | `[[Nota\|Texto]]` | |
| A un título | `[[Nota#Título]]` | |
| A un bloque | `[[Nota#^id]]` | |
| Insertar nota | `![[Nota]]` | |
| Imagen con ancho | `![[img.png\|300]]` | |
| Callout | `> [!tipo] Título` | |
| Callout plegable | `> [!tipo]-` o `> [!tipo]+` | |
| Etiqueta | `#etiqueta` | |
| Comentario | `%% texto %%` | |
| Fórmula | `$x^2$` | |

**Tipos de callout:** `note`, `info`, `tip`, `success`, `question`, `warning`, `failure`, `danger`, `bug`, `example`, `quote`, `abstract`, `todo`.

## GitHub

| Qué | Sintaxis | Nota |
| --- | --- | --- |
| Alerta | `> [!NOTE]` | [[Markdown 09 - Markdown en GitHub]] |
| Tipos de alerta | `NOTE` `TIP` `IMPORTANT` `WARNING` `CAUTION` | |
| Mención | `@usuario` | |
| Issue / PR | `#123` | |
| Emoji | `:rocket:` | |
| Desplegable | `<details><summary>` | |

## Compatibilidad

| Función | Estándar | GitHub | Obsidian |
| --- | :---: | :---: | :---: |
| Títulos, listas, enlaces, código | ✅ | ✅ | ✅ |
| Tablas | ❌ | ✅ | ✅ |
| Tachado y tareas | ❌ | ✅ | ✅ |
| Notas al pie | ❌ | ✅ | ✅ |
| Wikilinks `[[ ]]` | ❌ | ❌ | ✅ |
| Callouts `[!tipo]` | ❌ | ✅ (5 tipos) | ✅ (muchos) |
| Resaltado `==` | ❌ | ❌ | ✅ |
| Diagramas `mermaid` | ❌ | ✅ | ✅ |

## Consejos finales

> [!tip] Los 5 que más importan
> 1. Línea en blanco entre bloques.
> 2. Espacio después de `#`, `-` y `>`.
> 3. Sangría siempre igual.
> 4. Lenguaje en cada bloque de código.
> 5. Piensa dónde se va a leer antes de usar extras.