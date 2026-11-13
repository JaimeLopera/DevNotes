# 02 - Tipos básicos

> [!info] ¿Qué es?
> Los **tipos básicos** son los bloques con los que se describe cualquier dato en TypeScript: textos, números, booleanos, arrays, tuplas y algunos tipos especiales (`any`, `unknown`, `void`, `never`). Todo lo demás se construye combinándolos.

---

## 1. Antes de empezar

- Conviene haber leído [[TypeScript 01 - Fundamentos]].
- Los tipos de JavaScript en los que se basan: [[JavaScript 02 - Variables y tipos|Variables y tipos en JavaScript]].

---

## 2. Concepto fundamental

Un **tipo** es el conjunto de valores que puede tener algo. Al anotar `let edad: number`, le dices a TypeScript: "aquí solo pueden ir números". Si intentas meter otra cosa, avisa.

```typescript
let edad: number = 30;
edad = 31;        // ✔
edad = "treinta"; // ✘ Error
```

---

## 3. Sintaxis / estructura

```typescript
let variable: tipo = valor;
const lista: tipo[] = [...];
```

---

## 4. Elementos / propiedades / características

### 4.1 Primitivos

| Tipo | Valores | Ejemplo |
|---|---|---|
| `string` | Texto | `let nombre: string = "Ana"` |
| `number` | Números (enteros y decimales) | `let precio: number = 9.99` |
| `boolean` | `true` / `false` | `let activo: boolean = true` |
| `null` | Solo `null` | `let x: null = null` |
| `undefined` | Solo `undefined` | `let y: undefined` |
| `bigint` | Enteros enormes | `let n: bigint = 100n` |
| `symbol` | Identificadores únicos | `let s: symbol = Symbol()` |

> [!warning] En minúscula
> Usa `string`, `number`, `boolean`, nunca `String`, `Number`, `Boolean` (son los objetos envoltorio de JavaScript).

### 4.2 Arrays

```typescript
const numeros: number[] = [1, 2, 3];
const nombres: Array<string> = ["Ana", "Luis"];   // sintaxis equivalente
const mezcla: (string | number)[] = [1, "dos", 3];
```

### 4.3 Tuplas

Arrays de **longitud fija** con un tipo por posición:

```typescript
const punto: [number, number] = [10, 20];
const usuario: [string, number] = ["Ana", 30];
const respuesta: [ok: boolean, mensaje: string] = [true, "Guardado"]; // con etiquetas
```

### 4.4 Tipos literales

Un valor concreto como tipo:

```typescript
let direccion: "izquierda" | "derecha";
direccion = "izquierda"; // ✔
direccion = "arriba";    // ✘ Error
```

(Se combinan con uniones: ver [[TypeScript 05 - Uniones y narrowing]].)

### 4.5 Tipos especiales

| Tipo | Significa | Cuándo |
|---|---|---|
| `any` | "Cualquier cosa", **sin comprobar nada** | Evítalo |
| `unknown` | "Cualquier cosa", pero **hay que comprobarla** antes de usarla | Datos externos (JSON, APIs) |
| `void` | La función no devuelve nada | `function log(): void` |
| `never` | Nunca ocurre (la función lanza un error o no termina) | Funciones que siempre lanzan |
| `object` | Cualquier valor no primitivo | Poco útil; mejor una interface |

### 4.6 Enums

Conjunto de constantes con nombre:

```typescript
enum Estado {
  Pendiente = "PENDIENTE",
  Enviado = "ENVIADO",
  Entregado = "ENTREGADO",
}

let pedido: Estado = Estado.Enviado;
```

Hoy se prefiere a menudo una unión de literales (`type Estado = "PENDIENTE" | "ENVIADO"`), que no genera código extra.

### 4.7 Inferencia

```typescript
let a = 5;              // number
const b = 5;            // 5 (literal, porque const no cambia)
const c = [1, "x"];     // (string | number)[]
let d;                  // any (¡anótala!)
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```typescript
const producto: string = "Camiseta";
const precio: number = 19.99;
const enStock: boolean = true;
const tallas: string[] = ["S", "M", "L"];

console.log(`${producto}: ${precio} € (${enStock ? "disponible" : "agotado"})`);
```

### 5.2 Ejemplo habitual: `unknown` con datos externos

```typescript
const datos: unknown = JSON.parse('{"nombre":"Ana"}');

// datos.nombre;  ✘ Error: 'datos' is of type 'unknown'

if (typeof datos === "object" && datos !== null && "nombre" in datos) {
  console.log(datos.nombre); // ✔ ya se ha comprobado
}
```

---

## 6. Buenas prácticas

- **No anotes lo obvio**: `const nombre = "Ana"` ya es `string`.
- **Anota** las variables que declaras vacías (`let total: number;`).
- **`unknown` en lugar de `any`** para datos de origen desconocido.
- **Uniones de literales** en lugar de `enum` en la mayoría de casos.
- **Tipos en minúscula** (`string`, no `String`).

---

## 7. Diferencias importantes

### 7.1 `any` vs. `unknown`

| | `any` | `unknown` |
|---|---|---|
| Acepta cualquier valor | Sí | Sí |
| Se puede usar sin comprobar | Sí (peligroso) | No |
| Seguridad | Ninguna | Total |

### 7.2 `void` vs. `undefined` vs. `never`

| Tipo | La función… |
|---|---|
| `void` | Termina pero no devuelve nada útil |
| `undefined` | Devuelve explícitamente `undefined` |
| `never` | Nunca termina normalmente (lanza o bucle infinito) |

### 7.3 Array vs. tupla

Un array (`number[]`) tiene cualquier longitud y un mismo tipo. Una tupla (`[string, number]`) tiene una longitud fija y un tipo por posición.

### 7.4 `null` vs. `undefined`

Con `strict`, ninguno de los dos está permitido salvo que lo indiques: `let x: string | null`. Convención habitual: `undefined` = "no se ha dado valor"; `null` = "sin valor, a propósito".

---

## 8. Casos especiales

### 8.1 *Object is possibly 'undefined'*

TypeScript avisa de que algo podría no existir:

```typescript
const usuario = usuarios.find((u) => u.id === 1); // Usuario | undefined
usuario.nombre;   // ✘ Error
usuario?.nombre;  // ✔ encadenamiento opcional
```

### 8.2 Aserción de tipo (`as`)

Le dices a TypeScript "confía en mí":

```typescript
const input = document.querySelector("#email") as HTMLInputElement;
```

Úsalo con moderación: si te equivocas, el error aparece al ejecutar.

### 8.3 `as const`

Convierte un valor en literal de solo lectura:

```typescript
const COLORES = ["rojo", "verde"] as const; // readonly ["rojo", "verde"]
```

---

## 9. Resumen

- Primitivos: `string`, `number`, `boolean`, `null`, `undefined`.
- Arrays: `number[]` o `Array<number>`; tuplas: `[string, number]`.
- Literales: `"izquierda" | "derecha"`.
- `unknown` en lugar de `any`; `void` para funciones sin retorno.
- TypeScript infiere tipos: no anotes lo evidente.
- Siguiente paso: [[TypeScript 03 - Interfaces y type]].
