# 00 - Índice

> [!info] Node.js y npm
> **Node.js** permite ejecutar JavaScript **fuera del navegador**: en tu ordenador o en un servidor. Con él se crean servidores web, APIs, herramientas de terminal y scripts. **npm** es su gestor de paquetes: instala librerías de terceros y ejecuta las tareas del proyecto.

---

## 1. Antes de empezar

- Hay que saber JavaScript: [[JavaScript 00 - Índice|JavaScript]], sobre todo funciones, objetos, asincronía y módulos.
- Conviene manejarse con la terminal: [[Terminal 00 - Índice|Terminal y Linux]].
- Para construir APIs ayuda conocer [[Conceptos 01 - HTTP]] y [[Conceptos 02 - APIs REST]].

---

## 2. Notas de la carpeta

| Nota | De qué trata |
|---|---|
| [[Node 01 - Fundamentos]] | Qué es Node, instalarlo, ejecutar archivos y diferencias con el navegador |
| [[Node 02 - npm y package.json]] | Instalar paquetes, `package.json`, scripts y versiones |
| [[Node 03 - Módulos CommonJS y ESM]] | `require` vs. `import`, exportar e importar código |
| [[Node 04 - Servidor con Express]] | Crear un servidor web y una API con Express |
| [[Node 05 - Variables de entorno]] | Configuración y secretos con `.env` y `process.env` |
| [[Node 06 - Sistema de archivos y path]] | Leer y escribir archivos, trabajar con rutas |
| [[Node 07 - Asincronía y event loop en Node]] | Cómo Node hace muchas cosas a la vez con un solo hilo |
| [[Node 08 - API REST con Express]] | Una API completa: rutas, middleware, validación y errores |
| [[Node 09 - Referencia rápida]] | Chuleta de comandos y código de toda la carpeta |

---

## 3. Orden recomendado

1. **Fundamentos** para ejecutar tu primer archivo con Node.
2. **npm y package.json**: imprescindible para cualquier proyecto moderno (también de frontend).
3. **Módulos CommonJS y ESM** para dividir el código en archivos.
4. **Sistema de archivos** y **Asincronía**: la forma de trabajar de Node.
5. **Servidor con Express** y **Variables de entorno**.
6. **API REST con Express** para juntarlo todo.

> [!tip] Consulta rápida
> Si solo necesitas recordar un comando, ve a [[Node 09 - Referencia rápida]].

---

## 4. Para qué se usa Node.js

| Uso | Ejemplos |
|---|---|
| Servidores y APIs | Express, Fastify, NestJS |
| Herramientas de frontend | Vite, webpack, ESLint, Prettier |
| Frameworks *full stack* | Next.js, Nuxt, Astro |
| Scripts y automatización | Procesar archivos, tareas programadas |
| Aplicaciones de escritorio | Electron (VS Code está hecho con él) |

Aunque solo hagas frontend, usarás **npm** a diario para instalar librerías y arrancar el servidor de desarrollo.

---

## 5. Notas relacionadas en otras carpetas

- [[TypeScript 00 - Índice|TypeScript]] → JavaScript con tipos, muy usado con Node.
- [[Testing 00 - Índice|Testing]] → probar el código con Vitest o Jest.
- [[Docker 00 - Índice|Docker]] → empaquetar una app de Node en un contenedor.
- [[Errores 07 - Node.js y npm|Errores de Node.js y npm]] → mensajes típicos y su solución.
- [[Conceptos 03 - JSON]] y [[Conceptos 02 - APIs REST]] → la base de cualquier API.

---

## 6. Resumen

- **Node.js** ejecuta JavaScript fuera del navegador.
- **npm** instala paquetes y ejecuta scripts definidos en `package.json`.
- Se usa para backend, herramientas de frontend y scripts.
- Empieza por fundamentos y npm; después, Express para crear APIs.
