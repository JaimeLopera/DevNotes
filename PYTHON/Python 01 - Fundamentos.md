# Fundamentos de Python

> [!info] ¿Qué es?
> Python es un lenguaje de programación **interpretado**, de **tipado dinámico** y de sintaxis muy legible. Se usa para automatizar tareas, crear webs (backend), analizar datos, hacer scripts, trabajar con inteligencia artificial y mucho más. Esta nota explica cómo funciona Python por dentro y cómo ejecutar tu primer código.

## 1. Antes de empezar

No necesitas saber programar. Basta con tener Python 3 instalado y una terminal.

Comprobar que está instalado:

```bash
python --version
```

- En macOS y Linux suele llamarse `python3`: `python3 --version`.
- En Windows también existe el lanzador `py`: `py --version`.
- Si el comando no se reconoce, Python no está instalado o no está en el **PATH** (la lista de carpetas donde el sistema busca programas).

Dónde escribir código:

| Opción | Para qué sirve |
|---|---|
| Archivo `.py` + terminal | Programas completos (lo normal) |
| Intérprete interactivo (`python`) | Probar cosas sueltas al instante |
| VS Code, PyCharm | Editores con ayuda, depurador y sugerencias |
| Jupyter Notebook | Análisis de datos, mezcla de código y texto |

## 2. Concepto fundamental

### Qué significa "interpretado"

En lenguajes como C, un **compilador** convierte todo el programa en un ejecutable antes de usarlo. En Python no lo ves: el **intérprete** lee tu código y lo ejecuta directamente.

Por dentro, Python hace esto:

1. Lee tu archivo `.py`.
2. Lo convierte a **bytecode** (un código intermedio, guardado en la carpeta `__pycache__` como `.pyc`).
3. La **máquina virtual de Python** ejecuta ese bytecode.

No tienes que hacer nada de esto a mano; solo ejecutas el archivo.

### Qué significa "tipado dinámico"

No indicas el tipo de las variables: Python lo deduce según el valor, y una misma variable puede apuntar a valores de distinto tipo con el tiempo (ver [[Python 02 - Variables y tipos]]).

```python
x = 10        # entero
x = "diez"    # ahora es texto
```

### Qué significa "indentación como sintaxis"

En muchos lenguajes los bloques de código se marcan con llaves `{ }`. En Python se marcan con **dos puntos `:`** y **sangría** (espacios al principio de la línea). Todas las líneas con la misma sangría pertenecen al mismo bloque.

```python
if True:
    print("Dentro del bloque")
    print("También dentro")
print("Fuera del bloque")
```

> [!warning] Indentación obligatoria
> Si la sangría es incorrecta, Python lanza `IndentationError`. La convención es **4 espacios** por nivel. No mezcles tabuladores y espacios en el mismo archivo.

### Implementaciones de Python

Lo que normalmente usas se llama **CPython** (el Python "oficial", hecho en C). Existen otras (PyPy, Jython…), pero si no sabes cuál usas, es CPython.

## 3. Sintaxis / estructura

### Comentarios

```python
# Comentario de una línea

x = 5  # Comentario al final de una línea

"""
Texto de varias líneas.
Se usa sobre todo como docstring
(documentación de funciones, clases y módulos).
"""
```

Python no tiene comentarios de varias líneas "oficiales"; lo habitual es poner `#` en cada línea.

### Mostrar información: `print()`

```python
print("Hola")
print("a", "b", "c")              # a b c
print("a", "b", sep="-")          # a-b
print("Sin salto", end="")        # no añade salto de línea
print(f"Resultado: {2 + 3}")      # f-string
```

Parámetros útiles de `print`:

| Parámetro | Qué hace | Por defecto |
|---|---|---|
| `sep` | Separador entre valores | un espacio |
| `end` | Qué se escribe al final | salto de línea |
| `file` | Dónde escribir (por ejemplo `sys.stderr`) | pantalla |

### Pedir información: `input()`

```python
nombre = input("¿Cómo te llamas? ")
```

`input()` **siempre devuelve texto (`str`)**, aunque el usuario escriba un número. Para usarlo como número hay que convertirlo:

```python
edad = int(input("Edad: "))
altura = float(input("Altura: "))
```

### Instrucciones y líneas

- Cada instrucción va normalmente en su propia línea; no hace falta `;`.
- Una línea larga se puede partir con `\` o, mejor, dentro de paréntesis, corchetes o llaves:

```python
total = (10 + 20 +
         30 + 40)

lista = [
    "uno",
    "dos",
    "tres",
]
```

- Dos instrucciones en la misma línea se pueden separar con `;`, pero se considera mal estilo.

### Reglas de escritura

- Python distingue **mayúsculas y minúsculas**: `Nombre` y `nombre` son distintos.
- Los nombres no pueden empezar por número ni contener espacios o símbolos (salvo `_`).
- Palabras reservadas que no puedes usar como nombres: `if`, `else`, `for`, `while`, `def`, `class`, `return`, `import`, `True`, `False`, `None`, etc. Puedes verlas con:

```python
import keyword
print(keyword.kwlist)
```

## 4. Elementos y características

### Cómo ejecutar un programa

Guarda esto como `saludo.py`:

```python
nombre = input("¿Cómo te llamas? ")
print(f"Hola, {nombre}")
```

Y ejecútalo desde la terminal, en la carpeta del archivo:

```bash
python saludo.py
```

Opciones útiles:

| Comando | Qué hace |
|---|---|
| `python archivo.py` | Ejecuta el archivo |
| `python` | Abre el intérprete interactivo |
| `python -c "print(2+2)"` | Ejecuta una instrucción suelta |
| `python -m modulo` | Ejecuta un módulo como programa (por ejemplo `python -m venv .venv`) |
| `python -i archivo.py` | Ejecuta el archivo y deja el intérprete abierto |

### Intérprete interactivo (REPL)

Aparece el símbolo `>>>`. Escribes una instrucción, Python la ejecuta y muestra el resultado al momento.

```python
>>> 2 + 3
5
>>> nombre = "Ana"
>>> nombre.upper()
'ANA'
>>> exit()
```

Funciones muy útiles para explorar:

```python
help(str)      # documentación de un tipo o función
dir(str)       # lista de métodos y atributos disponibles
type(5)        # tipo de un valor
```

### Argumentos desde la terminal

```python
import sys
print(sys.argv)   # lista con el nombre del script y los argumentos
```

```bash
python programa.py hola 5
# ['programa.py', 'hola', '5']
```

Los argumentos llegan siempre como texto. Para programas más serios existe el módulo `argparse`.

### El bloque principal: `if __name__ == "__main__":`

Cada archivo `.py` tiene una variable especial `__name__`:

- Si lo ejecutas directamente, vale `"__main__"`.
- Si lo importas desde otro archivo, vale el nombre del módulo.

Por eso se escribe así el "punto de entrada":

```python
def main():
    print("Programa en marcha")

if __name__ == "__main__":
    main()
```

El código dentro del `if` **solo se ejecuta cuando lanzas ese archivo**, no cuando otro archivo lo importa (ver [[Python 13 - Módulos y paquetes]]).

### Docstrings y `help()`

Un texto entre `""" """` justo al principio de una función, clase o módulo se convierte en su documentación:

```python
def saludar(nombre):
    """Devuelve un saludo para la persona indicada."""
    return f"Hola, {nombre}"

help(saludar)
```

### Errores al ejecutar: el Traceback

Cuando algo falla, Python muestra un *Traceback*. Se lee **de abajo arriba**: la última línea indica el tipo de error y el mensaje; las anteriores muestran el camino (archivo y línea) hasta él.

```
Traceback (most recent call last):
  File "prueba.py", line 2, in <module>
    print(5 / 0)
ZeroDivisionError: division by zero
```

Más detalles en [[Python 11 - Excepciones]] y [[Python 18 - Debugging y buenas prácticas]].

## 5. Ejemplos prácticos

### Ejemplo básico

```python
print("Hola, mundo")
```

### Ejemplo habitual

```python
nombre = input("¿Cómo te llamas? ")
anio_nacimiento = int(input("¿En qué año naciste? "))

edad = 2026 - anio_nacimiento
print(f"Hola {nombre}, tienes unos {edad} años")
```

### Ejemplo completo: programa con estructura

```python
"""Calculadora sencilla de área de un rectángulo."""


def calcular_area(base, altura):
    """Devuelve el área de un rectángulo."""
    return base * altura


def main():
    base = float(input("Base: "))
    altura = float(input("Altura: "))
    print(f"El área es {calcular_area(base, altura)}")


if __name__ == "__main__":
    main()
```

## 6. Buenas prácticas

- Usa **4 espacios** de sangría y no mezcles con tabuladores.
- Nombres en `snake_case` (minúsculas con guiones bajos), claros y descriptivos.
- Comenta el **porqué**, no lo evidente.
- Divide el programa en funciones y usa `if __name__ == "__main__":`.
- Sigue la guía de estilo **PEP 8** (ver [[Python 18 - Debugging y buenas prácticas]]).
- Trabaja siempre con un **entorno virtual** por proyecto (ver [[Python 17 - Entornos y paquetes]]).
- Guarda los archivos en **UTF-8**, que es la codificación por defecto.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| Python 2 vs Python 3 | Python 2 está abandonado; hoy se usa Python 3 |
| Script vs módulo | Un script se ejecuta; un módulo se importa. Un mismo archivo puede ser ambos |
| `python` vs `python3` | Depende del sistema; comprueba con `--version` |
| Intérprete vs compilador | El intérprete ejecuta al momento; el compilador genera antes un ejecutable |
| REPL vs archivo | El REPL es para probar; los programas reales van en archivos |
| Tipado dinámico vs estático | En Python el tipo está en el valor, no en la variable (aunque se pueden añadir anotaciones) |

## 8. Casos especiales

- **Anotaciones de tipo:** puedes indicar tipos para ayudar a editores y herramientas, pero Python **no los obliga** al ejecutar:

```python
edad: int = 20
def sumar(a: int, b: int) -> int:
    return a + b
```

- **Shebang en Linux/macOS:** primera línea `#!/usr/bin/env python3` para ejecutar el archivo directamente (`./programa.py`) tras darle permisos.
- **`pass`:** instrucción que no hace nada; sirve para dejar un bloque vacío de forma válida.
- **Separador de miles:** puedes escribir `1_000_000` para leer mejor los números grandes.
- **El guion bajo `_`:** en el REPL guarda el último resultado; en bucles se usa como nombre de "variable que no me importa".
- **`IndentationError` por tabulaciones mezcladas:** suele pasar al copiar código de otros sitios.

> [!warning] Obsoleto / legado
> `print "hola"` (sin paréntesis) y `raw_input()` eran Python 2. Ahora: `print("hola")` e `input()`.

❌ Nombrar un archivo igual que un módulo estándar (`random.py`, `math.py`)
🤔 Python importa tu archivo en vez del módulo original y todo falla de forma extraña.
✅ Usa un nombre distinto y borra la carpeta `__pycache__` si hace falta.

## 9. Resumen

- Python es interpretado, de tipado dinámico y usa **indentación** para los bloques.
- Se ejecuta con `python archivo.py`; el REPL sirve para probar.
- `print()` muestra, `input()` pide (siempre devuelve texto).
- `#` para comentarios, `"""` para docstrings.
- `if __name__ == "__main__":` separa lo que se ejecuta de lo que se importa.
- El Traceback se lee de abajo arriba.
- Un entorno virtual por proyecto y estilo PEP 8 desde el principio.