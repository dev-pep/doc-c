# C - notas

Este documento es una recopilación de aclaraciones sobre diversos aspectos del lenguaje *C* que quizá no queden demasiado claros, estén muy dispersos o no tengan suficiente cobertura en el estándar.

## Alineación

Una dirección de memoria suele contener 1 *byte*, es decir, 8 *bits* (puede haber arquitecturas con *bytes* de 16 bits, pero no es lo habitual). Los datos cuyo tamaño supera el *byte* ocupan varias direcciones consecutivas de memoria. Aunque cada dirección de memoria corresponda a un *byte*, los accesos se hacen en grupos (a nivel de *word*, por ejemplo) para aumentar el rendimiento.

Para un mejor rendimiento, con los procesadores actuales, los datos deben estar alineados, lo que en la práctica significa que cada dato debe empezar en una dirección de memoria que sea *múltiplo de su tamaño*.

En las estructuras de datos esto puede significar añadir relleno (*padding*) entre los miembros y/o reordenarlos de forma inteligente. La otra opción es no usar *padding* para ahorrar espacio, lo cual se llama empaquetar (*packing*) los datos, aunque en este caso se ahorra memoria a expensas de un menor rendimiento. El estándar de *C* no indica ningún mecanismo de *packing*, pero las implementaciones (como *GCC*) pueden ofrecerlo.

Cada tipo de datos tiene sus propios requisitos de alineación, basados en su tamaño. Supongamos un tipo `double`, que ocupa 8 *bytes*. En este caso, el requisito de alineación de ese tipo es, normalmente, de 8 *bytes*, coincidiendo con su tamaño: `sizeof(double)` es igual a `_Alignof(double)`. Es decir, su *byte* inicial debería estar almacenado en alguna de estas direcciones de memoria: 0, 8, 16, 24, etc.

Los requisitos de alineación son siempre potencias de 2, con lo que un tipo de datos de 9 *bytes* (si existiera tal cosa) tendría un requisito de alineación de 16 *bytes*.

Los tipos primitivos de *C* tienen un `_Alignof` igual a su `sizeof`. Pero la cosa se complica cuando creamos estructuras y *arrays*.

El caso de un *array* es simple: el `sizeof` sera el tamaño del tipo de los elementos del *array* multplicado por el número de elementos, mientras el `_Alignof` será el del tipo del elemento. Así, un `short int[3]` tendrá `sizeof` de 6 *bytes*, y un `_Alignof` de 2 *bytes* (suponiendo que en la implementación concreta, el tipo `short int` tenga
un tamaño de 2 *bytes*).

El requisito de alineación de un tipo ***char*** es de 1 *byte*, con lo que en realidad puede residir alineado en cualquier dirección de memoria.

En el caso de estructuras, el sistema debe realizar el *padding* necesario para que cada miembro respete sus requisitos de alineación. Esto se consigue de la siguiente forma: el primero de los miembros ocupa la dirección (relativa al inicio de la estructura) cero. Después de cada miembro, se inserta, si es necesario, una serie de *bytes* de relleno, un *padding*, que hace que el siguiente miembro vaya a empezar en una dirección múltiplo de su requisito de alineación. En cuanto al requisito de alineación total de la estructura resultante, ¿cuál debe ser? Pues para que todos los miembros que contiene estén alineados, es suficiente con un requisito de alineación igual al del miembro mayor que contiene.

Después del último miembro, se inserta un *padding* asumiendo que el siguiente elemento en memoria será otro objeto del mismo tipo estructura, de tal modo que en un *array* de ese tipo estructura, todos los datos quedarían alineados.

Veamos un ejemplo. Considerando que `_Bool` ocupa 1 *byte*, `long long int` 8 *bytes*, y `long double` (el tamaño máximo posible hasta la fecha) 16 *bytes*:

```c
struct s1
{
    _Bool a;
    long long int b;
    _Bool c;
    long double d;
};
```

En este caso, el almacenamiento en memoria sería asi: ***a*** ocuparía el *byte* 0; ***b*** ocuparía desde el 8 (por su `_Alignof` de 8) hasta el 15; ***c*** ocuparia el *byte* 16; ***d*** debería empezar en el 32 (`_Alignof` es 16) hasta el 47. Por lo tanto el `sizeof` sería de 48 *bytes*, mientras su `_Alignof` total sería 16 (miembro más grande es `long double`). El tamaño de 48 es correcto para que el siguiente objeto de tipo `struct s1` empiece alineado, con lo que no es necesario un *padding* final. Veamos otro caso:

```c
struct s2
{
    _Bool a;
    long long int b;
    _Bool c;
    long double d;
    _Bool e;
};
```

Este caso es igual al anterior, pero añadiendo un `_Bool` al final. Esto no cambia los requisitos de alineación de la estructura, pero sí afecta al `sizeof`. En este caso, podríamos pensar que el tamaño sería 49, pero eso haría que el siguiente objeto de tipo `struct s2` empezara desalineado, así que se le añaden *trailing padding bytes* hasta que el `sizeof` de la estructura sea un múltiplo de su `_Alignof`. Así pues, el `sizeof` de esta estructura debería ser 64 *bytes*, mientras el `_Alignof` seguiría siendo 16.

Sin embargo, podríamos reordenar los miembros:

```c
struct s2_reordenada
{
    long double d;
    long long int b;
    _Bool a;
    _Bool c;
    _Bool e;
};
```

En este caso, es la misma estructura ***s2*** con los miembros cambiados de orden: ***d*** ocupa desde la posición 0 a la 15; no es necesario *padding* después, al empezar ***b*** en la 16, hasta la 23; los tres booleanos ocuparían desde la 24 hasta la 26. Como el requisito de alineación de la estructura sigue siendo 16, el tamaño debería ser un múltiplo de este, con lo que con un *trailing padding* de 5 *bytes* es suficiente. Ahora, en lugar de tener un tamaño de 64 *bytes*, la estructura ocupa 32, simplemente reordenando los datos.

Teniendo en cuenta esto, hay que tener cuidado con el orden con el que definimos los miembros de una estructura para minimizar el uso de memoria, si es que es importante ese aspecto. Si no nos importa, cualquier orden es bueno.

## Tipos complejos

Dado que los *declarators* (6.7.6 del estándar) pueden anidarse sin limitación, los tipos de datos en *C* pueden llegar a ser muy complejos de interpretar, de tal manera que hay que practicar dicha interpretación para no perderse, y por supuesto para poder crear los tipos de la forma que deseemos. En ocasiones se pueden formar unos tipos derivados extremadamente crípticos. En caso de tener que trabajar con un tipo así, es recomendable usar `typedef` para aumentar la legibilidad del código.

### La técnica

Una declaración está compuesta por un tipo básico, y *declarators* que derivan ese tipo básico. Existen básicamente tres tipos de elementos que pueden formar un tipo derivado. Dejando a parte las estructuras, uniones y tipos atómicos, los elementos que pueden entrar en juego en una declaración son:

- *Array*, caracterizado por un par de corchetes ***[]*** (con o sin una expresión entera) a la derecha del tipo que está derivando.
- Apuntador, caracterizado por un asterisco ***\**** a la izquierda del tipo que está derivando.
- Función, caracterizado por un par de paréntesis ***()*** (con o sin una lista de tipos), a la derecha del tipo que está derivando.

Los *declarators* de *array* y función tienen preferencia sobre el de apuntador.

Asumiendo que deseamos leer correctamente un tipo derivado construido a partir de un tipo base, el algoritmo a seguir sería:

1. Iniciar en el identificador, o en su defecto, en el paréntesis más interior, en el lugar justo donde se colocaría dicho identificador si hubiera que incluirlo (para identificar el sitio exacto, ver más abajo). Escribimos **«Declaración de un »**.
2. Mirar primero a la derecha, desde el punto donde estamos hasta el próximo paréntesis derecho ***)*** o el final del tipo (lo que suceda antes). Si no hay nada, pasamos al punto 3. Si lo hay, se realizará, para cada elemento encontrado, secuencialmente:
    - Si el elemento es *array*, puede tener una expresión entera dentro, en cuyo caso escribiremos **«array de N elementos »**, donde ***N*** es el valor de la expresión. Si no hay expresión, simplemente escribiremos **«array de elementos »**.
    - Si el elemento es función, habrá un par de paréntesis, dentro de los cuales habrá, opcionalmente, una lista de tipos separados por comas. En caso de que no haya nada dentro de los paréntesis, escribiremos **«función que retorna »**. Si existe una lista de tipos, escribiremos **«función con parámetros (...) que retorna»**, donde '...' será la lista de parámetros tal como aparece en el tipo.
3. Miraremos ahora a la izquierda del punto inicial, desde el punto actual hasta el próximo paréntesis izquierdo ***(*** o justo después del tipo base (lo que suceda antes). Si no hay nada, pasamos al siguiente punto. Si lo hay, se realizará, para cada elemento encontrado, secuencialmente:
    - Solo podemos encontrar elementos apuntador. En este caso, por cada ***\**** encontrado, escribimos **«apuntador a »**.
4. Si hemos terminado de procesar un paréntesis interior, ahora saldremos de ese paréntesis y seguiremos desde el paso 2. En cambio, si no se trataba de un paréntesis interior, es que hemos terminado con la declaración del tipo, y solo queda por procesar el tipo base. En ese caso, simplemente tenemos que escribir ese **tipo base y un punto final**.

En cuanto al segundo subpaso del paso 2, si queremos detallar los tipos de las listas de parámetros que pudiesen haber aparecido, podemos aplicarles este algoritmo a cada uno de ellos.

Si no hay identificador en la especificación de tipo (puede pasar en los *casts*, o como argumento de `sizeof`), hay que buscar el lugar exacto que ocuparía este. Solo hay que tener en cuenta que dicho punto cumple estas tres condiciones:

- Está a la derecha de todos los indicadores de apuntador (***\****).
- Está a la izquierda de todos los indicadores de función (***()***) y *array* (***[]***).
- Está dentro del par de paréntesis más interior posible.

Recordemos que las funciones no pueden retornar una función ni un *array*. Esto implica que en el nivel de anidamiento actual, tras el *declarator* de función no habrá otro *declarator* (no habrá corchetes ni listas de parámetros entre paréntesis).

Por otro lado, no pueden existir *arrays* de funciones. Esto implica que en el nivel de anidamiento actual, tras un *declarator* de tipo *array*, solo puede haber *declarators* de tipo *array*. Sin embargo, no se puede declarar un *array* de tipos incompletos, por lo que en un *array* multidimensional solo el primer declarator (el primer par de corchetes) puede estar vacío.

### Ejemplos

Veamos ejemplos de creciente dificultad en los que podemos aplicar el algoritmo:

`int` - «Declaración de un int.» Solo hay tipo base.

`int[5]` - «Declaración de un array de 5 elementos int.»

`int x[3][5]` - «Declaración de un array de 3 elementos array de 5 elementos int.»

`int *x[5]` - «Declaración de un array de 5 elementos apuntador a int.»

`int *x[3][5]` - «Declaración de un array de 3 elementos array de 5 elementos apuntador a int.»

`int (*[4])[3][5]` - «Declaración de un array de 4 elementos apuntador a array de 3 elementos array de 5 elementos int.»

`int *(*[4])[3][5]` - «Declaración de un array de 4 elementos apuntador a array de 3 elementos array de 5 elementos apuntador a int.»

`int (*)(int,float)` - «Declaración de un apuntador a función con parámetros (int,float) que retorna int.»

`int *(**[][7])(char)` - «Declaración de un array de elementos array de 7 elementos apuntador a apuntador a función con parámetros (char) que retorna apuntador a int.»

Para practicar con los tipos del lenguaje *C* puede resultar muy útil la ayuda de la aplicación ***cdecl***.

### *Type qualifiers*

Para incorporar también *type qualifiers*, lo más frecuente es utilizarlos en los *declarators* de tipo apuntador. Lo importante aquí es vincular el cualificador al asterisco indicador de apuntador (***\****) adecuado. Por suerte, es muy sencillo: corresponde siempre al asterisco más próximo a la izquierda del *qualifier*. Si no hay tal asterisco a su izquierda es porque el cualificador pertenece al tipo base (puede estar tanto a la derecha como a la izquierda de dicho tipo).

Veamos unos ejemplos (esta vez con identificador):

`int * const p` - «Declaración de un apuntador constante a int.»

`int const * p` - «Declaración de un apuntador a int constante.»

`const int * p` - igual que el anterior.

`int * const (* const *p[][7])(char)` - «Declaración de un array de elementos array de 7 elementos apuntador a apuntador **constante** a función con parámetros (char) que retorna apuntador **constante** a int.»

## Tipos compatibles y compuestos

Un objeto puede ser declarado varias veces en el mismo *scope* o *linkage* (siempre que exista *linkage* interno o externo) utilizando tipos distintos, siempre y cuando estos tipos sean *compatibles*. El tipo resultante de estas declaraciones será el que resulte de la combinación de todas ellas, y se le llama *tipo compuesto*.

La conversión de y a un tipo compatible no cambia valores, representación en memoria ni requisitos de alineación.

Las normas de compatibilidad de tipo son, resumidamente:

- En primer lugar, dos tipos son compatibles si son exactamente iguales, aunque no es necesario que sean iguales para ser compatibles.
- Dos declaraciones de tipo estructura, unión o enumeración tendrán tipo compatible si están declaradas con el mismo nombre (*tag*), y existe una correspondencia uno a uno entre sus miembros, de tal forma que cada uno esté declarado con el mismo nombre y con un tipo compatible. Si un miembro tiene especificador de alineación, el correspondiente debe tener un especificador equivalente.
- Un tipo enumeración será compatible con tipo ***char*** o entero, a elección de la implementación, pero deberá ser capaz de representar todos los valores de la misma.
- Para que dos tipos cualificados sean compatibles deben ser versiones idénticamente cualificadas de tipos compatibles.
- Para que 2 apuntadores sean compatibles deben ser idénticamente cualificados y apuntar a tipos compatibles.
- Para que dos tipos *array* sean compatibles, sus elementos deben ser de tipos compatibles, y si el tamaño de ambos está definido, estos deben ser iguales.
- Para que dos tipos de función sean compatibles deben tener tipos de retorno compatibles. Además, si las listas de parámetros de ambas funciones están descritas, deben coincidir en el número de parámetros, en uso de la *ellipsis* (`...`), y ser compatibles los tipos uno a uno tras el ajuste de tipos (conversión de tipos *array* y función a apuntadores).

### Construcción del tipo compuesto

Así se construye un tipo compuesto (*composite type*) a partir de dos tipos compatibles (si son más tipos hay que aplicar estas reglas sucesivamente):

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
