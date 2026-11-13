# 01 - Fundamentos de la terminal

> [!info] ¿Qué es?
> La **terminal** es un programa donde escribes órdenes (**comandos**) y el ordenador te responde con texto. Dentro de ella se ejecuta una **shell** (normalmente **Bash** o **Zsh**), que es la que interpreta lo que escribes.

---

## 1. Antes de empezar

No hace falta saber nada. Solo abre una terminal:

| Sistema | Qué abrir |
|---|---|
| Windows | **Git Bash** o **WSL** (Ubuntu) |
| macOS | Aplicación *Terminal* (usa Zsh) |
| Linux | *Terminal* (normalmente Bash) |

---

## 2. Concepto fundamental

Cuando abres la terminal ves el **prompt**, que indica que está esperando una orden:

```
usuario@equipo:~/proyectos$ █
```

| Parte | Significado |
|---|---|
| `usuario` | Con qué usuario estás conectado |
| `equipo` | Nombre de la máquina |
| `~/proyectos` | Carpeta en la que estás ahora (`~` = tu carpeta personal) |
| `$` | Usuario normal (`#` indica administrador, *root*) |

El ciclo siempre es: **escribes un comando → pulsas Enter → la shell lo ejecuta → muestra el resultado → vuelve el prompt**.

---

## 3. Sintaxis / estructura

Un comando tiene esta forma:

```
comando  -opciones  argumentos
```

```bash
ls -la /home
```

| Parte | En el ejemplo | Qué es |
|---|---|---|
| Comando | `ls` | El programa que se ejecuta |
| Opciones | `-la` | Modifican su comportamiento (empiezan por `-` o `--`) |
| Argumentos | `/home` | Sobre qué actúa |

- Las opciones cortas se pueden juntar: `-l -a` es igual que `-la`.
- Las opciones largas usan dos guiones: `--all`, `--help`.
- **Mayúsculas y minúsculas importan**: `ls` funciona, `LS` no.

---

## 4. Elementos / propiedades / características

### 4.1 Shells más comunes

| Shell | Dónde aparece |
|---|---|
| **Bash** | Por defecto en la mayoría de Linux y en Git Bash |
| **Zsh** | Por defecto en macOS |
| **PowerShell** | Windows (sintaxis distinta, no compatible con Bash) |

### 4.2 Pedir ayuda

```bash
comando --help     # ayuda corta (casi todos los comandos)
man comando        # manual completo (sal con q)
type comando       # qué es: programa, alias o función
which comando      # en qué ruta está el programa
```

### 4.3 Atajos de teclado imprescindibles

| Atajo | Qué hace |
|---|---|
| `Tab` | Autocompleta comandos y rutas (púlsalo dos veces para ver opciones) |
| `↑` / `↓` | Recorre los comandos anteriores |
| `Ctrl + C` | Cancela el comando que se está ejecutando |
| `Ctrl + L` | Limpia la pantalla (igual que `clear`) |
| `Ctrl + R` | Busca en el historial de comandos |
| `Ctrl + A` / `Ctrl + E` | Va al principio / final de la línea |
| `Ctrl + D` | Cierra la sesión (o la terminal) |

### 4.4 Historial

```bash
history          # lista de comandos usados
!!               # repite el último comando
sudo !!          # repite el último comando como administrador
!42              # repite el comando número 42 del historial
```

### 4.5 Variables de entorno

La shell guarda valores con nombre que usan los programas:

```bash
echo $HOME       # tu carpeta personal
echo $PATH       # carpetas donde se buscan los programas
echo $USER       # tu usuario
export MI_VAR=hola   # crea una variable para esta sesión
```

> [!tip] ¿Qué es el PATH?
> Cuando escribes `node`, la shell busca un programa llamado `node` en cada carpeta del `PATH`, en orden. Si dice *command not found*, o no está instalado o su carpeta no está en el `PATH`.

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico

```bash
pwd              # ¿dónde estoy?
ls               # ¿qué hay aquí?
echo "Hola"      # imprime un texto
date             # fecha y hora
whoami           # con qué usuario estoy
```

### 5.2 Ejemplo habitual: comprobar herramientas instaladas

```bash
git --version
node --version
python3 --version
docker --version
```

Si alguno responde `command not found`, no está instalado o no está en el `PATH`.

---

## 6. Buenas prácticas

- **Usa `Tab` siempre**: evita errores de escritura en rutas largas.
- **Lee el error completo** antes de buscarlo: casi siempre dice qué pasa.
- **Usa `--help`** antes de buscar en internet.
- **Copia comandos de internet con cuidado**: entiende qué hacen antes de ejecutarlos, sobre todo si llevan `sudo` o `rm`.
- **Pon comillas** en rutas con espacios: `cd "Mis documentos"`.

---

## 7. Diferencias importantes

### 7.1 Terminal vs. shell vs. consola

| Término | Qué es |
|---|---|
| Terminal | La **ventana** (el programa que muestra texto) |
| Shell | El **intérprete** que ejecuta los comandos (Bash, Zsh) |
| Consola | Se usa casi como sinónimo de terminal |

### 7.2 Bash vs. PowerShell

| | Bash | PowerShell |
|---|---|---|
| Sistema | Linux, macOS, Git Bash | Windows |
| Listar archivos | `ls` | `Get-ChildItem` (también acepta `ls`) |
| Variables | `$HOME` | `$env:HOME` |
| Rutas | `/c/Users/yo` | `C:\Users\yo` |

Los servidores usan Bash: por eso conviene aprender Bash aunque uses Windows.

---

## 8. Casos especiales

### 8.1 El comando no termina

Si un programa se queda ejecutando (un servidor, por ejemplo), la terminal no devuelve el prompt. Para pararlo: `Ctrl + C`.

### 8.2 Varios comandos en una línea

```bash
cd proyecto && npm install   # el segundo solo si el primero funciona
comando1 ; comando2          # el segundo siempre
comando1 || comando2         # el segundo solo si el primero falla
```

### 8.3 Comandos largos en varias líneas

```bash
docker run -d \
  -p 8080:80 \
  --name web \
  nginx
```

La barra `\` al final indica que el comando continúa en la línea siguiente.

---

## 9. Resumen

- La **terminal** es la ventana; la **shell** (Bash, Zsh) interpreta los comandos.
- Estructura: `comando -opciones argumentos`.
- El **prompt** muestra usuario, equipo y carpeta actual.
- `Tab` autocompleta, `↑` repite, `Ctrl + C` cancela.
- `--help` y `man` dan ayuda sin salir de la terminal.
- `PATH` es la lista de carpetas donde se buscan los programas.
- Siguiente paso: [[Terminal 02 - Navegación y archivos]].
