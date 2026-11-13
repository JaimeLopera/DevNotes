# 05 - Errores de SQL

> [!info] ¿Qué es?
> Los errores de SQL más frecuentes (sobre todo en **MySQL/MariaDB**, con notas para PostgreSQL), con el **mensaje literal**, qué significa, sus causas y cómo solucionarlo. El código de error (`1064`, `1452`…) permite buscarlo rápido.

---

## 1. Antes de empezar: cómo investigar

- Ejecuta la consulta **sola** en un cliente (phpMyAdmin, DBeaver, terminal) para ver el error sin el resto del código.
- Si la consulta se construye desde PHP/Node/Python, **imprímela** antes de ejecutarla.
- Empieza simplificando: quita condiciones y `JOIN` hasta que funcione y añádelos de nuevo uno a uno.

---

## 2. Sintaxis

### 2.1 `ERROR 1064 (42000): You have an error in your SQL syntax ... near '...'`

**Qué significa:** la consulta está mal escrita. El fragmento después de `near` indica **dónde** se perdió el analizador (el error suele estar justo antes).

**Causas típicas:**
- Coma de más antes de `FROM` (`SELECT a, b, FROM ...`).
- Falta una coma entre columnas.
- Comillas mal cerradas.
- Una palabra reservada usada como nombre (`order`, `group`, `desc`, `key`).
- Orden de cláusulas incorrecto (`WHERE` después de `GROUP BY`).

**Solución:**

```sql
-- Orden correcto de las cláusulas
SELECT ... FROM ... JOIN ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT ...;

-- Palabras reservadas como nombre: entre comillas invertidas (MySQL) o dobles (PostgreSQL)
SELECT `order` FROM pedidos;
```

### 2.2 Comillas: texto vs. nombres

| Para | MySQL | PostgreSQL |
|---|---|---|
| Textos | `'texto'` | `'texto'` |
| Nombres de columnas/tablas | `` `nombre` `` | `"nombre"` |

En PostgreSQL, `WHERE nombre = "Ana"` falla: `"Ana"` se interpreta como una **columna**.

---

## 3. Tablas y columnas

### 3.1 `ERROR 1146 (42S02): Table 'db.tabla' doesn't exist`

**Causas típicas:** nombre mal escrito, base de datos equivocada (`USE mi_db;`), o mayúsculas en servidores Linux (`Usuarios` ≠ `usuarios`).

### 3.2 `ERROR 1054 (42S22): Unknown column 'x' in 'field list'`

**Causas típicas:**
- Columna mal escrita o que no existe.
- Un texto sin comillas: `WHERE nombre = Ana` (busca una columna llamada `Ana`).
- Usar un alias de `SELECT` en el `WHERE` (el `WHERE` se evalúa antes).

**Solución:**

```sql
WHERE nombre = 'Ana'

-- El alias no existe todavía en WHERE: repite la expresión o usa HAVING / subconsulta
SELECT precio * 1.21 AS con_iva FROM productos WHERE precio * 1.21 > 100;
```

### 3.3 `ERROR 1052 (23000): Column 'id' in field list is ambiguous`

**Qué significa:** en un `JOIN`, dos tablas tienen una columna con ese nombre.

**Solución:** prefija con la tabla o su alias: `SELECT p.id, c.nombre FROM pedidos p JOIN clientes c ...`.

### 3.4 `ERROR 1050 (42S01): Table 'x' already exists`

**Solución:** `CREATE TABLE IF NOT EXISTS ...` o bórrala antes si es intencionado.

---

## 4. Restricciones y claves

### 4.1 `ERROR 1062 (23000): Duplicate entry 'x' for key 'PRIMARY'` (o `'email'`)

**Qué significa:** intentas insertar un valor que ya existe en una columna `PRIMARY KEY` o `UNIQUE`.

**Solución:** comprueba antes si existe, deja que el `id` lo genere `AUTO_INCREMENT` (no lo insertes a mano), o usa *upsert* (`ON DUPLICATE KEY UPDATE`).

### 4.2 `ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails`

**Qué significa:** insertas un valor de clave foránea que **no existe** en la tabla padre (por ejemplo, un pedido con un `cliente_id` inexistente).

**Solución:** inserta primero el registro padre, o corrige el id.

### 4.3 `ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails`

**Qué significa:** intentas borrar un registro que otros referencian (un cliente con pedidos).

**Solución:** borra antes los hijos, o define la clave foránea con `ON DELETE CASCADE` / `ON DELETE SET NULL` si ese es el comportamiento deseado.

### 4.4 `ERROR 1048 (23000): Column 'x' cannot be null`

**Solución:** envía un valor para esa columna o dale un `DEFAULT`.

### 4.5 `ERROR 1406 (22001): Data too long for column 'x'`

**Solución:** recorta el valor o amplía la columna (`VARCHAR(255)`, `TEXT`).

### 4.6 `ERROR 1366: Incorrect string value: '\xF0\x9F...'`

**Qué significa:** intentas guardar emojis u otros caracteres de 4 bytes en una columna `utf8` (que en MySQL solo admite 3 bytes).

**Solución:** usa `utf8mb4` en la tabla, la columna y la conexión.

---

## 5. Agrupaciones

### 5.1 `ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause ...`

**Qué significa:** seleccionas una columna que no está en el `GROUP BY` ni dentro de una función de agregado.

**Solución:**

```sql
-- ✘
SELECT cliente_id, nombre, SUM(total) FROM pedidos GROUP BY cliente_id;

-- ✔
SELECT cliente_id, SUM(total) FROM pedidos GROUP BY cliente_id;
-- o añade la columna al GROUP BY si depende del grupo
```

### 5.2 `ERROR 1111 (HY000): Invalid use of group function`

**Qué significa:** usas `SUM`, `COUNT`, `AVG`… en el `WHERE`.

**Solución:** las condiciones sobre agregados van en `HAVING`.

```sql
SELECT cliente_id, SUM(total) FROM pedidos GROUP BY cliente_id HAVING SUM(total) > 100;
```

Más en [[SQL 06 - GROUP BY y HAVING|GROUP BY y HAVING]].

---

## 6. Resultados inesperados (sin error)

### 6.1 `WHERE columna = NULL` no devuelve nada

**Solución:** `WHERE columna IS NULL`.

### 6.2 El `JOIN` multiplica las filas

**Causa típica:** la condición `ON` es incompleta o la relación es uno-a-muchos y se suman importes repetidos.

**Solución:** revisa el `ON`; agrega en una subconsulta antes de unir.

### 6.3 El `LEFT JOIN` se comporta como `INNER JOIN`

**Causa típica:** una condición sobre la tabla de la derecha en el `WHERE` elimina las filas con `NULL`.

**Solución:** mueve esa condición al `ON`.

### 6.4 `UPDATE`/`DELETE` afectó a todas las filas

**Causa típica:** se olvidó el `WHERE`.

**Prevención:** ejecuta antes un `SELECT` con el mismo `WHERE`, trabaja dentro de una transacción y, en MySQL, activa `SET SQL_SAFE_UPDATES = 1;`.

### 6.5 La consulta es muy lenta

**Causas típicas:** falta un índice en las columnas del `WHERE`/`JOIN`, funciones sobre columnas indexadas (`WHERE YEAR(fecha) = 2026`), `LIKE '%texto'` con comodín al principio.

**Solución:** analiza con `EXPLAIN SELECT ...` y añade índices. Más en [[SQL 14 - Índices|Índices]].

---

## 7. Conexión y permisos

### 7.1 `ERROR 1045 (28000): Access denied for user 'x'@'localhost'`

**Solución:** revisa usuario y contraseña; comprueba los permisos con `SHOW GRANTS FOR 'x'@'localhost';`.

### 7.2 `ERROR 2002 (HY000): Can't connect to local MySQL server`

**Solución:** arranca el servidor MySQL (XAMPP, servicio de Windows, `docker start ...`) y comprueba host y puerto.

### 7.3 `ERROR 1213 (40001): Deadlock found when trying to get lock`

**Qué significa:** dos transacciones se bloquean mutuamente.

**Solución:** reintenta la transacción; mantén las transacciones cortas y accede a las tablas siempre en el mismo orden.

---

## 8. Resumen

- `1064` sintaxis → mira justo antes de `near`; orden de cláusulas; comas; palabras reservadas.
- `1054` columna desconocida → textos entre comillas simples; alias no válidos en `WHERE`.
- `1062` duplicado → `UNIQUE`/`PRIMARY KEY`; no insertes ids a mano.
- `1452` / `1451` → clave foránea: el padre no existe / tiene hijos.
- `1055` / `1111` → reglas de `GROUP BY`; agregados en `HAVING`.
- `= NULL` nunca funciona: usa `IS NULL`.
- Antes de `UPDATE`/`DELETE`, prueba el `WHERE` con un `SELECT`.
