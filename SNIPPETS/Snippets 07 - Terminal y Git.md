# 07 - Snippets de terminal y Git

> [!info] ¿Qué es?
> Comandos combinados de **terminal**, **Git**, **npm** y **Docker** para situaciones del día a día. Las explicaciones están en [[Terminal 00 - Índice|Terminal y Linux]] y [[Git 00 - Índice|Git]].

---

## 1. Procesos y puertos

### 1.1 Liberar un puerto ocupado

```bash
# Linux / macOS / Git Bash con lsof
lsof -ti :3000 | xargs kill

# Windows (PowerShell)
Stop-Process -Id (Get-NetTCPConnection -LocalPort 3000).OwningProcess -Force
```

### 1.2 Ver qué ocupa espacio en una carpeta

```bash
du -sh * | sort -h
```

### 1.3 Borrar todas las carpetas `node_modules` de tus proyectos

```bash
find ~/proyectos -name node_modules -type d -prune -exec rm -rf {} +
```

> [!warning] Revisa la ruta
> Ejecuta primero sin `-exec ...` para ver qué se borraría.

Más en [[Terminal 05 - Procesos y red|Procesos y red]].

---

## 2. Buscar

### 2.1 Buscar texto en un proyecto

```bash
grep -rn "TODO" src/ --exclude-dir=node_modules
```

### 2.2 Reemplazar texto en muchos archivos

```bash
grep -rl "http://" src/ | xargs sed -i 's#http://#https://#g'
```

### 2.3 Encontrar archivos grandes

```bash
find . -type f -size +50M -not -path "./node_modules/*"
```

### 2.4 Contar líneas de código por tipo

```bash
find src -name "*.js" | xargs wc -l | tail -1
```

Más en [[Terminal 04 - Buscar y procesar texto|Buscar y procesar texto]].

---

## 3. Git del día a día

### 3.1 Empezar el día

```bash
git switch main && git pull && git switch -c feature/nombre
```

### 3.2 Guardar todo y subir

```bash
git add -A && git commit -m "feat: descripción" && git push
```

### 3.3 Historial bonito

```bash
git log --oneline --graph --all --decorate -20
```

### 3.4 Ver qué cambió en el último commit

```bash
git show --stat HEAD
```

### 3.5 Poner tu rama al día con `main`

```bash
git fetch && git rebase origin/main
```

### 3.6 Borrar ramas locales ya fusionadas

```bash
git branch --merged main | grep -v "main" | xargs git branch -d
```

### 3.7 Buscar en qué commit se introdujo un texto

```bash
git log -S "textoBuscado" --oneline
```

### 3.8 Quién cambió cada línea de un archivo

```bash
git blame -L 10,30 src/app.js
```

Más en [[Git 02 - Comandos básicos|Comandos básicos]].

---

## 4. Git: rescates

### 4.1 Deshacer el último commit (sin subir) manteniendo cambios

```bash
git reset --soft HEAD~1
```

### 4.2 Descartar todos los cambios locales

```bash
git restore . && git clean -fd
```

### 4.3 Commit en la rama equivocada

```bash
git switch -c rama-correcta
git switch -
git reset --hard HEAD~1
```

### 4.4 Dejar de seguir un archivo que debía estar en `.gitignore`

```bash
echo ".env" >> .gitignore
git rm --cached .env
git commit -m "chore: deja de seguir .env"
```

### 4.5 Recuperar algo "perdido"

```bash
git reflog
git switch -c rescate HASH
```

### 4.6 Guardar cambios a medias para cambiar de rama

```bash
git stash push -m "a medias: formulario"
git switch otra-rama
# ... vuelves ...
git stash pop
```

Más en [[Git 06 - Deshacer cambios|Deshacer cambios]].

---

## 5. npm

### 5.1 Reinstalar dependencias desde cero

```bash
rm -rf node_modules && npm ci
```

### 5.2 Ver dependencias desactualizadas y actualizarlas

```bash
npm outdated
npm update
```

### 5.3 Crear un proyecto con Vite

```bash
npm create vite@latest mi-app -- --template vanilla
cd mi-app && npm install && npm run dev
```

### 5.4 Servidor estático rápido para una carpeta

```bash
npx serve .
# o con Python:
python -m http.server 8000
```

Más en [[Node 02 - npm y package.json|npm y package.json]].

---

## 6. Docker

### 6.1 Limpiar todo lo que no se usa

```bash
docker system prune -a --volumes
```

> [!warning] Borra también volúmenes sin usar
> Si un volumen tiene datos de una base de datos que quieres conservar, no uses `--volumes`.

### 6.2 Entrar en un contenedor en marcha

```bash
docker exec -it nombre-contenedor sh
```

### 6.3 Base de datos de prueba en un minuto

```bash
docker run -d --name mysql-dev -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=dev mysql:8

docker run -d --name pg-dev -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres postgres:16
```

### 6.4 Reconstruir y arrancar con Compose

```bash
docker compose up -d --build && docker compose logs -f
```

Más en [[Docker 06 - Referencia rápida|Referencia rápida de Docker]].

---

## 7. Red y APIs

### 7.1 Probar una API con JSON bonito

```bash
curl -s http://localhost:3000/api/productos | python -m json.tool
```

### 7.2 Ver solo el código de estado

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://mi-web.com
```

### 7.3 ¿A qué IP apunta mi dominio?

```bash
nslookup mi-web.com
```

### 7.4 Tu IP pública

```bash
curl -s https://ifconfig.me
```

---

## 8. SSH y despliegue

### 8.1 Crear clave y probarla con GitHub

```bash
ssh-keygen -t ed25519 -C "tu-correo@ejemplo.com"
cat ~/.ssh/id_ed25519.pub     # pegar en GitHub → Settings → SSH keys
ssh -T git@github.com
```

### 8.2 Subir una web estática a un servidor

```bash
npm run build && rsync -avz --delete dist/ usuario@servidor:/var/www/mi-web/
```

### 8.3 Actualizar una app en el servidor

```bash
ssh mi-vps "cd /var/www/app && git pull && npm ci --omit=dev && pm2 restart app"
```

Más en [[Terminal 07 - SSH y servidores|SSH y servidores]].
