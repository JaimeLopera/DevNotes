# 10 - UPDATE

> [!info] ¿Qué es?
> **UPDATE** es la sentencia de SQL que **modifica los datos de filas que ya existen** en una tabla. Es la "U" de **CRUD** (*Update*): sirve para cambiar un email, subir un precio, marcar un pedido como enviado, desactivar un usuario, etc.

---

## 1. Antes de empezar

Conviene tener claro:

- `SELECT` y `WHERE` (el `WHERE` es **crucial** en `UPDATE`) → [[SQL 02 - SELECT]], [[SQL 03 - WHERE]]
- Funciones de SQL (se usan mucho en los valores nuevos) → [[SQL 05 - Funciones]]
- `JOIN` y subconsultas (se pueden usar para decidir qué filas o qué valores) → [[SQL 07 - JOIN]], [[SQL 08 - Subconsultas]]
- Cómo se añaden filas nuevas (para no confundirlo con `UPDATE`) → [[SQL 09 - INSERT]]

Tabla de ejemplo de toda la nota:

**`productos`**

| id | nombre | precio | stock | categoria_id | activo |
|----|--------|--------|-------|--------------|--------|
| 1 | Teclado | 40.00 | 15 | 1 | TRUE |
| 2 | Ratón | 20.00 | 0 | 1 | TRUE |
| 3 | Monitor | 180.00 | 8 | 2 | TRUE |
| 4 | Webcam | 55.00 | 3 | 2 | FALSE |
| 5 | Altavoces | 35.00 | NULL | 3 | TRUE |

---

## 2. Concepto fundamental

`UPDATE` no crea ni borra filas: **cambia valores de columnas en filas que ya están**. Para ello hay que decir:

1. **Qué tabla** se modifica.
2. **Qué columnas** cambian y a **qué valor nuevo** (`SET`).
3. **Qué filas** se modifican (`WHERE`).

```sql
UPDATE productos
SET precio = 45.00
WHERE id = 1;
```

Resultado: el Teclado pasa de 40.00 a 45.00. Las demás filas no se tocan.

> [!warning] LA REGLA MÁS IMPORTANTE
> **Si te olvidas del `WHERE`, el `UPDATE` modifica TODAS las filas de la tabla.**
> ```sql
> UPDATE productos SET precio = 0;   -- ¡¡Todos los productos a precio 0!!
> ```
> No hay botón de deshacer (salvo que uses una transacción o tengas copia de seguridad). Escribe siempre el `WHERE` **antes** de ejecutar.

### Cómo trabaja SQL

1. Busca las filas que cumplen el `WHERE`.
2. A cada una le aplica los cambios del `SET`.
3. Devuelve cuántas filas ha modificado.

---

## 3. Sintaxis / estructura

```sql
UPDATE tabla
SET columna1 = valor1,
    columna2 = valor2
WHERE condicion;
```

### Piezas importantes

| Parte | Función |
|-------|---------|
| `UPDATE tabla` | Indica la tabla a modificar |
| `SET col = valor` | Qué columna cambia y a qué valor. Se separan con **comas** (no con `AND`) |
| `WHERE` | Filtra qué filas se modifican. **Opcional, pero casi siempre necesario** |

### Reglas básicas

1. Los cambios se separan con **comas**: `SET a = 1, b = 2`. Un error típico es usar `AND`.
2. Los **textos y fechas** van con comillas simples: `SET nombre = 'Ratón láser'`.
3. Solo se cambian las columnas que nombras en `SET`; el resto queda igual.
4. Los valores nuevos pueden ser **constantes, expresiones, funciones o subconsultas**.
5. Si ninguna fila cumple el `WHERE`, **no pasa nada** (0 filas afectadas) y no da error.

> [!note] Comillas en SQL
> Usa comillas simples (`'texto'`) para que funcione en todos los motores. Las dobles están reservadas para nombres de columnas/tablas en PostgreSQL, SQL Server y Oracle.

---

## 4. Elementos / propiedades / características

### 4.1 Modificar una columna en una fila

```sql
UPDATE productos
SET stock = 20
WHERE id = 2;
```

### 4.2 Modificar varias columnas a la vez

```sql
UPDATE productos
SET precio = 60.00,
    stock  = 10,
    activo = TRUE
WHERE id = 4;
```

### 4.3 Modificar varias filas

El `WHERE` puede afectar a muchas filas a la vez.

```sql
-- Desactivar todos los productos sin stock
UPDATE productos
SET activo = FALSE
WHERE stock = 0;
```

Puedes usar cualquier condición del `WHERE`: `IN`, `BETWEEN`, `LIKE`, `IS NULL`, `AND`, `OR`… → [[SQL 03 - WHERE]].

### 4.4 Usar el valor actual de la columna (cálculos)

En el `SET` puedes usar el valor que la columna tiene **ahora** para calcular el nuevo.

```sql
-- Subir un 10 % los precios de la categoría 1
UPDATE productos
SET precio = precio * 1.10
WHERE categoria_id = 1;

-- Restar 1 unidad al stock (una venta)
UPDATE productos
SET stock = stock - 1
WHERE id = 3;
```

> [!tip] Hazlo en la propia sentencia
> `SET stock = stock - 1` es **seguro**. En cambio, leer el stock, restar en tu aplicación y guardar el resultado puede dar errores si dos usuarios compran a la vez.

### 4.5 Asignar NULL o el valor por defecto

```sql
UPDATE productos
SET stock = NULL          -- "no hay dato"
WHERE id = 2;

UPDATE productos
SET activo = DEFAULT      -- el valor por defecto de la columna
WHERE id = 4;
```

### 4.6 Usar funciones en el nuevo valor

```sql
UPDATE productos
SET nombre = UPPER(nombre)
WHERE categoria_id = 1;

UPDATE productos
SET nombre = TRIM(nombre);       -- quitar espacios sobrantes
```

### 4.7 Usar CASE para valores distintos según la fila

`CASE` permite que **cada fila reciba un valor diferente** en una sola sentencia.

```sql
UPDATE productos
SET precio = CASE
    WHEN categoria_id = 1 THEN precio * 1.10
    WHEN categoria_id = 2 THEN precio * 1.05
    ELSE precio
END
WHERE activo = TRUE;
```

> [!tip] No olvides el `ELSE precio`
> Si una fila no cumple ningún `WHEN` y no hay `ELSE`, la columna se pone en `NULL`.

### 4.8 UPDATE con subconsulta

Sirve para **calcular el nuevo valor** o para **decidir qué filas** se modifican.

```sql
-- Poner a todos los productos el precio medio de su categoría (valor calculado)
UPDATE productos
SET precio = (
    SELECT AVG(p2.precio)
    FROM productos p2
    WHERE p2.categoria_id = productos.categoria_id
);

-- Desactivar productos de categorías que ya no existen (filas elegidas)
UPDATE productos
SET activo = FALSE
WHERE categoria_id NOT IN (SELECT id FROM categorias);
```

> [!warning] MySQL y la misma tabla
> En MySQL no puedes modificar una tabla y leerla en una subconsulta directa de esa misma sentencia (error 1093). Se soluciona envolviendo la subconsulta en una tabla derivada con alias: `(SELECT ... ) AS t`.

### 4.9 UPDATE con JOIN (modificar usando datos de otra tabla)

Aquí **la sintaxis cambia según el motor**, así que conviene conocerlas todas.

**Ejemplo:** subir 5 € el precio de los productos que pertenecen a la categoría "Periféricos".

```sql
-- MySQL / MariaDB
UPDATE productos p
JOIN categorias c ON p.categoria_id = c.id
SET p.precio = p.precio + 5
WHERE c.nombre = 'Periféricos';
```

```sql
-- PostgreSQL / SQLite (3.33+)
UPDATE productos
SET precio = precio + 5
FROM categorias c
WHERE productos.categoria_id = c.id
  AND c.nombre = 'Periféricos';
```

```sql
-- SQL Server
UPDATE p
SET p.precio = p.precio + 5
FROM productos p
JOIN categorias c ON p.categoria_id = c.id
WHERE c.nombre = 'Periféricos';
```

```sql
-- Versión portable (funciona en todos): subconsulta con EXISTS / IN
UPDATE productos
SET precio = precio + 5
WHERE categoria_id IN (
    SELECT id FROM categorias WHERE nombre = 'Periféricos'
);
```

> [!tip] Si buscas portabilidad
> La versión con subconsulta (`IN` o `EXISTS`) funciona en **todos** los motores. Es la mejor opción cuando solo necesitas usar la otra tabla para *filtrar*.

### 4.10 UPDATE con ORDER BY y LIMIT (solo algunos motores)

```sql
-- MySQL / MariaDB: modificar solo las 5 filas más antiguas
UPDATE productos
SET activo = FALSE
ORDER BY id
LIMIT 5;
```

PostgreSQL, SQL Server y Oracle **no** admiten `LIMIT` directamente en un `UPDATE`; se hace con una subconsulta sobre la clave primaria:

```sql
UPDATE productos
SET activo = FALSE
WHERE id IN (SELECT id FROM productos ORDER BY id LIMIT 5);
```

(En SQL Server se usa `TOP`: `UPDATE TOP (5) productos ...`.)

### 4.11 Ver qué filas se han modificado: RETURNING / OUTPUT

```sql
-- PostgreSQL, SQLite 3.35+, MariaDB 10.5+ (RETURNING)
UPDATE productos
SET precio = precio * 1.10
WHERE categoria_id = 1
RETURNING id, nombre, precio;
```

```sql
-- SQL Server (OUTPUT)
UPDATE productos
SET precio = precio * 1.10
OUTPUT INSERTED.id, INSERTED.precio
WHERE categoria_id = 1;
```

MySQL **no** tiene esta función; solo informa del número de filas afectadas.

### 4.12 Modificar la clave primaria o claves foráneas

Es **posible**, pero peligroso: si otras tablas apuntan a ese valor, la base de datos lo bloqueará (o lo propagará, si la clave foránea tiene `ON UPDATE CASCADE`). Como norma, **las claves primarias no se modifican** → [[SQL 13 - Claves y restricciones]].

---

## 5. Ejemplos prácticos

### Ejemplo básico: cambiar un valor de una fila

```sql
UPDATE productos
SET precio = 45.00
WHERE id = 1;
```

### Ejemplo habitual: varias columnas con cálculo

```sql
-- Rebajar un 20 % y marcar como oferta los productos con poco stock
UPDATE productos
SET precio = precio * 0.80,
    en_oferta = TRUE
WHERE stock < 5;
```

### Ejemplo completo: valores distintos con CASE

```sql
-- Reponer stock según la categoría
UPDATE productos
SET stock = CASE
    WHEN categoria_id = 1 THEN stock + 50
    WHEN categoria_id = 2 THEN stock + 20
    ELSE stock + 10
END
WHERE activo = TRUE;
```

---

## 6. Buenas prácticas

1. **Escribe siempre el `WHERE`** y revísalo antes de ejecutar.
2. **Prueba primero con un `SELECT`**: pon la misma condición en un `SELECT` y comprueba que devuelve exactamente las filas que quieres modificar.
3. **Usa transacciones** para cambios importantes: `BEGIN` → `UPDATE` → comprobar → `COMMIT` o `ROLLBACK` → [[SQL 17 - Transacciones]].
4. **Filtra por la clave primaria** cuando quieras modificar una fila concreta (`WHERE id = 1`), no por columnas que pueden repetirse (como el nombre).
5. **Haz copia de seguridad** antes de actualizaciones masivas.
6. **Usa cálculos en la propia sentencia** (`stock = stock - 1`) en lugar de leer, calcular fuera y guardar.
7. **Modifica solo las columnas necesarias**; no reescribas toda la fila "por si acaso".
8. **Usa consultas preparadas con parámetros** para evitar inyección SQL.
9. **Revisa el número de filas afectadas**: si esperabas 1 y salen 5.000, algo va mal.

```sql
-- ✅ Patrón seguro: comprobar primero con SELECT y luego modificar
SELECT * FROM productos WHERE stock = 0;                 -- ¿son estas?

BEGIN;
UPDATE productos SET activo = FALSE WHERE stock = 0;
-- si el número de filas afectadas es el esperado:
COMMIT;                                                  -- (o ROLLBACK;)
```

> [!note] Modo seguro de MySQL Workbench
> MySQL Workbench trae un "modo seguro" (*Safe Updates*) que bloquea los `UPDATE` sin `WHERE` sobre clave. Es útil mientras aprendes; no conviene desactivarlo.

---

## 7. Diferencias importantes

### UPDATE vs INSERT vs DELETE

| | Qué hace | Afecta a |
|---|----------|----------|
| `INSERT` | Crea filas nuevas | Filas que no existían → [[SQL 09 - INSERT]] |
| `UPDATE` | Cambia valores | Filas existentes (solo columnas indicadas) |
| `DELETE` | Elimina filas | Filas existentes (enteras) → [[SQL 11 - DELETE]] |

### UPDATE vs REPLACE / upsert

- `UPDATE` solo modifica si la fila **ya existe**. Si no existe, no hace nada.
- Un *upsert* (`ON CONFLICT`, `ON DUPLICATE KEY`, `MERGE`) **inserta si no existe y actualiza si existe** → [[SQL 09 - INSERT]].

### SET con coma vs AND

```sql
-- ❌ Error típico: AND en lugar de coma (no cambia dos columnas; evalúa una condición)
UPDATE productos SET precio = 10 AND stock = 5 WHERE id = 1;

-- ✅ Correcto
UPDATE productos SET precio = 10, stock = 5 WHERE id = 1;
```

### Orden de evaluación en el SET

En **MySQL** los cambios del `SET` se aplican **de izquierda a derecha**, de modo que una columna puede usar el valor ya modificado de otra. En **PostgreSQL y SQL estándar**, todos los valores se calculan usando los datos **originales** de la fila.

```sql
UPDATE t SET a = a + 1, b = a;
-- MySQL: b recibe el valor NUEVO de a
-- PostgreSQL: b recibe el valor ANTIGUO de a
```

Para evitar sorpresas, no hagas que una columna dependa de otra modificada en el mismo `SET`.

---

## 8. Casos especiales

### 8.1 Valores NULL en los cálculos

Cualquier operación con `NULL` da `NULL`. En el ejemplo, Altavoces tiene `stock = NULL`:

```sql
UPDATE productos SET stock = stock + 10 WHERE id = 5;
-- stock sigue siendo NULL (NULL + 10 = NULL)
```

Solución: `COALESCE`, que sustituye `NULL` por un valor.

```sql
UPDATE productos SET stock = COALESCE(stock, 0) + 10 WHERE id = 5;
```

### 8.2 Comparar con NULL en el WHERE

```sql
-- ❌ No funciona (NULL nunca es igual a nada)
UPDATE productos SET stock = 0 WHERE stock = NULL;

-- ✅ Correcto
UPDATE productos SET stock = 0 WHERE stock IS NULL;
```

### 8.3 Cambiar un valor por el mismo

Si el valor nuevo es igual al actual, algunos motores (MySQL) lo cuentan como "0 filas cambiadas" aunque el `WHERE` sí las encontró. Es normal.

### 8.4 Fallo de restricciones

El `UPDATE` completo se cancela (ninguna fila cambia) si alguna fila viola una restricción:

| Causa | Ejemplo |
|-------|---------|
| `UNIQUE` / clave primaria repetida | Poner el mismo email a dos usuarios |
| `NOT NULL` | `SET nombre = NULL` en una columna obligatoria |
| Clave foránea inexistente | `categoria_id = 99` que no existe |
| `CHECK` no cumplido | `precio = -5` con `CHECK (precio >= 0)` |
| Tipo o longitud incorrectos | Texto demasiado largo |

### 8.5 Cambiar el mismo dato en tablas relacionadas

Si necesitas modificar varias tablas a la vez, haz **varios `UPDATE` dentro de una transacción**: así o se cambian todas o ninguna.

### 8.6 Triggers

Un `UPDATE` puede disparar **triggers** (por ejemplo, guardar un historial de cambios) → [[SQL 19 - Procedimientos y triggers]].

### 8.7 Actualizar a través de una vista

Solo funciona con vistas "simples" (una tabla, sin agregaciones) → [[SQL 18 - Vistas]].

### 8.8 Bloqueos y rendimiento

Un `UPDATE` **bloquea** las filas que modifica mientras dura la transacción. Los `UPDATE` masivos pueden ralentizar otras operaciones: en tablas enormes conviene hacerlos **por lotes** y con filtros sobre columnas indexadas → [[SQL 14 - Índices]].

---

## 9. Resumen

- **`UPDATE`** modifica los valores de filas que **ya existen**.
- Sintaxis: `UPDATE tabla SET columna = valor [, columna2 = valor2] WHERE condicion;`
- Los cambios del `SET` se separan con **comas**, no con `AND`.
- **Sin `WHERE`, se modifican TODAS las filas.** Es el error más peligroso.
- Antes de ejecutar, comprueba la condición con un **`SELECT`** y, si es importante, usa una **transacción**.
- El nuevo valor puede ser una constante, un cálculo (`precio * 1.10`), una función, un `CASE` o una subconsulta.
- `UPDATE ... JOIN` cambia de sintaxis según el motor (MySQL, PostgreSQL/SQLite, SQL Server); la versión con subconsulta (`IN`/`EXISTS`) es la portable.
- Cualquier operación con `NULL` da `NULL`: usa `COALESCE` y compara con `IS NULL`.
- Si una fila viola una restricción, **todo el `UPDATE` falla** y no cambia nada.
- `RETURNING` (PostgreSQL) y `OUTPUT` (SQL Server) devuelven las filas modificadas; MySQL no los tiene.
- No modifiques claves primarias; usa **consultas preparadas** para evitar inyección SQL.