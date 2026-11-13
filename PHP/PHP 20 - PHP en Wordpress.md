# 20 - PHP en WordPress

> [!info] ¿Qué es?
> WordPress está hecho en PHP. Todo lo que se puede personalizar (temas, plugins, formularios, APIs, bases de datos) se hace con PHP y con las funciones que WordPress ya trae listas. Esta nota reúne **todo el PHP específico de WordPress**: cómo está organizado, cómo se engancha tu código, cómo se muestran los contenidos, cómo se crean temas y plugins y cómo se hace todo de forma segura.

---

## 1. Antes de empezar

Conviene conocer:

- PHP básico: variables, funciones, arrays y POO → [[PHP 14 - POO]]
- Cómo funcionan las bases de datos con PHP → [[PHP 16 - PDO y bases de datos]]
- Seguridad en PHP (validar, escapar) → [[PHP 18 - Seguridad]]

> [!tip] Idea clave
> En WordPress **casi nunca modificas el núcleo**. No tocas los archivos de WordPress: añades tu código en un **tema** o en un **plugin** y lo "enganchas" al sistema con **hooks**.

---

## 2. Concepto fundamental

### 2.1 Estructura de carpetas

```
wordpress/
├── wp-admin/          ← panel de administración (no tocar)
├── wp-includes/       ← núcleo de WordPress (no tocar)
├── wp-content/
│   ├── themes/        ← temas (aquí trabajas)
│   ├── plugins/       ← plugins (aquí trabajas)
│   ├── mu-plugins/    ← plugins que siempre están activos
│   └── uploads/       ← imágenes y archivos subidos
├── wp-config.php      ← configuración (base de datos, claves, debug)
└── index.php
```

### 2.2 Cómo se carga WordPress (de forma simplificada)

1. Se carga `wp-config.php` (datos de la base de datos y ajustes).
2. Se cargan el núcleo y los **plugins**.
3. Se carga el **tema** (`functions.php`).
4. WordPress averigua qué se está pidiendo (una entrada, una página, una categoría…).
5. Elige el **archivo de plantilla** adecuado y lo muestra.

En cada paso WordPress lanza **hooks** (avisos) donde tú puedes meter tu código.

### 2.3 Hooks: la base de todo

Un **hook** es un punto del recorrido de WordPress donde puedes enganchar tu propia función. Hay dos tipos:

| Tipo | Para qué sirve | Funciones |
|---|---|---|
| **Action** | Ejecutar código en un momento concreto | `add_action()`, `do_action()` |
| **Filter** | Modificar un valor antes de que se use o se muestre | `add_filter()`, `apply_filters()` |

La diferencia práctica: una **action** *hace* algo; un **filter** *recibe un valor, lo cambia y lo devuelve* (siempre con `return`).

---

## 3. Sintaxis / estructura

### 3.1 Actions y filters

```php
// ACTION: ejecutar algo en un momento concreto
add_action( 'nombre_del_hook', 'mi_funcion', 10, 1 );
//           ↑ hook             ↑ tu función  ↑ prioridad  ↑ nº de argumentos

// FILTER: modificar un valor
add_filter( 'nombre_del_hook', 'mi_filtro', 10, 1 );
function mi_filtro( $valor ) {
    return $valor; // SIEMPRE devolver el valor
}
```

- **Prioridad** (por defecto `10`): cuanto más bajo, antes se ejecuta.
- **Argumentos**: cuántos valores del hook recibe tu función (por defecto `1`).
- Para quitar un hook: `remove_action()` / `remove_filter()` (mismos parámetros con los que se añadió).
- Para crear tus propios hooks: `do_action( 'mi_hook' )` y `apply_filters( 'mi_filtro', $valor )`.

### 3.2 Hooks más usados

| Hook | Tipo | Cuándo ocurre / para qué se usa |
|---|---|---|
| `plugins_loaded` | action | Todos los plugins ya están cargados |
| `after_setup_theme` | action | Configurar el tema (soportes, menús, tamaños) |
| `init` | action | WordPress listo: registrar tipos de contenido, taxonomías |
| `wp_enqueue_scripts` | action | Añadir CSS y JS en la web pública |
| `admin_enqueue_scripts` | action | Añadir CSS y JS en el panel de administración |
| `admin_menu` | action | Crear páginas en el menú del administrador |
| `admin_init` | action | Registrar ajustes del administrador |
| `save_post` | action | Cuando se guarda una entrada |
| `rest_api_init` | action | Registrar rutas de la API REST |
| `wp_head` / `wp_footer` | action | Meter cosas en `<head>` o antes de `</body>` |
| `pre_get_posts` | action | Modificar una consulta antes de ejecutarse |
| `the_content` | filter | Modificar el contenido de una entrada al mostrarla |
| `the_title` | filter | Modificar el título al mostrarlo |

### 3.3 Plantilla para el archivo principal de un plugin

```php
<?php
/**
 * Plugin Name: Mi Plugin
 * Description: Qué hace el plugin.
 * Version: 1.0.0
 * Author: Jaime
 * Text Domain: mi-plugin
 */

// Evita que se ejecute el archivo directamente desde el navegador
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}
```

> [!warning] Importante
> La comprobación `ABSPATH` va **al principio de todos los archivos PHP** de un tema o plugin. Impide que alguien abra el archivo directamente sin pasar por WordPress.

---

## 4. Elementos / propiedades / características

### 4.1 Temas

**Archivos mínimos** de un tema clásico (carpeta dentro de `wp-content/themes/`):

- `style.css` → la cabecera con los datos del tema (nombre, autor, versión) y los estilos.
- `index.php` → plantilla de respaldo, la que se usa si no hay otra más concreta.
- `functions.php` → aquí va tu PHP: soportes, menús, scripts, etc.

**Jerarquía de plantillas** (WordPress busca de lo más específico a lo más general):

| Qué se muestra | Archivos que busca (en orden) |
|---|---|
| Página de inicio | `front-page.php` → `home.php` → `index.php` |
| Una entrada | `single-{tipo}.php` → `single.php` → `singular.php` → `index.php` |
| Una página | `page-{slug}.php` → `page.php` → `singular.php` → `index.php` |
| Categoría / archivo | `category.php` → `archive.php` → `index.php` |
| Búsqueda | `search.php` → `index.php` |
| Error 404 | `404.php` → `index.php` |

**Partes reutilizables:**

- `get_header()` → carga `header.php`
- `get_footer()` → carga `footer.php`
- `get_sidebar()` → carga `sidebar.php`
- `get_template_part( 'partes/tarjeta' )` → carga `partes/tarjeta.php`

**Configuración típica en `functions.php`:**

```php
function mitema_setup() {
    add_theme_support( 'title-tag' );          // WordPress gestiona el <title>
    add_theme_support( 'post-thumbnails' );    // imágenes destacadas
    add_theme_support( 'html5', array( 'search-form', 'gallery', 'caption' ) );

    register_nav_menus( array(
        'principal' => __( 'Menú principal', 'mitema' ),
        'pie'       => __( 'Menú del pie', 'mitema' ),
    ) );
}
add_action( 'after_setup_theme', 'mitema_setup' );
```

Mostrar un menú en una plantilla:

```php
wp_nav_menu( array( 'theme_location' => 'principal' ) );
```

**Tema hijo (child theme):** un tema que hereda de otro (el "padre"). Sirve para personalizar sin perder los cambios cuando el padre se actualiza. Solo necesita su propio `style.css` (con `Template: nombre-del-padre`) y un `functions.php`.

### 4.2 El Loop y las consultas

**El Loop** es el bucle que recorre las entradas que WordPress ha cargado para la página actual:

```php
<?php if ( have_posts() ) : ?>
    <?php while ( have_posts() ) : the_post(); ?>
        <article>
            <h2><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h2>
            <?php the_post_thumbnail( 'medium' ); ?>
            <?php the_excerpt(); ?>
        </article>
    <?php endwhile; ?>
<?php else : ?>
    <p>No hay contenido.</p>
<?php endif; ?>
```

**Funciones habituales dentro del Loop** (todas "imprimen" directamente):

| Función | Qué muestra |
|---|---|
| `the_title()` | Título |
| `the_content()` | Contenido completo |
| `the_excerpt()` | Resumen |
| `the_permalink()` | URL de la entrada |
| `the_post_thumbnail()` | Imagen destacada |
| `the_author()` | Autor |
| `the_date()` / `get_the_date()` | Fecha |
| `the_category()` / `the_tags()` | Categorías / etiquetas |

> [!tip] Regla `the_` y `get_`
> Las funciones `the_...()` **imprimen** el valor. Las `get_the_...()` lo **devuelven** para que lo guardes en una variable o lo modifiques.

**Consultas propias con `WP_Query`:**

```php
$consulta = new WP_Query( array(
    'post_type'      => 'post',
    'posts_per_page' => 5,
    'category_name'  => 'noticias',
    'orderby'        => 'date',
    'order'          => 'DESC',
) );

if ( $consulta->have_posts() ) {
    while ( $consulta->have_posts() ) {
        $consulta->the_post();
        the_title( '<h3>', '</h3>' );
    }
    wp_reset_postdata(); // IMPRESCINDIBLE al terminar
}
```

- Parámetros comunes: `post_type`, `posts_per_page`, `category_name`, `tag`, `author`, `meta_key`, `meta_query`, `tax_query`, `s` (búsqueda), `paged` (paginación).
- `get_posts()` → versión más simple que devuelve un array de entradas.
- Para modificar la **consulta principal** de la página, no crees otra: usa el hook `pre_get_posts`.

```php
function ajustar_consulta( $query ) {
    if ( ! is_admin() && $query->is_main_query() && $query->is_home() ) {
        $query->set( 'posts_per_page', 6 );
    }
}
add_action( 'pre_get_posts', 'ajustar_consulta' );
```

**Condicionales de WordPress** (devuelven `true` o `false`): `is_home()`, `is_front_page()`, `is_single()`, `is_page()`, `is_archive()`, `is_category()`, `is_search()`, `is_404()`, `is_admin()`, `is_user_logged_in()`.

### 4.3 Scripts y estilos (enqueue)

En WordPress **no se escribe `<link>` ni `<script>` a mano**: se "encolan" desde PHP para que WordPress los ordene, evite duplicados y gestione dependencias.

```php
function mitema_assets() {
    wp_enqueue_style(
        'mitema-estilos',
        get_stylesheet_uri(),            // style.css del tema
        array(),                         // dependencias
        wp_get_theme()->get( 'Version' ) // versión (para la caché)
    );

    wp_enqueue_script(
        'mitema-main',
        get_template_directory_uri() . '/js/main.js',
        array(),                         // dependencias (p. ej. 'jquery')
        '1.0.0',
        true                             // true = cargar en el footer
    );
}
add_action( 'wp_enqueue_scripts', 'mitema_assets' );
```

Pasar datos de PHP a JavaScript:

```php
wp_localize_script( 'mitema-main', 'miDatos', array(
    'ajaxUrl' => admin_url( 'admin-ajax.php' ),
    'nonce'   => wp_create_nonce( 'mi_accion' ),
) );
```

**Funciones para rutas y URLs:**

| Función | Devuelve |
|---|---|
| `get_template_directory_uri()` | URL del tema (padre) |
| `get_stylesheet_directory_uri()` | URL del tema activo (hijo si lo hay) |
| `get_template_directory()` | Ruta en disco del tema |
| `plugin_dir_path( __FILE__ )` | Ruta en disco del plugin |
| `plugin_dir_url( __FILE__ )` | URL del plugin |
| `home_url()` / `site_url()` | URL del sitio |
| `admin_url()` | URL del administrador |

### 4.4 Plugins

Un plugin es una carpeta (o un archivo) dentro de `wp-content/plugins/` con una cabecera de comentario (ver 3.3).

**Activación, desactivación y desinstalación:**

```php
register_activation_hook( __FILE__, 'mi_plugin_activar' );
register_deactivation_hook( __FILE__, 'mi_plugin_desactivar' );

function mi_plugin_activar() {
    // crear tablas, opciones por defecto...
    flush_rewrite_rules();
}
```

- Para limpiar todo al **borrar** el plugin, se crea un archivo `uninstall.php` en la raíz del plugin.

**Evitar choques de nombres:** como todos los plugins comparten el mismo espacio, se usa un **prefijo único** en funciones, constantes y opciones (`miplugin_...`), o bien clases y `namespace` (ver [[PHP 14 - POO]]).

### 4.5 Tipos de contenido y taxonomías propios

**Custom Post Type (CPT):** un tipo de contenido nuevo además de "entradas" y "páginas" (por ejemplo: proyectos, productos, eventos).

```php
function registrar_proyectos() {
    register_post_type( 'proyecto', array(
        'labels'       => array(
            'name'          => 'Proyectos',
            'singular_name' => 'Proyecto',
        ),
        'public'       => true,
        'has_archive'  => true,
        'menu_icon'    => 'dashicons-portfolio',
        'supports'     => array( 'title', 'editor', 'thumbnail', 'excerpt' ),
        'show_in_rest' => true, // necesario para el editor de bloques y la API REST
    ) );
}
add_action( 'init', 'registrar_proyectos' );
```

**Taxonomía propia** (como las categorías, pero a medida):

```php
function registrar_tecnologias() {
    register_taxonomy( 'tecnologia', 'proyecto', array(
        'label'        => 'Tecnologías',
        'hierarchical' => true,  // true = como categorías, false = como etiquetas
        'show_in_rest' => true,
    ) );
}
add_action( 'init', 'registrar_tecnologias' );
```

> [!warning] Reglas de enlaces
> Tras registrar un CPT o una taxonomía nuevos, hay que **refrescar los enlaces permanentes** (Ajustes → Enlaces permanentes → Guardar) o llamar a `flush_rewrite_rules()` **solo en la activación del plugin**, nunca en cada carga.

### 4.6 Guardar datos: metadatos, opciones y transients

| Qué | Para qué | Funciones |
|---|---|---|
| **Post meta** | Datos extra de **una entrada** (precio, fecha, enlace…) | `add_post_meta()`, `get_post_meta()`, `update_post_meta()`, `delete_post_meta()` |
| **User meta** | Datos extra de **un usuario** | `get_user_meta()`, `update_user_meta()` |
| **Options** | Ajustes **globales** del sitio o del plugin | `get_option()`, `update_option()`, `add_option()`, `delete_option()` |
| **Transients** | Datos **temporales** con caducidad (caché) | `set_transient()`, `get_transient()`, `delete_transient()` |

```php
update_post_meta( $post_id, 'precio', 49.90 );
$precio = get_post_meta( $post_id, 'precio', true ); // true = devolver un solo valor

update_option( 'mi_plugin_ajustes', array( 'color' => 'azul' ) );
$ajustes = get_option( 'mi_plugin_ajustes', array() ); // 2.º parámetro = valor por defecto

// Caché de una llamada lenta durante 1 hora
$datos = get_transient( 'mis_datos' );
if ( false === $datos ) {
    $datos = obtener_datos_lentos();
    set_transient( 'mis_datos', $datos, HOUR_IN_SECONDS );
}
```

### 4.7 Shortcodes

Un **shortcode** es una etiqueta entre corchetes (`[saludo nombre="Ana"]`) que se escribe en el editor y WordPress sustituye por el resultado de una función PHP.

```php
function shortcode_saludo( $atts ) {
    $atts = shortcode_atts( array(
        'nombre' => 'visitante',
    ), $atts, 'saludo' );

    return '<p>Hola, ' . esc_html( $atts['nombre'] ) . '</p>';
}
add_shortcode( 'saludo', 'shortcode_saludo' );
```

> [!warning] Un shortcode siempre `return`, nunca `echo`
> Si imprimes con `echo`, el texto aparece **al principio de la página**, fuera de su sitio. Si necesitas HTML largo, usa `ob_start()` y `return ob_get_clean();`.

### 4.8 Base de datos: `$wpdb`

WordPress guarda todo en MySQL/MariaDB. Para consultas propias se usa el objeto global `$wpdb` (no hace falta conectar: ya está conectado).

```php
global $wpdb;

// Nombre de tabla con el prefijo correcto (normalmente wp_)
$tabla = $wpdb->prefix . 'mi_tabla';

// SELECT con parámetros SEGUROS
$resultados = $wpdb->get_results(
    $wpdb->prepare( "SELECT * FROM {$tabla} WHERE usuario_id = %d", $usuario_id )
);

// INSERT / UPDATE / DELETE sin escribir SQL
$wpdb->insert( $tabla, array( 'nombre' => 'Ana', 'edad' => 25 ), array( '%s', '%d' ) );
$wpdb->update( $tabla, array( 'edad' => 26 ), array( 'id' => 1 ) );
$wpdb->delete( $tabla, array( 'id' => 1 ) );
```

- Marcadores de `prepare()`: `%s` (texto), `%d` (entero), `%f` (decimal).
- Métodos de lectura: `get_results()` (varias filas), `get_row()` (una fila), `get_var()` (un valor), `get_col()` (una columna).
- Para crear tablas propias se usa `dbDelta()` (se llama en la activación del plugin).

### 4.9 Seguridad en WordPress

Tres ideas que se aplican siempre: **sanear al entrar, escapar al salir y comprobar permisos y nonces**.

**a) Sanear (limpiar datos que llegan):**

| Función | Uso |
|---|---|
| `sanitize_text_field()` | Texto corto normal |
| `sanitize_textarea_field()` | Texto largo con saltos de línea |
| `sanitize_email()` | Emails |
| `sanitize_key()` | Claves / slugs internos |
| `esc_url_raw()` | URLs que se van a guardar |
| `absint()` / `intval()` | Números enteros |
| `wp_kses_post()` | Permite el HTML seguro de las entradas |

**b) Escapar (proteger al mostrar):**

| Función | Uso |
|---|---|
| `esc_html()` | Texto dentro de etiquetas HTML |
| `esc_attr()` | Dentro de atributos (`value="..."`) |
| `esc_url()` | En `href` y `src` |
| `esc_js()` | Dentro de JavaScript en línea |
| `wp_kses_post()` | HTML permitido |

```php
echo '<a href="' . esc_url( $enlace ) . '">' . esc_html( $titulo ) . '</a>';
```

**c) Nonces** (códigos temporales que comprueban que la petición viene de tu propio formulario):

```php
// En el formulario
wp_nonce_field( 'guardar_ajustes', 'mi_nonce' );

// Al recibirlo
if ( ! isset( $_POST['mi_nonce'] ) || ! wp_verify_nonce( $_POST['mi_nonce'], 'guardar_ajustes' ) ) {
    wp_die( 'Petición no válida' );
}
```

**d) Permisos:** antes de hacer algo importante, comprueba que el usuario puede hacerlo.

```php
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( 'No tienes permisos.' );
}
```

Capacidades comunes: `manage_options` (administrador), `edit_posts`, `publish_posts`, `edit_others_posts`, `upload_files`, `delete_posts`.

**Roles de usuario:** Administrador, Editor, Autor, Colaborador, Suscriptor (y Super Administrador en multisitio).

### 4.10 Formularios, AJAX y API REST

**Formulario que envía a `admin-post.php`:**

```php
// El formulario apunta a:  <?php echo esc_url( admin_url( 'admin-post.php' ) ); ?>
// con un campo oculto:     <input type="hidden" name="action" value="enviar_contacto">

add_action( 'admin_post_enviar_contacto', 'procesar_contacto' );        // usuarios logueados
add_action( 'admin_post_nopriv_enviar_contacto', 'procesar_contacto' ); // visitantes

function procesar_contacto() {
    check_admin_referer( 'contacto', 'contacto_nonce' ); // comprueba el nonce
    $nombre = sanitize_text_field( wp_unslash( $_POST['nombre'] ?? '' ) );
    // ... procesar ...
    wp_safe_redirect( home_url( '/gracias/' ) );
    exit;
}
```

**AJAX:** la petición se envía a `admin-ajax.php` con un parámetro `action`.

```php
add_action( 'wp_ajax_mi_accion', 'mi_accion_ajax' );        // logueados
add_action( 'wp_ajax_nopriv_mi_accion', 'mi_accion_ajax' ); // visitantes

function mi_accion_ajax() {
    check_ajax_referer( 'mi_accion', 'nonce' );
    wp_send_json_success( array( 'mensaje' => 'Hecho' ) );
}
```

**API REST propia** (la forma moderna; ver [[PHP 13 - JSON y APIs]]):

```php
add_action( 'rest_api_init', function () {
    register_rest_route( 'miplugin/v1', '/saludo', array(
        'methods'             => 'GET',
        'callback'            => 'api_saludo',
        'permission_callback' => '__return_true', // pública. Siempre hay que indicarlo
    ) );
} );

function api_saludo( WP_REST_Request $request ) {
    $nombre = sanitize_text_field( $request->get_param( 'nombre' ) ?: 'mundo' );
    return rest_ensure_response( array( 'mensaje' => 'Hola, ' . $nombre ) );
}
// URL resultante:  /wp-json/miplugin/v1/saludo
```

WordPress ya incluye una API REST lista para usar, por ejemplo `/wp-json/wp/v2/posts`.

### 4.11 Administración: menús, ajustes y meta boxes

```php
// Página propia en el menú del administrador
function mi_plugin_menu() {
    add_menu_page(
        'Mi Plugin',            // título de la página
        'Mi Plugin',            // texto del menú
        'manage_options',       // permiso necesario
        'mi-plugin',            // slug
        'mi_plugin_pagina',     // función que dibuja la página
        'dashicons-admin-generic'
    );
}
add_action( 'admin_menu', 'mi_plugin_menu' );
```

- **Settings API** (`register_setting()`, `add_settings_section()`, `add_settings_field()`): la forma oficial de crear páginas de ajustes con guardado y seguridad ya resueltos.
- **Meta boxes** (`add_meta_box()`): cajas con campos extra en la pantalla de edición de una entrada. Se guardan con el hook `save_post`.

### 4.12 Traducción (internacionalización)

Todo texto visible debe poder traducirse:

| Función | Uso |
|---|---|
| `__( 'Texto', 'dominio' )` | **Devuelve** el texto traducido |
| `_e( 'Texto', 'dominio' )` | **Imprime** el texto traducido |
| `esc_html__()` / `esc_html_e()` | Traducir y escapar a la vez |
| `_n()` | Singular / plural |

El segundo parámetro es el **text domain** (normalmente el nombre del tema o plugin).

### 4.13 Bloques (Gutenberg) y temas de bloques

- El editor actual de WordPress trabaja con **bloques**. Desde PHP se registran con `register_block_type()`.
- Los **temas de bloques** (block themes) no usan `header.php` ni `footer.php`: usan plantillas HTML en una carpeta `templates/` y un archivo `theme.json` para colores, tipografías y espaciados. Aun así, `functions.php` se sigue usando para el PHP.
- Los temas con plantillas PHP que se han descrito arriba se llaman **temas clásicos**.

### 4.14 Depuración

En `wp-config.php`:

```php
define( 'WP_DEBUG', true );          // activa el modo depuración
define( 'WP_DEBUG_LOG', true );      // guarda los errores en wp-content/debug.log
define( 'WP_DEBUG_DISPLAY', false ); // no mostrarlos en pantalla
```

Para escribir tus propios mensajes: `error_log( 'texto' );` o `error_log( print_r( $variable, true ) );`

> [!warning] Producción
> En una web real, `WP_DEBUG` debe estar en `false`.

---

## 5. Ejemplos prácticos

### 5.1 Básico: modificar el contenido con un filter

Añadir una frase al final de todas las entradas:

```php
function anadir_aviso( $contenido ) {
    if ( is_single() && in_the_loop() && is_main_query() ) {
        $contenido .= '<p><em>Gracias por leer.</em></p>';
    }
    return $contenido; // SIEMPRE devolver
}
add_filter( 'the_content', 'anadir_aviso' );
```

### 5.2 Habitual: listar los últimos proyectos con un shortcode

```php
function shortcode_proyectos( $atts ) {
    $atts = shortcode_atts( array( 'cantidad' => 3 ), $atts, 'proyectos' );

    $consulta = new WP_Query( array(
        'post_type'      => 'proyecto',
        'posts_per_page' => absint( $atts['cantidad'] ),
    ) );

    if ( ! $consulta->have_posts() ) {
        return '<p>No hay proyectos.</p>';
    }

    ob_start();
    echo '<ul class="proyectos">';
    while ( $consulta->have_posts() ) {
        $consulta->the_post();
        printf(
            '<li><a href="%s">%s</a></li>',
            esc_url( get_permalink() ),
            esc_html( get_the_title() )
        );
    }
    echo '</ul>';
    wp_reset_postdata();

    return ob_get_clean();
}
add_shortcode( 'proyectos', 'shortcode_proyectos' );
// Uso en el editor:  [proyectos cantidad="5"]
```

### 5.3 Completo: guardar un campo extra (meta box) de forma segura

```php
// 1. Añadir la caja
function proyecto_meta_box() {
    add_meta_box( 'proyecto_datos', 'Datos del proyecto', 'proyecto_meta_box_html', 'proyecto' );
}
add_action( 'add_meta_boxes', 'proyecto_meta_box' );

// 2. Dibujarla
function proyecto_meta_box_html( $post ) {
    wp_nonce_field( 'guardar_proyecto', 'proyecto_nonce' );
    $url = get_post_meta( $post->ID, 'url_proyecto', true );
    ?>
    <label for="url_proyecto">URL del proyecto:</label>
    <input type="url" id="url_proyecto" name="url_proyecto" value="<?php echo esc_attr( $url ); ?>">
    <?php
}

// 3. Guardarla
function proyecto_guardar( $post_id ) {
    if ( ! isset( $_POST['proyecto_nonce'] ) ||
         ! wp_verify_nonce( $_POST['proyecto_nonce'], 'guardar_proyecto' ) ) {
        return;
    }
    if ( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE ) {
        return;
    }
    if ( ! current_user_can( 'edit_post', $post_id ) ) {
        return;
    }
    if ( isset( $_POST['url_proyecto'] ) ) {
        update_post_meta( $post_id, 'url_proyecto', esc_url_raw( wp_unslash( $_POST['url_proyecto'] ) ) );
    }
}
add_action( 'save_post_proyecto', 'proyecto_guardar' );
```

---

## 6. Buenas prácticas

- **Nunca modificar el núcleo** de WordPress ni plugins/temas de terceros: usa hooks o un tema hijo.
- **Prefijo único** en todas las funciones, constantes y opciones (o usa clases y `namespace`).
- Poner `if ( ! defined( 'ABSPATH' ) ) exit;` al inicio de cada archivo PHP.
- **Sanear lo que entra, escapar lo que sale**, y comprobar **nonce + permisos** en toda acción que guarde datos.
- Usar **`$wpdb->prepare()`** siempre que una consulta lleve variables.
- Encolar CSS y JS con `wp_enqueue_*`, nunca con `<script>` o `<link>` a mano.
- Usar funciones de WordPress antes que reinventarlas (`wp_remote_get()` en vez de cURL, `wp_mail()` en vez de `mail()`).
- Cargar los scripts **solo donde se necesitan** (condicionales como `is_page()` o el parámetro del hook de administración).
- Hacer **traducibles** todos los textos visibles.
- Llamar a **`wp_reset_postdata()`** después de cada `WP_Query` propia.
- Trabajar con `WP_DEBUG` activado en local, y desactivado en producción.
- Cachear con transients las operaciones lentas (APIs externas, consultas pesadas).

### Errores típicos

> [!failure] ❌ Un `filter` que no devuelve nada
> 🤔 **Por qué pasa:** en un filter hay que devolver el valor; si se olvida el `return`, el contenido desaparece.
> ✅ **Cómo se arregla:** terminar siempre la función con `return $valor;`.

> [!failure] ❌ Mostrar datos sin escapar
> 🤔 **Por qué pasa:** `echo $_POST['x']` o `echo get_post_meta(...)` permite que alguien inyecte código (XSS).
> ✅ **Cómo se arregla:** `echo esc_html( $valor );` (o `esc_attr`, `esc_url` según dónde vaya).

> [!failure] ❌ La web da error o "página en blanco" tras editar `functions.php`
> 🤔 **Por qué pasa:** un error de sintaxis (llave sin cerrar, `;` olvidado) rompe todo el sitio.
> ✅ **Cómo se arregla:** activar `WP_DEBUG` para ver el error y corregir; si no se puede entrar, restaurar el archivo por FTP.

---

## 7. Diferencias importantes

| Concepto | Diferencia |
|---|---|
| **Action vs Filter** | La action *ejecuta* algo; el filter *recibe, modifica y devuelve* un valor |
| **Tema vs Plugin** | El tema controla el **aspecto**; el plugin añade **funcionalidad** y debe seguir funcionando si cambias de tema |
| **Tema padre vs hijo** | El hijo hereda del padre y permite personalizar sin perder cambios al actualizar |
| **Tema clásico vs de bloques** | El clásico usa plantillas PHP; el de bloques usa plantillas HTML y `theme.json` |
| **`the_` vs `get_the_`** | `the_` imprime; `get_the_` devuelve |
| **`sanitize_` vs `esc_`** | `sanitize_` limpia lo que **entra**; `esc_` protege lo que **sale** |
| **Post meta vs Options** | Post meta pertenece a **una entrada**; Options son ajustes **globales** |
| **`admin-ajax.php` vs REST API** | AJAX es el método clásico; REST es el moderno y más ordenado |
| **Consulta principal vs `WP_Query`** | La principal ya viene cargada (se ajusta con `pre_get_posts`); `WP_Query` es una consulta extra creada por ti |

---

## 8. Casos especiales

- **`functions.php` del tema hijo** se carga **antes** que el del padre, y ambos se ejecutan (el del hijo no sustituye al del padre).
- **`mu-plugins`** (must-use): plugins que se cargan siempre y no se pueden desactivar desde el panel. Útiles para código imprescindible del sitio.
- **Multisitio (Multisite):** varios sitios dentro de una misma instalación; las funciones `switch_to_blog()` y `restore_current_blog()` permiten saltar entre ellos.
- **WP-CLI:** herramienta de terminal para gestionar WordPress (instalar plugins, crear usuarios, regenerar miniaturas…). Ver [[Terminal 05 - Procesos y red]] y la carpeta de Terminal.
- **Composer en WordPress:** se puede usar para cargar librerías PHP en un plugin (ver [[PHP 17 - Composer]]), pero hay que tener cuidado con los conflictos entre plugins que usan la misma librería.
- **Cron de WordPress** (`wp_schedule_event()`): tareas programadas. No es un cron real: se ejecuta cuando alguien visita la web.
- **Consultas con `meta_query`** sobre muchos datos pueden ser muy lentas; en esos casos conviene una tabla propia con índices.

> [!warning] Obsoleto / legado
> - `query_posts()` → **no usar**. Reemplazar por `pre_get_posts` (consulta principal) o `WP_Query` (consulta extra).
> - `wp_title()` y poner `<title>` a mano → reemplazar por `add_theme_support( 'title-tag' )`.
> - `mysql_*` y conexiones directas a la base de datos → reemplazar por `$wpdb`.
> - Escribir `<script>` y `<link>` directamente en las plantillas → reemplazar por `wp_enqueue_script()` / `wp_enqueue_style()`.
> - `get_bloginfo( 'template_directory' )` → reemplazar por `get_template_directory_uri()`.

---

## 9. Resumen

- WordPress está hecho en PHP y **se personaliza con temas y plugins**, no tocando el núcleo.
- Todo se basa en **hooks**: *actions* (hacer algo) y *filters* (modificar un valor y devolverlo).
- Un **tema** controla el aspecto (`style.css`, `index.php`, `functions.php` + plantillas); un **plugin** añade funcionalidad (cabecera + código).
- **El Loop** y **`WP_Query`** muestran contenidos; `pre_get_posts` modifica la consulta principal.
- CSS y JS se **encolan** con `wp_enqueue_style()` / `wp_enqueue_script()`.
- Con `register_post_type()` y `register_taxonomy()` creas tipos de contenido y clasificaciones propias.
- Los datos se guardan con **post meta**, **options** o **transients**; para SQL propio se usa `$wpdb` con `prepare()`.
- Los **shortcodes** devuelven (`return`) HTML; la **API REST** y **AJAX** sirven para comunicar con JavaScript.
- **Seguridad:** sanear al entrar, escapar al salir, comprobar **nonces** y **permisos** (`current_user_can`).
- Hacer todo traducible, depurar con `WP_DEBUG` y evitar funciones obsoletas como `query_posts()`.