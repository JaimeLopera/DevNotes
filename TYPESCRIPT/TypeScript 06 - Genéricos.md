# 06 - Genéricos

> [!info] ¿Qué es?
> Los **genéricos** permiten escribir funciones, interfaces y clases que funcionan con **cualquier tipo** sin perder la información de cuál es. Son como un **parámetro, pero de tipo**: se escriben entre `< >` y se rellenan al usarlos.

---

## 1. Antes de empezar

Conviene dominar [[TypeScript 04 - Funciones]] y [[TypeScript 03 - Interfaces y type]].

---

## 2. Concepto fundamental

Una función que devuelve el primer elemento de un array:

```typescript
// Con any: funciona, pero se pierde el tipo
function primeroAny(lista: any[]): any {
  return lista[0];
}
const a = primeroAny([1, 2, 3]); // any → sin autocompletado ni comprobaciones

// Con genérico: funciona con todo y conserva el tipo
function primero<T>(lista: T[]): T | undefined {
  return lista[0];
}
const b = primero([1, 2, 3]);        // number | undefined
const c = primero(["a", "b"]);       // string | undefined
```

`T` es una **variable de tipo**: TypeScript la deduce de lo que le pasas.

---

## 3. Sintaxis / estructura

```typescript
function nombre<T>(param: T): T { ... }          // función
const flecha = <T,>(param: T): T => param;        // flecha (la coma evita confusión con JSX)
interface Caja<T> { valor: T }                    // interface
type Par<A, B> = [A, B];                          // type
class Pila<T> { private items: T[] = []; }        // clase
```

Por convención se usan letras mayúsculas: `T` (tipo), `K` (clave), `V` (valor), `E` (elemento).

---

## 4. Elementos / propiedades / características

### 4.1 Inferencia vs. tipo explícito

```typescript
primero([1, 2, 3]);           // T se infiere como number
primero<string>(["a", "b"]);  // T indicado a mano
```

### 4.2 Varios parámetros de tipo

```typescript
function emparejar<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}
emparejar("edad", 30); // [string, number]
```

### 4.3 Restricciones (`extends`)

Limitan qué tipos se aceptan:

```typescript
function longitud<T extends { length: number }>(valor: T): number {
  return valor.length;
}

longitud("hola");     // ✔ string tiene length
longitud([1, 2, 3]);  // ✔
longitud(42);         // ✘ Error: number no tiene length
```

### 4.4 `keyof`: claves de un objeto

```typescript
function obtener<T, K extends keyof T>(objeto: T, clave: K): T[K] {
  return objeto[clave];
}

const usuario = { nombre: "Ana", edad: 30 };
obtener(usuario, "nombre"); // string
obtener(usuario, "email");  // ✘ Error: no es una clave de usuario
```

### 4.5 Genéricos con valor por defecto

```typescript
interface RespuestaApi<T = unknown> {
  datos: T;
  error: string | null;
}
```

### 4.6 Genéricos que ya usas sin saberlo

| Genérico | Significa |
|---|---|
| `Array<T>` / `T[]` | Array de `T` |
| `Promise<T>` | Promesa que resuelve a `T` |
| `Map<K, V>` | Mapa de claves `K` a valores `V` |
| `Set<T>` | Conjunto de `T` |
| `Record<K, V>` | Objeto con claves `K` y valores `V` |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: una caja genérica

```typescript
interface Caja<T> {
  valor: T;
  creadaEn: Date;
}

const cajaNumero: Caja<number> = { valor: 42, creadaEn: new Date() };
const cajaTexto: Caja<string> = { valor: "hola", creadaEn: new Date() };
```

### 5.2 Ejemplo habitual: función `fetch` tipada

```typescript
async function obtenerJSON<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`Error ${res.status}`);
  return res.json() as Promise<T>;
}

interface Producto {
  id: number;
  nombre: string;
  precio: number;
}

const productos = await obtenerJSON<Producto[]>("/api/productos");
productos[0].precio; // number, con autocompletado
```

> [!warning] Los tipos no validan
> `obtenerJSON<Producto[]>` **confía** en que la API devuelve eso. Si no estás seguro de la API, valida los datos (por ejemplo, con Zod).

### 5.3 Agrupar elementos por una clave

```typescript
function agruparPor<T, K extends keyof T>(lista: T[], clave: K): Map<T[K], T[]> {
  const grupos = new Map<T[K], T[]>();
  for (const item of lista) {
    const k = item[clave];
    grupos.set(k, [...(grupos.get(k) ?? []), item]);
  }
  return grupos;
}

agruparPor(productos, "precio");
```

---

## 6. Buenas prácticas

- **Usa genéricos cuando haya relación** entre entrada y salida; si no, basta con tipos normales.
- **Deja que TypeScript infiera** `T` siempre que pueda.
- **Restringe con `extends`** para poder usar propiedades dentro de la función.
- **Nombres claros** cuando hay varios: `TDatos`, `TError`, en lugar de `T`, `U`, `V`.
- **No los uses para todo**: un genérico que solo aparece una vez suele sobrar.

---

## 7. Diferencias importantes

### 7.1 Genérico vs. `any`

| | `any` | Genérico `<T>` |
|---|---|---|
| Acepta cualquier tipo | Sí | Sí |
| Conserva el tipo en el resultado | No | Sí |
| Seguridad | Ninguna | Total |

### 7.2 Genérico vs. unión

`string | number` acepta cualquiera de los dos **en cualquier momento**. `<T>` fija **un** tipo por llamada y lo mantiene: si entra `string`, sale `string`.

### 7.3 `extends` en genéricos vs. en interfaces

En `<T extends X>` significa "T debe ser compatible con X" (una restricción). En `interface A extends B` significa herencia.

---

## 8. Casos especiales

### 8.1 Genéricos en archivos `.tsx`

`const f = <T>(x: T) => x;` puede confundirse con JSX. Escribe `<T,>` o usa `function`.

### 8.2 Tipos condicionales

Genéricos que eligen un tipo según una condición (nivel avanzado):

```typescript
type EsTexto<T> = T extends string ? "sí" : "no";
type A = EsTexto<"hola">; // "sí"
```

### 8.3 Inferir el tipo de un elemento de array

```typescript
type Elemento<T> = T extends (infer E)[] ? E : never;
type N = Elemento<number[]>; // number
```

---

## 9. Resumen

- Los genéricos son **parámetros de tipo**: `<T>`.
- Permiten código reutilizable que **conserva el tipo** (a diferencia de `any`).
- TypeScript suele inferir `T` a partir de los argumentos.
- `T extends X` restringe; `K extends keyof T` limita a claves de un objeto.
- Ya los usas: `Array<T>`, `Promise<T>`, `Map<K, V>`.
- Siguiente paso: [[TypeScript 07 - Clases]] o [[TypeScript 08 - Utility types]].
