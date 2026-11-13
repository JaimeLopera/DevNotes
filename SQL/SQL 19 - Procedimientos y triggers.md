# 19 - Procedimientos y triggers

> [!info] ¿Qué son?
> Son dos formas de **guardar código dentro de la propia base de datos**:
> - Un **procedimiento almacenado** (`PROCEDURE`) es un bloque de instrucciones SQL con nombre que **tú ejecutas cuando quieres** con `CALL`.
> - Un **trigger** (disparador) es un bloque de instrucciones que **se ejecuta solo**, de forma automática, cuando ocurre un evento en una tabla (`INSERT`, `UPDATE` o `DELETE`).
>
> También veremos las **funciones almacenadas** (`FUNCTION`), que son parecidas a los procedimientos pero devuelven un valor.

---

## 1. Antes de empezar

Hasta ahora has escrito instrucciones SQL sueltas, una a una. Pero a veces necesitas **programar**: usar variables, condiciones (`IF`), bucles, y repetir la misma lógica muchas veces. SQL "puro" casi no puede hacer eso, así que cada sistema añade un **lenguaje procedural** encima:

| Sistema | Lenguaje procedural |
|---|---|
| MySQL / MariaDB | SQL/PSM (el propio SQL ampliado) |
| PostgreSQL | **PL/pgSQL** |
| Oracle | **PL/SQL** |
| SQL Server | **T-SQL** |
| SQLite | No tiene procedimientos ni funciones almacenadas |

> [!warning] Lo más importante de este tema
> A diferencia de casi todo lo anterior, **la sintaxis de procedimientos y triggers cambia MUCHO entre sistemas**. Las ideas son las mismas en todos, pero el código no se copia y pega de uno a otro. En esta nota los ejemplos principales usan **MySQL/MariaDB**, y en cada apartado verás cómo cambia en los demás.

Lo que necesitas saber de antes:

- `INSERT`, `UPDATE`, `DELETE` y transacciones (ver [[SQL 09 - INSERT]], [[SQL 10 - UPDATE]], [[SQL 11 - DELETE]] y [[SQL 17 - Transacciones]]).
- Qué es una vista (ver [[SQL 18 - Vistas]]), porque también guarda lógica en la base de datos.

---

## 2. Concepto fundamental

### Los tres tipos de código almacenado

| | **Procedimiento** | **Función** | **Trigger** |
|---|---|---|---|
| Cómo se ejecuta | Tú lo llamas: `CALL nombre(...)` | Se usa dentro de una consulta: `SELECT nombre(...)` | **Automáticamente**, al ocurrir un evento |
| ¿Devuelve valor? | No obligatoriamente (puede devolver datos por parámetros de salida) | **Sí**, siempre uno | No |
| ¿Recibe parámetros? | Sí | Sí | **No** (usa `OLD` y `NEW`) |
| Para qué sirve | Realizar una tarea de varios pasos | Calcular un valor reutilizable | Reaccionar a cambios en una tabla |

### Procedimiento almacenado

Es un "mini programa" guardado en la base de datos. Lo escribes una vez y lo llamas cuantas veces quieras. Puede recibir **parámetros** y contener variables, condiciones, bucles y varias instrucciones.

**Ventajas**:

- **Reutilizas** la lógica en lugar de repetirla.
- Menos viajes entre la aplicación y la base de datos (se ejecuta todo en el servidor).
- Más **seguridad**: puedes dar permiso para ejecutar el procedimiento sin dar acceso directo a las tablas.
- La lógica está **en un único sitio**, y la usan todas las aplicaciones.

### Función almacenada

Igual que un procedimiento, pero **devuelve un único valor** y se puede usar dentro de un `SELECT`, `WHERE`, etc., como las funciones que ya conoces (`UPPER`, `SUM`...). Ver [[SQL 05 - Funciones]].

### Trigger

Es código que se **dispara solo** cuando alguien hace un `INSERT`, `UPDATE` o `DELETE` sobre una tabla. No lo llamas tú: la base de datos lo ejecuta por ti.

Un trigger se define con tres cosas:

| Elemento | Opciones |
|---|---|
| **Evento** | `INSERT`, `UPDATE`, `DELETE` |
| **Momento** | `BEFORE` (antes de aplicar el cambio) o `AFTER` (después) |
| **Alcance** | Por cada fila afectada (`FOR EACH ROW`) |

Dentro de un trigger tienes acceso a los datos de la fila con dos palabras especiales:

| | `OLD` | `NEW` |
|---|---|---|
| Qué es | El valor de la fila **antes** del cambio | El valor de la fila **después** del cambio |
| En `INSERT` | No existe | Sí |
| En `UPDATE` | Sí | Sí |
| En `DELETE` | Sí | No existe |

---

## 3. Sintaxis / estructura

### Procedimiento almacenado (MySQL / MariaDB)

```sql
DELIMITER //

CREATE PROCEDURE nombre_procedimiento (
    IN  parametro_entrada INT,
    OUT parametro_salida  INT
)
BEGIN
    -- instrucciones
    SELECT COUNT(*) INTO parametro_salida
    FROM pedidos
    WHERE cliente_id = parametro_entrada;
END //

DELIMITER ;
```

Llamarlo:

```sql
CALL nombre_procedimiento(1, @resultado);
SELECT @resultado;
```

### Qué es `DELIMITER`

Normalmente el `;` termina una instrucción. Pero dentro de un procedimiento hay muchos `;`, y el cliente de MySQL creería que el procedimiento acaba en el primero. Por eso se **cambia temporalmente el delimitador** a `//` (o `$$`), se escribe el bloque, y después se **vuelve a poner `;`**. Solo hace falta en clientes de consola o scripts; **no es SQL real**, es una orden del cliente.

### Tipos de parámetros

| Modo | Significado |
|---|---|
| `IN` | **Entrada**: el procedimiento recibe el valor (por defecto) |
| `OUT` | **Salida**: el procedimiento devuelve un valor |
| `INOUT` | Entra con un valor y sale modificado |

### Función almacenada (MySQL / MariaDB)

```sql
DELIMITER //

CREATE FUNCTION nombre_funcion (parametro INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE resultado DECIMAL(10, 2);
    -- calcular...
    RETURN resultado;
END //

DELIMITER ;
```

Usarla:

```sql
SELECT nombre_funcion(5);
```

### Trigger (MySQL / MariaDB)

```sql
CREATE TRIGGER nombre_trigger
{BEFORE | AFTER} {INSERT | UPDATE | DELETE}
ON tabla
FOR EACH ROW
BEGIN
    -- instrucciones usando OLD y NEW
END;
```

(Si el cuerpo tiene varias instrucciones, también se usa `DELIMITER`.)

### Borrar

```sql
DROP PROCEDURE IF EXISTS nombre_procedimiento;
DROP FUNCTION  IF EXISTS nombre_funcion;
DROP TRIGGER   IF EXISTS nombre_trigger;
```

### Lo básico de programar dentro

```sql
DECLARE total INT DEFAULT 0;      -- declarar una variable
SET total = total + 1;            -- asignar valor

IF total > 10 THEN
    -- ...
ELSEIF total > 5 THEN
    -- ...
ELSE
    -- ...
END IF;

WHILE total < 100 DO
    SET total = total + 1;
END WHILE;
```

> [!warning] Diferencias entre sistemas
> **Procedimientos**
> - **MySQL/MariaDB**: `CREATE PROCEDURE ... BEGIN ... END`, se llama con `CALL`.
> - **PostgreSQL**: `CREATE PROCEDURE ... LANGUAGE plpgsql AS $$ ... $$;` (desde la versión 11). Antes solo había funciones. Se llama con `CALL`.
> - **Oracle**: `CREATE PROCEDURE nombre IS ... BEGIN ... END;` y se ejecuta con `EXEC` o `CALL`.
> - **SQL Server**: `CREATE PROCEDURE nombre AS BEGIN ... END` y se ejecuta con `EXEC`. Las variables llevan `@` (`DECLARE @total INT`).
> - **SQLite**: **no los tiene**. La lógica se hace en la aplicación.
>
> **Triggers**
> - **MySQL/MariaDB y SQLite**: el cuerpo se escribe directamente en el `CREATE TRIGGER`.
> - **PostgreSQL**: hay que crear **primero una función** que devuelve `TRIGGER` y después asociarla con `CREATE TRIGGER ... EXECUTE FUNCTION nombre();`.
> - **SQL Server**: no tiene `FOR EACH ROW`; el trigger se ejecuta **una vez por instrucción** y usa las tablas virtuales `inserted` y `deleted` (en lugar de `NEW` y `OLD`).
> - **Oracle**: usa `:NEW` y `:OLD` (con dos puntos) y `FOR EACH ROW`.

---

## 4. Elementos / propiedades / características

### 4.1 Elementos de un procedimiento

| Elemento | Para qué sirve |
|---|---|
| Parámetros (`IN`, `OUT`, `INOUT`) | Pasar datos al procedimiento y recibir resultados |
| Variables (`DECLARE`, `SET`) | Guardar valores temporales |
| Condicionales (`IF`, `CASE`) | Tomar decisiones |
| Bucles (`WHILE`, `LOOP`, `REPEAT`) | Repetir instrucciones |
| Cursores (`CURSOR`) | Recorrer un resultado **fila a fila** |
| Manejo de errores (`HANDLER` / `EXCEPTION` / `TRY...CATCH`) | Reaccionar cuando algo falla |
| Transacciones (`START TRANSACTION`, `COMMIT`...) | Agrupar cambios dentro del procedimiento |

### 4.2 Momento del trigger: `BEFORE` vs `AFTER`

| | `BEFORE` | `AFTER` |
|---|---|---|
| Se ejecuta | **Antes** de guardar el cambio | **Después** de guardar el cambio |
| ¿Puede modificar los datos que se van a guardar? | **Sí** (cambiando `NEW`) | No |
| ¿Puede cancelar la operación? | Sí (lanzando un error) | Solo con error, ya deshace todo |
| Uso típico | **Validar** o **corregir** datos | **Registrar** (auditoría), actualizar otras tablas |

### 4.3 Usos típicos de los triggers

- **Auditoría**: guardar en una tabla de historial quién cambió qué y cuándo.
- **Validación**: impedir datos incorrectos que un `CHECK` no puede controlar.
- **Valores automáticos**: rellenar un campo (por ejemplo, `fecha_modificacion`).
- **Mantener datos derivados**: actualizar un total o un stock cuando cambia otra tabla.

### 4.4 Funciones: `DETERMINISTIC` y similares

En MySQL, al crear una función hay que indicar su naturaleza:

| Palabra | Significado |
|---|---|
| `DETERMINISTIC` | Con los mismos parámetros devuelve **siempre** el mismo resultado |
| `NOT DETERMINISTIC` | Puede devolver resultados distintos (usa fechas, aleatorios...) |
| `READS SQL DATA` | Solo lee datos |
| `MODIFIES SQL DATA` | Modifica datos |

### 4.5 Gestión de errores

Un procedimiento puede fallar a medias. Para controlarlo:

- **MySQL**: `DECLARE ... HANDLER` y `SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = '...'` para **lanzar un error propio**.
- **PostgreSQL**: `RAISE EXCEPTION '...'` y bloques `EXCEPTION`.
- **Oracle**: `RAISE_APPLICATION_ERROR(...)` y bloques `EXCEPTION`.
- **SQL Server**: `TRY ... CATCH` y `THROW`.

---

## 5. Ejemplos prácticos

### Ejemplo básico: procedimiento que busca pedidos de un cliente

```sql
DELIMITER //

CREATE PROCEDURE pedidos_de_cliente (IN p_cliente_id INT)
BEGIN
    SELECT id, fecha
    FROM pedidos
    WHERE cliente_id = p_cliente_id
    ORDER BY fecha DESC;
END //

DELIMITER ;

CALL pedidos_de_cliente(1);
```

> [!tip] Consejo
> Ponle un prefijo al parámetro (`p_cliente_id`) para que no se confunda con el nombre de una columna (`cliente_id`). Si se llaman igual, la base de datos puede interpretar mal la consulta.

### Ejemplo habitual: procedimiento con transacción (transferencia)

```sql
DELIMITER //

CREATE PROCEDURE transferir (
    IN p_origen  INT,
    IN p_destino INT,
    IN p_importe DECIMAL(10, 2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;  -- si algo falla, se deshace todo
    END;

    START TRANSACTION;

    UPDATE cuentas SET saldo = saldo - p_importe WHERE id = p_origen;
    UPDATE cuentas SET saldo = saldo + p_importe WHERE id = p_destino;

    COMMIT;
END //

DELIMITER ;

CALL transferir(1, 2, 100.00);
```

### Ejemplo completo: trigger de auditoría y trigger de validación

**Tabla de historial** (para guardar los cambios):

```sql
CREATE TABLE historial_precios (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    producto_id     INT NOT NULL,
    precio_anterior DECIMAL(10, 2),
    precio_nuevo    DECIMAL(10, 2),
    fecha_cambio    DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Trigger `AFTER UPDATE`**: registra cada cambio de precio.

```sql
DELIMITER //

CREATE TRIGGER trg_productos_precio
AFTER UPDATE ON productos
FOR EACH ROW
BEGIN
    IF OLD.precio <> NEW.precio THEN
        INSERT INTO historial_precios (producto_id, precio_anterior, precio_nuevo)
        VALUES (OLD.id, OLD.precio, NEW.precio);
    END IF;
END //

DELIMITER ;
```

**Trigger `BEFORE INSERT`**: impide precios negativos.

```sql
DELIMITER //

CREATE TRIGGER trg_productos_validar
BEFORE INSERT ON productos
FOR EACH ROW
BEGIN
    IF NEW.precio < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El precio no puede ser negativo';
    END IF;
END //

DELIMITER ;
```

Funcionamiento:

```sql
UPDATE productos SET precio = 50 WHERE id = 10;   -- el trigger anota el cambio solo
INSERT INTO productos (id, nombre, precio) VALUES (99, 'Test', -5);  -- error
```

### El mismo trigger en PostgreSQL (para comparar)

En PostgreSQL se necesitan **dos pasos**: la función y el trigger.

```sql
CREATE FUNCTION registrar_precio() RETURNS TRIGGER AS $$
BEGIN
    IF OLD.precio <> NEW.precio THEN
        INSERT INTO historial_precios (producto_id, precio_anterior, precio_nuevo)
        VALUES (OLD.id, OLD.precio, NEW.precio);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_productos_precio
AFTER UPDATE ON productos
FOR EACH ROW
EXECUTE FUNCTION registrar_precio();
```

---

## 6. Buenas prácticas

- **Usa un prefijo claro** en los nombres: `sp_` o `proc_` para procedimientos, `fn_` para funciones, `trg_` para triggers.
- **Prefija los parámetros** (`p_`) y las variables (`v_`) para no confundirlos con columnas.
- **Mantén los procedimientos cortos y con un solo propósito.** Si uno hace demasiadas cosas, divídelo.
- **Usa transacciones y manejo de errores** en los procedimientos que modifican varias tablas.
- **Documenta con comentarios** (`-- ...`) qué hace cada uno y qué parámetros espera.
- **Usa triggers con moderación.** Son potentes pero **invisibles**: el código se ejecuta "por detrás" y cuesta entender por qué cambia un dato.
- **Evita triggers que disparan otros triggers** en cadena: se vuelve muy difícil de depurar.
- **Para validar datos, prefiere antes las restricciones** (`NOT NULL`, `CHECK`, `FOREIGN KEY`, ver [[SQL 13 - Claves y restricciones]]). Usa un trigger solo cuando la restricción no pueda expresar la regla.
- **No pongas lógica pesada en un trigger**: se ejecuta en cada operación y puede ralentizar toda la tabla.
- **Guarda el código en archivos `.sql`** bajo control de versiones (Git), no solo dentro de la base de datos.
- **Dar permisos con cabeza**: concede `EXECUTE` sobre el procedimiento en lugar de acceso directo a las tablas cuando quieras limitar lo que puede hacer un usuario.

---

## 7. Diferencias importantes

### Procedimiento vs función

| | Procedimiento | Función |
|---|---|---|
| Se llama con | `CALL` / `EXEC` | Dentro de un `SELECT` |
| Devuelve | Nada o varios valores (`OUT`) | **Un valor** (`RETURN`) |
| Puede modificar datos | Sí | Depende del sistema (a menudo restringido) |
| Se usa para | Ejecutar tareas | Calcular valores |

### Procedimiento vs trigger

| | Procedimiento | Trigger |
|---|---|---|
| ¿Quién lo ejecuta? | **Tú**, cuando quieres | **La base de datos**, automáticamente |
| Parámetros | Sí | No (usa `OLD`/`NEW`) |
| Visibilidad | Explícito: ves el `CALL` | Oculto: ocurre "por detrás" |

### Trigger `BEFORE` vs `AFTER`

`BEFORE` sirve para **validar o modificar** lo que se va a guardar; `AFTER` para **reaccionar** a lo que ya se guardó.

### Trigger vs restricción (`CHECK`)

Un `CHECK` es **declarativo, más simple y más rápido**, pero solo mira la propia fila. Un trigger puede **consultar otras tablas** y hacer acciones complejas, pero es más difícil de mantener.

### Lógica en la base de datos vs en la aplicación

| | En la base de datos (procedimientos/triggers) | En la aplicación (PHP, JS, Python...) |
|---|---|---|
| Ventajas | Rápido, centralizado, lo usan todas las apps | Más fácil de probar, versionar y depurar |
| Desventajas | Difícil de depurar y versionar; ata la app a un sistema concreto | La lógica se puede duplicar entre aplicaciones |

No hay una respuesta única: muchos proyectos usan solo lo imprescindible en la base de datos.

---

## 8. Casos especiales

- **Triggers y carga masiva**: un `INSERT` de un millón de filas dispara el trigger **un millón de veces** (en sistemas con `FOR EACH ROW`). Puede hacer lentísima la operación.
- **Recursividad**: un trigger que modifica su propia tabla puede dispararse a sí mismo sin fin. Muchos sistemas lo prohíben o limitan; hay que evitarlo.
- **Triggers en cascada**: un trigger activa otro, que activa otro... Es una de las causas más típicas de comportamientos "misteriosos".
- **`TRUNCATE` no dispara triggers `DELETE`**: vacía la tabla sin pasar por ellos (en la mayoría de sistemas).
- **Los borrados en cascada de una `FOREIGN KEY` pueden no disparar triggers** en algunos sistemas (por ejemplo, en MySQL las acciones referenciales no activan triggers).
- **Triggers `INSTEAD OF`**: existen en SQL Server, Oracle, PostgreSQL y SQLite; sirven sobre todo para hacer **actualizables las vistas complejas** (ver [[SQL 18 - Vistas]]). MySQL no los tiene.
- **Cursores**: recorrer fila a fila es **lento** comparado con una sola instrucción SQL que actúe sobre todo el conjunto. Úsalos solo cuando no haya alternativa.
- **Seguridad: inyección SQL**: si dentro de un procedimiento construyes consultas concatenando texto (SQL dinámico), puedes abrir la puerta a inyección SQL. Usa parámetros (`PREPARE`/`EXECUTE` con `?` en MySQL, `EXECUTE ... USING` en PostgreSQL).
- **Portabilidad**: todo el código de este tema está muy ligado al sistema. Si algún día migras de MySQL a PostgreSQL, **tendrás que reescribirlo**.
- **Opciones de seguridad**: algunos sistemas permiten que el procedimiento se ejecute con los permisos de su creador (`SQL SECURITY DEFINER` en MySQL) o de quien lo llama (`INVOKER`).

---

## 9. Resumen

| Concepto | Idea clave |
|---|---|
| Procedimiento almacenado | Bloque de SQL con nombre que ejecutas con `CALL` / `EXEC` |
| Función almacenada | Como un procedimiento pero **devuelve un valor** y se usa en consultas |
| Trigger | Código que se dispara **solo** ante `INSERT`, `UPDATE` o `DELETE` |
| Parámetros | `IN` (entrada), `OUT` (salida), `INOUT` (ambos) |
| `OLD` / `NEW` | Valores de la fila antes / después del cambio |
| `BEFORE` | Antes del cambio: validar o modificar |
| `AFTER` | Después del cambio: auditar o actualizar otras tablas |
| `DELIMITER` | Truco del cliente MySQL para poder escribir varios `;` dentro del bloque |
| Lenguaje | Cambia según el sistema: PL/pgSQL, PL/SQL, T-SQL... |
| SQLite | No tiene procedimientos; solo triggers |
| Cuidado | Los triggers son invisibles: úsalos con moderación |

> [!tip] Para recordar en una frase
> **Un procedimiento lo llamas tú cuando quieres; un trigger se ejecuta solo cuando ocurre un cambio en la tabla.**

Siguiente nota: [[SQL 20 - Referencia rápida]]