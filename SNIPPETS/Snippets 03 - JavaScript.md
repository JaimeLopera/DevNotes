# 03 - Snippets de JavaScript

> [!info] ¿Qué es?
> Fragmentos de **JavaScript** listos para copiar: DOM, eventos, peticiones, utilidades de arrays, fechas, almacenamiento y rendimiento. Las explicaciones están en la carpeta [[JavaScript 00 - Índice|JavaScript]].

---

## 1. DOM

### 1.1 Seleccionar elementos

```javascript
const $ = (selector, padre = document) => padre.querySelector(selector);
const $$ = (selector, padre = document) => [...padre.querySelectorAll(selector)];

const boton = $("#enviar");
const tarjetas = $$(".tarjeta");
```

### 1.2 Crear un elemento con contenido seguro

```javascript
function crearElemento(etiqueta, { clase, texto, atributos = {} } = {}) {
  const el = document.createElement(etiqueta);
  if (clase) el.className = clase;
  if (texto) el.textContent = texto;          // textContent evita XSS
  for (const [k, v] of Object.entries(atributos)) el.setAttribute(k, v);
  return el;
}

const enlace = crearElemento("a", { texto: "Ver más", atributos: { href: "/blog" } });
document.body.append(enlace);
```

### 1.3 Pintar una lista desde un array

```javascript
function pintarLista(contenedor, items) {
  contenedor.replaceChildren(
    ...items.map((item) => {
      const li = document.createElement("li");
      li.textContent = item.nombre;
      return li;
    })
  );
}
```

### 1.4 Mostrar / ocultar

```javascript
elemento.hidden = !elemento.hidden;
elemento.classList.toggle("activo");
```

Más en [[JavaScript 09 - DOM|DOM]].

---

## 2. Eventos

### 2.1 Delegación de eventos (un listener para muchos elementos)

```javascript
document.querySelector("#lista").addEventListener("click", (e) => {
  const boton = e.target.closest("[data-borrar]");
  if (!boton) return;
  borrarItem(boton.dataset.borrar);
});
```

### 2.2 Ejecutar cuando el DOM está listo

```javascript
document.addEventListener("DOMContentLoaded", () => {
  // tu código
});
```

(Innecesario si el script lleva `defer` o es `type="module"`.)

### 2.3 Cerrar con la tecla Escape

```javascript
document.addEventListener("keydown", (e) => {
  if (e.key === "Escape") cerrarModal();
});
```

### 2.4 Clic fuera de un elemento

```javascript
document.addEventListener("click", (e) => {
  if (!menu.contains(e.target) && !botonMenu.contains(e.target)) {
    menu.hidden = true;
  }
});
```

Más en [[JavaScript 10 - Eventos|Eventos]].

---

## 3. Peticiones (fetch)

### 3.1 GET con manejo de errores

```javascript
async function obtener(url) {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`Error ${res.status}: ${res.statusText}`);
  return res.json();
}

try {
  const productos = await obtener("/api/productos");
} catch (error) {
  console.error(error.message);
}
```

### 3.2 POST con JSON

```javascript
async function enviar(url, datos) {
  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(datos),
  });
  if (!res.ok) throw new Error(`Error ${res.status}`);
  return res.json();
}
```

### 3.3 Enviar un formulario sin recargar

```javascript
form.addEventListener("submit", async (e) => {
  e.preventDefault();
  const datos = Object.fromEntries(new FormData(form));
  boton.disabled = true;
  try {
    await enviar("/api/contacto", datos);
    form.reset();
    aviso.textContent = "¡Mensaje enviado!";
  } catch {
    aviso.textContent = "No se pudo enviar. Inténtalo de nuevo.";
  } finally {
    boton.disabled = false;
  }
});
```

### 3.4 Petición con tiempo máximo

```javascript
const res = await fetch(url, { signal: AbortSignal.timeout(5000) }); // 5 s
```

### 3.5 Varias peticiones en paralelo

```javascript
const [usuarios, pedidos] = await Promise.all([
  obtener("/api/usuarios"),
  obtener("/api/pedidos"),
]);
```

Más en [[JavaScript 13 - Fetch y APIs|Fetch y APIs]].

---

## 4. Arrays y objetos

### 4.1 Operaciones frecuentes

```javascript
const total = carrito.reduce((suma, p) => suma + p.precio * p.cantidad, 0);
const baratos = productos.filter((p) => p.precio < 20);
const nombres = productos.map((p) => p.nombre);
const producto = productos.find((p) => p.id === 3);
const hayAgotados = productos.some((p) => p.stock === 0);
const ordenados = productos.toSorted((a, b) => a.precio - b.precio);
```

### 4.2 Quitar duplicados

```javascript
const unicos = [...new Set(lista)];
const unicosPorId = [...new Map(lista.map((x) => [x.id, x])).values()];
```

### 4.3 Agrupar por una propiedad

```javascript
const porCategoria = Object.groupBy(productos, (p) => p.categoria);
```

### 4.4 Copia profunda

```javascript
const copia = structuredClone(objeto);
```

### 4.5 Ordenar textos con tildes correctamente

```javascript
nombres.sort((a, b) => a.localeCompare(b, "es"));
```

Más en [[JavaScript 07 - Arrays|Arrays]] y [[JavaScript 08 - Objetos|Objetos]].

---

## 5. Formato de números y fechas

### 5.1 Precio en euros

```javascript
const euros = new Intl.NumberFormat("es-ES", { style: "currency", currency: "EUR" });
euros.format(1234.5); // "1234,50 €"
```

### 5.2 Fecha legible

```javascript
new Date().toLocaleDateString("es-ES", { day: "numeric", month: "long", year: "numeric" });
// "13 de noviembre de 2026"
```

### 5.3 Fecha en formato YYYY-MM-DD

```javascript
const hoy = new Date().toISOString().slice(0, 10);
```

### 5.4 "Hace X tiempo"

```javascript
function haceTiempo(fecha) {
  const segundos = (fecha - Date.now()) / 1000;
  const rtf = new Intl.RelativeTimeFormat("es", { numeric: "auto" });
  const unidades = [["year", 31536000], ["month", 2592000], ["day", 86400], ["hour", 3600], ["minute", 60]];
  for (const [unidad, s] of unidades) {
    if (Math.abs(segundos) >= s) return rtf.format(Math.round(segundos / s), unidad);
  }
  return rtf.format(Math.round(segundos), "second");
}
```

---

## 6. Almacenamiento

### 6.1 Guardar y leer objetos en localStorage

```javascript
const almacen = {
  guardar: (clave, valor) => localStorage.setItem(clave, JSON.stringify(valor)),
  leer: (clave, porDefecto = null) => {
    try {
      return JSON.parse(localStorage.getItem(clave)) ?? porDefecto;
    } catch {
      return porDefecto;
    }
  },
};
```

### 6.2 Interruptor de modo oscuro que se recuerda

```javascript
const raiz = document.documentElement;
const prefiereOscuro = matchMedia("(prefers-color-scheme: dark)").matches;
raiz.classList.toggle("oscuro", almacen.leer("tema", prefiereOscuro ? "oscuro" : "claro") === "oscuro");

botonTema.addEventListener("click", () => {
  const oscuro = raiz.classList.toggle("oscuro");
  almacen.guardar("tema", oscuro ? "oscuro" : "claro");
});
```

Más en [[JavaScript 16 - Web Storage|Web Storage]].

---

## 7. Rendimiento

### 7.1 Debounce (esperar a que el usuario deje de escribir)

```javascript
function debounce(fn, espera = 300) {
  let temporizador;
  return (...args) => {
    clearTimeout(temporizador);
    temporizador = setTimeout(() => fn(...args), espera);
  };
}

buscador.addEventListener("input", debounce((e) => buscar(e.target.value), 400));
```

### 7.2 Throttle (como máximo una vez cada X ms)

```javascript
function throttle(fn, intervalo = 200) {
  let ultimo = 0;
  return (...args) => {
    const ahora = Date.now();
    if (ahora - ultimo >= intervalo) {
      ultimo = ahora;
      fn(...args);
    }
  };
}

window.addEventListener("scroll", throttle(actualizarBarra, 100));
```

### 7.3 Cargar algo al entrar en pantalla

```javascript
const observador = new IntersectionObserver((entradas) => {
  for (const entrada of entradas) {
    if (entrada.isIntersecting) {
      entrada.target.classList.add("visible");
      observador.unobserve(entrada.target);
    }
  }
});

document.querySelectorAll(".animar").forEach((el) => observador.observe(el));
```

---

## 8. Utilidades varias

### 8.1 Copiar al portapapeles

```javascript
await navigator.clipboard.writeText("Texto copiado");
```

### 8.2 Leer parámetros de la URL

```javascript
const params = new URLSearchParams(location.search);
const pagina = Number(params.get("pagina") ?? 1);
```

### 8.3 Esperar X milisegundos

```javascript
const esperar = (ms) => new Promise((r) => setTimeout(r, ms));
await esperar(1000);
```

### 8.4 Número aleatorio entre dos valores

```javascript
const aleatorio = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
```

### 8.5 Identificador único

```javascript
const id = crypto.randomUUID();
```

### 8.6 Descargar un archivo generado

```javascript
function descargar(nombre, contenido, tipo = "text/plain") {
  const url = URL.createObjectURL(new Blob([contenido], { type: tipo }));
  const a = Object.assign(document.createElement("a"), { href: url, download: nombre });
  a.click();
  URL.revokeObjectURL(url);
}

descargar("datos.json", JSON.stringify(datos, null, 2), "application/json");
```
