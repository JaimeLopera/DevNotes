# 09 - Errores de red y CORS

> [!info] ¿Qué es?
> Los errores que aparecen cuando el **navegador se comunica con un servidor**: CORS, códigos de estado 4xx y 5xx, contenido mixto, peticiones que no llegan o tardan demasiado. La herramienta clave es la pestaña **Network** de las herramientas del navegador (F12).

---

## 1. Antes de empezar: cómo investigar en Network

1. Abre F12 → **Network** y recarga o repite la acción.
2. Haz clic en la petición que falla (en rojo).
3. Revisa:
   - **Headers**: URL exacta, método, código de estado, cabeceras enviadas y recibidas.
   - **Payload**: qué datos enviaste.
   - **Response / Preview**: qué respondió el servidor.
4. Mira también la **terminal del servidor**: ahí aparecen sus errores.

Más en [[Conceptos 01 - HTTP]].

---

## 2. CORS

### 2.1 `Access to fetch at 'http://localhost:3000/api' from origin 'http://localhost:5173' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

**Qué significa:** el navegador bloquea la respuesta porque el servidor no ha autorizado a tu web (otro **origen**: distinto dominio, puerto o protocolo) a leerla.

> [!info] Clave
> CORS lo aplica **el navegador**, y se arregla **en el servidor**. Con `curl` o Postman la misma petición funciona porque no aplican CORS.

**Solución en Express:**

```javascript
import cors from "cors";
app.use(cors({ origin: "http://localhost:5173" }));
```

**Solución en PHP:**

```php
header('Access-Control-Allow-Origin: http://localhost:5173');
header('Access-Control-Allow-Headers: Content-Type, Authorization');
header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') exit;
```

**Alternativa en desarrollo:** usar el *proxy* de Vite para que la petición salga del mismo origen:

```javascript
// vite.config.js
export default {
  server: { proxy: { "/api": "http://localhost:3000" } },
};
// y en el frontend: fetch("/api/productos")
```

### 2.2 `Response to preflight request doesn't pass access control check`

**Qué significa:** antes de ciertas peticiones (con `Content-Type: application/json`, cabeceras como `Authorization`, o métodos `PUT`/`DELETE`), el navegador envía una petición **OPTIONS** (*preflight*), y el servidor no la responde correctamente.

**Solución:** el servidor debe responder al `OPTIONS` con las cabeceras `Access-Control-Allow-*` (el middleware `cors` lo hace solo).

### 2.3 `The value of the 'Access-Control-Allow-Origin' header must not be the wildcard '*' when the request's credentials mode is 'include'`

**Qué significa:** envías cookies (`credentials: "include"`) y el servidor responde con `*`.

**Solución:**

```javascript
app.use(cors({ origin: "http://localhost:5173", credentials: true }));
```

### 2.4 `Request header field authorization is not allowed by Access-Control-Allow-Headers`

**Solución:** añade esa cabecera a las permitidas: `allowedHeaders: ["Content-Type", "Authorization"]`.

---

## 3. Códigos de estado 4xx (el problema está en la petición)

| Código | Significa | Causas típicas | Qué revisar |
|---|---|---|---|
| **400** Bad Request | Datos inválidos | JSON mal formado, faltan campos, tipos erróneos | El *Payload* y el mensaje de error de la respuesta |
| **401** Unauthorized | No autenticado | Falta el token o ha caducado | La cabecera `Authorization` |
| **403** Forbidden | Sin permiso | El usuario no tiene ese rol; CSRF; permisos de archivos en el servidor | Roles, token CSRF |
| **404** Not Found | No existe | URL mal escrita, ruta no definida, prefijo `/api` olvidado | La URL exacta en *Headers* |
| **405** Method Not Allowed | Método no admitido | Haces `POST` a una ruta que solo acepta `GET` | Método de la petición y de la ruta |
| **409** Conflict | Conflicto | Email o nombre ya registrado | Datos duplicados |
| **413** Payload Too Large | Cuerpo demasiado grande | Subida de archivos grande | Límites del servidor (`express.json({ limit })`, `php.ini`) |
| **415** Unsupported Media Type | Formato no aceptado | Falta `Content-Type: application/json` | Cabeceras enviadas |
| **422** Unprocessable Entity | Validación fallida | Datos con formato correcto pero valores no válidos | Mensaje de validación |
| **429** Too Many Requests | Demasiadas peticiones | Límite de la API superado | Espera, o reduce la frecuencia |

### 3.1 404 en una SPA al recargar una ruta (`/perfil`)

**Qué significa:** el servidor busca un archivo `/perfil` que no existe; las rutas las gestiona el JavaScript del frontend.

**Solución:** configura el servidor para devolver `index.html` en cualquier ruta desconocida (*fallback*). Netlify: archivo `_redirects` con `/* /index.html 200`; Nginx: `try_files $uri /index.html;`.

---

## 4. Códigos de estado 5xx (el problema está en el servidor)

| Código | Significa | Qué hacer |
|---|---|---|
| **500** Internal Server Error | El código del servidor falló | Mira la terminal o los logs del servidor: ahí está el error real |
| **502** Bad Gateway | El proxy (Nginx, hosting) no obtiene respuesta de tu app | La app está caída o en otro puerto |
| **503** Service Unavailable | Servidor sobrecargado o en mantenimiento | Espera o revisa recursos |
| **504** Gateway Timeout | La app tarda demasiado en responder | Consulta lenta, bucle, servicio externo bloqueado |

> [!tip] Un 500 no se arregla en el frontend
> El navegador solo sabe que "algo falló". El mensaje útil está en el **servidor**.

---

## 5. Peticiones que no llegan

### 5.1 `net::ERR_CONNECTION_REFUSED`

**Qué significa:** no hay nada escuchando en esa dirección y puerto.

**Solución:** arranca el servidor; comprueba el puerto (`3000` vs `8000`) y el protocolo (`http` vs `https`).

### 5.2 `TypeError: Failed to fetch` / `NetworkError when attempting to fetch resource`

**Causas típicas:** servidor apagado, error de CORS (mira la consola), URL mal formada, sin conexión, o bloqueado por una extensión (bloqueador de anuncios).

### 5.3 `net::ERR_NAME_NOT_RESOLVED`

**Qué significa:** el dominio no existe o el DNS no lo encuentra.

**Solución:** revisa el dominio; si es nuevo, el DNS puede tardar en propagarse.

### 5.4 `Mixed Content: The page at 'https://...' was loaded over HTTPS, but requested an insecure resource 'http://...'`

**Qué significa:** una web segura (HTTPS) intenta cargar algo por HTTP; el navegador lo bloquea.

**Solución:** usa `https://` en todas las URLs (API, imágenes, scripts).

### 5.5 `net::ERR_CERT_AUTHORITY_INVALID` / `NET::ERR_CERT_DATE_INVALID`

**Qué significa:** el certificado SSL no es válido, es autofirmado o ha caducado.

**Solución:** renueva el certificado (Let's Encrypt se renueva automáticamente si está bien configurado).

### 5.6 La petición tarda muchísimo o nunca termina

**Causas típicas:** el servidor no responde nunca (falta `res.send()`), consulta lenta, servicio externo caído.

**Solución:** pon un tiempo máximo en el cliente (`AbortSignal.timeout(5000)`) y revisa el servidor.

---

## 6. Datos y cookies

### 6.1 El servidor recibe el cuerpo vacío

**Causas típicas:**
- Falta `Content-Type: application/json` o `JSON.stringify(...)` en el `body`.
- El servidor no tiene el middleware para leer JSON (`express.json()`).
- Envías un `FormData` y añades a mano `Content-Type` (no lo hagas: el navegador lo pone con el *boundary* correcto).

### 6.2 La cookie de sesión no se guarda o no se envía

**Causas típicas:**
- Front y API en dominios distintos sin `credentials: "include"` en `fetch` y `credentials: true` en CORS.
- Cookie con `SameSite=Strict` o sin `Secure` en HTTPS.

### 6.3 Veo datos antiguos

**Qué significa:** caché del navegador o de un CDN.

**Solución:** `Ctrl + Shift + R`; en *Network*, marca *Disable cache* mientras desarrollas.

---

## 7. Resumen

- **CORS** lo aplica el navegador y se arregla **en el servidor** (`cors()` en Express, cabeceras en PHP) o con el proxy de Vite.
- **4xx** → revisa la petición (URL, método, cabeceras, datos, token).
- **5xx** → mira los logs del servidor.
- **`ERR_CONNECTION_REFUSED`** → el servidor no está arrancado o el puerto es otro.
- **Mixed content** → todo en `https://`.
- **Cuerpo vacío** → `Content-Type` + `JSON.stringify` + `express.json()`.
- La pestaña **Network** responde casi todas las preguntas.
