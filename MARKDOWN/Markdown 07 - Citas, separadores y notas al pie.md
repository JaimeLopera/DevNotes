# 07 - Citas, separadores y notas al pie

> [!info] ¿Qué es?
> Son tres elementos pequeños pero muy útiles: **citas** (para destacar un texto de otra fuente), **líneas horizontales** (para separar partes) y **notas al pie** (para añadir aclaraciones sin ensuciar el texto).

## 1. Citas

Se crean con `>` al inicio de la línea.

`````markdown
> La simplicidad es la máxima sofisticación.
`````

**Varias líneas o párrafos:**

`````markdown
> Primera línea de la cita.
>
> Segundo párrafo de la cita.
`````

**Citas anidadas:**

`````markdown
> Cita principal
>> Respuesta dentro de la cita
`````

**Otros elementos dentro de una cita:**

`````markdown
> **Nota:**
> - Punto uno
> - Punto dos
`````

> [!tip] Callouts
> En Obsidian y GitHub, una cita con `[!tipo]` al principio se convierte en una caja especial. Mira [[Markdown 08 - Markdown en Obsidian]] y [[Markdown 09 - Markdown en GitHub]].

## 2. Líneas horizontales

Se hacen con tres o más guiones, asteriscos o guiones bajos, **solos en su línea**:

`````markdown
---
***
___
`````

Las tres dan el mismo resultado: una línea que separa secciones.

> [!warning] Cuidado con `---`
> Si pones `---` justo debajo de un texto **sin línea en blanco**, algunos programas lo convierten en un título (estilo antiguo). Deja siempre una línea en blanco antes.
>
> Además, `---` al **principio del archivo** se usa para el bloque de propiedades (frontmatter) en Obsidian.

## 3. Notas al pie

Marcas el lugar en el texto con `[^nombre]` y escribes la nota en otra parte (normalmente al final):

`````markdown
Markdown se creó en 2004[^1].

[^1]: Lo creó John Gruber.
`````

- La nota aparece numerada abajo, con un enlace para volver.
- El nombre puede ser un número o una palabra (`[^gruber]`); al mostrarse, se numeran solas.
- No es parte del Markdown original, pero GitHub y Obsidian las soportan.

**Nota al pie en una sola línea (Obsidian):**

`````markdown
Esto necesita explicación.^[Aquí va la explicación dentro del propio texto.]
`````

## 4. Buenas prácticas

- Usa citas solo para texto de otra fuente o avisos; no para dar sangría.
- Usa `---` con moderación: demasiadas líneas ensucian la nota.
- Pon las notas al pie todas juntas al final del archivo.

## 5. Casos especiales

- **Cita con código:** pon el bloque de código dentro, cada línea con `>`.
- **Línea horizontal dentro de una lista:** necesita sangría o se rompe la lista.
- **Notas al pie en otros programas:** si no se soportan, se verá `[^1]` como texto.

## 6. Resumen

- `>` = cita (se puede anidar con `>>`).
- `---`, `***` o `___` = línea horizontal (con línea en blanco antes).
- `[^1]` en el texto + `[^1]: texto` abajo = nota al pie.