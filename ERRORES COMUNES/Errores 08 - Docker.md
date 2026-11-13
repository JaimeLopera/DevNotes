# 08 - Errores de Docker

> [!info] ¿Qué es?
> Los errores más frecuentes con **Docker** y **Docker Compose**, con el **mensaje literal**, qué significa y cómo solucionarlo. La herramienta de diagnóstico principal es `docker logs nombre-contenedor`: casi siempre dice por qué un contenedor falla.

---

## 1. Antes de empezar: comandos de diagnóstico

```bash
docker ps -a                    # ¿está en marcha? ¿con qué estado salió?
docker logs nombre              # ¿qué dijo antes de fallar?
docker logs -f nombre           # seguir los logs en vivo
docker inspect nombre           # configuración completa (red, volúmenes, variables)
docker exec -it nombre sh       # entrar a mirar
docker compose ps               # estado de los servicios de Compose
docker compose logs -f servicio
```

Más en [[Docker 06 - Referencia rápida|Referencia rápida de Docker]].

---

## 2. Docker no arranca

### 2.1 `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

**Qué significa:** el servicio de Docker no está en marcha.

**Solución:** abre **Docker Desktop** (Windows/macOS) o arráncalo en Linux:

```bash
sudo systemctl start docker
```

### 2.2 `error during connect: ... open //./pipe/docker_engine: The system cannot find the file specified.` (Windows)

**Solución:** Docker Desktop no está abierto o no ha terminado de arrancar. Si persiste, revisa que WSL 2 esté instalado y actualizado (`wsl --update`).

### 2.3 `permission denied while trying to connect to the Docker daemon socket`

**Qué significa:** tu usuario de Linux no tiene permiso para usar Docker.

**Solución:**

```bash
sudo usermod -aG docker $USER
# cierra sesión y vuelve a entrar
```

---

## 3. Imágenes y construcción

### 3.1 `Unable to find image 'x:latest' locally` / `pull access denied` / `repository does not exist`

**Causas típicas:** nombre de imagen mal escrito, etiqueta inexistente, o imagen privada sin `docker login`.

**Solución:** comprueba el nombre exacto en Docker Hub.

### 3.2 `failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory`

**Qué significa:** no hay `Dockerfile` en la carpeta del contexto.

**Solución:** ejecuta `docker build` en la carpeta correcta, o indica el archivo: `docker build -f ruta/Dockerfile .`.

### 3.3 `COPY failed: file not found in build context`

**Causas típicas:**
- La ruta del `COPY` es relativa al **contexto** (la carpeta del `.` final), no al Dockerfile.
- El archivo está excluido en `.dockerignore`.

### 3.4 `npm ERR!` o errores de compilación durante el `RUN`

**Solución:** lee el error del paso concreto. Para ver toda la salida: `docker build --progress=plain --no-cache .`.

### 3.5 Los cambios en el código no aparecen en el contenedor

**Qué significa:** la imagen se construyó con el código antiguo.

**Solución:** reconstruye: `docker compose up -d --build`. En desarrollo, monta el código como volumen.

---

## 4. Contenedores

### 4.1 El contenedor se para nada más arrancar (`Exited (0)` o `Exited (1)`)

**Causas típicas:**
- `Exited (0)`: el proceso principal terminó (por ejemplo, un script que acaba). Un contenedor vive mientras vive su proceso principal.
- `Exited (1)`: la aplicación falló al arrancar.

**Solución:** `docker logs nombre` para ver el motivo.

### 4.2 `docker: Error response from daemon: Conflict. The container name "/web" is already in use`

**Solución:** borra el anterior (`docker rm -f web`) o usa otro nombre.

### 4.3 `Bind for 0.0.0.0:8080 failed: port is already allocated`

**Qué significa:** ese puerto de tu equipo ya lo usa otro contenedor u otro programa.

**Solución:** para el otro (`docker ps` y `docker stop`) o publica en otro puerto: `-p 8081:80`.

### 4.4 La app funciona dentro pero no puedo abrirla en el navegador

**Causas típicas:**
- No publicaste el puerto (`-p 3000:3000` / `ports:` en Compose).
- La app escucha en `localhost`/`127.0.0.1` **dentro** del contenedor: debe escuchar en `0.0.0.0`.

**Solución:**

```javascript
app.listen(3000, "0.0.0.0");
```

### 4.5 `exec /usr/local/bin/docker-entrypoint.sh: no such file or directory` / `exec format error`

**Causas típicas:**
- Un script con saltos de línea de Windows (CRLF): conviértelo a LF.
- Imagen construida para otra arquitectura (ARM vs. x86): usa `--platform linux/amd64`.

---

## 5. Redes y comunicación entre contenedores

### 5.1 `ECONNREFUSED 127.0.0.1:5432` desde un contenedor hacia la base de datos

**Qué significa:** dentro de un contenedor, `localhost` es **el propio contenedor**, no tu equipo ni otro contenedor.

**Solución:** en Compose, usa el **nombre del servicio** como host:

```yaml
services:
  api:
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/app   # "db", no "localhost"
  db:
    image: postgres:16
```

Para llegar a un servicio de tu equipo desde el contenedor: `host.docker.internal`.

### 5.2 La API arranca antes que la base de datos y falla

**Solución:** `depends_on` con comprobación de salud:

```yaml
services:
  api:
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 10
```

Más en [[Docker 05 - Docker Compose|Docker Compose]].

---

## 6. Volúmenes y archivos

### 6.1 Los datos de la base de datos desaparecen al recrear el contenedor

**Qué significa:** no había volumen; los datos vivían dentro del contenedor.

**Solución:**

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - datos-db:/var/lib/postgresql/data
volumes:
  datos-db:
```

### 6.2 `Permission denied` al escribir en un volumen montado

**Causa típica:** el proceso del contenedor usa otro usuario (UID) que el dueño de la carpeta.

**Solución:** ajusta permisos de la carpeta o ejecuta con tu usuario: `--user $(id -u):$(id -g)`.

### 6.3 Al montar el código, desaparece `node_modules`

**Qué significa:** el volumen del código tapa el `node_modules` instalado en la imagen.

**Solución:** añade un volumen anónimo para `node_modules`:

```yaml
volumes:
  - .:/app
  - /app/node_modules
```

### 6.4 `no space left on device`

**Solución:** limpia lo que no uses: `docker system df` para ver el espacio y `docker system prune` (con `-a` y `--volumes` solo si estás seguro).

---

## 7. Compose

### 7.1 `yaml: line X: did not find expected key` / `mapping values are not allowed here`

**Qué significa:** error de sangría o de sintaxis YAML.

**Solución:** usa **espacios** (nunca tabuladores), con la misma sangría por nivel; revisa los `:` y los guiones de las listas.

### 7.2 `docker-compose: command not found`

**Qué significa:** la versión antigua (con guion) no está instalada. La actual es un subcomando: `docker compose` (con espacio).

### 7.3 Cambié el `.env` o el `docker-compose.yml` y no se aplica

**Solución:** `docker compose up -d` recrea los servicios cambiados; si cambiaste el Dockerfile, añade `--build`.

---

## 8. Resumen

- Primer paso siempre: `docker ps -a` y `docker logs nombre`.
- **Daemon no disponible** → abre Docker Desktop / `systemctl start docker`.
- **Puerto ocupado** → para el otro o cambia el puerto publicado.
- **No se ve en el navegador** → publica el puerto y escucha en `0.0.0.0`.
- **Contenedor a contenedor** → usa el nombre del servicio, no `localhost`.
- **Datos que se pierden** → usa volúmenes.
- **Cambios que no aparecen** → `docker compose up -d --build`.
