# 04 - Tests en Python con pytest

> [!info] ¿Qué es?
> **pytest** es la herramienta estándar para testear código Python. Busca funciones que empiezan por `test_`, las ejecuta y usa la palabra clave `assert` de Python para comprobar resultados. Es sencillo para empezar y muy potente con sus **fixtures** y **parametrización**.

---

## 1. Antes de empezar

- Conviene haber leído [[Testing 01 - Fundamentos]].
- Hay que saber Python: [[Python 00 - Índice|Python]], y trabajar con entornos virtuales: [[Python 17 - Entornos y paquetes]].

---

## 2. Concepto fundamental

```
proyecto/
├── src/
│   └── precios.py
└── tests/
    └── test_precios.py    ← archivos test_*.py
```

pytest:
1. Busca archivos `test_*.py` o `*_test.py`.
2. Dentro, ejecuta las funciones `test_*` (y los métodos de clases `Test*`).
3. Cada `assert` que sea falso hace fallar el test, mostrando los valores comparados.

---

## 3. Sintaxis / estructura

```bash
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install pytest
```

```python
# tests/test_precios.py
from src.precios import calcular_iva

def test_calcula_el_iva_general():
    assert calcular_iva(100) == 21
```

```bash
pytest           # ejecuta todos
pytest -v        # detallado
```

---

## 4. Elementos / propiedades / características

### 4.1 Aserciones

pytest usa el `assert` normal de Python:

| Comprobación | Ejemplo |
|---|---|
| Igualdad | `assert resultado == 5` |
| Verdadero / falso | `assert usuario.activo` / `assert not lista` |
| Contiene | `assert "a" in texto` |
| Tipo | `assert isinstance(x, dict)` |
| Aproximado (decimales) | `assert 0.1 + 0.2 == pytest.approx(0.3)` |
| Lanza error | `with pytest.raises(ValueError): f()` |

```python
import pytest

def test_divide_entre_cero():
    with pytest.raises(ZeroDivisionError):
        1 / 0

def test_mensaje_de_error():
    with pytest.raises(ValueError, match="negativo"):
        calcular_iva(-5)
```

### 4.2 Fixtures: preparar datos

Una **fixture** es una función que prepara algo para los tests. Se pide poniendo su nombre como parámetro:

```python
import pytest

@pytest.fixture
def carrito():
    c = Carrito()
    c.anadir("Taza", precio=8, cantidad=2)
    return c

def test_total(carrito):
    assert carrito.total == 16

def test_unidades(carrito):
    assert carrito.unidades == 2
```

Cada test recibe **una fixture nueva**: los tests no se afectan entre sí.

### 4.3 Fixtures incluidas útiles

| Fixture | Para qué |
|---|---|
| `tmp_path` | Carpeta temporal para crear archivos |
| `monkeypatch` | Cambiar variables de entorno, funciones o atributos durante el test |
| `capsys` | Capturar lo que se imprime con `print` |

### 4.4 Parametrizar: varios datos, un test

```python
@pytest.mark.parametrize("precio, porcentaje, esperado", [
    (100, 21, 21),
    (50, 10, 5),
    (0, 21, 0),
])
def test_calcular_iva(precio, porcentaje, esperado):
    assert calcular_iva(precio, porcentaje) == esperado
```

### 4.5 Opciones de ejecución

| Comando | Qué hace |
|---|---|
| `pytest -v` | Muestra cada test |
| `pytest -x` | Para en el primer fallo |
| `pytest -k "iva"` | Solo tests cuyo nombre contiene "iva" |
| `pytest tests/test_precios.py` | Solo ese archivo |
| `pytest --lf` | Solo los que fallaron la última vez |
| `pytest -s` | Muestra los `print` |
| `pytest --cov=src` | Cobertura (requiere `pytest-cov`) |

### 4.6 Marcas

```python
@pytest.mark.skip(reason="Pendiente de la nueva API")
def test_futuro(): ...

@pytest.mark.slow
def test_lento(): ...    # ejecutar sin ellos: pytest -m "not slow"
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```python
# src/texto.py
def capitalizar(texto: str) -> str:
    if not texto:
        return ""
    return texto[0].upper() + texto[1:].lower()
```

```python
# tests/test_texto.py
from src.texto import capitalizar

def test_primera_letra_en_mayuscula():
    assert capitalizar("hola") == "Hola"

def test_resto_en_minusculas():
    assert capitalizar("hOLA") == "Hola"

def test_texto_vacio():
    assert capitalizar("") == ""
```

### 5.2 Ejemplo habitual: archivos y variables de entorno

```python
# src/config.py
import os, json

def cargar_config(ruta):
    with open(ruta, encoding="utf-8") as f:
        datos = json.load(f)
    datos["entorno"] = os.environ.get("APP_ENV", "desarrollo")
    return datos
```

```python
# tests/test_config.py
import json
from src.config import cargar_config

def test_carga_config_y_entorno(tmp_path, monkeypatch):
    archivo = tmp_path / "config.json"
    archivo.write_text(json.dumps({"puerto": 8000}), encoding="utf-8")
    monkeypatch.setenv("APP_ENV", "test")

    config = cargar_config(archivo)

    assert config == {"puerto": 8000, "entorno": "test"}
```

### 5.3 Simular una función externa

```python
def test_sin_internet(monkeypatch):
    monkeypatch.setattr("src.clima.obtener_temperatura", lambda ciudad: 25)
    assert recomendar_ropa("Sevilla") == "Manga corta"
```

---

## 6. Buenas prácticas

- **Carpeta `tests/`** con archivos `test_*.py`.
- **Nombres descriptivos**: `test_rechaza_email_sin_arroba`.
- **Fixtures en `conftest.py`** cuando las usen varios archivos (pytest lo carga solo).
- **`tmp_path`** en lugar de crear archivos reales en el proyecto.
- **`pytest.approx`** para decimales.
- **Instala pytest en el entorno virtual** del proyecto y anótalo en `requirements-dev.txt`.

---

## 7. Diferencias importantes

### 7.1 pytest vs. unittest

| | pytest | unittest (incluido en Python) |
|---|---|---|
| Sintaxis | Funciones + `assert` | Clases + `self.assertEqual` |
| Fixtures | `@pytest.fixture` | `setUp` / `tearDown` |
| Parametrizar | `@pytest.mark.parametrize` | Manual |
| Recomendado | Sí | Lo verás en código antiguo |

pytest también ejecuta tests escritos con `unittest`.

### 7.2 pytest vs. Vitest

| Concepto | pytest | Vitest |
|---|---|---|
| Comprobar | `assert x == y` | `expect(x).toBe(y)` |
| Preparar | Fixtures | `beforeEach` |
| Varios datos | `parametrize` | `it.each` |
| Mocks | `monkeypatch`, `unittest.mock` | `vi.fn`, `vi.spyOn` |
| Errores | `pytest.raises` | `toThrow` |

### 7.3 `monkeypatch` vs. `unittest.mock`

`monkeypatch` reemplaza atributos y variables de forma sencilla. `unittest.mock` (`Mock`, `patch`) además registra llamadas para comprobarlas (`assert_called_once_with`).

---

## 8. Casos especiales

### 8.1 *ModuleNotFoundError: No module named 'src'*

pytest no encuentra tu código. Soluciones: ejecuta desde la raíz del proyecto, añade un `conftest.py` vacío en la raíz, o configura `pythonpath = ["."]` en `pyproject.toml`:

```toml
[tool.pytest.ini_options]
pythonpath = ["."]
testpaths = ["tests"]
```

### 8.2 Tests de código asíncrono

```bash
pip install pytest-asyncio
```

```python
import pytest

@pytest.mark.asyncio
async def test_asincrono():
    assert await obtener_dato() == 42
```

### 8.3 Testear una API (FastAPI / Flask)

Ambos frameworks incluyen un cliente de pruebas (`TestClient` en FastAPI, `app.test_client()` en Flask) que hace peticiones sin arrancar el servidor. Ver [[Testing 05 - Tests de integración y API]].

---

## 9. Resumen

- `pip install pytest`; archivos `test_*.py` con funciones `test_*`.
- Se comprueba con `assert`; errores con `pytest.raises`; decimales con `pytest.approx`.
- **Fixtures** preparan datos; se piden como parámetro.
- `tmp_path`, `monkeypatch` y `capsys` vienen incluidas.
- `@pytest.mark.parametrize` repite un test con varios datos.
- `pytest -v`, `-x`, `-k`, `--lf` para controlar la ejecución.
