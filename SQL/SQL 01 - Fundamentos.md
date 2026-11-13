# 01 - Fundamentos de SQL

> [!info] ¿Qué es?
> **SQL** (*Structured Query Language*) es el lenguaje que se usa para hablar con bases de datos relacionales: pedir datos, guardarlos, cambiarlos, borrarlos y organizarlos. Es un lenguaje **estándar**, y lo usan MySQL, PostgreSQL, SQL Server, SQLite, Oracle y muchos más, con pequeñas diferencias entre ellos.

## 1. Antes de empezar

- No hace falta saber programar, pero ayuda entender lo que es una "tabla" (como una hoja de Excel).
- Las bases de datos se usan desde PHP, Node.js, Python... Mira las carpetas `PHP/` (nota de PDO) y `Node.js y npm/`.
- Esta carpeta está escrita para valer con **cualquier sistema SQL**. Los ejemplos usan SQL estándar siempre que es posible, y cuando algo cambia entre sistemas se avisa con un aviso como este:

> [!warning] Variantes por sistema
> Así se marcan las diferencias entre MySQL, PostgreSQL, SQL Server, SQLite y Oracle.

## 2. Concepto fundamental

Una **base de datos relacional** guarda la información en **tablas**. Las tablas pueden relacionarse entre sí.

| Concepto | Qué es | Equivalente en Excel |
| --- | --- | --- |
| **Base de datos** | Conjunto de tablas | El archivo |
| **Tabla** | Un tipo de dato (clientes, pedidos...) | Una hoja |
| **Columna** (campo) | Una característica (nombre, edad...) | Una columna |
| **Fila** (registro) | Un elemento concreto | Una fila |
| **Clave primaria** | Identificador único de cada fila | Un número de fila que nunca se repite |

Al programa que gestiona la base de datos se le llama **SGBD** (sistema gestor de bases de datos). **SQL es el idioma; el SGBD es el programa que lo entiende.**

## 3. SQL estándar y dialectos

SQL está definido por un estándar oficial (**ANSI/ISO**), que se ha ido ampliando con los años:

| Versión | Qué añadió (resumen) |
| --- | --- |
| SQL-92 | La base: `SELECT`, `JOIN`, `GROUP BY`, subconsultas |
| SQL:1999 | Consultas recursivas, triggers, `CASE` |
| SQL:2003 | Funciones de ventana, columnas autonuméricas (`IDENTITY`) |
| SQL:2016 | Soporte de JSON |

Cada SGBD cumple una parte del estándar y además añade **extensiones propias**. A cada versión de SQL de un sistema se le llama **dialecto**.

| | Qué es | Ejemplo |
| --- | --- | --- |
| **SQL estándar** | Funciona igual en (casi) todos | `SELECT`, `WHERE`, `JOIN`, `COALESCE`, `CASE` |
| **Extensión propia** | Solo funciona en un sistema | `LIMIT` (MySQL, PostgreSQL, SQLite), `TOP` (SQL Server) |

> [!tip] Regla práctica
> **Aprende primero lo estándar** y deja lo propio de cada sistema para cuando lo necesites. Así lo que sabes te sirve en cualquier sitio. Cuando una función suena rara o es de un solo sistema, sospecha de que es una extensión.

Dicho en números: con lo estándar cubres la gran mayoría de lo que escribes cada día. Las diferencias aparecen sobre todo en **fechas, texto, autonuméricos, limitar filas, y procedimientos y triggers**.

## 4. Principales sistemas gestores

| SGBD | Tipo y licencia | Dónde se usa | Detalles |
| --- | --- | --- | --- |
| **MySQL** | Servidor. Código abierto (de Oracle) | Webs y aplicaciones (WordPress, Laravel, PHP) | Muy extendido; algo permisivo con los errores |
| **MariaDB** | Servidor. Código abierto | Igual que MySQL (es un *fork*) | Casi idéntico a MySQL. **XAMPP trae MariaDB** |
| **PostgreSQL** | Servidor. Código abierto | Aplicaciones serias, datos complejos | Muy estricto y fiel al estándar, con muchas funciones avanzadas |
| **SQL Server** | Servidor. Comercial (versión Express gratuita) | Empresas, entorno Microsoft y .NET | Usa el dialecto **T-SQL** |
| **SQLite** | **Sin servidor**: toda la base está en un archivo | Móviles, apps de escritorio, pruebas, proyectos pequeños | Ligero, pero con menos funciones |
| **Oracle Database** | Servidor. Comercial | Grandes empresas, banca | Usa el dialecto **PL/SQL** |

> [!tip] Cuál usar al estudiar
> En el ciclo es habitual **MySQL o MariaDB** (por XAMPP y PHP). **PostgreSQL** es una gran segunda opción por ser más estricto: te obliga a escribir mejor SQL. **SQLite** es ideal para practicar sin instalar un servidor.

## 5. Tipos de comandos

| Grupo | Nombre | Para qué | Comandos |
| --- | --- | --- | --- |
| **DQL** | Consulta | Leer datos | `SELECT` |
| **DML** | Manipulación | Cambiar datos | `INSERT`, `UPDATE`, `DELETE` |
| **DDL** | Definición | Crear o cambiar la estructura | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DCL** | Control | Permisos | `GRANT`, `REVOKE` |
| **TCL** | Transacciones | Controlar grupos de operaciones | `COMMIT`, `ROLLBACK` |

(Algunos autores incluyen `SELECT` dentro de DML.)

## 6. Reglas de escritura

- Cada instrucción termina con **`;`**. En SQL Server es opcional, pero conviene ponerlo siempre.
- Las **palabras clave no distinguen mayúsculas de minúsculas** (`select` = `SELECT`), pero se escriben en **MAYÚSCULAS** por costumbre.
- Una instrucción puede ocupar varias líneas.
- Los **textos** y las **fechas** van entre **comillas simples**: `'Madrid'`.
- Para escribir un apóstrofe dentro de un texto, se **duplica**: `'l''iPad'`.

### Comentarios

```sql
-- Comentario de una línea
/* Comentario
   de varias líneas */
```

Funcionan en todos los sistemas. (En MySQL hace falta un espacio después de `--`, y también admite `#`, que es solo de MySQL.)

### Nombres de tablas y columnas (identificadores)

Recomendación válida en todos los sistemas: **minúsculas, con `_`, sin espacios ni acentos** (`fecha_pedido`). Así nunca hace falta usar comillas.

Si un nombre es especial (lleva espacios o es una palabra reservada, como `order` o `user`), hay que "entrecomillarlo", y **cada sistema lo hace de forma distinta**:

> [!warning] Variantes por sistema
> | Sistema | Cómo se entrecomilla un nombre |
> | --- | --- |
> | SQL estándar | `"nombre"` (comillas dobles) |
> | PostgreSQL, Oracle, SQLite | `"nombre"` |
> | MySQL / MariaDB | `` `nombre` `` (comillas invertidas) |
> | SQL Server | `[nombre]` (corchetes) o `"nombre"` |
>
> **Ojo:** las comillas dobles son para **nombres**, no para textos. MySQL deja usarlas para textos, pero es mala costumbre y no funciona en el resto.

### Mayúsculas y minúsculas en los datos

> [!warning] Variantes por sistema
> Comparar `'Ana'` con `'ana'` **no da lo mismo en todos los sistemas**:
>
> | Sistema | `WHERE nombre = 'ana'` encuentra a `'Ana'` |
> | --- | --- |
> | MySQL / MariaDB | ✅ Sí (por defecto no distingue) |
> | SQL Server | ✅ Sí (por defecto no distingue) |
> | PostgreSQL | ❌ No (distingue) |
> | Oracle | ❌ No (distingue) |
> | SQLite | ❌ No con `=`, pero `LIKE` sí lo ignora (solo con letras ASCII) |
>
> Para que funcione igual en todos: `WHERE LOWER(nombre) = 'ana'` (se explica en [[SQL 05 - Funciones]]).
>
> Con los **nombres** de tablas y columnas también hay diferencias: PostgreSQL convierte lo que no lleva comillas a minúsculas; Oracle, a mayúsculas; y en MySQL los nombres de tabla distinguen mayúsculas en Linux pero no en Windows. **Escribir siempre en minúsculas evita problemas.**

## 7. Tipos de datos

Cada columna tiene un **tipo** que define qué datos admite. Los nombres de los tipos básicos son casi iguales en todos los sistemas, pero hay diferencias:

| Para qué | Estándar | MySQL / MariaDB | PostgreSQL | SQL Server | SQLite | Oracle |
| --- | --- | --- | --- | --- | --- | --- |
| Entero | `INT` | `INT` | `INT` | `INT` | `INTEGER` | `NUMBER(10)` (`INT` también vale) |
| Entero grande | `BIGINT` | `BIGINT` | `BIGINT` | `BIGINT` | `INTEGER` | `NUMBER(19)` |
| Decimal exacto (dinero) | `DECIMAL(p,s)` | `DECIMAL(p,s)` | `NUMERIC(p,s)` | `DECIMAL(p,s)` | `NUMERIC` | `NUMBER(p,s)` |
| Decimal aproximado | `DOUBLE PRECISION` | `DOUBLE` | `DOUBLE PRECISION` | `FLOAT` | `REAL` | `BINARY_DOUBLE` |
| Texto con máximo | `VARCHAR(n)` | `VARCHAR(n)` | `VARCHAR(n)` | `VARCHAR(n)` / `NVARCHAR(n)` | `TEXT` | `VARCHAR2(n)` |
| Texto largo | — | `TEXT` | `TEXT` | `VARCHAR(MAX)` | `TEXT` | `CLOB` |
| Verdadero / falso | `BOOLEAN` | `BOOLEAN` (es un `TINYINT(1)`) | `BOOLEAN` | `BIT` | `0` y `1` | `NUMBER(1)` (`BOOLEAN` en las versiones más nuevas) |
| Fecha | `DATE` | `DATE` | `DATE` | `DATE` | texto `'AAAA-MM-DD'` | `DATE` (¡incluye la hora!) |
| Fecha y hora | `TIMESTAMP` | `DATETIME` / `TIMESTAMP` | `TIMESTAMP` | `DATETIME2` | texto | `TIMESTAMP` |
| JSON | `JSON` | `JSON` | `JSON` / `JSONB` | `NVARCHAR` + funciones JSON | texto + funciones JSON | `JSON` (versiones recientes) |

Notas sobre la tabla:

- Los tipos **más seguros para escribir código portable**: `INT`, `DECIMAL(p,s)`, `VARCHAR(n)`, `DATE`, `TIMESTAMP`.
- **SQLite** es especial: no obliga a que el dato coincida con el tipo (se llama *tipado dinámico*). Puedes guardar texto en una columna de enteros y no se queja. Los tipos son más bien una "sugerencia".
- **Dinero:** usa siempre `DECIMAL` (o `NUMERIC`), nunca los tipos aproximados, porque dan errores de redondeo.
- **Autonumérico** (que el `id` se genere solo): es de lo que más cambia, y se explica en el apartado siguiente.

### Columna autonumérica (`id` que se genera solo)

> [!warning] Variantes por sistema
> | Sistema | Cómo se escribe |
> | --- | --- |
> | SQL estándar (y PostgreSQL 10+, Oracle 12c+) | `id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
> | MySQL / MariaDB | `id INT AUTO_INCREMENT PRIMARY KEY` |
> | PostgreSQL (forma clásica) | `id SERIAL PRIMARY KEY` |
> | SQL Server | `id INT IDENTITY(1,1) PRIMARY KEY` |
> | SQLite | `id INTEGER PRIMARY KEY AUTOINCREMENT` |
> | Oracle (12c o superior) | `id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |

## 8. NULL

`NULL` significa **"no hay valor"** o "dato desconocido". No es lo mismo que `0` ni que un texto vacío `''`.

- Se comprueba con `IS NULL` o `IS NOT NULL`, **nunca con `= NULL`**.
- Cualquier operación con `NULL` da `NULL`: `5 + NULL` es `NULL`.
- Esto funciona igual en todos los sistemas. Se explica en detalle en [[SQL 03 - WHERE]].

> [!warning] Variantes por sistema
> **Oracle** trata el texto vacío `''` como `NULL`. En el resto son cosas distintas.

## 9. Base de datos de ejemplo

Todas las notas de esta carpeta usan esta tienda:

**`clientes`**

| id | nombre | ciudad | edad | email |
| --- | --- | --- | --- | --- |
| 1 | Ana | Madrid | 28 | ana@mail.com |
| 2 | Luis | Sevilla | 35 | luis@mail.com |
| 3 | Marta | Madrid | 42 | NULL |
| 4 | Pablo | Valencia | 19 | pablo@mail.com |

**`productos`**

| id | nombre | precio | stock |
| --- | --- | --- | --- |
| 1 | Teclado | 25.50 | 40 |
| 2 | Ratón | 12.00 | 100 |
| 3 | Monitor | 180.00 | 15 |

**`pedidos`**

| id | cliente_id | fecha | total |
| --- | --- | --- | --- |
| 1 | 1 | 2026-09-10 | 37.50 |
| 2 | 1 | 2026-10-01 | 180.00 |
| 3 | 2 | 2026-10-05 | 12.00 |

### Script para crearla

Está escrito en SQL estándar, con **ids puestos a mano** (así no depende de ningún autonumérico y funciona igual en todos los sistemas).

```sql
CREATE TABLE clientes (
  id INT PRIMARY KEY,
  nombre VARCHAR(50) NOT NULL,
  ciudad VARCHAR(50),
  edad INT,
  email VARCHAR(100)
);

CREATE TABLE productos (
  id INT PRIMARY KEY,
  nombre VARCHAR(50) NOT NULL,
  precio DECIMAL(10,2) NOT NULL,
  stock INT DEFAULT 0
);

CREATE TABLE pedidos (
  id INT PRIMARY KEY,
  cliente_id INT NOT NULL,
  fecha DATE NOT NULL,
  total DECIMAL(10,2) NOT NULL,
  FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

INSERT INTO clientes (id, nombre, ciudad, edad, email) VALUES (1, 'Ana', 'Madrid', 28, 'ana@mail.com');
INSERT INTO clientes (id, nombre, ciudad, edad, email) VALUES (2, 'Luis', 'Sevilla', 35, 'luis@mail.com');
INSERT INTO clientes (id, nombre, ciudad, edad, email) VALUES (3, 'Marta', 'Madrid', 42, NULL);
INSERT INTO clientes (id, nombre, ciudad, edad, email) VALUES (4, 'Pablo', 'Valencia', 19, 'pablo@mail.com');

INSERT INTO productos (id, nombre, precio, stock) VALUES (1, 'Teclado', 25.50, 40);
INSERT INTO productos (id, nombre, precio, stock) VALUES (2, 'Ratón', 12.00, 100);
INSERT INTO productos (id, nombre, precio, stock) VALUES (3, 'Monitor', 180.00, 15);

INSERT INTO pedidos (id, cliente_id, fecha, total) VALUES (1, 1, '2026-09-10', 37.50);
INSERT INTO pedidos (id, cliente_id, fecha, total) VALUES (2, 1, '2026-10-01', 180.00);
INSERT INTO pedidos (id, cliente_id, fecha, total) VALUES (3, 2, '2026-10-05', 12.00);
```

(Cada instrucción se explica en [[SQL 09 - INSERT]], [[SQL 12 - CREATE y ALTER]] y [[SQL 13 - Claves y restricciones]].)

> [!warning] Variantes por sistema
> Antes del script, o con pequeños ajustes:
>
> | Sistema | Qué hay que hacer |
> | --- | --- |
> | **MySQL / MariaDB / SQL Server** | `CREATE DATABASE tienda;` y luego `USE tienda;` |
> | **PostgreSQL** | `CREATE DATABASE tienda;` y conectarte a ella (`\c tienda` en `psql`) |
> | **SQLite** | No hay `CREATE DATABASE`: la base de datos es el **archivo** que abres (`sqlite3 tienda.db`) |
> | **Oracle** | No se crea la base de datos así: trabajas dentro de tu usuario (esquema) |
> | **Oracle** (fechas) | Escribe las fechas como `DATE '2026-09-10'`; con `'2026-09-10'` a secas puede fallar |
> | **Oracle** (versiones anteriores a 23ai) | Para insertar varias filas de una vez hay que usar otra sintaxis; por eso el script usa un `INSERT` por fila |
> | **SQLite** | Las claves foráneas vienen **desactivadas**: ejecuta antes `PRAGMA foreign_keys = ON;` |

## 10. Dónde practicar

| Sistema | Herramientas | Cómo probarlo sin instalar |
| --- | --- | --- |
| MySQL / MariaDB | MySQL Workbench, phpMyAdmin (con XAMPP), terminal `mysql` | Contenedor Docker `mysql` o `mariadb` |
| PostgreSQL | pgAdmin, terminal `psql` | Contenedor Docker `postgres` |
| SQL Server | SQL Server Management Studio (Windows), Azure Data Studio | Contenedor Docker de Microsoft |
| SQLite | DB Browser for SQLite, terminal `sqlite3` | No necesita servidor: es solo un archivo |
| Oracle | SQL Developer | Contenedor Docker de Oracle Free |
| **Cualquiera** | **DBeaver** (una sola herramienta gratuita para conectar con todos) | — |

Para los contenedores, mira la carpeta `Docker/`. Para empezar rápido, **SQLite** o **DBeaver** son lo más sencillo.

### Comandos útiles para explorar

Estos comandos **no son SQL estándar**: son del cliente de cada sistema.

| Qué quieres | MySQL / MariaDB | PostgreSQL (`psql`) | SQL Server | SQLite (`sqlite3`) | Oracle |
| --- | --- | --- | --- | --- | --- |
| Listar bases de datos | `SHOW DATABASES;` | `\l` | `SELECT name FROM sys.databases;` | `.databases` | — |
| Usar una base de datos | `USE tienda;` | `\c tienda` | `USE tienda;` | (abrir el archivo) | — |
| Listar tablas | `SHOW TABLES;` | `\dt` | `SELECT name FROM sys.tables;` | `.tables` | `SELECT table_name FROM user_tables;` |
| Ver columnas | `DESCRIBE clientes;` | `\d clientes` | `EXEC sp_help 'clientes';` | `PRAGMA table_info(clientes);` | `DESC clientes;` |

Hay también una forma casi estándar de consultar la estructura, que funciona en MySQL, PostgreSQL y SQL Server (no en SQLite ni Oracle):

```sql
SELECT table_name FROM information_schema.tables;
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'clientes';
```

## 11. Buenas prácticas

- Escribe **SQL estándar** siempre que puedas; usa lo propio de un sistema solo cuando haga falta.
- Nombres en **minúsculas con `_`**, sin espacios ni acentos, y sin palabras reservadas.
- Palabras clave en MAYÚSCULAS y nombres en minúsculas, para leer mejor.
- **Prueba tus consultas en el sistema que vas a usar de verdad**, no solo en el que tienes a mano.
- Guarda el script de creación de la base de datos en un archivo (`schema.sql`) y súbelo a Git.
- Si el código va a cambiar de sistema algún día, anota qué partes son específicas.

## 12. Diferencias importantes

| Cosa | Detalle |
| --- | --- |
| SQL vs SGBD | SQL es el **lenguaje**; el SGBD es el **programa** (MySQL, PostgreSQL...) |
| Estándar vs dialecto | Lo estándar funciona en todos; el dialecto (`LIMIT`, `TOP`, `AUTO_INCREMENT`...) solo en uno |
| Servidor vs archivo | MySQL, PostgreSQL, SQL Server y Oracle son servidores; SQLite es un archivo |
| SQL vs NoSQL | SQL usa tablas relacionales y este lenguaje; NoSQL (MongoDB...) usa otros modelos y lenguajes distintos |

## 13. Casos especiales

❌ El script de ejemplo falla al crear una tabla o insertar en un sistema concreto.
🤔 Una de las variantes de la tabla anterior (fechas en Oracle, `CREATE DATABASE` en SQLite...).
✅ Revisa el aviso de variantes y ajusta solo esa línea.

❌ `WHERE nombre = 'ana'` no devuelve nada en PostgreSQL, pero sí en MySQL.
🤔 PostgreSQL distingue mayúsculas en los textos; MySQL no.
✅ Usa `LOWER(nombre) = 'ana'`, que funciona igual en todos.

❌ En SQLite puedo insertar un pedido de un cliente que no existe.
🤔 Las claves foráneas están desactivadas por defecto.
✅ Ejecuta `PRAGMA foreign_keys = ON;` al abrir la base de datos.

❌ `SELECT 1 + 1;` da error en Oracle.
🤔 Oracle (hasta la versión 23ai) exige tabla en el `SELECT`.
✅ Usa `SELECT 1 + 1 FROM DUAL;`.

❌ Un campo guardado como `''` no sale al buscar `IS NULL` en MySQL, pero sí en Oracle.
🤔 Oracle considera el texto vacío como `NULL`.
✅ No dependas de eso: guarda `NULL` cuando no hay dato.

- **Códigos de error y mensajes:** son distintos en cada sistema. Si buscas un error en internet, añade el nombre del sistema.
- **Versiones:** algunas funciones (`CHECK` en MySQL, `IDENTITY` en Oracle, `WITH` en MySQL) solo existen a partir de cierta versión.

## 14. Resumen

- SQL = lenguaje para trabajar con bases de datos relacionales; el SGBD es el programa que lo ejecuta.
- Hay un **SQL estándar** que casi todos cumplen, más **extensiones propias** de cada sistema (dialectos).
- Sistemas principales: MySQL/MariaDB, PostgreSQL, SQL Server, SQLite y Oracle.
- Grupos de comandos: DQL (leer), DML (cambiar datos), DDL (estructura), DCL (permisos), TCL (transacciones).
- Textos y fechas entre comillas simples; nombres especiales entrecomillados de forma distinta según el sistema.
- Cambian entre sistemas: autonumérico, tipos de fecha y texto, mayúsculas en comparaciones, y cómo explorar la estructura.
- `NULL` = sin valor; se comprueba con `IS NULL`.
- Dinero con `DECIMAL`. Escribe en minúsculas y prueba en el sistema real.