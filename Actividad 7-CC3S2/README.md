# Ejercicios guiados
A) Evitar (o no) `--ff`
- Ejecutar una fusion real.
- Pregunta: ¿Cuándo evitarías `--ff` en un equipo y por qué?

Se evita cuando un miembro del equipo fusionó los cambios de su rama a la rama principal, esto significa que la rama principal avanzó desde que nosotros creamos nuestra rama a partir de esta. No se puede hacer fast-forward por que la rama principal tiene commits que nuestra rama no tiene. Entonces se debe crear un commit de fusion(--no-fast-forward).

B) Trabajo en equipo con `--no-ff`
- Crea dos ramas con cambios paralelos y fusiónalas con --no-ff.
- Preguntas: ¿Qué ventajas de trazabilidad aporta? ¿Qué problemas surgen con exceso de merges?

Se registran todos los commits y se crea un commit de fusión, debido al exceso de merges se puede acumular una gran cantidad de commits.

C) Squash con muchos commits
- Haz 3-4 commits en feature-3 y aplánalos con --squash.
- Preguntas: ¿Cuándo conviene? ¿Qué se pierde respecto a merges estándar?
Conviene cuando los cambios son pequeños, ya que se pueden guardar todos los cambios realizados en un solo commit. No se agregan todos los commits de la rama, ni tampoco se crea un commit de fusión.

### Conflictos reales con `--no-fast-forward`
```
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git init
Initialized empty Git repository in /home/ubuntu1/Desktop/desarrollo-de-software/actividad7/ej04/.git/
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ echo "<h1>Proyecto CC3S2</h1>" > index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ ls
index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git commit -m "commit inicial en main"
[main (root-commit) cb8883d] commit inicial en main
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git log
commit cb8883d2ed24499d58f46da79f3b1a070371f61f (HEAD -> main)
Author: sergio <sergio@example.com>
Date:   Mon Oct 6 09:39:50 2025 -0500

    commit inicial en main
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git checkout -b feature-update
Switched to a new branch 'feature-update'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ echo "<h1>Proyecto CC3S2 (feature)</h1>" > index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git commit -m "Cambio realizado desde feature-update"
[feature-update 019e50a] Cambio realizado desde feature-update
 1 file changed, 1 insertion(+), 1 deletion(-)
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git checkout main
Switched to branch 'main'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ echo "<h1>Proyecto CC3S2 (main)</h1>" > index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git commit -m "Cambio realizado desde main"
[main 44e88ad] Cambio realizado desde main
 1 file changed, 1 insertion(+), 1 deletion(-)
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git merge --no-ff feature-update
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   index.html

no changes added to commit (use "git add" and/or "git commit -a")
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git diff
diff --cc index.html
index 2a547e2,e9f19fe..0000000
--- a/index.html
+++ b/index.html
@@@ -1,1 -1,1 +1,5 @@@
++<<<<<<< HEAD
 +<h1>Proyecto CC3S2 (main)</h1>
++=======
+ <h1>Proyecto CC3S2 (feature)</h1>
++>>>>>>> feature-update
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ nano index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git add index.html
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej04$ git commit -m "Fusion main/feature-update"
[main c3fa02f] Fusion main/feature-update
```
Preguntas
- ¿Qué pasos adicionales hiciste para resolverlo?

Editar index.html para solucionar las diferencias.
- ¿Qué prácticas (convenciones, PRs pequeñas, tests) lo evitarían?

Planeación adecuada del sprint, donde se deleguen las responsabilidades de forma explícita. PRs donde los miembros discutan antes de hacer el merge.
## Comparar historiales tras cada método
Preguntas
- ¿Cómo se ve el DAG en cada caso?

En el primero se ven los commits de la rama principal, pero sin commit de fusion. En el tercero se ven todos los commits pero admás se agrega un commit de fusión. En el segundo solo se ven los commits de fusión.

- ¿Qué método prefieres para: trabajo individual, equipo grande, repos con auditoría estricta?

Para trabajo individual es mas común el --fast-forward ya que no nos encontramos con cambios en paralelo, normalmente avanzamos en la rama principal y para añadir una nueva característica creamos una nueva rama, mientras la principal no avanza. En un equipo grande se usa --no-fast-forward, ya que la rama principal puede cambiar mientras avanzamos en una funcionalidad en otra rama, se necesitaría un commit de fusión. Para repos con auditoría estricta se usaría el squash para mantener el historial limpio, ordenado y correctamente documentado.
### Revertir una fusión (solo si HEAD es un merge commit)
```
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej02$ git show -s --format=%P HEAD
4c2f41e869f6ac80ea511a495418790ea599a43a 40d8552f5e8aa7dfd9586ff66e6b0b5460ff1052
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej02$ git revert -m 1 HEAD
[main b4720f6] Revert "Este es un commit de fusion, no solo se mueve el puntero de la rama main"
 1 file changed, 2 deletions(-)
```
Preguntas
- ¿Cuándo usar git revert en vez de git reset?

Cuando se quiere revertir los cambios de un commit en especifico, sin afectar al historial.

- ¿Impacto en un repo compartido con historial público?

Se revierten los cambios, pero el historial se mantiene. Y Se observa el commit que revierte el commit específico.

## Variantes útiles para DevOps/DevSecOps
A) Fast-Forward Only (merge seguro)
```
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git init
Initialized empty Git repository in /home/ubuntu1/Desktop/desarrollo-de-software/actividad7/ej05/.git/
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ echo "Hola mundo" > README
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git commit -m "Commit inicial en main"
[main (root-commit) f97e568] Commit inicial en main
 1 file changed, 1 insertion(+)
 create mode 100644 README
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git checkout -b feature-ffonly
Switched to a new branch 'feature-ffonly'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ echo "cambio en feature-ffonly" >> README.md
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git commit -m "primer commit en feature-ffonly"
[feature-ffonly 5b2f5e1] primer commit en feature-ffonly
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git checkout main
Switched to branch 'main'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git merge --ff-only feature-ffonly
Updating f97e568..5b2f5e1
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ echo "Segundo cambio en main" >> README
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ ñ
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git commit -m "CAmbio en main para forzar fallo"
[main d2d2d73] CAmbio en main para forzar fallo
 1 file changed, 1 insertion(+)
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git checkout feature-ffonly
Switched to branch 'feature-ffonly'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ echo "Segundo cambio en feature-ffonly" >> README
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git add .
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git commit -m "segundo commit en feature-ffonly"
[feature-ffonly 9b96273] segundo commit en feature-ffonly
 1 file changed, 1 insertion(+)
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git checkout main
Switched to branch 'main'
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/actividad7/ej05$ git merge --ff-only feature-ffonly
hint: Diverging branches can't be fast-forwarded, you need to either:
hint:
hint:   git merge --no-ff
hint:
hint: or:
hint:
hint:   git rebase
hint:
hint: Disable this message with "git config advice.diverging false"
fatal: Not possible to fast-forward, aborting.
```
B) Rebase + FF (historial lineal con PRs)
C) Merge con validación previa (sin commitear)
D) Octopus Merge (varias ramas a la vez)
E) Subtree (integrar subproyecto conservando historial)
F) Sesgos de resolución y normalización (algoritmo ORT)
G) Firmar merges/commits (auditoría y cumplimiento)