# Debugging y buenas prácticas

> [!info] ¿Qué es?
> **Depurar** (*debugging*) es el proceso de **encontrar y corregir errores** en el código. Las **buenas prácticas** son hábitos y reglas que hacen que el código sea más legible, seguro, fácil de mantener y menos propenso a fallar. Esta nota reúne las herramientas para depurar y las convenciones de estilo y calidad más usadas en Python.

## 1. Antes de empezar

Conviene dominar las excepciones ([[Python 11 - Excepciones]]), las funciones ([[Python 06 - Funciones]]) y la organización en módulos ([[Python 13 - Módulos y paquetes]]).

Equivocarse es normal. Lo importante es tener un **método** para encontrar el problema en vez de cambiar cosas al azar.

## 2. Concepto fundamental

Existen tres tipos de errores:

| Tipo | Cuándo se detecta | Ejemplo |
|---|---|---|
| **Sintaxis** | Antes de ejecutar | Falta un `:` o un paréntesis |
| **Ejecución (excepciones)** | Mientras el programa corre | `ZeroDivisionError`, `KeyError` |
| **Lógicos** | Nunca avisan | El resultado es incorrecto aunque no haya error |

Los **errores lógicos** son los más difíciles porque Python no se queja: el programa "funciona" pero hace algo equivocado.

### Método para depurar

1. **Reproduce** el error de forma consistente.
2. **Lee el mensaje** completo.
3. **Aísla** el problema: reduce el código hasta el mínimo que falla.
4. **Formula una hipótesis** ("creo que esta variable vale otra cosa").
5. **Comprueba** la hipótesis (con `print`, depurador, tests).
6. **Corrige** y **verifica** que no has roto otra cosa.
7. **Añade un test** para que no vuelva a pasar.

## 3. Sintaxis / herramientas

### Leer un *Traceback*

Se lee **de abajo arriba**: la última línea es el tipo y el mensaje del error, y las anteriores muestran el camino de llamadas.

```
Traceback (most recent call last):
  File "main.py", line 12, in <module>
    resultado = procesar(datos)
  File "main.py", line 7, in procesar
    return datos["precio"] / datos["cantidad"]
ZeroDivisionError: division by zero
```

Aquí: el error es `ZeroDivisionError`, ocurrió en la línea 7 dentro de `procesar`, llamada desde la línea 12.

### Depurar con `print()`

Rápido y útil, pero deja "basura" en el código.

```python
print(f"{precio=}, {cantidad=}")        # imprime nombre y valor
print(type(x), len(x))
```

### Depurador `pdb`

```python
breakpoint()          # el programa se detiene aquí (Python 3.7+)
```

Comandos principales:

| Comando | Qué hace |
|---|---|
| `n` (next) | Ejecuta la línea siguiente |
| `s` (step) | Entra en la función de la línea actual |
| `c` (continue) | Sigue hasta el siguiente punto de parada |
| `r` (return) | Sigue hasta que termina la función actual |
| `p variable` | Muestra el valor |
| `pp variable` | Muestra el valor formateado |
| `l` (list) | Muestra el código alrededor |
| `w` (where) | Muestra la pila de llamadas |
| `b 25` | Pone un punto de parada en la línea 25 |
| `q` (quit) | Sale |

Se puede ejecutar un script entero en el depurador con `python -m pdb archivo.py`. Para depurar tras un fallo: `python -m pdb -c continue archivo.py`.

### Depurador de VS Code

Permite poner **puntos de parada** (clic junto al número de línea), ver variables, avanzar paso a paso y evaluar expresiones, todo de forma visual. Es la opción más cómoda para empezar.

### `logging`: registro profesional

Reemplaza a los `print` de depuración en proyectos reales:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
)
logger = logging.getLogger(__name__)

logger.debug("Detalle fino")
logger.info("Inicio del proceso")
logger.warning("Algo raro pero no grave")
logger.error("Algo falló")
logger.critical("Fallo grave")
logger.exception("Error con traceback incluido")   # dentro de un except
```

| Nivel | Cuándo usarlo |
|---|---|
| `DEBUG` | Detalles para depurar |
| `INFO` | Eventos normales |
| `WARNING` | Algo inesperado, pero sigue |
| `ERROR` | Una operación falló |
| `CRITICAL` | El programa puede no continuar |

Se puede guardar en archivo: `logging.basicConfig(filename="app.log", ...)`.

### Tests con `assert` y `pytest`

```python
def sumar(a, b):
    return a + b

assert sumar(2, 3) == 5
```

Con `pytest` (se instala con pip):

```python
# test_calculos.py
from calculos import sumar, dividir
import pytest

def test_sumar():
    assert sumar(2, 3) == 5

def test_dividir_entre_cero():
    with pytest.raises(ZeroDivisionError):
        dividir(1, 0)

@pytest.mark.parametrize("a, b, esperado", [(1, 1, 2), (0, 5, 5), (-1, 1, 0)])
def test_sumar_varios(a, b, esperado):
    assert sumar(a, b) == esperado
```

Ejecutar: `pytest` (o `pytest -v`). Más detalle en [[Testing 00 - Índice|Testing]].

### Medir rendimiento

```python
import time
inicio = time.perf_counter()
...
print(time.perf_counter() - inicio)
```

Y para medir en detalle: `timeit` (trozos pequeños), `cProfile` (dónde pasa el tiempo un programa).

## 4. Buenas prácticas de código

### Estilo: PEP 8

Es la guía de estilo oficial de Python.

| Regla | Detalle |
|---|---|
| Sangría | 4 espacios |
| Longitud de línea | Hasta 79 caracteres (muchos proyectos usan 88 o 100) |
| Nombres de variables y funciones | `snake_case` |
| Nombres de clases | `PascalCase` |
| Constantes | `MAYUSCULAS_CON_GUIONES` |
| Módulos | `minusculas_con_guiones_bajos` |
| Líneas en blanco | 2 entre funciones y clases de primer nivel; 1 entre métodos |
| Espacios | `a = b + c`, no `a=b+c`; sin espacios justo dentro de paréntesis |
| Imports | Arriba, uno por línea, agrupados (estándar, externos, propios) |
| Comillas | Sé consistente (`"` o `'`) |

### Herramientas automáticas

| Herramienta | Para qué |
|---|---|
| **`ruff`** | Revisa errores y estilo (reemplaza a flake8, isort y más), muy rápido |
| **`black`** o `ruff format` | Formatea el código automáticamente |
| **`mypy`** o `pyright` | Comprueba los tipos |
| **`isort`** | Ordena imports (también lo hace `ruff`) |
| **`pre-commit`** | Ejecuta estas comprobaciones antes de cada commit |
| **`pytest`** | Ejecuta los tests |
| **`coverage`** | Mide qué parte del código cubren los tests |
| **`pip-audit`** | Busca dependencias con vulnerabilidades |

### Código limpio

- **Nombres claros y descriptivos:** `precio_total`, `es_valido`, no `x`, `tmp`, `dato2`.
- **Funciones cortas** con una sola responsabilidad.
- **Evita números y textos "mágicos"**: ponles nombre.

```python
IVA = 0.21
precio_final = precio * (1 + IVA)       # mejor que  precio * 1.21
```

- **DRY** (*Don't Repeat Yourself*): si repites código, extráelo a una función.
- **Evita anidar demasiado:** usa cláusulas de guarda y sal pronto ([[Python 04 - Condicionales]]).
- **Comentarios** que expliquen el **porqué**, no lo obvio. Mejor un nombre claro que un comentario.
- **Docstrings** en módulos, clases y funciones públicas.
- **Anotaciones de tipo** en funciones públicas:

```python
def calcular_total(precios: list[float], iva: float = 0.21) -> float:
    ...
```

- **Prefiere lo pythónico:**

```python
for i, x in enumerate(lista): ...         # en vez de range(len(lista))
if lista: ...                             # en vez de len(lista) > 0
with open(...) as f: ...                  # en vez de open/close manual
a, b = b, a                               # intercambio
nombres = [p.nombre for p in personas]    # comprensión
```

### Estructura de un buen archivo

```python
"""Descripción breve del módulo."""

import os
import sys

import requests

from mi_paquete import utilidades

CONSTANTE = 10


def funcion_publica(x: int) -> int:
    """Hace algo y devuelve algo."""
    return x * CONSTANTE


def main() -> None:
    print(funcion_publica(2))


if __name__ == "__main__":
    main()
```

### Seguridad y robustez

- **No guardes contraseñas, claves ni tokens en el código;** usa variables de entorno o un `.env` fuera de Git.
- **Valida siempre los datos externos** (del usuario, de archivos, de APIs).
- **No uses `eval()` ni `exec()`** con datos que no controlas.
- **No cargues archivos `pickle`** de origen desconocido.
- **Usa consultas parametrizadas** en bases de datos (nunca construyas SQL concatenando textos).
- **Cierra recursos** con `with`.
- **Maneja excepciones concretas** y registra los errores.
- **Mantén las dependencias actualizadas** y revisa sus vulnerabilidades.
- **Usa `secrets`** (no `random`) para generar tokens y contraseñas.

### Control de versiones y documentación

- Haz commits pequeños y con mensajes claros ([[Git 00 - Índice|Git]]).
- Escribe un `README.md` con: qué hace el proyecto, cómo instalarlo, cómo ejecutarlo y cómo lanzar los tests.
- Mantén una estructura de carpetas clara ([[Python 17 - Entornos y paquetes]]).

## 5. Ejemplos prácticos

### Ejemplo básico: código mal hecho vs bien hecho

```python
# Mal
def f(l):
    r=[]
    for i in range(len(l)):
        if l[i]%2==0:
            r.append(l[i]*1.21)
    return r

# Bien
IVA = 0.21

def precios_pares_con_iva(precios: list[int]) -> list[float]:
    """Devuelve los precios pares con IVA aplicado."""
    return [precio * (1 + IVA) for precio in precios if precio % 2 == 0]
```

### Ejemplo habitual: depurar un error lógico

```python
def media(numeros):
    total = 0
    for n in numeros:
        total += n
    return total / len(numeros) + 1       # ← error lógico escondido

# Sospecha: el resultado es siempre 1 más alto de lo esperado
print(media([2, 4, 6]))                   # imprime 5.0, esperado 4.0

# Se aísla con un breakpoint o con prints:
# total = 12, len = 3 → 12/3 = 4.0 → algo pasa después → el "+ 1" sobra
```

### Ejemplo completo: función con validación, logging y tests

```python
import logging

logger = logging.getLogger(__name__)


def dividir(a: float, b: float) -> float:
    """Divide a entre b. Lanza ValueError si b es 0."""
    if b == 0:
        logger.error("Intento de dividir %s entre cero", a)
        raise ValueError("El divisor no puede ser 0")
    return a / b
```

```python
# test_dividir.py
import pytest
from operaciones import dividir

def test_dividir_normal():
    assert dividir(10, 2) == 5

def test_dividir_entre_cero():
    with pytest.raises(ValueError):
        dividir(1, 0)
```

## 6. Consejos para depurar mejor

- **Lee el error completo,** sobre todo la última línea; muchas veces ya dice qué pasa.
- **Busca el mensaje exacto** del error en el buscador (sin las partes específicas de tu código).
- **Reduce el problema:** crea el ejemplo más pequeño que reproduce el fallo.
- **Comprueba tus suposiciones:** imprime tipos y valores (`type(x)`, `repr(x)`).
- **Divide y vencerás:** comenta o desactiva mitades del código para ver cuál causa el fallo.
- **Cambia una sola cosa a la vez.**
- **Descansa:** un error que no ves tras mucho tiempo suele resolverse con un paseo.
- **Explícaselo a alguien** (o a un patito de goma): el método *rubber duck debugging* funciona.
- **Usa control de versiones:** `git diff` y `git bisect` ayudan a localizar cuándo apareció un error.
- **Revisa los datos de entrada** antes de sospechar del código.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `print` vs `logging` | `print` es temporal y sin niveles; `logging` se configura, tiene niveles y puede ir a archivos |
| `assert` vs excepción | `assert` es para desarrollo; la validación de datos reales usa excepciones |
| `breakpoint()` vs `print` | El depurador permite explorar el estado interactivamente |
| Test unitario vs de integración | Uno prueba una pieza aislada; el otro, varias juntas ([[Testing 00 - Índice\|Testing]]) |
| Linter vs formateador | El linter detecta problemas; el formateador reescribe el estilo |
| Tipado estático vs dinámico | `mypy` comprueba tipos sin ejecutar; Python no los obliga al ejecutar |
| Error de sintaxis vs de ejecución | Uno impide que arranque el programa; el otro aparece mientras corre |
| `repr()` vs `str()` | `repr` es para depurar (muestra comillas y escapes) |

## 8. Casos especiales

- **Errores que "desaparecen" con `print`:** a veces añadir un `print` cambia el comportamiento (por ejemplo, con hilos o con orden de ejecución). Usa el depurador o `logging`.
- **Variables que no son lo que crees:** muchos bugs vienen de un tipo inesperado (`"5"` en vez de `5`) o de `None`.
- **Mutables compartidos:** listas o diccionarios modificados desde otro sitio ([[Python 02 - Variables y tipos]], [[Python 07 - Listas]]).
- **Errores intermitentes:** dependen del tiempo, de la red o de la concurrencia; registra mucha información (`logging`) y reproduce con los mismos datos.
- **`except` que oculta errores:** un `except: pass` esconde la causa real; elimínalo para ver el error original.
- **Efectos de la caché:** `__pycache__` desactualizado raramente causa problemas; si dudas, bórralo.
- **Diferencias entre sistemas:** rutas (`/` vs `\`), codificación, saltos de línea y versiones de Python o librerías.
- **Depurar código de otras personas:** empieza por los tests, el `README` y el historial de Git.
- **Código asíncrono y con hilos:** las trazas son más difíciles; usa `logging` con nombre de hilo o tarea.
- **No optimices antes de medir:** "la optimización prematura es la raíz de todos los males"; mide con `cProfile` primero.
- **Cobertura de tests del 100 %** no garantiza que no haya errores; lo importante es probar los casos relevantes y los límites.

❌ Cambiar código al azar hasta que "parece funcionar"
🤔 No entiendes la causa y el error volverá o aparecerá en otro sitio.
✅ Reproduce, formula una hipótesis, compruébala y corrige con conocimiento de causa.

❌ Dejar `print` de depuración por todo el código
🤔 Ensucia la salida y se olvidan en producción.
✅ Usa `logging` con niveles, o elimina los `print` al terminar.

## 9. Resumen

- Hay errores de sintaxis, de ejecución y lógicos; los lógicos son los más traicioneros.
- Lee el *Traceback* de **abajo arriba**.
- Herramientas: `print(f"{x=}")`, `breakpoint()` / `pdb`, depurador de VS Code, `logging`, tests con `pytest`.
- Método: reproducir → aislar → hipótesis → comprobar → corregir → añadir test.
- Sigue **PEP 8** y automatiza con `ruff` y `black`; usa `mypy` para los tipos.
- Código limpio: nombres claros, funciones pequeñas, sin números mágicos, sin repetición, con docstrings.
- Seguridad: nada de secretos en el código, valida datos externos, evita `eval`, cierra recursos con `with`.
- Escribe tests para los errores que corriges, y un `README` para que otros (y tú) puedan usar el proyecto.