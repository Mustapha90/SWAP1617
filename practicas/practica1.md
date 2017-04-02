# Práctica 1. Preparación de las herramientas

##Instalación de las máquinas virtuales

El objetivo de la práctica es instalar y configurar dos máquinas virtuales con Ubuntu Server para poder realizar las prácticas de la asignatura.

He elegido Ubuntu server 16.04, el proceso de instalación es sencillo y se puede consultar en el siguiente [enlace](http://www.ubuntugeek.com/step-by-step-ubuntu-12-04-precise-lamp-server-setup.html)

Lo más destacabe del proceso de instalación es cuando llegamos a la etapa de ``seleccion de programas`` ahí debemos seleccionar el servicio ``OpenSSH server`` para disponer del servicio de acceso remoto ``SSH`` y el paquete ``LAMP`` que instala ``Apache`` y todas las herramientas necesarias para el desarrollo web.

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap1.png)

##Configuración de red

Tras la instalación de cada máquina configuramos la red para que las maquinas podrán comunicarse:

En Virtualbox-->Preferencias-->Red añadimos una red ``NAT``:

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap2.png)

 Ahora bien en las configuraciones de red de las dos maquinas, en la pestaña del primer adaptador elegimos la opción Red NAT y en nombre ponemos el nombre de la red NAT que hemos creado en la configuración general de VirtualBox.

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap3.png)

Ahora las maquinas están en la misma red NAT, tendrán direcciones IP diferentes y por supueto podrán comunicarse.

Arrancamos las dos maquinas y ejecutamos el comando ``ifconfig`` y apuntamos las direcciones IP de las maquinas:

![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap4.png)


##Comprobación de la instalación de apache

Después de la instalación de las máquinas podemos comprobar la versión del servidor instalado ejecutando:

``$ apache2 -v``

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap5.png)

Y para comprobar si está ejecutando:

``$ ps aux | grep apache``

![Imagen 6](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap6.png)

 
##Prueba del servidor

Para probar el servidor instalamos cURL que es una herramienta de línea de comandos para transferir archivos con sintaxis URL que soporta diferentes protocolos.

``$ sudo apt-get update``

``$ sudo apt-get install curl``

Una vez instalada la herramienta cURL creamos un fichero html de prueba en el directorio ``/var/www/``

```html
<HTML>
<BODY>
Prueba!, maquina X
</BODY>
</HTML>
```
Desde la maquina 1 accedemos al fichero ``hola.html`` de la maquina 2 y ``viceversa``, usando el siguiente comando:

``$ curl http://IP/hola.html``


![Imagen 7](http://i1210.photobucket.com/albums/cc420/mj4ever001/swap7.png)


