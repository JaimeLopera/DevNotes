# 02 - Tests unitarios con Vitest

> [!info] ¿Qué es?
> **Vitest** es un *test runner* moderno para JavaScript y TypeScript: busca tus archivos de test, los ejecuta y muestra qué pasa y qué falla. Es rápido, funciona con ES Modules y TypeScript sin configuración, y su sintaxis es compatible con **Jest** (el clásico).

---

## 1. Antes de empezar

- Conviene haber leído [[Testing 01 - Fundamentos]].
- Necesitas un proyecto con `package.json`: [[Node 02 - npm y package.json|npm y package.json]].

---

## 2. Concepto fundamental

```
src/
├── precio.js           ← código
└── precio.test.js      ← sus tests (mismo nombre + .test)
```

1. Vitest busca archivos que terminan en `.test.js` o `.spec.js` (y `.ts`).
2. Ejecuta cada `it`/`test`.
3. Cada `expect` compara el valor real con el esperado.
4. Muestra el resumen y, en modo *watch*, vuelve a ejecutar al guardar.

---

## 3. Sintaxis / estructura

```bash
npm i -D vitest
```

```json
// package.json
"scripts": {
  "test": "vitest",
  "test:run": "vitest run",
  "coverage": "vitest run --coverage"
}
```

```javascript
import { describe, it, expect } from "vitest";

describe("nombre de lo que pruebas", () => {
  it("hace tal cosa en tal caso", () => {
    expect(valorReal).toBe(valorEsperado);
  });
});
```

```bash
npm test            # modo watch: se repite al guardar
npm run test:run    # una sola vez (CI)
```

---

## 4. Elementos / propiedades / características

### 4.1 Matchers (comparaciones) más usados

| Matcher | Comprueba | Ejemplo |
|---|---|---|
| `toBe(x)` | Igualdad estricta (`===`), primitivos | `expect(2 + 2).toBe(4)` |
| `toEqual(x)` | Igualdad de contenido (objetos, arrays) | `expect(obj).toEqual({ a: 1 })` |
| `toStrictEqual(x)` | Como `toEqual` pero más estricto (`undefined`, clases) | |
| `toBeTruthy()` / `toBeFalsy()` | Valor verdadero / falso | |
| `toBeNull()` / `toBeUndefined()` | `null` / `undefined` | |
| `toBeGreaterThan(n)` | Mayor que | `expect(edad).toBeGreaterThan(17)` |
| `toBeCloseTo(n)` | Decimales aproximados | `expect(0.1 + 0.2).toBeCloseTo(0.3)` |
| `toContain(x)` | Array o texto contiene | `expect(lista).toContain("a")` |
| `toHaveLength(n)` | Longitud | `expect(lista).toHaveLength(3)` |
| `toHaveProperty("k")` | El objeto tiene esa propiedad | |
| `toMatch(/regex/)` | Texto coincide con regex | |
| `toThrow()` | La función lanza un error | `expect(() => f()).toThrow("mensaje")` |
| `.not` | Niega cualquier matcher | `expect(x).not.toBe(0)` |

### 4.2 Tests asíncronos

```javascript
it("obtiene el usuario", async () => {
  const usuario = await obtenerUsuario(1);
  expect(usuario.nombre).toBe("Ana");
});

it("falla si no existe", async () => {
  await expect(obtenerUsuario(999)).rejects.toThrow("No encontrado");
});
```

### 4.3 Preparar y limpiar (*hooks*)

| Hook | Se ejecuta |
|---|---|
| `beforeEach(fn)` | Antes de **cada** test del bloque |
| `afterEach(fn)` | Después de cada test |
| `beforeAll(fn)` | Una vez antes de todos |
| `afterAll(fn)` | Una vez después de todos |

```javascript
let carrito;
beforeEach(() => {
  carrito = new Carrito();   // cada test empieza con un carrito vacío
});
```

### 4.4 Controlar qué se ejecuta

| Forma | Efecto |
|---|---|
| `it.only(...)` | Ejecuta solo este test |
| `it.skip(...)` | Lo salta |
| `it.todo("...")` | Marca un test pendiente de escribir |
| `it.each([...])` | Repite el test con varios datos |

### 4.5 Tests con varios datos (`it.each`)

```javascript
it.each([
  [100, 21, 21],
  [50, 10, 5],
  [0, 21, 0],
])("calcularIVA(%i, %i) = %i", (precio, porcentaje, esperado) => {
  expect(calcularIVA(precio, porcentaje)).toBe(esperado);
});
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```javascript
// src/texto.js
export function capitalizar(texto) {
  if (!texto) return "";
  return texto[0].toUpperCase() + texto.slice(1).toLowerCase();
}
```

```javascript
// src/texto.test.js
import { describe, it, expect } from "vitest";
import { capitalizar } from "./texto.js";

describe("capitalizar", () => {
  it("pone en mayúscula la primera letra", () => {
    expect(capitalizar("hola")).toBe("Hola");
  });

  it("pasa el resto a minúsculas", () => {
    expect(capitalizar("hOLA")).toBe("Hola");
  });

  it("devuelve cadena vacía si no recibe texto", () => {
    expect(capitalizar("")).toBe("");
  });
});
```

### 5.2 Ejemplo habitual: testear una clase

```javascript
// src/carrito.js
export class Carrito {
  #lineas = [];

  añadir(producto, cantidad = 1) {
    if (cantidad <= 0) throw new Error("Cantidad no válida");
    this.#lineas.push({ ...producto, cantidad });
  }

  get total() {
    return this.#lineas.reduce((s, l) => s + l.precio * l.cantidad, 0);
  }

  get unidades() {
    return this.#lineas.reduce((s, l) => s + l.cantidad, 0);
  }
}
```

```javascript
// src/carrito.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { Carrito } from "./carrito.js";

describe("Carrito", () => {
  let carrito;

  beforeEach(() => {
    carrito = new Carrito();
  });

  it("empieza vacío", () => {
    expect(carrito.total).toBe(0);
    expect(carrito.unidades).toBe(0);
  });

  it("calcula el total con varias líneas", () => {
    carrito.añadir({ nombre: "Taza", precio: 8 }, 2);
    carrito.añadir({ nombre: "Libreta", precio: 4.5 });
    expect(carrito.total).toBe(20.5);
    expect(carrito.unidades).toBe(3);
  });

  it("rechaza cantidades no válidas", () => {
    expect(() => carrito.añadir({ nombre: "Taza", precio: 8 }, 0)).toThrow("Cantidad no válida");
  });
});
```

Salida:

```
 ✓ src/carrito.test.js (3)
 ✓ src/texto.test.js (3)

 Test Files  2 passed (2)
      Tests  6 passed (6)
```

---

## 6. Buenas prácticas

- **Archivo de test junto al código** (`carrito.js` → `carrito.test.js`).
- **`describe` por función o clase**, `it` por comportamiento.
- **Nombres que se lean como frases**: "rechaza cantidades no válidas".
- **`toEqual` para objetos y arrays**, `toBe` para primitivos.
- **`beforeEach` para partir siempre de un estado limpio**.
- **No dejes `it.only`** en el código: los demás tests dejarían de ejecutarse.
- **Ejecuta `vitest run`** en CI y antes de hacer push.

---

## 7. Diferencias importantes

### 7.1 `toBe` vs. `toEqual`

```javascript
expect({ a: 1 }).toBe({ a: 1 });     // ✘ falla: son objetos distintos
expect({ a: 1 }).toEqual({ a: 1 });  // ✔ mismo contenido
```

### 7.2 Vitest vs. Jest

| | Vitest | Jest |
|---|---|---|
| ES Modules y TypeScript | Sin configurar | Requiere configuración |
| Velocidad | Muy rápido | Más lento |
| Sintaxis | Compatible con Jest | — |
| Cuándo | Proyectos nuevos, Vite | Proyectos existentes que ya lo usan |

### 7.3 `expect(f()).toThrow()` vs. `expect(() => f()).toThrow()`

Hay que pasar **una función**: si escribes `f()` directamente, el error salta antes de que `expect` pueda capturarlo.

---

## 8. Casos especiales

### 8.1 Decimales

`0.1 + 0.2` no es exactamente `0.3`. Usa `toBeCloseTo`.

### 8.2 Testear código del navegador

Para código que usa el DOM, configura un entorno simulado:

```javascript
// vitest.config.js
export default { test: { environment: "jsdom" } };
```

(Requiere `npm i -D jsdom`.)

### 8.3 Globales como en Jest

Con `test: { globals: true }` no hace falta importar `describe`, `it`, `expect`.

### 8.4 Cobertura

```bash
npm i -D @vitest/coverage-v8
npx vitest run --coverage
```

Ver [[Testing 07 - TDD y buenas prácticas]] para interpretarla.

---

## 9. Resumen

- `npm i -D vitest` y el script `"test": "vitest"`.
- Archivos `*.test.js` junto al código.
- `describe` agrupa; `it` define un caso; `expect(...).matcher()` comprueba.
- `toBe` (primitivos), `toEqual` (objetos), `toThrow` (errores), `rejects` (promesas).
- `beforeEach` para estado limpio; `it.each` para varios datos.
- Siguiente paso: [[Testing 03 - Mocks y espías]].
