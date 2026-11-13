# Ficheros

> [!info] ¿Qué es?
> Python puede **leer y escribir archivos** del disco: textos, CSV, JSON, imágenes, etc. La forma correcta de hacerlo es con `with open(...)`, que abre el archivo y lo **cierra solo** al terminar, aunque haya errores. Para trabajar con rutas, se usa el módulo moderno `pathlib`.

## 1. Antes de empezar

Conviene conocer excepciones ([[Python 11 - Excepciones]]), porque trabajar con archivos puede fallar (no existe, sin permisos, disco lleno…), y strings ([[Python 10 - Strings]]) y diccionarios ([[Python 09 - Diccionarios]]).

## 2. Concepto fundamental

Abrir un archivo devuelve un **objeto archivo** (un "manejador") con el que lees o escribes. Al terminar hay que **cerrarlo** para liberar el recurso y asegurarte de que los datos se guardan. El bloque `with` se encarga de eso automáticamente.

```python
with open("datos.txt", "r", encoding="utf-8") as f:
    contenido = f.read()
# aquí el archivo ya está cerrado
```

Hay dos tipos de archivos:

| Tipo | Se abre con | Contiene |
|---|---|---|
| **Texto** | `"r"`, `"w"`, `"a"` | Textos (`str`) |
| **Binario** | `"rb"`, `"wb"` | Bytes (imágenes, PDF, ZIP…) |

## 3. Sintaxis / estructura

```python
open(ruta, modo="r", encoding=None, newline=None)
```

### Modos de apertura

| Modo | Significado | Si no existe | Si existe |
|---|---|---|---|
| `"r"` | Leer (por defecto) | Error | Lee |
| `"w"` | Escribir | Lo crea | **Lo borra** y escribe |
| `"a"` | Añadir al final | Lo crea | Añade al final |
| `"x"` | Crear en exclusiva | Lo crea | Error |
| `"r+"` | Leer y escribir | Error | Lee/escribe sin borrar |
| `"b"` | Modo binario (se combina: `"rb"`, `"wb"`) | | |

> [!warning] Cuidado con `"w"`
> Abrir con `"w"` **vacía el archivo** al instante, aunque no escribas nada.

## 4. Elementos y características

### Leer

```python
with open("datos.txt", encoding="utf-8") as f:
    todo = f.read()            # todo el texto en un str
    f.seek(0)                  # vuelve al principio
    una = f.readline()         # una línea (incluye el \n)
    lineas = f.readlines()     # lista con todas las líneas
```

La forma más eficiente de leer un archivo grande es **recorrerlo línea a línea**:

```python
with open("log.txt", encoding="utf-8") as f:
    for linea in f:
        print(linea.rstrip("\n"))
```

### Escribir

```python
with open("salida.txt", "w", encoding="utf-8") as f:
    f.write("Primera línea\n")           # no añade \n automáticamente
    f.writelines(["a\n", "b\n"])
    print("Otra línea", file=f)          # print también escribe en archivos
```

### Añadir sin borrar

```python
with open("registro.txt", "a", encoding="utf-8") as f:
    f.write("Nueva entrada\n")
```

### Archivos binarios

```python
with open("imagen.png", "rb") as origen, open("copia.png", "wb") as destino:
    destino.write(origen.read())
```

### Rutas con `pathlib` (la forma moderna)

```python
from pathlib import Path

ruta = Path("carpeta") / "archivo.txt"      # une rutas con /

ruta.exists()                 # ¿existe?
ruta.is_file(), ruta.is_dir()
ruta.name                     # "archivo.txt"
ruta.stem                     # "archivo"
ruta.suffix                   # ".txt"
ruta.parent                   # Path("carpeta")
ruta.resolve()                # ruta absoluta

ruta.read_text(encoding="utf-8")
ruta.write_text("hola", encoding="utf-8")
ruta.read_bytes()

Path("nueva/subcarpeta").mkdir(parents=True, exist_ok=True)
ruta.unlink()                 # borrar archivo
Path.home()                   # carpeta del usuario
Path.cwd()                    # carpeta actual

for archivo in Path(".").glob("*.py"):        # buscar archivos
    print(archivo)
for archivo in Path(".").rglob("*.md"):       # buscar en subcarpetas
    print(archivo)
```

### Ruta relativa al propio script

Las rutas relativas dependen de **desde dónde ejecutas** el programa. Para ser independiente de eso:

```python
base = Path(__file__).parent
datos = base / "datos" / "info.json"
```

### Copiar, mover y borrar carpetas (`shutil` y `os`)

```python
import shutil, os

shutil.copy("a.txt", "b.txt")
shutil.move("a.txt", "carpeta/a.txt")
shutil.copytree("origen", "destino")
shutil.rmtree("carpeta")               # borra una carpeta entera (¡cuidado!)

os.listdir(".")                        # lista de nombres
os.rename("viejo.txt", "nuevo.txt")
os.getenv("HOME")                      # variable de entorno
```

### JSON

```python
import json

datos = {"nombre": "Ana", "edad": 20}

with open("datos.json", "w", encoding="utf-8") as f:
    json.dump(datos, f, indent=2, ensure_ascii=False)

with open("datos.json", encoding="utf-8") as f:
    leidos = json.load(f)

texto = json.dumps(datos)        # dict -> str
objeto = json.loads(texto)       # str -> dict
```

- `ensure_ascii=False` guarda tildes y ñ tal cual.
- `indent=2` lo deja legible.
- JSON solo admite tipos básicos (texto, números, listas, diccionarios, `True`/`False`/`None`).

### CSV

```python
import csv

# Leer como diccionarios (usa la primera fila como nombres de columna)
with open("tabla.csv", newline="", encoding="utf-8") as f:
    for fila in csv.DictReader(f):
        print(fila["nombre"], fila["edad"])

# Escribir
with open("salida.csv", "w", newline="", encoding="utf-8") as f:
    escritor = csv.DictWriter(f, fieldnames=["nombre", "edad"])
    escritor.writeheader()
    escritor.writerow({"nombre": "Ana", "edad": 20})
```

`newline=""` evita líneas en blanco extra en Windows. Todo lo leído de un CSV llega como **texto**.

### Archivos temporales

```python
import tempfile

with tempfile.TemporaryDirectory() as carpeta:
    ...        # la carpeta se borra al salir
```

### Otros formatos

| Formato | Módulo |
|---|---|
| Binario de Python | `pickle` (no abrir archivos de origen desconocido: puede ejecutar código) |
| ZIP | `zipfile` |
| YAML / TOML | `PyYAML`, `tomllib` (3.11+) |
| Excel | `openpyxl`, `pandas` |

## 5. Ejemplos prácticos

### Ejemplo básico

```python
with open("notas.txt", "w", encoding="utf-8") as f:
    f.write("Hola, archivo\n")

with open("notas.txt", encoding="utf-8") as f:
    print(f.read())
```

### Ejemplo habitual: leer con manejo de errores

```python
from pathlib import Path

def leer_texto(ruta):
    try:
        return Path(ruta).read_text(encoding="utf-8")
    except FileNotFoundError:
        print(f"No existe: {ruta}")
    except PermissionError:
        print(f"Sin permisos: {ruta}")
    return None
```

### Ejemplo completo: guardar y cargar datos de una aplicación

```python
import json
from pathlib import Path

ARCHIVO = Path("tareas.json")

def cargar_tareas():
    if not ARCHIVO.exists():
        return []
    with ARCHIVO.open(encoding="utf-8") as f:
        return json.load(f)

def guardar_tareas(tareas):
    with ARCHIVO.open("w", encoding="utf-8") as f:
        json.dump(tareas, f, indent=2, ensure_ascii=False)

tareas = cargar_tareas()
tareas.append({"titulo": "Estudiar Python", "hecha": False})
guardar_tareas(tareas)
```

## 6. Buenas prácticas

- **Usa siempre `with`.**
- **Indica `encoding="utf-8"`** al abrir archivos de texto, para evitar problemas con tildes entre sistemas.
- Usa **`pathlib`** en lugar de concatenar rutas con `+` o `\`.
- Comprueba errores con `try/except` (`FileNotFoundError`, `PermissionError`).
- Para archivos grandes, **recorre línea a línea** en vez de usar `read()`.
- Usa `"x"` o comprueba `exists()` si no quieres sobrescribir.
- No construyas rutas con datos del usuario sin validar (riesgo de acceder a carpetas no deseadas).
- Para escribir de forma segura, guarda primero en un archivo temporal y luego renómbralo, así no corrompes el original si el programa falla a medias.
- Usa `json` para guardar estructuras de datos y `csv` para tablas; no inventes tu propio formato.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| `"w"` vs `"a"` | `"w"` sobrescribe; `"a"` añade al final |
| `read()` vs `readline()` vs `readlines()` | Todo en un texto / una línea / lista de líneas |
| `json.load` vs `json.loads` | `load` lee de un archivo; `loads` de un texto |
| `json.dump` vs `json.dumps` | `dump` escribe a un archivo; `dumps` devuelve un texto |
| Texto vs binario | Texto se codifica y decodifica; binario son bytes crudos |
| `os.path` vs `pathlib` | `pathlib` es más moderno y legible |
| Ruta absoluta vs relativa | La relativa depende del directorio desde el que ejecutas |
| `close()` manual vs `with` | `with` cierra siempre, incluso con errores |

## 8. Casos especiales

- **Rutas en Windows:** usa `/` o `Path`; si escribes `\`, usa una cadena cruda (`r"C:\datos"`) o dobla la barra (`"C:\\datos"`).
- **`write()` solo admite texto** (en modo texto): convierte los números con `str()` o f-strings.
- **Se pierde el contenido con `"w"`** si el programa falla a mitad; considera el truco del archivo temporal.
- **Posición del cursor:** después de leer todo, un segundo `read()` devuelve `""`. Usa `seek(0)` para volver al principio.
- **Saltos de línea:** `readlines()` y las líneas del bucle incluyen `\n`; usa `rstrip("\n")` o `strip()`.
- **Codificación incorrecta:** un `UnicodeDecodeError` indica que el archivo no está en la codificación esperada (prueba `encoding="latin-1"` o `errors="replace"`).
- **Archivos muy grandes en JSON:** `json.load` carga todo en memoria; para volúmenes enormes usa formatos de líneas (JSONL) o bases de datos.
- **Dos procesos escribiendo a la vez** en el mismo archivo pueden corromperlo; necesitas bloqueos o una base de datos.
- **Binario con `pickle`:** nunca cargues un `.pkl` que no sea tuyo.
- **Carpetas:** `open()` no crea carpetas; hay que crearlas antes con `mkdir(parents=True, exist_ok=True)`.

❌ `f = open("datos.txt")` y olvidar cerrarlo
🤔 Puede quedar abierto, bloquear el archivo o perder datos no guardados.
✅ `with open("datos.txt", encoding="utf-8") as f:`

❌ Abrir con `"w"` para "añadir una línea"
🤔 `"w"` borra todo lo anterior.
✅ Usa `"a"`.

## 9. Resumen

- `with open(ruta, modo, encoding="utf-8") as f:` es la forma estándar y segura.
- Modos: `r` leer, `w` escribir (borra), `a` añadir, `x` crear nuevo; `b` para binario.
- Leer: `read()`, `readline()`, `readlines()`, o recorrer el archivo línea a línea.
- Escribir: `write()` (sin salto de línea automático), `writelines()`, `print(..., file=f)`.
- `pathlib.Path` para rutas, comprobar existencia, leer y escribir de forma sencilla.
- `json` para estructuras de datos, `csv` para tablas, `shutil` para copiar y mover.
- Maneja `FileNotFoundError` y `PermissionError`.
- Las rutas relativas dependen del directorio de ejecución; usa `Path(__file__).parent` si hace falta.