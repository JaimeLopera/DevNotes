# 06 - Tablas

> [!info] ¿Qué es?
> Las tablas organizan datos en filas y columnas. Se dibujan con barras verticales `|` y guiones `-`. No son parte del Markdown original, pero Obsidian y GitHub las soportan.

## 1. Sintaxis básica

Tiene tres partes:

1. **Fila de cabecera** (los títulos de columna).
2. **Fila separadora** (guiones, obligatoria).
3. **Filas de datos**.

`````markdown
| Nombre | Edad | Ciudad |
| --- | --- | --- |
| Ana | 25 | Madrid |
| Luis | 30 | Sevilla |
`````

> [!tip] No hace falta alinear
> Los espacios sobrantes no importan. Para que se vea ordenado en el texto sin renderizar, puedes alinear las barras, pero es opcional.

## 2. Alinear columnas

Con dos puntos `:` en la fila separadora:

| Sintaxis | Alineación |
| --- | --- |
| `:---` | Izquierda |
| `:---:` | Centro |
| `---:` | Derecha |

`````markdown
| Producto | Cantidad | Precio |
| :--- | :---: | ---: |
| Lápiz | 10 | 1,50 € |
| Cuaderno | 3 | 4,00 € |
`````

## 3. Formato dentro de las celdas

Se pueden usar negrita, cursiva, código y enlaces:

`````markdown
| Comando | Qué hace |
| --- | --- |
| `git add` | **Prepara** los cambios |
| `git commit` | Guarda *una foto* del proyecto |
`````

## 4. Barras `|` dentro de una celda

Hay que escaparlas con `\`:

`````markdown
| Operador | Significado |
| --- | --- |
| `a \| b` | a o b |
`````

## 5. Saltos de línea dentro de una celda

No se pueden con Enter. Usa `<br>`:

`````markdown
| Pasos |
| --- |
| Primero<br>Segundo<br>Tercero |
`````

## 6. Buenas prácticas

- Usa tablas para **datos comparables**, no para maquetar.
- Mantén las celdas **cortas**; si hay mucho texto, mejor una lista.
- Pon siempre cabecera.
- No hagas tablas con más de 5 o 6 columnas: son difíciles de leer.
- Para tablas grandes, valora usar una herramienta o un plugin de tablas del editor.

## 7. Diferencias importantes

| Cosa | Markdown estándar | GFM / Obsidian |
| --- | --- | --- |
| Tablas | No existen | Sí |
| Alineación | — | Sí |

Si tu Markdown no las soporta, la tabla se verá como texto con barras.

## 8. Casos especiales

- **No se pueden unir celdas** (no hay `colspan` ni `rowspan`). Si lo necesitas, usa HTML con `<table>`.
- **No se pueden poner listas ni bloques de código de varias líneas** dentro de una celda.
- **Tabla vacía:** una celda sin contenido se deja con solo espacios entre las barras.
- **Las barras de los extremos** (`|` al inicio y al final de cada fila) son opcionales, pero conviene ponerlas.

## 9. Resumen

- Cabecera + fila `| --- |` + filas de datos.
- `:---`, `:---:`, `---:` para alinear.
- Escapa `|` con `\|` y usa `<br>` para saltos de línea.
- Sin celdas combinadas ni bloques complejos dentro.