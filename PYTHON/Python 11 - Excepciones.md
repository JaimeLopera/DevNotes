# Excepciones

> [!info] ¿Qué es?
> Una **excepción** es un error que ocurre **mientras el programa se ejecuta** (dividir entre cero, abrir un archivo que no existe, convertir "abc" a número…). Si no se controla, el programa se detiene y muestra un *Traceback*. Con `try` / `except` puedes **capturar** el error y decidir qué hacer en lugar de que el programa se caiga.

## 1. Antes de empezar

Necesitas conocer funciones y condicionales ([[Python 06 - Funciones]], [[Python 04 - Condicionales]]). Conviene saber leer un *Traceback* (se lee de **abajo arriba**: la última línea dice el tipo de error y el mensaje).

## 2. Concepto fundamental

Cuando algo falla, Python **lanza** (*raise*) una excepción, que "sube" por las funciones que estaban llamándose hasta que alguien la captura. Si nadie lo hace, el programa termina.

```
función C falla  ->  sube a función B  ->  sube a función A  ->  sube al programa -> se detiene
        (si alguien la captura con try/except, el viaje se detiene ahí)
```

Hay tres tipos de errores:

| Tipo | Cuándo aparece | Ejemplo |
|---|---|---|
| Sintaxis | Antes de ejecutar | Falta un `:` |
| Excepción (ejecución) | Mientras corre | `ZeroDivisionError` |
| Lógico | Nunca avisa | El resultado es incorrecto |

Esta nota trata las **excepciones**.

## 3. Sintaxis / estructura

```python
try:
    # código que puede fallar
except TipoError as e:
    # qué hacer si falla
else:
    # se ejecuta solo si NO hubo error
finally:
    # se ejecuta SIEMPRE, haya error o no
```

Reglas:

- `try` es obligatorio y necesita al menos un `except` o un `finally`.
- `else` y `finally` son opcionales.
- Puede haber **varios** `except`; se prueban en orden y se ejecuta el primero que coincide.

## 4. Elementos y características

### Excepciones más comunes

| Excepción | Cuándo ocurre |
|---|---|
| `ValueError` | Valor de tipo correcto pero inválido (`int("abc")`) |
| `TypeError` | Operación con un tipo incorrecto (`"a" + 1`) |
| `ZeroDivisionError` | División entre cero |
| `IndexError` | Índice fuera de rango |
| `KeyError` | Clave inexistente en un diccionario |
| `AttributeError` | Atributo o método que no existe |
| `NameError` | Variable no definida |
| `FileNotFoundError` | Archivo o carpeta no encontrados |
| `PermissionError` | Sin permisos |
| `ImportError` / `ModuleNotFoundError` | No se puede importar un módulo |
| `StopIteration` | No quedan más elementos en un iterador |
| `KeyboardInterrupt` | El usuario pulsó `Ctrl + C` |
| `RecursionError` | Demasiadas llamadas recursivas |

### Jerarquía (simplificada)

```
BaseException
├── KeyboardInterrupt
├── SystemExit
└── Exception
    ├── ArithmeticError  (ZeroDivisionError, OverflowError…)
    ├── LookupError      (IndexError, KeyError)
    ├── OSError          (FileNotFoundError, PermissionError…)
    ├── ValueError
    ├── TypeError
    └── …
```

Capturar una excepción **padre** captura también a sus hijas (`except LookupError` captura `KeyError` e `IndexError`).

### Capturar varias excepciones

```python
try:
    ...
except ValueError:
    ...
except (TypeError, KeyError):      # varias en un mismo bloque
    ...
except Exception as e:             # cualquier otra (al final)
    print(f"Error inesperado: {e}")
```

El objeto `e` contiene información: `str(e)` da el mensaje, `type(e).__name__` el nombre del tipo.

### `else` y `finally`

```python
try:
    archivo = open("datos.txt", encoding="utf-8")
except FileNotFoundError:
    print("No existe el archivo")
else:
    print("Se abrió correctamente")      # solo si no hubo error
    archivo.close()
finally:
    print("Esto se ejecuta siempre")
```

`finally` sirve para **liberar recursos** (cerrar archivos, conexiones). Hoy casi siempre se sustituye por `with` ([[Python 12 - Ficheros]]).

### Lanzar excepciones: `raise`

```python
def dividir(a, b):
    if b == 0:
        raise ValueError("El divisor no puede ser 0")
    return a / b
```

- `raise` a secas, dentro de un `except`, **vuelve a lanzar** la excepción actual.
- `raise NuevoError("...") from e` encadena la causa original.

### Excepciones propias

Creas una clase que hereda de `Exception`:

```python
class SaldoInsuficiente(Exception):
    def __init__(self, saldo, importe):
        super().__init__(f"Saldo {saldo} insuficiente para retirar {importe}")
        self.saldo = saldo
        self.importe = importe
```

Útil para que los errores de tu programa tengan nombre y significado propio ([[Python 14 - POO]]).

### `assert`

Comprueba una suposición durante el desarrollo:

```python
assert edad >= 0, "La edad no puede ser negativa"
```

Si es falsa, lanza `AssertionError`. No lo uses para validar datos del usuario (se puede desactivar al ejecutar con `-O`).

### Grupos de excepciones (Python 3.11+)

```python
try:
    ...
except* ValueError as grupo:
    ...
```

Sirven para manejar varios errores a la vez en código concurrente. Es un tema avanzado.

### Notas añadidas (3.11+)

`e.add_note("contexto extra")` añade texto al mensaje de la excepción.

## 5. Ejemplos prácticos

### Ejemplo básico

```python
try:
    numero = int(input("Número: "))
    print(10 / numero)
except ValueError:
    print("Eso no es un número")
except ZeroDivisionError:
    print("No se puede dividir entre cero")
```

### Ejemplo habitual: pedir un dato hasta que sea válido

```python
def pedir_entero(mensaje):
    while True:
        try:
            return int(input(mensaje))
        except ValueError:
            print("Inválido, inténtalo de nuevo")
```

### Ejemplo completo: operación con excepciones propias

```python
class SaldoInsuficiente(Exception):
    pass

class Cuenta:
    def __init__(self, saldo=0):
        self.saldo = saldo

    def retirar(self, importe):
        if importe <= 0:
            raise ValueError("El importe debe ser positivo")
        if importe > self.saldo:
            raise SaldoInsuficiente(f"Solo tienes {self.saldo}")
        self.saldo -= importe

cuenta = Cuenta(100)
try:
    cuenta.retirar(150)
except SaldoInsuficiente as e:
    print(f"No se pudo: {e}")
except ValueError as e:
    print(f"Dato inválido: {e}")
```

## 6. Buenas prácticas

- **Captura excepciones concretas**, no todas.
- Mantén el `try` lo **más pequeño posible**: solo la línea que puede fallar.
- **No silencies errores**: al menos regístralos (`logging`, ver [[Python 18 - Debugging y buenas prácticas]]).
- Usa `with` para recursos en vez de `try/finally` manual.
- Lanza excepciones con **mensajes claros** que digan qué pasó y con qué valor.
- Usa tus propias excepciones para errores del dominio de tu programa.
- Sigue la filosofía de Python **EAFP** ("más fácil pedir perdón que permiso"): intenta y captura el error, en vez de comprobar antes cada condición posible.

```python
# EAFP (pythónico)
try:
    valor = d["clave"]
except KeyError:
    valor = None

# LBYL ("mira antes de saltar")
if "clave" in d:
    valor = d["clave"]
```

Ambos valen; elige el que quede más claro.

- No uses excepciones como control de flujo normal en bucles muy calientes (son más lentas que un `if`).

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `else` vs `finally` | `else` solo si no hubo error; `finally` siempre |
| `except Exception` vs `except:` | `except:` a secas captura incluso `KeyboardInterrupt` y `SystemExit` (evítalo) |
| `raise` vs `raise e` | `raise` a secas conserva el *Traceback* original |
| `assert` vs `raise` | `assert` es para desarrollo; `raise` para errores reales |
| EAFP vs LBYL | Intentar y capturar vs comprobar antes |
| Error vs excepción | Un error de sintaxis impide ejecutar; una excepción ocurre durante la ejecución |
| `try/finally` vs `with` | `with` cierra el recurso automáticamente de forma más limpia |

## 8. Casos especiales

- **El orden de los `except` importa:** si pones primero `Exception`, los específicos nunca se alcanzan.

❌
```python
try: ...
except Exception: ...
except ValueError: ...    # nunca se ejecuta
```
🤔 `Exception` ya captura todo, incluido `ValueError`.
✅ Pon siempre lo **específico antes que lo general**.

- **`return` dentro de `try` y `finally`:** el `finally` se ejecuta igualmente antes de salir de la función; si el `finally` tiene su propio `return`, anula al del `try` (evítalo).
- **Una excepción dentro de un `except`** puede ocultar la original; usa `raise ... from e` para encadenar.
- **`except Exception as e` borra `e`** al salir del bloque `except`; guárdalo en otra variable si lo necesitas después.
- **Excepciones en bucles:** un `try` dentro del bucle permite seguir con el siguiente elemento; uno fuera detiene todo.
- **`StopIteration` dentro de generators** se trata de forma especial (ver [[Python 15 - Iteradores y generators]]).
- **Silenciar con contexto:** `contextlib.suppress(FileNotFoundError)` es una forma limpia de ignorar un error concreto a propósito.

```python
from contextlib import suppress
with suppress(FileNotFoundError):
    os.remove("temporal.txt")
```

❌ `except: pass`
🤔 Se traga cualquier error y luego es imposible averiguar qué falló.
✅ Captura un tipo concreto y registra o muestra el mensaje.

## 9. Resumen

- Una excepción es un error en ejecución; sin control, detiene el programa.
- `try` / `except` / `else` / `finally`: intenta, captura, "si todo fue bien", "siempre".
- Captura **tipos concretos**; lo específico antes que lo general.
- `raise` lanza excepciones; crea las tuyas heredando de `Exception`.
- Nunca silencies errores sin motivo; regístralos.
- EAFP: intenta y captura, en vez de comprobar todo antes.
- `with` para recursos; `assert` solo para desarrollo.