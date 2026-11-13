# 06 - Snippets de SQL

> [!info] ¿Qué es?
> Consultas **SQL** listas para copiar y adaptar: búsquedas, paginación, agregados, duplicados, fechas y estructura de tablas. Están escritas para **MySQL/MariaDB**; las diferencias con PostgreSQL se indican cuando las hay. Las explicaciones están en la carpeta [[SQL 00 - Índice|SQL]].

---

## 1. Consultas básicas

### 1.1 Buscar por texto

```sql
SELECT * FROM productos
WHERE nombre LIKE '%taza%';
```

(En PostgreSQL, `ILIKE` ignora mayúsculas.)

### 1.2 Varios valores posibles

```sql
SELECT * FROM pedidos
WHERE estado IN ('pendiente', 'enviado');
```

### 1.3 Rango de valores

```sql
SELECT * FROM productos
WHERE precio BETWEEN 10 AND 50;
```

### 1.4 Valores vacíos

```sql
SELECT * FROM usuarios WHERE telefono IS NULL;
SELECT * FROM usuarios WHERE telefono IS NOT NULL;
```

### 1.5 Valor por defecto si es NULL

```sql
SELECT nombre, COALESCE(telefono, 'Sin teléfono') AS telefono
FROM usuarios;
```

Más en [[SQL 03 - WHERE|WHERE]].

---

## 2. Ordenar y paginar

### 2.1 Los últimos N registros

```sql
SELECT * FROM pedidos
ORDER BY creado_en DESC
LIMIT 10;
```

### 2.2 Paginación

```sql
-- Página 3, 20 por página → OFFSET = (3 - 1) * 20
SELECT * FROM productos
ORDER BY id
LIMIT 20 OFFSET 40;
```

### 2.3 Paginación rápida en tablas grandes (por cursor)

```sql
SELECT * FROM productos
WHERE id > 1200        -- último id de la página anterior
ORDER BY id
LIMIT 20;
```

### 2.4 Total de resultados para la paginación

```sql
SELECT COUNT(*) AS total FROM productos WHERE categoria_id = 3;
```

Más en [[SQL 04 - ORDER BY y LIMIT|ORDER BY y LIMIT]].

---

## 3. Agregados e informes

### 3.1 Totales por grupo

```sql
SELECT categoria_id, COUNT(*) AS productos, AVG(precio) AS precio_medio
FROM productos
GROUP BY categoria_id
ORDER BY productos DESC;
```

### 3.2 Filtrar grupos

```sql
SELECT cliente_id, SUM(total) AS gastado
FROM pedidos
GROUP BY cliente_id
HAVING SUM(total) > 500;
```

### 3.3 Ventas por mes

```sql
-- MySQL
SELECT DATE_FORMAT(creado_en, '%Y-%m') AS mes, SUM(total) AS ventas
FROM pedidos
GROUP BY mes
ORDER BY mes;

-- PostgreSQL
SELECT TO_CHAR(creado_en, 'YYYY-MM') AS mes, SUM(total) AS ventas
FROM pedidos
GROUP BY mes
ORDER BY mes;
```

### 3.4 Top 5 productos más vendidos

```sql
SELECT p.nombre, SUM(lp.cantidad) AS unidades
FROM lineas_pedido lp
JOIN productos p ON p.id = lp.producto_id
GROUP BY p.id, p.nombre
ORDER BY unidades DESC
LIMIT 5;
```

### 3.5 Contar con condiciones en una sola consulta

```sql
SELECT
  COUNT(*) AS total,
  SUM(CASE WHEN estado = 'enviado' THEN 1 ELSE 0 END) AS enviados,
  SUM(CASE WHEN estado = 'pendiente' THEN 1 ELSE 0 END) AS pendientes
FROM pedidos;
```

Más en [[SQL 06 - GROUP BY y HAVING|GROUP BY y HAVING]].

---

## 4. Relaciones (JOIN)

### 4.1 Datos de dos tablas

```sql
SELECT p.id, p.total, c.nombre AS cliente
FROM pedidos p
JOIN clientes c ON c.id = p.cliente_id;
```

### 4.2 Registros sin relación (clientes sin pedidos)

```sql
SELECT c.*
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id
WHERE p.id IS NULL;
```

### 4.3 Contar hijos de cada registro

```sql
SELECT c.nombre, COUNT(p.id) AS num_pedidos
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id
GROUP BY c.id, c.nombre;
```

### 4.4 El último pedido de cada cliente

```sql
SELECT *
FROM (
  SELECT p.*, ROW_NUMBER() OVER (PARTITION BY cliente_id ORDER BY creado_en DESC) AS n
  FROM pedidos p
) t
WHERE n = 1;
```

Más en [[SQL 07 - JOIN|JOIN]] y [[SQL 08 - Subconsultas|Subconsultas]].

---

## 5. Duplicados

### 5.1 Encontrar duplicados

```sql
SELECT email, COUNT(*) AS veces
FROM usuarios
GROUP BY email
HAVING COUNT(*) > 1;
```

### 5.2 Borrar duplicados dejando el de menor id (MySQL)

```sql
DELETE u1 FROM usuarios u1
JOIN usuarios u2 ON u1.email = u2.email AND u1.id > u2.id;
```

### 5.3 Insertar o actualizar si ya existe (*upsert*)

```sql
-- MySQL
INSERT INTO stock (producto_id, cantidad) VALUES (5, 10)
ON DUPLICATE KEY UPDATE cantidad = cantidad + VALUES(cantidad);

-- PostgreSQL
INSERT INTO stock (producto_id, cantidad) VALUES (5, 10)
ON CONFLICT (producto_id) DO UPDATE SET cantidad = stock.cantidad + EXCLUDED.cantidad;
```

---

## 6. Fechas

### 6.1 Registros de hoy

```sql
SELECT * FROM pedidos WHERE DATE(creado_en) = CURRENT_DATE;
```

### 6.2 Últimos 7 días

```sql
-- MySQL
SELECT * FROM pedidos WHERE creado_en >= NOW() - INTERVAL 7 DAY;

-- PostgreSQL
SELECT * FROM pedidos WHERE creado_en >= NOW() - INTERVAL '7 days';
```

### 6.3 Mes actual (aprovechando índices)

```sql
SELECT * FROM pedidos
WHERE creado_en >= DATE_FORMAT(CURRENT_DATE, '%Y-%m-01')
  AND creado_en <  DATE_FORMAT(CURRENT_DATE + INTERVAL 1 MONTH, '%Y-%m-01');
```

### 6.4 Edad a partir de la fecha de nacimiento (MySQL)

```sql
SELECT nombre, TIMESTAMPDIFF(YEAR, nacimiento, CURRENT_DATE) AS edad FROM usuarios;
```

Más en [[SQL 05 - Funciones|Funciones SQL]].

---

## 7. Estructura de tablas

### 7.1 Tabla típica

```sql
CREATE TABLE productos (
  id INT AUTO_INCREMENT PRIMARY KEY,           -- PostgreSQL: id SERIAL PRIMARY KEY
  nombre VARCHAR(150) NOT NULL,
  precio DECIMAL(10, 2) NOT NULL CHECK (precio >= 0),
  stock INT NOT NULL DEFAULT 0,
  categoria_id INT,
  creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (categoria_id) REFERENCES categorias(id) ON DELETE SET NULL
);
```

### 7.2 Tabla intermedia (muchos a muchos)

```sql
CREATE TABLE producto_etiqueta (
  producto_id INT NOT NULL,
  etiqueta_id INT NOT NULL,
  PRIMARY KEY (producto_id, etiqueta_id),
  FOREIGN KEY (producto_id) REFERENCES productos(id) ON DELETE CASCADE,
  FOREIGN KEY (etiqueta_id) REFERENCES etiquetas(id) ON DELETE CASCADE
);
```

### 7.3 Añadir una columna y un índice

```sql
ALTER TABLE usuarios ADD COLUMN telefono VARCHAR(20) NULL;
CREATE INDEX idx_pedidos_cliente ON pedidos (cliente_id);
CREATE UNIQUE INDEX idx_usuarios_email ON usuarios (email);
```

### 7.4 Ver la estructura de una tabla

```sql
DESCRIBE productos;          -- MySQL
-- PostgreSQL (psql): \d productos
```

Más en [[SQL 12 - CREATE y ALTER|CREATE y ALTER]] y [[SQL 14 - Índices|Índices]].

---

## 8. Seguridad al modificar

### 8.1 Comprobar antes de borrar o actualizar

```sql
-- 1. Mira qué se va a ver afectado
SELECT * FROM usuarios WHERE ultimo_acceso < '2024-01-01';

-- 2. Solo entonces, con el MISMO WHERE
DELETE FROM usuarios WHERE ultimo_acceso < '2024-01-01';
```

### 8.2 Probar dentro de una transacción

```sql
START TRANSACTION;
UPDATE productos SET precio = precio * 1.10 WHERE categoria_id = 2;
SELECT * FROM productos WHERE categoria_id = 2;   -- revisa el resultado
ROLLBACK;   -- o COMMIT si está bien
```

Más en [[SQL 17 - Transacciones|Transacciones]].
