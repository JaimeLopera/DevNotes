# 03 - Errores de PHP

> [!info] ¿Qué es?
> Los errores de PHP más frecuentes, con el **mensaje literal**, qué significa, sus causas y cómo solucionarlo. PHP indica el **archivo y la línea** (`in /ruta/archivo.php on line 12`): empieza siempre por ahí.

---

## 1. Antes de empezar: ver los errores

Si ves una **página en blanco** o un error 500 sin detalles, activa los errores en desarrollo:

```php
ini_set('display_errors', '1');
error_reporting(E_ALL);
```

En producción, **nunca** los muestres en pantalla: guárdalos en un log (`log_errors = On` en `php.ini`).

| Nivel | Significa |
|---|---|
| `Warning` | Algo va mal, pero el script continúa |
| `Notice` / `Deprecated` | Aviso; conviene corregirlo |
| `Fatal error` | El script se detiene |
| `Parse error` | Error de sintaxis; no se ejecuta nada |

---

## 2. Variables y arrays

### 2.1 `Warning: Undefined variable $x`

**Causas típicas:** la variable se usa antes de asignarla, está mal escrita, o se definió dentro de una función o de un `if` que no se ejecutó.

**Solución:** inicialízala antes (`$errores = [];`) o usa un valor por defecto: `$x ?? ''`.

### 2.2 `Warning: Undefined array key "nombre"`

**Qué significa:** lees una clave que no existe, muy habitual con `$_POST`, `$_GET` o `$_SESSION`.

**Solución:**

```php
$nombre = $_POST['nombre'] ?? '';
if (isset($_GET['id'])) { /* ... */ }
```

### 2.3 `Fatal error: Uncaught TypeError: ... must be of type array, null given`

**Qué significa:** una función esperaba un array y recibió `null` (normalmente un dato que no existe).

**Solución:** comprueba el valor antes o da uno por defecto (`$lista ?? []`).

### 2.4 `Warning: Array to string conversion`

**Qué significa:** haces `echo` de un array.

**Solución:** muestra un elemento, usa `implode(', ', $array)` o `print_r($array)` para depurar.

---

## 3. Sintaxis

### 3.1 `Parse error: syntax error, unexpected ...`

**Causas típicas:**
- Falta un `;` al final de la línea **anterior**.
- Llaves `{}` o paréntesis sin cerrar.
- Comillas mal cerradas.
- `$` olvidado en una variable.

**Solución:** revisa la línea indicada **y la anterior**.

### 3.2 `Parse error: syntax error, unexpected end of file`

**Qué significa:** falta cerrar una llave `}` o un bloque `endif;` / `endforeach;`.

---

## 4. Cabeceras y salida

### 4.1 `Warning: Cannot modify header information - headers already sent by (output started at archivo.php:3)`

**Qué significa:** usas `header()`, `setcookie()` o `session_start()` **después** de haber enviado contenido al navegador.

**Causas típicas:**
- Un `echo`, HTML o **un espacio/salto de línea antes de `<?php`**.
- Un salto de línea después de `?>` en un archivo incluido.
- Un BOM (marca invisible) al principio del archivo.

**Solución:**
- Haz toda la lógica (redirecciones, sesiones) **antes** de cualquier salida.
- En archivos solo de PHP, **no cierres** con `?>`.
- Guarda los archivos en "UTF-8 sin BOM".

### 4.2 La redirección no funciona y el código sigue ejecutándose

**Solución:** pon `exit;` siempre después de `header('Location: ...')`.

---

## 5. Base de datos (PDO)

### 5.1 `SQLSTATE[HY000] [1045] Access denied for user`

**Qué significa:** usuario o contraseña de la base de datos incorrectos.

**Solución:** revisa las credenciales (y tus variables de entorno). En XAMPP, por defecto: usuario `root` sin contraseña.

### 5.2 `SQLSTATE[HY000] [2002] Connection refused` / `No such file or directory`

**Qué significa:** no se puede conectar al servidor de base de datos.

**Causas típicas:** MySQL no está arrancado, el host o el puerto son incorrectos, o en Docker usas `localhost` en lugar del nombre del servicio (`db`).

### 5.3 `SQLSTATE[HY000] [1049] Unknown database`

**Solución:** crea la base de datos o corrige su nombre en el DSN.

### 5.4 `SQLSTATE[42S02]: Base table or view not found`

**Solución:** la tabla no existe o está mal escrita (en Linux, los nombres de tabla pueden distinguir mayúsculas).

### 5.5 `SQLSTATE[HY093]: Invalid parameter number`

**Qué significa:** el número de parámetros de `execute()` no coincide con los marcadores (`?` o `:nombre`) de la consulta.

### 5.6 `Call to a member function fetch() on bool`

**Qué significa:** la consulta falló y devolvió `false`, pero no se lanzó una excepción.

**Solución:** configura PDO para lanzar excepciones: `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`.

Errores del propio SQL en [[Errores 05 - SQL]]. Más en [[PHP 16 - PDO y bases de datos|PDO y bases de datos]].

---

## 6. Archivos, clases y funciones

### 6.1 `Warning: include(archivo.php): Failed to open stream: No such file or directory`

**Qué significa:** la ruta del `include`/`require` no es correcta. Las rutas relativas dependen del script que se ejecuta, no del archivo que incluye.

**Solución:** usa rutas absolutas basadas en el archivo actual:

```php
require __DIR__ . '/config/db.php';
```

### 6.2 `Fatal error: Uncaught Error: Call to undefined function x()`

**Causas típicas:** nombre mal escrito, falta el `require` del archivo donde se define, o es una función de una extensión no instalada (por ejemplo `mb_strlen` sin `mbstring`).

### 6.3 `Fatal error: Uncaught Error: Class "X" not found`

**Causas típicas:** falta el `require` o el `use`, el *namespace* no coincide, o falta `composer dump-autoload` / `require 'vendor/autoload.php'`.

### 6.4 `Fatal error: Cannot redeclare function x()`

**Solución:** usa `require_once` en lugar de `require` para no cargar el archivo dos veces.

---

## 7. Formularios, sesiones y subidas

### 7.1 `$_POST` está vacío

**Causas típicas:** el formulario usa `method="get"`, los campos no tienen `name`, o los datos se envían como JSON (léelos con `file_get_contents('php://input')`).

### 7.2 La sesión no guarda nada

**Causas típicas:** falta `session_start()` al principio de **cada** página, o se llama después de enviar salida (ver 4.1).

### 7.3 `$_FILES` vacío o error al subir

**Causas típicas:**
- Falta `enctype="multipart/form-data"` en el `<form>`.
- El archivo supera `upload_max_filesize` o `post_max_size` de `php.ini`.
- La carpeta de destino no existe o no tiene permisos de escritura.

### 7.4 Caracteres raros (Ã±) desde la base de datos

**Solución:** `charset=utf8mb4` en el DSN de PDO y las tablas en `utf8mb4`.

---

## 8. Otros

### 8.1 `Fatal error: Allowed memory size of 134217728 bytes exhausted`

**Causas típicas:** bucle infinito, cargar demasiados registros en memoria, o recursión sin fin.

**Solución:** pagina los resultados o procesa por lotes; aumentar `memory_limit` solo si de verdad hace falta.

### 8.2 `Fatal error: Maximum execution time of 30 seconds exceeded`

**Solución:** busca el bucle lento o infinito; para tareas largas usa un script de terminal o una cola.

### 8.3 Página en blanco (error 500 sin mensaje)

**Solución:** activa `display_errors` (sección 1) o mira el log de errores del servidor (Apache: `error.log`).

---

## 9. Resumen

- Activa `display_errors` en desarrollo; nunca en producción.
- `Undefined variable/array key` → usa `??` e `isset()`.
- `Parse error` → revisa la línea indicada y la anterior (falta `;` o `}`).
- `headers already sent` → no envíes nada antes de `header()`/`session_start()`; no cierres con `?>`.
- `Access denied` / `Connection refused` → credenciales, servidor MySQL o host incorrecto.
- `Failed to open stream` → usa `__DIR__` en los `require`.
- `$_FILES` vacío → `enctype="multipart/form-data"` y límites de `php.ini`.
