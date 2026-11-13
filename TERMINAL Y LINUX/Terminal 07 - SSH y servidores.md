# 07 - SSH y servidores

> [!info] ¿Qué es?
> **SSH** (*Secure Shell*) permite **abrir una terminal en otro ordenador** a través de internet, de forma cifrada. Es como se trabaja con servidores: instalar programas, desplegar webs, ver logs. También se usa para autenticarse en **GitHub** sin contraseña.

---

## 1. Antes de empezar

- Necesitas manejarte con la terminal: [[Terminal 02 - Navegación y archivos]] y [[Terminal 03 - Permisos y usuarios]].
- Ayuda saber qué es una IP y un dominio: [[Conceptos 06 - Cómo funciona internet]].

---

## 2. Concepto fundamental

SSH usa un **par de claves**:

| Clave | Dónde está | Se comparte |
|---|---|---|
| **Privada** (`id_ed25519`) | Solo en tu ordenador | **Nunca** |
| **Pública** (`id_ed25519.pub`) | La copias al servidor o a GitHub | Sí |

```
Tu equipo (clave privada) ──SSH──► Servidor (tiene tu clave pública)
                                   "Esta clave encaja: adelante"
```

Es como un candado (pública) que puedes dar a cualquiera, y la única llave que lo abre (privada), que solo tienes tú.

---

## 3. Sintaxis / estructura

```bash
ssh usuario@servidor             # conectarse
ssh usuario@203.0.113.10         # con IP
ssh -p 2222 usuario@servidor     # con otro puerto (por defecto: 22)
exit                             # salir del servidor
```

---

## 4. Elementos / propiedades / características

### 4.1 Crear un par de claves

```bash
ssh-keygen -t ed25519 -C "tu-correo@ejemplo.com"
```

- Pulsa Enter para guardarla en `~/.ssh/id_ed25519`.
- Pon una **frase de contraseña** (*passphrase*) para protegerla.

### 4.2 Copiar la clave pública al servidor

```bash
ssh-copy-id usuario@servidor
```

A partir de ahí, `ssh usuario@servidor` entra sin pedir la contraseña del servidor.

### 4.3 Archivo de configuración `~/.ssh/config`

Evita escribir siempre usuario, IP y puerto:

```
Host mi-vps
    HostName 203.0.113.10
    User ana
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Ahora basta con:

```bash
ssh mi-vps
```

### 4.4 Copiar archivos

| Comando | Uso |
|---|---|
| `scp archivo usuario@servidor:/ruta/` | Subir un archivo |
| `scp usuario@servidor:/ruta/archivo .` | Descargar un archivo |
| `scp -r carpeta/ usuario@servidor:/ruta/` | Subir una carpeta |
| `rsync -avz carpeta/ usuario@servidor:/ruta/` | Sincronizar (solo copia lo que cambió) |

### 4.5 Ejecutar un comando remoto sin entrar

```bash
ssh mi-vps "df -h"
ssh mi-vps "cd /var/www/app && git pull && npm ci && pm2 restart app"
```

### 4.6 Archivos de la carpeta `~/.ssh`

| Archivo | Para qué |
|---|---|
| `id_ed25519` | Tu clave privada |
| `id_ed25519.pub` | Tu clave pública |
| `config` | Atajos de conexión |
| `known_hosts` | Servidores en los que ya confiaste |
| `authorized_keys` | (En el servidor) claves públicas que pueden entrar |

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: conectar y mirar

```bash
ssh ana@203.0.113.10
pwd
ls /var/www
exit
```

### 5.2 Ejemplo habitual: SSH con GitHub

```bash
# 1. Crear la clave (si no tienes)
ssh-keygen -t ed25519 -C "tu-correo@ejemplo.com"

# 2. Ver la clave pública y copiarla
cat ~/.ssh/id_ed25519.pub

# 3. Pegarla en GitHub → Settings → SSH and GPG keys → New SSH key

# 4. Comprobar
ssh -T git@github.com
# Hi usuario! You've successfully authenticated...

# 5. Clonar con SSH
git clone git@github.com:usuario/repo.git
```

### 5.3 Desplegar una web estática

```bash
rsync -avz --delete dist/ mi-vps:/var/www/mi-web/
```

---

## 6. Buenas prácticas

- **Nunca compartas tu clave privada** ni la subas a un repositorio.
- **Usa `ed25519`**: es moderna, segura y corta.
- **Protege la clave con una passphrase** y usa `ssh-agent` para no escribirla siempre.
- **Permisos**: `chmod 700 ~/.ssh` y `chmod 600 ~/.ssh/id_ed25519`.
- **En servidores propios**: desactiva el acceso con contraseña y el acceso directo de `root`.
- **Usa `~/.ssh/config`** para no memorizar IPs.

---

## 7. Diferencias importantes

### 7.1 `scp` vs. `rsync`

| | `scp` | `rsync` |
|---|---|---|
| Copia | Todo, siempre | Solo lo que cambió |
| Reanudar | No | Sí |
| Borrar en destino lo que ya no existe | No | Sí (`--delete`) |
| Uso | Copias puntuales | Despliegues y sincronización |

### 7.2 HTTPS vs. SSH en Git

| | HTTPS | SSH |
|---|---|---|
| URL | `https://github.com/u/repo.git` | `git@github.com:u/repo.git` |
| Autenticación | Token o gestor de credenciales | Clave SSH |
| Configuración | Ninguna | Crear y subir la clave una vez |

### 7.3 Contraseña vs. clave

Las claves son mucho más seguras que las contraseñas: no se pueden adivinar por fuerza bruta y no viajan por la red.

---

## 8. Casos especiales

### 8.1 *Permission denied (publickey)*

- La clave pública no está en el servidor (o en GitHub).
- Usas un usuario equivocado (`root` en vez de `ana`).
- La clave privada tiene permisos demasiado abiertos (`chmod 600`).
- Prueba con `ssh -v` para ver qué clave se está usando.

### 8.2 *Host key verification failed*

El servidor ha cambiado su identidad (por ejemplo, lo reinstalaste). Si sabes que es legítimo:

```bash
ssh-keygen -R 203.0.113.10
```

### 8.3 Mantener procesos al desconectarte

Si cierras SSH, los procesos lanzados se paran. Usa `tmux`/`screen`, o mejor un gestor de procesos (`systemd`, `pm2`, Docker).

### 8.4 SSH en Windows

Git Bash y PowerShell incluyen `ssh` y `ssh-keygen`. Las claves se guardan en `C:\Users\tu-usuario\.ssh\`.

---

## 9. Resumen

- SSH abre una terminal cifrada en otro equipo: `ssh usuario@servidor`.
- Funciona con un **par de claves**: la privada nunca se comparte.
- `ssh-keygen -t ed25519` crea las claves; `ssh-copy-id` sube la pública.
- `~/.ssh/config` guarda atajos de conexión.
- `scp` copia archivos; `rsync` sincroniza (ideal para desplegar).
- La misma clave sirve para GitHub: `git@github.com:usuario/repo.git`.
