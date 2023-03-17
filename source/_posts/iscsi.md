---
title: iSCSI
categories: Almacenamiento
---

![Descripción de la imagen](/images/iscsi-logo.png)

## Servidor

Actualizamos los paquetes e instalamos:

```
apt update
apt install tgt
```

este programa funciona con targets, crearemos uno:
`sudo tgtadm --lld iscsi --op new --mode target --tid 1 -T iqn.2021-11.org.example:target1`

Añadimos una unidad lógica llamada LUM:
`sudo tgtadm --lld iscsi --op new --mode logicalunit --tid 1 --lun 1 -b /dev/vdb`

![Descripción de la imagen](/images/iscsi-1.png)

`sudo tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL`

para hacerlo persistente:

`tgt-admin -s`
`tgt-admin --dump > /etc/tgt/conf.d/example.es.conf`



## Cliente

Escaneamos los dispositivos iSCSI disponibles, he empleado el nombre servidor porque Vagrant tiene resolución interna de nombres:

```
sudo iscsiadm --mode discovery --type sendtargets --portal servidor


iscsiadm --mode node -T iqn.2021-11.org.example:target1 --portal servidor --login
```


Luego le damos formato al nuevo dispositivo:

```
sudo mkfs.ext4 /dev/sda
mkdir /mnt/iscsi
```

lo montamos en /mnt/iscsi


![Descripción de la imagen](/images/iscsi-3.png)

Para hacerlo persistente:

```
sudo iscsiadm --mode node -T iqn.2021-11.org.example:target1 --portal servidor -o update -n node.startup -v automatic
```

`sudo nano /etc/systemd/system/mnt-iscsi.mount`
```
[Unit]
Description=Disco iSCSI cliente

[Mount]
What=/dev/sda
Where=/mnt/iscsi
Type=ext4
Options=_netdev

[Install]
WantedBy=multi-user.target

```

Alternativa por si hay que esperar a la red antes de que arranque el disco:

```
[Unit]
Description=Disco iSCSI cliente
Requires=network-online.target
After=network-online.target
[Mount]
What=/dev/sda
Where=/mnt/iscsi
Type=ext4
Options=defaults
[Install]
WantedBy=multi-user.target
```


![Descripción de la imagen](/images/iscsi-2.png)


## Cliente Windows

Primero vamos a crear un nuevo target en el servidor con dos LUM, lo haremos en un fichero de configuración:

`/etc/tgt/conf.d/target2.conf`

```
<target iqn.2021-11.org.example:target2>
    driver iscsi
    controller_tid 2
    backing-store /dev/vdc
    backing-store /dev/vdd
    incominguser antonio pruobandodisco
</target>

```

al reiniciar el servicio este procesará el fichero que hemos creado y creará el target:

`sudo systemctl restart tgt`


comprobamos el nuevo target definido con `sudo tgtadm --lld iscsi --op show --mode target`

```
        LUN: 1
            Type: disk
            SCSI ID: IET     00020001
            SCSI SN: beaf21
            Size: 2147 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/vdc
            Backing store flags: 
        LUN: 2
            Type: disk
            SCSI ID: IET     00020002
            SCSI SN: beaf22
            Size: 3221 MB, Block size: 512
            Online: Yes
            Removable media: No
            Prevent removal: No
            Readonly: No
            SWP: No
            Thin-provisioning: No
            Backing store type: rdwr
            Backing store path: /dev/vdd
            Backing store flags: 
    Account information:
        antonio
    ACL information:
        ALL

```



Posteriormente entramos en el programa isci initiator de windows y nos vamos a la pestaña de discovery:

![Descripción de la imagen](/images/iscsi-win-1.png)

Después nos vamos a targets, y veremos que tenemos los dos tarjets el cual el segundo será el que vayamos a utilizar:

![Descripción de la imagen](/images/iscsi-win-2.png)

Como tenemos que autentificarnos por CHAP iremos a avanzado, activaremos la casilla de log y pondremos el usuario y contraseña que hemos puesto en el servidor: 


![Descripción de la imagen](/images/iscsi-win-3.png)


Ahora podemos ver como está conectado al sistema:


![Descripción de la imagen](/images/iscsi-win-4.png)



Entraremos a la aplicación "create and format hard disk partitions" y marcamos los dos discos como GPT:


![Descripción de la imagen](/images/iscsi-win-5.png)


Ahora podemos ver como tenemos dos discos nuevos sin formatear, quedaría formar un nuevo volumen simple en cada uno y darle el formato de NTFS:


![Descripción de la imagen](/images/iscsi-win-6.png)


Y ya tendremos los dos nuevos discos en formato NTFS para almacenar la información persistente al reiniciar el sistema:.


