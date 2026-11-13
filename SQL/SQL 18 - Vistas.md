# 18 - Vistas

> [!info] ¿Qué es?
> Una **vista** (`VIEW`) es una **consulta `SELECT` guardada con un nombre**. Después la usas como si fuera una tabla, pero **no guarda datos propios**: cada vez que la consultas, la base de datos ejecuta la consulta de nuevo y te enseña el resultado actual. Es como una "tabla virtual".

---

## 1. Antes de empezar

Imagina que todos los días escribes esta consulta larga:

```sql
SELECT c.nombre, p.id, p.fecha, SUM(l.cantidad * l.precio_unitario) AS total
FROM clientes c
JOIN pedidos p       ON p.cliente_id = c.id
JOIN lineas_pedido l ON l.pedido_id = p.id
GROUP BY c.nombre, p.id, p.fecha;
```

Es larga, fácil de escribir mal y la repites en muchos sitios. Con una vista la guardas **una vez** y después solo haces `SELECT * FROM resumen_pedidos`.

Lo que necesitas saber de antes:

- `SELECT`, `JOIN` y `GROUP BY` (ver [[SQL 02 - SELECT]], [[SQL 07 - JOIN]] y [[SQL 06 - GROUP BY y HAVING]]).
- Una **tabla base** es una tabla real, la que guarda los datos de verdad. Las vistas se construyen sobre tablas base (o sobre otras vistas).

---

## 2. Concepto fundamental

### Qué es y qué no es

| Una vista **es** | Una vista **no es** |
|---|---|
| Una consulta con nombre | Una copia de los datos |
| Una forma de **mirar** los datos | Un sitio donde se almacenan datos |
| Siempre actualizada (muestra el estado actual de las tablas base) | Una foto fija del pasado |

> [!tip] Idea clave
> La vista **no guarda nada**: solo guarda la **receta** (el `SELECT`). Los datos siguen en las tablas base. Si cambian las tablas, la vista cambia sola.

### Para qué sirven

1. **Simplificar**: esconden consultas complejas detrás de un nombre sencillo.
2. **Reutilizar**: escribes la lógica una vez y la usas donde quieras.
3. **Seguridad**: puedes dejar que un usuario vea solo **ciertas columnas o filas** (por ejemplo, los clientes sin su contraseña) sin darle acceso a la tabla entera.
4. **Compatibilidad**: si cambias la estructura de las tablas, puedes mantener una vista con el formato antiguo para que las aplicaciones no se rompan.

### Cómo funciona por dentro

Cuando haces `SELECT * FROM mi_vista WHERE ...`, la base de datos **sustituye** el nombre de la vista por su consulta guardada y ejecuta todo junto. No hay magia: es como pegar el `SELECT` original dentro de tu consulta.

---

## 3. Sintaxis / estructura

### Crear una vista

```sql
CREATE VIEW nombre_vista AS
SELECT columna1, columna2
FROM tabla
WHERE condicion;
```

### Usar una vista

Se usa **exactamente igual que una tabla**:

```sql
SELECT * FROM nombre_vista;
SELECT columna1 FROM nombre_vista WHERE columna2 > 10;
```

### Modificar una vista

```sql
CREATE OR REPLACE VIEW nombre_vista AS
SELECT ...;
```

### Borrar una vista

```sql
DROP VIEW nombre_vista;
DROP VIEW IF EXISTS nombre_vista;
```

Borrar una vista **no borra datos**: solo elimina la "receta".

### Poner nombres a las columnas

```sql
CREATE VIEW nombre_vista (col_a, col_b) AS
SELECT columna1, columna2
FROM tabla;
```

> [!warning] Diferencias entre sistemas
> - **`CREATE OR REPLACE VIEW`**: funciona en MySQL, MariaDB, PostgreSQL y Oracle. En **SQL Server** se usa `CREATE OR ALTER VIEW` (versiones recientes) o `ALTER VIEW`. **SQLite** no tiene ninguna de las dos: hay que hacer `DROP VIEW` y volver a crearla.
> - **Vistas materializadas** (ver más abajo): existen en PostgreSQL y Oracle; **MySQL, MariaDB y SQLite no las tienen**.

---

## 4. Elementos / propiedades / características

### 4.1 Vista simple vs vista compleja

| | Vista simple | Vista compleja |
|---|---|---|
| Basada en | **Una** tabla | Varias tablas (`JOIN`) |
| Usa | Solo filtros y columnas | `GROUP BY`, funciones de agregación, `DISTINCT`, subconsultas |
| ¿Se puede modificar con `INSERT`/`UPDATE`/`DELETE`? | Normalmente **sí** | Normalmente **no** |

### 4.2 Vistas actualizables

A veces puedes hacer `INSERT`, `UPDATE` o `DELETE` **a través de la vista**, y el cambio se aplica a la tabla base. Pero solo si la base de datos puede saber **sin ambigüedad** qué fila real tocar.

Una vista **normalmente NO es actualizable** si contiene:

- `GROUP BY` o `HAVING`
- Funciones de agregación (`SUM`, `COUNT`, `AVG`...)
- `DISTINCT`
- `UNION`
- Varias tablas combinadas con `JOIN` (en muchos casos)
- Columnas calculadas

Es decir: si cada fila de la vista corresponde a **una fila real** de una tabla, suele funcionar. Si las filas son resultado de cálculos o agrupaciones, no.

### 4.3 `WITH CHECK OPTION`

Evita que, al modificar a través de la vista, insertes o cambies filas que **luego la vista no podría mostrar**.

```sql
CREATE VIEW clientes_madrid AS
SELECT id, nombre, ciudad
FROM clientes
WHERE ciudad = 'Madrid'
WITH CHECK OPTION;
```

Con esto, intentar insertar un cliente de Sevilla **a través de esa vista** da error, porque no cumpliría el `WHERE` de la vista.

### 4.4 Vistas materializadas

Una **vista materializada** sí **guarda los datos** del resultado en disco, como una copia. Lee mucho más rápido, pero **los datos pueden quedar desactualizados** y hay que **refrescarlos**.

| | Vista normal | Vista materializada |
|---|---|---|
| ¿Guarda datos? | No | **Sí** |
| ¿Datos actuales? | Siempre | Solo tras refrescarla |
| Velocidad de lectura | Depende de la consulta | Muy rápida |
| Espacio en disco | Ninguno | Ocupa espacio |
| Disponible en | Todos | PostgreSQL, Oracle (y SQL Server con *indexed views*) |

En PostgreSQL:

```sql
CREATE MATERIALIZED VIEW ventas_mensuales AS
SELECT ...;

REFRESH MATERIALIZED VIEW ventas_mensuales;  -- actualizar los datos
```

### 4.5 Vistas y seguridad

Se puede dar permiso a un usuario sobre la **vista** pero no sobre la tabla:

```sql
GRANT SELECT ON clientes_publico TO usuario_web;
```

Así ve solo lo que muestra la vista, aunque la tabla original tenga datos sensibles.

---

## 5. Ejemplos prácticos

### Ejemplo básico: ocultar columnas sensibles

La tabla `usuarios` tiene contraseña, y no queremos que todo el mundo la vea:

```sql
CREATE VIEW usuarios_publicos AS
SELECT id, nombre, email
FROM usuarios;
```

Uso:

```sql
SELECT * FROM usuarios_publicos WHERE nombre LIKE 'A%';
```

### Ejemplo habitual: simplificar una consulta con `JOIN` y `GROUP BY`

```sql
CREATE VIEW resumen_pedidos AS
SELECT
    p.id AS pedido_id,
    c.nombre AS cliente,
    p.fecha,
    SUM(l.cantidad * l.precio_unitario) AS total
FROM pedidos p
JOIN clientes c       ON c.id = p.cliente_id
JOIN lineas_pedido l  ON l.pedido_id = p.id
GROUP BY p.id, c.nombre, p.fecha;
```

Ahora cualquiera puede consultar los totales sin escribir los `JOIN`:

```sql
SELECT * FROM resumen_pedidos WHERE total > 100 ORDER BY fecha DESC;
```

### Ejemplo completo: vista actualizable con `WITH CHECK OPTION`

```sql
CREATE VIEW clientes_madrid AS
SELECT id, nombre, email, ciudad
FROM clientes
WHERE ciudad = 'Madrid'
WITH CHECK OPTION;

-- Funciona: cumple ciudad = 'Madrid'
INSERT INTO clientes_madrid (id, nombre, email, ciudad)
VALUES (60, 'Marta', 'marta@mail.com', 'Madrid');

-- Da error: no cumple el WHERE de la vista
INSERT INTO clientes_madrid (id, nombre, email, ciudad)
VALUES (61, 'Pablo', 'pablo@mail.com', 'Sevilla');
```

---

## 6. Buenas prácticas

- **Dale nombres claros** que digan qué muestra: `resumen_pedidos`, `clientes_activos`. Mucha gente usa un prefijo como `v_` o `vw_` para distinguirlas de las tablas.
- **Evita `SELECT *` dentro de la vista**: si luego añades columnas a la tabla base, la vista puede romperse o comportarse raro. Escribe las columnas.
- **Úsalas para repetir lógica**, no por usarlas: si la consulta solo se usa una vez, no hace falta una vista.
- **No apiles vistas sobre vistas sobre vistas**: cada capa añade complejidad y puede volver lentas las consultas.
- **Úsalas para seguridad**: dar acceso a la vista en vez de a la tabla es una forma sencilla de limitar qué ve cada usuario.
- **No esperes que una vista sea más rápida** que su consulta: una vista normal no acelera nada, solo la reutiliza. Para velocidad real, mira los índices (ver [[SQL 14 - Índices]]) o las vistas materializadas.
- **Usa `WITH CHECK OPTION`** en vistas que filtran con `WHERE` y que vayas a usar para insertar o modificar.
- **Documenta** qué hace cada vista (un comentario en tu nota o en el propio SQL), porque con el tiempo es fácil olvidar su propósito.
- **Cuidado con cambiar las tablas base**: si borras o renombras una columna que usa una vista, la vista dejará de funcionar.

---

## 7. Diferencias importantes

### Vista vs tabla

| | Tabla | Vista |
|---|---|---|
| ¿Guarda datos? | **Sí** | No (solo la consulta) |
| Se actualiza | Cuando tú insertas/modificas | Sola, al cambiar las tablas base |
| Espacio en disco | Ocupa | Casi nada |
| `INSERT`/`UPDATE`/`DELETE` | Siempre | Solo si es actualizable |

### Vista normal vs vista materializada

Normal: **siempre al día**, pero recalcula en cada consulta. Materializada: **muy rápida**, pero hay que refrescarla y puede estar desfasada.

### Vista vs subconsulta

Una subconsulta vive **dentro de una consulta** y desaparece al terminar. Una vista **se guarda con nombre** y se puede reutilizar en muchas consultas.

### Vista vs tabla temporal

| | Vista | Tabla temporal |
|---|---|---|
| Guarda datos | No | **Sí** (hasta que termina la sesión) |
| Duración | Permanente | Temporal |
| Se actualiza sola | Sí | No |

---

## 8. Casos especiales

- **Vistas que dependen de otras vistas**: funcionan, pero si cambias la de abajo, las de arriba pueden romperse. Mantén pocas capas.
- **`ORDER BY` dentro de una vista**: en muchos sistemas es ignorado o ni siquiera se permite (SQL Server lo prohíbe sin `TOP`). Lo normal es **ordenar al consultar la vista**, no dentro de ella.
- **Rendimiento**: como la vista ejecuta su consulta cada vez, una vista con muchos `JOIN` sobre tablas enormes puede ser lenta aunque tú solo escribas un `SELECT` corto.
- **Cambios en las tablas base**: si añades una columna a la tabla, la vista **no la incluye sola** (aunque uses `SELECT *`, en la mayoría de sistemas la lista de columnas se "congela" al crearla). Hay que recrearla.
- **Vistas y permisos**: en algunos sistemas la vista se ejecuta con los permisos de **quien la creó** (útil para dar acceso limitado) y en otros con los de **quien la consulta**. Revisa la documentación de tu sistema.
- **Información de las vistas**: el estándar ofrece `INFORMATION_SCHEMA.VIEWS` para ver las vistas que existen y su definición (MySQL, PostgreSQL, SQL Server).
- **Vistas indexadas** (SQL Server): permiten crear un índice sobre una vista para que sea más rápida, parecido a una vista materializada.
- **`DROP` de una tabla usada por una vista**: algunos sistemas lo impiden o dejan la vista inservible. En PostgreSQL, `DROP ... CASCADE` borra también las vistas que dependen de ella.

---

## 9. Resumen

| Concepto | Idea clave |
|---|---|
| Vista | `SELECT` guardado con nombre; una "tabla virtual" |
| ¿Guarda datos? | **No**, solo la consulta (la materializada sí) |
| Crear | `CREATE VIEW nombre AS SELECT ...` |
| Modificar | `CREATE OR REPLACE VIEW` (SQL Server: `CREATE OR ALTER`) |
| Borrar | `DROP VIEW [IF EXISTS] nombre` (no borra datos) |
| Usarla | `SELECT * FROM nombre_vista` como si fuera una tabla |
| Para qué | Simplificar, reutilizar, dar seguridad, mantener compatibilidad |
| Actualizable | Solo si cada fila de la vista = una fila real de una tabla |
| `WITH CHECK OPTION` | Impide cambios que la vista luego no podría mostrar |
| Materializada | Guarda el resultado; rápida pero hay que refrescarla |
| Evitar | `SELECT *` dentro de la vista y cadenas largas de vistas |

> [!tip] Para recordar en una frase
> **Una vista es una consulta con nombre: no guarda datos, solo la forma de verlos.**

Siguiente nota: [[SQL 19 - Procedimientos y triggers]]