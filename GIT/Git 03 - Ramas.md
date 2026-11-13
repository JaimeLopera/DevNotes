# 03 - Ramas

> [!info] ¿Qué es?
> Una **rama** (*branch*) es una línea de desarrollo independiente. Permite trabajar en una funcionalidad nueva o arreglar un error **sin tocar** la versión que funciona, y unirlo después cuando esté listo.

---

## 1. Antes de empezar

Conviene dominar [[Git 02 - Comandos básicos]] (`add`, `commit`, `log`).

---

## 2. Concepto fundamental

Una rama es simplemente un **puntero a un commit** que avanza cuando haces commits nuevos.

```
                 C4 ── C5   ← rama "formulario"
                /
C1 ── C2 ── C3              ← rama "main"
```

- `main` sigue en C3, intacta.
- En `formulario` se añaden C4 y C5 sin afectar a `main`.
- Cuando el formulario esté terminado, se **fusiona** en `main` (ver [[Git 04 - Fusionar y resolver conflictos]]).

Crear ramas en Git es instantáneo y no copia archivos, así que se usan constantemente.

---

## 3. Sintaxis / estructura

```bash
git branch                  # lista las ramas (* = la actual)
git switch -c nueva-rama    # crea una rama y se cambia a ella
git switch main             # cambia a otra rama
git branch -d nueva-rama    # borra una rama ya fusionada
```

---

## 4. Elementos / propiedades / características

### 4.1 Ver ramas

| Comando | Qué muestra |
|---|---|
| `git branch` | Ramas locales |
| `git branch -a` | Locales y remotas |
| `git branch -v` | Con el último commit de cada una |
| `git branch --merged` | Ramas ya fusionadas en la actual |

### 4.2 Crear y cambiar

| Comando | Qué hace |
|---|---|
| `git branch nombre` | Crea la rama (sin cambiarte) |
| `git switch nombre` | Cambia a esa rama |
| `git switch -c nombre` | Crea y cambia en un paso |
| `git switch -` | Vuelve a la rama anterior |
| `git switch -c nombre a1b2c3d` | Crea una rama desde un commit concreto |

### 4.3 Renombrar y borrar

```bash
git branch -m nuevo-nombre        # renombra la rama actual
git branch -d rama                # borra (solo si está fusionada)
git branch -D rama                # fuerza el borrado
git push origin --delete rama     # borra la rama en GitHub
```

### 4.4 Nombres habituales

| Prefijo | Para qué | Ejemplo |
|---|---|---|
| `feature/` | Funcionalidad nueva | `feature/login` |
| `fix/` | Corregir un error | `fix/email-validacion` |
| `hotfix/` | Corrección urgente en producción | `hotfix/pago-caido` |
| `docs/` | Documentación | `docs/readme` |
| `refactor/` | Reorganizar código sin cambiar el comportamiento | `refactor/api-client` |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```bash
git switch -c feature/footer
# ... editas footer.html ...
git add footer.html
git commit -m "Añade footer"
git switch main          # footer.html desaparece: no existe en main
git switch feature/footer # vuelve a aparecer
```

### 5.2 Ejemplo habitual: arreglar un error mientras trabajas en otra cosa

```bash
# Estás en feature/carrito con cambios a medias
git stash                     # guarda los cambios temporalmente
git switch main
git switch -c fix/menu-movil
# ... arreglas el error ...
git commit -am "Corrige menú en móvil"
git switch main
git merge fix/menu-movil
git switch feature/carrito
git stash pop                 # recuperas tus cambios a medias
```

(`stash` se explica en [[Git 06 - Deshacer cambios]].)

---

## 6. Buenas prácticas

- **Una rama por tarea**: una funcionalidad o un arreglo.
- **No trabajes directamente en `main`**: mantenla siempre funcionando.
- **Ramas cortas**: cuanto más tiempo vive una rama, más conflictos tendrá.
- **Nombres descriptivos** en minúsculas y con guiones: `feature/formulario-contacto`.
- **Borra las ramas** una vez fusionadas.
- **Haz commit (o stash) antes de cambiar de rama** para no arrastrar cambios.

---

## 7. Diferencias importantes

### 7.1 `git switch` vs. `git checkout`

| | `git switch` | `git checkout` |
|---|---|---|
| Para qué | Solo cambiar de rama | Cambiar de rama **y** restaurar archivos |
| Crear y cambiar | `git switch -c rama` | `git checkout -b rama` |
| Recomendado | Sí (más claro, desde Git 2.23) | Sigue funcionando; lo verás en tutoriales antiguos |

### 7.2 `-d` vs. `-D`

`-d` solo borra si la rama ya está fusionada (seguro). `-D` borra aunque tenga commits sin fusionar (los pierdes).

### 7.3 Rama local vs. remota

Una rama local existe solo en tu equipo. `origin/main` es la copia de la rama `main` del remoto tal como estaba en tu último `fetch`. Ver [[Git 05 - Repositorios remotos y GitHub]].

---

## 8. Casos especiales

### 8.1 Cambiar de rama con cambios sin guardar

Git no te deja si esos cambios chocan con la otra rama:

```
error: Your local changes would be overwritten by checkout
```

Solución: haz commit o `git stash` antes de cambiar.

### 8.2 *Detached HEAD*

Si haces `git switch --detach a1b2c3d` (o `git checkout a1b2c3d`), no estás en ninguna rama. Puedes mirar, pero si haces commits, crea una rama para no perderlos:

```bash
git switch -c rama-rescate
```

### 8.3 Traer una rama remota

```bash
git fetch
git switch nombre-rama   # crea la local siguiendo a origin/nombre-rama
```

---

## 9. Resumen

- Una rama es un puntero a un commit; crear ramas es instantáneo.
- `git switch -c rama` crea y cambia; `git switch rama` cambia.
- `git branch` lista; `git branch -d` borra.
- Una rama por tarea; `main` siempre debe funcionar.
- Haz commit o `stash` antes de cambiar de rama.
- Siguiente paso: [[Git 04 - Fusionar y resolver conflictos]].
