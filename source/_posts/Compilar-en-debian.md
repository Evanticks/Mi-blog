---
title: Compilar en debian
---

## apt source lvm2
nos metemos en la carpeta lvm2 y leemos el archivo.dsc
luego buscamos el archivo makefile y si no lo tenemos pues hacemos **./configure** , necesitaremos las dependencias así que procuraremos hacer un `apt build-dep lvm2`,
con esto instalará las dependencias, y una vez hecho esto compilamos de dos maneras:
- haciendo un make, una vez compilado realizamos un `make install`
- Otra forma de compilar el código fuente es de esta manera: `dpkg-buildpackage -b`

para diseccionar un paquete .deb utilizamos `ar -x loquesea.deb`
