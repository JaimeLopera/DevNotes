# Python

> [!info] ¿Qué es?
> Python es un lenguaje de programación de **sintaxis clara y muy versátil**, usado en automatización, scripts, desarrollo backend, análisis de datos, inteligencia artificial y mucho más. Esta carpeta recorre el lenguaje de principio a fin: desde la sintaxis básica hasta herramientas avanzadas como generators, decoradores y entornos virtuales.

Cada nota funciona de dos maneras:

- **Repaso rápido** de un tema que ya conoces pero tienes algo olvidado.
- **Explicación desde cero** de un tema que has olvidado por completo.

## Notas de la carpeta

### Base del lenguaje

| Nota | Contenido |
|---|---|
| [[Python 01 - Fundamentos]] | Cómo funciona Python, ejecutar código, `print`, `input`, indentación |
| [[Python 02 - Variables y tipos]] | Variables, tipos, conversión, mutabilidad, *truthy* y *falsy* |
| [[Python 03 - Operadores]] | Aritméticos, lógicos, comparación, pertenencia, precedencia |
| [[Python 04 - Condicionales]] | `if`, `elif`, `else`, ternario y `match` |
| [[Python 05 - Bucles]] | `for`, `while`, `range`, `break`, `continue`, comprensiones |
| [[Python 06 - Funciones]] | `def`, parámetros, `*args`, `**kwargs`, ámbito, `lambda`, recursión |

### Estructuras de datos

| Nota | Contenido |
|---|---|
| [[Python 07 - Listas]] | Colección ordenada y mutable; slicing, métodos, ordenación |
| [[Python 08 - Tuplas y conjuntos]] | Tuplas inmutables y conjuntos sin repetidos |
| [[Python 09 - Diccionarios]] | Pares clave-valor, `get`, `items`, `Counter`, `defaultdict` |
| [[Python 10 - Strings]] | Textos, f-strings, métodos, `split`/`join`, codificación |

### Robustez y datos externos

| Nota | Contenido |
|---|---|
| [[Python 11 - Excepciones]] | `try`/`except`, `raise`, excepciones propias |
| [[Python 12 - Ficheros]] | `open`, `with`, `pathlib`, JSON, CSV |

### Organizar y estructurar el código

| Nota | Contenido |
|---|---|
| [[Python 13 - Módulos y paquetes]] | `import`, crear módulos y paquetes, biblioteca estándar |
| [[Python 14 - POO]] | Clases, herencia, `property`, métodos especiales, `dataclass` |

### Python avanzado

| Nota | Contenido |
|---|---|
| [[Python 15 - Iteradores y generators]] | `iter`, `next`, `yield`, `itertools`, evaluación perezosa |
| [[Python 16 - Decoradores]] | Funciones que envuelven funciones, `functools`, `lru_cache` |

### Trabajo real con Python

| Nota | Contenido |
|---|---|
| [[Python 17 - Entornos y paquetes]] | `venv`, `pip`, `requirements.txt`, `pyproject.toml`, `uv` |
| [[Python 18 - Debugging y buenas prácticas]] | `pdb`, `logging`, `pytest`, PEP 8, código limpio |
| [[Python 19 - Referencia rápida]] | Resumen de sintaxis y reglas de oro |

## Orden recomendado

1. **Empezar:** 01 → 05 (sintaxis, datos básicos, decisiones y repeticiones).
2. **Organizar:** 06 (funciones) y 07 → 10 (estructuras de datos).
3. **Hacerlo robusto:** 11 y 12.
4. **Estructurar proyectos:** 13 y 14.
5. **Profundizar:** 15 y 16.
6. **Trabajar como profesional:** 17 y 18.
7. **Consultar:** 19 siempre que necesites un recordatorio.

## Si lo que necesitas es…

| Necesidad | Ve a |
|---|---|
| Entender un error | [[Python 11 - Excepciones]] y [[Python 18 - Debugging y buenas prácticas]] |
| Guardar o leer datos | [[Python 12 - Ficheros]] |
| Instalar una librería | [[Python 17 - Entornos y paquetes]] |
| Procesar mucha información sin gastar memoria | [[Python 15 - Iteradores y generators]] |
| Recordar una sintaxis | [[Python 19 - Referencia rápida]] |
| Organizar un proyecto grande | [[Python 13 - Módulos y paquetes]] y [[Python 14 - POO]] |

## Temas relacionados fuera de esta carpeta

- [[00 - Ruta de aprendizaje]]
- [[Git 00 - Índice|Git]]
- [[Testing 00 - Índice|Testing]]
- [[Docker 00 - Índice|Docker]]
- [[Terminal 00 - Índice|Terminal y Linux]]
- [[Conceptos 03 - JSON|JSON]] y [[Conceptos 02 - APIs REST|APIs REST]] (en Conceptos generales)
- [[Conceptos 04 - Regex|Regex]] (en Conceptos generales)