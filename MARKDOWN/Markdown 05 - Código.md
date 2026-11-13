# 05 - Código

> [!info] ¿Qué es?
> Markdown permite mostrar código dentro del texto, sin que se interprete como formato. Hay dos formas: **en línea** (dentro de una frase) y **en bloque** (varias líneas).

## 1. Código en línea

Se escribe entre comillas invertidas (tecla de acento grave, \`).

`````markdown
Usa el comando `git status` para ver los cambios.
`````

Sirve para nombres de archivos, comandos, variables, funciones, etc.

**Si el código lleva una comilla invertida dentro**, usa dos para abrir y cerrar:

`````markdown
`` mi `código` ``
`````

## 2. Bloques de código

Se escriben con **tres comillas invertidas** antes y después. Después de las primeras, puedes indicar el **lenguaje** para que se coloree.

`````markdown
````javascript
const nombre = "Jaime";
console.log(nombre);
````
`````

Resultado: el código sale en una caja, respetando espacios y sangrías, y con colores si el lenguaje está indicado.

## 3. Nombres de lenguaje habituales

| Lenguaje | Nombre |
| --- | --- |
| JavaScript | `javascript` o `js` |
| TypeScript | `typescript` o `ts` |
| HTML | `html` |
| CSS | `css` |
| Python | `python` o `py` |
| PHP | `php` |
| SQL | `sql` |
| Terminal | `bash` o `shell` |
| JSON | `json` |
| Markdown | `markdown` o `md` |
| Sin color | `text` |

## 4. Mostrar cambios con `diff`

Con el lenguaje `diff`, las líneas con `+` salen en verde y las de `-` en rojo:

`````markdown
````diff
- const x = 1;
+ const x = 2;
````
`````

## 5. Mostrar código que contiene tres comillas invertidas

Si tu código ya lleva ``` dentro, el bloque de fuera debe usar **más comillas** (4 o más) o usar tildes `~~~`.

`````markdown
````markdown
```js
console.log("hola");
```
````
`````

> [!tip] Regla
> El bloque que está fuera siempre debe tener **más** comillas que el de dentro. Así se escribió esta misma nota.

## 6. Bloque con sangría (forma antigua)

> [!warning] Obsoleto / legado
> También se puede crear un bloque sangrando cada línea con **4 espacios**:
>
> ```markdown
>     const x = 1;
> ```
>
> No permite indicar el lenguaje y se confunde con las listas. **Alternativa actual:** usa bloques con ``` y el lenguaje.

## 7. Buenas prácticas

- Indica **siempre** el lenguaje.
- Usa código en línea para nombres técnicos dentro de frases.
- No pongas bloques gigantes; si es muy largo, enlaza al archivo.
- Deja una línea en blanco antes y después del bloque.

## 8. Casos especiales

- **Lenguaje desconocido:** el bloque sale sin colores, pero se ve bien.
- **Código dentro de una lista:** sangra también las comillas del bloque con la misma cantidad que el elemento.
- **Diagramas:** en Obsidian y GitHub, un bloque con el lenguaje `mermaid` se dibuja como diagrama.

## 9. Resumen

- En línea: `` `código` ``.
- Bloque: tres comillas invertidas + lenguaje.
- Para escribir bloques dentro de bloques, usa más comillas fuera.
- La forma con sangría de 4 espacios es legado.