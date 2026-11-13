# 13 - Claves y restricciones

> [!info] ¿Qué es?
> Las **restricciones** (*constraints*) son reglas que defines sobre las tablas para que la base de datos **rechace datos incorrectos** por sí sola. Las **claves** (primaria y foránea) son un tipo especial de restricción que sirven para **identificar filas** y para **conectar tablas** entre sí. Gracias a ellas, la integridad de los datos no depende de que cada programa que escribe en la base de datos se acuerde de validar todo.

---

## 1. Antes de empezar

**Qué deberías saber ya:**
- Crear y modificar tablas con `CREATE TABLE` y `ALTER TABLE` → [[SQL 12 - CREATE y ALTER]]
- Qué es `NULL` (ausencia de valor, no es 0 ni cadena vacía) → [[SQL 01 - Fundamentos]]

**Idea clave:** una restricción se comprueba **cada vez que insertas o modificas** datos (`INSERT`, `UPDATE`, y en las claves foráneas también `DELETE`). Si el dato incumple la regla, la operación **falla entera** y no se guarda nada.

**Tabla de ejemplo que usaremos en la nota:**

```sql
-- clientes(id, email, nombre)
-- pedidos(id, cliente_id, fecha, total)
```

> [!warning] Dialectos
> La idea es igual en todos los motores, pero algunos detalles de sintaxis cambian entre MySQL, PostgreSQL, SQL Server, SQLite y Oracle. Donde cambian, está indicado.

---

## 2. Concepto fundamental

### Las restricciones disponibles

| Restricción | Qué garantiza |
|---|---|
| `NOT NULL` | El campo **no puede quedar vacío** (sin valor) |
| `UNIQUE` | No hay **dos filas con el mismo valor** en ese campo (o combinación) |
| `PRIMARY KEY` | Identifica cada fila de forma **única y obligatoria** (= `UNIQUE` + `NOT NULL`) |
| `FOREIGN KEY` | El valor debe **existir en otra tabla** (referencia válida) |
| `CHECK` | El valor cumple una **condición** que tú escribes |
| `DEFAULT` | Valor que se pone **si no indicas ninguno** (técnicamente no valida, rellena) |

### Dónde se pueden definir

1. **A nivel de columna:** junto a la columna, afecta solo a esa columna.
2. **A nivel de tabla:** después de todas las columnas, sirve para **varias columnas a la vez** (claves compuestas, `CHECK` entre columnas...).
3. **Después de crear la tabla**, con `ALTER TABLE`.

### Poner nombre a las restricciones

Con `CONSTRAINT nombre` le das un nombre propio a la restricción. Si no lo haces, el motor le inventa uno (poco legible). Con nombre:
- Los **mensajes de error** se entienden mejor.
- Puedes **borrarla o modificarla** fácilmente después.

---

## 3. Sintaxis / estructura

### Estructura general al crear una tabla

```sql
CREATE TABLE nombre_tabla (
    columna1 TIPO restricciones_de_columna,
    columna2 TIPO restricciones_de_columna,

    CONSTRAINT nombre_pk  PRIMARY KEY (columna1),
    CONSTRAINT nombre_uq  UNIQUE (columna2),
    CONSTRAINT nombre_fk  FOREIGN KEY (columna3) REFERENCES otra_tabla (id),
    CONSTRAINT nombre_chk CHECK (condicion)
);
```

### Añadir y quitar restricciones en una tabla existente

```sql
-- Añadir
ALTER TABLE nombre_tabla ADD CONSTRAINT nombre_uq UNIQUE (columna);

-- Quitar (estándar SQL, PostgreSQL, SQL Server, Oracle)
ALTER TABLE nombre_tabla DROP CONSTRAINT nombre_uq;
```

> [!warning] Quitar restricciones: cambia según el motor
> | Motor | Cómo se quita |
> |---|---|
> | PostgreSQL, SQL Server, Oracle | `DROP CONSTRAINT nombre` |
> | MySQL – clave foránea | `DROP FOREIGN KEY nombre` |
> | MySQL – `UNIQUE` | `DROP INDEX nombre` |
> | MySQL – clave primaria | `DROP PRIMARY KEY` |
> | MySQL – `CHECK` (8.0.16+) | `DROP CHECK nombre` (o `DROP CONSTRAINT`) |
> | SQLite | **No permite** casi ninguna: hay que recrear la tabla |

---

## 4. Elementos / propiedades / características

### 4.1 `NOT NULL`

Obliga a que la columna siempre tenga valor.

```sql
nombre VARCHAR(100) NOT NULL
```

- Si insertas una fila sin ese campo (y no tiene `DEFAULT`), da error.
- Es la restricción más simple y una de las más útiles: **por defecto, ponla en todo lo que realmente sea obligatorio**.

### 4.2 `DEFAULT`

Valor automático cuando no indicas ninguno en el `INSERT`.

```sql
estado VARCHAR(20) NOT NULL DEFAULT 'pendiente',
creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
```

- Se usa mucho junto a `NOT NULL`.
- Solo se aplica cuando **omites** la columna (o escribes `DEFAULT` explícitamente). Si escribes `NULL` a mano, se guarda `NULL` (o da error si es `NOT NULL`).

### 4.3 `UNIQUE`

Impide valores repetidos.

```sql
email VARCHAR(150) NOT NULL UNIQUE
```

- Una tabla puede tener **todas las `UNIQUE` que quieras** (a diferencia de la clave primaria, que es una sola).
- Puede ser **compuesta** (varias columnas): lo que no se puede repetir es la **combinación**.

```sql
CONSTRAINT uq_matricula UNIQUE (alumno_id, asignatura_id)
```

- **Comportamiento con `NULL`:** en la mayoría de motores (MySQL, PostgreSQL, SQLite, Oracle con matices) puedes tener **varios `NULL`** en una columna `UNIQUE`, porque `NULL` no se considera igual a otro `NULL`. **SQL Server** es la excepción clásica: solo admite **un** `NULL`.

### 4.4 `PRIMARY KEY` (clave primaria)

Es el **identificador oficial** de cada fila.

```sql
id INT PRIMARY KEY
```

**Reglas:**
- **Única** y **no nula** (obligatoria).
- **Solo una** por tabla (pero puede estar formada por varias columnas → *clave compuesta*).
- Se crea automáticamente un **índice** sobre ella (búsquedas muy rápidas por clave) → [[SQL 14 - Índices]].

**Clave compuesta (varias columnas):**

```sql
CONSTRAINT pk_matricula PRIMARY KEY (alumno_id, asignatura_id)
```

La que no se puede repetir es la **pareja completa**; cada valor suelto sí puede repetirse.

**Clave natural vs. clave sustituta (artificial):**

| Tipo | Qué es | Ejemplo | Cuándo |
|---|---|---|---|
| **Natural** | Un dato real que ya identifica | DNI, ISBN, código de país | Solo si es **estable y nunca cambia** |
| **Sustituta** (*surrogate*) | Un número inventado sin significado | `id` autoincremental | **Lo más habitual**, porque nunca cambia |

> [!tip] Regla práctica
> Lo normal es usar un `id` numérico sustituto como clave primaria y proteger el dato real (DNI, email...) con `UNIQUE`.

### 4.5 Autoincremento (identificadores automáticos)

Para que la base de datos genere el `id` sola. **La sintaxis cambia mucho según el motor:**

| Motor | Sintaxis |
|---|---|
| MySQL / MariaDB | `id INT AUTO_INCREMENT PRIMARY KEY` |
| PostgreSQL (moderno) | `id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| PostgreSQL (clásico) | `id SERIAL PRIMARY KEY` |
| SQL Server | `id INT IDENTITY(1,1) PRIMARY KEY` |
| SQLite | `id INTEGER PRIMARY KEY` (ya se autoincrementa solo; `AUTOINCREMENT` es opcional) |
| Oracle (12c+) | `id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |

> [!warning] Los huecos son normales
> Si borras una fila o una inserción falla, ese número **no se reutiliza**. No esperes ids consecutivos sin saltos.

### 4.6 `FOREIGN KEY` (clave foránea)

Es una columna que **apunta a la clave primaria (o a una columna `UNIQUE`) de otra tabla**. Garantiza que la relación entre tablas sea **siempre válida**.

```sql
CREATE TABLE pedidos (
    id         INT PRIMARY KEY,
    cliente_id INT NOT NULL,
    total      DECIMAL(10,2),

    CONSTRAINT fk_pedidos_clientes
        FOREIGN KEY (cliente_id) REFERENCES clientes (id)
);
```

**Qué hace cumplir:**
1. No puedes **insertar** un pedido con un `cliente_id` que no exista en `clientes`.
2. No puedes **borrar** (ni cambiar el `id` de) un cliente que tenga pedidos... **salvo que hayas definido qué hacer** (siguiente punto).

**Detalles:**
- La tabla con la clave foránea se llama **hija**; la referenciada, **padre**.
- Los tipos de dato de ambas columnas deben ser **compatibles** (mismo tipo y tamaño recomendado).
- La clave foránea **puede ser `NULL`** (si no la marcas `NOT NULL`), y significa "esta fila no está relacionada con nadie". Úsalo solo si la relación es opcional.

### 4.7 Acciones referenciales: `ON DELETE` y `ON UPDATE`

Definen **qué pasa en la tabla hija** cuando se borra o cambia la fila padre.

```sql
FOREIGN KEY (cliente_id) REFERENCES clientes (id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
```

| Acción | Qué ocurre al borrar/cambiar el padre |
|---|---|
| `RESTRICT` | **Prohíbe** la operación si hay hijos. Comprueba al instante |
| `NO ACTION` | Igual que `RESTRICT` en la práctica (se comprueba al final de la sentencia). **Es el valor por defecto** en la mayoría de motores |
| `CASCADE` | **Propaga** el cambio: si borras al padre, se **borran sus hijos**; si cambias su id, se actualiza en los hijos |
| `SET NULL` | Pone la clave foránea de los hijos a `NULL` (la columna debe admitir `NULL`) |
| `SET DEFAULT` | Pone el valor `DEFAULT` en los hijos (no lo admiten todos: MySQL/InnoDB no) |

> [!warning] Cuidado con `CASCADE`
> `ON DELETE CASCADE` es cómodo pero **peligroso**: un solo `DELETE` puede arrastrar miles de filas de tablas relacionadas. Úsalo solo cuando los hijos **no tienen sentido sin el padre** (por ejemplo, las líneas de un pedido sin el pedido).

### 4.8 `CHECK`

Obliga a que se cumpla una **condición** lógica.

```sql
precio   DECIMAL(10,2) CHECK (precio >= 0),
edad     INT,
estado   VARCHAR(20),

CONSTRAINT chk_edad   CHECK (edad BETWEEN 0 AND 120),
CONSTRAINT chk_estado CHECK (estado IN ('pendiente', 'enviado', 'entregado'))
```

- Puede comparar **varias columnas de la misma fila**: `CHECK (fecha_fin >= fecha_inicio)`.
- **No puede** consultar otras tablas (para eso están las claves foráneas) ni usar funciones no deterministas en muchos motores.
- Si la condición da `NULL` (porque un dato es `NULL`), el `CHECK` **se considera cumplido**. Para obligar a que haya valor, combínalo con `NOT NULL`.

> [!warning] Compatibilidad
> - **MySQL**: antes de la 8.0.16 el `CHECK` se **aceptaba pero se ignoraba** (¡no validaba nada!). Desde 8.0.16 sí funciona.
> - **SQLite**: sí lo valida.
> - **PostgreSQL, SQL Server y Oracle**: funcionan desde siempre.

---

## 5. Ejemplos prácticos

### Ejemplo básico: tabla con las restricciones habituales

```sql
CREATE TABLE clientes (
    id        INT AUTO_INCREMENT PRIMARY KEY,   -- (MySQL)
    email     VARCHAR(150) NOT NULL UNIQUE,
    nombre    VARCHAR(100) NOT NULL,
    activo    BOOLEAN NOT NULL DEFAULT TRUE,
    creado_en TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Ejemplo habitual: dos tablas relacionadas con nombres claros

```sql
CREATE TABLE pedidos (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    fecha      DATE NOT NULL,
    total      DECIMAL(10,2) NOT NULL DEFAULT 0,

    CONSTRAINT chk_pedidos_total CHECK (total >= 0),
    CONSTRAINT fk_pedidos_clientes
        FOREIGN KEY (cliente_id) REFERENCES clientes (id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

### Ejemplo completo: relación muchos a muchos con clave compuesta

Un alumno puede estar en muchas asignaturas y una asignatura tiene muchos alumnos. Se resuelve con una **tabla intermedia** cuya clave primaria es la pareja de claves foráneas:

```sql
CREATE TABLE matriculas (
    alumno_id     INT NOT NULL,
    asignatura_id INT NOT NULL,
    nota          DECIMAL(4,2),

    CONSTRAINT pk_matriculas PRIMARY KEY (alumno_id, asignatura_id),
    CONSTRAINT chk_nota CHECK (nota BETWEEN 0 AND 10),
    CONSTRAINT fk_mat_alumnos
        FOREIGN KEY (alumno_id) REFERENCES alumnos (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_mat_asignaturas
        FOREIGN KEY (asignatura_id) REFERENCES asignaturas (id)
        ON DELETE CASCADE
);
```

Con esto, un alumno **no puede matricularse dos veces** en la misma asignatura, y si se borra el alumno, desaparecen sus matrículas. Más sobre este patrón en [[SQL 15 - Relaciones]].

---

## 6. Buenas prácticas

- **Pon `NOT NULL` por defecto** y permite `NULL` solo cuando "no hay dato" tenga un significado real.
- **Toda tabla debe tener clave primaria.** Sin ella no puedes identificar filas de forma fiable.
- **Usa un `id` sustituto** como clave primaria y protege los datos reales con `UNIQUE`.
- **Pon nombre a tus restricciones** con un patrón fijo y consistente:
  - `pk_tabla`, `uq_tabla_columna`, `fk_hija_padre`, `chk_tabla_regla`
- **Define siempre claves foráneas** para las relaciones; no te fíes de que la aplicación "ya lo controla".
- **Elige con cuidado `ON DELETE`:** `RESTRICT` por defecto (lo más seguro); `CASCADE` solo si los hijos dependen totalmente del padre; `SET NULL` si la relación es opcional.
- **Mismo tipo en la clave foránea y en la clave que referencia** (por ejemplo, ambas `INT`; ambas con o sin signo en MySQL).
- **Indexa las claves foráneas** si tu motor no lo hace solo (PostgreSQL y SQL Server **no** crean índice automático en la clave foránea; MySQL/InnoDB sí). Acelera `JOIN` y borrados → [[SQL 14 - Índices]].
- **Usa `CHECK` para reglas simples** y deja las reglas muy complejas a la aplicación o a triggers.

---

## 7. Diferencias importantes

### `PRIMARY KEY` vs. `UNIQUE`

| | `PRIMARY KEY` | `UNIQUE` |
|---|---|---|
| Cuántas por tabla | **Una** | **Varias** |
| Admite `NULL` | **No** | **Sí** (con matices según motor) |
| Función | Identificador principal | Evitar duplicados en otro dato |

### Restricción a nivel de columna vs. a nivel de tabla

| | Columna | Tabla |
|---|---|---|
| Dónde se escribe | Junto a la columna | Al final, con `CONSTRAINT` |
| Varias columnas | No | **Sí** |
| Ponerle nombre | Posible, pero menos cómodo | Natural y claro |

### `RESTRICT` vs. `NO ACTION`

Casi lo mismo. `RESTRICT` falla de inmediato; `NO ACTION` falla al terminar la sentencia (útil con restricciones diferidas en PostgreSQL). En MySQL/InnoDB se comportan igual.

### Clave foránea vs. `CHECK`

La clave foránea valida contra **datos de otra tabla**; el `CHECK` valida contra **una condición sobre la propia fila**.

---

## 8. Casos especiales

### Añadir restricciones a una tabla que ya tiene datos

Si la tabla ya tiene filas que **incumplen** la nueva regla, el `ALTER TABLE` **falla**. Tienes que limpiar antes los datos.

```sql
-- ¿Hay emails duplicados antes de añadir UNIQUE?
SELECT email, COUNT(*) 
FROM clientes 
GROUP BY email 
HAVING COUNT(*) > 1;

-- ¿Hay pedidos huérfanos antes de añadir la clave foránea?
SELECT p.id
FROM pedidos p
LEFT JOIN clientes c ON c.id = p.cliente_id
WHERE c.id IS NULL;
```

> [!tip] Truco
> Esas dos consultas son la forma estándar de "encontrar lo que rompe la restricción" antes de crearla.

### Clave foránea que apunta a la misma tabla (autorreferencia)

Sirve para jerarquías (empleado → jefe, categoría → categoría padre).

```sql
CREATE TABLE empleados (
    id      INT PRIMARY KEY,
    nombre  VARCHAR(100) NOT NULL,
    jefe_id INT NULL,
    CONSTRAINT fk_empleados_jefe
        FOREIGN KEY (jefe_id) REFERENCES empleados (id)
);
```

El jefe máximo tiene `jefe_id = NULL`. Por eso esa columna **debe** admitir `NULL`.

### Claves foráneas circulares

Si la tabla A apunta a B y B apunta a A, no puedes insertar en ninguna primero. Soluciones: hacer una de las dos columnas **opcional (`NULL`)**, crear la clave foránea **después** con `ALTER TABLE`, o usar restricciones **diferibles** (PostgreSQL, Oracle).

### Clave foránea hacia una clave compuesta

La clave foránea debe tener **las mismas columnas, en el mismo orden**:

```sql
FOREIGN KEY (alumno_id, asignatura_id)
    REFERENCES matriculas (alumno_id, asignatura_id)
```

### SQLite: las claves foráneas vienen **desactivadas**

Hay que activarlas **en cada conexión**:

```sql
PRAGMA foreign_keys = ON;
```

Si no, SQLite acepta datos que rompen las relaciones sin avisar.

### SQL Server: ciclos de `CASCADE`

SQL Server **rechaza** crear varias rutas de `CASCADE` que puedan llegar a la misma tabla ("*may cause cycles or multiple cascade paths*"). Solución: usar `NO ACTION` en una de las rutas o gestionarlo con triggers.

### Desactivar temporalmente las comprobaciones

Útil al cargar grandes volúmenes de datos, pero **peligroso** (puedes dejar datos inválidos):

```sql
-- MySQL
SET FOREIGN_KEY_CHECKS = 0;
-- ... cargar datos ...
SET FOREIGN_KEY_CHECKS = 1;
```

> [!warning] Solo para casos controlados
> Al reactivarlas, MySQL **no revisa** lo que ya metiste. Comprueba tú mismo que los datos son coherentes.

### Qué error da cada violación (para reconocerlos)

| Qué incumples | Mensaje típico |
|---|---|
| `NOT NULL` | "*Column cannot be null*" / "*null value violates not-null constraint*" |
| `UNIQUE` / `PRIMARY KEY` | "*Duplicate entry*" / "*duplicate key value violates unique constraint*" |
| `FOREIGN KEY` | "*Cannot add or update a child row*" / "*Cannot delete or update a parent row*" |
| `CHECK` | "*Check constraint ... is violated*" |

---

## 9. Resumen

- Las **restricciones** son reglas que la base de datos hace cumplir sola en cada `INSERT`, `UPDATE` y `DELETE`.
- **`NOT NULL`**: el campo es obligatorio. **`DEFAULT`**: valor automático si no se indica. **`UNIQUE`**: sin duplicados. **`CHECK`**: cumple una condición.
- **`PRIMARY KEY`** = `UNIQUE` + `NOT NULL`; **una por tabla**, puede ser **compuesta**. Lo habitual es un `id` **sustituto** autoincremental (la sintaxis cambia según el motor).
- **`FOREIGN KEY`** conecta una columna de la tabla hija con la clave de la tabla padre y evita datos huérfanos.
- **`ON DELETE` / `ON UPDATE`**: `RESTRICT`/`NO ACTION` (prohíbe), `CASCADE` (propaga), `SET NULL`, `SET DEFAULT`. Por defecto, lo más seguro es **prohibir**.
- Ponles **nombre** con `CONSTRAINT` para entender los errores y poder borrarlas.
- Al añadir una restricción a una tabla con datos, **primero busca y corrige** lo que la incumple.
- **Dialectos:** cambian el autoincremento, la forma de borrar restricciones, el `CHECK` en MySQL antiguo, las claves foráneas desactivadas en SQLite y el `UNIQUE` con `NULL` en SQL Server.