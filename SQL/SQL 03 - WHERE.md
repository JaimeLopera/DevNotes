# 03 - WHERE

> [!info] ¿Qué es?
> `WHERE` es la cláusula de SQL que **filtra filas**: solo se quedan en el resultado las filas que cumplen una condición. Si `SELECT` decide qué columnas ves, `WHERE` decide **qué filas** ves.

---

## 1. Antes de empezar

- Se asume que ya conoces [[SQL 02 - SELECT]] y `FROM`.
- Una **condición** es una pregunta que SQL responde con verdadero o falso para cada fila (por ejemplo, "¿el precio es menor que 50?"). Solo pasan las filas donde la respuesta es **verdadero**.
- `WHERE` va siempre **después de `FROM`** y antes de `GROUP BY` y `ORDER BY`.
- Se usa también en `UPDATE` y `DELETE` (ver sección 4.9).

Usaremos la misma tabla `productos` de la nota anterior:

| id | nombre | categoria | precio | stock |
|----|--------|-----------|--------|-------|
| 1 | Teclado mecánico | Periféricos | 60.00 | 25 |
| 2 | Ratón inalámbrico | Periféricos | 25.00 | 40 |
| 3 | Monitor 27 pulgadas | Monitores | 230.00 | 8 |
| 4 | Webcam HD | Periféricos | 40.00 | NULL |
| 5 | Auriculares | Audio | 75.00 | 15 |

---

## 2. Concepto fundamental

SQL recorre la tabla **fila por fila** y, para cada una, comprueba la condición del `WHERE`. Cada comprobación puede dar tres resultados:

| Resultado | Qué pasa con la fila |
|---|---|
| Verdadero (`TRUE`) | Entra en el resultado |
| Falso (`FALSE`) | Se descarta |
| Desconocido (`UNKNOWN`) | Se descarta |

El "desconocido" aparece cuando hay un `NULL` de por medio: SQL no puede saber si `NULL > 10` es cierto o falso, y **por eso la fila se descarta**. Es la fuente de casi todos los líos con `NULL` (ver secciones 4.6 y 8).

> [!tip]
> Recuerda el orden de ejecución: `FROM` → **`WHERE`** → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. El `WHERE` se ejecuta **antes** del `SELECT`, así que no conoce los alias que crees allí.

---

## 3. Sintaxis / estructura

```sql
SELECT columnas
FROM tabla
WHERE condición;
```

Una condición se forma con:

```text
columna  operador  valor
```

Por ejemplo: `precio < 50`, `categoria = 'Audio'`.

- Las condiciones se pueden combinar con `AND`, `OR` y `NOT`.
- Los **textos** y las **fechas** van entre comillas simples; los **números**, sin comillas.

---

## 4. Elementos / propiedades / características

### 4.1 Operadores de comparación

| Operador | Significa | Ejemplo |
|---|---|---|
| `=` | Igual a | `categoria = 'Audio'` |
| `<>` | Distinto de | `categoria <> 'Audio'` |
| `<` | Menor que | `precio < 50` |
| `>` | Mayor que | `precio > 50` |
| `<=` | Menor o igual | `stock <= 10` |
| `>=` | Mayor o igual | `precio >= 60` |

- Para "igual" se usa **un solo `=`** (no `==` como en otros lenguajes).
- `<>` es el "distinto" del estándar. `!=` también funciona en casi todos los sistemas, pero `<>` funciona en todos.

### 4.2 Combinar condiciones: `AND`, `OR`, `NOT`

| Operador | Cuándo da verdadero |
|---|---|
| `A AND B` | Si se cumplen **las dos** |
| `A OR B` | Si se cumple **al menos una** |
| `NOT A` | Si **no** se cumple A |

**Prioridad:** SQL evalúa primero `NOT`, luego `AND` y al final `OR`. Si mezclas `AND` y `OR`, **usa paréntesis** para dejar claro lo que quieres (ver sección 8).

### 4.3 Rangos con `BETWEEN`

```sql
WHERE precio BETWEEN 30 AND 100
```

Equivale a `precio >= 30 AND precio <= 100`. Los dos extremos **están incluidos**. Se puede negar: `NOT BETWEEN`.

### 4.4 Lista de valores con `IN`

```sql
WHERE categoria IN ('Audio', 'Monitores')
```

Equivale a `categoria = 'Audio' OR categoria = 'Monitores'`, pero más corto y legible. Se puede negar: `NOT IN`. Dentro del paréntesis también puede ir una subconsulta (ver [[SQL 08 - Subconsultas]]).

### 4.5 Buscar patrones con `LIKE`

`LIKE` compara textos con un **patrón**. Tiene dos comodines:

| Comodín | Significa |
|---|---|
| `%` | Cualquier cantidad de caracteres (también ninguno) |
| `_` | Exactamente un carácter |

| Patrón | Encuentra |
|---|---|
| `'Mon%'` | Textos que **empiezan** por "Mon" |
| `'%HD'` | Textos que **terminan** en "HD" |
| `'%ratón%'` | Textos que **contienen** "ratón" |
| `'A_dio'` | "Audio", "Adio"... (un carácter en medio) |

- Si necesitas buscar un `%` o `_` de verdad, se usa `ESCAPE`: `LIKE '10\%' ESCAPE '\'`.
- Cuidado: un patrón que empieza por `%` obliga a revisar toda la tabla (es lento en tablas grandes).

### 4.6 Valores vacíos: `IS NULL` e `IS NOT NULL`

`NULL` significa "no hay dato". **No se puede comparar con `=`**: hay que usar `IS NULL`.

```sql
WHERE stock IS NULL        -- filas sin dato de stock
WHERE stock IS NOT NULL    -- filas con dato de stock
```

### 4.7 Comparar textos y fechas

- **Textos:** entre comillas simples. Si el texto lleva una comilla dentro, se duplica: `'L''Oréal'`.
- **Fechas:** el formato más seguro en todos los sistemas es `'AAAA-MM-DD'`, por ejemplo `fecha >= '2026-01-01'`.
- Si el texto distingue o no mayúsculas depende del sistema (ver sección 7).

### 4.8 Expresiones y funciones en el `WHERE`

Dentro de la condición puedes calcular y usar funciones (más en [[SQL 05 - Funciones]]):

```sql
WHERE precio * 1.21 > 50
WHERE UPPER(categoria) = 'AUDIO'
```

> [!warning]
> En el `WHERE` **no puedes usar alias** del `SELECT` (todavía no existen). Repite la expresión.

### 4.9 `WHERE` en `UPDATE` y `DELETE`

El `WHERE` también decide **qué filas se modifican o se borran**:

```sql
UPDATE productos SET precio = 65 WHERE id = 1;
DELETE FROM productos WHERE id = 4;
```

> [!warning] Importante
> Si **olvidas el `WHERE`** en un `UPDATE` o un `DELETE`, la instrucción afecta a **todas las filas de la tabla**. Más detalle en [[SQL 10 - UPDATE]] y [[SQL 11 - DELETE]].

---

## 5. Ejemplos prácticos

### Ejemplo básico

Productos de la categoría Periféricos:

```sql
SELECT nombre, precio
FROM productos
WHERE categoria = 'Periféricos';
```

| nombre | precio |
|---|---|
| Teclado mecánico | 60.00 |
| Ratón inalámbrico | 25.00 |
| Webcam HD | 40.00 |

### Ejemplo habitual

Periféricos que cuesten menos de 50:

```sql
SELECT nombre, precio
FROM productos
WHERE categoria = 'Periféricos'
  AND precio < 50;
```

| nombre | precio |
|---|---|
| Ratón inalámbrico | 25.00 |
| Webcam HD | 40.00 |

### Ejemplo completo

Productos de Periféricos o Audio, con precio entre 30 y 100, y que tengan más de 20 unidades **o** no tengan dato de stock:

```sql
SELECT nombre, categoria, precio, stock
FROM productos
WHERE categoria IN ('Periféricos', 'Audio')
  AND precio BETWEEN 30 AND 100
  AND (stock > 20 OR stock IS NULL);
```

| nombre | categoria | precio | stock |
|---|---|---|---|
| Teclado mecánico | Periféricos | 60.00 | 25 |
| Webcam HD | Periféricos | 40.00 | NULL |

> [!note]
> El Ratón queda fuera por precio (25 no está entre 30 y 100). Los Auriculares quedan fuera porque su stock es 15 (no es mayor que 20 ni es `NULL`).

---

## 6. Buenas prácticas

- **Usa paréntesis** siempre que mezcles `AND` y `OR`, aunque creas que no hacen falta.
- **Para `NULL`, usa siempre `IS NULL` / `IS NOT NULL`**, nunca `= NULL`.
- **Escribe una condición por línea** cuando haya varias: se lee y se depuera mejor.
- **Prefiere `IN` a una cadena de `OR`** sobre la misma columna.
- **No apliques funciones a la columna** si puedes evitarlo: `WHERE UPPER(nombre) = 'X'` suele impedir que la base de datos use un índice (ver [[SQL 14 - Índices]]). Es mejor comparar la columna "tal cual".
- **Evita `LIKE '%algo'`** (comodín al principio) en tablas grandes: es lento.
- **Antes de un `UPDATE` o `DELETE`, prueba el `WHERE` con un `SELECT`** para ver exactamente qué filas se verían afectadas.
- **Nunca olvides el `WHERE`** en `UPDATE` y `DELETE`.

---

## 7. Diferencias importantes

| Comparación | Diferencia |
|---|---|
| `WHERE` vs `HAVING` | `WHERE` filtra **filas** antes de agrupar. `HAVING` filtra **grupos** después de agrupar (ver [[SQL 06 - GROUP BY y HAVING]]). |
| `=` vs `LIKE` | `=` busca el valor exacto. `LIKE` busca un patrón con comodines. |
| `IN` vs varios `OR` | Hacen lo mismo; `IN` es más corto y claro. |
| `BETWEEN` vs `>=` y `<=` | Hacen lo mismo; `BETWEEN` incluye los dos extremos. |
| `IS NULL` vs `= NULL` | `IS NULL` funciona. `= NULL` nunca da verdadero. |
| `AND` vs `OR` | `AND` es más restrictivo (menos filas). `OR` es más amplio (más filas). |
| `WHERE` vs `ON` (JOIN) | `ON` dice cómo se unen las tablas; `WHERE` filtra el resultado (ver [[SQL 07 - JOIN]]). |

### Diferencias entre sistemas

| Qué | MySQL | PostgreSQL | SQL Server | Oracle | SQLite |
|---|---|---|---|---|---|
| `=` y `LIKE` con mayúsculas | No distingue (por defecto) | Distingue | No distingue (por defecto) | Distingue | `=` distingue; `LIKE` no (solo letras sin tilde) |
| `LIKE` sin distinguir mayúsculas | Ya lo hace | `ILIKE` | Ya lo hace | `UPPER()` en ambos lados | Ya lo hace |
| Valores verdadero/falso | `1` / `0` (o `TRUE`/`FALSE`) | `TRUE` / `FALSE` | `1` / `0` | Según versión (no hay tipo booleano clásico) | `1` / `0` |
| Fecha literal | `'2026-10-14'` | `'2026-10-14'` | `'2026-10-14'` | `DATE '2026-10-14'` | `'2026-10-14'` |

> [!note]
> En SQL Server y MySQL el comportamiento con mayúsculas depende de la *collation* (configuración de comparación de textos). Lo normal es que no distingan.

---

## 8. Casos especiales

### Comparar con `NULL` usando `=`

❌ **Qué está mal**
```sql
SELECT nombre FROM productos WHERE stock = NULL;
```
🤔 **Por qué pasa**: comparar cualquier cosa con `NULL` da "desconocido", y las filas desconocidas se descartan. No sale ninguna fila, ni siquiera la Webcam (que tiene stock `NULL`). Tampoco funciona `stock <> NULL`.

✅ **Cómo arreglarlo**: `WHERE stock IS NULL`.

### Mezclar `AND` y `OR` sin paréntesis

❌ **Qué está mal**
```sql
WHERE categoria = 'Audio' OR categoria = 'Monitores' AND precio < 100
```
🤔 **Por qué pasa**: `AND` se evalúa antes que `OR`, así que SQL lo lee como `categoria = 'Audio' OR (categoria = 'Monitores' AND precio < 100)`. Devuelve todos los de Audio sin mirar el precio.

✅ **Cómo arreglarlo**: pon los paréntesis donde los quieres: `(categoria = 'Audio' OR categoria = 'Monitores') AND precio < 100`.

### `NOT IN` con un `NULL` dentro

❌ **Qué está mal**
```sql
WHERE categoria NOT IN ('Audio', NULL)
```
🤔 **Por qué pasa**: `NOT IN` equivale a "distinto de Audio **y** distinto de NULL". Lo segundo nunca es verdadero (da "desconocido"), así que **no sale ninguna fila**. Es muy típico cuando la lista viene de una subconsulta que devuelve algún `NULL`.

✅ **Cómo arreglarlo**: asegúrate de que la lista no contiene `NULL` (por ejemplo, filtrando con `IS NOT NULL` dentro de la subconsulta).

### Usar un alias en el `WHERE`

❌ **Qué está mal**
```sql
SELECT nombre, precio * 1.21 AS precio_con_iva
FROM productos
WHERE precio_con_iva > 50;
```
🤔 **Por qué pasa**: el `WHERE` se ejecuta antes que el `SELECT`, así que el alias aún no existe.

✅ **Cómo arreglarlo**: repite la expresión: `WHERE precio * 1.21 > 50`.

### Comillas dobles en vez de simples para textos

❌ **Qué está mal**
```sql
WHERE categoria = "Audio"
```
🤔 **Por qué pasa**: en el estándar SQL, las comillas dobles son para **nombres** (columnas, tablas), no para textos. MySQL y SQLite lo toleran, pero PostgreSQL, Oracle o SQL Server dan error o lo toman como una columna llamada `Audio`.

✅ **Cómo arreglarlo**: usa siempre comillas simples para textos: `'Audio'`.

### Si ninguna fila cumple la condición

No es un error: el resultado es una tabla **vacía** (con las columnas pero sin filas).

---

## 9. Resumen

- `WHERE` **filtra filas**; `SELECT` filtra columnas.
- SQL comprueba la condición fila por fila: solo pasan las que dan **verdadero**; las falsas y las "desconocidas" se descartan.
- Operadores: `=`, `<>`, `<`, `>`, `<=`, `>=`.
- Combinar: `AND` (las dos), `OR` (una al menos), `NOT` (negar). Prioridad: `NOT` > `AND` > `OR`; **usa paréntesis**.
- `BETWEEN` incluye los dos extremos; `IN` compara con una lista; `LIKE` busca patrones (`%` y `_`).
- Para `NULL`: **`IS NULL` / `IS NOT NULL`**, nunca `= NULL`. Cuidado con `NOT IN` si hay `NULL`.
- Textos y fechas con **comillas simples**; fechas en formato `'AAAA-MM-DD'`.
- No se pueden usar alias del `SELECT` en el `WHERE`.
- En `UPDATE` y `DELETE`, **no olvides el `WHERE`**: sin él afectas a toda la tabla.
- Mayúsculas en comparaciones y fechas literales **cambian según el sistema**.