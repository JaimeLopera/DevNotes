# 20 - Referencia rápida

> [!info] ¿Qué es?
> Es la **chuleta de SQL**: un resumen de todo lo importante de la carpeta para consultarlo en segundos, sin leer las notas completas. Si necesitas entender algo a fondo, cada apartado indica la nota donde se explica.

---

## 1. Orden de una consulta

### Cómo se escribe

```sql
SELECT   columnas
FROM     tabla
JOIN     otra_tabla ON condicion
WHERE    filtro_de_filas
GROUP BY columnas_de_agrupacion
HAVING   filtro_de_grupos
ORDER BY columnas
LIMIT    n;
```

### Cómo se ejecuta (el orden real)

| Paso | Cláusula | Qué hace |
|---|---|---|
| 1 | `FROM` / `JOIN` | Elige y une las tablas |
| 2 | `WHERE` | Filtra filas |
| 3 | `GROUP BY` | Agrupa filas |
| 4 | `HAVING` | Filtra grupos |
| 5 | `SELECT` | Elige columnas y calcula |
| 6 | `ORDER BY` | Ordena |
| 7 | `LIMIT` | Recorta el resultado |

> [!tip] Consecuencia práctica
> `WHERE` se ejecuta **antes** que `SELECT`, así que **no puedes usar un alias** de `SELECT` dentro del `WHERE`. `ORDER BY` sí puede usarlo.

---

## 2. SELECT (ver [[SQL 02 - SELECT]])

```sql
SELECT * FROM clientes;                       -- todas las columnas
SELECT nombre, email FROM clientes;           -- columnas concretas
SELECT nombre AS cliente FROM clientes;       -- alias
SELECT DISTINCT ciudad FROM clientes;         -- sin repetidos
SELECT precio * 1.21 AS con_iva FROM productos;  -- cálculo
```

---

## 3. WHERE (ver [[SQL 03 - WHERE]])

| Operador | Significado | Ejemplo |
|---|---|---|
| `=` `<>` (o `!=`) | Igual / distinto | `ciudad = 'Madrid'` |
| `<` `<=` `>` `>=` | Comparaciones | `precio >= 10` |
| `AND` `OR` `NOT` | Lógicos | `a > 1 AND b < 5` |
| `BETWEEN a AND b` | En un rango (incluye extremos) | `precio BETWEEN 10 AND 20` |
| `IN (...)` | Dentro de una lista | `ciudad IN ('Madrid', 'Sevilla')` |
| `LIKE` | Patrón de texto | `nombre LIKE 'A%'` |
| `IS NULL` / `IS NOT NULL` | Comprobar nulos | `email IS NULL` |

Comodines de `LIKE`: `%` = cualquier cantidad de caracteres, `_` = exactamente uno.

> [!warning] Recuerda
> Con `NULL` **nunca** uses `= NULL`: usa `IS NULL`. Cualquier comparación con `NULL` da "desconocido", no verdadero.

---

## 4. ORDER BY y LIMIT (ver [[SQL 04 - ORDER BY y LIMIT]])

```sql
ORDER BY precio ASC              -- de menor a mayor (por defecto)
ORDER BY precio DESC             -- de mayor a menor
ORDER BY ciudad, nombre DESC     -- por varias columnas

LIMIT 10                         -- primeras 10 filas
LIMIT 10 OFFSET 20               -- salta 20 y devuelve 10 (paginación)
```

| Sistema | Equivalente de `LIMIT` |
|---|---|
| MySQL, PostgreSQL, SQLite | `LIMIT n` |
| SQL Server | `SELECT TOP n ...` o `OFFSET ... FETCH` |
| Oracle (12c+) y estándar | `FETCH FIRST n ROWS ONLY` |

---

## 5. Funciones (ver [[SQL 05 - Funciones]])

### De agregación (resumen de muchas filas en un valor)

| Función | Qué hace |
|---|---|
| `COUNT(*)` | Cuenta filas (incluye las que tienen `NULL`) |
| `COUNT(col)` | Cuenta valores **no nulos** de esa columna |
| `SUM(col)` | Suma |
| `AVG(col)` | Media |
| `MIN(col)` / `MAX(col)` | Mínimo / máximo |

### De texto, números, fechas y condicionales

| Tipo | Funciones habituales |
|---|---|
| Texto | `UPPER`, `LOWER`, `LENGTH`, `TRIM`, `SUBSTRING`, `REPLACE`, `CONCAT` |
| Números | `ROUND`, `CEIL`, `FLOOR`, `ABS`, `MOD` |
| Fechas | `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `EXTRACT(YEAR FROM f)` |
| Condicionales | `CASE`, `COALESCE`, `NULLIF`, `CAST` |

```sql
CASE
    WHEN precio < 10 THEN 'barato'
    WHEN precio < 50 THEN 'medio'
    ELSE 'caro'
END

COALESCE(telefono, 'sin teléfono')   -- primer valor que no sea NULL
CAST(precio AS INT)                  -- cambiar de tipo
```

> [!warning] Cambian según el sistema
> Concatenar texto: `CONCAT(a, b)` (casi todos), `a || b` (estándar, PostgreSQL, Oracle, SQLite), `a + b` (SQL Server). La fecha actual y las funciones de fecha también varían.

---

## 6. GROUP BY y HAVING (ver [[SQL 06 - GROUP BY y HAVING]])

```sql
SELECT ciudad, COUNT(*) AS total
FROM clientes
GROUP BY ciudad
HAVING COUNT(*) > 5;
```

| | `WHERE` | `HAVING` |
|---|---|---|
| Filtra | **Filas**, antes de agrupar | **Grupos**, después de agrupar |
| Puede usar `COUNT`, `SUM`... | No | **Sí** |

> [!tip] Regla de oro
> Toda columna del `SELECT` que **no** sea una función de agregación tiene que estar en el `GROUP BY`.

---

## 7. JOIN (ver [[SQL 07 - JOIN]])

| Tipo | Devuelve |
|---|---|
| `INNER JOIN` | Solo filas que tienen pareja en **ambas** tablas |
| `LEFT JOIN` | Todas las de la izquierda + las coincidencias (sin pareja → `NULL`) |
| `RIGHT JOIN` | Todas las de la derecha + las coincidencias |
| `FULL JOIN` | Todas las de ambas tablas |
| `CROSS JOIN` | Todas las combinaciones posibles (producto cartesiano) |
| *Self join* | Una tabla unida consigo misma (con alias distintos) |

```sql
SELECT c.nombre, p.id
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id;
```

> [!tip] Truco para encontrar "los que no tienen"
> `LEFT JOIN ... WHERE tabla_derecha.id IS NULL` devuelve las filas de la izquierda **sin** pareja (por ejemplo, clientes sin pedidos).

> [!warning] Diferencias entre sistemas
> MySQL y MariaDB **no tienen `FULL JOIN`** (se simula con `LEFT JOIN` + `UNION` + `RIGHT JOIN`). SQLite lo admite solo en versiones recientes.

---

## 8. Subconsultas (ver [[SQL 08 - Subconsultas]])

```sql
-- En WHERE: valor único
WHERE precio > (SELECT AVG(precio) FROM productos)

-- En WHERE: lista de valores
WHERE cliente_id IN (SELECT id FROM clientes WHERE ciudad = 'Madrid')

-- Comprobar existencia
WHERE EXISTS (SELECT 1 FROM pedidos p WHERE p.cliente_id = c.id)

-- En FROM: tabla derivada (necesita alias)
FROM (SELECT ... ) AS resumen
```

| Operador | Uso |
|---|---|
| `IN` / `NOT IN` | Pertenece / no pertenece a una lista |
| `EXISTS` / `NOT EXISTS` | Existe / no existe al menos una fila |
| `ANY` / `ALL` | Comparar con alguno / con todos los valores |

> [!warning] Cuidado con `NOT IN`
> Si la subconsulta devuelve algún `NULL`, `NOT IN` no devuelve **ninguna** fila. Es más seguro `NOT EXISTS`.

---

## 9. Modificar datos (ver [[SQL 09 - INSERT]], [[SQL 10 - UPDATE]], [[SQL 11 - DELETE]])

```sql
-- INSERT
INSERT INTO clientes (id, nombre, email) VALUES (1, 'Ana', 'ana@mail.com');
INSERT INTO clientes (id, nombre, email) VALUES (2, 'Luis', 'l@mail.com'), (3, 'Eva', 'e@mail.com');
INSERT INTO archivo SELECT * FROM clientes WHERE activo = 0;   -- desde un SELECT

-- UPDATE
UPDATE productos SET precio = precio * 1.10 WHERE categoria = 'libros';

-- DELETE
DELETE FROM pedidos WHERE fecha < '2020-01-01';
TRUNCATE TABLE pedidos;     -- vacía la tabla entera (rápido, sin WHERE)
```

> [!warning] Los dos errores más caros
> `UPDATE` o `DELETE` **sin `WHERE`** afecta a **todas** las filas. Comprueba primero con un `SELECT` con el mismo `WHERE`, y usa una transacción si puedes.

| Instrucción | `WHERE` | Se puede deshacer | Reinicia contador de `id` |
|---|---|---|---|
| `DELETE` | Sí | Sí (en transacción) | No |
| `TRUNCATE` | No | Depende del sistema | Normalmente sí |
| `DROP TABLE` | No | Depende del sistema | Borra la tabla entera |

---

## 10. Estructura: CREATE y ALTER (ver [[SQL 12 - CREATE y ALTER]])

```sql
CREATE TABLE productos (
    id        INT PRIMARY KEY,
    nombre    VARCHAR(100) NOT NULL,
    precio    DECIMAL(10, 2) NOT NULL DEFAULT 0,
    creado_el DATE
);

ALTER TABLE productos ADD COLUMN stock INT;               -- añadir columna
ALTER TABLE productos DROP COLUMN stock;                  -- quitar columna
ALTER TABLE productos RENAME COLUMN nombre TO titulo;     -- renombrar columna
DROP TABLE IF EXISTS productos;                           -- borrar tabla
```

### Tipos de datos habituales

| Tipo de dato | Tipos comunes |
|---|---|
| Enteros | `INT`, `BIGINT`, `SMALLINT` |
| Decimales | `DECIMAL(p, s)` (exacto, para dinero), `FLOAT`, `DOUBLE` (aproximados) |
| Texto | `CHAR(n)`, `VARCHAR(n)`, `TEXT` |
| Fechas | `DATE`, `TIME`, `DATETIME` / `TIMESTAMP` |
| Lógicos | `BOOLEAN` (en MySQL es `TINYINT(1)`) |

> [!warning] Cambian según el sistema
> Modificar una columna: `ALTER COLUMN` (SQL Server/PostgreSQL) frente a `MODIFY` (MySQL/Oracle). `ALTER TABLE` en SQLite es muy limitado. Para añadir columnas, SQL Server y Oracle no llevan la palabra `COLUMN` en `ADD`.

---

## 11. Claves y restricciones (ver [[SQL 13 - Claves y restricciones]])

| Restricción | Qué garantiza |
|---|---|
| `PRIMARY KEY` | Identifica cada fila: único y nunca `NULL`. Solo una por tabla |
| `FOREIGN KEY` | El valor debe existir en otra tabla (integridad referencial) |
| `UNIQUE` | No se repite el valor (admite `NULL`) |
| `NOT NULL` | Obligatorio: no puede quedar vacío |
| `DEFAULT` | Valor automático si no se indica ninguno |
| `CHECK` | El valor cumple una condición (`CHECK (precio >= 0)`) |

```sql
CONSTRAINT nombre_regla CHECK (precio >= 0)
```

---

## 12. Índices (ver [[SQL 14 - Índices]])

```sql
CREATE INDEX idx_clientes_email ON clientes (email);
CREATE UNIQUE INDEX idx_clientes_email ON clientes (email);
DROP INDEX idx_clientes_email;       -- MySQL/SQL Server: DROP INDEX nombre ON tabla
```

| Conviene indexar | No conviene indexar |
|---|---|
| Columnas de `WHERE`, `JOIN` y `ORDER BY` | Tablas pequeñas |
| Claves foráneas | Columnas con muy pocos valores distintos |
| Columnas con muchos valores distintos | Tablas con muchísimas escrituras |

> [!tip] Recuerda
> Un índice **acelera las lecturas** pero **ralentiza las escrituras** y ocupa espacio. Las claves primarias y las `UNIQUE` ya crean un índice solas.

---

## 13. Relaciones (ver [[SQL 15 - Relaciones]])

| Tipo | Cómo se implementa |
|---|---|
| **1:N** | FK en la tabla del lado "muchos" |
| **N:M** | Tabla intermedia con dos FK (y clave primaria compuesta) |
| **1:1** | FK + `UNIQUE` |

```sql
FOREIGN KEY (cliente_id) REFERENCES clientes (id)
    ON DELETE RESTRICT
    ON UPDATE CASCADE
```

| Acción `ON DELETE` | Efecto |
|---|---|
| `RESTRICT` / `NO ACTION` | Prohíbe borrar el padre si tiene hijos |
| `CASCADE` | Borra también los hijos |
| `SET NULL` | Pone la FK de los hijos a `NULL` |
| `SET DEFAULT` | Pone la FK de los hijos a su valor por defecto |

> [!tip] Recuerda
> La FK va en la tabla **hija** (la que "pertenece" a la otra). La tabla padre se crea primero.

---

## 14. Normalización (ver [[SQL 16 - Normalización]])

| Forma normal | Regla |
|---|---|
| **1FN** | Un solo valor por celda; sin grupos repetidos |
| **2FN** | Todo depende de **toda** la clave (importa con claves compuestas) |
| **3FN** | Todo depende **solo** de la clave (sin dependencias transitivas) |

> [!tip] En una frase
> Cada columna depende **de la clave, de toda la clave y de nada más que la clave**.

Desnormalizar = duplicar datos a propósito para leer más rápido (a costa de riesgo de incoherencias).

---

## 15. Transacciones (ver [[SQL 17 - Transacciones]])

```sql
START TRANSACTION;      -- o BEGIN
-- ... INSERT / UPDATE / DELETE ...
COMMIT;                 -- guardar todo
-- ROLLBACK;            -- deshacer todo

SAVEPOINT punto;
ROLLBACK TO punto;
```

| ACID | Significado |
|---|---|
| **A**tomicidad | Todo o nada |
| **C**onsistencia | De un estado válido a otro válido |
| **A**islamiento | Las transacciones no se estorban |
| **D**urabilidad | Tras `COMMIT`, los datos no se pierden |

| Nivel de aislamiento | Evita |
|---|---|
| `READ UNCOMMITTED` | Nada |
| `READ COMMITTED` | Lecturas sucias |
| `REPEATABLE READ` | + lecturas no repetibles |
| `SERIALIZABLE` | + lecturas fantasma |

---

## 16. Vistas (ver [[SQL 18 - Vistas]])

```sql
CREATE VIEW resumen_pedidos AS
SELECT ... ;

CREATE OR REPLACE VIEW resumen_pedidos AS SELECT ... ;   -- modificar
DROP VIEW IF EXISTS resumen_pedidos;                     -- borrar
```

> [!tip] Recuerda
> Una vista **no guarda datos**, solo la consulta. Se usa como una tabla. La **vista materializada** sí guarda el resultado (PostgreSQL, Oracle) y hay que refrescarla.

---

## 17. Procedimientos, funciones y triggers (ver [[SQL 19 - Procedimientos y triggers]])

| | Procedimiento | Función | Trigger |
|---|---|---|---|
| Se ejecuta | `CALL` / `EXEC` | Dentro de un `SELECT` | **Solo**, ante `INSERT`/`UPDATE`/`DELETE` |
| Devuelve | Nada o por `OUT` | Un valor | Nada |
| Parámetros | Sí (`IN`, `OUT`, `INOUT`) | Sí | No (usa `OLD` y `NEW`) |

```sql
CREATE TRIGGER trg_nombre
AFTER UPDATE ON tabla
FOR EACH ROW
BEGIN
    -- OLD.columna = valor antes, NEW.columna = valor después
END;
```

> [!warning] Cambian mucho según el sistema
> MySQL: SQL/PSM con `DELIMITER`. PostgreSQL: PL/pgSQL (el trigger necesita una función aparte). Oracle: PL/SQL. SQL Server: T-SQL (sin `FOR EACH ROW`). SQLite: **sin procedimientos**, solo triggers.

---

## 18. Diferencias entre sistemas (resumen)

| Tema | MySQL / MariaDB | PostgreSQL | SQL Server | Oracle | SQLite |
|---|---|---|---|---|---|
| Limitar filas | `LIMIT` | `LIMIT` | `TOP` / `OFFSET FETCH` | `FETCH FIRST` | `LIMIT` |
| Id automático | `AUTO_INCREMENT` | `GENERATED ... AS IDENTITY` | `IDENTITY` | `GENERATED ... AS IDENTITY` | `INTEGER PRIMARY KEY` |
| Concatenar | `CONCAT()` | `\|\|` | `+` / `CONCAT()` | `\|\|` | `\|\|` |
| `FULL JOIN` | No | Sí | Sí | Sí | Versiones recientes |
| Vistas materializadas | No | Sí | Vistas indexadas | Sí | No |
| Procedimientos | Sí | Sí | Sí | Sí | **No** |
| Cambios de estructura en transacción | No | Sí | Sí | No | Sí |
| FK activadas por defecto | Sí (InnoDB) | Sí | Sí | Sí | **No** (`PRAGMA foreign_keys = ON`) |

---

## 19. Errores y comprobaciones de última hora

| Si ves... | Revisa... |
|---|---|
| Resultado con **muchas más filas** de las esperadas | Falta la condición del `JOIN` (producto cartesiano) o hay duplicados |
| Sale `NULL` donde no lo esperabas | `LEFT JOIN` sin pareja, o cálculos con `NULL` |
| `Unknown column` | Nombre mal escrito, o alias usado donde aún no existe (por ejemplo, en `WHERE`) |
| Error al agrupar | Una columna del `SELECT` no está en el `GROUP BY` |
| No puedes borrar una fila | Tiene hijos y la FK es `RESTRICT` |
| Consulta lenta | Falta un índice en columnas de `WHERE`/`JOIN`, o `SELECT *` innecesario |
| `UPDATE`/`DELETE` que afectó a todo | Faltaba el `WHERE`: usa `ROLLBACK` si aún no hiciste `COMMIT` |

---

## 20. Resumen final

| Quiero... | Uso... |
|---|---|
| Leer datos | `SELECT ... FROM ... WHERE` |
| Ordenar o limitar | `ORDER BY`, `LIMIT` |
| Resumir | Funciones de agregación + `GROUP BY` + `HAVING` |
| Unir tablas | `JOIN` |
| Consulta dentro de otra | Subconsulta (`IN`, `EXISTS`) |
| Añadir / cambiar / borrar datos | `INSERT` / `UPDATE` / `DELETE` |
| Crear o cambiar tablas | `CREATE` / `ALTER` / `DROP` |
| Proteger los datos | Restricciones y claves foráneas |
| Acelerar búsquedas | Índices |
| Conectar tablas | Relaciones (1:N, N:M, 1:1) |
| Evitar datos repetidos | Normalización (hasta 3FN) |
| Operaciones "todo o nada" | Transacciones |
| Guardar una consulta con nombre | Vistas |
| Guardar lógica o automatizar | Procedimientos y triggers |

> [!tip] Para recordar en una frase
> **Piensa primero cómo se reparten los datos en tablas (normalización y relaciones), y después cómo los consultas (`SELECT`, `JOIN`, `GROUP BY`).**

Con esta nota se cierra la carpeta de SQL. Vuelve al [[SQL 00 - Índice]].