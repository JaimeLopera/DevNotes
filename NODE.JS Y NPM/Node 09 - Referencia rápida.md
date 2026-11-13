# 09 - Referencia rápida

> [!info] ¿Qué es?
> Chuleta de **Node.js y npm** para consultar rápido. No explica, solo recuerda. Si algo no te suena, sigue el enlace a su nota.

---

## 1. Node ([[Node 01 - Fundamentos]])

```bash
node --version
node archivo.js
node --watch archivo.js        # reinicia al guardar
node --env-file=.env app.js    # carga variables de entorno
node -e "console.log(1+1)"
```

```javascript
process.argv        // argumentos
process.env.X       // variables de entorno
process.cwd()       // carpeta actual
process.exit(1)     // salir con error
```

---

## 2. npm ([[Node 02 - npm y package.json]])

```bash
npm init -y
npm i paquete            # dependencia
npm i -D paquete         # desarrollo
npm i paquete@1.2.3      # versión concreta
npm uninstall paquete
npm install              # instalar todo
npm ci                   # instalación exacta (CI, Docker)
npm run script  |  npm start  |  npm test
npm outdated  |  npm update  |  npm audit
npx paquete              # ejecutar sin instalar
```

| Rango | Permite |
|---|---|
| `^1.2.3` | `1.x.x` |
| `~1.2.3` | `1.2.x` |
| `1.2.3` | Exacta |

---

## 3. Módulos ([[Node 03 - Módulos CommonJS y ESM]])

```javascript
// package.json → "type": "module"
export function f() {}
export default clase;
import def, { f } from "./archivo.js";   // con extensión
import fs from "node:fs/promises";
const mod = await import("./x.js");
import.meta.dirname                       // = __dirname
```

| CommonJS | ESM |
|---|---|
| `require("x")` | `import x from "x"` |
| `module.exports = x` | `export default x` |

---

## 4. Express ([[Node 04 - Servidor con Express]] · [[Node 08 - API REST con Express]])

```javascript
import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static("public"));

app.get("/api/x/:id", (req, res) => {
  req.params.id;  req.query.q;  req.body;
  res.status(200).json({ ok: true });
});

app.use((req, res) => res.status(404).json({ error: "No encontrado" }));
app.use((err, req, res, next) => res.status(500).json({ error: "Error" }));

app.listen(3000);
```

| Código | Uso |
|---|---|
| 200 | OK |
| 201 | Creado |
| 204 | Sin contenido (borrado) |
| 400 | Datos inválidos |
| 401 | No autenticado |
| 403 | Sin permiso |
| 404 | No existe |
| 500 | Error del servidor |

---

## 5. Variables de entorno ([[Node 05 - Variables de entorno]])

```bash
# .env  (en .gitignore)
PORT=3000
DATABASE_URL=...
```

```javascript
const puerto = Number(process.env.PORT ?? 3000);
```

---

## 6. Archivos ([[Node 06 - Sistema de archivos y path]])

```javascript
import fs from "node:fs/promises";
import path from "node:path";

await fs.readFile(ruta, "utf-8");
await fs.writeFile(ruta, datos);
await fs.appendFile(ruta, datos);
await fs.mkdir(ruta, { recursive: true });
await fs.readdir(ruta);
await fs.rm(ruta, { recursive: true, force: true });

path.join(import.meta.dirname, "datos.json");
path.extname("a.png");   // .png
path.basename("/a/b.png"); // b.png
```

---

## 7. Asincronía ([[Node 07 - Asincronía y event loop en Node]])

```javascript
const [a, b] = await Promise.all([tarea1(), tarea2()]);   // paralelo
for (const x of lista) { await procesar(x); }            // secuencial
try { await algo(); } catch (e) { console.error(e); }

import { setTimeout as esperar } from "node:timers/promises";
await esperar(1000);
```

Orden: síncrono → promesas → `setTimeout` → E/S.
