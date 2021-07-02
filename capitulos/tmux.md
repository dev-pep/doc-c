# tmux

La aplicación *tmux* (*Terminal MUlTiplexer*) permite dividir la pantalla de un terminal *Unix*.

*tmux* permite organizar sesiones, que contienen ventanas, las cuales contienen paneles. Todas las acciones de *tmux* tienen asociadas un **comando *tmux***. Hay varias formas de ejecutar un comando *tmux*:

Muchas acciones pueden realizarse a través de un *shortcut* de teclado desde dentro de *tmux*. Estos atajos de teclado consisten en una primera pulsación de una combinación de teclas, llamada **prefijo**, seguida de una tecla característica del atajo. Por defecto, el prefijo es `C-b` (***Ctrl+B***). Dado que en muchas ocasiones se prefiere cambiar este prefijo, nos referiremos a él simplemente como `P`.

Desde **fuera** de *tmux* se puede ejecutar un comando *tmux* introduciendo una orden de este estilo:

```
tmux <comando> [opciones]
```

El comando puede abreviarse todo lo que se quiera, siempre que no se produzca una ambigüedad.

La tercera forma de ejecutar un comando se realiza desde dentro de *tmux*. En este caso se usa el atajo `P :`, y a continuación se introduce el comando deseado, con sus opciones.

## Trabajar con sesiones

Cuando no hay ninguna sesión *tmux* creada, para crear una sesión nueva:

```
tmux new-session
```

Podemos abreviar el comando a `new`:

```
tmux new
```

Esto crea una sesión y la adjunta (*attach*), es decir, la muestra en el terminal. Al ser la primera sesión, se inicial el **servidor *tmux***, el cual cesa su ejecución cuando no existe ninguna sesión.

Las sesiones tienen un nombre asociado. Si no lo especificamos, la primera se denominará ***0***, la segunda ***1***, etc. Si deseamos crear una sesión dándole un nombre, usaremos el *flag* `-s` (de *session*):

Ejecutar `tmux` sin parámetros es equivalente a ejecutar `tmux new-session`.

```
tmux new -s MiNombre
```

No es necesario que haya espacio entre el *flag* y el nombre. Si dicho nombre contiene espacios, habrá que indicarlo entrecomillado.

Al crear una nueva sesión se puede indicar un comando que será ejecutado en esta.

```
tmux new -s Editor "vim ~/proyecto/main.c"
```

Si el comando no contiene espacios no es necesario entrecomillarlo.

También se puede hacer así:

```
tmux new -s Editor -- vim ~/proyecto/main.c
```

Todo lo que tecleemos tras el doble guión (`--`) será considerado comando, con lo que no habrá que entrecomillarlo nunca, por más espacios y argumentos que contenga.

Al crear una sesión, esta contiene una sola ventana, cuyo nombre se basa en lo que está ejecutando. Si queremos crear la sesión dando también nombre a la ventana inicial, usaremos el *flag* `-n` (de *name*).

```
tmux new -s Sesión -n Comandos
```

Como siempre, no hace falta separar el *flag* del nombre con espacios, y si el nombre contiene espacios, se debe indicar entrecomillado.

## Barra de estado

Aunque se puede configurar la información de la barra de estado, por defecto se muestra, de izquierda a derecha:

- El nombre de la sesión *attached* entre corchetes (***\[\]***).
- Los números de índice y nombres de las ventanas de la sesión. La ventana actual está marcada con asterisco (***\****), y la última ventana activa (anterior a la actual) con guión (***-***).
- El nombre del panel actual.
- Hora y fecha.

## Ayuda

Para ver una lista de atajos de teclado, se usa `P ?`. En dicha lista, el prefijo se muestra como `C-b`. Algunos atajos son del tipo `M-n`. Esta ***M*** se refiere a la tecla *meta*. En equipos *Mac* significa la tecla ***comando***, y en *PC*, la tecla ***Alt***.

## *Attaching* y *detaching*

Es posible desvincular el terminal de la sesión actual. De ese modo, la sesión sigue viva en segundo plano, gracias al servidor *tmux*. Las sesiones seguirán siendo accesibles mientras el servidor *tmux* esté activo. Cuando un usuario hace *logout* del sistema, se cierra el servidor. Puede que en algunos sistemas el servidor siga funcionando tras un *logout*, pero en todo caso no sobrevive a un reinicio del equipo.

Para desvincular el terminal de la sesión actual, se ejecuta el comando `detach`. Su atajo de teclado es `P d`.

Para volver a vincular un terminal a una sesión (se puede hacer en varios terminales, en cuyo caso varios terminales quedarán *attached* a la misma sesión) se debe usar el comando `attach-session` (o abreviaturas como `attach` o incluso `a`). Sin argumentos, vincula la última sesión *attached*.

```
tmux a
```

Podemos seleccionar una sesión concreta a la que vincular el terminal, especificando su nombre tras el *flag* `-t` (de *target session*).

Si al vincular el terminal a una sesión queremos que otros terminales que pudiesen estar vinculados a la misma se desvinculen de esta, indicaremos el flag `-d` (de *detach*):

```
tmux attach -dtSesión
```

El comando `new-session` tiene un *flag* `-A` que hace que el terminal se vincule a una sesión si ya existe, y si no la crea (y vincula):

```
tmux new -As NombreSesion
```

En caso de que exista y se vincule el terminal, `new-session` permite el *flag* `-D`, que funciona como el *flag* `-d` de `attach-session`.

Al crear una sesión nueva mediante `new-session`, si indicamos el *flag* `-d` (*detached*), la sesión es creada pero el terminal no se vincula a ella.

### Cambio de sesión

Se puede cambiar a otra sesión, simplemente usando el comando `attach-session` y especificando la deseada. Tambén existen atajos de teclado para cambiar de sesión rápidamente:

- `P )` cambia, cíclicamente, a la siguiente sesión (en la lista de sesiones).
- `P (` lo mismo que `P )` pero para la sesión anterior.

## Listado de sesiones

Podemos ver un listado de sesiones con el comando `ls`. Si el servidor *tmux* no está activo, no hay sesiones.

Si utilizamos el atajo `P s` desde *tmux*, se nos mostrará una lista de sesiones y nos permitirá seleccionar una de ellas para vincular (se cancela con `Esc` antes de seleccionar).

## *Kill* todas las sesiones

El comando `kill-server` cierra el servidor y se pierden todas las sesiones.

## Trabajar con ventanas

Las sesiones pueden contener varias ventanas. Solo una ventana puede ser visible a la vez.

Para crear una ventana se usa el comando `new-window` (hay un alias: `neww`), cuyo atajo de teclado es `P c` (*create*). Se crea con el índice más bajo posible. El *flag* `-d` (*detached*) hace que se cree la ventana pero que no pase a ser la ventana actual. `-n` (*name*) permite dar nombre a la ventana. También se le puede indicar el número de índice mediante el *flag* `-t` (*target*)

También es posible indicar un comando que definirá el contenido de la nueva ventana, igual que en `new-session`.

## *Splitting*

Es posible dividir (*split*) una ventana en **paneles** (*panes*). Para ello se utiliza el comando `split-window`. Este comando tiene varios *flags* útiles:

- `-h` produce una división horizontal, quedando un panel al lado del otro.
- `-v` produce una división vertical, quedando un panel encima del otro.
- `-d` crea el nuevo panel, pero no lo convierte en el panel activo.
- `-f` (de *full*) utiliza todo el ancho o alto de la pantalla, y no del panel que se está dividiendo.
- `-b` coloca el nuevo panel a la izquierda o encima, en lugar de a la derecha o abajo.

El posible también en este caso especificar un comando que definirá el contenido del nuevo panel.

Los atajos de teclado para la división son:

- `P %` - división horizontal.
- `P "` - división vertical.

## Cambio de ventana actual

Para cambiar la ventana actual se usa el comando `select-window`, aunque lo más frecuente es utilizar los distintos atajos de teclado:

- `P n` siguiente (*next*) ventana. Pasa cíclicamente a la ventana de índice superior a la actual.
- `P p` ventana anterior (*previous*), al contrario que `P n`.
- `P <n>` donde ***n*** es un número del 0 al 9. Cambia a la ventana con ese índice.
- `P l` pasa a la ventana anterior (última activa antes de la actual).

## Cambio de panel activo

Para cambiar el panel activo se usan los comandos `select-pane` y `display-panes`. Lo usual es utilizar los atajos de teclado:

- `P <Arriba>`, `P <Abajo>`, `P <Izquierda>`, `P <Derecha>` cambia, cíclicamente, al panel que está en la dirección de la flecha presionada.
- `P q` muestra durante unos momentos los números de índice de los paneles (que dependen exclusivamente de su posición en pantalla). El número del panel actual se muestra en rojo. Si antes de que desaparezcan los números presionamos uno de ellos, se cambiará el panel activo al indicado.
- `P o` va al siguiente panel (por número).
- `P C-o` intercambia el panel activo con el siguiente (por número).

## Trabajar con paneles

Existen algunos atajos de teclado útiles para los paneles:

- `P z` (de *zoom*) maximiza el panel activo dentro de la ventana actual. Volviendo a presional `P z` se restaura a su tamaño anterior.
- `P !` convierte el panel activo en ventana (activa). Es decir, elimina el panel activo de la ventana actual, y crea una nueva ventana cuyo único panel es precisamente ese.

## *Killing*

Los atajos de teclado relacionados con los comandos `kill-window` y `kill-pane` (`killw`, `killp`) son:

- `P &` mata la ventana actual (pide confirmación), terminando los procesos que se ejecutan en sus paneles.
- `P x` mata el panel activo (pide confirmación), terminando los procesos que se ejecutan en él.

Para matar una sesión se usa el comando `kill-session`.

Estos comandos matan el elemento actual o activo si no se les especifica el *target* (`-t`).

Una forma de cerrar un panel es terminando el proceso asociado al mismo. Por ejemplo, introduciendo `exit` (o pulsando `Ctrl-D`) desde un intérprete *bash*.

Cuando un panel se cierra, si este era el único existente en la ventana, se cierra la ventana.

Cuando una ventana se cierra, si esta era la única de la sesión, se cierra la sesión.

## Renombrar

Para renombrar una sesión (comando `rename-session`): `P $`.

Para renombrar una ventana (comando `rename-window`): `P ,`.

## Redimensionar

Se puede cambiar el tamaño del panel activo (comando `resize-pane`):

- `P C-<Arriba>`, `P C-<Abajo>`, `P C-<Izquierda>`, `P C-<Derecha>` cambia el tamaño en 1 unidad hacia la dirección indicada.
- `P Alt-<Arriba>`, `P Alt-<Abajo>`, `P Alt-<Izquierda>`, `P Alt-<Derecha>` cambia el tamaño hacia la dirección indicada, en pasos más grandes.

En estos casos, si tras el prefijo se presiona reiteradamente la tecla correspondiente y **sin pausa**, no hay que ir presionando el prefijo cada vez.

## Configuración de *tmux*

La configuración reside en el archivo ***~/.tmux.conf***. Son muchas las opciones configurables, aunque veremos solamente un ejemplo para cambiar el prefijo.

### Cambio de prefijo

Para cambiar, por ejemplo, de prefijo `C-b` a `C-a`:

```
set -g prefix C-a
unbind C-b
bind C-a send-prefix
```

## *Layouts* predefinidos

Dado que los comandos *tmux* pueden ejecutarse desde fuera de la aplicación, crear un *layout* personalizado es muy sencillo, mediante un simple *shell script*. Un ejemplo:

```bash
#!/bin/bash
tmux new-session -d -s IDE -n Edición -- vim main.c
tmux split-window -h -- vim main.h
tmux split-window -v
tmux select-pane -t 0
tmux new-window -n Consola
tmux send-keys 'cd ~/proyecto' Enter
tmux select-window -t Edición
tmux attach-session -t IDE
```

La primera línea crea la sesión (y renombra la ventana), y lo hace en modo *detached*, puesto que de lo contrario entraría en *tmux* y no seguiría ejecutando comandos hasta que *tmux* retornase. El comando también da nombre a la ventana de la sesión, y le asocia el proceso `vim main.c`. Los comandos a continuación actuarán sobre esta sesión, hasta que se ejecute un nuevo comando `new-session`.

A continuación crea un par de paneles más y selecciona el panel 0 como activo en esa ventana.

Luego crea otra ventana a la que llama ***Consola***. Como no se especifica el *flag* `-d`, esa ventana pasa a ser la ventana actual.

A continuación hacemos uso del comando `send-keys`. Este comando emula el envío de pulsaciones de teclado al panel activo de la ventana actual. Este comando utiliza un número arbitrario de argumentos: palabras o texto entre comillas. Los espacios que separan los argumentos son descartados. Algunas secuencias de caracteres son mapeadas a teclas específicas (`F1` a `F12`, `Enter`, `Home`, `End`, `PgDn`, `PgUp`, `Tab`, `Escape`, `BSpace`, `Up`, `Down`, `Left`, `Right`, `C-<tecla>`, `M-<tecla>`, etc.), siempre que tal secuencia no esté entre comillas.

En este caso, se introduce automáticamente el comando `cd ~/proyecto` en la nueva ventana.

Seguidamente se selecciona la otra ventana, y para finalizar se vincula el terminal a la sesión creada.

Podría mejorarse el *script* para que no creara nada si la sesión ya existe (usando el comando `has-session`):

```bash
#!/bin/bash

tmux has-session -t IDE 2>/dev/null

if [ $? != 0 ]; then
    tmux new-session -d -s IDE -n Edición -- vim main.c
    tmux split-window -h -- vim main.h
    tmux split-window -v
    tmux select-pane -t 0
    tmux new-window -n Consola
    tmux send-keys 'cd ~/proyecto' Enter
    tmux select-window -t Edición
fi

tmux attach-session -t IDE
```
