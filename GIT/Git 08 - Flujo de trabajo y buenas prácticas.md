# 08 - Flujo de trabajo y buenas prácticas

> [!info] ¿Qué es?
> Un **flujo de trabajo** (*workflow*) es la forma acordada de usar Git en un proyecto: cómo se nombran las ramas, cómo se escriben los commits y cómo llega el código a `main`. Seguir uno evita el caos cuando trabaja más de una persona (o tú mismo dentro de seis meses).

---

## 1. Antes de empezar

Conviene dominar [[Git 03 - Ramas]], [[Git 04 - Fusionar y resolver conflictos]] y [[Git 05 - Repositorios remotos y GitHub]].

---

## 2. Concepto fundamental

El flujo más usado hoy es **GitHub Flow**, sencillo y válido para casi cualquier proyecto web:

```
main ──●─────────────●─────────────●──►   (siempre desplegable)
        \           /               \
         ●──●──●───●  feature/login  ●──●  fix/menu
           commits + Pull Request
```

1. `main` **siempre funciona** y se puede desplegar.
2. Cada tarea se hace en una **rama nueva** desde `main`.
3. Se hacen commits pequeños y se sube la rama.
4. Se abre una **Pull Request** para revisarla.
5. Se fusiona en `main` y se despliega.

---

## 3. Sintaxis / estructura

Formato de mensaje de commit (**Conventional Commits**):

```
tipo(ámbito opcional): descripción corta en imperativo

Cuerpo opcional: qué cambia y por qué.
```

```
feat(carrito): añade botón de vaciar carrito
fix: corrige el cálculo del IVA en el total
docs: actualiza instrucciones de instalación
```

---

## 4. Elementos / propiedades / características

### 4.1 Tipos de commit (Conventional Commits)

| Tipo | Para qué |
|---|---|
| `feat` | Funcionalidad nueva |
| `fix` | Corrección de un error |
| `docs` | Documentación |
| `style` | Formato (espacios, comas), sin cambiar lógica |
| `refactor` | Reorganizar código sin cambiar su comportamiento |
| `test` | Añadir o corregir tests |
| `chore` | Tareas de mantenimiento (dependencias, configuración) |
| `perf` | Mejora de rendimiento |

### 4.2 Un buen mensaje de commit

| Bien | Mal |
|---|---|
| `fix: evita doble envío del formulario` | `arreglos` |
| `feat: añade filtro por categoría` | `cambios varios` |
| `refactor: extrae validación a utils` | `asdf` |

Reglas:
- **Imperativo**: "añade", "corrige" (como si completaras "Este commit…").
- **Primera línea corta** (unos 50–72 caracteres).
- **El cuerpo explica el porqué**, no el cómo (el cómo ya está en el código).

### 4.3 Flujos de trabajo habituales

| Flujo | Ramas | Para quién |
|---|---|---|
| **GitHub Flow** | `main` + ramas de tarea | La mayoría de proyectos web |
| **Git Flow** | `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` | Proyectos con versiones planificadas |
| **Trunk-based** | Todo a `main` con ramas de horas | Equipos con muchos tests automáticos |

### 4.4 Revisión de código (Pull Requests)

Una buena PR:
- Es **pequeña** (fácil de revisar).
- Tiene un **título claro** y una descripción de qué cambia y cómo probarlo.
- Pasa los **tests automáticos** (ver [[Testing 00 - Índice|Testing]]).
- Está **actualizada** con `main`.

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: trabajando solo

```bash
git switch main && git pull
git switch -c feat/modo-oscuro
# ... cambios ...
git commit -m "feat: añade interruptor de modo oscuro"
git commit -m "style: ajusta colores del modo oscuro"
git switch main
git merge feat/modo-oscuro
git push
git branch -d feat/modo-oscuro
```

### 5.2 Ejemplo habitual: en equipo con Pull Request

```bash
git switch main && git pull
git switch -c fix/total-carrito
# ... corrección + test ...
git commit -m "fix: redondea el total del carrito a 2 decimales"
git fetch && git rebase origin/main    # me pongo al día
git push -u origin fix/total-carrito
# En GitHub: abrir PR → revisión → Merge → borrar rama
```

---

## 6. Buenas prácticas

- **Commits pequeños y atómicos**: un cambio lógico por commit.
- **Commit a menudo, push a menudo**.
- **Nunca subas** secretos, `node_modules` ni archivos generados.
- **Protege `main`** en GitHub (Settings → Branches): obliga a usar PR y a que pasen los tests.
- **Pull antes de empezar**, rebase antes de abrir PR.
- **Revisa tu propio diff** antes de pedir revisión.
- **Escribe un README** útil: qué es, cómo instalar, cómo arrancar.

---

## 7. Diferencias importantes

### 7.1 GitHub Flow vs. Git Flow

| | GitHub Flow | Git Flow |
|---|---|---|
| Ramas principales | Solo `main` | `main` y `develop` |
| Complejidad | Baja | Alta |
| Despliegue | Continuo | Por versiones |
| Recomendado para empezar | Sí | No |

### 7.2 Squash merge vs. merge commit vs. rebase merge (en GitHub)

| Opción | Resultado en `main` |
|---|---|
| **Create a merge commit** | Todos los commits de la rama + un commit de merge |
| **Squash and merge** | Un solo commit con todo el trabajo de la PR |
| **Rebase and merge** | Los commits de la rama, en línea recta |

---

## 8. Casos especiales

### 8.1 Corrección urgente en producción

```bash
git switch main && git pull
git switch -c hotfix/pago
# arreglo + commit
git push -u origin hotfix/pago   # PR urgente → merge → despliegue
```

### 8.2 Versionado semántico

Las etiquetas de versión siguen `MAYOR.MENOR.PARCHE` (`v2.4.1`):
- **MAYOR**: cambios que rompen compatibilidad.
- **MENOR**: funcionalidades nuevas compatibles.
- **PARCHE**: correcciones.

### 8.3 Automatizar comprobaciones

Con **GitHub Actions** se pueden ejecutar los tests y el linter en cada push o PR. Con **hooks** locales (por ejemplo, Husky en Node) se ejecutan antes de cada commit.

---

## 9. Resumen

- **GitHub Flow**: `main` siempre funciona; una rama por tarea; PR para fusionar.
- Mensajes con **Conventional Commits**: `feat:`, `fix:`, `docs:`…
- Commits pequeños, en imperativo, explicando el porqué.
- PRs pequeñas, actualizadas con `main` y con tests.
- Protege `main` y automatiza comprobaciones.
