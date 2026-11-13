# 07 - Asincronía y event loop en Node

> [!info] ¿Qué es?
> Node ejecuta tu JavaScript en **un solo hilo**, pero puede atender miles de peticiones a la vez. Lo consigue con el **event loop**: mientras espera algo lento (leer un archivo, consultar una base de datos, una petición a otra API), sigue atendiendo otras tareas en lugar de quedarse parado.

---

## 1. Antes de empezar

Hay que dominar promesas y `async`/`await`: [[JavaScript 12 - Asincronía|Asincronía en JavaScript]]. Esta nota explica lo **propio de Node**.

---

## 2. Concepto fundamental

Imagina un camarero (el hilo de Node) en un restaurante:

```
Camarero bloqueante:  toma pedido 1 → espera en cocina 10 min → sirve → toma pedido 2…
Camarero de Node:     toma pedido 1 → lo pasa a cocina → toma pedido 2 → toma pedido 3
                      → "¡el 1 está listo!" → lo sirve → …
```

- La **cocina** son el sistema operativo y una pequeña reserva de hilos (*libuv*), que hacen el trabajo lento (disco, red).
- El **event loop** avisa al camarero cuando algo está listo y ejecuta la función que tenía pendiente (el *callback* o la continuación del `await`).

**Consecuencia clave:** el código asíncrono (E/S) escala muy bien; el código que **calcula mucho** sin parar **bloquea** a todos.

---

## 3. Sintaxis / estructura

```javascript
import fs from "node:fs/promises";

console.log("1. Inicio");

const promesa = fs.readFile("datos.txt", "utf-8"); // se pide al sistema y se sigue

console.log("2. Mientras tanto, sigo trabajando");

const texto = await promesa;
console.log("3. Archivo leído:", texto.length, "caracteres");
```

---

## 4. Elementos / propiedades / características

### 4.1 Orden de ejecución

Node procesa, por prioridad:

| Orden | Qué | Ejemplos |
|---|---|---|
| 1 | Código síncrono | `console.log`, bucles, cálculos |
| 2 | Microtareas | `process.nextTick`, promesas resueltas (`.then`, continuación de `await`) |
| 3 | Temporizadores | `setTimeout`, `setInterval` |
| 4 | E/S terminada | Lecturas de disco, respuestas de red |
| 5 | `setImmediate` | Justo después de la E/S |

```javascript
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promesa"));
console.log("síncrono");

// síncrono
// promesa
// timeout
```

### 4.2 Ejecutar en paralelo con `Promise.all`

```javascript
// Secuencial: ~3 segundos si cada una tarda 1 s
const a = await obtenerUsuario();
const b = await obtenerPedidos();
const c = await obtenerFacturas();

// En paralelo: ~1 segundo
const [usuario, pedidos, facturas] = await Promise.all([
  obtenerUsuario(),
  obtenerPedidos(),
  obtenerFacturas(),
]);
```

| Método | Termina cuando… |
|---|---|
| `Promise.all` | Todas se cumplen (o falla la primera) |
| `Promise.allSettled` | Todas terminan, bien o mal |
| `Promise.race` | Termina la primera |
| `Promise.any` | Se cumple la primera |

### 4.3 Eventos: `EventEmitter`

Muchas partes de Node funcionan con eventos (servidores, streams):

```javascript
import { EventEmitter } from "node:events";

const tienda = new EventEmitter();
tienda.on("pedido", (id) => console.log(`Nuevo pedido ${id}`));
tienda.emit("pedido", 42);
```

### 4.4 Temporizadores con promesas

```javascript
import { setTimeout as esperar } from "node:timers/promises";
await esperar(1000);   // espera 1 segundo
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: leer varios archivos a la vez

```javascript
import fs from "node:fs/promises";

const nombres = ["a.txt", "b.txt", "c.txt"];
const contenidos = await Promise.all(
  nombres.map((n) => fs.readFile(n, "utf-8"))
);
console.log(contenidos);
```

### 5.2 Ejemplo habitual: el peligro de bloquear

```javascript
app.get("/rapida", (req, res) => res.send("OK"));

app.get("/lenta", (req, res) => {
  // Cálculo de 5 segundos: BLOQUEA a todos los usuarios
  const fin = Date.now() + 5000;
  while (Date.now() < fin) {}
  res.send("Terminé");
});
```

Mientras alguien llama a `/lenta`, **nadie** recibe respuesta de `/rapida`. Las tareas de cálculo pesado deben ir a *worker threads*, a otro servicio o a una cola de trabajos.

### 5.3 Manejar errores asíncronos

```javascript
try {
  const datos = await fetch("https://api.ejemplo.com/datos").then((r) => r.json());
} catch (error) {
  console.error("Falló la petición:", error.message);
}
```

---

## 6. Buenas prácticas

- **Usa `async`/`await`** en lugar de callbacks anidados.
- **Lanza en paralelo** (`Promise.all`) lo que no depende entre sí.
- **No bloquees el event loop**: nada de bucles de cálculo largos ni funciones `Sync` en un servidor.
- **Captura siempre los errores** con `try/catch`: una promesa rechazada sin capturar puede tumbar el proceso.
- **No olvides el `await`**: sin él, recibes una promesa en lugar del valor.

---

## 7. Diferencias importantes

### 7.1 Asíncrono vs. paralelo

| | Asíncrono (event loop) | Paralelo (worker threads) |
|---|---|---|
| Hilos de tu JavaScript | Uno | Varios |
| Bueno para | Esperar E/S (red, disco, BD) | Cálculos pesados (CPU) |
| Uso habitual en web | Casi todo | Casos concretos |

### 7.2 `process.nextTick` vs. `setImmediate` vs. `setTimeout(fn, 0)`

`nextTick` se ejecuta **antes** que cualquier otra cosa pendiente; `setTimeout(0)` en la fase de temporizadores; `setImmediate` tras la E/S. En el día a día casi nunca hacen falta.

### 7.3 Secuencial vs. `Promise.all`

Usa secuencial (`await` uno tras otro) solo si cada paso **necesita** el resultado del anterior.

---

## 8. Casos especiales

### 8.1 *UnhandledPromiseRejection*

Una promesa falló y nadie capturó el error. Añade `try/catch` alrededor del `await` o `.catch()` a la promesa.

### 8.2 `await` dentro de `forEach`

`forEach` **no espera** a las funciones `async`. Usa `for...of` (secuencial) o `Promise.all` con `map` (paralelo).

```javascript
for (const id of ids) {
  await procesar(id);
}
```

### 8.3 Demasiadas tareas en paralelo

`Promise.all` con miles de peticiones puede saturar una API o la base de datos. Procésalas por lotes.

---

## 9. Resumen

- Node ejecuta JavaScript en **un solo hilo** con un **event loop**.
- La E/S (disco, red, BD) se delega y no bloquea; los cálculos largos sí bloquean.
- Orden: síncrono → microtareas (promesas) → temporizadores → E/S.
- `Promise.all` para tareas independientes en paralelo.
- Captura siempre los errores de las promesas.
- No uses `await` dentro de `forEach`.
