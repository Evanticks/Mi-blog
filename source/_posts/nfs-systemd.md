---
title: Montaje de volumen nfs a partir de systemd
categories: Sistemas Operativos
---

![status](/images/systemd-title.jpg)


Vamos a empezar en la máquina servidor, que en nuestro caso será alfa, actualizamos el sistema e instalamos los paquetes necesarios para la instalación nfs en la máquina servidora que en nuestro caso será alfa.

```
apt install nfs-kernel-server nfs-common
```

Tras esto vamos a ingresar el nuevo volumen, en el escenario podría ser tanto físico como virtualizado, pero antes de realizar la configuración de systemd necesitaremos darle formato al sistema de archivos, que en nuestro caso será ext4.

`mkfs.ext4 /dev/vdb`

Una vez hecho esto podemos crear el archivo en /etc/systemd/system/srv-compartida.mount, es necesario que pongamos la ruta en la que va a estar la carpeta compartida a partir de '-' en vez de '/' en el nombre del fichero a crear, .mount indica a systemd que es un archivo de montaje.

Ahora vamos a ver la sintaxis del archivo:


```
[Unit]
Description= volumen que va a ser montado para compartir nfs

[Mount]
What= /dev/vdb
Where= /srv/compartida/
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target
```

En [Unit] podemos poner la descripción de lo que va a realizar el servicio
En [Mount] es como una forma diferente de poner una línea del fstab, ya que se especifica el qué se va a montar, donde se va a montar, el tipo de archivo que va a ser y sus propias opciones de montaje.
En [Install] especificamos los usuarios a los que va a ser dirigido este servicio.

Una vez hecho esto vamos a ejecutar `systemctl enable srv-compartida.mount` para activar el servicio permanentemente y `systemctl start srv-compartida.mount` para iniciarlo.

Tras esto si hacemos systemctl status srv-compartida podemos ver que está activo el servicio

![status](/images/systemd-1.png)

Ahora vamos a configurar el servidor nfs, en el cual debemos escribir la siguiente línea en el archivo de /etc/exports, pero antes debe estar creada la carpeta compartida.

`/srv/compartida 172.16.0.200(rw,sync,no_subtree_check,all_squash)`

luego ejecutamos `exportfs -a` para que el servicio nfs lea el fichero de /etc/exports.


Ahora vamos a pasar a la máquina cliente:

crearemos la carpeta en /srv/nfs y escribiremos en el archivo de /etc/systemd/system/srv-nfs.mount lo siguiente:

```
[Unit]
Description= Montaje de carpeta compartida NFS  

[Mount]
What=172.16.0.1:/srv/compartida
Where=/srv/nfs
Type=nfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

Y tras esto:

```
systemctl enable srv-nfs.mount
systemctl start srv-nfs.mount
```

Ahora podemos comprobar que el servicio permanece activo:

![status](/images/systemd-2.png)


Y vemos con un archivo de prueba que el servidor nfs funciona:


![nfs](/images/systemd-3.png)