# C - estándar

Resumen del estándar *C17* (*ISO/IEC 9899:2018*), que también es conocido como *C18*. Este documento solo cubre la definición del lenguaje, no la biblioteca estándar. Los títulos se han dejado en inglés para que los apartados sean fácilmente localizables en el documento original.

Solo se ha resumido el contenido lo suficientemente relevante (a juicio del autor), con lo que la numeración de títulos no es exhaustiva.

Algunas explicaciones han sido ampliadas para dar mayor claridad y facilitar la comprensión de conceptos.

## 3. Terms, definitions, and symbols

**Alineación** (*alignment*) es un requisito que debe cumplir un tipo específico de objeto: debe residir en una dirección de memoria que sea múltiplo de un número de *bytes* específico.

**Argumento** (*argument*, *actual argument*) es una expresión usada en la llamada a una función o macro.

**Carácter *multibyte*** es una secuencia de **uno o más** *bytes* capaz de representar cualquier miembro del conjunto de caracteres (fuente o de ejecución) extendido.

**Carácter *wide*** es un valor representable mediante el tipo `wchar_t`, capaz de almacenar cualquier carácter del conjunto de caracteres de ejecución extendido.

**Objeto** es una región de datos en el entorno de ejecución que puede contener valores.

**Parámetro** (*parameter*, *formal parameter*) es un objeto que forma parte de la declaración o definición de una función (que toma valor a la entrada a la misma), o de la lista entre paréntesis tras una macro.

## 4. Conformance

Una implementación de este estándar debe cumplir todo lo especificado en este documento. Esto es cierto para los entornos *hosted*, normalmente implementaciones del lenguaje en un sistema operativo, con soporte para todas las bibliotecas. Sin embargo existen las implementaciones *freestanding*, que solo están obligadas a dar soporte a un pequeño subconjunto de las bibliotecas del estándar. Estos entornos son frecuentes en sistemas incrustados (*embedded systems*).

Cada implementación debe documentar las posibles extensiones que ofrece (en el lenguaje o la biblioteca), que nunca deben alterar el comportamiento de un programa que cumpla con el estándar. También deberá documentar todas las características específicas, incluyendo las del *locale* específico, si existen.

## 5. Environment

Existen 2 entornos: entorno de traducción (*translation environment*) y entorno de ejecución (*execution environment*).

### 5.1 Conceptual models

#### 5.1.1 Translation environment

El texto del programa reside en los archivos fuente. Una vez se ha ejecutado el preprocesado de un archivo fuente (ver fases más abajo), se obtiene la llamada unidad de traducción (*translation unit*). Cada una de ellas dará lugar a un archivo de código objeto (*object file*).

##### 5.1.1.2 Translation phases

Las fases de la traducción sobre un archivo fuente son (en este orden):

1. Los caracteres *multibyte* de los archivos fuente se mapean al juego de caracteres fuente (*source character set*) de una forma *implementation-defined* (la codificación del juego de caracteres en memoria interna, sin embargo, es irrelevante para el desarollador). La codificación interna es independiente de su codificación en el archivo fuente: *UTF-8*, *ISO-8859*, etc. Los archivos fuente solo pueden contener caracteres pertenecientes al *source character set* de la implementación. Se sustituyen los *trigraphs*.
2. Las secuencias de barra invertida (***\\***) + *newline* se eliminan, juntando así varias líneas lógicas. El archivo fuente debe terminar en *newline* (no precedido de ***\\***).
3. Los comentarios se sustituyen por *un espacio*. Se descompone el archivo fuente en *preprocessing tokens* y espacios (agrupándose en un solo espacio o no). Se mantienen los *newlines*.
4. Se ejecutan las directivas del preprocesador, se expanden macros y se ejecuta el operador `_Pragma`. Se insertan los archivos `#include` y se aplican todos estos pasos desde el principio en cada uno de estos, recursivamente.
5. **Dentro de las constantes carácter y literales *string***, cada carácter del juego fuente y cada secuencia escape se mapean al carácter correspondiente del juego de caracteres de ejecución. La codificación del *execution character set* sí puede ser relevante para el desarrollador al acceder a las constantes carácter *multibyte* y literales *string multibyte*.
6. Se concatenan literales *string* adyacentes.
7. Los *preprocessing tokens* se convierten en *tokens*, y se desecha todo el espacio en blanco (incluyendo *newlines*, espacios,...). Esto forma la *translation unit*.
8. Se resuelven todas las referencias externas (bibliotecas, funciones externas, etc.).

En cuanto a romper una línea lógica en *C* en varias línea físicas, la línea se puede romper en cualquier punto, incluso unir varias líneas lógicas una detrás de otra, ya que el espacio en blanco queda ignorado, y un *newline* no es distinto a un espacio. Sin embargo, no se puede romper una línea en mitad de un identificador, o en mitad de una constante, o de un literal *string*. Sí se puede sin embargo, cortar en mitad de identificador, constante o literal *string*, si inmediatamente después del trozo de elemento incluimos barra invertida (***\\***) seguida de *newline*, ya que estos dos caracteres será eliminados en una fase muy temprana del preproceso. La siguiente línea debe proseguir ese elemento desde el primer carácter.

#### 5.1.2 Execution environments

##### 5.1.2.1 Freestanding environment

En un entorno *freestanding* es la implementación la que decide qué función será la primera en ejecutarse cuando arranca el programa.

##### 5.1.2.2 Hosted environment

La ejecución se inicia en la función `main()`:

```c
int main(void) { /* ... */ }
```

o

```c
int main(int argc, char *argv[]) { /* ... */ }
```

Vale cualquier notación equivalente, como escribir `**argv`.

***argv*** es un *array* de apuntadores a *strings*, el primero de los cuales (`argv[0]`) es el nombre del programa y el resto son los parámetros (`argv[1]` a `argv[argc-1]`) pasados al programa desde el entorno *hosted*. Si el nombre del programa no está disponible en este entorno, el primer *string* será un *string* vacío (`argv[0][0]` es un carácter nulo). ***argc*** es el número de parámetros que se han pasado al programa, incluyendo el nombre del programa como primero de ellos.

El valor retornado por esta función (compatible con `int`) es el valor que retornará la ejecución del programa. Si se llega al ***}*** de `main()`, retorna 0.

##### 5.1.2.3 Program execution

Efectos secundarios (*side effects*) son acceder a un objeto volátil, modificar un objeto, modificar un archivo, o llamar a una función que haga alguna de estas cosas. Por ejemplo, una función (o expresión) sin *side effects*, solo es útil por el valor retornado.

Una evaluación está secuenciada después de otra dentro del mismo hilo (*thread*) si hay un *sequence point* entre ellas. Si no, el orden relativo no está especificado. Si una evaluación ***A*** está secuenciada antes que una evaluación ***B***, entonces todo cálculo de resultado y todo *side effect* de ***A*** ocurre antes que todo cálculo de resultado y todo *side effect* de ***B*** (para ver lista de *sequence points*, ver el **anexo C**).

La implementación no está obligada a evaluar una parte de una expresión si deduce que su valor no será usado (cortocircuito).

##### 5.1.2.4 Multi-threaded executions and data races

En una implementación *hosted*, el programa puede ejecutarse en varios hilos (*threads*) concurrentes. En una implementación *freestanding* esta capacidad es opcional.

Se dice que dos expresiones están en conflicto cuando una de ellas modifica una posición de memoria y la otra lee o modifica esa misma posición.

En la biblioteca estándar existen mecanismos que evitan estos conflictos en caso de operaciones concurrentes: operaciones atómicas (***stdatomic.h***) y código *mutex* (incluido en ***threads.h***).

### 5.2 Environmental considerations

#### 5.2.1 Character sets

El conjunto de caracteres fuente (*source character set*) es el conjunto de caracteres que admite la implementación, y que están escritos en los archivos de código fuente, en los que están codificados según cada archivo define. A partir de estos archivos fuente, el compilador crea la correspondiente *translation unit*, compuesta por *tokens*, entre los que encontramos los diversos literales *string* y constantes carácter. Esto ya corresponde al *execution character set*, y su codificación la decide el propio compilador, aunque se puede forzar una codificación concreta de este juego de ejecución estableciendo las opciones pertinentes del compilador; incluso se puede forzar la codificación *UTF-8* mediante los literales *string* de tipo `u8`, como se verá más adelante.

Cada conjunto de caracteres (fuente y ejecución) está compuesto del conjunto básico (del que trataremos ahora) y un conjunto (opcional) compuesto por caracteres específicos de un *locale* concreto, soportado por una implementación concreta. La suma de ambos subconjuntos se denomina conjunto de caracteres extendido (*extended source character set* y *extended execution character set*).

Conjunto básico de caracteres fuente:

- ***A***-***Z***, ***a***-***z***, ***0***-***9*** (63 miembros).
- ***!"#%&'()\*+,-./:;\<=>?[\\]^\_{|}~*** (29 miembros).
- Espacio, tabulador horizontal, tabulador vertical y *form feed*.
- Caracteres necesarios para marcar el fin de línea.

Conjunto básico de caracteres de ejecución:

- Los mismos que el conjunto básico de caracteres fuente.
- Carácter nulo (todo ceros), alerta, retroceso (*backspace*), retorno de carro (*carriage return*) y *newline*.

Dentro de literales *string* y constantes carácter, los miembros del conjunto de ejecución se representan en el archivo fuente mediante los miembros del conjunto de caracteres fuente, y por secuencias de escape.

Los miembros de los conjuntos de caracteres básicos se representan por un solo *byte* por carácter. Algunos caracteres del juego extendido que no pertenecen al básico podrían representarse, tras ser codificados (conjunto de caracteres de ejecución) también mediante 1 *byte* (es decir, un `char`, si así lo decide la implementación), pero lo normal es que utilicen más de uno, ya sea mediante *multibytes* (secuencia de `char`) o caracteres *wide* (`wchar_t`, `char16_t` o `char32_t`). Existe una correspondencia 1 a 1 entre los miembros fuente y los de ejecución.

##### 5.2.1.1 Trigraph sequences

Se trata de la primera de las sustituciones del preprocesador. Se trata de secuencias de tres caracteres que se sustituyen por el carácter indicado:

- `??=` ==>	***#***
- `??(` ==>	***[***
- `??/` ==>	***\\***
- `??)` ==>	***]***
- `??’` ==>	***^***
- `??<` ==>	***{***
- `??!` ==>	***|***
- `??>` ==>	***}***
- `??-` ==>	***~***

##### 5.2.1.2 Multibyte characters

Un carácter *multibyte* del *source character set* se codifica en el entorno de ejecución en un carácter *multibyte* (en un *string* de `char`, por ejemplo), o en un carácter *wide* (en un *string* de `wchar_t`, por ejemplo). La codificación (mapeo) entre estos juegos de caracteres (fuente y ejecución) depende de la implementación.

El tipo `char` es suficiente para almacenar cualquier valor del conjunto de caracteres de ejecución básico. El tipo `wchar_t` es suficiente para almacenar cualquier valor del conjunto de caracteres de ejecución extendido (que incluye al básico).

La existencia de caracteres más allá del conjunto básico es específica del *locale* concreto, o sea, de la implementación.

La codificación *multibyte* de caracteres, tanto para el conjunto fuente como el de ejecución, será tal que al descodificar (de *bytes* a caracteres), existirá un estado *shift*. En su estado inicial, cada *byte* se corresponderá a un carácter del conjunto **básico**, y no se alterará dicho *shift state* mientras se vayan encontrando caracteres de dicho conjunto. El *byte* con valor 0 no puede formar parte de ninguna codificación *multibyte*.

Un identificador, comentario, literal *string*, constante carácter o nombre de cabecera debe empezar y terminar en el estado *shift* inicial y debe formar una secuencia válida de caracteres *multibyte*.

#### 5.2.2 Character display semantics

Para indicar caracteres no gráficos (del conjunto de caracteres de ejecución) en el archivo fuente, disponemos de las siguientes  secuencias de escape: `\a` (*alert*), `\b` (*backspace*), `\f` (*formfeed*), `\n` (*newline*), `\r` (*carriage return*), `\t` (*tabulación horizontal*), `\v` (*tabulación vertical*).

## 6. Language

### 6.2 Concepts

#### 6.2.1 Scopes of identifiers

Un identificador puede denotar un objeto; función; etiqueta o miembro de una estructura o unión; etiqueta (*tag*) o constante de una enumeración; nombre `typedef`; o etiqueta (*label*), sin olvidar los nombres de macro o sus parámetros, aunque estos desaparecen tras el preprocesado.

Un identificador es visible (accesible) dentro de la región llamada ámbito (*scope*) de dicho identificador. Si un identificador designa entidades distintas, es que estas tienen diferente *scope* o pertenecen a *name spaces* distintos. Hay 4 tipos de *scope*: función, archivo, bloque, y prototipo de función (declaración de la función).

- Solo las etiquetas (*labels*) tienen ***function scope***: pueden usarse desde cualquier punto de la función donde son definidas. Se declaran simplemente mediante su nombre seguido por dos puntos (***:***).
- El resto de identificadores tienen *scope* desde el punto donde se declaran hasta el final de la región donde se declaran. Tienen ***file scope*** cuando se declaran fuera de todo bloque o función. Entonces, su *scope* va desde del punto donde se declara, hasta el final de la *translation unit*.
- Tienen ***block scope*** cuando se declaran dentro de un bloque (***{}***) o en la lista de parámetros dentro de la **definición** de una función. Su ámbito va desde el punto donde se declaran hasta el final del bloque o función.
- Tienen ***function prototype scope*** cuando se declara el identificador en la lista de parámetros dentro de la **declaración** (prototipo) de una función. Su *scope* termina al terminar la declaración de la función.

Si un *scope* está anidado dentro de otro y en el *scope* interior se declara una entidad con el mismo nombre que en el exterior, el identificador del *inner scope* pasa a referenciar a la entidad interior, estando la entidad exterior oculta (enmascarada por la interior) desde el punto de su declaración en el bloque interior hasta el final de este.

La etiqueta de una estructura, unión o enumeración tiene *scope* desde el punto donde aparece; lo mismo pasa con las constantes de una enumeración. Sin embargo, para el **resto de elementos**, su *scope* empieza en el punto en que se ha **completado** su **declaración**.

#### 6.2.2 Linkages of identifiers

Un identificador declarado en distintos *scopes*, o en el mismo *scope* más de una vez puede referirse al mismo objeto o función mediante el *linkage*, el cual puede ser de tres tipos: externo, interno y ninguno. El objeto se debe definir una sola vez dentro de su *linkage*.

- Cada declaración de un identificador con ***linkage* externo** dentro de cualquier lugar del programa (**conjunto de *translation units* y bibliotecas** que lo forman), se refiere a la misma entidad o función.
- Cada declaración de un identificador con ***linkage* interno** dentro de cualquier lugar de la ***translation unit***, se refiere a la misma entidad o función.
- Cada declaración de un identificador **sin *linkage*** se refiere a una entidad o función distinta, lo cual significa que cada declaración será a su vez una definición de esa entidad o función. Por tanto, no puede haber dos declaraciones del mismo identificador sin *linkage* en el mismo *scope*.

Los identificadores (de objeto o función) con *file scope* declarados con el *storage-class specifier* `static`, tienen ***linkage* interno**.

Un identificador declarado como `extern` en un *scope* donde hay una declaración visible del mismo identificador (anteriormente en el mismo *scope* o en un *enclosing scope*), y esa declaración indica *linkage* interno o externo, la declaración tardía adquiere **ese mismo *linkage***. En cambio, si no hay otra declaración visible o la que hay no tiene indicación de *linkage*, la declaración tendrá ***linkage* externo**.

Si la declaración de un identificador de **función** no tiene *storage-class specifier*, su *linkage* se determina **como si tuviera el *storage-class specifier*** `extern`. Si la declaración de un identificador de **objeto con *file scope*** no tiene *storage-class specifier*, tiene ***linkage* externo** directamente.

Los siguiente elementos **no tienen *linkage***: un identificador de algo que no sea un objeto o función; un identificador de parámetro a función; un identificador con *block scope* sin el *storage-class specifier* `extern`.

Veamos una tabla resumen:

| *Scope:* | *File*   | *File*      | *File*      | *Block*     | *Block*       |
| :------- | :------- | :---------- | :---------- | :---------- | :------------ |
|          | `static` | `extern`    | ∅           | `extern`    | ∅             |
| Función  | Interno  | Externo (?) | Externo (?) | Externo (?) | Externo (?)   |
| Objeto   | Interno  | Externo (?) | Externo     | Externo (?) | Sin *linkage* |

**Externo (?)** significa que es *linkage* externo siempre y cuando no haya una declaración visible anterior con *linkage* interno, en cuyo caso, será interno.

Dentro de *block scope*, `static` no define *linkage*, sino *storage duration*, con lo que no se puede usar en la declaración con *block scope* de una función. Como veremos, no se pueden definir funciones en *block scope*, pero sí declararlas.

Si un identificador queda definido dentro del mismo *scope* con *linkage* tanto interno como externo, el comportamiento es indefinido.

#### 6.2.3 Name spaces of identifiers

Hay 4 espacios distintos de nombres de identificadores, que quedan desambiguados por la sintaxis:

- Etiquetas: su declaración y uso tienen su propia sintaxis.
- Las etiquetas de estructuras, uniones y enumeraciones se identifican con las palabras clave `struct`, `union` y `enum`.
- Los miembros de estructuras y uniones, accedidos mediante los operadores `.` o `->`.
- Los identificadores ordinarios, declarados normalmente, o como constantes de enumeraciones.

#### 6.2.4 Storage durations of objects

La duración del almacenamiento marca el tiempo de vida de un objeto. Hay 4 duraciones distintas: estática (*static*), hilo (*thread*), automática (*automatic*), y dinámica (*allocated*). El tiempo de vida es el tiempo en el cual se reserva espacio en memoria para el objeto.

*Static storage duration*: el tiempo de vida es el tiempo de ejecución del programa completo, con inicialización antes del inicio del mismo.

Tiene duración estática cualquier objeto con *linkage* interno o externo, o con el *storage-class specifier* `static`, siempre y cuando no tenga el *storage-class specifier* `_Thread_local`.

*Thread storage duration*: el tiempo de vida es el tiempo de ejecución del hilo al que pertenece el objeto, con inicialización al inicio del hilo. Hay un objeto distinto por hilo, y cualquier acceso al objeto se realiza sobre el objeto perteneciente al hilo donde se evalúa la expresión.

Tienen duración de hilo los objetos con el *storage-class specifier* `_Thread_local`.

*Automatic storage duration*: el tiempo de vida es desde el momento en el que la ejecución entra en el bloque donde está definido el objeto (aunque todavía no sea visible, lo cual sucede a partir de su declaración), hasta la finalización definitiva de dicho bloque. Si se va entrando recursivamente al bloque, se crea una instancia del objeto cada vez. El objeto se inicializa al llegar a su declaración.

Sin embargo, si el objeto contiene un ***array* de longitud variable**, su tiempo de vida se inicia en el momento de su declaración, y no en el momento de entrada al bloque.

Tienen duración automática los objetos sin *linkage* y sin *storage-class specifier* `static`.

Existe un caso especial: una expresión de tipo estructura o unión que se hace a través de un *non-lvalue*, y que contiene (recursivamente) algún miembro de tipo *array*, es un objeto con *automatic storage duration* y su *lifetime* es **temporal** (*temporary*). Su tiempo de vida se inicia cuando la expresión es evaluada, y termina cuando la *full expression* (expresión que no forma parte de ninguna otra expresión) que la contiene termina. Veamos en un ejemplo cual es la utilidad de ello:

```c
#include <stdio.h>

struct sinar { int x; int y;};  // estructura sin array
struct conar { int p[2];};  // estructura con array

struct sinar fooA() {
    struct sinar r;
    printf("Dir. %p\n", (void *)&r.y);
    return r;
}

struct conar fooB() {
    struct conar r;
    printf("Dir. %p\n", (void *)r.p);
    return r;
}

int main() {
    printf("Dir. %p\n", (void *)&fooA().y);  // error
    printf("Dir. %p\n", (void *)fooB().p);
}
```

La función `fooA()` imprime la dirección de memoria del primer miembro de un objeto de tipo estructura ***sinar*** (que no contiene *arrays*), y retorna ese objeto. Por otro lado, `fooB()` imprime la dirección de memoria del primer elemento del *array* que contiene un objeto de tipo estructura ***conar***, y retorna ese objeto. Las dos funcionan correctamente.

En la función `main()` vamos a acceder a esos objetos mediante el valor retornado por estas funciones, es decir, mediante *non-lvalues*. Lo primero que vemos es que al intentar acceder a la dirección de memoria del miembro ***y*** de la estructura retornada por `fooA()`, mediante `&fooA().y`, obtenemos un error. Esto es debido a que el valor retornado por `fooA()` no es ningún objeto, sino que es un simple valor, sin *storage* alguno, una copia del valor retornado por la función. Como tal, no tiene una dirección de memoria física (dependiendo de la arquitectura, un valor de retorno puede estar en la pila, en un registro, etc.). Por lo tanto, no es legal hacer `&fooA()`.

Pero si la estructura contiene *arrays*, debido a que estos se tratan como apuntadores, necesitamos que tengan una ubicación en memoria, de lo contrario no podremos acceder a su contenido. En este caso, en cuanto a `fooB().p`, podemos comprobar que sí tiene una ubicación en memoria. Ello se debe a que el compilador ha creado un objeto de tiempo de vida temporal, gracias al cual podremos acceder a los elementos del *array*.

En la salida del programa podemos ver que la dirección de memoria del objeto original es distinta a la dirección de la copia temporal, lógicamente.

#### 6.2.5 Types

Existen los tipos de objeto y los tipos de funciones. En algún punto de la *translation unit*, un tipo puede ser **incompleto** cuando todavía no hay información sobre el tamaño de los objetos de ese tipo (p.e. mientras estamos definiendo una estructura, antes de llegar al ***}*** final).

El tipo `_Bool` es lo suficientemente grande para almacenar los valores 0 o 1.

El tipo `char` es lo suficientemente grande para almacenar cualquier carácter del *basic execution character set*, ninguno de los cuales podrá tener codificación negativa. Dependiendo de la implementación, puede almacenar otros caracteres y darles cualquier codificación.

Enteros: el tipo `int` tiene el tamaño adecuado a la arquitectura del entorno de ejecución. Otros enteros son `signed char`, que ocupa el mismo espacio que `char`, y los tipos `short int`, `long int` y `long long int`. La implementación puede definir tipos extendidos si lo desea. Todos ellos son enteros con signo.

Existen también los enteros sin signo, que son los mismos que los enteros con signo definidos, precedidos por `unsigned`, que ocupan el mismo tamaño en memoria y tienen los mismos requisitos de alineación que su correspondiente tipo con signo. `_Bool` es un tipo sin signo.

La representación de un valor positivo en un entero con signo es la misma que la de ese valor en el correspondiente tipo sin signo. Las operaciones aritméticas con enteros *unsigned* no produce *overflow*, ya que el número resultante se reduce módulo ***N***, donde ***N*** es el máximo número representable de ese tipo más uno.

Hay tres tipos de tipos representando números reales: `float`, `double` (doble precisión) y `long double` (todavía más precisión).

Números complejos (opcional): de forma análoga a los reales, existen los tipos `float _Complex`, `double _Complex` y `long double _Complex`. Cada tipo complejo tiene el mismo tamaño y requisitos de alineación que dos objetos del tipo real *floating* correspondiente (uno para la parte real y otro para la imaginaria).

Tipos de datos:

- ***Integer types:*** `char`, enteros con y sin signo y *enumerated types*.
- ***Real floating types:*** `ﬂoat`, `double` y `long double`.
- ***Complex types:*** `float _Complex`, `double _Complex` y `long double _Complex`.
- ***Basic types:*** `char`, enteros con y sin signo y tipos *real floating*.
- ***Character types:*** `char`, `signed char` y `unsigned char`. Tienen el mismo tamaño y representación, pero no son el mismo tipo (por tanto, `char` no es compatible con uno ni con otro).
- Cada enumeración representa un ***enumerated type*** distinto.
- ***Real types:*** *integer types* y *real floating types*.
- ***Floating types:*** *real floating types* y *complex types*.
- ***Arithmetic types*** (del dominio real o complejo):  *floating types* y *integer types*.
- ***Scalar types:*** *arithmetic types* y *pointer types*.
- ***Aggregate types:*** *arrays* y *estructuras*.
- Tipos derivados (*derived types*): *array* (de tipos completos siempre), estructuras, uniones, funciones, apuntadores y tipos atómicos (`_Atom`, opcional su implementación), porque usan como base un tipo `T` concreto, creando un **tipo derivado** (`T[]`, `*T`,...).

Booleanos, caracteres, enteros, reales y complejos son tipos completos. El tipo `void` es incompleto (no puede completarse nunca).

Un *array* de tamaño no especificado es un tipo incompleto hasta que, en una declaración posterior (si tiene *linkage* externo o interno) definimos su tamaño.

Un apuntador es un tipo completo.

Una estructura o unión de contenido desconocido son tipos incompletos hasta que se definen posteriormente a su primera aparición (en una declaración de una variable o función, por ejemplo) en el mismo *scope*. Mientras están siendo definidos siguen siendo tipos incompletos.

Todos los tipos vistos hasta ahora son *unqualified*. Cada tipo no cualificado tiene varias versiones cualificadas de su tipo, correspondientes a cualquier combinación de los *qualifiers* `const`, `volatile` y/o `restrict`. Un tipo derivado de un tipo cualificado no es *qualified* a no ser que lo cualifiquemos explícitamente. Los *qualified types* son tipos distintos que tienen los mismos requerimientos de tamaño, representación y alineación que los *unqualified types* correspondientes.

Hay un cuarto *qualifier*, `_Atom`, donde estos requerimientos no tienen por qué mantenerse. Si no nos referimos explícitamente al cualificador atómico, cuando hablemos de cualificadores se entienden los otros tres.

Un apuntador a `void` tiene los mismos requisitos de representación y alineación que un apuntador a un tipo carácter.

Apuntadores a tipos compatibles, cualificados o no, tendrán los mismos requisitos de representación y alineación. Lo mismo sucede con apuntadores a estructuras o apuntadores a uniones. Los *qualifiers* no cambian los requisitos de alineación y representación.

#### 6.2.6 Representations of types

El estándar define algunos requisitos en la representación de los diferentes tipos de objetos, dejando el resto a decisión de la implementación.

#### 6.2.7 Compatible type and composite type

Dos tipos son compatibles si son el mismo tipo, pero también existen otras reglas que amplían la definición de tipo compatible, así que en muchas ocasiones, dos tipos no necesitan ser idénticos para ser compatibles.

Para todas las declaraciones que se refieran al mismo objeto o función, es suficiente que estén declarados con tipos compatibles.

Dos declaraciones de tipo estructura, unión o enumeración declaradas en distintas *translation units* tendrán tipo compatible si están declaradas con el mismo nombre (*tag*), y existe una correspondencia uno a uno entre sus miembros, de tal forma que cada uno esté declarado con el mismo nombre y con un tipo compatible. Si un miembro tiene especificador de alineación, el correspondiente debe tener un especificador equivalente.

Si se trata de estructuras, los miembros de cada declaración estarán declarados en el mismo orden. En estructuras y uniones, los *bit fields* tendrán el mismo tamaño. Para enumeraciones, las constantes correspondientes tendrán los mismos valores.

Se verán más adelante reglas adicionales para determinar compatibilidad, al describir otros tipos, en las secciones 6.7.2, 6.7.3 y 6.7.6.

Cuando un objeto o función ha sido declarado usando distintos tipos compatibles, el tipo resultante, llamado tipo compuesto (*composite type*), será una combinación de dichos tipos.

El *composite type* se construye así a partir de dos tipos compatibles:

Si ambos tipos son *arrays*:

- Si uno de ellos tiene una longitud constante especificada, ese será el tamaño del tipo compuesto.
- En caso contrario, si uno de ellos en un *array* de longitud variable con tamaño especificado, el tipo compuesto será un *array* de longitud variable con ese tamaño.
- En caso contrario, si uno de ellos es un *array* de longitud variable con tamaño no especificado, el tipo compuesto será un *array* de longitud variable de tamaño no especificado.
- En caso contrario, el tipo compuesto es un *array* de tamaño no especificado.

El tipo del elemento del tipo compuesto es el tipo compuesto de los elementos de ambos *arrays* (que deben ser de tipos compatibles).

Si los tipos son función, y solo uno de los dos tiene lista de parámetros, el tipo compuesto será de tipo función con esa lista de parámetros.

Si los tipos son función y ambos tienen lista de parámetros, el tipo compuesto será un tipo función con una lista de parámetros en la que cada parámetro será del tipo compuesto correspondiente a cada uno de los parámetros de las listas iniciales (que deben tener tipos compatibles).

Por ejemplo:

```c
int f(int (*)(), double (*)[3]);
int f(int (*)(char *), double (*)[]);
```

Son dos declaraciones de tipos compatibles, y el *composite type* resultante es:

```c
int f(int (*)(char *), double (*)[3]);
```

#### 6.2.8 Alignment of objects

Los tipos completos tienen un requisito de alineación concreto, que es un entero que indica en qué direcciones de memoria se puede almacenar un dato, que deberá ser un múltiplo de ese entero (que es siempre una potencia de 2). Se puede imponer un *alignment* más estricto (número mayor) que el que tocaría, mediante `_Alignas`.

Para saber el requisito de alineación de un tipo se hace con `_Alignof`, pasándole como argumento el nombre de un tipo u objeto. El tipo con requisitos de alineación menos estricto es `char` (y familia).

Si un tipo cumple con un requisito de alineación concreto, también cumple con un requisito más débil (menos estricto, con un número menor).

### 6.3 Conversions

Normalmente la conversión a un tipo compatible no cambia el valor ni la representación interna.

#### 6.3.1 Arithmetic operands

##### 6.3.1.2 Boolean type

Si un escalar se convierte a `_Bool`, será ***0*** si era ***0***, o ***1*** en otro caso.

##### 6.3.1.3 Signed and unsigned integers

Si un valor entero se convierte a cualquier tipo de entero (excepto `_Bool`), el valor se mantiene si el tipo puede representarlo. En caso contrario, si el nuevo tipo es *unsigned*, el valor será el resultado de sumar (si es un valor negativo) o restar (si es positivo) sucesivamente un número ***N***, hasta dar con un valor representable. ***N*** es el valor máximo representable del tipo en cuestión, más uno. En caso de tipo *signed*, se deja a criterio de la implementación.

##### 6.3.1.4 Real floating and integer

De *real floating* a entero (no `_Bool`), se descarta la parte fraccionaria. Si la parte entera no cabe en el entero, el resultado depende de la implementación.

De entero a *real floating*, si el valor puede expresarse exactamente, se mantiene. Si no, se elige el valor más próximo hacia arriba o hacia abajo (dependiendo de la implementación) que pueda expresarse exactamente. Si el valor queda fuera del rango representable, el resultado es indefinido.

##### 6.3.1.5 Real floating types

La conversión entre tipos flotantes sigue la misma lógica que entre entero y *floating*.

##### 6.3.1.6 Complex types

Tanto la parte real como la imaginaria siguen las mismas reglas que las conversiones entre *floatings*.

##### 6.3.1.7 Real and complex

Entre tipos reales (no `_Bool`) y complejos se sigue la misma lógica que entre *floatings*, o entre enteros y *floatings*, con la particularidad que de real a complejo la parte imaginaria resultante es cero, y de complejo a real se descarta la parte imaginaria.

##### 6.3.1.8 Usual arithmetic conversions

Los operandos de una operación aritmética se convierten a un tipo común, y el resultado es de ese mismo tipo. Si ambos operandos pertenecen a dominios distintos (complejo/real), el tipo común será complejo.

Si un operando es `long double`, el otro se convierte a `long double`; si no, si uno es `double`, el otro se convierte a `double`; si no, si uno es `float`, el otro se convierte a `float`; si no, se efectúa la promoción de enteros (*integer promotion*).

*Integer promotion*: operacion aritmética entre dos enteros:

- Si ambos enteros son de igual tipo, no hay promoción alguna.
- Si no, si ambos son *signed* o *unsigned*, se promociona el de menor rango.
- Si no, si el *unsigned* tiene mayor o igual rango, se convierte el *signed* al tipo del otro.
- Si no (o sea, si el *signed* tiene mayor rango), si dicho *signed* puede representar todos los valores del *unsigned*, el *unsigned* se convierte al tipo del *signed*.
- Si no (es decir, si el *signed* tiene mayor rango pero no el suficiente como para poder representar todos los valores del *unsigned*), se convierten ambos al tipo *unsigned* correspondiente al tipo del operando *signed*.

El rango de un entero no tiene nada que ver con su *signedness*. Por ejemplo `long int` tiene el mismo rango que `unsigned long int`, y `unsigned short int` tiene rango inferior a `int`.

Veamos un ejemplo para ver lo que pasa con el signo de un operando cuando este es un *signed* que se promociona *unsigned*:

```c
int a = -10;    // este operando promocionará a unsigned
unsigned int b = 100;
int c = a + b;    // el resultado debería ser 90
```

En este caso, examinemos la operación ***a + b***: se trata del caso en el que un operando es *signed* (***a***) y el otro *unsigned* (***b***), y ambos tienen el mismo rango. Nos hallamos, pues en el caso descrito en el punto tercero: el *signed* ***a*** promociona a `unsigned int`. Al ser negativo (***-10***), el valor no es representable en ese tipo, con lo que, como hemos visto, a este valor se le sumará ***N*** (en este caso, valor máximo representable por un `unsigned int`, más uno), con lo que el nuevo valor de ***a*** pasará a ser ***N-10***, que sí es representable con un `unsigned int` (es el máximo representable menos ***9***). Ahora ya podemos sumar ***a + b***, que será ***(N - 10) + (100)***, lo cual es ***N + 90***. Como este número no es representable en el tipo de datos del resultado (`unsigned int` también), se le restará en esta ocasión ***N*** a dicho número, con lo que acabaremos obteniendo ***90***. Luego, ese ***90*** *unsigned* se convierte a ***90*** *signed* para poder asignarlo a ***c***. La conversión es inmediata, y ***c*** acaba con valor ***90***, que es el resultado correcto.

#### 6.3.2 Other operands

##### 6.3.2.1 Lvalues, arrays, and function designators

Un *lvalue* es una expresión (de tipo distinto a `void`) que designa (hace referencia a) un objeto. Cuando hacemos referencia a un objeto, el tipo de ese objeto es el tipo del *lvalue* que usamos para referirnos a él. Un *lvalue* modificable es un *lvalue* que no es de tipo *array*, ni de un tipo incompleto, ni un tipo cualificado con `const`, y en caso de ser una estructura o union no tiene ningún miembro (recursivamente) de un tipo *const-qualified*.

Un *lvalue* que no es de tipo *array* se convierte automáticamente al valor del objeto que designa, en lugar de ser una referencia a él, a no ser que sea el operando de `sizeof`, `&` unario, `++` o `--`; o cuando es el operando a la izquierda de `.` o `=`. Esto se llama conversión de *lvalue*.

Sea ***T*** un tipo cualquiera, una expresión de tipo *array de T*  es convertida a un valor *apuntador a T*, con la dirección del primer elemento del *array*, con lo que deja de ser un *lvalue*. Esto no sucede cuando esa expresión es el operando de `sizeof`, `&` unario, o es un literal *string* usado para inicializar un *array* (el operador `&` unario ya realiza esa acción explícitamente).

Un designador de función (*function designator*) es una expresión de tipo *función que retorna T*, y es convertido a *apuntador a función que retorna T*, a no ser que sea el operando de `sizeof` o del operador unario `&`, el cual ya realiza esa acción explícitamente.

##### 6.3.2.2 void

Una expresión de tipo `void` no retorna ningún valor. Solo interesan sus *side effects*.

##### 6.3.2.3 Pointers

Un apuntador a `void` puede convertirse a y desde apuntador a cualquier tipo, y ambos apuntadores deben compararse iguales. Lo mismo sucede entre apuntadores a tipo cualificado y sin cualificar.

Un apuntador con valor ***0*** (de cualquier tipo) es un *null pointer*, y no apunta a ningún objeto o función. Dos apuntadores nulos se comparan igual, independientemente de sus tipos.

La conversión de enteros a apuntadores y viceversa está definido por la implementación. Los valores de un apuntador no tienen por qué ser representables por ningún tipo de enteros.

Un apuntador de un tipo puede convertirse a apuntador de otro tipo (manteniendo su valor). En este caso, puede que el nuevo tipo haga que el apuntador no esté alineado, puede apuntar a un tipo incorrecto, incluso ser una *trap representation* (objeto cuya representación es distinta al tipo de datos de ese objeto). Una simple lectura a una *trap representation* produce *undefined behaviour*. Si volvemos a convertirlo de vuelta, el resultado debe ser idéntico al apuntador original.

Un apuntador a función de un tipo puede ser convertido a apuntador a función de otro tipo. Al volver a convertirlo a su tipo original, el resultado debe ser igual al apuntador original. Si convertimos a un tipo no compatible y llamamos a la función, el comportamiento es indefinido.

### 6.4 Lexical elements

Cada *token* tendrá léxicamente la forma de *keyword*, identificador, constante, literal *string* o *punctuator*.

#### 6.4.1 Keywords

`auto`, `break`, `case`, `char`, `const`, `continue`, `default`, `do`, `double`, `else`, `enum`, `extern`, `float`, `for`, `goto`, `if`, `inline`, `int`, `long`, `register`, `restrict`, `return`, `short`, `signed`, `sizeof`, `static`, `struct`, `switch`, `typedef`, `union`, `unsigned`, `void`, `volatile`, `while`, `__Alignas`, `_Alignof`, `_Atomic`, `_Bool`, `_Complex`, `_Generic`, `_Imaginary`, `_Noreturn`, `_Static_assert`, `_Thread_local`.

El keyword `_Imaginary` está reservado para usos futuros (tipos imaginarios). Son *case sensitive*.

#### 6.4.2 Identifiers

##### 6.4.2.1 General

En cuanto al *basic source character set*, los identificadores pueden contener los caracteres guión bajo (***\_***), ***a***-***z*** y ***A***-***Z***. Exceptuando el carácter inicial, también ***0***-***9***.

En cuanto al *extended source character set*, la implementación puede aceptar otros caracteres en identificadores (como *universal character names* en el código fuente). El **anexo D** especifica qué caracteres pueden usarse, y de estos, cuáles no se pueden usar como primer carácter. Estos caracteres pueden codificarse (conjunto de caracteres de ejecución) mediante caracteres *multibyte* y tendrán una codificación *implementation defined*.

Aunque no hay límite en la longitud de un identificador, la implementación puede limitar el número de caracteres significativos, que puede ser más restrictivo en identificadores con *linkage* externo.

##### 6.4.2.2 Predefined identifiers

Existe un identificador predeterminado. Siguiendo a la llave de apertura de cada función, queda implícitamente declarado:

```c
static const char __func__[] = "function-name";
```

#### 6.4.3 Universal character names

Los caracteres extendidos también pueden describirse en el archivo fuente como *universal character names*, que son descripciones del tipo `\unnnn` o `\Unnnnnnnn`, describiendo, respectivamente, el identificador corto de 4 dígitos de la codificación *ISO/IEC 10646* (equivale a los caracteres *Unicode*) y el de 8 dígitos (estos *short identifiers* están definidos en el estándar *Unicode*).

Estos no pueden describir caracteres del conjunto básico, ni caracteres *Unicode* de control.

El **anexo D** da instrucciones sobre su uso en identificadores, pero independientemente pueden usarse en literales *string*, y constantes carácter.

#### 6.4.4 Constants

Cada constante tiene su propio tipo. Las constantes son de tipo entero (decimal, octal o hexadecimal), enumeración, punto flotante (decimal o hexadecimal) o carácter.

##### 6.4.4.1 Integer constants

Las constantes enteras tienen la sintaxis decimal, octal o hexadecimal. Empiezan por dígito numérico y no tienen punto decimal ni parte exponencial:

- Decimal: serie de números ***0***-***9*** que no empieza por ***0***.
- Octal: ***0*** seguido por una serie opcional de números ***0***-***7***.
- Hexadecimal: serie de dígitos ***0***-***9***, ***a***-***f***, ***A***-***F*** precedida de prefijo ***0x*** o ***0X***.

Las constantes enteras pueden tener un sufijo opcional, indicando *unsignedness* (***u*** o ***U***), longitud (***l***, ***L***, ***ll*** o ***LL***), o ambas cosas. Una ele (***l*** o ***L***) indica `long`, mientras que dos (***ll*** o ***LL***) indica `long long`.

El tipo de la constante será el menor (empezando por `int`) en el que se pueda representar su valor, teniendo siempre en cuenta el sufijo. En las constantes decimales no se consideran los tipos *unsigned* (a no ser que el prefijo incluya ***u*** o ***U***). Si el valor no puede representarse mediante ningún tipo de entero, la constante no tiene tipo.

##### 6.4.4.2 Floating constants

Una **constante fraccionaria** consiste en dos secuencias de 0 o más dígitos separadas por un punto decimal. Si una de las dos secuencias tiene 0 dígitos, la otra deberá contener por lo menos uno. El punto decimal aparece siempre. Los dígitos serán decimales (***0***-***9***) o hexadecimales (***0***-***9***, ***a***-***f***, ***A***-***F***), según se trate de una constante fraccionaria **decimal** o **hexadecimal**.

Una **parte exponencial** consiste en un carácter ***e*** (o ***E***), seguido por un signo opcional (***+*** o ***-***), y finalmente una secuencia de dígitos decimales (la base del exponente es 10).

Una **parte exponencial binaria** consiste en un carácter ***p*** (o ***P***), seguido por un signo opcional (***+*** o ***-***), y finalmente una secuencia de dígitos decimales (la base del exponente es 2).

Una **constante de punto flotante** puede ser decimal o hexadecimal.

Una **constante de punto flotante decimal** puede consistir simplemente en una constante fraccionaria decimal, a la que opcionalmente se puede añadir una parte exponencial (decimal o binaria, pero no ambas). La otra opción es una secuencia de dígitos decimales (sin punto), en cuyo caso, deberá haber obligatoriamente una parte exponencial.

Una **constante de punto flotante hexadecimal** debe empezar siempre con un prefijo ***0x*** o ***0X***, después del cual puede haber una constante fraccionaria hexadecimal, seguida obligatoriamente por una parte exponencial binaria. La otra opción es, tras el prefijo inicial, una secuencia de dígitos hexadecimales (sin punto) seguida también de una parte exponencial binaria.

Todas las constantes así definidas son de tipo `double` por defecto. Pero se les puede añadir el sufijo ***f*** (o ***F***) para `float`; o ***l*** (o ***L***) para `long double`.

##### 6.4.4.3 Enumeration constants

Una constante de una enumeración tiene tipo `int`.

##### 6.4.4.4 Character constants

Un carácter puede ser *multibyte* o *wide*. Lo de 1 caracter 1 `char` es un concepto antiguo, ya que es improbable que el juego de ejecución completo esté codificado mediante un *byte* por miembro. Aunque el compilador *GCC*, por ejemplo, permite especificar explícitamente una codificación para el juego de caracteres de ejecución, no es práctica habitual especificar un juego de caracteres de esas características. Sin embargo, podríamos usarlo si solo utilizamos los 128 caracteres *ASCII* (y opcionalmente una codificación extendida de *ASCII* con 256 caracteres en total).

Una constante carácter (en el archivo fuente) está compuesta por una secuencia de uno (o más, si lo permite la implementación) caracteres *multibyte* (por su codificación en el archivo) entre comillas simples. Un carácter *wide* es lo mismo, pero con un prefijo ***L***, ***u*** o ***U***:

- `'x'` tiene tipo `int`, aunque su valor se puede representar en un `char`.
- `L'x'` tiene tipo `wchar_t` (***stddef.h***).
- `u'x'` tiene tipo `char16_t` y `U'x'` tipo `char32_t` (***uchar.h***).

Así es cómo se puede indicar una constante carácter:

- Cada miembro del *source character set*, excepto comilla simple (***'***) y barra invertida (***\\***), se mapea a su correspondiente carácter del conjunto de ejecución.
- Caracteres que se deben indicar mediante secuencias de escape: ***\\'***, ***\\\\***, ***\\a*** (alerta), ***\\b*** (retroceso), ***\\f*** (*form feed*), ***\\n*** (*newline*, 10), ***\\r*** (retorno de carro, 13), ***\\t*** (tabulador) y ***\\v*** (tabulador vertical). El interrogante y las dobles comillas pueden ponerse tanto tal cual (***?***, ***"***) como *escaped* (***\\?***, ***\\"***).
- Secuencia octal: ***\\*** seguido de 1, 2 o 3 dígitos octales (***0***-***7***). Se consideran parte de la secuencia todos los dígitos posibles (hasta 3).
- Secuencia hexadecimal: ***\\x*** seguido de dígitos hexadecimales, sin limite; el final lo marca un dígito no-hexadecimal.
- Un *universal character name*.

Si una constante carácter *multibyte* (o secuencia de escape), es decir, no *wide*, se corresponde con un carácter del conjunto básico de ejecución, tiene el valor numérico de la codificación de ese carácter. Si se corresponde con un carácter que no es del conjunto básico, es decir, si se corresponde con una secuencia de más de un *byte* (no se puede almacenar en un `char`), el comportamiento es definido por la implementación. Por ejemplo:

```c
char c1 = 'N';   // correcto
char c2 = 'Ñ';   // char no es suficiente
```

Tal carácter debería leerse en una variable de tipo carácter *wide*.

La implementación también debe decidir qué se hace si permite constantes con más de un carácter (tipo ***'xy'***).

#### 6.4.5 String literals

Un literal *string* de caracteres (`char`) es una secuencia de caracteres *multibyte* entre comillas dobles. Tal *string* de caracteres se codificará en el entorno de ejecución mediante la codificación por defecto de la implementación (o indicada en línea de comandos), es decir, la codificación del *execution character set*; aunque podemos forzar la codificación *UTF-8* para los *strings multibyte* si indicamos el prefijo ***u8***.

Un *string wide* llevará prefijo ***L*** (caracteres `wchar_t`), ***u*** (caracteres `char16_t`) o ***U*** (caracteres `char32_t`).

Análogamente a lo que sucede con el tipo carácter, un *string* es *multibyte* (sin prefijo o con prefijo ***u8***) o *wide* (con prefijo ***L***, ***u*** o ***U***).

Los mismos principios de las constantes carácter en cuanto a formato valen para los elementos de los *strings*, exceptuando que la comilla simple (***'***) puede representarse por sí misma o mediante secuencia de escape, mientras que la comilla doble debe representarse siempre mediante secuencia de escape (***\\"***).

Los *strings* adyacentes con idéntico prefijo se concatenan durante el preproceso. Una serie de *strings* adyacentes en los que uno o más tienen un prefijo concreto (siempre el mismo) y los demás no tienen prefijo, se consideran todos con ese prefijo. No se puede concatenar de este modo *strings multibyte* con *strings wide*, con lo que no puede haber una secuencia con *strings wide* y *strings UTF-8*. En el caso de que existan prefijos *wide* distintos en los *strings*, la implementación decidirá qué hacer.

Durante el preproceso también se añade al final de los literales *string* un *byte* o código *wide* de valor ***0***, y la secuencia resultante se usa para inicializar un *array* de *static storage duration* con la longitud justa para contener la secuencia. No se debe modificar el contenido de esas posiciones de memoria. Los literales *string* sin prefijo o *UTF-8* formarán secuencias `char` *multibyte* con la codificación correspondiente. Para literales *wide*, serán secuencias del tipo indicado por el prefijo.

Nota sobre los literales *string*: el compilador reserva memoria para los literales *string* en una tabla de **constantes *string***. Por lo tanto, esto es incorrecto:

```c
char *s = "Un literal string";  // correcto; s apunta a la constante
s[0] = `O`;  // incorrecto: no se puede cambiar esa zona de memoria
```

Lo correcto sería usar un *array*, ya que en ese caso sí se reserva una zona de memoria para el *string*:

```c
char s[] = "Un literal string";  // correcto; reservamos memoria para el string
s[0] = `O`;  // correcto
```

#### 6.4.6 Punctuators

Operadores y *tokens* de preproceso. Se incluyen los *digraphs*, que equivalen al puntuador indicado:

`<:` (***\[***), `:>` (***\]***), `<%` (***{***), `%>` (***}***), `%:` (***#***) y `%:%:` (***##***).

#### 6.4.8 Preprocessing numbers

Números utilizados en el preproceso. Sintácticamente empieza por dígito, opcionalmente precedido por un punto, y puede ir seguido por una cantidad arbitraria de números, caracteres de identificador, puntos y/o secuencias ***e+***, ***e-***, ***E+***, ***E-***, ***p+***, ***p-***, ***P+*** o ***p-***. La definición sintáctica es muy laxa, pero su objetivo es el de representar números enteros y flotantes (sin complicarse en la definición de la sintaxis).

#### 6.4.9 Comments

Pueden contener caracteres *multibyte*.

Se abre con `/*`, hasta `*/`. No anidable.

`//` hasta fin de linea. No funcionan como tal dentro de literal *string* o constante carácter.

### 6.5 Expressions

Una expresión es una secuencia de operadores y operandos que especifican un valor, designan un objeto, realizan una serie de *side effects*, o una combinación de todo ello. La evaluación de los operandos se secuencia antes que la del operador.

Los operadores *bitwise* y el unario `~` precisan operandos enteros, y su valor depende de la representación en memoria de dichos enteros (en tipos *signed*, su resultado es *implementation-defined*).

El valor de un objeto puede ser accedido a través de un *lvalue* cuyo tipo sea compatible con el del objeto al que accedemos, o una versión *qualified* de un tipo compatible, o la versión *signed* o *unsigned* de una versión *qualified* de un tipo compatible, o un *aggregate or union type* que contenga un tipo como los mencionados, o un tipo carácter.

En el estándar, el orden de precedencia viene definido por la sintaxis. A partir de la sintaxis definida en esta cláusula 6.5 completa, podemos fabricarnos esta lista ordenada (dentro de la misma subcláusula tienen todos la misma precedencia):

- `()`, selecciones genéricas (6.5.1).
- `[]`, `.`, `->`, *postfix* `++` y `--` (6.5.2).
- `sizeof`, `_Alignof`, operatores unarios `++`, `--`, `&`, `*`, `+`, `-`, `~`, y `!` (6.5.3).
- Operadores *cast* (6.5.4).
- `*`, `/`, `%` (6.5.5).
- `+`, `-` (6.5.6).
- `<<`, `>>` (6.5.7).
- `<`, `<=`, `>`, `>=` (6.5.8).
- `==`, `!=` (6.5.9).
- `&` (6.5.10).
- `^` (6.5.11).
- `|` (6.5.12).
- `&&` (6.5.13).
- `||` (6.5.14).
- `?:` (6.5.15).
- `=`, `*=`, `/=`, `%=`, `+=`, `-=`, `<<=`, `>>=`, `&=`, `^=`, `|=` (6.5.16).
- `,` (6.5.17).

#### 6.5.1 Primary expressions

Son los identificadores, constantes, literales *string*, expresiones entre paréntesis y selecciones genéricas.

##### 6.5.1.1 Generic selection

Retornará un valor u otro dependiendo del tipo del argumento:

```c
_Generic(objeto, int: 1, float: 2, double: 3, default: 0);
```

Es útil para simular la sobrecarga de funciones. Podríamos definir una macro `cbr()` que se convirtiera en `cbrtl()`, `cbrt()` o `cbrtf()` dependiendo del tipo de la expresión del argumento:

```c
#define cbr(X) _Generic((X),    \
    long double: cbrtl,          \
    default: cbrt,               \
    float: cbrtf)(X)
```

Solo puede haber 1 tipo por defecto. Los tipos de la lista deben ser completos, y no pueden ser tipos variablemente modificados o derivados de estos. No puede haber tipos compatibles entre los especificados.

Si la expresión retornada es un *lvalue*, un designador de función o una expresión `void`, también lo será el resultado, y actuará como tal. El tipo de la expresión del argumento será, cuando proceda, como si se aplicara una conversión *lvalue*, o *array* a apuntador, o función a apuntador. El tipo de la expresión indicada no puede ser compatible con más de uno de los tipos; y si no existe elemento `default`, debe serlo con exactamente uno de ellos.

#### 6.5.2 Postfix operators

##### 6.5.2.1 Array subscripting

Es del tipo `arr[ind]`, donde ***arr*** es de tipo apuntador a un tipo completo, y ***ind*** es un tipo de entero, y equivale a `*(arr+ind)`. Así, un identificador *array* se trata como un apuntador (***arr*** es tratado como un apuntador al primer elemento del *array*: `&arr[0]`), como se vio en 6.3.2.1.

Para *arrays* multidimensionales, supongamos: `int x[3][5];` Aquí ***x*** es un *array* de 3 elementos, siendo cada elemento un *array* de 5 enteros. Si hacemos `x[i]`, eso equivale a `*(x+i)`. ***x*** se trata como un apuntador al primero de los tres elementos de tipo `int[5]`, mientras `*x` (o `x[0]`) es un apuntador al primer *array* de cinco elementos `int`, siendo `**x` (o `x[0][0]`) ese `int`. Así, `x[1]` (o `x + 1`) es un apuntador al segundo *array* de 5 enteros.

De forma similar, `int x[10][3][5]` es un *array* de 10 elementos de tipo `int[3][5]`, con lo que en este caso, ***x*** se trataría como un apuntador al primero de los elementos de tipo `int[3][5]`.

##### 6.5.2.2 Function calls

La expresión que denota la función llamada será de tipo apuntador a función que retorna `void` o tipo completo, excepto *array*. Cada argumento de la lista (que puede estar vacía) será de un tipo completo. Cada argumento coincidirá con cada parámetro de la declaración de la función. La expresión será del tipo de retorno de la función (si es `void`, la expresión no retornará valor). Se permiten las llamadas recursivas.

En cuanto al tema de secuenciación:

```c
(*pf[f1()]) (f2(), f3() + f4());
```

Esta llamada se hace a través de ***pf***, un *array* de apuntadores a función. La función ***f1*** proporciona el índice (debe retornar un entero), y las funciones ***f2***, ***f3*** y ***f4*** el valor de los argumentos. Entonces la llamada a la función apuntada no se realizará hasta que todas las funciones, desde ***f1*** hasta ***f4***, hayan sido evaluadas y sus *side effects* realizados.

Por cierto, esto funcionaría igual:

```c
(pf[f1()]) (f2(), f3() + f4());  // suprimimos '*'
```

Ello es debido a que `pf[N]` es un apuntador a función. Sin embargo, `(*pf)[N]` es el valor apuntado, es decir, es un designador de función, que como hemos visto en 6.3.2.1, se convierte en apuntador a función automáticamente. Del mismo modo, una llamada simple a función, del tipo `foo()`, se convierte automáticamente en apuntador a función, con lo que podríamos hacer esa conversión explícitamente: `(&foo)()` y funcionaría igualmente.

##### 6.5.2.3 Structure and union members

Se usa operador `.` (`objeto.miembro`) para acceder a los miembros de un objeto de tipo estructura o unión, o `->` (`apuntador->miembro`) para hacerlo a través de un apuntador a estructura o unión.

Si el primer operando (expresión que denota la estructura o unión) está *qualified*, el resultado (expresión que denota al miembro) queda *qualified* del mismo modo (a parte de otras *qualifications* que ya tuviera el miembro de por sí).

Si el primer operando es un *lvalue*, el resultado es un *lvalue*; en el caso del operador `->`, el resultado es siempre un *lvalue*. Por ejemplo, si la función `foo()` retorna una unión o estructura y `pfoo()` retorna un apuntador a unión o estructura, `foo().member`, aunque es una expresión válida, no es un *lvalue*, pero `pfoo()->member` sí lo es. Por lo tanto:

```c
foo().member = valor;  // incorrecto
pfoo()->member = valor;  // correcto
```

##### 6.5.2.4 Postfix increment and decrement operators

Son del tipo `v++` y `v--`, siendo ***v*** un *lvalue* modificable, de tipo real o apuntador. El valor de la expresión es el valor del operando, aunque como *side effect* posterior a la evaluación, se incrementa (`++`) o decrementa (`--`) su valor en 1.

##### 6.5.2.5 Compound literals

Son del tipo `(tipo){lista de inicialización}`. Proporciona un objeto sin nombre, cuyo valor es el de la lista de inicialización. El tipo debe ser un tipo completo, o en su defecto un *array* de tamaño desconocido que se completa al inicializar (pero no un *variable length array*). El resultado es un *lvalue*.

Si está fuera de toda función, tiene *static storage duration*; de lo contrario, *automatic storage duration*. Las reglas sintácticas de las listas de inicialización (6.7.9) se aplican a los *compound literals*.

Ejemplos:

- `(int[]){2, 4}` se puede asignar a un `*int`.
- `(struct s){3, 5}` es un objeto sin nombre de tipo `struct s`.
- `(struct s){.x=3, .y=5}` inicializa con designadores de campo.
- `&(struct s){3, 5}` es un apuntador sin nombre al objeto.
- `(const int[]){1, 2, 3, 4}` es un *compound literal* de solo lectura.

Estos tres son distintos:

- `"Abcde"` es el clásico literal *string*. Tiene siempre *static storage duration*, y el resultado de su modificación es indefinido.
- `(char []){"Abcde"}` con *automatic* o *static storage duration* y modificable.
- `(const char []){"Abcde"}` es igual que el anterior, pero no es modificable.

#### 6.5.3 Unary operators

##### 6.5.3.1 Prefix increment and decrement operators

Son del tipo `++v` o `--v`. Incrementa (`++`) o decrementa (`--`) en 1 el valor del operando, y el resultado de la expresión es el nuevo valor. El operando debe ser un *modifiable lvalue* de tipo real o apuntador.

##### 6.5.3.2 Address and indirection operators

El operando de `&` debe ser un designador de función, el resultado de `[]`, o de indirección (`*`), o un *lvalue* (que no sea *bit field* ni `register`). Por otro lado, el operando de indirección (`*`) debe ser de tipo apuntador.

`&v` devuelve la direccion de `v` (si `v` es de tipo ***T***, `&v` es de tipo ***apuntador a T***). En el caso de `*v`, si el apuntador `v` apunta a una función, el resultado es un *function designator*; si apunta a un objeto, el resultado es un *lvalue* que designa al objeto.

##### 6.5.3.3 Unary arithmetic operators

Los operadores unarios de signo `+` y `-` tendrán tipo aritmético, `~` (complemento) tipo entero, y `!` (negación) tipo escalar.

El complemento conmuta cada bit del operando. Si realizamos la operación sobre un tipo sin signo, el resultado será el máximo representable en ese tipo menos el valor del operando.

En cuanto a la negación, retorna ***0*** si el operando es distinto de ***0***, y ***1*** en caso contrario.

##### 6.5.3.4 The sizeof and _Alignof operators

`sizeof` devuelve el tamaño (en *bytes*) del operando, que puede ser una expresión (entre paréntesis o no) o un nombre de tipo entre paréntesis. El operando no puede ser de tipo función, ni un tipo incompleto, ni un miembro *bit-field*. El resultado es un entero. El operando no se evalúa, a no ser que se trate de un *variable-length array*.

En tipo `char` y familia devuelve ***1***. Un *array* devolverá el número de *bytes* que contiene en total. En una estructura o unión, el número de *bytes*, incluidos los *padding bytes* (internos y *trailing*).

`_Alignof()` devuelve el requisito de alineación del operando (6.2.8), el cual no es evaluado. El operando es el nombre de un tipo o expresión. No se puede aplicar a operandos de tipo función o tipos incompletos.

El resultado de ambos operadores es del tipo `size_t` (un tipo de entero sin signo, deﬁnido en ***stddef.h*** y otros *headers*).

Puede ser útil para determinar el número de elementos de un *array*:

```c
n = sizeof array / sizeof array[0];
```

#### 6.5.4 Cast operators

Es del tipo `(tipo)expresion`, y es una conversión explícita al tipo indicado, que debe ser escalar o `void`. La expresión tendrá tipo escalar.

Las conversiones que impliquen apuntadores se deben indicar explícitamente con un *cast*, excepto cuando se permita hacer implícitamente con operadores de asignación.

No se puede convertir de *floating* a apuntador ni al revés.

#### 6.5.5 Multiplicative operators

Son `*` (multiplicación), `/` (división) y `%` (resto, módulo). El tipo de los operandos debe ser aritmético en multiplicación y división, y entero en el caso del módulo.

El segundo operando de división y módulo no puede ser cero. Al dividir dos enteros se desecha la parte fraccionaria y el resultado es entero.

#### 6.5.6 Additive operators

Son `+` (suma) y `-` (resta). Los operandos deben ser de tipo aritmético, o un apuntador a tipo completo y un entero. En el caso de la resta, también pueden ser dos apuntadores. El tipo del resultado será como el de los operandos (tras promoción).

Con un apuntador ***P*** y un entero ***i***, se puede hacer `P + i` o `P - i` para avanzar o retroceder por un *array*; se le suma/resta al apuntador el tamaño del elemento (según el tipo del apuntador) y el resultado es un nuevo apuntador del mismo tipo que el primero.

En caso de que un apuntador apunte a un elemento que no pertenezca a un *array*, se tratará como si estuviese apuntando a un *array* de 1 elemento del tipo indicado por el apuntador.

Si restamos dos apuntadores, estos deben apuntar a elementos del mismo *array* (o un elemento más allá del último) y el resultado, que es la diferencia de índice entre los dos, es de tipo `ptrdiff_t` (un entero con signo definido en ***stddef.h***).

#### 6.5.7 Bitwise shift operators

Son `>>` (*shift right*) y `<<` (*shift left*), con operandos enteros. El tipo resultante es el del operando izquierdo (tras la posible promoción). Si el operando derecho es negativo o superior o igual al ancho del izquierdo (tras promoción), el resultado es indefinido.

Los desplazamientos se rellenan con ceros. Si el primer operando es negativo, el resultado depende de la implementación.

#### 6.5.8 Relational operators

Son `>` (mayor), `<` (menor), `>=` (mayor o igual) y `<=` (menor o igual). Los operandos deben ser  de tipo real, o apuntadores a tipos compatibles.

Si se comparan apuntadores, el resultado depende de la posición relativa en memoria de los objetos a los que apuntan. Si apuntan al mismo objeto, se comparan igual. Si apuntan a miembros de un mismo objeto estructura, se tendrá en cuenta el orden de dichos miembros dentro del objeto (el último miembro declarado será el que retornará mayor valor). En un *array*, se sigue el orden según índice. Dentro de los miembros de una unión, todos tienen el mismo orden.

En caso de que un apuntador apunte a un elemento que no pertenezca a un *array*, se tratará como si estuviese apuntando a un *array* de 1 elemento del tipo indicado por el apuntador.

Dentro de un *array*, el resultado será definido si los apuntadores apuntan a elementos del mismo, o uno más allá del último; de lo contrario, es indefinido.

El resultado es un `int` que será 0 (falso) o 1 (verdadero).

#### 6.5.9 Equality operators

Son `==` (igualdad) y `!=` (desigualdad). Los operandos deben cumplir uno de estos puntos:

- Ambos serán de tipo aritmético.
- Serán dos apuntadores a tipos compatibles.
- Serán dos apuntadores, uno de los cuales es `*void`. El resultado será del tipo del apuntador `*void` con las *qualifications* oportunas.
- Un operando será un apuntador y el otro una constante ***NULL***. La constante se convierte al tipo del apuntador.

Dos complejos se evalúan igual si sus partes reales e imaginarias son iguales. Dos operandos aritméticos de distinto tipo se evalúan igual si sus conversiones al tipo pertinente son iguales.

En caso de que un apuntador apunte a un elemento que no pertenezca a un *array*, se tratará como si estuviese apuntando a un *array* de 1 elemento del tipo indicado por el apuntador.

Dos apuntadores se evalúan igual si son ambos ***NULL***, o apuntan al mismo objeto (incluyendo apuntador a objeto y apuntador al subobjeto inicial de este), o a la misma función, o ambos apuntan al elemento posterior al último en un *array*, o uno apunta a ese elemento y el otro apunta a otro *array* que siguiera inmediatamente a este en memoria (si el elemento que sigue no pertenece a un *array*, se tratará como si fuese un *array* de longitud 1, como hemos visto).

El resultado es un `int` que será ***0*** (falso) o ***1*** (verdadero).

#### 6.5.10 Bitwise AND operator

Operación *AND* bit a bit (`&`). Operandos enteros.

#### 6.5.11 Bitwise exclusive OR operator

Operación *OR-EX* bit a bit (`^`). Operandos enteros.

#### 6.5.12 Bitwise inclusive OR operator

Operación *OR* bit a bit: (`|`)  Operandos enteros.

#### 6.5.13 Logical AND operator

Operación *AND* lógico (`&&`) entre dos tipos escalares, con resultado de tipo `int` y valor ***0***, si alguno de los operandos tiene valor ***0***, o ***1*** en caso contrario. La evaluación es de izquierda a derecha, con un *sequence point* entre operandos. Si el primero es ***0***, el segundo ya no se evalúa.

#### 6.5.14 Logical OR operator

Operación OR lógico (`||`) entre dos tipos escalares, con resultado de tipo `int` y valor ***0***, si ambos operandos tienen valor ***0***, o ***1*** en caso contrario. La evaluación es de izquierda a derecha, con un *sequence point* entre operandos. Si el primero es ***1***, el segundo ya no se evalúa.

#### 6.5.15 Conditional operator

Es del tipo `op1 ? op2 : op3` y se evalúa primero el operando ***op1***, que debe ser de tipo escalar. Si este resulta diferente a ***0***, se evalúa únicamente ***op2***. De lo contrario, solo ***op3***.

En cuanto a los tipos de los operadores segundo y tercero, se aplican las mismas normas que en el caso de los operadores de igualdad (6.5.9), tanto en los tipos como en las conversiones, pero además:

- Los operandos pueden ser de tipo unión o estructura (la misma).
- Los operandos pueden ser ambos de tipo `void`.
- Si los operandos son apuntadores a tipos compatibles, el resultado es de tipo apuntador al tipo compuesto.

#### 6.5.16 Assignment operators

El primer operando de un operador de asignación es un *lvalue* modificable. El valor del segundo operando es asignado al objeto designado por el *lvalue* (primer operando), **tras ser convertido al tipo de este**, y el resultado de la expresión de asignación es ese mismo valor, aunque no es un *lvalue*.

##### 6.5.16.1 Simple assignment

Es del tipo `A = B`. Si el *lvalue* modificable (***A***) es de tipo aritmético, el segundo operando (***B***) también debe serlo. Si es de tipo unión o estructura, el segundo debe ser compatible con este. Si es de tipo apuntador, el segundo debe ser apuntador a tipo compatible, o bien uno de los dos debe ser un apuntador a `void`. Si ***A*** es `_Bool`, el segundo operando puede ser un apuntador.

##### 6.5.16.2 Compound assignment

Son los operadores `*=`, `/=`, `%=`, `+=`, `-=`, `<<=`, `>>=`, `&=`, `^=` y `|=`.

El primer operando debe ser un *lvalue* modificable. Al objeto que designa este se le asigna el valor de la expresión consistente en la operación indicada con los operandos primero y segundo. Por ejemplo:

```c
A *= B;
```

equivale a:

```c
A = A * B;
```

Con la diferencia de que en el primer caso ***A*** se evalúa una sola vez.

El resultado de la expresión es el mismo que se ha asignado al objeto designado por ***A***.

En el caso de `+=` y `-=`, el primer operando debe ser un apuntador a un tipo completo y el segundo debe ser entero, o bien ambos deben tener tipo aritmético.

En el resto, el primer operando debe tener tipo aritmético y el segundo un tipo aritmético acorde al operador concreto.

#### 6.5.17 Comma operator

Es de tipo `A, B`. Se evalúa el primer operando como expresión `void`, y el segundo operando se evalúa de tal modo que el resultado de la expresión tendrá su valor y tipo.

### 6.6 Constant expressions

Son evaluadas en *translation time*.

La evaluación de una expresión constante *floating* debe tener la misma capacidad y precisión que si se evaluara en tiempo de ejecución.

Una expresión constante entera tendrá un tipo entero, y sus operandos solo podrán ser constantes enteras, constantes de enumeración, constantes carácter, expresiones `sizeof` con valor constante, expresiones `_Alignof` y constantes *floating* con *cast* a entero.

En una **inicialización**, las expresiones constantes pueden ser de tipo aritmético, apuntador nulo, dirección de memoria constante, o dirección de memoria constante a un objeto de tipo completo más/menos una expresión constante entera.

Una expresión constante de tipo aritmético tendrá tipo aritmético, y sus operandos serán los mismos que en el caso de los enteros, aunque en este caso se aceptan también constantes *floating* sin *cast*.

Una dirección de memoria constante es un apuntador nulo, un apuntador a un *lvalue* designando a un objeto con *static storage duration* o un apuntador a función. Se declarará explícitamente con `&` o constante entera con *cast* a apuntador, o implícitamente a través de expresiones de tipo *array* o función.

### 6.7 Declarations

Dejando al margen la *static assert declaration*, una declaración será una combinación de *declaration specifiers* (*storage-class specifiers*, *type specifiers*, *type qualifiers*, *function specifiers* y *alignment specifiers*) seguido de una lista opcional de *declarators* (6.7.6).

Los *declarators*, si existen, contienen el identificador que está siendo declarado, y además pueden contener información adicional del tipo y/o un inicializador.

Si un identificador no tiene *linkage*, no puede haber más de una declaración de este el mismo *scope* y *name space*.

Todas las declaraciones dentro del mismo *scope* que se refieran al mismo elemento, deben especificar tipos compatibles.

Una **declaración** indica la interpretación y atributos de un identificador. Una **definición** es una declaración que además causa que se reserve *storage* para el objeto, o si es función, incluye el cuerpo de la misma.

Adicionalmente, la primera declaración de una **constante** de enumeración, también es una definición de dicha constante. Lo mismo sucede con el **nombre** de un `typedef`.

Si un identificador está declarado sin *linkage*, su tipo debe ser completo al terminar su *declarator* (incluyendo el posible inicializador). Esto es lógico, ya que no se puede completar el tipo en una declaración posterior.

#### 6.7.1 Storage-class specifiers

Son `typedef`, `extern`, `static`, `_Thread_local`, `auto` y `register`. Solo se puede dar uno, a excepción de `_Thread_local`, que se puede combinar con `static` o `extern`. De hecho, si un objeto con *block scope* se define con el primero, es obligatorio especificar también uno de los otros dos.

El especificador `auto` suele ser redundante.

Un objeto declarado como `_Thread_local` debe especificarse así en todas sus declaraciones. Una función no puede ser declarada `_Thread_local`.

En el caso de `typedef` se le considera *storage-class specifier* a efectos sintácticos, pero no lo es.

`register` indica que el acceso sera lo más rápido posible, pero cada implementación lo hará de la manera que estime conveniente.

Si declaramos (no definimos) una función, dentro de un *block scope*, solo podemos indicar `extern`.

Los *storage-class specifiers* de un *array*, estructura, o unión, pasan a serlo también de los miembros de estos, en cuanto a *storage* (no *linkage*).

#### 6.7.2 Type specifiers

El tipo base en sí. Debe haber exactamente uno en la declaración. Puede ser un nombre de tipo, estructura, unión, enumeración o nombre `typedef`.

Un tipo entero indicado sin `unsigned`, será como si se indicara `signed`. Un tipo entero que no indique `int` se considerará como si lo indicara. Es decir, `long` equivale a `long int`, o `signed short` equivale a `short int`.

En el caso de un *bit-field* de tipo `int`, la implementación decide si será equivalente a `signed int` o `unsigned int`.

##### 6.7.2.1 Structure and union specifiers

Sintaxis (igual para estructuras y uniones):

```
struct [tag] { /* miembros */ } [variables];
```

Si no le damos nombre (*tag*), debemos incluir variable(s), de lo contrario no será utilizable. A no ser que sea una estructura/unión anónima (ver más abajo).

Para declarar variables posteriormente a la definición de la estructura o unión:

```
struct <tag> var1, var2;
```

Una estructura está compuesta por una secuencia de miembros, que se almacenen en memoria en el orden en que son declarados; en una unión, los miembros se solapan en memoria (empiezan todos en la misma dirección de memoria). Representan un tipo incompleto hasta el ***}*** final de la definición. La aparición de una lista de declaraciones (miembros) dentro de `struct` o `union` crea un nuevo tipo.

La declaración de miembros se hace igual que en una declaración de objetos, con la diferencia que no pueden llevar inicializador ni *storage class specifier*, y sin embargo pueden llevar indicador de *bit-field*.

Un *bit-field* se indica mediante dos puntos tras el nombre del miembro (***:***), seguido de una expresión entera constante que no puede exceder el ancho del tipo del miembro. Si el valor de esa constante es ***0***, no habrá *declarator* para ese campo (solo tipo, por ejemplo `unsigned int :0`). El tipo del miembro debe ser `_Bool`, `signed int`, `unsigned int`, u otros tipos que la implementación permita.

Una estructura o unión no puede contener miembros de tipo incompleto (por tanto no puede contener miembros del tipo de la propia estructura, aunque sí miembros apuntadores al tipo de la propia estructura) o función, a excepción del último miembro: en una estructura con más de un miembro con nombre, el último miembro puede ser un *array* de tipo incompleto; tal estructura (o una unión que contenga, quizá recursivamente, una estructura así) no puede formar parte de una estructura o *array*.

Un miembro puede ser de cualquier tipo completo, exceptuando tipos variablemente modificados).

Si un miembro es un *bit-field* y quedan bits suficientes para albergar un posible *bit-field* inmediatamente posterior, los *bits* de este serán empaquetados consecutivamente dentro del tipo del anterior. Si no cabe el *bit-field* posterior, la implementación decidirá si este empieza después del espacio ocupado por el tipo completo del anterior, o antes de eso, solapándose ambos tipos. No se especifica requisito de alineación en este caso. Para evitar el empaquetado de bits, se debe indicar un *unnamed bit-field* de longitud ***0*** (ver a continuación), de tal modo que no se solaparán los tipos.

Una declaración de ***bit-field* sin nombre** (*unnamed bit-field*, del tipo `unsigned int :12`) sirve para *padding*, aunque no se puede especificar al principio de todo (sí al final).

Un **miembro sin nombre** cuyo tipo es una estructura o unión **sin etiqueta**, es una estructura o unión anónima. Sus miembros se consideran miembros de la estructura o unión que la contiene (se aplica recursivamente).

Cada miembro no *bit-field* de una estructura o unión se alinea adecuadamente a su tipo, de forma *implementation-defined* (mejor no hacer suposiciones sobre el tamaño de la estructura o unión).

Un apuntador a un objeto de tipo estructura apunta al su miembro inicial, y si es un *bit-field*, a la unidad que lo contiene. Similarmente, un apuntador a unión apunta a todos sus miembros, o unidad contenedora de *bit-field*.

El tamaño de una unión es el suficiente para albergar a su mayor miembro.

```c
struct v {
    union {  // unión anónima
        struct { int i, j; };  // estructura anónima
        struct { long k, l; } w;
    };
    int m;
} v1;

v1.i = 2;  // válido
v1.k = 3;  // no válido: la estructura interna no es anónima
v1.w.k = 5;  // válido
```

Como caso especial, el último miembro de una estructura con más de un miembro con nombre, puede ser de tipo de *array* incompleto (*ﬂexible array member*). El tamaño de la estructura se comporta como si no existiera el *array*, que es ignorado en la mayoría de ocasiones. Se usa así:

```c
struct s { int n; double d[]; };
int m = 33;
struct s *p = malloc(sizeof(struct s) + sizeof(double[m]));
```

El objeto apuntado por el apuntador ***p*** se comportará como si hubiésemos definido la estructura asi:

```c
struct { int n; double d[33]; } *p;
```

Aunque en la inicialización, el *array* flexible es ignorado, con lo que:

```c
struct s var1 = { 20, {4.2}};  // ¡error!
```

##### 6.7.2.2 Enumeration specifiers

Sintaxis:

```
enum [nombre] { const1 [=val1], const2 [=val2],... };
```

La lista puede terminar en coma (`,`). Los valores ***valN*** deben ser expresiones constantes enteras representables como `int`. Las constantes serán declaradas como si de un `int` se tratase, y se pueden utilizar allí donde se pueda utilizar un identificador de tipo `int`.

Si a la primera constante no se le da valor, es ***0***. Una constante sin valor asignado tendrá el valor de la constante de su izquierda más uno. Se pueden duplicar valores. El tipo es incompleto hasta llegar a la ***}***.

Un tipo enumeración será compatible con tipo `char` o entero, a elección de la implementación, pero deberá ser capaz de representar todos los valores de la misma.

##### 6.7.2.3 Tags

Una etiqueta (*tag*) precedida de `struct`, `union` o `enum` define un tipo. Un tipo puede definir su contenido como máximo una vez.

Dos tipos que tienen el mismo *tag* dentro del mismo *scope* definen el mismo tipo, por lo que dentro del mismo *scope*, dos tipos distintos no pueden utilizar la misma etiqueta.

Aunque se declaren estos tipos (declaración de la etiqueta), son tipos incompletos hasta que se definen, por lo que no se pueden usar en declaraciones hasta que no sean completos.

Cada declaración de estructura, unión o enumeración sin etiqueta define un tipo distinto.

##### 6.7.2.4 Atomic type specifiers

Opcional, según implementación. Es del tipo `_Atomic (<tipo>)`, y especifica un tipo atómico. Si tras `_Atomic` no sigue un paréntesis izquierdo, no es un *type specifier*, sino un *type qualifier*. El tipo indicado no puede ser un tipo *array*, tipo función, tipo atómico o tipo *qualified*.

`_Atomic(<tipo>)` equivale a `_Atomic <tipo>`.

Podemos necesitar la versión *qualifier*:

```c
_Atomic(int) * _Atomic p;
```

El *qualifier* se refiere al tipo del apuntador, mientras que el *specifier* se refiere al tipo apuntado (ver 6.7.6.1).

#### 6.7.3 Type qualifiers

Son `const`, `restrict`, `volatile` y `_Atomic`.

Las propiedades asociadas a los *qualifiers* solo tienen sentido en expresiones que son *lvalues*.

El *qualifier* `const` crea tipos de solo lectura. No es posible cambiar el valor de una variable de este tipo, más que en su inicialización.

`restrict` se puede incluir solo en apuntadores a tipos de objeto. `_Atomic` no puede incluirse en tipos *array* o función.

Si se intenta modificar un objeto de tipo cualificado con `const` a través de un *lvalue* con tipo no cualificado con `const`, el comportamiento es indefinido, al igual que el acceso a un objeto de tipo cualificado con `volatile` a través de un *lvalue* con tipo no cualificado con `volatile`.

Un objeto que tiene cualificación `volatile` puede ser modificado por mecanismos desconocidos para la implementación.

Un apuntador cualificado con `restrict` indica que **todos** los accesos realizados al objeto al que referencia este deben ser realizados a través de ese apuntador específico, directa o indirectamente.

Al igual que el *storage class specifier* `register`, el uso de `restrict` no cambia en absoluto el comportamiento del programa, sino que solo sirve para facilitar ciertas optimizaciones.

Si especificamos un *qualifier* para un *array*, los cualificados serán los elementos, no el *array*. Una función no se puede cualificar.

Para que dos tipos cualificados sean compatibles deben ser versiones idénticamente cualificadas de tipos compatibles.

Ejemplos:

```c
const struct s { int mem; } cs = { 1 };
struct s ncs; // el objeto ncs es modificable
typedef int A[2][3];
const A a = {{4, 5, 6}, {7, 8, 9}}; // array de arrays de const int
int *pi;
const int *pci;

ncs = cs;    // válido
cs = ncs;    // rompe la restricción de lvalue modificable para =
pi = &ncs.mem;    // válido
pi = &cs.mem;    // rompe las restricciones de tipo para =
pci = &cs.mem;    // válido
pi = a[0];    // no válido: a[0] tiene tipo (const int *)
```

#### 6.7.4 Function specifiers

Son `inline` (las llamadas a la función son lo más rápidas posible) y `_Noreturn` (la función no retorna nunca). Solo para funciones, excepto `main()`.

Cualquier función con *linkage* interno puede ser `inline`. Así:

```c
static inline foo() { /*...*/ }
```

Define una función *inline* que se puede utilizar en la presente *translation unit*.

En cuanto a una función *inline* con *linkage* externo, la cosa se complica un poco. Cuando **todas** las declaraciones **con *file scope*** de la función se hacen **con el especificador `inline`, y sin `extern`**, si en la *translation unit* se define la función, **no se generará una definición externa** de la misma, sino una ***inline definition*** de esta. Así, aunque tenga *linkage* externo y esté definida, no se podrá utilizar esa definición desde otra *translation unit*. Si se produce una llamada a la función en la presente *translation unit*, la implementación tiene dos alternativas: decantarse por la definición *inline* (definida en la misma *translation unit*), o por la definición externa, definida en otra *translation unit* (no olvidemos que es una función con external *linkage*).

Por lo tanto, en caso de que se produzca una llamada a la función en esta *translation unit*, debería estar definida **también** la versión externa, ya que no sabemos por cuál se decantará la implementación. Si no hay llamada a la función, no hacen falta esas definiciones. ¿Pero cuál será la elegida? En el caso del compilador *GCC*, por ejemplo, depende de los parámetros de optimización que le indiquemos (sin optimización se usará siempre la definición externa).

Por otro lado, y siguiendo con las funciones con *linkage* externo, si en alguna de las declaraciones se incluye `extern` y/o se omite `inline`, la definición que realicemos en la presente *translation unit* sí es una definición externa, que podrá ser llamada desde otras *translation units* (y por la actual).

Para que una función pueda ser *inlined* debe tener una *inline definition*, no una externa. Eso se consigue, como hemos dicho, o bien con una función con *linkage* externo en la que todas sus declaraciones son `inline` y ninguna `extern`, o bien con una función *inline* con *linkage* interno (con `static`). En este último caso, la función podrá ser llamada desde la *translation unit* actual, sin tener que definir otra versión de la función.

Las funciones que residen en otras *translation units* no pueden ser *inlined* en la presente *translation unit* (debe conocer el código, no esperar al *linker*), por lo que deberán ser **definidas** en la actual.

Las bibliotecas con funciones *inline* deben **definir** estas funciones en el *header file* (***.h***), no solamente declararlas, de tal modo que cada *translation unit* disponga de su *inline definition*. Si estas funciones no proporcionan definición externa para el *linker*, esa *inline definition* es simplemente como una **declaración** externa. Sin embargo, para convertir esa declaración externa en definición externa, en el archivo fuente (***.c***) que incluye ese *header file*, habrá que realizar la **declaración** de esa función como `extern inline` (aunque ya no es seguro que sea *inlined*). Esto solo es necesario cuando en el *header file* la función está definida `inline` a secas. Si es `static inline`, ya no hace falta, y además puede ser *inlined*.

Para terminar, la definición de una **función *inline* con *linkage* externo que genere una *inline definition*** no puede contener la definición de un objeto con *storage duration static* o *thread*, ni una referencia a un identificador con *linkage* interno.

#### 6.7.5 Alignment specifier

Especifica los requerimientos de alineación del objeto. Es de la forma `_Alignas(<tipo>)` o `_Alignas(<expresión constante>)`, donde la expresión constante es un entero que coincida con una alineación fundamental válida, o una alineación extendida soportada por la implementación, o cero (sin efecto). No puede ser menos estricto que el *alignment* que le tocaría al tipo.

`_Alignas(<tipo>)` equivale a `_Alignas(_Alignof(<tipo>))`.

No se puede especificar en `typedef`, *bit-field* o función, o conjuntamente con `register`.

Si la definición de un objeto tiene este especificador, las declaraciones del mismo deben tenerla igual, o no tenerla. Si no lo tiene, las declaraciones tampoco.

#### 6.7.6 Declarators

Veremos aquí el formato de una declaración.

Tras la serie de *declaration specifiers* (a la que llamaremos `T`) de la declaración, sigue la lista opcional de uno o más *declarators*, separados por comas. Cada *declarator* declara un identificador, y opcionalmente amplía la información sobre el tipo asociado a ese identificador, en cuanto a tres posibles aspectos: *array*, apuntador o función; o una combinación de las tres cosas, permitiéndose el anidamiento de *declarators* a voluntad (la implementación puede limitar el nivel de anidamiento). En esta composición anidada de tipos complejos, se pueden usar paréntesis para asociar (*bind*) los símbolos adecuadamente en *declarators* complejos.

Un *full declarator* es un *declarator* que no forma parte de ningún otro *declarator*. Si un *full declarator* contiene en su jerarquía de *declarators* uno que defina un tipo *variable length array*, ese *full declarator* define un tipo variablemente modificado (*variably modified*). Cualquier tipo que derive (mediante derivación por *declarators*) de un tipo variablemente modificado, es a su vez un tipo variablemente modificado.

Así, la declaración puede ser simplemente un tipo `T` con todos sus *specifiers* (p.e. `static const int`). O puede ser un tipo más un *declarator* `T D` (p.e. `static const int * p`) para declarar un objeto. O también puede ser un tipo con una lista de *declarators* separados por comas, del tipo `T D1,D2,...`, en cuyo caso estamos declarando varios objetos:

```c
static const int * p, * const q, a[10], n;
```

Cada *declarator* puede declarar un identificador, y opcionalmente amplía la información sobre el tipo del objeto. Los *declarators* pueden estar anidados. Un *full declarator* es un *declarator* que no forma parte de otro *declarator*. Si no declara identificador, no está declarando un objeto, sino un tipo.

Un *declarator* entre puede ir entre paréntesis o no, sin que cambie su sentido.

El ***declarator* más simple** es un simple **identificador** (o nada):

```c
static const int n;
```

Hay otros tres tipos de *declarators* (apuntador, *array* y función).

##### 6.7.6.1 Pointer declarators

Cuando el *declarator* es para un apuntador, su sintaxis es:

```
* [qualifiers] identificador
```

Los *type qualifiers* especificados aquí se aplican al apuntador que estamos declarando; en cambio, los especificados en el nivel más alto (junto a los *specifiers*, fuera del *declarator*) se aplican al tipo apuntado.

Para que 2 apuntadores sean compatibles deben ser idénticamente cualificados y apuntar a tipos compatibles.

Ejemplos:

```c
const int *ptr2ct;  // apuntador a entero constante
int * const ct_ptr;  // apuntador constante a entero
typedef int *int_ptr;
const int_ptr ct_ptr;  // apuntador constante a entero
```

##### 6.7.6.2 Array declarators

El elemento de un *array* no puede ser de un tipo incompleto ni tipo función.

En cuanto a la sintaxis del *declarator*, consiste en el identificador, tras el cual habrá un par de corchetes ***\[\]***, con toda la información. Los corchetes pueden estar vacíos.

También pueden contener una expresión (obligatoriamente entera) que definirá el tamaño del *array*. Si es una expresión constante, será mayor que cero.

En el caso específico de un *array* declarado en la lista de parámetros de una función, los corchetes pueden albergar al principio *type qualifiers* y la palabra `static` (explicado en 6.7.6.3).

Si declaramos un tipo variablemente modificado, el identificador será un identificador ordinario (6.2.3), no tendrá *linkage* y tendrá *block scope* o *function prototype scope*. Un objeto con *static* o *thread storage class* no podrá ser de tipo *variable length array*.

Si no se indica el tamaño del *array*, será un tipo incompleto. Si en lugar de una expresión, el tamaño se indica con ***\**** (solo en declaraciones o en *function prototype scope*), es que se trata de un *variable-length array* de tamaño no especificado, y sin embargo se trata de un tipo completo.

Si la expresión entera es constante y se conoce el tamaño del elemento, no se trata de un *variable length array*. De lo contrario sí.

Si la expresión de tamaño no es constante: Si ocurre en *prototype scope*, se trata como si se reemplazara por ***\****; de lo contrario, cada vez que se evalúe deberá tener un valor superior a cero. El tamaño de una instancia de *variable-length array* se define en tiempo de ejecución, pero una vez hecho no varía a lo largo de todo su tiempo de vida.

Para que dos tipos *array* sean compatibles, sus elementos deben ser de tipos compatibles, y si el tamaño de ambos está definido, estos deben ser iguales.

```c
extern int *x;
extern int y[];
```

Son dos tipos distintos: el primero es un apuntador a entero, mientras que el segundo es un *array* de tamaño no especificado (un tipo incompleto). Pero son compatibles, con lo que se puede hacer `x = y`.

Veamos un ejemplo de compatibilidad entre *variably modified types* (*VM*):

```c
extern int n;
extern int m;

void fcompat(void)
{
    int a[n][6][m];
    int (*p)[4][n+1];
    int c[n][n][6][m];
    int (*r)[n][n][n+1];
    p = a;    // inválido: incompatible porque 4 != 6
    r = c;    // compatible, pero defined behavior solo si
              // (n == 6) y (m == n+1)
}
```

Aquí, ***p*** es un apuntador a *array* `int[4][n+1]`. En cambio, ***a*** es un *array* cuyos (***n***) elementos son de tipo `int[6][m]`. Por eso `p = a` no es válido.

Ejemplo: podemos declarar un *array* de tamaño no especificado:

```c
int a[];
```

Esto no se puede hacer si el identificador no tiene *linkage* (la declaración es la definición, y no se puede definir un tipo incompleto). Pero hay algunas declaraciones que no son legales:

```c
int a[][];  // error!
int a[5][];  // error!
int a[][5];  // OK
```

En el primer caso, no está permitido, porque estamos declarando un *array* de tamaño no especificado cuyos elementos son del tipo `int[]`, el cual es, a su vez, un *array* de tamaño no especificado, el cual es un tipo incompleto, y recordemos que no podemos declarar un *array* de elementos con tipo incompleto.

El segundo caso es similar: declaración de un *array* de 5 elementos `int[]`, es decir, de 5 elementos de tipo incompleto.

Sin embargo, el tercer caso es correcto, ya que estamos declarando un *array* de tamaño sin especificar, cuyos elementos son `int[5]`, es decir, elementos de un tipo completo.

Resumiendo, **solo el primero de los índices de un *array* multidimensional puede estar en blanco en la declaración**.

##### 6.7.6.3 Function declarators (including prototypes)

Es del tipo `T D(lista-parms)`, donde la lista de parámetros especifica los tipos, y puede declarar los identificadores de los parámetros. Puede formar parte de la definición de la función, o ser simplemente un prototipo (declaración de la función).

No puede definirse un tipo de retorno *array* o función. El único *storage-class specifier* posible para los parámetros es `register`.

Los tipos de los parámetros en una definición no pueden ser incompletos.

La declaración de un parámetro de tipo *array* se ajustará a un tipo apuntador cualificado según los *qualifiers* indicados dentro de los corchetes ***[]***. Si dentro de estos también aparece la palabra `static`, entonces en cada llamada a la función, se pasará un *array* que contendrá, por lo menos, el número de elementos indicados en la expresión de tamaño de la declaración (útil para optimizaciones).

Del mismo modo, un parámetro de tipo *función que retorna tipo **T*** será convertido a *apuntador a función que retorna tipo **T***.

Añadir tres puntos (`...`) al final de la lista de parámetros indica número variable de parámetros (0 o más parámetros, de cualquier tipo).

Si la lista tiene un único parámetro sin nombre de tipo `void`, significa que la función no tiene parámetros. Una lista vacía en una definición de función indica también que la función no tiene parámetros, pero una lista vacía en una declaración indica que no se proporciona información sobre los parámetros en ese prototipo.

Si en la declaración de parámetros un identificador puede interpretarse como nombre de `typedef` o como nombre del parámetro, se considerará nombre de `typedef`.

Si el *declarator* de la función no forma parte de la definición de la función (es decir, es un prototipo), los parámetros pueden tener tipo incompleto, y usar la notación `[*]` para designar *variable-length arrays*.

El *storage class specifier* de los parámetros es ignorado en la declaración de una función, pero no en la definición de la misma.

En la declaración de la función, el nombre de los parámetros no es obligatorio (no se usa para nada).

Para que dos tipos de función sean compatibles deben tener tipos de retorno compatibles. Además, si las listas de parámetros de ambas funciones están descritas, deben coincidir en el número de parámetros, en uso de la *ellipsis* (`...`), y ser compatibles los tipos uno a uno tras el ajuste de tipos (conversión de tipos *array* y función a apuntadores).

No se pueden anidar funciones. No se puede definir una función dentro de otra, pero sí puede declararse una función dentro de otra.

Ejemplos de declaración de funciones:

```c
int fun(float a[]);
int fun(float restrict *a);
int fun(float a[restrict]);
int fun(float a[restrict static 5]);
```

En el último ejemplo, se convierte a:

```c
int fun(float * restrict a);
```

En este caso, pasarle como parámetro un *array* con menos de 5 elementos tendrá un comportamiento desconocido.

Veamos este ejemplo:

```c
int f(void), *fip(), (*pfi)();
```

Declara primero una función ***f*** sin parámetros que devuelve `int`; una función ***fip*** sin especificar los parámetros, que devuelve `*int`; y un apuntador ***pfi*** a una función que devuelve `int`, sin especificar parámetros.

```c
int (*apfi[3])(int *x, int *y);
```

Declara un *array* ***apfi*** de tres apuntadores a una función que devuelve `int`. Cada una de estas funciones tiene dos parámetros `*int` (los nombres ***x*** e ***y*** son meramente descriptivos).

```c
int (*fpfi(int (*)(long), int))(int,...);
```

Declara una función ***fpfi*** que devuelve un apuntador a una función que devuelve `int`. La función ***fpfi*** tiene dos parámetros: un apuntador a función que devuelve `int` (con un parámetro `long int`) y un `int`. El apuntador devuelto por ***fpfi*** apunta a una función que tiene un parámetro `int` y acepta argumentos adicionales.

#### 6.7.7 Type names

A veces debemos indicar el nombre de un tipo. Esto se consigue del mismo modo que en una declaración, pero omitiendo el identificador. Ejemplos:

- `int` es un tipo `int`.
- `int *` es un tipo apuntador a `int`.
- `int *[3]` es un tipo *array* de 3 apuntadores a `int`.
- `int (*)[3]` es un tipo apuntador a un *array* de 3 `int`s.
- `int (*)[*]` es un tipo apuntador a un *variable length array* de un número no especificado de `int`s.
- `int *()` es un tipo función de parámetros no especificados que retorna un apuntador a `int`.
- `int (*)(void)` es un tipo apuntador a una función sin parámetros que retorna un `int`.
- `int (*const []) (unsigned int, ...)` es un tipo *array* de un número no especificado de apuntadores constantes a funciones, cada una de las cuales toma un parámetro `unsigned int` más un número variable de otros parámetros y retorna un `int`.

#### 6.7.8 Type definitions

La sintaxis para definir un tipo es la misma que la de una declaración, con la diferencia que debe empezar por `typedef` (como si fuese un *storage class specifier*), y los nombres indicados, en lugar de ser identificadores de variables, pasan a ser alias del tipo especificado.

Si el tipo define un *variably modified type*, el tipo tendrá *block scope*. La expresión que define la longitud del *array* se evalúa cuando se define el tipo.

Ejemplos: `typedef float i1, i2(), *pi1;` define ***i1*** como `float`, ***i2*** como función sin parámetros definidos que devuelve `float` (si hacemos `i2 *p;` tendremos que ***p*** será apuntador a función sin parámetros definidos que devuelve `float`), y ***pi1*** como apuntador a `float`.

Si hacemos `typedef void fv(int), (*pfv)(int);` entonces las tres declaraciones más abajo son equivalentes, demostrando la utilidad de `typedef` para aumentar la legibilidad del código:

```c
void (*signal(int, void (*)(int)))(int);
fv *signal(int, fv *);
pfv signal(int, pfv);
```

#### 6.7.9 Initialization

En la declaración de un objeto podemos darle un valor inicial. Se pueden inicializar los objetos que tengan un tipo completo (excepto *variable-length arrays*) y *arrays* de tamaño desconocido.

Todas las expresiones del *initializer* (valor o conjunto de valores iniciales a asignar) de un objeto con *storage duration static* o *thread*, deben ser expresiones constantes o literales *string*.

Si el identificador tiene *block scope* y *linkage* interno o externo, no se puede inicializar.

Los miembros sin nombre de estructuras y uniones no participan de la inicialización, y terminan con valores indefinidos (donde no se indique lo contrario).

Si no se inicializan explícitamente, los objetos con *automatic storage duration* tienen valor inicial indefinido. En cambio, con *static* o *thread storage duration*:

- Los objetos de tipos aritméticos se inicializan a ***0***.
- Los apuntadores se inicializan a apuntador nulo.
- Los *arrays* y estructuras inicializan sus elementos y miembros del modo indicado (recursivamente) y todos los *paddings* quedan a ***0***.
- Las uniones inicializan el primer miembro del mismo modo (recursivamente) y todos los *paddings* quedan a ***0***.

El inicializador de un escalar será una expresión simple (opcionalmente entre llaves).

La inicialización de una estructura o unión con *automatic storage duration* se puede hacer mediante una expresión cuyo valor sea un objeto de un tipo compatible con esa estructura o unión.

Un *array* de tipo carácter puede inicializarse con un literal *string* de caracteres o literal *string  UTF-8*, opcionalmente entre llaves. Los caracteres del *string* van dando valor a los elementos del *array* (incluyendo el carácter nulo final), mientras queden elementos por inicializar.

Esto se aplica del mismo modo en la inicialización de un *array* de un tipo compatible con `wchar_t`, `char32_t` o `char64_t`, utilizando el respectivo prefijo de literal *string wide*.

La inicialización en tipos *aggregate* y unión se puede hacer también mediante una lista de inicialización entre llaves. Los elementos se van inicializando en el orden indicado, pero es posible adelantar o atrasar el elemento a inicializar (*current object*), mediante *designators*. En un *array* estos son del tipo `[exp] = v`, donde ***exp*** es una expresión constante (en el caso de un *array* de tamaño desconocido, puede tener cualquier valor no negativo), y en una estructura o unión es del tipo `.miembro = v`. En ambos casos, ***v*** es el valor que se dará al elemento o miembro. Si el siguiente elemento en la lista de inicialización no tiene *designator*, inicializará al elemento o miembro siguiente, en orden, al que hemos dado valor.

Si no se especifican *designators*, la inicialización se va haciendo en orden: en *arrays* según índice, en estructuras según el orden de declaración, y en uniones el primer elemento.

Los subobjetos se pueden inicializar mediante listas de inicialización entre llaves anidadas. Si la lista de inicialización tiene demasiados elementos para el objeto a inicializar, solo se usarán los necesarios. Si una sublista tiene demasiados elementos para el subobjeto, solo se usarán los necesarios, y el resto se usará para inicializar el siguiente subobjeto (si lo hay). Si hay demasiado pocos elementos en la lista de inicialización, los que no se hayan podido inicializar se inicializarán como en el caso de los objetos con *static storage duration*.

Por lo dicho, una estructura con una jerarquía concreta de miembros podría inicializarse sin necesidad de jerarquizar la lista de inicialización.

Si se inicializa un *array* de tamaño desconocido, su tamaño será determinado por la cantidad de elementos en el inicializador (lista entre llaves o *string* incluyendo carácter nulo final), teniendo en cuenta la acción de posibles *designators*.

La evaluación de los elementos de la lista de inicialización no tiene *sequence points* definidos.

Ejemplos:

```c
int i=3;  // podría ser int i={3};
int x[]={1,2,3};  // simple array
int y[4][3] = {
    {1, 3, 5},
    {2, 4, 6},
    {3, 5, 7},
};
```

El último inicializa desde ***y[0]*** a ***y[2]***, mientras ***y[3]*** queda a cero.

`int y[4][3] = {1,3,5,2,4,6,3,5,7};` es idéntico al anterior.

`int y[4][3] = {1,2,3};` inicializa la primera fila (resto ceros).

`int y[4][3] = {{1}, {2}, {3}};` inicializa la primera columna, o sea, el primer elemento de cada fila (resto ceros).

`char s[]="abc";` equivale a `char s[]={'a', 'b', 'c', '\0'};`

`char s[2]="abc";` equivale a `char s[]={'a', 'b'};`

`char *s="abc";` inicializa esa zona de memoria apuntada por ***s*** con 4 caracteres (*null-terminated*). No se debe modificar el contenido del *array* (comportamiento indefinido).

Ejemplos de *designators*:

`int a[MAX]={1,3,5,7,9,[MAX-5]=8,6,4,2,0};` inicializa ambos externos del *array*; si la constante ***MAX*** es menor a 10, la segunda parte sobrescribe parte de la primera.

`struct {int a[3],b;} w = { .a={1,3,6}, .b=3 };`

```c
struct {int a[3],b;} w[] = {
    [0].a={1,3,6}, [0].b=3,
    [1].a={6,8,4}, [1].b=5
};
```

#### 6.7.10 Static assertions

Son de este tipo:

```
_Static_assert(<expresión>, <literal string>);
```

La expresión debe ser constante y entera. Si su valor es ***0***, la implementación producirá un mensaje diagnóstico durante la compilación, que incluirá el mensaje indicado en el literal *string*.

La implementación no está obligada a mostrar los caracteres del literal *string* que no pertenezcan al *basic source character set* en caso de que se muestre tal mensaje diagnóstico.

### 6.8 Statements and blocks

Excepto cuando se indique lo contrario, las sentencias se ejecutan en secuencia. Cada sentencia simple debe terminar en `;`.

#### 6.8.1 Labeled statements

Las etiquetas son identificadores únicos dentro de una función. Son del tipo:

`etiq:`, `case <expr>:` o `default:`, donde ***expr*** es una expresión constante. Tras una etiqueta siempre debe ir una sentencia. Los dos últimos casos son para la sentencia `switch`.

#### 6.8.2 Compound statement

Es un bloque formado por un conjunto de sentencias y/o declaraciones encerrados entre llaves `{}`. Puede ser vacío, y son anidables.

#### 6.8.3 Expression and null statements

Una *expression statement* es una expresión cuyo valor se evalúa como `void`, pues solo nos interesan sus *side effects*. Por ejemplo `(void)foo();`.

Un *null statement* es un simple punto y coma `;`. Se puede usar, por ejemplo, para bucles vacíos, o para poner una etiqueta al final de un bloque.

#### 6.8.4 Selection statements

Selecciona entre *statements* en función de una expresión. Un *selection statement* es un bloque con un *scope* anidado dentro del *scope* en el que está definido. Cada *statement* que forma parte del *selection statement* forma un *scope* dentro de este.

##### 6.8.4.1 The if statement

```
if (<expression>) <statement>
if (<expression>) <statement> else <statement>
```

La expresión será de tipo escalar. El primer *statement* se ejecuta si la expresión se evalúa distinta de ***0***. En la forma con `else`, si la expresión resulta ***0***, se ejecuta el segundo *statement*.

En esa forma con `else`, el código al final del primer *statement* ejecuta un salto más allá del segundo *statement*, con lo que si se llega al primer *statement* por un label de `goto`, tampoco se ejecutará el *statement* del `else`.

Un `else` se asocia al `if` mas próximo posible.

##### 6.8.4.2 The switch statement

```
switch (<expression>) <statement>
```

La expresión será de tipo entero.

En el cuerpo del `switch` (*statement*) se pueden incluir todas las etiquetas `case` que permita la implementación, y hasta una etiqueta `default`.

Si alguna de las etiquetas de la sentencia `switch` queda dentro del *scope* de un identificador con un tipo variablemente modificado, todo el bloque `switch` debe estarlo. Es decir, la declaración de ese identificador podrá estar fuera del `switch`, o si está dentro, debe estar después de la última etiqueta (sea del tipo que sea).

Cada etiqueta `case` irá acompañada de una expresión entera constante, y no pueden repetirse los valores dentro del mismo `switch` (otra cosa son los `switch` anidados).

Al entrar en el cuerpo del `switch`, la ejecución salta directamente a la etiqueta `case` cuyo valor coincida con la expresión de control. Si no existe ninguna etiqueta que coincida, saltará a la etiqueta `default`. Si tampoco existe esta, ninguna parte del cuerpo del `switch` será ejecutado.

Una vez se haya producido el salto a la etiqueta correspondiente, el código se ejecutará hasta el final del bloque, independientemente de las otras etiquetas, con lo que si queremos que solo se ejecute el código correspondiente a la etiqueta donde hemos saltado, habrá que incluir una sentencia `break` antes de la siguiente etiqueta `case`.

#### 6.8.5 Iteration statements

```
while (<expressionC>) <statement>
do <statement> while (<expressionC>);
for ([<expression>]; [<expressionC>] ; [<expression>]) <statement>
for (<declaration>; [<expressionC>]; [<expression>]) <statement>
```

El ***statement*** se denomina aquí **cuerpo del bucle**.

Las expresiones de control (***expressionC***) serán de tipo escalar.

Los objetos declarados en el `for` solo pueden tener *storage class* `auto` o `register`.

Las sentencias de iteración repiten el cuerpo del bucle (***statement***) mientras la expresión de control (***expressionC***) se evalúe distinto de ***0***. La iteración se produce aunque hayamos entrado en el bucle desde un salto a una etiqueta.

La sentencia de iteración forma un *scope* incluido en el bloque que la contiene, y el cuerpo del bucle es un *scope* contenido dentro del de la sentencia de iteración.

##### 6.8.5.1 The while statement

La evaluación de la expresión de control se produce antes de cada ejecución del cuerpo del bucle.

##### 6.8.5.2 The do statement

La evaluación de la expresión de control se produce después de cada ejecución del cuerpo del bucle.

##### 6.8.5.3 The for statement

La primera expresión (o declaración) se ejecuta solo la primera vez. Si es una declaración, el *scope* de los identificadores es el *iteration statement* entero (es decir, el resto de la declaración, las otras dos expresiones y el cuerpo entero del bucle).

La segunda expresión es la expresión de control del bucle, que se evalúa antes de cada ejecución del cuerpo del bucle.

La última expresión se evalúa como una expresión `void` después de cada iteración.

Se pueden omitir todas las expresiones; si omitimos la segunda, se sustituye por una constante distinta de cero.

#### 6.8.6 Jump statements

```
goto <identifier>;
continue;
break;
return [<expression>];
```

Realizan un salto incondicional.

##### 6.8.6.1 The goto statement

Salta a una etiqueta, definida en la función actual.

No se puede realizar un salto dentro del *scope* de un tipo variablemente modificado desde fuera de ese *scope* (sí se puede de dentro a fuera).

##### 6.8.6.2 The continue statement

Dentro del cuerpo de un **bucle** (el más interior al que pertenece) salta directamente al final del mismo (itera).

##### 6.8.6.2 The break statement

Dentro del cuerpo de un **bucle** o `switch` (el más interior al que pertenece) termina su ejecución.

##### 6.8.6.4 The return statement

Retorna de una función; la expresión debe indicarse si y solo si la función no es de tipo `void`. Si es de ese tipo no debe indicarse tal expresión.

### 6.9 External definitions

Los *storage-class specifiers* `auto` y `register` no pueden aparecer en declaraciones externas.

Las **declaraciones externas** son las que aparecen fuera de cualquier función, por lo que tienen *file scope*. Las declaraciones que causan reserva de *storage* para un objeto o función son definiciones. Una **definición externa** es una declaración externa que es también definición.

Dentro de una *translation unit* no puede haber más de una definición externa de un identificador con *linkage* interno. Además, si el identificador es utilizado en alguna expresión, será obligatoria su definición externa en la *translation unit*.

Dentro del programa entero no puede haber más de una definición externa de un identificador con *linkage* externo. Además, si el identificador es utilizado en alguna expresión, será obligatoria su definición externa en alguna parte del programa.

#### 6.9.1 Function definitions

El tipo de retorno de la función será `void` o un tipo completo excepto *array*.

El *storage-class specifier*, si lo hay, será `extern` o `static`.

Si hay lista de tipos de parámetros, se deben incluir los nombres (identificadores) de los parámetros (al contrario que en la declaración), excepto en el caso de un solo tipo `void`, en cuyo caso no puede haber identificador.

Un nombre de tipo definido con `typedef` no puede redeclararse como parámetro en la **definición** de una función. Las declaraciones en la lista de parámetros no pueden contener *storage-class specifiers*, excepto `register`. Tampoco pueden contener inicializaciones.

Los parámetros de tipo *array* o función se ajustarán según lo dicho en 6.7.6.3. Los parámetros tendrán (tras el ajuste, si procede) tipos completos.

Si la función acepta un número variable de parámetros, debe definirse con los tres puntos (`...`) al final de la lista de parámetros.

El *declarator* de la definición también sirve como declaración (prototipo) de la misma, con lo que en la *translation unit* donde está esta definición no es preciso declararla para poder usarla (si se quiere usar **antes** de la definición sí que habrá que declararla).

> En este punto, puede que el compilador haga una *implicit function declaration* cuando invocamos una función que no ha sido declarada todavía. Esta declaración implícita es del tipo `int foo()`, es decir, función sin especificar parámetros que devuelve `int`. Esto ya no está permitido desde C99.

Los parámetros tienen *automatic storage duration*, y sus identificadores son *lvalues*.

A la entrada a la función (tras una llamada), los tamaños de los parámetros de tipo *variable length array* son calculados, y los valores de los parámetros son convertidos, como en cualquier asignación. Después se ejecuta el cuerpo (*compound statement*) de la función.

La función:

```c
int max(int a, int b) {
    return a * b;
}
```

Se puede definir asi:

```c
int max(a, b)
int a, b; {
    return a * b;
}
```

Para pasar una función como parámetro a otra función:

```c
int f(void);
g(f);
```

La definición de ***g*** debería ser:

```c
void g(int (*funcp)(void))
{
    /* ... */
    (*funcp)();  // o funcp();
}
```

Dado que los argumentos de tipo *array* y función se convierten a apuntador antes de la llamada, esto es equivalente:

```c
void g(int func(void))
{
    /* ... */
    func(); //  o (*func)();
}
```

#### 6.9.2 External object definitions

Si la declaración de un objeto con *file scope* tiene inicializador, es una *external definition*.

En cambio, si esa declaración no tiene inicializador y además tampoco tiene *storage-class specifier* (o como mucho tiene el especificador `static`), es una *tentative definition*.

Si una *translation unit* contiene una *tentative definition* o más de un identificador, y no contiene ninguna definición externa de ese identificador, el comportamiento será como si la *translation unit* contuviera una declaración del identificador inicializado con valor ***0*** y con el tipo compuesto correspondiente. Ese tipo compuesto no puede ser incompleto cuando el identificador tiene *linkage* interno.

Ejemplo:

```c
int i1 = 1;            // definition, external linkage
static int i2 = 2;     // definition, internal linkage
extern int i3 = 3;     // definition, external linkage
int i4;                // tentative definition, external linkage
static int i5;         // tentative definition, internal linkage
int i1;                // tentative definition, refers to previous
int i2;                // undefined, linkage disagreement
int i3;                // tentative definition, refers to previous
int i4;                // tentative definition, refers to previous
int i5;                // undefined, linkage disagreement
extern int i1;         // refers to previous => external linkage
extern int i2;         // refers to previous => internal linkage
extern int i3;         // refers to previous => external linkage
extern int i4;         // refers to previous => external linkage
extern int i5;         // refers to previous => internal linkage
```

En este ejemplo hay una línea que, aunque pertenezca a otro tema, merece atención:

```c
extern int i3 = 3;
```

El código es perfectamente legal, y sin embargo algunos compiladores (*GCC* entre ellos) lanzarán un *warning* al respecto. Ello es debido a que es código muy malo. El uso para el que está pensado `extern` es para enlazar con objetos definidos en otros archivos o bibliotecas. Normalmente en el *header file* (***.h***) se traerá el objeto al presente *scope* mediante `extern int i3;` pero no se inicializará. En la biblioteca o archivo externo, estará inicializado probablemente en *file scope* así: `int i3 = 3;`.

### 6.10 Preprocessing directives

Para la fase 4 de traducción. Una directiva de preproceso empieza con el *preprocessing token* `#` **a principio de línea** (admite espacio blanco antes) y termina con un *newline*, de tal modo que cada directiva ocupa una sola línea (no hay *newlines* dentro de la directiva, ni siquiera en la definición de macros).

Los únicos caracteres de espacio blanco que puede haber entre los *preprocessing tokens* de la directiva son el espacio y el tabulador horizontal (aunque se puede usar la barra invertida  más *newline* para cortar la línea, pues eso se procesa en una fase anterior). Se pueden intercalar espacios (o comentarios, que son convertidos en espacio) entre todos los *preprocessing tokens* (así, por ejemplo, `# directiva` equivale a `#directiva`).

Si la expansión de una macro diera lugar a una directiva de preproceso en una línea que antes de esa expansión no era tal directiva, la directiva resultante no será procesada.

#### 6.10.1 Conditional inclusion

La expresión que controla la inclusión condicional debe ser una constante entera, y puede incluir el operador unario `defined`:

```
defined <identifier>
defined (<identifier>)
```

Este operador es una expresión que se evalúa a ***1*** si el identificador está definido (mediante `#define`, o predefinido por la implementación, sin que se haya producido `#undef`), o ***0*** en caso contrario.

La directiva de inclusión condicional es `#if`:

```
#if <expression>
```

Antes de la evaluación de la directiva, se realizan las evaluaciones de las expresiones `defined` y las expansiones de macros que intervendrán en las expresiones de control de la directiva. Después de estas expansiones, no deberían quedar nombres de identiﬁcadores. Si quedan, se sustituyen por ***0***. Los números enteros resultantes serán tratados como si tuvieran tipo `intmax_t` (con signo) o `uintmax_t` (sin signo), tipos definidos en ***stdint.h***.

Por otro lado, las directivas:

```
#ifdef <identificador>
#ifndef <identificador>
```

equivalen respectivamente a:

```
#if defined <identificador>
#if !defined <identificador>
```

Las directivas `#if`, `#ifdef` y `#ifndef` controlan un grupo de líneas de código que se halla justo debajo de ellas. El grupo puede incluir código C, y/o directivas de preproceso de todo tipo, incluyendo directivas de inclusión condicional anidadas. Si la expresión evalúa a ***0*** se ignora ese código. Si es ***1*** se incluye.

Tras una directiva de este tipo, puede haber una o más directivas `#elif`:

```
#elif <expression>
```

Cada una de ellas irá con su correspondiente grupo de líneas de código. Tras estas, puede haber opcionalmente una directiva `#else`. Al final de todo, debe terminar todo con una directiva `#endif`.

Para cada directiva `#if`, `#ifdef` o `#ifndef` puede haber cero o más directivas `#elif`, una directiva `#else` opcional, y al final de todo una directiva `#endif`.

Del grupo de directivas formado por la primera más todas las `#elif`, se incluirá el grupo de código de la primera cuya expresión evalúe a ***1***, mientras que los demás grupos serán ignorados. Si ninguna evalúa a ***1***, no se incluirá ninguno de ellos, y en ese caso, si existe una directiva `#else`, será el grupo de esta el que se incluirá.

#### 6.10.2 Source file inclusion

```
#include <filename>
#include "filename"
```

Se sustituye la directiva por el contenido completo del archivo (archivo de cabecera). Al usar `<>`, la búsqueda se hace en lugares definidos por la implementación, normalmente los directorios estándar del sistema, donde residen dichos *header files*. Con `""` se busca en otro lugar establecido por la implementación (normalmente el directorio local del archivo fuente), de tal modo que si no lo encuentra allí o falla esa búsqueda, lo hace como si se hubiera especificado `<>`.

#### 6.10.3 Macro replacement

Macro tipo objeto:

```
#define <identifier> <replacement-list>
```

Cada aparición del identificador ***identifier*** es expandida (sustituida) por la lista de reemplazo ***replacement-list***, que es una secuencia de *tokens* de preproceso.

Dentro de literales *string* y constantes carácter no se realiza sustitución de macros.

Macros tipo función (el paréntesis izquierdo no lleva espacio antes):

```
#define <identifier>( [<identifier-list>] ) <replacement-list>
#define <identifier>( ... ) <replacement-list>
#define <identifier>( [<identifier-list>], ...) <replacement-list>
```

Los parámetros se indican mediante la lista de identificadores ***identifier-list*** opcional. No se puede repetir un identificador en la lista. Los parámetros de la macro tienen *scope* desde que son declarados en la lista de identificadores hasta el *newline* que termina la directiva.

En la invocación de una macro tipo función, el número de parámetros debe coincidir exactamente con el de su definición. En cambio, si la definición termina en puntos suspensivos (`...`), el número de parámetros en la llamada debe ser superior al numero de parámetros de la definición (sin contar los puntos suspensivos).

Dentro de la *replacement list* de una macro con puntos suspensivos en la lista de parámetros (macro variádica) podemos usar ***\_\_VA_ARGS\_\_***, que agrupa el resto de parámetros, incluyendo las comas entre ellos.

La llamada a una macro tipo función se expresa sintácticamente como la llamada a una función. En la llamada puede haber espacio entre el nombre de la macro y el paréntesis izquierdo de la lista de parámetros. Cada llamada a la macro es sustituida por la lista de reemplazo ***replacement-list***, y los parámetros son sustituidos, como se verá.

La **invocación** de una macro tipo función se puede fragmentar con *newlines*, que son tratados como espacios.

Aunque definamos un nombre de macro sin valor (`#define NOMBRE`), el nombre quedará definido (`defined NOMBRE` se evaluará a ***1***).

##### 6.10.3.1 Argument substitution

Cada parámetro presente en la *replacement list* en la definición de una función macro, es reemplazado por el argumento proporcionado en la llamada, aunque antes de ser sustituido, **cada ARGUMENTO de la llamada es expandido completamente** (recursivamente hasta que no se pueda más), **a no ser que esté precedido por `#` o `##`, o seguido de `##`**, en cuyo caso se sustituye tal cual en la *replacement list*.

##### 6.10.3.2 The # operator

Un **parámetro** precedido de `#` en la definición de la función macro (en la lista de reemplazo), será sustituido (junto con el *token* `#`) por el argumento proporcionado, y convertido en un literal *string*, tal cual (sin que se realice ninguna expansión del argumento):

- Se le añaden dobles comillas al principio y al final.
- A los caracteres ***\\*** y ***"*** se les prefija ***\\***.
- Todas las combinaciones de espacio blanco se convierten en un solo espacio.
- El posible espacio en blanco al principio y al final del *string* se elimina.

```c
#define MESSAGE(x) puts(#x)
MESSAGE(hola "mundo");
```

Se sustituye por:

```c
puts("hola \"mundo\"");
```

##### 6.10.3.3 The ## operator

No se puede usar `##` ni al principio ni al final de la *replacement list*.

Se utiliza para concatenación (no para *strings*) de *tokens* de preproceso (no solo parámetros). Por ejemplo, si tenemos las variables ***var1*** a ***var10*** de tipo estructura, cada una de las cuales tiene los miembros enteros ***m1*** a ***m10***, podemos hacer una macro que incremente en 1 el miembro número ***mn*** de la variable número ***varn***:

```
#define INC(varn, mn) var ## varn . m ## mn++
```

Aquí, la llamada `INC(3, 2);` equivale a `var3.m2++;`.

Si tras la concatenación se forma un nombre de macro, este es candidato a una subsiguiente expansión.

##### 6.10.3.4 Rescanning and further replacement

Una vez se ha realizado la expansión de la macro (de cualquier tipo), incluyendo las acciones de `#` y `##`, al formarse la nueva secuencia de *tokens* de preproceso, se produce un reescaneado del resultado para comprobar si se han formado nuevos nombres que se puedan sustituir. Si lo que aparece, al expandir la macro actual o en un reemplazo anidado, es el nombre de la macro que estamos expandiendo, este no sufrirá más expansiones, para evitar expansiones infinitas.

```c
#define foo (bar + 5)
#define bar (foo * 5)
int n = foo;
```

En este caso, `foo` se sustituye por `(bar + 5)`. Se reescanea esta expansión y vemos que `bar` se sustituye por `(foo * 5)`, quedando pues `((foo * 5) + 5)`. Al reescanear ahora, aparece `foo`, que no se expande por ser el nombre de la macro que estamos procesando. El resto del resultado no contiene nombres de macro, con lo que la línea quedará `int n = ((foo * 5) + 5);`.

Si la expansión de una invocación de macro produce una directiva de preproceso en una línea que no era una directiva de preproceso, dicha directiva, aunque tenga una sintaxis correcta como directiva no será procesada como tal, y se dejará tal cual. Es decir, una expansión macro no puede generar nuevas directivas de preproceso. Hay, sin embargo, un método para generar funcionalidad *pragma* (6.10.9).

##### 6.10.3.5 Scope of macro definitions

Las definiciones de macros no tienen ningún significado después de la fase de traducción 4. Los nombres tienen *scope* desde el punto en que son definidos en la *preprocessing unit* hasta el final de la misma, a no ser que antes se encuentre con una directiva `#undef`, que marcaría el final de su *scope*.

```
#undef <identifier>
```

A partir de este punto, el identificador indicado deja de estar definido, con lo que `defined <identifier>` se evaluaría a ***0***. Si ***identifier*** no está definida previamente, la directiva es simplemente ignorada.

Ejemplos:

```c
#define glue(a, b) a ## b
#define xglue(a, b) glue(a, b)
#define HIGHLOW "hello"
#define LOW LOW ", world"
```

Veamos qué produciría `glue(HIGH, LOW)`: lo primero es que, como tanto ***a*** como ***b*** están junto a un token `##`, no se expanden antes de la llamada. Por tanto, el resultado es `HIGH ## LOW`, es decir, `HIGHLOW`. Ahora se reescanea ese resultado, y vemos que se debe sustituir por `"hello"`.

Veamos ahora `xglue(HIGH, LOW)`: lo primero, como ninguno de los parámetros está junto a ningún token `#` ni `##`, se deben expandir primero. El argumento `HIGH` se queda tal cual, porque no se corresponde con ningún nombre de macro, pero el argumento `LOW` se expande a `LOW ", world"`. Vamos a ver si podemos volver a expandirlo: reescaneamos este resultado: lo primero es `LOW`, que es un nombre de macro, pero como es el mismo nombre de la macro que estamos expandiendo, se queda aquí. Por otro lado, `", world"` no es un nombre de macro, con lo que hemos terminado. Resumiendo, los argumentos `HIGH` y `LOW` de la llamada han sido expandidos a `HIGH` y `LOW ", world"` antes de sustituirlos en la *replacement list*, es decir, `glue(a, b)` queda, tras la sustitución de argumentos, como `glue(HIGH, LOW ", world")`. Ahora, al reescanear, vemos que se corresponde con una función macro, con lo que procedemos a sustituir los argumentos `HIGH` y `LOW ", world"` en `a ## b`. Como están junto a `##`, sustituimos directamente sin expandir, y queda `HIGH ## LOW ", world"`, lo cual nos da `HIGHLOW ", world"`. Reescaneamos nuevamente, y vemos que `HIGHLOW` se sustituye por `"hello"`, siendo el resultado final, `"hello" ", world"`. Estos dos *strings*, se concatenarán automáticamente en la fase 6 de traducción como `"hello, world"`.

#### 6.10.4 Line control

El **número de línea actual**, almacenado en la macro ***\_\_LINE\_\_***, se define como la cantidad de caracteres *newline* leídos en la fase 1 de traducción, más 1, hasta el *token* actual.

```
#line N <num>
```

Establece ***\_\_LINE\_\_*** de tal modo que el número de línea correspondiente a la siguiente línea de código tendrá el valor del número ***num***. No se puede especificar ***0***.

```
#line N <num> "nombre"
```

Hace lo mismo, pero además cambia el nombre del archivo fuente, almacenado en ***\_\_FILE\_\_***, por el nombre indicado.

#### 6.10.5 Error directive

```
#error [<message>]
```

Genera un mensaje diagnóstico en compilación, consistente en los *tokens* de preproceso indicados.

#### 6.10.6 Pragma directive

Utilizado para proporcionar información adicional al compilador.

```
#pragma [<pptokens>]
```

Se comporta de una forma establecida por la implementación. Sin embargo, si el primero de los *tokens* de preproceso indicados es ***STDC***, tiene unos usos concretos, relacionados con funcionalidades de la biblioteca estándar. En este caso específico solamente, no hay expansión de nombres de macro en esta directiva.

#### 6.10.7 Null directive

```
#
```

No tiene efecto alguno.

#### 6.10.8 Predefined macro names

Estos nombres (excepto ***\_\_LINE\_\_*** y posiblemente ***\_\_FILE\_\_***) mantienen su valor constante en la *translation unit* durante la ejecución. Ninguna de ellas, ni el identificador `defined` se pueden `#define` ni `#undef`. Si existen otros nombres de macro predefinidos por la implementación, deben empezar por guión bajo + mayúscula, o por dos guiones bajos. La implementación no puede definir `__cplusplus`.

***\_\_LINE\_\_*** es la línea actual del archivo fuente (constante entera).

***\_\_FILE\_\_*** es un literal *string* con el nombre del archivo fuente actual.

***\_\_DATE\_\_*** es la fecha de traducción de la *translation unit*. Es un literal *string* del tipo ***"Apr 21 1990"***, ***"Apr  1 1990"***.

***\_\_TIME\_\_*** es la hora de traducción de la *translation unit*, un literal *string* con formato ***"hh:mm:ss"***.

***\_\_STDC\_\_*** tiene valor ***1*** si es una implementación *standard-conforming*.

***\_\_STDC_HOSTED\_\_*** ***1*** si la implementation es *hosted*, o ***0*** si no.

***\_\_STDC_VERSION\_\_*** para *C11* es ***201112L***; para *C18* es ***201710L*** (`long int` con año y mes del estándar).

##### 6.10.8.2 Environment macros

Estos nombres son opcionalmente definidos por las implementaciones:

***\_\_STDC_ISO_10646\_\_*** es una constante entera del tipo `yyyymmL`. Si está definida, la codificación de los caracteres *wide* `wchar_t` se corresponde con el código corto definido en el estándar *ISO/IEC 10646*.

***\_\_STDC_MB_MIGHT_NEQ_WC\_\_*** es una constante entera con valor ***1*** que indica que una constante carácter de un carácter del juego básico puede tener un valor distinto al pasar a `wchar_t`.

***\_\_STDC_UTF_16\_\_*** y ***\_\_STDC_UTF_32\_\_*** indican, respectivamente, que los valores de tipo `char16_t` y `char32_t` están codificados mediante *UTF-16* y *UTF-32*.

##### 6.10.8.3 Conditional feature macros

Estos nombres de macro, si están definidos (con valor ***1*** si no se dice lo contrario), indican la presencia o ausencia de una característica disponible.

***\_\_STDC_ANALYZABLE\_\_***, ***\_\_STDC_IEC_559\_\_***, ***\_\_STDC_IEC_559_COMPLEX\_\_*** y ***\_\_STDC_LIB_EXT1\_\_*** indican respectivamente, en caso de estar definidos, que la implementación es conforme a los anexos ***L***, ***F***, ***G*** y ***K*** del estándar. En el caso de este último, su valor es ***201710L*** si está definido.

***\_\_STDC_NO_ATOMICS\_\_*** indica que no están soportados los tipos atómicos, el qualifier `_Atomic`, ni ***stdatomic.h***.

***\_\_STDC_NO_COMPLEX\_\_*** indica que no están soportados los tipos complejos ni ***complex.h***. Incompatible con la definición de ***\_\_STDC_IEC_559_COMPLEX\_\_***.

***\_\_STDC_NO_THREADS\_\_*** indica que no está soportado ***threads.h***.

***\_\_STDC_NO_VLA\_\_*** indica que no están soportados los *variably modified types*.

#### 6.10.9 Pragma operator

```
_Pragma(<string-literal>)
```

El literal *string* es "de-stringizado" (se eliminan los prefijos *string* y las comillas inicial y final, y se sustituyen ***\\\\*** y ***\\"*** por ***\\*** y ***"***). Esto produce, en la fase 3 de traducción una serie de *tokens* de preproceso que se incluyen en una directiva `#pragma`.

La ventaja de declarar los *pragmas* con este operador es que podemos definir macros que generen operadores *pragma*, mientras que una expansión macro no puede generar una directiva de preproceso.

## Annex C

*Sequence points*:

- Después de evaluar los argumentos, antes de llamar a la función.
- Entre la evaluación del primer y segundo operando de los operadores `&&`, `||` y coma.
- Entre la evaluación del primer operando del operador `?:` y el siguiente que se evalúe.
- Al completar la evaluación de una *full expression* (expresión que no forma parte de otra expresión).
- Justo antes de que una función de la biblioteca estándar retorne.
- Después de la evaluación de cada especificador de conversión en las funciones de entrada o salida formateada.
- Inmediatamente antes y después de cada llamada a una función de comparación; también entre la llamada a una función de comparación y cualquier movimiento de los objetos pasados como argumentos en esa llamada.
