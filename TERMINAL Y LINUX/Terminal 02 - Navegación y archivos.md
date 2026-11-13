# 02 - Navegación y archivos

> [!info] ¿Qué es?
> Los comandos para **moverte entre carpetas** y **manejar archivos**: ver dónde estás, listar, crear, copiar, mover, renombrar y borrar. Es lo que más vas a usar en la terminal.

---

## 1. Antes de empezar

Conviene haber leído [[Terminal 01 - Fundamentos de la terminal]] (qué es un comando, opciones y argumentos).

---

## 2. Concepto fundamental

En Linux todo cuelga de una única carpeta raíz, `/`. No hay `C:` ni `D:`.

```
/
├── home/
│   └── usuario/        ← tu carpeta personal (~)
│       └── proyectos/
├── etc/                ← configuración del sistema
├── var/
│   └── log/            ← logs
└── usr/                ← programas
```

Una **ruta** indica dónde está un archivo:

| Tipo | Ejemplo | Empieza en |
|---|---|---|
| **Absoluta** | `/home/usuario/proyectos` | La raíz `/` |
| **Relativa** | `proyectos/web` | La carpeta actual |

Símbolos especiales:

| Símbolo | Significa |
|---|---|
| `.` | La carpeta actual |
| `..` | La carpeta de arriba (padre) |
| `~` | Tu carpeta personal |
| `-` | La carpeta en la que estabas antes (solo con `cd`) |

---

## 3. Sintaxis / estructura

```bash
pwd                    # muestra la carpeta actual
ls                     # lista el contenido
cd ruta                # cambia de carpeta
mkdir nombre           # crea una carpeta
touch archivo.txt      # crea un archivo vacío
cp origen destino      # copia
mv origen destino      # mueve o renombra
rm archivo             # borra
```

---

## 4. Elementos / propiedades / características

### 4.1 Moverse: `cd`

```bash
cd proyectos       # entra en "proyectos"
cd ..              # sube un nivel
cd ../..           # sube dos niveles
cd ~               # va a tu carpeta personal (también: cd)
cd -               # vuelve a la carpeta anterior
cd /var/log        # ruta absoluta
```

### 4.2 Listar: `ls`

| Opción | Qué muestra |
|---|---|
| `ls -l` | Lista detallada (permisos, tamaño, fecha) |
| `ls -a` | Incluye archivos ocultos (los que empiezan por `.`) |
| `ls -h` | Tamaños legibles (K, M, G), junto con `-l` |
| `ls -t` | Ordena por fecha de modificación |
| `ls -R` | Recursivo (subcarpetas) |

Lo más habitual: `ls -lah`.

### 4.3 Crear

```bash
mkdir web                  # una carpeta
mkdir -p src/css/base      # crea toda la ruta aunque no existan las intermedias
touch index.html style.css # varios archivos vacíos
```

### 4.4 Copiar y mover

```bash
cp index.html copia.html          # copiar archivo
cp -r src/ backup/                # copiar carpeta (-r = recursivo)
mv viejo.txt nuevo.txt            # renombrar
mv foto.png imagenes/             # mover a otra carpeta
mv -i a.txt b.txt                 # pregunta antes de sobrescribir
```

### 4.5 Borrar

```bash
rm archivo.txt          # borra un archivo
rm -i archivo.txt       # pide confirmación
rm -r carpeta/          # borra una carpeta y todo su contenido
rmdir carpeta/          # borra una carpeta solo si está vacía
```

> [!warning] `rm` no tiene papelera
> Lo que borras con `rm` desaparece para siempre. Nunca ejecutes `rm -rf` sin revisar dos veces la ruta, y nunca con `/` o `~` por error.

### 4.6 Ver el contenido de archivos

| Comando | Para qué |
|---|---|
| `cat archivo` | Muestra todo el archivo |
| `less archivo` | Lo abre paginado (flechas para moverse, `q` para salir, `/texto` para buscar) |
| `head -n 20 archivo` | Las primeras 20 líneas |
| `tail -n 20 archivo` | Las últimas 20 líneas |
| `tail -f archivo.log` | Sigue el archivo en tiempo real (logs) |
| `wc -l archivo` | Cuenta líneas |

### 4.7 Comodines (*wildcards*)

| Comodín | Coincide con | Ejemplo |
|---|---|---|
| `*` | Cualquier texto | `rm *.log` |
| `?` | Un solo carácter | `ls foto?.png` |
| `[abc]` | Uno de esos caracteres | `ls archivo[12].txt` |
| `{a,b}` | Cada alternativa | `mkdir {css,js,img}` |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```bash
cd ~
mkdir prueba
cd prueba
touch notas.txt
ls -l
```

### 5.2 Ejemplo habitual: estructura de un proyecto web

```bash
mkdir -p mi-web/{css,js,img}
cd mi-web
touch index.html css/style.css js/main.js
ls -R
```

Resultado:

```
.:
css  img  index.html  js

./css:
style.css

./img:

./js:
main.js
```

---

## 6. Buenas prácticas

- **Comprueba dónde estás** (`pwd`) antes de borrar o mover.
- **Usa `Tab`** para autocompletar rutas: evita errores de escritura.
- **Usa `-i`** con `rm`, `cp` y `mv` cuando no estés seguro.
- **Evita espacios y tildes** en nombres de archivos de proyectos: usa `mi-archivo.txt`.
- **Pon comillas** si una ruta tiene espacios: `cd "Mis proyectos"`.

---

## 7. Diferencias importantes

### 7.1 `cp` vs. `mv`

| | `cp` | `mv` |
|---|---|---|
| El original | Se mantiene | Desaparece |
| Para carpetas | Necesita `-r` | No necesita nada |
| Renombrar | No | Sí |

### 7.2 `rm -r` vs. `rmdir`

`rmdir` solo borra carpetas **vacías** (es más seguro). `rm -r` borra la carpeta **con todo dentro**.

### 7.3 `cat` vs. `less`

`cat` vuelca todo de golpe (bien para archivos cortos). `less` permite moverse y buscar (mejor para archivos largos o logs).

---

## 8. Casos especiales

### 8.1 Archivos ocultos

Los archivos que empiezan por punto (`.gitignore`, `.env`, `.git/`) están ocultos. Usa `ls -a` para verlos.

### 8.2 Nombres con espacios

```bash
cd "Mis documentos"
cd Mis\ documentos      # escapando el espacio
```

### 8.3 Rutas de Windows en Git Bash

`C:\Users\yo\proyectos` se escribe `/c/Users/yo/proyectos`.

### 8.4 Enlaces simbólicos

Un enlace simbólico es un "acceso directo":

```bash
ln -s /ruta/real enlace
```

---

## 9. Resumen

- Todo cuelga de `/`; tu carpeta personal es `~`.
- `pwd` (dónde estoy), `ls` (qué hay), `cd` (ir a).
- `mkdir -p` crea rutas completas; `touch` crea archivos.
- `cp -r` copia carpetas; `mv` mueve **y** renombra.
- `rm` borra sin papelera: cuidado con `rm -rf`.
- `cat`, `less`, `head`, `tail -f` para leer archivos.
- Comodines: `*`, `?`, `{a,b}`.
- Siguiente paso: [[Terminal 03 - Permisos y usuarios]] o [[Terminal 04 - Buscar y procesar texto]].
