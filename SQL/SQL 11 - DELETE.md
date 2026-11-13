# 11 - DELETE

> [!info] ¿Qué es?
> `DELETE` es la sentencia de SQL que **borra filas** de una tabla. Solo elimina datos: la tabla, sus columnas y sus restricciones siguen existiendo. Es una de las cuatro operaciones básicas (CRUD) junto con `INSERT`, `SELECT` y `UPDATE`, y es la que **más cuidado exige**, porque lo que borras no se recupera fácilmente.

---

## 1. Antes de empezar

Lo que conviene tener claro antes de borrar nada:

- `DELETE` borra **filas completas**, nunca columnas sueltas. Si quieres vaciar solo un valor de una columna, eso es un `UPDATE ... SET columna = NULL` (ver [[SQL 10 - UPDATE]]).
- Se apoya totalmente en `WHERE` para decidir qué filas borrar (ver [[SQL 03 - WHERE]]). Sin `WHERE`, **borra todas las filas**.
- No confundir tres cosas parecidas pero distintas:
  - `DELETE` → borra filas (todas o las que elijas).
  - `TRUNCATE` → vacía la tabla entera de golpe.
  - `DROP TABLE` → elimina la tabla completa, con su estructura.
- Si la base de datos tiene **claves foráneas** (ver [[SQL 13 - Claves y restricciones]]), borrar una fila puede estar bloqueado o puede borrar más cosas de las que esperas.

> [!tip] Regla de oro
> Antes de ejecutar un `DELETE`, escribe primero un `SELECT` con **el mismo `WHERE`**. Si el `SELECT` devuelve exactamente las filas que quieres borrar, entonces cambias `SELECT *` por `DELETE`.

---

## 2. Concepto fundamental

`DELETE` recorre la tabla, **comprueba cada fila contra la condición del `WHERE`** y elimina las que la cumplen.

- Si la condición es verdadera → la fila se borra.
- Si es falsa o desconocida (`NULL`) → la fila se queda.
- Si no hay `WHERE` → todas las filas cumplen y se borran todas.

Cuando termina, la base de datos te dice **cuántas filas ha borrado**. Ese número es tu primera comprobación: si esperabas borrar 3 filas y dice 3.000, para y revisa (si estás dentro de una transacción, haz `ROLLBACK`).

Detalles importantes:

- La tabla queda **vacía pero viva**: conserva columnas, índices, restricciones y permisos.
- Los contadores automáticos (`AUTO_INCREMENT`, `SERIAL`, `IDENTITY`) **no se reinician**. Si borras la fila con `id = 10`, la siguiente fila nueva no reutiliza el 10.
- Es una operación de **datos** (DML), por eso se puede deshacer dentro de una transacción (ver [[SQL 17 - Transacciones]]).

---

## 3. Sintaxis / estructura

```sql
DELETE FROM nombre_tabla
WHERE condicion;
```

Partes:

| Parte | Qué hace |
|---|---|
| `DELETE FROM nombre_tabla` | Indica de qué tabla se borran las filas |
| `WHERE condicion` | Filtra qué filas se borran (opcional pero casi siempre necesario) |
| `;` | Termina la sentencia |

Con alias (útil cuando hay subconsultas o varias tablas):

```sql
DELETE FROM pedidos AS p
WHERE p.estado = 'cancelado';
```

> [!warning] Ojo con el alias
> Algunos motores no permiten alias en el `DELETE FROM` (por ejemplo, SQL Server lo admite con una sintaxis distinta, y MySQL solo en ciertas formas). Si te da error, prueba sin alias.

---

## 4. Elementos / propiedades / características

### 4.1 `WHERE`: elegir qué se borra

Funciona igual que en un `SELECT`: comparaciones, `AND`, `OR`, `IN`, `BETWEEN`, `LIKE`, `IS NULL`...

```sql
DELETE FROM clientes
WHERE pais = 'España' AND activo = 0;
```

> [!warning] `NULL` no se compara con `=`
> Para borrar filas con valor vacío usa `IS NULL`, no `= NULL`. Este `DELETE` no borra nada (y no da error, lo que lo hace traicionero):
> ```sql
> DELETE FROM clientes WHERE telefono = NULL;    -- ❌ no borra nada
> DELETE FROM clientes WHERE telefono IS NULL;   -- ✅
> ```

### 4.2 Subconsultas en el `WHERE`

Sirven para borrar según datos de **otra tabla** (ver [[SQL 08 - Subconsultas]]). Esta forma funciona en todos los motores:

```sql
DELETE FROM pedidos
WHERE cliente_id IN (SELECT id FROM clientes WHERE activo = 0);
```

### 4.3 `DELETE` con `JOIN`

Otra forma de borrar usando otra tabla (ver [[SQL 07 - JOIN]]). **Aquí cada motor tiene su propia sintaxis**, no es estándar:

```sql
-- MySQL / MariaDB / SQL Server
DELETE p
FROM pedidos AS p
JOIN clientes AS c ON c.id = p.cliente_id
WHERE c.activo = 0;

-- PostgreSQL
DELETE FROM pedidos AS p
USING clientes AS c
WHERE c.id = p.cliente_id AND c.activo = false;
```

Oracle y SQLite no tienen `DELETE` con `JOIN`: allí se usa la subconsulta de 4.2. Por eso **la subconsulta es la opción más portable**.

### 4.4 Limitar cuántas filas se borran

El estándar no tiene un `LIMIT` para `DELETE`, y cada motor lo resuelve distinto:

```sql
-- MySQL / MariaDB
DELETE FROM logs ORDER BY fecha LIMIT 1000;

-- SQL Server
DELETE TOP (1000) FROM logs;

-- PostgreSQL / SQLite / portable
DELETE FROM logs
WHERE id IN (SELECT id FROM logs ORDER BY fecha LIMIT 1000);
```

### 4.5 Ver qué se ha borrado: `RETURNING` y `OUTPUT`

Algunos motores te devuelven las filas borradas en la misma sentencia:

```sql
-- PostgreSQL, SQLite (3.35+), MariaDB
DELETE FROM clientes WHERE id = 7 RETURNING *;

-- SQL Server
DELETE FROM clientes OUTPUT deleted.* WHERE id = 7;
```

MySQL no lo tiene. Oracle usa `RETURNING ... INTO` dentro de código PL/SQL.

### 4.6 Claves foráneas y `ON DELETE`

Si otra tabla depende de la fila que quieres borrar, la **clave foránea** decide qué pasa. Se define al crear la tabla:

```sql
FOREIGN KEY (cliente_id) REFERENCES clientes(id) ON DELETE CASCADE
```

| Opción | Qué ocurre al borrar la fila "padre" |
|---|---|
| `RESTRICT` / `NO ACTION` | Da **error** y no borra (lo más seguro, y el comportamiento por defecto) |
| `CASCADE` | Borra también automáticamente las filas "hijas" |
| `SET NULL` | Deja las filas hijas pero pone la clave foránea a `NULL` |
| `SET DEFAULT` | Pone el valor por defecto de la columna en las filas hijas |

> [!danger] `CASCADE` borra en cadena
> Con `ON DELETE CASCADE`, un solo `DELETE` puede eliminar miles de filas en otras tablas sin avisarte. Úsalo solo cuando tenga sentido que "si desaparece el padre, desaparecen los hijos".

---

## 5. Ejemplos prácticos

Tablas de ejemplo: `clientes (id, nombre, email, activo)` y `pedidos (id, cliente_id, estado, fecha)`.

**Ejemplo básico: borrar una fila concreta**

```sql
DELETE FROM clientes
WHERE id = 5;
```

Borra solo el cliente con `id = 5`. Filtrar por la clave primaria es la forma más segura.

**Ejemplo habitual: borrar según una condición, con comprobación previa**

```sql
-- 1) Comprobar qué se va a borrar
SELECT * FROM pedidos
WHERE estado = 'cancelado' AND fecha < '2024-01-01';

-- 2) Borrar con exactamente la misma condición
DELETE FROM pedidos
WHERE estado = 'cancelado' AND fecha < '2024-01-01';
```

**Ejemplo completo: borrar dentro de una transacción**

```sql
BEGIN;  -- en MySQL también vale START TRANSACTION;

DELETE FROM pedidos
WHERE cliente_id IN (SELECT id FROM clientes WHERE activo = 0);

DELETE FROM clientes
WHERE activo = 0;

-- Revisa cuántas filas dijo que borró cada sentencia.
-- Si todo es correcto:
COMMIT;
-- Si algo no cuadra:
-- ROLLBACK;
```

Primero se borran los pedidos (hijos) y después los clientes (padres), así no choca con la clave foránea. Si algo falla, `ROLLBACK` deja todo como estaba.

---

## 6. Buenas prácticas

- **Haz siempre un `SELECT` antes** con el mismo `WHERE`.
- **Usa transacciones** para borrados importantes: así puedes hacer `ROLLBACK` si algo sale mal.
- **Filtra por clave primaria** siempre que puedas (`WHERE id = ...`).
- **Haz copia de seguridad** antes de borrados grandes o en producción.
- **Escribe el `WHERE` primero** y la palabra `DELETE` al final cuando estés escribiendo la sentencia a mano; así, si se ejecuta sin querer, no borra todo.
- **Revisa el número de filas afectadas** y compáralo con lo que esperabas.
- **Piensa si de verdad quieres borrar**: a veces es mejor marcar la fila como inactiva (ver "borrado lógico" en el punto 8).
- **Borra en lotes** si son millones de filas, para no bloquear la tabla ni llenar el registro de transacciones.

---

## 7. Diferencias importantes

### `DELETE` vs `TRUNCATE` vs `DROP`

| | `DELETE` | `TRUNCATE` | `DROP TABLE` |
|---|---|---|---|
| Qué elimina | Las filas que elijas (o todas) | Todas las filas | La tabla entera |
| ¿Admite `WHERE`? | Sí | No | No |
| ¿La estructura sobrevive? | Sí | Sí | **No** |
| Reinicia contadores automáticos | No | Sí (en la mayoría de motores) | No aplica |
| Velocidad en tablas grandes | Más lento | Muy rápido | Muy rápido |
| ¿Se puede deshacer? | Sí, en una transacción | Depende del motor | Depende del motor |
| Activa triggers de borrado | Sí | Normalmente no | No |

> [!warning] `TRUNCATE` según el motor
> - **MySQL / Oracle:** hace *commit* implícito, así que **no se puede deshacer**.
> - **PostgreSQL / SQL Server:** sí se puede deshacer dentro de una transacción.
> - **SQLite:** no existe `TRUNCATE`; se usa `DELETE FROM tabla;`.
> - En PostgreSQL, para reiniciar los contadores hay que añadir `RESTART IDENTITY`.

### Diferencias de sintaxis entre motores (resumen)

| Necesidad | MySQL | PostgreSQL | SQL Server | SQLite | Oracle |
|---|---|---|---|---|---|
| Borrar con `JOIN` | `DELETE p FROM ... JOIN` | `DELETE ... USING` | `DELETE p FROM ... JOIN` | Subconsulta | Subconsulta |
| Limitar filas | `LIMIT` | Subconsulta | `TOP (n)` | Subconsulta | `FETCH FIRST` / `ROWNUM` |
| Ver lo borrado | No | `RETURNING` | `OUTPUT` | `RETURNING` | `RETURNING INTO` |

---

## 8. Casos especiales

### Borrar todas las filas

```sql
DELETE FROM tabla;
```

Vacía la tabla fila a fila. Es lento en tablas grandes; para vaciarla entera, `TRUNCATE` es más rápido (con las precauciones del punto 7).

### Borrado lógico (*soft delete*)

En lugar de borrar la fila de verdad, se marca como borrada:

```sql
UPDATE clientes SET eliminado = 1, fecha_eliminacion = CURRENT_TIMESTAMP WHERE id = 5;
```

Y en las consultas se filtra con `WHERE eliminado = 0`. Es lo habitual cuando necesitas **historial**, **poder recuperar** datos o **conservar relaciones** con otras tablas.

### Borrar duplicados dejando uno

```sql
DELETE FROM clientes
WHERE id NOT IN (
    SELECT MIN(id) FROM clientes GROUP BY email
);
```

Conserva el `id` más bajo de cada `email` y borra el resto.

> [!warning] Error 1093 en MySQL
> MySQL no deja usar en una subconsulta la misma tabla que estás borrando. La solución es envolverla en otra subconsulta con alias:
> ```sql
> DELETE FROM clientes
> WHERE id NOT IN (
>     SELECT id FROM (SELECT MIN(id) AS id FROM clientes GROUP BY email) AS t
> );
> ```

### Borrar en lotes (tablas enormes)

Borrar millones de filas de una vez puede bloquear la tabla durante mucho tiempo. Se hace en trozos repitiendo hasta que no queden filas:

```sql
-- MySQL: repetir hasta que diga "0 rows affected"
DELETE FROM logs WHERE fecha < '2023-01-01' LIMIT 10000;
```

### Modo seguro de MySQL Workbench

Si recibes el error 1175 (*safe update mode*), significa que tu `DELETE` no usa una clave en el `WHERE`. Es una protección, no un fallo: normalmente lo correcto es mejorar el `WHERE`, no desactivar el modo seguro.

### Triggers

Si la tabla tiene un *trigger* de borrado (ver [[SQL 19 - Procedimientos y triggers]]), se ejecutará automáticamente con cada fila borrada. Puede registrar el borrado, impedirlo o borrar datos en otras tablas.

### Permisos

Para borrar necesitas el permiso `DELETE` sobre la tabla (y `SELECT` si tu `WHERE` lee columnas). En producción es habitual que las cuentas de la aplicación **no** tengan este permiso en tablas críticas.

---

## 9. Resumen

- `DELETE FROM tabla WHERE condicion;` borra **filas** que cumplen la condición; la tabla y su estructura siguen existiendo.
- **Sin `WHERE` se borra todo.** Haz siempre un `SELECT` previo con la misma condición.
- Para valores vacíos usa `IS NULL`, nunca `= NULL`.
- Con otra tabla de por medio, la **subconsulta con `IN`** funciona en todos los motores; `DELETE ... JOIN` y `USING` dependen del motor.
- Las **claves foráneas** deciden qué pasa con las filas relacionadas: `RESTRICT` (error), `CASCADE` (borra en cadena), `SET NULL`.
- `DELETE` borra filas y se puede deshacer en una transacción; `TRUNCATE` vacía la tabla entera; `DROP` elimina la tabla.
- No reinicia contadores automáticos.
- Para tablas enormes, borra **en lotes**; si necesitas conservar historial, usa **borrado lógico**.
- Usa **transacciones**, revisa el número de **filas afectadas** y ten **copia de seguridad** antes de borrar en serio.