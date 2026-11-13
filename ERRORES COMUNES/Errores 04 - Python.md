# 04 - Errores de Python

> [!info] ¿Qué es?
> Los errores de Python más frecuentes, con el **mensaje literal**, qué significa, sus causas y cómo solucionarlo. Python muestra un **traceback**: léelo **de abajo arriba**. La última línea dice el tipo de error; las anteriores, por dónde pasó el programa hasta llegar ahí.

---

## 1. Antes de empezar: leer un traceback

```
Traceback (most recent call last):
  File "app.py", line 12, in <module>
    total = calcular(precios)
  File "app.py", line 5, in calcular
    return sum(p["precio"] for p in lista)
KeyError: 'precio'                  ← el error (empieza a leer aquí)
```

- **Última línea**: tipo de error y mensaje.
- **Línea justo encima**: dónde ocurrió exactamente.
- **Más arriba**: quién llamó a esa función.

Más en [[Python 18 - Debugging y buenas prácticas|Debugging y buenas prácticas]].

---

## 2. Sintaxis e indentación

### 2.1 `IndentationError: expected an indented block`

**Qué significa:** después de `if`, `for`, `def`, `class`… falta el bloque sangrado.

**Solución:** indenta el contenido 4 espacios, o pon `pass` si aún está vacío.

### 2.2 `IndentationError: unindent does not match any outer indentation level`

**Qué significa:** sangrías inconsistentes, a menudo por **mezclar tabuladores y espacios**.

**Solución:** configura el editor para usar 4 espacios y convierte los tabuladores.

### 2.3 `SyntaxError: invalid syntax`

**Causas típicas:**
- Falta `:` al final de `if`, `for`, `def`, `while`, `class`.
- Usar `=` en lugar de `==` en una condición.
- Paréntesis, corchetes o comillas sin cerrar en la línea **anterior**.

### 2.4 `SyntaxError: unterminated string literal`

**Solución:** cierra las comillas; para textos de varias líneas usa `"""..."""`.

---

## 3. Nombres e importaciones

### 3.1 `NameError: name 'x' is not defined`

**Causas típicas:** error tipográfico, la variable se define después de usarse o dentro de otra función, o falta un `import`.

### 3.2 `ModuleNotFoundError: No module named 'requests'`

**Causas típicas:**
- El paquete no está instalado.
- Está instalado en **otro entorno** (otro Python o fuera del entorno virtual).

**Solución:**

```bash
source .venv/bin/activate      # Windows: .venv\Scripts\activate
python -m pip install requests
```

Usa `python -m pip` para instalar en el mismo Python que ejecutas. En VS Code, elige el intérprete del `.venv` (abajo a la derecha).

Más en [[Python 17 - Entornos y paquetes|Entornos y paquetes]].

### 3.3 `ImportError: cannot import name 'x' from 'modulo'`

**Causas típicas:**
- El nombre no existe en ese módulo (o está mal escrito).
- **Importación circular**: dos módulos se importan entre sí.
- Tu archivo se llama igual que un módulo (`random.py`, `json.py`, `requests.py`) y lo "tapa".

**Solución:** renombra tu archivo si coincide con un módulo y borra la carpeta `__pycache__`.

### 3.4 `UnboundLocalError: cannot access local variable 'x' where it is not associated with a value`

**Qué significa:** asignas una variable dentro de una función y Python la considera local, pero la usas antes de asignarla.

**Solución:** pásala como parámetro y devuélvela, o (con cuidado) usa `global`/`nonlocal`.

---

## 4. Tipos

### 4.1 `TypeError: can only concatenate str (not "int") to str`

**Qué significa:** intentas unir texto y número con `+`.

**Solución:**

```python
print("Tienes " + str(edad) + " años")
print(f"Tienes {edad} años")      # mejor
```

### 4.2 `TypeError: unsupported operand type(s) for +: 'int' and 'str'`

**Causa típica:** `input()` siempre devuelve texto.

**Solución:** `edad = int(input("Edad: "))`.

### 4.3 `TypeError: 'NoneType' object is not subscriptable`

**Qué significa:** haces `x[0]` o `x["clave"]` y `x` es `None`.

**Causas típicas:** una función sin `return`, o métodos que modifican en el sitio y devuelven `None` (`lista = lista.sort()`).

**Solución:**

```python
lista.sort()               # modifica la lista, devuelve None
ordenada = sorted(lista)   # devuelve una nueva
```

### 4.4 `AttributeError: 'NoneType' object has no attribute 'x'`

**Qué significa:** igual que 4.3, pero con `.x`.

### 4.5 `TypeError: f() missing 1 required positional argument: 'x'`

**Causas típicas:** llamas a la función con menos argumentos, u olvidas `self` en un método de clase.

### 4.6 `TypeError: 'int' object is not iterable`

**Causa típica:** `for i in len(lista)` en lugar de `for i in range(len(lista))` (o mejor, `for elemento in lista`).

### 4.7 `ValueError: invalid literal for int() with base 10: 'abc'`

**Qué significa:** intentas convertir a número un texto que no lo es.

**Solución:**

```python
try:
    numero = int(texto)
except ValueError:
    print("Introduce un número válido")
```

---

## 5. Colecciones

### 5.1 `IndexError: list index out of range`

**Causas típicas:** índice mayor o igual que la longitud, lista vacía, o bucle con `range(len(lista) + 1)`.

**Solución:** recuerda que los índices van de `0` a `len(lista) - 1`; comprueba `if lista:` antes de `lista[0]`.

### 5.2 `KeyError: 'clave'`

**Qué significa:** la clave no existe en el diccionario.

**Solución:**

```python
valor = datos.get("clave")              # None si no existe
valor = datos.get("clave", "defecto")
if "clave" in datos: ...
```

### 5.3 `RuntimeError: dictionary changed size during iteration`

**Qué significa:** añades o borras claves mientras recorres el diccionario.

**Solución:** recorre una copia: `for k in list(datos): ...`.

### 5.4 Valores por defecto mutables que "se acumulan"

```python
def anadir(item, lista=[]):   # ✘ la misma lista se reutiliza entre llamadas
def anadir(item, lista=None): # ✔
    lista = lista if lista is not None else []
```

---

## 6. Archivos y codificación

### 6.1 `FileNotFoundError: [Errno 2] No such file or directory: 'datos.json'`

**Qué significa:** la ruta es relativa a la carpeta **desde la que ejecutas**, no a la del script.

**Solución:**

```python
from pathlib import Path
ruta = Path(__file__).parent / "datos.json"
```

### 6.2 `UnicodeDecodeError: 'charmap' codec can't decode byte ...`

**Qué significa:** en Windows, Python abre los archivos con otra codificación por defecto.

**Solución:** indica siempre `encoding="utf-8"` al abrir archivos.

### 6.3 `PermissionError: [Errno 13] Permission denied`

**Causas típicas:** el archivo está abierto en otro programa (Excel), no tienes permisos, o intentas abrir una carpeta como archivo.

### 6.4 `json.decoder.JSONDecodeError: Expecting value: line 1 column 1`

**Qué significa:** el texto no es JSON válido (a menudo está vacío o es HTML de una página de error).

**Solución:** imprime el contenido antes de parsearlo; en `requests`, comprueba `res.status_code`.

---

## 7. Otros

### 7.1 `RecursionError: maximum recursion depth exceeded`

**Qué significa:** recursión sin caso base, o demasiado profunda.

### 7.2 `ZeroDivisionError: division by zero`

**Solución:** comprueba el divisor antes de dividir.

### 7.3 `python` no se reconoce como comando (Windows)

**Solución:** reinstala Python marcando *Add Python to PATH*, o usa el lanzador `py`.

---

## 8. Resumen

- Lee el traceback **de abajo arriba**.
- `IndentationError` → 4 espacios, sin mezclar tabuladores.
- `ModuleNotFoundError` → activa el entorno virtual y usa `python -m pip install`.
- `can only concatenate str` → usa f-strings; `input()` devuelve texto.
- `NoneType ...` → una función no devolvió nada (`sort()` devuelve `None`).
- `KeyError` → `dict.get()`; `IndexError` → índices de `0` a `len - 1`.
- `FileNotFoundError` → `Path(__file__).parent`; abre siempre con `encoding="utf-8"`.
