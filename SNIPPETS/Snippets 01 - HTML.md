# 01 - Snippets de HTML

> [!info] ¿Qué es?
> Fragmentos de **HTML** listos para copiar: la plantilla base, metaetiquetas, formularios, imágenes y patrones accesibles. Las explicaciones están en la carpeta [[HTML 00 - Índice|HTML]].

---

## 1. Plantillas

### 1.1 Documento base

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Título de la página</title>
  <meta name="description" content="Descripción de 150-160 caracteres para buscadores.">
  <link rel="icon" href="/favicon.svg" type="image/svg+xml">
  <link rel="stylesheet" href="css/style.css">
  <script src="js/main.js" defer></script>
</head>
<body>
  <header></header>
  <main></main>
  <footer></footer>
</body>
</html>
```

Más en [[HTML 01 - Fundamentos|Fundamentos de HTML]].

### 1.2 Estructura semántica de una página

```html
<header>
  <a href="/" class="logo">Mi web</a>
  <nav aria-label="Principal">
    <ul>
      <li><a href="/" aria-current="page">Inicio</a></li>
      <li><a href="/blog">Blog</a></li>
      <li><a href="/contacto">Contacto</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h1>Título del artículo</h1>
    <p>Contenido…</p>
  </article>
  <aside>Contenido relacionado</aside>
</main>

<footer>
  <p>&copy; 2026 Mi web</p>
</footer>
```

Más en [[HTML 02 - Estructura y semántica|Estructura y semántica]].

---

## 2. Metaetiquetas

### 2.1 Para compartir en redes (Open Graph)

```html
<meta property="og:title" content="Título al compartir">
<meta property="og:description" content="Descripción al compartir">
<meta property="og:image" content="https://mi-web.com/imagen-1200x630.jpg">
<meta property="og:url" content="https://mi-web.com/pagina">
<meta property="og:type" content="website">
<meta name="twitter:card" content="summary_large_image">
```

### 2.2 Color de la barra del navegador en móvil y modo oscuro

```html
<meta name="theme-color" content="#0f172a">
<meta name="color-scheme" content="light dark">
```

### 2.3 Evitar que se indexe una página

```html
<meta name="robots" content="noindex, nofollow">
```

Más en [[HTML 09 - SEO|SEO]].

---

## 3. Formularios

### 3.1 Formulario de contacto accesible y validado

```html
<form action="/contacto" method="post">
  <label for="nombre">Nombre</label>
  <input id="nombre" name="nombre" type="text" required autocomplete="name">

  <label for="email">Email</label>
  <input id="email" name="email" type="email" required autocomplete="email">

  <label for="mensaje">Mensaje</label>
  <textarea id="mensaje" name="mensaje" rows="5" required minlength="10"></textarea>

  <label>
    <input type="checkbox" name="privacidad" required>
    Acepto la política de privacidad
  </label>

  <button type="submit">Enviar</button>
</form>
```

### 3.2 Grupo de opciones (radio)

```html
<fieldset>
  <legend>Método de envío</legend>
  <label><input type="radio" name="envio" value="normal" checked> Normal</label>
  <label><input type="radio" name="envio" value="urgente"> Urgente</label>
</fieldset>
```

### 3.3 Selector con opción por defecto

```html
<label for="pais">País</label>
<select id="pais" name="pais" required>
  <option value="" disabled selected>Elige un país</option>
  <option value="ES">España</option>
  <option value="MX">México</option>
</select>
```

### 3.4 Campo con patrón (código postal español)

```html
<input type="text" name="cp" inputmode="numeric" pattern="[0-9]{5}" title="5 dígitos" required>
```

Más en [[HTML 06 - Formularios|Formularios]].

---

## 4. Imágenes y multimedia

### 4.1 Imagen optimizada

```html
<img src="foto-800.jpg" alt="Descripción de la imagen"
     width="800" height="600" loading="lazy" decoding="async">
```

### 4.2 Imagen responsive con varios tamaños

```html
<img src="foto-800.jpg"
     srcset="foto-400.jpg 400w, foto-800.jpg 800w, foto-1600.jpg 1600w"
     sizes="(max-width: 600px) 100vw, 50vw"
     alt="Descripción" width="800" height="600" loading="lazy">
```

### 4.3 Formatos modernos con alternativa

```html
<picture>
  <source srcset="foto.avif" type="image/avif">
  <source srcset="foto.webp" type="image/webp">
  <img src="foto.jpg" alt="Descripción" width="800" height="600">
</picture>
```

### 4.4 Vídeo de YouTube responsive

```html
<iframe src="https://www.youtube-nocookie.com/embed/ID_DEL_VIDEO"
        title="Título del vídeo" loading="lazy"
        style="width: 100%; aspect-ratio: 16 / 9; border: 0;"
        allowfullscreen></iframe>
```

Más en [[HTML 04 - Imágenes y multimedia|Imágenes y multimedia]].

---

## 5. Componentes sin JavaScript

### 5.1 Acordeón / FAQ

```html
<details>
  <summary>¿Hacéis envíos internacionales?</summary>
  <p>Sí, a toda la Unión Europea.</p>
</details>
```

### 5.2 Ventana modal nativa

```html
<dialog id="modal">
  <h2>Confirmar</h2>
  <p>¿Seguro que quieres continuar?</p>
  <form method="dialog">
    <button value="cancelar">Cancelar</button>
    <button value="ok">Aceptar</button>
  </form>
</dialog>

<button onclick="document.getElementById('modal').showModal()">Abrir</button>
```

### 5.3 Barra de progreso

```html
<label for="subida">Subida:</label>
<progress id="subida" max="100" value="70">70 %</progress>
```

Más en [[HTML 10 - APIs HTML|APIs HTML]].

---

## 6. Accesibilidad

### 6.1 Enlace para saltar al contenido

```html
<a href="#contenido" class="saltar">Saltar al contenido</a>
<!-- ... -->
<main id="contenido">…</main>
```

### 6.2 Texto solo para lectores de pantalla

```html
<button>
  <svg aria-hidden="true">…</svg>
  <span class="visually-hidden">Cerrar menú</span>
</button>
```

(CSS de `.visually-hidden` en [[Snippets 02 - CSS]] § 6.)

### 6.3 Enlace externo seguro

```html
<a href="https://otra-web.com" target="_blank" rel="noopener noreferrer">
  Otra web <span class="visually-hidden">(se abre en una pestaña nueva)</span>
</a>
```

### 6.4 Mensajes dinámicos que se anuncian

```html
<div role="status" aria-live="polite" id="aviso"></div>
```

Más en [[HTML 08 - Accesibilidad|Accesibilidad]].

---

## 7. Tablas

### 7.1 Tabla accesible

```html
<table>
  <caption>Ventas por trimestre</caption>
  <thead>
    <tr><th scope="col">Trimestre</th><th scope="col">Ventas</th></tr>
  </thead>
  <tbody>
    <tr><th scope="row">T1</th><td>12 000 €</td></tr>
    <tr><th scope="row">T2</th><td>15 500 €</td></tr>
  </tbody>
</table>
```

Más en [[HTML 05 - Listas y tablas|Listas y tablas]].
