# POO (Programación Orientada a Objetos)

> [!info] ¿Qué es?
> La **programación orientada a objetos** organiza el código en **objetos** que combinan **datos** (atributos) y **comportamiento** (métodos). Una **clase** es el molde o plano; un **objeto** (o instancia) es algo concreto creado a partir de ese molde. Permite modelar cosas del mundo real (usuarios, productos, cuentas…) de forma ordenada y reutilizable.

## 1. Antes de empezar

Necesitas dominar funciones ([[Python 06 - Funciones]]), diccionarios ([[Python 09 - Diccionarios]]) y excepciones ([[Python 11 - Excepciones]]). Todo en Python ya es un objeto: números, textos, listas y funciones tienen atributos y métodos.

Piensa en una **clase** `Coche` (el plano) y en `mi_coche` (un coche concreto con su color, su marca y su velocidad actual).

## 2. Concepto fundamental

| Término | Significado |
|---|---|
| **Clase** | Plantilla que define cómo son y qué hacen los objetos |
| **Objeto / instancia** | Ejemplar concreto creado a partir de una clase |
| **Atributo** | Dato que guarda el objeto (`nombre`, `edad`) |
| **Método** | Función que pertenece a la clase (`saludar()`) |
| **`self`** | El propio objeto sobre el que se llama al método |
| **Constructor** | `__init__`: se ejecuta al crear el objeto para inicializarlo |

### Los cuatro pilares

| Pilar | Idea |
|---|---|
| **Encapsulación** | Agrupar datos y métodos, y proteger lo interno |
| **Herencia** | Una clase puede reutilizar y ampliar a otra |
| **Polimorfismo** | Mismo método, comportamiento distinto según el objeto |
| **Abstracción** | Mostrar solo lo esencial y ocultar los detalles |

## 3. Sintaxis / estructura

```python
class Persona:
    especie = "humano"                  # atributo de clase (compartido)

    def __init__(self, nombre, edad):   # constructor
        self.nombre = nombre            # atributos de instancia
        self.edad = edad

    def saludar(self):                  # método
        return f"Hola, soy {self.nombre}"

ana = Persona("Ana", 20)                # crear un objeto
print(ana.nombre)                       # acceder a un atributo
print(ana.saludar())                    # llamar a un método
```

- `self` **siempre** es el primer parámetro de los métodos de instancia; Python lo pasa solo al llamar.
- Los nombres de clase van en `PascalCase`.
- Cada objeto tiene sus propios atributos de instancia.

## 4. Elementos y características

### Atributos de clase vs de instancia

```python
class Perro:
    especie = "canis"           # de clase: lo comparten todos
    def __init__(self, nombre):
        self.nombre = nombre    # de instancia: propio de cada perro
```

### Tipos de métodos

```python
class Util:
    contador = 0

    def instancia(self):            # recibe el objeto
        ...

    @classmethod
    def desde_texto(cls, texto):    # recibe la clase; sirve de constructor alternativo
        return cls(texto)

    @staticmethod
    def sumar(a, b):                # no recibe ni objeto ni clase
        return a + b
```

| Tipo | Primer parámetro | Se usa para |
|---|---|---|
| Instancia | `self` | Trabajar con los datos del objeto |
| `@classmethod` | `cls` | Constructores alternativos, datos de la clase |
| `@staticmethod` | ninguno | Funciones relacionadas pero independientes |

### Encapsulación (por convención)

Python no tiene atributos "privados" reales; se usan convenciones:

| Forma | Significado |
|---|---|
| `nombre` | Público |
| `_nombre` | "Uso interno" (convención; sigue siendo accesible) |
| `__nombre` | Python cambia su nombre internamente (*name mangling*) para evitar choques en herencia |

### Propiedades (`@property`)

Permiten acceder a un método como si fuera un atributo, y **validar** al asignar:

```python
class Cuenta:
    def __init__(self, saldo):
        self._saldo = saldo

    @property
    def saldo(self):                 # getter
        return self._saldo

    @saldo.setter
    def saldo(self, valor):          # setter
        if valor < 0:
            raise ValueError("El saldo no puede ser negativo")
        self._saldo = valor

c = Cuenta(100)
c.saldo = 50      # usa el setter
print(c.saldo)    # usa el getter
```

### Herencia

```python
class Animal:
    def __init__(self, nombre):
        self.nombre = nombre

    def hablar(self):
        return "..."

class Perro(Animal):                      # Perro hereda de Animal
    def __init__(self, nombre, raza):
        super().__init__(nombre)          # llama al constructor del padre
        self.raza = raza

    def hablar(self):                     # sobrescribe el método
        return "Guau"

class Gato(Animal):
    def hablar(self):
        return "Miau"
```

- `super()` accede a la clase padre.
- `isinstance(obj, Clase)` comprueba si un objeto es de esa clase **o de una hija**.
- `issubclass(Perro, Animal)` comprueba relaciones entre clases.

### Polimorfismo

```python
for animal in [Perro("Rex", "labrador"), Gato("Mia")]:
    print(animal.hablar())       # cada uno responde a su manera
```

Python usa **duck typing**: "si camina como un pato y suena como un pato, es un pato". No importa de qué clase sea el objeto, sino que tenga el método.

### Herencia múltiple y MRO

```python
class C(A, B):
    ...
```

El orden en que Python busca métodos se llama **MRO** y se puede ver con `C.__mro__` o `C.mro()`. Úsala con cuidado.

### Clases abstractas

Obligan a las clases hijas a implementar ciertos métodos:

```python
from abc import ABC, abstractmethod

class Figura(ABC):
    @abstractmethod
    def area(self):
        ...

class Cuadrado(Figura):
    def __init__(self, lado):
        self.lado = lado
    def area(self):
        return self.lado ** 2
```

No se puede crear un objeto de `Figura` directamente.

### Métodos especiales ("dunder")

Permiten que tus objetos se comporten como los tipos nativos.

| Método | Se activa con | Uso |
|---|---|---|
| `__init__` | Crear el objeto | Inicializar |
| `__str__` | `print(obj)`, `str(obj)` | Texto legible para el usuario |
| `__repr__` | `repr(obj)`, el REPL | Texto para depurar |
| `__eq__` | `a == b` | Igualdad |
| `__lt__`, `__le__`… | `a < b` | Comparaciones y orden |
| `__add__`, `__sub__`… | `a + b` | Sobrecarga de operadores |
| `__len__` | `len(obj)` | Longitud |
| `__getitem__` | `obj[i]` | Acceso por índice o clave |
| `__iter__`, `__next__` | `for x in obj` | Hacerlo iterable ([[Python 15 - Iteradores y generators]]) |
| `__contains__` | `x in obj` | Pertenencia |
| `__bool__` | `if obj:` | Valor de verdad |
| `__call__` | `obj()` | Hacerlo "llamable" |
| `__enter__`, `__exit__` | `with obj:` | Gestor de contexto |
| `__hash__` | Claves y conjuntos | Hacerlo hashable |

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, otro):
        return Vector(self.x + otro.x, self.y + otro.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

print(Vector(1, 2) + Vector(3, 4))    # Vector(4, 6)
```

### Gestor de contexto propio

```python
class Temporizador:
    def __enter__(self):
        import time
        self.inicio = time.perf_counter()
        return self
    def __exit__(self, tipo, valor, traza):
        import time
        print(f"Tardó {time.perf_counter() - self.inicio:.3f}s")

with Temporizador():
    sum(range(1_000_000))
```

### `dataclass`: clases para guardar datos

Genera automáticamente `__init__`, `__repr__` y `__eq__`:

```python
from dataclasses import dataclass, field

@dataclass
class Producto:
    nombre: str
    precio: float
    etiquetas: list[str] = field(default_factory=list)

p = Producto("Libro", 20.0)
print(p)         # Producto(nombre='Libro', precio=20.0, etiquetas=[])
```

Opciones útiles: `@dataclass(frozen=True)` (inmutable), `@dataclass(order=True)` (comparable).

### `Enum`: conjunto de valores con nombre

```python
from enum import Enum

class Estado(Enum):
    PENDIENTE = 1
    HECHO = 2

Estado.HECHO.name     # "HECHO"
Estado.HECHO.value    # 2
```

### `__slots__`

Limita los atributos permitidos y ahorra memoria en clases con muchísimos objetos:

```python
class Punto:
    __slots__ = ("x", "y")
```

### Composición

Un objeto **contiene** a otros en vez de heredar de ellos ("tiene un" en vez de "es un"):

```python
class Motor:
    def arrancar(self):
        return "Brrr"

class Coche:
    def __init__(self):
        self.motor = Motor()
    def arrancar(self):
        return self.motor.arrancar()
```

## 5. Ejemplos prácticos

### Ejemplo básico

```python
class Rectangulo:
    def __init__(self, base, altura):
        self.base = base
        self.altura = altura

    def area(self):
        return self.base * self.altura

r = Rectangulo(3, 4)
print(r.area())     # 12
```

### Ejemplo habitual

```python
class Producto:
    def __init__(self, nombre, precio):
        self.nombre = nombre
        self.precio = precio

    def con_iva(self, iva=0.21):
        return round(self.precio * (1 + iva), 2)

    def __str__(self):
        return f"{self.nombre}: {self.precio} €"

p = Producto("Libro", 20)
print(p)             # Libro: 20 €
print(p.con_iva())   # 24.2
```

### Ejemplo completo: herencia, polimorfismo y propiedad

```python
from abc import ABC, abstractmethod

class Empleado(ABC):
    def __init__(self, nombre, salario_base):
        self.nombre = nombre
        self._salario_base = salario_base

    @property
    def salario_base(self):
        return self._salario_base

    @abstractmethod
    def salario_total(self):
        ...

    def __str__(self):
        return f"{self.nombre} ({self.__class__.__name__}): {self.salario_total()} €"

class Operario(Empleado):
    def salario_total(self):
        return self.salario_base

class Gerente(Empleado):
    def __init__(self, nombre, salario_base, bono):
        super().__init__(nombre, salario_base)
        self.bono = bono

    def salario_total(self):
        return self.salario_base + self.bono

equipo = [Operario("Luis", 1500), Gerente("Marta", 2500, 600)]
for e in equipo:
    print(e)
```

## 6. Buenas prácticas

- Nombres de clase en `PascalCase`; métodos y atributos en `snake_case`.
- **Una clase, una responsabilidad.**
- Define siempre `__repr__` (y `__str__` si hace falta) para depurar mejor.
- Prefiere **composición** a herencias profundas ("tiene un" antes que "es un").
- Inicializa **todos los atributos** en `__init__`.
- Usa `@property` en vez de métodos `get_x()` / `set_x()` al estilo de otros lenguajes.
- Usa `@dataclass` para clases que solo guardan datos.
- Usa `super().__init__()` cuando heredes.
- No abuses de la POO: en Python, una función simple suele bastar.
- Añade docstrings a clases y métodos públicos.
- No uses listas ni diccionarios como valores por defecto en parámetros ni como atributos de clase mutables compartidos sin querer.

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| Clase vs objeto | Molde frente a ejemplar concreto |
| Atributo de clase vs de instancia | Compartido por todos vs propio de cada objeto |
| `@staticmethod` vs `@classmethod` | El estático no recibe nada; el de clase recibe `cls` |
| Método vs función | El método pertenece a una clase y recibe `self` |
| `__str__` vs `__repr__` | Para el usuario vs para el desarrollador |
| Herencia vs composición | "Es un" vs "tiene un" |
| `isinstance` vs `type()` | `isinstance` respeta la herencia |
| `==` vs `is` en objetos | Igualdad definida por `__eq__` vs mismo objeto |
| Clase abstracta vs normal | La abstracta no se puede instanciar |
| `dataclass` vs clase normal | Genera código repetitivo automáticamente |

## 8. Casos especiales

- **Atributo de clase mutable compartido:**

❌
```python
class Carrito:
    items = []           # compartida por TODOS los carritos
```
🤔 Todos los objetos comparten la misma lista.
✅ Crea la lista en `__init__`: `self.items = []`.

- **Olvidar `self`:**

❌ `def saludar():` dentro de una clase
🤔 Python pasa el objeto automáticamente y sobra un argumento (`TypeError`).
✅ `def saludar(self):`

- **Comprobar `==` sin definir `__eq__`:** por defecto compara identidad, no contenido (dos objetos con los mismos datos serían distintos).
- **Si defines `__eq__`**, también conviene definir `__hash__` si el objeto va a usarse en conjuntos o como clave.
- **`__init__` no devuelve nada;** si hace `return valor` da error.
- **Herencia múltiple y "diamante":** el MRO resuelve el orden, pero `super()` debe usarse con cuidado y de forma consistente.
- **Los atributos se pueden añadir al vuelo** (`obj.nuevo = 5`) salvo que uses `__slots__` o `dataclass(frozen=True)`.
- **Acceso a "privados":** `_x` y `__x` no impiden el acceso; son avisos para quien lea el código.
- **Métodos mágicos de comparación:** con `functools.total_ordering` basta definir `__eq__` y uno de `<`, `>`, etc.
- **Copiar objetos:** `copy.copy()` (superficial) y `copy.deepcopy()` (profunda), igual que con listas ([[Python 07 - Listas]]).
- **Clases anidadas, metaclases y descriptores** existen, pero son temas avanzados que rara vez necesitarás al principio.

## 9. Resumen

- Clase = molde; objeto = instancia; atributos = datos; métodos = comportamiento.
- `__init__` construye el objeto; `self` es el propio objeto.
- Herencia con `class Hija(Padre)` y `super()`; polimorfismo y *duck typing*.
- Encapsulación por convención (`_x`, `__x`); `@property` para validar.
- `@classmethod` y `@staticmethod` para métodos que no necesitan la instancia.
- Métodos especiales (`__str__`, `__repr__`, `__eq__`, `__add__`, `__len__`…) integran tus clases con Python.
- `dataclass` para datos, `ABC` para contratos, `Enum` para valores fijos.
- Prefiere composición a herencia profunda y no abuses de la POO.