# 00 - Ruta de aprendizaje

> [!info] ¿Qué es?
> El **mapa general** de todos los apuntes de desarrollo web: en qué orden estudiar cada carpeta, qué necesitas saber antes de cada una y cómo encajan entre sí. Es el punto de partida de toda la documentación.

---

## 1. Antes de empezar

- No necesitas conocimientos previos: la ruta empieza desde cero.
- Cada carpeta tiene su **índice** (`00 - Índice`) con el orden de sus notas y una **Referencia rápida** al final.
- No hace falta terminar una carpeta entera para pasar a la siguiente: basta con sus fundamentos.

---

## 2. Mapa general

```
                    ┌────────────────────────┐
                    │  CONCEPTOS GENERALES   │  (internet, HTTP, JSON, APIs)
                    └───────────┬────────────┘
                                │
     ┌──────────────────────────┼──────────────────────────┐
     ▼                          ▼                          ▼
 HERRAMIENTAS               FRONTEND                    BACKEND
 ────────────               ────────                    ───────
 Terminal y Linux           HTML                        Node.js y npm
 Git                        CSS                         PHP
 Markdown                   JavaScript                  Python
                            TypeScript                  SQL
     │                          │                          │
     └──────────────────────────┼──────────────────────────┘
                                ▼
                       CALIDAD Y DESPLIEGUE
                       ────────────────────
                       Testing · Docker

        Consulta transversal: SNIPPETS · ERRORES COMUNES
```

---

## 3. Ruta recomendada por etapas

### Etapa 1 — Base (imprescindible)

| Orden | Carpeta | Por qué ahora |
|---|---|---|
| 1 | [[Conceptos 00 - Índice\|Conceptos generales]] | Entender cómo funciona la web antes de construirla |
| 2 | [[HTML 00 - Índice\|HTML]] | La estructura de cualquier página |
| 3 | [[CSS 00 - Índice\|CSS]] | El aspecto y el diseño responsive |
| 4 | [[Markdown 00 - Índice\|Markdown]] | Para documentar (y para estos mismos apuntes) |

### Etapa 2 — Programación y herramientas

| Orden | Carpeta | Por qué ahora |
|---|---|---|
| 5 | [[JavaScript 00 - Índice\|JavaScript]] | Dar comportamiento a las páginas |
| 6 | [[Terminal 00 - Índice\|Terminal y Linux]] | Todas las herramientas se usan desde aquí |
| 7 | [[Git 00 - Índice\|Git]] | Guardar el historial y subir proyectos a GitHub |

### Etapa 3 — Backend y datos (elige un lenguaje para empezar)

| Orden | Carpeta | Por qué ahora |
|---|---|---|
| 8 | [[SQL 00 - Índice\|SQL]] | Guardar y consultar datos; lo usan todos los backends |
| 9a | [[Node 00 - Índice\|Node.js y npm]] | Backend con el mismo lenguaje que el frontend |
| 9b | [[PHP 00 - Índice\|PHP]] | Backend clásico, WordPress, hosting barato |
| 9c | [[Python 00 - Índice\|Python]] | Backend, automatización y datos |

### Etapa 4 — Profesionalizar

| Orden | Carpeta | Por qué ahora |
|---|---|---|
| 10 | [[TypeScript 00 - Índice\|TypeScript]] | Menos errores en proyectos medianos y grandes |
| 11 | [[Testing 00 - Índice\|Testing]] | Cambiar código sin miedo a romperlo |
| 12 | [[Docker 00 - Índice\|Docker]] | Entornos reproducibles y despliegue |

### Consulta en cualquier momento

| Carpeta | Para qué |
|---|---|
| [[Snippets 00 - Índice\|Snippets]] | Código listo para copiar |
| [[Errores 00 - Índice\|Errores comunes]] | Mensajes de error y su solución |

---

## 4. Rutas según objetivo

| Quiero… | Ruta mínima |
|---|---|
| Hacer webs estáticas (portfolio, landing) | Conceptos → HTML → CSS → JavaScript básico → Git |
| Ser desarrollador frontend | Lo anterior + JavaScript completo → TypeScript → Testing |
| Ser desarrollador backend | Conceptos → JavaScript o Python → Terminal → Git → SQL → Node.js o Python → Testing → Docker |
| Trabajar con WordPress | HTML → CSS → JavaScript básico → PHP → SQL |
| *Full stack* | Las cuatro etapas en orden |

---

## 5. Cómo estudiar cada carpeta

1. Lee el **índice** para tener el mapa.
2. Sigue las notas **en orden** la primera vez.
3. **Escribe el código** de los ejemplos, no solo lo leas.
4. Haz un **mini proyecto** al terminar cada carpeta (una landing, una API de tareas, una base de datos de una tienda…).
5. Usa la **Referencia rápida** para repasar.
6. Cuando algo falle, busca en **Errores comunes**.

> [!tip] Proyectos que unen varias carpetas
> - **Lista de tareas**: HTML + CSS + JavaScript + localStorage.
> - **API de tareas**: Node.js + Express + SQL + Testing.
> - **Tienda sencilla**: PHP + SQL + formularios + sesiones.
> - **Despliegue**: cualquiera de los anteriores con Git + Docker.

---

## 6. Resumen

- Empieza por **Conceptos generales**, **HTML** y **CSS**.
- Sigue con **JavaScript**, **Terminal** y **Git**.
- Elige un backend (**Node.js**, **PHP** o **Python**) y aprende **SQL**.
- Profesionaliza con **TypeScript**, **Testing** y **Docker**.
- **Snippets** y **Errores comunes** sirven en cualquier etapa.
