# 04 - Enlaces e imágenes

> [!info] ¿Qué es?
> Los enlaces llevan a otras páginas, archivos o partes de la misma página. Las imágenes insertan una foto o dibujo. Su sintaxis es casi igual; la imagen lleva un `!` delante.

## 1. Enlaces básicos

Formato: `[texto visible](dirección)`

`````markdown
[Mi GitHub](https://github.com)
`````

Con título (aparece al pasar el ratón):

`````markdown
[Mi GitHub](https://github.com "Ir a GitHub")
`````

## 2. Tipos de dirección

| Tipo | Ejemplo |
| --- | --- |
| Web completa | `[Web](https://ejemplo.com)` |
| Archivo del proyecto | `[Guía](docs/guia.md)` |
| Apartado de la misma nota | `[Ir a Resumen](#resumen)` |
| Correo | `[Escríbeme](mailto:correo@ejemplo.com)` |

Para ir a un apartado (**ancla**), usa `#` y el título en **minúsculas**, con **guiones en lugar de espacios** y sin acentos en algunos sitios. Ejemplo: `## Casos especiales` → `#casos-especiales`.

## 3. Enlaces automáticos

Si pones la dirección entre `<` y `>`, se convierte en enlace solo:

`````markdown
<https://github.com>
`````

En GitHub, incluso una URL pegada sin nada se convierte en enlace.

## 4. Enlaces por referencia

Sirven para no repetir direcciones largas. Pones una **etiqueta** en el texto y defines la dirección en otro sitio (normalmente al final).

`````markdown
Visita [GitHub][gh] o [MDN][mdn].

[gh]: https://github.com
[mdn]: https://developer.mozilla.org
`````

## 5. Imágenes

Formato: `![texto alternativo](ruta "título opcional")`

`````markdown
![Logo de la empresa](img/logo.png)
`````

- El **texto alternativo** describe la imagen. Se muestra si no carga y lo leen los lectores de pantalla (mira [[HTML 08 - Accesibilidad|accesibilidad en HTML]]).
- La ruta puede ser una URL completa o una ruta relativa.

**Imagen que es también un enlace:**

`````markdown
[![Logo](img/logo.png)](https://ejemplo.com)
`````

## 6. Tamaño de imagen

Markdown estándar **no permite** cambiar el tamaño. Opciones:

`````markdown
<img src="img/logo.png" alt="Logo" width="200">
`````

En Obsidian es más fácil: `![[logo.png|200]]` o `![Logo|200](logo.png)` (mira [[Markdown 08 - Markdown en Obsidian]]).

## 7. Buenas prácticas

- Escribe textos de enlace que se entiendan solos: `[Descargar la guía]`, no `[haz clic aquí]`.
- Pon **siempre** texto alternativo en las imágenes.
- Usa rutas relativas dentro del proyecto para que no se rompan al moverlo.
- Evita espacios y acentos en nombres de archivos de imágenes.

## 8. Casos especiales

- **Espacios en la ruta:** escribe `%20` (`mi%20foto.png`) o pon la ruta entre `<>`: `![foto](<mi foto.png>)`.
- **Paréntesis en la URL:** escápalos con `\(` y `\)` o cámbialos por `%28` y `%29`.
- **Enlace que no funciona:** revisa que no haya espacio entre `]` y `(`.

## 9. Resumen

- Enlace: `[texto](url)`.
- Imagen: `![alternativo](ruta)`.
- Ancla interna: `[texto](#titulo-en-minusculas)`.
- Referencias: `[texto][etiqueta]` + `[etiqueta]: url`.
- Para tamaño en Markdown estándar, usa `<img>` con HTML.