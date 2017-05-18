# Práctica 4. Asegurar la granja web

## Configuración del acceso por HTTPS 

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

## Configuración del cortafuegos

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

Por últimos para que la configuración del cortafuegos sea permanente, copiamos el contenido del script de configuración al fichero ``/etc/rc.local``

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/p4cap5.png)

