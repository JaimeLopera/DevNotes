# 06 - Deshacer cambios

> [!info] ¿Qué es?
> Git permite deshacer casi cualquier cosa: cambios sin guardar, archivos preparados por error, commits equivocados o incluso commits "borrados". La clave es elegir el comando adecuado según **dónde** está el cambio que quieres deshacer.

---

## 1. Antes de empezar

Conviene tener claras las tres zonas de [[Git 01 - Fundamentos]] y los comandos de [[Git 02 - Comandos básicos]].

---

## 2. Concepto fundamental

Elige el comando según dónde esté lo que quieres deshacer:

| Quiero deshacer… | Comando |
|---|---|
| Cambios en un archivo **sin preparar** | `git restore archivo` |
| Un archivo que **preparé** con `add` | `git restore --staged archivo` |
| El **último commit** (no subido) | `git reset --soft HEAD~1` |
| Un commit **ya subido** | `git revert hash` |
| Guardar cambios **temporalmente** | `git stash` |
| Recuperar algo que creía **perdido** | `git reflog` |

---

## 3. Sintaxis / estructura

```bash
git restore archivo             # descarta cambios locales
git restore --staged archivo    # quita del staging
git reset [--soft|--mixed|--hard] commit
git revert hash
git stash / git stash pop
git reflog
```

`HEAD~1` significa "un commit antes de HEAD"; `HEAD~3`, tres commits antes.

---

## 4. Elementos / propiedades / características

### 4.1 `git restore`: archivos

```bash
git restore index.html               # vuelve al último commit (pierdes los cambios)
git restore .                        # todos los archivos
git restore --staged index.html      # lo saca del staging (conserva los cambios)
git restore --source=a1b2c3d index.html   # recupera la versión de un commit
```

### 4.2 `git reset`: mover la rama hacia atrás

| Modo | Commits deshechos | Staging | Tus archivos |
|---|---|---|---|
| `--soft` | Sí | Los cambios quedan **preparados** | Intactos |
| `--mixed` (por defecto) | Sí | Se vacía | Intactos (cambios sin preparar) |
| `--hard` | Sí | Se vacía | **Se pierden los cambios** |

```bash
git reset --soft HEAD~1    # deshace el commit, mantiene todo preparado
git reset HEAD~1           # deshace el commit, cambios sin preparar
git reset --hard HEAD~1    # borra el commit Y los cambios
```

### 4.3 `git revert`: deshacer creando un commit nuevo

```bash
git revert a1b2c3d
```

Crea un commit que hace **lo contrario** del indicado. No reescribe el historial, así que es seguro en ramas compartidas.

### 4.4 `git stash`: guardar cambios en un cajón

| Comando | Qué hace |
|---|---|
| `git stash` | Guarda los cambios y deja el directorio limpio |
| `git stash -u` | Incluye archivos nuevos (*untracked*) |
| `git stash push -m "mensaje"` | Con descripción |
| `git stash list` | Lista lo guardado |
| `git stash pop` | Recupera lo último y lo borra del cajón |
| `git stash apply` | Recupera sin borrarlo del cajón |
| `git stash drop` | Borra un stash |

### 4.5 `git reflog`: la red de seguridad

Registra **todos** los movimientos de HEAD, incluso los commits que ya no aparecen en `git log`:

```bash
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~1
# 9f8e7d6 HEAD@{1}: commit: Añade login     ← el commit "perdido"

git reset --hard 9f8e7d6    # vuelves a él
```

### 4.6 `git clean`: borrar archivos no seguidos

```bash
git clean -n     # muestra qué borraría (simulación)
git clean -fd    # borra archivos y carpetas no seguidos
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: descartar un experimento

```bash
# Has tocado style.css y lo has roto todo
git restore style.css
```

### 5.2 Ejemplo habitual: commit con un error, sin subir

```bash
git log --oneline
# 3c4d5e6 Añade carrito (con un error)
# 1a2b3c4 Añade productos

git reset --soft HEAD~1   # deshago el commit, los cambios siguen preparados
# corrijo el error
git add .
git commit -m "Añade carrito"
```

### 5.3 Commit con error, ya subido

```bash
git revert 3c4d5e6
git push
```

---

## 6. Buenas prácticas

- **Antes de cualquier `--hard`**, haz `git status` y `git stash` si dudas: lo que borra no se recupera.
- **En ramas compartidas, usa `revert`**, nunca `reset` + `push --force`.
- **Usa `git clean -n`** antes de `git clean -fd`.
- **Recuerda `git reflog`**: si algo tuvo commit, casi seguro se puede recuperar.
- **Pon mensajes a los stash**: con varios, sin mensaje es imposible saber cuál es cuál.

---

## 7. Diferencias importantes

### 7.1 `reset` vs. `revert`

| | `git reset` | `git revert` |
|---|---|---|
| Qué hace | Mueve la rama atrás (los commits "desaparecen") | Crea un commit nuevo que deshace otro |
| Reescribe el historial | Sí | No |
| Seguro si ya hiciste push | No | Sí |

### 7.2 `restore` vs. `reset`

`restore` trabaja con **archivos**. `reset` trabaja con **commits** (mueve la rama).

### 7.3 `stash pop` vs. `stash apply`

`pop` aplica y **elimina** el stash. `apply` aplica y lo **conserva** (útil para aplicarlo en varias ramas).

### 7.4 `--soft` vs. `--mixed` vs. `--hard`

Ver tabla 4.2. Resumen: `soft` conserva todo preparado, `mixed` conserva los cambios sin preparar, `hard` lo borra todo.

---

## 8. Casos especiales

### 8.1 Hice commit en la rama equivocada

```bash
# Estás en main y el commit debía ir en una rama nueva
git switch -c feature/nueva    # la rama nueva se lleva el commit
git switch main
git reset --hard HEAD~1        # lo quitas de main
```

### 8.2 Recuperar un archivo borrado

```bash
git restore archivo.js                      # si el borrado no tiene commit
git restore --source=HEAD~1 archivo.js      # si ya hiciste commit del borrado
```

### 8.3 Recuperar una rama borrada

```bash
git reflog          # busca el último commit de esa rama
git switch -c rama-recuperada a1b2c3d
```

### 8.4 Comandos antiguos

En tutoriales viejos verás `git checkout -- archivo` (= `git restore archivo`) y `git reset HEAD archivo` (= `git restore --staged archivo`).

---

## 9. Resumen

- Cambios sin preparar → `git restore archivo`.
- Quitar del staging → `git restore --staged archivo`.
- Último commit sin subir → `git reset --soft HEAD~1`.
- Commit ya subido → `git revert hash`.
- Guardar temporalmente → `git stash` / `git stash pop`.
- Algo "perdido" → `git reflog`.
- `--hard` y `git clean -fd` borran de verdad: úsalos con cuidado.
