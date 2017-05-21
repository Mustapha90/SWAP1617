# Práctica 5. Replicación de bases de datos MySQL

## 1. Creación de una base de datos en el MASTER 

Accedemos a la linea de comandos de ``mysql``:

``$ mysql -u root -p``

Creamos una base de datos ``contactos`` y dentro de esa base de datos creamos una tabla ``datos``

```sql

mysql> create database contactos;

mysql> use contactos;

mysql> create table datos(nombre varchar(100),tlf int);

```

Ahora insertamos una entrada en la tabla datos:


```sql
mysql> insert into datos(nombre,tlf) values ("pepe",95834987);
```

Comprobamos que los datos se han insertado correctamente:

```sql
mysql> select * from datos;
```

Para mostrar las tablas de la base de datos, ejecutamos:

```sql
mysql> show tables;
```

Para mostrar el equema de la tabla creada, ejecutamos:

```sql
mysql> describe datos;
```

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/p5cap1.png)

## 2. Replicar una BD MySQL con mysqldump

### 2.1 Creación de la copia de seguridad en la maquina1 "MASTER"

MySQL ofrece la posibilidad de realizar copias de seguridad de bases de datos utilizando la herramienta mysqldump, que es muy útil cuando se trata de bases de datos grandes.

El fichero de copia de seguridad generado por mysqldump contiene comandos SQL para crear
la BD, las tablas y rellenarlas.

Para realizar la copia de seguridad, primero tenemos que bloquear temporalmente la base de datos para evitar que se acceda a ella mientras se está haciendo la copia de seguridad.

Accedemos otra vez a la linea de comandos de mysql:

``$ mysql -u root –p``

Ejecutamos el siguiente comando:

```sql
mysql> FLUSH TABLES WITH READ LOCK;
```

Para salir ejecutamos:

```sql
mysql> quit;
```

Ahora podemos realizar la copia de seguridad usando mysqldump:

``mysqldump contactos -u root -p > /tmp/contactos.sql``

El comando anterior nos guardará el fichero de la copia de seguridad en la carpeta ``/tmp``

Y para terminar, desbloqueamos las tablas de la base de datos:

```sql
mysql> UNLOCK TABLES;

mysql> quit
```
### 2.2 Restauración de la copia de seguridad en la maquina2 "SLAVE"

En la máquina 2 copiamos el fichero de la copia de seguridad desde la máquina 1 usando el comando ``scp`` al directorio ``tmp``

``scp 10.0.2.4:/tmp/contactos.sql /tmp/``

Ahora ya tenemos el fichero de copia de seguridad en el directorio ``/tmp/`` de la máquina 2

Antes de restaurar la copia de seguridad, primero debemos crear una base de datos

Accedemos a la interfaz de linea de comandos mysql

``$ mysql -u root –p``

Creamos una base de datos

```sql
mysql> CREATE DATABASE contactos;

mysql> quit;
```

Y por último importamos la base de datos desde la copia de seguridad:

``$ mysql -u root -p contactos < /tmp/contactos.sql``

Comprobamos que la replicación se ha realizado correctamente:

```sql
mysql> use contactos;

mysql> show tables;

mysql> describe datos;

mysql> select * from datos;

```

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/p5cap2.png)


## 3. Replicación de BD automática mediante una configuración maestro-esclavo

mysql ofrece la posibilidad de configurar la replicación automática de bases de datos entre una máquina master y otra slave, lo que es muy útil en entornos de producción reales.

### 3.1 configuración del MASTER

Editamos el fichero de configuración de mysql:

``$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf``

Comentamos el parámetro bind-address que sirve para que escuche a un servidor:

```
#bind-address 127.0.0.1
```

Indicamos la ubicación del fichero de log de errores:

```
log_error = /var/log/mysql/error.log
```

Asignamos un identificadir para el servidor:

```
server-id = 1
```
Le indicamos la ubicación del registro binario:

```
log_bin = /var/log/mysql/bin.log
```

Guardamos la configuración y reiniciamos mysql:

``$ /etc/init.d/mysql restart``

Ahora creamos un usuario y le damos permisos de acceso para la replicación, este usuario será usado por el slave para poder acceder a los datos:

Ejecutamos las siguientes sentencias ``sql``

```sql
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';

mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';

mysql> FLUSH PRIVILEGES;

mysql> FLUSH TABLES;

mysql> FLUSH TABLES WITH READ LOCK;
```

Y por último, mostramos el estado del master:

``mysql> SHOW MASTER STATUS;``

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/p5cap3.png)

Apuntamos el atributo ``File`` y ``Position``, estos atributos se usarán en la configuración del esclavo, en nuestro caso son:

```
File: mysql-bin.000002

Position: 154
```
Ya hemos terminado con la configuración del ``MASTER``

### 3.2 Configuración del SLAVE

Accedemos al fichero de configuración mysql, y realizamos la misma configuración que hicimos en el MASTER, la única diferencia será la asignación del identificador del servidor, en este caso vamos a usar el identificador ``2``

```
server-id = 2
```

Ahora ejectamos la siguiente sentencia sql para que el ``SLAVE`` pueda acceder a la base de datos ``MASTER``:

```
mysql> CHANGE MASTER TO MASTER_HOST='10.0.2.4',
MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=154,
MASTER_PORT=3306; 
```
Donde:

``10.0.2.4`` es la dirección IP del MASTER

``MASTER_USER`` y ``MASTER_PASSWORD`` son el usuario y contraseña del usuario de la base de datos ``MASTER`` creado en el apartado anterior.

``mysql-bin.000002`` y ``MASTER_LOG_POS=154`` son los atributos del fichero del log del MASTER que hemos apuntado en el apartado anterior.

``MASTER_PORT=3306;``, es el puerto usado para acceder a los datos.



Por último arrancamos el servidor ``SLAVE``:

``mysql> START SLAVE;``

Comprobamos que la configuración se ha realizado correctamente ejecutando:

``mysql> SHOW SLAVE STATUS\G``


![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/p5cap4.png)

Nos fijamos en el parámetro ``Seconds_Behind_Master``, cuando es igual a 0 indica que todo está funcionando correctamente.

### 3.3 Prueba de la replicación

Para probar la replicación, nos vamos al servidor ``MASTER`` y insertamos nuevos datos en la base de datos:

Primero desbloqueamos las tablas:

``mysql> UNLOCK TABLES;``

Insertamos algunos datos:

```
insert into datos(nombre,tlf) values ("Mustapha",603297721);

insert into datos(nombre,tlf) values ("Juan",675433234);

insert into datos(nombre,tlf) values ("Alberto",654321233);

```

Comprobamos que los datos insertados se han replicado en el servidor ``SLAVE``

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/p5cap5.png)

Y por último comprobamos que el parámetro ``Seconds_Behind_Master`` sigue siendo igual a 0

![Imagen 6](http://i1210.photobucket.com/albums/cc420/mj4ever001/p5cap6.png)













