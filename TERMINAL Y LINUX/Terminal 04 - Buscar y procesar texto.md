# 04 - Buscar y procesar texto

> [!info] ¿Qué es?
> Herramientas para **encontrar archivos**, **buscar texto** dentro de ellos y **transformarlo**. Combinadas con tuberías (`|`) y redirecciones (`>`), permiten hacer en una línea lo que a mano costaría minutos.

---

## 1. Antes de empezar

- Conviene haber leído [[Terminal 02 - Navegación y archivos]].
- `grep` y `sed` usan expresiones regulares: ver [[Conceptos 04 - Regex]].

---

## 2. Concepto fundamental

La filosofía de Linux: **cada programa hace una sola cosa bien**, y se **encadenan** pasando la salida de uno a la entrada del siguiente.

```
comando1  |  comando2  |  comando3  >  archivo.txt
   │            │            │             │
 produce    filtra       ordena       guarda
```

| Símbolo | Nombre | Qué hace |
|---|---|---|
| `\|` | Tubería (*pipe*) | La salida de la izquierda entra en el de la derecha |
| `>` | Redirección | Guarda la salida en un archivo (lo **sobrescribe**) |
| `>>` | Redirección | **Añade** la salida al final del archivo |
| `<` | Redirección | Usa un archivo como entrada |
| `2>` | Redirección de errores | Guarda solo los errores |
| `2>&1` | | Junta errores y salida normal |

---

## 3. Sintaxis / estructura

```bash
grep "texto" archivo              # busca líneas que contienen "texto"
find ruta -name "patrón"          # busca archivos por nombre
comando | grep "texto"            # filtra la salida de otro comando
comando > salida.txt              # guarda la salida
sed 's/viejo/nuevo/g' archivo     # reemplaza texto
```

---

## 4. Elementos / propiedades / características

### 4.1 `grep`: buscar texto

| Opción | Qué hace |
|---|---|
| `-i` | Ignora mayúsculas/minúsculas |
| `-r` | Busca en carpetas de forma recursiva |
| `-n` | Muestra el número de línea |
| `-v` | Muestra las líneas que **no** coinciden |
| `-c` | Cuenta coincidencias |
| `-l` | Solo muestra nombres de archivo |
| `-w` | Solo palabras completas |
| `-E` | Expresiones regulares extendidas |
| `-A 3` / `-B 3` / `-C 3` | Muestra 3 líneas después / antes / alrededor |

```bash
grep -rn "TODO" src/
grep -i "error" app.log
grep -E "^(GET|POST)" access.log
```

### 4.2 `find`: buscar archivos

```bash
find . -name "*.js"                 # por nombre
find . -iname "readme*"             # sin distinguir mayúsculas
find . -type d -name node_modules   # solo carpetas
find . -type f -size +10M           # archivos de más de 10 MB
find . -mtime -1                    # modificados en las últimas 24 h
find . -name "*.log" -delete        # buscar y borrar
find . -name "*.tmp" -exec rm {} \; # ejecutar un comando por cada uno
```

### 4.3 Ordenar, contar y recortar

| Comando | Qué hace |
|---|---|
| `sort` | Ordena líneas (`-n` numérico, `-r` inverso) |
| `uniq` | Quita líneas repetidas **consecutivas** (`-c` cuenta) |
| `wc -l` | Cuenta líneas |
| `cut -d',' -f2` | Saca la columna 2 separando por comas |
| `head` / `tail` | Primeras / últimas líneas |
| `tr 'a-z' 'A-Z'` | Cambia caracteres |

### 4.4 `sed`: reemplazar texto

```bash
sed 's/http:/https:/' archivo        # primera aparición de cada línea
sed 's/http:/https:/g' archivo       # todas (g = global)
sed -i 's/localhost/0.0.0.0/g' config.js   # modifica el archivo (-i)
sed -n '10,20p' archivo              # muestra solo las líneas 10 a 20
sed '/^#/d' archivo                  # borra líneas que empiezan por #
```

### 4.5 `awk`: trabajar con columnas

```bash
awk '{print $1}' access.log          # primera columna
awk -F',' '{print $2}' datos.csv     # columna 2 de un CSV
awk '$3 > 100' datos.txt             # líneas cuya columna 3 es > 100
```

### 4.6 `xargs`: pasar resultados como argumentos

```bash
find . -name "*.js" | xargs grep "console.log"
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```bash
ls -l | grep ".md"        # solo los archivos .md
history | grep docker     # comandos de docker que usaste
```

### 5.2 Ejemplo habitual: analizar un log

```bash
# Las 10 IPs que más peticiones hacen
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Cuántos errores 500 hay
grep -c '" 500 ' access.log

# Seguir solo los errores en tiempo real
tail -f app.log | grep -i error
```

### 5.3 Buscar en un proyecto sin `node_modules`

```bash
grep -rn "fetch(" src/ --include="*.js" --exclude-dir=node_modules
```

### 5.4 Guardar errores y salida

```bash
npm run build > build.log 2>&1   # todo al mismo archivo
comando 2> /dev/null             # descarta los errores
```

---

## 6. Buenas prácticas

- **Pon comillas** al patrón de `grep` y `find`: `"*.js"`, no `*.js` (si no, la shell lo expande antes).
- **Prueba `sed` sin `-i`** primero: comprueba el resultado y luego aplica el cambio.
- **Excluye carpetas pesadas** (`node_modules`, `.git`, `vendor`) al buscar.
- **Cuidado con `>`**: sobrescribe el archivo sin preguntar. Usa `>>` para añadir.
- Para proyectos grandes, `rg` (*ripgrep*) es mucho más rápido que `grep -r` y respeta `.gitignore`.

---

## 7. Diferencias importantes

### 7.1 `grep` vs. `find`

| | `grep` | `find` |
|---|---|---|
| Busca | **Dentro** de los archivos (contenido) | **Archivos** (nombre, tipo, tamaño, fecha) |

### 7.2 `>` vs. `>>`

| | `>` | `>>` |
|---|---|---|
| Si el archivo existe | Lo **sobrescribe** | **Añade** al final |

### 7.3 `|` vs. `>`

`|` envía la salida a **otro comando**. `>` la envía a un **archivo**.

### 7.4 `uniq` sin `sort`

`uniq` solo elimina duplicados **seguidos**. Para quitar todos, ordena antes: `sort | uniq` (o `sort -u`).

---

## 8. Casos especiales

### 8.1 `sed -i` en macOS

En macOS hay que pasar una extensión vacía: `sed -i '' 's/a/b/g' archivo`.

### 8.2 Buscar texto con caracteres especiales

Usa `grep -F` para buscar el texto literal sin interpretar regex: `grep -F "a.b*c" archivo`.

### 8.3 Archivos con espacios en `xargs`

```bash
find . -name "*.txt" -print0 | xargs -0 rm
```

### 8.4 `/dev/null`

Es un "agujero negro": todo lo que se envía ahí desaparece. Útil para silenciar salidas.

---

## 9. Resumen

- `|` encadena comandos; `>` guarda en archivo; `>>` añade; `2>` captura errores.
- `grep` busca **texto** (`-rn`, `-i`, `-v`, `-c`).
- `find` busca **archivos** (`-name`, `-type`, `-size`, `-exec`).
- `sort`, `uniq -c`, `wc -l`, `cut`, `head` para analizar resultados.
- `sed 's/a/b/g'` reemplaza texto; `-i` modifica el archivo.
- `awk '{print $1}'` saca columnas.
- Patrones entre comillas y excluye `node_modules`.
