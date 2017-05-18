# Práctica 4. Asegurar la granja web

## 1. Configuración del acceso por HTTPS 

Para configurar el acceso por HTTPS a las máquinas servidoras tenemos que crear primero un certificado autofirmado, usando los siguientes comandos:

```
a2enmod ssl

service apache2 restart

mkdir /etc/apache2/ssl

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```

El último comando nos pedirá una serie de datos para configurar el dominio, después de introducir estos datos se generará el certificado en la carpeta ``/etc/apache2``

Ahora pasamos a configurar apache para permitir el acceso por HTTPS editando el fichero ``/etc/apache2/sites-available/default-ssl``

```
$ nano /etc/apache2/sites-available/default-ssl
```

Localizamos la directiva SSLEngine, justo por debajo de la directiva especificamos las rutas de los ficheros del certificado ``apache.crt`` y la clave generada ``apache.key``


```
SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
```
Activamos el sitio default--ssl y reiniciamos apache:

```
a2ensite default-ssl

service apache2 reload
```

Una vez reiniciado Apache, podemos probar el acceso por HTTPS usando el navegador:

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap1.png)

El navegador marca nuestro servidor como no seguro, lo que es normal ya que estamos usando un certificado autofirmado.

Podemos también probar el acceso HTTPS usando el comando cURL:

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap2.png)

Hemos usado el argumento ``--insecure`` para evitar los avisos de seguridad.

## 2. Configuración del cortafuegos

Para asegurar nuestra granja web tenemos que configurar el cortafuego usando la herramienta ``iptables`` para denegar accesos indebidos, permitiendo solamente el acceso por los puertos que nosotros definimos, en nuestro caso van a ser el puerto 80 (HTTP), el 443 (HTTPS) y el 22 (SSH).

Creamos un script con los comandos necesarios para configurar el cortafuegos:

```shell
# (1) Eliminar todas las reglas (configuración limpia)
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

# (2) Política por defecto: denegar todo el tráfico
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

# (3) Permitir cualquier acceso desde localhost (interface lo)
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# (4) Abrir el puerto 22 para permitir el acceso por SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# (5) Abrir los puertos HTTP (80) de servidor web
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

# (6) Abrir los puertos HTTPS (443) de servidor web
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
```

Guardamos el script, y le damos el permiso de ejecución:

``$ sudo chmod +x script_firewall``

Ejecutamos el script:

``$ sudo ./script_firewall``

Ahora ejecutamos el comando netstat para comprobar que la configuración se ha realizado correctamente:

``$ netstat -tulpn``

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap3.png)

Volvemos a comprobar el acceso por HTTP y HTTPS:

![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap4.png)

Por último, para que la configuración del cortafuegos sea permanente, copiamos el contenido del script de configuración al fichero ``/etc/rc.local``

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap5.png)

## 3. Configuración de una máquina como cortafuegos para la granja web (Segunda tarea opcional)

En este apartado explicaremos como configurar una máquina para funcionar como un cortafuegos que permite el acceso solamente a los puertos HTTP y HTTPS, estos accesos se redireccionan al balanceador que repartirá la carga entre las dos máquinas servidoras.

### 3.1 Configuración del acceso por HTTPS en el balanceador

Para que nginx pueda recibir peticiones HTTPS hay que que seguir los siguientes pasos:

Primero creamos un certificado autofirmado de la misma manera del apartado 1.

Una vez creado el certificado, pasamos a configurar Nginx para aceptar peticiones HTTPS, para ello abrimos el fichero de configuración de nginx:

``$ sudo nano /etc/nginx/conf.d/default.conf``

El fichero de configuración tendrá el siguiente contenido:

```
upstream apaches{
	server 10.0.2.4 weight=2;
	server 10.0.2.15 weight=1;
}

server{
        listen 10.0.2.5:80;
	listen 443 default ssl;
	ssl_certificate      /etc/nginx/ssl/server.crt;
	ssl_certificate_key  /etc/nginx/ssl/server.key;
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

Guardamos los cambios y reiniciamos nginx:

``$ sudo service nginx reload``

### 3.2 Configuración del cortafuegos de la máquina del balanceador

Como todo el tráfico tiene que pasar por la máquina que tendrá configurado el cortafuegos que vamos a llamar ``m5``, tenemos que prohibir el acceso directo al balanceador y permitir solamente los accesos que provienen desde la propia granja, o sea desde la m5, m1 o m2.

Creamos un script para configurar el cortafuegos con el siguiente contenido:

```shell
#!/bin/bash

# Eliminar la configuración actual
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# Denegar todo el tráfico
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP;

# Permitir el acceso al puerto 22 (SSH)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# Permitir tráfico entre el balanceador y la máquina cortafuegos (m5) 
iptables -I INPUT -s 10.0.2.7 -j ACCEPT
iptables -I OUTPUT -d 10.0.2.7 -j ACCEPT

# Permitir tráfico entre el balanceador y la máquina servidora (m1) 
iptables -I INPUT -s 10.0.2.4 -j ACCEPT
iptables -I OUTPUT -d 10.0.2.4 -j ACCEPT

# Permitir tráfico entre el balanceador y la máquina servidora (m2) 
iptables -I INPUT -s 10.0.2.15 -j ACCEPT
iptables -I OUTPUT -d 10.0.2.15 -j ACCEPT
```

Ejecutamos el script para terminar la configuración del balanceador.

### 3.3 Configuración de la máquina cortafuegos

Creamos un script con las reglas necesarias para configurar el cortafuegos:

```
#!/bin/bash

# Eliminar la configuración actual
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# Denegar el tráfico entrante y saliente
iptables -P INPUT DROP
iptables -P OUTPUT DROP

# Permitir el redirecionamiento de tráfico
iptables -P FORWARD ACCEPT

# Permitir el acceso al puerto 22
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# Redireccionar todas las peticiones HTTP al balanceador
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.2.5:80

iptables -t nat -A POSTROUTING -j MASQUERADE

# Redireccionar todas las peticiones HTTPS al balanceador
iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.0.2.5:443

iptables -t nat -A POSTROUTING -j MASQUERADE

```

Ejecutamos el script

``$ sudo ./script_firewall``

Por último, probamos la configuración que hemos realizado, haciendo peticiones a la máquina cortafuegos:

**Por HTTP:**

![Imagen 6](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap6.png)

**Y por HTTPS:**

![Imagen 7](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap7.png)
