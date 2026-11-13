# 02 - Comandos básicos

> [!info] ¿Qué es?
> Los comandos del **ciclo diario** de Git: ver el estado, preparar cambios, hacer commits y consultar el historial. Con estos cinco o seis comandos se hace el 90 % del trabajo.

---

## 1. Antes de empezar

Conviene haber leído [[Git 01 - Fundamentos]] (las tres zonas: trabajo, staging y repositorio).

---

## 2. Concepto fundamental

El ciclo que repetirás cientos de veces:

```
1. Modificas archivos
2. git status              → ¿qué ha cambiado?
3. git diff                → ¿qué cambios exactos?
4. git add archivo         → preparo lo que quiero guardar
5. git commit -m "..."     → lo guardo en el historial
```

---

## 3. Sintaxis / estructura

```bash
git status
git add <archivo>
git commit -m "Mensaje"
git log
git diff
```

---

## 4. Elementos / propiedades / características

### 4.1 `git status`: qué está pasando

```bash
git status       # completo
git status -s    # resumido
```

En la versión resumida:

| Código | Significa |
|---|---|
| `??` | Archivo nuevo sin seguir |
| ` M` | Modificado, sin preparar |
| `M ` | Modificado y preparado |
| `A ` | Nuevo y preparado |
| ` D` | Borrado |

### 4.2 `git add`: preparar cambios

| Comando | Qué prepara |
|---|---|
| `git add archivo.js` | Un archivo |
| `git add src/` | Una carpeta entera |
| `git add .` | Todo lo de la carpeta actual hacia abajo |
| `git add -A` | Todos los cambios del repositorio (incluye borrados) |
| `git add -p` | Pregunta trozo a trozo qué preparar |

### 4.3 `git commit`: guardar en el historial

```bash
git commit -m "Añade formulario de contacto"
git commit                       # abre el editor para un mensaje largo
git commit -am "Corrige estilos" # add + commit de archivos YA seguidos
```

### 4.4 `git log`: ver el historial

| Comando | Qué muestra |
|---|---|
| `git log` | Historial completo (sal con `q`) |
| `git log --oneline` | Un commit por línea |
| `git log --oneline --graph --all` | Árbol de todas las ramas |
| `git log -5` | Los últimos 5 |
| `git log -p archivo` | Cambios de un archivo a lo largo del tiempo |
| `git log --author="Ana"` | Commits de una persona |
| `git show a1b2c3d` | Detalle de un commit |

### 4.5 `git diff`: ver cambios exactos

| Comando | Compara |
|---|---|
| `git diff` | Trabajo vs. staging (lo que aún no has preparado) |
| `git diff --staged` | Staging vs. último commit (lo que va a entrar en el commit) |
| `git diff main rama` | Dos ramas |
| `git diff a1b2c3d` | Un commit vs. ahora |

Las líneas con `-` se han quitado y las de `+` se han añadido.

### 4.6 Borrar y mover archivos

```bash
git rm archivo.js           # lo borra y prepara el borrado
git rm --cached archivo.js  # deja de seguirlo pero lo mantiene en disco
git mv viejo.js nuevo.js    # renombrar
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```bash
echo "body { margin: 0; }" > style.css
git status                  # style.css: untracked
git add style.css
git commit -m "Añade estilos base"
git log --oneline
# 3f2a1b9 Añade estilos base
```

### 5.2 Ejemplo habitual: separar cambios en dos commits

Has arreglado un bug y también has cambiado colores. Mejor en commits separados:

```bash
git status
#  M js/form.js
#  M css/style.css

git add js/form.js
git commit -m "Corrige validación del email"

git add css/style.css
git commit -m "Actualiza colores del header"
```

---

## 6. Buenas prácticas

- **Mira `git status` y `git diff --staged` antes de cada commit**: evita subir cosas por error.
- **Un commit = un cambio lógico**. Si el mensaje necesita "y", probablemente son dos commits.
- **Mensajes en imperativo y claros**: "Añade login", no "cambios" ni "arreglos".
- **No hagas commit de código que no funciona** en la rama principal.
- **Cuidado con `git add .`**: revisa antes que no estás añadiendo archivos que no deben subirse.

---

## 7. Diferencias importantes

### 7.1 `git add .` vs. `git add -A`

En versiones modernas de Git son casi iguales. La diferencia: `git add .` actúa desde la **carpeta actual** hacia abajo; `git add -A` actúa sobre **todo el repositorio**.

### 7.2 `git commit -am` vs. `git add` + `git commit`

`-a` solo incluye archivos que Git **ya seguía**. Los archivos nuevos (*untracked*) no entran: necesitan `git add`.

### 7.3 `git diff` vs. `git diff --staged`

| | `git diff` | `git diff --staged` |
|---|---|---|
| Muestra | Lo que has cambiado y **no** has preparado | Lo que **sí** va a entrar en el commit |

### 7.4 `git rm` vs. `git rm --cached`

`git rm` borra el archivo del disco y del repositorio. `git rm --cached` solo deja de seguirlo (útil cuando subiste algo que debía estar en `.gitignore`).

---

## 8. Casos especiales

### 8.1 Corregir el último commit

```bash
git commit --amend -m "Nuevo mensaje"   # cambia el mensaje
git add olvidado.js
git commit --amend --no-edit            # añade un archivo olvidado
```

> [!warning] Solo antes de hacer push
> `--amend` reescribe el commit. Si ya lo subiste, evita usarlo (ver [[Git 06 - Deshacer cambios]]).

### 8.2 Salir de `git log` o de un editor

- En `git log` o `less`: pulsa `q`.
- Si se abre **Vim** al hacer commit: escribe el mensaje, pulsa `Esc`, escribe `:wq` y Enter. Para cambiar de editor: `git config --global core.editor "code --wait"`.

### 8.3 Archivo que no aparece en `git status`

Probablemente está en `.gitignore`. Compruébalo con `git check-ignore -v archivo`.

---

## 9. Resumen

- `git status` → qué ha cambiado (úsalo siempre).
- `git add` → prepara cambios; `git add -p` para elegir trozos.
- `git commit -m "..."` → guarda en el historial.
- `git log --oneline --graph` → historial resumido.
- `git diff` / `git diff --staged` → cambios exactos.
- `git commit --amend` corrige el último commit (solo si no está subido).
- Siguiente paso: [[Git 03 - Ramas]] o [[Git 05 - Repositorios remotos y GitHub]].
