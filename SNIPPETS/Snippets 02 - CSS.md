# 02 - Snippets de CSS

> [!info] ¿Qué es?
> Fragmentos de **CSS** listos para copiar: reset, centrado, layouts, componentes y utilidades. Las explicaciones están en la carpeta [[CSS 00 - Índice|CSS]].

---

## 1. Base

### 1.1 Reset moderno

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

* {
  margin: 0;
}

html {
  -webkit-text-size-adjust: none;
}

body {
  min-height: 100vh;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select {
  font: inherit;
}

p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}
```

### 1.2 Variables de diseño

```css
:root {
  --color-fondo: #ffffff;
  --color-texto: #1e293b;
  --color-primario: #2563eb;
  --color-borde: #e2e8f0;

  --espacio-xs: 0.25rem;
  --espacio-s: 0.5rem;
  --espacio-m: 1rem;
  --espacio-l: 2rem;

  --radio: 0.5rem;
  --sombra: 0 4px 12px rgb(0 0 0 / 0.08);
  --fuente: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
}

body {
  font-family: var(--fuente);
  color: var(--color-texto);
  background: var(--color-fondo);
}
```

Más en [[CSS 13 - Variables y funciones|Variables y funciones]].

---

## 2. Centrar

### 2.1 Centrar con Grid (lo más corto)

```css
.contenedor {
  display: grid;
  place-items: center;
  min-height: 100vh;
}
```

### 2.2 Centrar con Flexbox

```css
.contenedor {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### 2.3 Centrar un bloque horizontalmente

```css
.caja {
  max-width: 70ch;
  margin-inline: auto;
}
```

### 2.4 Centrar sobre otro elemento (posición absoluta)

```css
.padre { position: relative; }
.hijo {
  position: absolute;
  inset: 0;
  margin: auto;
  width: fit-content;
  height: fit-content;
}
```

---

## 3. Layouts

### 3.1 Contenedor de ancho máximo

```css
.contenedor {
  width: min(100% - 2rem, 1200px);
  margin-inline: auto;
}
```

### 3.2 Rejilla responsive sin media queries

```css
.rejilla {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}
```

### 3.3 Footer siempre abajo

```css
body {
  min-height: 100vh;
  display: grid;
  grid-template-rows: auto 1fr auto;  /* header, main, footer */
}
```

### 3.4 Barra lateral + contenido

```css
.layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  gap: 2rem;
}

@media (max-width: 768px) {
  .layout { grid-template-columns: 1fr; }
}
```

### 3.5 Header con logo a la izquierda y menú a la derecha

```css
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
}
```

### 3.6 Header fijo arriba

```css
.header {
  position: sticky;
  top: 0;
  z-index: 10;
  background: var(--color-fondo);
}
```

Más en [[CSS 07 - Flexbox|Flexbox]] y [[CSS 08 - Grid|Grid]].

---

## 4. Componentes

### 4.1 Botón

```css
.boton {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.2rem;
  border: none;
  border-radius: var(--radio);
  background: var(--color-primario);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
}

.boton:hover { background: #1d4ed8; }
.boton:active { transform: scale(0.98); }
.boton:focus-visible { outline: 3px solid #93c5fd; outline-offset: 2px; }
.boton:disabled { opacity: 0.5; cursor: not-allowed; }
```

### 4.2 Tarjeta

```css
.tarjeta {
  padding: 1.5rem;
  border: 1px solid var(--color-borde);
  border-radius: var(--radio);
  background: white;
  box-shadow: var(--sombra);
}
```

### 4.3 Imagen que rellena su caja sin deformarse

```css
.portada {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}
```

### 4.4 Texto truncado con "…"

```css
/* Una línea */
.una-linea {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Varias líneas */
.tres-lineas {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

### 4.5 Fondo semitransparente con desenfoque

```css
.cristal {
  background: rgb(255 255 255 / 0.6);
  backdrop-filter: blur(10px);
}
```

---

## 5. Modo oscuro y preferencias

### 5.1 Modo oscuro automático

```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-fondo: #0f172a;
    --color-texto: #e2e8f0;
    --color-borde: #334155;
  }
}
```

### 5.2 Modo oscuro con interruptor (clase en `<html>`)

```css
:root.oscuro {
  --color-fondo: #0f172a;
  --color-texto: #e2e8f0;
}
```

(JavaScript del interruptor en [[Snippets 03 - JavaScript]] § 6.)

### 5.3 Respetar a quien no quiere animaciones

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## 6. Utilidades

### 6.1 Ocultar visualmente (accesible)

```css
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### 6.2 Desplazamiento suave

```css
html { scroll-behavior: smooth; }
```

### 6.3 Tipografía fluida

```css
h1 { font-size: clamp(1.75rem, 4vw + 1rem, 3.5rem); }
```

### 6.4 Selección de texto con el color de la marca

```css
::selection {
  background: var(--color-primario);
  color: white;
}
```

### 6.5 Barra de scroll fina

```css
* {
  scrollbar-width: thin;
  scrollbar-color: #94a3b8 transparent;
}
```

---

## 7. Animaciones

### 7.1 Aparecer suavemente

```css
@keyframes aparecer {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.aparece { animation: aparecer 0.4s ease-out both; }
```

### 7.2 Indicador de carga (spinner)

```css
.spinner {
  width: 2rem;
  height: 2rem;
  border: 3px solid #e2e8f0;
  border-top-color: var(--color-primario);
  border-radius: 50%;
  animation: girar 0.8s linear infinite;
}

@keyframes girar { to { transform: rotate(360deg); } }
```

Más en [[CSS 11 - Transiciones y animaciones|Transiciones y animaciones]].

---

## 8. Media queries habituales

```css
/* Móvil primero: estilos base para móvil y luego… */
@media (min-width: 640px)  { /* tablet pequeña */ }
@media (min-width: 768px)  { /* tablet */ }
@media (min-width: 1024px) { /* escritorio */ }
@media (min-width: 1280px) { /* pantallas grandes */ }

/* Dispositivos táctiles (sin hover) */
@media (hover: none) { }
```

Más en [[CSS 10 - Responsive Design|Responsive Design]].
