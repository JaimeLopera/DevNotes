# Referencia rápida de Python

> [!info] ¿Qué es?
> Resumen de consulta rápida de toda la carpeta de Python. Sirve para recordar sintaxis y reglas clave sin leer cada nota. Para el detalle, sigue los enlaces.

## Fundamentos ([[Python 01 - Fundamentos]])

```python
# comentario
print("hola", end="")                   # sep=, end=
nombre = input("¿Nombre? ")             # siempre devuelve str

if __name__ == "__main__":              # punto de entrada
    main()
```

```bash
python archivo.py          python -m modulo          python --version
```

- Bloques con `:` + **4 espacios** de sangría.
- Sensible a mayúsculas. Traceback: se lee de **abajo arriba**.

## Variables y tipos ([[Python 02 - Variables y tipos]])

| Tipo | Ejemplo | Mutable |
|---|---|---|
| `int` | `42` | No |
| `float` | `3.14` | No |
| `str` | `"hola"` | No |
| `bool` | `True` / `False` | No |
| `NoneType` | `None` | No |
| `list` | `[1, 2]` | **Sí** |
| `tuple` | `(1, 2)` | No |
| `set` | `{1, 2}` | **Sí** |
| `dict` | `{"a": 1}` | **Sí** |

```python
int("5"); float("2.5"); str(10); bool(0); list("ab")
type(x); isinstance(x, int); id(x)
a, b = b, a                             # intercambio
primero, *resto = [1, 2, 3]             # desempaquetado
```

**Falsy:** `False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`. Todo lo demás es *truthy*.

## Operadores ([[Python 03 - Operadores]])

| Tipo | Operadores |
|---|---|
| Aritméticos | `+ - * / // % **` |
| Asignación | `= += -= *= /= //= %= **=` |
| Comparación | `== != < > <= >=` (se pueden encadenar: `0 < x < 10`) |
| Lógicos | `and`, `or`, `not` (cortocircuito) |
| Pertenencia | `in`, `not in` |
| Identidad | `is`, `is not` |
| Morsa | `:=` |

Precedencia: `()` → `**` → `* / // %` → `+ -` → comparaciones → `not` → `and` → `or`.

## Condicionales ([[Python 04 - Condicionales]])

```python
if cond1:
    ...
elif cond2:
    ...
else:
    ...

estado = "mayor" if edad >= 18 else "menor"      # ternario

match comando:                                   # 3.10+
    case "inicio": ...
    case "salir" | "q": ...
    case _: ...
```

## Bucles ([[Python 05 - Bucles]])

```python
for i in range(5): ...                  # 0..4
for i, v in enumerate(lista): ...
for a, b in zip(l1, l2): ...
for k, v in d.items(): ...
while cond: ...
break   continue   else:                # else: si no hubo break

[n * 2 for n in datos if n > 0]         # lista
{n: n ** 2 for n in datos}              # diccionario
{n for n in datos}                      # conjunto
(n for n in datos)                      # generator
```

`range(inicio, fin, paso)` → el final **no** se incluye.

## Funciones ([[Python 06 - Funciones]])

```python
def f(a, b=1, /, c=2, *args, d, **kwargs) -> int:
    """Docstring."""
    return a

doble = lambda x: x * 2
f(*lista)   f(**diccionario)
```

- Sin `return` → devuelve `None`. Cuidado con **valores por defecto mutables** (usa `None`).
- Ámbito LEGB: Local → Enclosing → Global → Built-in. `global` / `nonlocal`.

## Listas ([[Python 07 - Listas]])

```python
l = [1, 2, 3]
l[0]; l[-1]; l[1:3]; l[::-1]; l[::2]
l.append(x); l.insert(i, x); l.extend(otra)
l.remove(x); l.pop(); l.pop(i); del l[i]; l.clear()
l.index(x); l.count(x); x in l
l.sort(key=..., reverse=True)           # modifica, devuelve None
sorted(l)                               # nueva lista
l.copy()   copy.deepcopy(l)
sum(l); min(l); max(l); len(l); any(l); all(l)
```

## Tuplas y conjuntos ([[Python 08 - Tuplas y conjuntos]])

```python
t = (1, 2);  uno = (5,)                 # la coma crea la tupla
x, y = t

s = {1, 2, 3};  vacio = set()
s.add(x); s.remove(x); s.discard(x)
a | b    a & b    a - b    a ^ b        # unión, intersección, diferencia, simétrica
a <= b   a.isdisjoint(b)
```

## Diccionarios ([[Python 09 - Diccionarios]])

```python
d = {"a": 1, "b": 2}
d["a"]; d.get("z", 0); d["c"] = 3; del d["a"]
d.keys(); d.values(); d.items()
d.pop(k, defecto); d.update(otro); d.setdefault(k, v)
d | otro                                # 3.9+
"a" in d                                # busca claves

from collections import Counter, defaultdict
```

## Strings ([[Python 10 - Strings]])

```python
f"{x:.2f}"  f"{x:,}"  f"{x:>10}"  f"{x=}"
t.upper(); t.lower(); t.title(); t.strip(); t.replace(a, b)
t.split(","); ",".join(lista); t.splitlines()
t.find(x); t.startswith(x); t.endswith(x); x in t
t.isdigit(); t.isalpha(); t.count(x)
t[::-1]       r"C:\ruta"      """varias líneas"""
```

Inmutables: los métodos **devuelven** un texto nuevo.

## Excepciones ([[Python 11 - Excepciones]])

```python
try:
    ...
except (ValueError, TypeError) as e:
    ...
except Exception as e:
    ...
else:        # si no hubo error
    ...
finally:     # siempre
    ...

raise ValueError("mensaje")
raise Nuevo("...") from e
class MiError(Exception): ...
assert cond, "mensaje"
```

Lo específico **antes** que lo general. Nunca `except: pass`.

## Ficheros ([[Python 12 - Ficheros]])

```python
with open("a.txt", "r", encoding="utf-8") as f:
    f.read();  f.readline();  f.readlines()
    for linea in f: ...

with open("a.txt", "w", encoding="utf-8") as f:
    f.write("texto\n")

import json
json.dump(obj, f, indent=2, ensure_ascii=False);  json.load(f)
json.dumps(obj);  json.loads(texto)

from pathlib import Path
p = Path("carpeta") / "a.txt"
p.exists(); p.read_text(encoding="utf-8"); p.write_text("x", encoding="utf-8")
p.parent.mkdir(parents=True, exist_ok=True)
Path(__file__).parent
```

Modos: `r` leer · `w` escribir (**borra**) · `a` añadir · `x` crear nuevo · `b` binario.

## Módulos y paquetes ([[Python 13 - Módulos y paquetes]])

```python
import x
import x as y
from x import a, b
from . import modulo                    # relativa (solo en paquetes)
```

Un paquete = carpeta con `__init__.py`. Orden de imports: estándar → externos → propios. Evita `import *`.

## POO ([[Python 14 - POO]])

```python
class A(Base):
    atributo_clase = 0

    def __init__(self, x):
        super().__init__()
        self.x = x

    def __str__(self): ...
    def __repr__(self): ...

    @property
    def doble(self): return self.x * 2

    @classmethod
    def crear(cls): ...

    @staticmethod
    def util(): ...

from dataclasses import dataclass
@dataclass
class P:
    x: int
    y: int = 0
```

Dunders: `__eq__`, `__lt__`, `__add__`, `__len__`, `__getitem__`, `__iter__`, `__enter__`/`__exit__`, `__call__`.

## Iteradores y generators ([[Python 15 - Iteradores y generators]])

```python
it = iter(lista);  next(it);  next(it, None)

def gen():
    yield 1
    yield from otro

(x for x in datos)                      # solo se recorre UNA vez

import itertools   # count, cycle, chain, islice, product, permutations, groupby
```

## Decoradores ([[Python 16 - Decoradores]])

```python
import functools

def deco(func):
    @functools.wraps(func)
    def envoltura(*args, **kwargs):
        return func(*args, **kwargs)
    return envoltura

@deco
def f(): ...
```

Con argumentos: una capa extra (`@repetir(3)`). Útiles: `@lru_cache`, `@property`, `@dataclass`, `@contextmanager`.

## Entornos y paquetes ([[Python 17 - Entornos y paquetes]])

```bash
python -m venv .venv
source .venv/bin/activate               # Windows: .venv\Scripts\activate
deactivate

python -m pip install paquete
python -m pip install -r requirements.txt
python -m pip freeze > requirements.txt
python -m pip list
```

`.venv/` y `.env` → siempre en `.gitignore`.

## Debugging y buenas prácticas ([[Python 18 - Debugging y buenas prácticas]])

```python
print(f"{x=}")
breakpoint()                            # pdb: n, s, c, p var, l, q
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
```

PEP 8: 4 espacios · `snake_case` · `PascalCase` (clases) · `MAYUSCULAS` (constantes). Herramientas: `ruff`, `black`, `mypy`, `pytest`.

## Funciones integradas más útiles

| Función | Qué hace |
|---|---|
| `len`, `sum`, `min`, `max`, `abs`, `round` | Cálculos básicos |
| `sorted`, `reversed`, `enumerate`, `zip` | Trabajar con secuencias |
| `map`, `filter`, `any`, `all` | Funciones sobre iterables |
| `range` | Secuencia de números |
| `int`, `float`, `str`, `bool`, `list`, `dict`, `set`, `tuple` | Conversión de tipos |
| `type`, `isinstance`, `id`, `dir`, `help` | Inspección |
| `open`, `input`, `print` | Entrada y salida |
| `getattr`, `setattr`, `hasattr` | Atributos dinámicos |
| `divmod`, `pow`, `bin`, `hex` | Matemáticas |

## Reglas de oro

1. Un entorno virtual por proyecto.
2. `with` para archivos y recursos.
3. `is None` para comprobar `None`.
4. Nunca uses mutables como valores por defecto.
5. No modifiques una colección mientras la recorres.
6. Captura excepciones concretas.
7. Comprensiones y funciones integradas antes que bucles manuales.
8. f-strings para formatear.
9. Nombres claros; funciones pequeñas con un solo propósito.
10. Nada de secretos en el código; valida los datos externos.