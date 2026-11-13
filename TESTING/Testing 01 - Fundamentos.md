# 01 - Fundamentos

> [!info] ¿Qué es?
> Un **test automático** es una función que ejecuta un trozo de tu código con unos datos concretos y **comprueba** que el resultado es el esperado. Si lo es, el test **pasa** (verde); si no, **falla** (rojo) y te dice qué esperaba y qué obtuvo.

---

## 1. Antes de empezar

No hace falta conocer ninguna herramienta. Solo saber escribir funciones en tu lenguaje.

---

## 2. Concepto fundamental

Probar a mano vs. probar con un test:

```
A mano:   cambias código → abres el navegador → rellenas el formulario → miras el resultado
          (y lo repites con cada cambio… y te olvidas de algún caso)

Con test: cambias código → npm test → ✔ 48 tests pasados en 1,2 s
```

Todo test sigue la estructura **AAA**:

| Paso | Qué haces | Ejemplo |
|---|---|---|
| **Arrange** (preparar) | Creas los datos de partida | `const carrito = [{ precio: 10, cantidad: 2 }]` |
| **Act** (actuar) | Ejecutas lo que quieres probar | `const total = calcularTotal(carrito)` |
| **Assert** (comprobar) | Compruebas el resultado | `expect(total).toBe(20)` |

---

## 3. Sintaxis / estructura

La forma general (Vitest/Jest; pytest es muy parecido):

```javascript
import { describe, it, expect } from "vitest";
import { calcularTotal } from "./carrito.js";

describe("calcularTotal", () => {            // agrupa tests relacionados
  it("suma precio por cantidad", () => {     // un caso concreto
    const carrito = [{ precio: 10, cantidad: 2 }];   // Arrange
    const total = calcularTotal(carrito);            // Act
    expect(total).toBe(20);                          // Assert
  });
});
```

---

## 4. Elementos / propiedades / características

### 4.1 Tipos de tests

| Tipo | Qué prueba | Velocidad | Ejemplo |
|---|---|---|---|
| **Unitario** | Una función o clase aislada | Milisegundos | `calcularIVA(100) === 21` |
| **Integración** | Varias piezas juntas | Rápido-medio | La API guarda en la base de datos |
| **End-to-end (E2E)** | La aplicación completa como un usuario | Lento | Registrarse, iniciar sesión y comprar |

### 4.2 La pirámide de tests

```
        /\
       /E2E\        pocos: lentos y frágiles, pero prueban lo real
      /──────\
     /Integra-\     bastantes
    /  ción    \
   /────────────\
  /  Unitarios   \  muchos: rápidos, baratos y precisos
 /────────────────\
```

### 4.3 Vocabulario

| Término | Significa |
|---|---|
| **Test runner** | Programa que busca y ejecuta los tests (Vitest, pytest) |
| **Aserción** | La comprobación (`expect(x).toBe(y)`, `assert x == y`) |
| **Suite** | Grupo de tests (`describe`) |
| **Fixture** | Datos o estado preparado para los tests |
| **Mock** | Sustituto falso de una dependencia ([[Testing 03 - Mocks y espías]]) |
| **Cobertura** | Porcentaje de código que ejecutan los tests |
| **Regresión** | Algo que funcionaba y deja de funcionar tras un cambio |
| **Flaky test** | Test que a veces pasa y a veces falla sin cambios |

### 4.4 Qué casos probar

Para cada función, piensa en:

| Caso | Ejemplo para `dividir(a, b)` |
|---|---|
| **Normal** | `dividir(10, 2) → 5` |
| **Límite** | `dividir(0, 5) → 0` |
| **Error** | `dividir(1, 0) → lanza error` |
| **Raro** | Números negativos, decimales, textos vacíos |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```javascript
// utils.js
export function esEmailValido(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

```javascript
// utils.test.js
import { it, expect } from "vitest";
import { esEmailValido } from "./utils.js";

it("acepta un email correcto", () => {
  expect(esEmailValido("ana@mail.com")).toBe(true);
});

it("rechaza un email sin arroba", () => {
  expect(esEmailValido("ana.mail.com")).toBe(false);
});

it("rechaza un email con espacios", () => {
  expect(esEmailValido("ana @mail.com")).toBe(false);
});
```

### 5.2 Ejemplo habitual: un test que detecta un error

```
 FAIL  carrito.test.js > calcularTotal > aplica el descuento
AssertionError: expected 18.000000000000004 to be 18

- Expected   18
+ Received   18.000000000000004
```

El test ha descubierto un problema de redondeo con decimales que a mano habría pasado desapercibido.

---

## 6. Buenas prácticas

- **Un test, una idea**: si falla, el nombre ya dice qué se ha roto.
- **Nombres descriptivos**: "rechaza un email sin arroba", no "test 3".
- **Tests independientes**: ninguno debe depender de que otro se ejecute antes.
- **Rápidos**: si tardan mucho, nadie los ejecuta.
- **Deterministas**: mismo código → mismo resultado (cuidado con fechas, azar y red).
- **Prueba el comportamiento**, no los detalles internos de la implementación.
- **Cuando encuentres un bug, escribe primero un test que lo reproduzca** y luego arréglalo.

---

## 7. Diferencias importantes

### 7.1 Unitario vs. integración vs. E2E

| | Unitario | Integración | E2E |
|---|---|---|---|
| Alcance | Una pieza | Varias piezas | Todo el sistema |
| Dependencias externas | Simuladas (mocks) | Algunas reales | Todas reales |
| Velocidad | Muy rápida | Media | Lenta |
| Si falla, sabes dónde | Exactamente | Más o menos | Hay que investigar |

### 7.2 Test manual vs. automático

El test manual sirve para explorar; el automático, para **no volver a romper** lo que ya funciona.

### 7.3 Testing vs. debugging

Testear **detecta** que algo falla. Depurar (*debugging*) **encuentra por qué** falla.

---

## 8. Casos especiales

### 8.1 Código difícil de testear

Si una función es difícil de testear, suele ser señal de que hace demasiadas cosas o depende de cosas globales. Dividirla en funciones pequeñas y **puras** (mismo resultado para la misma entrada, sin efectos secundarios) la hace fácil de probar.

### 8.2 Tests y tipos

TypeScript ([[TypeScript 00 - Índice|TypeScript]]) ya evita muchos errores de "forma" de los datos; los tests se centran en la **lógica**.

### 8.3 Ejecutarlos automáticamente

Lo ideal es ejecutar los tests en cada push con **GitHub Actions** (integración continua), para que nadie fusione código que rompe algo.

---

## 9. Resumen

- Un test ejecuta tu código y **comprueba** el resultado automáticamente.
- Estructura **AAA**: preparar, actuar, comprobar.
- Tipos: **unitarios** (muchos), **integración** (bastantes), **E2E** (pocos).
- Prueba casos normales, límites, errores y raros.
- Tests rápidos, independientes, deterministas y con nombres claros.
- Siguiente paso: [[Testing 02 - Tests unitarios con Vitest]].
