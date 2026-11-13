# 04 - Snippets de PHP

> [!info] ¿Qué es?
> Fragmentos de **PHP** listos para copiar: formularios seguros, conexión a base de datos con PDO, sesiones, JSON y subida de archivos. Las explicaciones están en la carpeta [[PHP 00 - Índice|PHP]].

---

## 1. Formularios

### 1.1 Recoger y validar un formulario

```php
<?php
$errores = [];
$nombre = '';
$email = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $nombre = trim($_POST['nombre'] ?? '');
    $email  = trim($_POST['email'] ?? '');

    if ($nombre === '') {
        $errores['nombre'] = 'El nombre es obligatorio';
    }
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errores['email'] = 'El email no es válido';
    }

    if (!$errores) {
        // guardar, enviar…
        header('Location: gracias.php');   // patrón PRG: evita reenviar al recargar
        exit;
    }
}
?>
```

### 1.2 Mostrar datos del usuario de forma segura

```php
<?php
function e(?string $texto): string {
    return htmlspecialchars($texto ?? '', ENT_QUOTES, 'UTF-8');
}
?>
<input name="nombre" value="<?= e($nombre) ?>">
<?php if (isset($errores['nombre'])): ?>
  <p class="error"><?= e($errores['nombre']) ?></p>
<?php endif; ?>
```

### 1.3 Token CSRF

```php
<?php
session_start();
$_SESSION['csrf'] ??= bin2hex(random_bytes(32));
?>
<input type="hidden" name="csrf" value="<?= $_SESSION['csrf'] ?>">

<?php
// Al procesar el formulario:
if (!hash_equals($_SESSION['csrf'], $_POST['csrf'] ?? '')) {
    http_response_code(403);
    exit('Petición no válida');
}
```

Más en [[PHP 10 - Formularios|Formularios]] y [[PHP 18 - Seguridad|Seguridad]].

---

## 2. Base de datos con PDO

### 2.1 Conexión

```php
<?php
function conectar(): PDO {
    $dsn = 'mysql:host=localhost;dbname=mi_db;charset=utf8mb4';
    return new PDO($dsn, getenv('DB_USER'), getenv('DB_PASS'), [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ]);
}

$pdo = conectar();
```

### 2.2 Consultas preparadas (siempre)

```php
// Varios resultados
$stmt = $pdo->prepare('SELECT * FROM productos WHERE precio < :max ORDER BY nombre');
$stmt->execute(['max' => 50]);
$productos = $stmt->fetchAll();

// Uno solo
$stmt = $pdo->prepare('SELECT * FROM usuarios WHERE id = ?');
$stmt->execute([$id]);
$usuario = $stmt->fetch();   // false si no existe
```

### 2.3 Insertar y obtener el id

```php
$stmt = $pdo->prepare('INSERT INTO productos (nombre, precio) VALUES (:nombre, :precio)');
$stmt->execute(['nombre' => $nombre, 'precio' => $precio]);
$nuevoId = (int) $pdo->lastInsertId();
```

### 2.4 Transacción

```php
try {
    $pdo->beginTransaction();
    $pdo->prepare('UPDATE cuentas SET saldo = saldo - ? WHERE id = ?')->execute([$cantidad, $origen]);
    $pdo->prepare('UPDATE cuentas SET saldo = saldo + ? WHERE id = ?')->execute([$cantidad, $destino]);
    $pdo->commit();
} catch (Throwable $e) {
    $pdo->rollBack();
    throw $e;
}
```

### 2.5 Paginación

```php
$porPagina = 20;
$pagina = max(1, (int) ($_GET['pagina'] ?? 1));
$offset = ($pagina - 1) * $porPagina;

$stmt = $pdo->prepare('SELECT * FROM productos ORDER BY id LIMIT :limite OFFSET :offset');
$stmt->bindValue('limite', $porPagina, PDO::PARAM_INT);
$stmt->bindValue('offset', $offset, PDO::PARAM_INT);
$stmt->execute();
```

Más en [[PHP 16 - PDO y bases de datos|PDO y bases de datos]].

---

## 3. Usuarios y sesiones

### 3.1 Registrar con contraseña cifrada

```php
$hash = password_hash($password, PASSWORD_DEFAULT);
$pdo->prepare('INSERT INTO usuarios (email, password) VALUES (?, ?)')->execute([$email, $hash]);
```

### 3.2 Iniciar sesión

```php
session_start();

$stmt = $pdo->prepare('SELECT id, password FROM usuarios WHERE email = ?');
$stmt->execute([$email]);
$usuario = $stmt->fetch();

if ($usuario && password_verify($password, $usuario['password'])) {
    session_regenerate_id(true);
    $_SESSION['usuario_id'] = $usuario['id'];
    header('Location: panel.php');
    exit;
}
$error = 'Email o contraseña incorrectos';
```

### 3.3 Proteger una página

```php
session_start();
if (empty($_SESSION['usuario_id'])) {
    header('Location: login.php');
    exit;
}
```

### 3.4 Cerrar sesión

```php
session_start();
$_SESSION = [];
session_destroy();
header('Location: login.php');
exit;
```

Más en [[PHP 11 - Sesiones y cookies|Sesiones y cookies]].

---

## 4. JSON y APIs

### 4.1 Responder JSON

```php
function responderJson(mixed $datos, int $codigo = 200): never {
    http_response_code($codigo);
    header('Content-Type: application/json; charset=utf-8');
    echo json_encode($datos, JSON_UNESCAPED_UNICODE);
    exit;
}

responderJson(['ok' => true, 'productos' => $productos]);
```

### 4.2 Leer un cuerpo JSON recibido

```php
$datos = json_decode(file_get_contents('php://input'), true);
if (!is_array($datos)) {
    responderJson(['error' => 'JSON no válido'], 400);
}
```

### 4.3 Llamar a una API externa

```php
$respuesta = file_get_contents('https://api.ejemplo.com/datos');
$datos = json_decode($respuesta, true, flags: JSON_THROW_ON_ERROR);
```

Más en [[PHP 13 - JSON y APIs|JSON y APIs]].

---

## 5. Archivos

### 5.1 Subir una imagen de forma segura

```php
$archivo = $_FILES['imagen'] ?? null;

if (!$archivo || $archivo['error'] !== UPLOAD_ERR_OK) {
    exit('Error al subir el archivo');
}
if ($archivo['size'] > 2 * 1024 * 1024) {
    exit('Máximo 2 MB');
}

$permitidos = ['image/jpeg' => 'jpg', 'image/png' => 'png', 'image/webp' => 'webp'];
$tipo = (new finfo(FILEINFO_MIME_TYPE))->file($archivo['tmp_name']);
if (!isset($permitidos[$tipo])) {
    exit('Formato no permitido');
}

$nombre = bin2hex(random_bytes(16)) . '.' . $permitidos[$tipo];
move_uploaded_file($archivo['tmp_name'], __DIR__ . '/uploads/' . $nombre);
```

```html
<form method="post" enctype="multipart/form-data">
  <input type="file" name="imagen" accept="image/*">
  <button>Subir</button>
</form>
```

### 5.2 Leer y escribir un archivo

```php
$contenido = file_get_contents(__DIR__ . '/datos.txt');
file_put_contents(__DIR__ . '/log.txt', date('c') . " Evento\n", FILE_APPEND);
```

### 5.3 Leer un CSV

```php
$filas = array_map('str_getcsv', file(__DIR__ . '/datos.csv', FILE_IGNORE_NEW_LINES));
$cabecera = array_shift($filas);
$registros = array_map(fn($f) => array_combine($cabecera, $f), $filas);
```

Más en [[PHP 12 - Ficheros|Ficheros]].

---

## 6. Utilidades

### 6.1 Depurar una variable

```php
echo '<pre>'; var_dump($variable); echo '</pre>';
```

### 6.2 Mostrar errores en desarrollo

```php
ini_set('display_errors', '1');
error_reporting(E_ALL);
```

### 6.3 Fecha en español

```php
$fmt = new IntlDateFormatter('es_ES', IntlDateFormatter::LONG, IntlDateFormatter::NONE);
echo $fmt->format(new DateTime());   // 13 de noviembre de 2026
```

### 6.4 Precio en euros

```php
echo number_format(1234.5, 2, ',', '.') . ' €';   // 1.234,50 €
```

### 6.5 Crear un slug para URLs

```php
function slug(string $texto): string {
    $texto = iconv('UTF-8', 'ASCII//TRANSLIT', $texto);
    $texto = preg_replace('/[^a-zA-Z0-9]+/', '-', $texto);
    return strtolower(trim($texto, '-'));
}

slug('¡Hola, Mundo Ñandú!'); // hola-mundo-nandu
```

### 6.6 Redirigir

```php
header('Location: /otra-pagina.php');
exit;
```
