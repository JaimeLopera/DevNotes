# Listas

> [!info] ¿Qué es?
> Una **lista** (`list`) es una colección **ordenada** y **modificable** de elementos. Puede contener cualquier tipo de dato, incluso mezclados, y es la estructura de datos más usada en Python.

## 1. Antes de empezar

Conviene conocer variables, bucles y el concepto de mutabilidad ([[Python 02 - Variables y tipos]], [[Python 05 - Bucles]]).

## 2. Concepto fundamental

- **Ordenada:** cada elemento tiene una posición (índice) y se mantiene el orden en que los añadiste.
- **Mutable:** puedes añadir, quitar y cambiar elementos sin crear otra lista.
- **Admite repetidos** y tipos mezclados.
- Los índices empiezan en **0**.

```
lista:   ["a", "b", "c", "d"]
índice:     0    1    2    3
negativo:  -4   -3   -2   -1
```

## 3. Sintaxis / estructura

### Crear listas

```python
frutas = ["manzana", "pera", "uva"]
vacia = []
mezcla = [1, "dos", 3.0, True, None]
desde_texto = list("hola")          # ["h", "o", "l", "a"]
desde_rango = list(range(5))        # [0, 1, 2, 3, 4]
repetida = [0] * 5                  # [0, 0, 0, 0, 0]
```

### Acceder y modificar

```python
frutas[0]        # "manzana"
frutas[-1]       # "uva" (último)
frutas[1] = "kiwi"
len(frutas)      # 3
```

Un índice fuera de rango lanza `IndexError`.

### Slicing: `[inicio:fin:paso]`

El elemento `fin` **no se incluye**. Todos los valores son opcionales.

```python
n = [0, 1, 2, 3, 4, 5]
n[1:4]       # [1, 2, 3]
n[:3]        # [0, 1, 2]
n[3:]        # [3, 4, 5]
n[-2:]       # [4, 5]
n[::2]       # [0, 2, 4]
n[::-1]      # [5, 4, 3, 2, 1, 0]  (invertida)
n[:]         # copia superficial
```

El slicing nunca da error por salirse del rango, y también sirve para **modificar**:

```python
n[1:3] = ["a", "b", "c"]     # sustituye un tramo por otro (puede cambiar el tamaño)
del n[0:2]                   # borra un tramo
```

## 4. Elementos y características

### Métodos para añadir

| Método | Qué hace |
|---|---|
| `append(x)` | Añade `x` al final |
| `insert(i, x)` | Inserta `x` en la posición `i` |
| `extend(iterable)` | Añade todos los elementos de otro iterable |
| `lista + otra` | Crea una lista nueva uniendo ambas |

### Métodos para quitar

| Método | Qué hace |
|---|---|
| `remove(x)` | Quita la **primera** aparición de `x` (error si no está) |
| `pop()` | Quita y devuelve el **último** |
| `pop(i)` | Quita y devuelve el de la posición `i` |
| `del lista[i]` | Borra por posición (también tramos) |
| `clear()` | Vacía la lista |

### Métodos de búsqueda y orden

| Método | Qué hace |
|---|---|
| `index(x)` | Posición de la primera aparición (error si no está) |
| `count(x)` | Cuántas veces aparece |
| `x in lista` | ¿Está? (`True`/`False`) |
| `sort()` | Ordena la **propia lista** (devuelve `None`) |
| `reverse()` | Invierte la propia lista |
| `copy()` | Copia superficial |

### Funciones que se usan con listas

```python
len(l)              # cantidad de elementos
sum(l)              # suma (solo números)
min(l), max(l)      # menor y mayor
sorted(l)           # NUEVA lista ordenada
reversed(l)         # iterador al revés
any(l), all(l)      # ¿alguno / todos son truthy?
```

### Ordenar con criterio

```python
palabras = ["banana", "kiwi", "manzana"]
palabras.sort(key=len)                    # por longitud
palabras.sort(reverse=True)               # de mayor a menor

personas.sort(key=lambda p: p["edad"])    # por un campo
personas.sort(key=lambda p: (p["edad"], p["nombre"]))   # por varios criterios
```

La ordenación es **estable**: los elementos iguales mantienen su orden relativo.

### Listas anidadas (matrices)

```python
matriz = [
    [1, 2, 3],
    [4, 5, 6],
]
matriz[1][2]    # 6 (fila 1, columna 2)

for fila in matriz:
    for valor in fila:
        print(valor)
```

### Desempaquetado

```python
a, b, c = [1, 2, 3]
primero, *resto = [1, 2, 3, 4]        # primero=1, resto=[2, 3, 4]
*inicio, ultimo = [1, 2, 3, 4]        # inicio=[1, 2, 3], ultimo=4
```

### Comprensiones de lista

```python
cuadrados = [n ** 2 for n in range(5)]
pares = [n for n in numeros if n % 2 == 0]
aplanada = [x for fila in matriz for x in fila]
```

(Más detalle en [[Python 05 - Bucles]].)

### Listas como pila o cola

- **Pila** (último en entrar, primero en salir): `append()` y `pop()` van muy bien.
- **Cola** (primero en entrar, primero en salir): `pop(0)` es lento en listas grandes; usa `collections.deque`.

```python
from collections import deque
cola = deque([1, 2, 3])
cola.append(4)
cola.popleft()      # 1
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
tareas = ["estudiar", "comprar"]
tareas.append("limpiar")
tareas.remove("comprar")
print(tareas)    # ['estudiar', 'limpiar']
```

### Ejemplo habitual

```python
notas = [7, 5, 9, 4, 8]

print(f"Media: {sum(notas) / len(notas):.2f}")
print(f"Mejor: {max(notas)}, peor: {min(notas)}")

aprobadas = [n for n in notas if n >= 5]
notas_ordenadas = sorted(notas, reverse=True)
```

### Ejemplo completo: quitar repetidos manteniendo el orden

```python
def sin_repetidos(elementos):
    vistos = set()
    resultado = []
    for e in elementos:
        if e not in vistos:
            vistos.add(e)
            resultado.append(e)
    return resultado

sin_repetidos([3, 1, 3, 2, 1])    # [3, 1, 2]
```

(Con `list(dict.fromkeys(elementos))` se consigue lo mismo en una línea.)

## 6. Buenas prácticas

- Usa **comprensiones** para transformar o filtrar listas.
- Usa `in` para comprobar si un elemento está, y `if lista:` para saber si no está vacía.
- Recorre con `for elemento in lista` y, si necesitas el índice, `enumerate()`.
- No modifiques una lista mientras la recorres.
- Si los datos **no deben cambiar**, usa una tupla ([[Python 08 - Tuplas y conjuntos]]).
- Si buscas elementos con mucha frecuencia, un **conjunto** o un **diccionario** es mucho más rápido que una lista.
- Para listas enormes de números, considera `array` o NumPy.
- No uses `list` como nombre de variable: tapa la función `list()`.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `sort()` vs `sorted()` | `sort()` modifica la lista y devuelve `None`; `sorted()` devuelve una lista nueva |
| `append` vs `extend` | `append([1, 2])` añade **una lista** como elemento; `extend([1, 2])` añade **dos elementos** |
| `remove` vs `pop` vs `del` | Por valor / por posición devolviendo el valor / por posición sin devolver |
| `index` vs `in` | `index` da la posición (y falla si no está); `in` solo dice si está |
| `lista + otra` vs `extend` | `+` crea una lista nueva; `extend` modifica la existente |
| `l = m` vs `l = m.copy()` | La primera comparte el mismo objeto; la segunda crea una copia |
| Lista vs tupla | La lista es mutable; la tupla, no |
| Lista vs `deque` | `deque` es eficiente añadiendo/quitando por ambos extremos |

## 8. Casos especiales

- **Copia superficial vs profunda:** `copy()`, `list(l)` y `l[:]` copian la lista pero **no** las listas que haya dentro. Para copiar todo:

```python
import copy
copia = copy.deepcopy(matriz)
```

- **Trampa de las listas multiplicadas:**

❌ `matriz = [[0] * 3] * 3`
🤔 Las tres filas son **la misma lista**, así que cambiar una cambia todas.
✅ `matriz = [[0] * 3 for _ in range(3)]`

- **`sort()` devuelve `None`:** `lista = lista.sort()` borra tu lista. Escribe `lista.sort()` solo, o `lista = sorted(lista)`.
- **No se pueden ordenar tipos incompatibles:** `[3, "a", 1].sort()` da `TypeError`.
- **`remove` solo borra la primera coincidencia;** para borrar todas, usa una comprensión: `[x for x in l if x != valor]`.
- **`+=` en listas** modifica la lista en el sitio (como `extend`), a diferencia de lo que ocurre con números o textos.
- **Listas dentro de tuplas:** la tupla es inmutable, pero la lista de dentro se puede modificar.
- **Rendimiento:** añadir al final es rápido; insertar o borrar al principio es lento en listas grandes; `x in lista` recorre toda la lista.
- **Listas por comprensión muy largas** se vuelven ilegibles: usa un `for` normal.

## 9. Resumen

- Lista = colección **ordenada, mutable**, con repetidos y tipos mezclados.
- Índices desde 0 y negativos desde el final; slicing `[inicio:fin:paso]` sin incluir el final.
- Añadir: `append`, `insert`, `extend`. Quitar: `remove`, `pop`, `del`, `clear`.
- `sort()` modifica y devuelve `None`; `sorted()` devuelve una lista nueva; `key=` ordena por criterio.
- Cuidado con copias (`copy` vs `deepcopy`) y con `[[0]] * 3`.
- Comprensiones para transformar y filtrar.
- Usa conjuntos o diccionarios si buscas mucho, y tuplas si los datos no deben cambiar.