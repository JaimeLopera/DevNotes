# 06 - Tests end-to-end con Playwright

> [!info] ¿Qué es?
> Un **test end-to-end (E2E)** prueba la aplicación **de principio a fin, como lo haría un usuario**: abre un navegador real, visita la web, hace clic, rellena formularios y comprueba lo que aparece en pantalla. **Playwright** es la herramienta más usada hoy para escribirlos.

---

## 1. Antes de empezar

- Conviene haber leído [[Testing 01 - Fundamentos]] y [[Testing 02 - Tests unitarios con Vitest]].
- Ayuda saber HTML y accesibilidad (roles, etiquetas): [[HTML 08 - Accesibilidad|Accesibilidad]].

---

## 2. Concepto fundamental

```
Test de Playwright
   │
   ├─► abre Chromium / Firefox / WebKit
   ├─► va a http://localhost:5173
   ├─► rellena "Email" y "Contraseña"
   ├─► pulsa "Entrar"
   └─► comprueba que aparece "Hola, Ana"
```

Prueba **todo junto**: HTML, CSS, JavaScript, la API y la base de datos. Es lo más parecido a un usuario real, pero también lo más lento: por eso se usan **pocos** E2E, para los flujos críticos (registro, login, compra).

---

## 3. Sintaxis / estructura

```bash
npm init playwright@latest   # instala Playwright, navegadores y un ejemplo
```

```javascript
// tests/inicio.spec.js
import { test, expect } from "@playwright/test";

test("la página de inicio muestra el título", async ({ page }) => {
  await page.goto("http://localhost:5173");
  await expect(page.getByRole("heading", { name: "Mi tienda" })).toBeVisible();
});
```

```bash
npx playwright test            # ejecuta todos
npx playwright test --ui       # interfaz visual paso a paso
npx playwright show-report     # informe HTML con capturas
```

---

## 4. Elementos / propiedades / características

### 4.1 Localizar elementos (*locators*)

| Locator | Busca por | Ejemplo |
|---|---|---|
| `getByRole` | Rol accesible + nombre (**recomendado**) | `page.getByRole("button", { name: "Entrar" })` |
| `getByLabel` | Etiqueta de un campo | `page.getByLabel("Email")` |
| `getByPlaceholder` | Placeholder | `page.getByPlaceholder("Buscar…")` |
| `getByText` | Texto visible | `page.getByText("Pedido confirmado")` |
| `getByTestId` | Atributo `data-testid` | `page.getByTestId("carrito-total")` |
| `locator(css)` | Selector CSS (último recurso) | `page.locator(".precio")` |

### 4.2 Acciones

| Acción | Ejemplo |
|---|---|
| Ir a una URL | `await page.goto("/login")` |
| Clic | `await boton.click()` |
| Escribir | `await campo.fill("ana@mail.com")` |
| Pulsar tecla | `await campo.press("Enter")` |
| Marcar | `await casilla.check()` |
| Elegir opción | `await select.selectOption("ES")` |
| Pasar el ratón | `await menu.hover()` |
| Subir archivo | `await input.setInputFiles("foto.png")` |

### 4.3 Comprobaciones (con espera automática)

| Aserción | Comprueba |
|---|---|
| `toBeVisible()` | Está visible |
| `toHaveText("...")` | Tiene ese texto |
| `toContainText("...")` | Contiene ese texto |
| `toHaveValue("...")` | Valor de un campo |
| `toHaveCount(n)` | Número de elementos |
| `toBeEnabled()` / `toBeDisabled()` | Activo / desactivado |
| `expect(page).toHaveURL(/dashboard/)` | La URL actual |
| `expect(page).toHaveTitle("...")` | El `<title>` |

Playwright **espera automáticamente** (por defecto hasta 5 s) a que la condición se cumpla: no hace falta poner pausas.

### 4.4 Configuración (`playwright.config.js`)

```javascript
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "tests/e2e",
  use: {
    baseURL: "http://localhost:5173",   // permite page.goto("/login")
    screenshot: "only-on-failure",
    trace: "on-first-retry",
  },
  webServer: {
    command: "npm run dev",             // arranca la web antes de los tests
    url: "http://localhost:5173",
    reuseExistingServer: true,
  },
});
```

### 4.5 Grabar tests

```bash
npx playwright codegen http://localhost:5173
```

Abre un navegador y **genera el código** mientras haces clic. Útil para empezar, aunque luego conviene limpiarlo.

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: buscador

```javascript
import { test, expect } from "@playwright/test";

test("buscar un producto muestra resultados", async ({ page }) => {
  await page.goto("/");
  await page.getByPlaceholder("Buscar…").fill("taza");
  await page.getByPlaceholder("Buscar…").press("Enter");

  await expect(page).toHaveURL(/q=taza/);
  await expect(page.getByRole("listitem")).toHaveCount(3);
});
```

### 5.2 Ejemplo habitual: inicio de sesión

```javascript
import { test, expect } from "@playwright/test";

test.describe("Inicio de sesión", () => {
  test("con datos correctos entra al panel", async ({ page }) => {
    await page.goto("/login");
    await page.getByLabel("Email").fill("ana@mail.com");
    await page.getByLabel("Contraseña").fill("secreto123");
    await page.getByRole("button", { name: "Entrar" }).click();

    await expect(page).toHaveURL("/panel");
    await expect(page.getByRole("heading", { name: "Hola, Ana" })).toBeVisible();
  });

  test("con contraseña incorrecta muestra un error", async ({ page }) => {
    await page.goto("/login");
    await page.getByLabel("Email").fill("ana@mail.com");
    await page.getByLabel("Contraseña").fill("mala");
    await page.getByRole("button", { name: "Entrar" }).click();

    await expect(page.getByRole("alert")).toContainText("Credenciales incorrectas");
  });
});
```

---

## 6. Buenas prácticas

- **Pocos E2E y para flujos críticos**: registro, login, pago, lo que da dinero o no puede fallar.
- **Localiza por rol y etiqueta** (`getByRole`, `getByLabel`): son estables y además comprueban la accesibilidad.
- **Nunca uses pausas fijas** (`waitForTimeout`): confía en la espera automática de las aserciones.
- **Tests independientes**: cada uno prepara sus datos (usuario de prueba, sesión).
- **Usa `baseURL` y `webServer`** en la configuración.
- **Revisa trazas y capturas** cuando un test falle en CI (`show-report`).

---

## 7. Diferencias importantes

### 7.1 E2E vs. integración vs. unitario

| | Unitario | Integración | E2E |
|---|---|---|---|
| Navegador real | No | No | Sí |
| Velocidad | ms | ~100 ms | segundos |
| Cuántos | Muchos | Bastantes | Pocos |
| Si falla, ¿dónde está el error? | Exacto | Bastante claro | Hay que investigar |

### 7.2 Playwright vs. Cypress

| | Playwright | Cypress |
|---|---|---|
| Navegadores | Chromium, Firefox, WebKit (Safari) | Chromium, Firefox |
| Varias pestañas/dominios | Sí | Limitado |
| Lenguajes | JS/TS, Python, Java, .NET | JS/TS |
| Velocidad en paralelo | Muy buena | Buena |

### 7.3 `fill` vs. `type`

`fill` pone el valor de golpe (rápido, lo habitual). `pressSequentially` escribe tecla a tecla (solo si la web reacciona a cada pulsación).

---

## 8. Casos especiales

### 8.1 Iniciar sesión una sola vez

Para no repetir el login en cada test, Playwright puede guardar la sesión (`storageState`) en un paso de preparación y reutilizarla.

### 8.2 Simular la API

```javascript
await page.route("**/api/productos", (route) =>
  route.fulfill({ json: [{ id: 1, nombre: "Taza" }] })
);
```

Útil para probar la interfaz sin depender del backend.

### 8.3 Móvil y varios navegadores

En `playwright.config.js` se definen **proyectos** (`Desktop Chrome`, `iPhone 13`…) y cada test se ejecuta en todos.

### 8.4 Playwright en Python

```python
def test_titulo(page):
    page.goto("https://ejemplo.com")
    expect(page).to_have_title("Ejemplo")
```

(`pip install pytest-playwright` y `playwright install`.)

---

## 9. Resumen

- Los E2E prueban la app completa en un navegador real, como un usuario.
- `npm init playwright@latest` instala todo.
- Localiza con `getByRole`, `getByLabel`, `getByText`.
- Acciones: `goto`, `click`, `fill`, `press`; aserciones con espera automática.
- `webServer` arranca la web; `codegen` graba tests.
- Pocos E2E, solo para los flujos críticos.
- Siguiente paso: [[Testing 07 - TDD y buenas prácticas]].
