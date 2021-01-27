# Terminal, Linea de comando, TTY, PTY, Shell: cómo todo en UNIX es un archivo

<p> 22 ene 2021 </p>

## ¿Qué es una terminal?

Hace mucho, mucho tiempo, se utilizaban aparatos llamados teletipos, o
*teletypes* en inglés (de aquí surge la sigla *tty*). Estos teletipos es lo que
se conoce como terminal. Las terminales son un equipo electrónico con el cual se
interactúa (o se interactuaba, hoy raramente se hace de la misma manera) con un
sistema en tiempo compartido, como era usado UNIX en la época de los setenta y
los ochenta. Estas terminales poseían un display (las más cancheras, la mayoría
se trataba de una impresora, y cada salida del sistema se imprimía, esto es, se
hacía un `print`) y un teclado para interactuar con el sistema, esto es, con la
máquina "madre". Estas terminales eran conectadas mediante un cable serial, y
esta es una de las razones por lo cual en UNIX los comandos suelen ser cortos:
`cd`, `sh`, `rm`, etcétera.

![Teletipos de la época de los 1940's](img/teltypes.jpg){height=200px}

Hoy día, en la época de computadoras con multiprocesadores y tarjetas de video,
el concepto de terminal como una máquina separada se ha abandonado. Las
terminales tenían mucho sentido en los sistemas de tiempo compartido, dado que
proveer a cada empleado de la empresa con una computadora era algo que no podía
hacerse por su alto costo. Con la llegada de las computadoras personales, los
sistemas de tiempo compartido fueron cayendo a favor de utilizar distintas
computadoras conectadas, en vez de un sistema centralizado. Sin embargo, el
concepto de terminal se mantuvo.

![Terminal VT100](img/vt100.jpg){height=200px}

No es casualidad que siga estando el concepto de terminal entre nosotros. UNIX,
al tratar todo como archivos, pudo mantener su modelo viejo (apto para sistemas
compartidos) y lograr adaptarlo a computadoras personales. Para lograr esto, el
kernel de los sistemas UNIX proveen terminales virtuales, a las cuales se
accede, generalmente, mediante una combinación de teclas (`CTRL+ALT+F1` para ir
a la terminal virtual 1). Sin embargo, no importa que sean virtuales, los
sistemas UNIX las tratan de la misma manera que a las terminales "reales". Esto
quiere decir, que si hoy día quisiéramos conectar una terminal como la VT100 en
un sistema moderno, podríamos (como se puede ver con los HP thin clients,
"computadoras" que son más bien terminales y aprovechan la computación basada en
la nube). Si te encontrás entre las/os lectoras/es curiosas/os, ya te estarás
preguntando si los emuladores de terminal están incluidos dentro del grupo de
terminales virtuales, y te voy a dar una respuesta rápida, pero tenés que saber
que después vamos a tocar ese tema: es complicado, definitivamente no son lo que
se llama tty (que se verá también después), y siempre corren debajo de una
terminal virtual; entonces, redondeando, semánticamente sí son terminales
virtuales, pero para un sistema UNIX, no.

![HP Thin Client, una especie de modem al cual se le conecta un monitor y los periféricos](img/hp-thin-client.jpg){height=200px}

## ¿Qué es un SHELL?

El shell o la shell, como cada uno/a quiera llamarlo/a, es la interfaz que
permite conectar al usuario del sistema (o un programa que hace de usuario) con
el kernel. Shell en inglés es el caparazón de la tortuga (o bueno, de otro
animal que tenga caparazón), y por lo tanto, se trata de aquella porción de
software que cubre al sistema operativo, aquella que se encarga de hacer las
llamadas de sistema (*system calls* como `write`, `open`, `fork`, `exec`,
etcétera). Mediante el shell podemos, por ejemplo, crear archivos, directorios,
copiar archivos, mover archivos, crear particiones de discos, conectarse
remotamente a otro sistema, entre otras funciones. El shell puede ejecutar
comandos tales como `cd`, o puede ejecutar programas, tales como `ls`.

![Composición del sistema operativo UNIX V de AT&T](img/unix_kern_shell_util.png){height=200px}

Ahora bien, y ¿qué es todo eso de línea de comandos, *windows manager*, *desktop
environment*? Como vimos, la definición de shell no nos indica *cómo* el usuario
debe comunicarse con el sistema, quiero decir, no nos indica si se debe hacer
mediante un medio gráfico, basado en texto, basado en señales, etcétera. En
círculos UNIX, se le dice shell al software que se conoce como *command-line
shell* o como una shell del tipo CLI (del inglés command-line interface), y lo
que nosotros hispanohablantes conocemos como *línea de comandos*.  Entonces, en
pocas palabras, la linea de comandos es un shell basado en texto.

![Shell CLI: `bash`](img/bash_demo.png){height=200px}

Pero también existen shells que son del tipo GUI (del inglés graphical user
interface), esto es, shells gráficas, que corren encima de sistemas de ventanas
(*windowing systems*), como `X` o `wayland`. Entonces, los `X` *windows
managers* o los `wayland` *compositors*, pueden ser vistos como shells, dado que
permiten organizar archivos y ejecutar comandos y programas. Algunos windows
managers vienen con un propio interprete capaz de ejecutar comandos en caliente,
como `awesomewm` y su interprete embebido de `lua`.

![Drag and drop en KDE plasma. El windows manager permite que `KDesktop` y `Konqueror` se comuniquen y se muevan archivos](img/dragdrop.png){height=200px}

## ¿Qué es una TTY y una PTY?

Una tty es un dispositivo terminal, ya sea real como el VT100, o emulado, como
los que nos proporciona el kernel de Linux cuando pulsamos una combinación de
teclas. Para el kernel, una tty es un driver que permite conectar la terminal
real o virtual con el kernel. El sistema tendrá definida una tty por cada punto
de acceso al mismo, los cuales al abrirse tienen un comportamiento determinado
por el administrador del sistema.

Una pty es una pseudo terminal, esto es, un dispositivo terminal emulado por
otro programa. Este concepto se utiliza como una capa intermedia entre el
dispositivo emulado y una tty, de tal manera que el kernel maneje a ambos como
si se tratara de una terminal común. En un principio, el concepto de pty se creó
para mapear entradas y salidas de conexiones a un sistema de tiempo compartido
vía red (como Internet) en entradas y salidas del estilo de una terminal común.
Hoy día, también se utliza este concepto para correr emuladores de terminal,
tales como `konsole`, `gnome-terminal` o `xterm`. Las pseudo terminales están
compuestas por dos partes: una parte maestra que emula la terminal (ptmx) y una
parte esclava que se comporta como una terminal común (pts).

Dado que en UNIX todo es tratado como archivos, los programas y el kernel mismo
se comunican con las distintas terminales mediante los archivos alojados en
`/dev/tty` o `/dev/pts`.

## Relacionando conceptos

Relacionemos un poco todo esto que dijimos. Veamos muy por encima un sistema
Linux moderno.

El kernel nos provee una terminal virtual, la cual se comunica con el usuario
mediante el teclado y una pantalla, y se comunica con el kernel mediante un tty
driver. Así mismo se genera una conexión entre tty y shell, el cual será el
encargado de ejecutar los comandos que el usuario escriba.

![Diagrama con relación de conceptos](img/tty_kernel_shell_term.png){height=150px}

## ¿Qué es un terminal emulator?

Los terminal emulator buscan emular a las *smart terminals* que surgieron en los
últimos tiempos de los sistemas de tiempos compartidos, esto es, emulan
funcionalidades como procesar la entrada del usuario (backspace borra sin tener
que preguntar al sistema que se debe hacer), se refrescan instantáneamente, y
permiten realizar ediciones en línea.

Para crear estos programas se utilizó el concepto de pseudo terminales. Lo
primero que hacen estos programas es crear una pseudo terminal, compuesta por
dos archivos ya vistos: el pty master, aquella parte que el terminal emulator
abre, y el pty slave, aquella parte que los programas que corren en una
determinada terminal abren para hacer operaciones de I/O.

Una vez que se abre la pseudo terminal, el programa comienza un subproceso, que
suele ser una shell como `bash`. El terminal emulator realiza estas acciones
casi igual que un inicio de sesión en una terminal real o en una terminal
virtual.

1. `fork`.
2. Abre el pts con los file descriptors 0, 1 y 2.
3. `exec` el shell en el proceso hijo.

Cuando el proceso hijo escribe en el pts, el terminal emulator ve la entrada en
el ptmx. De la misma manera, cuando el emulador escribe en el ptmx, se ve como
una entrada en el pts.

## Conclusión

Un sistema operativo construido en los 70's hoy día es tan (o quizás más)
importante como otros sistemas operativos más modernos. Sus mecanismos internos
fueron tan bien pensados que aún siguen funcionando con pequeñas modificaciones.
Esto es, sin dudas, el beneficio de un diseño a conciencia, con una filosofía de
y una manera de hacer las cosas de probado éxito.

Y te preguntarás, ¿como funcionan los multiplexers de terminales como `screen`
de GNU o `tmux`? Y yo te contesto: es una capa más encima de los emuladores de
terminal, ya que al final del día son tan solo un nuevo programa; será un tema
de futura discusión.
