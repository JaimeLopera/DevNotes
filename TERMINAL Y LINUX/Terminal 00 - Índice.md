# 00 - Índice

> [!info] Terminal y Linux
> La **terminal** es una ventana donde le das órdenes al ordenador escribiendo en lugar de hacer clic. **Linux** es el sistema operativo de casi todos los servidores del mundo, así que saber moverse por su terminal es imprescindible para desarrollar, desplegar y depurar aplicaciones web.

---

## 1. Antes de empezar

- No necesitas conocimientos previos.
- En **Windows**, usa **Git Bash** (viene con Git) o **WSL** (Linux dentro de Windows). Así los comandos de esta carpeta funcionan igual que en un servidor.
- En **macOS** y **Linux**, abre la aplicación *Terminal*.

---

## 2. Notas de la carpeta

| Nota | De qué trata |
|---|---|
| [[Terminal 01 - Fundamentos de la terminal]] | Qué es la terminal, la shell, el prompt y cómo se escribe un comando |
| [[Terminal 02 - Navegación y archivos]] | Moverse entre carpetas, crear, copiar, mover y borrar archivos |
| [[Terminal 03 - Permisos y usuarios]] | Quién puede leer, escribir o ejecutar cada archivo; `sudo` y `chmod` |
| [[Terminal 04 - Buscar y procesar texto]] | `grep`, `find`, tuberías (`\|`), redirecciones y `sed` |
| [[Terminal 05 - Procesos y red]] | Ver y parar programas, puertos ocupados, `curl` y comprobaciones de red |
| [[Terminal 06 - Bash scripting]] | Automatizar tareas con scripts: variables, condiciones y bucles |
| [[Terminal 07 - SSH y servidores]] | Conectarse a un servidor remoto, claves SSH y copiar archivos |
| [[Terminal 08 - Referencia rápida]] | Chuleta de comandos de toda la carpeta |

---

## 3. Orden recomendado

1. **Fundamentos** para perder el miedo a la pantalla negra.
2. **Navegación y archivos**: es el 80 % del uso diario.
3. **Buscar y procesar texto**: te ahorra horas.
4. **Permisos y usuarios** cuando aparezca el primer *Permission denied*.
5. **Procesos y red** cuando necesites arrancar servidores o liberar un puerto.
6. **SSH y servidores** para trabajar en una máquina remota.
7. **Bash scripting** para automatizar lo que repites.

> [!tip] Consulta rápida
> Si solo necesitas recordar un comando, ve directo a [[Terminal 08 - Referencia rápida]].

---

## 4. Dónde se usa

| Tema | Se usa sobre todo en |
|---|---|
| Navegación y archivos | Todo: Git, Node.js, Python, Docker |
| Buscar texto | Depurar logs, buscar en proyectos grandes |
| Permisos | Servidores, Docker, scripts que no se ejecutan |
| Procesos y red | Servidores de desarrollo, puertos ocupados, APIs |
| SSH | Despliegues, servidores, GitHub |

---

## 5. Notas relacionadas en otras carpetas

- [[Git 00 - Índice|Git]] → se maneja casi siempre desde la terminal.
- [[Node 00 - Índice|Node.js y npm]] → `node` y `npm` son comandos de terminal.
- [[Docker 00 - Índice|Docker]] → todo Docker se usa desde la terminal.
- [[Conceptos 04 - Regex]] → los patrones que usan `grep` y `sed`.
- [[00 - Ruta de aprendizaje]] → dónde encaja esta carpeta en el plan general.

---

## 6. Resumen

- La terminal permite controlar el ordenador con **comandos de texto**.
- Linux es el sistema de los servidores: aprender su terminal sirve para todo el desarrollo web.
- En Windows, trabaja con **Git Bash** o **WSL**.
- Empieza por fundamentos y navegación; el resto se aprende cuando hace falta.
