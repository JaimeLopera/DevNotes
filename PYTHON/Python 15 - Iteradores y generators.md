# Iteradores y generators

> [!info] ¿Qué es?
> Un **iterador** es un objeto que entrega los elementos de una secuencia **de uno en uno**. Un **generator** es la forma más sencilla de crear iteradores: una función (o expresión) que produce valores **bajo demanda**, sin guardarlos todos en memoria. Son la base de cómo funciona `for` por dentro y permiten trabajar con datos enormes o incluso infinitos.

## 1. Antes de empezar

Necesitas dominar bucles ([[Python 05 - Bucles]]), funciones ([[Python 06 - Funciones]]) y conocer las colecciones ([[Python 07 - Listas]]). Para entender la clase iteradora a mano, ayuda saber POO ([[Python 14 - POO]]).

## 2. Concepto fundamental

### Iterable vs iterador

| Concepto | Qué es | Ejemplos |
|---|---|---|
| **Iterable** | Algo que se puede recorrer (tiene `__iter__`) | Lista, texto, diccionario, conjunto, `range`, archivo |
| **Iterador** | Objeto que produce el siguiente elemento (tiene `__next__`) | `iter(lista)`, generators |

Un iterable **produce un iterador** cuando lo recorres.

```python
lista = [10, 20, 30]
it = iter(lista)      # convierte el iterable en iterador

next(it)   # 10
next(it)   # 20
next(it)   # 30
next(it)   # StopIteration: ya no quedan elementos
```

### Qué hace `for` por dentro

```python
for x in iterable:
    ...
```

equivale a:

```python
it = iter(iterable)
while True:
    try:
        x = next(it)
    except StopIteration:
        break
    ...
```

### Evaluación perezosa (*lazy*)

Un iterador o generator **no calcula todo de golpe**: calcula cada valor justo cuando se lo pides. Esto ahorra memoria y permite secuencias infinitas.

## 3. Sintaxis / estructura

### Función generadora con `yield`

```python
def contar(limite):
    n = 0
    while n < limite:
        yield n          # entrega un valor y se "pausa" aquí
        n += 1

for x in contar(3):
    print(x)             # 0, 1, 2
```

- Una función con `yield` **no se ejecuta al llamarla**: devuelve un generator.
- Cada `next()` hace que la función avance hasta el siguiente `yield`.
- Al terminar la función, se lanza `StopIteration` automáticamente.

### Expresión generadora

Como una comprensión de lista, pero con paréntesis:

```python
cuadrados = (n ** 2 for n in range(1_000_000))   # no calcula nada todavía
next(cuadrados)      # 0
```

### Diferencia sintáctica con las comprensiones

```python
[n for n in range(5)]    # lista: se crea entera en memoria
(n for n in range(5))    # generator: valores bajo demanda
{n for n in range(5)}    # conjunto
```

## 4. Elementos y características

### Un generator guarda su estado

Entre un `yield` y el siguiente, recuerda el valor de todas sus variables locales.

```python
def fibonacci():
    a, b = 0, 1
    while True:          # infinito, pero no pasa nada: es perezoso
        yield a
        a, b = b, a + b

from itertools import islice
list(islice(fibonacci(), 8))    # [0, 1, 1, 2, 3, 5, 8, 13]
```

### Solo se pueden recorrer una vez

```python
g = (n for n in range(3))
list(g)    # [0, 1, 2]
list(g)    # []   <- ya está agotado
```

Si necesitas recorrerlo varias veces, convierte a lista (`list(g)`) o crea otro generator.

### `yield from`: delegar en otro iterable

```python
def aplanar(listas):
    for sublista in listas:
        yield from sublista        # equivale a: for x in sublista: yield x

list(aplanar([[1, 2], [3], [4, 5]]))   # [1, 2, 3, 4, 5]
```

### Usar generators con funciones integradas

`sum`, `min`, `max`, `any`, `all`, `sorted`, `list`, `tuple`, `set`, `join` aceptan cualquier iterable:

```python
sum(n ** 2 for n in range(1000))       # sin paréntesis extra y sin crear la lista
any(x < 0 for x in numeros)
", ".join(str(n) for n in range(5))
```

### Funciones integradas que devuelven iteradores

| Función | Qué hace |
|---|---|
| `range(n)` | Secuencia de números (no se guarda entera) |
| `enumerate(it)` | Pares `(índice, valor)` |
| `zip(a, b)` | Empareja elementos |
| `map(f, it)` | Aplica una función a cada elemento |
| `filter(f, it)` | Filtra los que cumplen la función |
| `reversed(seq)` | Recorre al revés |
| `open(archivo)` | Recorre línea a línea |

### `itertools`: herramientas listas para usar

```python
import itertools as it

it.count(10)                         # 10, 11, 12, ... (infinito)
it.cycle("AB")                       # A, B, A, B, ... (infinito)
it.repeat("x", 3)                    # x, x, x
it.islice(iterable, 5)               # primeros 5
it.chain([1, 2], [3, 4])             # 1, 2, 3, 4
it.zip_longest(a, b, fillvalue=0)    # como zip pero sin cortar en el más corto
it.product("AB", repeat=2)           # AA, AB, BA, BB
it.permutations([1, 2, 3])           # todas las ordenaciones
it.combinations([1, 2, 3], 2)        # (1,2), (1,3), (2,3)
it.groupby(datos, key=...)           # agrupa elementos consecutivos
it.accumulate([1, 2, 3, 4])          # 1, 3, 6, 10
it.takewhile(lambda x: x < 5, datos) # toma mientras se cumpla
it.batched(iterable, 3)              # lotes de 3 (Python 3.12+)
```

### Crear un iterador con una clase

Aunque casi siempre es más cómodo un generator, esta es la forma "manual":

```python
class Cuenta:
    def __init__(self, tope):
        self.i = 0
        self.tope = tope

    def __iter__(self):
        return self              # el propio objeto es el iterador

    def __next__(self):
        if self.i >= self.tope:
            raise StopIteration
        self.i += 1
        return self.i

list(Cuenta(3))    # [1, 2, 3]
```

### Hacer iterable una clase con un generator

```python
class Playlist:
    def __init__(self, canciones):
        self.canciones = canciones

    def __iter__(self):
        for c in self.canciones:
            yield c

for c in Playlist(["a", "b"]):
    print(c)
```

Como `__iter__` crea un generator nuevo cada vez, esta clase **se puede recorrer varias veces**.

### Generators avanzados: `send()`, `close()` y corrutinas

```python
def acumulador():
    total = 0
    while True:
        valor = yield total        # recibe un valor desde fuera
        total += valor

g = acumulador()
next(g)        # arranca; devuelve 0
g.send(5)      # 5
g.send(10)     # 15
g.close()      # termina el generator
```

Es un tema avanzado; hoy la concurrencia se hace con `async`/`await` en lugar de corrutinas basadas en generators.

## 5. Ejemplos prácticos

### Ejemplo básico

```python
def pares(limite):
    for n in range(0, limite, 2):
        yield n

print(list(pares(10)))    # [0, 2, 4, 6, 8]
```

### Ejemplo habitual: procesar un archivo enorme sin cargarlo entero

```python
def lineas_con_error(ruta):
    with open(ruta, encoding="utf-8") as f:
        for linea in f:
            if "ERROR" in linea:
                yield linea.strip()

for linea in lineas_con_error("log.txt"):
    print(linea)
```

### Ejemplo completo: tubería (*pipeline*) de generators

```python
def leer(ruta):
    with open(ruta, encoding="utf-8") as f:
        for linea in f:
            yield linea.rstrip("\n")

def sin_vacias(lineas):
    return (l for l in lineas if l.strip())

def a_numeros(lineas):
    for l in lineas:
        try:
            yield int(l)
        except ValueError:
            continue

total = sum(a_numeros(sin_vacias(leer("datos.txt"))))
```

Cada paso procesa un elemento cada vez, sin crear listas intermedias.

## 6. Buenas prácticas

- Usa generators cuando los datos sean **muy grandes**, **infinitos** o solo necesites **recorrerlos una vez**.
- Usa una **lista** si necesitas índices, `len()`, recorrerlo varias veces u ordenarlo.
- Pasa las expresiones generadoras directamente a `sum`, `any`, `all`, `max`, `min`, `join`.
- Combina generators en **tuberías** para procesar datos paso a paso.
- Usa `itertools` antes de escribir bucles a mano para combinar, agrupar o limitar secuencias.
- Si un generator se vuelve muy complejo, extrae partes a funciones más pequeñas con `yield from`.
- Documenta qué produce el generator (`Yields:` en el docstring).
- Para hacer una clase iterable, prefiere `__iter__` con `yield` en vez de implementar `__next__` a mano.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| Iterable vs iterador | El iterable se puede recorrer; el iterador produce los elementos y se agota |
| `return` vs `yield` | `return` termina la función; `yield` entrega un valor y continúa después |
| Lista vs generator | La lista guarda todo en memoria y se recorre muchas veces; el generator produce uno a uno y se recorre una vez |
| `[x for ...]` vs `(x for ...)` | Lista inmediata vs generator perezoso |
| `iter()` vs `next()` | `iter` obtiene el iterador; `next` pide el siguiente valor |
| `yield` vs `yield from` | `yield` entrega un valor; `yield from` entrega todos los de otro iterable |
| `map`/`filter` vs comprensión | Hacen lo mismo; las comprensiones suelen ser más legibles |
| Generator vs función normal | La función ejecuta todo y devuelve un resultado; el generator se pausa y reanuda |

| | Lista | Generator |
|---|---|---|
| Memoria | Guarda todo | Un valor cada vez |
| Recorridos | Los que quieras | Uno |
| `len()` | Sí | No |
| Índices `[i]` | Sí | No |
| Tamaño infinito | No | Sí |

## 8. Casos especiales

- **Generator agotado:** recorrerlo de nuevo no da error, simplemente no devuelve nada. Es una fuente típica de bugs silenciosos.

❌
```python
g = (n for n in range(3))
total = sum(g)
maximo = max(g)      # ValueError: secuencia vacía
```
🤔 `sum` ya consumió todos los valores.
✅ Guarda en lista (`datos = list(g)`) o crea el generator otra vez.

- **`next()` con valor por defecto:** `next(it, None)` devuelve `None` en lugar de lanzar `StopIteration` cuando se acaba.
- **`len()` no funciona** en generators; si necesitas contar, `sum(1 for _ in g)` (y consumes el generator).
- **Generators y excepciones:** `StopIteration` dentro de un generator se convierte en `RuntimeError`; usa `return` para terminarlo (Python 3.7+).
- **Un generator con `return valor`** guarda el valor en `StopIteration.value` (se usa en `yield from`).
- **Evaluación diferida y variables que cambian:** como los valores se calculan al consumirlos, si las variables externas cambian entre medias, el resultado puede sorprender.

```python
n = 2
g = (x * n for x in range(3))
n = 10
list(g)      # [0, 10, 20]  <- usa el n actual (10), no el de la creación
```

- **Archivos como iteradores:** se leen línea a línea, pero una vez recorridos hay que hacer `seek(0)` para repetir.
- **Diccionarios:** al recorrer, se itera sobre las claves; `.items()`, `.keys()` y `.values()` dan vistas iterables.
- **`range` no es un iterador,** es un iterable (se puede recorrer varias veces y admite `len()` e índices).
- **Depuración:** un generator mostrado con `print` solo enseña `<generator object ...>`; conviértelo a lista para ver su contenido (si es pequeño).
- **Programación asíncrona:** los generators son la base histórica de `async`/`await`, que hoy es el mecanismo recomendado.

## 9. Resumen

- **Iterable** = se puede recorrer; **iterador** = entrega elementos con `next()` hasta lanzar `StopIteration`.
- `for` usa `iter()` y `next()` por dentro.
- **Generator:** función con `yield` o expresión `(x for ...)`; produce valores bajo demanda y recuerda su estado.
- Ahorra memoria, permite secuencias infinitas y se combina bien en tuberías.
- Se recorre **una sola vez**; si lo necesitas varias, pásalo a lista.
- `yield from` delega en otro iterable.
- `itertools` ofrece herramientas listas para combinar, agrupar y limitar iterables.
- Para hacer iterable una clase, define `__iter__` con `yield`.