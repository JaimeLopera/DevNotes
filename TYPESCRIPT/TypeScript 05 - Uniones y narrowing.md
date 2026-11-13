# 05 - Uniones y narrowing

> [!info] ¿Qué es?
> Una **unión** indica que un valor puede ser de **varios tipos** (`string | number`). El **narrowing** (estrechamiento) es cómo TypeScript averigua, dentro de un `if`, **cuál de ellos es** en ese momento, para dejarte usarlo con seguridad.

---

## 1. Antes de empezar

Conviene haber leído [[TypeScript 02 - Tipos básicos]], [[TypeScript 03 - Interfaces y type]] y [[TypeScript 04 - Funciones]].

---

## 2. Concepto fundamental

```typescript
function mostrarId(id: string | number) {
  // Aquí id puede ser string o number:
  // id.toUpperCase();  ✘ Error: no existe en number

  if (typeof id === "string") {
    console.log(id.toUpperCase());  // aquí TypeScript SABE que es string
  } else {
    console.log(id.toFixed(0));     // y aquí que es number
  }
}
```

TypeScript **sigue la lógica de tu código**: después de comprobar el tipo, lo "estrecha" automáticamente dentro de cada rama.

---

## 3. Sintaxis / estructura

```typescript
type Id = string | number;
type Estado = "cargando" | "exito" | "error";
let resultado: Usuario | null = null;
```

---

## 4. Elementos / propiedades / características

### 4.1 Formas de estrechar

| Comprobación | Sirve para | Ejemplo |
|---|---|---|
| `typeof` | Primitivos | `typeof x === "string"` |
| `instanceof` | Clases | `error instanceof TypeError` |
| `in` | Si un objeto tiene una propiedad | `"email" in usuario` |
| Igualdad | Literales, `null` | `estado === "error"`, `x !== null` |
| Veracidad | `null`, `undefined`, `""` | `if (usuario) { ... }` |
| `Array.isArray` | Arrays | `Array.isArray(x)` |
| *Type guard* propio | Lo que quieras | `function esAdmin(u): u is Admin` |

### 4.2 Uniones de literales

```typescript
type Talla = "S" | "M" | "L" | "XL";
type Metodo = "GET" | "POST" | "PUT" | "DELETE";

function pedir(metodo: Metodo, url: string) { /* ... */ }
pedir("GET", "/api");    // ✔
pedir("FETCH", "/api");  // ✘ Error, y el editor autocompleta las opciones válidas
```

### 4.3 Uniones discriminadas

Objetos con una **propiedad común** (el "discriminante") que indica de qué tipo son. Es uno de los patrones más útiles de TypeScript:

```typescript
type Respuesta =
  | { estado: "cargando" }
  | { estado: "exito"; datos: string[] }
  | { estado: "error"; mensaje: string };

function pintar(r: Respuesta) {
  switch (r.estado) {
    case "cargando":
      return "Cargando…";
    case "exito":
      return r.datos.join(", ");   // aquí existe datos
    case "error":
      return `Error: ${r.mensaje}`; // aquí existe mensaje
  }
}
```

### 4.4 Comprobación exhaustiva con `never`

Si añades un nuevo estado y olvidas tratarlo, TypeScript avisa:

```typescript
default: {
  const imposible: never = r;
  throw new Error(`Estado no tratado: ${imposible}`);
}
```

### 4.5 *Type guards* propios

```typescript
interface Gato { maullar(): void }
interface Perro { ladrar(): void }

function esGato(animal: Gato | Perro): animal is Gato {
  return "maullar" in animal;
}

if (esGato(mascota)) mascota.maullar();
```

### 4.6 Intersecciones (`&`)

Lo contrario de la unión: el valor tiene **todo** lo de ambos tipos.

```typescript
type ConFecha = { creadoEn: Date };
type Post = { titulo: string } & ConFecha;
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: valores que pueden faltar

```typescript
function longitud(texto: string | null): number {
  if (texto === null) return 0;
  return texto.length;
}

// Con operadores modernos:
const largo = (texto: string | null) => texto?.length ?? 0;
```

### 5.2 Ejemplo habitual: estado de una petición en el frontend

```typescript
type EstadoPeticion<T> =
  | { tipo: "inactivo" }
  | { tipo: "cargando" }
  | { tipo: "exito"; datos: T }
  | { tipo: "error"; error: string };

let estado: EstadoPeticion<Producto[]> = { tipo: "inactivo" };

async function cargarProductos() {
  estado = { tipo: "cargando" };
  try {
    const res = await fetch("/api/productos");
    estado = { tipo: "exito", datos: await res.json() };
  } catch (e) {
    estado = { tipo: "error", error: e instanceof Error ? e.message : "Desconocido" };
  }
}
```

Así es **imposible** tener a la vez `datos` y `error`, o `datos` mientras se carga.

---

## 6. Buenas prácticas

- **Uniones de literales** en lugar de `string` cuando los valores posibles son pocos y conocidos.
- **Uniones discriminadas** para estados (cargando, éxito, error) en lugar de muchos booleanos.
- **Añade comprobación exhaustiva** (`never`) en los `switch` sobre uniones.
- **Evita `as`** para "forzar" un tipo: estrecha con comprobaciones reales.
- **Usa `?.` y `??`** para tratar `null`/`undefined` de forma breve.

---

## 7. Diferencias importantes

### 7.1 Unión (`|`) vs. intersección (`&`)

| | Unión `A \| B` | Intersección `A & B` |
|---|---|---|
| El valor es | A **o** B | A **y** B a la vez |
| Propiedades accesibles sin comprobar | Solo las comunes | Todas |

### 7.2 Narrowing vs. aserción (`as`)

| | Narrowing | `as` |
|---|---|---|
| Comprueba de verdad | Sí (en ejecución) | No |
| Seguro | Sí | Solo si aciertas |

### 7.3 `typeof` vs. `instanceof`

`typeof` sirve para primitivos (`"string"`, `"number"`, `"boolean"`, `"object"`, `"function"`, `"undefined"`). `instanceof` sirve para objetos creados con clases (`Date`, `Error`, tus clases). Las interfaces no existen al ejecutar: para ellas usa `in` o un *type guard*.

---

## 8. Casos especiales

### 8.1 `typeof null === "object"`

Es una rareza heredada de JavaScript. Comprueba `x !== null` además de `typeof x === "object"`.

### 8.2 El error en un `catch` es `unknown`

```typescript
try { ... } catch (error) {
  if (error instanceof Error) console.error(error.message);
}
```

### 8.3 El narrowing se pierde en callbacks

```typescript
if (usuario) {
  setTimeout(() => usuario.nombre); // puede dar error si usuario es let
}
```

Guarda el valor en una `const` antes de usarlo dentro del callback.

---

## 9. Resumen

- Una **unión** (`A | B`) es un valor que puede ser de varios tipos.
- El **narrowing** estrecha el tipo con `typeof`, `instanceof`, `in`, `===` o *type guards*.
- Uniones de **literales** para valores fijos (`"GET" | "POST"`).
- Uniones **discriminadas** para estados: una propiedad común indica la variante.
- `never` en el `default` garantiza que tratas todos los casos.
- Siguiente paso: [[TypeScript 06 - Genéricos]].
