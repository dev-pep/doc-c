Standard ISO/IEC 9899:2011 (C11)

Resumen del final draft\
(N1570): stdio.h

Actualizado: 18/11/2017

[]{#anchor}7.21 Input/output \<stdio.h\>
========================================

### []{#anchor-1}7.21.1 Introduction

Deﬁne 3 tipos:

-   ***size\_t*** - (definido en *stddef.h*) es el unsigned integer type
    que devuelve el operador ***sizeof***.
-   ***FILE*** - para controlar un stream: contiene indicador de
    posicion, pointer a buffer (si hay), indicador de error, indicador
    de end-of-file.
-   ***fpos\_t*** - tipo completo no array capaz de almacenar la
    información necesaria para especificar una posición única dentro de
    un fichero.

Macros:

-   ***NULL*** - (definido en *stddef.h*) expands to an
    implementation-defined null pointer constant.
-   ***\_IOFBF***, ***\_IOLBF***, ***\_IONBF*** - expresiones constantes
    enteras para usar como tercer argumento de función ***setvbuf()***.
-   ***BUFSIZ*** - expresión constante entera que indica tamaño de
    buffer usado por función ***setbuf()***.
-   ***EOF*** - expresión constante entera negativa devuelta por varias
    funciones para indicar end-of-ﬁle.
-   ***FOPEN\_MAX*** - expresión constante entera con número mínimo de
    ficheros que la implementación garantiza que se pueden abrir
    simultáneamente.
-   ***FILENAME\_MAX*** - expresión constante entera con el tamaño que
    necesita un array de ***char*** para almacenar el nombre de fichero
    más largo posible que la implementación garantiza que pueda abrirse.
-   ***L\_tmpnam*** - como ***FILENAME\_MAX*** pero para nombres de
    ficheros temporales generados por la función ***tmpnam()***.
-   ***SEEK\_CUR***, ***SEEK\_END***, ***SEEK\_SET*** - constantes
    enteras para tercer argumento de función ***fseek()***.
-   ***TMP\_MAX*** - constante entera con numero mínimo de nombres
    únicos de ficheros temporales que pueden generarse con la función
    ***tmpnam()***.
-   ***stderr***, ***stdin***, ***stdout*** - pointers to ***FILE***
    para standard error, standard input and standard output text
    streams.

El header *wchar.h* declara funciones con operaciones análogas a la
mayoría de las descritas aqui, teniendo como diferencia que las unidades
fundamentales internas al programa son wide characters. Su
representación externa (en fichero) son secuencias de caracteres
multibyte (descrito en 7.21.3).

### []{#anchor-2}7.21.2 Streams

La entrada/salida (terminales, cintas, discos,\...) se realiza sobre
streams. Hay 2 tipos: text and binary streams.

Text stream se compone de líneas, terminadas en newline (implementación
puede decidir que en última línea no hace falta newline). Binary stream
permite almacenar datos internos, y puede contener un número
implementation-defined de trailing null chars.

Orientación del stream: si escribimos un wide char en un stream sin
orientación, it becomes wide-oriented stream. Si en lugar de ello
escribimos un byte, it becomes byte-oriented. ***freopen()*** elimina la
orientacion de un stream. ***fwide()*** puede cambiar la orientación del
stream. No se deben escribir bytes en un wide-oriented ni wides en un
byte-oriented. Los wide-oriented streams tienen un objeto
***mbstate\_t*** (*wchar.h*) que guarda el current parse state; en
ellos, ***fgetpos()***/***fsetpos()*** guarda/restaura en el objeto
***fpos\_t*** devuelto/escrito este objeto ***mbstate\_t***.

Los streams tienen un lock de tal modo que solo un thread tiene el lock
en un momento dado. Las funciones que acceden a los streams los bloquean
antes de acceder a ellos.

La implementacion debe soportar text files con líneas de al menos 254
caracteres, incluyendo el newline. ***BUFSIZ*** debe ser al menos 256.

### []{#anchor-3}7.21.3 Files

Un stream se asocia a un fichero externo. Si un stream es unbuffered,
cada carácter escrito/leído lo hace cuanto antes. De lo contrario, se
pueden escribir/leer bloque a bloque. Fully buffered, los bloques se
escriben/leen al llenarse el buffer; line buffered, el bloque se
escribe/lee al encontrar un newline.

Cuando se cierra el fichero, se disocia del stream. Los output streams
son flushed. Si se llega al final de ***main()***, o se llama a la
función ***exit()***, todos los files are closed , por lo que los output
streams son ﬂushed. Otros caminos de salida, incluida llamada a función
***abort()***, pueden dejar archivos abiertos.

Una copia de un objeto ***FILE*** a otra variable no tiene por que
servir como sustituto correcto, ya que la dirección de almacenamiento
del objeto puede ser significante.

El standard error stream no es fully buffered. Los standard input y
output son fully buffered si y solo si no se refieren a streams
interactivos.

Si un mismo fichero puede abrirse varias veces simultáneamente es
implementation-defined.

Los text y binary wide-oriented streams son secuencias de wide chars, y
los ficheros asociados son multibyte ﬁles. Las codiﬁcaciones multibyte
de los ficheros son implementation-defined. Colocar la file position al
final (***fseek(file,0,SEEK\_END)***) tiene undefined behaviour en un
wide binary stream (por los posibles trailing null chars) o cualquier
wide stream con state-dependent encoding que no termine forzosamente en
el initial shift state.

***FOPEN\_MAX*** deberia ser por lo menos 8 (incluyendo los 3 streams de
texto estándar).

### []{#anchor-4}7.21.4 Operations on files

***int remove(const char \*filename)*** borra fichero. Returns zero on
success, nonzero on fail. Si el fichero está abierto, el comportamiento
es implementation-defined.

***int rename(const char \*old,const char \*new)*** renombra el fichero
de \'old\' a \'new\'. Si el fichero \'new\' ya existe, el comportamiento
es implementation-defined. Return zero on success, nonzero on fail.

***FILE \*tmpfile(void)*** crea un temporary file en modo ***wb+***,
diferente de todos los abiertos; se borra automaticamente al cerrarse o
terminar el programa. Devuelve pointer al stream, o ***NULL*** on fail.

***char \*tmpnam(char \*s)*** genera un file name distinto al de todos
los files abiertos y lo almacena en \'s\'. La funcion es capaz de
generar como mínimo ***TMP\_MAX*** strings distintos, aunque si ya
existen varios (o todos) de esos nombres en uso, no se pueden considerar
return values válidos. La función genera un string distinto cada vez.
Devuelve ***NULL*** si no puede generar un nombre; de lo contrario,
devuelve el mismo apuntador del argumento, a no ser que le pasemos
***NULL***, en cuyo caso almacenara el resultado en un string estático
interno y devuelve un apuntador al mismo. El string que le pasamos debe
tener al menos ***L\_tmpnam*** caracteres de capacidad. Llamadas
simultáneas con argumento ***NULL*** pueden producir data races.

### []{#anchor-5}7.21.5 File access functions

***int fclose(FILE \*stream)*** closes file, lo disocia del stream y
ﬂushes the stream (writes output data pendiente, discards input data
pendiente). Returns 0 on success or ***EOF*** on fail.

***int fflush(FILE \*stream)*** it flushes output streams, or update
(i+o) streams donde la última operación no fue de input. If \'stream\'
is a null pointer, it performs the operation on all open streams of the
mentioned characteristics. Returns 0 on success; on fail, ***EOF*** (and
sets the error indicator for the stream).

***FILE \*fopen(const char \* restrict filename,const char \* restrict
mode)*** opens the file \'filename\' and associates a stream with it.
Modes: ***r*** (text file read only), ***w*** (truncate to 0 length or
create text file; write only), ***wx*** (create text file for writing),
***a*** (append; open or create text file for writing at the end of
file). Same for binary: ***rb***, ***wb***, ***wbx***, ***ab***. Same
for update (read *and* write): ***r+*** (read/write text file), ***w+***
(truncate to 0 or create text file; read/write), ***a+*** (append; open
or create text file for update; writes at the end of ﬁle). Update for
binary: ***r+b***, ***w+b***, ***w+bx***, ***a+b***; o ***rb+***,
***wb+***, ***wb+x***, ***ab+***.

En readonly, fails si no existe el fichero. En exclusive mode (***x***)
fails si ya existe o no se puede crear. Si se crea, lo hace en modo
exclusivo (no compartido). En algunas implementaciones, abrir en modo
append binario puede dejar el file position mas allá del último carécter
escrito, debido al trailing null padding.

Cuando se abre en modo update (***+***), se puede hace tanto entrada
como salida en el stream, pero en ese caso no debemos hacer un input
justo después de un output sin hacer un ﬂush o un file positioning
(***fseek()***, ***fsetpos()***, ***rewind()***) antes; del mismo modo,
no hacer un output después de un input sin una llamada a una file
positioning function, excepto si ese input ha encontrado el end of file.

Al abrirse, un stream es fully buffered si y solo si se puede determinar
que no es un interactive device; los indicadores de error y end of file
son cleared.

***fopen()*** devuelve un pointer al objeto controlador del stream, o un
null pointer si falla.

***FILE \*freopen(const char \* restrict filename, const char \*
restrict mode, FILE \* restrict stream)*** abre el file \'filename\' y
lo asocia al stream \'stream\', usando el mode \'mode\'. Si \'filename\'
es un pointer null, simplemente cambiará el modo del stream (los cambios
de modo permitidos son implementation-defined). ***freopen()*** intenta
antes de nada cerrar cualquier fichero que estuviera asociado al
\'stream\', pero ignorará posibles errores en este punto. Los
indicadores de error y end of file son cleared. Devuelve \'stream\', o
null pointer si falla.

***void setbuf(FILE \* restrict stream, char \* restrict buf)***
exceptuando que no devuelve nada, equivale a ***setvbuf()*** con
\'mode\' ***\_IOFBF*** y \'size\' ***BUFSIZ***, o si \'buf\' es un null
pointer, con \'mode\' ***\_IONBF***.

***int setvbuf(FILE \* restrict stream, char \* restrict buf, int mode,
size\_t size)*** debe ser la primera operación después de asociar
\'stream\' a un file (excepto alguna llamada anterior sin éxito a
***setvbuf()***). \'mode\' define el tipo de buffer para el stream:
***\_\_IOFBF*** (full buffered), ***\_\_IOLBF*** (line buffered), o
***\_IONBF*** (unbuffered). Si \'buf\' es un null pointer, el buffer lo
reservará la misma función; en cualquier caso, \'size\' marca el tamaño
deseado para el buffer. Devuelve nonzero on error, or 0 on success.

### []{#anchor-6}7.21.6 Formatted input/output functions

Todas estas funciones se comportan como si hubiera un sequence point
después de las acciones asociadas a cada especiﬁcador de conversión.

***int fprintf(FILE \* restrict stream, const char \* restrict
format,\...)*** escribe en \'stream\'; \'format\' especifica cómo deben
ser convertidos los subsiguientes argumentos. Si sobran argumentos, son
evaluados e ignorados; si faltan, undefined behaviour. La función
retorna al llegar al ﬁnal de la format string.

La format string será una multibyte char sequence, iniciando y acabando
en el initial shift state. Tipo de caracteres en la format string:

1\. Caracteres multibyte ordinarios excepto \'***%***\'. Aqui incluimos
todo tipo de caracteres, que se transcriben literalmente.

2\. Conversion specification: \'***%***\' seguido de (por este orden):

-   Zero or more *ﬂags* (in any order).
-   Minimum field *width* (optional). Consiste en un \'***\****\' o un
    entero positivo que no empiece por \'***0***\' (porque \'***0***\'
    se interpretaria como un ﬂag). Si el valor a convertir tiene menos
    width, se añaden padding spaces, a la izquierda por defecto.
-   *Precision* (optional). Consiste en un punto \'.\' seguido por un
    \'***\****\' o por un entero decimal; si solo se especifica el
    punto, se toma como precisión cero. Especifica el mínimo número de
    dígitos que aparecerán en la conversión para ***diouxX***, el número
    de dígitos después del punto decimal en ***aAeEfF***, el número
    máximo de dígitos significantes para ***gG***, o el máximo número de
    bytes escritos para ***s***.
-   *Length* modifier (optional).
-   *Conversion* specifier.

En el caso del uso de \'***\****\' (width y precision), indicamos que el
valor de esa anchura o precisión viene dada en un argurnento, que deberá
ser ***int***, y que irá antes del argumento a convertir; en caso de
aparecer ambos campos como argumento, el que indica width va antes del
que indica precisión; un argumento width negativo se toma como un ﬂag
\'***-***\' seguido de una width positive; un argumento precision
negativo se toma como si no estuviera especificada la precisión.

Flags:

-   \'***-***\' - left-justified (right justified if not present).
-   \'***+***\' - preceder siempre el número de su signo.
-   \' \' - si es positivo, o si no resultan caracteres de la
    conversion, prefix a space. Si se usa con \'***+***\', se ignora \'
    \'.
-   \'***\#***\' - «forma alternativa»: en ***o***, el primer digito es
    \'***0***\' (ampliando precision si hace falta); en ***x***, le
    precede \'***0x***\'; en ***X***, \'***0X***\'; en ***aAeEfFgG***,
    aparecerá el punto decimal aunque no aparezcan decimales; en
    ***gG***, no se eliminan los trailing zeros.
-   \'***0***\' - en ***diouxXaAeEfFgG***, en lugar de rellenar con
    espacios por delante, rellenamos con ceros, exceptuando las
    conversiones de infinity y NaN. Si se especifica \'***-***\', se
    ignora \'***0***\'. En ***diouxX***, si se especiﬁca una precisión,
    se ignora \'***0***\'.

Los length modifiers definen el tipo del argumento pasado en
conversiones enteras (***diouxX***), pointer (***n***), reales
(***aAeEfFgG***), carácter (***c***) o string (***s***). Se convierte al
tipo especificado por estos modifiers antes de la conversión.

-   ***hh*** - el entero es ***signed char*** o ***unsigned char***; o
    el pointer es ***signed char\****.
-   ***h*** - el entero es ***short int*** o ***unsigned short int***; o
    el pointer es ***short int\****.
-   ***l*** - el entero es ***long int*** o ***unsigned long int***; o
    el pointer es ***long int\****; o el carácter es ***wint\_t***; o el
    string es ***wchar\_t***\*.
-   ***ll*** - el entero es ***long long int*** o ***unsigned long long
    int***; o el pointer es ***long long int\****.
-   ***j*** - el entero es ***intmax\_t*** o ***uintmax\_t***; o el
    pointer es ***intmax\_t\****.
-   ***z*** - el entero es ***size\_t*** o su correspondiente signed
    type; o el pointer es un apuntador a este último.
-   ***t*** - el siguiente entero es ***ptrdiff\_t*** o su
    correspondiente unsigned type; o el siguiente pointer es
    ***ptrdiff\_t\****.
-   ***L*** - el siguiente real es ***long double***.

Conversion specifiers:

-   ***d***, ***i*** - argumento ***int***. Estilo \[-\]nnnn. Precision
    indica mínimo número de dígitos (padding con ceros si hace falta).
    Default precision es 1. Valor 0 con precisión 0 sería no characters.
-   ***o***, ***u***, ***x***, ***X*** - igual para ***unsigned int***,
    pero sin signo, y representación octal (***o***), decimal (***u***)
    o hexa (***x***, ***X***); las letras \'***abcdef***\' (***x***)
    seran \'***ABCDEF***\' en el caso de ***X***.
-   ***f***, ***F*** - argumento ***double***. Estilo \[-\]nnn.nnn.
    Precision marca número de decimales; default precision 6 (siempre se
    redondea a la precisión adecuada). Si la precisión es 0, y no
    añadimos el ﬂag \'***\#***\', no aparece el punto decimal. Si
    aparece el punto, por lo menos hay un digito precediéndole.
    Dependiendo del valor puede producir ***\[-\]inf*** o
    ***\[-\]infinity*** (implementation defined); ***\[-\]nan*** o
    ***\[-\]nan(char sequence)***, being the meaning de la secuencia
    implementation defined. Si la conversión es tipo ***F***, será lo
    mismo pero en mayúsculas todo. Cuando el valor es infinito o NaN,
    los ﬂags \'***\#***\' y \'***0***\' no tienen efecto.
-   ***e***, ***E*** - igual, pero estilo \[-\]n.nnn**e**±nn; en la
    parte entera siempre 1 solo digito (diferente de 0 si el valor no es
    0); default precision es 6. Exponente (\'***e***\', o \'***E***\' si
    conversion tipo ***E***) siempre 2 digitos por lo menos (exponente 0
    si valor es 0).
-   ***g***, ***G*** - argumento ***double***. Hará una conversión
    ***f*** o ***e*** (***F*** o ***E*** si hemos especificado ***G***)
    dependiendo del valor del parámetro y de la precisión elegida. Sea P
    la precisión definida, o 6 si se omite, o 1 si la definimos en 0. Si
    una conversión de tipo ***E*** resultará en un exponente X,
    entonces: si P \> X \>= -4, la conversión será tipo ***fF*** y la
    precisión resultante P-(X+1); si no, conversión ***eE*** y precisión
    P-1. Excepto si se especifica el ﬂag \'***\#***\', se eliminan los
    ceros finales de la parte decimal, y si es posible, el punto
    decimal.
-   ***a***, ***A*** - argumento ***double***. Estilo
    \[-\]0xn.nnnn**p**±d. Formato hexadecimal. 1 dígito en la parte
    entera (nonzero si es un ﬂoating point normalizado). Dígitos después
    del punto, igual a la precisión especificada. Si no se especifica
    precisión y ***FLT\_RADIX*** es una potencia de 2, la precisión es
    suficiente para representar el valor, y si no es una potencia de 2,
    la precisión es la suficiente para representar valores ***double***,
    aunque trailing zeros might be omitted. El exponente tiene mínimo un
    dígito, y tantos como sean necesarios para expresar el exponente
    decimal de 2. Todo minúsculas (***a***) o mayúsculas (***A***).
-   ***c*** - argumento ***int***. Lo convierte a ***unsigned char***, y
    se escribe el carácter resultante (no el número). Pero si se incluye
    un length modifier ***l***, el argumento es ***wint\_t***, y se
    convierte a wide character: se trata como si fuera un string de
    ***wchar\_t*** con un solo carácter (más el wide null) y
    escribiéramos ese array (con ***%ls***), siendo ese elemento el
    valor del ***win\_t*** (convertido a ***wchar\_t***).
-   ***s*** - argumento ***char\****. Escribe el string hasta (not
    including) el carácter nulo, o hasta lo definido por la precisión.
    Pero si se incluye un length modifier ***l***, el argumento seré un
    pointer a array ***wchar\_t***, conviertiendo sus caracteres a
    multibyte characters, empezando con un initial shift state cero; se
    escribe esa cadena multibyte hasta el carácter nulo (no incluido).
    La precisión te marca el número *máximo* de *bytes* a producir
    (incluyendo shift sequences). En ningún caso se queda un carácter
    multibyte escrito parcialmente.
-   ***p*** - argumento ***void\****. Se imprime el valor del pointer en
    una forma implementation-defined.
-   ***n*** - no se convierte ese argumento; debe ser un apuntador a
    entero con signo, y sirve para almacenar el *número de caracteres*
    escritos so far.

Para escribir un signo \'***%***\' se debe especificar \'***%%***\'.

Un width pequeño no trunca el campo; solo sirve para ampliarlo.

La función ***fprintf()*** devuelve el número de caracteres escritos o
un número negativo si hay error.

***int fscanf(FILE \* restrict stream, const char \* restrict
format,\...)*** lee del stream deﬁnido por \'stream\'. \'format\' marca
las secuencias de entrada admisibles y como se irán convirtiendo para
asignación a los apuntadores que forman el resto de argumentos.

\'format\' será una secuencia de carateres multibyte, empezando y
finalizando en su shift state inicial, compuesta por espacios,
caracteres normales (excepto espacio y \'***%***\') o secuencias de
conversión. Estas últimas empiezan por \'***%***\', seguido de:

-   \'***\****\' - optional assignment-suppressing character.
-   Un entero opcional mayor que cero, que especifica el ancho máximo
    del campo (en caracteres).
-   Un length modifier opcional.
-   Un conversion specifier.

La función termina por fallo (de entrada o de matching) o por terminar
de procesar la format string. Un espacio (o varios) en la format string
se interpreta leyendo todos los caracteres espacio hasta el primero que
no lo es (que no queda leído), o hasta que no hay caracteres para leer
(los espacios de \'format\' nunca producen error). Un carácter ordinario
se intenta match con el siguiente del fichero; si no coinciden, fallo.
Una conversión se ejecuta mediante los siguientes pasos:

-   White space de la entrada (as defined in ***isspace()***, de
    *ctype.h*) es skipped (a no ser que la conversión incluya
    especificador ***\[***, ***c*** o ***n***).
-   Se lee un item del stream, a no ser que la conversión incluya
    especificador ***n***. Un item es la secuencia más larga de
    caracteres que no excedan ninguna anchura de campo especificada y
    que es (o es prefijo de) una matching sequence.
-   Excepto en el caso del especificador \'***%***\', el item de entrada
    (o el número de caracteres de entrada, en el caso de ***%n***) se
    convierte al tipo apropiado. El valor se almacena en la dirección
    apuntada por el siguiente argumento, a no ser que hayamos
    especificado assignment supression con \'***\****\'.

Length modifiers (por defecto nos referimos a conversiones
***diouxXn***):

-   ***hh*** - la conversión se aplica a un argumento ***signed
    char\**** o ***unsigned char\****.
-   ***h*** - la conversión se aplica a un argumento ***short int\**** o
    ***unsigned short int\****.
-   ***l*** - la conversión se aplica a un argumento ***long int\**** o
    ***unsigned long int\****; en conversiones ***aAeEfFgG***, se aplica
    a argumento ***double\****; en conversiones ***cs\[*** se aplica a
    argumento ***wchar\_t\****.
-   ***ll*** - la conversión se aplica a un argumento ***long long
    int\**** o ***unsigned long long int\****.
-   ***j*** - la conversión se aplica a un argumento ***intmax\_t\**** o
    ***uintmax\_t\****.
-   ***z*** - la conversión se aplica a un argumento ***size\_t\**** o
    apuntador al correspondiente signed type.
-   ***t*** - la conversión se aplica a un argumento ***ptrdiff\_t\****
    o apuntador al correspondiente unsigned type.
-   ***L*** - la conversién ***aAeEfFgG*** se aplica a un argurnento
    ***long double\****.

Conversion specifiers:

-   ***d*** - matches un entero decimal con signo opcional, con el mismo
    formato que esperaría ***strtol()*** (*stdlib.h*) con \'base\' 10.
    Argumento pointer a un entero con signo.
-   ***i*** - matches un entero con signo opcional, con el mismo formato
    que esperaría ***strtol()*** con \'base\' 0. Argumento pointer a un
    entero con signo.
-   ***o*** - matches un entero octal con signo opcional, con el mismo
    formato que esperaría ***strtoul()*** (*stdlib.h*) con \'base\' 8.
    Argumento pointer a un entero sin signo.
-   ***u*** - matches un entero decimal con signo opcional, con el mismo
    formato que esperaría ***strtoul()*** con \'base\' 10. Argurnento
    pointer a un entero sin signo.
-   ***x*** - matches un entero hexadecimal con signo opcional, con el
    mismo formato que esperaría ***strtoul()*** con \'base\' 16.
    Argumento pointer a un entero sin signo.
-   ***a***, ***e***, ***f***, ***g*** - matches un ﬂoating-point con
    signo opcional, infinito o NaN con el mismo formato que esperaría
    ***strtod()***. Argumento pointer a un ﬂoating.
-   ***c*** - matches el número de caracteres especificado por width (si
    no se especifica es 1). Si no hay length speciﬁer ***l***, argumento
    será ***char\**** y será capaz de almacenar la secuencia (no se
    añade carácter nulo). Si sí existe ***l***, argumento sera
    ***wchar\_t\**** y será capaz de almacenar la secuencia de
    caracteres multibyte que empezará y terminará en el initial shift
    state (no se añade carácter nulo); la conversión de multibyte a
    ***wchar\_t*** se hace según ***mbrtowc()***, con ***mbstate\_t***
    inicializado a 0 antes de convertir el primer carácter.
-   ***s*** - es igual que la conversión ***c*** excepto que solo
    matches una secuencia de caracteres diferentes de espacios. Otra
    diferencia es que sí inserta carácter nulo al final.
-   ***\[*** - es igual que la conversión ***c*** excepto que solo
    matches una secuencia de caracteres esperados (el scanset). Al igual
    que en la conversión ***s***, inserta carácter nulo al final. La
    scanlist se define escribiendo los caracteres de la misma justo
    después del \'***\[***\', y finalizando con \'***\]***\'. Si el
    primer carácter después del corchete inicial es \'***\^***\', el
    scanset estará formado por todos los caracteres que no están en la
    scanlist. Si queremos incluir el carácter \'***\]***\' en la
    scanlist, debe ser el primero de la misma (después de \'***\[***\' o
    después del \'***\[\^***\' inicial). El carácter \'***-***\' no
    tiene ningún significado especial si es el primero o último de la
    lista; de lo contrario puede tener una función
    implementation-defined.
-   ***p*** - matches una dirección de memoria con un formato
    implementation-defined y lo guarda en el argumento, que es un
    apuntador a ***void\**** (es decir, un ***void\*\****). El formato
    es el mismo que produce ***fprintf*** en una conversión ***%p***.
-   ***n*** - argumento apuntador a un entero con signo. Almacena en la
    variable el número de caracteres que se han leído del stream hasta
    el momento. No se asigna el valor a ninguna variable. No incrementa
    el número de conversiones realizadas en cuanto al return value.

\'***%%***\' matches un solo caracter \'***%***\'. Los especificadores
de conversión ***AEFGX*** se comportan igual que, respectivamente,
***aefgx***.

***fscanf()*** devuelve el valor del macro ***EOF*** si hay input
failure antes de que se produzca la primera conversion, de lo contrario
devuelve el número de asignaciones realizadas.

***int printf(const char \* restrict format,\...)*** es equivalente a
***fprintf()***, pero enviando la salida a ***stdout***.

***int scanf(const char \* restrict format,\...***) es equivalente a
***fscanf()*** pero lee de ***stdin***.

***int snprintf(char \* restrict s, size\_t n, const char \* restrict
format,\...)*** es equivalente a ***fprintf()***, pero escribiendo en el
array \'s\' hasta \'n\'-1 caracteres; al final escribe un carácter nulo.
Si \'n\' es 0, no se escribe nada (y \'s\' puede ser ***NULL***).
Devuelve la cantidad de caracteres que se habrían escrito si \'n\'
hubiera tenido el tamaño necesario (sin contar el null character final),
o un número negativo si hay encoding error.

***int sprintf(char \* restrict s, const char \* restrict
format,\...)*** es equivalente a ***snprintf()***, pero sin especificar
tamaño del array \'s\'.

***int sscanf(const char \* restrict s, const char \* restrict
format,\...)*** es equivalente a ***fscanf()*** pero lee del string
\'s\'. Llegar al final del string es equivalente al end-of-file del
stream en ***fscanf()***.

***int vfprintf(FILE \* restrict stream, const char \* restrict format,
va\_list arg)*** es equivalente a ***fprintf()***, pero se cambia la
lista variable de parámetros por \'arg\', que deberemos haber
inicializado con la macro ***va\_start***. ***vfprintf()*** no invoca
***va\_end***. Necesita el header *stdarg.h*.

***int vfscanf(FILE \* restrict stream, const char \* restrict format,
va\_list arg)*** es equivalente a ***fscanf()***, pero se cambia la
lista variable de parámetros por \'arg\', que deberemos haber
inicializado con la macro ***va\_start***. ***vfscanf()*** no invoca
***va\_end***. Necesita el header *stdarg.h*.

***int vprintf(const char \* restrict format, va\_list arg)*** es
equivalente a ***printf()***, pero se cambia la lista variable de
parámetros por \'arg\', que deberernos haber inicializado con la macro
***va\_start***. ***vprintf()*** no invoca ***va\_end***. Necesita el
header *stdarg.h*.

***int vscanf(const char \* restrict format, va\_list arg)*** es
equivalente a ***scanf()***, pero se cambia la lista variable de
parámetros por \'arg\', que deberemos haber inicializado con la macro
***va\_start***. ***vscanf()*** no invoca ***va\_end***. Necesita el
header *stdarg.h*.

***int vsnprintf(char \* restrict s, size\_t n, const char \* restrict
format, va\_list arg)*** es equivalente a ***snprintf()***, pero se
cambia la lista variable de parámetros por \'arg\', que deberemos haber
inicializado con la macro va\_start. ***vsnprintf()*** no invoca
***va\_end***. Necesita el header *stdarg.h*.

***int vsprintf(char \* restrict s, const char \* restrict format,
va\_list arg)*** es equivalente a ***sprintf()***, pero se cambia la
lista variable de parámetros por \'arg\', que deberemos haber
inicializado con la macro ***va\_start***. ***vsprintf()*** no invoca
***va\_end***. Necesita el header *stdarg.h*.

***int vsscanf(const char \* restrict s, const char \* restrict format,
va\_list arg)*** es equivalente a ***sscanf()***, pero se cambia la
lista variable de parámetros por \'arg\', que deberemos haber
inicializado con la macro ***va\_start***. ***sscanf()*** no invoca
***va\_end***. Necesita el header *stdarg.h*.

### []{#anchor-7}7.21.7 Character input/output functions

***int fgetc(FILE \*stream)*** lee el siguiente carácter del stream como
***unsigned char***, y lo devuelve convertido a ***int***, siempre que
haya carácter disponible y el indicador de end-of-file para el stream no
esté activado. On read error (sets stream error indicator), end-of-file
activado, o llegamos al end of ﬁle, devuelve ***EOF***.

***char \*fgets(char \* restrict s, int n, FILE \* restrict stream)***
lee un maximo de \'n\'-1 caracteres de \'stream\' y los escribe en
\'s\'. No se escriben más caracteres después de un end of file ni
después de un newline (que se copia). Al final coloca un null character.
On success devuelve \'s\'; si llega el end of file y no se ha leído
ningún carácter, \'s\' queda unchanged y devuelve ***NULL***. On read
error, devuelve ***NULL***.

***int fputc(int c, FILE \*stream)*** escribe el caracter \'c\'
convertido a ***unsigned char*** al \'stream\'. On success devuelve el
carácter y avanza el puntero del stream. On error, sets el error
indicator del stream y devuelve ***EOF***.

***int fputs(const char \* restrict s, FILE \* restrict stream)***
escribe el string \'s\' al \'stream\'. El null terminator no se escribe.
Devuelve un número no negativo on success; ***EOF*** si no.

***int getc(FILE \*stream)*** es igual que ***fgetc()***, pero
implementado como macro. Puede evaluar \'stream\' más de una vez, con lo
que es mejor que el argumento no sea una expresión con side effects.

***int getchar(void)*** es como ***getc()***, con ***stdin*** como
stream.

***int putc(int c, FILE \*stream)*** es igual que ***fputc()***, pero
implementado como macro. Puede evaluar \'stream\' más de una vez, con lo
que es mejor que el argumento no sea una expresión con side effects.

***int putchar(int c)*** es como ***putc()***, con ***stdout*** como
stream.

***int puts(const char \*s)*** escribe el string \'s\' + un newline en
***stdout***. No escribe null terminator. Devuelve non-negative on
success; ***EOF*** si no.

***int ungetc(int c, FILE \*stream)*** pushes el carácter \'c\'
convertido a ***unsigned char*** al stream. Los caracteres pushed leidos
del stream se recuperan en orden inverso a su push (como una pila). Una
llamada exitosa a una función de reposicionamiento del puntero del
stream descarta los últimos pushed-back characters sin leer. Se
garantiza 1 pushback al stream, pero si se hacen demasiados
***ungetc()*** seguidos sin una sola operacion de lectura o
reposicionamiento, puede fallar. Si intentarnos unread el carácter
***EOF***, la operación falla y no hace nada. Al descartar o leer todos
los caracteres pushed en un stream, el indicador de posicion del stream
recupera el mismo valor que tenía antes de unread el primero de esos
caracteres. On success, el end-of-file indicator del stream se borra.
Devuelve el carácter pushed back o ***EOF*** on error.

### []{#anchor-8}7.21.8 Direct input/output functions

***size\_t fread(void \* restrict ptr, size\_t size, size\_t nmemb, FILE
\* restrict stream)*** lee de \'stream\' hasta \'nmemb\' elementos (o
hasta end of file) de \'size\' bytes y los coloca byte a byte
(***unsigned char***) en \'ptr\'. El indicador de posición del stream es
incrementado en el número de caracteres leídos correctamente. Si hay
error o se lee un elemento parcialmente, el valor del indicador es
indeterminado. La función devuelve el número de elementos leídos, que
será menor a \'nmemb\' si hay end of file o error de lectura. Si
\'nmemb\' o \'size\' son 0, la función no hace absolutamente nada y
devuelve 0.

***size\_t fwrite(const void \* restrict ptr, size\_t size, size\_t
nmemb, FILE \* restrict stream)*** escribe hasta \'nmemb\' elementos de
\'size\' bytes, byte a byte (***unsigned char***), del array apuntado
por \'ptr\' hacia \'stream\'. El indicador de posición del stream avanza
tantos bytes como se hayan escrito successfully; si hay error, su valor
es indeterminado. La función devuelve el número de elementos escritos,
que será menor a \'nmemb\' si hay error de escrituta. Si \'nmemb\' o
\'size\' son 0, la función no hace absolutamente nada y devuelve 0.

### []{#anchor-9}7.21.9 File positioning functions

***int fgetpos(FILE \* restrict stream, fpos\_t \* restrict pos)*** lee
el position indicator del stream y los valores de su parse state (si lo
hay) y lo almacena en el objeto apuntado por \'pos\'. On success,
retorna 0; on failure, returns nonzero and sets ***errno*** (*errno.h*)
to an implementation-specific positive value.

***int fseek(FILE \*stream, long int offset, int whence)*** coloca el
indicador de posición del stream binario en la posicion \'offset\'
(caracteres), en relación a \'whence\', que puede ser ***SEEK\_SET***
(principio del stream), ***SEEK\_CUR*** (posición actual) o
***SEEK\_END*** (ﬁnal del stream). Si hay error, sets el indicador de
error del stream. En text streams el funcionamiento se debería limitar a
un offset de 0, o posiciones devueltas por ***ftell()***, y usando
siempre ***SEEK\_SET*** para asegurarnos de un funcionamiento correcto.
Una llamada exitosa a ***fseek()*** descarta los últimos caracteres
***ungetc()***, borra el indicador end-of-file del stream y establece la
nueva posición. Solo devuelve nonzero on fail.

***int fsetpos(FILE \*stream, const fpos\_t \*pos)*** sets el indicador
de posición del stream, así como los valores de su parse state
(***mbstate\_t*** object, si lo hay), según especifique el objeto
apuntado por \'pos\' (obtenido anteriormente con ***fgetpos()*** sobre
el mismo fichero). Si es successful, descarta los últimos caracteres
***ungetc()***, borra el indicador end-of-file del stream y establece la
nueva posición. On success, retorna 0; on failure, returns nonzero and
sets ***errno*** to an implementation-specific positive value.

***long int ftell(FILE \*stream)*** lee el valor del indicador de
posición del stream y devuelve ese dato. Para binary files, es el número
de caracteres desde el principio del fichero. Para text files, el valor
es unspecified, pero puede usarse con ***fseek()*** para restaurar esa
posición. Si hay error, sets ***errno*** to an implementation-specific
positive value, y devuelve -1L.

***void rewind(FILE \*stream)*** equivale a ***(void)fseek(stream, 0L,
SEEK\_SET)***, con la diferencia de que también borra el indicador de
error del stream. No devuelve valor.

### []{#anchor-10}7.21.10 Error-handling functions

***void clearerr(FILE \*stream)*** borra los indicadores de error y
end-of-file del stream. No devuelve valor.

***int feof(FILE \*stream)*** returns nonzero if and only if el
indicador de end-of-file del stream is set.

***int ferror(FILE \*stream)*** returns nonzero if and only if el
indicador de error del stream is set.

***void perror(const char \*s)*** escribe en ***stderr*** la secuencia
de caracteres siguiente: primero el string \'s\', seguido por dos puntos
(***:***) y un espacio; a continuacon, el mensaje de error, como el que
devolvería ***strerror()*** (*string.h*) con argumento ***errno*** y un
newline. Si \'s\' es ***NULL*** o apunta a una cadena vacía, solo se
escribe el mensaje de error con el newline. No devuelve valor.
