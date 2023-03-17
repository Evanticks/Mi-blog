---
title: Journalctl
---

![imagen](/images/jurnal-logo.png)

# Centralizar journalctl en alfa.


Para ello primero debemos actualizar los sistemas y e instalar el siguiente paquete en todo el entorno:

`sudo apt update && sudo apt install systemd-journal-remote -y`

En rocky:

`sudo dnf update && sudo dnf install systemd-journal-remote -y`

Tras esto debemos de iniciar en alfa los servicios de journalctl:


```
systemctl enable --now systemd-journal-remote.socket

systemctl enable systemd-journal-remote.service
```

Y en los clientes debemos de activar el servicio de journalctl:

`systemctl enable systemd-journal-upload.service`

Ahora vamos a editar el fichero `/etc/systemd/system/systemd-journal-remote.service`
De forma que funcione a través de http y no de https ya que estamos dentro de un escenario cerrado.


```

[Unit]
Description=Journal Remote Sink Service
Documentation=man:systemd-journal-remote(8) man:journal-remote.conf(5)
Requires=systemd-journal-remote.socket

[Service]
ExecStart=/lib/systemd/systemd-journal-remote --listen-http=-3 --output=/var/log/journal/remote/
User=systemd-journal-remote
Group=systemd-journal-remote
PrivateTmp=yes
PrivateDevices=yes
PrivateNetwork=yes
WatchdogSec=3min

[Install]
Also=systemd-journal-remote.socket
```


Ahora vamos a crear la carpeta donde vamos a recopilar los logs de los clientes y darle permisos al usuario systemd-journal-remote:

```
mkdir /var/log/journal/remote
chown systemd-journal-remote /var/log/journal/remote
```

Ahora reiniciamos sudo systemctl daemon-reload: esta orden nos permite recargar todos los servicios de nuevo, muy útil por si modificamos varios de ellos, de esta forma, podremos hacerlo de forma global con todos.

`systemctl daemon-reload`

Ahora iniciamos el servicio de recolección de logs en alfa:

`systemctl start systemd-journal-remote.service`

Debemos de crear el uduario systemd-journal-upload en todos los clientes para que se encargue de recopilar los logs y enviarlos a alfa:

```
#En Debian/Ubuntu
adduser --system --home /run/systemd --no-create-home --disabled-login --group systemd-journal-upload 

#En Rocky
adduser --system --home /run/systemd --no-create-home --user-group systemd-journal-upload
```

Tras esto debemos de editar el fichero de journal-upload.conf para que se conecten a alfa:

```
nano /etc/systemd/journal-upload.conf

[Upload]
URL=http://alfa.antonio.gonzalonazareno.org:19532
```

Y por último reiniciamos el servicio de journal-upload en todos los clientes:


`systemctl restart systemd-journal-upload.service`


Para comprobar que todo funciona correctamente, podemos ver en el directorio /var/log/journal/remote/ los logs de los clientes:

![imagen](/images/journal-1.png)

Para ver los logs de los clientes, podemos usar el comando journalctl para cada uno de los logs de los clientes:

charlie:
`journalctl --file=/var/log/journal/remote/remote-192.168.0.2.journal`
![imagen](/images/journal-2.png)

delta:
`journalctl --file=/var/log/journal/remote/remote-192.168.0.2.journal`
![imagen](/images/journal-3.png)

bravo:

`journalctl --file=/var/log/journal/remote/remote-172.16.0.200.journal`
![imagen](/images/journal-4.png)

Debemos realizarlo con este comando y no con un cat ya que lo que se encuentra en la carpeta es un fichero binario, lo cual no está en texto plano.