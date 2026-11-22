# SistemasEmpotrados


### PRÁCTICA 3: MAQUINA EXPENDEDORA
##### OBJETIVOS
<p>
En esta práctica teníamos que crear nuestra propia máquina registradora, la cual tenía que cumplir con una serie de funciones y algunas de las funciones que hemos dado este año: threads, Watchdog, timerone...  
</p>

##### EXPLICACIÓN DE MI ALGORITMO
<p>
Mi máquina expendedora funciona básicamente con una máquina de estados y cada uno de estos, es una función de la máquina. Los estados que tienen son MODO_SERVICIO, MONITOR_Y_CATALOGO, NAVEGACION_Y_SELECCION, BEBIDA_SELECCIONADA, PREPARADO, ADMIN, TEMPERATURA, DISTANCIA, CONTADOR, MOD_PRECIOS, SUMAR_Y_RESTAR.


Primero el programa empieza en la pantalla cargando, que no forma parte de ninguno de estos estados y el cual se regula con un contador, para realizar el parpadeo del LED1 y que en el LCD salga CARGANDO..., una vez que el contador llega a tres; para que parpadee tres veces por segundo, limpiamos el LCD y el LED1 se queda apagado y pasamos a la máquina de estados.

*¿Para qué es cada estado?*

- MODO_SERVICIO: En este estado lo que hacemos es identificar con el sensor de ultrasonidos si tenemos a algún cliente a un metro de distancia o menos, para pasar así al siguiente estado, sino nos mantenemos en este hasta que llegue uno. En la máquina de estados como tal el case está vacío porque he hecho que funcione con un thread, así que el thread cuenta con la función.

- MONITOR_Y_CATALOGO: Este tambíen su case está vacío, ya que como el anterior es un thread y lo que hace es mostrar la temperatura y la humedad durante un tiempo y pasa a NAVEGACION_Y_SELECCION.

- NAVEGACION_Y_SELECCION: En esta función lo que hacemos es básicamente escoger el producto que queremos tomar, entonces tenemos en el monitor LCD los distintos productos acompañado de sus precios, y con el Joystick, vamos moviendo para arriba y para abajo para poder seleccionar luego pulsando este mismo qué producto queremos para entrar en el estado de BEBIDA_SELECCIONADA.

- BEBIDA_SELECCIONADA: Una vez que tenemos la bebida, comenzamos con su preparación, la cual tiene un tiempo aleatorio de entre 4-8 segundos, durante el tiempo que sea sale en la pantalla LCD  PREPARANDO CAFE... y a su vez el LED2, va aumentando su intensidad conforme va avanzando el tiempo de preparación y ya después de este pasamos al último estado del cliente.

- PREPARADO: El cual nos indica que podemos retirar nuestra bebida y retorna al MODO-SERVICIO esperando a que aparezca un nuevo cliente.

 Todos estos que he descrito anteriormente forman parte del modo-cliente, pero luego hay otro modo usuario, el cual se puede activar en cualquier momento, 
 manteniendo pulsado el botón durante unos cinco segundos, cuando llegamos o superamos los cinco segundos se activan los dos LEDS a la vez y entramos en el modo administrador el cual contiene estos estados:

 - ADMIN: En este estado, ya estamos dentro del modo admin y lo que tenemos es una pantalla de menú completamente distinta a la anterior, la cual muestra las distintas funciones que tiene el administrador y que explicaré después con más detalle, para ver el menú nos movemos con el Joystick y seleccionamos la función pulsando este también, luego en cada uno de los estados tengo una funcion, salir(), que sirve para poder volver a este menú y consiste en que si movemos el joystick para la izquierda nos retorne a este estado.

 - TEMPERATURA: Esta función lo que hace es mostrarnos la temperatura y la humedad que hay en el ambiente en tiempo real, para ello lo que utilizo es el thread de MONITOR_Y_CATALOGO.

 - DISTANCIA: Lo que hace es mostrarnos la distancia que hay de nosotros al sensor en tiempo real, sin que nos expulse al superar el rango de la distancia a no ser que el administrador se lo pida.

 - CONTADOR: El contador lo que nos muestra es el tiempo que lleva el arduino encendido, lo que lleva el programa activo.

 - MOD_PRECIOS: es una función la cual nos permite cambiar los precios, entonces aquí hacemos el mismo movimiento que en NAVEGACION_Y_SELECCION y una vez que escogemos el producto pasamos a la segunda parte de este estado, ya que a este lo acompaña otro estado.

 - SUMAR_Y_RESTAR: Una vez que escogemos el producto, el LCD nos muestra el precio y luego nosotros subimos o bien bajamos el precio 0.05 cent, si no queremos aplicar el cambio lo que hacemos es mover el Joystick a la izquierda y sino lo confirmamos pulsando este, guardando así el cambio y volviendo otra vez a MOD_PRECIOS.

Si en algún momento quisiéramos salir del modo admin, en cada uno de estos estados, incluido el ADMIN llamamos a una función la cual hace lo mismo que el thread de admin pero con la función opuesta y es que si nos encontramos en alguno de estos estados y mantenemos el botón pulsado otra vez cinco segundos entonces salimos del modo administrador y por lo tanto también se apagan los dos LEDS.
</p>

*Esquemático de la transición de los distintos estados*

<img width="1055" height="1045" alt="Screenshot from 2026-11-22 11-07-32" src="https://github.com/user-attachments/assets/08557982-8871-4379-9b69-a689180258dc" />

##### FUNCIONES NUEVAS USADAS Y PORQUE
<p>
Para este programa, algunas de las funciones nuevas que he utilizado son:

Threads -> Los threads lo que hace es que nos ayudan a que podamos ejecutar algunas funciones en paralelo, mientras va corriendo el programa, entonces yo por ejemplo los he utilizado para la función del sensor de distancia, para que vaya midiendo cada cierto tiempo la distancia que tiene el cliente con el sensor y así, si el cliente se va en algún punto, reiniciar la máquina a modo servicio, también lo he utilizado para la función de temperatura para que vaya midiendo esta de forma constante y por último para el administrador, ya que podemos entrar en este modo en cualquier momento de la ejecución, entonces lo que hace es que va comprobando si se está pulsando o no el botón para entrar en el estado.

Watchdog-> Esta funcion la uso basicamente para depurar, es decir, por si el programa tiene algun tipo de error o de falla con algun bucle en el que se quede pillado, como por ejemplo puede pasar con funciones como pulseIn(), pues que si supera un periodo de tiempo lo expulse

TimerOne -> es una función la cual nos ayuda a utilizar un temporizador que viene incorporado en el arduino, entonces yo lo he utilizado para la función contador, tengo una función que es empezarContador(), y otra que es pararContador(), en empezarContador lo que hago es que lo inicializo en 1 segundo al igual que la variable segundo que la inicializo con micro(), para tener ambas como punto de partida, luego hacemos una interrupción para incrementar el valor de segundo y en pararContador quitamos la interrupción y en mi caso como el pin influye sobre el LED2, entonces cuando trabajo para timer1 tiene que hacer una cosa y cuando uso el LED otra y que no se pisen aplicó estas líneas:


```
TCCR1A = _BV(WGM10);
TCCR1B = _BV(CS11) | _BV(CS10);
```
WGM10-> nos dice cómo tiene que contar el timer

CS11/CS10->elije la velocidad del timer

_BV()-> encender un interruptor, y al ponerle la | lo que hacemos es que encendemos varios a la vez
Para de esta forma restaurar el timer1 a analogWrite, básicamente ponerlo a la velocidad normal del Arduino
</p>

##### IMAGENES DEL MONTAJE DEL ARDUINO


<img width="3000" height="2160" alt="20261122_134039(1)" src="https://github.com/user-attachments/assets/bc6ca410-1450-491b-93d6-0e942dedc8e7" />

En esta primera imagen podemos ver el LCD y el joystick y el pulsador, el cual esta entre el LCD y el sensor de temperatura y de humedad.


<img width="3000" height="2042" alt="20261122_133945(1)" src="https://github.com/user-attachments/assets/77646ca0-c6e6-47bc-82d9-cb7e5ba4c15f" />

Y en esta segunda imagen podemos ver el sensor de temperatura y humedad, el sensor de distancia por ultrasonidos y los dos LEDS, rojo y amarillo.


##### VIDEO DEL FUNCIONAMIENTO

*FUNCIONAMIENTO EN LOS ESTADOS DEL CÓDGIO CLIENTE*

https://github.com/user-attachments/assets/c40dbe7e-c2c4-4634-a83b-c05751278cf0





https://github.com/user-attachments/assets/7ed5b24f-37a1-4ac4-8333-530de78e12dc



*FUNCIONAMIENTO EN LOS ESTAODS DEL CÓDIGO ADMIN*


https://github.com/user-attachments/assets/70431547-a046-4fb7-8567-b0180c23c253


https://github.com/user-attachments/assets/1da59800-633f-46f6-bd58-7e637cadc972


https://github.com/user-attachments/assets/5df9c7b9-79fe-4f7a-95d8-97673d87cf5e


Nota sobre los videos: Al comenzar a grabar creo que no grabe los tres parpadeos del cargando, y solo grabe dos, pero en verdad realiza los tres parpadeos y luego he tenido que recortar los videos, ya que en uno solo ambos videos pesaban demasiado, pero los videos estan puestos en el orden correspondiente.

##### ESQUEMATICO DEL ARDUINO


<img width="1997" height="1064" alt="Screenshot from 2026-11-22 18-42-21" src="https://github.com/user-attachments/assets/492f6fb2-2123-460e-b7e0-d3a21ca67341" />


*Descripción de algunos pines:*
- Cables rojos +, Cables negros -
- LCD: (3,4,5,6,11,12). Cables cian.
- Joystick: Vertical-> A2, Horizontal-> A1, Z-> A0. Cables marrones.
- Sensor de temperatura y humedad: 13. Cable gris.
- Sensor de ultrasonidos: Tring-> 7, Echo -> 8. Cables Amarillos.
- Pulsador: 2. Cable naranja.
- LED1: 9. Cable verde. 
- LED2: 10. Cable verde.


##### PROBLEMAS QUE HE TENIDO CON LA PRACTICA
<p>
Los principales problemas que me he encontrado en esta práctica han sido, principalmente situar los ejes del joystick, ya que tienen una orientación distinta a la que pensaba ya que el eje y, sería el pin x y el eje x, el pin y, por lo que esto me retrasó mucho a la hora de avanzar.

 Otro problema que he tenido ha sido que por lo visto la función timerOne necesita trabajar con los pines 9 y 10, que son pines que he tenido que usar del arduino, entonces me ha costado mucho el encontrar la forma de poder usarlo tanto para el led como para el contador.

 Y por último que me hubiese gustado que fuese más visible el hardware, es decir, en mi opinión el material veo que está demasiado ajustado, sobre todo el cableado, entonces por ejemplo a veces que tengo que mover el Joystick, tocó un cable que forma parte del LCD sin querer y se pierde la visión del mensaje, por lo que tengo que reiniciar el programa para que se pueda ver bien de nuevo.
</p>
