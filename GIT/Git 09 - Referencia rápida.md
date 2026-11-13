# 09 - Referencia rápida

> [!info] ¿Qué es?
> Chuleta de **Git** para consultar rápido. No explica, solo recuerda. Si algo no te suena, sigue el enlace a su nota.

---

## 1. Configuración e inicio ([[Git 01 - Fundamentos]])

```bash
git config --global user.name "Nombre"
git config --global user.email "correo"
git config --global init.defaultBranch main
git init
git clone URL
```

---

## 2. Ciclo diario ([[Git 02 - Comandos básicos]])

```bash
git status  |  git status -s
git diff  |  git diff --staged
git add archivo  |  git add .  |  git add -p
git commit -m "mensaje"
git commit --amend            # corregir último commit (sin push)
git log --oneline --graph --all
git show hash
git rm --cached archivo       # dejar de seguir
```

---

## 3. Ramas ([[Git 03 - Ramas]])

```bash
git branch                    # listar
git branch -a                 # incluir remotas
git switch -c rama            # crear y cambiar
git switch rama               # cambiar
git switch -                  # rama anterior
git branch -m nuevo-nombre    # renombrar
git branch -d rama            # borrar (fusionada)
git branch -D rama            # forzar borrado
```

---

## 4. Fusionar ([[Git 04 - Fusionar y resolver conflictos]])

```bash
git merge rama
git merge --squash rama
git merge --abort
git rebase main
git rebase --continue  |  --abort
git rebase -i HEAD~3
git checkout --ours archivo  |  --theirs archivo
```

Conflicto: editar → borrar `<<<<<<<` `=======` `>>>>>>>` → `git add` → `git commit`.

---

## 5. Remotos ([[Git 05 - Repositorios remotos y GitHub]])

```bash
git remote -v
git remote add origin URL
git remote set-url origin URL
git push -u origin rama       # primera vez
git push
git push --force-with-lease
git push origin --delete rama
git fetch
git pull  |  git pull --rebase
git tag v1.0.0 && git push origin v1.0.0
```

---

## 6. Deshacer ([[Git 06 - Deshacer cambios]])

| Situación | Comando |
|---|---|
| Descartar cambios de un archivo | `git restore archivo` |
| Quitar del staging | `git restore --staged archivo` |
| Deshacer último commit (mantener cambios) | `git reset --soft HEAD~1` |
| Deshacer último commit (borrar cambios) | `git reset --hard HEAD~1` |
| Deshacer commit ya subido | `git revert hash` |
| Guardar temporalmente | `git stash` / `git stash pop` |
| Recuperar lo "perdido" | `git reflog` |
| Borrar archivos no seguidos | `git clean -n` → `git clean -fd` |

---

## 7. `.gitignore` ([[Git 07 - .gitignore y configuración]])

```gitignore
node_modules/
.env
!.env.example
dist/
*.log
.DS_Store
```

```bash
git check-ignore -v archivo
git status --ignored
git config --global alias.lg "log --oneline --graph --all"
```

---

## 8. Commits ([[Git 08 - Flujo de trabajo y buenas prácticas]])

| Tipo | Uso |
|---|---|
| `feat:` | Funcionalidad |
| `fix:` | Corrección |
| `docs:` | Documentación |
| `style:` | Formato |
| `refactor:` | Reorganizar |
| `test:` | Tests |
| `chore:` | Mantenimiento |
