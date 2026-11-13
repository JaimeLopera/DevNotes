# 01 - Fundamentos

> [!info] ¿Qué es?
> **Git** es un programa que guarda **fotos** (*commits*) del estado de tu proyecto a lo largo del tiempo. Con esas fotos puedes ver qué cambió, quién lo cambió, volver a una versión anterior o trabajar en varias versiones a la vez.

---

## 1. Antes de empezar

- Instala Git: git-scm.com (en Windows, acepta las opciones por defecto; incluye Git Bash).
- Comprueba que funciona: `git --version`.
- Conviene saber moverse por la terminal: [[Terminal 02 - Navegación y archivos|Navegación y archivos]].

---

## 2. Concepto fundamental

Git trabaja con **tres zonas**:

```
 Directorio de trabajo  ──git add──►  Staging area  ──git commit──►  Repositorio
 (tus archivos)                       (preparados)                   (historial)
```

| Zona | Qué es |
|---|---|
| **Directorio de trabajo** | Los archivos tal como los ves y editas |
| **Staging area** (área de preparación) | Los cambios que has elegido para el próximo commit |
| **Repositorio** (`.git/`) | El historial de todos los commits |

El staging existe para que **elijas qué entra en cada commit**: puedes haber cambiado 5 archivos y guardar solo 2.

---

## 3. Sintaxis / estructura

```bash
git comando [opciones] [argumentos]
```

Configuración inicial (una sola vez por ordenador):

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
git config --global init.defaultBranch main
```

Empezar a usar Git en un proyecto:

```bash
cd mi-proyecto
git init        # crea la carpeta oculta .git/
```

---

## 4. Elementos / propiedades / características

### 4.1 Conceptos clave

| Concepto | Qué es |
|---|---|
| **Repositorio** (*repo*) | Proyecto controlado por Git (tiene una carpeta `.git/`) |
| **Commit** | Foto del proyecto en un momento, con mensaje, autor y fecha |
| **Hash** | Identificador único de cada commit (`a1b2c3d…`) |
| **Rama** (*branch*) | Línea de desarrollo independiente; `main` es la principal |
| **HEAD** | Puntero a "dónde estás ahora" (normalmente, el último commit de tu rama) |
| **Remoto** | Copia del repositorio en otro sitio (GitHub); suele llamarse `origin` |

### 4.2 Estados de un archivo

| Estado | Significa |
|---|---|
| *Untracked* | Git aún no lo sigue (archivo nuevo) |
| *Modified* | Cambiado desde el último commit, sin preparar |
| *Staged* | Preparado para el próximo commit |
| *Committed* | Guardado en el historial |

### 4.3 La carpeta `.git`

Contiene **todo el historial**. Si la borras, el proyecto deja de ser un repositorio (tus archivos se quedan, pero pierdes el historial). No la toques a mano.

### 4.4 Niveles de configuración

| Nivel | Opción | Afecta a |
|---|---|---|
| Sistema | `--system` | Todos los usuarios del equipo |
| Global | `--global` | Todos tus repositorios |
| Local | (ninguna) | Solo el repositorio actual |

```bash
git config --list          # ver la configuración
git config user.email      # ver un valor concreto
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: primer commit

```bash
mkdir mi-web && cd mi-web
git init
echo "# Mi web" > README.md
git status              # README.md aparece como untracked
git add README.md
git commit -m "Añade README"
git log                 # aparece tu primer commit
```

### 5.2 Ejemplo habitual: seguir un proyecto que ya existe

```bash
cd proyecto-existente
git init
git add .
git commit -m "Versión inicial"
```

> [!tip] Antes del primer `git add .`
> Crea el `.gitignore` para no subir `node_modules`, `.env` y similares: ver [[Git 07 - .gitignore y configuración]].

---

## 6. Buenas prácticas

- **Configura nombre y correo** antes del primer commit (aparecen en cada commit).
- **Haz commits pequeños y frecuentes**, cada uno con un cambio con sentido.
- **Usa `main`** como nombre de la rama principal.
- **Crea el `.gitignore` al principio** del proyecto.
- **Consulta `git status` a menudo**: te dice siempre qué está pasando y qué hacer.

---

## 7. Diferencias importantes

### 7.1 Git vs. GitHub

| | Git | GitHub |
|---|---|---|
| Qué es | Un **programa** | Una **web** (servicio) |
| Dónde funciona | En tu ordenador | En la nube |
| Necesita internet | No | Sí |
| Alternativas | — | GitLab, Bitbucket |

### 7.2 Guardar el archivo vs. hacer commit

Guardar (`Ctrl + S`) cambia el archivo en disco. **Hacer commit** registra ese estado en el historial. Puedes guardar cien veces y hacer un solo commit.

### 7.3 `master` vs. `main`

Son solo nombres de la rama principal. `master` era el nombre antiguo por defecto; hoy GitHub y la mayoría de proyectos usan `main`.

---

## 8. Casos especiales

### 8.1 Repositorio dentro de otro

No hagas `git init` dentro de una carpeta que ya está en un repositorio: crearás un repositorio anidado y Git ignorará esa subcarpeta. Para incluir otro repo dentro, se usan *submódulos* (tema avanzado).

### 8.2 Saltos de línea Windows/Linux

Windows usa `CRLF` y Linux `LF`. Para evitar diferencias falsas:

```bash
git config --global core.autocrlf true    # en Windows
git config --global core.autocrlf input   # en macOS/Linux
```

### 8.3 Carpetas vacías

Git no guarda carpetas vacías. Si necesitas una, mete dentro un archivo `.gitkeep` vacío.

---

## 9. Resumen

- Git guarda el **historial** de un proyecto mediante **commits**.
- Tres zonas: **trabajo → staging → repositorio** (`add` y `commit`).
- `git init` crea un repositorio; todo se guarda en `.git/`.
- Configura `user.name` y `user.email` una vez con `--global`.
- **Git** es el programa; **GitHub** es la web donde se guardan repositorios.
- Siguiente paso: [[Git 02 - Comandos básicos]].
