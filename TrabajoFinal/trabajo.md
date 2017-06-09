# Comparación de prestaciones en servidores web

## Resumen

En este trabajo explicaremos los conceptos de servidor web, benchmarking, y hablaremos
sobre la utilidad que nos facilita esta tarea, ab (ApacheBenchmark), también presentaremos una serie de medidas y consejos que hay que tener en cuenta a la hora de hacer el benchmarking. También estudiaremos las características de los 3 servidores web Apache, Nginx, Lighttpd y haremos una comparación de prestaciones de los 3, analizaremos los resultados y sacaremos conclusiones.

## Qué es un servidor web?

Un servidor web es una tecnología de la información que procesa las solicitudes a través de HTTP,
el protocolo de red básica que se utiliza para distribuir información en la World Wide Web. El
término puede referirse a todo el sistema informático, un aparato, o específicamente para el software que acepta y supervisa las peticiones HTTP.

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/trabajo1.png)

Hoy en día, existen muchos servidores de páginas web, tanto libres como privativos. La siguiente figura muestra los servidores web más usados actualmente.

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/trabajo2.png)

Nosotros en este trabajo nos dedicaremos a estudiar y comparar las prestaciones que ofrecen los servidores web: Apache, Nginx, LightHttpd.

## Apache

El servidor web Apache es un software (o programa) que se ejecuta en segundo plano en un sistema operativo. Fue desarrollado por primera vez para trabajar con los sistemas operativos Linux/Unix,
pero se adaptó más tarde a trabajar en otros sistemas, incluyendo Windows y Mac.

La instalación y configuración de Apache en Linux requiere un poco de conocimientos de comandos de linea de órdenes, pero en Windows es más sencillo ya que se puede ejecutar a través de una interfaz gráfica de usuario.

El Núcleo original del Apache es bastante básico y contiene un número limitado de características. Su poder viene de las funcionalidades nuevas que se pueden introducir a través de muchos módulos que están escritos por los programadores y se pueden instalar para ampliar las capacidades del servidor, para añadir un nuevo módulo, todo lo que se necesita hacer es instalarlo y reiniciar el servidor, además, las funcionalidades que no se necesitan se pueden quitar fácilmente, esto se considera una ventaja ya que mantiene el servidor pequeño y ligero, se inicia más rápido, consume menos recursos del sistema, y hace al servidor menos propenso a los agujeros de seguridad.

El servidor Apache también es compatible con los módulos de terceros, algunos de los cuales han sido añadidos como características permanentes. El servidor Apache se integra muy fácilmente con otras aplicaciones de código abierto, tales como PHP y MySQL, lo que lo hace aún más poderoso de lo que ya es.
 
## Nginx

Nginx es un servidor web multiplataforma y de código libre que tiene más de 10 años en internet, y ya ha conseguido una eficiencia muy alta en el uso de recursos y en la velocidad de respuesta al servir peticiones.

Nginx es un servidor web ligero de alto rendimiento, que es un excelente proxy inverso para contenido web o para protocolos de correo electrónico como IMAP o POP3. La primera versión de Nginx apareció en Octubre de 2004 y actualmente este servidor web ya se encuentra en una versión muy estable, y esto ha hecho que muchos grandes sitios web como WordPress.com o SourceForge, hayan decidido abandonar Apache y pasarse a Nginx como servidor web principal.

La principal ventaja de Nginx como servidor web es que consume muy pocos recursos al servir contenido estático, lo que le convierte en una excelente alternativa para funcionar como proxy inverso o como balanceador de carga para otros servidores web optimizando la entrega de contenidos. Lo que hace que Nginx sea diferente a otros servidores web es su arquitectura, que permite responder a millones de peticiones por segundo aprovechando al máximo los núcleos o hilos de ejecución del servidor con una configuración muy simple.

## Lighttpd

Lighttpd es un servidor web seguro, rápido, ligero, y muy flexible que ha sido optimizado para entornos de alto rendimiento. Tiene un uso de memoria muy bajo en comparación con otros servidores web y es ligero sobre el CPU. Su conjunto de características avanzadas (FastCGI, CGI, autenticación, HTTP compression , URL-rewrite y muchas más) le hacen un servidor web perfecto para servidores que sufren problemas de carga.

## Que es el benchmarking de servidores web?

El benchmarking es el proceso de estimación del rendimiento del servidor web con el fin de averiguar si el servidor puede servir suficientemente una alta carga de trabajo, existen varios programas y utilidades que facilitan esta tarea como:

* ApacheBench (o ab)
* Apache JMeter
* Httperf
* OpenSTA

En este trabajo trabajaremos con la herramienta ``apacheBench``

## ApacheBench (ab)

Apache Benchmark (ab) es la utilidad que viene incorporada con el servidor Apache y permite hacer benchmarks (pruebas de rendimiento) de cualquier servidor web. es muy útil cuando necesitamos comprobar el rendimiento de un servidor web, ya sea del hardware o software, o de alguna nueva configuración que hemos realizado.

Para ello, debemos entrar en un terminal y ejecutar el comando (ab) como sigue:

``ab -n 1000 -c 10 http://www.example.com/test.html``

Los parámetros indicados en la orden anterior le indican a la herramienta ``ab`` que solicite la
página con dirección ``http://www.example.com/test.html`` 1000 veces (-n 1000 indica el número de peticiones) y hacer esas peticiones concurrentemente de 10 en 10 (-c 10 indica el nivel de concurrencia).

Hay algunos detalles a tener en cuenta cuando vamos a evaluar el rendimiento de un servidor web, lo más importante es el tiempo medio de respuesta cuando hay un alto número de usuarios haciendo peticiones al servidor, a partir de estos resultados, lo siguiente a evaluar es cómo se comporta el servidor cuando tiene el doble de usuarios, la idea es que un servidor que tarda el doble en atender al doble de usuarios será mejor que otro que al doblar el número de usuarios (la carga) pase a tardar el triple. 

ab no simula con total fidelidad el uso del sitio web que pueden hacer los usuarios habitualmente, en realidad pide el mismo recurso (misma página) repetidamente. La herramienta va bien para testear cómo se comporta el servidor antes y después de modificar cierta configuración, teniendo los datos, podemos averiguar qué efecto tiene esa nueva configuración sobre el servidor web. 

Por supuesto, los usuarios reales no van a solicitar siempre la misma página, por lo que las medidas obtenidas con ab sólo dan una idea aproximada del rendimiento del servidor, pero no reflejan el rendimiento real, es conveniente ejecutar el benchmark en otra máquina diferente a la que hace de servidor web, de forma que ambos procesos no consuman recursos de la misma máquina (veríamos un menor rendimiento). Sin embargo, hay que tener en cuenta que al hacerlo remotamente, introducimos cierta latencia debido a la comunicación, además, cada vez que ejecutemos el test obtendremos resultados ligeramente diferentes, esto es debido a que en el servidor hay diferente número de procesos en cada instante, y además la red puede encontrarse más sobrecargada en un momento que en otro, lo ideal es hacer al menos 30 ejecuciones y sacar resultados en media y desviación estándar. En resumen:

* hay que usar la misma configuración hardware y software en todos los test que
hagamos al servidor
* hay que usar la misma configuración de la red (por ejemplo, mismo puerto a
100Mbps) en todos los tests
* tomar nota de la carga del servidor usando los comandos top o uptime
* hacer varias medidas y obtener medias y desviación estándar
* reiniciar la máquina tras cada test para tener las mismas condiciones
* llevar a cabo tests no sólo con páginas estáticas simples, sino también con scripts CGI y PHP

## Comparación de prestaciones Apache Nginx Lighttpd

En esta sección realizaremos un experimento con el fin de comparar las prestaciones de
Apache, Nginx, Lighttpd

### Configuración del entorno de pruebas

Usaremos la siguiente configuración hardware:

* Equipo: MacBook Pro
* Processor 2x Intel(R) Core(TM)2 Duo CPU T9800 @ 2.93GHz
* Memory 7915MB (1295MB used)
* Operating System: Ubuntu server 16.04
* Resolution 1920x1200 pixels
* OpenGL Renderer GeForce 9400M/integrated/SSE2

Se han creado 3 maquinas virtuales idénticas con la siguiente configuración:

* Ubuntu server 16.04
* 3GB de memoria RAM
* Utilizando los dos cores del ordenador
* Aceleración: VT-x/AMD-V, Paginación anidada

En cada máquina virtual se ha instalado un servidor web, con su configuración por defecto, sin optimizaciones.

Las pruebas que se van a realizar son:

* Medir el tiempo de respuesta
* Medir el número de peticiones por segundo

Ambas pruebas se realizarán usando páginas HTML estáticas con muchas imágenes.

### Prueba 1: El tiempo de respuesta

La prueba consiste en lo siguiente:

1- Un cliente solicita algo al servidor web

2- El servidor web responde con el recurso solicitado

El tiempo transcurrido desde la llegada de la petición hasta la respuesta es el tiempo de
respuesta del servidor.

Para medir el tiempo de respuesta haremos uso de la utilidad ab (Apache Benchmark), a cada servidor le enviaremos 12000 peticiones con un nivel de concurrencia de 50 usuarios a la vez, el siguiente gráfico muestra los resultados de la prueba:

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/trabajo3.png)

Los resultados pueden resultar algo sorprendentes, ya que como podemos ver en el gráfico apache es el que mas tarda, Nginx y Lighttpd tienen un tiempo de respuesta casi igual.

### Prueba 2: Peticiones por segundo

Esta métrica es más importante que la anterior y probablemente la mas importante de todas, ya que pretende medir el número de peticiones que puede manejar el servidor a diferentes niveles de concurrencia, cuanto más grande este número más grande será la capacidad del servidor web a manejar altos niveles de tráfico. El siguiente gráfico muestra el resultado de la prueba:

![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/trabajo4.png)

Como podemos observar en el gráfico el rendimiento de apache baja muchísimo conforme sube el nivel de concurrencia al contrario Nginx y Lighttpd dan un rendimiento mucho más alto que apache.

La causa de este bajo rendimiento de apache es porque Apache crea procesos y hebras para manejar las peticiones que le llegan, lo que provoca un uso de memoria muy elevado, lo que fuerza la máquina a hacer swapping (intercambio de memoria) con el disco duro, además cuando se alcanza el límite de procesos, apache se niega conexiones adicionales lo que afecta mucho su rendimiento.

Nginx funciona de manera diferente a Apache especialmente en el manejo de hebras, Nginx no crea nuevos procesos para cada petición, el número de procesos máximo que se pueda crear lo puede determinar el administrador, cada uno de estos procesos es de un único subproceso, y puede manejar miles de peticiones simultaneas y lo hace de forma asíncrona con una hebra, en lugar de utilizar la programación multihebra que consume mucha memoria.

La siguiente figura muestra el manejo de hebras por Nginx:

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/trabajo5.png)

Por otro lado Lighttpd es un servidor asíncrono, se ejecuta como proceso individual con un solo hilo y sin bloqueo de E/S.

Lighttpd y Nginx son muy parecidos y tienen características similares. Lighttpd se ejecuta
como un solo proceso con un solo hilo y sin bloqueo de E/S, por otro lado Nginx trabaja como
un solo proceso maestro, pero delega su trabajo a otras hebras que crea.

## La ventaja de Apache contra Nginx y Lighttpd

Aunque el rendimiento de apache contra Lighttpd y Nginx no fue bueno, Apache tiene una ventaja sobre Nginx y Lighttpd. Apache viene con soporte incorporado para PHP, Python, Perl y otros lenguajes por ejemplo el mod-python y mod-php que son módulos de apache, procesan el codigo PHP y Perl dentro del proceso de apache, mod-python es más eficiente que usar CGI o FastCGI porque no tiene que cargar el interpretador de python para cada petición lo que ahorra mucho tiempo y recursos. Del mismo modo los módulos ``mod-rails`` y ``mod-rack`` dan la habilidad Apache para ejecutar código ``Ruby``.

## Conclusiones

Lighttpd y Nginx con un rendimiento parecido fueron los dos mejores que apache, pero no olvidamos que Apache tiene la ventaja de soporte de módulos, también hay que tener en consideración que las pruebas se han hecho con la configuración por defecto de los 3 servidores web, los 3 se pueden optimizar y configurar para ofrecer mayor rendimiento. Aunque tenemos estos resultados no podemos decir con certeza que servidor web es el mejor, porque esto depende de las necesidades de cada uno, así que estos resultados sirven como orientación a la hora de elegir un servidor web o otro, y hay que elegir el que más se adapta a la situación.

Por la limitación en el tiempo no se ha podido usar otros generadores de peticiones o hacer más pruebas de rendimiento como el uso de memoria, el Throughput (número de bytes transferidos por segundo) etc, estas pruebas son también recomendables a la hora de hacer benchmarking de
 servidores web.



