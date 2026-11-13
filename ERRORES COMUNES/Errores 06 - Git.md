# 06 - Errores de Git

> [!info] ¿Qué es?
> Los errores y situaciones de Git más frecuentes, con el **mensaje literal**, qué significa y cómo salir de ellos. Antes de nada: `git status` casi siempre explica en qué estado estás y sugiere el comando para continuar.

---

## 1. Antes de empezar: tranquilidad

- Si algo tuvo **commit**, casi seguro se puede recuperar con `git reflog`.
- Antes de un comando destructivo (`reset --hard`, `clean -fd`, `push --force`), haz una copia: `git stash` o `git branch copia-seguridad`.
- Si estás a medias en un merge o rebase y te has perdido: `git merge --abort` / `git rebase --abort`.

Más en [[Git 06 - Deshacer cambios|Deshacer cambios]].

---

## 2. Subir y bajar cambios

### 2.1 `! [rejected] main -> main (fetch first)` / `(non-fast-forward)`

**Qué significa:** el remoto tiene commits que tú no tienes; Git no te deja sobrescribirlos.

**Solución:**

```bash
git pull --rebase
git push
```

(Si aparecen conflictos, ver 3.1.)

### 2.2 `fatal: The current branch X has no upstream branch.`

**Qué significa:** la rama local no está enlazada con ninguna rama remota.

**Solución:**

```bash
git push -u origin X
```

### 2.3 `fatal: refusing to merge unrelated histories`

**Qué significa:** intentas unir dos repositorios sin historia común (por ejemplo, creaste el repo en GitHub con un README y también hiciste `git init` en local).

**Solución:**

```bash
git pull origin main --allow-unrelated-histories
```

### 2.4 `error: src refspec main does not match any`

**Causas típicas:** no has hecho ningún commit todavía, o tu rama se llama `master` y no `main`.

**Solución:** haz un commit, o comprueba el nombre con `git branch` (puedes renombrarla: `git branch -M main`).

### 2.5 `fatal: 'origin' does not appear to be a git repository`

**Solución:** añade el remoto: `git remote add origin URL` (comprueba con `git remote -v`).

### 2.6 `remote: Repository not found.`

**Causas típicas:** URL mal escrita, el repositorio es privado y no tienes acceso, o estás autenticado con otra cuenta.

---

## 3. Conflictos y fusiones

### 3.1 `CONFLICT (content): Merge conflict in archivo`

**Qué significa:** las dos ramas cambiaron las mismas líneas.

**Solución:**

1. Abre el archivo y busca `<<<<<<<`, `=======`, `>>>>>>>`.
2. Deja la versión correcta y **borra las marcas**.
3. `git add archivo`.
4. `git commit` (merge) o `git rebase --continue` (rebase).

Más en [[Git 04 - Fusionar y resolver conflictos|Fusionar y resolver conflictos]].

### 3.2 `error: Your local changes to the following files would be overwritten by merge/checkout`

**Qué significa:** tienes cambios sin guardar que chocarían con los que vas a traer o con la otra rama.

**Solución:**

```bash
git stash          # guarda tus cambios
git pull           # o git switch otra-rama
git stash pop      # los recuperas
```

### 3.3 `error: you need to resolve your current index first`

**Qué significa:** hay un merge o rebase a medias con conflictos sin resolver.

**Solución:** resuelve y haz `git add`, o aborta con `git merge --abort`.

### 3.4 Se abre un editor raro (Vim) al hacer merge o commit

**Solución:** escribe el mensaje (o déjalo), pulsa `Esc`, escribe `:wq` y Enter. Para usar VS Code: `git config --global core.editor "code --wait"`.

---

## 4. Autenticación

### 4.1 `remote: Support for password authentication was removed.` / `Authentication failed`

**Qué significa:** GitHub ya no acepta la contraseña de la cuenta por HTTPS.

**Solución:** usa SSH (recomendado) o un *Personal Access Token* como contraseña.

```bash
git remote set-url origin git@github.com:usuario/repo.git
```

### 4.2 `git@github.com: Permission denied (publickey).`

**Causas típicas:** no tienes clave SSH, no está añadida a GitHub, o el agente no la carga.

**Solución:**

```bash
ls ~/.ssh                     # ¿existe id_ed25519?
ssh-keygen -t ed25519 -C "tu-correo@ejemplo.com"   # si no existe
cat ~/.ssh/id_ed25519.pub     # pegar en GitHub → Settings → SSH keys
ssh -T git@github.com         # comprobar
```

Más en [[Terminal 07 - SSH y servidores|SSH y servidores]].

### 4.3 Los commits aparecen con otro nombre o sin foto en GitHub

**Qué significa:** el `user.email` configurado no coincide con el de tu cuenta de GitHub.

**Solución:** `git config --global user.email "el-de-github@ejemplo.com"`.

---

## 5. Estados extraños

### 5.1 `You are in 'detached HEAD' state.`

**Qué significa:** estás en un commit concreto, no en una rama. Los commits que hagas aquí pueden perderse.

**Solución:**

```bash
git switch main                   # volver a una rama
git switch -c rama-nueva          # o guardar lo hecho en una rama nueva
```

### 5.2 `fatal: not a git repository (or any of the parent directories): .git`

**Qué significa:** estás en una carpeta que no es un repositorio.

**Solución:** muévete a la carpeta del proyecto (`cd`) o inicia uno (`git init`).

### 5.3 `fatal: Unable to create '.git/index.lock': File exists.`

**Qué significa:** otro proceso de Git se quedó colgado (o sigue en marcha, por ejemplo el editor).

**Solución:** cierra otros programas que usen Git y, si no hay ninguno, borra el archivo: `rm .git/index.lock`.

### 5.4 `warning: LF will be replaced by CRLF`

**Qué significa:** aviso de conversión de saltos de línea entre Windows y Linux. No es un error.

**Solución:** configura `git config --global core.autocrlf true` (Windows) o añade un `.gitattributes` con `* text=auto`.

---

## 6. Archivos que no deberían estar

### 6.1 `node_modules` o `.env` aparecen en el repositorio

**Solución:**

```bash
echo "node_modules/" >> .gitignore
echo ".env" >> .gitignore
git rm -r --cached node_modules .env
git commit -m "chore: deja de seguir node_modules y .env"
```

Si `.env` tenía secretos y se subió: **cámbialos**, siguen en el historial.

### 6.2 El `.gitignore` no ignora un archivo

**Qué significa:** el archivo ya estaba seguido antes de añadirlo al `.gitignore`.

**Solución:** `git rm --cached archivo` y commit. Comprueba reglas con `git check-ignore -v archivo`.

### 6.3 `remote: error: File X is 123.45 MB; this exceeds GitHub's file size limit of 100.00 MB`

**Solución:** quita el archivo del último commit y añádelo al `.gitignore`:

```bash
git rm --cached archivo-grande
git commit --amend --no-edit
git push
```

Si está en commits más antiguos, hace falta reescribir el historial (`git filter-repo`). Para archivos grandes legítimos, usa **Git LFS**.

---

## 7. Commits equivocados

### 7.1 Mensaje de commit equivocado (sin push)

```bash
git commit --amend -m "Mensaje correcto"
```

### 7.2 Olvidé un archivo en el último commit (sin push)

```bash
git add olvidado.js
git commit --amend --no-edit
```

### 7.3 Commit en la rama equivocada

```bash
git switch -c rama-correcta   # la rama nueva se queda con el commit
git switch -
git reset --hard HEAD~1       # lo quitas de la rama original
```

### 7.4 Ya hice push de un commit con errores

```bash
git revert HASH
git push
```

---

## 8. Resumen

- **Push rechazado** → `git pull --rebase` y vuelve a hacer push.
- **Sin upstream** → `git push -u origin rama`.
- **Conflicto** → editar, borrar marcas, `git add`, continuar.
- **Cambios locales se sobrescribirían** → `git stash` antes.
- **Autenticación** → usa SSH o un token; nunca la contraseña.
- **Detached HEAD** → `git switch -c rama-nueva` para no perder nada.
- **Archivo que no debía subirse** → `.gitignore` + `git rm --cached`.
- Casi todo se recupera con `git reflog`.
