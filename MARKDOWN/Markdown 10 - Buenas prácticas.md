# 10 - Buenas prácticas

> [!info] ¿Qué es?
> Reglas sencillas para que tu Markdown sea **limpio, fácil de leer, fácil de mantener y compatible** con distintos programas.

## 1. Estructura

- **Un solo `#`** por nota (el título).
- Los títulos siguen un orden lógico: `##` → `###` → `####`, sin saltos.
- Cada apartado trata **una sola idea**.
- Usa títulos descriptivos: "Instalación", no "Cosas".
- Las notas largas se dividen en notas más cortas enlazadas.

## 2. Formato

| Haz | Evita |
| --- | --- |
| Usar siempre `-` para viñetas | Mezclar `-`, `*` y `+` |
| Usar `**negrita**` y `*cursiva*` | Mezclar `__` y `**` |
| Dejar línea en blanco entre bloques | Pegar listas, tablas y párrafos |
| Sangría de 2 o 4 espacios, siempre igual | Mezclar tabulaciones y espacios |
| Un bloque de código con lenguaje indicado | Bloques sin lenguaje |

## 3. Texto

- Negrita para lo realmente importante; si todo es negrita, nada destaca.
- No uses negrita como si fuera un título.
- Frases cortas y párrafos pequeños.
- No dependas de espacios finales para los saltos de línea: usa `\` o `<br>` si es necesario.

## 4. Enlaces e imágenes

- Texto de enlace descriptivo.
- Texto alternativo en todas las imágenes.
- Rutas relativas dentro del proyecto.
- Nombres de archivo sin espacios ni acentos.

## 5. Compatibilidad

Antes de usar un extra, pregúntate **dónde se va a leer** la nota:

| Si se va a leer en... | Puedes usar | Evita |
| --- | --- | --- |
| Solo Obsidian | Todo | — |
| GitHub | GFM (tablas, tareas, alertas) | Wikilinks, callouts de Obsidian, `==resaltado==` |
| Cualquier sitio | Solo sintaxis básica | Todos los extras |

> [!tip] Idea
> Si dudas, usa solo lo básico y cuando necesites algo especial, usa HTML.

## 6. Herramientas útiles

| Herramienta | Para qué |
| --- | --- |
| **markdownlint** | Detecta errores y falta de consistencia |
| **Prettier** | Formatea el Markdown automáticamente |
| **Vista previa** (VS Code, Obsidian) | Ver el resultado mientras escribes |
| **Pandoc** | Convierte Markdown a PDF, Word, HTML... |

En VS Code: `Ctrl + Shift + V` abre la vista previa del Markdown.

## 7. Errores típicos

❌ La lista no sale como lista.
🤔 Falta la línea en blanco antes o la sangría es irregular.
✅ Deja una línea en blanco antes y usa siempre la misma sangría.

❌ El título no se convierte en título.
🤔 Falta el espacio después de `#`.
✅ Escribe `# Título`, no `#Título`.

❌ Un asterisco o guion bajo desaparece o pone cursiva.
🤔 Markdown lo interpreta como formato.
✅ Escápalo con `\*` o `\_`, o ponlo en código.

❌ La tabla se ve como texto con barras.
🤔 Falta la fila de guiones o el programa no soporta tablas.
✅ Revisa la fila `| --- |` y comprueba el programa.

## 8. Resumen

- Estructura clara, un solo título principal, niveles en orden.
- Siempre el mismo estilo para cada elemento.
- Línea en blanco entre bloques.
- Piensa dónde se va a leer antes de usar extras.
- Usa vista previa y un linter para revisar.