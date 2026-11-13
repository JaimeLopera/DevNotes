# 00 - Índice

> [!info] Testing
> **Testear** es escribir código que **comprueba automáticamente** que tu código funciona. En lugar de probar a mano cada vez que cambias algo, ejecutas un comando y en segundos sabes si has roto alguna cosa. Es lo que permite cambiar y mejorar un proyecto sin miedo.

---

## 1. Antes de empezar

- Hay que saber programar en el lenguaje que vas a testear: [[JavaScript 00 - Índice|JavaScript]] o [[Python 00 - Índice|Python]].
- Para JavaScript necesitas npm: [[Node 02 - npm y package.json|npm y package.json]].

---

## 2. Notas de la carpeta

| Nota | De qué trata |
|---|---|
| [[Testing 01 - Fundamentos]] | Qué es un test, tipos de tests, la pirámide y la estructura AAA |
| [[Testing 02 - Tests unitarios con Vitest]] | Escribir y ejecutar tests de JavaScript/TypeScript |
| [[Testing 03 - Mocks y espías]] | Sustituir dependencias (APIs, fechas, BD) para testear en aislamiento |
| [[Testing 04 - Tests en Python con pytest]] | El equivalente en Python |
| [[Testing 05 - Tests de integración y API]] | Probar varias piezas juntas y endpoints HTTP |
| [[Testing 06 - Tests end-to-end con Playwright]] | Probar la web completa en un navegador real |
| [[Testing 07 - TDD y buenas prácticas]] | Escribir el test primero, cobertura y qué testear |
| [[Testing 08 - Referencia rápida]] | Chuleta de toda la carpeta |

---

## 3. Orden recomendado

1. **Fundamentos** para entender los tipos de tests y su estructura.
2. **Tests unitarios con Vitest** (o **pytest** si trabajas en Python).
3. **Mocks y espías** cuando tu código dependa de cosas externas.
4. **Tests de integración y API** cuando construyas un backend.
5. **End-to-end** para los flujos críticos de una web.
6. **TDD y buenas prácticas** para hacerlo como un profesional.

> [!tip] Consulta rápida
> Si solo necesitas recordar una sintaxis, ve a [[Testing 08 - Referencia rápida]].

---

## 4. Herramientas por lenguaje

| Lenguaje | Unitarios / integración | End-to-end |
|---|---|---|
| JavaScript / TypeScript | **Vitest**, Jest | **Playwright**, Cypress |
| Python | **pytest** | Playwright para Python |
| PHP | PHPUnit, Pest | — |

---

## 5. Notas relacionadas en otras carpetas

- [[Git 08 - Flujo de trabajo y buenas prácticas|Flujo de trabajo en Git]] → ejecutar tests antes de fusionar una PR.
- [[Node 08 - API REST con Express|API REST con Express]] → la API que se testea en la nota de integración.
- [[Python 18 - Debugging y buenas prácticas]] (Python) → depurar cuando un test falla.
- [[00 - Ruta de aprendizaje]] → dónde encaja el testing en el plan general.

---

## 6. Resumen

- Un test es código que comprueba que otro código funciona.
- Hay tests **unitarios** (una pieza), de **integración** (varias piezas) y **end-to-end** (la app completa).
- En JavaScript: **Vitest** y **Playwright**; en Python: **pytest**.
- Los tests permiten cambiar el código sin miedo a romperlo.
