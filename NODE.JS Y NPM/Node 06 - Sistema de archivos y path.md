# 06 - Sistema de archivos y path

> [!info] ¿Qué es?
> El módulo **`fs`** (*file system*) permite **leer, escribir, crear y borrar archivos y carpetas** desde Node. El módulo **`path`** construye rutas de forma segura en cualquier sistema operativo. Juntos sirven para scripts, generar archivos, guardar datos o procesar contenido.

---

## 1. Antes de empezar

- Conviene haber leído [[Node 03 - Módulos CommonJS y ESM]] (importar módulos de Node).
- Y entender `async`/`await`: [[JavaScript 12 - Asincronía|Asincronía en JavaScript]].

---

## 2. Concepto fundamental

`fs` tiene **tres versiones** de casi cada función:

| Versión | Ejemplo | Cómo funciona |
|---|---|---|
| **Promesas** (recomendada) | `await fs.readFile(...)` | Asíncrona, con `async`/`await` |
| **Síncrona** | `fs.readFileSync(...)` | Bloquea Node hasta terminar |
| **Callbacks** (antigua) | `fs.readFile(..., (err, data) => {})` | Asíncrona, con funciones de retorno |

```javascript
import fs from "node:fs/promises";   // versión con promesas
import path from "node:path";
```

---

## 3. Sintaxis / estructura

```javascript
import fs from "node:fs/promises";

const texto = await fs.readFile("notas.txt", "utf-8");   // leer
await fs.writeFile("salida.txt", "Hola");                 // escribir (sobrescribe)
await fs.appendFile("log.txt", "Nueva línea\n");          // añadir al final
```

---

## 4. Elementos / propiedades / características

### 4.1 Funciones principales de `fs/promises`

| Función | Qué hace |
|---|---|
| `readFile(ruta, "utf-8")` | Lee un archivo como texto |
| `writeFile(ruta, datos)` | Crea o sobrescribe |
| `appendFile(ruta, datos)` | Añade al final |
| `mkdir(ruta, { recursive: true })` | Crea carpetas (y las intermedias) |
| `readdir(ruta)` | Lista el contenido de una carpeta |
| `stat(ruta)` | Información: tamaño, fechas, si es carpeta |
| `rename(origen, destino)` | Mueve o renombra |
| `copyFile(origen, destino)` | Copia |
| `rm(ruta, { recursive: true, force: true })` | Borra archivos o carpetas |
| `access(ruta)` | Comprueba si existe (lanza error si no) |

### 4.2 Funciones principales de `path`

| Función | Ejemplo | Resultado |
|---|---|---|
| `path.join(...)` | `path.join("src", "img", "a.png")` | `src/img/a.png` |
| `path.resolve(...)` | `path.resolve("datos.json")` | Ruta absoluta |
| `path.basename(ruta)` | `path.basename("/a/b/foto.png")` | `foto.png` |
| `path.extname(ruta)` | `path.extname("foto.png")` | `.png` |
| `path.dirname(ruta)` | `path.dirname("/a/b/foto.png")` | `/a/b` |
| `path.parse(ruta)` | `path.parse("/a/foto.png")` | `{ root, dir, base, ext, name }` |

### 4.3 Rutas relativas al archivo (no a la terminal)

Una ruta relativa como `"datos.json"` se resuelve desde la carpeta **desde la que ejecutas** `node`, no desde el archivo. Para que sea relativa al archivo:

```javascript
// ESM (Node 20.11+)
const ruta = path.join(import.meta.dirname, "datos.json");

// CommonJS
const ruta = path.join(__dirname, "datos.json");
```

### 4.4 Leer y guardar JSON

```javascript
const datos = JSON.parse(await fs.readFile("datos.json", "utf-8"));
await fs.writeFile("datos.json", JSON.stringify(datos, null, 2));
```

### 4.5 Archivos grandes: *streams*

Para archivos enormes (logs, vídeos) se usan **streams**, que procesan el archivo por trozos sin cargarlo entero en memoria:

```javascript
import { createReadStream } from "node:fs";
createReadStream("enorme.log").pipe(process.stdout);
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: contador de visitas en un archivo

```javascript
import fs from "node:fs/promises";

const archivo = "visitas.txt";
let visitas = 0;

try {
  visitas = Number(await fs.readFile(archivo, "utf-8"));
} catch {
  // el archivo aún no existe: empezamos en 0
}

visitas++;
await fs.writeFile(archivo, String(visitas));
console.log(`Visita número ${visitas}`);
```

### 5.2 Ejemplo habitual: listar todos los `.md` de una carpeta

```javascript
import fs from "node:fs/promises";
import path from "node:path";

const carpeta = path.join(import.meta.dirname, "notas");
const archivos = await fs.readdir(carpeta, { recursive: true });

const markdown = archivos.filter((a) => path.extname(a) === ".md");
console.log(`Hay ${markdown.length} notas`);
```

### 5.3 Guardar datos de una API en un "almacén" JSON

```javascript
const RUTA = path.join(import.meta.dirname, "tareas.json");

export async function leerTareas() {
  try {
    return JSON.parse(await fs.readFile(RUTA, "utf-8"));
  } catch {
    return [];
  }
}

export async function guardarTareas(tareas) {
  await fs.writeFile(RUTA, JSON.stringify(tareas, null, 2));
}
```

---

## 6. Buenas prácticas

- **Usa `node:fs/promises`** con `async`/`await`.
- **Evita las funciones `Sync`** en servidores: bloquean todas las peticiones mientras leen.
- **Construye rutas con `path.join`**, nunca concatenando con `"/"` (Windows usa `\`).
- **Indica la codificación** (`"utf-8"`) al leer texto; si no, obtienes un `Buffer`.
- **Envuelve en `try/catch`**: leer un archivo que no existe lanza un error.
- **No uses archivos como base de datos** en producción: para eso está SQL ([[SQL 00 - Índice|SQL]]).
- **Nunca construyas rutas con datos del usuario** sin validarlos (riesgo de leer archivos del sistema).

---

## 7. Diferencias importantes

### 7.1 Síncrono vs. asíncrono

| | `readFileSync` | `await readFile` |
|---|---|---|
| Bloquea Node | Sí | No |
| Uso | Scripts sencillos, configuración al arrancar | Servidores y casi todo lo demás |

### 7.2 `path.join` vs. `path.resolve`

| | `path.join` | `path.resolve` |
|---|---|---|
| Resultado | Une las partes tal cual | Siempre una ruta **absoluta** |
| `("a", "b")` | `a/b` | `/carpeta/actual/a/b` |

### 7.3 `writeFile` vs. `appendFile`

`writeFile` **sobrescribe** el archivo entero. `appendFile` **añade** al final.

---

## 8. Casos especiales

### 8.1 *ENOENT: no such file or directory*

El archivo o la carpeta no existen. Revisa la ruta (¿relativa a dónde?) o crea la carpeta con `mkdir({ recursive: true })`.

### 8.2 *EACCES: permission denied*

No tienes permisos sobre ese archivo (ver [[Terminal 03 - Permisos y usuarios|Permisos y usuarios]]).

### 8.3 Comprobar si existe

```javascript
import { existsSync } from "node:fs";
if (existsSync("config.json")) { /* ... */ }
```

### 8.4 Vigilar cambios

```javascript
import { watch } from "node:fs/promises";
for await (const evento of watch("src")) {
  console.log(evento.eventType, evento.filename);
}
```

---

## 9. Resumen

- `fs` lee y escribe archivos; usa `node:fs/promises` con `await`.
- `readFile(ruta, "utf-8")`, `writeFile`, `appendFile`, `mkdir`, `readdir`, `rm`.
- `path.join` para construir rutas; `import.meta.dirname` para rutas relativas al archivo.
- JSON: `JSON.parse(await readFile(...))` y `writeFile(..., JSON.stringify(x, null, 2))`.
- Evita las funciones `Sync` en servidores.
- Archivos grandes → streams.
