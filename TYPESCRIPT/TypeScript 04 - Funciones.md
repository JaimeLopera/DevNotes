# 04 - Funciones

> [!info] ¿Qué es?
> En TypeScript se indica **qué tipo tiene cada parámetro** y **qué devuelve** cada función. Es donde los tipos más ayudan: cualquiera que llame a la función sabe exactamente qué pasarle y qué recibirá, y el editor avisa si se equivoca.

---

## 1. Antes de empezar

- Conviene haber leído [[TypeScript 02 - Tipos básicos]] y [[TypeScript 03 - Interfaces y type]].
- Y conocer las funciones de JavaScript: [[JavaScript 06 - Funciones|Funciones en JavaScript]].

---

## 2. Concepto fundamental

La **firma** de una función describe su "contrato":

```typescript
function calcularTotal(precio: number, cantidad: number): number {
//                     └─ parámetros con tipo ─────────┘  └─ tipo de retorno
  return precio * cantidad;
}
```

- Los **parámetros** siempre deben llevar tipo (TypeScript no puede adivinarlos).
- El **retorno** se infiere, pero anotarlo en funciones exportadas documenta y protege.

---

## 3. Sintaxis / estructura

```typescript
// Declaración
function sumar(a: number, b: number): number {
  return a + b;
}

// Función flecha
const restar = (a: number, b: number): number => a - b;

// Tipo de función
type Operacion = (a: number, b: number) => number;
const multiplicar: Operacion = (a, b) => a * b; // a y b se infieren
```

---

## 4. Elementos / propiedades / características

### 4.1 Parámetros opcionales y por defecto

```typescript
function saludar(nombre: string, saludo?: string): string {
  return `${saludo ?? "Hola"}, ${nombre}`;
}

function crearUsuario(nombre: string, rol: string = "lector") {
  return { nombre, rol };
}
```

Los opcionales van **al final**.

### 4.2 Parámetros rest

```typescript
function sumarTodos(...numeros: number[]): number {
  return numeros.reduce((a, b) => a + b, 0);
}
sumarTodos(1, 2, 3, 4);
```

### 4.3 Parámetros como objeto (desestructurados)

```typescript
interface OpcionesBusqueda {
  texto: string;
  pagina?: number;
  limite?: number;
}

function buscar({ texto, pagina = 1, limite = 20 }: OpcionesBusqueda) {
  // ...
}

buscar({ texto: "camiseta", limite: 10 });
```

Ideal cuando hay más de 2–3 parámetros.

### 4.4 Tipos de retorno especiales

| Retorno | Cuándo |
|---|---|
| `void` | No devuelve nada |
| `Promise<T>` | Función `async` que resuelve a `T` |
| `never` | Siempre lanza un error |
| `T \| undefined` | Puede no encontrar nada |

```typescript
async function obtenerUsuario(id: number): Promise<Usuario> {
  const res = await fetch(`/api/usuarios/${id}`);
  return res.json();
}

function fallar(mensaje: string): never {
  throw new Error(mensaje);
}
```

### 4.5 Callbacks

```typescript
function repetir(veces: number, accion: (indice: number) => void) {
  for (let i = 0; i < veces; i++) accion(i);
}

repetir(3, (i) => console.log(`Vuelta ${i}`));
```

### 4.6 Sobrecargas

Varias firmas para una misma función:

```typescript
function formatear(valor: number): string;
function formatear(valor: Date): string;
function formatear(valor: number | Date): string {
  return typeof valor === "number" ? valor.toFixed(2) : valor.toISOString();
}
```

Muchas veces basta con una unión; usa sobrecargas solo si el retorno cambia según el parámetro.

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```typescript
function calcularIVA(precio: number, porcentaje: number = 21): number {
  return precio * (porcentaje / 100);
}

calcularIVA(100);      // 21
calcularIVA(100, 10);  // 10
calcularIVA("100");    // ✘ Error
```

### 5.2 Ejemplo habitual: funciones de un carrito

```typescript
interface LineaCarrito {
  producto: string;
  precio: number;
  cantidad: number;
}

const subtotal = (linea: LineaCarrito): number => linea.precio * linea.cantidad;

function totalCarrito(lineas: LineaCarrito[], descuento = 0): number {
  const bruto = lineas.reduce((suma, l) => suma + subtotal(l), 0);
  return Math.round(bruto * (1 - descuento) * 100) / 100;
}

function buscarLinea(lineas: LineaCarrito[], nombre: string): LineaCarrito | undefined {
  return lineas.find((l) => l.producto === nombre);
}
```

---

## 6. Buenas prácticas

- **Tipa siempre los parámetros**.
- **Anota el retorno** de funciones exportadas o complejas.
- **Objeto de opciones** cuando haya muchos parámetros o varios opcionales.
- **Evita `Function`** como tipo: usa la firma concreta `(x: number) => string`.
- **`Promise<T>`** en todas las funciones `async`, con `T` concreto.

---

## 7. Diferencias importantes

### 7.1 Opcional (`?`) vs. valor por defecto

| | `saludo?: string` | `saludo = "Hola"` |
|---|---|---|
| Tipo dentro de la función | `string \| undefined` | `string` |
| Hay que comprobar `undefined` | Sí | No |

### 7.2 `void` vs. `undefined` en callbacks

Un callback tipado `() => void` puede devolver cualquier cosa (se ignora). Por eso `array.forEach((x) => lista.push(x))` es válido aunque `push` devuelva un número.

### 7.3 Declaración vs. expresión con tipo

```typescript
function f(a: number): number { ... }            // declaración
const f: (a: number) => number = (a) => ...;     // tipo aplicado a la variable
```

La segunda permite reutilizar un `type` de función para varias implementaciones.

---

## 8. Casos especiales

### 8.1 `this` en funciones

```typescript
function mostrar(this: HTMLButtonElement) {
  console.log(this.textContent);
}
```

### 8.2 Funciones que comprueban tipos (*type guards*)

```typescript
function esTexto(valor: unknown): valor is string {
  return typeof valor === "string";
}
```

Ver [[TypeScript 05 - Uniones y narrowing]].

### 8.3 Funciones genéricas

Cuando una función debe funcionar con cualquier tipo manteniendo la relación entre entrada y salida, se usan genéricos: ver [[TypeScript 06 - Genéricos]].

---

## 9. Resumen

- Parámetros con tipo siempre; el retorno se infiere pero conviene anotarlo.
- `?` opcional, `= valor` por defecto, `...rest: T[]` para muchos argumentos.
- Objeto de opciones cuando haya muchos parámetros.
- `Promise<T>` para `async`; `void` sin retorno; `never` si siempre lanza.
- Tipos de función: `(a: number) => string`.
- Siguiente paso: [[TypeScript 05 - Uniones y narrowing]].
