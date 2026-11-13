# 04 - Fusionar y resolver conflictos

> [!info] ¿Qué es?
> **Fusionar** es unir el trabajo de una rama en otra. Normalmente Git lo hace solo, pero si dos ramas cambiaron **las mismas líneas** de un archivo, aparece un **conflicto** y tienes que decidir tú qué versión queda.

---

## 1. Antes de empezar

Conviene haber leído [[Git 03 - Ramas]].

---

## 2. Concepto fundamental

Hay dos formas de unir ramas:

**Merge** (fusión): crea un commit nuevo que une las dos historias.

```
          C4 ── C5           feature
         /         \
C1 ── C2 ── C3 ──── M        main   (M = commit de merge)
```

**Rebase**: mueve tus commits para que empiecen al final de la otra rama, como si los hubieras hecho después. El historial queda en línea recta.

```
C1 ── C2 ── C3 ── C4' ── C5'   feature (rebasada sobre main)
```

---

## 3. Sintaxis / estructura

```bash
# Merge: me pongo en la rama que RECIBE los cambios
git switch main
git merge feature/login

# Rebase: me pongo en la rama que quiero MOVER
git switch feature/login
git rebase main
```

---

## 4. Elementos / propiedades / características

### 4.1 Tipos de merge

| Tipo | Cuándo ocurre | Resultado |
|---|---|---|
| **Fast-forward** | `main` no ha cambiado desde que creaste la rama | Git solo mueve el puntero; no hay commit de merge |
| **Three-way** (de tres vías) | Las dos ramas tienen commits nuevos | Se crea un commit de merge |

```bash
git merge --no-ff feature   # fuerza commit de merge aunque pudiera ser fast-forward
git merge --squash feature  # junta todos los commits de la rama en uno
```

### 4.2 Cómo se ve un conflicto

```
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

Dentro del archivo:

```html
<<<<<<< HEAD
<h1>Bienvenido a mi web</h1>
=======
<h1>Hola, bienvenida</h1>
>>>>>>> feature/titulo
```

| Marca | Significa |
|---|---|
| `<<<<<<< HEAD` | Empieza tu versión (la rama en la que estás) |
| `=======` | Separador |
| `>>>>>>> feature/titulo` | Termina la versión de la otra rama |

### 4.3 Pasos para resolver un conflicto

1. `git status` → muestra los archivos en conflicto (*both modified*).
2. Abre cada archivo y **decide**: tu versión, la otra, o una mezcla.
3. **Borra las marcas** `<<<<<<<`, `=======` y `>>>>>>>`.
4. `git add archivo` → marca el conflicto como resuelto.
5. `git commit` (en merge) o `git rebase --continue` (en rebase).

> [!tip] VS Code
> VS Code muestra botones encima de cada conflicto: *Accept Current*, *Accept Incoming*, *Accept Both*. Muy cómodo para empezar.

### 4.4 Abortar

```bash
git merge --abort     # vuelve a como estaba antes del merge
git rebase --abort    # vuelve a como estaba antes del rebase
```

### 4.5 Rebase interactivo

Permite reordenar, unir o cambiar mensajes de tus últimos commits:

```bash
git rebase -i HEAD~3
```

| Palabra | Qué hace con ese commit |
|---|---|
| `pick` | Lo deja tal cual |
| `reword` | Cambia el mensaje |
| `squash` | Lo une con el anterior |
| `drop` | Lo elimina |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: merge sin conflictos

```bash
git switch main
git merge feature/footer
# Fast-forward
git branch -d feature/footer
```

### 5.2 Ejemplo habitual: resolver un conflicto

```bash
git switch main
git merge feature/titulo
# CONFLICT in index.html

# Editas index.html y dejas:
# <h1>Bienvenido a mi web</h1>

git add index.html
git commit -m "Fusiona feature/titulo"
```

### 5.3 Actualizar tu rama con los cambios de `main`

```bash
git switch feature/login
git fetch
git rebase origin/main     # o: git merge origin/main
```

---

## 6. Buenas prácticas

- **Fusiona a menudo**: ramas cortas = conflictos pequeños.
- **Actualiza tu rama con `main`** antes de abrir una Pull Request.
- **No hagas rebase de commits ya compartidos** (que otros tienen): reescribe la historia y les causarás problemas.
- **Prueba el proyecto después de resolver un conflicto**: que no haya marcas `<<<<<<<` y que funcione.
- **Si te lías, aborta** (`--abort`) y empieza de nuevo con calma.

---

## 7. Diferencias importantes

### 7.1 Merge vs. rebase

| | Merge | Rebase |
|---|---|---|
| Historial | Conserva cómo ocurrió (con bifurcaciones) | Lineal y limpio |
| Crea commit extra | Sí (en three-way) | No |
| Reescribe commits | No | Sí (cambian sus hashes) |
| Seguro con ramas compartidas | Sí | No |
| Uso típico | Unir una rama terminada a `main` | Poner al día tu rama local |

> [!tip] Regla de oro
> **Rebase** para limpiar tu trabajo local antes de compartirlo. **Merge** para unir trabajo ya compartido.

### 7.2 *Current* vs. *incoming*

- **Current** (`HEAD`): la rama en la que estás.
- **Incoming**: la rama que estás trayendo.

¡En un rebase se invierten! *Current* es la rama base y *incoming* son tus commits.

### 7.3 `merge --squash` vs. merge normal

`--squash` mete todos los cambios de la rama como **un solo commit** en `main` (historial más limpio, pero pierdes el detalle de los commits).

---

## 8. Casos especiales

### 8.1 Conflicto en un archivo borrado

Si una rama borró el archivo y la otra lo modificó: decide si lo quieres (`git add archivo`) o no (`git rm archivo`).

### 8.2 Quedarse con una versión entera

```bash
git checkout --ours archivo     # tu versión
git checkout --theirs archivo   # la versión de la otra rama
git add archivo
```

### 8.3 Conflictos en `package-lock.json`

No los resuelvas a mano: acepta una versión, ejecuta `npm install` para regenerarlo y haz `git add package-lock.json`.

---

## 9. Resumen

- **Merge** une ramas con un commit de merge; **rebase** reescribe tus commits encima de la otra rama.
- Un **conflicto** aparece cuando las dos ramas cambiaron las mismas líneas.
- Resolver: editar → borrar marcas → `git add` → `git commit` / `git rebase --continue`.
- `--abort` deshace un merge o rebase a medias.
- Nunca hagas rebase de commits que otros ya tienen.
