# 7.22 General utilities \<stdlib.h>

Define 5 tipos:

- `size_t` (definido también en ***stddef.h***) es el tipo entero *unsigned* que retorna el operador `sizeof`.
- `wchar_t` (definido también en ***stddef.h***) es un tipo entero capaz de representar todos los miembros del *extended character set* más extenso de los *locales* soportados.
- `div_t`, `ldiv_t`, `lldiv_t` son los tipos de los valores retornados por las funciones `div()`, `ldiv()` y `lldiv()` respectivamente.

Macros definidas:

- ***NULL*** (definido también en ***stddef.h***) se expande a una constante apuntador nulo *implementation-defined*.
- ***EXIT_FAILURE***, ***EXIT_SUCCESS*** ==> expresiones enteras constantes para pasar como argumento a la función `exit()` para retornar estado *unsuccessful* o *successful* respectivamente.
- ***RAND_MAX*** ==> expresión entera constante que define el valor máximo que puede retornar la funcion `rand()`.
- ***MB_CUR_MAX*** ==> expresión positiva de tipo `size_t` que define el número máximo de *bytes* en un carácter *multibyte* del *extended character set* del *locale* actual (categoria ***LC_CTYPE***, de ***locale.h***), y nunca es mayor que ***MB_LEN_MAX*** (***limits.h***).

## 7.22.1 Numeric conversion functions

Las funciones:

`double atof(const char *nptr)`\
`int atoi(const char *nptr)`\
`long int atol(const char *nptr)`\
`long long int atoll(const char *nptr)`

Convierten la parte inicial del *string* apuntado por ***nptr*** en un `double`, `int`, `long int` y `long long int` respectivamente, y devuelven ese valor. No tienen por qué afectar al valor de la expresión entera ***errno*** (***errno.h***). A excepción de ese comportamiento en caso de error, equivalen, respectivamente, a:

`strtod(nptr,(char**)NULL)`\
`(int)strtol(nptr,(char**)NULL,10)`\
`strtol(nptr,(char**)NULL,10)`\
`strtoll(nptr,(char**)NULL,10)`

Las funciones:

`double strtod(const char * restrict nptr, char ** restrict endptr)`\
`float strtof(const char * restrict nptr, char ** restrict endptr)`\
`long double strtold(const char * restrict nptr, char ** restrict endptr)`

Convierten la parte inicial del *string* apuntado por ***nptr*** en un `double`, `float` y `long double` respectivamente. Descomponen el *string* en una primera parte opcional de espacios (como está definido en in `isspace()`, de ***ctype.h***), una parte (*subject sequence*) representando un *floating-point* o ***infinity*** o ***NaN***, y una parte (*final string*) de caracteres no pertenecientes a la *subject sequence* (entre los que está el *null character*). En cuanto a la *subject sequence*, está formada por un signo opcional, seguido de uno de estos elementos:

- Una secuencia no vacía de dígitos decimales incluyendo o no punto decimal, más una parte de exponente opcional (con el formato de una constante *floating point*), o
- ***0x*** o ***0X*** seguido por una secuencia no vacía de dígitos hexadecimales incluyendo o no punto decimal, más una parte de exponente opcional (con el formato de una constante *floating point*), o
- ***INF*** o ***INFINITY***, *ignoring case*, o
- ***NAN*** o ***NAN(secuencia)***, *ignoring case*, donde esa secuencia de caracteres es de significado *implementation-defined*.

La *subject sequence* puede tener otros formatos en *locales* distintos al *locale* ***"C"***. Si ***endptr*** no es un apuntador nulo, estas funciones almacenan allí la dirección del *final string* que sigue a la *subject sequence*, o si la *subject sequence* no tiene el formato correcto, almacenan allí la direccion de ***nptr***. Las funciones retornan el valor convertido, o ***0*** si no se pudo; si el resultado *overflows* el tipo del valor de retorno, retornan (±) ***HUGE_VAL***, ***HUGE_VALF*** o ***HUGE_VALL***, y ***errno*** es ***ERANGE***. Si el resultado *underflows*, retornan un número cuya magnitud no excede al mínimo número *floating* normalizado representable para el tipo del valor de retorno (si ***errno*** pasa a ser ***ERANGE*** o no, es *implementation-defined*).

Las funciones:

`long int strtol(const char * restrict nptr, char ** restrict endptr, int base)`\
`long long int strtoll(const char * restrict nptr, char ** restrict endptr, int base)`\
`unsigned long int strtoul(const char * restrict nptr, char ** restrict endptr, int base)`\
`unsigned long long int strtoull(const char * restrict nptr, char ** restrict endptr, int base)`

Convierten la parte inicial del *string* apuntado por ***nptr*** en un `long int`, `long long int`, `unsigned long int` y `unsigned long long int` respectivamente. Funciones análogas a la familia `strtod()`, pero en este caso, la *subject sequence* es un entero en base ***base***. Dicha secuencia empieza con un signo opcional en todos los casos. Si ***base*** es ***0***, la secuencia es con un número con el formato de una constante entera, sin incluir sufijo. Con ***base 2*** a ***36***, la secuencia será una ristra de dígitos representando el entero con la base especificada (se usarán los dígitos ***aA***-***zZ*** necesarios); si la base es ***16*** podemos preceder de ***0x*** o ***0X***, aunque no es necesario. Si el valor excede el tipo de retorno, retornan ***LONG_MIN***, ***LONG_MAX***, ***LLONG_MIN***, ***LLONG_MAX***, ***ULONG_MAX*** o ***ULLONG_MAX*** según sea el tipo de retorno y el valor resultante.

## 7.22.2 Pseudo-random sequence generation functions

`int rand(void)`

Genera sucesivos enteros pseudo aleatorios entre ***0*** y ***RAND_MAX*** (al menos *32767*). La implementación se comportará como si ninguna *library function* llamara a `rand()`.

`void srand(unsigned int seed)`

Usa el argumento como semilla de una nueva secuencia para `rand()`. Si se llama con la misma semilla que ya está usando `rand()`, se repite nuevamente la secuencia. Al iniciar el programa es como si se hubiera llamado a `srand()` con valor 1. La implementación se comportará como si ninguna *library function* llamara a `srand()`. No retorna nada.

## 7.22.3 Memory management functions

El orden y continuidad del *allocated storage* a través de las funciones de reserva de memoria son *unspecified*. El apuntador retornado si la reserva sale bien está debidamente alineado, de tal modo que puede ser asignado a un apuntador a cualquier tipo de objeto con un requisito de alineación fundamental, y luego utilizado para acceder a ese objeto o *array* de objetos. Si la reserva falla, las funciones retornan ***NULL***.

Para determinar existencia de *data races*, las funciones de reserva de memoria se comportan como si accediesen solamente a direcciones de memoria accesibles a través de sus argumentos, y no a otro almacenamiento estático.

`void *aligned_alloc(size_t alignment, size_t size)`

Reserva espacio para un objeto con alineación ***alignment*** (soportado por la implementación), y cuyo tamaño es ***size*** (múltiplo entero de ***alignment***). Retorna ***NULL*** (fallo) o un apuntador a la zona reservada.

`void *calloc(size_t nmemb, size_t size)`

Reserva espacio para *array* de ***nmemb*** objetos de tamaño ***size***. Inicializa todos los bits a ***0*** (lo cual no significa forzosamente un ***0*** en punto flotante ni un apuntador nulo). Devuelve apuntador a zona reservada o ***NULL*** si falla.

`void free(void *ptr)`

Libera el espacio apuntado por ***ptr***, el cual debe haber sido reservado anteriormente mediante una de las funciones de reserva de memoria. Si ***ptr*** es nulo, no hace nada. No retorna nada.

`void *malloc(size_t size)`

Es como `calloc()`, pero con un solo objeto.

`void *realloc(void *ptr, size_t size)`

Libera lo apuntado por ***ptr*** y lo reserva de nuevo, ocupando esta vez ***size*** bytes. El contenido anterior de memoria se copia hasta el tamaño menor de los dos (el antiguo liberado y el nuevo). Si la nueva región es más grande que la antigua, el tramo de memoria extra tiene valores no determinados. Si ***ptr*** es un apuntador nulo, la función se comporta igual que `malloc()`. Si no se puede reservar la nueva zona, no se libera la antigua. Devuelve el valor del apuntador a la nueva zona reservada o ***NULL*** si no se pudo reservar.

## 7.22.4 Communication with the environment

`_Noreturn void abort(void)`

Provoca una *abnormal program termination*. `abort()` no retorna, a no ser que capturemos la señal ***SIGABRT*** y el *handler* no retorne. Al entorno se le devuelve un valor que indica terminación sin éxito. Si se cierran los recursos abiertos (*streams*,...) o no, es *implementation-defined*.

`int atexit(void (*func)(void))`

Registra la función apuntada por ***func*** (sin argumentos ni retorno de valor) como función que será llamada a la terminación normal del programa. No es seguro que una llamada a `atexit()` que no suceda antes de que se llame a `exit()` tenga éxito. Se deben poder registrar por lo menos 32 funciones. Retorna *nonzero* si falla, 0 si tiene éxito.

`int at_quick_exit(void (*func)(void))`

Registra la función apuntada por ***func*** (sin argumentos ni retorno de valor) como función que será llamada si es invocada `quick_exit()`. No es seguro que una llamada a `at_quick_exit()` que no suceda antes de que se llame a `quick_exit()` tenga éxito. Se deben poder registrar por lo menos 32 funciones. Retorna *nonzero* si falla, 0 si tiene éxito.

`_Noreturn void exit(int status)`

Terminación normal del programa. Primero se llama a las funciones registradas con `atexit()` (las funciones registradas con `at_quick_exit()` no son llamadas) en orden inverso de registro; después se hace *flush* y cierran los *streams* y los archivos temporales se borran; luego se devuelve el control al entorno del *host*. Con ***status 0*** o ***EXIT_SUCCESS*** se retornará el estado *successful termination*; con ***EXIT_FAILURE***, *unsuccessful termination*. La función no retorna.

`_Noreturn void _Exit(int status)`

Como `exit()`, pero no se llama a las funciones registradas ni a los *handlers* de señales definidos con la función `signal()` (***signal.h***), y si se cierran los *streams* o no, es *implementation defined*.

`char *getenv(const char *name)`

Retorna el valor de la variable del sistema *host* especificada por ***name***. No debe escribirse en la zona apuntada por el apuntador devuelto. La implementación se comporta como si ninguna *function library* llamara a esta función. Si la variable no se encuentra, retorna ***NULL***.

`_Noreturn void quick_exit(int status)`

Terminación normal del programa. Primero se llama a las funciones registradas con `at_quick_exit()` (las funciones registradas con `atexit()` o *signal handlers* registrados con `signal()` no son llamadas) en orden inverso de registro. Luego se pasa el control al *host environment* mediante `_Exit(status)`.

`int system(const char *string)`

En caso de que ***string*** sea ***NULL***, comprueba si el *host environment* posee procesador de comandos; en caso de que sea así, retorna *nonzero* (si no tiene, retorna ***0***). En caso de que ***string*** no sea ***NULL***, envía al *command processor* del entorno el comando en ***string***. Si dicho comando retorna, la función retornará a su vez un valor *implementation-defined*.

## 7.22.5 Searching and sorting utilities

Las funciones `qsort()` y `bsearch()` pueden tener un argumento ***nmemb*** igual a cero. En ese caso no se llamará a la función apuntada por ***compar*** ni se buscará ni reordenará nada; sin embargo los apuntadores pasados sí deben tener valores válidos.

La implementación se asegurará de que el segundo argumento de ***compar*** (en el caso de `bsearch()`, ya que el primero será ***key***) o ambos (en el caso de `qsort()`) son apuntadores a elementos del *array* (teniendo en cuenta ***base***, ***nmemb*** y ***size***).

La función ***compar*** no altera el contenido del *array*. La implementación puede reordenar el *array* entre llamadas a ***compar***, pero no puede alterar el contenido de un elemento.

Hay un *sequence point* justo antes y justo después de cada llamada a ***compar***.

`void *bsearch(const void *key, const void *base, size_t nmemb, size_t size, int (*compar)(const void *, const void *))`

Busca en un *array* de ***nmemb*** elementos (apuntado por ***base***), un elemento igual al apuntado por ***key***. El tamaño de cada elemento es ***size***. También se especifica un apuntador ***compar*** a la función que compara el objeto apuntado por ***key*** (como primer argumento) con un objeto del *array* (como segundo argumento), la cual devuelve un número negativo, o ***0*** positivo si ***key*** es, repectivamente, menor, igual o mayor que el elemento del *array*. Para que funcione la búsqueda, el *array* consistirá en todos los valores inferiores a ***key***, todos los iguales a ***key*** y todos los superiores a ***key***, **en este orden**. Retorna un apuntador al *matching element* del *array*, o ***NULL*** si no hay coincidencia. Si hay varios elementos coincidentes, es *unspecified* cuál de ellos se retornará.

`void qsort(void *base, size_t n, size_t size, int (*compar) (const void *, const void *))`

Funciona de forma similar a `bsearch()`, pero esta ordena el *array*. En lugar de pasarle un valor para buscar, se le pasa solo la dirección del *array*, que dejará ordenado. Lo ordena ascendentemente, según ***compar***, que devuelve un número negativo, o ***0*** positivo según si el primer argumento se evalúa menor, igual o superior al segundo, respectivamente. `qsort()` no retorna nada.

## 7.22.6 Integer arithmetic functions

Las funciones:

`int abs(int j)`\
`long int labs(long int j)`\
`long long int llabs(long long int j)`

Retornan el valor absoluto del `int`, `long int` o `long long int` del argumento respectivamente.

Las funciones:

`div_t div(int numer, int denom)`\
`ldiv_t ldiv(long int numer, long int denom)`\
`lldiv_t lldiv(long long int numer, long long int denom)`

Calculan el cociente y resto de ***numer***/***denom***, y los almacenan en los campos ***quot*** (cociente) y ***rem*** (resto) de la estructura retornada, los cuales serán de tipo `int`, `long int` o `long long int` respectivamente.

## 7.22.7 Multibyte/wide character conversion functions

El *locale* actual (categoría ***LC_CTYPE***) afecta a las funciones de caracteres *multibyte*. En cuanto a las codificaciones *state-dependent*, cada función de este apartado queda en su estado inicial al *startup* del programa y puede volver a ese estado pasandole ***NULL*** como argumento ***s***. Llamadas subsiguientes (con ***s*** no ***NULL***) a dichas funciones pueden dejar el estado interno de la codificación cambiado. Usando ***NULL*** como argumento ***s***, las funciones de este apartado retornan *nonzero* si la codificación es *state-dependent*, y ***0*** si no lo es.

`int mblen(const char *s, size_t n)`

Retorna el número de *bytes* que tiene el carácter *multibyte* apuntado por ***s***. Equivale exactamente a `mbtowc((wchar_t *)0, s, n)`, aunque cada función guarda su propio *shift state* interno. La implementación se comportará como si ninguna *library function* llamara a esta función. Si ***s*** apunta a un carácter nulo, retorna ***0***; si la secuencia (hasta ***n*** *bytes* máximo) no puede definir un carácter *multibyte* válido, retorna ***-1***.

`int mbtowc(wchar_t * restrict pwc, const char * restrict s, size_t n)`

Inspecciona como máximo ***n*** *bytes* empezando por el apuntado por ***s***, para ver cuántos *bytes* son necesarios (incluyendo *shift sequences*) para leer el carácter *multibyte*, y retorna ese valor. Si este es completo y válido, lo convierte a *wide* y lo coloca en el `wchar_t` apuntado por ***pwc*** (si ***pwc*** no es un apuntador nulo). La implemantación se comportará como si ninguna *library function* llamara a esta función. Retorna lo mismo que `mblen()`. No devuelve nunca un valor superior a ***n*** o a ***MB_CUR_MAX***. Si se lee el carácter nulo, vuelve al *initial shift state*.

`int wctomb(char *s, wchar_t wc)`

Convierte el carácter *wide* ***wc*** a una secuencia *multibyte* incluyendo *shift sequences* y la almacena en ***s***. Nunca serán más de ***MB_CUR_MAX*** *bytes* en total. Si el carácter es un nulo, escribirá el *byte*, precedido, si es necesario, por una secuencia que ponga el estado en inicial, y la función quedará en estado inicial. Retorna ***-1*** (si ningún carácter *multibyte* válido corresponde a ***wc***) o el número de *bytes* necesarios para codificar ***wc***. La implemantación se comportará como si ninguna *library function* llamara a esta función.

## 7.22.8 Multibyte/wide string conversion functions

`size_t mbstowcs(wchar_t * restrict pwcs, const char * restrict s, size_t n)`

Convierte una secuencia de caracteres *multibyte* que empiezan en el *initial state* (***s***), a un *string* de *wides* (***pwcs***) que no contendrá más de ***n*** elementos `wchar_t`. Si se encuentra un *multibyte null*, se escribe un *wide null* y se termina. Retorna el número de caracteres convertidos, sin incluir el *null terminator*, si lo hay. Si encuentra un carácter *multibyte* inválido, retorna `(size_t)(-1)`.

`size_t wcstombs(char * restrict s, const wchar_t * restrict pwcs, size_t n)`

Convierte un *string* de *wides* (***pwcs***) a una secuencia de *multibytes* que empiezan en su *initial state*. y lo almacena en ***s***; no se escribirán más de ***n*** *bytes* en destino incluyendo *shift sequences*. Si se encuentra un *wide null*, se escribe un *multibyte null* y se termina. Retorna el número de *bytes* escritos, sin incluir el *null terminator*, si lo hay. Si encuentra un carácter *wide* que no corresponde a un carácter *multibyte* válido, retorna `(size_t)(-1)`.
