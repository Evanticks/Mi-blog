---
title: Instalación preseed PXE+APACHE
categories: Sistemas Operativos
---

Vamos a crear la máquina virtual como hicimos anteriormente en [Instalación automatizada basada en medio de almacenamiento extraíble.](https://entrebytes.neocities.org/2022/09/30/preseed/)

En vagrant creamos una máquina que tenga una ip estática 192.168.100.3 que va a ser la tarjeta de red por la que funcionará pxe, en mi caso no creó esa ip y la tuve que escribir a mano en el /etc/network/interfaces, la otra tarjeta de red es la de vagrant-libvirt que será la que nos conecte al exterior.

Para proceder a configurar el protocolo **TFTP** Y **DHCP** para que sea el host el que conceda ip a la máquina y establezca la conexión al preseed, utilizando dnsmasq

apt install dnsmasq


Establecemos en el fichero /etc/dnsmaq.conf con los siguientes parámetros:

`dhcp-range=192.168.100.100,192.168.100.200,255.255.255.0`

Establecemos el fichero con  el que va a bootear el dhcp:

`dhcp-boot=pxelinux.0`

Habilitamos el tftp:
`enable-tftp`

Establecemos una ruta donde va a albergar nuestro Debian:
`tftp-root=/srv/tftp`


Creamos la carpeta en /srv
`sudo mkdir /srv/tftp`

a continuación instalamos el wget y lo usamos en /srv/tftp


`wget http://ftp.debian.org/debian/dists/bullseye/main/installer-amd64/current/images/netboot/netboot.tar.gz
`

`tar -zxf netboot.tar.gz` (para descomprimir los archivos)
`rm netboot.tar.gz`



`srv/tftp/debian-installer/amd64/boot-screens/txt.cfg`

En este fichero de configuración utilizamos los siguientes parámetros:
```
label install
        menu label ^Install
        kernel debian-installer/amd64/linux
        append vga=788 initrd=debian-installer/amd64/initrd.gz --- quiet
label unattended-gnome
        menu label ^Instalacion Debian Desatendida Preseed
        kernel debian-installer/amd64/linux
        append vga=788 initrd=debian-installer/amd64/initrd.gz hostname=preseed domain=preseed preseed/url=192.168.100.5/preseed.cfg locale=en_US.UTF-8 console-setup/charmap=UTF-8 console-setup/ask_detect=false keyboard-configuration/xkb-keymap=us --
```

En este label he puesto la url del servidor apache que vamos a utilizar, para que descargue el preseed y lo inyecte en la instalación.

Instalamos apache2:
`sudo apt install apache2`

luego copiamos el preseed, lo ponemos en `/var/www/html/` junto al index.html, hacemos la página HTML que albergue el preseed y la ruta ya está establecida en el txt.cfg


Una vez hecho esto, para que nuestra máquina que se conecta a la tarjeta de red estática que hemos creado, necesitamos establecer unas reglas de nftables que nos ayudará a conseguir que la máquina que conecte con el servidor pxe salga al exterior para descargar las dependencias.

Para ello, debemos activar el bit de forwarding que se halla en `/etc/sysctl.conf` y descomentamos `#net.ipv4.ip_forward=1`

Instalamos y habilitamos nftables(Todo esto siendo root):

`apt install nftables`
`systemctl start nftables.service`
`systemctl enable nftables.service`
`nft add table nat`
`nft list tables`

ahora realizamos las reglas de nftables para conseguir que esas máquinas tengan internet:
`nft add chain nat postrouting { type nat hook postrouting priority 100 \; }`
`nft add rule ip nat postrouting oifname "eth0" ip saddr 192.168.100.0/24 counter masquerade`

Ahora guardamos los cambios: `nft list ruleset > /etc/nftables.conf`

Ahora en virt-manager procedemos a crear y enlazar una máquina a una red aislada:
```
<network connections="2">
  <name>red_muy_aislada</name>
  <uuid>b0083374-5cb6-4a8d-bd3c-32cf0d870b54</uuid>
  <bridge name="virbr3" stp="on" delay="0"/>
  <mac address="52:54:00:e9:be:50"/>
</network>
```

 debemos arrancarla por red como prioridad, luego de esto funcionará perfectamente nuestra instalación desatendida.
