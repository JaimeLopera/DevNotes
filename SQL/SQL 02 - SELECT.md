# 02 - SELECT

> [!info] ¿Qué es?
> `SELECT` es la instrucción de SQL que sirve para **consultar datos**: le pides a la base de datos que te enseñe información de una tabla. Es la instrucción que más vas a usar. Solo **lee**: nunca modifica ni borra nada.

---

## 1. Antes de empezar

- Una **tabla** está formada por **filas** (cada registro) y **columnas** (cada dato: nombre, precio...).
- El resultado de un `SELECT` es otra tabla, temporal, que solo existe mientras ves la consulta.
- Las palabras clave (`SELECT`, `FROM`...) **no distinguen mayúsculas de minúsculas**, pero por costumbre se escriben en MAYÚSCULAS para que se distingan de los nombres de tablas y columnas.
- Cada instrucción termina con `;`.
- Los comentarios se escriben con `--` (una línea) o `/* ... */` (varias líneas).
- Los textos se escriben con **comillas simples**: `'Audio'`.

Para todos los ejemplos de esta nota usaremos esta tabla, `productos`:

| id | nombre | categoria | precio | stock |
|----|--------|-----------|--------|-------|
| 1 | Teclado mecánico | Periféricos | 60.00 | 25 |
| 2 | Ratón inalámbrico | Periféricos | 25.00 | 40 |
| 3 | Monitor 27 pulgadas | Monitores | 230.00 | 8 |
| 4 | Webcam HD | Periféricos | 40.00 | NULL |
| 5 | Auriculares | Audio | 75.00 | 15 |

> [!note]
> `NULL` significa "no hay dato" (vacío de verdad). No es lo mismo que `0` ni que un texto vacío.

---

## 2. Concepto fundamental

Un `SELECT` responde a dos preguntas:

1. **¿De dónde saco los datos?** → `FROM` (la tabla).
2. **¿Qué columnas quiero ver?** → `SELECT` (la lista de columnas).

Piensa en una hoja de cálculo: `FROM` elige la hoja y `SELECT` elige qué columnas dejas visibles.

### El orden en que SQL lo "piensa"

Escribes las partes en un orden, pero SQL las **ejecuta en otro**. Entenderlo evita muchas confusiones (por ejemplo, por qué un alias no funciona en `WHERE`).

| Orden en que lo escribes | Orden en que SQL lo ejecuta |
|---|---|
| 1. `SELECT` | 1. `FROM` (elige la tabla) |
| 2. `FROM` | 2. `WHERE` (filtra filas) |
| 3. `WHERE` | 3. `GROUP BY` (agrupa) |
| 4. `GROUP BY` | 4. `HAVING` (filtra grupos) |
| 5. `HAVING` | 5. `SELECT` (elige columnas y crea alias) |
| 6. `ORDER BY` | 6. `DISTINCT` (quita repetidos) |
| 7. `LIMIT` | 7. `ORDER BY` (ordena) |
| | 8. `LIMIT` (recorta filas) |

> [!tip]
> Lo importante de esta nota es el paso 5: el `SELECT` se ejecuta **casi al final**. Las partes anteriores (`WHERE`, `GROUP BY`) todavía no "conocen" los alias que crees en el `SELECT`.

---

## 3. Sintaxis / estructura

La forma mínima:

```sql
SELECT columna1, columna2
FROM tabla;
```

La estructura completa (las partes entre corchetes son opcionales):

```sql
SELECT [DISTINCT] columnas
FROM tabla
[WHERE condición]
[GROUP BY columnas]
[HAVING condición]
[ORDER BY columnas]
[LIMIT n];
```

- Solo `SELECT` es obligatorio. `FROM` se usa siempre que quieras datos de una tabla.
- Cada cláusula opcional tiene su propia nota: [[SQL 03 - WHERE]], [[SQL 04 - ORDER BY y LIMIT]], [[SQL 06 - GROUP BY y HAVING]].
- Las columnas se separan con **comas**, y no se pone coma después de la última.

---

## 4. Elementos / propiedades / características

### 4.1 Elegir columnas concretas

Escribes los nombres de las columnas separados por comas. Salen **en el orden en que los escribes**, no en el orden de la tabla.

```sql
SELECT precio, nombre
FROM productos;
```

### 4.2 Todas las columnas con `*`

El asterisco significa "todas las columnas".

```sql
SELECT *
FROM productos;
```

### 4.3 Alias con `AS`

Un **alias** es un nombre temporal para una columna en el resultado. No cambia la tabla original.

```sql
SELECT nombre AS producto, precio AS precio_base
FROM productos;
```

- `AS` es opcional en la mayoría de sistemas, pero conviene escribirlo: se lee mejor y evita errores (ver sección 8).
- Si el alias tiene espacios o tildes, va entre **comillas dobles**: `AS "Precio con IVA"`.

### 4.4 Quitar repetidos con `DISTINCT`

`DISTINCT` elimina las filas repetidas del resultado.

```sql
SELECT DISTINCT categoria
FROM productos;
```

- Con **varias columnas**, `DISTINCT` mira la **combinación completa**: solo quita filas donde todas las columnas coinciden.
- Va justo después de `SELECT` y afecta a todas las columnas, no solo a la primera.

### 4.5 Columnas calculadas (expresiones)

En el `SELECT` puedes hacer operaciones: `+`, `-`, `*`, `/`. El cálculo se hace **fila por fila** y no cambia los datos guardados.

```sql
SELECT nombre, precio * 1.21 AS precio_con_iva
FROM productos;
```

> [!warning]
> La división entre dos números enteros depende del sistema: en unos (PostgreSQL, SQL Server) `5 / 2` da `2`, y en otros (MySQL, Oracle) da `2.5`. Si necesitas decimales, convierte alguno de los números (por ejemplo `5 / 2.0`).

### 4.6 Valores fijos (literales)

Puedes poner un valor fijo y se repetirá en todas las filas.

```sql
SELECT nombre, 'EUR' AS moneda
FROM productos;
```

### 4.7 Unir textos (concatenar)

Aquí **cada sistema es distinto**:

| Forma | Funciona en |
|---|---|
| `nombre \|\| ' (' \|\| categoria \|\| ')'` | Estándar SQL: PostgreSQL, SQLite, Oracle |
| `CONCAT(nombre, ' (', categoria, ')')` | MySQL, PostgreSQL, SQL Server (2012+) |
| `nombre + ' (' + categoria + ')'` | SQL Server |

> [!warning]
> En MySQL, `||` por defecto significa **OR**, no unir textos. Allí usa `CONCAT()`. Además, si alguna de las partes es `NULL`, el resultado puede ser `NULL` (depende del sistema).

### 4.8 Valores condicionales con `CASE`

`CASE` permite mostrar un valor u otro según una condición. Es estándar y funciona en todos los sistemas (más detalle en [[SQL 05 - Funciones]]).

```sql
SELECT nombre,
       CASE
         WHEN precio >= 100 THEN 'Caro'
         ELSE 'Normal'
       END AS tipo
FROM productos;
```

### 4.9 `NULL` en el resultado

- Cualquier operación con `NULL` da `NULL`: `NULL + 5` → `NULL`.
- `COALESCE(valor, otro)` devuelve el primer valor que **no** sea `NULL`. Es estándar y sirve para poner un valor por defecto.

```sql
SELECT nombre, COALESCE(stock, 0) AS unidades
FROM productos;
```

### 4.10 Nombres entre comillas (identificadores)

Si una tabla o columna tiene un nombre raro (espacios, palabra reservada), hay que "protegerlo":

| Sistema | Forma |
|---|---|
| Estándar SQL, PostgreSQL, Oracle, SQLite | `"nombre columna"` |
| MySQL | `` `nombre columna` `` |
| SQL Server | `[nombre columna]` |

> [!tip]
> Lo mejor es evitar espacios y tildes en los nombres de tablas y columnas (usa `precio_base`, no `Precio base`) y así no necesitas comillas nunca.

---

## 5. Ejemplos prácticos

### Ejemplo básico

Ver el nombre y el precio de todos los productos:

```sql
SELECT nombre, precio
FROM productos;
```

| nombre | precio |
|---|---|
| Teclado mecánico | 60.00 |
| Ratón inalámbrico | 25.00 |
| Monitor 27 pulgadas | 230.00 |
| Webcam HD | 40.00 |
| Auriculares | 75.00 |

### Ejemplo habitual

Lista de categorías sin repetir:

```sql
SELECT DISTINCT categoria
FROM productos;
```

| categoria |
|---|
| Periféricos |
| Monitores |
| Audio |

> [!note]
> Sin `DISTINCT`, "Periféricos" saldría 3 veces. Sin `ORDER BY`, el orden de las filas **no está garantizado**.

### Ejemplo completo

Combina alias, cálculo, `COALESCE` y `CASE` en una sola consulta:

```sql
SELECT
  nombre AS producto,
  precio,
  precio * 1.21 AS precio_con_iva,
  COALESCE(stock, 0) AS unidades,
  CASE
    WHEN stock IS NULL THEN 'Sin dato'
    WHEN stock < 10 THEN 'Quedan pocas'
    ELSE 'Disponible'
  END AS estado
FROM productos;
```

| producto | precio | precio_con_iva | unidades | estado |
|---|---|---|---|---|
| Teclado mecánico | 60.00 | 72.60 | 25 | Disponible |
| Ratón inalámbrico | 25.00 | 30.25 | 40 | Disponible |
| Monitor 27 pulgadas | 230.00 | 278.30 | 8 | Quedan pocas |
| Webcam HD | 40.00 | 48.40 | 0 | Sin dato |
| Auriculares | 75.00 | 90.75 | 15 | Disponible |

---

## 6. Buenas prácticas

- **Evita `SELECT *` en código real.** Está bien para explorar una tabla, pero en una aplicación trae columnas que no necesitas (más lento) y, si la tabla cambia, tu código puede romperse. Escribe siempre las columnas que necesitas.
- **Una columna por línea** cuando la consulta sea larga: se lee y se modifica mejor.
- **Palabras clave en MAYÚSCULAS**, nombres en minúsculas.
- **Pon alias claros** a todo lo calculado (`precio_con_iva`, no `precio * 1.21`).
- **Escribe siempre `AS`** al crear un alias.
- **No uses `DISTINCT` para "arreglar" duplicados raros.** Si salen repetidos sin querer, normalmente el problema está en la consulta (por ejemplo, un `JOIN` mal hecho). Busca la causa.
- **Evita espacios y tildes** en nombres de tablas, columnas y alias para no depender de comillas.
- **Cuando una consulta use varias tablas**, escribe `tabla.columna` para que no haya dudas (se verá en [[SQL 07 - JOIN]]).

---

## 7. Diferencias importantes

| Comparación | Diferencia |
|---|---|
| `SELECT columnas` vs `SELECT *` | Las columnas: solo lo necesario y estable. `*`: todo, cómodo para explorar pero frágil. |
| `SELECT` vs `WHERE` | `SELECT` decide **qué columnas** ves. `WHERE` decide **qué filas** ves. |
| `DISTINCT` vs `GROUP BY` | `DISTINCT` solo quita repetidos. `GROUP BY` agrupa para poder calcular (sumas, conteos...). |
| Alias en `SELECT` vs tabla real | El alias solo existe en el resultado; la tabla no cambia. |
| `NULL` vs `0` vs `''` | `NULL` = sin dato. `0` = número cero. `''` = texto vacío. Son tres cosas distintas. |

### Diferencias entre sistemas

| Qué | MySQL | PostgreSQL | SQL Server | Oracle | SQLite |
|---|---|---|---|---|---|
| Unir textos | `CONCAT()` | `\|\|` o `CONCAT()` | `+` o `CONCAT()` | `\|\|` | `\|\|` |
| Comillas de nombres | `` ` ` `` | `" "` | `[ ]` | `" "` | `" "` |
| `SELECT` sin `FROM` (`SELECT 1 + 1;`) | Sí | Sí | Sí | Versiones antiguas piden `FROM DUAL` | Sí |
| Limitar filas | `LIMIT` | `LIMIT` o `FETCH FIRST` | `TOP` o `FETCH` | `FETCH FIRST` | `LIMIT` |

> [!note]
> `FETCH FIRST n ROWS ONLY` es la forma del estándar SQL. Cómo limitar filas se explica en [[SQL 04 - ORDER BY y LIMIT]].

---

## 8. Casos especiales

### Falta una coma y sale un alias "fantasma"

❌ **Qué está mal**
```sql
SELECT nombre precio
FROM productos;
```
🤔 **Por qué pasa**: en la mayoría de sistemas, `AS` es opcional. SQL entiende que `precio` es el alias de `nombre` y devuelve **una sola columna** con el nombre `precio`, pero con los nombres de producto dentro.

✅ **Cómo arreglarlo**: pon la coma `SELECT nombre, precio` y escribe `AS` en los alias para que nunca haya confusión.

### Usar un alias en el `WHERE`

❌ **Qué está mal**
```sql
SELECT nombre, precio * 1.21 AS precio_con_iva
FROM productos
WHERE precio_con_iva > 50;
```
🤔 **Por qué pasa**: `WHERE` se ejecuta **antes** que `SELECT` (ver sección 2), así que el alias todavía no existe.

✅ **Cómo arreglarlo**: repite la expresión: `WHERE precio * 1.21 > 50`. (El alias sí se puede usar en `ORDER BY`, que se ejecuta después del `SELECT`.)

### `DISTINCT` con varias columnas

`SELECT DISTINCT categoria, precio` no devuelve categorías únicas: devuelve **combinaciones únicas** de categoría y precio. Si quieres una sola fila por categoría, pide solo la columna `categoria`.

### Operaciones con `NULL`

`precio * stock` para la Webcam (stock `NULL`) da `NULL`, no `0`. Si quieres tratarlo como cero, usa `COALESCE(stock, 0)`.

### `SELECT` sin tabla

Sirve para hacer cálculos o probar funciones sin usar ninguna tabla:

```sql
SELECT 2 + 3 AS resultado;
```

En versiones antiguas de Oracle hay que añadir `FROM DUAL`.

---

## 9. Resumen

- `SELECT` **consulta** datos; nunca los modifica.
- Estructura mínima: `SELECT columnas FROM tabla;`
- `SELECT` elige **columnas**; `WHERE` elige **filas**.
- `*` = todas las columnas (cómodo para explorar, evitar en código real).
- `AS` crea un **alias** temporal; si tiene espacios, va entre comillas dobles.
- `DISTINCT` quita filas repetidas (mira la combinación completa de columnas).
- Puedes calcular con `+ - * /`, unir textos y usar `CASE` y `COALESCE`.
- Cualquier operación con `NULL` da `NULL`.
- El `SELECT` se ejecuta **después** de `FROM`, `WHERE` y `GROUP BY`: por eso un alias no vale en `WHERE`.
- Unir textos, comillas de nombres y limitar filas **cambian según el sistema** (MySQL, PostgreSQL, SQL Server, Oracle, SQLite).