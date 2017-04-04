# Práctica 2. Clonar la información de un sitio web

## Objetivos de la práctica

Los objetivos de la práctica son:

	* Aprender a copiar archivos mediante ssh

	* Clonar contenido entre máquinas

	* Configurar el ssh para acceder a máquinas remotas sin contraseña

	* Establecer tareas en cron

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

## Acceso sin contraseña para ssh

Queremos poder acceder desde la máquina secundaria (máquina 2) a la máquina 1 sin introducir la contraseña, esto es importante cuando se quiere automatizar tareas mediante scripts o cron.

Hay que seguir los siguientes pasos:

### 1. Generación de un par de claves ssh en la máquina secundaria

En la máquina secundaria generamos un par de claves (pública/privada) usando el comando ``ssh-keygen``

``$ ssh-keygen -b 4096 -t rsa``

Donde:

    **-t** : Especifica el tipo de clave que hay que generar, en este caso hemos generado una clave rsa que generará el fichero ``~/.ssh/id_rsa`` para la clave privada y el
fichero ``~/.ssh/id_rsa.pub`` para la clave pública.

	**-b**: Especifica el tamaño de la clave en bytes

El comando nos pedirá una contraseña que es usada para hacer la clave más segura, debemos dejarla en blanco ya que el objetivo es conectar sin contraseña.

### 2. Copia de la clave pública generada al equipo remoto (máquina principal)

Copiamos la clave pública generada en el paso anterior a la máquina principal (máquina 1) usando el comando ``ssh-copy-id``


``$ ssh-copy-id 10.0.2.4``

``10.0.2.4`` es la dirección IP de la máquina remota (máquina 1).

Nos pedirá la contraseña de la máquina remota, la introducimos y listo.

![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/p2cap4.png)

### 2. Prueba del acceso ssh sin contraseña

Por último probamos el acceso sin contraseña que acabamos de configurar usando el siguiente comando:

``$ ssh 10.0.2.4``

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/p2cap5.png)

Como podemos ver en la imagen ya podemos acceder sin contraseña.


## Programación de tareas con crontab

En este apartado vamos a programar una tarea con ``cron`` para que se ejecute cada hora para mantener actualizado el contenido del directorio ``/var/www`` entre las dos máquinas.

Abrimos el fichero ``/etc/crontab`` y añadimos la siguiente linea:

```
0   *   *   *   *   mustapha   rsync -avz -e ssh 10.0.2.4:/var/www/ /var/www/``
```

![Imagen 6](http://i1210.photobucket.com/albums/cc420/mj4ever001/p2cap6.png)



