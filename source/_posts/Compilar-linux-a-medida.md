
---
title: Configurar kernel a medida
categories: Sistemas Operativos
tags: Sistemas Operativos
---

![Descripción de la imagen](/images/linux-6.0.jpg)

Comenzaremos creando una carpeta que nos sirva de entorno de trabajo

`mkdir kernel && cd kernel`


Instalamos las dependencias que nos ayudarán a compilar el kernel, el cual nos ahorrará errores en la compilación:


`sudo apt install build-essential flex nbison openssl libssl-dev dkms libncurses-dev ncurses-dev qtbase5-dev libelf-dev`

Nos vamos ahora a kernel.org a descargarnos la última versión del kernel de linux.

Una vez descargado el kernel lo descomprimimos con el comando tar -xvf linux-6.0.7.tar.gz, ingresamos dentro y vemos que entre muchos archivos, encontramos un makefile, entramos dentro y encontraremos el archivo EXTRAVERSION, el cual pondremos una versión para poder llevar un control de versiones.

Entramos dentro del directorio descomprimido y procedemos a generar el `make oldconfig` que creará el archivo .config con los módulos que deberemos compilar, intentamos contestar a las preguntas de forma negativa para que no genere módulos opcionales.

Una vez hecho esto debemos de adaptar los módulos que está utilizando nuestra máquina para compilar el kernel a medida, entonces utilizaremos un `make localyesconfig`

```
╭─antonio@debian ~/Programas/kernel/linux-6.0.7  
╰─➤  egrep '=y' .config | wc -l                                                                                                 130 ↵
1934
╭─antonio@debian ~/Programas/kernel/linux-6.0.7  
╰─➤  egrep '=m' .config | wc -l
3
```

Una vez hecho esto procederemos a probar nuestro kernel a medida `make -j8 bindeb-pkg` pudiendo así compilar el kernel, le otorgaremos 8 jobs y si agregamos un time `time make -j8 bindeb-pkg` podemos ver la duración que ha tardado el sistema en compilarlo.

Reiniciamos nuestra máquina y entramos en el nuevo kernel, utilizamos `uname -r`

Luego volvemos a nuestro kernel, y en concreto a nuestro espacio de trabajo, ejecutamos un `make clean` para eliminar los 'residuos' generados tras al compilación, y hacemos un control de versiones del .config, como nos funcionó la primera versión realizamos un `cp .config ../v1.config`

![Descripción de la imagen](/images/xconfig.png)