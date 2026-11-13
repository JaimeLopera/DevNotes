# Entornos y paquetes

> [!info] ¿Qué es?
> Un **entorno virtual** es una carpeta aislada con su **propio Python y sus propias librerías**, separada del resto del sistema. **pip** es la herramienta que instala paquetes de terceros desde **PyPI**, el repositorio público de Python. Juntos permiten que cada proyecto tenga sus dependencias sin que choquen entre sí.

## 1. Antes de empezar

Necesitas saber usar la terminal básica ([[Terminal 00 - Índice|Terminal y Linux]]) y entender qué son módulos y paquetes ([[Python 13 - Módulos y paquetes]]).

## 2. Concepto fundamental

Imagina que un proyecto necesita la librería `X` versión 1 y otro la versión 2. Si instalas todo en el mismo Python del sistema, solo puede haber una. La solución es dar a **cada proyecto su propio entorno**.

```
Sistema
├── Python global (no lo ensucies)
├── proyecto_a/.venv/   →  requests 2.28
└── proyecto_b/.venv/   →  requests 2.32
```

Ventajas:

- Sin conflictos de versiones entre proyectos.
- El proyecto es **reproducible**: otra persona puede instalar exactamente lo mismo.
- No necesitas permisos de administrador.
- Puedes borrar el entorno y recrearlo cuando quieras.

## 3. Sintaxis / estructura

### Crear un entorno virtual

```bash
python -m venv .venv
```

Crea la carpeta `.venv/` con una copia de Python y de `pip`. Se suele llamar `.venv` o `venv`.

### Activarlo

| Sistema | Comando |
|---|---|
| Windows (PowerShell) | `.venv\Scripts\Activate.ps1` |
| Windows (cmd) | `.venv\Scripts\activate.bat` |
| macOS / Linux | `source .venv/bin/activate` |

Cuando está activo, el nombre aparece entre paréntesis al inicio de la terminal: `(.venv) $`.

Para salir: `deactivate`.

### Comandos de pip

```bash
pip install requests                 # instalar
pip install requests==2.32.3         # versión exacta
pip install "requests>=2.30,<3"      # rango de versiones
pip install -U requests              # actualizar
pip uninstall requests               # desinstalar
pip list                             # paquetes instalados
pip show requests                    # información de un paquete
pip freeze                           # lista con versiones exactas
pip install -r requirements.txt      # instalar desde un archivo
pip check                            # detecta dependencias rotas
```

Es más seguro llamar a pip a través de Python, para asegurarte de que usa **el Python correcto**:

```bash
python -m pip install requests
```

## 4. Elementos y características

### `requirements.txt`

Lista de dependencias del proyecto:

```
requests==2.32.3
flask>=3.0
pytest
```

Para generarlo a partir de lo instalado:

```bash
pip freeze > requirements.txt
```

`freeze` incluye también las **dependencias de las dependencias**. Para proyectos serios se separan en `requirements.txt` (producción) y `requirements-dev.txt` (desarrollo y tests).

### `pyproject.toml`

Es el archivo de configuración **moderno y estándar** de un proyecto Python: nombre, versión, dependencias y configuración de herramientas.

```toml
[project]
name = "mi-proyecto"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "requests>=2.30",
]

[project.optional-dependencies]
dev = ["pytest", "ruff"]
```

Se instala con `pip install .` o, para desarrollo, `pip install -e .` (modo editable: los cambios en el código se ven al momento).

### Versionado semántico

Las versiones suelen tener forma `MAYOR.MENOR.PARCHE` (`2.32.3`):

| Parte | Cambia cuando… |
|---|---|
| MAYOR | Hay cambios incompatibles |
| MENOR | Se añaden funciones compatibles |
| PARCHE | Se corrigen errores |

Operadores de versión habituales:

| Escritura | Significado |
|---|---|
| `==2.32.3` | Exactamente esa |
| `>=2.30` | Esa o superior |
| `~=2.32` | Compatible (≥ 2.32 y < 3.0) |
| `<3` | Menor que 3 |

### Herramientas alternativas

| Herramienta | Para qué sirve |
|---|---|
| **`uv`** | Muy rápida; reemplaza a pip, venv y más |
| **`poetry`** | Gestiona dependencias y publicación con `pyproject.toml` |
| **`pipenv`** | Entornos y archivo de bloqueo (`Pipfile.lock`) |
| **`pipx`** | Instala **aplicaciones de línea de comandos** (como `black`) en entornos aislados |
| **`pyenv`** | Instala y cambia entre **versiones de Python** |
| **`conda`** | Entornos y paquetes (muy usado en ciencia de datos) |

Ejemplo con `uv`:

```bash
uv venv
uv pip install requests
uv add requests              # añade al pyproject.toml
uv run python main.py
```

### Archivos de bloqueo (*lock files*)

Guardan las versiones **exactas** de todas las dependencias (directas e indirectas) para reproducir el entorno idéntico. Los generan `poetry` (`poetry.lock`), `uv` (`uv.lock`) y `pipenv` (`Pipfile.lock`).

### Paquetes con partes compiladas (*wheels*)

Muchos paquetes se distribuyen ya compilados (formato **wheel**, `.whl`), por lo que se instalan al instante. Si no hay wheel para tu sistema, pip compila desde el código fuente, y puede necesitar herramientas de compilación.

### Instalar desde otros orígenes

```bash
pip install git+https://github.com/usuario/repo.git
pip install ./carpeta_local
pip install -e ./carpeta_local          # editable
pip install paquete --index-url https://mi-servidor/simple
```

### Variables de entorno y secretos

Las claves (APIs, contraseñas) **no se escriben en el código**. Se leen del entorno:

```python
import os
clave = os.getenv("API_KEY")
```

Se pueden cargar desde un archivo `.env` con la librería `python-dotenv`, y ese archivo **nunca se sube a Git** ([[Git 00 - Índice|Git]]).

## 5. Ejemplos prácticos

### Ejemplo básico: empezar un proyecto desde cero

```bash
mkdir mi_proyecto
cd mi_proyecto
python -m venv .venv
source .venv/bin/activate          # en Windows: .venv\Scripts\activate
python -m pip install requests
python -m pip freeze > requirements.txt
```

### Ejemplo habitual: reproducir un proyecto de otra persona

```bash
git clone https://github.com/usuario/proyecto.git
cd proyecto
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python main.py
```

### Ejemplo completo: estructura de un proyecto organizado

```
mi_proyecto/
├── .venv/                  (NO se sube a Git)
├── .gitignore
├── .env                    (secretos, NO se sube a Git)
├── pyproject.toml
├── requirements.txt
├── README.md
├── src/
│   └── mi_proyecto/
│       ├── __init__.py
│       └── main.py
└── tests/
    └── test_main.py
```

Contenido mínimo de `.gitignore`:

```
.venv/
__pycache__/
*.pyc
.env
.pytest_cache/
dist/
build/
*.egg-info/
```

## 6. Buenas prácticas

- **Un entorno virtual por proyecto**, creado dentro de la carpeta del proyecto.
- **Nunca instales paquetes en el Python global** (salvo herramientas con `pipx`).
- Añade `.venv/` y `.env` al `.gitignore`: **no se suben a Git**.
- **Fija versiones** en `requirements.txt` o usa un archivo de bloqueo para que sea reproducible.
- Usa `python -m pip` para asegurar que instalas en el Python activo.
- Actualiza dependencias con cuidado y ejecuta los tests después.
- Revisa vulnerabilidades con herramientas como `pip-audit`.
- Instala solo paquetes de **fuentes de confianza** y comprueba el nombre exacto (cuidado con el *typosquatting*: nombres parecidos con código malicioso).
- Documenta en el `README.md` cómo crear el entorno y ejecutar el proyecto.
- Separa dependencias de **producción** y de **desarrollo**.
- Indica la versión de Python requerida (`requires-python`).

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| Entorno global vs virtual | El global afecta a todo el sistema; el virtual, solo a su proyecto |
| `pip freeze` vs `requirements.txt` a mano | `freeze` lista **todo** lo instalado con versiones exactas; a mano solo pones lo que necesitas |
| `requirements.txt` vs `pyproject.toml` | El primero lista dependencias; el segundo describe todo el proyecto (estándar moderno) |
| `pip` vs `pipx` | `pip` instala librerías en el entorno; `pipx` instala aplicaciones aisladas |
| `venv` vs `conda` | `venv` solo gestiona paquetes de Python; `conda` también otros lenguajes y bibliotecas del sistema |
| `pip install -e .` vs `pip install .` | El primero (editable) refleja los cambios del código al instante |
| Versión de Python vs versión de paquete | Son cosas distintas: `pyenv` gestiona la primera, `pip` la segunda |
| `pip` vs `uv` | `uv` hace lo mismo mucho más rápido |

## 8. Casos especiales

- **PowerShell bloquea la activación** (error de "ejecución de scripts deshabilitada"): ejecuta una vez

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

- **`pip` instala en otro Python:** comprueba con `which python` / `where python` y `python -m pip --version`.
- **Mover o renombrar la carpeta del proyecto rompe el entorno** (las rutas quedan guardadas dentro). Bórralo y recréalo con `requirements.txt`.
- **No se copia un `.venv` a otro ordenador;** se **recrea** desde la lista de dependencias.
- **Errores de compilación al instalar** (por ejemplo, falta un compilador de C): instala las herramientas del sistema o busca una versión con *wheel*.
- **Conflictos de dependencias:** dos paquetes piden versiones incompatibles de un tercero. Revisa con `pip check` o `pipdeptree`, y ajusta versiones.
- **Python del sistema "gestionado externamente"** (en algunas distribuciones de Linux): pip se niega a instalar en el global con el error `externally-managed-environment`; usa un entorno virtual.
- **Importaciones que funcionan en tu máquina y no en otra:** suele ser una dependencia que falta en `requirements.txt`.
- **VS Code y otros editores:** hay que **seleccionar el intérprete** del `.venv` para que reconozca las librerías.
- **Entornos y Docker:** en contenedores a veces no hace falta venv, porque el contenedor ya es un entorno aislado ([[Docker 00 - Índice|Docker]]).
- **Varias versiones de Python:** usa `pyenv` o `uv python install 3.12` para instalarlas y crear entornos con una versión concreta.

❌ Subir `.venv/` a Git
🤔 Ocupa muchísimo, depende del sistema operativo y no es reproducible.
✅ Añádelo a `.gitignore` y comparte solo `requirements.txt` o `pyproject.toml`.

❌ `pip install` con el entorno desactivado
🤔 Instala en el Python global y mezcla dependencias.
✅ Activa el entorno antes y comprueba que aparece `(.venv)` en la terminal.

## 9. Resumen

- Un **entorno virtual** aísla las dependencias de cada proyecto: `python -m venv .venv` → activar → `pip install`.
- `pip` instala desde PyPI: `install`, `uninstall`, `list`, `show`, `freeze`, `-r requirements.txt`.
- Usa `python -m pip` para asegurar el Python correcto.
- `requirements.txt` o, mejor, `pyproject.toml` (más archivo de bloqueo) para reproducir el proyecto.
- Alternativas modernas: `uv`, `poetry`, `pipx`, `pyenv`, `conda`.
- **Nunca subas `.venv` ni `.env` a Git;** nunca guardes secretos en el código.
- Recrea el entorno en vez de moverlo; documenta cómo instalarlo.