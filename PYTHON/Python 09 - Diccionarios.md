# Diccionarios

> [!info] ¿Qué es?
> Un **diccionario** (`dict`) guarda datos en pares **clave: valor**. En vez de acceder por posición, se accede por la **clave**, como buscar una palabra en un diccionario real o un contacto en una agenda. Es la estructura más parecida a un objeto JSON.

## 1. Antes de empezar

Conviene conocer listas y tuplas ([[Python 07 - Listas]], [[Python 08 - Tuplas y conjuntos]]) y el concepto de objeto inmutable/hashable.

## 2. Concepto fundamental

- Cada elemento es un par **clave → valor**.
- Las **claves son únicas** y deben ser **inmutables** (hashables): textos, números, tuplas.
- Los **valores** pueden ser de cualquier tipo, incluso otros diccionarios o listas.
- **Mantiene el orden de inserción** (desde Python 3.7).
- Es **mutable**: puedes añadir, cambiar y borrar pares.
- Buscar por clave es **muy rápido**, sin importar el tamaño.

## 3. Sintaxis / estructura

### Crear diccionarios

```python
persona = {"nombre": "Ana", "edad": 20, "ciudad": "Sevilla"}
vacio = {}
con_dict = dict(nombre="Ana", edad=20)               # claves como texto simple
desde_pares = dict([("a", 1), ("b", 2)])
desde_listas = dict(zip(["a", "b"], [1, 2]))         # {"a": 1, "b": 2}
valores_iguales = dict.fromkeys(["a", "b"], 0)       # {"a": 0, "b": 0}
```

### Acceder, añadir, modificar y borrar

```python
persona["nombre"]              # "Ana"   (KeyError si la clave no existe)
persona["edad"] = 21           # modificar
persona["email"] = "a@a.com"   # añadir (si no existía)
del persona["ciudad"]          # borrar
len(persona)                   # cantidad de pares
"nombre" in persona            # True (comprueba claves)
```

## 4. Elementos y características

### Acceso seguro con `get()`

```python
persona.get("telefono")            # None (sin error)
persona.get("telefono", "sin dato")   # valor por defecto
```

### Métodos principales

| Método | Qué hace |
|---|---|
| `keys()` | Vista con todas las claves |
| `values()` | Vista con todos los valores |
| `items()` | Vista con pares `(clave, valor)` |
| `get(k, d)` | Valor de `k`, o `d` si no existe |
| `pop(k, d)` | Quita `k` y devuelve su valor (`d` si no existe) |
| `popitem()` | Quita y devuelve el **último** par añadido |
| `update(otro)` | Añade/actualiza con los pares de otro diccionario |
| `setdefault(k, v)` | Devuelve el valor de `k`; si no existe, la crea con `v` |
| `clear()` | Vacía el diccionario |
| `copy()` | Copia superficial |

### Recorrer un diccionario

```python
for clave in d:                    # solo claves
    ...
for valor in d.values():           # solo valores
    ...
for clave, valor in d.items():     # ambos (lo más habitual)
    ...
```

### Unir diccionarios

```python
a = {"x": 1, "y": 2}
b = {"y": 99, "z": 3}

a | b                  # {"x": 1, "y": 99, "z": 3}   (3.9+; gana el de la derecha)
{**a, **b}             # lo mismo, válido en versiones anteriores
a.update(b)            # modifica "a" en el sitio
```

### Comprensiones de diccionario

```python
cuadrados = {n: n ** 2 for n in range(4)}            # {0: 0, 1: 1, 2: 4, 3: 9}
largos = {k: v for k, v in d.items() if len(k) > 3}
invertido = {v: k for k, v in d.items()}
```

### Diccionarios anidados

```python
usuario = {
    "nombre": "Ana",
    "contacto": {"email": "ana@mail.com", "telefono": None},
    "roles": ["admin", "editor"],
}

usuario["contacto"]["email"]       # "ana@mail.com"
usuario["roles"][0]                # "admin"
usuario.get("contacto", {}).get("email")    # acceso seguro en cadena
```

### Herramientas útiles de `collections`

```python
from collections import defaultdict, Counter

# defaultdict: valor por defecto automático
grupos = defaultdict(list)
grupos["a"].append(1)             # no hace falta crear la lista antes

# Counter: cuenta repeticiones
Counter("banana")                 # Counter({'a': 3, 'n': 2, 'b': 1})
Counter("banana").most_common(1)  # [('a', 3)]
```

### Ordenar un diccionario

Los diccionarios no se "ordenan" en sí, pero puedes crear uno nuevo con el orden deseado:

```python
por_clave = dict(sorted(d.items()))
por_valor = dict(sorted(d.items(), key=lambda par: par[1], reverse=True))
```

### Diccionarios y JSON

Se convierten fácilmente con el módulo `json` (ver [[Python 12 - Ficheros]]):

```python
import json
texto = json.dumps(persona, indent=2, ensure_ascii=False)
datos = json.loads(texto)
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
persona = {"nombre": "Ana", "edad": 20}
print(persona["nombre"])
persona["ciudad"] = "Sevilla"
```

### Ejemplo habitual: contar repeticiones

```python
conteo = {}
for letra in "banana":
    conteo[letra] = conteo.get(letra, 0) + 1
# {"b": 1, "a": 3, "n": 2}
```

### Ejemplo completo: agrupar elementos

```python
alumnos = [
    {"nombre": "Ana", "curso": "1A"},
    {"nombre": "Luis", "curso": "1B"},
    {"nombre": "Marta", "curso": "1A"},
]

por_curso = {}
for alumno in alumnos:
    por_curso.setdefault(alumno["curso"], []).append(alumno["nombre"])

# {"1A": ["Ana", "Marta"], "1B": ["Luis"]}
```

## 6. Buenas prácticas

- Usa `get()` cuando la clave pueda no existir; usa `[]` cuando **debe** existir (así el error te avisa).
- Comprueba claves con `in`: `if "edad" in persona:`.
- Usa claves **descriptivas y consistentes** (`"nombre"`, no `"n"`).
- Para contar usa `Counter`; para agrupar, `defaultdict`.
- Para datos con estructura fija y muchos campos, considera una **clase o `dataclass`** ([[Python 14 - POO]]).
- No modifiques el diccionario mientras lo recorres.
- No uses `dict` como nombre de variable: tapa el tipo `dict`.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `d[k]` vs `d.get(k)` | El primero lanza `KeyError` si no existe; el segundo devuelve `None` (o el valor por defecto) |
| `in` en diccionario vs lista | En el diccionario busca **claves** (rápido); en la lista busca elementos (lento) |
| `pop(k)` vs `del d[k]` | `pop` devuelve el valor; `del` no |
| `update` vs `\|` | `update` modifica el existente; `\|` crea uno nuevo |
| `setdefault` vs `get` | `setdefault` además **inserta** la clave si no existe |
| `copy()` vs `deepcopy()` | `copy` es superficial: los objetos anidados se comparten |
| Diccionario vs lista | Diccionario: acceso por clave; lista: acceso por posición |
| `keys()` vs `list(d.keys())` | `keys()` es una vista que se actualiza; `list()` es una copia fija |

## 8. Casos especiales

- **Claves duplicadas al crear:** gana la última.

```python
{"a": 1, "a": 2}    # {"a": 2}
```

- **Claves de tipos distintos:** `1`, `1.0` y `True` cuentan como la misma clave.
- **Claves no válidas:** las listas, otros diccionarios y conjuntos no pueden ser claves.

❌ `d = {[1, 2]: "valor"}`
🤔 Las listas son mutables y no hashables.
✅ Usa una tupla: `{(1, 2): "valor"}`.

- **Modificar mientras se recorre** da `RuntimeError`; recorre una copia:

```python
for clave in list(d):
    if condicion(clave):
        del d[clave]
```

- **Valores mutables compartidos con `fromkeys`:**

❌ `d = dict.fromkeys(["a", "b"], [])`
🤔 Todas las claves apuntan a **la misma lista**.
✅ `d = {k: [] for k in ["a", "b"]}`

- **Acceso encadenado sin protección** (`d["a"]["b"]["c"]`) falla si falta algún nivel; protege con `get` o `try/except KeyError`.
- **Orden de inserción:** se conserva, pero al volver a asignar una clave existente **no** cambia de posición.
- **`dict.items()` devuelve vistas, no listas:** no se indexan (`d.items()[0]` falla); usa `list(d.items())[0]`.

## 9. Resumen

- Diccionario = pares **clave: valor**; claves únicas e inmutables, valores de cualquier tipo.
- Acceso `d[k]` (falla si no existe) o `d.get(k, defecto)` (seguro).
- Recorrer con `.items()`, `.keys()`, `.values()`.
- Unir con `|` o `update`; crear con comprensiones.
- `Counter` para contar y `defaultdict` para agrupar.
- Mantiene el orden de inserción y búsqueda muy rápida por clave.
- Casi un espejo del JSON: ideal para datos estructurados.