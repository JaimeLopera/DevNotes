# 05 - Variables de entorno

> [!info] ¿Qué es?
> Las **variables de entorno** son valores de configuración que se pasan a un programa **desde fuera del código**: el puerto, la URL de la base de datos, claves de APIs. Permiten que el mismo código funcione en desarrollo y en producción, y que los **secretos no acaben en Git**.

---

## 1. Antes de empezar

- Conviene haber leído [[Node 01 - Fundamentos]] (el objeto `process`).
- Para entender por qué los secretos no van en el código: [[Conceptos 05 - Seguridad web]].

---

## 2. Concepto fundamental

```
                 ┌─────────────┐
 .env (local) ──►│             │
                 │ process.env │──► tu código lee process.env.PORT
 servidor     ──►│             │
 (producción)    └─────────────┘
```

- En **desarrollo**, los valores se guardan en un archivo `.env` que **no se sube a Git**.
- En **producción**, se configuran en el servidor o en el panel del hosting.
- El código **siempre** los lee igual: `process.env.NOMBRE`.

---

## 3. Sintaxis / estructura

Archivo `.env` en la raíz del proyecto:

```bash
PORT=3000
NODE_ENV=development
DATABASE_URL=postgres://usuario:clave@localhost:5432/mi_db
API_KEY=sk_test_123456
```

Cargarlo (Node 20.6+ lo hace sin librerías):

```bash
node --env-file=.env index.js
```

Leerlo en el código:

```javascript
const puerto = process.env.PORT ?? 3000;
const apiKey = process.env.API_KEY;
```

---

## 4. Elementos / propiedades / características

### 4.1 Reglas del archivo `.env`

| Regla | Ejemplo |
|---|---|
| Una variable por línea | `PORT=3000` |
| Sin espacios alrededor del `=` | `CLAVE=valor` |
| Nombres en MAYÚSCULAS con `_` | `DATABASE_URL` |
| Comentarios con `#` | `# Configuración de la BD` |
| Comillas si hay espacios | `NOMBRE_APP="Mi tienda"` |

### 4.2 Formas de cargar el `.env`

| Forma | Cómo |
|---|---|
| Node 20.6+ | `node --env-file=.env index.js` |
| Paquete `dotenv` | `npm i dotenv` + `import "dotenv/config";` al principio |
| En la terminal | `PORT=4000 node index.js` (Linux/macOS/Git Bash) |
| Herramientas | Vite, Next.js, Docker Compose las cargan automáticamente |

### 4.3 Archivos habituales

| Archivo | Para qué | ¿Se sube a Git? |
|---|---|---|
| `.env` | Tus valores locales | **No** |
| `.env.example` | Plantilla con los nombres, sin valores reales | **Sí** |
| `.env.local`, `.env.production` | Variantes por entorno | No |

### 4.4 `NODE_ENV`

Variable convencional que indica el entorno:

| Valor | Significa |
|---|---|
| `development` | Desarrollando en local |
| `production` | En el servidor real (muchas librerías se optimizan) |
| `test` | Ejecutando tests |

### 4.5 Todo es texto

Las variables de entorno **siempre son strings**:

```javascript
const puerto = Number(process.env.PORT);           // "3000" → 3000
const debug = process.env.DEBUG === "true";        // "true" → true
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```bash
# .env
SALUDO=Hola
```

```javascript
// index.js
console.log(process.env.SALUDO);   // Hola
```

```bash
node --env-file=.env index.js
```

### 5.2 Ejemplo habitual: configuración centralizada y validada

```javascript
// src/config.js
const requeridas = ["DATABASE_URL", "JWT_SECRET"];

for (const nombre of requeridas) {
  if (!process.env[nombre]) {
    console.error(`Falta la variable de entorno ${nombre}`);
    process.exit(1);
  }
}

export const config = {
  puerto: Number(process.env.PORT ?? 3000),
  entorno: process.env.NODE_ENV ?? "development",
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
};
```

```javascript
// src/index.js
import { config } from "./config.js";
app.listen(config.puerto);
```

```bash
# .env.example (se sube a Git)
PORT=3000
DATABASE_URL=
JWT_SECRET=
```

### 5.3 Script en `package.json`

```json
"scripts": {
  "dev": "node --env-file=.env --watch src/index.js"
}
```

---

## 6. Buenas prácticas

- **Añade `.env` a `.gitignore`** antes del primer commit (ver [[Git 07 - .gitignore y configuración|.gitignore]]).
- **Sube un `.env.example`** para que otros sepan qué variables hacen falta.
- **Valida las variables al arrancar**: mejor un error claro al principio que un fallo raro más tarde.
- **Centraliza la lectura** en un `config.js`, no repartas `process.env` por todo el código.
- **Nunca expongas secretos al navegador**: en el frontend todo es visible.
- **Si un secreto se filtra, cámbialo** (rótalo); borrarlo del repositorio no basta.

---

## 7. Diferencias importantes

### 7.1 Variables de entorno en el backend vs. en el frontend

| | Backend (Node) | Frontend (Vite, React…) |
|---|---|---|
| Dónde se leen | En el servidor, en tiempo de ejecución | Se **incrustan** en el código al compilar |
| ¿Las ve el usuario? | No | **Sí**, cualquiera puede leerlas |
| Prefijo | Ninguno | `VITE_`, `NEXT_PUBLIC_`… |
| Secretos | Sí | **Nunca** |

### 7.2 `--env-file` vs. `dotenv`

Hacen lo mismo. `--env-file` viene incluido en Node moderno; `dotenv` funciona en cualquier versión y lo verás en muchos proyectos.

### 7.3 `??` vs. `||` para valores por defecto

```javascript
process.env.PORT ?? 3000   // usa 3000 solo si no existe
process.env.PORT || 3000   // usa 3000 también si es "" (cadena vacía)
```

---

## 8. Casos especiales

### 8.1 La variable sale `undefined`

- El `.env` no se ha cargado (falta `--env-file` o `import "dotenv/config"`).
- El `.env` no está en la carpeta desde la que ejecutas Node.
- Hay un error de escritura en el nombre.

### 8.2 Variables en Docker

```yaml
# docker-compose.yml
services:
  api:
    env_file: .env
```

Ver [[Docker 05 - Docker Compose|Docker Compose]].

### 8.3 Variables en el hosting

Servicios como Vercel, Render o Railway tienen un apartado *Environment Variables* en su panel: ahí se ponen los valores de producción.

### 8.4 En Python y PHP

La idea es la misma: Python usa `os.environ` y `python-dotenv` (ver [[Python 17 - Entornos y paquetes]]); PHP usa `getenv()` o `$_ENV`.

---

## 9. Resumen

- Las variables de entorno son configuración externa al código.
- En local, en un `.env` que **nunca** se sube; en producción, en el servidor.
- Se leen con `process.env.NOMBRE`; siempre son **texto**.
- Cárgalas con `node --env-file=.env` o con `dotenv`.
- Sube un `.env.example` y valida las variables al arrancar.
- En el frontend no hay secretos: todo es visible.
