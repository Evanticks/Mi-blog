
![imagen](/images/forense-logo.png)

La informática forense es el conjunto de técnicas que nos permite obtener la máxima información posible tras un incidente o delito informático.

En esta práctica, realizarás la fase de toma de evidencias y análisis de las mismas sobre una máquina Linux y otra Windows. Supondremos que pillamos al delincuente in fraganti y las máquinas se encontraban encendidas. Opcionalmente, podéis realizar el análisis de un dispositivo Android.

Sobre cada una de las máquinas debes realizar un volcado de memoria y otro de disco duro, tomando las medidas necesarias para certificar posteriormente la cadena de custodia.

Debes tratar de obtener las siguientes informaciones:

# Apartado A) Máquina Windows.

Por comandos:

## 1. Procesos en ejecución.

En la máquina linux, tras haber instalado volatility, ejecutamos el comando sobre el volcado de memoria de la máquina windows:

`python3 vol.py -f /home/vagrant/memwin7.mem windows.pslist`

![imagen](/images/forense-1.png)

## 2. Servicios en ejecución.

`python3 vol.py -f /home/vagrant/memwin7.mem windows.getservicesids.GetServiceSIDs`

![imagen](/images/forense-2.png)

## 3. Puertos abiertos.

`python3 vol.py -f /home/vagrant/memwin7.mem windows.netscan`

![imagen](/images/forense-3.png)


## 4. Conexiones establecidas por la máquina.



## 5. Sesiones de usuario establecidas remotamente.

`python3 vol.py -f /home/vagrant/memwin7.mem windows.sessions.Sessions`

![imagen](/images/forense-5.png)

## 6. Ficheros transferidos recientemente por NetBios.



## 7. Contenido de la caché DNS.



## 8. Variables de entorno.

`python3 vol.py -f /home/vagrant/memwin7.mem envars`

![imagen](/images/forense-8.png)


Analizando el Registro de Windows:

## 9. Dispositivos USB conectados

![imagen](/images/forense-9.png)

## 10. Redes wifi utilizadas recientemente.

Para mostrar las conexiones realizadas recientemente, vamos al registro a la ruta /Network/Connections y veremos la información en formato hexadecimal:

![imagen](/images/forense-10.png)



## 11. Configuración del firewall de nodo.

![imagen](/images/forense-11.png)

## 12. Programas que se ejecutan en el Inicio.


## 13. Asociación de extensiones de ficheros y aplicaciones.

![imagen](/images/forense-13.png)

## 14. Aplicaciones usadas recientemente.

Si nos vamos a el spartado run programs podemos ver los primeros los que se han ejecutado recientemente.

![imagen](/images/forense-14.png)

## 15. Ficheros abiertos recientemente.

![imagen](/images/forense-15.png)

## 16. Software Instalado.

Nos vamos a installed programs y podemos ver una lista con los programas instalados.

![imagen](/images/forense-16.png)

## 17. Contraseñas guardadas.


![imagen](/images/forense-17-1.png)


## 18. Cuentas de Usuario

![imagen](/images/forense-18.png)

## 19. Historial de navegación y descargas. Cookies.

![imagen](/images/forense-19-2.png)
![imagen](/images/forense-19-3.png)
![imagen](/images/forense-19-4.png)


## 20. Volúmenes cifrados

![imagen](/images/forense-20.png)

Sobre la imagen del disco:

## 21. Archivos con extensión cambiada.

Ponemos el tipo de archivo en la herramientas de búsqueda por atributos.

![imagen](/images/forense-21.png)


## 22. Archivos eliminados.

En el programa en el apartado de archivos eliminados podemos encontrar el fichero con contraseña que borré de la máquina.

![imagen](/images/forense-22.png)

## 23. Archivos Ocultos.

![imagen](/images/forense-23.png)


## 24. Archivos que contienen una cadena determinada.

He buscado por palabras clave y he encontrado el ficher oculto.

![imagen](/images/forense-24.png)

## 25. Búsqueda de imágenes por ubicación.

![imagen](/images/forense-25.jpg)


## 26. Búsqueda de archivos por autor.



# Apartado B) Máquina Linux.

Intenta realizar las mismas operaciones en una máquina Linux para aquellos apartados que tengan sentido y no se realicen de manera idéntica a Windows.


## 1. Procesos en ejecución.

En la máquina linux, tras haber instalado volatility, ejecutamos el comando sobre el volcado de memoria de la máquina windows:

`python3 vol.py -f /home/vagrant/memwin7.mem windows.pslist`

![imagen](/images/forense-1.png)

## 2. Servicios en ejecución.

`python3 vol.py -f /home/vagrant/memwin7.mem windows.getservicesids.GetServiceSIDs`

![imagen](/images/forense-2.png)

## 3. Puertos abiertos.

`python3 vol.py -f /home/vagrant/memwin7.mem windows.netscan`

![imagen](/images/forense-3.png)


## 4. Conexiones establecidas por la máquina.

En el mismo comando como el anterior podemos ver las conexiones establecidas por la máquina.

## 5. Sesiones de usuario establecidas remotamente.

`python3 vol.py -f /home/vagrant/memwin7.mem windows.sessions.Sessions`

![imagen](/images/forense-5.png)

## 6. Ficheros transferidos recientemente por NetBios.



## 7. Contenido de la caché DNS.



## 8. Variables de entorno.


![imagen](/images/forense-linux-8.png)

## 9. Dispositivos USB conectados

`lsusb`

![imagen](/images/forense-linux-9.png)

## 10. Redes wifi utilizadas recientemente.

cat /etc/NetworkManager/system-connections/* | egrep id

![imagen](/images/forense-linux-10.png)

## 11. Configuración del firewall de nodo.

`iptables -L`

![imagen](/images/forense-linux-11.png)


## 12. Programas que se ejecutan en el Inicio.

systemctl list-unit-files | egrep "enabled"

![imagen](/images/forense-linux-12.png)

## 13. Asociación de extensiones de ficheros y aplicaciones.

![imagen](/images/forense-linux-13.png)

## 14. Aplicaciones usadas recientemente.



## 15. Ficheros abiertos recientemente.

`ls /* -Artls | tail -10`

![imagen](/images/forense-linux-15.png)


## 16. Software Instalado.

`apt list --installed`

![imagen](/images/forense-linux-16.png)

## 17. Contraseñas guardadas.

`cat /etc/shadow`

![imagen](/images/forense-17.png)

## 18. Cuentas de Usuario
Como vemos que múltiples cuentas que no son de usuarios reales, sino que son de programas que contienen permisos para ejecutar comandos como root, podemos filtrar por el shell que en mi caso será zsh.

cat /etc/passwd | egrep zsh

![imagen](/images/forense-linux-18.png)


## 19. Historial de navegación y descargas. Cookies.

![imagen](/images/forense-linux-19-1.png)
![imagen](/images/forense-linux-19-2.png)


## 20. Volúmenes cifrados

![imagen](/images/forense-linux-20.png)

Sobre la imagen del disco:

## 21. Archivos con extensión cambiada.

Ponemos el tipo de archivo en la herramientas de búsqueda por atributos.

![imagen](/images/forense-linux-21.png)


## 22. Archivos eliminados.

En el programa en el apartado de archivos eliminados podemos encontrar el fichero con contraseña que borré de la máquina.

![imagen](/images/forense-linux-22.png)

## 23. Archivos Ocultos.

![imagen](/images/forense-linux-23.png)


## 24. Archivos que contienen una cadena determinada.

He buscado por palabras clave y he encontrado el ficher oculto.

![imagen](/images/forense-linux-24.png)

## 25. Búsqueda de imágenes por ubicación.

![imagen](/images/forense-linux-25.png)


## 26. Búsqueda de archivos por autor.


# Apartado C) Android

En un dispositivo Android, trata de hacer un volcado de memoria y recuperar información de ubicación, llamadas, mensajes, aplicaciones de mensajería, perfiles en redes sociales, etc...

Al hacer el volcado de memoria con Andriller, podemos ver como muestra las cuentas que utiliza el usuario del teléfono:

![imagen](/images/forense-Android-1.png)

Si entramos dentro de la carpeta generada llamada Apps, podemos ver las diferentes aplicaciones activas en el momento del volcado de memoria:

![imagen](/images/forense-Android-3.png)

Si nos vamos por ejemplo dentro de la siguiente carpeta, podemos ver metainformación del emulador que estuvo usando Macarena antes del volcado de memoria:

![imagen](/images/forense-Android-2.png)


He utilizado el teléfono de mi chica porque yo utilizo un Samsung S21, el cual tiene un sistema de seguridad llamado Knox que impide el volcado de memoria del dispositivo.