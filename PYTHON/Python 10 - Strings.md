# Strings

> [!info] ¿Qué es?
> Un **string** (`str`) es una secuencia de caracteres, es decir, un **texto**. Es **inmutable**: no se puede modificar, cada operación crea un texto nuevo. Python guarda los textos en **Unicode**, así que admite tildes, ñ, símbolos y emojis sin problema.

## 1. Antes de empezar

Los textos se comportan como una secuencia, igual que las listas: tienen índices y slicing ([[Python 07 - Listas]]). Conviene conocer variables y tipos ([[Python 02 - Variables y tipos]]).

## 2. Concepto fundamental

- Un texto es una **secuencia inmutable** de caracteres.
- Cada carácter tiene una posición (índice desde 0, y negativos desde el final).
- Como son inmutables, métodos como `upper()` o `replace()` **devuelven un texto nuevo** y no modifican el original.

```python
nombre = "ana"
nombre.upper()     # "ANA"
print(nombre)      # "ana"  <- el original no cambia
```

## 3. Sintaxis / estructura

### Crear textos

```python
a = "comillas dobles"
b = 'comillas simples'
c = """texto
de varias
líneas"""
d = "Dijo: \"hola\""          # comillas escapadas
e = 'Dijo: "hola"'            # alternativa más limpia
```

### Caracteres de escape

| Secuencia | Significado |
|---|---|
| `\n` | Salto de línea |
| `\t` | Tabulador |
| `\\` | Barra invertida |
| `\"` / `\'` | Comillas |
| `\u00e9` | Carácter Unicode (é) |

### Cadenas crudas (*raw*)

Con `r` delante, las barras no se interpretan. Útil para rutas de Windows y expresiones regulares.

```python
ruta = r"C:\nueva\carpeta"
```

### Concatenar y repetir

```python
"Hola, " + "mundo"      # "Hola, mundo"
"ab" * 3                # "ababab"
```

### Índices y slicing

```python
t = "Python"
t[0]         # "P"
t[-1]        # "n"
t[1:4]       # "yth"
t[:2]        # "Py"
t[2:]        # "thon"
t[::2]       # "Pto"
t[::-1]      # "nohtyP"
len(t)       # 6
```

## 4. Elementos y características

### f-strings: la forma recomendada de formatear

```python
nombre, precio = "Ana", 3.14159

f"Hola {nombre}"                  # inserta variables
f"Total: {precio:.2f}"            # 2 decimales -> 3.14
f"{1234567:,}"                    # 1,234,567 (separador de miles)
f"{0.256:.1%}"                    # 25.6%
f"{5:03d}"                        # 005 (rellena con ceros)
f"{nombre:>10}"                   # alinea a la derecha en 10 espacios
f"{nombre:<10}|"                  # a la izquierda
f"{nombre:^10}"                   # centrado
f"{2 + 3}"                        # expresiones: 5
f"{nombre=}"                      # nombre='Ana'  (útil para depurar)
```

### Métodos de transformación

| Método | Qué hace |
|---|---|
| `upper()` / `lower()` | Mayúsculas / minúsculas |
| `capitalize()` | Solo la primera letra en mayúscula |
| `title()` | Cada palabra con mayúscula inicial |
| `swapcase()` | Invierte mayúsculas y minúsculas |
| `casefold()` | Minúsculas agresivas, mejor para comparar |
| `strip()` | Quita espacios (y saltos) al principio y al final |
| `lstrip()` / `rstrip()` | Solo por la izquierda / derecha |
| `replace(a, b)` | Sustituye `a` por `b` |
| `center(n)`, `ljust(n)`, `rjust(n)` | Rellena hasta `n` caracteres |
| `zfill(n)` | Rellena con ceros a la izquierda |

### Métodos de búsqueda y comprobación

| Método | Qué hace |
|---|---|
| `find(x)` | Posición de `x` (o `-1` si no está) |
| `index(x)` | Posición de `x` (error si no está) |
| `count(x)` | Cuántas veces aparece |
| `startswith(x)` / `endswith(x)` | Comprueba el principio / el final |
| `x in texto` | ¿Contiene el subtexto? |
| `isdigit()` | ¿Solo dígitos? |
| `isalpha()` | ¿Solo letras? |
| `isalnum()` | ¿Solo letras y números? |
| `isspace()` | ¿Solo espacios? |
| `isupper()` / `islower()` | ¿Todo mayúsculas / minúsculas? |

### `split()` y `join()`: texto ↔ lista

```python
"a,b,c".split(",")           # ["a", "b", "c"]
"hola  mundo".split()        # ["hola", "mundo"] (separa por cualquier espacio)
"a,b,c".split(",", 1)        # ["a", "b,c"] (máximo una división)
"línea1\nlínea2".splitlines()   # separa por saltos de línea

"-".join(["2026", "10", "10"])   # "2026-10-10"
" ".join(["hola", "mundo"])      # "hola mundo"
```

`join` se llama sobre el **separador** y recibe una lista de textos.

### Otras formas de formatear (más antiguas)

```python
"Hola {}".format("Ana")
"Hola %s" % "Ana"
```

Siguen funcionando, pero se prefieren los f-strings.

### Caracteres y códigos

```python
ord("A")        # 65  (código Unicode)
chr(65)         # "A"
```

### Codificación (texto ↔ bytes)

```python
b = "añb".encode("utf-8")     # texto -> bytes
b.decode("utf-8")             # bytes -> texto
```

Al leer y escribir archivos conviene indicar siempre `encoding="utf-8"` ([[Python 12 - Ficheros]]).

### Recorrer un texto

```python
for letra in "hola":
    print(letra)
```

### Expresiones regulares

Para buscar patrones complejos (emails, teléfonos, fechas) se usa el módulo `re` (ver [[Conceptos 04 - Regex]]):

```python
import re
re.findall(r"\d+", "tengo 3 gatos y 12 peces")    # ["3", "12"]
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
nombre = "  ana lópez  "
print(nombre.strip().title())      # "Ana López"
```

### Ejemplo habitual: limpiar y validar una entrada

```python
email = input("Email: ").strip().lower()

if "@" in email and email.endswith(".com"):
    usuario, dominio = email.split("@")
    print(f"Usuario: {usuario}, dominio: {dominio}")
else:
    print("Email no válido")
```

### Ejemplo completo: contar palabras y detectar palíndromos

```python
def contar_palabras(texto):
    frecuencias = {}
    for palabra in texto.lower().split():
        palabra = palabra.strip(".,;:!?¿¡")
        frecuencias[palabra] = frecuencias.get(palabra, 0) + 1
    return frecuencias

def es_palindromo(texto):
    limpio = "".join(c.lower() for c in texto if c.isalnum())
    return limpio == limpio[::-1]

es_palindromo("Anita lava la tina")    # True
```

## 6. Buenas prácticas

- Usa **f-strings** para mezclar texto y variables.
- Para unir muchos textos, usa `"".join(lista)` en vez de `+=` en un bucle.
- Limpia las entradas con `strip()`, y normaliza con `lower()` o `casefold()` antes de comparar.
- Usa `in` para comprobar si contiene algo, no `find() != -1`.
- Usa `startswith()` / `endswith()` en lugar de cortar con slicing a mano.
- Mantén una **única convención de comillas** en el proyecto (la mayoría de guías usan dobles o simples de forma consistente).
- Indica siempre `encoding="utf-8"` al abrir archivos de texto.
- Para patrones complejos usa regex, no un montón de `if` con `find`.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `find()` vs `index()` | `find` devuelve `-1` si no existe; `index` lanza `ValueError` |
| `split()` vs `split(" ")` | Sin argumentos agrupa cualquier espacio en blanco; con `" "` genera elementos vacíos si hay espacios repetidos |
| `split` vs `join` | `split` convierte texto en lista; `join` convierte lista en texto |
| `strip()` vs `replace(" ", "")` | `strip` quita solo los extremos; `replace` quita todos |
| `isdigit()` vs `isnumeric()` vs `isdecimal()` | Distintos niveles de amplitud con caracteres Unicode; para un número normal sirven igual |
| `lower()` vs `casefold()` | `casefold` es más completo para comparar entre idiomas (por ejemplo `"ß"`) |
| `str` vs `bytes` | `str` es texto; `bytes` son datos crudos |
| `repr()` vs `str()` | `str` es para humanos; `repr` es para depuración e incluye comillas y escapes |

## 8. Casos especiales

- **Inmutabilidad:** no se puede cambiar un carácter.

❌ `texto[0] = "X"`
🤔 Los textos son inmutables.
✅ `texto = "X" + texto[1:]`, o convierte a lista (`list(texto)`), modifica y vuelve a unir con `"".join(...)`.

- **`+=` en bucles:** funciona, pero con textos grandes es lento (crea un texto nuevo cada vez); usa `join`.
- **Comparar textos:** distingue mayúsculas (`"a" != "A"`) y compara por código Unicode, así que `"Z" < "a"` es `True`.
- **Tildes y normalización:** `"é"` puede escribirse como un carácter o como `e` + acento combinado, y entonces `==` da `False`. Se arregla con `unicodedata.normalize("NFC", texto)`.
- **Emojis y `len()`:** un emoji suele contar como 1 carácter, pero algunos (con modificadores, banderas) son varios.
- **Números y textos no se mezclan:**

❌ `"Edad: " + 20`
🤔 No se pueden sumar `str` e `int`.
✅ `f"Edad: {20}"` o `"Edad: " + str(20)`.

- **Textos con comillas dentro:** alterna comillas (`'dijo "hola"'`) o escápalas.
- **Strings de varias líneas con `"""`** conservan saltos y sangrías; para quitarlas, usa `textwrap.dedent()`.
- **Multiplicar por número negativo o cero** da texto vacío: `"ab" * 0 == ""`.
- **`"a" in "casa"`** busca subtextos, no elementos; `"ca" in "casa"` es `True`.
- **Slicing fuera de rango no falla** (`"hola"[10:]` da `""`), pero un índice suelto sí (`"hola"[10]` lanza `IndexError`).

## 9. Resumen

- `str` = secuencia **inmutable** de caracteres Unicode; índices, negativos y slicing como las listas.
- Comillas simples, dobles o triples; `\n`, `\t`; prefijo `r` para cadenas crudas.
- **f-strings** para formatear: `f"{valor:.2f}"`.
- Métodos clave: `strip`, `lower`/`upper`, `replace`, `split`, `join`, `find`, `startswith`/`endswith`, `isdigit`.
- Los métodos devuelven textos nuevos; el original no cambia.
- `"".join(lista)` para construir textos grandes; `split()` para dividir.
- Usa `encoding="utf-8"` en archivos y regex (`re`) para patrones complejos.