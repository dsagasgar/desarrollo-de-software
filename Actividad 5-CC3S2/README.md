# Parte 1-Construir:
## Ejercicio 1
Ejecuta make help y guarda la salida para análisis. Luego inspecciona .DEFAULT_GOAL y .PHONY dentro del Makefile. Comandos:
```bash
mkdir -p logs evidencia
make help | tee logs/make-help.txt
grep -E '^\.(DEFAULT_GOAL|PHONY):' -n Makefile | tee -a logs/make-help.txt
```
La etiqueta help imprime todas las etiquetas y un comentario explicando su utilidad. DEFAULT_GOAL es una variable de make que sirve para indicar cual será la etiqueta o objetivo que se ejecutará al correr make sin argumentos, al asignarle `help` como valor, esté será la etiqueta que se ejecute por defecto. PHONY sirve para asegurar la ejecución de las etiquetas declaradas, make no buscará la existencia de un archivo con ese nombre, simplemente ejecutará la etiqueta cada vez que sea invocada.
## Ejercicio 2
Comprueba la generación e idempotencia de build. Limpia salidas previas, ejecuta build, verifica el contenido y repite build para constatar que no rehace nada si no cambió la fuente. Comandos:
```bash
rm -rf out dist
make build | tee logs/build-run1.txt
cat out/hello.txt | tee evidencia/out-hello-run1.txt
make build | tee logs/build-run2.txt
stat -c '%y %n' out/hello.txt | tee -a logs/build-run2.txt
```
La primera ejecución llama a build, el cuál tiene como dependencia el archivo out/hello.txt, make busca este objetivo. El objetivo out/hello.py depende de src/hello.py, ejecuta el script y guarda la salida en out/hello.txt, una vez creado este archivo, se acaba la ejecución ya que este era el unico objetivo de build.
La segunda ejecución vuelve a llamar a build, build apunta a out/hello.txt, out/hello.txt(ya existe) apunta a src/hello.py, make comprueba que este script no ha sido modificado, como el archivo de salida ya existe y el script no ha sido modificado, make no vuelve a ejecutar el objetivo out/hello.txt, finaliza la ejecución.
## Ejercicio 3
Fuerza un fallo controlado para observar el modo estricto del shell y .DELETE_ON_ERROR. Sobrescribe PYTHON con un intérprete inexistente y verifica que no quede artefacto corrupto. Comandos:
```bash
rm -f out/hello.txt
PYTHON=python4 make build ; echo "exit=$?" | tee logs/fallo-python4.txt || echo "falló (esperado)"
ls -l out/hello.txt | tee -a logs/fallo-python4.txt || echo "no existe (correcto)"
```
`set -euo pipefail` sirve para activar el modo estricto en bash, detiene la ejecución ante cualquier error, propaga los errores en los pipelines y por ultimo produce un error cuando existe una variable no declarada. Al intentar ejecutar el script src/hello.py con python4(no existe) se produce un error que detiene el flujo del programa, .DELETE_ON_ERROR hace que make evite los estados inconsistentes, es decir elimina los archivos corruptos que quedan al ocurrir un error y detenerse el flujo del programa, en este caso como el error ocurre en el objetivo out/hello.txt, make borra este archivo después de producirse el error.
## Ejercicio 4
Realiza un "ensayo" (dry-run) y una depuración detallada para observar el razonamiento de Make al decidir si rehacer o no. Comandos:
```bash
make -n build | tee logs/dry-run-build.txt
make -d build |& tee logs/make-d.txt
grep -n "Considerando el archivo objetivo 'out/hello.txt'" logs/make-d.txt
```
`make -n` hace un dry-run, es decir simula que ejecuta el flujo del programa, mostrando los pasos que se realizarían se se ejecutara. El `make -d` sirve para ejecutar y mostrar información detallada de la ejecución. Primero busca los makefiles, encuentra Makefile, busca reglas implicitas para este archivo, no las encuentra, no rehace el Makefile. Ahora busca el objetivo build, el objetivo build tiene como dependencia el archivo out/hello.txt, el archivo no existe, busca el objetivo out/hello.txt que tiene como dependencia a src/hello.py, buscar reglas implícitas para src/hello.py, no las encuentra, por lo tanto no vuelve a crear este archivo. Vuelve al anterior, out/hello.txt ejecuta las instrucciones de este objetivo, al terminar se completan los requerimientos del objetivo build, no hay instrucciones, se termina la ejecución de forma exitosa.
## Ejercicio 5
Demuestra la incrementalidad con marcas de tiempo. Primero toca la fuente y luego el target para comparar comportamientos. Comandos:
```bash
touch src/hello.py
make build | tee logs/rebuild-after-touch-src.txt

touch out/hello.txt
make build | tee logs/no-rebuild-after-touch-out.txt
```
Make compara marcas de tiempo, al hacer touch al script src/hello.py, en el makefile está definido que out/hello.txt depende de src/hello.py, entonces compara la fecha de última modificación, como la fuente ha sido tocada, entonces tiene fecha de modificación más reciente que el target, por esto se tiene que rehacer el target. En cambio cuando se toca el target, make compara fechas y encuentra que la fecha de modificación del target es más reciente que la de la fuente, make concluye que no es necesario rehacer el target.
## Ejercicio 6
Ejecuta verificación de estilo/formato manual (sin objetivos lint/tools). Si las herramientas están instaladas, muestra sus diagnósticos; si no, deja evidencia de su ausencia. Comandos:
```bash
command -v shellcheck >/dev/null && shellcheck scripts/run_tests.sh | tee logs/lint-shellcheck.txt || echo "shellcheck no instalado" | tee logs/lint-shellcheck.txt
command -v shfmt >/dev/null && shfmt -d scripts/run_tests.sh | tee logs/format-shfmt.txt || echo "shfmt no instalado" | tee logs/format-shfmt.txt
```
Shellcheck no prodece ninguna salida, seguramente no encontro ningún problema ni error en el script. Igualmente shfmt no muestra el diff.
## Ejercicio 7
Construye un paquete reproducible de forma manual, fijando metadatos para que el hash no cambie entre corridas idénticas. Repite el empaquetado y compara hashes. Comandos:
```bash
mkdir -p dist
tar --sort=name --mtime='@0' --owner=0 --group=0 --numeric-owner -cf dist/app.tar src/hello.py
gzip -n -9 -c dist/app.tar > dist/app.tar.gz
sha256sum dist/app.tar.gz | tee logs/sha256-1.txt

rm -f dist/app.tar.gz
tar --sort=name --mtime='@0' --owner=0 --group=0 --numeric-owner -cf dist/app.tar src/hello.py
gzip -n -9 -c dist/app.tar > dist/app.tar.gz
sha256sum dist/app.tar.gz | tee logs/sha256-2.txt

diff -u logs/sha256-1.txt logs/sha256-2.txt | tee logs/sha256-diff.txt || true
```
El hash es: `da0ee78e63abe78a9ea0dd4d8962f2ae636d2fddef082ff9fa115a60df236882` , la flag --sort=name hace que se ordenen los archivos usando el nombre, --mtime='@0' sirve para poner la fecha de ultima modificacion en segundos desde la epoca unix, --owner y --group asignan user id y group id para todos los archivos, 0 es el UID GID de root, --numeric-owner sirve para guardar propietario usando UID, elimina variabilidad porque ejecutar el mismo comando nos va dar el mismo resultado. Esto se comprueba al comparar el hash.
## Ejercicio 8
Reproduce el error clásico "missing separator" sin tocar el Makefile original. Crea una copia, cambia el TAB inicial de una receta por espacios, y confirma el error. Comandos:
```bash
cp Makefile Makefile_bad
# (Edita Makefile_bad: en la línea de la receta de out/hello.txt, reemplaza el TAB inicial por espacios)
make -f Makefile_bad build |& tee evidencia/missing-separator.txt || echo "error reproducido (correcto)"
```
Exige un tab al inicio de las lineas de receta por es la forma en la que make las diferencia de variables, comentarios o targets. Cuando ocurre este problema, make envia el siguiente mensaje de error: `*** missing separator`, esta es la forma de diagnosticar el problema que causa el fallo.
## Script de bash
- Ejecuta ./scripts/run_tests.sh en un repositorio limpio. Observa las líneas "Demostrando pipefail": primero sin y luego con pipefail. Verifica que imprime "Test pasó" y termina exitosamente con código 0 (echo $?).
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ ./scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ echo $?
0
```
- Edita src/hello.py para que no imprima "Hello, World!". Ejecuta el script: verás "Test falló", moverá hello.py a hello.py.bak, y el trap lo restaurará. Confirma código 2 y ausencia de .bak.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ ./scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test falló: salida inesperada
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ echo $?
2
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ cd src/;ls
__init__.py  hello.py
```
- Ejecuta bash -x scripts/run_tests.sh. Revisa el trace: expansión de tmp y PY, llamadas a funciones, here-doc y tuberías. Observa el trap armado al inicio y ejecutándose al final; estado 0.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ bash -x scripts/run_tests.sh
+ set -euo pipefail
+ IFS='
        '
+ umask 027
+ set -o noclobber
+ PY=python3
+ SRC_DIR=src
++ mktemp
+ tmp=/tmp/tmp.crkHGznLR1
+ trap 'cleanup $?' EXIT INT TERM
+ echo 'Demostrando pipefail:'
Demostrando pipefail:
+ set +o pipefail
+ false
+ true
+ echo 'Sin pipefail: el pipe se considera exitoso (status 0).'
Sin pipefail: el pipe se considera exitoso (status 0).
+ set -o pipefail
+ false
+ true
+ echo 'Con pipefail: se detecta el fallo (status != 0).'
Con pipefail: se detecta el fallo (status != 0).
+ cat
+ check_deps
+ deps=('python3' 'grep')
+ local -a deps
+ for dep in "${deps[@]}"
+ command -v python3
+ for dep in "${deps[@]}"
+ command -v grep
+ run_tests src/hello.py
+ local script=src/hello.py
+ local output
++ python3 src/hello.py
+ output='Hello, World!'
+ echo 'Hello, World!'
+ grep -Fq 'Hello, World!'
+ echo 'Test pasó: Hello, World!'
Test pasó: Hello, World!
+ cleanup 0
+ rc=0
+ rm -f /tmp/tmp.crkHGznLR1
+ '[' -f src/hello.py.bak ']'
+ exit 0
```
- Sustituye output=$("$PY" "$script") por ("$PY" "$script"). Ejecuta script. output queda indefinida; con set -u, al referenciarla en echo aborta antes de grep. El trap limpia y devuelve código distinto no-cero.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ ./scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Hello, World!
./scripts/run_tests.sh: line 45: output: unbound variable
./scripts/run_tests.sh: line 19: wait_for: No record of process 33895
Test falló: salida inesperada
```

# Parte 2-Leer:
# Parte 3-Extender: