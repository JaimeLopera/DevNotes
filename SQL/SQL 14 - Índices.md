# 14 - Índices

> [!info] ¿Qué es?
> Un **índice** es una estructura auxiliar que la base de datos mantiene para **encontrar filas mucho más rápido**, sin tener que leer la tabla entera. Funciona como el índice alfabético de un libro: en vez de hojear todas las páginas para encontrar una palabra, vas directo a la página que toca. A cambio, ocupa espacio y hace un poco más lentas las escrituras.

---

## 1. Antes de empezar

**Qué deberías saber ya:**
- Consultar datos con `SELECT`, `WHERE`, `ORDER BY` y `JOIN` → [[SQL 02 - SELECT]], [[SQL 03 - WHERE]], [[SQL 07 - JOIN]]
- Claves primarias, `UNIQUE` y claves foráneas → [[SQL 13 - Claves y restricciones]]

**El problema que resuelven:**

Sin índice, para ejecutar esto:

```sql
SELECT * FROM clientes WHERE email = "ana@correo.com";
```

la base de datos debe **leer todas las filas** una por una hasta comprobarlas todas (*full table scan*). Con 100 filas no se nota; con 10 millones, tarda segundos o minutos. Con un índice sobre `email`, salta casi directamente a la fila correcta.

> [!warning] Dialectos
> Los conceptos son los mismos en todos los motores, pero los tipos de índice disponibles y algunos comandos (sobre todo para borrarlos) cambian entre MySQL, PostgreSQL, SQL Server, SQLite y Oracle.

---

## 2. Concepto fundamental

### Cómo funciona (sin entrar en detalles técnicos)

El índice guarda los valores de una o varias columnas **ordenados**, junto con un "puntero" a la fila real. Como están ordenados, buscar es rápido (se descarta la mitad de los datos en cada paso, como cuando adivinas un número del 1 al 100).

Casi todos los motores usan por defecto una estructura llamada **B-tree** (árbol balanceado). Es la que debes tener en la cabeza por defecto.

### El equilibrio: lecturas vs. escrituras

| | Con índice | Sin índice |
|---|---|---|
| `SELECT` con filtro | **Mucho más rápido** | Lento (lee todo) |
| `INSERT` / `UPDATE` / `DELETE` | **Algo más lento** (hay que actualizar también el índice) | Más rápido |
| Espacio en disco | **Ocupa más** | Menos |

> [!tip] Idea clave
> Un índice **no es gratis**. Cada índice que creas hay que mantenerlo al escribir. Por eso **no se indexa todo**, solo lo que se consulta mucho.

### Qué columnas conviene indexar

- Las que aparecen mucho en **`WHERE`** (filtros).
- Las que se usan en **`JOIN`** (sobre todo las claves foráneas).
- Las que se usan en **`ORDER BY`** y **`GROUP BY`** sobre tablas grandes.
- Las que tienen **muchos valores distintos** (alta *selectividad*): `email`, `dni`, `id`.

### Qué columnas NO conviene indexar

- Tablas **pequeñas** (leer todo ya es rápido).
- Columnas con **pocos valores distintos** (baja selectividad): un `sexo` o un `activo` (sí/no). El índice casi no ayuda porque devuelve media tabla.
- Columnas que **cambian constantemente**.
- Tablas con **muchas más escrituras que lecturas**.

### Índices que ya existen sin que los crees

Estas restricciones crean un índice **automáticamente**:

| Restricción | Índice automático |
|---|---|
| `PRIMARY KEY` | Sí (en todos los motores) |
| `UNIQUE` | Sí (en todos los motores) |
| `FOREIGN KEY` | **MySQL (InnoDB): sí. PostgreSQL y SQL Server: no** (hay que crearlo a mano) |

---

## 3. Sintaxis / estructura

### Crear un índice

```sql
CREATE INDEX nombre_indice ON tabla (columna);
```

### Crear un índice único

```sql
CREATE UNIQUE INDEX nombre_indice ON tabla (columna);
```

Hace lo mismo que una restricción `UNIQUE`: impide valores repetidos y además acelera búsquedas.

### Índice sobre varias columnas (compuesto)

```sql
CREATE INDEX nombre_indice ON tabla (columna1, columna2);
```

### Índice en el `CREATE TABLE` (MySQL)

```sql
CREATE TABLE clientes (
    id     INT PRIMARY KEY,
    email  VARCHAR(150),
    ciudad VARCHAR(80),
    INDEX idx_clientes_ciudad (ciudad)
);
```

> [!warning] Esto es propio de MySQL
> En PostgreSQL, SQL Server o SQLite se crean **después**, con `CREATE INDEX`.

### Borrar un índice

```sql
-- PostgreSQL, SQLite, Oracle
DROP INDEX nombre_indice;

-- MySQL y SQL Server (hay que indicar la tabla)
DROP INDEX nombre_indice ON tabla;
```

### Ver los índices de una tabla

| Motor | Comando |
|---|---|
| MySQL | `SHOW INDEX FROM tabla;` |
| PostgreSQL | `\d tabla` (en `psql`) o consultar `pg_indexes` |
| SQL Server | `EXEC sp_helpindex "tabla";` |
| SQLite | `PRAGMA index_list(tabla);` |
| Oracle | `SELECT * FROM user_indexes;` |

---

## 4. Elementos / propiedades / características

### 4.1 Índice simple vs. compuesto

- **Simple:** una sola columna. Sirve para buscar por esa columna.
- **Compuesto:** varias columnas. Sirve cuando **filtras por esa combinación**.

```sql
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos (cliente_id, fecha);
```

### 4.2 La regla del prefijo izquierdo (muy importante)

En un índice compuesto **el orden de las columnas importa**. El índice `(cliente_id, fecha)` funciona como una agenda ordenada **primero por cliente y, dentro de cada cliente, por fecha**.

| Consulta | ¿Usa el índice `(cliente_id, fecha)`? |
|---|---|
| `WHERE cliente_id = 5` | **Sí** |
| `WHERE cliente_id = 5 AND fecha = "2026-10-01"` | **Sí** (completo) |
| `WHERE fecha = "2026-10-01"` | **No** (o muy mal), porque se salta la primera columna |

> [!tip] Regla práctica
> Pon primero la columna que **más se usa para filtrar con igualdad** y que más reduce resultados. Piensa en la agenda: no puedes buscar por nombre de pila si está ordenada por apellido.

### 4.3 Índice único

Además de acelerar, **impide duplicados**. Es lo que hay detrás de `UNIQUE`.

### 4.4 Índice agrupado (*clustered*) vs. no agrupado

| | Agrupado (*clustered*) | No agrupado (*non-clustered*) |
|---|---|---|
| Qué es | Define **cómo se ordenan físicamente las filas** en la tabla | Estructura **aparte** que apunta a las filas |
| Cuántos por tabla | **Solo uno** (las filas solo pueden estar ordenadas de una manera) | **Muchos** |
| Quién lo usa | MySQL/InnoDB (la clave primaria), SQL Server (por defecto la clave primaria) | Todos |

- En **MySQL/InnoDB** la clave primaria **es** el índice agrupado, siempre.
- En **SQL Server** puedes elegir qué índice es el agrupado (`CREATE CLUSTERED INDEX`).
- **PostgreSQL** no tiene índices agrupados automáticos: las filas se guardan sin orden (existe el comando `CLUSTER`, pero es un reordenado puntual, no se mantiene).

### 4.5 Índice de cobertura (*covering index*)

Es un índice que contiene **todas las columnas que necesita la consulta**, así la base de datos **ni siquiera toca la tabla**: responde solo con el índice. Es lo más rápido posible.

```sql
CREATE INDEX idx_pedidos_cobertura ON pedidos (cliente_id, fecha, total);

SELECT fecha, total FROM pedidos WHERE cliente_id = 5;
```

Esa consulta se resuelve entera con el índice.

### 4.6 Tipos de índice según el motor

| Tipo | Para qué sirve | Dónde |
|---|---|---|
| **B-tree** | Igualdad, rangos (`<`, `>`, `BETWEEN`), `ORDER BY`, prefijos (`LIKE "ab%"`) | **Todos** (es el estándar) |
| **Hash** | Solo igualdad exacta (`=`) | PostgreSQL, MySQL (tablas en memoria) |
| **Full-text** | Búsqueda de **palabras dentro de textos largos** | MySQL (`FULLTEXT`), PostgreSQL (GIN + `tsvector`), SQL Server |
| **GIN / GiST** | JSON, arrays, texto, datos geométricos | PostgreSQL |
| **Espacial** (*spatial*) | Datos geográficos (coordenadas, mapas) | MySQL (`SPATIAL`), PostgreSQL (PostGIS) |
| **Bitmap** | Columnas con muy pocos valores distintos, en almacenes de datos | Oracle |

> [!tip] Para empezar
> Con **B-tree** cubres el 95 % de los casos. El resto son para necesidades concretas.

### 4.7 Índices parciales y funcionales (según motor)

**Parcial:** indexa solo las filas que cumplen una condición. Ahorra espacio.

```sql
-- PostgreSQL y SQLite
CREATE INDEX idx_pedidos_pendientes ON pedidos (fecha) WHERE estado = "pendiente";
```

**Funcional (sobre expresión):** indexa el resultado de una función, no la columna cruda.

```sql
-- PostgreSQL, Oracle
CREATE INDEX idx_clientes_email_lower ON clientes (LOWER(email));

-- MySQL 8.0.13+ (con doble paréntesis)
CREATE INDEX idx_clientes_email_lower ON clientes ((LOWER(email)));
```

---

## 5. Ejemplos prácticos

### Ejemplo básico: acelerar una búsqueda frecuente

```sql
-- Se busca mucho a los clientes por email
CREATE INDEX idx_clientes_email ON clientes (email);

SELECT * FROM clientes WHERE email = "ana@correo.com";
```

### Ejemplo habitual: indexar una clave foránea y una consulta con filtro y orden

```sql
-- Acelera los JOIN con clientes y las búsquedas de pedidos de un cliente
CREATE INDEX idx_pedidos_cliente ON pedidos (cliente_id);

-- Pedidos de un cliente, del más reciente al más antiguo
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos (cliente_id, fecha);

SELECT * 
FROM pedidos 
WHERE cliente_id = 5 
ORDER BY fecha DESC;
```

El índice compuesto sirve para **filtrar y ordenar a la vez**, sin ordenar después en memoria. (Si creas este compuesto, el simple sobre `cliente_id` suele sobrar, porque el compuesto ya cubre el prefijo izquierdo.)

### Ejemplo completo: comprobar si la base de datos usa tu índice con `EXPLAIN`

`EXPLAIN` muestra el **plan de ejecución**: cómo piensa resolver la consulta.

```sql
EXPLAIN SELECT * FROM pedidos WHERE cliente_id = 5;
```

**Qué mirar** (cambia según motor):

| Motor | Mala señal (lee toda la tabla) | Buena señal (usa índice) |
|---|---|---|
| MySQL | `type: ALL` | `type: ref`, `range`, `const` y `key: nombre_indice` |
| PostgreSQL | `Seq Scan` | `Index Scan` / `Index Only Scan` |
| SQL Server | `Table Scan` | `Index Seek` |
| SQLite | `SCAN tabla` | `SEARCH tabla USING INDEX ...` |

> [!tip] Flujo de trabajo recomendado
> 1. Detecta una consulta lenta.
> 2. Mira su plan con `EXPLAIN`.
> 3. Crea el índice que falta.
> 4. Vuelve a ejecutar `EXPLAIN` para **confirmar** que lo usa.

---

## 6. Buenas prácticas

- **No indexes por indexar:** crea índices para consultas reales y lentas, no "por si acaso".
- **Indexa las claves foráneas** (si tu motor no lo hace solo): acelera `JOIN` y los borrados en cascada.
- **Mide con `EXPLAIN`** antes y después; no adivines.
- **Cuida el orden en índices compuestos:** la columna más usada para filtrar va primero.
- **Prefiere un compuesto bien pensado** a varios simples sueltos.
- **Elimina los índices que no se usan** o que están repetidos: cuestan espacio y ralentizan escrituras.
- **Pon nombres claros y consistentes:** `idx_tabla_columnas` (y `uq_` para únicos).
- **Usa índices únicos para reglas de negocio** (un email, un DNI) en lugar de comprobar duplicados solo desde la aplicación.
- **Piensa en la proporción lecturas/escrituras** de la tabla antes de añadir muchos índices.
- **En tablas gigantes en producción**, crear un índice puede bloquear la tabla; usa las opciones online del motor (`CREATE INDEX CONCURRENTLY` en PostgreSQL, `ONLINE = ON` en SQL Server).

---

## 7. Diferencias importantes

### Índice vs. clave primaria vs. `UNIQUE`

| | Índice normal | `UNIQUE` | `PRIMARY KEY` |
|---|---|---|---|
| Acelera búsquedas | Sí | Sí | Sí |
| Impide duplicados | No | **Sí** | **Sí** |
| Admite `NULL` | Sí | Sí (con matices) | **No** |
| Cuántos por tabla | Muchos | Muchos | **Uno** |

### Índice simple vs. compuesto

- **Simple:** una columna, uso general.
- **Compuesto:** varias columnas, ideal para consultas que **siempre filtran por esa combinación**; el orden importa.

### Agrupado vs. no agrupado

El agrupado **es la propia tabla ordenada** (solo uno). El no agrupado **es una lista aparte** que apunta a la tabla (muchos).

### Índice vs. vista

Un índice **acelera** consultas sobre datos existentes; una vista **reutiliza** una consulta guardada. No hacen lo mismo → [[SQL 18 - Vistas]].

### Escaneo de tabla (*scan*) vs. búsqueda por índice (*seek*)

- **Scan:** lee todo.
- **Seek / Index scan:** va directo a lo necesario usando el índice.

---

## 8. Casos especiales

### Cuándo la base de datos **ignora** tu índice

Aunque exista el índice, no siempre se usa. Casos típicos:

**1. Función aplicada sobre la columna**

```sql
-- No usa el índice de "email"
SELECT * FROM clientes WHERE LOWER(email) = "ana@correo.com";
```

Solución: crear un índice funcional (apartado 4.7) o guardar los datos ya normalizados.

**2. `LIKE` que empieza por comodín**

```sql
WHERE nombre LIKE "%ana"    -- NO usa el índice (no sabe por dónde empezar)
WHERE nombre LIKE "ana%"    -- SÍ lo usa
```

Para buscar "en medio" de un texto usa índices *full-text*.

**3. Tipos de dato distintos (conversión implícita)**

```sql
-- "telefono" es VARCHAR y comparas con un número
WHERE telefono = 600123456     -- puede ignorar el índice
WHERE telefono = "600123456"   -- correcto
```

**4. Operaciones sobre la columna**

```sql
WHERE precio * 1.21 > 100   -- no usa el índice de "precio"
WHERE precio > 100 / 1.21   -- sí
```

**5. Condiciones con `OR` entre columnas distintas** o con `<>` / `NOT IN`: a menudo obligan a leer la tabla.

**6. El optimizador decide que no compensa:** si el filtro devuelve gran parte de la tabla (por ejemplo, el 30 % de las filas), leer todo puede ser más rápido que saltar entre el índice y la tabla.

### `NULL` en los índices

- En general, los valores `NULL` **sí se indexan** (MySQL, PostgreSQL, SQL Server, SQLite).
- **Oracle** es distinto: **no** indexa las filas donde *todas* las columnas del índice son `NULL`. Por eso `WHERE columna IS NULL` puede no usar el índice.

### Índices en columnas de texto largo

Indexar un `VARCHAR(500)` ocupa mucho. En **MySQL** puedes indexar solo un **prefijo**:

```sql
CREATE INDEX idx_titulo ON articulos (titulo(50));
```

### Búsqueda de texto completo (*full-text*)

```sql
-- MySQL
CREATE FULLTEXT INDEX idx_articulos_texto ON articulos (titulo, contenido);

SELECT * FROM articulos
WHERE MATCH(titulo, contenido) AGAINST ("base de datos");
```

En PostgreSQL se hace con `tsvector` y un índice GIN; en SQL Server con `CONTAINS`/`FREETEXT`.

### Mantenimiento

Con el tiempo, los índices se **fragmentan** y las **estadísticas** (lo que el optimizador sabe sobre tus datos) se quedan desactualizadas, y eso hace que elija mal los planes.

| Motor | Actualizar estadísticas | Reconstruir índices |
|---|---|---|
| MySQL | `ANALYZE TABLE tabla;` | `OPTIMIZE TABLE tabla;` |
| PostgreSQL | `ANALYZE tabla;` | `REINDEX INDEX nombre;` |
| SQL Server | `UPDATE STATISTICS tabla;` | `ALTER INDEX ... REBUILD` |

### Índices y cargas masivas

Si vas a insertar millones de filas de golpe, a veces es más rápido **borrar los índices, cargar los datos y volver a crearlos** que mantenerlos fila a fila.

---

## 9. Resumen

- Un **índice** es una estructura ordenada que permite **encontrar filas rápido** sin leer toda la tabla.
- A cambio, **ocupa espacio** y **ralentiza un poco** `INSERT`, `UPDATE` y `DELETE`.
- Indexa lo que se usa en **`WHERE`, `JOIN`, `ORDER BY`** y tiene **muchos valores distintos**; evita tablas pequeñas y columnas con pocos valores.
- `PRIMARY KEY` y `UNIQUE` crean índice solos; las **claves foráneas** también en MySQL, pero **no en PostgreSQL ni SQL Server**.
- Sintaxis: `CREATE [UNIQUE] INDEX nombre ON tabla (columnas);` y `DROP INDEX` (con `ON tabla` en MySQL y SQL Server).
- En un índice **compuesto el orden importa**: se usa desde la primera columna (regla del prefijo izquierdo).
- Tipos: **B-tree** (el estándar), hash, full-text, GIN/GiST, espacial. También existen los **agrupados**, **de cobertura**, **parciales** y **funcionales** según el motor.
- Usa **`EXPLAIN`** para comprobar si tu consulta usa el índice.
- Pueden ignorarse por: funciones sobre la columna, `LIKE "%...`, tipos distintos, operaciones sobre la columna o porque el optimizador decide que no compensa.
- **Mantenimiento:** actualiza estadísticas, reconstruye si hace falta y **borra los índices que no uses**.