# 04 - Servidor con Express

> [!info] ¿Qué es?
> **Express** es el framework más usado para crear **servidores web y APIs** con Node.js. Simplifica lo que con el módulo `http` sería tedioso: definir rutas, leer datos de la petición, enviar respuestas JSON y servir archivos.

---

## 1. Antes de empezar

- Conviene haber leído [[Node 02 - npm y package.json]] y [[Node 03 - Módulos CommonJS y ESM]].
- Hay que entender cómo funciona HTTP: [[Conceptos 01 - HTTP]] (métodos, códigos de estado, cabeceras).

---

## 2. Concepto fundamental

Un servidor Express recibe **peticiones** y devuelve **respuestas**. Para cada combinación de **método + ruta** defines una función (*handler*):

```
Petición: GET /api/productos
              │
              ▼
app.get("/api/productos", (req, res) => { ... })
              │
              ▼
Respuesta: 200 OK  [ { "id": 1, "nombre": "Camiseta" } ]
```

| Objeto | Qué contiene |
|---|---|
| `req` (*request*) | Lo que envía el cliente: URL, parámetros, cuerpo, cabeceras |
| `res` (*response*) | Herramientas para responder: `json()`, `send()`, `status()` |

---

## 3. Sintaxis / estructura

```bash
npm init -y
npm i express
```

```javascript
// index.js  (con "type": "module" en package.json)
import express from "express";

const app = express();
const PORT = 3000;

app.get("/", (req, res) => {
  res.send("Hola desde Express");
});

app.listen(PORT, () => {
  console.log(`Servidor en http://localhost:${PORT}`);
});
```

```bash
node --watch index.js
```

---

## 4. Elementos / propiedades / características

### 4.1 Métodos de ruta

| Método | Uso habitual |
|---|---|
| `app.get(ruta, fn)` | Leer datos |
| `app.post(ruta, fn)` | Crear |
| `app.put(ruta, fn)` | Reemplazar |
| `app.patch(ruta, fn)` | Modificar parcialmente |
| `app.delete(ruta, fn)` | Borrar |
| `app.use(fn)` | Middleware para todas las rutas |

### 4.2 Datos de la petición (`req`)

| Propiedad | Ejemplo de URL | Valor |
|---|---|---|
| `req.params` | `/usuarios/42` con ruta `/usuarios/:id` | `{ id: "42" }` |
| `req.query` | `/productos?orden=precio` | `{ orden: "precio" }` |
| `req.body` | Cuerpo de un POST (JSON) | `{ nombre: "Ana" }` (necesita `express.json()`) |
| `req.headers` | Cabeceras | `{ authorization: "Bearer ..." }` |
| `req.method` | | `"GET"` |

### 4.3 Formas de responder (`res`)

| Método | Qué hace |
|---|---|
| `res.send("texto")` | Envía texto o HTML |
| `res.json(objeto)` | Envía JSON |
| `res.status(201).json(...)` | Cambia el código de estado |
| `res.sendStatus(204)` | Solo el código, sin cuerpo |
| `res.redirect("/otra")` | Redirige |
| `res.sendFile(ruta)` | Envía un archivo |

### 4.4 Middleware

Un **middleware** es una función que se ejecuta **antes** de llegar a la ruta. Recibe `req`, `res` y `next` (para continuar).

```javascript
app.use(express.json());                 // convierte el cuerpo JSON en req.body
app.use(express.static("public"));       // sirve archivos de la carpeta public

app.use((req, res, next) => {            // middleware propio: log
  console.log(req.method, req.url);
  next();
});
```

### 4.5 Router: separar rutas

```javascript
import { Router } from "express";
const router = Router();
router.get("/", ...);
router.post("/", ...);
export default router;

// en index.js
app.use("/api/productos", router);
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: rutas con parámetros

```javascript
app.get("/saludo/:nombre", (req, res) => {
  const { nombre } = req.params;
  const { idioma } = req.query;      // /saludo/Ana?idioma=en
  const texto = idioma === "en" ? "Hello" : "Hola";
  res.json({ mensaje: `${texto}, ${nombre}` });
});
```

### 5.2 Ejemplo habitual: mini API de tareas en memoria

```javascript
import express from "express";

const app = express();
app.use(express.json());

let tareas = [{ id: 1, texto: "Aprender Express", hecha: false }];
let siguienteId = 2;

app.get("/api/tareas", (req, res) => {
  res.json(tareas);
});

app.get("/api/tareas/:id", (req, res) => {
  const tarea = tareas.find((t) => t.id === Number(req.params.id));
  if (!tarea) return res.status(404).json({ error: "Tarea no encontrada" });
  res.json(tarea);
});

app.post("/api/tareas", (req, res) => {
  const { texto } = req.body;
  if (!texto) return res.status(400).json({ error: "Falta el texto" });
  const nueva = { id: siguienteId++, texto, hecha: false };
  tareas.push(nueva);
  res.status(201).json(nueva);
});

app.delete("/api/tareas/:id", (req, res) => {
  tareas = tareas.filter((t) => t.id !== Number(req.params.id));
  res.sendStatus(204);
});

app.listen(3000, () => console.log("API en http://localhost:3000"));
```

Probar con `curl` (ver [[Terminal 05 - Procesos y red|Procesos y red]]):

```bash
curl http://localhost:3000/api/tareas
curl -X POST http://localhost:3000/api/tareas \
  -H "Content-Type: application/json" -d '{"texto":"Probar la API"}'
```

### 5.3 Servir una web estática

```javascript
app.use(express.static("public"));   // public/index.html → http://localhost:3000/
```

---

## 6. Buenas prácticas

- **Usa `express.json()`** si recibes JSON; sin él, `req.body` es `undefined`.
- **Devuelve el código de estado correcto**: 200, 201, 204, 400, 404, 500.
- **Usa `return`** al responder antes de tiempo, para no enviar dos respuestas.
- **Valida siempre** lo que llega en `req.body`, `req.params` y `req.query`: nunca confíes en el cliente.
- **Separa las rutas** con `Router` en cuanto haya más de unas pocas.
- **El puerto y los secretos, en variables de entorno**: [[Node 05 - Variables de entorno]].
- Para una API completa (errores, CORS, estructura), ver [[Node 08 - API REST con Express]].

---

## 7. Diferencias importantes

### 7.1 `req.params` vs. `req.query` vs. `req.body`

| | `params` | `query` | `body` |
|---|---|---|---|
| Dónde va | En la ruta: `/usuarios/42` | Tras `?`: `?orden=precio` | En el cuerpo de la petición |
| Uso | Identificar un recurso | Filtrar, ordenar, paginar | Enviar datos (crear, editar) |
| Tipo | Siempre texto | Siempre texto | Lo que se envíe (JSON) |

### 7.2 `res.send()` vs. `res.json()`

`res.json()` convierte a JSON y pone la cabecera `Content-Type: application/json`. `res.send()` envía texto/HTML (o JSON si le pasas un objeto, pero `json()` es más explícito).

### 7.3 `app.use()` vs. `app.get()`

`app.use()` se aplica a **todos los métodos** y a todas las rutas que **empiezan** por la indicada. `app.get()` solo a peticiones GET de esa ruta exacta.

---

## 8. Casos especiales

### 8.1 *Cannot set headers after they are sent*

Has respondido dos veces en la misma petición. Añade `return` antes de `res.status(...).json(...)`.

### 8.2 `req.params.id` es texto

`"42" === 42` es `false`. Conviértelo: `Number(req.params.id)`.

### 8.3 Errores de CORS

Si tu frontend (otro puerto) no puede llamar a la API, instala y usa el middleware `cors` (ver [[Node 08 - API REST con Express]]).

### 8.4 El orden importa

Express revisa rutas y middleware **en el orden en que se declaran**. Los middleware (`express.json()`) van antes de las rutas; el manejador de 404, al final.

---

## 9. Resumen

- Express crea servidores y APIs en Node con poco código.
- `app.get/post/put/delete(ruta, (req, res) => ...)`.
- `req.params` (ruta), `req.query` (`?`), `req.body` (cuerpo, con `express.json()`).
- `res.json()`, `res.status()`, `res.sendStatus()`.
- **Middleware**: funciones que se ejecutan antes de las rutas (`next()`).
- `Router` para separar rutas; `express.static` para archivos.
- Siguiente paso: [[Node 05 - Variables de entorno]] y [[Node 08 - API REST con Express]].
