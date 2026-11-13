# Funciones

> [!info] ¿Qué es?
> Una **función** es un bloque de código con nombre que se puede reutilizar. Recibe datos de entrada (**parámetros**), hace algo con ellos y puede devolver un resultado. Sirven para no repetir código, dividir problemas grandes en piezas pequeñas y dar nombre a lo que hace cada parte.

## 1. Antes de empezar

Necesitas conocer variables, condicionales y bucles ([[Python 02 - Variables y tipos]], [[Python 04 - Condicionales]], [[Python 05 - Bucles]]).

Ya has usado funciones sin saberlo: `print()`, `len()`, `input()`, `range()`… son funciones que vienen con Python (*funciones integradas*).

## 2. Concepto fundamental

Una función se **define** una vez (con `def`) y se **llama** todas las veces que haga falta. Al definirla no se ejecuta nada: el código del interior solo corre cuando la llamas.

Las funciones en Python son **objetos**: se pueden guardar en variables, meter en listas y pasar como argumento a otras funciones.

```python
def saludar():
    print("Hola")

saludar()           # llamada: se ejecuta
mi_funcion = saludar   # sin paréntesis: se guarda la función, no se ejecuta
mi_funcion()
```

## 3. Sintaxis / estructura

```python
def nombre_funcion(parametro1, parametro2):
    """Descripción breve de lo que hace (docstring)."""
    resultado = parametro1 + parametro2
    return resultado
```

Partes:

| Parte | Qué es |
|---|---|
| `def` | Palabra que define la función |
| `nombre_funcion` | Nombre (en `snake_case`, mejor con verbo) |
| `(parametro1, ...)` | Parámetros: nombres que recibirán los datos |
| `:` + sangría | Inicio del cuerpo |
| `""" """` | Docstring (documentación, opcional) |
| `return` | Devuelve un valor y termina la función |

Llamada: `nombre_funcion(2, 3)`. Los valores que pasas al llamar se llaman **argumentos**.

## 4. Elementos y características

### `return`

- Devuelve un valor y **termina** la función en ese punto.
- Si no hay `return` (o es `return` a secas), devuelve `None`.
- Puede devolver varios valores: en realidad devuelve una **tupla**.

```python
def minmax(numeros):
    return min(numeros), max(numeros)

menor, mayor = minmax([4, 8, 1])   # desempaquetado
```

### Tipos de argumentos al llamar

```python
def presentar(nombre, edad, ciudad="Sevilla"):
    ...

presentar("Ana", 20)                    # posicionales (por orden)
presentar(edad=20, nombre="Ana")        # con nombre (el orden no importa)
presentar("Ana", 20, ciudad="Madrid")   # mezcla (posicionales primero)
```

### Parámetros por defecto

Se usan si no se pasa el argumento. Van **al final** de la lista de parámetros.

```python
def potencia(base, exponente=2):
    return base ** exponente
```

### `*args` y `**kwargs`

```python
def mostrar(*args, **kwargs):
    print(args)      # tupla con los argumentos posicionales extra
    print(kwargs)    # diccionario con los argumentos con nombre extra

mostrar(1, 2, a=3, b=4)
# (1, 2)
# {'a': 3, 'b': 4}
```

Y al revés, para **desempaquetar** al llamar:

```python
numeros = [1, 2, 3]
print(*numeros)              # equivale a print(1, 2, 3)

datos = {"nombre": "Ana", "edad": 20}
presentar(**datos)           # equivale a presentar(nombre="Ana", edad=20)
```

### Parámetros solo posicionales y solo con nombre

```python
def f(a, b, /, c, d, *, e, f):
    ...
```

- Antes de `/`: solo se pueden pasar **por posición**.
- Después de `*`: solo se pueden pasar **por nombre**.
- Entre ambos: de cualquiera de las dos formas.

Es útil para hacer las funciones más claras y evitar errores.

### Anotaciones de tipo (opcionales)

```python
def sumar(a: int, b: int = 0) -> int:
    return a + b
```

Python no las obliga al ejecutar; ayudan al editor y a herramientas como `mypy`.

### Docstrings

```python
def area_circulo(radio):
    """Calcula el área de un círculo.

    Args:
        radio: radio del círculo (número positivo).

    Returns:
        El área como float.
    """
    return 3.1416 * radio ** 2
```

Se leen con `help(area_circulo)` o `area_circulo.__doc__`.

### Ámbito de las variables (scope)

Regla **LEGB**: Python busca un nombre en este orden.

| Nivel | Dónde |
|---|---|
| **L**ocal | Dentro de la función actual |
| **E**nclosing | En funciones que la contienen |
| **G**lobal | A nivel de módulo (archivo) |
| **B**uilt-in | Nombres integrados (`print`, `len`…) |

```python
x = 10                  # global

def f():
    x = 5               # local, distinta de la global
    print(x)            # 5

f()
print(x)                # 10
```

Para modificar una global desde una función se usa `global` (mejor evitarlo), y para modificar una variable de una función exterior, `nonlocal`:

```python
contador = 0
def sumar():
    global contador
    contador += 1
```

### Funciones dentro de funciones y *closures*

```python
def crear_multiplicador(n):
    def multiplicar(x):
        return x * n        # recuerda "n" aunque la externa ya terminó
    return multiplicar

doble = crear_multiplicador(2)
doble(5)    # 10
```

Una función que "recuerda" variables de su entorno se llama **closure**.

### Funciones lambda

Funciones anónimas de **una sola expresión**:

```python
doble = lambda x: x * 2
sorted(personas, key=lambda p: p["edad"])
```

Úsalas solo para cosas cortas; si necesitas más, escribe un `def`.

### Funciones de orden superior

Reciben o devuelven funciones. Las más comunes:

```python
list(map(lambda x: x * 2, [1, 2, 3]))        # [2, 4, 6]
list(filter(lambda x: x > 1, [1, 2, 3]))     # [2, 3]

from functools import reduce
reduce(lambda a, b: a + b, [1, 2, 3, 4])     # 10
```

Hoy se prefieren las **comprensiones** (`[x * 2 for x in lista]`) a `map` y `filter` por legibilidad.

### Recursión

Una función que se llama a sí misma. Necesita un **caso base** que detenga la recursión.

```python
def factorial(n):
    if n <= 1:           # caso base
        return 1
    return n * factorial(n - 1)
```

Python limita la profundidad (por defecto unas 1000 llamadas); para repeticiones largas usa un bucle.

## 5. Ejemplos prácticos

### Ejemplo básico

```python
def saludar(nombre):
    return f"Hola, {nombre}"

print(saludar("Ana"))
```

### Ejemplo habitual

```python
def calcular_precio(precio, descuento=0, iva=0.21):
    """Devuelve el precio final aplicando descuento e IVA."""
    precio_con_descuento = precio * (1 - descuento)
    return round(precio_con_descuento * (1 + iva), 2)

calcular_precio(100)                    # 121.0
calcular_precio(100, descuento=0.1)     # 108.9
```

### Ejemplo completo: función flexible con `*args` y `**kwargs`

```python
def registrar(mensaje, *etiquetas, nivel="INFO", **extra):
    partes = [f"[{nivel}] {mensaje}"]
    if etiquetas:
        partes.append("etiquetas=" + ",".join(etiquetas))
    for clave, valor in extra.items():
        partes.append(f"{clave}={valor}")
    print(" | ".join(partes))

registrar("Inicio", "app", "web", nivel="DEBUG", usuario="ana")
# [DEBUG] Inicio | etiquetas=app,web | usuario=ana
```

## 6. Buenas prácticas

- **Una función, una tarea.** Si necesitas la palabra "y" para describirla, probablemente son dos funciones.
- Nombres con **verbo** y descriptivos: `calcular_total`, `leer_datos`, `es_valido`.
- Escribe **docstring** en las funciones importantes.
- Prefiere **devolver** valores a imprimirlos: así la función se puede reutilizar y probar.
- Evita modificar variables globales; pasa los datos como parámetros.
- Pocos parámetros (idealmente 3 o menos); si hay muchos, agrúpalos en un diccionario o clase.
- Funciones cortas (que quepan en pantalla).
- Usa anotaciones de tipo en funciones públicas.
- No modifiques los argumentos mutables recibidos, salvo que sea el propósito claro de la función.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `print` vs `return` | `print` muestra en pantalla; `return` entrega el valor para usarlo |
| Parámetro vs argumento | Parámetro: nombre en la definición; argumento: valor al llamar |
| `*args` vs `**kwargs` | `*args` recoge posicionales (tupla); `**kwargs` recoge con nombre (diccionario) |
| `def` vs `lambda` | `def` admite varias líneas y nombre propio; `lambda` solo una expresión |
| Variable local vs global | La local solo existe dentro de la función |
| `global` vs `nonlocal` | `global` apunta al módulo; `nonlocal` a la función exterior más cercana |
| Función vs método | El método pertenece a una clase u objeto ([[Python 14 - POO]]) |

## 8. Casos especiales

- **Argumentos por referencia de objeto:** Python pasa la **referencia** al objeto. Si es mutable (lista, diccionario) y lo modificas dentro, **cambia fuera**; si es inmutable (número, texto), no.

```python
def añadir(lista):
    lista.append(1)       # modifica la lista original

def incrementar(n):
    n += 1                # solo cambia la copia local
```

- **Valores por defecto mutables (trampa clásica):**

❌ `def agregar(x, lista=[]):`
🤔 La lista por defecto se crea **una sola vez**, al definir la función, y se comparte entre todas las llamadas.
✅ `def agregar(x, lista=None):` y dentro `if lista is None: lista = []`.

- **Funciones sin `return` explícito** devuelven `None`; un error típico es hacer `x = lista.sort()` y encontrarse `None`.
- **Recursión sin caso base** provoca `RecursionError`.
- **Funciones generadoras:** si usan `yield` en vez de `return`, no devuelven un valor sino un generator ([[Python 15 - Iteradores y generators]]).
- **Funciones como argumentos (callbacks):** se pasa el nombre **sin paréntesis** (`sorted(lista, key=len)`); con paréntesis se ejecuta en ese momento.
- **Decoradores:** envuelven funciones para añadirles comportamiento ([[Python 16 - Decoradores]]).
- **Orden de definición:** una función debe estar definida **antes de llamarla** (aunque puede llamar a otras definidas más abajo, siempre que existan en el momento de ejecutarse).

## 9. Resumen

- `def nombre(parametros):` define; `nombre(argumentos)` llama; `return` devuelve (y si no hay, devuelve `None`).
- Parámetros posicionales, con nombre, por defecto, `*args` y `**kwargs`; `/` y `*` para forzar cómo se pasan.
- Las variables creadas dentro son **locales** (regla LEGB).
- Las funciones son objetos: se pueden pasar, guardar y devolver (closures, `lambda`, `map`, `filter`).
- La recursión necesita caso base.
- Cuidado con mutables por defecto y con modificar argumentos mutables sin querer.
- Funciones pequeñas, con un propósito, nombre con verbo y docstring.