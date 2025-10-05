# Introducción a Git conceptos básicos y operaciones esenciales
## git config
Sirve para establecer o consultar datos de identificación como nombre y email. Se pueden establecer a nivel 'system'(todos los usuarios y todos los repositorios), a nivel 'global'(todos los repositorios) y a nivel local(solo a un repositorio).
## git init
Sirve para inicializar un repositorio, crea el directorio .git donde se registran todos lo necesario para que el repositorio cumpla sus funciones.
## git add
El repositorio no registra los cambios automáticamente, para esta tarea existe el comando git add, sirve añadir los cambios al repositorio. Se pueden agregar cambios específicos o agregar todos los cambios realizados.
## git commit
Sirve para agrupar un conjunto de cambios realizados y registrarlos con un mensaje y un identificador. Nos sirve para poder volver a este punto de la historia del repositorio.
## git log
Sirve para visualizar los detalles de los commits del repositorio.

$ git log --graph --pretty=format:'%x09 %h %ar ("%an") %s'
*        db7c23c 19 minutes ago ("sergio") Configura la documentación base del repositorio
*        d560f68 26 minutes ago ("sergio") Commit inicial con README.md
Devuelve un identificador, el tiempo transcurrido desde el commit, el username del autor y el mensaje del commit.
## git branch
Sirve para mostrar las ramas del repositorio y para crear nuevas ramas.
## git checkout/git switch
git checkout sirve para volver a un commit especifico, cambiar ramas, crear y cambiarse a la nueva rama. git switch sirve para cambiar de rama.
# git merge
Sirve para fusionar una rama origen con una rama destino, los cambios que se aplicaron en la ramas origen desde que se creo serán introducidos a la rama destino, si existen inconsistencias se corrigen manualmente.
# git branch -d
Después de aplicar los cambios a la rama destino, es posible que la rama origen ya no tenga uso. Para mantener el repositorio limpio se debe borrar esta rama, para eso sirve el comando `git branch -d`.


Preguntas
- ¿Cómo te ha ayudado Git a mantener un historial claro y organizado de tus cambios?

Con los commits, sirven para agrupar una secuencia de cambios, además están adecuadamente registrados.
- ¿Qué beneficios ves en el uso de ramas para desarrollar nuevas características o corregir errores?

Las ramas son independientes las unas a las otras, crear una nueva rama para desarrollar una nueva característica nos da la libertad de cometer errores sin comprometer el trabajo anterior. Para corregir errores, los errores se pueden detectar después de que se haya avanzado sobre esa implementación, con git podemos crear una rama desde la versión más cercana al momento en el que se generó el problema, y no comprometer lo que se ha avanzado a partir de ese punto.
- Realiza una revisión final del historial de commits para asegurarte de que todos los cambios se han registrado correctamente.

```
c301444 (HEAD -> master) Agrega main.py
db7c23c Configura la documentación base del repositorio
d560f68 Commit inicial con README.md
```
- Revisa el uso de ramas y merges para ver cómo Git maneja múltiples líneas de desarrollo.

```
$ git branch -h
usage: git branch [<options>] [-r | -a] [--merged] [--no-merged]
   or: git branch [<options>] [-f] [--recurse-submodules] <branch-name> [<start-point>]
   or: git branch [<options>] [-l] [<pattern>...]
   or: git branch [<options>] [-r] (-d | -D) <branch-name>...
   or: git branch [<options>] (-m | -M) [<old-branch>] <new-branch>
   or: git branch [<options>] (-c | -C) [<old-branch>] <new-branch>
   or: git branch [<options>] [-r | -a] [--points-at]
   or: git branch [<options>] [-r | -a] [--format]

git merge -h
usage: git merge [<options>] [<commit>...]
   or: git merge --abort
   or: git merge --continue
```
## Ejercicios
### Ejercicio 1 (Manejo avanzado de ramas y resolución de conflictos)
Objetivo: Practicar la creación, fusión y eliminación de ramas, así como la resolución de conflictos que puedan surgir durante la fusión.
Instrucciones:
1. Crea una nueva rama llamada feature/advanced-feature desde la rama main:
```
$ git branch feature/advanced-feature
$ git checkout feature/advanced-feature
$ git branch
* feature/advanced-feature
  master
```
2. Modificar archivos en la nueva rama:
```
#Editar main.py
$ git add main.py
$ git commit -m "Agrega la funcion greet como función avanzada"
[feature/advanced-feature eb1062a] Agrega la funcion greet como función avanzada
 1 file changed, 4 insertions(+)
```
3. Simular un desarrollo paralelo en la rama main.
```
$ git checkout master
Switched to branch 'master'
#Editar main.py
$ git add main.py
$ git commit -m "Actualizar el mensaje main.py en la rama master"
[master cd1a7e7] Actualizar el mensaje main.py en la rama master
 1 file changed, 1 insertion(+), 1 deletion(-)
```
4. Intentar fusionar la rama feature/advanced-feature en master.
```
$ git merge feature/advanced-feature
Auto-merging main.py
CONFLICT (content): Merge conflict in main.py
Automatic merge failed; fix conflicts and then commit the result.
```
5. Resolver el conflicto de fusión:
```
#Resolver conflicto generado en el archivo main.py
$ git add main.py
$ git commit -m "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
[master d8c2e86] Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature
```
6. Eliminar la rama fusionada.
```
$ git branch -d feature/advanced-feature
Deleted branch feature/advanced-feature (was eb1062a).
```
### Ejercicio 2(Exploración y manipulación del historial de commits)
Objetivo: Aprender a navegar y manipular el historial de commits usando comandos avanzados de Git.
1. Ver el historial detallado de commits.
```
$ git log -p
commit d8c2e86802a867b21a4af9e884b7d796b82b7527 (HEAD -> master)
Merge: cd1a7e7 eb1062a
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 19:49:56 2025 -0500

    Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature

commit cd1a7e7f377fdb25c69348ea2f7fe1a3aaf7f83b
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 19:43:06 2025 -0500

    Actualizar el mensaje main.py en la rama master

diff --git a/main.py b/main.py
index df1dc68..3b4d580 100644
--- a/main.py
+++ b/main.py
@@ -1 +1 @@
-print('Hello World')
+print('Hello World-actualizado en master')

commit eb1062a0202e18875680d435a308e96ae81ac15b
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 19:38:34 2025 -0500

    Agrega la funcion greet como función avanzada

diff --git a/main.py b/main.py
index df1dc68..62cdb7b 100644
--- a/main.py
+++ b/main.py
@@ -1 +1,5 @@
 print('Hello World')
+def greet():
+    print('Hello como una función avanzada')
+
+greet()

commit c30144484bc90c04f4950378d071d6a5c38144bf
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 17:56:02 2025 -0500

    Agrega main.py

diff --git a/main.py b/main.py
new file mode 100644
index 0000000..df1dc68
--- /dev/null
+++ b/main.py
@@ -0,0 +1 @@
+print('Hello World')

commit db7c23c64e8b53eb136b7e2f8470619b95f8ead4
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 17:34:06 2025 -0500

    Configura la documentación base del repositorio

diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md
new file mode 100644
index 0000000..2e8cc63
--- /dev/null
+++ b/CONTRIBUTING.md
@@ -0,0 +1 @@
+ CONTRIBUTING
diff --git a/README.md b/README.md
index 2772834..159a628 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,3 @@
  README
+
+Bienvenido al proyecto

commit d560f683d37a3b4c53e79a26d8ed9793834ca368
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 17:26:51 2025 -0500

    Commit inicial con README.md

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..2772834
--- /dev/null
+++ b/README.md
@@ -0,0 +1 @@
+ README
```
2. Filtrar commits por autor
```
$ git log --author="sergio"
Merge: cd1a7e7 eb1062a
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 19:49:56 2025 -0500

    Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature

commit cd1a7e7f377fdb25c69348ea2f7fe1a3aaf7f83b
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 19:43:06 2025 -0500

    Actualizar el mensaje main.py en la rama master

commit eb1062a0202e18875680d435a308e96ae81ac15b
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 19:38:34 2025 -0500

    Agrega la funcion greet como función avanzada

commit c30144484bc90c04f4950378d071d6a5c38144bf
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 17:56:02 2025 -0500

    Agrega main.py

commit db7c23c64e8b53eb136b7e2f8470619b95f8ead4
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 17:34:06 2025 -0500

    Configura la documentación base del repositorio

commit d560f683d37a3b4c53e79a26d8ed9793834ca368
Author: sergio <sergio@example.com>
Date:   Sun Sep 14 17:26:51 2025 -0500

    Commit inicial con README.md
```
3. Revertir un commit.
```
$ git revert HEAD
error: commit d8c2e86802a867b21a4af9e884b7d796b82b7527 is a merge but no -m option was given.
fatal: revert failed
$ git revert -m 1 HEAD
#<nano
Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"

This reverts commit d8c2e86802a867b21a4af9e884b7d796b82b7527, reversing
changes made to eb1062a0202e18875680d435a308e96ae81ac15b.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Changes to be committed:
#       modified:   main.py
#
Este es el commit de revert del HEAD
#nano>
[master 11800b8] Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
 1 file changed, 2 deletions(-)
$ git log --oneline
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git log --oneline
11800b8 (HEAD -> master) Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
d8c2e86 Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature
cd1a7e7 Actualizar el mensaje main.py en la rama master
eb1062a Agrega la funcion greet como función avanzada
c301444 Agrega main.py
db7c23c Configura la documentación base del repositorio
d560f68 Commit inicial con README.md
```
4. Rebase interactivo.
```
$ git rebase -i HEAD~3
#<nano
pick cd1a7e7 Actualizar el mensaje main.py en la rama master
squash eb1062a Agrega la funcion greet como función avanzada
squash 11800b8 Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
#nano>
Auto-merging main.py
CONFLICT (content): Merge conflict in main.py
error: could not apply eb1062a... Agrega la funcion greet como función avanzada
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply eb1062a... Agrega la funcion greet como función avanzada
#Solucionar conflictos en main.py
$ git add main.py
$ git rebase --continue
#<nano
# This is a combination of 2 commits.
# This is the 1st commit message:

Actualizar el mensaje main.py en la rama master

# This is the commit message #2:

Agrega la funcion greet como función avanzada

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sun Sep 14 19:43:06 2025 -0500
#
# interactive rebase in progress; onto c301444
# Last commands done (2 commands done):
#    pick cd1a7e7 Actualizar el mensaje main.py en la rama master
#    squash eb1062a Agrega la funcion greet como función avanzada
# Next command to do (1 remaining command):
#    squash 11800b8 Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
# You are currently rebasing branch 'master' on 'c301444'.
#
# Changes to be committed:
#       modified:   main.py
#
Este es el mensaje del commit resultante de la mezcla de los commits
#nano>
[detached HEAD 789bb49] Actualizar el mensaje main.py en la rama master
 Date: Sun Sep 14 19:43:06 2025 -0500
 1 file changed, 6 insertions(+)
Auto-merging main.py
CONFLICT (content): Merge conflict in main.py
error: could not apply 11800b8... Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 11800b8... Revert "Resuelve el conflicto de fusión entre la versión main y feature/advanced-feature"
#Solucionar conflictos en main.py
$ git add main.py
$ git rebase --continue
Successfully rebased and updated refs/heads/master.
```
5. Visualización gráfica del historial.
```
$ git log --graph --oneline --all
* 789bb49 (HEAD -> master) Actualizar el mensaje main.py en la rama master
* c301444 Agrega main.py
* db7c23c Configura la documentación base del repositorio
* d560f68 Commit inicial con README.md
```
### Ejercicio 3(Creación y gestión de ramas desde commits específico)
Objetivo: Practicar la creación de ramas desde commits específicos y comprender cómo Git maneja las referencias históricas.
1. Crea una nueva rama desde un commit específico.
```
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git log --oneline
789bb49 (HEAD -> master) Actualizar el mensaje main.py en la rama master
c301444 Agrega main.py
db7c23c Configura la documentación base del repositorio
d560f68 Commit inicial con README.md
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git branch bugfix/rollback-feature c301444
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git checkout bugfix/rollback-feature
Switched to branch 'bugfix/rollback-feature'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git branch
* bugfix/rollback-feature
  master
```
2. Modificar y confirmar cambios en la nueva rama.
```
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git add main.py
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git commit -m "Corregir error en la funcionalidad de rollback"
[bugfix/rollback-feature abc079f] Corregir error en la funcionalidad de rollback
 1 file changed, 3 insertions(+)
```
3. Fusionar los cambios en la rama principal.
```
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git checkout master
Switched to branch 'master'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git merge bugfix/rollback-feature
Auto-merging main.py
CONFLICT (content): Merge conflict in main.py
Automatic merge failed; fix conflicts and then commit the result.
## Corregir conflictos manualmente
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git add main.py
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git status
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   main.py

ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git commit -m "Conflictos corregidos en main.py"
[master dfaf92a] Conflictos corregidos en main.py
```
4. Explorar el historial después de la fusión.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad6$ git log --graph --oneline
*   dfaf92a (HEAD -> master) Conflictos corregidos en main.py
|\
| * abc079f (bugfix/rollback-feature) Corregir error en la funcionalidad de rollback
* | 789bb49 Actualizar el mensaje main.py en la rama master
|/
* c301444 Agrega main.py
* db7c23c Configura la documentación base del repositorio
* d560f68 Commit inicial con README.md
```







