---
title: Docker Django
Categoría: Contenedores
---


![docker](/images/docker-django.png)




# Dockerizar la aplicación Django

## Cambios en la aplicación

En este post vamos a dockerizar la aplicación python de Django:

Para ello lo primero que tenemos que hacer es obtener la aplicación que vamos a dockerizar, la cual procederá de aquí:

https://github.com/Evanticks/docker-django

Tras clonarlo en nuestro entorno de desarrollo, procederemos a cambiar varios aspectos los cuales nos servirán para obtener las variables de entorno al haber importado el os, entonces nos vamos a `django_tutorial/django_tutorial/settings.py` y añadimos las siguientes instrucciones:


```
import os
```

Esta opción es opcional, si lo ponemos luego podremos especificar en el docker-compose.yml los host permitidos para realizar la conexión a nuestra aplicación.

```
ALLOWED_HOSTS = [os.environ.get("ALLOWED_HOSTS")]
```

La base de datos vamos a cambiarla, cuando antes era un sqlite3 ahora es un mariadb, por tanto vamos a poner variables de entorno que afecten a mysql:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ.get("DJANGODB"),
        'USER': os.environ.get('DJANGODB_USER'),
        'PASSWORD': os.environ.get("DJANGODB_PASS"),
        'HOST': os.environ.get('DJANGODB_HOST'),
        'PORT': '3306',
    }
}
```

Una vez hecho esto, nuestra aplicación buscará las variables de entorno que le asignemos al crear el contenedor, lo haremos luego a través de docker-compose.

```
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

STATIC_ROOT es la ruta donde se almacenarán los archivos generados estáticos, en nuestro caso será la carpeta static.

```
CSRF_TRUSTED_ORIGINS = ['http://*.entrebytes.org','http://*.127.0.0.1','https://*.entrebytes.org','https://*.127.0.0.1']
```

En Python, la variable CSRF_TRUSTED_ORIGINS se utiliza en aplicaciones web Django para especificar una lista de orígenes que se consideran confiables para la protección CSRF (Cross-site Request Forgery).

CSRF es un tipo de ataque que se produce cuando un atacante engaña a un usuario para que realice una acción no deseada en una aplicación web. La protección CSRF se utiliza para evitar que los atacantes realicen este tipo de ataques mediante la validación del origen de la solicitud, el cual será nuestro dominio.


Una vez hecho esto, podemos hacer un commit y subirlo a nuestro repositorio, porque al crear el Dockerfile necesitará obtener los archivos de la aplicación modificada.

## Script de migraciones.sh

```
#! /bin/sh

sleep 2
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py createsuperuser --noinput
python3 manage.py collectstatic --no-input
python3 manage.py runserver 0.0.0.0:4000
```

- El contenedor por defecto ejecutará sh
- damos dos segundos antes de realizar alguna acción para que pueda conectarse a la base de datos
- makemigrations crea las migraciones a la base de datos
- creamos un usuario administrador de manera no interactiva
- creamos las colecciones estáticas de la carpeta static
- ejecutamos el servidor de django en el puerto 4000


## Creación del Dockerfile


```
FROM python:3
WORKDIR /usr/src/app
MAINTAINER Antonio Marchán "wildworld14@gmail.com"
RUN pip install --root-user-action=ignore --upgrade pip && pip install --root-user-action=ignore django mysqlclient && git clone https://github.com/Evanticks/docker-django.git /usr/src/app && mkdir static
ADD migraciones.sh /usr/src/app/
RUN chmod +x /usr/src/app/migraciones.sh
ENTRYPOINT ["/usr/src/app/migraciones.sh"]
```


- Se aplicará una imagen base de python3
- directorio donde se alojará la aplicación
- Mantenedor de la imagen
- ignora la petición de permisos de usuario root upgradeando pip de la imagen python e instalando django y mysqlclient a través de pip, a su vez clonamos nuestra aplicación y creamos un directorio llamado static que será donde se genere el contenido estático de la aplicación.
- Enviamos migraciones.sh al directorio de trabajo /usr/src/app
- Le damos permisos de ejecución al archivo migraciones.sh
- Con entrypoint estamos ejecutando el script de migraciones.sh al iniciar el contenedor.


Tras esto, ejecutamos el siguiente comando para crear la imagen y la subiremos a dockerhub:

```
docker build -t evanticks/django_tutorial:v1 .
docker push evanticks/django_tutorial:v1
```

## Creación del docker-compose.yml

```
version: '3.1'
services:
  django-tutorial:
    container_name: django-tutorial
    image: evanticks/django_tutorial:v1
    restart: always
    environment:
      ALLOWED_HOSTS: "*"
      DJANGODB_HOST: django-mariadb
      DJANGODB_USUARIO: django
      DJANGODB_CONTRASENA: django
      NAME: django     
      DJANGO_SUPERUSER_PASSWORD: admin
      DJANGO_SUPERUSER_USERNAME: admin
      DJANGO_SUPERUSER_EMAIL: admin@example.org
    ports:
      - 8082:4000
    depends_on:
      - db_django
  db_django:
    container_name: django-mariadb
    image: mariadb:10.5
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: django
      MARIADB_USER: django
      MARIADB_PASSWORD: django
    volumes:
      - mariadb_data_django:/var/lib/mysql
volumes:
    mariadb_data_django:
```

```
Docker-compose up -d
```

![docker](/images/django-1.png)


Y aquí podemos ver la aplicación funcionando en el entorno de desarrollo:

![docker](/images/django-2.png)

## Puesta en producción de nuestra aplicación


Primer crearemos nuestro virtual host en `/etc/nginx/sites-available/docker-django`:


```
server {
        listen 80;
        listen [::]:80;

        server_name django.entrebytes.org;

        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl    on;
        ssl_certificate /etc/letsencrypt/live/django.entrebytes.org/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/django.entrebytes.org/privkey.pem;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name django.entrebytes.org;

        location / {
                proxy_pass http://localhost:8082;
                include proxy_params;
        }
}

```

Ahora vamos a generar el certificado bookmedik.entrebytes.org, para ello antes debemos poner un CNAME bookmedik.entrebytes.org que señale a nuestro servidor.

Tras esto haremos un stop al servicio nginx y ejecutaremos certbot para generar el certificado, activamos el sitio y reiniciamos nginx:

```
systemctl stop nginx

certbot certonly --standalone -d django.entrebytes.org

ln -s /etc/nginx/sites-available/django-docker /etc/nginx/sites-enabled/

systemctl start nginx
```

A continuación crearemos un directorio donde guardaremos el docker-compose generado y lo ejecutaremos:

```
docker-compose up -d
```

Y tras esto, podemos ver que en nuestro servidor estamos alojando la aplicación completamente funcional:

![docker](/images/django-3.png)