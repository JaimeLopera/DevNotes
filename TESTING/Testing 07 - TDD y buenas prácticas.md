# 07 - TDD y buenas prácticas

> [!info] ¿Qué es?
> **TDD** (*Test-Driven Development*, desarrollo guiado por tests) es una forma de trabajar en la que **escribes el test antes que el código**. Esta nota también recoge las buenas prácticas generales de testing: qué testear, cómo nombrar, qué significa la cobertura y cómo automatizarlo todo.

---

## 1. Antes de empezar

Conviene haber practicado con [[Testing 02 - Tests unitarios con Vitest]] o [[Testing 04 - Tests en Python con pytest]].

---

## 2. Concepto fundamental

El ciclo de TDD tiene tres pasos que se repiten en ciclos de pocos minutos:

```
   ┌──────────────┐
   │  1. ROJO     │  Escribe un test pequeño que FALLA
   └──────┬───────┘  (la funcionalidad aún no existe)
          ▼
   ┌──────────────┐
   │  2. VERDE    │  Escribe el código MÍNIMO para que pase
   └──────┬───────┘
          ▼
   ┌──────────────┐
   │ 3. REFACTOR  │  Mejora el código sin cambiar lo que hace
   └──────┬───────┘  (los tests te protegen)
          │
          └──────► vuelve al paso 1 con el siguiente caso
```

Ventajas: diseñas pensando en **cómo se usa** el código, nunca te queda código sin probar y avanzas en pasos pequeños y seguros.

---

## 3. Sintaxis / estructura

Un ciclo completo para `esPalindromo`:

```javascript
// 1. ROJO: el test falla (la función no existe)
it("detecta un palíndromo simple", () => {
  expect(esPalindromo("ana")).toBe(true);
});

// 2. VERDE: lo mínimo para pasar
export function esPalindromo(texto) {
  return texto === texto.split("").reverse().join("");
}

// 1. ROJO: siguiente caso
it("ignora mayúsculas y espacios", () => {
  expect(esPalindromo("Anita lava la tina")).toBe(true);
});

// 2. VERDE
export function esPalindromo(texto) {
  const limpio = texto.toLowerCase().replaceAll(" ", "");
  return limpio === [...limpio].reverse().join("");
}

// 3. REFACTOR: el código ya es claro; los dos tests siguen en verde
```

---

## 4. Elementos / propiedades / características

### 4.1 Qué testear (y qué no)

| Sí | No |
|---|---|
| Lógica de negocio (precios, descuentos, permisos) | Código de librerías externas |
| Validaciones | *Getters* y *setters* triviales |
| Casos límite y errores | Detalles internos que pueden cambiar |
| Bugs ya encontrados (para que no vuelvan) | Estilos visuales concretos |
| Endpoints de la API | Configuración |

### 4.2 Cómo nombrar los tests

Un buen nombre dice **qué hace** y **en qué caso**:

| Mal | Bien |
|---|---|
| `test 1` | `calcula el IVA general del 21 %` |
| `funciona` | `rechaza emails sin dominio` |
| `carrito` | `vacía el carrito tras confirmar el pedido` |

### 4.3 Cobertura (*coverage*)

Indica qué porcentaje del código se ejecuta durante los tests:

| Métrica | Mide |
|---|---|
| Líneas (*lines*) | Líneas ejecutadas |
| Ramas (*branches*) | Caminos de `if`/`else`, `? :` recorridos |
| Funciones | Funciones llamadas al menos una vez |

```bash
npx vitest run --coverage
pytest --cov=src --cov-report=html
```

> [!warning] La cobertura no mide la calidad
> Un 100 % de cobertura solo dice que el código **se ejecutó**, no que se **comprobó** bien. Úsala para encontrar zonas sin tests, no como objetivo en sí. Un 70–80 % bien pensado vale más que un 100 % de tests vacíos.

### 4.4 Principios FIRST

| Letra | Principio | Significa |
|---|---|---|
| **F** | *Fast* | Rápidos |
| **I** | *Independent* | No dependen unos de otros |
| **R** | *Repeatable* | Mismo resultado siempre y en cualquier equipo |
| **S** | *Self-validating* | Pasan o fallan solos, sin revisar nada a mano |
| **T** | *Timely* | Se escriben a tiempo (antes o junto con el código) |

### 4.5 Integración continua (CI)

Ejecutar los tests automáticamente en cada push con GitHub Actions:

```yaml
# .github/workflows/tests.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npm run test:run
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: TDD de una validación de contraseña

```javascript
describe("validarPassword", () => {
  it("rechaza contraseñas de menos de 8 caracteres", () => {
    expect(validarPassword("abc12")).toEqual({ valida: false, error: "Mínimo 8 caracteres" });
  });

  it("exige al menos un número", () => {
    expect(validarPassword("abcdefgh")).toEqual({ valida: false, error: "Debe incluir un número" });
  });

  it("acepta una contraseña correcta", () => {
    expect(validarPassword("abcdefg1")).toEqual({ valida: true });
  });
});
```

```javascript
export function validarPassword(password) {
  if (password.length < 8) return { valida: false, error: "Mínimo 8 caracteres" };
  if (!/\d/.test(password)) return { valida: false, error: "Debe incluir un número" };
  return { valida: true };
}
```

### 5.2 Ejemplo habitual: arreglar un bug con un test

1. Te reportan: "el total sale negativo con un cupón del 100 %".
2. Escribes el test que lo reproduce (rojo):

```javascript
it("el total nunca es negativo con cupones grandes", () => {
  expect(totalConCupon(50, { tipo: "fijo", valor: 80 })).toBe(0);
});
```

3. Lo arreglas (verde) con `Math.max(0, ...)`.
4. El bug ya no puede volver sin que un test lo detecte.

---

## 6. Buenas prácticas

- **Un comportamiento por test** y nombres que lo describan.
- **Escribe el test de un bug antes de arreglarlo**.
- **Tests rápidos**: la mayoría unitarios, pocos E2E (pirámide de [[Testing 01 - Fundamentos]]).
- **No testees la implementación**: si cambias el *cómo* sin cambiar el *qué*, los tests no deberían romperse.
- **Los tests también son código**: mantenlos limpios, sin duplicaciones absurdas.
- **Ejecútalos en CI** y no fusiones nada en rojo.
- **Un test que falla a veces (*flaky*) se arregla o se borra**: si se ignora, se pierde la confianza en todos.

---

## 7. Diferencias importantes

### 7.1 TDD vs. tests después

| | TDD | Tests después |
|---|---|---|
| Cuándo se escribe el test | Antes del código | Después |
| Diseño | Guiado por el uso | Guiado por la implementación |
| Riesgo de saltarse tests | Bajo | Alto ("ya lo testearé") |
| Curva de aprendizaje | Mayor | Menor |

No hace falta hacer TDD siempre: es muy útil en lógica compleja y al arreglar bugs.

### 7.2 Cobertura de líneas vs. de ramas

Con `if (a && b)`, puedes ejecutar la línea (100 % de líneas) sin probar el caso en que `b` es falso. La cobertura de **ramas** es más exigente y más útil.

### 7.3 BDD

*Behavior-Driven Development*: como TDD, pero escribiendo los tests en lenguaje casi natural ("Dado… cuando… entonces…"). La estructura `describe`/`it` ya sigue esa idea.

---

## 8. Casos especiales

### 8.1 Código heredado sin tests

Antes de cambiarlo, escribe **tests de caracterización**: comprueban lo que hace **ahora** (aunque sea raro). Así sabrás si tu cambio altera algo.

### 8.2 Código difícil de testear

Suele indicar que hace demasiado o depende de cosas globales. Extrae la lógica a funciones puras y pasa las dependencias como parámetros ([[Testing 03 - Mocks y espías]]).

### 8.3 Snapshots

Guardan la salida (por ejemplo, un HTML) y avisan si cambia. Útiles con moderación: es fácil aceptar cambios sin revisarlos.

---

## 9. Resumen

- **TDD**: rojo (test que falla) → verde (código mínimo) → refactor.
- Testea lógica, validaciones, límites y bugs; no librerías ni trivialidades.
- Nombres que describen comportamiento y caso.
- La **cobertura** encuentra huecos, pero no mide calidad.
- **FIRST**: rápidos, independientes, repetibles, autovalidados, a tiempo.
- Automatiza con CI: nada se fusiona en rojo.
