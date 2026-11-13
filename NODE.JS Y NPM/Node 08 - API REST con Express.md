# 08 - API REST con Express

> [!info] ¿Qué es?
> Esta nota junta todo lo anterior para construir una **API REST completa** con Express: estructura de carpetas, rutas por recurso, validación, manejo de errores, CORS y conexión con el frontend. Es la base de casi cualquier backend en Node.

---

## 1. Antes de empezar

- [[Node 04 - Servidor con Express]] (rutas, `req`, `res`, middleware).
- [[Node 05 - Variables de entorno]] (configuración).
- [[Conceptos 02 - APIs REST]] (recursos, verbos y códigos de estado).

---

## 2. Concepto fundamental

Una API REST organiza las rutas por **recursos** y usa los **verbos HTTP** para las acciones:

| Acción | Método | Ruta | Respuesta |
|---|---|---|---|
| Listar | `GET` | `/api/productos` | `200` + lista |
| Ver uno | `GET` | `/api/productos/:id` | `200` + objeto / `404` |
| Crear | `POST` | `/api/productos` | `201` + objeto creado / `400` |
| Reemplazar | `PUT` | `/api/productos/:id` | `200` + objeto / `404` |
| Modificar | `PATCH` | `/api/productos/:id` | `200` + objeto / `404` |
| Borrar | `DELETE` | `/api/productos/:id` | `204` / `404` |

El recorrido de una petición:

```
Petición → cors → express.json() → log → router → controlador → respuesta
                                                        │
                                                   (error) → manejador de errores → 4xx/5xx
```

---

## 3. Sintaxis / estructura

Estructura de carpetas habitual:

```
mi-api/
├── .env
├── package.json
└── src/
    ├── index.js            ← arranca el servidor
    ├── app.js              ← configura Express (middleware, rutas)
    ├── config.js           ← variables de entorno
    ├── routes/
    │   └── productos.js    ← rutas del recurso
    ├── controllers/
    │   └── productos.js    ← lógica de cada ruta
    └── middleware/
        └── errores.js      ← 404 y manejador de errores
```

---

## 4. Elementos / propiedades / características

### 4.1 Middleware imprescindible

| Middleware | Para qué | Instalación |
|---|---|---|
| `express.json()` | Leer cuerpos JSON | Incluido |
| `cors()` | Permitir llamadas desde otro origen (tu frontend) | `npm i cors` |
| `helmet()` | Cabeceras de seguridad | `npm i helmet` |
| `morgan("dev")` | Log de peticiones | `npm i morgan` |

### 4.2 Manejador de errores

Express reconoce un middleware de errores porque tiene **cuatro parámetros**:

```javascript
export function manejarErrores(err, req, res, next) {
  console.error(err);
  const estado = err.status ?? 500;
  res.status(estado).json({
    error: estado === 500 ? "Error interno del servidor" : err.message,
  });
}
```

Desde Express 5, los errores lanzados en funciones `async` llegan solos a este manejador. En Express 4 hay que pasarlos con `next(error)` (o usar un `try/catch`).

### 4.3 Validación

Nunca confíes en lo que llega. Valida tipo, formato y obligatoriedad antes de usarlo. Para APIs grandes se usan librerías como **Zod**:

```javascript
import { z } from "zod";

const ProductoSchema = z.object({
  nombre: z.string().min(1),
  precio: z.number().positive(),
});

const resultado = ProductoSchema.safeParse(req.body);
if (!resultado.success) {
  return res.status(400).json({ error: resultado.error.issues });
}
```

### 4.4 Formato de respuesta coherente

```json
// Éxito
{ "id": 1, "nombre": "Camiseta", "precio": 19.99 }

// Error
{ "error": "Producto no encontrado" }
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: `app.js` e `index.js`

```javascript
// src/app.js
import express from "express";
import cors from "cors";
import productos from "./routes/productos.js";
import { noEncontrado, manejarErrores } from "./middleware/errores.js";

export const app = express();

app.use(cors({ origin: "http://localhost:5173" }));
app.use(express.json());

app.use("/api/productos", productos);

app.use(noEncontrado);      // 404 para rutas que no existen
app.use(manejarErrores);    // siempre el último
```

```javascript
// src/index.js
import { app } from "./app.js";
const PORT = process.env.PORT ?? 3000;
app.listen(PORT, () => console.log(`API en http://localhost:${PORT}`));
```

### 5.2 Ejemplo habitual: rutas y controlador

```javascript
// src/routes/productos.js
import { Router } from "express";
import * as ctrl from "../controllers/productos.js";

const router = Router();
router.get("/", ctrl.listar);
router.get("/:id", ctrl.obtener);
router.post("/", ctrl.crear);
router.delete("/:id", ctrl.borrar);
export default router;
```

```javascript
// src/controllers/productos.js
let productos = [];
let siguienteId = 1;

export function listar(req, res) {
  res.json(productos);
}

export function obtener(req, res) {
  const producto = productos.find((p) => p.id === Number(req.params.id));
  if (!producto) return res.status(404).json({ error: "Producto no encontrado" });
  res.json(producto);
}

export function crear(req, res) {
  const { nombre, precio } = req.body;
  if (typeof nombre !== "string" || typeof precio !== "number") {
    return res.status(400).json({ error: "nombre (texto) y precio (número) son obligatorios" });
  }
  const nuevo = { id: siguienteId++, nombre, precio };
  productos.push(nuevo);
  res.status(201).json(nuevo);
}

export function borrar(req, res) {
  const antes = productos.length;
  productos = productos.filter((p) => p.id !== Number(req.params.id));
  if (productos.length === antes) return res.status(404).json({ error: "Producto no encontrado" });
  res.sendStatus(204);
}
```

```javascript
// src/middleware/errores.js
export function noEncontrado(req, res) {
  res.status(404).json({ error: `Ruta ${req.method} ${req.originalUrl} no encontrada` });
}

export function manejarErrores(err, req, res, next) {
  console.error(err);
  res.status(err.status ?? 500).json({ error: "Error interno del servidor" });
}
```

### 5.3 Consumirla desde el frontend

```javascript
const respuesta = await fetch("http://localhost:3000/api/productos", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ nombre: "Taza", precio: 8.5 }),
});
const producto = await respuesta.json();
```

Ver [[JavaScript 13 - Fetch y APIs|Fetch y APIs]].

---

## 6. Buenas prácticas

- **Separa responsabilidades**: rutas, controladores, acceso a datos.
- **Separa `app.js` de `index.js`**: así puedes testear la app sin arrancar el servidor ([[Testing 05 - Tests de integración y API|Tests de API]]).
- **Valida todas las entradas** y responde `400` con un mensaje claro.
- **No muestres detalles internos** de los errores al cliente (rutas, consultas SQL, *stack traces*).
- **Configura CORS con orígenes concretos**, no `*`, en producción.
- **Prefijo `/api`** y nombres de recursos en plural.
- **Persistencia real** con una base de datos ([[SQL 00 - Índice|SQL]]), no arrays en memoria.

---

## 7. Diferencias importantes

### 7.1 Rutas vs. controladores

| | Rutas | Controladores |
|---|---|---|
| Responden a | ¿**Qué** URL y método? | ¿**Qué** hacer con la petición? |
| Contienen | `router.get("/", ...)` | La lógica y la respuesta |

### 7.2 `400` vs. `404` vs. `500`

| Código | Significa | Culpa de |
|---|---|---|
| `400 Bad Request` | Datos inválidos | El cliente |
| `404 Not Found` | El recurso no existe | El cliente (pide algo que no hay) |
| `500 Internal Server Error` | Algo falló en el servidor | El servidor |

### 7.3 `PUT` vs. `PATCH`

`PUT` **reemplaza** el recurso entero (envías todos los campos). `PATCH` **modifica** solo los campos que envías.

---

## 8. Casos especiales

### 8.1 Error de CORS en el navegador

```
Access to fetch at 'http://localhost:3000' from origin 'http://localhost:5173' has been blocked by CORS policy
```

La API no permite ese origen. Añade `app.use(cors({ origin: "http://localhost:5173" }))`. Más en [[Errores 09 - Red y CORS|Errores de red y CORS]].

### 8.2 Paginación

```
GET /api/productos?pagina=2&limite=20
```

```javascript
const pagina = Number(req.query.pagina ?? 1);
const limite = Number(req.query.limite ?? 20);
const resultado = productos.slice((pagina - 1) * limite, pagina * limite);
```

### 8.3 Autenticación

Las rutas privadas se protegen con un middleware que comprueba un token (normalmente JWT en la cabecera `Authorization: Bearer ...`) y responde `401` si falta o no es válido.

---

## 9. Resumen

- Organiza la API por **recursos** y usa los **verbos HTTP** correctos.
- Estructura: `app.js` (config), `routes/`, `controllers/`, `middleware/`.
- Middleware clave: `cors`, `express.json()`, 404 y manejador de errores (4 parámetros).
- Valida las entradas; responde `400`, `404`, `201`, `204` según el caso.
- No expongas errores internos; configura CORS con orígenes concretos.
