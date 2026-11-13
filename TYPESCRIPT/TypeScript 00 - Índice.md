# 00 - Índice

> [!info] TypeScript
> **TypeScript** es **JavaScript con tipos**. Añade a JavaScript la posibilidad de indicar qué tipo de dato tiene cada variable, parámetro o propiedad, y comprueba **antes de ejecutar** que no te equivocas. Al final se convierte en JavaScript normal.

---

## 1. Antes de empezar

- Hay que saber JavaScript con soltura: [[JavaScript 00 - Índice|JavaScript]] (funciones, objetos, arrays, clases, módulos).
- Para instalarlo y compilarlo necesitas Node y npm: [[Node 00 - Índice|Node.js y npm]].

---

## 2. Notas de la carpeta

| Nota | De qué trata |
|---|---|
| [[TypeScript 01 - Fundamentos]] | Qué es TypeScript, por qué usarlo, instalar y compilar |
| [[TypeScript 02 - Tipos básicos]] | `string`, `number`, `boolean`, arrays, tuplas, `any`, `unknown`, inferencia |
| [[TypeScript 03 - Interfaces y type]] | Dar forma a los objetos: propiedades opcionales, de solo lectura y extensión |
| [[TypeScript 04 - Funciones]] | Tipar parámetros, retornos, callbacks y sobrecargas |
| [[TypeScript 05 - Uniones y narrowing]] | Valores que pueden ser de varios tipos y cómo distinguirlos |
| [[TypeScript 06 - Genéricos]] | Código reutilizable que funciona con cualquier tipo |
| [[TypeScript 07 - Clases]] | Clases con tipos: modificadores de acceso, interfaces y abstractas |
| [[TypeScript 08 - Utility types]] | `Partial`, `Pick`, `Omit`, `Record` y compañía |
| [[TypeScript 09 - tsconfig y configuración]] | El archivo `tsconfig.json` y las opciones importantes |
| [[TypeScript 10 - TypeScript en proyectos]] | Usarlo con el DOM, con Node/Express, con `fetch` y con librerías |
| [[TypeScript 11 - Referencia rápida]] | Chuleta de sintaxis de toda la carpeta |

---

## 3. Orden recomendado

1. **Fundamentos** y **Tipos básicos**: lo mínimo para empezar.
2. **Interfaces y type** y **Funciones**: con esto ya escribes el 80 % del código.
3. **Uniones y narrowing**: lo que hace a TypeScript realmente útil.
4. **tsconfig** y **TypeScript en proyectos** para usarlo de verdad.
5. **Genéricos**, **Clases** y **Utility types** cuando los necesites.

> [!tip] Consulta rápida
> Si solo necesitas recordar una sintaxis, ve a [[TypeScript 11 - Referencia rápida]].

---

## 4. Dónde se usa

| Ámbito | Ejemplos |
|---|---|
| Frontend | React, Vue, Angular (Angular lo exige) |
| Backend | Node con Express, NestJS |
| Herramientas | Vite, Astro y Next.js lo traen configurado |
| Librerías | La mayoría publican sus tipos |

---

## 5. Notas relacionadas en otras carpetas

- [[JavaScript 18 - JavaScript moderno|JavaScript moderno]] → la base sobre la que se añaden los tipos.
- [[Node 02 - npm y package.json|npm y package.json]] → instalar TypeScript y tipos.
- [[Testing 00 - Índice|Testing]] → tipos y tests se complementan.
- [[00 - Ruta de aprendizaje]] → dónde encaja TypeScript en el plan general.

---

## 6. Resumen

- TypeScript = JavaScript + **tipos** comprobados antes de ejecutar.
- Detecta errores al escribir, mejora el autocompletado y documenta el código.
- Se compila a JavaScript normal: el navegador y Node no ejecutan TypeScript directamente.
- Empieza por tipos básicos, interfaces y funciones.
