# Operadores

> [!info] ¿Qué es?
> Los **operadores** son símbolos que hacen operaciones entre valores: sumar, comparar, combinar condiciones, comprobar si algo está dentro de otra cosa, etc. Los valores sobre los que actúan se llaman **operandos**.

## 1. Antes de empezar

Necesitas conocer los tipos básicos ([[Python 02 - Variables y tipos]]). Los operadores se comportan distinto según el tipo de los operandos.

## 2. Concepto fundamental

Un operador toma uno o dos operandos y devuelve un resultado. Los hay:

- **Unarios:** un operando (`-x`, `not x`).
- **Binarios:** dos operandos (`a + b`).

Un mismo símbolo puede significar cosas distintas según el tipo:

```python
5 + 3          # 8       (suma)
"ab" + "cd"    # "abcd"  (concatenación)
[1] + [2]      # [1, 2]  (unión de listas)
"ab" * 3       # "ababab"
```

## 3. Sintaxis / estructura

### Aritméticos

| Operador | Nombre | Ejemplo | Resultado |
|---|---|---|---|
| `+` | Suma | `7 + 2` | `9` |
| `-` | Resta | `7 - 2` | `5` |
| `*` | Multiplicación | `7 * 2` | `14` |
| `/` | División (siempre `float`) | `7 / 2` | `3.5` |
| `//` | División entera | `7 // 2` | `3` |
| `%` | Resto (módulo) | `7 % 2` | `1` |
| `**` | Potencia | `7 ** 2` | `49` |
| `-x` | Cambio de signo | `-7` | `-7` |

### De asignación

| Operador | Equivale a |
|---|---|
| `x = 5` | Asigna |
| `x += 2` | `x = x + 2` |
| `x -= 2` | `x = x - 2` |
| `x *= 2` | `x = x * 2` |
| `x /= 2` | `x = x / 2` |
| `x //= 2` | `x = x // 2` |
| `x %= 2` | `x = x % 2` |
| `x **= 2` | `x = x ** 2` |

### De comparación (devuelven `True` o `False`)

| Operador | Significado |
|---|---|
| `==` | Igual a |
| `!=` | Distinto de |
| `<` / `>` | Menor / mayor que |
| `<=` / `>=` | Menor o igual / mayor o igual |

### Lógicos

| Operador | Significado | Ejemplo |
|---|---|---|
| `and` | Verdadero si **ambas** lo son | `edad > 18 and tiene_dni` |
| `or` | Verdadero si **al menos una** lo es | `es_admin or es_dueño` |
| `not` | Invierte el valor | `not activo` |

### De pertenencia

| Operador | Significado |
|---|---|
| `in` | ¿Está dentro? |
| `not in` | ¿No está dentro? |

### De identidad

| Operador | Significado |
|---|---|
| `is` | ¿Es el **mismo objeto**? |
| `is not` | ¿No es el mismo objeto? |

### Bit a bit

Trabajan sobre los bits de un entero. Su uso es poco frecuente (permisos, banderas, optimización).

| Operador | Nombre | Ejemplo |
|---|---|---|
| `&` | AND | `5 & 3` → `1` |
| `\|` | OR | `5 \| 3` → `7` |
| `^` | XOR | `5 ^ 3` → `6` |
| `~` | NOT | `~5` → `-6` |
| `<<` | Desplazar a la izquierda | `1 << 3` → `8` |
| `>>` | Desplazar a la derecha | `8 >> 2` → `2` |

## 4. Elementos y características

### División, división entera y resto

```python
7 / 2     # 3.5   siempre devuelve float
7 // 2    # 3     redondea hacia abajo
7 % 2     # 1     lo que sobra
-7 // 2   # -4    hacia abajo, no hacia cero
```

Se pueden obtener los dos a la vez: `divmod(7, 2)` → `(3, 1)`.

Usos típicos del resto `%`:

```python
n % 2 == 0     # ¿es par?
n % 10         # última cifra
indice % len(lista)   # dar la vuelta a una lista (circular)
```

### Comparaciones encadenadas

Python permite escribir rangos como en matemáticas:

```python
0 < x < 10         # equivale a 0 < x and x < 10
18 <= edad <= 65
a == b == c        # ¿los tres son iguales?
```

### Comparar textos y colecciones

- Los textos se comparan **alfabéticamente por código de carácter**: `"a" < "b"` es `True`, pero `"Z" < "a"` también (las mayúsculas van antes).
- Las listas y tuplas se comparan elemento a elemento: `[1, 2] < [1, 3]` es `True`.
- No se pueden ordenar tipos incompatibles: `3 < "a"` da `TypeError`.

### Cortocircuito y valor devuelto por `and` / `or`

`and` y `or` **no devuelven siempre `True` o `False`**: devuelven uno de los operandos.

- `a and b`: si `a` es falsy, devuelve `a` y **no evalúa `b`**; si no, devuelve `b`.
- `a or b`: si `a` es truthy, devuelve `a` y **no evalúa `b`**; si no, devuelve `b`.

```python
"" or "valor por defecto"      # "valor por defecto"
nombre = entrada or "Anónimo"  # patrón muy común
lista and lista[0]             # evita error si la lista está vacía
```

### Precedencia (de mayor a menor)

| Orden | Operadores |
|---|---|
| 1 | `( )` paréntesis |
| 2 | `**` |
| 3 | `+x`, `-x`, `~x` (unarios) |
| 4 | `*`, `/`, `//`, `%` |
| 5 | `+`, `-` |
| 6 | `<<`, `>>` |
| 7 | `&`, luego `^`, luego `\|` |
| 8 | Comparaciones, `in`, `is` |
| 9 | `not` |
| 10 | `and` |
| 11 | `or` |

Casos que sorprenden:

```python
-2 ** 2        # -4   (la potencia va antes que el signo)
2 ** 3 ** 2    # 512  (la potencia se agrupa por la derecha: 2 ** 9)
not a == b     # not (a == b)
```

Si dudas, **usa paréntesis**.

### Operador morsa `:=` (Python 3.8+)

Asigna un valor **dentro de una expresión**:

```python
if (n := len(lista)) > 3:
    print(f"Hay {n} elementos")

while (linea := f.readline()):
    procesar(linea)
```

### Operadores con colecciones y textos

```python
"ab" * 3            # "ababab"
[0] * 3             # [0, 0, 0]
"a" in "casa"       # True (busca subtexto)
3 in [1, 2, 3]      # True
"clave" in dic      # True si es una clave del diccionario
{1, 2} | {2, 3}     # unión de conjuntos
{1, 2} & {2, 3}     # intersección
d1 | d2             # unión de diccionarios (3.9+)
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
a, b = 17, 5
print(a // b, a % b)    # 3 2
```

### Ejemplo habitual

```python
edad = 25
es_adulto = edad >= 18
puede_conducir = es_adulto and edad < 80
descuento = 0.2 if edad < 26 or edad > 65 else 0
```

### Ejemplo completo: año bisiesto

```python
def es_bisiesto(anio):
    return (anio % 4 == 0 and anio % 100 != 0) or anio % 400 == 0

es_bisiesto(2024)   # True
es_bisiesto(1900)   # False
es_bisiesto(2000)   # True
```

## 6. Buenas prácticas

- Usa **paréntesis** cuando mezcles `and` y `or`, aunque no hagan falta.
- Prefiere `x += 1` a `x = x + 1`.
- Comprueba `None` con `is None`.
- Para comparar decimales usa `math.isclose()`.
- Para condiciones largas, guárdalas en variables con nombre claro:

```python
es_valido = edad >= 18 and tiene_dni and not esta_bloqueado
```

- No abuses de la morsa: si hace el código menos claro, evítala.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `=` vs `==` | `=` asigna; `==` compara |
| `/` vs `//` | `/` siempre da decimal; `//` redondea hacia abajo |
| `==` vs `is` | Valor igual vs mismo objeto |
| `and` / `or` vs `&` / `\|` | Los primeros son lógicos; los segundos, bit a bit (y en conjuntos, intersección/unión) |
| `in` en texto vs lista | En texto busca **subtextos**; en listas, elementos exactos |
| `not x == y` vs `x != y` | Equivalen, pero `!=` es más claro |

## 8. Casos especiales

- **Operadores sobrecargados:** tus propias clases pueden definir qué hace `+`, `==`, `<`… con métodos especiales (ver [[Python 14 - POO]]).
- **Dividir entre cero:** `ZeroDivisionError` con `/`, `//` y `%`.
- **Precisión de float:** `0.1 + 0.2 == 0.3` es `False` (ver [[Python 02 - Variables y tipos]]).
- **`+=` en listas:** modifica la lista en el sitio (equivale a `extend`); en números y textos crea un objeto nuevo.
- **Comparar con `True`:** `if x == True:` casi nunca es lo que quieres; escribe `if x:`.
- **Encadenar `!=`:** `a != b != c` solo comprueba pares vecinos (a≠b y b≠c), no que los tres sean distintos.

❌ `if edad > 18 or 65:`
🤔 `65` es siempre *truthy*, así que la condición siempre es verdadera.
✅ `if edad > 18 or edad == 65:` o `if edad in (18, 65):`.

## 9. Resumen

- Aritméticos: `+ - * / // % **`; `/` siempre da decimal.
- Asignación compuesta: `+=`, `-=`, `*=`…
- Comparación: `== != < > <= >=`, y se pueden **encadenar** (`0 < x < 10`).
- Lógicos: `and`, `or`, `not`; hacen **cortocircuito** y devuelven operandos.
- `in` para pertenencia; `is` para identidad.
- Precedencia: paréntesis → `**` → `* / // %` → `+ -` → comparaciones → `not` → `and` → `or`.
- `:=` asigna dentro de una expresión.
- Ante la duda, paréntesis.