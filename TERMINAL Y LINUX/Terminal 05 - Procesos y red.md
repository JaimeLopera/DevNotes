# 05 - Procesos y red

> [!info] ¿Qué es?
> Un **proceso** es un programa en ejecución (un servidor de Node, una base de datos, el propio editor). Esta nota explica cómo **verlos, pararlos y dejarlos en segundo plano**, y cómo hacer las **comprobaciones de red** más habituales: puertos ocupados, peticiones HTTP y conectividad.

---

## 1. Antes de empezar

- Conviene haber leído [[Terminal 01 - Fundamentos de la terminal]] y [[Terminal 04 - Buscar y procesar texto]] (tuberías con `grep`).
- Para entender puertos y peticiones: [[Conceptos 01 - HTTP]] y [[Conceptos 06 - Cómo funciona internet]].

---

## 2. Concepto fundamental

Cada proceso tiene un número único, el **PID** (*Process ID*). Con ese número puedes consultarlo o pararlo.

```
PID   USUARIO  COMANDO
1234  ana      node server.js     ← escucha en el puerto 3000
5678  ana      postgres           ← escucha en el puerto 5432
```

Un **puerto** es una "puerta" numerada por la que un proceso recibe conexiones. **Solo un proceso** puede escuchar en un puerto a la vez: de ahí el error *address already in use*.

---

## 3. Sintaxis / estructura

```bash
ps aux                  # lista todos los procesos
top                     # monitor en tiempo real (sal con q)
kill PID                # pide a un proceso que termine
lsof -i :3000           # qué proceso usa el puerto 3000
curl https://url        # hace una petición HTTP
ping dominio.com        # comprueba si un equipo responde
```

---

## 4. Elementos / propiedades / características

### 4.1 Ver procesos

| Comando | Qué muestra |
|---|---|
| `ps aux` | Todos los procesos del sistema |
| `ps aux \| grep node` | Solo los que contienen "node" |
| `pgrep -a node` | PID y comando de los procesos "node" |
| `top` | Monitor en vivo (CPU, memoria) |
| `htop` | Como `top`, más visual (si está instalado) |

### 4.2 Parar procesos

| Comando | Qué hace |
|---|---|
| `Ctrl + C` | Para el proceso que está en primer plano |
| `kill PID` | Le pide que termine (señal `TERM`, permite cerrar limpio) |
| `kill -9 PID` | Lo mata a la fuerza (señal `KILL`, último recurso) |
| `pkill node` | Termina todos los procesos llamados "node" |
| `killall node` | Igual que `pkill` |

### 4.3 Primer y segundo plano

| Comando | Qué hace |
|---|---|
| `comando &` | Lo arranca en segundo plano |
| `Ctrl + Z` | Pausa el proceso en primer plano |
| `bg` | Continúa el proceso pausado en segundo plano |
| `fg` | Lo trae de vuelta al primer plano |
| `jobs` | Lista los procesos de esta terminal |
| `nohup comando &` | Sigue ejecutándose aunque cierres la terminal |

### 4.4 Puertos

```bash
lsof -i :3000                 # qué proceso usa el puerto 3000
ss -tulpn                     # todos los puertos en escucha (Linux)
netstat -ano | findstr :3000  # equivalente en Windows (cmd/PowerShell)
```

### 4.5 Peticiones HTTP con `curl`

| Opción | Qué hace |
|---|---|
| `curl url` | GET y muestra el cuerpo |
| `-i` | Incluye las cabeceras de la respuesta |
| `-I` | Solo cabeceras (petición HEAD) |
| `-X POST` | Cambia el método |
| `-H "Clave: valor"` | Añade una cabecera |
| `-d '{...}'` | Envía un cuerpo |
| `-o archivo` | Guarda la respuesta en un archivo |
| `-L` | Sigue redirecciones |
| `-v` | Modo detallado (depurar) |

### 4.6 Diagnóstico de red

| Comando | Para qué |
|---|---|
| `ping google.com` | ¿Hay conexión? ¿Responde ese equipo? |
| `nslookup dominio.com` / `dig dominio.com` | ¿A qué IP apunta un dominio? (DNS) |
| `traceroute dominio.com` | Por dónde pasa la conexión |
| `ip a` (Linux) / `ipconfig` (Windows) | Tu dirección IP |
| `wget url` | Descarga un archivo |

### 4.7 Recursos del sistema

```bash
df -h          # espacio en disco
du -sh carpeta # tamaño de una carpeta
free -h        # memoria RAM (Linux)
uptime         # tiempo encendido y carga
```

---

## 5. Ejemplos prácticos

### 5.1 Ejemplo básico: liberar un puerto ocupado

```bash
npm run dev
# Error: listen EADDRINUSE: address already in use :::3000

lsof -i :3000
# COMMAND  PID   USER
# node     1234  ana

kill 1234
npm run dev    # ahora sí arranca
```

### 5.2 Ejemplo habitual: probar una API

```bash
# GET
curl -i http://localhost:3000/api/usuarios

# POST con JSON
curl -X POST http://localhost:3000/api/usuarios \
  -H "Content-Type: application/json" \
  -d '{"nombre": "Ana", "email": "ana@mail.com"}'

# Con token
curl -H "Authorization: Bearer MI_TOKEN" http://localhost:3000/api/perfil
```

### 5.3 Comprobar si una web responde

```bash
curl -I https://mi-web.com
# HTTP/2 200   → funciona
```

---

## 6. Buenas prácticas

- **Prueba `kill` antes que `kill -9`**: el primero deja que el programa cierre conexiones y guarde datos.
- **Comprueba el PID** antes de matar un proceso: no pares uno equivocado.
- **No dejes servidores olvidados** en segundo plano: ocupan puertos y memoria.
- **Para procesos permanentes** en un servidor usa un gestor (`systemd`, `pm2`, Docker), no `nohup`.
- **Usa `curl -i`** para ver el código de estado, no solo el cuerpo.

---

## 7. Diferencias importantes

### 7.1 `kill` vs. `kill -9`

| | `kill PID` | `kill -9 PID` |
|---|---|---|
| Señal | `TERM` (15) | `KILL` (9) |
| El programa puede limpiar | Sí | No |
| Cuándo usarlo | Siempre primero | Si el proceso no responde |

### 7.2 `Ctrl + C` vs. `Ctrl + Z`

`Ctrl + C` **termina** el proceso. `Ctrl + Z` lo **pausa** (sigue existiendo; vuelve con `fg`).

### 7.3 `curl` vs. `wget`

`curl` sirve para cualquier petición HTTP (APIs, cabeceras, métodos). `wget` está pensado para **descargar archivos**.

### 7.4 `localhost` vs. `0.0.0.0`

| Dirección | Quién puede conectarse |
|---|---|
| `localhost` / `127.0.0.1` | Solo tu propio equipo |
| `0.0.0.0` | Cualquier equipo de la red (necesario dentro de Docker) |

---

## 8. Casos especiales

### 8.1 Liberar un puerto en Windows

```powershell
netstat -ano | findstr :3000
taskkill /PID 1234 /F
```

### 8.2 Puertos menores de 1024

En Linux, escuchar en puertos como el 80 o el 443 requiere permisos de administrador. En desarrollo usa puertos altos (3000, 8000, 8080).

### 8.3 Ver logs de un servicio del sistema

```bash
sudo systemctl status nginx
sudo journalctl -u nginx -f
```

### 8.4 WP-CLI y otras herramientas de servidor

Herramientas como WP-CLI (ver [[PHP 20 - PHP en Wordpress]]) se ejecutan como procesos normales de terminal: se pueden combinar con `&&`, redirigir a logs o lanzar por SSH ([[Terminal 07 - SSH y servidores]]).

---

## 9. Resumen

- Un **proceso** es un programa en ejecución; se identifica por su **PID**.
- `ps aux | grep`, `pgrep`, `top` para verlos.
- `kill PID` para pararlos; `kill -9` solo si no responden.
- `&`, `Ctrl + Z`, `bg`, `fg` para el segundo plano.
- `lsof -i :PUERTO` encuentra quién ocupa un puerto.
- `curl -i` prueba APIs; `ping` y `dig` comprueban la red.
- `localhost` solo acepta conexiones locales; `0.0.0.0` acepta de fuera.
