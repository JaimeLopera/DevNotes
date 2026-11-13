# Variables y tipos

> [!info] ¿Qué es?
> Una **variable** es un nombre que apunta a un valor guardado en memoria. Cada valor tiene un **tipo** que determina qué operaciones admite. Entender bien esto evita casi todos los errores raros de Python (copias, comparaciones, mutabilidad…).

## 1. Antes de empezar

Necesitas saber escribir y ejecutar un programa básico (ver [[Python 01 - Fundamentos]]).

## 2. Concepto fundamental

### La variable es una etiqueta, no una caja

En Python, una variable **no contiene** el valor: **apunta** a él. Los valores son **objetos** que viven en memoria, y las variables son nombres que los señalan.

```python
a = [1, 2, 3]
b = a            # b apunta al MISMO objeto que a
b.append(4)
print(a)         # [1, 2, 3, 4]  <- a también cambió
```

Cada objeto tiene tres cosas:

| Parte | Qué es | Cómo verla |
|---|---|---|
| Identidad | "Dirección" única del objeto | `id(x)` |
| Tipo | Qué clase de objeto es | `type(x)` |
| Valor | Su contenido | `x` |

### Tipado dinámico y fuerte

- **Dinámico:** el tipo está en el valor, así que la misma variable puede apuntar a tipos distintos.
- **Fuerte:** Python no mezcla tipos de forma silenciosa; `"5" + 3` da error en vez de adivinar.

```python
x = 5
x = "cinco"      # válido
"5" + 3          # TypeError
```

## 3. Sintaxis / estructura

### Crear variables

```python
nombre = "Ana"
edad = 20
altura = 1.68
es_estudiante = True
nada = None
```

### Asignación múltiple

```python
a, b, c = 1, 2, 3        # varias a la vez
x = y = 0                # mismo valor para ambas
a, b = b, a              # intercambio sin variable auxiliar
primero, *resto = [1, 2, 3, 4]   # primero=1, resto=[2, 3, 4]
```

### Reglas de nombres

- Solo letras, números y `_`; no empiezan por número.
- Distinguen mayúsculas de minúsculas.
- No pueden ser palabras reservadas (`if`, `for`, `class`…).
- Convenciones: `snake_case` para variables y funciones; `MAYUSCULAS` para constantes; `PascalCase` para clases.
- `_nombre` indica "uso interno"; `__nombre__` está reservado para Python.

Python **no tiene constantes reales**: `PI = 3.14` es solo una convención y nada impide cambiarla.

### Borrar una variable

```python
del x
```

## 4. Elementos / tipos

### Tipos básicos

| Tipo | Ejemplo | Descripción |
|---|---|---|
| `int` | `42`, `-7`, `1_000` | Enteros de tamaño ilimitado |
| `float` | `3.14`, `2e3` | Decimales (coma flotante) |
| `complex` | `2 + 3j` | Números complejos |
| `str` | `"hola"` | Texto (ver [[Python 10 - Strings]]) |
| `bool` | `True`, `False` | Verdadero o falso |
| `NoneType` | `None` | Ausencia de valor |
| `bytes` | `b"abc"` | Secuencia de bytes |

### Tipos de colecciones

| Tipo | Ejemplo | Nota |
|---|---|---|
| `list` | `[1, 2]` | Ordenada y modificable ([[Python 07 - Listas]]) |
| `tuple` | `(1, 2)` | Ordenada e inmutable ([[Python 08 - Tuplas y conjuntos]]) |
| `set` | `{1, 2}` | Sin orden ni repetidos |
| `dict` | `{"a": 1}` | Pares clave-valor ([[Python 09 - Diccionarios]]) |

### Enteros y decimales

```python
7 // 2        # 3 (división entera)
2 ** 100      # enteros enormes sin problema
0b101         # 5 (binario)
0o17          # 15 (octal)
0xFF          # 255 (hexadecimal)
```

**Problema de precisión de los `float`:**

```python
0.1 + 0.2     # 0.30000000000000004
```

Los decimales se guardan en binario y algunos no se representan exactos. Soluciones:

```python
round(0.1 + 0.2, 2)            # 0.3

from decimal import Decimal
Decimal("0.1") + Decimal("0.2")   # Decimal('0.3') exacto (útil para dinero)

import math
math.isclose(0.1 + 0.2, 0.3)      # True (para comparar floats)
```

### Booleanos

`True` y `False` (con mayúscula). Además son enteros: `True == 1` y `False == 0`.

### None

Representa "no hay valor". Es lo que devuelve una función sin `return`. Se comprueba con `is None`.

### Consultar el tipo

```python
type(5)                  # <class 'int'>
isinstance(5, int)       # True (también acepta clases hijas)
isinstance(5, (int, float))   # True: ¿es uno de estos tipos?
```

Se prefiere `isinstance()` a comparar `type(x) == int`.

### Conversión de tipos (casting)

```python
int("7")         # 7
int(3.9)         # 3 (trunca, no redondea)
float("2.5")     # 2.5
str(10)          # "10"
bool("")         # False
list("abc")      # ["a", "b", "c"]
tuple([1, 2])    # (1, 2)
set([1, 1, 2])   # {1, 2}
int("abc")       # ValueError
```

### Valores *truthy* y *falsy*

Cualquier valor puede evaluarse como verdadero o falso.

| Falsy (se comportan como `False`) | Truthy |
|---|---|
| `False`, `None` | Todo lo demás |
| `0`, `0.0` | Números distintos de 0 |
| `""` (texto vacío) | Textos con contenido |
| `[]`, `()`, `{}`, `set()` | Colecciones con elementos |

```python
if lista:      # equivale a "si la lista no está vacía"
    ...
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
nombre = "Ana"
edad = 20
print(f"{nombre} tiene {edad} años")
```

### Ejemplo habitual: conversión de datos del usuario

```python
texto = input("Precio: ")
try:
    precio = float(texto)
except ValueError:
    print("Eso no es un número")
else:
    print(f"Con IVA: {precio * 1.21:.2f}")
```

### Ejemplo completo: copia vs referencia

```python
original = [1, 2, [3, 4]]

referencia = original           # mismo objeto
copia_superficial = original.copy()

import copy
copia_profunda = copy.deepcopy(original)

original[2].append(5)
print(copia_superficial)   # [1, 2, [3, 4, 5]]  <- la sublista se comparte
print(copia_profunda)      # [1, 2, [3, 4]]     <- totalmente independiente
```

## 6. Buenas prácticas

- Nombres descriptivos: `precio_total`, no `pt`.
- No reutilices la misma variable para cosas distintas.
- Crea las variables **cerca de donde se usan**.
- Usa `is None` para comprobar `None`.
- Para dinero o cálculos exactos, usa `Decimal`.
- Añade anotaciones de tipo cuando el código crezca: `edad: int = 20`.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `==` vs `is` | `==` compara valores; `is` comprueba si es el **mismo objeto** |
| Mutable vs inmutable | Mutables (`list`, `dict`, `set`) se modifican en el sitio; inmutables (`int`, `str`, `tuple`, `bool`) no: se crea un objeto nuevo |
| Copia superficial vs profunda | La superficial copia el contenedor, no los objetos de dentro; la profunda copia todo |
| `int()` vs `round()` | `int(3.9)` da 3 (trunca); `round(3.9)` da 4 |
| `None` vs `0` vs `""` | Los tres son *falsy* pero significan cosas distintas |
| `type()` vs `isinstance()` | `isinstance` respeta la herencia |

## 8. Casos especiales

- **Enteros pequeños y `is`:** Python reutiliza algunos enteros y textos pequeños, así que `a is b` puede dar `True` por casualidad. Nunca uses `is` para comparar valores (salvo con `None`, `True`, `False`).
- **Ámbito de las variables:** las creadas dentro de una función son locales (ver [[Python 06 - Funciones]]).
- **Inmutable no significa "sin cambios aparentes":** `x += 1` en un `int` no modifica el objeto, crea otro y hace que `x` lo apunte.
- **Tupla con contenido mutable:** `t = ([1], 2)` no se puede reasignar, pero `t[0].append(5)` sí funciona.
- **Variables sin crear:** usar un nombre que no existe da `NameError`.

❌ `copia = lista` y luego modificar `copia`
🤔 Ambas variables señalan la misma lista, así que se modifican las dos.
✅ Usa `lista.copy()`, `list(lista)` o `copy.deepcopy(lista)` si hay listas anidadas.

❌ `if x == None:`
🤔 Funciona casi siempre, pero `==` puede estar sobrescrito en algunos objetos.
✅ `if x is None:`

## 9. Resumen

- Una variable es un nombre que **apunta** a un objeto; el tipo vive en el objeto.
- Tipos básicos: `int`, `float`, `str`, `bool`, `None`; colecciones: `list`, `tuple`, `set`, `dict`.
- Convierte con `int()`, `float()`, `str()`, `bool()`.
- Mutable (`list`, `dict`, `set`) vs inmutable (`int`, `str`, `tuple`).
- `==` compara valores, `is` compara identidad.
- Los valores *falsy*: `0`, `""`, `[]`, `{}`, `()`, `set()`, `None`, `False`.
- Los `float` tienen errores de precisión; usa `Decimal` para dinero.