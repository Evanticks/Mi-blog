---
title: Compilar en debian
---

![Descripción de la imagen](/images/debian-11.jpg)


Necesitamos descargar el código fuente de xorriso:
`apt source xorriso`
Nos pide que instalemos dpkg-dev debido a que necesita el paquete dpkg-source
`apt install dpkg-dev`
Con esto al descargar el código fuente de un paquete este se descomprimirá, también podemos hacerlo a través de tar:

`tar -xvf libisoburn_1.5.2.orig.tar.gz`

![Descripción de la imagen](/images/xorriso-1.png)

nos metemos en la carpeta de xorriso y leemos el archivo.dsc

Como podemos ver en la imagen dentro del paquete libisoburn está las diferentes dependencias para el binario que queremos compilar.


![Descripción de la imagen](/images/xorriso-2.png)

Como podemos observar hay dos makefile, pero estos no son más que unas plantillas que no lograremos compilar ya que solo sirven de guía.


necesitaremos las dependencias así que procuraremos hacer un `apt build-dep xorriso`.


Una vez instaladas las dependencias ejecutamos `./configure --prefix /usr/local/xorriso/`, con esto se habrá generado el makefile, y el binario generado se guardará en /usr/local/xorriso


![Descripción de la imagen](/images/xorriso-3.png)



Ahora una vez tengamos el archivo del makefile, ejecutamos un `make` en el directorio del makefile, una vez compilado realizamos un `make install`


![Descripción de la imagen](/images/xorriso-4.png)

Como podemos ver existe el binario que ejecutaremos y viene incluído el archivo de ayuda.

Otra forma después de proceder con el make a través de dpkg sería `dpkg-buildpackage -b`, se creará elarchivo .deb con el que emplearemos `dpkg -i archivo.deb`