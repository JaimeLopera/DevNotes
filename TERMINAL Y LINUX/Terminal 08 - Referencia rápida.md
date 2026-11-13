# 08 - Referencia rápida

> [!info] ¿Qué es?
> Chuleta de **Terminal y Linux** para consultar rápido. No explica, solo recuerda. Si algo no te suena, sigue el enlace a su nota.

---

## 1. Fundamentos ([[Terminal 01 - Fundamentos de la terminal]])

| Atajo | Acción |
|---|---|
| `Tab` | Autocompletar |
| `↑` / `↓` | Historial |
| `Ctrl + C` | Cancelar |
| `Ctrl + L` | Limpiar pantalla |
| `Ctrl + R` | Buscar en el historial |

```bash
comando --help
man comando
which comando
echo $PATH
history
!!                     # repetir último comando
cmd1 && cmd2           # cmd2 solo si cmd1 funciona
cmd1 || cmd2           # cmd2 solo si cmd1 falla
```

---

## 2. Navegación y archivos ([[Terminal 02 - Navegación y archivos]])

```bash
pwd                    # dónde estoy
ls -lah                # listar todo con detalles
cd ruta  |  cd ..  |  cd ~  |  cd -
mkdir -p a/b/c         # crear ruta completa
touch archivo          # crear archivo vacío
cp a b  |  cp -r dir1 dir2
mv a b                 # mover o renombrar
rm archivo  |  rm -r carpeta
cat  |  less  |  head -n 20  |  tail -n 20  |  tail -f log
```

| Símbolo | Significa |
|---|---|
| `/` | Raíz |
| `~` | Carpeta personal |
| `.` / `..` | Actual / padre |
| `*` `?` `{a,b}` | Comodines |

---

## 3. Permisos ([[Terminal 03 - Permisos y usuarios]])

```bash
ls -l                  # ver permisos
chmod +x script.sh     # hacer ejecutable
chmod 755 carpeta      # rwxr-xr-x
chmod 644 archivo      # rw-r--r--
chmod 600 clave        # rw-------
chown usuario:grupo archivo
sudo comando
whoami  |  id
```

| Número | Permisos |
|---|---|
| 7 | rwx |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |

---

## 4. Buscar y procesar texto ([[Terminal 04 - Buscar y procesar texto]])

```bash
grep -rn "texto" carpeta/
grep -i  |  -v  |  -c  |  -l  |  -w  |  -E
find . -name "*.js"
find . -type d -name node_modules
find . -type f -size +10M
sed 's/viejo/nuevo/g' archivo
sed -i 's/viejo/nuevo/g' archivo
awk '{print $1}' archivo
sort | uniq -c | sort -rn | head
wc -l archivo
```

| Símbolo | Uso |
|---|---|
| `\|` | Pasar salida a otro comando |
| `>` | Guardar (sobrescribe) |
| `>>` | Añadir al final |
| `2>&1` | Juntar errores y salida |
| `/dev/null` | Descartar |

---

## 5. Procesos y red ([[Terminal 05 - Procesos y red]])

```bash
ps aux | grep node
pgrep -a node
top  |  htop
kill PID  |  kill -9 PID  |  pkill node
comando &  |  jobs  |  fg  |  bg
lsof -i :3000          # quién usa el puerto
ss -tulpn              # puertos en escucha
curl -i url
curl -X POST url -H "Content-Type: application/json" -d '{"a":1}'
ping dominio  |  dig dominio
df -h  |  du -sh carpeta  |  free -h
```

---

## 6. Bash scripting ([[Terminal 06 - Bash scripting]])

```bash
#!/usr/bin/env bash
set -euo pipefail

nombre="Ana"
fecha=$(date +%Y-%m-%d)

if [ -f "$archivo" ]; then ... fi
for x in a b c; do echo "$x"; done
while [ $n -lt 5 ]; do n=$((n+1)); done

funcion() { local x="$1"; echo "$x"; }
```

| Variable | Contenido |
|---|---|
| `$1`, `$2` | Argumentos |
| `$#` | Nº de argumentos |
| `$@` | Todos los argumentos |
| `$?` | Código de salida (0 = OK) |

---

## 7. SSH ([[Terminal 07 - SSH y servidores]])

```bash
ssh-keygen -t ed25519 -C "correo"
ssh-copy-id usuario@servidor
ssh usuario@servidor
ssh -T git@github.com
scp archivo usuario@servidor:/ruta/
rsync -avz --delete dist/ servidor:/var/www/web/
```

---

## 8. Comprimir

```bash
tar -czf archivo.tar.gz carpeta/   # comprimir
tar -xzf archivo.tar.gz            # descomprimir
zip -r archivo.zip carpeta/
unzip archivo.zip
```
