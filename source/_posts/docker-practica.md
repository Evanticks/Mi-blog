---
title: Docker
Categoría: Contenedores
---

![imagen](/images/docker-logo-2.png)


## Tarea 1: Creación de una imagen docker con una aplicación web desde una imagen base

- Entrega la url del repositorio GitHub donde tengas los ficheros necesarios para hacer la construcción de la imagen.

https://github.com/Evanticks/bookmedik-docker/tree/main/tarea-1

- Entrega una captura de pantalla donde se vea la imagen en el registro de tu entorno de desarrollo.

![imagen](/images/docker-1.png)


Para esta práctica vamos a comenzar creando una red diferente a la default para que las máquinas que se conecten a ella puedan comunicarse entre sí a través de resolución de nombres.

```
docker network create red_bookmedik
```


Ahora vamos a crear el contenedor de MYSQL, que contendrá los valores por defecto de bookmedik:

```
docker run -d --name mariadb -v bookmedik_vol:/var/lib/mysql --network red_bookmedik -e MARIADB_ROOT_PASSWORD=root -e MARIADB_DATABASE=bookmedik -e MARIADB_USER=bookmedik -e MARIADB_PASSWORD=bookmedik mariadb
```


Ahora vamos a realizar la clonación de la aplicación bookmedik y lo vamos a clonar en nuestro entorno de trabajo:

```
git@github.com:Evanticks/bookmedik.git
```

Después de esto nos vamos al schema.sql y eliminaremos las líneas de creación de la base de datos ya que en el otro contenedor ya la hemos creado.

Seguidamente nos vamos a core/controller/Database.php y modificamos las variables de conexión para que se conecte a la base de datos del contenedor de mariadb.

```
<?php
class Database {
        public static $db;
        public static $con;
        function Database(){
                $this->user=getenv('USUARIO_BOOKMEDIK');$this->pass=getenv('CONTRA_BOOKMEDIK');$this->host=getenv('DATABASE_HOST');$this->ddbb=getenv('NOMBRE_DB');
        }

        function connect(){
                $con = new mysqli($this->host,$this->user,$this->pass,$this->ddbb);
                $con->query("set sql_mode=''");
                return $con;
        }

        public static function getCon(){
                if(self::$con==null && self::$db==null){
                        self::$db = new Database();
                        self::$con = self::$db->connect();
                }
                return self::$con;
        }

}
?>

```


```
sudo nano script.sh

#! /bin/sh

while ! mysql -u ${USUARIO_BOOKMEDIK} -p${CONTRA_BOOKMEDIK} -h ${DATABASE_HOST}  -e ";" ; do
        sleep 1
done
mysql -u $USUARIO_BOOKMEDIK --password=$CONTRA_BOOKMEDIK -h $DATABASE_HOST $NOMBRE_DB < /var/www/html/schema.sql
/usr/sbin/apache2ctl -D FOREGROUND

```

Con este script que añadiremos dentro del contenedor especificaremos que espere a que el contenedor de mariadb esté listo para poder ejecutar el script de creación de la base de datos.


Ahora vamos a construir el contenedor de bookmedik:

```
docker build -t evanticks/bookmedik:v1 .
docker push evanticks/bookmedik:v1
```

![imagen](/images/docker-4.png)


Una vez hecho esto vamos a crear el docker-compose.yml para poder levantar el contenedor de bookmedik y el de mariadb.


## Tarea 2: Despliegue en el entorno de desarrollo

- Entrega la url del repositorio GitHub donde hayas añadido el fichero docker-compose.yml.

https://github.com/Evanticks/bookmedik-docker/tree/main/tarea-2

- Entrega la instrucción para ver los dos contenedores del escenario funcionando.

```
docker ps

docker-compose ps
```

![imagen](/images/docker-2.png)

- Entrega una captura de pantalla donde se vea funcionando la aplicación, una vez que te has logueado.

![imagen](/images/docker-3.png)

---

docker-compose.yaml

```
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: evanticks/bookmedik:v1
    restart: always
    environment:
      USUARIO_BOOKMEDIK: admin
      CONTRA_BOOKMEDIK: admin
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8081:80
    depends_on:
      - db
  db:
    container_name: bd_mariadb
    image: mariadb:10.5.19
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: admin
      MARIADB_PASSWORD: admin
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```


Luego lo ejecutaremos con el siguiente comando:

```
docker-compose up -d
```

Para borrarlo junto con el almacenamiento creado ejecutaremos:

```
docker-compose down -v
```

## Tarea 3: Creación de una imagen docker con una aplicación web desde una imagen PHP

- Entrega la url del repositorio GitHub donde hayas añadido el fichero docker-compose.yml.

https://github.com/Evanticks/bookmedik-docker/tree/main/tarea-3

- Entrega la instrucción para ver los dos contenedores del escenario funcionando.

```
docker ps

docker-compose ps
```

![imagen](/images/docker-5.png)

- Entrega una captura de pantalla donde se vea funcionando la aplicación, una vez que te has logueado.

![imagen](/images/docker-6.png)


Ahora procedamos a explicar los pasos que hemos seguido para realizar esta tarea.

En esta ocasión vamos a editar el dockerfile para que instale una imagen de php-apache y no una imagen de apache solamente, y le otorgaremos unos determinados paquetes para qeu funcione correctamente la conexión con la base de datos.

```
nano Dockerfile

FROM php:7.4-apache-bullseye
MAINTAINER Antonio Marchán Posada "wildworld14@gmail.com"
RUN apt update && apt upgrade -y && docker-php-ext-install mysqli pdo pdo_mysql && apt install mariadb-client -y && apt clean && rm -rf /var/lib/apt/lists/*
ADD bookmedik /var/www/html/
ADD script.sh /opt/
RUN chmod +x /opt/script.sh
ENTRYPOINT ["/opt/script.sh"]
```

- mysqli: Instala la extensión mysqli, que proporciona una interfaz orientada a objetos para interactuar con la base de datos MySQL.
- pdo: Instala la extensión PDO (PHP Data Objects), que proporciona una interfaz consistente para acceder a varias bases de datos, incluyendo MySQL.
- pdo_mysql: Instala el controlador PDO para MySQL, que permite que PHP se comunique con una base de datos MySQL utilizando PDO.


Ahora vamos a construir el contenedor de bookmedik:

```
docker build -t evanticks/bookmedik:v2 .
docker push evanticks/bookmedik:v2
```


Luego vamos a tomar nuestro docker-compose que ya habíamos creado en la tarea anterior y le añadiremos el nuevo contenedor v2.


```
nano docker-compose.yml

version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: evanticks/bookmedik:v2
    restart: always
    environment:
      USUARIO_BOOKMEDIK: admin
      CONTRA_BOOKMEDIK: admin
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8081:80
    depends_on:
      - db
  db:
    container_name: bd_mariadb
    image: mariadb:10.5.19
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: admin
      MARIADB_PASSWORD: admin
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```

Una vez hecho esto, los contenedores funcionarán correctamente.


## Tarea 4: Ejecución de una aplicación PHP en docker con nginx (OPTATIVA)

- Entrega la url del repositorio GitHub donde tengas los ficheros necesarios para hacer la construcción de la imagen.

https://github.com/Evanticks/bookmedik-docker/tree/main/tarea-4

- Entrega una captura de pantalla donde se vea la imagen en el registro de tu entorno de desarrollo.

![imagen](/images/docker-7.png)

- Entrega la instrucción para ver los tres contenedores del escenario funcionando.

![imagen](/images/docker-8.png)

- Entrega una captura de pantalla donde se vea funcionando la aplicación, una vez que te has logueado.

![imagen](/images/docker-9.png)

### Contenedor Nginx

El nginx el cual se encontrará la aplicación web, se encargará de recibir las peticiones de los clientes y de redirigirlas al contenedor de PHP-FPM que crearemos más adelante, que serán los encargados de procesarlas y devolver el resultado al cliente.

Para ello antes debemos crear un fichero de configuración para hacer uso de php-fpm, para ello crearemos un fichero llamado default.conf el cual luego añadiremos a través del Dockerfile a la ruta /etc/nginx/conf.d/default.conf.

nano default.conf

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root   /usr/share/nginx/html;
    index  index.php index.html;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass book_php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

El correspondiente script.sh que se ejecutará al iniciar el contenedor será el siguiente:

```
#! /bin/sh

sleep 10

mysql -u $USUARIO_BOOKMEDIK --password=$CONTRA_BOOKMEDIK -h $DATABASE_HOST $NOMBRE_DB < /usr/share/nginx/html/schema.sql
nginx -g "daemon off;"
```

Ahora empaquetamos y subimos a dockerhub:

```
docker build -t evanticks/bookmedik:v3 .
docker push evanticks/bookmedik:v3
```

### Contenedor PHP-FPM

Ahora ejecutaremos el siguiente dockerfile el cual instalará mysqli que sirve para conectarse a la base de datos.

```
FROM php:7.4-fpm-bullseye
MAINTAINER Antonio Marchán Posada "wildworld14@gmail.com"
RUN docker-php-ext-install mysqli
```

Ahora empaquetamos y subimos a dockerhub:

```
docker build -t evanticks/fpm7.4-mysql:v1 .
docker push evanticks/fpm7.4-mysql:v1
```

### Docker-compose

```
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: evanticks/bookmedik:v3
    restart: always
    environment:
      USUARIO_BOOKMEDIK: admin
      CONTRA_BOOKMEDIK: admin
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8082:80
    depends_on:
      - db
      - php
    volumes:
      - phpdocs:/usr/share/nginx/html/
  db:
    container_name: bd_mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: admin
      MARIADB_PASSWORD: admin
    volumes:
      - mariadb_data:/var/lib/mysql
  php:
    container_name: book_php
    image: evanticks/fpm7.4-mysql:v1
    restart: always
    environment:
      USUARIO_BOOKMEDIK: admin
      CONTRA_BOOKMEDIK: admin
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    volumes:
      - phpdocs:/usr/share/nginx/html/

volumes:
    mariadb_data:
    phpdocs:
```

## Tarea 5: Puesta en producción de nuestra aplicación

Para poner en producción nuestra aplicación debemos de configurar el nginx para que funcione con el certificado SSL de Let's Encrypt.

```
/etc/nginx/sites-available/bookmedik
```

Y pondremos la siguiente configuración en el puerto 8084, el cual después deberemos cambiar en el docker-compose.yml.

```
server {
        listen 80;
        listen [::]:80;

        server_name bookmedik.entrebytes.org;

        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl    on;
        ssl_certificate /etc/letsencrypt/live/bookmedik.entrebytes.org/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/bookmedik.entrebytes.org/privkey.pem;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name bookmedik.entrebytes.org;

        location / {
                proxy_pass http://localhost:8084;
                include proxy_params;
        }
}
```

Ahora vamos a generar el certificado bookmedik.entrebytes.org, para ello antes debemos poner un CNAME bookmedik.entrebytes.org que señale a nuestro servidor.

Tras esto en la VPS ejecutaremos el siguiente comando:

```
systemctl nginx stop
certbot certonly --standalone -d bookmedik.entrebytes.org
systemctl nginx start
```

Por último para que descargue y ponga en producción los contenedores ejecutaremos:

```
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: evanticks/bookmedik:v2
    restart: always
    environment:
      USUARIO_BOOKMEDIK: admin
      CONTRA_BOOKMEDIK: admin
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8084:80
    depends_on:
      - db
  db:
    container_name: bd_mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: admin
      MARIADB_PASSWORD: admin
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:

```

```
docker-compose up -d
```

Y con esto ya lo tendríamos en producción.

- Entrega una captura de pantalla de Docker Hub donde se vea tu imagen subida.

![imagen](/images/docker-10.png)

- Entrega la configuración de nginx.

```
server {
        listen 80;
        listen [::]:80;

        server_name bookmedik.entrebytes.org;

        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl    on;
        ssl_certificate /etc/letsencrypt/live/bookmedik.entrebytes.org/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/bookmedik.entrebytes.org/privkey.pem;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name bookmedik.entrebytes.org;

        location / {
                proxy_pass http://localhost:8084;
                include proxy_params;
        }
}
```

- Entrega una captura de pantalla donde se vea funcionando la aplicación, una vez que te has logueado.


![imagen](/images/docker-11.png)

## Tarea 6: Modificación de la aplicación

- Entrega una captura de pantalla de Docker Hub donde se vea tu imagen subida.

![imagen](/images/docker-13.png)

- Entrega una captura de pantalla donde se vea funcionando la aplicación, una vez que te has logueado.

En mi caso lo pongo antes para que se pueda apreciar el cambio en producción.

![imagen](/images/docker-12.png)