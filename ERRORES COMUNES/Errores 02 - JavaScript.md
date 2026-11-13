# 02 - Errores de JavaScript

> [!info] ¿Qué es?
> Los errores de JavaScript más frecuentes, con el **mensaje literal** que muestra la consola (F12 → *Console*), qué significa, sus causas y cómo solucionarlo. La consola siempre indica el **archivo y la línea**: haz clic en ese enlace para ir directo al problema.

---

## 1. Antes de empezar: tipos de error

| Tipo | Significa |
|---|---|
| `SyntaxError` | El código está mal escrito; no se ejecuta nada del archivo |
| `ReferenceError` | Usas una variable que no existe |
| `TypeError` | Usas un valor de forma incorrecta (llamar a algo que no es función, leer de `undefined`) |
| `RangeError` | Un valor fuera de rango (recursión infinita, array de tamaño negativo) |

Más en [[JavaScript 15 - Errores y debugging|Errores y debugging]].

---

## 2. `undefined` y `null`

### 2.1 `TypeError: Cannot read properties of undefined (reading 'x')`

**Qué significa:** intentas leer `.x` de algo que vale `undefined`.

**Causas típicas:**
- El objeto aún no ha llegado (datos de una API que todavía se están cargando).
- Un error tipográfico en una propiedad anterior: `usuario.dirección.calle` cuando es `usuario.direccion`.
- `array[i]` con un índice que no existe.
- Una función que no devuelve nada (falta `return`).

**Solución:**

```javascript
console.log(usuario);                         // 1. comprueba qué hay realmente
const calle = usuario?.direccion?.calle;      // 2. encadenamiento opcional
const nombre = usuario?.nombre ?? "Anónimo";  // 3. valor por defecto
```

### 2.2 `TypeError: Cannot read properties of null (reading 'addEventListener')`

**Qué significa:** `querySelector` / `getElementById` no encontró el elemento y devolvió `null`.

**Causas típicas:**
- El script se ejecuta **antes** de que exista el HTML (está en el `<head>` sin `defer`).
- El selector está mal escrito (falta `#` o `.`, o el id no coincide).

**Solución:**

```html
<script src="main.js" defer></script>
```

```javascript
const boton = document.querySelector("#enviar");
if (!boton) console.warn("No existe #enviar");
```

### 2.3 `TypeError: Cannot set properties of undefined (setting 'x')`

**Qué significa:** igual que 2.1, pero al **asignar**.

**Solución:** crea el objeto antes: `config.tema ??= {}; config.tema.color = "azul";`.

---

## 3. Funciones y variables

### 3.1 `TypeError: x is not a function`

**Causas típicas:**
- Nombre mal escrito (`documnet.querySelector`, `arr.lenght()`).
- Llamar a métodos de array sobre algo que no es array (por ejemplo, un `NodeList` con `.map`, o un objeto).
- Una variable con el mismo nombre que la función la "tapa".
- Importación equivocada (por defecto vs. con nombre).

**Solución:**

```javascript
const items = [...document.querySelectorAll(".item")];  // NodeList → array
items.map((el) => el.textContent);
```

### 3.2 `ReferenceError: x is not defined`

**Causas típicas:**
- Error tipográfico en el nombre.
- La variable está declarada en otro ámbito (dentro de una función o de un bloque `{}`).
- Falta importar el módulo o cargar el script.

### 3.3 `ReferenceError: Cannot access 'x' before initialization`

**Qué significa:** usas una variable `let`/`const` antes de la línea donde se declara.

**Solución:** mueve la declaración arriba.

### 3.4 `TypeError: Assignment to constant variable.`

**Qué significa:** intentas reasignar una `const`.

**Solución:** usa `let` si el valor debe cambiar. (Modificar propiedades de un objeto `const` sí está permitido.)

---

## 4. Sintaxis

### 4.1 `SyntaxError: Unexpected token`

**Causas típicas:** falta o sobra un paréntesis, llave, corchete o coma; o comillas sin cerrar.

**Solución:** ve a la línea indicada **y a la anterior**. Un formateador (Prettier) delata el bloque mal cerrado.

### 4.2 `SyntaxError: Unexpected token '<', "<!DOCTYPE "... is not valid JSON`

**Qué significa:** esperabas JSON y llegó **HTML** (normalmente una página de error 404 o 500).

**Causas típicas:** URL de la API incorrecta, el servidor devolvió un error, o la ruta no existe.

**Solución:** revisa en *Network* qué respondió el servidor y comprueba `res.ok` antes de `res.json()`.

```javascript
const res = await fetch(url);
if (!res.ok) throw new Error(`Error ${res.status}`);
const datos = await res.json();
```

### 4.3 `SyntaxError: Cannot use import statement outside a module`

**Solución:** en el navegador, `<script type="module" src="main.js"></script>`. En Node, ver [[Errores 07 - Node.js y npm]].

### 4.4 `SyntaxError: await is only valid in async functions`

**Solución:** marca la función como `async`, o usa un módulo (`type="module"`), donde se permite `await` en el nivel superior.

---

## 5. Asincronía

### 5.1 Obtengo `Promise { <pending> }` en lugar del dato

**Qué significa:** falta `await`.

```javascript
const datos = obtenerDatos();        // ✘ una promesa
const datos = await obtenerDatos();  // ✔ el dato
```

### 5.2 `Uncaught (in promise) ...`

**Qué significa:** una promesa falló y nadie capturó el error.

**Solución:** `try/catch` alrededor del `await`, o `.catch()`.

### 5.3 El código "se ejecuta en desorden"

**Causa típica:** código que depende de un dato asíncrono está escrito fuera del `await`/`.then`.

### 5.4 `forEach` con `async` no espera

**Solución:** usa `for...of` o `Promise.all(lista.map(async ...))`.

Más en [[JavaScript 12 - Asincronía|Asincronía]].

---

## 6. Resultados inesperados (sin error)

### 6.1 `"5" + 3` da `"53"`

**Qué significa:** el `+` con un texto concatena. Los valores de los inputs **siempre son texto**.

**Solución:** `Number(input.value) + 3` o `input.valueAsNumber`.

### 6.2 `0.1 + 0.2 === 0.3` es `false`

**Qué significa:** imprecisión de los decimales en binario.

**Solución:** redondea al mostrar (`toFixed(2)`) o trabaja con céntimos (enteros).

### 6.3 `NaN`

**Causa típica:** operación matemática con algo que no es número (`undefined * 2`, `Number("abc")`).

**Solución:** valida antes; comprueba con `Number.isNaN(x)`.

### 6.4 `[object Object]` en pantalla

**Qué significa:** conviertes un objeto en texto.

**Solución:** muestra una propiedad (`usuario.nombre`) o usa `JSON.stringify(objeto)` para depurar.

### 6.5 Comparar objetos o arrays con `===` da `false`

**Qué significa:** `===` compara **referencias**, no contenido.

### 6.6 El evento se ejecuta al cargar, no al hacer clic

```javascript
boton.addEventListener("click", guardar());   // ✘ ejecuta guardar() ya
boton.addEventListener("click", guardar);     // ✔ pasa la función
boton.addEventListener("click", () => guardar(id)); // ✔ con argumentos
```

### 6.7 El formulario recarga la página

**Solución:** `evento.preventDefault()` en el `submit`.

---

## 7. Otros

### 7.1 `RangeError: Maximum call stack size exceeded`

**Qué significa:** recursión infinita (una función que se llama a sí misma sin fin), o un setter que se llama a sí mismo.

**Solución:** revisa la condición de parada.

### 7.2 `QuotaExceededError` en localStorage

**Qué significa:** se superó el límite (~5 MB). Guarda menos datos o usa IndexedDB.

### 7.3 Errores de CORS

Ver [[Errores 09 - Red y CORS]].

---

## 8. Resumen

- `Cannot read properties of undefined/null` → el valor no existe aún o el selector no encontró nada: usa `?.`, `defer` y comprueba con `console.log`.
- `is not a function` → nombre mal escrito o el valor no es lo que crees.
- `Unexpected token '<'` → llegó HTML en lugar de JSON: revisa la URL y `res.ok`.
- `Promise { <pending> }` → falta `await`.
- `"5" + 3 = "53"` → los inputs son texto: convierte con `Number()`.
- `addEventListener("click", f())` ejecuta al momento: pasa `f` sin paréntesis.
