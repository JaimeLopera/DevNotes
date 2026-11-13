# 12 - CREATE y ALTER

> [!info] ¿Qué es?
> `CREATE` y `ALTER` son las instrucciones que **construyen y modifican la estructura** de una base de datos: crear bases de datos y tablas (`CREATE`) y cambiarlas después, añadiendo, quitando o modificando columnas y reglas (`ALTER`). Pertenecen al grupo **DDL** (*Data Definition Language*): no trabajan con los datos, sino con el **molde** que los contiene.

---

## 1. Antes de empezar

### DDL frente a DML

SQL se divide en grupos según lo que hace cada instrucción:

| Grupo | Para qué sirve | Instrucciones |
|---|---|---|
| **DDL** | Definir la **estructura** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Manipular los **datos** | `INSERT`, `UPDATE`, `DELETE` |
| **DQL** | Consultar datos | `SELECT` |
| **DCL** | Gestionar permisos | `GRANT`, `REVOKE` |
| **TCL** | Controlar transacciones | `COMMIT`, `ROLLBACK` |

Esta nota trata el DDL. Los datos (`INSERT`, `UPDATE`, `DELETE`) se ven en sus propias notas ([[SQL 09 - INSERT]], [[SQL 10 - UPDATE]], [[SQL 11 - DELETE]]).

### Lo que conviene saber antes

- Qué es una **tabla**, una **fila** y una **columna**.
- Qué son los **tipos de datos** (números, texto, fechas…).
- Que normalmente necesitas **permisos** especiales para ejecutar DDL.

### Importante: el DDL cambia mucho entre sistemas

El `SELECT`, el `WHERE` o el `JOIN` son casi iguales en todos los sistemas. El **DDL no**: cada gestor (MySQL, PostgreSQL, SQL Server, Oracle, SQLite) tiene su propia forma de hacer ciertas cosas, sobre todo con los **autoincrementales**, los **tipos de datos** y el **`ALTER`**.

En esta nota la sintaxis **estándar** va primero y, cuando un sistema hace algo distinto, se indica con su nombre.

---

## 2. Concepto fundamental

Piensa en una tabla como una **plantilla de formulario**: define qué campos existen, de qué tipo es cada uno y qué reglas cumplen. Los datos son los formularios rellenados.

Hay tres instrucciones principales para trabajar con esa plantilla:

| Instrucción | Qué hace | Analogía |
|---|---|---|
| `CREATE` | **Crea** un objeto nuevo (base de datos, tabla, índice, vista…) | Diseñar la plantilla |
| `ALTER` | **Modifica** un objeto que ya existe | Corregir la plantilla |
| `DROP` | **Elimina** un objeto entero | Tirar la plantilla |

> [!warning] El DDL es más delicado que el DML
> Cambiar la estructura afecta a **todos los datos y a todas las aplicaciones** que usan esa tabla. Además, en muchos sistemas **no se puede deshacer** (ver el apartado 8). Antes de tocar una base de datos real, haz una **copia de seguridad**.

---

## 3. Sintaxis / estructura

### 3.1 Crear una base de datos

```sql
CREATE DATABASE tienda;
```

En algunos sistemas se usa `CREATE SCHEMA` (un *esquema* es un "cajón" donde agrupar tablas):

```sql
CREATE SCHEMA ventas;
```

> [!note] Base de datos y esquema según el sistema
> - **MySQL**: `DATABASE` y `SCHEMA` son **sinónimos**.
> - **PostgreSQL / SQL Server**: son cosas distintas. Una base de datos contiene varios esquemas, y cada esquema contiene tablas.
> - **SQLite**: no tiene `CREATE DATABASE`; cada base de datos es **un archivo**.

Para empezar a usarla: `USE tienda;` (MySQL / SQL Server) o conectarte a ella desde el cliente (PostgreSQL).

### 3.2 Crear una tabla: `CREATE TABLE`

```sql
CREATE TABLE nombre_tabla (
    columna1  TIPO  [restricciones],
    columna2  TIPO  [restricciones],
    ...
    [restricciones de tabla]
);
```

Cada columna lleva **nombre**, **tipo** y, opcionalmente, **restricciones**:

| Restricción | Qué significa |
|---|---|
| `PRIMARY KEY` | Identifica cada fila de forma única |
| `NOT NULL` | El valor es obligatorio |
| `UNIQUE` | No se puede repetir |
| `DEFAULT valor` | Valor automático si no se indica otro |
| `CHECK (condición)` | El valor debe cumplir una condición |
| `REFERENCES tabla(col)` | Clave foránea (apunta a otra tabla) |

> [!tip]
> Aquí solo se muestra lo básico de las restricciones. Se explican a fondo en [[SQL 13 - Claves y restricciones]].

### 3.3 Crear solo si no existe: `IF NOT EXISTS`

Si intentas crear una tabla que ya existe, da **error**. Con `IF NOT EXISTS` simplemente no hace nada:

```sql
CREATE TABLE IF NOT EXISTS clientes (
    id INT PRIMARY KEY
);
```

Lo soportan MySQL, PostgreSQL y SQLite. **SQL Server y Oracle clásico no**: allí se comprueba antes si existe.

### 3.4 Autoincrementales (la parte donde más cambian los sistemas)

Un autoincremental es una columna cuyo valor se genera solo (1, 2, 3…). Cada sistema lo escribe distinto:

| Sistema | Sintaxis |
|---|---|
| **MySQL / MariaDB** | `id INT AUTO_INCREMENT PRIMARY KEY` |
| **PostgreSQL** | `id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| **SQL Server** | `id INT IDENTITY(1,1) PRIMARY KEY` |
| **Oracle (12c+)** | `id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| **SQLite** | `id INTEGER PRIMARY KEY AUTOINCREMENT` |

> [!warning] Obsoleto / legado
> En PostgreSQL existe también `SERIAL` (`id SERIAL PRIMARY KEY`). Funciona y lo verás en muchos tutoriales antiguos, pero hoy se recomienda **`GENERATED ... AS IDENTITY`**, que es el estándar SQL.

### 3.5 Modificar una tabla: `ALTER TABLE`

La forma general es:

```sql
ALTER TABLE nombre_tabla ACCIÓN;
```

Las acciones más comunes:

**Añadir una columna** (igual en todos los sistemas, salvo que SQL Server no lleva la palabra `COLUMN`):

```sql
ALTER TABLE clientes ADD COLUMN telefono VARCHAR(20);
```

**Eliminar una columna:**

```sql
ALTER TABLE clientes DROP COLUMN telefono;
```

**Renombrar una columna** (estándar y muy extendido):

```sql
ALTER TABLE clientes RENAME COLUMN telefono TO movil;
```

**Renombrar la tabla:**

```sql
ALTER TABLE clientes RENAME TO usuarios;
```

**Cambiar el tipo de una columna** (aquí cada sistema es distinto):

| Sistema | Sintaxis |
|---|---|
| **MySQL / MariaDB** | `ALTER TABLE t MODIFY COLUMN col VARCHAR(100);` |
| **PostgreSQL** | `ALTER TABLE t ALTER COLUMN col TYPE VARCHAR(100);` |
| **SQL Server** | `ALTER TABLE t ALTER COLUMN col VARCHAR(100);` |
| **Oracle** | `ALTER TABLE t MODIFY (col VARCHAR2(100));` |
| **SQLite** | Muy limitado (ver apartado 8) |

**Poner o quitar un valor por defecto:**

```sql
ALTER TABLE clientes ALTER COLUMN pais SET DEFAULT 'España';
ALTER TABLE clientes ALTER COLUMN pais DROP DEFAULT;
```

**Poner o quitar `NOT NULL`** (PostgreSQL):

```sql
ALTER TABLE clientes ALTER COLUMN email SET NOT NULL;
ALTER TABLE clientes ALTER COLUMN email DROP NOT NULL;
```

> En MySQL se hace con `MODIFY`, repitiendo la definición completa de la columna; en SQL Server, con `ALTER COLUMN` indicando el tipo y `NOT NULL` / `NULL`.

**Añadir o quitar restricciones:**

```sql
ALTER TABLE clientes ADD CONSTRAINT uq_email UNIQUE (email);
ALTER TABLE clientes DROP CONSTRAINT uq_email;
```

> [!note]
> Quitar una restricción tiene excepciones por sistema: en MySQL, por ejemplo, las claves foráneas se quitan con `DROP FOREIGN KEY` y los índices únicos con `DROP INDEX`.

### 3.5.1 Varias acciones en un solo `ALTER`

Muchos sistemas permiten encadenar acciones separadas por comas (se ejecutan en una sola operación, que suele ser más eficiente):

```sql
ALTER TABLE clientes
    ADD COLUMN ciudad VARCHAR(50),
    ADD COLUMN codigo_postal VARCHAR(10);
```

### 3.6 Eliminar objetos: `DROP`

```sql
DROP TABLE clientes;
DROP TABLE IF EXISTS clientes;
DROP DATABASE tienda;
```

> [!danger] `DROP` borra la estructura **y todos los datos**
> No pide confirmación. Si no tienes copia de seguridad, no hay vuelta atrás.

---

## 4. Elementos / propiedades / características

### 4.1 Tipos de datos más habituales

Los nombres exactos varían, pero casi todos los sistemas tienen estos grupos:

| Tipo de dato | Para qué | Ejemplos |
|---|---|---|
| **Entero** | Números sin decimales | `INT`, `BIGINT`, `SMALLINT` |
| **Decimal exacto** | Dinero y cantidades precisas | `DECIMAL(10,2)`, `NUMERIC(10,2)` |
| **Decimal aproximado** | Cálculos científicos | `FLOAT`, `DOUBLE`, `REAL` |
| **Texto corto** | Texto con límite | `VARCHAR(100)`, `CHAR(2)` |
| **Texto largo** | Texto sin límite práctico | `TEXT` |
| **Fecha / hora** | Fechas y momentos | `DATE`, `TIME`, `TIMESTAMP` |
| **Lógico** | Verdadero / falso | `BOOLEAN` (en SQL Server: `BIT`) |

> [!tip] Para dinero, usa siempre `DECIMAL`
> `FLOAT` guarda aproximaciones y puede dar resultados como `0.30000000000000004`. `DECIMAL(10,2)` guarda exactamente 2 decimales.

### 4.2 Restricciones a nivel de columna y de tabla

Una restricción se puede escribir **junto a la columna** o **al final de la tabla**. Esto último es obligatorio cuando afecta a **varias columnas** (por ejemplo, una clave primaria compuesta) y permite ponerle nombre:

```sql
CREATE TABLE matriculas (
    alumno_id  INT,
    curso_id   INT,
    fecha      DATE NOT NULL,
    CONSTRAINT pk_matriculas PRIMARY KEY (alumno_id, curso_id)
);
```

### 4.3 Copiar la estructura de otra tabla

**Copiar estructura y datos** a partir de una consulta:

```sql
CREATE TABLE clientes_madrid AS
SELECT * FROM clientes WHERE ciudad = 'Madrid';
```

> En **SQL Server** se escribe distinto: `SELECT * INTO clientes_madrid FROM clientes WHERE ...;`

**Copiar solo la estructura** (sin datos):

```sql
CREATE TABLE clientes_copia (LIKE clientes);   -- PostgreSQL
CREATE TABLE clientes_copia LIKE clientes;     -- MySQL
```

> [!warning]
> `CREATE TABLE ... AS SELECT` copia columnas y datos, pero **normalmente no copia** claves, índices ni restricciones. Tendrás que añadirlas después con `ALTER`.

### 4.4 Tablas temporales

Existen solo durante la sesión y se borran solas al cerrar la conexión. Sirven para guardar resultados intermedios:

```sql
CREATE TEMPORARY TABLE resultados (id INT, total DECIMAL(10,2));
```

En SQL Server se crean con `#` delante del nombre: `CREATE TABLE #resultados (...)`.

---

## 5. Ejemplos prácticos

### Ejemplo básico: una tabla sencilla

```sql
CREATE TABLE productos (
    id      INT PRIMARY KEY,
    nombre  VARCHAR(100) NOT NULL,
    precio  DECIMAL(10,2)
);
```

### Ejemplo habitual: tabla con relación y restricciones (MySQL)

```sql
CREATE TABLE pedidos (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id  INT NOT NULL,
    fecha       DATE NOT NULL DEFAULT (CURRENT_DATE),
    total       DECIMAL(10,2) NOT NULL CHECK (total >= 0),
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);
```

Lo que define:
- `id` se genera solo y es la clave primaria.
- `cliente_id` es obligatorio y apunta a un cliente que **debe existir**.
- `fecha` toma la fecha de hoy si no se indica otra.
- `total` no puede ser negativo.

### Ejemplo completo: evolucionar una tabla con `ALTER`

Partimos de una tabla `clientes` con `id`, `nombre` y `email`. Con el tiempo hay que cambiarla:

```sql
-- 1. Añadir una columna nueva
ALTER TABLE clientes ADD COLUMN ciudad VARCHAR(50);

-- 2. Evitar emails repetidos
ALTER TABLE clientes ADD CONSTRAINT uq_clientes_email UNIQUE (email);

-- 3. Renombrar una columna
ALTER TABLE clientes RENAME COLUMN nombre TO nombre_completo;

-- 4. Eliminar una columna que ya no hace falta
ALTER TABLE clientes DROP COLUMN ciudad;
```

---

## 6. Buenas prácticas

- **Piensa la estructura antes de crearla.** Cambiarla con datos reales es mucho más difícil que acertar desde el principio.
- **Usa nombres claros y consistentes**: minúsculas, sin espacios ni tildes, con guion bajo (`fecha_pedido`). Elige una convención (singular o plural en las tablas) y mantenla.
- **Elige el tipo adecuado y el tamaño justo**: `DECIMAL` para dinero, `DATE` para fechas (no texto), `VARCHAR(255)` en vez de `TEXT` si el texto tiene un límite razonable.
- **Pon siempre clave primaria** en cada tabla.
- **Usa `NOT NULL`** en todo lo que sea obligatorio; es la forma más sencilla de evitar datos incompletos.
- **Ponle nombre a tus restricciones** (`CONSTRAINT uq_clientes_email ...`): así es fácil quitarlas después, y los mensajes de error son más claros.
- **Haz copia de seguridad** antes de un `ALTER` o `DROP` en una base real.
- **Prueba primero en un entorno de desarrollo**, nunca directamente en producción.
- **Guarda tus scripts DDL** en archivos (idealmente con Git) para poder reproducir la base de datos. Los cambios de estructura se suelen gestionar con **migraciones**, que son scripts numerados y versionados.

---

## 7. Diferencias importantes

### `DELETE` vs `TRUNCATE` vs `DROP`

| | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| Qué quita | Filas concretas (o todas) | **Todas** las filas | La tabla **entera** |
| Mantiene la estructura | Sí | Sí | **No** |
| Admite `WHERE` | Sí | No | No |
| Tipo | DML | DDL | DDL |
| Reinicia el autoincremental | No | Normalmente sí | (la tabla desaparece) |
| ¿Se puede deshacer? | Sí, dentro de una transacción | Depende del sistema | Depende del sistema |

### `CREATE` vs `ALTER`

- `CREATE` **construye algo nuevo** y falla si ya existe.
- `ALTER` **cambia algo que ya existe** y falla si no existe.
- Algunos sistemas tienen `CREATE OR REPLACE` (para vistas, funciones…) que lo crea o lo sustituye, según el caso.

### `ALTER TABLE` vs `UPDATE`

- `ALTER TABLE` cambia **la estructura** (añade una columna).
- `UPDATE` cambia **los datos** (rellena esa columna).

Es muy habitual usar los dos juntos: primero `ALTER` para añadir la columna y luego `UPDATE` para rellenarla.

---

## 8. Casos especiales

### El DDL puede no ser transaccional

En una **transacción** puedes deshacer cambios con `ROLLBACK`. Con el DDL depende del sistema:

- **PostgreSQL** y **SQL Server**: el DDL **sí** se puede deshacer dentro de una transacción.
- **MySQL** y **Oracle**: el DDL hace un **`COMMIT` implícito**; en cuanto lo ejecutas, **queda confirmado** y no hay `ROLLBACK`.

Por eso conviene tener mucho cuidado en MySQL y Oracle.

### Añadir una columna `NOT NULL` a una tabla que ya tiene datos

Si la tabla ya tiene filas, esas filas no tendrán valor para la nueva columna, y `NOT NULL` lo prohíbe. Hay dos formas de resolverlo:

```sql
-- Opción 1: darle un valor por defecto
ALTER TABLE clientes ADD COLUMN pais VARCHAR(50) NOT NULL DEFAULT 'España';
```

```sql
-- Opción 2: añadirla permitiendo NULL, rellenarla y luego exigir NOT NULL
ALTER TABLE clientes ADD COLUMN pais VARCHAR(50);
UPDATE clientes SET pais = 'España';
ALTER TABLE clientes ALTER COLUMN pais SET NOT NULL;   -- PostgreSQL
```

### Cambiar el tipo de una columna con datos

Si los valores existentes **no caben o no se pueden convertir** al nuevo tipo (por ejemplo, de `VARCHAR` a `INT` con texto como `"abc"`), el cambio falla. Si cabe, algunos sistemas convierten solos; otros piden una conversión explícita (en PostgreSQL: `ALTER COLUMN col TYPE INT USING col::INT`).

Reducir el tamaño (de `VARCHAR(100)` a `VARCHAR(20)`) puede **truncar o rechazar** datos largos.

### Renombrar o borrar una columna rompe lo que dependa de ella

Consultas, vistas, índices, restricciones y el código de tu aplicación que usen el nombre antiguo **dejarán de funcionar**. Antes de hacerlo, busca dónde se usa.

### `ALTER TABLE` en tablas muy grandes

En tablas con millones de filas, algunos `ALTER` **bloquean la tabla** mientras se ejecutan (nadie puede escribir, y a veces ni leer). Puede tardar minutos u horas. En producción se hacen en horarios de poco uso o con herramientas específicas de cada sistema.

### SQLite tiene un `ALTER TABLE` muy limitado

SQLite solo permite **renombrar** tablas y columnas, **añadir** columnas y (en versiones recientes) **borrar** columnas. Para cambiar el tipo o las restricciones hay que:

1. Crear una tabla nueva con la estructura correcta.
2. Copiar los datos con `INSERT INTO ... SELECT ...`.
3. Borrar la tabla antigua.
4. Renombrar la nueva con el nombre original.

### Renombrar en SQL Server

SQL Server no usa `RENAME`. Se utiliza el procedimiento `sp_rename`:

```sql
EXEC sp_rename 'clientes.telefono', 'movil', 'COLUMN';
EXEC sp_rename 'clientes', 'usuarios';
```

### No se puede borrar una tabla a la que apunta una clave foránea

Si otra tabla tiene una clave foránea hacia ella, `DROP TABLE` da error. Hay que borrar primero la tabla dependiente (o la restricción), o usar `DROP TABLE ... CASCADE` donde esté disponible (PostgreSQL, Oracle), sabiendo que elimina también lo que dependa de ella.

---

## 9. Resumen

- **`CREATE`** crea objetos (base de datos, tabla, índice, vista…); **`ALTER`** los modifica; **`DROP`** los elimina. Las tres son **DDL**: cambian la **estructura**, no los datos.
- `CREATE TABLE` define columnas con **nombre + tipo + restricciones** (`PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `DEFAULT`, `CHECK`, `FOREIGN KEY`).
- `IF NOT EXISTS` / `IF EXISTS` evitan errores si el objeto ya existe (o no existe), pero no están en todos los sistemas.
- `ALTER TABLE` permite **añadir, borrar y renombrar columnas**, cambiar tipos o valores por defecto, y añadir o quitar restricciones.
- Lo que **más cambia entre sistemas**: los **autoincrementales**, la forma de **cambiar el tipo** de una columna y el soporte de `ALTER` (SQLite es muy limitado).
- Usa `DECIMAL` para dinero, `NOT NULL` para lo obligatorio y **nombres claros** para las restricciones.
- En **MySQL y Oracle** el DDL se confirma solo (sin `ROLLBACK`); en **PostgreSQL y SQL Server** sí se puede deshacer en una transacción.
- `DROP` y `TRUNCATE` **borran datos de forma definitiva**: copia de seguridad antes de tocar nada importante.
- Las restricciones se ven a fondo en [[SQL 13 - Claves y restricciones]] y los índices en [[SQL 14 - Índices]].