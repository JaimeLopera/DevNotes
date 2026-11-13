# 03 - Interfaces y type

> [!info] ¿Qué es?
> **`interface`** y **`type`** sirven para **dar nombre a la forma de un objeto**: qué propiedades tiene y de qué tipo es cada una. Así puedes reutilizar esa forma en todo el proyecto y TypeScript comprueba que cada objeto la cumple.

---

## 1. Antes de empezar

- Conviene haber leído [[TypeScript 02 - Tipos básicos]].
- Y conocer los objetos de JavaScript: [[JavaScript 08 - Objetos|Objetos en JavaScript]].

---

## 2. Concepto fundamental

Sin un tipo con nombre, repetirías la forma del objeto en cada sitio:

```typescript
function mostrar(usuario: { nombre: string; edad: number }) { /* ... */ }
```

Con una interface la defines **una vez** y la reutilizas:

```typescript
interface Usuario {
  nombre: string;
  edad: number;
}

function mostrar(usuario: Usuario) { /* ... */ }
const ana: Usuario = { nombre: "Ana", edad: 30 };
```

Si falta una propiedad, sobra una o tiene otro tipo, TypeScript avisa.

---

## 3. Sintaxis / estructura

```typescript
interface Producto {
  id: number;
  nombre: string;
  precio: number;
}

type Producto2 = {
  id: number;
  nombre: string;
  precio: number;
};
```

Por convención, los nombres van en **PascalCase** (`Producto`, `UsuarioAdmin`).

---

## 4. Elementos / propiedades / características

### 4.1 Propiedades opcionales y de solo lectura

```typescript
interface Usuario {
  readonly id: number;   // no se puede cambiar tras crearlo
  nombre: string;
  email: string;
  telefono?: string;     // opcional: puede no estar
}

const u: Usuario = { id: 1, nombre: "Ana", email: "ana@mail.com" }; // ✔ sin teléfono
u.id = 2; // ✘ Error: es readonly
```

### 4.2 Objetos anidados y arrays

```typescript
interface Pedido {
  id: number;
  cliente: {
    nombre: string;
    direccion: string;
  };
  productos: Producto[];
}
```

### 4.3 Extender (heredar)

```typescript
interface Persona {
  nombre: string;
}

interface Empleado extends Persona {
  puesto: string;
}
// Empleado tiene nombre y puesto
```

Con `type` se usa la intersección `&`:

```typescript
type Empleado = Persona & { puesto: string };
```

### 4.4 Métodos

```typescript
interface Calculadora {
  sumar(a: number, b: number): number;
  restar: (a: number, b: number) => number;
}
```

### 4.5 Firmas de índice (claves dinámicas)

```typescript
interface Traducciones {
  [clave: string]: string;
}

const es: Traducciones = { hola: "hola", adios: "adiós" };
```

(Muchas veces es más claro `Record<string, string>`: ver [[TypeScript 08 - Utility types]].)

### 4.6 Lo que solo puede hacer `type`

```typescript
type Id = number | string;              // uniones
type Coordenadas = [number, number];    // tuplas
type Estado = "activo" | "inactivo";    // literales
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```typescript
interface Libro {
  titulo: string;
  autor: string;
  paginas: number;
  leido?: boolean;
}

const libros: Libro[] = [
  { titulo: "El Quijote", autor: "Cervantes", paginas: 1200 },
  { titulo: "Rayuela", autor: "Cortázar", paginas: 600, leido: true },
];

const totalPaginas = libros.reduce((suma, l) => suma + l.paginas, 0);
```

### 5.2 Ejemplo habitual: tipar la respuesta de una API

```typescript
interface Comentario {
  id: number;
  postId: number;
  autor: string;
  texto: string;
  fecha: string;   // en JSON las fechas llegan como texto
}

async function obtenerComentarios(postId: number): Promise<Comentario[]> {
  const res = await fetch(`/api/posts/${postId}/comentarios`);
  return res.json();
}

const comentarios = await obtenerComentarios(1);
comentarios[0].autor; // autocompletado y comprobación
```

---

## 6. Buenas prácticas

- **Elige una convención y sé coherente**: muchos equipos usan `interface` para objetos y `type` para todo lo demás.
- **Nombres en PascalCase** y sin prefijo `I` (`Usuario`, no `IUsuario`).
- **Usa `readonly`** en lo que no debe cambiar (ids, configuración).
- **Opcionales solo si de verdad pueden faltar**: cada `?` obliga a comprobar `undefined` al usarla.
- **Define los tipos cerca de donde se usan**, o en un archivo `types.ts` si se comparten.

---

## 7. Diferencias importantes

### 7.1 `interface` vs. `type`

| | `interface` | `type` |
|---|---|---|
| Objetos | Sí | Sí |
| Uniones, tuplas, primitivos | No | Sí |
| Extender | `extends` | `&` (intersección) |
| Declararla dos veces | Se **fusionan** | Error |
| Clases pueden implementarla | Sí | Sí (si es un objeto) |

Para objetos, ambas sirven. Si dudas: `interface` para objetos, `type` para lo demás.

### 7.2 Opcional (`?`) vs. `| undefined`

```typescript
interface A { x?: string }            // la propiedad puede no existir
interface B { x: string | undefined } // la propiedad debe existir, aunque valga undefined
```

### 7.3 `readonly` vs. `const`

`const` impide **reasignar la variable**. `readonly` impide **cambiar una propiedad** del objeto.

---

## 8. Casos especiales

### 8.1 Propiedades de más en objetos literales

```typescript
const u: Usuario = { id: 1, nombre: "Ana", email: "a@a.com", edad: 30 };
// ✘ Error: 'edad' does not exist in type 'Usuario'
```

TypeScript es estricto con los objetos escritos directamente, para atrapar errores tipográficos.

### 8.2 Tipos compatibles por forma

TypeScript compara **la forma**, no el nombre: si un objeto tiene todas las propiedades que pide una interface, sirve, aunque no se declarase con ella (*tipado estructural*).

### 8.3 Obtener el tipo de un objeto existente

```typescript
const config = { puerto: 3000, host: "localhost" };
type Config = typeof config; // { puerto: number; host: string }
```

---

## 9. Resumen

- `interface` y `type` dan nombre a la forma de un objeto.
- `?` = opcional; `readonly` = no modificable.
- `interface` se extiende con `extends`; `type` con `&`.
- `type` además sirve para uniones, tuplas y literales.
- Tipa las respuestas de las APIs para tener autocompletado.
- Siguiente paso: [[TypeScript 04 - Funciones]].
