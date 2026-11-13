# 09 - tsconfig y configuración

> [!info] ¿Qué es?
> **`tsconfig.json`** es el archivo que le dice al compilador de TypeScript **cómo comportarse**: qué archivos revisar, qué versión de JavaScript generar, dónde dejar el resultado y qué tan estricto ser. Cada proyecto TypeScript tiene uno en la raíz.

---

## 1. Antes de empezar

Conviene haber leído [[TypeScript 01 - Fundamentos]] (qué hace `tsc`).

---

## 2. Concepto fundamental

```
tsconfig.json
   ├── ¿Qué archivos?         → include / exclude
   ├── ¿Qué JavaScript genero? → target / module
   ├── ¿Dónde lo dejo?         → outDir / rootDir
   └── ¿Qué tan estricto?      → strict y compañía
```

Cuando ejecutas `npx tsc` sin argumentos, busca el `tsconfig.json` de la carpeta y aplica su configuración. El editor (VS Code) también lo lee para saber qué errores mostrar.

---

## 3. Sintaxis / estructura

```bash
npx tsc --init   # genera un tsconfig.json comentado
```

Configuración recomendada para un proyecto de Node moderno:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 4. Elementos / propiedades / características

### 4.1 Opciones de salida

| Opción | Qué hace | Valor habitual |
|---|---|---|
| `target` | Versión de JavaScript generada | `ES2022` |
| `module` | Sistema de módulos generado | `NodeNext` (Node), `ESNext` (con bundler) |
| `moduleResolution` | Cómo se buscan las importaciones | `NodeNext` o `Bundler` |
| `outDir` | Carpeta de los `.js` generados | `dist` |
| `rootDir` | Carpeta de los `.ts` fuente | `src` |
| `sourceMap` | Genera mapas para depurar en el `.ts` | `true` |
| `declaration` | Genera archivos `.d.ts` (para librerías) | `true` en librerías |
| `noEmit` | No genera nada, solo comprueba | `true` con Vite/Next |

### 4.2 Opciones de rigor

| Opción | Qué hace |
|---|---|
| `strict` | Activa todas las comprobaciones estrictas (**recomendado**) |
| `noImplicitAny` | Prohíbe `any` implícito (incluida en `strict`) |
| `strictNullChecks` | Obliga a tratar `null`/`undefined` (incluida en `strict`) |
| `noUnusedLocals` | Error en variables sin usar |
| `noUnusedParameters` | Error en parámetros sin usar |
| `noUncheckedIndexedAccess` | `array[i]` puede ser `undefined` |
| `noImplicitReturns` | Todas las ramas deben devolver algo |

### 4.3 Opciones de entorno

| Opción | Para qué |
|---|---|
| `lib` | APIs disponibles: `["DOM", "ES2022"]` para navegador |
| `types` | Tipos globales a incluir: `["node"]`, `["vitest/globals"]` |
| `jsx` | Cómo tratar JSX: `react-jsx` |
| `esModuleInterop` | Facilita importar paquetes CommonJS con `import x from` |
| `skipLibCheck` | No revisa los `.d.ts` de las librerías (más rápido) |
| `allowJs` | Permite mezclar `.js` y `.ts` |
| `baseUrl` / `paths` | Alias de importación (`@/utils`) |

### 4.4 Archivos incluidos

| Campo | Para qué |
|---|---|
| `include` | Carpetas o patrones a revisar |
| `exclude` | Lo que se ignora (`node_modules` por defecto) |
| `files` | Lista exacta de archivos (poco usado) |
| `extends` | Hereda de otro tsconfig |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: proyecto de Node con scripts

```json
// package.json
{
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "typecheck": "tsc --noEmit"
  },
  "devDependencies": {
    "typescript": "^5.6.0",
    "tsx": "^4.19.0",
    "@types/node": "^22.0.0"
  }
}
```

### 5.2 Ejemplo habitual: frontend con Vite

Vite compila TypeScript por su cuenta; `tsc` solo comprueba tipos:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

### 5.3 Alias de rutas

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

```typescript
import { formatear } from "@/utils/formato";
```

(El bundler o el runtime también deben conocer el alias.)

---

## 6. Buenas prácticas

- **Activa `strict: true`** desde el principio; activarlo después cuesta mucho más.
- **Usa `skipLibCheck: true`**: acelera y evita errores de librerías ajenas.
- **Separa `src/` y `dist/`** con `rootDir` y `outDir`.
- **Añade `dist/` al `.gitignore`**.
- **Script `typecheck`** (`tsc --noEmit`) para comprobar tipos en CI.
- **Parte de una plantilla** (Vite, `@tsconfig/node22`) en lugar de configurar todo a mano.

---

## 7. Diferencias importantes

### 7.1 `target` vs. `lib`

| | `target` | `lib` |
|---|---|---|
| Controla | El JavaScript que **se genera** | Las APIs que **puedes usar** sin error |
| Ejemplo | `ES2022` genera `class`, `?.`, etc. | `DOM` permite `document`, `window` |

### 7.2 `module: NodeNext` vs. `ESNext` + `Bundler`

| | `NodeNext` | `ESNext` + `Bundler` |
|---|---|---|
| Para | Node ejecutando el resultado | Vite, webpack, Next.js |
| Extensión en importaciones | Obligatoria (`./a.js`) | No necesaria |

### 7.3 `tsc` vs. `tsc --noEmit`

`tsc` comprueba y **genera** `.js`. `--noEmit` solo **comprueba**.

---

## 8. Casos especiales

### 8.1 *Cannot find name 'document'*

Falta `"DOM"` en `lib`.

### 8.2 *Cannot find name 'process'* o *'require'*

Faltan los tipos de Node: `npm i -D @types/node`.

### 8.3 Importar `.ts` con extensión `.js`

Con `NodeNext`, se importa `import { x } from "./utils.js"` aunque el archivo sea `utils.ts`: la ruta se refiere al archivo **generado**.

### 8.4 Varios tsconfig

Proyectos grandes usan `tsconfig.json` base y otros que lo extienden (`tsconfig.build.json`, `tsconfig.test.json`) con `"extends": "./tsconfig.json"`.

---

## 9. Resumen

- `tsconfig.json` configura el compilador y el editor.
- `target` = JavaScript generado; `lib` = APIs disponibles.
- `outDir`/`rootDir` separan `dist/` y `src/`.
- `strict: true` siempre; `skipLibCheck: true` casi siempre.
- `NodeNext` para Node; `Bundler` para Vite y similares.
- `tsc --noEmit` comprueba tipos sin generar archivos.
- Siguiente paso: [[TypeScript 10 - TypeScript en proyectos]].
