# 09 - INSERT

> [!info] ¿Qué es?
> **INSERT** es la sentencia de SQL que **añade filas nuevas** a una tabla. Es la "C" de **CRUD** (*Create*): con ella se guardan datos nuevos, como un usuario que se registra, un pedido que se hace o un producto que se da de alta.

---

## 1. Antes de empezar

Conviene tener claro:

- Qué es una tabla, una fila y una columna → [[SQL 01 - Fundamentos]]
- `SELECT` (se usa dentro de algunos `INSERT`) → [[SQL 02 - SELECT]]
- Subconsultas → [[SQL 08 - Subconsultas]]

Los restricciones de las columnas (clave primaria, `NOT NULL`, `UNIQUE`, `DEFAULT`, clave foránea) se explican a fondo en [[SQL 13 - Claves y restricciones]], pero aquí es importante saber que **un `INSERT` falla si no las cumple**.

Tabla de ejemplo de toda la nota:

**`usuarios`**

| Columna | Tipo | Restricciones |
|---------|------|---------------|
| id | INT | Clave primaria, se genera sola (autoincremental) |
| nombre | VARCHAR(50) | `NOT NULL` |
| email | VARCHAR(100) | `NOT NULL`, `UNIQUE` |
| edad | INT | Puede ser `NULL` |
| activo | BOOLEAN | `DEFAULT TRUE` |
| creado_en | TIMESTAMP | `DEFAULT CURRENT_TIMESTAMP` |

---

## 2. Concepto fundamental

Cada `INSERT` crea **una o más filas nuevas**. Para ello hay que decir tres cosas:

1. **En qué tabla** se inserta.
2. **En qué columnas** se van a poner datos.
3. **Qué valores** va a tener cada columna.

```sql
INSERT INTO usuarios (nombre, email, edad)
VALUES ('Ana', 'ana@correo.com', 25);
```

Después de ejecutarlo, la tabla tiene una fila nueva:

| id | nombre | email | edad | activo | creado_en |
|----|--------|-------|------|--------|-----------|
| 1 | Ana | ana@correo.com | 25 | TRUE | 2026-10-14 10:30:00 |

Fíjate en que **no hemos dicho nada** de `id`, `activo` ni `creado_en`: la base de datos los ha rellenado sola (autoincremento y valores por defecto).

> [!tip] La regla clave
> Las columnas que no nombras en el `INSERT` reciben su valor por defecto (`DEFAULT`), se autogeneran, o quedan en `NULL`. Si una columna es `NOT NULL` y no tiene valor por defecto, **tienes que darle valor** o el `INSERT` falla.

---

## 3. Sintaxis / estructura

### Forma recomendada: nombrando las columnas

```sql
INSERT INTO tabla (columna1, columna2, columna3)
VALUES (valor1, valor2, valor3);
```

### Forma corta: sin nombrar las columnas

```sql
INSERT INTO tabla
VALUES (valor1, valor2, valor3, ...);
```

Aquí hay que dar **un valor para cada columna de la tabla, en el mismo orden en que se definieron**.

### Reglas básicas

1. El **número de valores** debe coincidir con el **número de columnas** nombradas.
2. El **orden de los valores** debe corresponder al orden de las columnas nombradas.
3. Los **textos y fechas** van entre comillas simples: `'Ana'`, `'2026-10-14'`. Los números y `NULL` no llevan comillas.
4. Las columnas se pueden nombrar **en cualquier orden**, siempre que los valores sigan ese mismo orden.
5. Para escribir una comilla simple dentro de un texto, se duplica: `'O''Brien'`.

> [!note] Comillas en SQL
> En SQL estándar los textos van con **comillas simples** (`'Ana'`). Las comillas dobles se reservan para nombres de columnas/tablas en PostgreSQL, SQL Server y Oracle, aunque MySQL las acepta también para textos. Con comillas simples funciona en todos los motores.

---

## 4. Elementos / propiedades / características

### 4.1 Insertar una sola fila

```sql
INSERT INTO usuarios (nombre, email, edad)
VALUES ('Luis', 'luis@correo.com', 31);
```

### 4.2 Insertar varias filas a la vez

Se separan los grupos de valores con comas. Es **más rápido** que hacer un `INSERT` por fila.

```sql
INSERT INTO usuarios (nombre, email, edad)
VALUES
    ('Marta', 'marta@correo.com', 28),
    ('Pablo', 'pablo@correo.com', 35),
    ('Sara',  'sara@correo.com',  22);
```

> [!note] Disponibilidad
> Funciona en MySQL, PostgreSQL, SQL Server (2008+), SQLite (3.7.11+) y Oracle 23c+. En versiones antiguas de Oracle se necesita `INSERT ALL`.

Además es **atómico**: si una de las filas falla (por ejemplo, un email duplicado), **no se inserta ninguna** de ese grupo.

### 4.3 Insertar solo algunas columnas

Las que no nombres usan su valor por defecto, se autogeneran o quedan en `NULL`.

```sql
INSERT INTO usuarios (nombre, email)
VALUES ('Diego', 'diego@correo.com');
-- edad → NULL, activo → TRUE, creado_en → fecha actual, id → siguiente número
```

### 4.4 Insertar sin nombrar columnas

```sql
INSERT INTO usuarios
VALUES (10, 'Elena', 'elena@correo.com', 40, TRUE, '2026-10-14 09:00:00');
```

Funciona, pero es **frágil**: si más adelante alguien añade o reordena una columna, tu `INSERT` se rompe o, peor, mete datos en la columna equivocada.

### 4.5 NULL y DEFAULT explícitos

```sql
INSERT INTO usuarios (nombre, email, edad, activo)
VALUES ('Raúl', 'raul@correo.com', NULL, DEFAULT);
```

- `NULL` → "no hay valor" (solo si la columna lo permite).
- `DEFAULT` → "usa el valor por defecto de la columna".

### 4.6 Columnas autoincrementales (generar el id)

Normalmente **no se da valor al `id`**: lo genera la base de datos. La forma de definirlo cambia según el motor:

| Motor | Cómo se define la columna |
|-------|---------------------------|
| MySQL / MariaDB | `AUTO_INCREMENT` |
| PostgreSQL | `GENERATED ALWAYS AS IDENTITY` (o `SERIAL`) |
| SQL Server | `IDENTITY(1,1)` |
| SQLite | `INTEGER PRIMARY KEY` (se autoincrementa solo) |
| Oracle 12c+ | `GENERATED ALWAYS AS IDENTITY` |

Para **conocer el id que se acaba de generar**:

| Motor | Cómo obtenerlo |
|-------|----------------|
| MySQL | `SELECT LAST_INSERT_ID();` |
| PostgreSQL | `INSERT ... RETURNING id;` |
| SQL Server | `SELECT SCOPE_IDENTITY();` (o cláusula `OUTPUT`) |
| SQLite | `SELECT last_insert_rowid();` |

### 4.7 INSERT ... SELECT (insertar el resultado de una consulta)

En vez de `VALUES`, se puede insertar **lo que devuelva un `SELECT`**. Muy útil para copiar o transformar datos entre tablas.

```sql
INSERT INTO usuarios_archivo (nombre, email)
SELECT nombre, email
FROM usuarios
WHERE activo = FALSE;
```

- No lleva `VALUES` ni paréntesis alrededor del `SELECT`.
- El número y el tipo de columnas del `SELECT` deben coincidir con las columnas indicadas.

### 4.8 Insertar y devolver la fila: RETURNING / OUTPUT

Algunos motores permiten recuperar los datos insertados en la misma sentencia:

```sql
-- PostgreSQL, SQLite 3.35+, MariaDB 10.5+
INSERT INTO usuarios (nombre, email)
VALUES ('Lucía', 'lucia@correo.com')
RETURNING id, creado_en;
```

```sql
-- SQL Server
INSERT INTO usuarios (nombre, email)
OUTPUT INSERTED.id, INSERTED.creado_en
VALUES ('Lucía', 'lucia@correo.com');
```

MySQL **no** tiene `RETURNING`: se usa `LAST_INSERT_ID()`.

### 4.9 Qué pasa si hay un conflicto (clave duplicada)

Si insertas un valor que viola una restricción `UNIQUE` o `PRIMARY KEY`, el `INSERT` **falla con error**. Cada motor ofrece formas de manejarlo:

| Objetivo | MySQL | PostgreSQL | SQLite | SQL Server / Oracle |
|----------|-------|------------|--------|---------------------|
| **Ignorar** la fila duplicada | `INSERT IGNORE` | `ON CONFLICT DO NOTHING` | `INSERT OR IGNORE` / `ON CONFLICT DO NOTHING` | `MERGE` |
| **Actualizar** si ya existe (*upsert*) | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT (col) DO UPDATE` | `ON CONFLICT (col) DO UPDATE` | `MERGE` |
| **Reemplazar** la fila entera | `REPLACE INTO` | — | `INSERT OR REPLACE` | — |

Ejemplos de *upsert* (insertar o, si ya existe, actualizar):

```sql
-- MySQL
INSERT INTO usuarios (email, nombre)
VALUES ('ana@correo.com', 'Ana García')
ON DUPLICATE KEY UPDATE nombre = VALUES(nombre);

-- PostgreSQL y SQLite
INSERT INTO usuarios (email, nombre)
VALUES ('ana@correo.com', 'Ana García')
ON CONFLICT (email) DO UPDATE SET nombre = EXCLUDED.nombre;
```

> [!warning] Peligro de REPLACE
> `REPLACE INTO` **borra la fila existente y crea una nueva**. Eso puede romper claves foráneas, disparar *triggers* de borrado y cambiar el `id`. Casi siempre es mejor un *upsert* con `ON DUPLICATE KEY UPDATE` / `ON CONFLICT`.

### 4.10 Insertar valores calculados y funciones

En `VALUES` se pueden usar expresiones y funciones:

```sql
INSERT INTO usuarios (nombre, email, creado_en)
VALUES ('Marcos', LOWER('MARCOS@CORREO.COM'), CURRENT_TIMESTAMP);
```

### 4.11 Insertar con subconsulta como valor

```sql
INSERT INTO pedidos (cliente_id, total)
VALUES (
    (SELECT id FROM clientes WHERE email = 'ana@correo.com'),
    59.90
);
```

La subconsulta debe devolver **un solo valor**.

---

## 5. Ejemplos prácticos

### Ejemplo básico: una fila

```sql
INSERT INTO usuarios (nombre, email, edad)
VALUES ('Ana', 'ana@correo.com', 25);
```

### Ejemplo habitual: varias filas a la vez

```sql
INSERT INTO usuarios (nombre, email, edad)
VALUES
    ('Luis',  'luis@correo.com',  31),
    ('Marta', 'marta@correo.com', 28),
    ('Pablo', 'pablo@correo.com', 35);
```

### Ejemplo completo: copiar datos filtrados con INSERT ... SELECT

```sql
-- Mover a una tabla de archivo los usuarios inactivos de más de 40 años
INSERT INTO usuarios_archivo (nombre, email, edad)
SELECT nombre, email, edad
FROM usuarios
WHERE activo = FALSE AND edad > 40;
```

---

## 6. Buenas prácticas

1. **Nombra siempre las columnas** en el `INSERT`: el código sigue funcionando si la tabla cambia.
2. **No des valor a columnas autogeneradas** (`id`, fechas por defecto): deja que la base de datos lo haga.
3. **Inserta varias filas en un solo `INSERT`** en lugar de muchos `INSERT` sueltos: es mucho más rápido.
4. **Usa transacciones** cuando insertes en varias tablas relacionadas, para que o se guarde todo o nada → [[SQL 17 - Transacciones]].
5. **Inserta primero las filas "padre"** y después las "hijas" (por ejemplo, el cliente antes que sus pedidos), por las claves foráneas.
6. **Nunca construyas el `INSERT` pegando texto del usuario** en el SQL: usa **consultas preparadas con parámetros** para evitar inyección SQL.
7. **Usa `NULL` solo cuando "no hay dato"** de verdad; no lo uses como sustituto de cadena vacía o cero.
8. **Maneja los conflictos de forma explícita** (`ON CONFLICT`, `ON DUPLICATE KEY`) en vez de capturar el error.

```sql
-- ❌ Peligroso: concatenar datos del usuario (inyección SQL)
-- "INSERT INTO usuarios (nombre) VALUES ('" + nombreIntroducido + "')"

-- ✅ Seguro: consulta preparada con parámetros (el valor viaja aparte)
INSERT INTO usuarios (nombre) VALUES (?);
```

---

## 7. Diferencias importantes

### INSERT con VALUES vs INSERT ... SELECT

| | `INSERT ... VALUES` | `INSERT ... SELECT` |
|---|---------------------|---------------------|
| Origen de los datos | Los escribes tú | Salen de otra consulta |
| Número de filas | Las que listes | Las que devuelva el `SELECT` |
| Uso típico | Alta de datos nuevos | Copiar, archivar, migrar |

### INSERT vs UPDATE

- `INSERT` **crea filas nuevas**.
- `UPDATE` **modifica filas que ya existen** → [[SQL 10 - UPDATE]].

### INSERT con y sin lista de columnas

| | Con columnas | Sin columnas |
|---|--------------|--------------|
| Seguridad ante cambios en la tabla | ✅ Alta | ❌ Frágil |
| Puedes omitir columnas | ✅ Sí | ❌ No (todas obligatorias) |
| Recomendado | ✅ Sí | Solo en pruebas rápidas |

### Upsert según el motor

> [!warning] Obsoleto / legado
> `INSERT IGNORE` y `REPLACE INTO` son propios de MySQL (y `INSERT OR ...` de SQLite). El estándar moderno es `ON CONFLICT` (PostgreSQL, SQLite) o `MERGE` (SQL estándar, SQL Server, Oracle). Si necesitas código portable, usa `MERGE` o resuelve el conflicto desde tu aplicación.

---

## 8. Casos especiales

### 8.1 Fallo de una restricción

El `INSERT` completo se cancela y devuelve error si:

| Causa | Ejemplo |
|-------|---------|
| Clave primaria o `UNIQUE` repetida | Email que ya existe |
| `NOT NULL` sin valor | Falta el `nombre` |
| Clave foránea inexistente | `cliente_id` que no existe en `clientes` |
| Tipo de dato incorrecto | Texto en una columna numérica |
| Texto demasiado largo | 80 caracteres en un `VARCHAR(50)` |
| `CHECK` no cumplido | `edad = -5` con `CHECK (edad >= 0)` |

### 8.2 Insertar fechas

Usa el formato estándar **`'AAAA-MM-DD'`** (y `'AAAA-MM-DD HH:MM:SS'` para fecha y hora). Es el único que funciona igual en todos los motores.

```sql
INSERT INTO pedidos (cliente_id, fecha) VALUES (1, '2026-10-14');
```

### 8.3 Insertar booleanos

- PostgreSQL: `TRUE` / `FALSE`.
- MySQL: `TRUE` / `FALSE` (son alias de `1` y `0`).
- SQL Server y Oracle (antiguo): no tienen tipo booleano real; se usa `BIT` (`1`/`0`) o `NUMBER(1)`.

### 8.4 Límite de filas por INSERT

Insertar miles de filas en un solo `INSERT` puede superar límites del motor (SQL Server permite máximo **1000** filas por `VALUES`; MySQL tiene `max_allowed_packet`). Para cargas grandes, divide en lotes o usa herramientas de importación (`LOAD DATA`, `COPY`, `BULK INSERT`).

### 8.5 Insertar en una vista

Solo es posible en vistas "simples" (una sola tabla, sin agregaciones). Se ve en [[SQL 18 - Vistas]].

### 8.6 Triggers

Un `INSERT` puede disparar **triggers** (código automático que se ejecuta al insertar). Se estudian en [[SQL 19 - Procedimientos y triggers]].

### 8.7 Valor insertado mayor del permitido (modo estricto)

En MySQL con modo estricto activado, un valor fuera de rango da error. Con el modo estricto desactivado, **lo recorta en silencio**, lo que puede dejar datos corruptos sin que te enteres. Mantén siempre el modo estricto activado.

---

## 9. Resumen

- **`INSERT INTO`** añade filas nuevas a una tabla.
- Sintaxis recomendada: `INSERT INTO tabla (columnas) VALUES (valores);`
- Los **valores** deben coincidir en **número y orden** con las columnas nombradas.
- Textos y fechas van con **comillas simples**; números y `NULL`, sin comillas.
- Las columnas omitidas usan su `DEFAULT`, se autogeneran o quedan en `NULL` (si son `NOT NULL` sin valor por defecto, el `INSERT` falla).
- Se pueden insertar **varias filas** a la vez separando los grupos con comas.
- **`INSERT ... SELECT`** inserta el resultado de una consulta.
- Un `INSERT` falla si viola alguna restricción: clave duplicada, `NOT NULL`, clave foránea, `CHECK` o tipo incorrecto.
- Para manejar duplicados: **upsert** (`ON CONFLICT` / `ON DUPLICATE KEY` / `MERGE`); evita `REPLACE`.
- Para obtener el id generado: `RETURNING` (PostgreSQL), `LAST_INSERT_ID()` (MySQL), `SCOPE_IDENTITY()` (SQL Server).
- **Nombra siempre las columnas** y usa **consultas preparadas** para evitar inyección SQL.