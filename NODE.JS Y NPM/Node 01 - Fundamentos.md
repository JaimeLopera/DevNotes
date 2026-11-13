# 01 - Fundamentos

> [!info] ¿Qué es?
> **Node.js** es un **entorno de ejecución** de JavaScript: toma el motor que usa Chrome (V8) y lo saca del navegador para que puedas ejecutar JavaScript en tu ordenador o en un servidor, con acceso a archivos, red y sistema operativo.

---

## 1. Antes de empezar

- Hay que saber lo básico de JavaScript: [[JavaScript 01 - Fundamentos|Fundamentos de JavaScript]].
- Descarga Node desde nodejs.org: elige la versión **LTS** (la estable de largo soporte).
- Comprueba la instalación:

```bash
node --version
npm --version
```

---

## 2. Concepto fundamental

El mismo lenguaje, dos entornos distintos:

```
          JavaScript
         /          \
   Navegador          Node.js
   ─────────          ───────
   window, document   process, fs, http
   DOM, eventos       archivos, red, servidores
   fetch              fetch (desde Node 18)
```

Node no tiene `window` ni `document` (no hay página web), pero tiene módulos para leer archivos, crear servidores y acceder al sistema.

---

## 3. Sintaxis / estructura

Ejecutar un archivo:

```javascript
// hola.js
console.log("Hola desde Node");
```

```bash
node hola.js
# Hola desde Node
```

Modo interactivo (REPL):

```bash
node
> 2 + 2
4
> .exit
```

---

## 4. Elementos / propiedades / características

### 4.1 Objetos globales de Node

| Global | Para qué |
|---|---|
| `console` | Mostrar mensajes (igual que en el navegador) |
| `process` | Información del proceso: argumentos, variables de entorno, salir |
| `globalThis` | El objeto global (equivale a `window` del navegador) |
| `__dirname` / `__filename` | Carpeta y archivo actuales (solo en CommonJS) |
| `setTimeout`, `setInterval` | Temporizadores |

### 4.2 El objeto `process`

```javascript
process.argv          // argumentos de la línea de comandos
process.env.PORT      // variables de entorno
process.cwd()         // carpeta desde la que se ejecutó
process.exit(1)       // terminar con código de error
process.platform      // 'win32', 'linux', 'darwin'
```

### 4.3 Módulos incluidos (*core modules*)

| Módulo | Para qué |
|---|---|
| `fs` | Leer y escribir archivos ([[Node 06 - Sistema de archivos y path]]) |
| `path` | Trabajar con rutas |
| `http` | Crear servidores web |
| `os` | Información del sistema |
| `events` | Emitir y escuchar eventos |
| `crypto` | Hashes, cifrado, UUIDs |
| `child_process` | Ejecutar otros programas |

Se importan con el prefijo `node:`:

```javascript
import fs from "node:fs";
import path from "node:path";
```

### 4.4 Versiones de Node

| Tipo | Significado |
|---|---|
| **LTS** (números pares: 20, 22) | Estable, con soporte largo. **Usa esta.** |
| **Current** (impares) | Últimas novedades, soporte corto |

Para tener varias versiones instaladas se usa un gestor como **nvm** (o **nvm-windows**, **fnm**).

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: argumentos

```javascript
// saludo.js
const nombre = process.argv[2] ?? "mundo";
console.log(`Hola, ${nombre}`);
```

```bash
node saludo.js Ana
# Hola, Ana
```

### 5.2 Ejemplo habitual: un servidor web mínimo

```javascript
// servidor.js
import http from "node:http";

const servidor = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
  res.end("Hola desde mi servidor");
});

servidor.listen(3000, () => {
  console.log("Servidor en http://localhost:3000");
});
```

```bash
node servidor.js
```

Abre `http://localhost:3000` en el navegador. Para pararlo: `Ctrl + C`.

> [!tip] Recarga automática
> `node --watch servidor.js` reinicia el servidor cada vez que guardas un cambio.

---

## 6. Buenas prácticas

- **Usa la versión LTS**.
- **Usa el prefijo `node:`** al importar módulos propios de Node.
- **Usa `--watch`** (o `nodemon`) en desarrollo para no reiniciar a mano.
- **No bloquees el hilo principal** con operaciones síncronas largas (ver [[Node 07 - Asincronía y event loop en Node]]).
- **Termina con `process.exit(1)`** cuando un script falle, para que otros programas lo detecten.

---

## 7. Diferencias importantes

### 7.1 Navegador vs. Node.js

| | Navegador | Node.js |
|---|---|---|
| Para qué | Interfaces web | Servidores, herramientas, scripts |
| DOM (`document`) | Sí | No |
| Archivos del sistema | No | Sí (`fs`) |
| Objeto global | `window` | `globalThis` / `global` |
| Módulos | ES Modules | ES Modules y CommonJS |
| Quién controla la versión | El usuario (su navegador) | Tú |

### 7.2 Node.js vs. npm

**Node** ejecuta JavaScript. **npm** instala paquetes y ejecuta scripts. npm viene incluido al instalar Node.

### 7.3 `node archivo.js` vs. `npm start`

`node archivo.js` ejecuta ese archivo directamente. `npm start` ejecuta lo que diga el script `start` de `package.json` (ver [[Node 02 - npm y package.json]]).

---

## 8. Casos especiales

### 8.1 *document is not defined*

Estás ejecutando en Node código pensado para el navegador. El DOM no existe en Node.

### 8.2 Ejecutar código sin archivo

```bash
node -e "console.log(2 + 2)"
```

### 8.3 Leer datos del usuario en la terminal

```javascript
import readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";

const rl = readline.createInterface({ input, output });
const nombre = await rl.question("¿Cómo te llamas? ");
console.log(`Hola, ${nombre}`);
rl.close();
```

### 8.4 Alternativas a Node

**Deno** y **Bun** son entornos más recientes que también ejecutan JavaScript/TypeScript fuera del navegador. Node sigue siendo el estándar.

---

## 9. Resumen

- Node.js ejecuta JavaScript fuera del navegador (motor V8).
- `node archivo.js` ejecuta un archivo; `node` abre el modo interactivo.
- No hay DOM; a cambio hay `process`, `fs`, `http`, `path`…
- Instala siempre la versión **LTS**.
- `node --watch` reinicia al guardar.
- Siguiente paso: [[Node 02 - npm y package.json]].
