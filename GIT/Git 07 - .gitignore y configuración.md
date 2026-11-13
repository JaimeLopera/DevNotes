# 07 - .gitignore y configuración

> [!info] ¿Qué es?
> El archivo **`.gitignore`** le dice a Git qué archivos **no debe seguir** (dependencias, secretos, archivos generados). La **configuración** de Git permite personalizar su comportamiento: tu identidad, el editor, alias para comandos largos y más.

---

## 1. Antes de empezar

Conviene haber leído [[Git 01 - Fundamentos]] y [[Git 02 - Comandos básicos]].

---

## 2. Concepto fundamental

`.gitignore` es un archivo de texto en la raíz del proyecto. Cada línea es un **patrón**: lo que coincide, Git lo ignora.

```
mi-proyecto/
├── .gitignore        ← contiene "node_modules/" y ".env"
├── node_modules/     ← ignorado
├── .env              ← ignorado
├── src/
└── package.json
```

> [!warning] Solo afecta a archivos no seguidos
> Si un archivo **ya tenía commit**, añadirlo a `.gitignore` no basta: hay que dejar de seguirlo con `git rm --cached`.

---

## 3. Sintaxis / estructura

```gitignore
# Comentario

archivo.txt          # un archivo concreto (en cualquier carpeta)
/archivo.txt         # solo en la raíz
carpeta/             # una carpeta entera
*.log                # todos los .log
!importante.log      # excepción: este sí se sigue
**/temp              # "temp" a cualquier profundidad
docs/**/*.pdf        # PDFs dentro de docs, a cualquier nivel
```

---

## 4. Elementos / propiedades / características

### 4.1 Reglas de los patrones

| Patrón | Significa |
|---|---|
| `#` | Comentario |
| `*` | Cualquier texto (sin `/`) |
| `**` | Cualquier número de carpetas |
| `?` | Un carácter |
| `/` al principio | Relativo a la raíz del repositorio |
| `/` al final | Solo carpetas |
| `!` | Excepción (vuelve a incluir) |

### 4.2 Qué ignorar casi siempre

| Tipo | Ejemplos |
|---|---|
| Dependencias | `node_modules/`, `vendor/`, `venv/`, `.venv/` |
| Secretos | `.env`, `.env.local`, `*.pem`, `*.key` |
| Generados | `dist/`, `build/`, `coverage/`, `__pycache__/` |
| Logs | `*.log`, `npm-debug.log*` |
| Sistema | `.DS_Store`, `Thumbs.db` |
| Editor | `.vscode/` (salvo ajustes compartidos), `.idea/` |

### 4.3 Configuración útil

```bash
# Identidad
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"

# Rama por defecto y editor
git config --global init.defaultBranch main
git config --global core.editor "code --wait"

# Pull con rebase por defecto
git config --global pull.rebase true

# Colores
git config --global color.ui auto
```

### 4.4 Alias

Atajos para comandos largos:

```bash
git config --global alias.st status
git config --global alias.co switch
git config --global alias.lg "log --oneline --graph --all --decorate"
```

Ahora `git lg` muestra el árbol de commits.

### 4.5 `.gitignore` global

Para archivos de tu sistema o editor que no quieres en ningún proyecto:

```bash
git config --global core.excludesfile ~/.gitignore_global
echo ".DS_Store" >> ~/.gitignore_global
```

### 4.6 Archivo `.gitkeep`

Git no guarda carpetas vacías. Para mantener una, se crea dentro un archivo vacío llamado `.gitkeep` (es una convención, no un archivo especial de Git).

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: proyecto web con Node

```gitignore
# Dependencias
node_modules/

# Variables de entorno
.env
.env.*
!.env.example

# Build
dist/
build/

# Logs
*.log

# Sistema
.DS_Store
Thumbs.db
```

### 5.2 Ejemplo habitual: dejar de seguir algo que ya se subió

```bash
echo "node_modules/" >> .gitignore
git rm -r --cached node_modules
git commit -m "Deja de seguir node_modules"
```

### 5.3 Ignorar todo excepto una carpeta

```gitignore
/*
!/.gitignore
!/apuntes/
```

### 5.4 Python

```gitignore
__pycache__/
*.pyc
.venv/
venv/
.env
.pytest_cache/
```

---

## 6. Buenas prácticas

- **Crea `.gitignore` antes del primer commit**.
- **Sube un `.env.example`** con los nombres de las variables (sin valores reales) para que otros sepan cuáles hacen falta.
- **No ignores `package-lock.json`**: garantiza que todos instalan las mismas versiones.
- **Usa plantillas**: GitHub ofrece `.gitignore` por lenguaje al crear un repositorio (y el sitio gitignore.io).
- **Ignora lo personal en el global**, no en el `.gitignore` del proyecto.

---

## 7. Diferencias importantes

### 7.1 `.gitignore` vs. `git rm --cached`

`.gitignore` evita que Git **empiece** a seguir un archivo. `git rm --cached` hace que **deje** de seguir uno que ya seguía. A menudo se necesitan los dos.

### 7.2 `.gitignore` vs. `.git/info/exclude`

| | `.gitignore` | `.git/info/exclude` |
|---|---|---|
| Se sube al repositorio | Sí (lo comparte todo el equipo) | No (solo tu copia) |

### 7.3 `carpeta` vs. `carpeta/` vs. `/carpeta`

| Patrón | Ignora |
|---|---|
| `carpeta` | Archivos **y** carpetas con ese nombre, en cualquier nivel |
| `carpeta/` | Solo **carpetas** con ese nombre |
| `/carpeta` | Solo la que está en la **raíz** |

---

## 8. Casos especiales

### 8.1 ¿Por qué se ignora este archivo?

```bash
git check-ignore -v archivo
# .gitignore:3:*.log   archivo.log
```

### 8.2 Excepciones dentro de carpetas ignoradas

Si ignoras una carpeta entera (`logs/`), `!logs/importante.log` **no funciona**: Git no entra en carpetas ignoradas. Hay que ignorar su contenido (`logs/*`) en lugar de la carpeta.

### 8.3 Ver todos los archivos ignorados

```bash
git status --ignored
```

---

## 9. Resumen

- `.gitignore` lista patrones de archivos que Git no seguirá.
- Ignora siempre: `node_modules/`, `.env`, `dist/`, logs y archivos del sistema.
- Si ya estaba seguido: `git rm --cached` + commit.
- `!` crea excepciones; `/` al final = solo carpetas; `/` al principio = solo raíz.
- `git config --global` guarda tu identidad, editor y alias.
- `git check-ignore -v` explica por qué se ignora algo.
