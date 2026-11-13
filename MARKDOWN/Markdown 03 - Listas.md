# 03 - Listas

> [!info] ¿Qué es?
> Las listas organizan elementos en forma de viñetas, números o tareas por hacer. Se pueden anidar unas dentro de otras.

## 1. Antes de empezar

Recuerda dejar una **línea en blanco antes de empezar la lista** (mira [[Markdown 02 - Encabezados y texto]]).

## 2. Lista con viñetas

Usa `-`, `*` o `+` seguido de un espacio.

`````markdown
- Manzana
- Pera
- Naranja
`````

> [!tip] Elige uno
> Los tres símbolos funcionan igual. Usa siempre el mismo (lo más común es `-`).

## 3. Lista numerada

Usa un número, un punto y un espacio.

`````markdown
1. Abrir el editor
2. Escribir la nota
3. Guardar el archivo
`````

> [!tip] Truco
> Solo importa el **primer número**. Si escribes `1.` `1.` `1.` se numerará solo 1, 2, 3. Así, si añades o quitas un elemento, no tienes que renumerar.

## 4. Listas anidadas

Para meter una lista dentro de otra, **sangra** (indenta) el elemento con 2 o 4 espacios (o una tabulación). Mantén siempre la misma cantidad.

`````markdown
- Frontend
  - HTML
  - CSS
- Backend
  - PHP
  - Node.js
`````

También puedes mezclar tipos:

`````markdown
1. Preparar
   - Instalar Node
   - Instalar Git
2. Ejecutar
`````

## 5. Listas de tareas

Se escriben como una lista con `[ ]` (pendiente) o `[x]` (hecha).

`````markdown
- [x] Crear la carpeta
- [ ] Escribir la nota
- [ ] Revisar la nota
`````

Funcionan en Obsidian y GitHub (mira [[Markdown 08 - Markdown en Obsidian]] y [[Markdown 09 - Markdown en GitHub]]). En Obsidian puedes marcarlas con un clic.

## 6. Párrafos y código dentro de listas

Si quieres meter más texto dentro de un elemento, sangra el contenido y deja una línea en blanco:

`````markdown
1. Instala el programa.

   Puede tardar unos minutos.

2. Ábrelo.
`````

## 7. Buenas prácticas

- Un mismo símbolo para todas las viñetas.
- Sangría consistente (siempre 2 o siempre 4 espacios).
- No anides más de 3 niveles; si se complica, divide en secciones.
- Cada elemento debe ser corto y del mismo tipo.

## 8. Casos especiales

- **Lista que se rompe:** suele pasar por falta de línea en blanco antes o por una sangría irregular.
- **Un elemento con salto de línea:** termina la línea con dos espacios o `<br>`.
- **Empezar la numeración en otro número:** `5.` como primer elemento hace que siga 5, 6, 7 (en la mayoría de sitios).

## 9. Resumen

- Viñetas: `-`, `*` o `+`.
- Numeradas: `1.` (basta con el primero).
- Anidar = sangrar 2 o 4 espacios.
- Tareas: `- [ ]` y `- [x]`.
- Línea en blanco antes de la lista y sangría coherente.