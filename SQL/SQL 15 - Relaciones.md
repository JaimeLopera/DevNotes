# 15 - Relaciones

> [!info] ¿Qué es?
> Una **relación** es el vínculo que une dos tablas. Se crea con una **clave foránea** (`FOREIGN KEY`): una columna de una tabla que guarda el valor de la clave primaria de otra. Gracias a ella, la base de datos sabe qué fila de una tabla pertenece a qué fila de otra, y **no te deja romper ese vínculo**.

---

## 1. Antes de empezar

Una base de datos bien hecha **no mete todo en una sola tabla**. Cada cosa va en su tabla (clientes, pedidos, productos...) y después se conectan entre sí.

¿Por qué? Imagina guardar el nombre y el email del cliente en cada pedido: si el cliente tiene 50 pedidos, sus datos se repiten 50 veces. Si cambia de email, tendrías que cambiarlo 50 veces y es fácil que alguno se quede mal. Separando las tablas, el cliente se guarda **una sola vez** y los pedidos solo apuntan a él.

Lo único que necesitas saber de antes (se explica a fondo en [[SQL 13 - Claves y restricciones]]):

- **Clave primaria (`PRIMARY KEY`)**: identifica de forma única cada fila de una tabla (normalmente `id`).
- **Clave foránea (`FOREIGN KEY`)**: columna que guarda el `id` de otra tabla para apuntar a una de sus filas.

---

## 2. Concepto fundamental

### Tabla padre y tabla hija

En toda relación hay dos papeles:

| Papel | Qué es | Ejemplo |
|---|---|---|
| **Tabla padre** (referenciada) | La tabla a la que se apunta. Tiene la clave primaria. | `clientes` |
| **Tabla hija** (referenciadora) | La tabla que apunta. Tiene la clave foránea. | `pedidos` |

> [!tip] Regla para no liarte
> **La clave foránea siempre va en la tabla hija**, es decir, en la que "pertenece" a la otra. Un pedido *pertenece a* un cliente, así que `pedidos` lleva la columna `cliente_id`.

### Los tres tipos de relación

La **cardinalidad** dice cuántas filas de una tabla se pueden relacionar con cuántas de la otra:

| Tipo | Se lee | Ejemplo | Cómo se implementa |
|---|---|---|---|
| **1:N** (uno a muchos) | Un cliente tiene muchos pedidos | `clientes` → `pedidos` | FK en la tabla del lado "muchos" |
| **N:M** (muchos a muchos) | Un estudiante tiene muchos cursos y un curso muchos estudiantes | `estudiantes` ↔ `cursos` | Una **tabla intermedia** con dos FK |
| **1:1** (uno a uno) | Un usuario tiene un perfil | `usuarios` → `perfiles` | FK + `UNIQUE` |

La relación **1:N es, con diferencia, la más común**. Las otras dos son variaciones de la misma idea.

---

## 3. Sintaxis / estructura

### Opción A: dentro de `CREATE TABLE`

```sql
CREATE TABLE pedidos (
    id         INT PRIMARY KEY,
    fecha      DATE NOT NULL,
    cliente_id INT NOT NULL,
    CONSTRAINT fk_pedidos_clientes
        FOREIGN KEY (cliente_id)
        REFERENCES clientes (id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

### Opción B: añadirla después con `ALTER TABLE`

```sql
ALTER TABLE pedidos
ADD CONSTRAINT fk_pedidos_clientes
    FOREIGN KEY (cliente_id)
    REFERENCES clientes (id);
```

### Cómo leerlo

| Parte | Significado |
|---|---|
| `CONSTRAINT fk_pedidos_clientes` | Nombre que le das a la regla (te ayuda a borrarla o a entender los errores). |
| `FOREIGN KEY (cliente_id)` | La columna de **esta** tabla que apunta a otra. |
| `REFERENCES clientes (id)` | La tabla y columna **a la que** apunta. |
| `ON DELETE ...` | Qué pasa con los hijos si borras el padre. |
| `ON UPDATE ...` | Qué pasa con los hijos si cambias la clave del padre. |

> [!warning] Importante
> La tabla padre (`clientes`) **tiene que existir antes** que la hija (`pedidos`), y la columna a la que apuntas debe ser clave primaria o, como mínimo, `UNIQUE`. Además, las dos columnas deben tener **el mismo tipo de dato** (`INT` con `INT`, no `INT` con `VARCHAR`).

---

## 4. Elementos / propiedades / características

### 4.1 Lo que garantiza una clave foránea

Es la **integridad referencial**: la base de datos vigila que el vínculo siempre sea válido.

- No puedes insertar un pedido con un `cliente_id` que no existe.
- No puedes borrar un cliente si tiene pedidos (salvo que lo hayas configurado de otra forma).
- No puedes cambiar el `id` de un cliente dejando pedidos huérfanos.

Una fila hija sin padre se llama **huérfana**. La clave foránea existe precisamente para que no haya huérfanas.

### 4.2 Acciones referenciales (`ON DELETE` / `ON UPDATE`)

Definen qué hace la base de datos cuando el padre se borra o cambia:

| Acción | Qué hace |
|---|---|
| `RESTRICT` | **Prohíbe** borrar/cambiar el padre mientras tenga hijos. |
| `NO ACTION` | Parecido a `RESTRICT`: da error. Es el comportamiento por defecto en el estándar. |
| `CASCADE` | **Propaga** el cambio: si borras el padre, se borran también sus hijos. |
| `SET NULL` | Pone la clave foránea de los hijos a `NULL` (la columna debe admitir `NULL`). |
| `SET DEFAULT` | Pone la clave foránea de los hijos a su valor por defecto. |

Cuál elegir:

- **`RESTRICT`**: lo más seguro. Úsalo cuando los hijos son datos importantes (pedidos, facturas).
- **`CASCADE`**: cuando los hijos no tienen sentido sin el padre (las líneas de un pedido sin el pedido).
- **`SET NULL`**: cuando el hijo puede existir sin padre (un empleado cuyo departamento desaparece).

### 4.3 Relación obligatoria u opcional

Se controla con `NOT NULL` en la clave foránea:

| Columna | Significado |
|---|---|
| `cliente_id INT NOT NULL` | **Obligatoria**: todo pedido debe tener cliente. |
| `departamento_id INT` (admite `NULL`) | **Opcional**: un empleado puede no tener departamento. |

### 4.4 Relación N:M y tabla intermedia

Una clave foránea sola no puede representar "muchos a muchos". La solución es una tercera tabla, llamada **intermedia** (o de unión, o puente), que tiene **dos claves foráneas**, una hacia cada tabla. Cada fila de la intermedia es una "pareja" entre las dos.

Su clave primaria suele ser **compuesta**: la combinación de las dos FK. Así no se puede repetir la misma pareja.

### 4.5 Relación 1:1

Es una 1:N con una restricción extra: la clave foránea es `UNIQUE`, así que cada padre solo puede tener **un** hijo. Se usa para separar datos opcionales o poco usados, o datos sensibles.

### 4.6 Relación recursiva (autorreferencia)

Una tabla puede relacionarse **consigo misma**. Ejemplo típico: un empleado tiene un jefe, que es otro empleado. La clave foránea apunta a la propia tabla.

---

## 5. Ejemplos prácticos

### Ejemplo básico: 1:N (clientes y pedidos)

```sql
CREATE TABLE clientes (
    id     INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email  VARCHAR(150) NOT NULL UNIQUE
);

CREATE TABLE pedidos (
    id         INT PRIMARY KEY,
    fecha      DATE NOT NULL,
    cliente_id INT NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes (id)
);
```

Para ver los datos relacionados se usa un `JOIN` (ver [[SQL 07 - JOIN]]):

```sql
SELECT c.nombre, p.id AS pedido, p.fecha
FROM clientes c
JOIN pedidos p ON p.cliente_id = c.id;
```

### Ejemplo habitual: N:M (estudiantes y cursos)

```sql
CREATE TABLE estudiantes (
    id     INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE cursos (
    id     INT PRIMARY KEY,
    titulo VARCHAR(100) NOT NULL
);

-- Tabla intermedia
CREATE TABLE matriculas (
    estudiante_id INT NOT NULL,
    curso_id      INT NOT NULL,
    fecha         DATE NOT NULL,
    PRIMARY KEY (estudiante_id, curso_id),
    FOREIGN KEY (estudiante_id) REFERENCES estudiantes (id) ON DELETE CASCADE,
    FOREIGN KEY (curso_id)      REFERENCES cursos (id)      ON DELETE CASCADE
);
```

Fíjate en dos cosas: la clave primaria es la **pareja** de columnas (no se puede matricular dos veces al mismo estudiante en el mismo curso), y la tabla intermedia puede tener **datos propios** (`fecha`).

Para consultarlo hay que pasar por la intermedia:

```sql
SELECT e.nombre, c.titulo
FROM estudiantes e
JOIN matriculas m ON m.estudiante_id = e.id
JOIN cursos c     ON c.id = m.curso_id;
```

### Ejemplo completo: 1:1 y autorreferencia

**1:1** (un usuario, un perfil):

```sql
CREATE TABLE perfiles (
    usuario_id INT PRIMARY KEY,
    bio        TEXT,
    FOREIGN KEY (usuario_id) REFERENCES usuarios (id) ON DELETE CASCADE
);
```

Aquí la propia clave primaria es la clave foránea: eso ya garantiza que sea única (1:1) sin escribir `UNIQUE`.

**Autorreferencia** (empleados y jefes):

```sql
CREATE TABLE empleados (
    id      INT PRIMARY KEY,
    nombre  VARCHAR(100) NOT NULL,
    jefe_id INT,  -- admite NULL: el director no tiene jefe
    FOREIGN KEY (jefe_id) REFERENCES empleados (id) ON DELETE SET NULL
);
```

Para ver cada empleado con su jefe se hace un *self join*:

```sql
SELECT e.nombre AS empleado, j.nombre AS jefe
FROM empleados e
LEFT JOIN empleados j ON e.jefe_id = j.id;
```

---

## 6. Buenas prácticas

- **Declara siempre las claves foráneas.** Aunque "funcione" sin ellas, sin `FOREIGN KEY` la base de datos no protege tus datos y acabarás con huérfanas.
- **Ponles nombre** con una convención clara, por ejemplo `fk_hija_padre` (`fk_pedidos_clientes`).
- **Mismo tipo de dato** en la FK y en la columna a la que apunta.
- **Nombra la columna con claridad**: `cliente_id` (tabla en singular + `_id`).
- **Crea un índice en la clave foránea** si tu sistema no lo hace solo. Acelera los `JOIN` y los borrados del padre (ver [[SQL 14 - Índices]]).
- **Elige a propósito el `ON DELETE`.** Ante la duda, `RESTRICT`: es mejor un error que perder datos sin querer.
- **Usa `CASCADE` con cuidado**: un solo `DELETE` puede borrar muchísimas filas en cadena.
- **Pon `NOT NULL`** en la FK siempre que la relación sea obligatoria.
- **Crea primero las tablas padre** y después las hijas (y bórralas en el orden contrario).

---

## 7. Diferencias importantes

### Relación vs `JOIN`

| | Relación (`FOREIGN KEY`) | `JOIN` |
|---|---|---|
| Dónde vive | En la **estructura** de la base de datos | En una **consulta** |
| Para qué sirve | **Proteger** la coherencia de los datos | **Unir** datos al consultar |
| ¿Es obligatoria para hacer JOIN? | No | — |

Puedes hacer un `JOIN` entre dos tablas sin que exista ninguna `FOREIGN KEY`. La clave foránea no sirve para consultar, sino para **impedir datos incoherentes**.

### `RESTRICT` vs `NO ACTION`

Casi iguales en la práctica: los dos impiden el borrado. La diferencia fina es *cuándo* comprueban: `RESTRICT` lo hace al instante y `NO ACTION` al final de la sentencia (y, donde se permite, de la transacción). En MySQL/InnoDB se comportan igual.

### `CASCADE` vs `SET NULL`

- `CASCADE`: los hijos **desaparecen** con el padre.
- `SET NULL`: los hijos **se quedan**, pero pierden el vínculo.

### Clave primaria vs clave foránea

| | Primaria | Foránea |
|---|---|---|
| Por tabla | Solo una | Puede haber varias |
| `NULL` | Nunca | Puede, si la relación es opcional |
| Valores repetidos | No | Sí (muchos pedidos del mismo cliente) |

---

## 8. Casos especiales

- **Claves foráneas compuestas**: si la clave primaria del padre son dos columnas, la FK también lleva dos: `FOREIGN KEY (a, b) REFERENCES padre (a, b)`.
- **FK hacia una columna `UNIQUE`**: no tiene que ser siempre la clave primaria; vale cualquier columna única (por ejemplo `email`), aunque lo normal es apuntar a la primaria.
- **Varias FK hacia la misma tabla**: una tabla puede apuntar dos veces a otra (un `mensaje` con `emisor_id` y `receptor_id`, ambos hacia `usuarios`). Cada una lleva su propio nombre.
- **Dependencias circulares**: si A apunta a B y B apunta a A, no puedes crear ninguna primero. Se resuelve creando las tablas sin la FK y añadiéndola después con `ALTER TABLE`, o dejando una de las FK como opcional (`NULL`).
- **Insertar en el orden correcto**: primero el padre, luego el hijo. Al borrar, al revés (o usar `CASCADE`).
- **Tabla intermedia con datos propios**: como `matriculas.fecha`. Si la intermedia crece mucho en importancia, a veces conviene darle su propio `id` en vez de clave compuesta.
- **Relaciones "polimórficas"** (una columna que apunta a tablas distintas según un tipo): el SQL estándar **no** permite una FK así. Suele considerarse mal diseño; es mejor una tabla intermedia por cada caso.

> [!warning] Diferencias entre sistemas
> - **MySQL**: solo respeta las claves foráneas en el motor **InnoDB** (el de por defecto hoy). El motor antiguo MyISAM las acepta pero las **ignora**.
> - **SQLite**: las claves foráneas vienen **desactivadas** por defecto; hay que activarlas en cada conexión con `PRAGMA foreign_keys = ON;`.
> - **SQL Server**: da error si creas varias rutas de `CASCADE` hacia la misma tabla ("multiple cascade paths"). Tampoco tiene `ON UPDATE`/`ON DELETE` con `RESTRICT` (usa `NO ACTION`).
> - **Oracle**: no tiene `ON UPDATE`; solo `ON DELETE` (`CASCADE` o `SET NULL`).
> - **PostgreSQL**: permite `DEFERRABLE`, para comprobar la FK al final de la transacción y no sentencia a sentencia (útil con dependencias circulares).
> - **Ids automáticos**: la sintaxis cambia según el sistema (`AUTO_INCREMENT` en MySQL, `IDENTITY` en SQL Server, `GENERATED ... AS IDENTITY` en PostgreSQL/Oracle). Por eso en los ejemplos se ha usado `INT PRIMARY KEY` a secas.

---

## 9. Resumen

| Concepto | Idea clave |
|---|---|
| Relación | Vínculo entre dos tablas mediante una clave foránea |
| Clave foránea | Columna de la tabla **hija** que guarda la clave primaria del **padre** |
| Integridad referencial | La BD impide filas huérfanas y vínculos rotos |
| 1:N | FK en la tabla del lado "muchos" (el caso más común) |
| N:M | Tabla intermedia con dos FK (y clave primaria compuesta) |
| 1:1 | FK + `UNIQUE` (o la FK como clave primaria) |
| `ON DELETE` | `RESTRICT` (prohíbe), `CASCADE` (borra hijos), `SET NULL` (suelta hijos) |
| Obligatoria / opcional | `NOT NULL` o admite `NULL` en la FK |
| Autorreferencia | Una tabla con una FK hacia sí misma (empleado → jefe) |
| Relación vs JOIN | La relación protege los datos; el JOIN los une al consultar |

> [!tip] Para recordar en una frase
> **La clave foránea va en el "muchos" y apunta al "uno"; y si es muchos a muchos, necesitas una tabla en medio.**

Siguiente nota: [[SQL 16 - Normalización]]