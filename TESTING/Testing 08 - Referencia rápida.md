# 08 - Referencia rápida

> [!info] ¿Qué es?
> Chuleta de **Testing** para consultar rápido. No explica, solo recuerda. Si algo no te suena, sigue el enlace a su nota.

---

## 1. Conceptos ([[Testing 01 - Fundamentos]])

| Tipo | Prueba | Cuántos |
|---|---|---|
| Unitario | Una función | Muchos |
| Integración | Varias piezas | Bastantes |
| E2E | La app en un navegador | Pocos |

Estructura **AAA**: Arrange (preparar) → Act (actuar) → Assert (comprobar).

---

## 2. Vitest ([[Testing 02 - Tests unitarios con Vitest]])

```bash
npm i -D vitest
npx vitest            # watch
npx vitest run        # una vez
npx vitest run --coverage
```

```javascript
import { describe, it, expect, beforeEach } from "vitest";

describe("grupo", () => {
  beforeEach(() => { /* preparar */ });
  it("caso", () => { expect(real).toBe(esperado); });
  it("async", async () => { await expect(p).rejects.toThrow(); });
  it.each([[1, 2], [3, 4]])("con %i da %i", (a, b) => { ... });
});
```

| Matcher | Uso |
|---|---|
| `toBe` | Primitivos |
| `toEqual` | Objetos/arrays |
| `toMatchObject` | Contiene propiedades |
| `toBeCloseTo` | Decimales |
| `toContain` | Elemento en array/texto |
| `toHaveLength` | Longitud |
| `toThrow` | Lanza error (pasar función) |
| `.not` | Negar |

---

## 3. Mocks ([[Testing 03 - Mocks y espías]])

```javascript
const fn = vi.fn().mockReturnValue(1);
vi.fn().mockResolvedValue(datos);
vi.fn().mockRejectedValue(new Error("x"));
vi.spyOn(globalThis, "fetch").mockResolvedValue({ json: async () => ({}) });
vi.mock("./modulo.js", () => ({ f: vi.fn() }));
vi.useFakeTimers(); vi.setSystemTime(new Date("2026-01-01"));
vi.restoreAllMocks();

expect(fn).toHaveBeenCalledTimes(1);
expect(fn).toHaveBeenCalledWith("a", 2);
```

---

## 4. pytest ([[Testing 04 - Tests en Python con pytest]])

```bash
pip install pytest pytest-cov
pytest -v  |  -x  |  -k "nombre"  |  --lf  |  -s
pytest --cov=src
```

```python
import pytest

@pytest.fixture
def datos():
    return {"a": 1}

def test_algo(datos, tmp_path, monkeypatch):
    monkeypatch.setenv("X", "1")
    assert datos["a"] == 1
    assert 0.1 + 0.2 == pytest.approx(0.3)
    with pytest.raises(ValueError, match="texto"):
        funcion()

@pytest.mark.parametrize("a, b", [(1, 2), (3, 4)])
def test_varios(a, b): ...
```

---

## 5. Tests de API ([[Testing 05 - Tests de integración y API]])

```javascript
import request from "supertest";
import { app } from "../src/app.js";   // app sin listen

const res = await request(app)
  .post("/api/productos")
  .set("Authorization", "Bearer token")
  .send({ nombre: "Taza", precio: 8 });

expect(res.status).toBe(201);
expect(res.body).toMatchObject({ nombre: "Taza" });
```

Probar: éxito, `400`, `404`, `401`/`403`.

---

## 6. Playwright ([[Testing 06 - Tests end-to-end con Playwright]])

```bash
npm init playwright@latest
npx playwright test  |  --ui  |  show-report
npx playwright codegen URL
```

```javascript
import { test, expect } from "@playwright/test";

test("flujo", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill("a@a.com");
  await page.getByRole("button", { name: "Entrar" }).click();
  await expect(page).toHaveURL("/panel");
  await expect(page.getByText("Hola")).toBeVisible();
});
```

---

## 7. TDD y CI ([[Testing 07 - TDD y buenas prácticas]])

Rojo → Verde → Refactor.

```yaml
# .github/workflows/tests.yml
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: npm ci
      - run: npx vitest run
```
