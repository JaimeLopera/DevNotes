# 03 - Módulos CommonJS y ESM

> [!info] ¿Qué es?
> Un **módulo** es un archivo que exporta código para que otros archivos lo usen. Node tiene **dos sistemas de módulos**: el antiguo, **CommonJS** (`require` / `module.exports`), y el estándar moderno, **ES Modules** o **ESM** (`import` / `export`). Hay que saber leer los dos y elegir uno.

---

## 1. Antes de empezar

- Conviene conocer los módulos en el navegador: [[JavaScript 14 - Módulos|Módulos en JavaScript]].
- Y tener un `package.json`: [[Node 02 - npm y package.json]].

---

## 2. Concepto fundamental

Los dos sistemas hacen lo mismo con distinta sintaxis:

| | CommonJS (CJS) | ES Modules (ESM) |
|---|---|---|
| Importar | `const x = require("x")` | `import x from "x"` |
| Exportar | `module.exports = x` | `export default x` |
| Estándar | Propio de Node | Oficial de JavaScript (navegador y Node) |

Node decide qué sistema usa un archivo así:

| Archivo | Sistema |
|---|---|
| `.mjs` | Siempre ESM |
| `.cjs` | Siempre CommonJS |
| `.js` con `"type": "module"` en `package.json` | ESM |
| `.js` sin `"type"` (o `"commonjs"`) | CommonJS |

---

## 3. Sintaxis / estructura

### ESM (recomendado)

```javascript
// matematicas.js
export function sumar(a, b) {
  return a + b;
}
export const PI = 3.1416;

export default function restar(a, b) {
  return a - b;
}
```

```javascript
// index.js
import restar, { sumar, PI } from "./matematicas.js";

console.log(sumar(2, 3), restar(5, 1), PI);
```

### CommonJS

```javascript
// matematicas.js
function sumar(a, b) {
  return a + b;
}
module.exports = { sumar, PI: 3.1416 };
```

```javascript
// index.js
const { sumar, PI } = require("./matematicas");
```

---

## 4. Elementos / propiedades / características

### 4.1 Formas de exportar (ESM)

| Forma | Ejemplo | Se importa con |
|---|---|---|
| Con nombre | `export function x() {}` | `import { x } from "./a.js"` |
| Por defecto | `export default x` | `import cualquierNombre from "./a.js"` |
| Al final | `export { x, y }` | `import { x, y } from "./a.js"` |
| Renombrando | `export { x as y }` | `import { y } from "./a.js"` |
| Reexportar | `export * from "./a.js"` | — |

### 4.2 Formas de importar (ESM)

```javascript
import express from "express";            // paquete de npm
import fs from "node:fs";                 // módulo de Node
import { leer } from "./utils/leer.js";   // archivo propio (extensión obligatoria)
import * as utils from "./utils.js";      // todo como objeto
import { sumar as add } from "./mat.js";  // renombrar
const modulo = await import("./plugin.js"); // importación dinámica
```

### 4.3 Rutas de importación

| Ruta | Busca en |
|---|---|
| `"express"` | `node_modules/` |
| `"node:fs"` | Módulos de Node |
| `"./archivo.js"` | Relativa al archivo actual |
| `"../carpeta/archivo.js"` | Una carpeta arriba |

### 4.4 Equivalencias que cambian en ESM

| CommonJS | ESM |
|---|---|
| `__dirname` | `import.meta.dirname` (Node 20.11+) |
| `__filename` | `import.meta.filename` |
| `require("./datos.json")` | `import datos from "./datos.json" with { type: "json" }` |
| `require` condicional | `await import()` |

### 4.5 *Top-level await*

En ESM puedes usar `await` fuera de funciones:

```javascript
const respuesta = await fetch("https://api.ejemplo.com/datos");
const datos = await respuesta.json();
```

En CommonJS no está permitido.

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: dividir en archivos

```
src/
├── index.js
└── utils/
    └── formato.js
```

```javascript
// src/utils/formato.js
export const formatearPrecio = (n) => `${n.toFixed(2)} €`;
```

```javascript
// src/index.js
import { formatearPrecio } from "./utils/formato.js";
console.log(formatearPrecio(9.5)); // 9.50 €
```

### 5.2 Ejemplo habitual: rutas de Express en archivos separados

```javascript
// src/rutas/usuarios.js
import { Router } from "express";
const router = Router();

router.get("/", (req, res) => res.json([]));

export default router;
```

```javascript
// src/index.js
import express from "express";
import usuarios from "./rutas/usuarios.js";

const app = express();
app.use("/api/usuarios", usuarios);
app.listen(3000);
```

---

## 6. Buenas prácticas

- **Usa ESM** en proyectos nuevos: añade `"type": "module"` a `package.json`.
- **Escribe la extensión** `.js` en las importaciones relativas (en ESM es obligatoria).
- **Prefiere exportaciones con nombre**: el editor autocompleta mejor y los renombres son más seguros.
- **No mezcles** `require` e `import` en el mismo archivo.
- **Un módulo, una responsabilidad**: archivos pequeños y con nombre claro.

---

## 7. Diferencias importantes

### 7.1 CommonJS vs. ESM

| | CommonJS | ESM |
|---|---|---|
| Carga | Síncrona | Asíncrona |
| `await` en el nivel superior | No | Sí |
| Extensión en rutas relativas | Opcional | Obligatoria |
| `__dirname` | Sí | No (`import.meta.dirname`) |
| Funciona en el navegador | No | Sí |
| Cuándo lo verás | Proyectos y tutoriales antiguos | Proyectos nuevos |

### 7.2 Exportación por defecto vs. con nombre

| | Por defecto | Con nombre |
|---|---|---|
| Cuántas por archivo | Una | Muchas |
| Nombre al importar | El que quieras | El exportado (o `as`) |
| Llaves al importar | No | Sí |

---

## 8. Casos especiales

### 8.1 *Cannot use import statement outside a module*

El archivo se está ejecutando como CommonJS. Añade `"type": "module"` a `package.json` o renombra el archivo a `.mjs`.

### 8.2 *require is not defined in ES module scope*

Estás usando `require` en un archivo ESM. Cámbialo por `import`.

### 8.3 *ERR_MODULE_NOT_FOUND*

Falta la extensión `.js` en una importación relativa, o la ruta está mal escrita.

### 8.4 Importar un paquete CommonJS desde ESM

Normalmente funciona con `import paquete from "paquete"`. Si no, usa:

```javascript
import { createRequire } from "node:module";
const require = createRequire(import.meta.url);
const paquete = require("paquete");
```

---

## 9. Resumen

- Node tiene dos sistemas: **CommonJS** (`require`) y **ESM** (`import`).
- ESM es el estándar: usa `"type": "module"` en `package.json`.
- En ESM, las rutas relativas llevan extensión: `"./archivo.js"`.
- `export` con nombre (muchos) o `export default` (uno).
- `__dirname` en ESM → `import.meta.dirname`.
- Los errores típicos se arreglan revisando `"type"` y las extensiones.
