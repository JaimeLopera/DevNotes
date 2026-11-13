# 00 - Índice

> [!info] Git
> **Git** es un sistema de **control de versiones**: guarda un historial de todos los cambios de un proyecto, permite volver atrás, trabajar en varias cosas a la vez sin mezclarlas y colaborar con otras personas. **GitHub** es una web donde se guardan repositorios de Git en la nube.

---

## 1. Antes de empezar

- Instala Git desde git-scm.com (en Windows incluye **Git Bash**).
- Conviene manejarse con la terminal: [[Terminal 00 - Índice|Terminal y Linux]].

---

## 2. Notas de la carpeta

| Nota | De qué trata |
|---|---|
| [[Git 01 - Fundamentos]] | Qué es Git, repositorio, commit, las tres zonas y la configuración inicial |
| [[Git 02 - Comandos básicos]] | El ciclo diario: `status`, `add`, `commit`, `log`, `diff` |
| [[Git 03 - Ramas]] | Trabajar en paralelo con `branch` y `switch` |
| [[Git 04 - Fusionar y resolver conflictos]] | `merge`, `rebase` y cómo resolver conflictos |
| [[Git 05 - Repositorios remotos y GitHub]] | `clone`, `push`, `pull`, `fetch` y Pull Requests |
| [[Git 06 - Deshacer cambios]] | `restore`, `reset`, `revert`, `stash` y recuperar lo perdido |
| [[Git 07 - .gitignore y configuración]] | Qué no subir, alias y ajustes útiles |
| [[Git 08 - Flujo de trabajo y buenas prácticas]] | Mensajes de commit, ramas por tarea y flujos de equipo |
| [[Git 09 - Referencia rápida]] | Chuleta de comandos de toda la carpeta |

---

## 3. Orden recomendado

1. **Fundamentos** para entender las tres zonas (es la base de todo).
2. **Comandos básicos**: con esto ya puedes usar Git a diario.
3. **.gitignore y configuración** antes de tu primer proyecto real.
4. **Repositorios remotos y GitHub** para subir tu trabajo.
5. **Ramas** y **Fusionar** cuando trabajes en varias cosas o en equipo.
6. **Deshacer cambios** cuando algo salga mal (pasará).
7. **Flujo de trabajo** para hacerlo como en una empresa.

> [!tip] Consulta rápida
> Si solo necesitas recordar un comando, ve a [[Git 09 - Referencia rápida]].

---

## 4. Situaciones frecuentes

| Necesito… | Nota |
|---|---|
| Guardar mis cambios | [[Git 02 - Comandos básicos]] |
| Subir el proyecto a GitHub | [[Git 05 - Repositorios remotos y GitHub]] |
| Probar algo sin romper lo que funciona | [[Git 03 - Ramas]] |
| Arreglar un conflicto | [[Git 04 - Fusionar y resolver conflictos]] |
| Deshacer un commit | [[Git 06 - Deshacer cambios]] |
| Que no se suba `node_modules` o `.env` | [[Git 07 - .gitignore y configuración]] |

---

## 5. Notas relacionadas en otras carpetas

- [[Terminal 07 - SSH y servidores|SSH y servidores]] → autenticarse en GitHub con claves SSH.
- [[Markdown 09 - Markdown en GitHub]] → escribir READMEs y documentación.
- [[Errores 06 - Git|Errores de Git]] → mensajes de error típicos y su solución.
- [[00 - Ruta de aprendizaje]] → dónde encaja Git en el plan general.

---

## 6. Resumen

- Git guarda el **historial** de un proyecto; GitHub lo guarda **en la nube**.
- El ciclo diario es: modificar → `git add` → `git commit` → `git push`.
- Las **ramas** permiten trabajar en paralelo sin romper lo que funciona.
- Casi todo se puede deshacer si se ha hecho commit.
