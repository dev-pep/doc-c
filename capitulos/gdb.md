# Depurador *GDB*

Esto es un resumen de las funciones más útiles del depurador *GDB*.

Para depurar un archivo ejecutable programado en *C*, dicho archivo debe contener la información de depuración adecuada. En *GCC*, esto se consigue indicando el *flag* `-g` (o `-ggdb`). En este caso el compilador no podrá aplicar todas las optimizaciones. El *flag* `-Og` aplica las que son compatibles con la inclusión de información de depuración. En todo caso, es posible deshabilitar las optimizaciones completamente, con `-O0`.

Para depurar un programa, se debe teclear:

```
gdb <nombre>
```

Donde ***nombre*** es el nombre del archivo ejecutable a depurar.

## Comandos

Los comandos pueden abreviarse a voluntad para ganar agilidad, a no ser que tal abreviatura produzca ambigüedad. Algunos comandos poseen un alias abreviado no ambiguo. Se incluyen estos alias entre paréntesis:

### backtrace (bt)

Muestra el *call stack* entero (la última llamada se muestra arriba).

### break

Se utiliza para establecer *breakpoints*. Se puede indicar un número de línea, o incluso un nombre de función:

```
break 45
break foo
```

### continue

Prosigue con la ejecución. Es como `run`, pero en lugar de empezar desde el principio, prosigue con la ejecución desde el punto actual.

### delete

Elimina un *breakpoint* o *watchpoint*, pasándole como argumento su *ID* (número), que puede obtenerse mediante `info breakpoints`.

Si no se le pasa argumento, simplemente elimina todos los puntos de interrupción.

```
delete 3
delete
```

### display

Nos muestra el valor de una expresión, que puede contener variables (como si usáramos `print`) *en cada interrupción* de la ejecución (por *breakpoint*, por `next`, etc.). Cuando las variables involucradas dejan de estar en *scope*, no se muestra ese *display*.

Sin argumentos, muestra los *displays* actuales (en *scope*).

```
display i
display i+j
display
```

### down

Muestra el siguiente nivel (hacia abajo) en el *call stack*. Si se le pasa un entero, indicamos el número de niveles a descender.

```
down
down 3
```

### finish

Termina la ejecución de la función actual (a no ser que exista un *breakpoint* antes), e interrumpe la ejecución al finalizar esta.

### info

Muestra información variada.

Uno de los usos, es para ver los *breakpoints* y *watchpoints* actuales. En este caso se hace mediante:

```
info breakpoints
```

Se puede abreviar así;

```
i b
```

### list

Muestra las líneas alrededor de la línea indicada, o en su defecto, alrededor de la línea actual.

```
list
list 45
```

### next

Avanza una línea en la ejecución. Si es una llamada a una función, la ejecuta entera (a no ser que la función tenga *breakpoints* en su interior).

Si se le pasa un argumento numérico, será el número de líneas a avanzar (a no ser que haya un punto de interrupción antes).

### print

Muestra el resultado de una expresión cualquiera, en sintaxis *C*. Muy útil para ver el valor de las variables en el momento actual.

```
print i
print i+j
```

### quit

Sale de *GDB*. Equivale a pulsar ***Ctrl-D***.

### reverse-continue (rc)

Para retomar la ejecución **hacia atrás**, al contrario que `continue`. Véase `target record-full`.

### reverse-next (rn)

Para retroceder un paso en la ejecución, al contrario que `next`. Véase `target record-full`.

### reverse-step (rs)

Para retroceder un paso en la ejecución, al contrario que `step`. Véase `target record-full`.

### run

Este comando ejecuta el programa **desde el principio** del mismo. Si deseamos ejecutar el programa con argumentos, o incluso redireccionar (entrada, salida y/o error), se hace de la misma forma que se haría desde la línea de comandos:

```
run arg1 arg2 <infile.in >outfile.out 2>error.err
```

### set

Cambia elementos del entorno.

Junto con `var` cambia el valor de una variable.

```
set var i=50
```

### start

Reinicia la ejecución, pero se para en la primera línea del programa. Los argumentos dados a este comando tienen el mismo significado que en el caso de `run`.

### step

Avanza una línea en la ejecución, pero al contrario que `next`, si es una llamada a una función, entra en ella.

Al igual que con `next`, un argumento define la cantidad de pasos a realizar (a no ser que haya un punto de interrupción en medio).

### target record-full

Este comando nos facilita ir hacia atrás en la ejecución. Al compilador le falta información para hacerlo con éxito, estableciendo correctamente el estado anterior. Mediante este comando, todas las ejecuciones a partir de ese momento son registradas, de tal forma que se puede retroceder con seguridad, ya que guarda el camino exacto que describe la ejecución, así como el estado de todas las variables.

```
target record-full
```

### undisplay

Para dejar de mostrar una variable determinada. Se le debe pasar como parámetro el *ID* (número) del *display* correspondiente. Sin argumentos elimina todos los *displays*.

```
undisplay 3
undisplay
```

### up

Muestra el siguiente nivel (hacia arriba) en el *call stack*. Si se le pasa un entero, indicamos el número de niveles a ascender.

### watch

Establece un *watchpoint*: vigila una variable; cuando esta variable cambia su valor, la ejecución se interrumpe.

```
watch var
```

### whatis (what)

Muestra el tipo de una variable.

```
whatis i
```

# Bonus: errores de memoria (*Valgrind*)

En relación a la reserva de memoria dinámica (en el *heap*), existen dos tipos de errores de memoria:

- *Leaks* (fugas): reserva (*allocation*) de memoria que no es liberada.
- Accesos a memoria liberada.

Mediante *GDB* puede ser muy complejo encontrar estos tipos de error, sobre todo si el código adquiere un tamaño considerable. En este caso, puede ser muy útil una herramienta que detecte estos errores automáticamente. Esta herramienta se llama ***Valgrind***.

Su uso básico es muy simple. En primer lugar, los requisitos de compilación son similares a los usados por *GDB* (flags `-g` y si se desea `-O0`). Para hacer la comprobación solo hay que teclear:

```
valgrind <programa>
```

Donde ***programa*** es el la orden que ejecutaríamos para ejecutar nuestro programa:

```
valgrind ./myprog arg1 arg2
```

Tras la comprobación obtendremos un informe de los errores encontrados.
