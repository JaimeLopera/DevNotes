# 05 - Repositorios remotos y GitHub

> [!info] ¿Qué es?
> Un **remoto** es una copia de tu repositorio guardada en otro lugar, normalmente **GitHub**. Sirve para tener una copia de seguridad, trabajar desde varios equipos y colaborar: cada persona trabaja en su copia local y sincroniza con el remoto.

---

## 1. Antes de empezar

- Conviene dominar [[Git 02 - Comandos básicos]] y [[Git 03 - Ramas]].
- Necesitas una cuenta en GitHub y autenticarte: con **SSH** (ver [[Terminal 07 - SSH y servidores|SSH y servidores]]) o con HTTPS y un token.

---

## 2. Concepto fundamental

```
   Tu equipo                          GitHub (origin)
┌──────────────┐    git push     ┌──────────────┐
│  repositorio │ ──────────────► │  repositorio │
│    local     │ ◄────────────── │    remoto    │
└──────────────┘    git pull     └──────────────┘
```

- **`push`** sube tus commits al remoto.
- **`pull`** baja los commits nuevos del remoto y los une a tu rama.
- **`origin`** es el nombre por defecto del remoto principal.

---

## 3. Sintaxis / estructura

```bash
git clone URL                    # descargar un repositorio
git remote add origin URL        # conectar un repo local con uno remoto
git push -u origin main          # primera subida de una rama
git push                         # subidas siguientes
git pull                         # traer y fusionar cambios
git fetch                        # traer cambios sin fusionar
```

---

## 4. Elementos / propiedades / características

### 4.1 Gestionar remotos

| Comando | Qué hace |
|---|---|
| `git remote -v` | Lista los remotos y sus URLs |
| `git remote add origin URL` | Añade un remoto |
| `git remote set-url origin URL` | Cambia la URL (p. ej., de HTTPS a SSH) |
| `git remote remove origin` | Elimina un remoto |

### 4.2 Subir: `git push`

| Comando | Qué hace |
|---|---|
| `git push -u origin rama` | Sube la rama y la deja enlazada (*upstream*) |
| `git push` | Sube la rama actual (si ya está enlazada) |
| `git push origin --delete rama` | Borra la rama en el remoto |
| `git push --tags` | Sube las etiquetas |
| `git push --force-with-lease` | Fuerza la subida de forma segura (tras un rebase) |

### 4.3 Bajar: `fetch` y `pull`

| Comando | Qué hace |
|---|---|
| `git fetch` | Descarga los cambios pero **no** toca tus archivos |
| `git pull` | `fetch` + `merge` de la rama remota en la tuya |
| `git pull --rebase` | `fetch` + `rebase` (historial más limpio) |

### 4.4 Pull Requests (PR)

Una **Pull Request** es una petición en GitHub para fusionar una rama en otra. Permite que otros **revisen** el código antes de que llegue a `main`.

Flujo:

1. Creas una rama y haces commits.
2. `git push -u origin feature/login`.
3. En GitHub: **Compare & pull request**.
4. Se revisa, se comenta y se corrige.
5. Se fusiona (*Merge*) y se borra la rama.

### 4.5 Fork

Un **fork** es una copia de un repositorio ajeno en tu cuenta de GitHub. Se usa para contribuir a proyectos en los que no tienes permisos: haces fork, trabajas en tu copia y abres una PR al original.

### 4.6 Etiquetas (*tags*)

Marcan versiones concretas:

```bash
git tag v1.0.0
git push origin v1.0.0
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: subir un proyecto nuevo a GitHub

1. En GitHub: **New repository** (sin README para que esté vacío).
2. En tu terminal:

```bash
cd mi-proyecto
git init
git add .
git commit -m "Versión inicial"
git branch -M main
git remote add origin git@github.com:usuario/mi-proyecto.git
git push -u origin main
```

### 5.2 Ejemplo habitual: el día a día en equipo

```bash
git switch main
git pull                          # me pongo al día
git switch -c feature/buscador
# ... trabajo y commits ...
git push -u origin feature/buscador
# abro una Pull Request en GitHub
```

### 5.3 Clonar y empezar a trabajar

```bash
git clone git@github.com:usuario/proyecto.git
cd proyecto
npm install       # si es un proyecto de Node
```

---

## 6. Buenas prácticas

- **Haz `git pull` antes de empezar** a trabajar cada día.
- **Haz push a menudo**: es tu copia de seguridad.
- **Nunca subas secretos** (`.env`, contraseñas, claves): aunque los borres después, quedan en el historial.
- **Usa `--force-with-lease`** en lugar de `--force` si necesitas forzar.
- **Nunca fuerces el push en `main`** de un proyecto compartido.
- **Escribe un README** que explique qué es el proyecto y cómo arrancarlo (ver [[Markdown 09 - Markdown en GitHub]]).

---

## 7. Diferencias importantes

### 7.1 `fetch` vs. `pull`

| | `git fetch` | `git pull` |
|---|---|---|
| Descarga cambios | Sí | Sí |
| Modifica tu rama | No | Sí (merge o rebase) |
| Riesgo de conflictos | Ninguno | Puede haberlos |
| Uso | Ver qué hay nuevo antes de unirlo | Ponerse al día directamente |

### 7.2 Clone vs. fork

| | `git clone` | Fork |
|---|---|---|
| Dónde | Copia en **tu equipo** | Copia en **tu cuenta de GitHub** |
| Para qué | Trabajar con un repo | Contribuir a un repo ajeno |

### 7.3 `--force` vs. `--force-with-lease`

`--force` sobrescribe el remoto sin mirar. `--force-with-lease` solo lo hace si nadie ha subido nada nuevo desde tu último fetch: evita borrar trabajo ajeno.

### 7.4 HTTPS vs. SSH

Ver [[Terminal 07 - SSH y servidores|SSH y servidores]]. GitHub ya no acepta la contraseña de la cuenta por HTTPS: hace falta un **token** o usar SSH.

---

## 8. Casos especiales

### 8.1 *rejected: non-fast-forward*

El remoto tiene commits que tú no tienes. Solución:

```bash
git pull --rebase
git push
```

### 8.2 Subiste un archivo que no debías

```bash
echo ".env" >> .gitignore
git rm --cached .env
git commit -m "Deja de seguir .env"
git push
```

> [!warning] Secretos expuestos
> Si era una contraseña o clave, **cámbiala**: sigue en el historial y cualquiera con acceso al repositorio puede verla.

### 8.3 Repositorio con varios remotos

Al trabajar con un fork, es habitual tener `origin` (tu fork) y `upstream` (el original):

```bash
git remote add upstream https://github.com/original/proyecto.git
git fetch upstream
git merge upstream/main
```

### 8.4 GitHub Pages

GitHub puede publicar gratis una web estática desde un repositorio: **Settings → Pages → Branch: main**.

---

## 9. Resumen

- Un **remoto** (`origin`) es la copia del repositorio en GitHub.
- `git clone` descarga; `git push` sube; `git pull` baja y fusiona; `git fetch` solo baja.
- La primera vez: `git push -u origin main`.
- Las **Pull Requests** permiten revisar antes de fusionar en `main`.
- Nunca subas secretos; si pasa, cámbialos.
- Si el push es rechazado: `git pull --rebase` y vuelve a subir.
