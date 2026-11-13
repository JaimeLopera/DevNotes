# 05 - Funciones

> [!info] ¿Qué es?
> Una **función** es una pequeña herramienta de SQL que recibe uno o varios valores y devuelve un resultado: pasar un texto a mayúsculas, redondear un número, contar filas, calcular una media... Se escribe con el nombre de la función y unos paréntesis: `UPPER(nombre)`.

---

## 1. Antes de empezar

- Se asume que ya conoces [[SQL 02 - SELECT]], [[SQL 03 - WHERE]] y [[SQL 04 - ORDER BY y LIMIT]].
- Las funciones se pueden usar en el `SELECT`, en el `WHERE`, en el `ORDER BY` y dentro de otras funciones.
- Hay **dos grandes familias**: las funciones **escalares** (una por fila) y las de **agregado** (resumen de muchas filas). Se explican en la sección 2.
- **Cada sistema tiene sus propias funciones** y a veces con nombres distintos. Hay un núcleo estándar (`COALESCE`, `CAST`, `CASE`, `COUNT`...) que funciona en todos; el resto se muestra por sistema.
- Usaremos la misma tabla `productos`:

| id | nombre | categoria | precio | stock |
|----|--------|-----------|--------|-------|
| 1 | Teclado mecánico | Periféricos | 60.00 | 25 |
| 2 | Ratón inalámbrico | Periféricos | 25.00 | 40 |
| 3 | Monitor 27 pulgadas | Monitores | 230.00 | 8 |
| 4 | Webcam HD | Periféricos | 40.00 | NULL |
| 5 | Auriculares | Audio | 75.00 | 15 |

---

## 2. Concepto fundamental

| Tipo | Cómo trabaja | Ejemplo | Resultado |
|---|---|---|---|
| **Escalar** | Se aplica a **cada fila** y devuelve **un valor por fila** | `UPPER(nombre)` | 5 filas → 5 resultados |
| **Agregado** | Junta **muchas filas** y devuelve **un solo valor** | `AVG(precio)` | 5 filas → 1 resultado |

Reglas que conviene recordar desde el principio:

- Si a una función escalar le llega un `NULL`, normalmente devuelve `NULL`.
- Las funciones de agregado **ignoran los `NULL`** (salvo `COUNT(*)`).
- Se pueden **anidar**: `ROUND(AVG(precio), 1)` calcula primero la media y luego la redondea.

---

## 3. Sintaxis / estructura

```sql
FUNCION(argumento1, argumento2, ...)
```

- Los **argumentos** son los valores que le das a la función: columnas, números, textos u otras funciones.
- Algunas funciones no llevan argumentos, pero **casi siempre** llevan paréntesis vacíos: `NOW()`. Excepción: las del estándar `CURRENT_DATE` y `CURRENT_TIMESTAMP` se escriben **sin paréntesis**.
- Ponle siempre un **alias** al resultado: `ROUND(precio, 1) AS precio_redondeado`.

---

## 4. Elementos / propiedades / características

### 4.1 Funciones de agregado

Funcionan igual en **todos** los sistemas.

| Función | Qué hace |
|---|---|
| `COUNT(*)` | Cuenta **todas las filas** (aunque tengan `NULL`) |
| `COUNT(columna)` | Cuenta las filas donde esa columna **no es `NULL`** |
| `COUNT(DISTINCT columna)` | Cuenta los valores **distintos** (sin repetidos y sin `NULL`) |
| `SUM(columna)` | Suma los valores |
| `AVG(columna)` | Calcula la media |
| `MIN(columna)` | El valor más pequeño (en textos, el primero alfabéticamente) |
| `MAX(columna)` | El valor más grande |

- Sin `GROUP BY`, el resultado es **una sola fila** con el resumen de toda la tabla.
- Para sacar un resumen **por grupos** (por categoría, por cliente...) se usa [[SQL 06 - GROUP BY y HAVING]].
- `SUM` y `AVG` solo sirven con números. `MIN`, `MAX` y `COUNT` también con textos y fechas.

### 4.2 Funciones de texto

| Qué hace | Forma habitual | Variantes por sistema |
|---|---|---|
| Mayúsculas / minúsculas | `UPPER(t)`, `LOWER(t)` | Igual en todos |
| Longitud | `LENGTH(t)` | SQL Server: `LEN(t)`. Estándar: `CHAR_LENGTH(t)` (en MySQL, `LENGTH` cuenta **bytes**, no caracteres) |
| Quitar espacios de los extremos | `TRIM(t)` | SQL Server antiguo (antes de 2017): `LTRIM(RTRIM(t))` |
| Extraer una parte | `SUBSTRING(t, inicio, longitud)` | Oracle y SQLite: `SUBSTR(t, inicio, longitud)`. La primera posición es la **1** |
| Reemplazar | `REPLACE(t, 'viejo', 'nuevo')` | Igual en todos |
| Posición de un texto | `POSITION('x' IN t)` | SQL Server: `CHARINDEX('x', t)`. Oracle y SQLite: `INSTR(t, 'x')` (MySQL acepta también `INSTR`) |
| Primeros / últimos caracteres | `LEFT(t, n)`, `RIGHT(t, n)` | **No existen** en Oracle ni SQLite: usa `SUBSTR` |
| Unir textos | `CONCAT(a, b, ...)` | Ver [[SQL 02 - SELECT]], apartado 4.7 |

> [!note]
> Si un texto es `NULL`, casi todas estas funciones devuelven `NULL`.

### 4.3 Funciones numéricas

| Qué hace | Forma habitual | Variantes por sistema |
|---|---|---|
| Redondear | `ROUND(x, decimales)` | En PostgreSQL, con decimales solo acepta `NUMERIC`: si el valor es decimal "flotante", conviértelo con `CAST(x AS NUMERIC)` |
| Redondear hacia arriba | `CEIL(x)` | SQL Server: solo `CEILING(x)` (MySQL y PostgreSQL aceptan las dos) |
| Redondear hacia abajo | `FLOOR(x)` | Igual en todos |
| Valor absoluto | `ABS(x)` | Igual en todos |
| Resto de una división | `MOD(a, b)` | SQL Server y SQLite: `a % b` |
| Potencia y raíz | `POWER(x, y)`, `SQRT(x)` | En SQLite solo en versiones recientes (según cómo se haya compilado) |

### 4.4 Funciones de fecha

Aquí hay **más diferencias entre sistemas**. Lo único estándar y común a todos es `CURRENT_TIMESTAMP` (fecha y hora actuales).

| Qué hace | MySQL | PostgreSQL | SQL Server | Oracle | SQLite |
|---|---|---|---|---|---|
| Fecha y hora actuales | `CURRENT_TIMESTAMP` | `CURRENT_TIMESTAMP` | `CURRENT_TIMESTAMP` | `CURRENT_TIMESTAMP` | `CURRENT_TIMESTAMP` |
| Solo la fecha de hoy | `CURRENT_DATE` | `CURRENT_DATE` | `CAST(GETDATE() AS DATE)` | `CURRENT_DATE` | `CURRENT_DATE` |
| Año de una fecha | `YEAR(f)` | `EXTRACT(YEAR FROM f)` | `YEAR(f)` | `EXTRACT(YEAR FROM f)` | `STRFTIME('%Y', f)` |
| Sumar 7 días | `DATE_ADD(f, INTERVAL 7 DAY)` | `f + INTERVAL '7 days'` | `DATEADD(day, 7, f)` | `f + 7` | `DATE(f, '+7 days')` |
| Días entre dos fechas | `DATEDIFF(f2, f1)` | `f2 - f1` | `DATEDIFF(day, f1, f2)` | `f2 - f1` | `JULIANDAY(f2) - JULIANDAY(f1)` |

> [!tip]
> `EXTRACT(parte FROM fecha)` es la forma estándar: sirve para `YEAR`, `MONTH`, `DAY`, `HOUR`... Funciona en MySQL, PostgreSQL y Oracle.

### 4.5 Convertir tipos: `CAST`

`CAST(valor AS tipo)` convierte un valor a otro tipo. Es la forma **estándar**:

```sql
CAST('123' AS INTEGER)
CAST(precio AS DECIMAL(10, 2))
CAST('2026-10-14' AS DATE)
```

| Sistema | Particularidades |
|---|---|
| MySQL | Para enteros se usa `SIGNED` (o `UNSIGNED`) en vez de `INTEGER`: `CAST('123' AS SIGNED)` |
| PostgreSQL | También `valor::INTEGER` |
| SQL Server | Tipo entero: `INT`. También `CONVERT(INT, valor)` y `TRY_CAST` (devuelve `NULL` si no se puede convertir) |
| Oracle | También `TO_NUMBER`, `TO_CHAR`, `TO_DATE` |

### 4.6 Funciones para `NULL`

| Función | Qué hace |
|---|---|
| `COALESCE(a, b, c...)` | Devuelve el **primer valor que no sea `NULL`**. Estándar, funciona en todos |
| `NULLIF(a, b)` | Devuelve `NULL` si `a` es igual a `b`; si no, devuelve `a`. Estándar |
| `IFNULL(a, b)` | Como `COALESCE` con 2 valores. MySQL y SQLite |
| `ISNULL(a, b)` | Como `COALESCE` con 2 valores. SQL Server |
| `NVL(a, b)` | Como `COALESCE` con 2 valores. Oracle |

> [!tip]
> Usa siempre `COALESCE`: es estándar y vale para todos los sistemas. Las otras tres son versiones propias de cada sistema.

### 4.7 `CASE`: elegir un valor según una condición

`CASE` es como un "si... entonces... si no..." dentro de SQL. Es estándar y funciona en todos los sistemas.

```sql
CASE
  WHEN condición1 THEN resultado1
  WHEN condición2 THEN resultado2
  ELSE resultado_por_defecto
END
```

- Se evalúan las condiciones **en orden** y gana la **primera** que se cumple.
- Si no hay `ELSE` y ninguna se cumple, devuelve `NULL`.
- Todos los resultados deben ser de tipos compatibles (todos números, o todos textos...).
- Hay una versión corta para comparar con igualdad: `CASE categoria WHEN 'Audio' THEN 1 ELSE 0 END`.
- Se puede usar en `SELECT`, `WHERE`, `ORDER BY` y **dentro de funciones de agregado** (muy útil para contar con condiciones, ver ejemplo completo).

> [!note]
> Existen también las **funciones de ventana** (`ROW_NUMBER()`, `RANK()`, `SUM() OVER (...)`), que son más avanzadas y no se explican en esta nota.

---

## 5. Ejemplos prácticos

### Ejemplo básico

Nombres de los productos en mayúsculas:

```sql
SELECT UPPER(nombre) AS nombre_mayusculas
FROM productos;
```

| nombre_mayusculas |
|---|
| TECLADO MECÁNICO |
| RATÓN INALÁMBRICO |
| MONITOR 27 PULGADAS |
| WEBCAM HD |
| AURICULARES |

### Ejemplo habitual

Resumen de toda la tabla con funciones de agregado:

```sql
SELECT COUNT(*)      AS total_productos,
       COUNT(stock)  AS con_stock,
       MIN(precio)   AS minimo,
       MAX(precio)   AS maximo,
       AVG(precio)   AS media
FROM productos;
```

| total_productos | con_stock | minimo | maximo | media |
|---|---|---|---|---|
| 5 | 4 | 25.00 | 230.00 | 86.00 |

> [!note]
> `COUNT(*)` da 5 pero `COUNT(stock)` da 4, porque la Webcam tiene el stock a `NULL` y no se cuenta.

### Ejemplo completo

Contar **con condiciones** usando `CASE` dentro de `SUM`, y redondear la media:

```sql
SELECT SUM(CASE WHEN categoria = 'Periféricos' THEN 1 ELSE 0 END) AS perifericos,
       SUM(CASE WHEN precio >= 100 THEN 1 ELSE 0 END)             AS caros,
       ROUND(AVG(precio), 1)                                      AS precio_medio
FROM productos;
```

| perifericos | caros | precio_medio |
|---|---|---|
| 3 | 1 | 86.0 |

> [!tip]
> El truco: `CASE` convierte cada fila en un `1` (cumple) o un `0` (no cumple), y `SUM` los suma. Así cuentas filas que cumplen una condición en una sola consulta.

---

## 6. Buenas prácticas

- **Usa las funciones estándar** (`COALESCE`, `CAST`, `CASE`, `CURRENT_TIMESTAMP`, `EXTRACT`) cuando puedas: tu código funcionará en más sistemas.
- **Pon un alias** a todo resultado de una función.
- **Ten claro qué cuentas**: `COUNT(*)` cuenta filas; `COUNT(columna)` cuenta filas con dato en esa columna.
- **Controla los `NULL`**: usa `COALESCE` si quieres un valor por defecto.
- **Evita dividir por una columna que pueda ser 0 o `NULL`** sin protegerla con `NULLIF(divisor, 0)`.
- **Redondea solo al final**, para mostrar. Si redondeas a mitad de un cálculo, el error se va acumulando.
- **Para dinero usa `DECIMAL`** (no decimales "flotantes"), así las sumas y medias no arrastran errores de precisión.
- **No uses funciones sobre una columna dentro del `WHERE`** si puedes evitarlo: impide usar índices (ver [[SQL 14 - Índices]]).
- **Consulta la documentación de tu sistema** para fechas y textos: es donde más cambian los nombres.

---

## 7. Diferencias importantes

| Comparación | Diferencia |
|---|---|
| Escalar vs agregado | Escalar: un resultado **por fila**. Agregado: un resultado **para varias filas**. |
| `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)` | Todas las filas / filas con dato / valores distintos con dato. |
| `COALESCE` vs `IFNULL` / `ISNULL` / `NVL` | `COALESCE` es estándar y acepta muchos valores. Las otras son propias de un sistema y aceptan solo dos. |
| `CAST` vs `CONVERT` | `CAST` es estándar. `CONVERT` es propio de MySQL y SQL Server y se escribe distinto en cada uno. |
| `WHERE` vs `HAVING` con agregados | Los agregados **no se pueden usar en `WHERE`**. Se usan en `HAVING` (ver [[SQL 06 - GROUP BY y HAVING]]). |
| `CASE` vs `COALESCE` | `COALESCE` solo sustituye `NULL`. `CASE` sirve para cualquier condición. |

### Diferencias entre sistemas (las que más tropiezos dan)

| Qué | MySQL | PostgreSQL | SQL Server | Oracle | SQLite |
|---|---|---|---|---|---|
| Longitud de texto | `CHAR_LENGTH` | `LENGTH` | `LEN` | `LENGTH` | `LENGTH` |
| Extraer parte de un texto | `SUBSTRING` | `SUBSTRING` | `SUBSTRING` | `SUBSTR` | `SUBSTR` |
| `NULL` por un valor | `IFNULL` / `COALESCE` | `COALESCE` | `ISNULL` / `COALESCE` | `NVL` / `COALESCE` | `IFNULL` / `COALESCE` |
| Entero en `CAST` | `SIGNED` | `INTEGER` | `INT` | `INTEGER` | `INTEGER` |
| Resto de división | `MOD` o `%` | `MOD` o `%` | `%` | `MOD` | `%` |

---

## 8. Casos especiales

### Función de agregado en el `WHERE`

❌ **Qué está mal**
```sql
SELECT nombre FROM productos WHERE precio > AVG(precio);
```
🤔 **Por qué pasa**: el `WHERE` filtra fila a fila **antes** de calcular ningún resumen, así que aún no se sabe cuánto vale `AVG`.

✅ **Cómo arreglarlo**: calcula la media en una subconsulta: `WHERE precio > (SELECT AVG(precio) FROM productos)` (ver [[SQL 08 - Subconsultas]]). Si estás agrupando, usa `HAVING`.

### Mezclar una columna normal con un agregado, sin `GROUP BY`

❌ **Qué está mal**
```sql
SELECT nombre, COUNT(*) FROM productos;
```
🤔 **Por qué pasa**: `COUNT(*)` devuelve **una sola fila** y `nombre` tiene cinco valores. SQL no sabe cuál mostrar, y la mayoría de sistemas da error.

✅ **Cómo arreglarlo**: agrupa con `GROUP BY` o quita la columna normal (ver [[SQL 06 - GROUP BY y HAVING]]).

### La media ignora los `NULL`

`AVG(stock)` da **22** (88 ÷ 4 filas con dato), no 17,6 (88 ÷ 5). La Webcam, con stock `NULL`, no cuenta. Si quieres que cuente como cero, usa `AVG(COALESCE(stock, 0))`.

### `SUM` y `AVG` sobre una tabla vacía o con todo `NULL`

Devuelven `NULL`, no `0`. En cambio, `COUNT` sí devuelve `0`. Para obtener un 0: `COALESCE(SUM(stock), 0)`.

### División entre cero

❌ **Qué está mal**: `total / cantidad` cuando `cantidad` vale `0`.

🤔 **Por qué pasa**: en PostgreSQL, SQL Server y Oracle da **error** (MySQL devuelve `NULL`).

✅ **Cómo arreglarlo**: `total / NULLIF(cantidad, 0)`. Si `cantidad` es 0, el divisor pasa a `NULL` y el resultado también (sin error).

### Una función que "no existe"

Si te da un error como "función desconocida" con `LEN`, `LENGTH`, `NVL` o `DATEDIFF`, casi seguro es porque ese nombre es de **otro sistema**. Mira las tablas de las secciones 4 y 7.

---

## 9. Resumen

- Una **función** recibe valores entre paréntesis y devuelve un resultado.
- **Escalares**: un valor por fila (texto, números, fechas, conversión, `NULL`, `CASE`). **Agregados**: un valor para varias filas (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
- Los agregados **ignoran los `NULL`**, salvo `COUNT(*)`, y **no se pueden usar en `WHERE`**.
- Funciones del estándar que valen en todos los sistemas: `COALESCE`, `NULLIF`, `CAST`, `CASE`, `CURRENT_TIMESTAMP`, `UPPER`, `LOWER`, `REPLACE`, `ROUND`, `ABS`, `FLOOR`.
- **Texto y fechas son lo que más cambia** entre MySQL, PostgreSQL, SQL Server, Oracle y SQLite.
- `COALESCE` pone un valor por defecto a los `NULL`; `NULLIF` evita divisiones entre cero.
- `CASE` permite decidir un valor según condiciones, incluso dentro de `SUM` o `COUNT`.
- Pon siempre alias, redondea solo al final y evita funciones sobre columnas en el `WHERE`.