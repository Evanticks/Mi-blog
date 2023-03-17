---
title: "Introducción a Docker"
categories: Contenedores
---

![imagen](/images/docker-logo.png)

# Introducción a Docker

## ¿Qué es Docker?

Docker es una plataforma de código abierto que permite crear, probar y desplegar aplicaciones de forma rápida y sencilla. Docker permite crear contenedores que contienen todo lo necesario para ejecutar una aplicación, incluyendo el sistema operativo, herramientas, librerías y código. Los contenedores son independientes entre sí y se pueden ejecutar en cualquier máquina que tenga Docker instalado.


## Usando volumen

Crearemos un volumen que se almacenará en /var/lib/docker/volumes/

Instrucción para crear el volumen docker.

`docker volume create miweb`

Instrucción para crear el contenedor.

`docker run -d --name apache -v miweb:/var/www/html -p 8081:80 php:7.4-apache`

Podemos ver una imagen accediendo a la página web.

![imagen](/images/docker-taller-1-1.png)


Instrucción para borrar el contenedor.

`docker rm -f apache`

Instrucción para crear de nuevo el contenedor con el volumen y pantallazo accediendo de nuevo a la página.

`docker run -d --name apache -v miweb:/var/www/html -p 8081:80 php:7.4-apache`



## Usando bind

Usando bind podemos elegir una ruta de nuestro host para montarla en el contenedor y así preservar los datos que se quieran guardar.

Instrucción para crear el contenedor.

`docker run -d --name apache -v /home/antonio/directorio:/var/www/html -p 8080:80 php:7.4-apache`
Pantallazo accediendo a la página web.

![imagen](/images/docker-taller1-2.png)

Pantallazo accediendo a la página web, después de cambiar el fichero index.html en tu host.


![imagen](/images/docker-taller1-3.png)

Redes

Instrucción para crear la red

`docker network create rednextcloud`

Instrucción para crear el contenedor de base de datos.
```
docker run -d --name servidor_mysql \
                --network rednextcloud \
                -v /home/antonio/nextcloud-mariadb:/var/lib/mysql \
                -e MYSQL_DATABASE=nextcloud \
                -e MYSQL_USER=nextcloud \
                -e MYSQL_PASSWORD=antonio \
                -e MYSQL_ROOT_PASSWORD=hola \
                mariadb
```
Instrucción para crear el contenedor de nextcloud.

```
docker run -d --name nexcloud \
                --network rednextcloud \
                -v /home/antonio/nextcloud:/var/www/html \
                -e MYSQL_DATABASE=nextcloud \
                -e MYSQL_USER=nextcloud \
                -e MYSQL_PASSWORD=antonio \
                -e MYSQL_HOST=servidor_mysql \
                -p 8080:80 \
                nextcloud
```

Pantallazos accediendo a nextcloud para comprobar que funciona de manera correcta.

![imagen](/images/docker-taller1-4.png)


## Escenarios multicontenedor en Docker (Docker compose)

Instalamos docker compose con el siguiente comando:

`sudo apt install docker-compose`

Creamos el fichero docker-compose.yml

```
version: '3.1'
services:
  nc:
    container_name: servidor_nc
    image: nextcloud
    restart: always
    environment:
      MYSQL_DATABASE: bd_nc
      MYSQL_USER: user_nc
      MYSQL_PASSWORD: asdasd
      MYSQL_HOST: servidor_mysql
    ports:
      - 8085:80
    volumes:
      - nextcloud:/var/www/html
    depends_on:
      - db
  db:
    container_name: servidor_mysql
    image: mariadb:10.5
    restart: always
    environment:
      MYSQL_DATABASE: bd_nc
      MYSQL_USER: user_nc
      MYSQL_PASSWORD: asdasd
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - /opt/mysql_wp:/var/lib/mysql

networks:
  mi_red:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: 1420

volumes:
    nextcloud:
```


Lo levantamos con `docker-compose up -d` estando dentro del directorio en el que se halle el docker-compose.yml


Como podemos apreciar ya está corriendo el contenedor de nextcloud y el de la base de datos:

![imagen](/images/docker-taller2-1.png)



Ahora con docker `docker volume ls` podemos apreciar los volúmenes docker que han sido creados (No los creados por bind):

![imagen](/images/docker-taller2-2.png)


También podemos ver la red creada en el docker compose con el comando `docker network ls`

![imagen](/images/docker-taller2-3.png)


Para borrar un docker-compose podemos usar el comando `docker-compose down -v`, también borrará el volumen creado si no es por bind mount.

![imagen](/images/docker-taller2-4.png)


## Creación de una imagen a partir de un Dockerfile


Pantallazo donde se vea el contenido del fichero Dockerfile.

![imagen](/images/docker-taller3-1.png)

Pantallazo donde se vea el comando que crea la nueva imagen.

`docker build -t evanticks/mi_servidor_web:v1 .`

![imagen](/images/docker-taller3-2.png)

Pantallazo donde se vea el acceso a la página web primera versión.

![imagen](/images/docker-taller3-3.png)

Pantallazo donde se vean las dos imágenes subidas a tu cuenta de Docker Hub.

![imagen](/images/docker-taller3-4.png)

Pantallazo donde se vea el acceso a la página web segunda versión.

`docker run -d -p 8086:80 --name entrebytes evanticks/mi_servidor_web:v2`

![imagen](/images/docker-taller3-5.png)


## Comandos de Docker compose

Otros comandos de Docker compose:

`docker-compose up`: Crear los contenedores (servicios) que están descritos en el docker-compose.yml.
`docker-compose up -d`: Crear en modo detach los contenedores (servicios) que están descritos en el docker-compose.yml. Eso significa que no muestran mensajes de log en el terminal y que se nos vuelve a mostrar un prompt.
`docker-compose stop`: Detiene los contenedores que previamente se han lanzado con docker-compose up.
`docker-compose rm`: Borra los contenedores parados del escenario. Con las opción -f elimina también los contenedores en ejecución.
`docker-compose run`: Inicia los contenedores descritos en el docker-compose.yml que estén parados.
`docker-compose pause`: Pausa los contenedores que previamente se han lanzado con docker-compose up.
`docker-compose unpause`: Reanuda los contenedores que previamente se han pausado.
`docker-compose restart`: Reinicia los contenedores. Orden ideal para reiniciar servicios con nuevas configuraciones.
`docker-compose down`: Para los contenedores, los borra y también borra las redes que se han creado con docker-compose up (en caso de haberse creado).
`docker-compose down -v`: Para los contenedores y borra contenedores, redes y volúmenes.
docker-compose logs: Muestra los logs de todos los servicios del escenario. Con el parámetro -fpodremos ir viendo los logs en "vivo".
`docker-compose logs servicio1:` Muestra los logs del servicio llamado servicio1 que estaba descrito en el docker-compose.yml.
`docker-compose exec servicio1` /bin/bash: Ejecuta una orden, en este caso /bin/bash en un contenedor llamado servicio1 que estaba descrito en el docker-compose.yml
`docker-compose build`: Ejecuta, si está indicado, el proceso de construcción de una imagen que va a ser usado en el docker-compose.yml a partir de los ficheros Dockerfile que se indican.
`docker-compose top`: Muestra los procesos que están ejecutándose en cada uno de los contenedores de los servicios.




