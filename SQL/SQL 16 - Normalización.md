# 16 - Normalización

> [!info] ¿Qué es?
> La **normalización** es el proceso de **organizar las tablas de una base de datos para que los datos no se repitan** y no se líen. Se hace siguiendo unas reglas llamadas **formas normales** (1FN, 2FN, 3FN...). Cada forma normal arregla un tipo concreto de problema. El objetivo: que cada dato se guarde **una sola vez y en el lugar que le toca**.

---

## 1. Antes de empezar

Lo que necesitas saber de antes (ver [[SQL 13 - Claves y restricciones]] y [[SQL 15 - Relaciones]]):

- **Clave primaria**: identifica cada fila de forma única.
- **Clave foránea**: columna que apunta a la clave primaria de otra tabla.
- **Clave compuesta**: clave primaria formada por **dos o más columnas** juntas.
- **Dependencia**: decimos que la columna B *depende* de la columna A cuando, si conoces A, ya sabes cuál es B. Ejemplo: si conoces el `id` de un cliente, ya sabes su nombre. Se escribe `id → nombre`.

Esa idea de **dependencia** es la base de toda la normalización. Si la entiendes, lo demás sale solo.

---

## 2. Concepto fundamental

### El problema: la tabla "todo en uno"

Mira esta tabla de pedidos mal diseñada:

| pedido_id | cliente | email_cliente | producto | precio | ciudad_cliente |
|---|---|---|---|---|---|
| 1 | Ana | ana@mail.com | Teclado | 40 | Madrid |
| 2 | Ana | ana@mail.com | Ratón | 20 | Madrid |
| 3 | Luis | luis@mail.com | Teclado | 40 | Sevilla |

Los datos de Ana están repetidos y el precio del teclado también. Esto provoca tres problemas, llamados **anomalías**:

| Anomalía | Qué pasa | Ejemplo |
|---|---|---|
| **De inserción** | No puedes guardar un dato sin otro que no tiene que ver. | No puedes dar de alta un cliente nuevo hasta que haga un pedido. |
| **De actualización** | Cambiar un dato obliga a cambiarlo en muchas filas (y es fácil olvidar alguna). | Ana cambia de email: hay que modificarlo en todas sus filas. |
| **De borrado** | Al borrar una fila pierdes datos que querías conservar. | Borras el único pedido de Luis y desaparece también la información de Luis. |

### La solución

Dividir la tabla en varias, **cada una sobre una sola cosa**, y unirlas con claves foráneas. Eso es normalizar.

### Las formas normales

Son niveles acumulativos: para estar en 2FN hay que cumplir antes la 1FN, y así sucesivamente.

| Forma normal | Regla en una frase |
|---|---|
| **1FN** | Cada celda tiene **un solo valor** (nada de listas ni grupos repetidos). |
| **2FN** | Todo depende de **toda** la clave primaria (no solo de una parte). |
| **3FN** | Todo depende **solo de la clave** y de nada más (sin dependencias entre columnas normales). |
| **FNBC** | Versión más estricta de la 3FN: todo determinante debe ser clave. |

> [!tip] La frase para recordarlas
> Cada columna debe depender de **la clave** (1FN), **de toda la clave** (2FN) y **de nada más que la clave** (3FN).

En la práctica, **llegar a la 3FN es lo normal** y suficiente en casi todos los proyectos. La 4FN y 5FN existen, pero casi nunca se aplican.

---

## 3. Sintaxis / estructura

La normalización **no es una instrucción SQL**: es una forma de *diseñar* las tablas. No existe un `NORMALIZE`. Lo que haces es crear varias tablas con `CREATE TABLE` y unirlas con claves foráneas.

El proceso sigue siempre los mismos pasos:

1. **Reúne todos los datos** que necesitas guardar (la tabla "todo en uno").
2. **Identifica la clave** que distingue cada fila.
3. **Aplica 1FN**: separa los valores múltiples y los grupos repetidos.
4. **Aplica 2FN**: saca a otra tabla lo que depende solo de una parte de la clave.
5. **Aplica 3FN**: saca a otra tabla lo que depende de otra columna normal.
6. **Une las tablas** con claves foráneas (ver [[SQL 15 - Relaciones]]).
7. **Comprueba** que con un `JOIN` puedes reconstruir la información original.

---

## 4. Elementos / propiedades / características

### 4.1 Primera forma normal (1FN)

**Regla**: cada celda contiene **un único valor** (valores *atómicos*), no hay columnas repetidas del estilo `telefono1, telefono2, telefono3`, y cada fila es única.

❌ **No cumple 1FN**:

| id | nombre | telefonos |
|---|---|---|
| 1 | Ana | 600111222, 600333444 |

Dos teléfonos en una celda: no puedes buscar, ordenar ni validar uno solo.

✅ **Cumple 1FN**: una fila por cada teléfono (en una tabla aparte).

| cliente_id | telefono |
|---|---|
| 1 | 600111222 |
| 1 | 600333444 |

### 4.2 Segunda forma normal (2FN)

**Regla**: cumple 1FN y, además, **ninguna columna depende solo de una parte de la clave primaria**.

> [!note] Solo importa con claves compuestas
> Si tu clave primaria es **una sola columna**, ya cumples 2FN automáticamente (no hay "parte" de la clave). El problema aparece cuando la clave tiene dos o más columnas.

❌ **No cumple 2FN**. Clave primaria: `(pedido_id, producto_id)`.

| pedido_id | producto_id | cantidad | nombre_producto |
|---|---|---|---|
| 1 | 10 | 2 | Teclado |

`cantidad` depende de **la pareja** (pedido + producto): bien. Pero `nombre_producto` depende **solo de `producto_id`**: es una dependencia parcial.

✅ **Solución**: sacar el nombre a la tabla `productos`.

### 4.3 Tercera forma normal (3FN)

**Regla**: cumple 2FN y, además, **ninguna columna normal depende de otra columna normal** (a esto se le llama *dependencia transitiva*).

❌ **No cumple 3FN**. Clave primaria: `id`.

| id | nombre | codigo_postal | ciudad |
|---|---|---|---|
| 1 | Ana | 28001 | Madrid |
| 2 | Luis | 41001 | Sevilla |

`ciudad` no depende directamente de `id`: depende de `codigo_postal`. La cadena es `id → codigo_postal → ciudad`.

✅ **Solución**: crear una tabla `codigos_postales` (`codigo_postal`, `ciudad`) y dejar en `clientes` solo el código postal.

### 4.4 Forma normal de Boyce-Codd (FNBC)

Es una 3FN un poco más estricta. **Regla**: si una columna (o grupo) determina a otra, esa columna tiene que ser una **clave candidata**.

La diferencia con la 3FN solo se nota en casos raros con **varias claves candidatas que se solapan**. Si tus tablas están en 3FN y no tienes casos raros así, casi seguro ya cumples FNBC.

### 4.5 Desnormalización

Es **deshacer la normalización a propósito**: duplicar algún dato para que las consultas sean más rápidas (evita `JOIN`s pesados). Se usa sobre todo en informes, almacenes de datos y sistemas con muchísimas lecturas. Tiene un coste: vuelven los riesgos de inconsistencia, y hay que mantener las copias sincronizadas.

---

## 5. Ejemplos prácticos

### Ejemplo básico: de la tabla "todo en uno" a 3FN

Partimos de la tabla `pedidos_mal` de la sección 2. Se normaliza en **cuatro tablas**, cada una sobre una sola cosa:

```sql
CREATE TABLE clientes (
    id     INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email  VARCHAR(150) NOT NULL UNIQUE,
    ciudad VARCHAR(100)
);

CREATE TABLE productos (
    id     INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10, 2) NOT NULL
);

CREATE TABLE pedidos (
    id         INT PRIMARY KEY,
    fecha      DATE NOT NULL,
    cliente_id INT NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes (id)
);

-- Tabla intermedia: qué productos lleva cada pedido
CREATE TABLE lineas_pedido (
    pedido_id   INT NOT NULL,
    producto_id INT NOT NULL,
    cantidad    INT NOT NULL,
    PRIMARY KEY (pedido_id, producto_id),
    FOREIGN KEY (pedido_id)   REFERENCES pedidos (id),
    FOREIGN KEY (producto_id) REFERENCES productos (id)
);
```

Ahora cada dato vive **en un solo sitio**:

- Ana cambia de email → se modifica **una fila** en `clientes`.
- El teclado cambia de precio → se modifica **una fila** en `productos`.
- Puedes dar de alta un cliente sin pedidos, y borrar un pedido sin perder al cliente.

La información original se recupera con `JOIN`s (ver [[SQL 07 - JOIN]]):

```sql
SELECT p.id AS pedido, c.nombre AS cliente, pr.nombre AS producto, l.cantidad
FROM pedidos p
JOIN clientes c       ON c.id = p.cliente_id
JOIN lineas_pedido l  ON l.pedido_id = p.id
JOIN productos pr     ON pr.id = l.producto_id;
```

### Ejemplo habitual: el precio histórico (cuándo NO "arreglar" la duplicación)

Fíjate en un detalle importante: en la solución anterior el precio está **solo** en `productos`. Si mañana el teclado sube de 40 € a 50 €, **los pedidos antiguos aparecerían con el precio nuevo**, y eso es incorrecto (el cliente pagó 40 €).

La solución correcta es **guardar el precio pagado en la línea del pedido**:

```sql
ALTER TABLE lineas_pedido
ADD COLUMN precio_unitario DECIMAL(10, 2) NOT NULL;
```

Esto parece duplicar `productos.precio`, pero **no es redundancia**: son dos datos distintos (el precio *actual* del producto y el precio *que se pagó* en aquel momento). Normalizar no significa eliminar todo lo que se parezca, sino que cada dato guarde **un solo hecho**.

---

## 6. Buenas prácticas

- **Apunta a la 3FN** como objetivo normal. Es el equilibrio entre orden y sencillez.
- **Una tabla = una cosa** (clientes, productos, pedidos). Si una tabla habla de dos cosas, divídela.
- **Cada dato, en un solo sitio.** Si lo ves repetido, pregúntate si debería tener su propia tabla.
- **Una columna = un solo valor.** Nunca guardes listas separadas por comas.
- **Empieza normalizando y desnormaliza solo si hay un problema real de rendimiento** (y medido), no "por si acaso".
- **No normalices en exceso**: partir una tabla en diez pequeñas por obsesión complica las consultas sin ganar nada.
- **Usa un `id` numérico como clave primaria** en vez de datos que puedan cambiar (nombre, email).
- **Comprueba el resultado**: con `JOIN` debes poder reconstruir la información original sin perder ni inventar filas.
- **Piensa en el diseño antes de crear tablas**: arreglar la estructura con datos ya cargados es mucho más difícil.

---

## 7. Diferencias importantes

### Las tres primeras formas normales de un vistazo

| | Qué arregla | Cómo detectarlo |
|---|---|---|
| **1FN** | Valores múltiples en una celda y grupos repetidos | Ves listas o columnas tipo `telefono1`, `telefono2` |
| **2FN** | Columnas que dependen de **parte** de una clave compuesta | Clave compuesta + una columna que solo habla de una de sus partes |
| **3FN** | Columnas que dependen de **otra columna normal** | Una columna que se podría deducir de otra que no es la clave |

### 3FN vs FNBC

La 3FN permite una excepción rara (cuando la columna que determina a otra forma parte de una clave). La FNBC no permite ninguna. En el día a día, si estás en 3FN, normalmente ya estás en FNBC.

### Normalizar vs desnormalizar

| | Normalizado | Desnormalizado |
|---|---|---|
| Datos repetidos | Casi ninguno | Algunos, a propósito |
| Escrituras (`INSERT`, `UPDATE`) | Más simples y seguras | Más complejas (hay que sincronizar copias) |
| Lecturas (`SELECT`) | Más `JOIN`s | Más rápidas |
| Riesgo de incoherencias | Bajo | Mayor |
| Uso típico | Aplicaciones normales (OLTP) | Informes y análisis (almacenes de datos) |

### Normalización vs relaciones

Son conceptos distintos que van de la mano: la **normalización** decide *cómo repartir* los datos en tablas, y las **relaciones** (claves foráneas) son *cómo se vuelven a conectar* esas tablas.

---

## 8. Casos especiales

- **Datos que "parecen" repetidos pero no lo son**: el precio histórico del ejemplo anterior. Si el dato representa algo distinto (lo que pasó en un momento concreto), guardarlo es correcto.
- **Campos calculados** (por ejemplo, el `total` de un pedido): se pueden calcular con `SUM` en cada consulta. Guardarlos rompe la normalización, pero a veces se hace por rendimiento (desnormalización controlada).
- **Datos en formato JSON dentro de una columna**: los sistemas modernos (PostgreSQL, MySQL) lo permiten y puede ser útil para datos sin estructura fija, pero **no cumple 1FN** en sentido estricto. Úsalo con criterio, no como atajo para no diseñar tablas.
- **Tablas de catálogo o "lookup"**: para valores que se repiten mucho (estados como "pendiente", "enviado", "entregado") se crea una tabla pequeña y se apunta a ella con FK, o bien se usa un `CHECK`/`ENUM` si la lista es muy corta y estable.
- **Direcciones**: se suele normalizar en una tabla propia (`direcciones`), sobre todo si un cliente puede tener varias (casa, trabajo...), lo que además es una relación 1:N.
- **4FN y 5FN**: tratan problemas de dependencias múltiples muy concretos. Se estudian en teoría, pero rara vez se aplican en proyectos reales.
- **Normalizar una base de datos ya en uso**: es posible, pero exige migrar los datos con cuidado (crear tablas nuevas, copiar, comprobar, y solo entonces borrar las columnas antiguas). Haz copia de seguridad antes.

---

## 9. Resumen

| Concepto | Idea clave |
|---|---|
| Normalización | Organizar las tablas para que cada dato se guarde una sola vez |
| Objetivo | Evitar repetición y las anomalías de inserción, actualización y borrado |
| Dependencia | B depende de A si conocer A te dice cuál es B |
| **1FN** | Un solo valor por celda, sin grupos repetidos |
| **2FN** | Todo depende de **toda** la clave (importa con claves compuestas) |
| **3FN** | Todo depende **solo** de la clave (sin dependencias transitivas) |
| **FNBC** | 3FN más estricta: todo determinante es clave |
| Desnormalización | Duplicar datos a propósito para ganar velocidad de lectura |
| Cómo se hace | Dividiendo tablas y uniéndolas con claves foráneas |

> [!tip] Para recordar en una frase
> **Cada columna depende de la clave, de toda la clave y de nada más que la clave.**

Siguiente nota: [[SQL 17 - Transacciones]]