# 09 - Markdown en GitHub

> [!info] ¿Qué es?
> GitHub usa **GFM** (GitHub Flavored Markdown): Markdown estándar con extras pensados para proyectos. Es lo que se ve en los `README.md`, las issues, los pull requests y los comentarios.

## 1. Antes de empezar

- Necesitas conocer la sintaxis básica de [[Markdown 02 - Encabezados y texto]] a [[Markdown 07 - Citas, separadores y notas al pie]].
- Si aún no usas GitHub, repasa la carpeta de Git (`Git/`), sobre todo la parte de remotos.

## 2. El archivo README.md

Es la "portada" de un repositorio. GitHub lo muestra automáticamente. Suele incluir:

1. Nombre y descripción del proyecto.
2. Cómo instalarlo.
3. Cómo usarlo.
4. Tecnologías usadas.
5. Autor y licencia.

Ejemplo de estructura:

`````markdown
# Nombre del proyecto

Breve descripción de qué hace.

## Instalación

​```bash
npm install
​```

## Uso

​```bash
npm start
​```

## Autor

Jaime Lopera Velázquez
`````

> [!tip] Si copias este ejemplo
> Los bloques de código del ejemplo llevan un carácter invisible delante para poder mostrarlos dentro de otro bloque. Al usarlo en tu README, escribe las tres comillas normales.

## 3. Extras de GFM

| Función | Sintaxis | Resultado |
| --- | --- | --- |
| Tachado | `~~texto~~` | ~~texto~~ |
| Tablas | `\| a \| b \|` | Ver [[Markdown 06 - Tablas]] |
| Tareas | `- [x] hecho` | Casillas marcables |
| URLs automáticas | `https://github.com` | Se convierte en enlace |
| Emojis | `:rocket:` | 🚀 |
| Mención | `@usuario` | Enlaza a un usuario |
| Issue o PR | `#123` | Enlaza a la issue 123 |
| Commit | Escribir su hash | Enlaza al commit |

## 4. Alertas

GitHub tiene su propia versión de callouts:

`````markdown
> [!NOTE]
> Información útil.

> [!TIP]
> Un consejo.

> [!IMPORTANT]
> Algo clave que no hay que olvidar.

> [!WARNING]
> Cuidado con esto.

> [!CAUTION]
> Puede causar problemas graves.
`````

Solo existen estos **5 tipos** y van en **mayúsculas**. No son plegables ni admiten título propio (a diferencia de Obsidian, mira [[Markdown 08 - Markdown en Obsidian]]).

## 5. Contenido desplegable

Con HTML:

`````markdown
<details>
<summary>Ver más</summary>

Contenido oculto. Aquí sí se puede usar **Markdown**.

</details>
`````

> [!warning] Línea en blanco
> Deja una línea en blanco después de `</summary>` o el Markdown de dentro no se interpretará.

## 6. Diagramas, fórmulas e imágenes

- **Diagramas:** bloque con el lenguaje `mermaid`.
- **Fórmulas:** `$x^2$` en línea y `$$ ... $$` en bloque.
- **Imágenes:** igual que siempre (`![alt](ruta)`). Puedes arrastrarlas al editor de GitHub y se suben solas.
- **Imagen distinta en modo claro/oscuro:** con la etiqueta `<picture>` de HTML.

## 7. Insignias (badges)

Son pequeñas imágenes con información (versión, licencia, estado). Se insertan como imágenes con enlace:

`````markdown
![Licencia](https://img.shields.io/badge/licencia-MIT-green)
`````

## 8. Buenas prácticas

- Escribe un README claro y con ejemplos.
- Usa rutas relativas para enlaces y imágenes del propio repositorio.
- No subas documentación enorme en un solo archivo: divídela en una carpeta `docs/`.
- Usa alertas poco, solo para lo importante.
- Añade una captura o GIF si el proyecto es visual.

## 9. Casos especiales

- **Wikilinks `[[ ]]`:** GitHub **no** los entiende (aparecen como texto). Si la nota va a GitHub, usa enlaces `[texto](archivo.md)`.
- **Callouts de Obsidian:** solo funcionan los 5 de GitHub.
- **Saltos de línea en comentarios:** en issues y comentarios, un Enter sí crea salto; en archivos `.md`, no.
- **Archivos especiales:** `README.md`, `CONTRIBUTING.md`, `LICENSE.md`, `CHANGELOG.md` se reconocen por su nombre.

## 10. Resumen

- GitHub usa GFM: Markdown + tablas, tareas, tachado, emojis, menciones y alertas.
- El `README.md` es la portada del proyecto.
- Alertas: `NOTE`, `TIP`, `IMPORTANT`, `WARNING`, `CAUTION`.
- No soporta wikilinks ni callouts de Obsidian.