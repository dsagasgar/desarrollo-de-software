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
Ejercicios:
- Ejecuta make -n all para un dry-run que muestre comandos sin ejecutarlos; identifica expansiones $@ y $<, el orden de objetivos y cómo all encadena tools, lint, build, test, package.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make -n all
command -v python3 >/dev/null || { echo "Falta python3"; exit 1; }
command -v shellcheck >/dev/null || { echo "Falta shellcheck"; exit 1; }
command -v shfmt >/dev/null || { echo "Falta shfmt"; exit 1; }
command -v grep >/dev/null || { echo "Falta grep"; exit 1; }
command -v awk >/dev/null || { echo "Falta awk"; exit 1; }
command -v tar >/dev/null || { echo "Falta tar"; exit 1; }
tar --version 2>/dev/null | grep -q 'GNU tar' || { echo "Se requiere GNU tar"; exit 1; }
command -v sha256sum >/dev/null || { echo "Falta sha256sum"; exit 1; }
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
command -v ruff >/dev/null 2>&1 && ruff check src || echo "ruff no instalado; omitiendo lint Python"
mkdir -p out
python3 src/hello.py > out/hello.txt
scripts/run_tests.sh
python3 -m unittest discover -s tests -v
mkdir -p dist
tar --sort=name --owner=0 --group=0 --numeric-owner --mtime='UTC 1970-01-01' -czf dist/app.tar.gz -C out hello.txt
```
- Ejecuta make -d build y localiza líneas "Considerando el archivo objetivo" y "Debe deshacerse", explica por qué recompila o no out/hello.txt usando marcas de tiempo y cómo mkdir -p $(@D) garantiza el directorio.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ echo $?
0
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make -d build
GNU Make 4.3
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Reading makefile 'Makefile'...
Updating makefiles....
 Considering target file 'Makefile'.                # Considerando archivo
  Looking for an implicit rule for 'Makefile'.
  No implicit rule found for 'Makefile'.
  Finished prerequisites of target file 'Makefile'.
 No need to remake target 'Makefile'.
Updating goal targets....
Considering target file 'build'.
 File 'build' does not exist.
  Considering target file 'out/hello.txt'.          # Considerando archivo
    Considering target file 'src/hello.py'.         # Considerando archivo
     Looking for an implicit rule for 'src/hello.py'.
     No implicit rule found for 'src/hello.py'.
     Finished prerequisites of target file 'src/hello.py'.
    No need to remake target 'src/hello.py'.
   Finished prerequisites of target file 'out/hello.txt'.
   Prerequisite 'src/hello.py' is newer than target 'out/hello.txt'.
  Must remake target 'out/hello.txt'.               ## Debe reahacerse
mkdir -p out
Putting child 0x558301d00450 (out/hello.txt) PID 58588 on the chain.
Live child 0x558301d00450 (out/hello.txt) PID 58588
Reaping winning child 0x558301d00450 PID 58588
python3 src/hello.py > out/hello.txt
Live child 0x558301d00450 (out/hello.txt) PID 58589
Reaping winning child 0x558301d00450 PID 58589
Removing child 0x558301d00450 PID 58589 from chain.
  Successfully remade target file 'out/hello.txt'.
 Finished prerequisites of target file 'build'.
Must remake target 'build'.
Successfully remade target file 'build'.
```
Indica que src/hello.py es mas nuevo que out/hello.txt, por lo tanto se debe rehacer out/hello.txt. mkdir -p crea el directorio, pero si el directorio ya existe no devuelve error. De esta forma se garantiza la existencia del directorio out antes de crear archivos dentro.
- Fuerza un entorno con BSD tar en PATH y corre make tools; comprueba el fallo con "Se requiere GNU tar" y razona por qué --sort, --numeric-owner y --mtime son imprescindibles para reproducibilidad determinista.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make tools
Se requiere GNU tar
make: *** [Makefile:50: tools] Error 1
```
--sort, sirve para ordenar usando un criterio, si no se especifica, entonces pueden ser distintos dependiendiendo en el entorno en el que se ejecute.
--numeric-owner, sirve para guardar al usuario y al grupo por los numeros que los identifican(UID y GID), normalmente se acompaña con una flag especificando user y grupo, el usuario privilegiado puede tener distinto nombre dependiendo del entorno, pero los identificadores suelen ser 0.
--mtime, sirve para guardar la fecha de modificación en un mismo formato.
Definiendo estos datos explícitamente, se garantiza que el hash del archivo sea el mismo, esto sirve para garantizar integridad( el archivo no ha sido manipulado ).
- Ejecuta make verify-repro; observa que genera dos artefactos y compara SHA256_1 y SHA256_2. Si difieren, hipótesis: zona horaria, versión de tar, contenido no determinista o variables de entorno no fijadas.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make verify-repro
SHA256_1=ad917bfc2c042f07c8de4ef70fcbff91a49977d589de88932da474592ba1c545
SHA256_2=ad917bfc2c042f07c8de4ef70fcbff91a49977d589de88932da474592ba1c545
OK: reproducible
```
- Corre make clean && make all, cronometrando; repite make all sin cambios y compara tiempos y logs. Explica por qué la segunda es más rápida gracias a timestamps y relaciones de dependencia bien declaradas.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ time (make clean && make all)
rm -rf out dist
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
ruff no instalado; omitiendo lint Python
mkdir -p out
python3 src/hello.py > out/hello.txt
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
mkdir -p dist
tar --sort=name --owner=0 --group=0 --numeric-owner --mtime='UTC 1970-01-01' -czf dist/app.tar.gz -C out hello.txt

real    0m0.170s
user    0m0.095s
sys     0m0.008s
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ time make all
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
ruff no instalado; omitiendo lint Python
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK

real    0m0.138s
user    0m0.078s
sys     0m0.020s
```
El tiempo de user, que es el tiempo en el que el sistema ejecuta tareas como usuario, se reduce. Esto se debe a que make no vuelve a crear archivos que ya existen y que su fuente no ha sido modificada.
- Ejecuta PYTHON=python3.12 make test (si existe). Verifica con python3.12 --version y mensajes que el override funciona gracias a ?= y a PY="${PYTHON:-python3}" en el script; confirma que el artefacto final no cambia respecto al intérprete por defecto.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ python3.12 --version
Python 3.12.3
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ PYTHON=python3.12 make test
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3.12 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```
- Ejecuta make test; describe cómo primero corre scripts/run_tests.sh y luego python -m unittest. Determina el comportamiento si el script de pruebas falla y cómo se propaga el error a la tarea global.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make test
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
## Fallando
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make test
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... FAIL

======================================================================
FAIL: test_greet (test_hello.TestGreet.test_greet)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/ubuntu1/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2/tests/test_hello.py", line 6, in test_greet
    self.assertEqual(greet("Paulette"), "Hello, Paulettee!")
AssertionError: 'Hello, Paulette!' != 'Hello, Paulettee!'
- Hello, Paulette!
+ Hello, Paulettee!
?                +


----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
make: *** [Makefile:30: test] Error 1
```
La prueba falla, python describe el error y sale con código de estado distinto de cero, make recibe el error y detiene el programa, informa sobre la linea en la que ocurre el error.
- Ejecuta touch src/hello.py y luego make all; identifica qué objetivos se rehacen (build, test, package) y relaciona el comportamiento con el timestamp actualizado y la cadena de dependencias especificada.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ touch src/hello.py
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make all
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
ruff no instalado; omitiendo lint Python
mkdir -p out
python3 src/hello.py > out/hello.txt
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
mkdir -p dist
tar --sort=name --owner=0 --group=0 --numeric-owner --mtime='UTC 1970-01-01' -czf dist/app.tar.gz -C out hello.txt
```
Se rehace el archivo out/hello.txt y el empaquetado reproducible.
- Ejecuta make -j4 all y observa ejecución concurrente de objetivos independientes; confirma resultados idénticos a modo secuencial y explica cómo mkdir -p $(@D) y dependencias precisas evitan condiciones de carrera.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make clean
rm -rf out dist
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make -j4 all
shellcheck scripts/run_tests.sh
mkdir -p out
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
python3 src/hello.py > out/hello.txt
mkdir -p dist
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
tar --sort=name --owner=0 --group=0 --numeric-owner --mtime='UTC 1970-01-01' -czf dist/app.tar.gz -C out hello.txt
shfmt -d scripts/run_tests.sh
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
ruff no instalado; omitiendo lint Python
```
Se observa que no se sigue el mismo orden, ya que los objetivos independientes se ejecutan de forma concurrente. Pero esto podría producir condiciones de carrera, si las dependencias no están bien definidas, algunos procesos podrían entrar en conflicto.
- Ejecuta make lint y luego make format; interpreta diagnósticos de shellcheck, revisa diferencias aplicadas por shfmt y, si está disponible, considera la salida de ruff sobre src/ antes de empaquetar.
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make lint
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
ruff no instalado; omitiendo lint Python
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make format
shfmt -w scripts/run_tests.sh
```
No se muestran cambios ni diferencias.
# Parte 3-Extender:
## lint mejorado
Al quitar comillas a un texto y correr make lint ocurre lo siguiente:
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make lint
shellcheck scripts/run_tests.sh

In scripts/run_tests.sh line 60:
if false | true; then
^-- SC1046 (error): Couldn't find 'fi' for this 'if'.
^-- SC1073 (error): Couldn't parse this if expression. Fix to allow more checks.


In scripts/run_tests.sh line 63:
        echo Con pipefail: se detecta el fallo (status != 0).
                                               ^-- SC1036 (error): '(' is invalid here. Did you forget to escape it?
                                               ^-- SC1047 (error): Expected 'fi' matching previously mentioned 'if'.
                                               ^-- SC1072 (error): Expected 'fi'. Fix any mentioned problems and try again.

For more information:
  https://www.shellcheck.net/wiki/SC1046 -- Couldn't find 'fi' for this 'if'.
  https://www.shellcheck.net/wiki/SC1036 -- '(' is invalid here. Did you forg...
  https://www.shellcheck.net/wiki/SC1047 -- Expected 'fi' matching previously...
make: *** [Makefile:39: lint] Error 1
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ make format
shfmt -w scripts/run_tests.sh
scripts/run_tests.sh:63:41: a command can only contain words and redirects; encountered (
make: *** [Makefile:65: format] Error 1
```
## Rollback adicional
```bash
ubuntu1@LAPTOP-V377DISR:~/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2$ ./scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
Error: archivo temporal perdido
```
## Incrementalidad
Ejecuta dos veces make benchmark para comparar un build limpio frente a uno cacheado; revisa out/benchmark.txt. Después, modifica el timestamp del origen con touch src/hello.py y repite. Observa que build, test y package se rehacen. Interpreta tiempos y relación con dependencias y marcas de tiempo.
```bash
Benchmark: 2025-10-03 07:05:54 / Commit: 8601119
make[1]: Entering directory '/home/ubuntu1/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2'
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
ruff no instalado; omitiendo lint Python
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
make[1]: Leaving directory '/home/ubuntu1/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2'
Tiempo: 0:00.16
```
```bash
Benchmark: 2025-10-03 07:08:18 / Commit: 8601119
make[1]: Entering directory '/home/ubuntu1/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2'
shellcheck scripts/run_tests.sh
shfmt -d scripts/run_tests.sh
ruff no instalado; omitiendo lint Python
mkdir -p out
python3 src/hello.py > out/hello.txt
scripts/run_tests.sh
Demostrando pipefail:
Sin pipefail: el pipe se considera exitoso (status 0).
Con pipefail: se detecta el fallo (status != 0).
Test pasó: Hello, World!
python3 -m unittest discover -s tests -v
test_greet (test_hello.TestGreet.test_greet) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
mkdir -p dist
tar --sort=name --owner=0 --group=0 --numeric-owner --mtime='UTC 1970-01-01' -czf dist/app.tar.gz -C out hello.txt
make[1]: Leaving directory '/home/ubuntu1/Desktop/desarrollo-de-software/Curso-CC3S2/labs/Laboratorio2'
Tiempo: 0:00.15
```

