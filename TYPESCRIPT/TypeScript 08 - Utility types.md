# 08 - Utility types

> [!info] ¿Qué es?
> Los **utility types** son tipos genéricos que TypeScript trae incluidos para **crear tipos nuevos a partir de otros**: hacer todas las propiedades opcionales, quedarse con unas pocas, quitar algunas, etc. Evitan repetir definiciones casi iguales.

---

## 1. Antes de empezar

Conviene haber leído [[TypeScript 03 - Interfaces y type]] y [[TypeScript 06 - Genéricos]].

---

## 2. Concepto fundamental

Partiendo de un único tipo base:

```typescript
interface Usuario {
  id: number;
  nombre: string;
  email: string;
  password: string;
  creadoEn: Date;
}
```

Puedes derivar todas las variantes que necesites sin copiar propiedades:

| Necesito… | Tipo derivado |
|---|---|
| Datos para crear un usuario (sin `id` ni fecha) | `Omit<Usuario, "id" \| "creadoEn">` |
| Datos para editar (todo opcional) | `Partial<Usuario>` |
| Lo que se muestra en público (sin contraseña) | `Omit<Usuario, "password">` |
| Solo nombre y email | `Pick<Usuario, "nombre" \| "email">` |

Si cambias `Usuario`, todas las variantes se actualizan solas.

---

## 3. Sintaxis / estructura

```typescript
type Nuevo = UtilityType<TipoBase, ...opciones>;
```

---

## 4. Elementos / propiedades / características

### 4.1 Para objetos

| Utility | Qué hace | Ejemplo |
|---|---|---|
| `Partial<T>` | Todas las propiedades opcionales | `Partial<Usuario>` |
| `Required<T>` | Todas obligatorias | `Required<Config>` |
| `Readonly<T>` | Todas de solo lectura | `Readonly<Usuario>` |
| `Pick<T, K>` | Solo las propiedades indicadas | `Pick<Usuario, "id" \| "nombre">` |
| `Omit<T, K>` | Todas menos las indicadas | `Omit<Usuario, "password">` |
| `Record<K, V>` | Objeto con claves `K` y valores `V` | `Record<string, number>` |

### 4.2 Para uniones

| Utility | Qué hace | Ejemplo | Resultado |
|---|---|---|---|
| `Exclude<T, U>` | Quita miembros de una unión | `Exclude<"a" \| "b" \| "c", "a">` | `"b" \| "c"` |
| `Extract<T, U>` | Se queda con los comunes | `Extract<"a" \| "b", "a" \| "z">` | `"a"` |
| `NonNullable<T>` | Quita `null` y `undefined` | `NonNullable<string \| null>` | `string` |

### 4.3 Para funciones

| Utility | Qué obtiene | Ejemplo |
|---|---|---|
| `ReturnType<F>` | El tipo de retorno | `ReturnType<typeof crearUsuario>` |
| `Parameters<F>` | Los parámetros como tupla | `Parameters<typeof buscar>` |
| `Awaited<T>` | El valor dentro de una promesa | `Awaited<Promise<string>>` → `string` |

### 4.4 Combinarlos

```typescript
// Para editar: el id es obligatorio y lo demás opcional
type EditarUsuario = Pick<Usuario, "id"> & Partial<Omit<Usuario, "id">>;
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: configuración con valores por defecto

```typescript
interface Config {
  tema: "claro" | "oscuro";
  idioma: string;
  notificaciones: boolean;
}

const porDefecto: Config = { tema: "claro", idioma: "es", notificaciones: true };

function crearConfig(opciones: Partial<Config>): Config {
  return { ...porDefecto, ...opciones };
}

crearConfig({ tema: "oscuro" }); // solo pasas lo que cambia
```

### 5.2 Ejemplo habitual: tipos de una API de usuarios

```typescript
type UsuarioPublico = Omit<Usuario, "password">;
type CrearUsuarioDTO = Omit<Usuario, "id" | "creadoEn">;
type ActualizarUsuarioDTO = Partial<CrearUsuarioDTO>;

async function crear(datos: CrearUsuarioDTO): Promise<UsuarioPublico> { /* ... */ }
async function actualizar(id: number, cambios: ActualizarUsuarioDTO) { /* ... */ }
```

### 5.3 `Record` para diccionarios

```typescript
type Rol = "admin" | "editor" | "lector";

const permisos: Record<Rol, string[]> = {
  admin: ["leer", "escribir", "borrar"],
  editor: ["leer", "escribir"],
  lector: ["leer"],
};
// Si falta un rol o sobra uno, TypeScript avisa
```

---

## 6. Buenas prácticas

- **Un tipo base y variantes derivadas**: nunca copies propiedades a mano.
- **`Omit` para quitar datos sensibles** de lo que se envía al cliente.
- **`Partial` para actualizaciones** y opciones con valores por defecto.
- **`Record` con una unión de claves** garantiza que no falta ninguna.
- **Da nombre** a las combinaciones complejas (`type EditarUsuario = ...`) en lugar de repetirlas.

---

## 7. Diferencias importantes

### 7.1 `Pick` vs. `Omit`

| | `Pick` | `Omit` |
|---|---|---|
| Indicas | Las que **quieres** | Las que **no quieres** |
| Mejor cuando | Quieres pocas | Quieres casi todas |

### 7.2 `Omit` vs. `Exclude`

`Omit` quita **propiedades de un objeto**. `Exclude` quita **miembros de una unión**.

### 7.3 `Readonly<T>` vs. `as const`

`Readonly<T>` es un tipo que aplicas a una interface. `as const` convierte un valor concreto en literal de solo lectura (y profundo).

---

## 8. Casos especiales

### 8.1 `Partial` no es profundo

`Partial<T>` solo hace opcionales las propiedades del **primer nivel**. Los objetos anidados siguen exigiendo todas sus propiedades.

### 8.2 `Omit` no avisa de claves inexistentes

`Omit<Usuario, "pasword">` (mal escrito) no da error: simplemente no quita nada. Revisa los nombres.

### 8.3 Crear tus propios utility types

```typescript
type Nullable<T> = { [K in keyof T]: T[K] | null };
```

(Son *mapped types*: recorren las claves de un tipo.)

---

## 9. Resumen

- Los utility types crean tipos nuevos a partir de otros.
- `Partial`, `Required`, `Readonly` cambian todas las propiedades.
- `Pick` se queda con algunas; `Omit` quita algunas.
- `Record<K, V>` para diccionarios.
- `Exclude`, `Extract`, `NonNullable` para uniones.
- `ReturnType`, `Parameters`, `Awaited` para funciones.
- Siguiente paso: [[TypeScript 09 - tsconfig y configuración]].
