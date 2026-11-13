# 11 - Referencia rápida

> [!info] ¿Qué es?
> Chuleta de **TypeScript** para consultar rápido. No explica, solo recuerda. Si algo no te suena, sigue el enlace a su nota.

---

## 1. Instalación y compilación ([[TypeScript 01 - Fundamentos]])

```bash
npm i -D typescript tsx @types/node
npx tsc --init
npx tsc               # compilar
npx tsc --watch
npx tsc --noEmit      # solo comprobar tipos
npx tsx archivo.ts    # ejecutar sin compilar
```

---

## 2. Tipos básicos ([[TypeScript 02 - Tipos básicos]])

```typescript
let a: string = "x";
let b: number = 1;
let c: boolean = true;
let d: number[] = [1, 2];
let e: Array<string> = ["a"];
let f: [string, number] = ["Ana", 30];   // tupla
let g: "si" | "no" = "si";               // literal
let h: unknown;                          // comprobar antes de usar
let i: any;                              // evitar
function j(): void {}
function k(): never { throw new Error(); }
const l = ["a", "b"] as const;
```

---

## 3. Objetos ([[TypeScript 03 - Interfaces y type]])

```typescript
interface Usuario {
  readonly id: number;
  nombre: string;
  email?: string;
}
interface Admin extends Usuario { permisos: string[] }

type Punto = { x: number; y: number };
type Admin2 = Usuario & { permisos: string[] };
type Config = typeof objetoConfig;
```

---

## 4. Funciones ([[TypeScript 04 - Funciones]])

```typescript
function f(a: number, b = 2, c?: string): number { ... }
const g = (x: string): string => x;
function h(...nums: number[]) { ... }
function i({ texto, limite = 10 }: Opciones) { ... }
async function j(): Promise<Usuario[]> { ... }
type Callback = (valor: number) => void;
```

---

## 5. Uniones y narrowing ([[TypeScript 05 - Uniones y narrowing]])

```typescript
type Id = string | number;

if (typeof x === "string") { ... }
if (x instanceof Date) { ... }
if ("email" in obj) { ... }
if (x !== null) { ... }
function esGato(a: Animal): a is Gato { return "maullar" in a; }

type Estado =
  | { tipo: "ok"; datos: string[] }
  | { tipo: "error"; mensaje: string };

switch (e.tipo) { case "ok": ...; case "error": ...; }

valor?.propiedad;
valor ?? "por defecto";
```

---

## 6. Genéricos ([[TypeScript 06 - Genéricos]])

```typescript
function primero<T>(lista: T[]): T | undefined { return lista[0]; }
function largo<T extends { length: number }>(x: T) { return x.length; }
function obtener<T, K extends keyof T>(o: T, k: K): T[K] { return o[k]; }
interface Caja<T = unknown> { valor: T }
```

---

## 7. Clases ([[TypeScript 07 - Clases]])

```typescript
class Cuenta implements Almacen {
  static readonly LIMITE = 1000;
  #secreto = "";
  constructor(public titular: string, private saldo = 0) {}
  get total() { return this.saldo; }
  protected log() {}
}
abstract class Figura { abstract area(): number; }
class Circulo extends Figura { override area() { return 1; } }
```

---

## 8. Utility types ([[TypeScript 08 - Utility types]])

| Utility | Resultado |
|---|---|
| `Partial<T>` | Todo opcional |
| `Required<T>` | Todo obligatorio |
| `Readonly<T>` | Todo solo lectura |
| `Pick<T, "a" \| "b">` | Solo esas |
| `Omit<T, "a">` | Todas menos esa |
| `Record<K, V>` | Diccionario |
| `Exclude<U, X>` | Unión sin X |
| `NonNullable<T>` | Sin null/undefined |
| `ReturnType<typeof f>` | Retorno de f |
| `Parameters<typeof f>` | Parámetros de f |
| `Awaited<Promise<T>>` | T |

---

## 9. tsconfig ([[TypeScript 09 - tsconfig y configuración]])

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true
  },
  "include": ["src"]
}
```

Navegador con Vite: `"module": "ESNext"`, `"moduleResolution": "Bundler"`, `"lib": ["ES2022", "DOM"]`, `"noEmit": true`.

---

## 10. En proyectos ([[TypeScript 10 - TypeScript en proyectos]])

```typescript
const input = document.querySelector<HTMLInputElement>("#email");
import type { Tarea } from "./types.js";

import { z } from "zod";
const Schema = z.object({ id: z.number(), nombre: z.string() });
type Item = z.infer<typeof Schema>;
const item = Schema.parse(datos);
```

```bash
npm i -D @types/express @types/node
```
