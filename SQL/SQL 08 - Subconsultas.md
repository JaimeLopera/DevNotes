# 08 - Subconsultas

> [!info] ¿Qué es?
> Una **subconsulta** es un `SELECT` escrito **dentro de otra consulta**, entre paréntesis. La consulta de dentro (subconsulta) se resuelve y su resultado lo usa la consulta de fuera (consulta principal). Sirve para responder preguntas que necesitan **dos pasos**: "primero averigua esto, y con ese dato, calcula lo otro".

---

## 1. Antes de empezar

Conviene tener claro:

- `SELECT`, `WHERE` y alias → [[SQL 02 - SELECT]], [[SQL 03 - WHERE]]
- Funciones de agregación (`AVG`, `MAX`, `COUNT`…) → [[SQL 05 - Funciones]]
- `GROUP BY` y `HAVING` → [[SQL 06 - GROUP BY y HAVING]]
- `JOIN` (muchas subconsultas se pueden reescribir como JOIN) → [[SQL 07 - JOIN]]

Tablas de ejemplo de toda la nota:

**`empleados`**

| id | nombre | departamento_id | salario |
|----|--------|-----------------|---------|
| 1 | Ana | 1 | 2200 |
| 2 | Luis | 1 | 1900 |
| 3 | Pablo | 2 | 3100 |
| 4 | Sara | 2 | 2900 |
| 5 | Diego | NULL | 2000 |

**`departamentos`**

| id | nombre | ciudad |
|----|--------|--------|
| 1 | Ventas | Madrid |
| 2 | IT | Sevilla |
| 3 | RRHH | Madrid |

---

## 2. Concepto fundamental

### El problema que resuelve

Imagina esta pregunta: **"¿Qué empleados cobran más que la media?"**

No se puede resolver en un solo paso, porque necesitas **saber primero cuál es la media**:

```sql
-- ❌ No funciona: WHERE no admite funciones de agregación
SELECT nombre FROM empleados WHERE salario > AVG(salario);
```

Con una subconsulta se hace en dos pasos, pero dentro de una sola sentencia:

```sql
SELECT nombre, salario
FROM empleados
WHERE salario > (SELECT AVG(salario) FROM empleados);
```

| nombre | salario |
|--------|---------|
| Pablo | 3100 |
| Sara | 2900 |

La media es 2420. SQL calcula eso primero (subconsulta) y después filtra con ese valor (consulta principal).

### Cómo se lee

De **dentro hacia fuera**: primero se resuelve el paréntesis y luego se usa su resultado.

```
(SELECT AVG(salario) FROM empleados)   →   2420
SELECT nombre FROM empleados WHERE salario > 2420
```

### Qué puede devolver una subconsulta

Esto es lo más importante, porque **determina qué operador puedes usar** con ella:

| Devuelve | Se llama | Ejemplo | Operadores válidos |
|----------|----------|---------|--------------------|
| **Un solo valor** (1 fila, 1 columna) | Subconsulta escalar | `(SELECT AVG(salario) ...)` | `=`, `>`, `<`, `>=`, `<=`, `<>` |
| **Una columna, varias filas** | Subconsulta de lista | `(SELECT id FROM departamentos ...)` | `IN`, `NOT IN`, `ANY`, `ALL` |
| **Varias columnas y filas** | Subconsulta de tabla | `(SELECT ... GROUP BY ...)` | Se usa en `FROM` como una tabla |
| Solo importa si **existen filas** | Subconsulta de existencia | `EXISTS (SELECT 1 ...)` | `EXISTS`, `NOT EXISTS` |

---

## 3. Sintaxis / estructura

```sql
SELECT columnas
FROM tabla
WHERE columna operador (SELECT columna FROM otra_tabla WHERE ...);
```

### Reglas básicas

1. **Siempre entre paréntesis.**
2. La subconsulta **no lleva `;`** dentro: el punto y coma va al final de toda la sentencia.
3. Si se usa con `=`, `>`, `<`…, debe devolver **como máximo un valor**; si devuelve varias filas, da error.
4. Si se usa con `IN`, debe devolver **una sola columna**.
5. Una subconsulta en el `FROM` **necesita un alias** (obligatorio en la mayoría de motores).

### Dónde se puede escribir una subconsulta

| Lugar | Para qué se usa |
|-------|-----------------|
| `WHERE` | Filtrar filas (el uso más común) |
| `HAVING` | Filtrar grupos |
| `FROM` | Usar el resultado como si fuera una tabla (tabla derivada) |
| `SELECT` | Calcular una columna extra para cada fila |
| `INSERT`, `UPDATE`, `DELETE` | Decidir qué datos insertar o qué filas modificar |

---

## 4. Elementos / propiedades / características

### 4.1 Subconsulta escalar (devuelve un valor)

Se usa como si fuera un número o texto cualquiera.

```sql
-- Empleados que cobran más que Ana
SELECT nombre, salario
FROM empleados
WHERE salario > (SELECT salario FROM empleados WHERE nombre = "Ana");
```

> [!warning] Cuidado con que devuelva más de una fila
> Si hubiera dos empleadas llamadas "Ana", la subconsulta devolvería 2 valores y `>` daría error. Para asegurarte de que solo hay uno, usa una clave primaria (`WHERE id = 1`) o una agregación (`MAX`, `MIN`).

> [!note] Comillas en SQL
> En SQL estándar los textos van entre **comillas simples** (`'Ana'`). Aquí se escriben dobles por estilo, pero en PostgreSQL, SQL Server y Oracle solo valen las simples. Si quieres que funcione en cualquier motor, usa siempre `'...'`.

### 4.2 Subconsulta de lista con IN / NOT IN

Devuelve varias filas de **una columna** y se comprueba si un valor está en esa lista.

```sql
-- Empleados que trabajan en departamentos de Madrid
SELECT nombre
FROM empleados
WHERE departamento_id IN (
    SELECT id FROM departamentos WHERE ciudad = 'Madrid'
);
```

| nombre |
|--------|
| Ana |
| Luis |

La subconsulta devuelve `1, 3` (Ventas y RRHH). Solo Ana y Luis tienen `departamento_id` 1 o 3.

```sql
-- Empleados que NO trabajan en departamentos de Madrid
SELECT nombre
FROM empleados
WHERE departamento_id NOT IN (
    SELECT id FROM departamentos WHERE ciudad = 'Madrid'
);
```

> [!warning] La trampa del NOT IN con NULL
> Si la lista de la subconsulta contiene aunque sea **un `NULL`**, `NOT IN` **no devuelve ninguna fila**. Esto pasa porque `x NOT IN (1, NULL)` equivale a `x <> 1 AND x <> NULL`, y comparar con `NULL` nunca es verdadero.
> **Soluciones:**
> - Añadir `WHERE columna IS NOT NULL` dentro de la subconsulta.
> - Usar `NOT EXISTS` (ver 4.4), que no tiene este problema.

### 4.3 ANY y ALL

Comparan un valor con **todos los de una lista** usando un operador.

| Operador | Significado |
|----------|-------------|
| `> ANY (lista)` | Mayor que **al menos uno** de la lista (mayor que el mínimo) |
| `> ALL (lista)` | Mayor que **todos** los de la lista (mayor que el máximo) |
| `= ANY (lista)` | Igual a alguno (equivale a `IN`) |
| `<> ALL (lista)` | Distinto de todos (equivale a `NOT IN`) |

```sql
-- Empleados que cobran más que TODOS los del departamento 1
SELECT nombre, salario
FROM empleados
WHERE salario > ALL (SELECT salario FROM empleados WHERE departamento_id = 1);
```

| nombre | salario |
|--------|---------|
| Pablo | 3100 |
| Sara | 2900 |

> [!note] Disponibilidad
> `ANY` y `ALL` existen en MySQL, PostgreSQL, SQL Server y Oracle. **SQLite no los tiene.** Se pueden sustituir por `MAX` y `MIN`: `> ALL (...)` es igual a `> (SELECT MAX(...))`.

### 4.4 EXISTS y NOT EXISTS

`EXISTS` solo comprueba **si la subconsulta devuelve alguna fila**; no importa qué columnas ni qué valores. Devuelve verdadero o falso.

```sql
-- Departamentos que tienen al menos un empleado
SELECT d.nombre
FROM departamentos d
WHERE EXISTS (
    SELECT 1 FROM empleados e WHERE e.departamento_id = d.id
);
```

| nombre |
|--------|
| Ventas |
| IT |

```sql
-- Departamentos SIN empleados
SELECT d.nombre
FROM departamentos d
WHERE NOT EXISTS (
    SELECT 1 FROM empleados e WHERE e.departamento_id = d.id
);
```

| nombre |
|--------|
| RRHH |

> [!tip] ¿Por qué `SELECT 1`?
> Como `EXISTS` solo mira si hay filas, lo que se selecciona da igual. Por convención se escribe `SELECT 1` para dejar claro que no se usan los datos.

### 4.5 Subconsultas correlacionadas

Hasta ahora las subconsultas eran **independientes**: se calculaban una sola vez. Una subconsulta **correlacionada** usa una columna de la consulta principal, así que **se vuelve a ejecutar para cada fila**.

```sql
-- Empleados que cobran más que la media DE SU DEPARTAMENTO
SELECT e.nombre, e.salario
FROM empleados e
WHERE e.salario > (
    SELECT AVG(e2.salario)
    FROM empleados e2
    WHERE e2.departamento_id = e.departamento_id   -- ← usa la fila "de fuera"
);
```

| nombre | salario |
|--------|---------|
| Ana | 2200 |
| Pablo | 3100 |

Para Ana (depto. 1) calcula la media del depto. 1 (2050); para Pablo (depto. 2), la del 2 (3000). Cada fila tiene su propia comparación.

La referencia a `e.departamento_id` dentro de la subconsulta es lo que la hace **correlacionada**. `EXISTS` casi siempre se usa de forma correlacionada (como en 4.4).

> [!note] Diego no aparece
> Su `departamento_id` es `NULL`, y `e2.departamento_id = NULL` nunca es verdadero, así que su media sale `NULL` y la comparación falla.

### 4.6 Subconsulta en el FROM (tabla derivada)

El resultado de la subconsulta se trata como **una tabla temporal con nombre**. Es obligatorio ponerle alias.

```sql
-- Salario medio por departamento, y luego quedarnos con los que superan 2500
SELECT t.departamento_id, t.media
FROM (
    SELECT departamento_id, AVG(salario) AS media
    FROM empleados
    GROUP BY departamento_id
) AS t
WHERE t.media > 2500;
```

| departamento_id | media |
|-----------------|-------|
| 2 | 3000 |

Útil cuando necesitas **agrupar primero y filtrar o unir después** sobre ese resultado.

### 4.7 Subconsulta en el SELECT (columna calculada)

Añade una columna cuyo valor sale de una subconsulta. Debe devolver **un solo valor por fila**.

```sql
SELECT
    d.nombre,
    (SELECT COUNT(*) FROM empleados e WHERE e.departamento_id = d.id) AS num_empleados
FROM departamentos d;
```

| nombre | num_empleados |
|--------|---------------|
| Ventas | 2 |
| IT | 2 |
| RRHH | 0 |

### 4.8 Subconsulta en el HAVING

Filtra grupos comparando con un valor calculado.

```sql
-- Departamentos cuya media supera la media global
SELECT departamento_id, AVG(salario) AS media
FROM empleados
GROUP BY departamento_id
HAVING AVG(salario) > (SELECT AVG(salario) FROM empleados);
```

### 4.9 Subconsultas con INSERT, UPDATE y DELETE

Se verán a fondo en sus notas, pero la idea es la misma:

```sql
-- Borrar empleados de departamentos de Sevilla
DELETE FROM empleados
WHERE departamento_id IN (SELECT id FROM departamentos WHERE ciudad = 'Sevilla');
```

> [!warning] MySQL y la misma tabla
> En MySQL no puedes modificar una tabla y, en la misma sentencia, leerla en una subconsulta directa (error 1093). Se soluciona envolviendo la subconsulta en una tabla derivada con alias.

### 4.10 Subconsultas con varias columnas (de fila)

Se pueden comparar **varias columnas a la vez** con una subconsulta.

```sql
-- Empleado con el salario máximo de cada departamento
SELECT nombre, departamento_id, salario
FROM empleados
WHERE (departamento_id, salario) IN (
    SELECT departamento_id, MAX(salario)
    FROM empleados
    GROUP BY departamento_id
);
```

> [!note] Disponibilidad
> Funciona en MySQL, PostgreSQL y Oracle. SQL Server **no** admite esta sintaxis (se reescribe con `JOIN` o `EXISTS`).

### 4.11 CTE: la alternativa más legible (WITH)

Cuando una subconsulta es larga o se repite, se puede dar nombre con `WITH` (*Common Table Expression*). Hace lo mismo que una tabla derivada, pero **el código se lee de arriba abajo**.

```sql
WITH medias AS (
    SELECT departamento_id, AVG(salario) AS media
    FROM empleados
    GROUP BY departamento_id
)
SELECT e.nombre, e.salario, m.media
FROM empleados e
JOIN medias m ON e.departamento_id = m.departamento_id
WHERE e.salario > m.media;
```

> [!note] Disponibilidad
> `WITH` existe en PostgreSQL, SQL Server, Oracle, SQLite y **MySQL 8.0+**. No está en MySQL 5.7 ni anteriores.

---

## 5. Ejemplos prácticos

### Ejemplo básico: comparar con un valor calculado

```sql
-- Empleado mejor pagado
SELECT nombre, salario
FROM empleados
WHERE salario = (SELECT MAX(salario) FROM empleados);
```

### Ejemplo habitual: IN con subconsulta

```sql
-- Empleados de departamentos de Madrid
SELECT nombre
FROM empleados
WHERE departamento_id IN (SELECT id FROM departamentos WHERE ciudad = 'Madrid');
```

### Ejemplo completo: correlacionada con EXISTS

```sql
-- Clientes que han hecho al menos un pedido de más de 100
SELECT c.nombre
FROM clientes c
WHERE EXISTS (
    SELECT 1
    FROM pedidos p
    WHERE p.cliente_id = c.id AND p.total > 100
);
```

---

## 6. Buenas prácticas

1. **Pon siempre paréntesis y sangra la subconsulta**: se lee mucho mejor.
2. **Usa alias claros** en cada nivel (`e`, `e2`, `d`) para saber de qué tabla es cada columna.
3. **Elige el operador según lo que devuelve**: `=` para un valor; `IN` para una lista.
4. **Prefiere `NOT EXISTS` a `NOT IN`** si la columna puede contener `NULL`.
5. **Valora un `JOIN`** cuando solo necesitas columnas de ambas tablas: suele ser más claro y rápido.
6. **Usa `WITH` (CTE)** cuando la consulta sea larga, tenga varios niveles o repita la misma subconsulta.
7. **Para existencia, usa `EXISTS`** en vez de `COUNT(*) > 0`: se detiene en cuanto encuentra una fila.
8. **No abuses de las correlacionadas** con tablas enormes: se ejecutan una vez por fila.

---

## 7. Diferencias importantes

### Subconsulta vs JOIN

| | Subconsulta | `JOIN` |
|---|-------------|--------|
| Devuelve columnas de la 2ª tabla | ❌ Normalmente no | ✅ Sí |
| Ideal para | Filtrar ("está en", "existe", "mayor que la media") | Mostrar datos de varias tablas juntos |
| Riesgo de duplicar filas | No | Sí (relaciones uno a muchos) |
| Legibilidad | Más fácil de leer como pregunta en dos pasos | Mejor con muchas tablas |

Estas dos consultas dan el mismo resultado:

```sql
-- Con subconsulta
SELECT nombre FROM empleados
WHERE departamento_id IN (SELECT id FROM departamentos WHERE ciudad = 'Madrid');

-- Con JOIN
SELECT e.nombre FROM empleados e
JOIN departamentos d ON e.departamento_id = d.id
WHERE d.ciudad = 'Madrid';
```

### Independiente vs correlacionada

| | Independiente | Correlacionada |
|---|---------------|----------------|
| Usa columnas de la consulta de fuera | ❌ No | ✅ Sí |
| Se ejecuta | Una vez | Una vez por cada fila |
| Se puede probar sola | ✅ Sí | ❌ No (depende de la externa) |

### IN vs EXISTS

| | `IN` | `EXISTS` |
|---|------|----------|
| Compara | Un valor con una lista | Solo si hay filas |
| Con `NULL` | `NOT IN` falla si hay `NULL` | `NOT EXISTS` es seguro |
| Subconsulta | Devuelve una columna | Devuelve cualquier cosa (`SELECT 1`) |

---

## 8. Casos especiales

### 8.1 La subconsulta escalar no devuelve filas

Si no encuentra nada, el resultado es **`NULL`**, y comparar con `NULL` no es verdadero para nadie. La consulta principal devolverá cero filas.

### 8.2 La subconsulta escalar devuelve más de una fila

Da **error** (por ejemplo, "Subquery returns more than 1 row"). Solución: usar `IN`, `ANY`/`ALL`, o forzar un valor con `MAX`, `MIN` o `LIMIT 1`.

### 8.3 ORDER BY y LIMIT dentro de una subconsulta

`ORDER BY` solo tiene sentido dentro si lo acompaña `LIMIT` (o `TOP`/`FETCH`). Y algunos motores no permiten `LIMIT` en subconsultas con `IN` (MySQL: error "doesn't yet support LIMIT in IN subquery"). Se resuelve con una tabla derivada.

### 8.4 Varios niveles de anidación

Se pueden anidar subconsultas dentro de otras, pero a partir de dos niveles se vuelve ilegible. En ese caso, usa `WITH`.

### 8.5 Subconsulta que devuelve una lista vacía

`IN (lista vacía)` es falso para todos; `NOT IN (lista vacía)` es verdadero para todos (devuelve todas las filas).

### 8.6 Segundo valor más alto (patrón clásico)

```sql
-- Segundo salario más alto
SELECT MAX(salario)
FROM empleados
WHERE salario < (SELECT MAX(salario) FROM empleados);
```

---

## 9. Resumen

- Una **subconsulta** es un `SELECT` dentro de otro, siempre **entre paréntesis**.
- Se resuelve **de dentro hacia fuera**.
- El operador depende de lo que devuelve: **un valor** → `=`, `>`, `<`; **una lista** → `IN`, `ANY`, `ALL`; **solo existencia** → `EXISTS`.
- **Escalar**: 1 fila y 1 columna. **De lista**: 1 columna, varias filas. **Tabla derivada**: en el `FROM`, con alias obligatorio.
- **Correlacionada**: usa columnas de la consulta externa y se ejecuta **una vez por fila**.
- Pueden ir en `WHERE`, `HAVING`, `FROM`, `SELECT` y en `INSERT`/`UPDATE`/`DELETE`.
- **`NOT IN` falla si la lista tiene `NULL`**: usa `NOT EXISTS` o filtra los `NULL`.
- Muchas subconsultas se pueden reescribir con **`JOIN`**.
- Para consultas largas o repetidas, usa **`WITH` (CTE)**.
- Cada motor tiene sus límites (`ANY`/`ALL` en SQLite, columnas múltiples en SQL Server, `LIMIT` en MySQL).