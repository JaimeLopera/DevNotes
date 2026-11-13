# 05 - Tests de integración y API

> [!info] ¿Qué es?
> Un **test de integración** comprueba que **varias piezas funcionan bien juntas**: las rutas de una API con su lógica y su base de datos, por ejemplo. Los **tests de API** son el caso más habitual en desarrollo web: hacen peticiones HTTP a tu servidor y comprueban el código de estado y el JSON que devuelve.

---

## 1. Antes de empezar

- Conviene dominar [[Testing 02 - Tests unitarios con Vitest]] (o [[Testing 04 - Tests en Python con pytest]]).
- Y saber construir una API: [[Node 08 - API REST con Express|API REST con Express]].
- Ayuda conocer los códigos de estado: [[Conceptos 01 - HTTP]].

---

## 2. Concepto fundamental

Un test unitario prueba `crearProducto()` aislada. Un test de API prueba **el recorrido completo** de una petición:

```
Test ──POST /api/productos──► Express ──► middleware ──► validación ──► controlador ──► datos
  ▲                                                                                      │
  └──────────────── 201 Created + { id: 1, nombre: "Taza" } ◄────────────────────────────┘
```

Con **Supertest** se hacen estas peticiones **sin arrancar el servidor** en un puerto: se le pasa directamente la `app` de Express.

---

## 3. Sintaxis / estructura

```bash
npm i -D vitest supertest
```

Para poder testear, **separa la app del arranque**:

```javascript
// src/app.js     → crea y exporta la app (rutas, middleware)
// src/index.js   → importa la app y hace app.listen()
```

```javascript
import request from "supertest";
import { app } from "../src/app.js";

const res = await request(app).get("/api/productos");
expect(res.status).toBe(200);
```

---

## 4. Elementos / propiedades / características

### 4.1 Peticiones con Supertest

| Código | Petición |
|---|---|
| `request(app).get("/ruta")` | GET |
| `.post("/ruta").send({ ... })` | POST con cuerpo JSON |
| `.put(...)`, `.patch(...)`, `.delete(...)` | Otros métodos |
| `.set("Authorization", "Bearer x")` | Añadir cabecera |
| `.query({ pagina: 2 })` | Parámetros `?pagina=2` |

### 4.2 Qué comprobar en la respuesta

| Propiedad | Ejemplo |
|---|---|
| `res.status` | `expect(res.status).toBe(201)` |
| `res.body` | `expect(res.body).toEqual({ id: 1, nombre: "Taza" })` |
| `res.headers` | `expect(res.headers["content-type"]).toMatch(/json/)` |

### 4.3 Qué casos probar en cada endpoint

| Caso | Respuesta esperada |
|---|---|
| Petición correcta | `200` / `201` / `204` y datos correctos |
| Datos inválidos | `400` y mensaje de error |
| Recurso inexistente | `404` |
| Sin autenticación (si aplica) | `401` |
| Sin permisos (si aplica) | `403` |

### 4.4 La base de datos en los tests

| Estrategia | Ventajas | Inconvenientes |
|---|---|---|
| BD de test real (contenedor Docker) | Lo más parecido a producción | Más lenta y compleja |
| BD en memoria (SQLite) | Rápida | Puede comportarse distinto |
| Repositorio falso en memoria | Muy rápida | No prueba las consultas reales |

Sea cual sea: **cada test debe empezar con datos limpios** (`beforeEach` que vacía o recarga los datos).

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: un endpoint de salud

```javascript
// src/app.js
import express from "express";
export const app = express();
app.get("/api/salud", (req, res) => res.json({ estado: "ok" }));
```

```javascript
// tests/salud.test.js
import { it, expect } from "vitest";
import request from "supertest";
import { app } from "../src/app.js";

it("GET /api/salud responde ok", async () => {
  const res = await request(app).get("/api/salud");
  expect(res.status).toBe(200);
  expect(res.body).toEqual({ estado: "ok" });
});
```

### 5.2 Ejemplo habitual: CRUD de productos

```javascript
// tests/productos.test.js
import { describe, it, expect, beforeEach } from "vitest";
import request from "supertest";
import { app } from "../src/app.js";
import { reiniciarDatos } from "../src/datos.js";

describe("API /api/productos", () => {
  beforeEach(() => reiniciarDatos());

  it("crea un producto válido", async () => {
    const res = await request(app)
      .post("/api/productos")
      .send({ nombre: "Taza", precio: 8.5 });

    expect(res.status).toBe(201);
    expect(res.body).toMatchObject({ nombre: "Taza", precio: 8.5 });
    expect(res.body.id).toBeDefined();
  });

  it("rechaza un producto sin precio", async () => {
    const res = await request(app).post("/api/productos").send({ nombre: "Taza" });
    expect(res.status).toBe(400);
    expect(res.body.error).toBeDefined();
  });

  it("devuelve 404 si el producto no existe", async () => {
    const res = await request(app).get("/api/productos/999");
    expect(res.status).toBe(404);
  });

  it("lista los productos creados", async () => {
    await request(app).post("/api/productos").send({ nombre: "Taza", precio: 8.5 });
    const res = await request(app).get("/api/productos");
    expect(res.body).toHaveLength(1);
  });
});
```

### 5.3 En Python con FastAPI

```python
from fastapi.testclient import TestClient
from app.main import app

cliente = TestClient(app)

def test_crear_producto():
    res = cliente.post("/api/productos", json={"nombre": "Taza", "precio": 8.5})
    assert res.status_code == 201
    assert res.json()["nombre"] == "Taza"
```

---

## 6. Buenas prácticas

- **Separa `app` de `listen`** para poder testear sin abrir puertos.
- **Prueba los caminos felices y los de error** (400, 404, 401).
- **Datos limpios en cada test**: nunca dependas del orden de ejecución.
- **Usa `toMatchObject`** cuando la respuesta tenga campos variables (ids, fechas).
- **No uses la base de datos de desarrollo** (ni mucho menos la de producción) en los tests.
- **Usa variables de entorno de test** (`NODE_ENV=test`) para apuntar a la BD de pruebas.

---

## 7. Diferencias importantes

### 7.1 Unitario vs. integración

| | Unitario | Integración |
|---|---|---|
| Qué se prueba | Una función | Una ruta completa con sus piezas |
| Dependencias | Simuladas | Reales (o casi) |
| Detecta | Errores de lógica | Errores de "conexión" entre piezas |

### 7.2 Supertest vs. `fetch` a un servidor arrancado

Supertest no necesita un servidor escuchando en un puerto: es más rápido y no deja procesos colgados. `fetch` a `localhost` obliga a arrancar y parar el servidor en cada ejecución.

### 7.3 `toEqual` vs. `toMatchObject`

`toEqual` exige que el objeto sea **idéntico**. `toMatchObject` comprueba que **contiene** esas propiedades (ignora las demás).

---

## 8. Casos especiales

### 8.1 Rutas protegidas

```javascript
const token = generarTokenDePrueba({ id: 1, rol: "admin" });
const res = await request(app).delete("/api/productos/1").set("Authorization", `Bearer ${token}`);
```

### 8.2 Subida de archivos

```javascript
await request(app).post("/api/avatar").attach("imagen", "tests/fixtures/foto.png");
```

### 8.3 Contratos con el frontend

Si el frontend depende de la forma exacta del JSON, testear esa forma (o validarla con un esquema compartido) evita romperlo sin darte cuenta.

---

## 9. Resumen

- Los tests de integración prueban varias piezas reales juntas.
- En APIs de Node: **Supertest** + Vitest, pasando la `app` sin `listen`.
- Comprueba `res.status`, `res.body` y cabeceras.
- Prueba éxito, `400`, `404` y autenticación.
- Datos limpios en cada test y BD de pruebas separada.
- Siguiente paso: [[Testing 06 - Tests end-to-end con Playwright]].
