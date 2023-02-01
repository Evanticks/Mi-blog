---
title: Instalación automatizada basada en medio de almacenamiento extraíble.
categories: Sistemas Operativos
---

He comenzado descargando el archivo iso para poder abrirlo y colocar el preseed en él, el comando que he utilizado es xorriso.
Tras esto he utilizado una plantilla de la web de debian

https://www.debian.org/releases/buster/example-preseed.txt

Tras esto he estado configurando preseed.cfg para poner la hora, el país y el teclado en español y resulta que estuve un tiempo estancado en esto porque hay que corroborar los cambios en el archivo isofiles/isolinux/txt.cfg
A su vez también debemos indicar la ruta donde vama a estar el preseed para que el sistema lo localice, en mi caso estará en la raíz de la ISO (CDROM)
```
label install
        menu label ^Install
        kernel /install.amd/vmlinuz
        append vga=788 initrd=/install.amd/initrd.gz --- quiet
label unattended-gnome
 menu label ^Instalación Debian Desatendida Preseed Antonio
 kernel /install.amd/gtk/vmlinuz
 append vga=788 initrd=/install.amd/gtk/initrd.gz preseed/file=/cdrom/preseed.cfg locale=es_ES console-setup/ask_detect=false keyboard-configuration/xkb-keymap=es
```
Al tener que desensamblar y ensamblar constantemente el iso para poner el preseed actualizado llegó un momento en el que se hacía inviable seguir probando sin hacer un script con los pasos que se repetían, entonces lo realicé:

```
#!/usr/bin/env bash
fichero=/home/antonio/Descargas/preseed-instalacion/preseed-debian-10.1.0-amd64-script-netinst.iso
chmod u+w isofiles
cp preseed.cfg ~/Descargas/preseed-instalacion/isofiles
chmod u-w isofiles
cd ~/Descargas/preseed-instalacion
chmod a+w isofiles/isolinux/isolinux.bin
if [ -f $fichero ]
then
        rm -f $fichero
fi      
genisoimage -r -J -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o preseed-debian-10>
```
Tuve que meter la condición de que si el iso creado existía lo borrara porque al ejecutar genisoimage si el nombre ya existe provoca conflictos.

Entonces estuve probando, metiendo el usuario de la máquina con la contraseña sin encriptar, para ver el funcionamiento y que lo anterior configurado fuese por buen camino.

Estuve probando formas de encriptar la contraseña, la plantilla me recomendaba que usara  **Crypt 3** pero es un método muy vulnerable, así que mirando la documentación vi que podía meter un hash **md5**.

Entonces empecé a encriptar la contraseña, para ello utilicé `mkpasswd -m sha-512` el cual al ingresar la contraseña me la devuelve encriptada, activo la opción de `d-i passwd/user-password-crypted password` junto con la contraseña encriptada

Ya entonces, con la localización y el teclado en español, la cuenta del usuario con su contraseña encriptada decido comenzar a realizar las particiones de lvm.
Tuve muchísimos errores de sintaxis pero gracias al script podía probar de manera más rápida las diferentes combinaciones de particionado y ajustando la sintaxis conseguí realizarlo:

![Descripción de la imagen](/images/ASO-PRACTICA1.png)
