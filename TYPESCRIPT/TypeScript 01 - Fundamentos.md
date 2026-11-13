# 01 - Fundamentos

> [!info] ¿Qué es?
> **TypeScript** es un lenguaje creado por Microsoft que **amplía JavaScript** con un sistema de tipos. Todo código JavaScript válido es TypeScript válido; TypeScript solo añade anotaciones (`: string`, `: number`…) que permiten detectar errores **mientras escribes**, antes de ejecutar nada.

---

## 1. Antes de empezar

- Hay que dominar JavaScript: [[JavaScript 00 - Índice|JavaScript]].
- Necesitas Node y npm: [[Node 01 - Fundamentos|Fundamentos de Node]].

---

## 2. Concepto fundamental

```
archivo.ts  ──tsc (compilador)──►  archivo.js  ──►  navegador / Node
   │
   └── aquí se comprueban los tipos y se avisan los errores
```

1. Escribes código en archivos `.ts`.
2. El **compilador** (`tsc`) revisa los tipos.
3. Genera archivos `.js` **sin** los tipos, que es lo que se ejecuta.

Los tipos **solo existen al desarrollar**: no cambian cómo se ejecuta el código ni lo hacen más lento.

El mismo error, en JavaScript y en TypeScript:

```javascript
// JavaScript: falla al EJECUTAR (y quizá en producción)
function saludar(nombre) {
  return "Hola, " + nombre.toUpperCase();
}
saludar(42); // TypeError: nombre.toUpperCase is not a function
```

```typescript
// TypeScript: el error aparece AL ESCRIBIR, subrayado en el editor
function saludar(nombre: string) {
  return "Hola, " + nombre.toUpperCase();
}
saludar(42); // Error: Argument of type 'number' is not assignable to parameter of type 'string'
```

---

## 3. Sintaxis / estructura

Los tipos se escriben **después** del nombre, con dos puntos:

```typescript
let edad: number = 30;
const nombre: string = "Ana";

function sumar(a: number, b: number): number {
  return a + b;
}
```

Instalar y compilar:

```bash
npm i -D typescript
npx tsc --init          # crea tsconfig.json
npx tsc                 # compila todo el proyecto
npx tsc --watch         # recompila al guardar
npx tsc --noEmit        # solo comprueba tipos, sin generar .js
```

---

## 4. Elementos / propiedades / características

### 4.1 Qué aporta TypeScript

| Ventaja | Ejemplo |
|---|---|
| Detecta errores antes de ejecutar | Pasar un número donde va un texto |
| Autocompletado preciso | El editor sabe qué propiedades tiene cada objeto |
| Refactorizar con seguridad | Renombrar una propiedad avisa de todos los usos |
| Documentación viva | La firma de una función dice qué recibe y qué devuelve |
| Menos tests triviales | Los tipos ya cubren errores de "forma" de los datos |

### 4.2 Inferencia de tipos

No hace falta anotar todo: TypeScript **deduce** el tipo cuando puede.

```typescript
let ciudad = "Madrid";   // TypeScript infiere: string
ciudad = 42;             // Error: Type 'number' is not assignable to type 'string'

const numeros = [1, 2, 3];  // number[]
```

### 4.3 Formas de ejecutar TypeScript

| Forma | Uso |
|---|---|
| `tsc` + `node archivo.js` | Compilar y ejecutar el resultado |
| `npx tsx archivo.ts` | Ejecutar directamente en desarrollo |
| `node archivo.ts` (Node 23.6+) | Node elimina los tipos y ejecuta (sintaxis compatible) |
| Vite, Next.js, Astro… | Lo compilan automáticamente |

### 4.4 Archivos

| Extensión | Qué es |
|---|---|
| `.ts` | Código TypeScript |
| `.tsx` | TypeScript con JSX (React) |
| `.d.ts` | Solo declaraciones de tipos (sin código) |
| `tsconfig.json` | Configuración del compilador ([[TypeScript 09 - tsconfig y configuración]]) |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```typescript
// saludo.ts
function saludar(nombre: string, veces: number = 1): string {
  return `Hola, ${nombre}! `.repeat(veces);
}

console.log(saludar("Ana", 2));
```

```bash
npx tsx saludo.ts
# Hola, Ana! Hola, Ana!
```

### 5.2 Ejemplo habitual: el error que TypeScript evita

```typescript
interface Usuario {
  nombre: string;
  email: string;
}

function enviarCorreo(usuario: Usuario) {
  console.log(`Enviando a ${usuario.emial}`);
  // Error: Property 'emial' does not exist on type 'Usuario'. Did you mean 'email'?
}
```

En JavaScript este error tipográfico solo se descubriría enviando correos a `undefined`.

---

## 6. Buenas prácticas

- **Activa el modo estricto** (`"strict": true`): es donde TypeScript realmente ayuda.
- **Deja que infiera** tipos obvios; anota parámetros de funciones y lo que se exporta.
- **Evita `any`**: desactiva todas las comprobaciones (ver [[TypeScript 02 - Tipos básicos]]).
- **Lee los errores completos**: suelen indicar exactamente qué esperaba y qué recibió.
- **Ejecuta `tsc --noEmit`** en tus scripts de comprobación o en CI.

---

## 7. Diferencias importantes

### 7.1 JavaScript vs. TypeScript

| | JavaScript | TypeScript |
|---|---|---|
| Tipos | Dinámicos (se descubren al ejecutar) | Estáticos (se comprueban al escribir) |
| Errores de tipo | Al ejecutar | Al escribir/compilar |
| Se ejecuta directamente | Sí | No: se compila (o se le quitan los tipos) |
| Curva de aprendizaje | Menor | Algo mayor |

### 7.2 Comprobación de tipos vs. ejecución

TypeScript **no valida datos en tiempo de ejecución**. Si una API devuelve algo inesperado, los tipos no lo detectan: hay que validar (por ejemplo, con Zod).

### 7.3 `tsc` vs. `tsx`

`tsc` **comprueba tipos** y genera `.js`. `tsx` **ejecuta** `.ts` rápidamente pero **no comprueba tipos**.

---

## 8. Casos especiales

### 8.1 Migrar un proyecto de JavaScript

Se puede hacer poco a poco: activa `"allowJs": true` en `tsconfig.json` y renombra archivos de `.js` a `.ts` uno a uno.

### 8.2 Comprobar tipos en archivos `.js`

Añadiendo `// @ts-check` al principio de un `.js`, VS Code lo revisa usando comentarios JSDoc.

### 8.3 Silenciar un error concreto

```typescript
// @ts-expect-error: la librería tiene mal los tipos
llamadaRara();
```

Úsalo solo cuando sepas que el código es correcto.

---

## 9. Resumen

- TypeScript = JavaScript + tipos; se compila a JavaScript.
- Los tipos se escriben con `:` después del nombre.
- Detecta errores al escribir, no al ejecutar.
- Infiere tipos automáticamente cuando puede.
- `npm i -D typescript`, `npx tsc --init`, `npx tsc`.
- Activa `strict` y evita `any`.
- Siguiente paso: [[TypeScript 02 - Tipos básicos]].
