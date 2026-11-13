# 08 - Markdown en Obsidian

> [!info] ¿Qué es?
> Obsidian usa Markdown normal (CommonMark + GFM) y le añade **funciones propias** pensadas para conectar notas entre sí: enlaces internos, callouts, etiquetas, propiedades y más.

## 1. Antes de empezar

- Todo lo de las notas [[Markdown 02 - Encabezados y texto]] a [[Markdown 07 - Citas, separadores y notas al pie]] funciona igual en Obsidian.
- Lo de esta nota **solo funciona en Obsidian** (u otros programas compatibles). En GitHub u otros sitios saldrá como texto raro.

## 2. Wikilinks (enlaces entre notas)

Se escriben con dobles corchetes:

| Sintaxis | Qué hace |
| --- | --- |
| `[[Nota]]` | Enlaza a la nota "Nota" |
| `[[Nota\|Texto]]` | Enlaza mostrando otro texto (alias) |
| `[[Nota#Apartado]]` | Enlaza a un título concreto de esa nota |
| `[[Nota#^id]]` | Enlaza a un bloque concreto |
| `[[#Apartado]]` | Enlaza a un título de la misma nota |

> [!tip] Cómo crear un ID de bloque
> Escribe `^mi-id` al final de un párrafo y luego enlaza con `[[Nota#^mi-id]]`.

Si el enlace apunta a una nota que no existe, se muestra en otro color; al hacer clic, Obsidian la crea.

**Enlaces estilo Markdown:** Obsidian también acepta `[texto](Nota.md)`. Los wikilinks son más cómodos, pero los de estilo Markdown son más compatibles con otros programas.

## 3. Insertar contenido (embeds)

Con un `!` delante, el contenido se **muestra dentro** de la nota en lugar de solo enlazarlo:

`````markdown
![[Otra nota]]
![[Otra nota#Apartado]]
![[imagen.png]]
![[imagen.png|300]]
![[documento.pdf#page=3]]
`````

`|300` define el ancho en píxeles.

## 4. Callouts

Son cajas de aviso hechas con una cita y `[!tipo]`:

`````markdown
> [!note] Título opcional
> Contenido del aviso.
`````

**Tipos principales:**

| Tipo | Uso |
| --- | --- |
| `note` | Nota general |
| `info` | Información |
| `tip` | Consejo |
| `success` | Algo que salió bien |
| `question` | Pregunta o duda |
| `warning` | Advertencia |
| `failure` | Fallo |
| `danger` | Peligro |
| `bug` | Error conocido |
| `example` | Ejemplo |
| `quote` | Cita |
| `abstract` | Resumen |
| `todo` | Tarea pendiente |

**Plegables:** añade `-` (empieza cerrado) o `+` (empieza abierto):

`````markdown
> [!tip]- Solución
> Este texto está oculto hasta que se hace clic.
`````

## 5. Etiquetas (tags)

Se escriben con `#`, sin espacios:

`````markdown
#idea #daw/markdown
`````

- Se pueden **anidar** con `/` (`#daw/markdown`).
- Pueden llevar letras, números, `_`, `-` y `/`, pero **no solo números** (`#2026` no vale).
- También se pueden poner en las propiedades (ver más abajo).

## 6. Propiedades (frontmatter)

Es un bloque al **inicio del archivo**, entre `---`, en formato YAML:

`````markdown
---
tags: [markdown, apuntes]
fecha: 2026-10-13
estado: terminado
aliases: [MD]
---
`````

- `aliases` permite enlazar la nota con otros nombres.
- `tags` define etiquetas sin usar `#`.
- Debe ser **lo primero** del archivo.

## 7. Otros extras

| Función | Sintaxis | Para qué |
| --- | --- | --- |
| Resaltado | `==texto==` | Marcar texto importante |
| Comentario | `%% texto %%` | Nota oculta en la vista de lectura |
| Fórmula en línea | `$x^2$` | Matemáticas (LaTeX) |
| Fórmula en bloque | `$$ ... $$` | Matemáticas grandes |
| Diagrama | ` ```mermaid ` | Diagramas y gráficos |
| Tarea | `- [ ] algo` | Lista de tareas |

Ejemplo de diagrama:

`````markdown
````mermaid
graph LR
  A[Escribir] --> B[Renderizar]
````
`````

## 8. Buenas prácticas

- Usa nombres de nota **claros y únicos** para que los wikilinks no se confundan.
- Usa alias en los enlaces cuando el nombre sea largo: `[[01 - Fundamentos\|Fundamentos]]`.
- No abuses de etiquetas: mejor pocas y con sentido.
- Los embeds son útiles para reutilizar texto, pero no los uses en exceso (hacen las notas pesadas).
- Si quieres que tu nota se pueda leer fuera de Obsidian, evita depender de los extras.

## 9. Casos especiales

- **Barra `|` en tablas:** dentro de una tabla, un wikilink con alias necesita `\|`: `[[Nota\|Texto]]`.
- **Renombrar una nota:** Obsidian actualiza los enlaces automáticamente (si la opción está activa).
- **Espacios en wikilinks:** se permiten, no hace falta `%20`.
- **Callout dentro de lista:** hay que sangrar toda la caja.

## 10. Resumen

- `[[Nota]]` enlaza; `![[Nota]]` inserta.
- Callouts: `> [!tipo] Título`.
- Etiquetas con `#` y propiedades en YAML al inicio.
- Extras: `==resaltado==`, `%%comentario%%`, `$fórmula$`, `mermaid`.
- Todo esto es propio de Obsidian: no es Markdown estándar.