# 17 - Transacciones

> [!info] ¿Qué es?
> Una **transacción** es un grupo de instrucciones SQL que se ejecutan **como si fueran una sola**: o se hacen **todas** correctamente, o **no se hace ninguna**. Sirve para que la base de datos nunca se quede a medias si algo falla en mitad de una operación.

---

## 1. Antes de empezar

Imagina una transferencia bancaria de 100 € de Ana a Luis. Son **dos pasos**:

1. Restar 100 € a la cuenta de Ana.
2. Sumar 100 € a la cuenta de Luis.

¿Qué pasa si el sistema se cae justo entre el paso 1 y el paso 2? Ana habría perdido 100 € y Luis no los habría recibido: **el dinero desaparece**. Una transacción evita esto: si algo falla, se **deshace** todo y vuelve al estado inicial.

Lo que necesitas saber de antes:

- Instrucciones que modifican datos: `INSERT`, `UPDATE`, `DELETE` (ver [[SQL 09 - INSERT]], [[SQL 10 - UPDATE]] y [[SQL 11 - DELETE]]).
- Una **conexión** o **sesión** es tu "conversación" abierta con la base de datos. Las transacciones viven dentro de una sesión.

---

## 2. Concepto fundamental

### El ciclo de una transacción

Toda transacción sigue siempre este camino:

1. **Empieza** (`BEGIN` / `START TRANSACTION`).
2. Ejecutas tus instrucciones. Los cambios son **provisionales**: solo los ves tú.
3. Terminas de **una de dos formas**:
   - `COMMIT`: **confirmas**. Los cambios se guardan de forma definitiva y los ven los demás.
   - `ROLLBACK`: **cancelas**. Todos los cambios se deshacen como si no hubieran existido.

### Propiedades ACID

Una transacción fiable cumple cuatro garantías, conocidas por sus siglas:

| Letra | Nombre | Qué garantiza |
|---|---|---|
| **A** | **Atomicidad** | Todo o nada. Nunca se queda a medias. |
| **C** | **Consistencia** | La base de datos pasa de un estado válido a otro válido (se respetan claves, restricciones, etc.). |
| **I** | **Aislamiento** (*Isolation*) | Varias transacciones a la vez no se estorban entre sí. |
| **D** | **Durabilidad** | Una vez hecho `COMMIT`, los datos se conservan aunque se caiga el sistema. |

> [!tip] Truco para recordarlo
> **A**tómica, **C**oherente, **A**islada, **D**urable → ACID.

### Autocommit

Por defecto, la mayoría de sistemas funcionan en modo **autocommit**: **cada instrucción es su propia transacción** y se confirma sola al terminar. Por eso un `UPDATE` suelto queda guardado sin que hagas nada. Cuando abres una transacción de forma explícita, ese modo se pausa hasta que haces `COMMIT` o `ROLLBACK`.

---

## 3. Sintaxis / estructura

### Estructura básica

```sql
START TRANSACTION;   -- o BEGIN;

-- ... tus instrucciones (INSERT, UPDATE, DELETE) ...

COMMIT;              -- confirmar
-- o bien:
ROLLBACK;            -- cancelar todo
```

### Puntos de guardado (`SAVEPOINT`)

Permiten deshacer **solo una parte** de la transacción:

```sql
SAVEPOINT nombre_punto;          -- crear un punto de guardado
ROLLBACK TO nombre_punto;        -- volver a ese punto (la transacción sigue abierta)
RELEASE SAVEPOINT nombre_punto;  -- borrar el punto (ya no lo necesitas)
```

### Nivel de aislamiento

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

> [!warning] Diferencias entre sistemas
> La forma de **abrir** una transacción cambia un poco:
> - **MySQL / MariaDB**: `START TRANSACTION;` o `BEGIN;`
> - **PostgreSQL**: `BEGIN;` o `START TRANSACTION;`
> - **SQL Server**: `BEGIN TRANSACTION;` (cerrar con `COMMIT TRANSACTION;`)
> - **SQLite**: `BEGIN;` (también `BEGIN TRANSACTION;`)
> - **Oracle**: no necesita `BEGIN`: la transacción empieza sola con la primera instrucción que modifica datos y termina con `COMMIT` o `ROLLBACK`.
>
> `COMMIT` y `ROLLBACK` funcionan igual en todos.

---

## 4. Elementos / propiedades / características

### 4.1 `COMMIT` y `ROLLBACK`

| Instrucción | Qué hace | Se puede deshacer después |
|---|---|---|
| `COMMIT` | Guarda todos los cambios de forma permanente | **No** |
| `ROLLBACK` | Descarta todos los cambios desde el inicio de la transacción | — |

> [!warning] Cuidado
> Una vez hecho `COMMIT`, **ya no hay marcha atrás** con `ROLLBACK`. Por eso conviene comprobar con un `SELECT` antes de confirmar.

### 4.2 Qué ocurre si algo falla

- Si se **cae la conexión** o el servidor antes del `COMMIT`, la base de datos hace `ROLLBACK` automáticamente al recuperarse.
- Si una instrucción da **error**, en algunos sistemas la transacción entera queda inutilizable hasta que hagas `ROLLBACK` (PostgreSQL), y en otros puedes seguir (MySQL). Lo más seguro es **hacer `ROLLBACK` tú mismo** cuando detectes un error.

### 4.3 Qué instrucciones se pueden deshacer

Las instrucciones que modifican **datos** (`INSERT`, `UPDATE`, `DELETE`) se pueden deshacer con `ROLLBACK`.

Las instrucciones que cambian la **estructura** (`CREATE`, `ALTER`, `DROP`) dependen del sistema:

| Sistema | ¿Se pueden deshacer `CREATE`/`ALTER`/`DROP`? |
|---|---|
| PostgreSQL, SQL Server, SQLite | **Sí** (son transaccionales) |
| MySQL, MariaDB, Oracle | **No**: provocan un `COMMIT` implícito (confirman lo anterior) |

### 4.4 Problemas de concurrencia

Cuando **varios usuarios** trabajan a la vez, pueden aparecer estos problemas:

| Problema | Qué pasa | Ejemplo |
|---|---|---|
| **Lectura sucia** (*dirty read*) | Lees datos de otra transacción que **aún no ha confirmado** (y quizá se cancele) | Ves un saldo que luego se deshace con `ROLLBACK` |
| **Lectura no repetible** (*non-repeatable read*) | Lees la misma fila dos veces y **el valor cambia** entre medias | Consultas un precio y, al repetir, otro usuario ya lo ha modificado |
| **Lectura fantasma** (*phantom read*) | Repites una consulta y **aparecen o desaparecen filas** | Cuentas 5 pedidos y, al repetir, hay 6 |

### 4.5 Niveles de aislamiento

El estándar define cuatro niveles. Cuanto más alto, **más protección** pero **menos rendimiento**:

| Nivel | Lectura sucia | Lectura no repetible | Lectura fantasma |
|---|---|---|---|
| `READ UNCOMMITTED` | Posible | Posible | Posible |
| `READ COMMITTED` | Evitada | Posible | Posible |
| `REPEATABLE READ` | Evitada | Evitada | Posible* |
| `SERIALIZABLE` | Evitada | Evitada | Evitada |

\* Según el sistema, `REPEATABLE READ` puede evitar también los fantasmas (en MySQL/InnoDB y PostgreSQL ocurre).

Nivel por defecto según el sistema:

- **PostgreSQL, Oracle, SQL Server**: `READ COMMITTED`
- **MySQL / MariaDB (InnoDB)**: `REPEATABLE READ`

### 4.6 Bloqueos (*locks*) e interbloqueos (*deadlocks*)

Para mantener el aislamiento, la base de datos **bloquea** filas (o tablas) mientras una transacción las usa; las demás deben **esperar**.

Un **interbloqueo** (*deadlock*) ocurre cuando dos transacciones se esperan mutuamente para siempre:

- La transacción 1 bloquea la fila A y quiere la B.
- La transacción 2 bloquea la fila B y quiere la A.

Ninguna puede avanzar. La base de datos lo detecta y **cancela una de las dos** (devuelve un error). Tu aplicación debería poder **reintentar** la transacción cancelada.

---

## 5. Ejemplos prácticos

### Ejemplo básico: transferencia bancaria

```sql
START TRANSACTION;

UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;  -- Ana pierde 100
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;  -- Luis gana 100

COMMIT;
```

Si el segundo `UPDATE` falla, en vez de `COMMIT` ejecutas `ROLLBACK` y las cuentas quedan como estaban.

Un patrón más seguro, comprobando antes de confirmar:

```sql
START TRANSACTION;

UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;

-- Comprobamos que nadie se ha quedado en negativo
SELECT id, saldo FROM cuentas WHERE id IN (1, 2);

-- Si todo está bien:
COMMIT;
-- Si algo no cuadra:
-- ROLLBACK;
```

### Ejemplo habitual: pedido con varias tablas

Crear un pedido implica insertar en **varias tablas** que deben quedar coherentes entre sí:

```sql
START TRANSACTION;

INSERT INTO pedidos (id, fecha, cliente_id) VALUES (500, '2026-10-15', 1);

INSERT INTO lineas_pedido (pedido_id, producto_id, cantidad) VALUES (500, 10, 2);
INSERT INTO lineas_pedido (pedido_id, producto_id, cantidad) VALUES (500, 11, 1);

UPDATE productos SET stock = stock - 2 WHERE id = 10;
UPDATE productos SET stock = stock - 1 WHERE id = 11;

COMMIT;
```

Si cualquier paso falla, el `ROLLBACK` evita tener un pedido sin líneas o stock descontado sin pedido.

### Ejemplo completo: `SAVEPOINT`

```sql
START TRANSACTION;

INSERT INTO clientes (id, nombre, email) VALUES (50, 'Marta', 'marta@mail.com');

SAVEPOINT despues_cliente;

INSERT INTO pedidos (id, fecha, cliente_id) VALUES (900, '2026-10-15', 50);
-- Algo sale mal con el pedido...

ROLLBACK TO despues_cliente;  -- deshace solo el pedido; el cliente sigue ahí

COMMIT;  -- se guarda únicamente el cliente
```

---

## 6. Buenas prácticas

- **Agrupa en una transacción todo lo que debe ir junto** (una transferencia, un pedido completo, etc.).
- **Mantenlas cortas**: cuanto más dura una transacción, más tiempo bloquea filas y más molestas a los demás usuarios.
- **Nunca dejes una transacción abierta** (sin `COMMIT` ni `ROLLBACK`): puede bloquear tablas durante horas.
- **No pidas datos al usuario dentro de una transacción**: haz primero todo lo que necesites preguntar y luego abre la transacción.
- **Haz `ROLLBACK` ante cualquier error** en lugar de seguir como si nada.
- **Comprueba con `SELECT` antes de `COMMIT`** cuando hagas cambios importantes a mano (por ejemplo, un `UPDATE` o `DELETE` masivo).
- **Accede a las tablas siempre en el mismo orden** en todo tu código: así reduces mucho el riesgo de interbloqueos.
- **Usa el nivel de aislamiento más bajo que te sirva**: el más alto (`SERIALIZABLE`) es el más seguro pero el más lento.
- **Prepara la aplicación para reintentar** si la base de datos cancela una transacción por interbloqueo.
- **Usa un motor transaccional**: en MySQL, `InnoDB` (el de por defecto hoy) soporta transacciones; el antiguo `MyISAM` **no**.

---

## 7. Diferencias importantes

### `COMMIT` vs `ROLLBACK`

| | `COMMIT` | `ROLLBACK` |
|---|---|---|
| Efecto | Guarda los cambios | Los descarta |
| ¿Reversible? | No | — |
| Cuándo usarlo | Todo ha ido bien | Algo ha fallado |

### Autocommit vs transacción explícita

| | Autocommit | Transacción explícita |
|---|---|---|
| Cuándo se guarda | Tras cada instrucción | Al hacer `COMMIT` |
| ¿Se puede deshacer? | No | Sí, hasta el `COMMIT` |
| Uso típico | Consultas y cambios sueltos | Operaciones de varios pasos |

### `ROLLBACK` vs `ROLLBACK TO savepoint`

- `ROLLBACK`: **cancela toda** la transacción y la cierra.
- `ROLLBACK TO savepoint`: deshace **solo hasta ese punto**; la transacción **sigue abierta**.

### Transacción vs bloqueo

La **transacción** es el grupo de operaciones; el **bloqueo** es el mecanismo interno que la base de datos usa para que varias transacciones no se pisen. Tú abres transacciones; los bloqueos los gestiona el sistema.

### Transacciones vs restricciones

Las **restricciones** (`FOREIGN KEY`, `CHECK`...) protegen que **cada dato** sea válido. Las **transacciones** protegen que **un conjunto de cambios** se aplique completo o nada. Se complementan.

---

## 8. Casos especiales

- **`COMMIT` implícito**: en MySQL, Oracle y MariaDB, instrucciones como `CREATE`, `ALTER` o `DROP` confirman automáticamente lo que hubiera pendiente. Después de ellas ya no puedes hacer `ROLLBACK` de lo anterior.
- **Transacciones anidadas**: el SQL estándar no las admite como tal. Si necesitas "deshacer una parte", usa `SAVEPOINT`.
- **Lecturas con bloqueo**: `SELECT ... FOR UPDATE` bloquea las filas leídas hasta el final de la transacción, para que nadie las cambie mientras decides. Es muy útil en reservas y stock (lo admiten MySQL, PostgreSQL y Oracle; SQL Server usa pistas como `WITH (UPDLOCK)`).
- **Tiempo de espera (*timeout*)**: si una transacción espera demasiado un bloqueo, la base de datos cancela la instrucción con un error. Conviene capturarlo y reintentar.
- **Error en mitad de una transacción**: en **PostgreSQL**, tras un error la transacción queda "abortada" y todo lo siguiente falla hasta que haces `ROLLBACK`. En **MySQL**, solo falla la instrucción que dio error y puedes continuar.
- **Transacciones largas en SQLite**: SQLite bloquea **toda la base de datos** al escribir, así que las transacciones de escritura deben ser especialmente cortas.
- **Transacciones distribuidas**: si una operación afecta a **varias bases de datos o servidores**, hace falta un protocolo extra (*two-phase commit*). Es un tema avanzado y poco habitual al empezar.
- **Aplicaciones web**: normalmente no escribes `START TRANSACTION` a mano: el lenguaje (PHP con PDO, Node, Python...) ofrece funciones para empezar, confirmar y cancelar, pero por debajo hacen exactamente lo mismo.

---

## 9. Resumen

| Concepto | Idea clave |
|---|---|
| Transacción | Grupo de instrucciones que se ejecutan **todas o ninguna** |
| Inicio | `START TRANSACTION` / `BEGIN` (Oracle: automático) |
| Confirmar | `COMMIT`: guarda de forma definitiva |
| Cancelar | `ROLLBACK`: deshace todo |
| `SAVEPOINT` | Punto intermedio para deshacer solo una parte |
| ACID | Atomicidad, Consistencia, Aislamiento, Durabilidad |
| Autocommit | Cada instrucción se confirma sola |
| Problemas de concurrencia | Lectura sucia, no repetible y fantasma |
| Niveles de aislamiento | `READ UNCOMMITTED` < `READ COMMITTED` < `REPEATABLE READ` < `SERIALIZABLE` |
| Deadlock | Dos transacciones que se esperan entre sí; la BD cancela una |
| Regla de oro | Transacciones **cortas** y nunca dejarlas abiertas |

> [!tip] Para recordar en una frase
> **Una transacción es "todo o nada": `COMMIT` si todo va bien, `ROLLBACK` si algo falla.**

Siguiente nota: [[SQL 18 - Vistas]]