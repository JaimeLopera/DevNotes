# Bucles

> [!info] ¿Qué es?
> Un **bucle** repite un bloque de código varias veces. Python tiene dos: `for`, que recorre los elementos de una colección o secuencia, y `while`, que repite mientras una condición sea verdadera.

## 1. Antes de empezar

Conviene dominar los condicionales ([[Python 04 - Condicionales]]) y conocer las colecciones básicas ([[Python 07 - Listas]]), aunque aquí se explican lo justo para entender los bucles.

## 2. Concepto fundamental

- **`for`**: se usa cuando sabes **qué quieres recorrer** (una lista, un rango, un texto, un diccionario…).
- **`while`**: se usa cuando **no sabes cuántas vueltas** harán falta y repites hasta que algo cambie.

Cada ejecución del bloque se llama **iteración**.

### Iterables

El `for` de Python recorre cualquier **iterable**: algo que se puede recorrer elemento a elemento.

| Iterable | Qué entrega en cada vuelta |
|---|---|
| Lista, tupla, conjunto | Cada elemento |
| Texto | Cada carácter |
| Diccionario | Cada clave |
| `range()` | Cada número de la secuencia |
| Archivo | Cada línea |

(El mecanismo interno se explica en [[Python 15 - Iteradores y generators]].)

## 3. Sintaxis / estructura

### `for`

```python
for elemento in coleccion:
    # se ejecuta una vez por cada elemento
```

```python
for fruta in ["manzana", "pera", "uva"]:
    print(fruta)

for letra in "hola":
    print(letra)
```

### `while`

```python
while condicion:
    # se ejecuta mientras la condición sea verdadera
```

```python
contador = 0
while contador < 3:
    print(contador)
    contador += 1
```

> [!warning] Bucle infinito
> Si la condición del `while` nunca se vuelve falsa, el programa no termina. Asegúrate de que algo dentro del bucle cambia la condición. Para detener un programa colgado: `Ctrl + C`.

### `range()`

Genera una secuencia de números sin guardarlos todos en memoria.

| Forma | Resultado |
|---|---|
| `range(5)` | 0, 1, 2, 3, 4 |
| `range(2, 6)` | 2, 3, 4, 5 |
| `range(0, 10, 2)` | 0, 2, 4, 6, 8 |
| `range(5, 0, -1)` | 5, 4, 3, 2, 1 |

El **final nunca se incluye**.

## 4. Elementos y características

### Control del flujo: `break`, `continue`, `else`

| Palabra | Efecto |
|---|---|
| `break` | Sale del bucle de inmediato |
| `continue` | Salta el resto de la vuelta y pasa a la siguiente |
| `else` | Se ejecuta si el bucle **terminó sin `break`** |

```python
for n in numeros:
    if n < 0:
        continue          # ignora negativos
    if n == 99:
        break             # corta al encontrar 99
    print(n)
else:
    print("Terminó sin encontrar el 99")
```

El `else` del bucle es muy útil para **búsquedas**: si no hubo `break`, significa que no se encontró.

### Funciones que mejoran los bucles

```python
# enumerate: índice + valor
for i, valor in enumerate(["a", "b", "c"]):
    print(i, valor)

for i, valor in enumerate(["a", "b"], start=1):   # empezar en 1
    ...

# zip: recorrer varias secuencias a la vez
for nombre, nota in zip(nombres, notas):
    print(nombre, nota)

# reversed y sorted
for x in reversed(lista): ...
for x in sorted(lista): ...
```

`zip` se detiene cuando se acaba la secuencia **más corta**.

### Recorrer diccionarios

```python
for clave in d:                 # claves
    ...
for valor in d.values():        # valores
    ...
for clave, valor in d.items():  # ambos
    ...
```

### Bucles anidados

Un bucle dentro de otro. El interno se completa **entero** en cada vuelta del externo.

```python
for fila in range(3):
    for columna in range(3):
        print(f"({fila},{columna})", end=" ")
    print()
```

`break` y `continue` solo afectan al bucle **más interno**.

### Comprensiones: bucles en una línea

Sirven para **crear** una colección a partir de otra.

```python
cuadrados = [n ** 2 for n in range(5)]              # [0, 1, 4, 9, 16]
pares = [n for n in range(10) if n % 2 == 0]        # con filtro
etiquetas = ["par" if n % 2 == 0 else "impar" for n in range(4)]   # con if/else
cuadrado_de = {n: n ** 2 for n in range(4)}         # diccionario
unicos = {n % 3 for n in range(10)}                 # conjunto
pares_gen = (n for n in range(10) if n % 2 == 0)    # generator (no crea lista)
```

Estructura: `[expresión for elemento in iterable if condición]`.

Si se vuelve difícil de leer, usa un `for` normal.

### Repetir "hasta que..." con `while True`

Python no tiene `do...while`, pero se imita así:

```python
while True:
    texto = input("Escribe un número: ")
    if texto.isdigit():
        break
    print("Inválido, inténtalo de nuevo")
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
for i in range(1, 6):
    print(i)
```

### Ejemplo habitual: sumar y contar

```python
notas = [7, 5, 9, 4]

total = 0
aprobados = 0
for nota in notas:
    total += nota
    if nota >= 5:
        aprobados += 1

print(f"Media: {total / len(notas):.2f}, aprobados: {aprobados}")
```

(En la práctica, `sum(notas) / len(notas)` hace lo mismo más corto.)

### Ejemplo completo: buscar un número primo

```python
def es_primo(n):
    if n < 2:
        return False
    for divisor in range(2, int(n ** 0.5) + 1):
        if n % divisor == 0:
            return False
    return True

primos = [n for n in range(2, 30) if es_primo(n)]
print(primos)    # [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

## 6. Buenas prácticas

- Prefiere `for` a `while` cuando puedas: es más seguro frente a bucles infinitos.
- Usa `enumerate()` en lugar de `range(len(lista))`.
- Usa `zip()` para recorrer dos listas a la vez.
- **No modifiques una lista mientras la recorres**: crea una nueva o recorre una copia.

```python
# Quitar negativos de forma segura
numeros = [n for n in numeros if n >= 0]
```

- Usa una **comprensión** cuando solo transformas o filtras; un `for` cuando haya varios pasos.
- Para sumas, mínimos o máximos usa `sum()`, `min()`, `max()` en lugar de bucles manuales.
- Para comprobar "¿alguno cumple?" o "¿todos cumplen?", usa `any()` y `all()`:

```python
any(n < 0 for n in numeros)
all(n > 0 for n in numeros)
```

- Evita bucles anidados profundos; extrae el bucle interno a una función.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `for` vs `while` | `for` recorre algo conocido; `while` repite hasta que cambie una condición |
| `break` vs `continue` | `break` sale del bucle; `continue` solo salta una vuelta |
| `else` del bucle vs `else` del `if` | El del bucle se ejecuta si **no hubo `break`** |
| `range(5)` vs `[0,1,2,3,4]` | `range` no guarda los números; ahorra memoria |
| Lista por comprensión vs generator | La lista guarda todo; el generator produce uno a uno |
| `pass` vs `continue` | `pass` no hace nada y sigue en la misma vuelta; `continue` salta a la siguiente |

## 8. Casos especiales

- **Bucle `for` y variable de bucle:** tras acabar el bucle, la variable sigue existiendo con el último valor.
- **Recorrer un diccionario mientras se modifica** da `RuntimeError`: recorre `list(d)` (una copia de las claves).
- **`range` solo admite enteros;** para decimales usa un `while` o `numpy.arange`.
- **Bucle con `else` y `continue`:** `continue` no impide que se ejecute el `else`; solo `break` lo evita.
- **Salir de varios bucles anidados:** `break` solo sale del interno. Opciones: meterlos en una función y usar `return`, o usar una bandera (variable booleana).

```python
def buscar(matriz, objetivo):
    for fila in matriz:
        for valor in fila:
            if valor == objetivo:
                return True
    return False
```

- **Iterar y borrar a la vez** en una lista salta elementos, porque los índices se desplazan.
- **Cierre tardío en funciones creadas en un bucle:** las funciones `lambda` creadas dentro de un `for` capturan la **variable**, no su valor en ese momento (se soluciona con parámetros por defecto: `lambda x=x: ...`).

❌ `for i in range(len(lista)): print(lista[i])`
🤔 Funciona, pero es largo y propenso a errores de índice.
✅ `for elemento in lista: print(elemento)` o `enumerate(lista)` si necesitas el índice.

❌ `while x < 10:` sin cambiar `x` dentro
🤔 La condición nunca cambia y el bucle no termina.
✅ Actualiza la variable dentro del bucle (`x += 1`).

## 9. Resumen

- `for` recorre cualquier iterable; `while` repite mientras la condición sea verdadera.
- `range(inicio, fin, paso)` no incluye el final.
- `break` sale, `continue` salta una vuelta, `else` se ejecuta si no hubo `break`.
- `enumerate()` da índice y valor; `zip()` recorre varias secuencias a la vez.
- Las **comprensiones** crean listas, diccionarios, conjuntos y generators en una línea.
- No modifiques una colección mientras la recorres.
- Usa `sum`, `min`, `max`, `any`, `all` antes de escribir un bucle a mano.
- Cuidado con los bucles infinitos: algo dentro debe cambiar la condición.