# Compilador *GCC* para *C*

De la documentación para la versión de *GCC* 10.2.

GCC realiza preproceso, compilado, ensamblado y enlazado.

Las opciones de línea de comandos con una sola letra no se pueden unir en una sola: `gcc -v -d` no equivale a `gcc -vd`. Se pueden mezclar argumentos y opciones, sin importar el orden, excepto en circunstancias específicas (por ejemplo, usando `-L` varias veces, el orden de búsqueda se hará en el orden especificado; la posición de `-l` también es importante).

Muchas opciones con nombres largos empezados por `-f` o `-W` tienen su contraparte negativa. Por ejemplo, `-Wfoo` vs. `-Wno-foo`.

Las opciones que toman argumentos suelen ser del tipo `-x arg` o bien `-x=arg`. Se especificará en cada caso.

## Opciones que controlan el tipo de salida

Lo realmente obligatorio es pasarle a *GCC* el archivo o archivos fuente. Si nos ceñimos a C, hay varios tipos de archivo a considerar: ***<archivo>.c*** (código fuente que hay que tratar desde el principio), ***<archivo>.i*** (código fuente que no hay que preprocesar, no es muy usado), y ***<archivo>.h*** (archivo de cabecera).

Por otro lado, los archivos en ensamblador tienen las siguientes extensiones: ***<archivo>.s*** (no debe ser preprocesado), ***<archivo>.S*** o ***<archivo>.sx***.

A parte de los archivos, podemos utilizar multitud de opciones. En este caso, veremos las que controlan el tipo de salida que producirá *GCC*.

En primer lugar, el compilador decide de qué lenguaje se trata en base a las extensiones de los archivos fuente. Pero si queremos le podemos especificar nosotros el lenguaje, sin importar la extensión. Para ello se utiliza la opción `-x`. Esta opción se debe incluir antes de los archivos en cuestión. Si tenemos varios archivos de lenguajes distintos y queremos especificar el lenguaje con esta opción, cada archivo será compilado según la última opción `-x` aparecida en la línea de comandos. Para que los subsiguientes archivos se compilen según su extensión, se debe usar `-x none`.

```
gcc -x c file1.c -x assembler file2.c -x none file3.c
```

Este ejemplo compila ***file1.c*** y ***file3.c*** como archivos *C* y ***file2.c*** como archivo ensamblador.

`-E` se detiene en la fase de preprocesado, es decir, no efectúa la compilación, el ensamblado ni el enlazado. Simplemente realiza el preprocesado de los archivos fuente y envía la salida a la salida estándar. Los archivos que no precisan preprocesado son ignorados.

`-S` se detiene en la fase de compilación, es decir, no efectúa el ensamblado ni el enlazado. Produce, por cada archivo fuente, un archivo en ensamblador, con el mismo nombre del archivo fuente original, pero con extensión *.s*. Los archivos que no precisan compilación o archivos en ensamblador son ignorados.

`-c` se detiene en la fase de ensamblado, es decir, no efectúa el enlazado. Produce, por cada archivo fuente, un archivo de código objeto, con el mismo nombre del archivo fuente original, pero con extensión *.o*. Los archivos que no precisan compilación o ensamblado son ignorados.

Cuando no indicamos ninguna de las opciones `-E`, `-S` o `-c`, el compilador llega hasta el final y crea un solo archivo ejecutable con todos los archivos fuente. En ese caso, podemos indicar el nombre del archivo resultante mediante `-o nombre`. Si no se indica, por defecto, un ejecutable tendrá nombre *a.out*.

`--version` muestra la versión de *GCC*.

## Opciones que controlan el dialecto de *C*

`-std=` determina el estándar que deseamos utilizar. Por ejemplo, entre otros, ***c90*** y ***c89*** se refieren al estándar *ISO C90*. ***c99*** indica estándar *ISO C99*. ***c11*** es *ISO C11*. ***c17*** y ***c18*** indican estándar *ISO C17*, también denominado *ISO C18*.

`-ansi` equivale, cuando estamos compilando un programa en *C*, a `-std=c90`.

## Opciones que solicitan o suprimen avisos (*warnings*)

`-w` inhibe todos los *warnings*.

`-Werror` convierte los *warnings* en errores.

`-Wfatal-errors` se detiene al primer error.

`-Wpedantic` y `-pedantic` producen todos los *warnings* que indica el estándar *C* estricto.

`-Wall` habilita incluso todos los *warnings* sobre construcciones que algunos usuarios consideran cuestionables y que son fáciles de arreglar.

`-Wextra` habilita todavía algunos *warnings* adicionales.

Existen un montón de opciones que habilitan tipos de *warnings* específicos (variable no utilizada, etc.). Son las que empiezan por `-W` (o `-Wno` para suprimir el *warning*).

## Opciones de depuración de errores

Opciones para que *GCC* añada información de depuración (*debug*) en el archivo final.

`-g` es la opción principal para ello. Se puede utilizar incluso con opciones de optimización, aunque se recomienda usar `-Og` en este caso.

Aunque el depurador *GDB* puede funcionar con la opción indicada, si va a ser ese el depurador que utilizaremos, es mejor utilizar `-ggdb`, ya que añade más posibilidades a la depuración con dicho programa.

## Opciones que controlan la optimización

Al compilar sin optimización, se intenta reducir el coste de la compilación y producir un código depurable. Al añadir optimización, se intenta conseguir velocidad de ejecución y/o minimizar el tamaño del ejecutable, a expensas de más tiempo de compilación, y en quizá comprometer las posibilidades de depuración del programa.

`-O0` establece un nivel de optimización 0, es decir lo desactiva. Es lo mismo que no indicar un nivel de optimización (mediante las opciones `-O`). En este caso, por muchas optimizaciones que se indiquen (existen multitud de opciones que activan optimizaciones
individuales), no se llevarán a cabo.

`-O1` equivale a `-O`, y establece un nivel de optimización 1. La compilación toma más tiempo y se utiliza más memoria. Activa una serie de optimizaciones destinadas a reducir el tamaño resultante y el tiempo de ejecución, sin entrar en complejas optimizaciones que precisan de mucho más tiempo de compilación.

`-O2` optimiza todavía más. Este nivel 2 de optimización añade optimizaciones adicionales, que presentan un buen equilibrio entre velocidad y tamaño.

`-O3` produce el nivel 3 de optimización, con todavía más optimizaciones.

`-Os` optimiza para minimizar tamaño. Activa las mismas optimizaciones que `-O2`, excepto aquellas que aumentan el tamaño frecuentemente.

`-Og` activa las optimizaciones que no afectan a la depuración. Ofrecen un nivel razonable de optimización mientras proporciona tiempos de compilación no demasiado largos y una buena experiencia de depuración.

Para depurar, es incluso mejor esta opción que compilar sin optimización, ya que algunos pases del compilador que recogen datos de depuración son eliminados cuando la optimización está desactivada.

Si incluimos varias opciones `-O`, la última tiene preferencia.

## Opciones que controlan el preprocesador

Algunas de estas opciones solo tienen sentido cuando se usan junto con `-E`, ya que pueden dejar el preprocesado inservible para compilar.

`-D nombre` define la macro 'nombre' con valor 1.

`-D nombre=valor` define la macro 'nombre' y le da el valor indicado. Las opciones `-D` y `-U`, así como las posibles definiciones o indefiniciones dentro del código, se procesan en el orden en el que aparecen en la línea de comandos.

`-U nombre` cancela toda definición de la macro con el nombre indicado, ya sea que se definió mediante `-D` o desde el código fuente.

`-undef` provoca que las macros específicas del sistema y las específicos de *GCC* no estén definidas (las del estándar sí quedan definidas).

`-finput-charset=<charset>` se utiliza para definir el juego de caracteres (y su respectivo *encoding*) de los archivos fuente. El juego de caracteres especificado se traduce al juego de caracteres fuente (*source character set*). Se utiliza para la correcta traducción de los literales *string* y constantes carácter desde los archivos fuente. Por defecto es *UTF-8*, con lo que si esa es la codificación del archivo, no hace falta indicarlo. El juego de caracteres indicado puede ser cualquiera de los soportados por la biblioteca ***iconv*** del sistema. En sistemas *Unix* pueden verse dichas codificaciones mediante `iconv --list` desde la línea de comandos.

`-fexec-charset=<charset>` establece el juego de caracteres de ejecución. Se usa para la codificación de *strings* y caracteres `char`. Por defecto es *UTF-8*. Los juegos de caracteres soportados son los mismos que en el caso de `-finput-charset`.

`-fwide-exec-charset=<charset\>` establece el juego de caracteres *wide* de ejecución. Se usa para la codificación de *strings* y caracteres *wide*. Por defecto es *UTF-32* o *UTF-16* (dependiendo del tamaño de `wchar_t`). Los juegos de caracteres soportados son los mismos que en el caso de `-finput-charset`. Debe tenerse cuidado de no especificar una codificación que no encaje en el tipo `wchar_t`.

## Opciones de enlazado (*linking*)

Estas opciones no se deben usar si indicamos al compilador que se detenga antes del enlazado.

`-l<biblioteca>` hace que el código de dicha biblioteca se incluya en el ejecutable final. Dicha biblioteca se busca en los directorios estándar, más los especificados con `-L`. Si por ejemplo la opción es `-lfuns`, entonces se buscará ***libfuns.a*** (*static library*), o, si el ejecutable lo admite, ***libfuns.so*** (*shared library*), que tendrá preferencia (a no ser que indiquemos la opción `-static`). Si finalmente es la versión *shared*, el *linker* no incrusta el código en el ejecutable, con lo que el tamaño resultará menor.

## Enlazado de código

El orden es importante aquí. Las bibliotecas se cargan en el archivo final solo si hay referencia a sus funciones cuando se llega a la opción que las añade. Por ejemplo, si compilamos así:

```
gcc foo.o -lzz bar.o
```

Entonces ***foo.o*** se cargará primero, luego la biblioteca ***libzz.a***, y después ***bar.o***. Si ***bar.o*** se refiere a funciones en la biblioteca, esas funciones podrían no cargarse. Si ***foo.o*** hace referencia a esas funciones, al llegar a `-lzz` seguro que las carga.

Cuando creamos un ejecutable a partir de archivos de entrada que utilizan bibliotecas, es importante, pues, el orden en que lo especificamos. Veamos los posibles casos:

Si la biblioteca es *estática*, se enlazarán (añadirán) al ejecutable el código fuente íntegro de todos los archivos de código objeto (***.o***) pertenecientes a esa biblioteca, tales que se hay hecho alguna referencia a alguna de sus funciones (o variables) desde el código fuente (o desde otros archivos de código objeto). El resto de archivos objeto de la biblioteca no se enlazarán. Por lo tanto, las bibliotecas no se enlazan íntegramente, sino selectivamente, por partes (archivos ***.o***) según las necesidades del código fuente. Si una biblioteca estática estuviese formada únicamente por un solo archivo objeto, cualquier referencia a alguna de sus funciones provocaría el enlazado de la biblioteca completa.

En cambio, si la biblioteca es *dinámica*, no se enlaza ningún código de esta. Sin embargo, para cada función utilizada de la biblioteca se añade información de referencia (modo de localizar la función) en el ejecutable. Esta información no es por cada archivo objeto de la biblioteca, sino por cada función (o variable) de la misma a la que se haga referencia desde el código fuente (independientemente del número de llamadas o referencias que sufra).

Por otro lado, cuando junto al código fuente enlazamos una serie de archivos de código objeto, se enlaza absolutamente todo el código de estos, tanto si es llamado como si no. Incluso podríamos estar compilando una serie de archivos objeto, sin ningún archivo ***.c*** (el punto de entrada sería la función `main()` del archivo que la contuviera). Los archivos ***.c*** también se enlazan enteros.

En el caso de archivos objeto, al igual que con los archivos fuente (***.c***), el orden de aparición es indiferente.

En cuanto a las referencias al código de una biblioteca, la mera *declaración* de una función de esta no constituye una referencia suficiente para su enlazado: debe producirse por lo menos una *llamada* a la función.

### Crear una biblioteca compartida (*.so*) en entorno *Unix*

`-shared` se usa para generar código compartido, es decir, una *shared library*. Para ello tendremos que tener código objeto que sea *position-independent code* (*PIC*):

```
gcc -fPIC -c file1.c file2.c
```

No todos los sistemas precisan `-fPIC`. En este caso, en *Linux* es necesario.

Ahora, creados ya los archivos objeto, con código independiente de la posición, ya podemos crear la biblioteca:

```
gcc -shared -o libcosas.so file1.o file2.o
```

Se podrían hacer estas dos acciones en un solo paso:

```
gcc -shared -fPIC -o libcosas.so file1.c file2.c
```

### Crear una biblioteca estática (*.a*) en entorno *Unix*

En este caso se compila primero el código objeto, sin ninguna restricción:

```
gcc -c file1.c file2.c
```

Y simplemente se empaquetan los diferentes archivos de código objeto en una biblioteca utilizando el archivador `ar` con los parámetros `r` (añadir con reemplazo), `c` (crear archivo nuevo) y `s` (crear o regenerar índice en el archivo):

```
ar rcs libcosas.a file1.o file2.o
```

### Crear bibiliotecas en *Windows* (con *Cygwin*)

En entorno *Cygwin*, los ejecutables generados necesitan ***cygwin1.dll*** para funcionar. Se puede simplemente incluir esta biblioteca en el mismo directorio que el ejecutable principal, ya que *Windows* busca las *dlls*, a parte de en los directorios del sistema, en el mismo directorio del programa.

En cuanto a la compilación de bibliotecas compartidas no hace falta especificar `-fPIC`, ya que las *dlls* son todas *relocatable* (*position-independent code*). Por otro lado, la convención de nombres de estas bibliotecas dinámicas no es del tipo *libfuns.so*, sino *funs.dll* (las bibliotecas estáticas sí mantienen la convención de *Unix*):

```
gcc -shared -o cosas.dll file1.c file2.c
```

Si en lugar de los archivos ***.c*** le damos los *.o* correspondientes, el resultado es idéntico.

Para crear bibliotecas estáticas, el proceso es igual que en *Linux* (siempre desde entorno *Cygwin*, claro).

En cuanto al momento de compilar un **ejecutable** que utiliza bibliotecas, si dispone de las dos opciones (estática y dinámica), se decanta por la estática (al contrario que en *Linux*).

#### Crear bibliotecas nativas de *Windows* (no *Cygwin*)

En realidad *Cygwin* es un entorno *POSIX* dentro de *Windows*. Por eso, desde dentro de *Cygwin* se pueden ejecutar los ejecutables sin necesidad de *cygwin1.dll* (el archivo es necesario al ejecutar desde fuera de *Cygwin*).

Por lo tanto, `gcc` crea ejecutables para entorno *Cygwin*. Si queremos crear un ejecutable nativo para *Windows*, es decir, si queremos eliminar la dependencia de ***cygwin1.dll***, tenemos que *cross-compile*. Desde *Cygwin* podemos ejecutar el *cross-compiler* `x86_64-w64-mingw32-gcc`, incluido en el paquete ***mingw64-x86_64-gcc-core*** de *Cygwin* para *cross-compile* un ejecutable nativo de *Windows* de 64 *bits*. Aunque si no nos importa lo de acompañar siempre *cygwin1.dll* a nuestros ejecutables (o copiar el *dll* en una carpeta de bibliotecas del sistema), no haría falta hacer nada de eso.

Si queremos *cross-compile* una biblioteca estática nativa, deberíamos usar `x86_64-w64-mingw32-ar` para empaquetarl los archivos de código objeto.

De todas formas, hay que tener en cuenta que si necesitamos implementar funciones *POSIX* en nuestro programa, tenemos que compilar obligatoriamente para *Cygwin*, de la forma habitual.

#### *Cross-compile* para *Windows* desde *Linux*

También podemos crear código ejecutable (y bibliotecas) nativo para *Windows* desde *Linux*.

Simplemente hay que compilar con los *cross-compilers* `gcc` de paquetes como `mingw-w64` o `mingw32` (disponibles, por ejemplo en los repositorios de *Ubuntu*). Se sigue aplicando todo lo dicho hasta ahora.

## Opciones para directorios de búsqueda

`-I <dir>`, `-iquote <dir>`, `-isystem <dir>` o `-idirafter <dir>` añaden el directorio especificado a la lista de directorios donde encontrar *header files*. En el caso de `-iquote`, se utiliza en las búsquedas del tipo `#include "file.h"`, mientras que las otras opciones valen tanto para `#include "file.h"` como para `#include <file.h>`.

El orden de búsqueda es el siguiente: para *includes* de tipo comillas, se busca primero en el directorio del archivo actual. En segundo lugar, se buscan todas las apariciones de `-iquote`, de izquierda a derecha.

Luego, y ya para cualquier tipo de *include*, se buscan todas las apariciones de `-I` de izquierda a derecha. Luego `-isystem`, también de izquierda a derecha. Seguidamente los directorios del sistema. Y finalmente, `-idirafter`, de izquierda a derecha.

`-L<dir>` se usa para añadir directorios a la lista de directorios donde buscar bibliotecas a enlazar (indicadas con `-l`).
