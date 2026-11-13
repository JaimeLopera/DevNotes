# Tuplas y conjuntos

> [!info] ¿Qué es?
> Una **tupla** (`tuple`) es una colección **ordenada** e **inmutable**: una vez creada, no se puede cambiar. Un **conjunto** (`set`) es una colección **sin orden** y **sin elementos repetidos**, pensada para comprobar pertenencia y hacer operaciones matemáticas de conjuntos.

## 1. Antes de empezar

Compara estas estructuras con las listas ([[Python 07 - Listas]]) y recuerda la diferencia entre mutable e inmutable ([[Python 02 - Variables y tipos]]).

## 2. Concepto fundamental

| Estructura | Idea |
|---|---|
| **Tupla** | Un grupo fijo de datos relacionados (coordenadas, fecha, un registro). Se crea una vez y no cambia |
| **Conjunto** | Una "bolsa" de elementos únicos donde importa **qué hay**, no el orden ni las repeticiones |

Una forma sencilla de verlo:

- Lista: lista de la compra que vas editando.
- Tupla: tus coordenadas (latitud, longitud), que van juntas y no se editan.
- Conjunto: el grupo de países distintos que has visitado.

## 3. Sintaxis / estructura

### Tuplas

```python
punto = (3, 4)
vacia = ()
un_solo = (5,)               # la coma es lo que crea la tupla
sin_parentesis = 1, 2, 3     # también es una tupla
desde_lista = tuple([1, 2, 3])
```

> [!warning] Una tupla de un elemento
> `(5)` es solo el número 5 entre paréntesis. La tupla es `(5,)`: lo que la define es la **coma**.

### Conjuntos

```python
colores = {"rojo", "verde", "azul"}
vacio = set()                    # {} crea un diccionario, NO un conjunto
sin_repetidos = set([1, 2, 2, 3])    # {1, 2, 3}
letras = set("banana")               # {'b', 'a', 'n'}
```

## 4. Elementos y características

### Tuplas

**Acceso** igual que en las listas: índices, negativos y slicing.

```python
t = (10, 20, 30, 40)
t[0]        # 10
t[-1]       # 40
t[1:3]      # (20, 30)
len(t)      # 4
30 in t     # True
```

**Métodos** (solo dos, porque no se modifica):

| Método | Qué hace |
|---|---|
| `count(x)` | Cuántas veces aparece |
| `index(x)` | Posición de la primera aparición |

**Inmutabilidad:**

```python
t[0] = 99     # TypeError: no se puede modificar
```

**Desempaquetado** (muy usado):

```python
x, y = (3, 4)
a, b = b, a                   # intercambio de variables
primero, *resto = (1, 2, 3)   # primero=1, resto=[2, 3]
```

**Se pueden usar como claves de diccionario** y como elementos de un conjunto (las listas no):

```python
distancias = {(0, 0): "origen", (1, 2): "punto A"}
```

**Tuplas con nombre** (`namedtuple`): acceso por nombre además de por posición.

```python
from collections import namedtuple
Punto = namedtuple("Punto", ["x", "y"])
p = Punto(3, 4)
p.x       # 3
```

Para registros más completos se usa `dataclass` ([[Python 14 - POO]]).

### Conjuntos

**Características:**

- Sin orden: no hay índices (`s[0]` da error).
- Sin repetidos: añadir algo que ya existe no hace nada.
- Los elementos deben ser **hashables** (inmutables): números, textos, tuplas. No se admiten listas, diccionarios ni otros conjuntos.
- Comprobar pertenencia (`x in s`) es **muy rápido**, mucho más que en una lista.

**Métodos para modificar:**

| Método | Qué hace |
|---|---|
| `add(x)` | Añade un elemento |
| `update(iterable)` | Añade varios |
| `remove(x)` | Quita `x` (error `KeyError` si no está) |
| `discard(x)` | Quita `x` (sin error si no está) |
| `pop()` | Quita y devuelve un elemento **cualquiera** |
| `clear()` | Vacía el conjunto |

**Operaciones de conjuntos:**

| Operación | Operador | Método | Resultado |
|---|---|---|---|
| Unión | `a \| b` | `a.union(b)` | Todos los elementos |
| Intersección | `a & b` | `a.intersection(b)` | Los que están en ambos |
| Diferencia | `a - b` | `a.difference(b)` | Los de `a` que no están en `b` |
| Diferencia simétrica | `a ^ b` | `a.symmetric_difference(b)` | Los que están solo en uno |

**Comparaciones:**

| Operación | Método | Significado |
|---|---|---|
| `a <= b` | `a.issubset(b)` | ¿`a` está contenido en `b`? |
| `a >= b` | `a.issuperset(b)` | ¿`a` contiene a `b`? |
| — | `a.isdisjoint(b)` | ¿No tienen nada en común? |

**`frozenset`:** un conjunto inmutable (se puede usar como clave o dentro de otro conjunto).

**Comprensión de conjunto:**

```python
cuadrados = {n ** 2 for n in range(-3, 4)}    # {0, 1, 4, 9}
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
# Quitar repetidos
numeros = [1, 2, 2, 3, 3, 3]
unicos = set(numeros)         # {1, 2, 3}
```

### Ejemplo habitual: operaciones entre conjuntos

```python
clase_a = {"Ana", "Luis", "Marta"}
clase_b = {"Luis", "Pablo", "Marta"}

print(clase_a & clase_b)     # {'Luis', 'Marta'}   en ambas
print(clase_a | clase_b)     # todos
print(clase_a - clase_b)     # {'Ana'}              solo en A
print(clase_a ^ clase_b)     # {'Ana', 'Pablo'}     en una sola
```

### Ejemplo completo: tuplas para devolver varios valores y registros

```python
def estadisticas(numeros):
    return min(numeros), max(numeros), sum(numeros) / len(numeros)

minimo, maximo, media = estadisticas([4, 8, 6])

alumnos = [("Ana", 8.5), ("Luis", 6.0), ("Marta", 9.2)]
for nombre, nota in alumnos:
    print(f"{nombre}: {nota}")

mejor = max(alumnos, key=lambda a: a[1])
```

## 6. Buenas prácticas

- Usa **tuplas** para datos que van juntos y no deben cambiar (coordenadas, registros, valores devueltos por funciones).
- Usa **conjuntos** para:
  - eliminar duplicados,
  - comprobar pertenencia rápidamente,
  - comparar colecciones (qué tienen en común, qué les falta).
- Crea un conjunto vacío con `set()`, nunca con `{}`.
- Si necesitas **conservar el orden** al quitar repetidos, no uses `set`: usa `list(dict.fromkeys(lista))`.
- Para listas de tuplas con significado ("registros"), considera `namedtuple` o `dataclass` para tener nombres claros.

## 7. Diferencias importantes

| | Lista | Tupla | Conjunto |
|---|---|---|---|
| Sintaxis | `[1, 2]` | `(1, 2)` | `{1, 2}` |
| Ordenada | Sí | Sí | No |
| Mutable | Sí | No | Sí |
| Admite repetidos | Sí | Sí | No |
| Acceso por índice | Sí | Sí | No |
| Puede ser clave de diccionario | No | Sí (si todo su contenido es inmutable) | No (`frozenset` sí) |
| Búsqueda `x in ...` | Lenta (recorre) | Lenta (recorre) | Muy rápida |

Otras diferencias:

| Concepto | Diferencia |
|---|---|
| `remove` vs `discard` | `remove` falla si no existe; `discard` no |
| `set` vs `frozenset` | El segundo no se puede modificar |
| `a - b` vs `b - a` | La diferencia **no es simétrica**: depende del orden |
| `|` vs `or` | `\|` une conjuntos; `or` es lógico |

## 8. Casos especiales

- **La coma crea la tupla**, no los paréntesis: `x = 1,` es una tupla; `x = (1)` no.
- **Inmutabilidad superficial:** una tupla que contiene una lista permite modificar esa lista.

```python
t = ([1, 2], 3)
t[0].append(99)      # funciona: t == ([1, 2, 99], 3)
t[0] = [5]           # TypeError
```

- **Una tupla con contenido mutable no es hashable:** no puede ser clave de diccionario ni entrar en un conjunto.
- **Conjuntos no ordenados:** al imprimir un conjunto, el orden puede variar. No dependas de él. Si necesitas ordenarlo: `sorted(conjunto)` (devuelve una lista).
- **`True` y `1` son iguales para un conjunto:** `{1, True}` tiene un solo elemento (igual con `0` y `False`).
- **`set("hola")`** separa por caracteres; para un conjunto con textos enteros: `{"hola"}`.
- **Un elemento "mutable" no entra en un conjunto:**

❌ `{[1, 2], [3, 4]}`
🤔 Las listas no son hashables, así que no pueden ser elementos de un conjunto.
✅ Usa tuplas: `{(1, 2), (3, 4)}`.

❌ `vacio = {}`
🤔 Eso crea un **diccionario** vacío.
✅ `vacio = set()`

## 9. Resumen

- **Tupla:** ordenada e inmutable; `(1, 2)`; la coma la define; se desempaqueta fácilmente; puede ser clave de diccionario.
- **Conjunto:** sin orden y sin repetidos; `{1, 2}` o `set()`; búsqueda muy rápida; solo admite elementos inmutables.
- Operaciones de conjuntos: unión `|`, intersección `&`, diferencia `-`, diferencia simétrica `^`.
- `remove` falla si no existe, `discard` no.
- Para quitar repetidos conservando el orden: `list(dict.fromkeys(lista))`.
- Elige: lista si cambia, tupla si es fijo, conjunto si importa la unicidad o la rapidez de búsqueda.