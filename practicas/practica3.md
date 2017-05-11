# Práctica 3. Balanceo de carga

## Nginx como balanceador de carga 

### Instalación y configuración de la máquina virtual

Pirmero instalamos Ubuntu server 16.04 en una máquina virtual como hicimos en la práctica 1

Después instalamos nginx con los siguiente comandos:

```
sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get
autoremove
sudo apt-get install nginx
sudo systemctl start nginx
```

Antes de configurar nginx tenemos que asegurar que la máquina puede comunicarse con las máquinas servidoras, para ello todas las máquinas tienen que estar en la misma red NAT que hemos creado en las prácticas anteriores:

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap1.png)

Además, necesitamos poder comunicarnos con la máquina que hemos creado, desde el anfitrión, para ello añadimos un adaptador solo-anfitrión en la configuración de red de la máquina en virtualbox:

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap2.png)

Ya tenemos la máquina preparada:

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap3.png)

### Configuración de nginx

Abrimos el fichero de configuración de nginx ``/etc/nginx/conf.d/default.conf``

``$ sudo vim /etc/nginx/conf.d/default.conf``

Y ponemos la siguiente configuración:

```
upstream apaches{
	server 10.0.2.4 weight=2;
	server 10.0.2.15 weight=1;
}
server{
	listen 80;
	server_name balanceador;
	access_log /var/log/nginx/balanceador.access.log;
	error_log /var/log/nginx/balanceador.error.log;
	root /var/www/;

	location /
	{
		proxy_pass http://apaches;	
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
	}
}
```

Donde 10.0.2.4 y 10.0.2.15 son las direcciones IP de las máquinas servidoras 1 y 2 respectivamente.

Una vez terminada la configuración, iniciamos nginx:

``$ sudo systemctl start nginx``

Para probar el balanceado, arrancamos las dos máquinas servidoras y lanzamos peticiones desde el anfitrión usando el comando curl:

``$ curl http://balanceador/hola.html``

"balanceador" es el nombre del servidor de la máquina nginx que se ha definido en el fichero hosts en el anfitrión.

![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap4.png)

Con la configuración anterior conseguimos un balanceado de carga round-robin alternando peticiones entre las dos máquinas.

Para conseguir un balanceo con ponderación asumiendo que la máquina 1 tiene el doble de capacidad que la máquina 2, modificamos esta parte del fichero de configuración.

```
upstream apaches{
	server 10.0.2.4 weight=2;
	server 10.0.2.15 weight=1;
}
```
Usando la directiva weight indicamos la ponderación que queremos asignar a cada servidor, en este caso el servidor 1 con dirección IP 10.0.2.4 servirá el doble de peticiones que el servidor 2 10.0.2.15.

Reiniciamos nginx con la nueva configuración.

``$ sudo systemctl restart nginx``

Volvemos a probar el balanceado:

``$ curl http://balanceador/hola.html``

 ![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap5.png)

## Balanceado de carga con haproxy

### Instalación y configuración de la máquina virtual

La instalación y la configuración de red de la máquina virtual es idéntica a la de nginx.

Instalamos haproxy:

``$ sudo apt-get install haproxy``

Ya tenemos la máquina preparada:

![Imagen 6](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap6.png)

### Configuración de haproxy

Abrimos el fichero de configuración de haproxy localizado en ``/etc/haproxy/haproxy.cfg``

``$ sudo vim /etc/haproxy/haproxy.cfg``

Y ponemos la siguiente configuración:

```
global
	daemon
	maxconn 256
defaults
	mode http
	contimeout 4000
	clitimeout 42000
	srvtimeout 43000

frontend http-in
	bind *:80
	default_backend servers

backend servers
	server m1 10.0.2.4 maxconn 32
	server m2 10.0.2.15 maxconn 32
```

Guardamos la configuración y lanzamos el servicio haproxy:

``$ sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg``

Probamos el balanceado de carga haciendo peticiones a haproxy:

``$ curl http://balanceador1/hola.html``

"balanceador1" es el nombre del servidor de la máquina haproxy que se ha definido en el fichero hosts en el anfitrión.

![Imagen 7](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap7.png)

## Balanceado de carga con pound 

### Instalación y configuración de la máquina virtual

La instalación y la configuración de red de la máquina virtual es idéntica las anteriores.

Instalamos pound:

``$ sudo apt-get install pound``

Ya tenemos la máquina preparada:

![Imagen 8](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap8.png)

### Configuración de pound

Abrimos el fichero de configuración de pound localizado en ``/etc/pound/pound.cfg``

``$ sudo vim /etc/pound/pound.cfg``

Y ponemos la siguiente configuración:

```
ListenHTTP
	Address 0.0.0.0
	Port 80
End

Service
	BackEnd
		Address 10.0.2.4
		Port    80
		Priority 1
	End

	BackEnd
		Address 10.0.2.15
		Port    80
		Priority 1
	End
End
```
En el bloque ``ListenHTTP`` especificamos donde escuchará pound las peticiones, usando ``0.0.0.0`` como dirección escuchará en el puerto 80 en todas las interfaces de red de la máquina.

Dentro del bloque ``Service`` podemos añadir tantos bloques BackEnd como servidores tengamos.

Priority indica la prioridad que el balanceador usará para desviar peticiones a una máquina o a otra, usando ``Priority 1`` para las dos máquinas, la carga se repartirá de manera equitativa.

Guardamos el fichero de configuración y abrimos ahora el fichero ``/etc/default/pound``, este fichero contiene una variable llamada ``startup`` que inicialmente es igual 0, si no se cambia su valor a 1 no podemos inciar el servicio pound.

``$ sudo vim /etc/default/pound``

```
# Defaults for pound initscript
# sourced by /etc/init.d/pound
# installed at /etc/default/pound by the maintainer scripts

# prevent startup with default configuration
# set the below varible to 1 in order to allow pound to start
startup=1
```

Salvamos el fichero anterior y lanzamos el servicio pound:

``$ sudo systemctl start pound``

Probamos ahora el balanceado de carga:

``$ curl http://balanceador2/hola.html``
 
![Imagen 9](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap9.png)

## Someter a una alta carga el servidor balanceado

En este apartado usaremos la herramienta Apache Benchmark (ab) para someter el servidor balanceado a una alta carga.

Empezamos con Nginx, usando ab lanzamos 1000 peticiones con un nivel de concurrencia de 10:

``$ ab -n 1000 -c 10 http://balanceador/index.html``

![Imagen 12](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap12.png)

Hacemos lo mismo con haproxy:

``$ ab -n 1000 -c 10 http://balanceador1/index.html``

![Imagen 13](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap13.png)

Y por último con pound:

``$ ab -n 1000 -c 10 http://balanceador2/index.html``

![Imagen 14](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap14.png)

**Resultados del experimento:**

Para comparar los resultados he usado el número de peticiones por segundo como criterio, ya que es el criterio más importante.

![Imagen 15](http://i1210.photobucket.com/albums/cc420/mj4ever001/p3cap15.png)

Como se ve en la gráfica, el mejor resultado lo hemos obtenido con nginx.


