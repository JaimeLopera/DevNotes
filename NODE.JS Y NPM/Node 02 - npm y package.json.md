# 02 - npm y package.json

> [!info] ¿Qué es?
> **npm** (*Node Package Manager*) es la herramienta que **instala librerías** de terceros (paquetes) y **ejecuta tareas** del proyecto. El archivo **`package.json`** es la ficha del proyecto: su nombre, sus dependencias y sus scripts.

---

## 1. Antes de empezar

Conviene haber leído [[Node 01 - Fundamentos]] y tener Node instalado (npm viene incluido).

---

## 2. Concepto fundamental

```
package.json        → qué paquetes necesita el proyecto (rangos de versión)
package-lock.json   → qué versión EXACTA se instaló de cada paquete
node_modules/       → el código de los paquetes descargados
```

Flujo:

1. `npm install express` descarga Express en `node_modules/` y lo anota en `package.json`.
2. Subes a Git `package.json` y `package-lock.json`, **pero no** `node_modules/`.
3. Otra persona clona el proyecto y ejecuta `npm install`: npm reconstruye `node_modules/` con las mismas versiones.

---

## 3. Sintaxis / estructura

```bash
npm init -y                 # crea package.json con valores por defecto
npm install paquete         # instala una dependencia
npm install -D paquete      # instala una dependencia de desarrollo
npm install                 # instala todo lo que diga package.json
npm run nombre-script       # ejecuta un script
```

`package.json` típico:

```json
{
  "name": "mi-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "node src/index.js",
    "test": "vitest"
  },
  "dependencies": {
    "express": "^4.19.2"
  },
  "devDependencies": {
    "vitest": "^2.1.0"
  }
}
```

---

## 4. Elementos / propiedades / características

### 4.1 Campos principales de `package.json`

| Campo | Para qué |
|---|---|
| `name` | Nombre del proyecto (minúsculas, sin espacios) |
| `version` | Versión del proyecto |
| `type` | `"module"` para usar `import`/`export` (ver [[Node 03 - Módulos CommonJS y ESM]]) |
| `main` | Archivo principal |
| `scripts` | Comandos del proyecto |
| `dependencies` | Paquetes necesarios para **funcionar** |
| `devDependencies` | Paquetes necesarios solo para **desarrollar** |
| `engines` | Versión de Node requerida |

### 4.2 Comandos de instalación

| Comando | Qué hace |
|---|---|
| `npm install` / `npm i` | Instala todas las dependencias |
| `npm i paquete` | Añade una dependencia |
| `npm i -D paquete` | Añade una dependencia de desarrollo |
| `npm i paquete@1.2.3` | Instala una versión concreta |
| `npm uninstall paquete` | La elimina |
| `npm ci` | Instalación limpia y exacta desde el lock (servidores, CI, Docker) |
| `npm update` | Actualiza dentro de los rangos permitidos |
| `npm outdated` | Lista paquetes con versiones nuevas |
| `npm ls` | Árbol de dependencias instaladas |

### 4.3 Scripts

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "start": "node server.js",
  "test": "vitest",
  "lint": "eslint ."
}
```

| Comando | Qué ejecuta |
|---|---|
| `npm run dev` | El script `dev` |
| `npm start` | El script `start` (no necesita `run`) |
| `npm test` | El script `test` (no necesita `run`) |
| `npm run` | Lista todos los scripts |

Dentro de los scripts se pueden usar directamente los programas de `node_modules/.bin` (como `vite` o `eslint`) sin instalarlos globalmente.

### 4.4 Versiones semánticas (SemVer)

Formato `MAYOR.MENOR.PARCHE` (por ejemplo `4.19.2`):

| Símbolo | Ejemplo | Permite instalar |
|---|---|---|
| `^` | `^4.19.2` | Cambios menores y parches: `4.x.x` (por defecto) |
| `~` | `~4.19.2` | Solo parches: `4.19.x` |
| (nada) | `4.19.2` | Exactamente esa versión |
| `*` / `latest` | | Cualquiera (evítalo) |

### 4.5 `npx`: ejecutar sin instalar

```bash
npx create-vite@latest mi-app    # ejecuta un paquete sin instalarlo
npx eslint .                     # ejecuta el eslint del proyecto
```

### 4.6 Seguridad

```bash
npm audit          # busca vulnerabilidades conocidas
npm audit fix      # intenta corregirlas
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: proyecto nuevo

```bash
mkdir mi-proyecto && cd mi-proyecto
npm init -y
npm i dayjs
```

```javascript
// index.js
import dayjs from "dayjs";
console.log(dayjs().format("YYYY-MM-DD"));
```

(Añade `"type": "module"` a `package.json` para usar `import`.)

### 5.2 Ejemplo habitual: clonar un proyecto y arrancarlo

```bash
git clone git@github.com:usuario/proyecto.git
cd proyecto
npm install
npm run dev
```

### 5.3 Crear un proyecto de frontend con Vite

```bash
npm create vite@latest mi-web
cd mi-web
npm install
npm run dev
```

---

## 6. Buenas prácticas

- **Sube `package-lock.json` a Git**; **nunca** `node_modules/`.
- **Usa `npm ci`** en servidores, Docker y CI: instalación reproducible.
- **Separa `dependencies` y `devDependencies`** correctamente.
- **Revisa un paquete antes de instalarlo**: descargas semanales, mantenimiento, tamaño. Cada dependencia es código ajeno en tu proyecto.
- **Evita instalar globalmente** (`-g`); usa `npx` o scripts.
- **Ejecuta `npm audit`** de vez en cuando.
- **Define los comandos del proyecto como scripts** para que todo el equipo use los mismos.

---

## 7. Diferencias importantes

### 7.1 `dependencies` vs. `devDependencies`

| | `dependencies` | `devDependencies` |
|---|---|---|
| Se necesita | Para que la app **funcione** | Solo para **desarrollar** |
| Ejemplos | `express`, `react`, `dayjs` | `vitest`, `eslint`, `vite`, `typescript` |
| Instalar | `npm i paquete` | `npm i -D paquete` |

### 7.2 `npm install` vs. `npm ci`

| | `npm install` | `npm ci` |
|---|---|---|
| Puede modificar el lock | Sí | No (falla si no coincide) |
| Borra `node_modules` antes | No | Sí |
| Uso | Desarrollo | Servidores, Docker, CI |

### 7.3 `npm` vs. `npx`

`npm` **instala** paquetes. `npx` **ejecuta** un paquete (lo descarga temporalmente si no lo tienes).

### 7.4 npm vs. pnpm vs. yarn

Son gestores de paquetes alternativos que usan el mismo `package.json`. **pnpm** es más rápido y ahorra espacio. No mezcles gestores en un mismo proyecto (cada uno tiene su archivo de lock).

---

## 8. Casos especiales

### 8.1 Algo raro con las dependencias

La solución universal:

```bash
rm -rf node_modules package-lock.json
npm install
```

(Borra el lock solo si sabes que quieres regenerarlo.)

### 8.2 *ERESOLVE unable to resolve dependency tree*

Dos paquetes piden versiones incompatibles de un tercero. Actualiza el paquete que causa el conflicto o, como último recurso, `npm install --legacy-peer-deps`.

### 8.3 Variables en scripts que funcionen en Windows y Linux

`"start": "NODE_ENV=production node app.js"` no funciona en Windows. Usa el paquete `cross-env` o define la variable en un `.env` ([[Node 05 - Variables de entorno]]).

### 8.4 Ejecutar varios scripts

```json
"build": "npm run lint && npm run test && vite build"
```

---

## 9. Resumen

- `package.json` describe el proyecto: dependencias y scripts.
- `package-lock.json` fija versiones exactas: súbelo a Git.
- `node_modules/` nunca se sube: se regenera con `npm install`.
- `npm i` (dependencia), `npm i -D` (desarrollo), `npm ci` (instalación exacta).
- `npm run script`; `npm start` y `npm test` sin `run`.
- `^` permite actualizaciones menores; `~` solo parches.
- `npx` ejecuta paquetes sin instalarlos.
