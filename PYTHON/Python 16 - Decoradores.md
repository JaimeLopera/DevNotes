# Decoradores

> [!info] ¿Qué es?
> Un **decorador** es una función que **recibe otra función, la envuelve y devuelve una versión con comportamiento extra**, sin tocar el código original. Se aplica escribiendo `@nombre` justo encima de la función. Se usan para registrar llamadas, medir tiempos, comprobar permisos, guardar resultados en caché y mucho más.

## 1. Antes de empezar

Necesitas tener muy claro que **las funciones son objetos**: se pueden guardar en variables, pasar como argumento y devolver desde otras funciones. También conviene entender `*args` / `**kwargs` y los *closures* ([[Python 06 - Funciones]]).

## 2. Concepto fundamental

Escribir esto:

```python
@decorador
def saludar():
    ...
```

es **exactamente lo mismo** que escribir esto:

```python
def saludar():
    ...

saludar = decorador(saludar)
```

Es decir: el nombre `saludar` pasa a apuntar a la **versión envuelta** que devuelve el decorador.

Idea visual:

```
llamada  ->  [ envoltura: antes ] -> función original -> [ envoltura: después ]  ->  resultado
```

## 3. Sintaxis / estructura

### Decorador básico (plantilla que se repite siempre)

```python
import functools

def mi_decorador(func):
    @functools.wraps(func)                 # conserva nombre y docstring de la original
    def envoltura(*args, **kwargs):
        # código ANTES de la función
        resultado = func(*args, **kwargs)  # llama a la original
        # código DESPUÉS de la función
        return resultado
    return envoltura

@mi_decorador
def sumar(a, b):
    return a + b
```

Tres piezas:

| Pieza | Para qué sirve |
|---|---|
| `func` | La función original que se decora |
| `envoltura` | La nueva función que la rodea |
| `*args, **kwargs` | Para aceptar cualquier combinación de argumentos |

## 4. Elementos y características

### `functools.wraps`

Sin `@functools.wraps(func)`, la función decorada pierde su identidad:

```python
sumar.__name__    # con wraps: "sumar"   |   sin wraps: "envoltura"
sumar.__doc__     # con wraps conserva el docstring
```

Esto afecta a la depuración, a la documentación y a herramientas que inspeccionan funciones. **Úsalo siempre.**

### Decoradores con argumentos

Necesitan **una capa más**: una función que recibe los argumentos y devuelve el decorador.

```python
def repetir(veces):
    def decorador(func):
        @functools.wraps(func)
        def envoltura(*args, **kwargs):
            for _ in range(veces):
                resultado = func(*args, **kwargs)
            return resultado
        return envoltura
    return decorador

@repetir(3)
def saludar():
    print("Hola")
```

Equivale a `saludar = repetir(3)(saludar)`.

### Decoradores opcionales con y sin paréntesis

```python
def registrar(func=None, *, nivel="INFO"):
    def decorador(f):
        @functools.wraps(f)
        def envoltura(*a, **k):
            print(f"[{nivel}] llamando a {f.__name__}")
            return f(*a, **k)
        return envoltura
    return decorador(func) if func else decorador

@registrar                  # sin argumentos
def a(): ...

@registrar(nivel="DEBUG")   # con argumentos
def b(): ...
```

Es un patrón avanzado; solo úsalo si de verdad lo necesitas.

### Varios decoradores a la vez

Se aplican **de abajo hacia arriba** (el más cercano a la función primero):

```python
@a
@b
def f(): ...

# equivale a: f = a(b(f))
```

### Decoradores como clases

Una clase con `__call__` también puede ser un decorador, útil si necesita **guardar estado** (por ejemplo, contar llamadas):

```python
class ContarLlamadas:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.llamadas = 0

    def __call__(self, *args, **kwargs):
        self.llamadas += 1
        return self.func(*args, **kwargs)

@ContarLlamadas
def hola():
    print("hola")

hola(); hola()
print(hola.llamadas)    # 2
```

### Decorar clases

Un decorador también puede recibir una **clase** y devolverla modificada. El ejemplo más conocido es `@dataclass` ([[Python 14 - POO]]).

### Decoradores incorporados y de la biblioteca estándar

| Decorador | Para qué sirve |
|---|---|
| `@property` | Un método se usa como atributo |
| `@staticmethod` / `@classmethod` | Tipos de método de clase |
| `@functools.wraps` | Conserva la identidad de la función envuelta |
| `@functools.lru_cache` | Guarda en caché resultados de llamadas repetidas |
| `@functools.cache` | Caché sin límite (3.9+) |
| `@functools.cached_property` | Calcula una propiedad una sola vez |
| `@functools.singledispatch` | Comportamiento distinto según el tipo del argumento |
| `@functools.total_ordering` | Completa los métodos de comparación de una clase |
| `@dataclasses.dataclass` | Genera código de clases de datos |
| `@contextlib.contextmanager` | Crea gestores de contexto con un generator |
| `@abc.abstractmethod` | Marca métodos abstractos |
| `@typing.overload` | Declara varias firmas para tipado |

### `@contextmanager`

```python
from contextlib import contextmanager

@contextmanager
def en_carpeta(ruta):
    import os
    anterior = os.getcwd()
    os.chdir(ruta)
    try:
        yield
    finally:
        os.chdir(anterior)

with en_carpeta("/tmp"):
    ...
```

### Decoradores en frameworks web

Muy habituales para definir rutas y permisos (`@app.route("/")` en Flask, `@app.get("/")` en FastAPI, `@login_required` en Django).

## 5. Ejemplos prácticos

### Ejemplo básico: medir el tiempo de una función

```python
import time, functools

def medir_tiempo(func):
    @functools.wraps(func)
    def envoltura(*args, **kwargs):
        inicio = time.perf_counter()
        resultado = func(*args, **kwargs)
        print(f"{func.__name__} tardó {time.perf_counter() - inicio:.4f} s")
        return resultado
    return envoltura

@medir_tiempo
def calcular():
    return sum(range(1_000_000))
```

### Ejemplo habitual: caché con `lru_cache`

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    return n if n < 2 else fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(80)               # instantáneo gracias a la caché
fibonacci.cache_info()      # estadísticas de aciertos
fibonacci.cache_clear()     # vaciar la caché
```

### Ejemplo completo: comprobar permisos y reintentar

```python
import functools, time

def requiere_rol(rol):
    def decorador(func):
        @functools.wraps(func)
        def envoltura(usuario, *args, **kwargs):
            if rol not in usuario.get("roles", []):
                raise PermissionError(f"Se necesita el rol {rol}")
            return func(usuario, *args, **kwargs)
        return envoltura
    return decorador

def reintentar(veces=3, espera=1):
    def decorador(func):
        @functools.wraps(func)
        def envoltura(*args, **kwargs):
            for intento in range(1, veces + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if intento == veces:
                        raise
                    print(f"Fallo ({e}); reintento {intento}/{veces}")
                    time.sleep(espera)
        return envoltura
    return decorador

@requiere_rol("admin")
@reintentar(veces=2, espera=0.5)
def borrar_cuenta(usuario, id_cuenta):
    ...
```

## 6. Buenas prácticas

- **Usa siempre `@functools.wraps(func)`** dentro del decorador.
- Acepta `*args, **kwargs` y **devuelve el resultado** de la función original.
- Haz que cada decorador tenga **una sola responsabilidad** (medir, registrar, validar…).
- Dale un **nombre descriptivo** (`medir_tiempo`, `requiere_login`).
- Documenta qué hace y si modifica argumentos o resultado.
- Usa los decoradores de `functools` antes de escribir los tuyos.
- No abuses: si apilas muchos decoradores, el flujo se vuelve difícil de seguir.
- Si el decorador necesita guardar estado, valora una clase o un diccionario externo.
- Para decoradores con argumentos, mantén la estructura de tres niveles clara y con nombres distintos (`repetir` → `decorador` → `envoltura`).

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| Decorador simple vs con argumentos | El simple tiene 2 niveles de función; el que lleva argumentos, 3 |
| `@decorador` vs `@decorador()` | El primero pasa la función; el segundo **llama** al decorador (devuelve otro decorador) |
| Función vs closure | El closure recuerda variables de la función que lo creó |
| Decorador vs herencia | El decorador añade comportamiento sin crear una subclase |
| Decorador de función vs de clase | Reciben una función o una clase, respectivamente |
| `lru_cache` vs `cache` | `lru_cache` puede limitar el tamaño; `cache` no |
| Decorar vs llamar | El decorador se ejecuta **al definir** la función; la envoltura, **al llamarla** |

## 8. Casos especiales

- **Cuándo se ejecuta cada parte:** el cuerpo del decorador (fuera de `envoltura`) corre **una vez, al definir la función**; el cuerpo de `envoltura` corre **cada vez que la llamas**.

```python
def deco(f):
    print("Decorando")         # una vez, al importar o definir
    def w(*a, **k):
        print("Llamando")      # en cada llamada
        return f(*a, **k)
    return w
```

- **Olvidar el `return` de la envoltura:** la función decorada devolverá siempre `None`.

❌
```python
def envoltura(*args, **kwargs):
    func(*args, **kwargs)      # falta return
```
🤔 El resultado de la función original se pierde.
✅ `return func(*args, **kwargs)`

- **Olvidar `return envoltura`:** el nombre decorado pasa a ser `None` y al llamarlo da `TypeError`.
- **`lru_cache` y argumentos mutables:** los argumentos deben ser **hashables** (nada de listas); si no, `TypeError`.
- **`lru_cache` en métodos:** guarda una referencia a `self`, lo que puede mantener objetos vivos; valora `cached_property` o una caché propia.
- **Decorar métodos:** la envoltura recibe `self` como primer argumento dentro de `*args`; funciona con la plantilla estándar.
- **Orden con `@staticmethod` / `@classmethod`:** normalmente van como decorador **más externo** (arriba).
- **Funciones asíncronas:** si decoras una `async def`, la envoltura también debe ser `async` y usar `await`.
- **Depuración:** si falta `wraps`, los mensajes de error y las herramientas mostrarán el nombre `envoltura`.
- **Anotaciones de tipo:** tipar bien un decorador es difícil; se usan `ParamSpec` y `TypeVar` del módulo `typing` (tema avanzado).

## 9. Resumen

- Un decorador es una función que **recibe una función y devuelve otra mejorada**.
- `@decorador` equivale a `f = decorador(f)`.
- Plantilla: decorador → `envoltura(*args, **kwargs)` → `return func(...)` → `return envoltura`, siempre con `@functools.wraps`.
- Con argumentos: una capa extra (`@repetir(3)`).
- Varios decoradores se aplican de abajo hacia arriba.
- También existen decoradores de clase y decoradores como clases.
- De la biblioteca estándar: `property`, `lru_cache`, `cache`, `dataclass`, `contextmanager`, `singledispatch`.
- Úsalos para tareas transversales: tiempos, registro, permisos, caché y reintentos.