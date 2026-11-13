# 03 - Permisos y usuarios

> [!info] ¿Qué es?
> En Linux cada archivo tiene un **dueño** y unos **permisos** que dicen quién puede **leerlo**, **modificarlo** o **ejecutarlo**. Entenderlos explica la mayoría de errores *Permission denied* en servidores, scripts y Docker.

---

## 1. Antes de empezar

Conviene haber leído [[Terminal 02 - Navegación y archivos]], sobre todo `ls -l`.

---

## 2. Concepto fundamental

Cada archivo tiene permisos para **tres grupos de personas**:

| Quién | Letra | Significado |
|---|---|---|
| Usuario (dueño) | `u` | Quien creó el archivo |
| Grupo | `g` | Usuarios del grupo del archivo |
| Otros | `o` | Todos los demás |

Y **tres tipos de permiso**:

| Permiso | Letra | En un archivo | En una carpeta |
|---|---|---|---|
| Lectura | `r` | Ver el contenido | Listar lo que hay dentro |
| Escritura | `w` | Modificarlo | Crear o borrar archivos dentro |
| Ejecución | `x` | Ejecutarlo como programa | Entrar en ella (`cd`) |

---

## 3. Sintaxis / estructura

`ls -l` muestra los permisos así:

```
-rwxr-xr--  1  ana  dev  1024  nov 13 10:00  script.sh
```

```
-  rwx  r-x  r--
│   │    │    └── otros: solo leer
│   │    └─────── grupo: leer y ejecutar
│   └──────────── usuario (dueño): leer, escribir y ejecutar
└──────────────── tipo: - archivo, d carpeta, l enlace
```

Después vienen: número de enlaces, **dueño** (`ana`), **grupo** (`dev`), tamaño, fecha y nombre.

---

## 4. Elementos / propiedades / características

### 4.1 Cambiar permisos: `chmod` (modo simbólico)

```bash
chmod u+x script.sh     # añade ejecución al dueño
chmod g-w archivo       # quita escritura al grupo
chmod o=r archivo       # otros: solo lectura
chmod a+r archivo       # todos (a = all) pueden leer
chmod +x script.sh      # ejecutable para todos
```

### 4.2 Cambiar permisos: `chmod` (modo numérico)

Cada permiso vale un número y se suman:

| Permiso | Valor |
|---|---|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |

| Número | Permisos | Uso típico |
|---|---|---|
| `7` | `rwx` | Todo |
| `6` | `rw-` | Leer y escribir |
| `5` | `r-x` | Leer y ejecutar |
| `4` | `r--` | Solo leer |
| `0` | `---` | Nada |

Se escriben tres cifras: usuario, grupo, otros.

```bash
chmod 755 script.sh   # rwxr-xr-x → scripts y carpetas
chmod 644 index.html  # rw-r--r-- → archivos normales
chmod 600 ~/.ssh/id_ed25519   # rw------- → claves privadas
```

### 4.3 Cambiar dueño: `chown`

```bash
sudo chown ana archivo          # nuevo dueño
sudo chown ana:dev archivo      # dueño y grupo
sudo chown -R www-data:www-data /var/www   # recursivo
```

### 4.4 Usuarios y administrador

| Comando | Qué hace |
|---|---|
| `whoami` | Tu usuario |
| `id` | Tu usuario, grupo y grupos a los que perteneces |
| `sudo comando` | Ejecuta un comando como administrador (*root*) |
| `sudo -i` | Abre una sesión como root (úsalo poco) |
| `su usuario` | Cambia a otro usuario |
| `sudo adduser nombre` | Crea un usuario (Debian/Ubuntu) |
| `sudo usermod -aG grupo usuario` | Añade un usuario a un grupo |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: hacer ejecutable un script

```bash
echo 'echo "Hola"' > saludo.sh
./saludo.sh          # bash: ./saludo.sh: Permission denied
chmod +x saludo.sh
./saludo.sh          # Hola
```

### 5.2 Ejemplo habitual: permisos de una web en un servidor

```bash
# El servidor web (nginx/apache) se ejecuta como www-data
sudo chown -R www-data:www-data /var/www/mi-web
sudo find /var/www/mi-web -type d -exec chmod 755 {} \;
sudo find /var/www/mi-web -type f -exec chmod 644 {} \;
```

### 5.3 Usar Docker sin `sudo`

```bash
sudo usermod -aG docker $USER
# cierra sesión y vuelve a entrar para que se aplique
```

---

## 6. Buenas prácticas

- **Principio de mínimo privilegio**: da solo los permisos necesarios.
- **Nunca uses `chmod 777`**: cualquiera podría modificar o ejecutar el archivo. Si "lo arregla", el problema real es el dueño.
- **Usa `sudo` solo cuando haga falta**, no por costumbre.
- **Claves privadas con `600`**: SSH las rechaza si tienen permisos abiertos.
- **No trabajes como root** en el día a día.

---

## 7. Diferencias importantes

### 7.1 `chmod` vs. `chown`

| | `chmod` | `chown` |
|---|---|---|
| Cambia | **Qué** se puede hacer | **De quién** es |
| Ejemplo | `chmod 644 archivo` | `chown ana archivo` |

### 7.2 `sudo` vs. `su`

| | `sudo` | `su` |
|---|---|---|
| Qué hace | Ejecuta **un comando** como root | **Cambia** de usuario |
| Contraseña que pide | La tuya | La del otro usuario |
| Recomendado | Sí | Solo en casos concretos |

### 7.3 `x` en archivo vs. en carpeta

En un archivo, `x` permite **ejecutarlo**. En una carpeta, `x` permite **entrar** en ella. Por eso las carpetas suelen tener `755` y los archivos `644`.

---

## 8. Casos especiales

### 8.1 *Permission denied* al ejecutar un script

Falta el permiso de ejecución (`chmod +x script.sh`) o lo ejecutas sin `./` delante.

### 8.2 *Permission denied* al escribir

El archivo o la carpeta pertenecen a otro usuario. Revisa con `ls -l` y corrige el dueño con `chown`, no con `chmod 777`.

### 8.3 Permisos en Windows / Git Bash

NTFS no usa permisos de Linux, así que `chmod` apenas tiene efecto en Git Bash. Git sí guarda el bit de ejecución: `git update-index --chmod=+x script.sh`.

### 8.4 Archivos creados por Docker

Los archivos creados dentro de un contenedor pueden pertenecer a `root` en tu equipo. Ejecuta el contenedor con tu usuario (`--user $(id -u):$(id -g)`) o corrige con `sudo chown`.

---

## 9. Resumen

- Permisos: **r** (leer), **w** (escribir), **x** (ejecutar) para **usuario**, **grupo** y **otros**.
- `ls -l` los muestra: `-rwxr-xr--`.
- `chmod` cambia permisos (simbólico `u+x` o numérico `755`).
- `chown` cambia el dueño.
- Valores típicos: `755` carpetas y scripts, `644` archivos, `600` claves.
- `sudo` ejecuta como administrador: úsalo con cuidado.
- Nunca `chmod 777`.
