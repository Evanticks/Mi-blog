---
title: Interconexiones entre Bases de Datos
categories: Bases de Datos
---
![remoto](/images/oracle-19c-logo.png)


## Interconexión entre dos bases de datos Oracle.

Antes que nada debemos saber que para conectarnos a una base de datos, debemos tener activados los listener y seguidamente tener en el tsnames.ora la base de datos a la que queremos conectarnos, de esta manera:
sudo nano /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora

```
ORCLCDB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.122.20)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ORCLCDB)
    )
  )

LISTENER_ORCLCDB =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.122.20)(PORT = 1521))

ORACLESERV =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.122.168)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ORCLCDB)
    )
  )
```
Hemos creado una nueva conexión llamada ORACLESERV en el cual procederemos a ingresar la ip del servidor del que queremos recibir los datos, tras esto debemos crear en la otra máquina la tabla de ejemplo que queremos extraer:

```
CREATE TABLE armas (
codarma varchar2 (3),
nombre varchar2 (20),
fuerza number (2),
destreza number (2),
inteligencia number (2),
rareza varchar2 (10),
nivel number (2),
CONSTRAINT pk_armas PRIMARY KEY (codarma)
);

insert into armas values ('001','Espada Corta',8,10,0,'D',5);
insert into armas values ('002','Espada Larga',10,10,0,'C',8);
insert into armas values ('003','Espada Artorias',24,18,20,'S',30);
insert into armas values ('004','Hacha de Mano',8,8,0,'D',6);
insert into armas values ('005','Hacha de Gárgola',14,14,0,'A',15);
insert into armas values ('006','Hacha de Demonio',46,0,0,'S',40);
```

Una vez hecho esto, **nos vamos a la máquina en la que vamos a trabajar con la consulta**, debemos crear un enlace a la base de datos, el cual hemos predefinido como ORACLESERV:

`create database link ORACLESERVIDOR connect to antonio identified by antonio using 'ORACLESERV';`

Tras esto, viene una parte un poco compleja, ya que la tabla que vamos a consultar en el otro servidor viene relacionada, y a la hora de crear relaciones entre tablas no se puede especificar una base de datos remota en el DDL, entonces, cómo podemos hacer que esto funcione? bueno pues mi idea ha sido crear una vista materializada:

`create materialized view mv_armas as select codarma from armas@ORACLESERVIDOR;`

Una vez hecho esto, procedemos a crear las tablas personaje y equipar, siendo una relación N,M.

```
CREATE TABLE personaje (
codpersonaje varchar2 (3),
nombre varchar2 (15),
altura number (3,2),
peso number (3),
raza varchar2 (10),
CONSTRAINT pk_codpersonaje PRIMARY KEY (codpersonaje),
CONSTRAINT ck_codpersonaje CHECK (REGEXP_LIKE(codpersonaje,'^1.*$'))
);

insert into personaje values ('101','Solaire',1.70,80,'humano');
insert into personaje values ('102','Artorias',1.90,90,'hueco');
insert into personaje values ('103','Gargola',3.10,680,'Gárgola');

CREATE TABLE equipar (
codpersonaje varchar2 (3),
codarma varchar2 (3),
fecha date,
CONSTRAINT pk_equipar PRIMARY KEY (codpersonaje,codarma,fecha),
CONSTRAINT fk_codpersonaje FOREIGN KEY (codpersonaje) REFERENCES personaje (codpersonaje),
CONSTRAINT fk_codarma FOREIGN KEY (codarma) REFERENCES mv_armas(codarma);
);

insert into equipar values ('102','003',to_date('2011/02/11','YYYY/MM/DD'));
insert into equipar values ('103','005',to_date('2011/05/04','YYYY/MM/DD'));
insert into equipar values ('101','002',to_date('2011/06/03','YYYY/MM/DD'));
insert into equipar values ('103','002',to_date('2011/09/02','YYYY/MM/DD'));
insert into equipar values ('101','006',to_date('2011/08/03','YYYY/MM/DD'));
insert into equipar values ('102','004',to_date('2011/07/01','YYYY/MM/DD'));

```

¡¡Mucho ojo!! la restricción que he establecido en equipar, que es la foreign key que relaciona el código de armas con nuestra base de datos, llamará a la view que hemos creado y contendrá los datos de la consulta al servidor externo.





![remoto](/images/sql-remoto.png)


![remoto](/images/sql-remoto2.png)


## Interconexión entre dos bases de datos Postgres.

Primero vamos a modificar el fichero /etc/postgresql/13/main/postgresql.conf para abrir la escucha a la ip que quiera conectarse:
`listen_addresses = '*'`

Modificamos el fichero /etc/postgresql/13/main/pg_hba.conf, y aladimos la siguiente línea:
`host    all             all             192.168.122.0/24        md5`

Procedemos a reiniciar Postgres para efectuar los cambios:
`sudo systemctl restart postgresql`

Luego vamos a crear la base de datos souls, luego vamos a darle permiso al usuario antonio2 para poder manejar la base de datos:

```
postgres=# create database souls
postgres-# ;
CREATE DATABASE
GRANT ALL PRIVILEGES ON DATABASE souls TO antonio2;
postgres=# grant connect on database souls to antonio2;
GRANT
postgres=# grant usage on schema public to antonio2;
GRANT
```

En el otro servidor establecemos la configuración de antonio1:

```
postgres=# create user antonio1 with password 'antonio1';
CREATE ROLE
postgres=# create database souls;
CREATE DATABASE
GRANT ALL PRIVILEGES ON DATABASE souls TO antonio1;
postgres=# grant connect on database souls to antonio1;
GRANT
postgres=# grant usage on schema public to antonio1;
GRANT
postgres=# \c souls;
```

Ahora vamos a instalar el paquete que nos permitirá realizar el dblink:
`sudo apt install postgresql-contrib`



De modo que si hacemos una consulta con dblink especificando el host, usuario y base de datos del que se habla, podremos sacar la información de las bases de datos respectivamente.


`select * from dblink('dbname=souls host=192.168.122.168 user=antonio2 password=antonio2', 'select nombre from armas') as armas (Nombre VARCHAR);`

![Descripción de la imagen](/images/postgres-postgres.png)



## Interconexión entre bases de datos Oracle y Postgres.

En nuestro caso la paquetería que necesitamos para conectar Oracle a Postgres es la de Debian Bullseye, por tanto el comando sería el siguiente:
`sudo apt install odbc-postgresql unixodbc -y`

Ahora vamos a entrar en /etc/odbc.ini y vamos a ingresar los siguientes parámetros adaptándolos a nuestro usuario, host y base de datos:

```
[PSQLA]
Debug = 0
CommLog = 0
ReadOnly = 1
Driver = PostgreSQL ANSI
Servername = 192.168.122.168
Username = antonio2
Password = antonio2
Port = 5432
Database = souls
Trace = 0
TraceFile = /tmp/sql.log

[PSQLU]
Debug = 0
CommLog = 0
ReadOnly = 0
Driver = PostgreSQL Unicode
Servername = 192.168.122.168
Username = antonio2
Password = antonio2
Port = 5432
Database = souls
Trace = 0
TraceFile = /tmp/sql.log

[Default]
Driver = /usr/lib/x86_64-linux-gnu/odbc/liboplodbcS.so
```
Ahora vamos a comprobar el fichero /etc/ocdbinst.ini y debe venir configurado como se muestra abajo.

```
[PostgreSQL ANSI]
Description=PostgreSQL ODBC driver (ANSI version)
Driver=psqlodbca.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1

[PostgreSQL Unicode]
Description=PostgreSQL ODBC driver (Unicode version)
Driver=psqlodbcw.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1
```


Podemos comprobar la configuración ingresando los siguientes comandos:

```
odbcinst -q -d
    [PostgreSQL ANSI]
    [PostgreSQL Unicode]


odbcinst -q -s
    [PSQLA]
    [PSQLU]
    [Default]
```

ejecutamos `isql -v PSQLU` y si todo ha ido bien nos devolverá esto:
```
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
```

A continuación vamos a `/opt/oracle/product/19c/dbhome_1/hs/admin/initPSQLU.ora` e ingresamos los siguientes datos:

```
HS_FDS_CONNECT_INFO = PSQLU
HS_FDS_TRACE_LEVEL = Debug
HS_FDS_SHAREABLE_NAME = /usr/lib/x86_64-linux-gnu/odbc/psqlodbcw.so
HS_LANGUAGE = AMERICAN_AMERICA.WE8ISO8859P1
set ODBCINI=/etc/odbc.ini
```

Vamos al listener.ora e ingresamos los siguentes parámetros:


```
SID_LIST_LISTENER =
 (SID_LIST =
  (SID_DESC =
   (GLOBAL_DBNAME = ORCLCDB)
   (ORACLE_HOME = /opt/oracle/product/19c/dbhome_1)
   (SID_NAME = ORCLCDB)
  )
  (SID_DESC =
    (SID_NAME = PSQLU)
    (PROGRAM = dg4odbc)
    (ORACLE_HOME = /opt/oracle/product/19c/dbhome_1)
  )
 )
```

Ahora realizaremos un lsnrctl stop y lisnrctl start, y nos debe salir un mensaje como el siguiente:

```
Service "PSQLU" has 1 instance(s).
  Instance "PSQLU", status UNKNOWN, has 1 handler(s) for this service...
Service "orcl" has 1 instance(s).
  Instance "orcl", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully

```
Ahora nos vamos al tnsnames.ora y añadimos lo siguiente:
```
PSQLU =
 (DESCRIPTION =
 (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
   (CONNECT_DATA = (SID = PSQLU))
   (HS = OK)
  )
```

En localhost estamos especificando que la ip de la máquina sería la misma en la que está alojado el servicio postgre, esto es debido a que se redirecciona a través de la configuración en odbc.ini entonces ya tan solo quedaría conceder permisos de conexión publica de la base de datos a antonio:

```
GRANT CREATE PUBLIC DATABASE LINK to antonio;

Concesion terminada correctamente.
```

Ahora solo nos queda realizar la conexión:
```
CREATE PUBLIC DATABASE LINK CONEXIONPOSTGRES2
CONNECT TO "antonio2"
IDENTIFIED BY "antonio2"
USING 'PSQLU';

Enlace con la base de datos creado.
```

Comprobamos que se pueda consultar el nombre de las armas en **CONEXIONPOSTGRES2**

![Descripción de la imagen](/images/oracle-postgres.png)



## Interconexión entre bases de datos Postgres y Oracle.


Primero vamos a instalar unos paquetes que nos servirán tanto para establecer la conexión con Oracle como a la hora de compilar el Makefile que necesitaremos más adelante:

`apt install git build-essential libaio1 postgresql-server-dev-all -y`

Para realizar la interconexión entre Postgres y Oracle necesitaremos software de terceros, en mi caso vamos a descargar los paquetes que se encuentran en el siguiente enlace:

https://drive.google.com/file/d/1UmBNjVLffaj-hXDXdi6hPGpz6CIZr8eN/view?usp=share_link

En él se encuentran los paquetes en formato.deb, del cual me ocupé de transformar con alien.

Ahora procedemos a instalar los paquetes que adaptarán el formato sqlplus:
```
sudo dpkg -i oracle-instantclient19.5-devel_19.5.0.0.0-2_amd64.deb
sudo dpkg -i oracle-instantclient19.5-basic_19.5.0.0.0-2_amd64.deb
sudo dpkg -i oracle-instantclient19.5-tools_19.5.0.0.0-2_amd64.deb
sudo dpkg -i oracle-instantclient19.5-sqlplus_19.5.0.0.0-2_amd64.deb
```

Ahora vamos a clonar el repositorio con el que trabajaremos para establecer la conexión de Postgres a Oracle:

git clone https://github.com/laurenz/oracle_fdw.git

Ahora en nuestra máquina postgres necesitaremos crear las variables de entorno de Oracle, para ello ingresaremos lo siguiente al final de nuestro .bashrc

```
sudo nano ~/.bashrc

export ORACLE_HOME="/usr/lib/oracle/19.5/client64"
export LD_LIBRARY_PATH="/usr/lib/oracle/19.5/client64/lib"
export PATH=$ORACLE_HOME:$PATH
export USE_PGXS=1
```

Ahora vamos a generar el makefile ejecutando un `make` dentro del directorio que hemos clonado.

UNa vez hecho esto, ante de proceder a instalar el binario, debamo incluir las siguientes líneas a nuestro makefile.
```
PG_CPPFLAGS = -I"$(ORACLE_HOME)/sdk/include" -I"$(ORACLE_HOME)/oci/include" -I"$(ORACLE_HOME)/rdbms/public" -I"$(ORACLE_HOME)/" $(FIN>

SHLIB_LINK = -L"$(ORACLE_HOME)/" -L"$(ORACLE_HOME)/bin" -L"$(ORACLE_HOME)/lib" -L"$(ORACLE_HOME)/lib/amd64" $(FIND_LDFLAGS) -l$(ORACL>
```

Ahora podemos ejecutar un `make install` sin errores ya que las dependencias necesarias para la instalación fueron descargadas con anterioridad.

Luego entramos en nuestra base de datos con el usuario postgres para crear nuestro enlace a Oracle, debemos especificar la ip que tendrá el sevidor y el nombre de la base de datos que en nuestro caso será el por defecto 'ORCLCDB'.

```
CREATE SERVER oracleantonio FOREIGN DATA WRAPPER oracle_fdw OPTIONS(dbserver '//192.168.122.168:1521/ORCLCDB');
```

Una vez hecho esto vamos a enlazar la conexión de nuestro oracleantonio con el usuario que tenga acceso a los registros de la tabla armas, de forma que quedaría de la siguiente manera:

```
create user mapping for postgres server oracleantonio options(user 'antonio',password 'antonio');
DROP user mapping for postgres server oracleantonio; (En caso de que haya algún tipo de error).
```

Crearemos el esqueleto de la tabla que necesitamos en la base de datos pero sin restricciones ni unique, ya que colisionan con la ejecución de la conexión remota, solo necesitaremos los campos que vayan a ser rellenados:


```
CREATE FOREIGN TABLE personaje (
codpersonaje varchar (3),
nombre varchar (15),
altura numeric (3,2),
peso numeric (3),
raza varchar (10) DEFAULT ('Humano'))
SERVER oracleantonio OPTIONS(schema 'ANTONIO', table 'PERSONAJE'
);
```

Ahora solo nos queda realizar la consulta y podemos comprobar como esta se resuelve con éxito.


![Descripción de la imagen](/images/postgres-oracle.png)
