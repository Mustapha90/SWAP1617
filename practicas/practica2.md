# Práctica 1. Clonar la información de un sitio web

## Objetivos de la práctica

Los objetivos de la práctica son:
	* aprender a copiar archivos mediante ssh
	* clonar contenido entre máquinas
	* configurar el ssh para acceder a máquinas remotas sin contraseña
	* establecer tareas en cron

Para ello hay que configurar las maquinas para que puedan comunicarse entre ellas, esta configuración ya se ha realizado en la [práctica 1](https://github.com/Mustapha90/SWAP1617/blob/master/practicas/practica1.md#configuración-de-red)

## Prueba de la copia de archivos por ssh

Para probar la copia de archivos por ssh vamos a crear un tar del directorio home de la máquina 1 directamente en la máquina 2 combinando el comando tar y ssh con un ``pipeline``.

``$ tar czf - home | ssh 10.0.2.15 'cat > ~/tar.tgz'``

Donde ``home`` es la carpeta home de la máquina 1 y ``10.0.2.15`` es la dirección IP de la máquina 2, usando el comando ``cat > ~/tar.tgz`` conseguimos guardar el fichero transferido en la carpeta home de la máquina 2.

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/p2cap1.png)

## Clonado de una carpeta entre las dos máquinas usando rsync

La copia de archivos por ssh no es adecuada para sincronizar grandes cantidades de información, en este caso podemos usar la herramienta ``rsync`` que está desarrollada para realizar este tipo de tareas.

### 1 Instalación de la herramienta rsync

La herramienta viene instalada en Ubuntu Server 16.04 pero si no está instalada la podemos instalar con los siguientes comandos:

``$ sudo apt-get update``

``$ sudo apt-get install rsync``

### 2 Prueba de la herramienta rsync

La prueba la podemos hacer como usuario root o usuario sin privilegios, la vamos a realizar como usuario sin privilegios porque es una buena práctica usar root sólo cuando sea necesario y este no es el caso.

La prueba consiste realizar un respaldo del espacio web ``/var/www`` de una máquina en otra, para ello hacemos que el usuario si privilegios de cada máquina sea el dueño del espacio web de su máquina usando el comando ``chown``

``sudo chown mustapha:mustapha –R /var/www``

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/p2cap2.png)

Usando ``rsync`` y ``ssh`` copiamos el directorio ``/var/www/`` de la máquina 1 en la máquina 2, el comando lo ejecutamos desde la máquina 2:

``$ rsync -avz -e ssh 10.0.2.4:/var/www/ /var/www/``

Donde ``10.0.2.4`` es la dirección IP de la máquina que queremos copiar su directorio ``/var/www/`` (máquina 1)

Nos pedirá la contraseña del usuario de la máquina 1, la introducimos y listo.

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/p2cap3.png)
