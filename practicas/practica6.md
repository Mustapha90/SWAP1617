# Práctica 6. Discos en RAID

## Configuración RAID 1 

Es un tipo de RAID por software, es utilizado para garantizar la integridad de los datos: en caso de fallo de un disco duro, es posible continuar las operaciones en el otro disco duro sin ningun problema. No se mejora el rendimiento y los otros discos duros son ocultos. Es necesario tener al menos dos discos duros. 

El RAID 1 comúnmente llamado "mirroring" debido a que éste hace una simple copia del primer disco (por lo que para 2 discos de igual tamaño, obtenemos un espacio de almacenamiento igual al espacio de un solo disco).

En esta práctica vamos a usar dos discos de igual tamaño (4GB) para crear disco RAID 1.

En la configuración de discos de la máquina virtual añadimos los discos:

![Imagen 1](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap1.gif)

Instalamos la herramienta ``mdadm`` necesaria para configurar RAID por software

``$ sudo apt-get install mdadm``

Listamos los discos del sistema usando ``fdisk``:

``sudo fdisk -l``

![Imagen 2](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap2.png)

Apuntamos la ubicación de los discos añadidos, en este caso son ``/dev/sdb`` y  ``/dev/sdc``

Ejecutamos el siguiente comando para crear el disco RAID:

``$ sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc``

``/dev/md0`` es el dispositivo donde se creará el disco RAID

``--level=raid1`` Especifica el tipo de RAID que queremos usar

``--raid-devices=2`` Especifica el número de discos usados para crear el RAID, son 2 discos en nuestro caso

``/dev/sdb`` y ``/dev/sdc`` Los discos con los que se construirá el RAID

![Imagen 3](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap3.png)

Le damos formato al disco RAID usando la herramienta ``mkfs``, el disco tendrá el formato ext2

``$ sudo mkfs /dev/md0``

Creamos un directorio donde se montará la unidad RAID y la montamos en ese directorio

``$ sudo mkdir /dat && sudo mount /dev/md0 /dat``

Ahora comprobamos el estado del disco RAID 1

![Imagen 4](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap4.png)

``$ sudo mdadm --detail /dev/md0``

Como podemos ver en la captura anterior, el disco RAID está configurado correctamente.

Para terminar la configuración del RAID 1 editamos el fichero ``/etc/fstab`` para que el sistema monte el disco RAID automáticamente a la hora de arrancar el sistema, para ello necesitamos el identificador ``UUID`` del disco que podemos obtener de la sigueinte orden:

``ls -l /dev/disk/by-uuid/``

![Imagen 5](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap5.png)

Usando el ``UUID`` obtenido añadimos esta linea al fichero ``/etc/fstab``

```
UUID=367511ac-f6fb-4518-b464-76746b251c6a /dat ext2 defaults 0 0
```

Ahora probamos el disco RAID 1 creando un fichero en el disco.

![Imagen 6](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap6.png)

Simulamos un fallo en alguno de los discos de RAID

``$ sudo mdadm --manage --set-faulty /dev/md0 /dev/sdb``

Quitamos el disco marcado como que ha fallado

``$ sudo mdadm --manage --remove /dev/md0 /dev/sdb``

Intentamos acceder a la información almacenada en el RAID:

``$ cat prueba``

![Imagen 7](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap7.png)

Como se ve en la captura anterior el hecho de un disco ha fallado no causa ningún problema a la hora de acceder a los datos en el RAID.

Comprobamos ahora el estado del RAID:

![Imagen 8](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap8.png)

Como podemos ver solo hay un disco funcionando.

Y por último añadimos el disco retirado:

``$ sudo mdadm --manage --add /dev/md0 /dev/sdb``

Comprobamos que se ha reconstruido correctamente, mostrando el estado el RAID:

``$ sudo mdadm --detail /dev/md0``

![Imagen 10](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap10.png)

``Rebuild status`` indica el porcentaje completado del proceso de reconstrucción, después de esperar un rato y volver a comprobar el estado del RAID, nos indicará que el proceso de reconstrución ha terminado y el disco está funcionando correctamente:

![Imagen 11](http://i1210.photobucket.com/albums/cc420/mj4ever001/p6cap11.png)


