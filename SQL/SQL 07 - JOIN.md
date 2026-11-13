# 07 - JOIN

> [!info] ¿Qué es?
> **JOIN** sirve para **combinar filas de dos o más tablas** en un solo resultado, uniéndolas mediante una columna que tienen en común (normalmente una clave primaria y una clave foránea). Es lo que permite que una base de datos relacional guarde cada dato una sola vez y aun así puedas consultarlo todo junto.

---

## 1. Antes de empezar

Conviene tener claro:

- Cómo funciona `SELECT` y los alias → [[SQL 02 - SELECT]]
- Cómo filtrar con `WHERE` → [[SQL 03 - WHERE]]
- Cómo agrupar con `GROUP BY` → [[SQL 06 - GROUP BY y HAVING]]
- Qué son clave primaria (PK) y clave foránea (FK): la PK identifica cada fila de su tabla; la FK es una columna que guarda la PK de **otra** tabla para enlazarlas.

Tablas de ejemplo de toda la nota:

**`departamentos`**

| id | nombre |
|----|--------|
| 1 | Ventas |
| 2 | IT |
| 3 | RRHH |
| 4 | Marketing |

**`empleados`**

| id | nombre | departamento_id | jefe_id |
|----|--------|-----------------|---------|
| 1 | Ana | 1 | NULL |
| 2 | Luis | 1 | 1 |
| 3 | Pablo | 2 | 1 |
| 4 | Sara | 2 | 3 |
| 5 | Diego | NULL | 1 |

Fíjate en dos detalles que usaremos mucho:

- **Marketing** (id 4) no tiene ningún empleado.
- **Diego** no tiene departamento (`departamento_id` es `NULL`).

---

## 2. Concepto fundamental

### Por qué existen los JOIN

Si guardáramos el nombre del departamento dentro de cada empleado, lo repetiríamos muchas veces y sería fácil equivocarse. Por eso se guarda **solo el id** del departamento en `empleados.departamento_id`.

Pero cuando queremos ver "empleado + nombre de su departamento", necesitamos **juntar las dos tablas**. Eso es un `JOIN`.

### Cómo funciona por dentro

SQL toma cada fila de la primera tabla y busca en la segunda las filas que cumplen la **condición de unión** (`ON`). Cada pareja que encaja se convierte en una fila del resultado.

```
empleados.departamento_id  ──►  departamentos.id
        (FK)                         (PK)
```

### Los tipos de JOIN en una frase

| Tipo | Qué devuelve |
|------|--------------|
| `INNER JOIN` | Solo las filas que tienen pareja en **ambas** tablas |
| `LEFT JOIN` | **Todas** las de la tabla izquierda + sus parejas (si no hay, `NULL`) |
| `RIGHT JOIN` | **Todas** las de la tabla derecha + sus parejas (si no hay, `NULL`) |
| `FULL JOIN` | **Todas** las de ambas tablas, con o sin pareja |
| `CROSS JOIN` | **Todas las combinaciones** posibles (sin condición) |
| Self join | Una tabla unida **consigo misma** |

---

## 3. Sintaxis / estructura

```sql
SELECT columnas
FROM tabla_a
[tipo] JOIN tabla_b
    ON tabla_a.columna = tabla_b.columna
WHERE ...
```

Ejemplo mínimo:

```sql
SELECT e.nombre, d.nombre AS departamento
FROM empleados e
INNER JOIN departamentos d
    ON e.departamento_id = d.id;
```

### Piezas importantes

- **`FROM tabla_a`**: la tabla "izquierda" (la primera que escribes).
- **`JOIN tabla_b`**: la tabla "derecha" (la que unes).
- **`ON`**: la condición que dice qué filas se emparejan. Casi siempre es `FK = PK`.
- **Alias de tabla** (`e`, `d`): nombres cortos para no escribir el nombre completo cada vez. Son casi obligatorios cuando ambas tablas tienen columnas con el mismo nombre (aquí, `nombre` e `id`).

> [!tip] Prefijo de columna
> Cuando dos tablas tienen una columna con el mismo nombre, hay que indicar de cuál hablas: `e.nombre` o `d.nombre`. Si no, SQL da error de **columna ambigua**.

### Palabras opcionales

- `INNER JOIN` se puede escribir solo `JOIN` (es lo mismo).
- `LEFT OUTER JOIN` se puede escribir `LEFT JOIN` (el `OUTER` sobra).
- `RIGHT OUTER JOIN` → `RIGHT JOIN`; `FULL OUTER JOIN` → `FULL JOIN`.

---

## 4. Elementos / propiedades / características

### 4.1 INNER JOIN

Devuelve **solo las filas que encuentran pareja en las dos tablas**. Lo que no encaja, desaparece.

```sql
SELECT e.nombre AS empleado, d.nombre AS departamento
FROM empleados e
INNER JOIN departamentos d ON e.departamento_id = d.id;
```

| empleado | departamento |
|----------|--------------|
| Ana | Ventas |
| Luis | Ventas |
| Pablo | IT |
| Sara | IT |

- **Diego** no aparece: su `departamento_id` es `NULL`, no encuentra pareja.
- **RRHH** y **Marketing** no aparecen: ningún empleado apunta a ellos.

### 4.2 LEFT JOIN

Devuelve **todas las filas de la tabla izquierda** (la del `FROM`), aunque no tengan pareja. Donde no hay pareja, las columnas de la derecha salen como `NULL`.

```sql
SELECT e.nombre AS empleado, d.nombre AS departamento
FROM empleados e
LEFT JOIN departamentos d ON e.departamento_id = d.id;
```

| empleado | departamento |
|----------|--------------|
| Ana | Ventas |
| Luis | Ventas |
| Pablo | IT |
| Sara | IT |
| Diego | NULL |

Ahora **Diego sí aparece**, con `NULL` en el departamento.

> [!tip] Es el JOIN que más se usa después del INNER
> Se usa cuando quieres "todo lo de la tabla principal, tenga o no datos relacionados".

### 4.3 RIGHT JOIN

Es el espejo del `LEFT JOIN`: conserva **todas las filas de la tabla derecha**.

```sql
SELECT e.nombre AS empleado, d.nombre AS departamento
FROM empleados e
RIGHT JOIN departamentos d ON e.departamento_id = d.id;
```

| empleado | departamento |
|----------|--------------|
| Ana | Ventas |
| Luis | Ventas |
| Pablo | IT |
| Sara | IT |
| NULL | RRHH |
| NULL | Marketing |

Aparecen RRHH y Marketing aunque no tengan empleados. Diego no, porque es de la tabla izquierda y no tiene pareja.

> [!note] En la práctica casi nadie usa RIGHT JOIN
> Cualquier `RIGHT JOIN` se puede reescribir como `LEFT JOIN` cambiando el orden de las tablas, y leer siempre "de izquierda a derecha" resulta más claro.

### 4.4 FULL JOIN

Devuelve **todo de ambas tablas**: las parejas, más las filas sin pareja de cada lado (con `NULL` en el lado que falta).

```sql
SELECT e.nombre AS empleado, d.nombre AS departamento
FROM empleados e
FULL JOIN departamentos d ON e.departamento_id = d.id;
```

| empleado | departamento |
|----------|--------------|
| Ana | Ventas |
| Luis | Ventas |
| Pablo | IT |
| Sara | IT |
| Diego | NULL |
| NULL | RRHH |
| NULL | Marketing |

> [!warning] MySQL y MariaDB no tienen FULL JOIN
> PostgreSQL, SQL Server, Oracle y SQLite (3.39+) sí. En MySQL se simula uniendo un `LEFT JOIN` y un `RIGHT JOIN` con `UNION`:
> ```sql
> SELECT e.nombre, d.nombre
> FROM empleados e LEFT JOIN departamentos d ON e.departamento_id = d.id
> UNION
> SELECT e.nombre, d.nombre
> FROM empleados e RIGHT JOIN departamentos d ON e.departamento_id = d.id;
> ```

### 4.5 CROSS JOIN

Devuelve **todas las combinaciones posibles** entre las filas de ambas tablas (producto cartesiano). **No lleva `ON`.**

```sql
SELECT e.nombre, d.nombre AS departamento
FROM empleados e
CROSS JOIN departamentos d;
```

Con 5 empleados y 4 departamentos salen **5 × 4 = 20 filas**. Se usa poco: para generar combinaciones (tallas × colores, fechas × productos…).

> [!warning] Cuidado con los productos cartesianos accidentales
> Si te olvidas del `ON` (o de la condición en el `WHERE` en la sintaxis antigua), obtienes todas las combinaciones sin querer. Con tablas grandes puede generar millones de filas.

### 4.6 SELF JOIN (unir una tabla consigo misma)

No es una palabra clave nueva: es hacer un `JOIN` de una tabla **con ella misma**, usando **dos alias distintos**. Sirve para jerarquías, como empleado → jefe.

```sql
SELECT e.nombre AS empleado, j.nombre AS jefe
FROM empleados e
LEFT JOIN empleados j ON e.jefe_id = j.id;
```

| empleado | jefe |
|----------|------|
| Ana | NULL |
| Luis | Ana |
| Pablo | Ana |
| Sara | Pablo |
| Diego | Ana |

Aquí `e` es "el empleado" y `j` es "el jefe": son la misma tabla leída dos veces. Se usa `LEFT JOIN` para que Ana (sin jefe) no desaparezca.

### 4.7 ON frente a USING y NATURAL JOIN

| Forma | Cómo se escribe | Comentario |
|-------|-----------------|------------|
| `ON` | `ON e.departamento_id = d.id` | La más general y clara. Sirve siempre. |
| `USING` | `USING (departamento_id)` | Atajo cuando **ambas tablas tienen una columna con exactamente el mismo nombre**. La columna sale una sola vez. |
| `NATURAL JOIN` | `NATURAL JOIN tabla` | Une automáticamente por **todas** las columnas con el mismo nombre. |

```sql
-- Si pedidos.cliente_id y clientes.cliente_id se llaman igual:
SELECT *
FROM pedidos
JOIN clientes USING (cliente_id);
```

> [!warning] Evita NATURAL JOIN
> Si alguien añade más tarde una columna con el mismo nombre en ambas tablas, la unión cambia **sin avisar**. Es una fuente de errores muy difíciles de ver. `USING` es aceptable; `ON` es lo más seguro.

### 4.8 Unir más de dos tablas

Se encadenan varios `JOIN`; cada uno une con lo ya construido.

```sql
SELECT p.id AS pedido, c.nombre AS cliente, pr.nombre AS producto
FROM pedidos p
JOIN clientes c       ON p.cliente_id  = c.id
JOIN lineas_pedido l  ON l.pedido_id   = p.id
JOIN productos pr     ON l.producto_id = pr.id;
```

Cada `ON` explica cómo se engancha esa tabla nueva con alguna de las anteriores.

### 4.9 Condiciones extra: ON frente a WHERE

Aquí hay una diferencia **clave** en los `LEFT JOIN`.

- **Condición en `ON`**: se aplica **mientras se buscan las parejas**. Las filas de la izquierda se conservan igualmente.
- **Condición en `WHERE`**: se aplica **después de unir**, eliminando filas del resultado (incluidas las que tenían `NULL`).

```sql
-- Todos los departamentos, y solo se emparejan los empleados de IT... (condición en ON)
SELECT d.nombre AS departamento, e.nombre AS empleado
FROM departamentos d
LEFT JOIN empleados e
    ON e.departamento_id = d.id AND e.nombre = 'Pablo';
```

| departamento | empleado |
|--------------|----------|
| Ventas | NULL |
| IT | Pablo |
| RRHH | NULL |
| Marketing | NULL |

```sql
-- La misma condición en WHERE
SELECT d.nombre AS departamento, e.nombre AS empleado
FROM departamentos d
LEFT JOIN empleados e ON e.departamento_id = d.id
WHERE e.nombre = 'Pablo';
```

| departamento | empleado |
|--------------|----------|
| IT | Pablo |

Con `WHERE`, las filas con `NULL` se descartan y el `LEFT JOIN` **se comporta como un `INNER JOIN`**.

> [!tip] Regla práctica
> En un `LEFT JOIN`, las condiciones sobre la tabla **derecha** van en el `ON` si quieres conservar las filas sin pareja. Las condiciones sobre la tabla **izquierda** pueden ir en el `WHERE` sin problema.

### 4.10 Encontrar filas SIN pareja (anti-join)

Un patrón muy útil: `LEFT JOIN` + `WHERE columna_derecha IS NULL`.

```sql
-- Departamentos que no tienen ningún empleado
SELECT d.nombre
FROM departamentos d
LEFT JOIN empleados e ON e.departamento_id = d.id
WHERE e.id IS NULL;
```

| nombre |
|--------|
| RRHH |
| Marketing |

### 4.11 JOIN con GROUP BY

Muy habitual: unir y luego agrupar para contar o sumar.

```sql
-- Número de empleados por departamento (incluyendo los que tienen 0)
SELECT d.nombre AS departamento, COUNT(e.id) AS empleados
FROM departamentos d
LEFT JOIN empleados e ON e.departamento_id = d.id
GROUP BY d.id, d.nombre;
```

| departamento | empleados |
|--------------|-----------|
| Ventas | 2 |
| IT | 2 |
| RRHH | 0 |
| Marketing | 0 |

> [!tip] COUNT(e.id), no COUNT(*)
> `COUNT(*)` contaría la fila de relleno con `NULL` y daría 1 para RRHH. `COUNT(e.id)` ignora los `NULL` y da el 0 correcto.

---

## 5. Ejemplos prácticos

### Ejemplo básico: INNER JOIN

```sql
SELECT e.nombre, d.nombre AS departamento
FROM empleados e
JOIN departamentos d ON e.departamento_id = d.id;
```

### Ejemplo habitual: todos los clientes y su gasto total (aunque no hayan comprado)

```sql
SELECT c.nombre, COALESCE(SUM(p.total), 0) AS gasto_total
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id
GROUP BY c.id, c.nombre
ORDER BY gasto_total DESC;
```

`COALESCE` cambia el `NULL` (cliente sin pedidos) por 0.

### Ejemplo completo: empleado, jefe y departamento a la vez

```sql
SELECT
    e.nombre AS empleado,
    j.nombre AS jefe,
    d.nombre AS departamento
FROM empleados e
LEFT JOIN empleados j      ON e.jefe_id = j.id
LEFT JOIN departamentos d  ON e.departamento_id = d.id;
```

Mezcla un self join y un join normal, ambos como `LEFT JOIN` para no perder a nadie.

---

## 6. Buenas prácticas

1. **Usa siempre alias cortos y significativos** (`e` para empleados, `d` para departamentos).
2. **Prefija todas las columnas con su alias**, aunque no sea obligatorio: el código se entiende mejor.
3. **Escribe `JOIN ... ON` explícito**, no la sintaxis antigua con comas y `WHERE`.
4. **Elige el tipo de JOIN según la pregunta**: ¿quiero solo lo que encaja (`INNER`) o también lo que no (`LEFT`)?
5. **Prefiere `LEFT JOIN` a `RIGHT JOIN`**: se lee siempre de izquierda a derecha.
6. **Une por claves indexadas** (PK y FK): las uniones son mucho más rápidas.
7. **Selecciona solo las columnas que necesitas**, no `SELECT *`, sobre todo con varias tablas (las columnas repetidas confunden).
8. **Evita `NATURAL JOIN`**; usa `ON` o, como mucho, `USING`.

---

## 7. Diferencias importantes

### INNER JOIN vs LEFT JOIN

| | `INNER JOIN` | `LEFT JOIN` |
|---|--------------|-------------|
| Filas sin pareja de la izquierda | ❌ Se pierden | ✅ Se conservan (con `NULL`) |
| Úsalo cuando | Solo importan los datos relacionados | Quieres la lista completa de la tabla principal |

### JOIN moderno vs sintaxis antigua

> [!warning] Obsoleto / legado
> La forma antigua une las tablas con comas y condiciones en el `WHERE`:
> ```sql
> SELECT e.nombre, d.nombre
> FROM empleados e, departamentos d
> WHERE e.departamento_id = d.id;
> ```
> Funciona, pero mezcla la condición de unión con los filtros y, si olvidas la condición, genera un producto cartesiano sin avisar. **Usa siempre `JOIN ... ON`.**

### JOIN vs UNION

- `JOIN` añade **columnas** (junta tablas "en horizontal").
- `UNION` añade **filas** (apila resultados "en vertical") y exige que ambos `SELECT` tengan las mismas columnas.

---

## 8. Casos especiales

### 8.1 NULL en la columna de unión

`NULL` **nunca es igual a nada**, ni siquiera a otro `NULL`. Por eso Diego (`departamento_id = NULL`) no empareja con ningún departamento: la condición `NULL = d.id` nunca es verdadera.

### 8.2 Relaciones uno a muchos duplican filas

Si unes un departamento con sus 2 empleados, el departamento aparece **2 veces** (una por cada empleado). Es normal, pero afecta a las agregaciones: si sumas una columna del lado "uno", se suma repetida.

### 8.3 Relaciones muchos a muchos

Se resuelven con una **tabla intermedia** (por ejemplo `alumnos_asignaturas`) y se unen **tres tablas**:

```sql
SELECT a.nombre AS alumno, s.nombre AS asignatura
FROM alumnos a
JOIN alumnos_asignaturas aa ON aa.alumno_id = a.id
JOIN asignaturas s          ON aa.asignatura_id = s.id;
```

### 8.4 Columnas con el mismo nombre en el resultado

Con `SELECT *` en un `JOIN`, las columnas repetidas (`id`, `nombre`) salen duplicadas y confusas. Usa alias de columna (`AS`) para distinguirlas.

### 8.5 Unir por condiciones distintas de la igualdad

`ON` acepta cualquier condición, no solo `=`: rangos, desigualdades, etc. (se llama *non-equi join*).

```sql
-- Asignar a cada venta su tramo de comisión según el importe
SELECT v.id, v.importe, t.nombre AS tramo
FROM ventas v
JOIN tramos t ON v.importe BETWEEN t.minimo AND t.maximo;
```

### 8.6 UPDATE y DELETE con JOIN

La sintaxis **varía mucho según el motor** (MySQL, PostgreSQL y SQL Server lo hacen distinto). Se verá en las notas de `UPDATE` y `DELETE`.

---

## 9. Resumen

- **`JOIN`** combina filas de varias tablas mediante una condición (`ON`), normalmente `FK = PK`.
- **`INNER JOIN`**: solo las filas con pareja en ambas tablas.
- **`LEFT JOIN`**: todas las de la izquierda; sin pareja → `NULL` a la derecha.
- **`RIGHT JOIN`**: lo contrario; casi siempre se reescribe como `LEFT JOIN`.
- **`FULL JOIN`**: todas las filas de ambas tablas (no existe en MySQL; se simula con `UNION`).
- **`CROSS JOIN`**: todas las combinaciones, sin `ON`.
- **Self join**: una tabla unida consigo misma con dos alias (jerarquías).
- Usa **alias** y **prefija las columnas** para evitar ambigüedades.
- En un `LEFT JOIN`, las condiciones sobre la tabla derecha van en `ON`; en `WHERE` convierten el join en un `INNER`.
- **Anti-join**: `LEFT JOIN ... WHERE derecha.id IS NULL` encuentra filas sin pareja.
- Con `LEFT JOIN` + `GROUP BY`, cuenta con `COUNT(columna_derecha)`, no con `COUNT(*)`.
- `NULL` nunca empareja con nada.
- Evita la sintaxis antigua con comas y `NATURAL JOIN`.