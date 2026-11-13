# 01 - Fundamentos de Markdown

> [!info] ¿Qué es?
> **Markdown** es una forma de escribir texto con formato (títulos, negritas, listas, enlaces...) usando solo caracteres normales del teclado. Después, un programa lo convierte en algo bonito, normalmente HTML.

## 1. Antes de empezar

- No necesitas instalar nada: cualquier editor de texto sirve.
- Los archivos de Markdown terminan en **`.md`** (a veces `.markdown`).
- Conviene saber lo básico de [[HTML 01 - Fundamentos|HTML]], porque Markdown se convierte a él, pero no es obligatorio.

## 2. Concepto fundamental

Markdown tiene dos partes:

1. **El texto que escribes** (con símbolos como `#`, `*`, `-`).
2. **El resultado que se ve** cuando un programa lo "renderiza" (lo interpreta y lo muestra con formato).

```markdown
# Hola
Esto es **importante**.
```

Se convierte en:

```html
<h1>Hola</h1>
<p>Esto es <strong>importante</strong>.</p>
```

La gran ventaja es que el texto sigue siendo **legible aunque no se renderice**. Un archivo `.md` se entiende bien incluso abierto en el Bloc de notas.

## 3. Historia y "sabores"

Markdown lo creó **John Gruber** en 2004. Como la primera versión no definía todos los casos, cada programa lo interpretó un poco distinto. Por eso hoy existen varios "sabores" (variantes):

| Sabor | Qué es |
| --- | --- |
| **Markdown original** | La versión de Gruber, muy básica |
| **CommonMark** | Un estándar que fija las reglas exactas para evitar diferencias |
| **GFM** (GitHub Flavored Markdown) | CommonMark + tablas, tareas, tachado y más. Lo usa GitHub |
| **Obsidian** | Basado en CommonMark + GFM, con extras propios (wikilinks, callouts...) |

> [!tip] Idea clave
> La sintaxis básica (notas 02 a 07) funciona igual en casi todos los sitios. Lo que cambia son los **extras**.

## 4. Dónde se usa

- Documentación de proyectos (`README.md`).
- Notas personales (Obsidian, Notion, Joplin).
- Blogs y webs estáticas.
- Chats y foros (Discord, Reddit, Slack usan variantes).
- Documentación técnica.

## 5. Reglas generales

**Líneas en blanco.** Para separar párrafos, listas, tablas o bloques, deja una línea vacía entre ellos. Es lo que más evita problemas.

**Escapar símbolos.** Si quieres escribir un símbolo especial tal cual (sin que haga formato), pon una barra invertida delante:

```markdown
\*esto no es cursiva\*
```

Resultado: \*esto no es cursiva\*

Se pueden escapar: `\ ` ` * _ { } [ ] ( ) # + - . ! |`

**HTML dentro de Markdown.** Si Markdown no tiene lo que necesitas, puedes escribir HTML directamente y se respeta:

```markdown
Texto con <mark>resaltado</mark> y <sub>subíndice</sub>.
```

**Comentarios.** Para dejar notas que no se ven en el resultado:

```markdown
<!-- Esto no aparece en la vista renderizada -->
```

(En Obsidian también puedes usar `%% comentario %%`, mira [[Markdown 08 - Markdown en Obsidian]].)

## 6. Buenas prácticas

- Usa siempre una sola forma de escribir cada cosa (por ejemplo, siempre `-` para listas).
- Deja líneas en blanco entre bloques.
- No dependas de extras si el archivo se va a ver en sitios distintos.

## 7. Diferencias importantes

| Markdown | HTML |
| --- | --- |
| Rápido de escribir | Más largo |
| Pocas opciones de formato | Control total |
| Legible sin renderizar | Difícil de leer en bruto |
| Ideal para texto | Ideal para estructura y diseño de webs |

## 8. Casos especiales

- **Saltos de línea:** un solo Enter dentro de un párrafo **no** crea una línea nueva en todos los sitios. Mira [[Markdown 02 - Encabezados y texto]].
- **Espacios al final de línea:** en algunos editores se borran solos, y eso puede romper saltos de línea.
- **Archivos sin renderizar:** en un editor normal verás los símbolos; es lo esperado.

## 9. Resumen

- Markdown = texto plano + símbolos sencillos → texto con formato.
- Se guarda en archivos `.md`.
- Se convierte normalmente a HTML.
- Hay varios sabores: CommonMark, GFM y Obsidian son los importantes.
- La sintaxis básica es casi universal; los extras dependen del programa.