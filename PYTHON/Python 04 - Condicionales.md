# Condicionales

> [!info] ¿Qué es?
> Los **condicionales** permiten que el programa **tome decisiones**: ejecutar un bloque de código solo si se cumple una condición. Son la base de cualquier lógica.

## 1. Antes de empezar

Necesitas entender los booleanos, la idea de *truthy/falsy* ([[Python 02 - Variables y tipos]]) y los operadores de comparación y lógicos ([[Python 03 - Operadores]]).

## 2. Concepto fundamental

Python evalúa una **condición**. Si es verdadera (o *truthy*), ejecuta el bloque indentado; si no, lo salta y pasa a la siguiente opción.

```
¿condición 1? ──sí──> bloque 1
      │no
¿condición 2? ──sí──> bloque 2
      │no
    else ───────────> bloque final
```

Solo se ejecuta **una** de las ramas de una cadena `if / elif / else`.

## 3. Sintaxis / estructura

```python
if condicion1:
    # se ejecuta si condicion1 es verdadera
elif condicion2:
    # se ejecuta si la 1 no se cumple y la 2 sí
else:
    # se ejecuta si no se cumple ninguna
```

Reglas:

- Cada línea de condición termina en **dos puntos `:`**.
- El bloque va **indentado** (4 espacios).
- `elif` y `else` son **opcionales**; puede haber varios `elif`.
- No hacen falta paréntesis alrededor de la condición.
- Un bloque no puede quedar vacío: usa `pass` si no hace nada todavía.

```python
if edad >= 18:
    pass
```

## 4. Elementos y características

### Condiciones simples y compuestas

```python
if edad >= 18:
    print("Mayor de edad")

if edad >= 18 and tiene_carnet:
    print("Puede conducir")

if es_admin or es_moderador:
    print("Acceso permitido")

if not usuario_bloqueado:
    print("Puede entrar")
```

### Condiciones habituales

```python
if lista:                # lista no vacía
if not lista:            # lista vacía
if x is None:            # no tiene valor
if "a" in texto:         # contiene
if nombre == "Ana":      # igualdad
if 1 <= n <= 10:         # rango
if tipo in ("a", "b"):   # uno de varios valores
if isinstance(x, int):   # comprobación de tipo
```

### `if` anidados

```python
if usuario:
    if usuario.activo:
        print("Bienvenido")
```

Equivale a `if usuario and usuario.activo:`. Si hay demasiados niveles, simplifica (ver "Buenas prácticas").

### Operador ternario (condicional en una línea)

```python
estado = "mayor" if edad >= 18 else "menor"
```

Sintaxis: `valor_si_cierto if condicion else valor_si_falso`. Úsalo solo para casos sencillos.

### `match` / `case` (Python 3.10+)

Sirve como un "switch", pero más potente porque compara **patrones**.

```python
match comando:
    case "inicio":
        print("Empezando")
    case "salir" | "q":          # varias opciones con |
        print("Adiós")
    case _:                      # caso por defecto
        print("Desconocido")
```

Patrones más avanzados:

```python
match punto:
    case (0, 0):
        print("Origen")
    case (x, 0):
        print(f"En el eje X, x={x}")
    case (x, y):
        print(f"Punto {x},{y}")

match usuario:
    case {"rol": "admin"}:
        print("Administrador")
    case {"nombre": nombre}:
        print(f"Hola {nombre}")

match n:
    case int() if n > 0:         # con condición extra (guard)
        print("Entero positivo")
```

Los casos se evalúan **en orden** y se ejecuta el primero que coincide; no hace falta `break`.

## 5. Ejemplos prácticos

### Ejemplo básico

```python
edad = 20
if edad >= 18:
    print("Eres mayor de edad")
else:
    print("Eres menor de edad")
```

### Ejemplo habitual: clasificar una nota

```python
nota = 7.5

if nota >= 9:
    print("Sobresaliente")
elif nota >= 7:
    print("Notable")
elif nota >= 5:
    print("Aprobado")
else:
    print("Suspenso")
```

El orden importa: se empieza por la condición más exigente porque, al cumplirse una, el resto no se evalúa.

### Ejemplo completo: validar un registro

```python
def validar_usuario(nombre, edad, email):
    if not nombre:
        return "El nombre es obligatorio"
    if not isinstance(edad, int) or edad < 0:
        return "La edad no es válida"
    if edad < 18:
        return "Debes ser mayor de edad"
    if "@" not in email:
        return "El email no es válido"
    return "Usuario válido"
```

Esto es una **cláusula de guarda**: se descartan los casos malos primero y se evita anidar.

## 6. Buenas prácticas

- Usa `if lista:` en lugar de `if len(lista) > 0:`.
- Usa `is None` para `None`, y no escribas `if x == True:` (mejor `if x:`).
- **Evita el anidamiento excesivo**: sal pronto con `return`, `continue` o `raise`.

```python
# En vez de esto:
def procesar(u):
    if u is not None:
        if u.activo:
            hacer_algo(u)

# Mejor esto:
def procesar(u):
    if u is None or not u.activo:
        return
    hacer_algo(u)
```

- Guarda las condiciones largas en variables con nombre claro.
- Usa `in` para comparar con varios valores: `if x in (1, 2, 3):` en vez de `x == 1 or x == 2 or x == 3`.
- Usa un **diccionario** cuando tengas muchos `elif` que solo asocian un valor con otro:

```python
dias = {1: "lunes", 2: "martes", 3: "miércoles"}
nombre = dias.get(n, "desconocido")
```

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `elif` vs varios `if` | Con `elif` solo se ejecuta la primera rama que cumple; con varios `if`, se evalúan **todos** |
| `if x:` vs `if x == True:` | El primero comprueba *truthy*; el segundo exige que sea exactamente `True` |
| `if x:` vs `if x is not None:` | `if x:` también es falso con `0` o `""`; el otro solo con `None` |
| `match` vs `if/elif` | `match` compara **estructuras y patrones**; `if` evalúa expresiones |
| Ternario vs `if` normal | El ternario devuelve un valor; el `if` ejecuta instrucciones |

Ejemplo de la tercera diferencia:

```python
cantidad = 0
if cantidad:                  # False: 0 es falsy
    ...
if cantidad is not None:      # True: existe un valor
    ...
```

## 8. Casos especiales

- **Bloque vacío:** necesita `pass`; si no, error de sintaxis.
- **Condiciones con efectos:** `and` y `or` hacen cortocircuito, así que se puede escribir `if lista and lista[0] == 1:` sin error aunque la lista esté vacía (ver [[Python 03 - Operadores]]).
- **Asignar dentro de la condición:** con `:=`, por ejemplo `if (n := len(datos)) > 10:`.
- **`match` con clases y valores constantes:** los nombres sueltos en un `case` se interpretan como **variables que capturan**, no como comparaciones. Para comparar con una constante, usa un valor con punto (`Color.ROJO`) o un literal.
- **`match` no existe antes de 3.10.** En versiones anteriores, usa `if/elif` o un diccionario.
- **Comparar flotantes con `==`** puede fallar por precisión (usa `math.isclose`).
- **Condiciones con `else` en otros bloques:** `for` y `while` también admiten `else` (ver [[Python 05 - Bucles]]).

❌ `if x == 1 or 2 or 3:`
🤔 `2` y `3` son siempre *truthy*, así que la condición siempre es verdadera.
✅ `if x in (1, 2, 3):`

❌ `if edad = 18:`
🤔 `=` asigna; en una condición se necesita comparar.
✅ `if edad == 18:`

## 9. Resumen

- `if` / `elif` / `else`, con dos puntos e indentación; solo se ejecuta una rama.
- Las condiciones usan comparaciones, `and`, `or`, `not`, `in`, `is`.
- Cualquier valor puede ser condición (*truthy* / *falsy*).
- Ternario: `a if condicion else b`.
- `match` / `case` (3.10+) para patrones y estructuras.
- Reduce el anidamiento con cláusulas de guarda.
- Usa `in` o diccionarios para evitar cadenas largas de `elif`.