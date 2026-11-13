# 10 - TypeScript en proyectos

> [!info] ¿Qué es?
> Esta nota muestra cómo se usa TypeScript **en situaciones reales**: manipular el DOM, crear una API con Express, tipar las respuestas de `fetch`, usar librerías de terceros y validar datos externos. Es el paso de "conozco la sintaxis" a "lo uso en mis proyectos".

---

## 1. Antes de empezar

- Conviene haber leído toda la carpeta hasta [[TypeScript 09 - tsconfig y configuración]].
- Para la parte de servidor: [[Node 04 - Servidor con Express|Servidor con Express]].
- Para la parte de navegador: [[JavaScript 09 - DOM|DOM]] y [[JavaScript 13 - Fetch y APIs|Fetch y APIs]].

---

## 2. Concepto fundamental

En un proyecto, los tipos vienen de **tres sitios**:

| Origen | Ejemplos | Cómo se obtienen |
|---|---|---|
| **Del entorno** | `document`, `HTMLInputElement`, `process` | `lib: ["DOM"]` o `@types/node` |
| **De las librerías** | Tipos de Express, React, Zod | Incluidos en el paquete o en `@types/paquete` |
| **Tuyos** | `Usuario`, `Producto`, `Pedido` | Interfaces y types que defines |

La regla de oro: **los datos que vienen de fuera** (formularios, APIs, archivos) **no tienen tipo garantizado** hasta que los validas.

---

## 3. Sintaxis / estructura

Estructura típica:

```
proyecto/
├── tsconfig.json
├── package.json
└── src/
    ├── types/          ← tipos compartidos
    │   └── index.ts
    ├── services/       ← llamadas a APIs
    └── main.ts
```

---

## 4. Elementos / propiedades / características

### 4.1 Tipos del DOM

| Tipo | Elemento |
|---|---|
| `HTMLElement` | Cualquier elemento |
| `HTMLInputElement` | `<input>` (tiene `.value`, `.checked`) |
| `HTMLButtonElement` | `<button>` |
| `HTMLFormElement` | `<form>` |
| `HTMLSelectElement` | `<select>` |
| `MouseEvent`, `KeyboardEvent`, `SubmitEvent` | Eventos |

```typescript
const input = document.querySelector<HTMLInputElement>("#email");
if (input) console.log(input.value);

// getElementById devuelve HTMLElement | null
const boton = document.getElementById("enviar") as HTMLButtonElement | null;
```

### 4.2 Tipos de librerías: `@types`

Si una librería no incluye sus tipos, suelen estar en DefinitelyTyped:

```bash
npm i express
npm i -D @types/express @types/node
```

| Situación | Qué hacer |
|---|---|
| El paquete incluye tipos (Zod, Vite, Axios) | Nada |
| Existe `@types/paquete` | `npm i -D @types/paquete` |
| No hay tipos | Declarar un módulo: `declare module "paquete";` en un `.d.ts` |

### 4.3 Tipar Express

```typescript
import express, { Request, Response } from "express";

interface CrearTareaBody {
  titulo: string;
}

app.post("/api/tareas", (req: Request<{}, {}, CrearTareaBody>, res: Response) => {
  const { titulo } = req.body; // string
  res.status(201).json({ id: 1, titulo });
});
```

### 4.4 Validar datos externos con Zod

Zod define un esquema que **valida en ejecución** y del que se **deriva el tipo**:

```typescript
import { z } from "zod";

const ProductoSchema = z.object({
  id: z.number(),
  nombre: z.string(),
  precio: z.number().positive(),
});

type Producto = z.infer<typeof ProductoSchema>; // tipo sin escribirlo dos veces

const datos = await fetch("/api/productos/1").then((r) => r.json());
const producto = ProductoSchema.parse(datos);   // lanza error si no cumple
```

### 4.5 Tipos compartidos entre frontend y backend

En proyectos *full stack*, poner los tipos (o los esquemas de Zod) en una carpeta compartida evita que el frontend y la API se desincronicen.

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: formulario en el navegador

```typescript
const form = document.querySelector<HTMLFormElement>("#contacto");

form?.addEventListener("submit", (evento: SubmitEvent) => {
  evento.preventDefault();
  const datos = new FormData(form);
  const email = String(datos.get("email") ?? "");

  if (!email.includes("@")) {
    alert("Email no válido");
    return;
  }
  console.log("Enviando", email);
});
```

### 5.2 Ejemplo habitual: servicio de API tipado en el frontend

```typescript
// src/types/index.ts
export interface Tarea {
  id: number;
  titulo: string;
  hecha: boolean;
}
export type NuevaTarea = Omit<Tarea, "id" | "hecha">;

// src/services/tareas.ts
import type { Tarea, NuevaTarea } from "../types/index.js";

const API = "/api/tareas";

async function peticion<T>(url: string, opciones?: RequestInit): Promise<T> {
  const res = await fetch(url, {
    headers: { "Content-Type": "application/json" },
    ...opciones,
  });
  if (!res.ok) throw new Error(`Error ${res.status}`);
  return res.json() as Promise<T>;
}

export const tareas = {
  listar: () => peticion<Tarea[]>(API),
  crear: (nueva: NuevaTarea) =>
    peticion<Tarea>(API, { method: "POST", body: JSON.stringify(nueva) }),
  borrar: (id: number) => peticion<void>(`${API}/${id}`, { method: "DELETE" }),
};
```

### 5.3 API con Express y TypeScript

```bash
npm i express
npm i -D typescript tsx @types/express @types/node
```

```typescript
// src/index.ts
import express from "express";
import type { Tarea } from "./types/index.js";

const app = express();
app.use(express.json());

const lista: Tarea[] = [];

app.get("/api/tareas", (_req, res) => {
  res.json(lista);
});

app.listen(3000, () => console.log("API en http://localhost:3000"));
```

```bash
npx tsx watch src/index.ts
```

---

## 6. Buenas prácticas

- **Valida en la frontera**: todo lo que entra de fuera (APIs, formularios, `JSON.parse`) se valida antes de confiar en su tipo.
- **Usa `import type`** para importar solo tipos (no generan código).
- **Centraliza los tipos compartidos** en `src/types/`.
- **Deriva tipos de esquemas** (`z.infer`) en lugar de escribirlos dos veces.
- **Evita `as` en el DOM** cuando puedas usar el genérico: `querySelector<HTMLInputElement>()`.
- **Comprueba tipos en CI** con `tsc --noEmit`.

---

## 7. Diferencias importantes

### 7.1 Tipos vs. validación

| | Tipos de TypeScript | Validación (Zod) |
|---|---|---|
| Cuándo actúan | Al escribir/compilar | Al ejecutar |
| Protegen de | Tus propios errores | Datos externos incorrectos |
| Existen en el `.js` final | No | Sí |

### 7.2 `import` vs. `import type`

`import type { X }` desaparece al compilar. `import { X }` se mantiene (si `X` es un valor). Usa `import type` para interfaces y types.

### 7.3 `querySelector<T>()` vs. `as`

El genérico (`querySelector<HTMLInputElement>`) mantiene el `| null` y te obliga a comprobarlo. `as HTMLInputElement` lo oculta.

---

## 8. Casos especiales

### 8.1 Variables globales del navegador

```typescript
// src/global.d.ts
declare global {
  interface Window {
    miConfig: { apiUrl: string };
  }
}
export {};
```

### 8.2 Importar archivos que no son código

```typescript
// src/assets.d.ts
declare module "*.svg" {
  const url: string;
  export default url;
}
```

(Vite ya incluye estas declaraciones con `/// <reference types="vite/client" />`.)

### 8.3 Variables de entorno tipadas

```typescript
const API_URL = process.env.API_URL;  // string | undefined
if (!API_URL) throw new Error("Falta API_URL");
```

Ver [[Node 05 - Variables de entorno|Variables de entorno]].

---

## 9. Resumen

- Los tipos vienen del entorno (`DOM`, `@types/node`), de las librerías (`@types/...`) y de ti.
- `querySelector<HTMLInputElement>()` para el DOM.
- `npm i -D @types/express @types/node` para Express.
- Los datos externos se **validan** (Zod) y de ahí se deriva el tipo con `z.infer`.
- `import type` para importar solo tipos.
- Siguiente paso: [[TypeScript 11 - Referencia rápida]].
