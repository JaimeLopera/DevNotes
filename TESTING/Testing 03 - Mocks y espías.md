# 03 - Mocks y espías

> [!info] ¿Qué es?
> Un **mock** es un **sustituto falso** de algo de lo que depende tu código (una API, la base de datos, la fecha actual, el envío de correos). Un **espía** registra cómo se ha llamado a una función. Sirven para testear una pieza **aislada**, de forma rápida y predecible.

---

## 1. Antes de empezar

Conviene dominar [[Testing 02 - Tests unitarios con Vitest]].

---

## 2. Concepto fundamental

Una función que depende de una API externa:

```javascript
export async function nombreDeUsuario(id) {
  const res = await fetch(`https://api.ejemplo.com/usuarios/${id}`);
  const usuario = await res.json();
  return usuario.nombre.toUpperCase();
}
```

Si la testeas tal cual, el test:
- depende de internet y de que la API esté encendida,
- es lento,
- puede dar resultados distintos cada vez.

Con un mock **sustituyes `fetch`** por una función falsa que devuelve lo que tú decidas:

```
Test real:   nombreDeUsuario → fetch REAL → internet → API → ...
Con mock:    nombreDeUsuario → fetch FALSO → { nombre: "ana" } (al instante)
```

---

## 3. Sintaxis / estructura

```javascript
import { vi } from "vitest";

const fn = vi.fn();                         // función falsa
fn.mockReturnValue(42);                     // qué devuelve
fn.mockResolvedValue({ ok: true });         // qué devuelve como promesa

vi.spyOn(objeto, "metodo");                 // espía un método real
vi.mock("./modulo.js");                     // simula un módulo entero
```

---

## 4. Elementos / propiedades / características

### 4.1 Tipos de dobles de prueba

| Tipo | Qué hace | Ejemplo |
|---|---|---|
| **Stub** | Devuelve una respuesta fija | `fetch` que siempre devuelve el mismo usuario |
| **Spy** (espía) | Registra llamadas (y puede dejar pasar la real) | Comprobar que se llamó a `enviarEmail` |
| **Mock** | Stub + espía: responde y verifica llamadas | `vi.fn()` |
| **Fake** | Implementación simplificada pero funcional | Base de datos en memoria |

En el día a día, a todo se le suele llamar "mock".

### 4.2 Configurar lo que devuelve `vi.fn()`

| Método | Qué hace |
|---|---|
| `mockReturnValue(x)` | Siempre devuelve `x` |
| `mockReturnValueOnce(x)` | Devuelve `x` solo la próxima vez |
| `mockResolvedValue(x)` | Devuelve `Promise.resolve(x)` |
| `mockRejectedValue(err)` | Devuelve `Promise.reject(err)` |
| `mockImplementation(fn)` | Ejecuta tu propia función |

### 4.3 Comprobar llamadas

| Matcher | Comprueba |
|---|---|
| `toHaveBeenCalled()` | Se llamó al menos una vez |
| `toHaveBeenCalledTimes(n)` | Se llamó `n` veces |
| `toHaveBeenCalledWith(a, b)` | Se llamó con esos argumentos |
| `toHaveBeenLastCalledWith(...)` | Argumentos de la última llamada |

### 4.4 Simular fechas y temporizadores

```javascript
vi.useFakeTimers();
vi.setSystemTime(new Date("2026-01-01"));   // "hoy" es 1 de enero
vi.advanceTimersByTime(5000);               // avanza 5 s sin esperar
vi.useRealTimers();                         // restaurar
```

### 4.5 Simular un módulo entero

```javascript
vi.mock("./email.js", () => ({
  enviarEmail: vi.fn().mockResolvedValue(true),
}));
```

### 4.6 Limpiar entre tests

```javascript
afterEach(() => {
  vi.restoreAllMocks();   // restaura espías y limpia registros
});
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: espiar un callback

```javascript
import { it, expect, vi } from "vitest";

function repetir(veces, accion) {
  for (let i = 0; i < veces; i++) accion(i);
}

it("llama a la acción tantas veces como se indica", () => {
  const accion = vi.fn();
  repetir(3, accion);
  expect(accion).toHaveBeenCalledTimes(3);
  expect(accion).toHaveBeenLastCalledWith(2);
});
```

### 5.2 Ejemplo habitual: simular `fetch`

```javascript
import { describe, it, expect, vi, afterEach } from "vitest";
import { nombreDeUsuario } from "./usuarios.js";

describe("nombreDeUsuario", () => {
  afterEach(() => vi.restoreAllMocks());

  it("devuelve el nombre en mayúsculas", async () => {
    vi.spyOn(globalThis, "fetch").mockResolvedValue({
      json: async () => ({ nombre: "ana" }),
    });

    const nombre = await nombreDeUsuario(1);

    expect(nombre).toBe("ANA");
    expect(fetch).toHaveBeenCalledWith("https://api.ejemplo.com/usuarios/1");
  });

  it("propaga el error si la API falla", async () => {
    vi.spyOn(globalThis, "fetch").mockRejectedValue(new Error("Sin conexión"));
    await expect(nombreDeUsuario(1)).rejects.toThrow("Sin conexión");
  });
});
```

### 5.3 Inyectar dependencias en lugar de mockear módulos

```javascript
// Más fácil de testear: la dependencia entra como parámetro
export async function registrar(usuario, { enviarEmail }) {
  // ... guardar usuario ...
  await enviarEmail(usuario.email, "¡Bienvenido!");
}

// Test
const enviarEmail = vi.fn().mockResolvedValue(true);
await registrar({ email: "ana@mail.com" }, { enviarEmail });
expect(enviarEmail).toHaveBeenCalledWith("ana@mail.com", "¡Bienvenido!");
```

---

## 6. Buenas prácticas

- **Mockea solo lo externo** (red, BD, tiempo, azar, correo), no tu propia lógica.
- **No mockees de más**: si todo es falso, el test no prueba nada real.
- **Restaura los mocks** después de cada test (`vi.restoreAllMocks()`).
- **Prefiere inyectar dependencias**: hace los mocks triviales.
- **Usa temporizadores falsos** en lugar de esperar de verdad.
- **Comprueba el resultado**, y solo cuando importe, también las llamadas.

---

## 7. Diferencias importantes

### 7.1 `vi.fn()` vs. `vi.spyOn()`

| | `vi.fn()` | `vi.spyOn(obj, "m")` |
|---|---|---|
| Crea | Una función falsa nueva | Un espía sobre un método **existente** |
| Ejecuta el código real | No | Sí, salvo que le cambies la implementación |
| Se restaura | — | Con `mockRestore()` / `restoreAllMocks()` |

### 7.2 Mock vs. test de integración

Un test con mocks comprueba **tu lógica** aislada. Un test de integración ([[Testing 05 - Tests de integración y API]]) comprueba que **las piezas reales encajan**. Hacen falta los dos.

### 7.3 `mockReturnValue` vs. `mockResolvedValue`

`mockReturnValue(x)` devuelve `x` directamente. `mockResolvedValue(x)` devuelve una promesa que se resuelve con `x` (para funciones `async`).

---

## 8. Casos especiales

### 8.1 Simular `Math.random`

```javascript
vi.spyOn(Math, "random").mockReturnValue(0.5);
```

### 8.2 Simular `localStorage`

Con `environment: "jsdom"` ya existe un `localStorage` en memoria; basta con limpiarlo en `beforeEach` (`localStorage.clear()`).

### 8.3 Simular APIs completas

Para muchos endpoints, librerías como **MSW** (*Mock Service Worker*) interceptan las peticiones de red y responden con datos definidos por ti, tanto en tests como en desarrollo.

---

## 9. Resumen

- Un **mock** sustituye una dependencia externa por una falsa y controlada.
- `vi.fn()` crea funciones falsas; `vi.spyOn()` espía métodos reales; `vi.mock()` simula módulos.
- `mockReturnValue`, `mockResolvedValue`, `mockRejectedValue` controlan la respuesta.
- `toHaveBeenCalledWith` comprueba cómo se llamó.
- `vi.useFakeTimers()` controla fechas y esperas.
- Mockea solo lo externo y restaura los mocks tras cada test.
