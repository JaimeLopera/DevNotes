# 01 - Errores de HTML y CSS

> [!info] ¿Qué es?
> HTML y CSS casi nunca muestran mensajes de error: simplemente **no se ve lo que esperabas**. Esta nota recoge los síntomas más frecuentes, su causa y su solución. La herramienta clave es el **inspector del navegador** (F12 → *Elements*).

---

## 1. Antes de empezar: cómo investigar

- **Clic derecho → Inspeccionar** sobre el elemento problemático.
- En el panel *Styles* verás qué reglas se aplican y cuáles están **tachadas** (anuladas por otra).
- En *Computed* verás el valor final de cada propiedad.
- En la pestaña *Network* comprueba si el CSS o las imágenes cargan (código 200) o fallan (404).

---

## 2. El CSS no se aplica

### 2.1 Ningún estilo funciona

**Qué significa:** el navegador no está cargando el archivo CSS.

**Causas típicas:**
- Ruta incorrecta en el `<link>` (la ruta es relativa al HTML, no al CSS).
- Error tipográfico: `rel="stylesheet"` mal escrito o ausente.
- La caché del navegador muestra una versión antigua.

**Solución:**

```html
<!-- index.html en la raíz, CSS en css/style.css -->
<link rel="stylesheet" href="css/style.css">
```

Comprueba en *Network* que `style.css` responde 200. Para saltarte la caché: `Ctrl + Shift + R`.

### 2.2 Una regla concreta no se aplica (aparece tachada)

**Qué significa:** otra regla más **específica** o posterior la sobrescribe.

**Causas típicas:**
- Un selector con más especificidad (`#id` gana a `.clase`, `.a .b` gana a `.b`).
- Otra regla igual de específica aparece **después** en el CSS.
- Un `!important` en otra parte.

**Solución:** iguala o sube la especificidad de forma controlada, o reordena. Evita usar `!important` como parche.

```css
/* En lugar de .boton { color: red !important; } */
.header .boton { color: red; }
```

Más en [[CSS 01 - Fundamentos|Fundamentos de CSS]] (cascada y especificidad).

### 2.3 El selector no coincide con nada

**Causas típicas:**
- Falta el punto o la almohadilla: `boton` en lugar de `.boton` / `#boton`.
- La clase está mal escrita en el HTML (las clases distinguen mayúsculas).
- Espacio de más: `.tarjeta .activa` (descendiente) no es lo mismo que `.tarjeta.activa` (mismo elemento).

### 2.4 Error de sintaxis que anula el resto

**Causa típica:** falta un `;` o una `}`. El navegador ignora todo hasta el siguiente punto válido.

**Solución:** revisa la regla anterior a la que falla. Un editor con *linter* (o el validador del W3C) lo marca.

---

## 3. Layout roto

### 3.1 El elemento es más ancho de lo que indiqué

**Qué significa:** `padding` y `border` se suman al `width`.

**Solución:**

```css
*, *::before, *::after { box-sizing: border-box; }
```

Más en [[CSS 03 - Box Model|Box Model]].

### 3.2 Aparece scroll horizontal en móvil

**Causas típicas:**
- Un elemento con ancho fijo mayor que la pantalla (`width: 1200px`).
- Imágenes sin `max-width: 100%`.
- Textos largos sin espacios (URLs).
- Uso de `100vw` (incluye el ancho de la barra de scroll).

**Solución:**

```css
img, video { max-width: 100%; height: auto; }
p { overflow-wrap: break-word; }
.contenedor { width: min(100% - 2rem, 1200px); }
```

Truco para encontrar al culpable: `* { outline: 1px solid red; }`.

### 3.3 `margin: auto` no centra

**Causas típicas:**
- El elemento es *inline* (`<span>`, `<a>`): necesita `display: block`.
- No tiene ancho definido (ocupa el 100 %, no hay nada que centrar).

**Solución:**

```css
.caja { display: block; max-width: 600px; margin-inline: auto; }
```

### 3.4 `height: 100%` no funciona

**Qué significa:** el porcentaje se calcula sobre la altura del padre, y el padre no tiene altura definida.

**Solución:** usa `min-height: 100vh` (o `100dvh` en móvil), o Grid/Flexbox en el padre.

### 3.5 Los márgenes verticales "desaparecen"

**Qué significa:** *colapso de márgenes*: dos márgenes verticales adyacentes se fusionan en el mayor.

**Solución:** usa `gap` en un contenedor Flex/Grid, o `padding` en el padre.

### 3.6 `z-index` no hace nada

**Causas típicas:**
- El elemento tiene `position: static` (el valor por defecto).
- Está dentro de otro *contexto de apilamiento* (un padre con `transform`, `opacity < 1` o su propio `z-index`).

**Solución:** añade `position: relative` y revisa los `z-index` de los padres.

### 3.7 `position: sticky` no se queda fijo

**Causas típicas:**
- Falta `top: 0`.
- Algún padre tiene `overflow: hidden` o `overflow: auto`.
- El padre es tan alto como el propio elemento.

### 3.8 Los elementos flex no se reparten como espero

**Causas típicas:**
- `display: flex` está en el elemento equivocado (va en el **padre**).
- Los hijos no encogen por contenido largo: añade `min-width: 0` al hijo.

Más en [[CSS 07 - Flexbox|Flexbox]].

---

## 4. Imágenes, fuentes y recursos

### 4.1 La imagen no aparece (icono roto)

**Causas típicas:**
- Ruta mal escrita o relativa a otra carpeta.
- Mayúsculas: `Foto.JPG` ≠ `foto.jpg` (en servidores Linux importa).
- Espacios o tildes en el nombre del archivo.

**Solución:** comprueba en *Network* la URL exacta que se pide; renombra los archivos en minúsculas y sin espacios.

### 4.2 La imagen de fondo de CSS no carga

**Qué significa:** en CSS, la ruta es relativa **al archivo CSS**, no al HTML.

```css
/* css/style.css → img/fondo.jpg */
.hero { background-image: url("../img/fondo.jpg"); }
```

### 4.3 Hay un hueco debajo de las imágenes

**Qué significa:** la imagen es *inline* y deja espacio para los descendentes del texto.

**Solución:** `img { display: block; }`.

### 4.4 La fuente no se aplica

**Causas típicas:** el nombre en `font-family` no coincide con el de `@font-face`, la ruta del archivo es incorrecta o falta el `<link>` de Google Fonts.

---

## 5. HTML

### 5.1 Caracteres raros (Ã±, Ã©)

**Qué significa:** problema de codificación.

**Solución:** `<meta charset="UTF-8">` como primera línea del `<head>` y guarda el archivo en UTF-8.

### 5.2 En móvil la web se ve diminuta

**Solución:** falta la etiqueta viewport:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### 5.3 El formulario recarga la página o envía vacío

**Causas típicas:**
- Los campos no tienen atributo `name` (sin `name`, no se envían).
- Un `<button>` dentro de un `<form>` es de tipo `submit` por defecto: usa `type="button"` si no debe enviar.

### 5.4 El `<label>` no enfoca el campo

**Solución:** el `for` del label debe coincidir con el `id` del input.

### 5.5 El enlace abre una ruta extraña

**Causa típica:** falta `https://` en un enlace externo (`href="google.com"` se trata como ruta relativa).

---

## 6. Resumen

- **CSS no carga** → revisa la ruta en *Network* y la caché (`Ctrl + Shift + R`).
- **Regla tachada** → otra más específica gana; ajusta la especificidad.
- **Más ancho de lo esperado** → `box-sizing: border-box`.
- **Scroll horizontal** → imágenes sin `max-width`, anchos fijos, `100vw`.
- **`z-index` no va** → falta `position` o hay otro contexto de apilamiento.
- **Imagen rota** → ruta, mayúsculas o espacios.
- Sin `name` los campos no se envían; sin `viewport` el móvil se ve mal.
