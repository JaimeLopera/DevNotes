# 07 - Errores de Node.js y npm

> [!info] ¿Qué es?
> Los errores más frecuentes al trabajar con **Node.js**, **npm** y **Express**, con el **mensaje literal**, qué significa y cómo solucionarlo. Los códigos en mayúsculas (`EADDRINUSE`, `ENOENT`, `ERESOLVE`…) son la parte clave para buscar.

---

## 1. Antes de empezar: la solución universal de dependencias

Muchos problemas raros de npm se arreglan reinstalando desde cero:

```bash
rm -rf node_modules
npm ci           # instala exactamente lo del package-lock.json
```

Si el `package-lock.json` está corrupto o desactualizado: bórralo también y usa `npm install`.

---

## 2. Módulos

### 2.1 `Error: Cannot find module 'express'`

**Causas típicas:**
- No has ejecutado `npm install` (por ejemplo, tras clonar el proyecto).
- El paquete no está en `package.json`.
- Estás ejecutando `node` desde otra carpeta.

**Solución:** `npm install` (o `npm i express`) en la carpeta del proyecto.

### 2.2 `Error: Cannot find module '/ruta/proyecto/index.js'`

**Qué significa:** el archivo que intentas ejecutar no existe en esa ruta.

**Solución:** comprueba el nombre y la carpeta (`ls`), o el campo `main` / el script `start` de `package.json`.

### 2.3 `Error [ERR_MODULE_NOT_FOUND]: Cannot find module './utils'`

**Qué significa:** en ES Modules, las importaciones relativas **necesitan la extensión**.

**Solución:** `import { x } from "./utils.js";`.

### 2.4 `SyntaxError: Cannot use import statement outside a module`

**Qué significa:** el archivo se ejecuta como CommonJS pero usa `import`.

**Solución:** añade `"type": "module"` a `package.json` (o renombra a `.mjs`).

### 2.5 `ReferenceError: require is not defined in ES module scope`

**Qué significa:** usas `require` en un archivo ES Module.

**Solución:** cámbialo por `import`. Para `__dirname`, usa `import.meta.dirname`.

### 2.6 `SyntaxError: The requested module 'x' does not provide an export named 'y'`

**Causas típicas:** el módulo exporta por defecto y lo importas con llaves (o al revés), o es un paquete CommonJS.

**Solución:**

```javascript
import paquete from "paquete";   // en lugar de import { y } from "paquete"
const { y } = paquete;
```

Más en [[Node 03 - Módulos CommonJS y ESM|Módulos CommonJS y ESM]].

---

## 3. npm

### 3.1 `npm ERR! code ERESOLVE` / `unable to resolve dependency tree`

**Qué significa:** dos paquetes exigen versiones incompatibles de un tercero (*peer dependencies*).

**Solución:**
1. Actualiza el paquete que causa el conflicto a una versión compatible.
2. Como último recurso: `npm install --legacy-peer-deps`.

### 3.2 `npm ERR! code ENOENT ... package.json`

**Qué significa:** no hay `package.json` en la carpeta actual.

**Solución:** muévete a la carpeta del proyecto, o créalo con `npm init -y`.

### 3.3 `npm ERR! Missing script: "dev"`

**Qué significa:** ese script no existe en `package.json`.

**Solución:** ejecuta `npm run` para ver los scripts disponibles.

### 3.4 `npm ERR! code EACCES` (al instalar con `-g`)

**Qué significa:** no tienes permisos sobre la carpeta global de npm.

**Solución:** no uses `sudo npm`; usa `npx` o instala Node con un gestor de versiones (nvm, fnm), que no requiere permisos de administrador.

### 3.5 `npm ERR! code EINTEGRITY`

**Solución:** `npm cache clean --force`, borra `node_modules` y `package-lock.json` y reinstala.

### 3.6 `'vite' no se reconoce como un comando` / `vite: command not found`

**Qué significa:** la herramienta está en `node_modules/.bin`, no en el sistema.

**Solución:** ejecútala con `npx vite` o mediante un script (`npm run dev`); si falta, `npm install`.

### 3.7 `npm WARN deprecated ...`

**Qué significa:** un paquete (a menudo una subdependencia) está obsoleto. Es un **aviso**, no un error; revisa si hay una versión nueva del paquete principal.

---

## 4. Servidor y red

### 4.1 `Error: listen EADDRINUSE: address already in use :::3000`

**Qué significa:** otro proceso (a menudo otra instancia de tu servidor) ya usa ese puerto.

**Solución:**

```bash
lsof -i :3000            # ver el proceso (Linux/macOS)
kill PID
```

En Windows: `netstat -ano | findstr :3000` y `taskkill /PID número /F`. O cambia de puerto.

Más en [[Terminal 05 - Procesos y red|Procesos y red]].

### 4.2 `Error: connect ECONNREFUSED 127.0.0.1:5432`

**Qué significa:** intentas conectarte a un servicio (base de datos, otra API) que no está escuchando.

**Causas típicas:** el servicio está apagado, el puerto es otro, o dentro de Docker usas `localhost` en lugar del nombre del servicio.

### 4.3 `Error: getaddrinfo ENOTFOUND api.ejemplo.com`

**Qué significa:** el dominio no existe o no hay conexión a internet / DNS.

### 4.4 `TypeError: fetch failed`

**Causas típicas:** sin conexión, URL incorrecta, certificado no válido o el servidor no responde. Mira `error.cause` para el detalle.

---

## 5. Express

### 5.1 `Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client`

**Qué significa:** respondes dos veces a la misma petición.

**Solución:** añade `return` al responder antes de tiempo.

```javascript
if (!producto) return res.status(404).json({ error: "No encontrado" });
res.json(producto);
```

### 5.2 `req.body` es `undefined`

**Causas típicas:**
- Falta `app.use(express.json())` (o está **después** de las rutas).
- El cliente no envía `Content-Type: application/json`.

### 5.3 `Cannot GET /ruta` (404)

**Causas típicas:** la ruta o el método no coinciden, falta montar el router (`app.use("/api", router)`), o el prefijo es distinto.

### 5.4 La petición se queda colgada (nunca responde)

**Causa típica:** un middleware no llama a `next()` ni responde, o una ruta olvida `res.json()`/`res.send()`.

### 5.5 Error de CORS desde el frontend

Ver [[Errores 09 - Red y CORS]].

---

## 6. Ejecución

### 6.1 `ReferenceError: document is not defined` / `window is not defined`

**Qué significa:** ejecutas en Node código del navegador. Node no tiene DOM.

### 6.2 `UnhandledPromiseRejection` / el proceso se cierra solo

**Qué significa:** una promesa falló sin `try/catch`; en Node moderno esto **termina el proceso**.

**Solución:** captura los errores en cada `await` o con un manejador de errores de Express.

### 6.3 `FATAL ERROR: ... JavaScript heap out of memory`

**Causas típicas:** bucle infinito, cargar demasiados datos en memoria, fuga de memoria.

**Solución:** procesa por lotes o con streams; solo si es necesario, `node --max-old-space-size=4096 app.js`.

### 6.4 `process.env.X` es `undefined`

**Solución:** carga el `.env` (`node --env-file=.env`) y comprueba el nombre. Más en [[Node 05 - Variables de entorno|Variables de entorno]].

### 6.5 Versión de Node incompatible (`The engine "node" is incompatible`, sintaxis no soportada)

**Solución:** comprueba `node --version` y el campo `engines` del proyecto; cambia de versión con nvm/fnm.

---

## 7. Resumen

- `Cannot find module` → `npm install`, carpeta correcta, extensión `.js` en ESM.
- `import`/`require` mezclados → revisa `"type": "module"`.
- `ERESOLVE` → actualiza el paquete en conflicto; último recurso `--legacy-peer-deps`.
- `EADDRINUSE` → puerto ocupado: mata el proceso o cambia de puerto.
- `ECONNREFUSED` → el servicio no está arrancado o el host/puerto es otro.
- `headers already sent` → `return` tras responder.
- `req.body` undefined → `express.json()` antes de las rutas.
- Ante la duda: `rm -rf node_modules && npm ci`.
