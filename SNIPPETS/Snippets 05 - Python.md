# 05 - Snippets de Python

> [!info] ¿Qué es?
> Fragmentos de **Python** listos para copiar: archivos, JSON, CSV, peticiones HTTP, fechas y utilidades de colecciones. Las explicaciones están en la carpeta [[Python 00 - Índice|Python]].

---

## 1. Archivos y rutas

### 1.1 Leer y escribir texto

```python
from pathlib import Path

ruta = Path("notas.txt")
texto = ruta.read_text(encoding="utf-8")
ruta.write_text("Nuevo contenido", encoding="utf-8")

with ruta.open("a", encoding="utf-8") as f:
    f.write("Línea añadida\n")
```

### 1.2 Ruta relativa al propio script

```python
from pathlib import Path

CARPETA = Path(__file__).parent
datos = CARPETA / "datos" / "usuarios.json"
```

### 1.3 Recorrer archivos de una carpeta

```python
for archivo in Path("notas").rglob("*.md"):
    print(archivo.name, archivo.stat().st_size)
```

### 1.4 Crear carpetas si no existen

```python
Path("salida/informes").mkdir(parents=True, exist_ok=True)
```

Más en [[Python 12 - Ficheros|Ficheros]].

---

## 2. JSON y CSV

### 2.1 Leer y guardar JSON

```python
import json
from pathlib import Path

datos = json.loads(Path("datos.json").read_text(encoding="utf-8"))

Path("salida.json").write_text(
    json.dumps(datos, ensure_ascii=False, indent=2), encoding="utf-8"
)
```

### 2.2 Leer un CSV como diccionarios

```python
import csv

with open("productos.csv", newline="", encoding="utf-8") as f:
    productos = list(csv.DictReader(f))

print(productos[0]["nombre"])
```

### 2.3 Escribir un CSV

```python
import csv

with open("informe.csv", "w", newline="", encoding="utf-8") as f:
    escritor = csv.DictWriter(f, fieldnames=["nombre", "precio"])
    escritor.writeheader()
    escritor.writerows(productos)
```

(Para abrirlo bien en Excel con tildes, usa `encoding="utf-8-sig"`.)

---

## 3. Peticiones HTTP

### 3.1 GET a una API (librería `requests`)

```python
import requests

res = requests.get("https://api.ejemplo.com/productos", params={"pagina": 1}, timeout=10)
res.raise_for_status()          # lanza error si es 4xx o 5xx
productos = res.json()
```

### 3.2 POST con JSON y token

```python
res = requests.post(
    "https://api.ejemplo.com/productos",
    json={"nombre": "Taza", "precio": 8.5},
    headers={"Authorization": f"Bearer {TOKEN}"},
    timeout=10,
)
```

### 3.3 Sin librerías externas

```python
import json
from urllib.request import urlopen

with urlopen("https://api.ejemplo.com/datos", timeout=10) as res:
    datos = json.load(res)
```

---

## 4. Fechas

### 4.1 Hoy, ahora y formato

```python
from datetime import date, datetime

hoy = date.today().isoformat()                         # "2026-11-13"
ahora = datetime.now().strftime("%Y-%m-%d %H:%M")      # "2026-11-13 10:30"
```

### 4.2 Convertir texto en fecha

```python
fecha = datetime.strptime("13/11/2026", "%d/%m/%Y")
fecha_iso = datetime.fromisoformat("2026-11-13T10:30:00")
```

### 4.3 Sumar y restar tiempo

```python
from datetime import timedelta

dentro_de_una_semana = date.today() + timedelta(days=7)
dias_hasta = (date(2026, 12, 25) - date.today()).days
```

### 4.4 Fecha con zona horaria

```python
from datetime import datetime
from zoneinfo import ZoneInfo

madrid = datetime.now(ZoneInfo("Europe/Madrid"))
```

---

## 5. Colecciones

### 5.1 Filtrar, transformar y sumar

```python
baratos = [p for p in productos if p["precio"] < 20]
nombres = [p["nombre"] for p in productos]
total = sum(p["precio"] * p["cantidad"] for p in carrito)
por_id = {p["id"]: p for p in productos}
```

### 5.2 Ordenar por una clave

```python
ordenados = sorted(productos, key=lambda p: p["precio"], reverse=True)
```

### 5.3 Contar apariciones

```python
from collections import Counter

palabras = Counter(texto.lower().split())
print(palabras.most_common(5))
```

### 5.4 Agrupar por una propiedad

```python
from collections import defaultdict

por_categoria = defaultdict(list)
for p in productos:
    por_categoria[p["categoria"]].append(p)
```

### 5.5 Quitar duplicados manteniendo el orden

```python
unicos = list(dict.fromkeys(lista))
```

### 5.6 Dividir una lista en trozos

```python
from itertools import batched   # Python 3.12+

for lote in batched(registros, 100):
    procesar(lote)
```

Más en [[Python 07 - Listas|Listas]] y [[Python 09 - Diccionarios|Diccionarios]].

---

## 6. Scripts

### 6.1 Estructura de un script

```python
def main() -> None:
    ...

if __name__ == "__main__":
    main()
```

### 6.2 Argumentos de línea de comandos

```python
import argparse

parser = argparse.ArgumentParser(description="Procesa un archivo")
parser.add_argument("archivo")
parser.add_argument("--limite", type=int, default=10)
args = parser.parse_args()

print(args.archivo, args.limite)
```

```bash
python script.py datos.csv --limite 5
```

### 6.3 Variables de entorno con `.env`

```python
import os
from dotenv import load_dotenv   # pip install python-dotenv

load_dotenv()
API_KEY = os.environ["API_KEY"]               # error si falta
DEBUG = os.getenv("DEBUG", "false") == "true"
```

### 6.4 Medir cuánto tarda algo

```python
import time

inicio = time.perf_counter()
proceso()
print(f"Tardó {time.perf_counter() - inicio:.2f} s")
```

### 6.5 Registrar mensajes (logging)

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logging.info("Proceso iniciado")
logging.error("Algo falló")
```

---

## 7. Texto

### 7.1 Formatear números

```python
f"{1234.5:,.2f}"                       # '1,234.50'
f"{1234.5:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")  # '1.234,50'
f"{0.256:.1%}"                         # '25.6%'
```

### 7.2 Quitar tildes

```python
import unicodedata

def sin_tildes(texto: str) -> str:
    return "".join(
        c for c in unicodedata.normalize("NFD", texto) if unicodedata.category(c) != "Mn"
    )
```

### 7.3 Crear un slug

```python
import re

def slug(texto: str) -> str:
    return re.sub(r"[^a-z0-9]+", "-", sin_tildes(texto).lower()).strip("-")
```

### 7.4 Validar un email (básico)

```python
import re

def es_email(texto: str) -> bool:
    return re.fullmatch(r"[^\s@]+@[^\s@]+\.[^\s@]+", texto) is not None
```

Más en [[Python 10 - Strings|Strings]] y [[Conceptos 04 - Regex]].
