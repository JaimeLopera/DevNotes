# 06 - Bash scripting

> [!info] ¿Qué es?
> Un **script de Bash** es un archivo de texto con comandos de terminal que se ejecutan en orden. Sirve para **automatizar** tareas repetitivas: copias de seguridad, despliegues, preparar un proyecto, procesar archivos.

---

## 1. Antes de empezar

- Conviene dominar [[Terminal 02 - Navegación y archivos]] y [[Terminal 04 - Buscar y procesar texto]].
- Para ejecutar un script necesitas permiso de ejecución: ver [[Terminal 03 - Permisos y usuarios]].

---

## 2. Concepto fundamental

Todo lo que escribes en la terminal se puede guardar en un archivo `.sh` y ejecutarlo de una vez. Bash añade lo que tiene cualquier lenguaje: **variables**, **condiciones**, **bucles** y **funciones**.

```
script.sh  ──bash script.sh──►  ejecuta cada línea en orden
```

---

## 3. Sintaxis / estructura

```bash
#!/usr/bin/env bash
# La primera línea (shebang) indica con qué programa se ejecuta

set -euo pipefail   # para el script ante cualquier error

NOMBRE="mundo"
echo "Hola, $NOMBRE"
```

Ejecutarlo:

```bash
chmod +x script.sh
./script.sh
# o, sin dar permisos:
bash script.sh
```

---

## 4. Elementos / propiedades / características

### 4.1 Variables

```bash
nombre="Ana"            # SIN espacios alrededor del =
echo "$nombre"          # usar con $ y entre comillas
echo "${nombre}_backup" # llaves para separar del texto

fecha=$(date +%Y-%m-%d) # guardar la salida de un comando
archivos=$(ls | wc -l)
```

### 4.2 Argumentos del script

| Variable | Contenido |
|---|---|
| `$0` | Nombre del script |
| `$1`, `$2`… | Primer, segundo… argumento |
| `$#` | Número de argumentos |
| `$@` | Todos los argumentos |
| `$?` | Código de salida del último comando (0 = éxito) |

```bash
./deploy.sh produccion   # $1 vale "produccion"
```

### 4.3 Condiciones

```bash
if [ "$1" = "produccion" ]; then
  echo "Desplegando en producción"
elif [ "$1" = "pruebas" ]; then
  echo "Desplegando en pruebas"
else
  echo "Uso: ./deploy.sh produccion|pruebas"
  exit 1
fi
```

| Comprobación | Significa |
|---|---|
| `[ -f ruta ]` | Existe y es un archivo |
| `[ -d ruta ]` | Existe y es una carpeta |
| `[ -z "$var" ]` | La variable está vacía |
| `[ -n "$var" ]` | La variable no está vacía |
| `[ "$a" = "$b" ]` | Textos iguales |
| `[ "$a" != "$b" ]` | Textos distintos |
| `[ "$n" -eq 5 ]` | Números iguales (`-ne`, `-lt`, `-le`, `-gt`, `-ge`) |
| `[ cond1 ] && [ cond2 ]` | Las dos se cumplen |

### 4.4 Bucles

```bash
# Recorrer una lista
for lenguaje in html css js; do
  echo "Aprendiendo $lenguaje"
done

# Recorrer archivos
for archivo in *.jpg; do
  echo "Procesando $archivo"
done

# Rango de números
for i in {1..5}; do
  echo "Vuelta $i"
done

# Mientras se cumpla una condición
contador=0
while [ $contador -lt 3 ]; do
  echo $contador
  contador=$((contador + 1))
done
```

### 4.5 Funciones

```bash
saludar() {
  local nombre="$1"     # local: solo existe dentro de la función
  echo "Hola, $nombre"
}

saludar "Ana"
```

### 4.6 Operaciones matemáticas

```bash
total=$((5 + 3))
doble=$((total * 2))
```

### 4.7 Leer del usuario

```bash
read -p "¿Continuar? (s/n) " respuesta
if [ "$respuesta" != "s" ]; then
  exit 0
fi
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: copia de seguridad con fecha

```bash
#!/usr/bin/env bash
set -euo pipefail

ORIGEN="$HOME/proyectos"
DESTINO="$HOME/backups"
FECHA=$(date +%Y-%m-%d)

mkdir -p "$DESTINO"
tar -czf "$DESTINO/proyectos-$FECHA.tar.gz" "$ORIGEN"
echo "Copia creada: $DESTINO/proyectos-$FECHA.tar.gz"
```

### 5.2 Ejemplo habitual: preparar un proyecto web

```bash
#!/usr/bin/env bash
set -euo pipefail

if [ $# -eq 0 ]; then
  echo "Uso: ./nuevo-proyecto.sh nombre"
  exit 1
fi

PROYECTO="$1"

mkdir -p "$PROYECTO"/{css,js,img}
cd "$PROYECTO"

cat > index.html <<EOF
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>$PROYECTO</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <script src="js/main.js"></script>
</body>
</html>
EOF

touch css/style.css js/main.js
git init -q
echo "Proyecto $PROYECTO creado"
```

---

## 6. Buenas prácticas

- **Empieza siempre con** `#!/usr/bin/env bash` y `set -euo pipefail`.
- **Pon comillas a las variables**: `"$archivo"`. Sin ellas, un nombre con espacios rompe el script.
- **Usa `local`** dentro de las funciones.
- **Comprueba los argumentos** al principio y muestra un mensaje de uso.
- **Termina con `exit 1`** cuando algo falla, para que otros scripts lo detecten.
- **Para lógica compleja**, usa Python o Node: Bash es ideal para encadenar comandos, no para programas grandes.
- Revisa tus scripts con **ShellCheck** (detecta errores típicos).

---

## 7. Diferencias importantes

### 7.1 `./script.sh` vs. `bash script.sh` vs. `source script.sh`

| Forma | Necesita `chmod +x` | Se ejecuta en |
|---|---|---|
| `./script.sh` | Sí | Una shell nueva |
| `bash script.sh` | No | Una shell nueva |
| `source script.sh` | No | **La shell actual** (sus variables se quedan) |

### 7.2 Comillas dobles vs. simples

| | `"..."` | `'...'` |
|---|---|---|
| Expande variables | Sí: `"Hola $nombre"` → `Hola Ana` | No: `'Hola $nombre'` → `Hola $nombre` |

### 7.3 `[ ]` vs. `[[ ]]`

`[[ ]]` es la versión moderna de Bash: admite `&&`, `||` y patrones dentro (`[[ $archivo == *.js ]]`). `[ ]` es más compatible con otras shells.

---

## 8. Casos especiales

### 8.1 Espacios en las asignaciones

```bash
nombre = "Ana"   # ERROR: Bash cree que "nombre" es un comando
nombre="Ana"     # correcto
```

### 8.2 Saltos de línea de Windows

Un script creado en Windows puede fallar con `$'\r': command not found`. Conviértelo con `dos2unix script.sh` o cambia el final de línea a **LF** en el editor.

### 8.3 Programar tareas con `cron`

```bash
crontab -e
# minuto hora día mes día_semana comando
0 3 * * * /home/ana/backup.sh   # todos los días a las 03:00
```

### 8.4 Heredoc

`<<EOF ... EOF` escribe varias líneas de golpe en un archivo o comando (ver ejemplo 5.2).

---

## 9. Resumen

- Un script es un archivo con comandos que se ejecutan en orden.
- Empieza con `#!/usr/bin/env bash` y `set -euo pipefail`.
- Variables sin espacios: `x="valor"`; se usan como `"$x"`.
- `$1`, `$#`, `$@` para argumentos; `$(comando)` guarda salidas.
- `if [ ... ]; then ... fi`, `for ... do ... done`, `while`.
- `chmod +x` y `./script.sh` para ejecutarlo.
- Comillas dobles expanden variables; simples no.
