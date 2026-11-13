# 06 - GROUP BY y HAVING

> [!info] ¿Qué es?
> **GROUP BY** junta las filas que tienen el mismo valor en una o varias columnas y las convierte en **un grupo**. Sobre cada grupo se pueden calcular resultados con funciones de agregación (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`). **HAVING** filtra esos grupos una vez creados, igual que `WHERE` filtra filas.

---

## 1. Antes de empezar

Para entender esta nota conviene tener claro:

- Cómo funciona un `SELECT` básico → [[SQL 02 - SELECT]]
- Cómo se filtran filas con `WHERE` → [[SQL 03 - WHERE]]
- Qué son las funciones de agregación → [[SQL 05 - Funciones]]

Las tablas de ejemplo de toda la nota:

**`empleados`**

| id | nombre | departamento | ciudad | salario |
|----|--------|--------------|--------|---------|
| 1 | Ana | Ventas | Madrid | 2200 |
| 2 | Luis | Ventas | Madrid | 1900 |
| 3 | Marta | Ventas | Sevilla | 2500 |
| 4 | Pablo | IT | Madrid | 3100 |
| 5 | Sara | IT | Madrid | 2900 |
| 6 | Diego | RRHH | Sevilla | 2000 |
| 7 | Elena | RRHH | NULL | 2100 |

**`pedidos`**

| id | cliente_id | total | fecha |
|----|-----------|-------|-------|
| 1 | 10 | 50.00 | 2026-01-05 |
| 2 | 10 | 80.00 | 2026-02-11 |
| 3 | 11 | 20.00 | 2026-02-15 |
| 4 | 12 | 200.00 | 2026-03-01 |
| 5 | 12 | 150.00 | 2026-03-20 |
| 6 | 12 | 90.00 | 2026-04-02 |

---

## 2. Concepto fundamental

### El problema que resuelve

Una función de agregación sin `GROUP BY` calcula **un único resultado para toda la tabla**:

```sql
SELECT AVG(salario) FROM empleados;
-- 2385.71  (un solo número para todos)
```

Pero normalmente queremos **un resultado por categoría**: el salario medio *de cada departamento*, el total gastado *por cada cliente*, los pedidos *de cada mes*. Eso es justo lo que hace `GROUP BY`.

```sql
SELECT departamento, AVG(salario) AS salario_medio
FROM empleados
GROUP BY departamento;
```

| departamento | salario_medio |
|--------------|---------------|
| Ventas | 2200.00 |
| IT | 3000.00 |
| RRHH | 2050.00 |

### Cómo piensa SQL esta consulta

Imagina que SQL reparte las filas en montones, uno por cada valor distinto de `departamento`, y luego calcula la función sobre cada montón:

```
Ventas → Ana, Luis, Marta   → AVG(2200, 1900, 2500) = 2200
IT     → Pablo, Sara        → AVG(3100, 2900)       = 3000
RRHH   → Diego, Elena       → AVG(2000, 2100)       = 2050
```

**Resultado: una fila por grupo**, no una fila por empleado.

### Qué es HAVING

`HAVING` se escribe después de `GROUP BY` y sirve para **descartar grupos** según el resultado de una agregación.

```sql
SELECT departamento, AVG(salario) AS salario_medio
FROM empleados
GROUP BY departamento
HAVING AVG(salario) > 2100;
```

| departamento | salario_medio |
|--------------|---------------|
| Ventas | 2200.00 |
| IT | 3000.00 |

RRHH desaparece porque su media (2050) no supera 2100.

---

## 3. Sintaxis / estructura

```sql
SELECT   columnas_de_agrupacion, funciones_de_agregacion
FROM     tabla
WHERE    condicion_sobre_filas        -- opcional
GROUP BY columnas_de_agrupacion
HAVING   condicion_sobre_grupos       -- opcional
ORDER BY ...                          -- opcional
LIMIT    ...;                         -- opcional
```

### Orden de escritura (el que debes teclear)

`SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT`

### Orden de ejecución (el que sigue SQL por dentro)

Este orden es **la clave** para entender por qué algunas cosas funcionan y otras dan error:

| Paso | Cláusula | Qué hace |
|------|----------|----------|
| 1 | `FROM` | Elige la tabla (y los `JOIN`) |
| 2 | `WHERE` | Descarta **filas** |
| 3 | `GROUP BY` | Forma los **grupos** |
| 4 | Agregaciones | Calcula `COUNT`, `SUM`, etc. por grupo |
| 5 | `HAVING` | Descarta **grupos** |
| 6 | `SELECT` | Elige las columnas finales y pone los alias |
| 7 | `ORDER BY` | Ordena el resultado |
| 8 | `LIMIT` | Recorta el número de filas |

> [!tip] Consecuencia práctica
> `WHERE` se ejecuta **antes** de que existan los grupos, así que no puede usar funciones de agregación. `HAVING` se ejecuta **después**, así que sí puede.

---

## 4. Elementos / propiedades / características

### 4.1 Funciones de agregación más usadas con GROUP BY

| Función | Qué calcula en cada grupo |
|---------|---------------------------|
| `COUNT(*)` | Número de filas del grupo |
| `COUNT(columna)` | Número de filas donde esa columna **no es NULL** |
| `COUNT(DISTINCT columna)` | Número de valores distintos no nulos |
| `SUM(columna)` | Suma de los valores |
| `AVG(columna)` | Media de los valores |
| `MIN(columna)` | Valor mínimo |
| `MAX(columna)` | Valor máximo |

```sql
SELECT
    departamento,
    COUNT(*)      AS empleados,
    SUM(salario)  AS masa_salarial,
    MIN(salario)  AS minimo,
    MAX(salario)  AS maximo
FROM empleados
GROUP BY departamento;
```

| departamento | empleados | masa_salarial | minimo | maximo |
|--------------|-----------|---------------|--------|--------|
| Ventas | 3 | 6600 | 1900 | 2500 |
| IT | 2 | 6000 | 2900 | 3100 |
| RRHH | 2 | 4100 | 2000 | 2100 |

### 4.2 La regla de oro del SELECT

> [!warning] Regla fundamental
> Con `GROUP BY`, cada columna del `SELECT` debe ser **una de estas dos cosas**:
> 1. Una columna que aparece en el `GROUP BY`.
> 2. Una función de agregación.

Esto es lógico: si agrupas por departamento, SQL no sabe qué nombre de empleado mostrar para el grupo "Ventas" (¿Ana? ¿Luis? ¿Marta?).

```sql
-- ❌ nombre no está en el GROUP BY ni dentro de una agregación
SELECT departamento, nombre, AVG(salario)
FROM empleados
GROUP BY departamento;
```

La mayoría de bases de datos (PostgreSQL, SQL Server, Oracle, SQLite recientes y MySQL con `ONLY_FULL_GROUP_BY`) dan **error** aquí.

### 4.3 Agrupar por varias columnas

Se agrupa por **la combinación** de valores. Cada combinación distinta es un grupo.

```sql
SELECT departamento, ciudad, COUNT(*) AS empleados
FROM empleados
GROUP BY departamento, ciudad;
```

| departamento | ciudad | empleados |
|--------------|--------|-----------|
| Ventas | Madrid | 2 |
| Ventas | Sevilla | 1 |
| IT | Madrid | 2 |
| RRHH | Sevilla | 1 |
| RRHH | NULL | 1 |

El orden de las columnas en el `GROUP BY` no cambia los grupos, solo el orden en que pueden salir las filas (y eso no está garantizado sin `ORDER BY`).

### 4.4 Agrupar por una expresión

No hace falta agrupar por una columna "pelada": puedes agrupar por el resultado de una expresión o función.

```sql
-- Pedidos por mes (sintaxis de MySQL; en otros motores cambia la función de fecha)
SELECT
    MONTH(fecha) AS mes,
    COUNT(*)     AS pedidos,
    SUM(total)   AS facturado
FROM pedidos
GROUP BY MONTH(fecha);
```

| mes | pedidos | facturado |
|-----|---------|-----------|
| 1 | 1 | 50.00 |
| 2 | 2 | 100.00 |
| 3 | 2 | 350.00 |
| 4 | 1 | 90.00 |

> [!note] Funciones de fecha según el motor
> `MONTH(fecha)` es MySQL/SQL Server. En PostgreSQL se usa `EXTRACT(MONTH FROM fecha)` y en SQLite `strftime('%m', fecha)`.

### 4.5 NULL y GROUP BY

Los valores `NULL` **se agrupan juntos** en un único grupo (aunque en SQL `NULL = NULL` no sea verdadero, `GROUP BY` los trata como iguales). Lo viste en 4.3: Elena (ciudad `NULL`) forma su propio grupo.

### 4.6 WHERE frente a HAVING

| | `WHERE` | `HAVING` |
|---|---------|----------|
| Filtra | **Filas** individuales | **Grupos** completos |
| Se ejecuta | **Antes** de agrupar | **Después** de agrupar |
| Puede usar agregaciones | ❌ No | ✅ Sí |
| Puede usar columnas normales | ✅ Sí | Solo las del `GROUP BY` (o dentro de agregaciones) |
| Sin `GROUP BY` | Funciona | Funciona, pero trata toda la tabla como un grupo (poco habitual) |

```sql
-- ❌ WHERE no puede usar funciones de agregación
SELECT departamento, AVG(salario)
FROM empleados
WHERE AVG(salario) > 2100
GROUP BY departamento;

-- ✅ Para filtrar por el resultado de una agregación se usa HAVING
SELECT departamento, AVG(salario)
FROM empleados
GROUP BY departamento
HAVING AVG(salario) > 2100;
```

### 4.7 Usar WHERE y HAVING a la vez

Cada uno hace su trabajo en su momento:

```sql
SELECT departamento, AVG(salario) AS salario_medio
FROM empleados
WHERE ciudad = 'Madrid'          -- 1º: solo empleados de Madrid
GROUP BY departamento            -- 2º: agrupa esos empleados
HAVING AVG(salario) > 2500;      -- 3º: se queda con departamentos de media alta
```

| departamento | salario_medio |
|--------------|---------------|
| IT | 3000.00 |

Ventas en Madrid tiene media 2050 (Ana y Luis), así que no pasa el `HAVING`.

> [!tip] Regla práctica
> Si la condición se puede comprobar **mirando una sola fila**, ponla en `WHERE` (es más eficiente porque reduce las filas antes de agrupar). Si necesita **un resultado calculado del grupo**, va en `HAVING`.

### 4.8 Alias en GROUP BY, HAVING y ORDER BY

Por el orden de ejecución, `SELECT` (donde se crean los alias) llega **después** de `GROUP BY` y `HAVING`, pero **antes** de `ORDER BY`. En la práctica:

| Cláusula | ¿Acepta alias del SELECT? |
|----------|---------------------------|
| `ORDER BY` | ✅ Sí, en todos los motores |
| `GROUP BY` | Depende: MySQL, PostgreSQL y SQLite sí; SQL Server y Oracle no |
| `HAVING` | Depende: MySQL y SQLite sí; PostgreSQL, SQL Server y Oracle no |

Para que tu código funcione en **cualquier** base de datos, repite la expresión en `GROUP BY` y `HAVING`, y usa el alias solo en `ORDER BY`:

```sql
SELECT cliente_id, SUM(total) AS gasto
FROM pedidos
GROUP BY cliente_id
HAVING SUM(total) > 100       -- expresión repetida (portable)
ORDER BY gasto DESC;          -- alias (válido en todos)
```

---

## 5. Ejemplos prácticos

### Ejemplo básico: contar filas por grupo

```sql
SELECT departamento, COUNT(*) AS empleados
FROM empleados
GROUP BY departamento;
```

| departamento | empleados |
|--------------|-----------|
| Ventas | 3 |
| IT | 2 |
| RRHH | 2 |

### Ejemplo habitual: ranking de clientes por gasto

Clientes que han gastado más de 100 en total, de mayor a menor:

```sql
SELECT
    cliente_id,
    COUNT(*)   AS num_pedidos,
    SUM(total) AS gasto_total
FROM pedidos
GROUP BY cliente_id
HAVING SUM(total) > 100
ORDER BY gasto_total DESC;
```

| cliente_id | num_pedidos | gasto_total |
|-----------|-------------|-------------|
| 12 | 3 | 440.00 |
| 10 | 2 | 130.00 |

El cliente 11 (gasto 20) queda fuera por el `HAVING`.

### Ejemplo completo: detectar duplicados

Uno de los usos más prácticos de `GROUP BY ... HAVING COUNT(*) > 1` es encontrar valores repetidos:

```sql
-- ¿Hay emails repetidos en la tabla usuarios?
SELECT email, COUNT(*) AS veces
FROM usuarios
GROUP BY email
HAVING COUNT(*) > 1;
```

Si no devuelve filas, no hay duplicados. Es la comprobación estándar antes de añadir una restricción `UNIQUE`.

---

## 6. Buenas prácticas

1. **Filtra primero con `WHERE`** todo lo que puedas; `HAVING` solo para condiciones sobre agregaciones.
2. **Escribe en el `SELECT` solo columnas del `GROUP BY` o agregaciones**, aunque tu motor te deje hacer otra cosa.
3. **Pon alias claros** a las agregaciones (`AS gasto_total`) para que el resultado se entienda.
4. **Usa `ORDER BY` siempre que el orden importe**: `GROUP BY` no garantiza ningún orden en el resultado (aunque algunos motores lo devuelvan ordenado).
5. **Repite la expresión en `HAVING`** en vez de usar el alias si quieres código portable entre motores.
6. **Elige bien entre `COUNT(*)` y `COUNT(columna)`**: el primero cuenta filas; el segundo ignora los `NULL`.

---

## 7. Diferencias importantes

### `COUNT(*)` vs `COUNT(columna)` vs `COUNT(DISTINCT columna)`

```sql
SELECT
    COUNT(*)                AS filas,
    COUNT(ciudad)           AS con_ciudad,
    COUNT(DISTINCT ciudad)  AS ciudades_distintas
FROM empleados;
```

| filas | con_ciudad | ciudades_distintas |
|-------|-----------|--------------------|
| 7 | 6 | 2 |

### `GROUP BY` vs `DISTINCT`

Sin agregaciones, ambos pueden devolver lo mismo, pero con intenciones distintas:

```sql
SELECT DISTINCT departamento FROM empleados;
SELECT departamento FROM empleados GROUP BY departamento;
```

- `DISTINCT` → "quítame los repetidos".
- `GROUP BY` → "forma grupos para calcular algo sobre ellos".

Si no necesitas calcular nada por grupo, `DISTINCT` expresa mejor la intención.

### `GROUP BY` vs funciones de ventana

`GROUP BY` **colapsa** las filas del grupo en una sola. Las funciones de ventana (`OVER (PARTITION BY ...)`) calculan la agregación **sin colapsar**, manteniendo todas las filas. Se estudian más adelante.

---

## 8. Casos especiales

### 8.1 Agregación sin GROUP BY

Si el `SELECT` solo tiene agregaciones, toda la tabla es un único grupo:

```sql
SELECT COUNT(*), AVG(salario), MAX(salario) FROM empleados;
```

Devuelve **siempre una fila**, incluso si la tabla está vacía (`COUNT` da 0 y las demás dan `NULL`).

### 8.2 Mezclar columnas normales y agregaciones sin GROUP BY

```sql
-- ❌ Error: nombre no es agregación y no hay GROUP BY
SELECT nombre, MAX(salario) FROM empleados;
```

Para obtener *quién* cobra el máximo, no se resuelve con `GROUP BY`, sino con una subconsulta o ordenando:

```sql
SELECT nombre, salario
FROM empleados
ORDER BY salario DESC
LIMIT 1;
```

### 8.3 Extensión de MySQL (cuidado)

Si `ONLY_FULL_GROUP_BY` está desactivado, MySQL **permite** columnas fuera del `GROUP BY` y elige un valor cualquiera del grupo. No da error, pero el resultado es impredecible. Evítalo: en otros motores falla y en MySQL puede dar datos engañosos.

### 8.4 HAVING sin GROUP BY

Es válido pero raro: toda la tabla cuenta como un solo grupo y `HAVING` decide si se devuelve esa única fila.

```sql
SELECT COUNT(*) AS total FROM empleados HAVING COUNT(*) > 5;
```

### 8.5 GROUP BY con JOIN

Muy habitual: primero se unen las tablas (`FROM`/`JOIN`) y luego se agrupa el resultado.

```sql
SELECT d.nombre, COUNT(e.id) AS empleados
FROM departamentos d
LEFT JOIN empleados e ON e.departamento_id = d.id
GROUP BY d.id, d.nombre;
```

> [!tip] Cuenta sobre la tabla del lado "muchos"
> Con `LEFT JOIN`, usa `COUNT(e.id)` y no `COUNT(*)`: así un departamento sin empleados da 0 en vez de 1.

### 8.6 Extensiones por motor (para saber que existen)

| Extensión | Para qué sirve | Disponible en |
|-----------|----------------|---------------|
| `GROUP BY ... WITH ROLLUP` | Añade filas de subtotales y total general | MySQL |
| `GROUP BY ROLLUP(...)`, `CUBE(...)`, `GROUPING SETS(...)` | Subtotales y combinaciones múltiples | PostgreSQL, SQL Server, Oracle |
| `GROUP_CONCAT(col)` | Une los valores del grupo en un texto | MySQL, SQLite |
| `STRING_AGG(col, ',')` | Igual que el anterior | PostgreSQL, SQL Server |

> [!warning] Obsoleto / legado
> En MySQL antiguo (anterior a 8.0) `GROUP BY` ordenaba implícitamente el resultado y existía `GROUP BY col ASC|DESC`. Ya no es así: usa siempre `ORDER BY` para ordenar.

---

## 9. Resumen

- **`GROUP BY`** agrupa filas con el mismo valor y devuelve **una fila por grupo**.
- Las agregaciones (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) se calculan **dentro de cada grupo**.
- En el `SELECT` solo pueden ir **columnas del `GROUP BY` o agregaciones**.
- Se puede agrupar por **varias columnas** (la combinación forma el grupo) o por **expresiones**.
- Los `NULL` se agrupan **juntos** en un solo grupo.
- **`WHERE`** filtra **filas antes** de agrupar y no admite agregaciones.
- **`HAVING`** filtra **grupos después** de agrupar y sí admite agregaciones.
- Orden de ejecución: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`.
- Los alias del `SELECT` se pueden usar en `ORDER BY` en todos los motores; en `GROUP BY` y `HAVING` depende del motor.
- Patrón típico para duplicados: `GROUP BY col HAVING COUNT(*) > 1`.