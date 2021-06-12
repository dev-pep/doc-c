# 7.21 Input/output \<stdio.h>

## 7.21.1 Introduction

Este *header* define 3 tipos:

- ***size_t*** (definido también en *stddef.h*) es el tipo *unsigned integer* que retorna el operador ***sizeof***.
- ***FILE*** para controlar un *stream*: contiene indicador de posicion, apuntador a *buffer* (si lo hay), indicador de error e indicador de *end-of-file*.
- ***fpos_t*** es un tipo completo no *array* capaz de almacenar la información necesaria para especificar una posición única dentro de un fichero.

Macros definidas:

- ***NULL*** (definido también en *stddef.h*), se expande a una constante apuntador nulo *implementation-defined*.
- ***\_IOFBF***, ***\_IOLBF***, ***\_IONBF*** son expresiones constantes enteras para usar como tercer argumento de la función `setvbuf()`.
- ***BUFSIZ*** es una expresión constante entera que indica tamaño de *buffer* usado por la función `setbuf()`.
- ***EOF*** es una expresión constante entera negativa devuelta por varias funciones para indicar *end-of-file*.
- ***FOPEN_MAX*** es una expresión constante entera con número mínimo de ficheros que la implementación garantiza que se pueden abrir simultáneamente.
- ***FILENAME_MAX***: expresión constante entera con el tamaño que necesita un *array* de `char` para almacenar el nombre de fichero más largo posible que la implementación garantiza que pueda abrirse.
- ***L_tmpnam***: como ***FILENAME_MAX*** pero para nombres de ficheros temporales generados por la función `tmpnam()`.
- ***SEEK_CUR***, ***SEEK_END*** y ***SEEK_SET*** son constantes enteras para el tercer argumento de función `fseek()`.
- ***TMP_MAX***: constante entera con el numero mínimo de nombres únicos de ficheros temporales que pueden generarse con la función `tmpnam()`.
- ***stderr***, ***stdin*** y ***stdout*** son apuntadores a tipo ***FILE*** para los *streams* de texto *standard error*, *standard input* y *standard output*.

El *header* ***wchar.h*** declara funciones con operaciones análogas a la mayoría de las descritas aquí, teniendo como diferencia que las unidades fundamentales internas al programa son caracteres *wide* y no `char`. Su representación externa (en fichero) son secuencias de caracteres *multibyte* (descrito en 7.21.3).

## 7.21.2 Streams

La entrada/salida (terminales, cintas, discos,...) se realiza sobre *streams*. Hay 2 tipos: de texto y binarios.

Los *streams* de texto se componen de líneas, terminadas en *newline* (implementación puede decidir que en la última línea no hace falta *newline*). Un *stream* binario permite almacenar datos internos, y puede contener un número *implementation-defined* de caracteres *trailing* nulos.

Orientación del *stream*: si escribimos un carácter *wide* en un *stream* sin orientación, se convierte en un *stream wide-oriented*. Si en lugar de ello escribimos un *byte*, pasa a ser orientado a *byte*. `freopen()` elimina la orientacion de un *stream*. `fwide()` puede cambiar su orientación. No se deben escribir *bytes* en un *wide-oriented stream* ni *wides* en un *byte-oriented stream*. Los *wide-oriented streams* tienen un objeto ***mbstate_t*** (***wchar.h***) que almacena el *current parse state*; en ellos, `fgetpos()`/`fsetpos()` guarda/restaura este objeto ***mbstate_t*** en el objeto ***fpos_t*** que especifica la posición.

Los *streams* tienen un *lock* de tal modo que solo un *thread* tiene el *lock* en un momento dado. Las funciones que acceden a los *streams* los bloquean antes de acceder a ellos.

La implementacion debe soportar archivos de texto con líneas de al menos 254 caracteres, incluyendo el *newline*. ***BUFSIZ*** debe ser al menos 256.

## 7.21.3 Files

Un *stream* se asocia a un fichero externo. Si un *stream* no tiene un *buffer* asociado (es *unbuffered*), cada carácter escrito/leído lo hace cuanto antes. De lo contrario, se pueden escribir/leer físicamente bloque a bloque. Con un archivo *fully buffered*, los bloques se escriben/leen al llenarse el *buffer*; si es *line buffered*, el bloque se escribe/lee al encontrar un *newline*.

Cuando se cierra el fichero, se disocia del *stream* y los *streams* de salida son *flushed*. Si se llega al final de `main()`, o se llama a la función `exit()`, todos los archivos son cerrados, por lo que los *streams* son *flushed*. Otros caminos de salida, incluida llamada a función `abort()`, pueden dejar archivos abiertos.

Una copia de un objeto ***FILE*** a otra variable no tiene por que servir como sustituto correcto, ya que la dirección de almacenamiento del objeto puede ser significativa.

El *standard error stream* no es *fully buffered*. Los *standard input* y
*output* sí son *fully buffered*, siempre y cuando no estén asociados a *streams interactivos*.

Si un mismo fichero puede abrirse varias veces simultáneamente o no, es *implementation-defined*.

Los *text* y *binary wide-oriented streams* son secuencias de *wide chars*, y los ficheros asociados son *multibyte files*. Las codificaciones *multibyte* de estos ficheros son *implementation-defined*. Colocar la posición de archivo al final (`fseek(<file>, 0, SEEK_END)`) tiene *undefined behaviour* en un *wide binary stream* (por los posibles *trailing null chars*) o cualquier *wide stream* con *state-dependent encoding* que no termine forzosamente en el *initial shift state*.

***FOPEN_MAX*** deberia ser por lo menos 8 (incluyendo los 3 *streams* de texto estándar).

## 7.21.4 Operations on files

`int remove(const char *filename)`

Elimina el archivo. Retorna cero si tiene éxito, y otro número si falla. Si el fichero está abierto, el comportamiento es *implementation-defined*.

`int rename(const char *old, const char *new)`

Renombra el fichero de ***old*** a ***new***. Si el fichero ***new*** ya existe, el comportamiento es *implementation-defined*. Retorna cero si tiene éxito, y *nonzero* si falla.

`FILE *tmpfile(void)`

Crea un archivo temporal en modo ***wb+***, diferente de todos los abiertos; se elimina automaticamente al cerrarse o al terminar el programa. Retorna un apuntador al *stream*, o ***NULL*** si falla.

`char *tmpnam(char *s)`

Genera un nombre de archivo distinto al de todos los archivos abiertos y lo almacena en la dirección apuntada por ***s***. La función es capaz de generar como mínimo ***TMP_MAX*** *strings* distintos. Si se crea un archivo con ese nombre, el archivo deberá eliminarse (`remove()`) cuando acabemos de usarlo. La función genera un *string* distinto cada vez (hasta ***TMP_MAX*** veces garantizado).

Devuelve ***NULL*** si no puede generar un nombre; de lo contrario, devuelve el mismo apuntador del argumento (donde almacenará el nombre), a no ser que le pasemos ***NULL***, en cuyo caso almacenará el resultado en un *string* estático interno y retornará un apuntador al mismo. El *string* que le pasamos debe tener al menos ***L_tmpnam*** caracteres de capacidad. Llamadas simultáneas con argumento ***NULL*** pueden producir *data races* en ese *buffer* estático.

## 7.21.5 File access functions

`int fclose(FILE *stream)`

Cierra el archivo, lo disocia del *stream* y *flushes* el *stream* (escribe todos los datos de salida pendientes, y desecha datos de entrada pendiente). Retorna 0 si tiene éxito y ***EOF*** si falla.

`int fflush(FILE *stream)`

Hace *flush* del *stream* de salida, o actualiza (entrada + salida) *streams* donde la última operación no fue de entrada. Si ***stream*** es un apuntador nulo, realiza la operación en todos los *streams* abiertos de las características mencionadas. Retorna 0 si tiene éxito o ***EOF*** si falla (y establece el indicador de error para el *stream*).

`FILE *fopen(const char * restrict filename, const char * restrict mode)`

Abre el archivo ***filename*** y le asocia un *stream*. Modos: ***r*** (archivo de texto de solo lectura), ***w*** (trunca a longitud ***0*** o crea un archivo de texto; solo escritura), ***wx*** (crea un archivo de texto para escritura), ***a*** (*append*; abre o crea un archivo de texto para escribir en el final del archivo). Es igual para modos binarios: ***rb***, ***wb***, ***wbx***, ***ab***. También igual para modos *update* (lectura + escritura): ***r+*** (lectura/escritura de archivo de texto), ***w+***
(trunca a ***0*** o crea archivo de texto para lectura/escritura), ***a+*** (*append*; abre o crea archivo de texto para *update*, aunque las escrituras son al final del archivo). Modos *update* binarios: ***r+b***, ***w+b***, ***w+bx***, ***a+b***; o ***rb+***, ***wb+***, ***wb+x***, ***ab+***.

En solo lectura, falla si no existe el fichero. En modo exclusivo (***x***) falla si ya existe o no se puede crear. Si se crea, lo hace en modo exclusivo (no compartido). En algunas implementaciones, abrir en modo *append* binario puede dejar la posición de archivo más allá del último carácter escrito, debido al *trailing null padding*.

Cuando se abre en modo update (***+***), se puede hace tanto entrada como salida en el *stream*, pero en ese caso no debemos realizar una entrada justo después de una salida sin que haya entre medio un *flush* o un reposicionamiento en el archivo (con `fseek()`, `fsetpos()`, p `rewind()`); del mismo modo, no hay que hacer una salida después de una entrada sin hacer entre medio un reposicionamiento, exceptio si esa entrada ha encontrado el fin de archivo.

Al abrirse, un *stream* es *fully buffered* si y solo si se puede determinar que no es un dispositivo interactivo; los indicadores de error y *end of file* son limpiados.

`fopen()` retorna una apuntador al objeto controlador del *stream*, o un apuntador nulo si falla.

`FILE *freopen(const char * restrict filename, const char * restrict mode, FILE * restrict stream)`

Abre el archivo ***filename*** y lo asocia al *stream* ***stream***, usando el modo ***mode***. Si ***filename*** es un apuntador nulo, simplemente cambiará el modo del *stream* (los cambios de modo permitidos son *implementation-defined*). `freopen()` intenta antes de nada cerrar cualquier fichero que estuviera asociado a ***stream***, pero ignorará posibles errores en este punto. Los indicadores de error y *end of file* son limpiados. Devuelve ***stream*** si tiene éxito, o un apuntador nulo si falla.

`void setbuf(FILE * restrict stream, char * restrict buf)`

Exceptuando que no retorna nada, equivale a `setvbuf()` con ***mode*** igual a ***\_IOFBF*** y ***size*** igual a ***BUFSIZ***, o si ***buf*** es un apuntador nulo, con ***mode*** igual a ***\_IONBF***.

`int setvbuf(FILE * restrict stream, char * restrict buf, int mode, size_t size)`

Debe ser la primera operación después de asociar ***stream*** a un archivo (excepto alguna llamada anterior sin éxito a `setvbuf()`). ***mode*** define el tipo de *buffer* para el *stream*: ***\_\_IOFBF*** (*fully buffered*), ***\_\_IOLBF*** (*line buffered*), o ***\_IONBF*** (*unbuffered*). Si ***buf*** es un apuntador nulo, el *buffer* lo reservará la misma función; en cualquier caso, ***size*** marca el tamaño deseado para el *buffer*. Devuelve *nonzero* si hay error, o cero si tiene éxito.

## 7.21.6 Formatted input/output functions

Todas estas funciones se comportan como si hubiera un *sequence point* después de las acciones asociadas a cada especificador de conversión.

`int fprintf(FILE * restrict stream, const char * restrict format,...)`

Escribe en ***stream***; ***format*** especifica cómo deben ser convertidos los subsiguientes argumentos. Si sobran argumentos, son evaluados e ignorados; si faltan, *undefined behaviour*. La función retorna al llegar al final de la *format string*.

La *format string* será una secuencia de caracteres *multibyte*, iniciando y acabando en el *shift state* inicial. Tipo de caracteres en la *format string*:

1. Caracteres *multibyte* ordinarios excepto '***%***'. Aquí incluimos todo tipo de caracteres, que se transcriben literalmente.
2. Especificación de conversión: carácter '***%***' seguido de (por este orden):
  - Cero o más *flags* (en cualquier orden).
  - Anchura mínima del campo (opcional). Consiste en un asterisco (***\****) o un entero positivo que no empiece por ***0*** (porque ***0*** se interpretaría como un *flag*). Si el valor a convertir tiene menos anchura, se añaden espacios de relleno, a la izquierda por defecto.
  - Precisión (opcional). Consiste en un punto '***.***' seguido por un asterisco '***\****' o por un entero decimal; si solo se especifica el punto, se toma como precisión cero. Especifica el mínimo número de dígitos que aparecerán en la conversión para los tipos ***diouxX***, el número de dígitos después del punto decimal en los tipos ***aAeEfF***, el número máximo de dígitos significantes para los tipos ***gG***, o el máximo número de *bytes* escritos para el tipo ***s***.
  - Modificador de longitud (opcional).
  - Especificador de conversión concreta.

En el caso del uso de '***\****' (anchura y precisión), indicamos que el valor de esa anchura o precisión viene dada en un argumento, que deberá ser `int`, y que irá antes del argumento a convertir; en caso de aparecer ambos campos como argumento, el que indica anchura va antes del que indica precisión; un argumento de anchura negativo se toma como un *flag* '***-***' seguido de una anchura positiva; un argumento de precisión negativo se toma como si no estuviera especificada la precisión.

*Flags*:

- '***-***' ==> justificado a la izquierda (a la derecha si no está presente).
- '***+***' ==>preceder siempre el número de su signo.
- ' ' (espacio) ==> si es positivo, o si no resultan caracteres de la conversión, prefijar a espacio. Si se usa con '***+***', se ignora ' '.
- '***#***' ==> "forma alternativa": en ***o***, el primer digito es '***0***' (ampliando precisión si hace falta); en ***x***, le precede '***0x***'; en ***X***, '***0X***'; en ***aAeEfFgG***, aparecerá el punto decimal aunque no aparezcan decimales; en ***gG***, no se eliminan los *trailing zeros*.
- '***0***' ==> en ***diouxXaAeEfFgG***, en lugar de rellenar con espacios por delante, rellenamos con ceros, exceptuando las conversiones de ***infinity*** y ***NaN***. Si se especifica '***-***', se ignora '***0***'. En ***diouxX***, si se especifica una precisión, se ignora '***0***'.

Los modificadores de longitud definen el tipo del argumento pasado en conversiones enteras (***diouxX***), apuntador (***n***), reales (***aAeEfFgG***), carácter (***c***) o *string* (***s***). Se convierte al tipo especificado por estos modificadores antes de la conversión.

- ***hh*** ==> el entero es `signed char` o `unsigned char`; o el apuntador es `signed char*`.
- ***h*** ==> el entero es `short int` o `unsigned short int`; o el apuntador es `short int*`.
- ***l*** ==> el entero es `long int` o `unsigned long int`; o el apuntador es `long int*`; o el carácter es `wint_t`; o el *string* es `wchar_t*`.
- ***ll*** ==> el entero es `long long int` o `unsigned long long int`; o el apuntador es `long long int*`.
- ***j*** ==> el entero es `intmax_t` o `uintmax_t`; o el apuntador es `intmax_t*`.
- ***z*** ==> el entero es `size_t` o su correspondiente tipo *signed*; o el apuntador es un apuntador a este último.
- ***t*** ==> el siguiente entero es `ptrdiff_t` o su correspondiente tipo *unsigned*; o el siguiente apuntador es `ptrdiff_t*`.
- ***L*** ==> el siguiente real es `long double`.

Especificadores de conversión:

- ***d***, ***i*** ==> argumento `int`. Estilo ***[-]nnnn***. Precisión indica mínimo número de dígitos (relleno con ceros si hace falta). Precisión por defecto es 1. Valor 0 con precisión 0 sería "no caracteres".
- ***o***, ***u***, ***x***, ***X*** ==> igual para `unsigned int`, pero sin signo, y representación octal (***o***), decimal (***u***) o hexa (***x***, ***X***); las letras '***abcdef***' (***x***) serán '***ABCDEF***' en el caso de ***X***.
- ***f***, ***F*** ==> argumento `double`. Estilo ***[-]nnn.nnn***. Precisión marca número de decimales; precisión por defecto 6 (siempre se redondea a la precisión adecuada). Si la precisión es 0, y no añadimos el *flag* '***#***', no aparece el punto decimal. Si aparece el punto, por lo menos hay un digito precediéndole. Dependiendo del valor puede producir ***[-]inf*** o ***[-]infinity*** (*implementation defined*); ***[-]nan*** o ***[-]nan(secuencia de caracteres)***, siendo el significado de la secuencia de caracteres *implementation defined*. Si la conversión es tipo ***F***, será lo mismo pero en mayúsculas todo. Cuando el valor es infinito o *NaN*, los *flags* '***#***' y '***0***' no tienen efecto.
- ***e***, ***E*** ==> igual, pero estilo ***[-]n.nnne±nn***; en la parte entera siempre 1 solo digito (diferente de 0 si el valor no es 0); precisión por defecto es 6. Exponente ('***e***', o '***E***' si conversión tipo ***E***) siempre 2 dígitos por lo menos (exponente 0 si valor es 0).
- ***g***, ***G*** ==> argumento `double`. Hará una conversión ***f*** o ***e*** (***F*** o ***E*** si hemos especificado ***G***) dependiendo del valor del parámetro y de la precisión elegida. Sea ***P*** la precisión definida, o 6 si se omite, o 1 si la definimos en 0. Si una conversión de tipo ***E*** resultara en un exponente X, entonces: si ***P > X >= -4***, la conversión será tipo ***fF*** y la precisión resultante ***P - (X + 1)***; si no, conversión ***eE*** y precisión ***P - 1***. Excepto si se especifica el *flag* '***#***', se eliminan los ceros finales de la parte decimal, y si es posible, el punto decimal.
- ***a***, ***A*** ==> argumento `double`. Estilo ***[-]0xn.nnnnp±d***. Formato hexadecimal. 1 dígito en la parte entera (*nonzero* si es un *floating point* normalizado). Dígitos después del punto, igual a la precisión especificada. Si no se especifica precisión y ***FLT_RADIX*** es una potencia de 2, la precisión es suficiente para representar el valor, y si no es una potencia de 2, la precisión es la suficiente para representar valores `double`, aunque se pueden omitir los *trailing zeros*. El exponente tiene mínimo un dígito, y tantos como sean necesarios para expresar el exponente decimal de 2. Todo minúsculas (***a***) o mayúsculas (***A***).
- ***c*** ==> argumento `int`. Lo convierte a `unsigned char`, y se escribe el carácter resultante (no el número). Pero si se incluye un modificador de longitud ***l***, el argumento es `wint_t`, y se convierte a carácter *wide*: se trata como si fuera un *string* de `wchar_t` con un solo carácter (más el *wide null*) y escribiéramos ese *array* (con ***%ls***), siendo ese elemento el valor del `win_t` (convertido a `wchar_t`).
- ***s*** ==> argumento `char*`. Escribe el *string* hasta (no incluyendo) el carácter nulo, o hasta lo definido por la precisión. Pero si se incluye un modificador de longitud ***l***, el argumento será un apuntador a un *array* `wchar_t`, conviertiendo sus caracteres a caracteres *multibyte*, empezando con un *initial shift state* cero; se escribe esa cadena *multibyte* hasta el carácter nulo (no incluido). La precisión marca el número **máximo** de *bytes* a producir (incluyendo *shift sequences*). En ningún caso se queda un carácter *multibyte* escrito parcialmente.
-   ***p*** ==> argumento `void*`. Se imprime el valor del apuntador en una forma *implementation-defined*.
- ***n*** ==> no se convierte ese argumento; debe ser un apuntador a entero con signo, y sirve para almacenar el *número de caracteres* escritos hasta el momento.

Para escribir un signo '***%***' se debe especificar '***%%***'.

Una anchura pequeña no trunca el campo; solo sirve para ampliarlo.

La función `fprintf()` devuelve el número de caracteres escritos o un número negativo si hay error.

`int fscanf(FILE * restrict stream, const char * restrict format,...)`

Lee del *stream* definido por ***stream***. ***format*** marca las secuencias de entrada admisibles y como se irán convirtiendo para asignación a los apuntadores que forman el resto de argumentos.

***format*** será una secuencia de caracteres *multibyte*, empezando y finalizando en su shift state inicial, compuesta por espacios, caracteres normales (excepto espacio y '***%***') o secuencias de conversión. Estas últimas empiezan por '***%***', seguido de:

- '***\****' ==> *optional assignment-suppressing character*.
- Un entero opcional mayor que cero, que especifica el ancho máximo del campo (en caracteres).
- Un modificador de longitud opcional.
- Un especificador de conversión.

La función termina por fallo (de entrada o de *matching*) o por terminar de procesar la *format string*. Un espacio (o varios) en la *format string* se interpretan leyendo todos los caracteres espacio consecutivamente hasta encontrar un carácter que no lo es (que no quedará leído), o hasta que no hay caracteres para leer (los espacios de ***format*** nunca producen error). Un carácter ordinario se intenta *match* con el siguiente del fichero; si no coinciden, fallo.

Una conversión se ejecuta mediante los siguientes pasos:

- El espacio blanco de la entrada (tal como se define en `isspace()`, de ***ctype.h***) es ignorado (a no ser que la conversión incluya especificador ***[***, ***c*** o ***n***).
- Se lee un elemento (*item*) del *stream*, a no ser que la conversión incluya especificador ***n***. Un *item* es la secuencia más larga de caracteres que no excede ninguna anchura de campo especificada y que es (o es prefijo de) una *matching sequence*.
- Excepto en el caso del especificador '***%***', el *item* de entrada (o el número de caracteres de entrada, en el caso de ***%n***) se convierte al tipo apropiado. El valor se almacena en la dirección apuntada por el siguiente argumento, a no ser que hayamos especificado *assignment supression* con '***\****'.

Modificadores de longitud (por defecto nos referimos a conversiones ***diouxXn***):

- ***hh*** ==> la conversión se aplica a un argumento `signed char*` o `unsigned char*`.
- ***h*** ==> la conversión se aplica a un argumento `short int*` o `unsigned short int*`.
- ***l*** ==> la conversión se aplica a un argumento `long int*` o `unsigned long int*`; en conversiones ***aAeEfFgG***, se aplica a argumento `double*`; en conversiones ***cs[*** se aplica a argumento `wchar_t*`.
- ***ll*** ==> la conversión se aplica a un argumento `long long int*` o `unsigned long long int*`.
- ***j*** ==> la conversión se aplica a un argumento `intmax_t*` o `uintmax_t*`.
- ***z*** ==> la conversión se aplica a un argumento `size_t*` o apuntador al correspondiente tipo *signed*.
- ***t*** ==> la conversión se aplica a un argumento `ptrdiff_t*` o apuntador al correspondiente tipo *unsigned*.
- ***L*** ==> la conversión ***aAeEfFgG*** se aplica a un argurnento `long double*`.

Especificadores de conversión:

- ***d*** ==> *matches* un entero decimal con signo opcional, con el mismo formato que esperaría `strtol()` (***stdlib.h***) con ***base*** 10. Argumento apuntador a un entero con signo.
- ***i*** ==> *matches* un entero con signo opcional, con el mismo formato que esperaría `strtol()` con ***base*** 0. Argumento apuntador a un entero con signo.
- ***o*** ==> *matches* un entero octal con signo opcional, con el mismo formato que esperaría `strtoul()` (***stdlib.h***) con ***base*** 8. Argumento apuntador a un entero sin signo.
- ***u*** ==> *matches* un entero decimal con signo opcional, con el mismo formato que esperaría `strtoul()` con ***base*** 10. Argumento apuntador a un entero sin signo.
- ***x*** ==> *matches* un entero hexadecimal con signo opcional, con el mismo formato que esperaría `strtoul()` con ***base*** 16. Argumento apuntador a un entero sin signo.
- ***a***, ***e***, ***f***, ***g*** ==> *matches* un *floating-point* con signo opcional, infinito o ***NaN*** con el mismo formato que esperaría `strtod()`. Argumento apuntador a un *floating point*.
- ***c*** ==> *matches* el número de caracteres especificado por *width* (si no se especifica es 1). Si no hay especificador de longitud ***l***, argumento será `char*` y será capaz de almacenar la secuencia (no se añade carácter nulo). Si sí existe ***l***, argumento será `wchar_t*` y será capaz de almacenar la secuencia de caracteres *multibyte* que empezará y terminará en el *initial shift state* (no se añade carácter nulo); la conversión de *multibyte* a `wchar_t` se hace según `mbrtowc()`, con ***mbstate_t*** inicializado a ***0*** antes de convertir el primer carácter.
- ***s*** ==> es igual que la conversión ***c*** excepto que solo *matches* una secuencia de caracteres diferentes de espacios. Otra diferencia es que sí inserta carácter nulo al final.
- ***[*** ==> es igual que la conversión ***c***, excepto que solo *matches* una secuencia de caracteres esperados (el *scanset*). Al igual que en la conversión ***s***, inserta carácter nulo al final. La *scanlist* se define escribiendo los caracteres de la misma justo después del '***[***', y finalizando con '***]***'. Si el primer carácter después del corchete inicial es '***^***', el *scanset* estará formado por todos los caracteres que no están en la *scanlist*. Si queremos referirnos al carácter '***]***' en la *scanlist*, debe ser el primero de la misma (después de '***[***' o después del '***[^***' inicial). El carácter '***-***' no tiene ningún significado especial si es el primero o último de la lista; de lo contrario puede tener una función *implementation-defined*.
- ***p*** ==> *matches* una dirección de memoria con un formato *implementation-defined* y lo guarda en el argumento, que es un apuntador a `void*` (es decir, un `void**`). El formato es el mismo que produce `fprintf()` en una conversión ***%p***.
- ***n*** ==> argumento apuntador a un entero con signo. Almacena en la variable el número de caracteres que se han leído del *stream* hasta el momento. No se asigna el valor a ninguna variable. No incrementa el número de conversiones realizadas en cuanto al valor de retorno.

'***%%***' *matches* un solo carácter '***%***'. Los especificadores de conversión ***AEFGX*** se comportan igual que, respectivamente, ***aefgx***.

`fscanf()` retorna el valor de la macro ***EOF*** si hay fallo en la entrada antes de que se produzca la primera conversión, de lo contrario retorna el número de asignaciones realizadas.

`int printf(const char * restrict format,...)`

Es equivalente a `fprintf()`, pero enviando la salida a ***stdout***.

`int scanf(const char * restrict format,...)`

Es equivalente a `fscanf()` pero lee de ***stdin***.

`int snprintf(char * restrict s, size_t n, const char * restrict format,...)`

Es equivalente a `fprintf()`, pero escribiendo en el *array* ***s*** hasta ***n - 1*** caracteres; al final escribe un carácter nulo. Si ***n*** es ***0***, no se escribe nada (y ***s*** puede ser ***NULL***). Retorna la cantidad de caracteres que se habrían escrito si ***n*** hubiera tenido el tamaño necesario (sin contar el *null character* final), o un número negativo si hay error de codificación.

`int sprintf(char * restrict s, const char * restrict format,...)`

Es equivalente a `snprintf()`, pero sin especificar tamaño del *array* ***s***.

`int sscanf(const char * restrict s, const char * restrict format,...)`

Es equivalente a `fscanf()` pero lee del *string* ***s***. Llegar al final del *string* es equivalente al *end-of-file* del *stream* en `fscanf()`.

`int vfprintf(FILE * restrict stream, const char * restrict format, va_list arg)`

Es equivalente a `fprintf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `vfprintf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

`int vfscanf(FILE * restrict stream, const char * restrict format, va_list arg)`

Es equivalente a `fscanf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `vfscanf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

`int vprintf(const char * restrict format, va_list arg)`

Es equivalente a `printf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `vprintf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

`int vscanf(const char * restrict format, va_list arg)`

Es
equivalente a `scanf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `vscanf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

`int vsnprintf(char * restrict s, size_t n, const char * restrict format, va_list arg)`

Es equivalente a `snprintf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `vsnprintf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

`int vsprintf(char * restrict s, const char * restrict format, va_list arg)`

Es equivalente a `sprintf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `vsprintf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

`int vsscanf(const char * restrict s, const char * restrict format, va_list arg)`

Es equivalente a `sscanf()`, pero se cambia la lista variable de parámetros por ***arg***, que deberemos haber inicializado con la macro ***va_start***. `sscanf()` no invoca ***va_end***. Necesita el *header* ***stdarg.h***.

## 7.21.7 Character input/output functions

`int fgetc(FILE *stream)`

Lee el siguiente carácter del *stream* como `unsigned char`, y lo retorna convertido a `int`, siempre que haya carácter disponible y el indicador de *end-of-file* para el *stream* no esté activado. Si hay error de lectura (activa el indicador de error del *stream*), *end-of-file* está activado, o llegamos al fin del archivo, retorna ***EOF***.

`char *fgets(char * restrict s, int n, FILE * restrict stream)`

Lee un maximo de ***n - 1*** caracteres de ***stream*** y los escribe en ***s***. No se escriben más caracteres después de un *end of file* ni después de un *newline* (que se copia). Al final coloca un null character. Si tiene éxito retorna ***s***; si llega el *end of file* y no se ha leído ningún carácter, ***s*** queda sin cambio y retorna ***NULL***. Si hay error de lectura, retorna ***NULL***.

`int fputc(int c, FILE *stream)`

Escribe el carácter ***c*** convertido a `unsigned char` al ***stream***. Si tiene éxito retorna el carácter y avanza el puntero del *stream*. En caso de error, activa el indicador de error del *stream* y retorna ***EOF***.

`int fputs(const char * restrict s, FILE * restrict stream)`

Escribe el *string* ***s*** al ***stream***. El *null terminator* no se escribe. Retorna un número no negativo si tiene éxito; ***EOF*** si no.

`int getc(FILE *stream)`

Es igual que `fgetc()`, pero implementado como macro. Puede evaluar ***stream*** más de una vez, con lo que es mejor que el argumento no sea una expresión con *side effects*.

`int getchar(void)`

Es como `getc()`, con ***stdin*** como *stream* entrada.

`int putc(int c, FILE *stream)`

Es igual que `fputc()`, pero implementado como macro. Puede evaluar ***stream*** más de una vez, con lo que es mejor que el argumento no sea una expresión con *side effects*.

`int putchar(int c)`

Es como `putc()`, con ***stdout*** como *stream* de salida.

`int puts(const char *s)`

Escribe el *string* ***s*** + un *newline* en ***stdout***. No escribe el *null terminator*. Retorna un número no negativo si tiene éxito; de lo contrario, ***EOF***.

`int ungetc(int c, FILE *stream)`

Empuja (*pushes*) el carácter ***c*** convertido a `unsigned char` al *stream*. Los caracteres *pushed* leídos del *stream* se recuperan en orden inverso al que fueron *pushed* (como una pila). Una llamada exitosa a una función de reposicionamiento del puntero del *stream* descarta los últimos *pushed-back characters* sin leer. Se garantiza 1 *pushback* al *stream*, pero si se hacen demasiados `ungetc()` seguidos sin una sola operación de lectura o reposicionamiento, puede fallar. Si intentamos *unread* el carácter ***EOF***, la operación falla y no hace nada. Al descartar o leer todos los caracteres *pushed* en un *stream*, el indicador de posicion del *stream* recupera el mismo valor que tenía antes de *unread* el primero de esos caracteres. En caso de éxito, el indicador *end-of-file* del *stream* se borra. Retorna el carácter *pushed back*, o ***EOF*** si hay error.

## 7.21.8 Direct input/output functions

`size_t fread(void * restrict ptr, size_t size, size_t nmemb, FILE * restrict stream)`

Lee de ***stream*** hasta ***nmemb*** elementos (o hasta el *end of file*) de ***size*** *bytes* y los coloca *byte* a *byte* (`unsigned char`) en ***ptr***. El indicador de posición del *stream* es incrementado en el número de caracteres leídos correctamente. Si hay error o se lee un elemento parcialmente, el valor del indicador es indeterminado. La función devuelve el número de elementos leídos, que será menor a ***nmemb*** si hay *end of file* o error de lectura. Si ***nmemb*** o ***size*** son ***0***, la función no hace absolutamente nada y retorna ***0***.

`size_t fwrite(const void * restrict ptr, size_t size, size_t nmemb, FILE * restrict stream)`

Escribe hasta ***nmemb*** elementos de ***size*** *bytes*, *byte* a *byte* (`unsigned char`), del *array* apuntado por ***ptr*** hacia ***stream***. El indicador de posición del *stream* avanza tantos *bytes* como se hayan escrito con éxito; si hay error, su valor es indeterminado. La función retorna el número de elementos escritos, que será menor a ***nmemb*** si hay error de escritura. Si ***nmemb*** o ***size*** son ***0***, la función no hace absolutamente nada y retorna ***0***.

## 7.21.9 File positioning functions

`int fgetpos(FILE * restrict stream, fpos_t * restrict pos)`

Lee el indicador de posición del *stream* y los valores de su *parse state* (si lo hay) y lo almacena en el objeto apuntado por ***pos***. Si tiene éxito, retorna ***0***; en caso de fallo, retorna *nonzero* y establece ***errno*** (***errno.h***) a un valor positivo *implementation-specific*.

`int fseek(FILE *stream, long int offset, int whence)`

Coloca el indicador de posición del *stream* binario en la posicion ***offset*** (caracteres), en relación a ***whence***, que puede ser ***SEEK_SET*** (principio del *stream*), ***SEEK_CUR*** (posición actual) o ***SEEK_END*** (final del *stream*). Si hay error, establece el indicador de error del *stream*. En *streams* de texto el funcionamiento se debería limitar a un ***offset*** de ***0***, o posiciones retornadas por `ftell()`, y usando siempre ***SEEK_SET*** para asegurarnos de un funcionamiento correcto. Una llamada exitosa a `fseek()` descarta los últimos caracteres *unread* (`ungetc()`), borra el indicador de *end-of-file* del *stream*, y establece la
nueva posición. Solo retorna *nonzero* si hay error.

`int fsetpos(FILE *stream, const fpos_t *pos)`

Establece el indicador de posición del *stream*, así como los valores de su *parse state* (objeto ***mbstate_t***, si lo hay), según especifique el objeto apuntado por ***pos*** (obtenido anteriormente con `fgetpos()` sobre el mismo fichero). Si es exitoso, descarta los últimos caracteres *unread* (`ungetc()`), borra el indicador *end-of-file* del *stream* y establece la nueva posición. Si tiene éxito, retorna ***0***; si hay fallo, retorna *nonzero* y establece ***errno*** a un valor positivo *implementation-specific*.

`long int ftell(FILE *stream)`

Lee el valor del indicador de posición del *stream* y retorna ese dato. Para archivos binarios, es el número de caracteres desde el principio del fichero. Para archivos de texto, el valor es inespecífico, aunque puede usarse en conjunción con `fseek()` para restaurar esa posición. Si hay error, establece ***errno*** a un valor positivo *implementation-specific*, y retorna ***-1L***.

`void rewind(FILE *stream)`

Equivale a `(void)fseek(stream, 0L, SEEK_SET)`, con la diferencia de que también borra el indicador de error del *stream*. No retorna valor.

## 7.21.10 Error-handling functions

`void clearerr(FILE *stream)`

Borra los indicadores de error y *end-of-file* del *stream*. No retorna valor.

`int feof(FILE *stream)`

Retorna *nonzero* si y solo si el indicador de *end-of-file* del *stream* está activo (vale ***1***).

`int ferror(FILE *stream)`

Retorna *nonzero* si y solo si el indicador de error del *stream* está activo (vale ***1***).

`void perror(const char *s)`

Escribe en ***stderr*** la secuencia de caracteres siguiente: primero el *string* ***s***, seguido por dos puntos (***:***) y un espacio; a continuación, el mensaje de error, como el que devolvería `strerror()` (***string.h***) con argumento ***errno*** y un *newline*. Si ***s*** es ***NULL*** o apunta a una cadena vacía, solo se
escribe el mensaje de error con el *newline*. No devuelve valor.
