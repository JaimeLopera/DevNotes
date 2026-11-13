# 00 - Índice

> [!info] Errores comunes
> Recopilación de los **mensajes de error más frecuentes** en desarrollo web, con **qué significan**, **por qué ocurren** y **cómo se solucionan**. Está pensada para consultarla cuando algo falla: copia el mensaje de error y búscalo aquí.

---

## 1. Cómo usar esta carpeta

1. **Copia la parte clave del mensaje** (por ejemplo, `Cannot read properties of undefined`).
2. Búscala en Obsidian con `Ctrl + Shift + F`.
3. Lee la causa y aplica la solución.
4. Si no está, añádela cuando la resuelvas (ver formato abajo).

---

## 2. Notas de la carpeta

| Nota | Errores de… |
|---|---|
| [[Errores 01 - HTML y CSS]] | Estilos que no se aplican, layouts rotos, imágenes que no cargan |
| [[Errores 02 - JavaScript]] | `undefined`, `null`, `is not a function`, promesas, DOM |
| [[Errores 03 - PHP]] | Variables indefinidas, cabeceras, conexión a BD, sesiones |
| [[Errores 04 - Python]] | Indentación, módulos, tipos, claves, codificación |
| [[Errores 05 - SQL]] | Sintaxis, claves duplicadas, claves foráneas, GROUP BY |
| [[Errores 06 - Git]] | Push rechazado, conflictos, autenticación, detached HEAD |
| [[Errores 07 - Node.js y npm]] | Módulos, dependencias, puertos, ES Modules |
| [[Errores 08 - Docker]] | Daemon, puertos, permisos, contenedores que se cierran |
| [[Errores 09 - Red y CORS]] | CORS, 404, 500, mixed content, timeouts |

---

## 3. Método general para cualquier error

| Paso | Qué hacer |
|---|---|
| 1. **Lee el mensaje entero** | Dice qué pasa y, casi siempre, en qué archivo y línea |
| 2. **Ve a la línea indicada** | El error está ahí o justo antes |
| 3. **Reprodúcelo** | Si no sabes provocarlo, no sabrás si lo has arreglado |
| 4. **Aísla** | Comenta código o usa `console.log` / `print` hasta encontrar el punto exacto |
| 5. **Cambia una cosa cada vez** | Si cambias cinco, no sabrás cuál lo arregló |
| 6. **Busca el mensaje literal** | Aquí, en la documentación oficial o en internet |
| 7. **Explícalo en voz alta** | Contar el problema paso a paso (*rubber duck*) suele revelar el fallo |

> [!tip] Dónde mirar
> - **Navegador**: F12 → pestañas *Console* (errores JS) y *Network* (peticiones).
> - **Servidor**: la terminal donde corre Node/PHP/Python, o los logs.
> - **Base de datos**: el mensaje que devuelve la consulta.

---

## 4. Formato de cada error

Todas las notas siguen el mismo formato para que se lean igual:

````markdown
### N.N `Mensaje de error literal`

**Qué significa:** explicación en una frase.

**Causas típicas:**
- Causa 1
- Causa 2

**Solución:**

```lenguaje
código corregido
```
````

---

## 5. Notas relacionadas en otras carpetas

- [[Snippets 00 - Índice|Snippets]] → código correcto listo para copiar.
- [[JavaScript 15 - Errores y debugging|Errores y debugging en JavaScript]] → cómo depurar a fondo.
- [[Python 18 - Debugging y buenas prácticas]] (Python) → depuración en Python.
- [[Testing 00 - Índice|Testing]] → que el error no vuelva a aparecer.

---

## 6. Resumen

- Busca el mensaje literal del error en esta carpeta.
- Cada error tiene: qué significa, causas y solución.
- Método: leer → ir a la línea → reproducir → aislar → cambiar una cosa cada vez.
- Cuando resuelvas un error nuevo, añádelo aquí.
