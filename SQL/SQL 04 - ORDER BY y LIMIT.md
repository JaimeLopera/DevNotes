# 04 - ORDER BY y LIMIT

> [!info] ¿Qué es?
> `ORDER BY` **ordena** las filas del resultado y `LIMIT` **recorta** cuántas filas se devuelven. Juntos sirven para cosas como "los 3 productos más caros" o "la página 2 de una lista".

---

## 1. Antes de empezar

- Se asume que ya conoces [[SQL 02 - SELECT]] y [[SQL 03 - WHERE]].
- **Sin `ORDER BY`, el orden de las filas no está garantizado.** Puede parecer que salen en el orden en que las guardaste, pero la base de datos no lo asegura y puede cambiar en cualquier momento.
- `ORDER BY` no cambia la tabla; solo cambia el orden en que ves el resultado.
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

SQL primero **filtra** (`WHERE`), luego **ordena** (`ORDER BY`) y por último **recorta** (`LIMIT`). Este orden es el que hace que "los 3 más caros" funcione: primero se ordena de mayor a menor precio y después se cogen las 3 primeras.

| Orden en que SQL lo ejecuta | Qué hace |
|---|---|
| 1. `FROM` | Elige la tabla |
| 2. `WHERE` | Filtra filas |
| 3. `GROUP BY` / `HAVING` | Agrupa y filtra grupos |
| 4. `SELECT` | Elige columnas y crea alias |
| 5. **`ORDER BY`** | **Ordena** |
| 6. **`LIMIT`** | **Recorta** |

> [!tip]
> Como `ORDER BY` se ejecuta **después** del `SELECT`, aquí **sí puedes usar los alias** que creaste (en `WHERE` no).

---

## 3. Sintaxis / estructura

```sql
SELECT columnas
FROM tabla
[WHERE condición]
ORDER BY columna1 [ASC | DESC], columna2 [ASC | DESC]
LIMIT n [OFFSET m];
```

- `ORDER BY` va **después** de `WHERE`, `GROUP BY` y `HAVING`, y **antes** de `LIMIT`.
- `ASC` = de menor a mayor (es el valor por defecto, se puede omitir). `DESC` = de mayor a menor.
- `LIMIT n` = devuelve como máximo `n` filas. `OFFSET m` = salta las primeras `m` filas.

> [!warning]
> `LIMIT` **no es estándar** y no existe en todos los sistemas. Cada uno tiene su forma (ver sección 4.7).

---

## 4. Elementos / propiedades / características

### 4.1 Ordenar por una columna

```sql
ORDER BY precio          -- de menor a mayor (ASC por defecto)
ORDER BY precio DESC     -- de mayor a menor
```

Funciona con números, textos (alfabético) y fechas (cronológico).

### 4.2 Ordenar por varias columnas

Se separan con comas. SQL ordena por la primera; **solo si hay empate**, desempata con la segunda, y así sucesivamente. Cada columna tiene su propio `ASC` o `DESC`.

```sql
ORDER BY categoria ASC, precio DESC
```

### 4.3 Ordenar por un alias o una expresión

Como `ORDER BY` va después del `SELECT`, puedes usar alias y cálculos:

```sql
SELECT nombre, precio * 1.21 AS precio_con_iva
FROM productos
ORDER BY precio_con_iva DESC;
```

### 4.4 Ordenar por posición (número de columna)

`ORDER BY 2` ordena por la **segunda columna del `SELECT`**. Funciona en casi todos los sistemas, pero es frágil (si cambias el `SELECT`, cambia el orden sin avisar). Mejor usa el nombre.

### 4.5 Orden personalizado con `CASE`

Si el orden que quieres no es alfabético ni numérico, puedes inventarlo con `CASE` (ver [[SQL 05 - Funciones]]):

```sql
ORDER BY CASE categoria
           WHEN 'Audio' THEN 1
           WHEN 'Monitores' THEN 2
           ELSE 3
         END
```

### 4.6 Dónde van los `NULL`

El estándar permite decidirlo con `NULLS FIRST` o `NULLS LAST`:

```sql
ORDER BY stock ASC NULLS LAST
```

Si no lo indicas, depende del sistema (ver sección 7).

### 4.7 Limitar filas: `LIMIT`, `TOP` y `FETCH`

El mismo resultado ("las 3 primeras filas") se escribe distinto según el sistema:

| Sistema | Forma |
|---|---|
| MySQL, SQLite | `LIMIT 3` |
| PostgreSQL | `LIMIT 3` o la forma estándar |
| SQL Server | `SELECT TOP (3) ...` (justo después de `SELECT`) |
| Oracle (12c o superior) | `FETCH FIRST 3 ROWS ONLY` |
| **Estándar SQL** | `FETCH FIRST 3 ROWS ONLY` (al final de la consulta) |

La forma estándar funciona en PostgreSQL, Oracle 12c+ y SQL Server (este último pide que haya `ORDER BY`).

### 4.8 Saltar filas: `OFFSET` (paginación)

`OFFSET` se salta filas antes de empezar a devolver. Con `LIMIT` sirve para **paginar**:

| Sistema | Forma (saltar 2 y coger 2) |
|---|---|
| MySQL, PostgreSQL, SQLite | `LIMIT 2 OFFSET 2` |
| SQL Server, Oracle, estándar | `OFFSET 2 ROWS FETCH NEXT 2 ROWS ONLY` |

**Fórmula de paginación:** `OFFSET = (página - 1) × filas_por_página`.

> [!warning] Obsoleto / legado
> En Oracle antiguo (antes de 12c) se limitaba con `ROWNUM` (`WHERE ROWNUM <= 3`), y es poco fiable porque se evalúa **antes** de `ORDER BY`. Usa `FETCH FIRST n ROWS ONLY`. En MySQL también existe `LIMIT m, n` (salto, cantidad): funciona, pero confunde porque el orden de los números es al revés que en `OFFSET`. Mejor usa `LIMIT n OFFSET m`.

### 4.9 Empates en el último puesto

Si pides las 3 primeras filas y la 3.ª y la 4.ª empatan, `LIMIT` solo devuelve 3 (y elige una de las dos al azar). La forma estándar permite incluir los empates: `FETCH FIRST 3 ROWS WITH TIES` (PostgreSQL 13+, Oracle 12c+; en SQL Server es `TOP (3) WITH TIES`). MySQL y SQLite no lo tienen.

---

## 5. Ejemplos prácticos

### Ejemplo básico

Productos de más barato a más caro:

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio ASC;
```

| nombre | precio |
|---|---|
| Ratón inalámbrico | 25.00 |
| Webcam HD | 40.00 |
| Teclado mecánico | 60.00 |
| Auriculares | 75.00 |
| Monitor 27 pulgadas | 230.00 |

### Ejemplo habitual

Los 3 productos más caros:

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC
LIMIT 3;
```

| nombre | precio |
|---|---|
| Monitor 27 pulgadas | 230.00 |
| Auriculares | 75.00 |
| Teclado mecánico | 60.00 |

> [!note]
> En SQL Server sería `SELECT TOP (3) nombre, precio FROM productos ORDER BY precio DESC;` y en Oracle, `... ORDER BY precio DESC FETCH FIRST 3 ROWS ONLY;`.

### Ejemplo completo

Página 2 de la lista (2 productos por página), ordenada por categoría y, dentro de cada categoría, del más caro al más barato. Se añade `id` como desempate final para que el orden sea siempre el mismo:

```sql
SELECT nombre, categoria, precio
FROM productos
ORDER BY categoria ASC, precio DESC, id ASC
LIMIT 2 OFFSET 2;
```

El orden completo sería: Auriculares (Audio), Monitor (Monitores), Teclado, Webcam, Ratón (Periféricos). La página 2 son las filas 3 y 4:

| nombre | categoria | precio |
|---|---|---|
| Teclado mecánico | Periféricos | 60.00 |
| Webcam HD | Periféricos | 40.00 |

---

## 6. Buenas prácticas

- **Si el orden importa, pon siempre `ORDER BY`**. Nunca te fíes del orden "natural" de la tabla.
- **Usa `LIMIT` solo junto con `ORDER BY`**. Sin orden, las filas que devuelve son arbitrarias.
- **Añade una columna única al final** (como `id`) en el `ORDER BY` cuando pagines, para que el orden sea siempre el mismo.
- **Ordena por nombre de columna o alias**, no por posición (`ORDER BY 2`).
- **Pon siempre `ASC` o `DESC`** cuando sea importante: se lee mejor.
- **Ordena lo mínimo posible**: ordenar muchas filas es costoso. Filtra primero con `WHERE`. Un índice sobre la columna ordenada acelera mucho (ver [[SQL 14 - Índices]]).
- **Paginar con `OFFSET` muy grandes es lento**: SQL tiene que recorrer y descartar todas las filas saltadas.
- **Decide dónde van los `NULL`** (`NULLS FIRST` o `NULLS LAST`) si importa, porque cambia según el sistema.

---

## 7. Diferencias importantes

| Comparación | Diferencia |
|---|---|
| `ASC` vs `DESC` | `ASC`: menor a mayor, A a Z, antiguo a reciente. `DESC`: al revés. |
| `ORDER BY` vs `GROUP BY` | `ORDER BY` solo ordena. `GROUP BY` agrupa filas para calcular (ver [[SQL 06 - GROUP BY y HAVING]]). |
| `LIMIT` vs `WHERE` | `WHERE` filtra por una condición. `LIMIT` solo corta por cantidad, después de ordenar. |
| `LIMIT` vs `OFFSET` | `LIMIT` = cuántas filas devolver. `OFFSET` = cuántas saltarse antes. |
| `ORDER BY nombre` vs `ORDER BY 2` | Por nombre es claro y seguro. Por número es frágil. |

### Diferencias entre sistemas

| Qué | MySQL | PostgreSQL | SQL Server | Oracle | SQLite |
|---|---|---|---|---|---|
| Limitar filas | `LIMIT` | `LIMIT` o `FETCH` | `TOP` o `FETCH` | `FETCH` (12c+) | `LIMIT` |
| `NULL` en orden `ASC` | Primero | Al final | Primero | Al final | Primero |
| `NULLS FIRST/LAST` | No | Sí | No | Sí | Sí (3.30+) |
| Orden aleatorio | `ORDER BY RAND()` | `ORDER BY RANDOM()` | `ORDER BY NEWID()` | `ORDER BY DBMS_RANDOM.VALUE` | `ORDER BY RANDOM()` |

> [!note]
> Cómo se ordenan los **textos** (mayúsculas, tildes, la "ñ") depende de la *collation* de la base de datos. Dos sistemas pueden dar órdenes distintos con los mismos datos.

---

## 8. Casos especiales

### `LIMIT` sin `ORDER BY`

❌ **Qué está mal**
```sql
SELECT nombre, precio FROM productos LIMIT 3;
```
🤔 **Por qué pasa**: sin orden definido, la base de datos devuelve 3 filas cualesquiera. Hoy salen unas y mañana (tras un cambio en la tabla) pueden salir otras.

✅ **Cómo arreglarlo**: añade siempre un `ORDER BY` que defina qué significa "las primeras".

### Paginar con un orden que tiene empates

❌ **Qué está mal**
```sql
ORDER BY categoria LIMIT 2 OFFSET 2
```
🤔 **Por qué pasa**: tres productos son de Periféricos. Al estar empatados, SQL puede ordenarlos distinto en cada consulta, y una fila puede **repetirse o desaparecer** al cambiar de página.

✅ **Cómo arreglarlo**: añade una columna única como desempate: `ORDER BY categoria, id`.

### Los `NULL` aparecen donde no los esperas

Con `ORDER BY stock ASC`, la Webcam (stock `NULL`) sale **la primera** en MySQL, SQL Server y SQLite, pero **la última** en PostgreSQL y Oracle. Si importa, fuerza el orden con `NULLS LAST` (donde exista) o con una condición:

```sql
ORDER BY CASE WHEN stock IS NULL THEN 1 ELSE 0 END, stock
```

Esa forma pone los `NULL` al final y funciona en todos los sistemas.

### Números guardados como texto

❌ **Qué está mal**: ordenar una columna de tipo texto que contiene números ('1', '10', '2', '9').

🤔 **Por qué pasa**: los textos se ordenan carácter a carácter, así que `'10'` sale antes que `'9'` (porque `'1'` va antes que `'9'`).

“✅ Cómo arreglarlo: guarda los números con tipo numérico, o conviértelos al ordenar con `CAST(columna AS INTEGER)` (en MySQL, `CAST(columna AS SIGNED)`).”

### `ORDER BY` con `DISTINCT`

En algunos sistemas (PostgreSQL, por ejemplo), si usas `SELECT DISTINCT`, las columnas del `ORDER BY` **tienen que estar en el `SELECT`**. Si no, da error, porque no tendría sentido ordenar por algo que se ha descartado.

### Si no hay filas que mostrar

Si `OFFSET` es mayor que el número de filas, o el `WHERE` no encuentra nada, el resultado es una tabla vacía. No es un error.

---

## 9. Resumen

- `ORDER BY` **ordena**; `LIMIT` **recorta**. Van al final de la consulta, en ese orden.
- Sin `ORDER BY`, **no hay orden garantizado**. `LIMIT` sin `ORDER BY` devuelve filas arbitrarias.
- `ASC` (por defecto) = menor a mayor; `DESC` = mayor a menor.
- Con varias columnas, la segunda solo desempata a la primera; cada una tiene su `ASC` o `DESC`.
- `ORDER BY` sí puede usar alias y expresiones (se ejecuta después del `SELECT`).
- `OFFSET` salta filas. Paginación: `OFFSET = (página - 1) × filas_por_página`.
- **Limitar filas cambia según el sistema**: `LIMIT` (MySQL, PostgreSQL, SQLite), `TOP` (SQL Server), `FETCH FIRST` (estándar, Oracle).
- Los `NULL` se ordenan distinto según el sistema; usa `NULLS FIRST/LAST` o un `CASE` si importa.
- Para paginar con seguridad, añade una columna única (`id`) como desempate.