# Módulos y paquetes

> [!info] ¿Qué es?
> Un **módulo** es un archivo `.py` con código reutilizable (funciones, clases, variables). Un **paquete** es una carpeta que agrupa varios módulos. Sirven para **organizar el código** en piezas pequeñas y **reutilizarlo** en otros archivos y proyectos, además de usar la enorme **biblioteca estándar** de Python y librerías de terceros.

## 1. Antes de empezar

Debes dominar funciones ([[Python 06 - Funciones]]). Para instalar librerías externas necesitarás los entornos virtuales ([[Python 17 - Entornos y paquetes]]).

## 2. Concepto fundamental

- **Módulo:** un archivo `.py`. Su nombre (sin `.py`) es el nombre del módulo.
- **Paquete:** una carpeta con módulos (tradicionalmente con un archivo `__init__.py`).
- **Librería:** conjunto de paquetes y módulos que se instala (por ejemplo `requests`).
- **Biblioteca estándar:** módulos que ya vienen con Python (`math`, `random`, `json`, `datetime`, `pathlib`…).

Al importar un módulo, Python **lo ejecuta una vez** y guarda el resultado en memoria (`sys.modules`); si lo importas otra vez, reutiliza el mismo.

## 3. Sintaxis / estructura

### Formas de importar

```python
import math                       # usas math.sqrt(9)
import math as m                  # alias: m.sqrt(9)
from math import sqrt, pi         # traes nombres concretos: sqrt(9)
from math import sqrt as raiz     # con alias
from math import *                # trae todo (evitar)
```

### Importar de un paquete

```python
import paquete.modulo
from paquete import modulo
from paquete.modulo import funcion
```

## 4. Elementos y características

### Crear tu propio módulo

`utilidades.py`:

```python
PI = 3.1416

def area_circulo(radio):
    return PI * radio ** 2

def main():
    print(area_circulo(2))

if __name__ == "__main__":
    main()
```

`principal.py` (en la misma carpeta):

```python
import utilidades
from utilidades import area_circulo

print(utilidades.PI)
print(area_circulo(3))
```

El bloque `if __name__ == "__main__":` hace que `main()` **solo se ejecute si lanzas `utilidades.py` directamente**, no cuando se importa ([[Python 01 - Fundamentos]]).

### Crear un paquete

```
mi_proyecto/
├── main.py
└── herramientas/
    ├── __init__.py
    ├── texto.py
    └── numeros.py
```

```python
# main.py
from herramientas.texto import limpiar
from herramientas import numeros
```

- `__init__.py` se ejecuta al importar el paquete; puede estar **vacío** o usarse para exponer nombres cómodos:

```python
# herramientas/__init__.py
from .texto import limpiar
from .numeros import sumar
```

Así puedes escribir `from herramientas import limpiar`.

- Desde Python 3.3 existen los **paquetes de espacio de nombres** (sin `__init__.py`), pero lo habitual y recomendado sigue siendo incluirlo.

### Importaciones absolutas y relativas

```python
# Absoluta (recomendada): ruta completa desde la raíz del proyecto
from herramientas.texto import limpiar

# Relativa: solo dentro de un paquete
from .texto import limpiar        # mismo paquete
from . import numeros
from ..otro_paquete import algo   # un nivel arriba
```

Las importaciones relativas **solo funcionan dentro de un paquete**, no en un script lanzado directamente.

### Dónde busca Python los módulos

En este orden:

1. Los módulos ya importados (`sys.modules`).
2. Los módulos integrados.
3. Las carpetas de `sys.path`: la carpeta del script, `PYTHONPATH` y las carpetas de instalación (`site-packages`).

```python
import sys
print(sys.path)
```

### Atributos útiles

```python
import math
math.__name__          # "math"
math.__file__          # ruta del archivo
dir(math)              # nombres definidos en el módulo
help(math)             # documentación
```

### `__all__`

Define qué nombres se importan con `from modulo import *`:

```python
__all__ = ["area_circulo", "PI"]
```

### Biblioteca estándar: módulos más útiles

| Módulo | Para qué sirve |
|---|---|
| `math` | Funciones matemáticas |
| `random` | Números aleatorios |
| `datetime` | Fechas y horas |
| `pathlib`, `os`, `shutil` | Rutas y sistema de archivos |
| `sys` | Argumentos y datos del intérprete |
| `json`, `csv` | Formatos de datos |
| `re` | Expresiones regulares |
| `collections` | `Counter`, `defaultdict`, `deque`, `namedtuple` |
| `itertools`, `functools` | Herramientas para iterables y funciones |
| `logging` | Registro de mensajes |
| `argparse` | Argumentos de línea de comandos |
| `unittest` | Pruebas |
| `typing`, `dataclasses` | Tipos y clases de datos |
| `urllib`, `http` | Red básica |
| `sqlite3` | Base de datos SQLite incluida |
| `statistics` | Estadística básica |
| `time` | Pausas y medición de tiempo |

### Librerías externas populares

| Librería | Uso |
|---|---|
| `requests` | Peticiones HTTP |
| `pandas`, `numpy` | Datos y cálculo numérico |
| `flask`, `django`, `fastapi` | Aplicaciones web |
| `pytest` | Tests |
| `beautifulsoup4` | Extraer datos de HTML |
| `matplotlib` | Gráficas |
| `pydantic` | Validación de datos |

Se instalan con `pip` ([[Python 17 - Entornos y paquetes]]).

## 5. Ejemplos prácticos

### Ejemplo básico

```python
import random
from datetime import datetime

print(random.choice(["rojo", "verde", "azul"]))
print(datetime.now().strftime("%d/%m/%Y %H:%M"))
```

### Ejemplo habitual: proyecto dividido en módulos

```
calculadora/
├── main.py
├── operaciones.py
└── interfaz.py
```

```python
# operaciones.py
def sumar(a, b):
    return a + b

def dividir(a, b):
    if b == 0:
        raise ValueError("No se puede dividir entre 0")
    return a / b
```

```python
# interfaz.py
def pedir_numero(mensaje):
    return float(input(mensaje))
```

```python
# main.py
from operaciones import sumar, dividir
from interfaz import pedir_numero

a = pedir_numero("a: ")
b = pedir_numero("b: ")
print(sumar(a, b))
```

### Ejemplo completo: paquete con `__init__.py`

```
mi_app/
├── main.py
└── utiles/
    ├── __init__.py
    ├── texto.py
    └── fechas.py
```

```python
# utiles/texto.py
def capitalizar(frase):
    return frase.strip().capitalize()

# utiles/fechas.py
from datetime import date
def hoy():
    return date.today().isoformat()

# utiles/__init__.py
from .texto import capitalizar
from .fechas import hoy

# main.py
from utiles import capitalizar, hoy
print(capitalizar("  hola mundo "), hoy())
```

## 6. Buenas prácticas

- Pon los `import` **al principio** del archivo, uno por línea.
- Ordénalos en tres grupos separados por una línea en blanco:
  1. biblioteca estándar,
  2. librerías externas,
  3. tus propios módulos.
- **Evita `from x import *`**: no se sabe de dónde vienen los nombres y puede tapar los tuyos.
- Usa **importaciones absolutas** salvo dentro de un paquete grande.
- Usa `if __name__ == "__main__":` en los archivos que también se ejecutan solos.
- Un módulo debería tener **una responsabilidad clara** (texto, red, base de datos…).
- Nombres de módulo en **minúsculas y `snake_case`**; evita guiones y espacios.
- No dejes código "suelto" ejecutándose al importar (llamadas a funciones, `input()`, `print()`).
- Usa los módulos de la **biblioteca estándar** antes de instalar librerías de terceros.
- Herramientas como `isort` y `ruff` ordenan los imports solas.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| Módulo vs paquete | Archivo `.py` vs carpeta con módulos |
| Paquete vs librería | Un paquete es una carpeta de código; una librería es lo que se distribuye e instala |
| `import x` vs `from x import y` | El primero exige `x.y`; el segundo trae `y` directamente |
| Importación absoluta vs relativa | Desde la raíz del proyecto vs relativa al paquete actual |
| Ejecutar vs importar | Al ejecutar, `__name__ == "__main__"`; al importar, `__name__` es el nombre del módulo |
| `import x` vs `importlib.reload(x)` | `import` no vuelve a ejecutar el módulo; `reload` sí |
| `sys.path` vs `PYTHONPATH` | `PYTHONPATH` es una variable de entorno que añade carpetas a `sys.path` |

## 8. Casos especiales

- **Importaciones circulares:** si `a.py` importa `b.py` y `b.py` importa `a.py`, puede fallar con `ImportError`. Soluciones: reorganizar el código, mover lo común a un tercer módulo, o importar dentro de la función que lo necesita.
- **Tapar módulos de la biblioteca estándar:** si llamas a tu archivo `random.py`, `math.py` o `json.py`, Python importará **el tuyo** en lugar del de la biblioteca.

❌ Un archivo llamado `random.py` que hace `import random`
🤔 Python se importa a sí mismo y falla con errores extraños.
✅ Usa otro nombre y borra la carpeta `__pycache__`.

- **`ModuleNotFoundError`:** el módulo no está instalado en el entorno activo, o la carpeta no está en `sys.path`. Comprueba que el entorno virtual está activado ([[Python 17 - Entornos y paquetes]]).
- **Importaciones relativas en scripts:** `from .modulo import x` falla si ejecutas el archivo directamente. Ejecuta el paquete con `python -m paquete.modulo`.
- **`from x import *` y `__all__`:** sin `__all__`, se importan todos los nombres que no empiezan por `_`.
- **Importación perezosa:** poner un `import` dentro de una función retrasa su carga (útil con módulos pesados o para evitar ciclos).
- **Los módulos son objetos únicos (singleton):** si un módulo guarda una variable, todos los que lo importan comparten esa misma variable.
- **Cambiar un módulo en el REPL:** Python no recarga solo; reinicia el intérprete o usa `importlib.reload`.
- **`__pycache__`:** carpeta con bytecode compilado; es seguro borrarla y no debe subirse a Git.

## 9. Resumen

- Módulo = archivo `.py`; paquete = carpeta con módulos (con `__init__.py`).
- `import x`, `import x as y`, `from x import y`; evita `import *`.
- `if __name__ == "__main__":` separa lo que se ejecuta de lo que se importa.
- Python busca módulos en `sys.modules`, los integrados y `sys.path`.
- Importaciones absolutas por defecto; relativas (`.`, `..`) solo dentro de paquetes.
- Orden de imports: estándar, externos, propios.
- Cuidado con los ciclos de importación y con nombrar tus archivos como módulos estándar.
- Aprovecha la biblioteca estándar antes de instalar nada.