---
title: Auditoría
categoria: Bases de datos
---

![image](/images/audit-logo.png)

## 1. Activa desde SQL*Plus la auditoría de los intentos de acceso exitosos al sistema. Comprueba su funcionamiento.

Vemos si tenemos activado el registro de auditoría:


```
SELECT name, value FROM v$parameter WHERE name = 'audit_trail';
```

También podemos emplear la siguiente sentencia:

```
SHOW PARAMETER AUDIT
```

![image](/images/audit-1.png)

Emplearemos la siguiente sentencia para activar la auditoría:

```
AUDIT CREATE SESSION WHENEVER SUCCESSFUL;
```

Ahora nos desconectamos del susuario administrador y entraremos con el de antonio:

![image](/images/audit-2.png)

```
SELECT OS_USERNAME, USERNAME, EXTENDED_TIMESTAMP, ACTION_NAME FROM DBA_AUDIT_SESSION WHERE USERNAME = 'ANTONIO';
```

![image](/images/audit-3.png)


Como podemos ver el usuario ha quedado registrado en la tabla DBA_AUDIT_SESSION.



## 2. Realiza un procedimiento en PL/SQL que te muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible. Contempla todos los motivos posibles para que un acceso sea fallido.


```
CREATE OR REPLACE PROCEDURE AUDIT_ACCESOS
IS
cursor c_conexion is SELECT OS_USERNAME,TIMESTAMP, USERNAME, RETURNCODE FROM DBA_AUDIT_SESSION;
BEGIN
for v_conexion in c_conexion loop
    Dbms_Output.Put_Line(v_conexion.OS_USERNAME);
    Dbms_Output.Put_Line(v_conexion.USERNAME);
    Dbms_Output.Put_Line(v_conexion.TIMESTAMP);
    MOSTRAR_DEF_ERROR(v_conexion.RETURNCODE);
end loop;
END;
/

CREATE OR REPLACE PROCEDURE MOSTRAR_DEF_ERROR (p_codigo NUMBER)
IS
BEGIN
IF p_codigo = 1017 THEN
    dbms_Output.Put_Line('Error ' || p_codigo || ' El usuario o la contraseña no existe');
ELSIF p_codigo = 2391 THEN
    dbms_Output.Put_Line('Error ' || p_codigo || ' Excedidas las conexiones simultáneas permitidas');
ELSIF p_codigo = 1045 THEN
    dbms_Output.Put_Line('Error ' || p_codigo || ' El usuario no tiene permisos de creación de sesión');
ELSIF p_codigo = 28001 THEN
    dbms_Output.Put_Line('Error ' || p_codigo || ' La contraseña del usuario ha caducado');
ELSIF p_codigo = 28009 THEN
    dbms_Output.Put_Line('Error ' || p_codigo || ' La conexión debe ser a través de SYS o SYSOPER');
ELSIF p_codigo = 1034 THEN
    dbms_Output.Put_Line('Error ' || p_codigo || ' La base de datos no está levantada');
END IF;
END;
/
```

![image](/images/audit-4.png)

## 3. Activa la auditoría de las operaciones DML realizadas por SCOTT. Comprueba su funcionamiento.

```
AUDIT INSERT TABLE, UPDATE TABLE, DELETE TABLE BY SCOTT BY ACCESS;
```



Ahora vamos a rellenar con diferentes cambios para registrarlo en la auditoría:


![image](/images/audit-5.png)

Y con la siguiente sentencia podemos ver los cambios que se han realizado desde el usuario sys:

```
SELECT OS_USERNAME, USERNAME, TIMESTAMP, ACTION_NAME FROM DBA_AUDIT_OBJECT WHERE USERNAME = 'SCOTT';
```

![image](/images/audit-6.png)

## 4. Realiza una auditoría de grano fino para almacenar información sobre la inserción de empleados con sueldo superior a 2000 en la tabla emp de scott.

¿Qué es una auditoría de grano fino?

Una auditoría de grano fino es una auditoría más detallada que no solo contempla el hecho de inserción de filas y columnas sino que también pueden auditarse qué datos han cambiado para así tener un major control sobre la base de datos.

Así que con el objeto DBMS_FGA.ADD_POLICY podemos crear una auditoría de grano fino para la tabla emp de scott:

```
BEGIN
    DBMS_FGA.ADD_POLICY (
        object_schema      =>  'SCOTT',
        object_name        =>  'EMP',
        policy_name        =>  'AUDIT_EMP_SAL',
        audit_condition    =>  'SAL > 2000',
        statement_types    =>  'INSERT'
    );
END;
/
```

![image](/images/audit-7.png)

Ahora vamos a insertar un empleado con un salario superior a 2000 con el usuario SCOTT:

![image](/images/audit-8.png)

Y entramos en como administrador de nuevo y comprobamos que se ha registrado el insert que contiene el salario mayor a 2000:

```
SELECT DB_USER, OBJECT_NAME, SQL_TEXT, TIMESTAMP FROM DBA_FGA_AUDIT_TRAIL WHERE POLICY_NAME='AUDIT_EMP_SAL';
```

![image](/images/audit-9.png)


Alternativamente podríamos hacer nosotros nuestra auditoría con un trigger, como veremos en los siguientes casos, pero viendo que la utilidad de DBMS_FGA es más sencilla y rápida, es la que hemos empleado en este caso.

## 5. Explica la diferencia entre auditar una operación by access o by session ilustrándolo con ejemplos.

La auditoría by access registra cada acceso a los objetos de la base de datos, mientras que la auditoría by session registra la actividad del usuario solo una vez, lo que evita recursividad. La elección de qué tipo de auditoría utilizar dependerá de los objetivos de la auditoría y del nivel de detalle necesario para cumplirlos, siendo así más detallado el hecho de la auditoría by access.

Como ejemplos podemos tomar de ejercicios anteriores la auditoría de accesos exitosos:

![image](/images/audit-3.png)

Y la auditoría by session:

![image](/images/audit-6.png)




## 6. Documenta las diferencias entre los valores db y db, extended del parámetro audit_trail de ORACLE. Demuéstralas poniendo un ejemplo de la información sobre una operación concreta recopilada con cada uno de ellos.

Los valores que alberga el parámetro deb y db extended son los mismos, exceptos en varias columnas, que es SQLBIND y SQLTEXT:

Si añadimor SQLBIND a nuestra consulta podemos ver que no logra encontrar el campo SQLBIND:

```
SELECT DB_USER, OBJECT_NAME, SQL_TEXT, TIMESTAMP FROM DBA_FGA_AUDIT_TRAIL WHERE POLICY_NAME='AUDIT_EMP_SAL';
```

![image](/images/audit-10.png)

Con esto podemos ver el parámetro audit_trail como vimos en el ejercicio 1:

```
SHOW PARAMETER AUDIT
```

![image](/images/audit-1.png)


Entonces, si queremos cambiar el valor de audit_trail a db extended, debemos hacer lo siguiente:

```
ALTER SYSTEM SET audit_trail = DB,EXTENDED SCOPE=SPFILE;
```

Y seguidamente reiniciar la base de datos:

```
shutdown
startup
```

Ahora podemos comprobar que se encuentra de manera extendida:

![image](/images/audit-11.png)


Y al entrar en la base de datos con el usuario SCOTT y hacer un insert en la tabla emp, podemos ver que ahora si se encuentra el campo SQLBIND y SQLTEXT:

```
SELECT USERNAME,ACTION_NAME,TIMESTAMP, OBJ_NAME, SQL_TEXT, SQL_BIND from DBA_AUDIT_OBJECT where USERNAME='SCOTT';
```

![image](/images/audit-12.png)

## 7. Averigua si en Postgres se pueden realizar los cuatro primeros apartados. Si es así, documenta el proceso adecuadamente.

En postgres podemos ver gracias a la ruta `/var/log/postgresql/postgresql-13-main.log` los logs de intentos fallidos al sistema:

![image](/images/audit-13.png)

A parte de esto, Postgres no contempla las auditorías, para ello debemos descargar una herramienta de terceros llamada audit trigger 91 plus que nos permitirá realizar auditorías en la base de datos.

Seguidamente importaremos el esquema de base de datos a nuestro sistema gestor de base de datos:

```
\i audit.sql
```

Como podemos contemplar en la imagen se ha creado diferentes funciones, tablas y disparadores que nos permitirán realizar auditorías en la base de datos:

![image](/images/audit-14.png)

Vamos a activar la auditoría de la tabla personaje:

```
SELECT audit.audit_table('personaje');
```

![image](/images/audit-16.png)


AHora vamos a insertar un nuevo registro en la tabla personaje:

![image](/images/audit-15.png)


Seguidamente vamos a comprobar que funciona la auditoría:

```
select session_user_name, action, table_name, action_tstamp_clk, client_query 
from audit.logged_actions;
```
![image](/images/audit-17.png)

Nota: para poder realizar la auditoría de manera correcta, la importación del esquema debe hacerse dentro de la base de datos con la que se va a trabajar, de manera que si queremos que se audite la tabla personaje de la base de datos souls, deberemos importar el esquema dentro de esta base de datos.

Ahora vamos a resolver el ejercicio 4 que consiste en crear una política de auditoría que registre los accesos a la tabla EMP de SCOTT que tengan un salario superior a 2000, por defecto no tiene una funcionalidad bd como pudiese tenerlo Oracle, pero sí que podemos crear un trigger que nos permita realizar esta auditoría.


Primero crearemos la tabla auditoria_emp:

```
CREATE TABLE auditoria_emp (
  id SERIAL PRIMARY KEY,
  EMPNO INT NOT NULL,
  ACCION VARCHAR(10) NOT NULL,
  SALARIO DECIMAL(7, 2) NOT NULL,
  FECHA_HORA TIMESTAMP NOT NULL DEFAULT NOW()
);
```

Vamos a explicar esta tabla:
- id: es un campo autoincremental que nos permitirá identificar cada registro de la tabla.
- EMPNO: es el número de empleado.
- ACCION: es el salario del empleado.
- SALARIO: es el salario del empleado.
- FECHA_HORA: es la fecha y hora en la que se ha realizado la acción.


```
CREATE OR REPLACE FUNCTION insert_auditoria_emp()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO auditoria_emp (EMPNO, SALARIO, ACCION)
  VALUES (NEW.EMPNO, NEW.SAL, 'INSERT');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tr_insert_emp
AFTER INSERT ON EMP
FOR EACH ROW
WHEN (NEW.SAL > 2000)
EXECUTE FUNCTION insert_auditoria_emp();
```

```
CREATE OR REPLACE FUNCTION update_auditoria_emp()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO auditoria_emp (EMPNO, SALARIO, ACCION)
  VALUES (NEW.EMPNO, NEW.SAL, 'UPDATE');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tr_update_emp
AFTER UPDATE ON EMP
FOR EACH ROW
WHEN (NEW.SAL > 2000)
EXECUTE FUNCTION update_auditoria_emp();
```

Ahora vamos a insertar un registro y seguidamente actualizar otro para comprobar que funciona correctamente:

```
INSERT INTO EMP VALUES(7909, 'JAMES', 'CLERK', 7698, '1981-12-03', 2950, NULL, 30);
UPDATE EMP SET SAL = 4000 WHERE EMPNO = 7369;
```

![image](/images/audit-22.png)

## 8. Averigua si en MySQL se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.


Primero para activar la auditoría en Maríadb, que es la alternativa más a MySQL, debemos modificar el archivo de configuración que se encuentra en `/etc/mysql/mariadb.conf.d/50-server.cnf` y descomentar las líneas que vemos en la siguiente imagen:

![image](/images/audit-18.png)

Seguidamente necesitamos que el usuario mysql tenga permisos para poder escribir en el fichero que hemos descomentado en la línea de configuración `log_error`:


`chown -R mysql: /var/log/mysql`

seguidamente reiniciamos el servicio:

`systemctl restart mysql`

Si intentamos acceder a través de mysql -u root -p y fallamos la contraseña se mantendrá registrado en el log:

![image](/images/audit-19.png)

Ahora nos conectaremos con el usuario SCOTT, crearemos la tabla personaje del anterior ejercicio de postgres y haremos varios inserts:

```
CREATE TABLE personaje (
codpersonaje varchar (3),
nombre varchar (15),
altura decimal (3,2),
peso decimal (3),
raza varchar (10) DEFAULT ('Humano'),
CONSTRAINT pk_codpersonaje PRIMARY KEY (codpersonaje),
CONSTRAINT ck_codpersonaje CHECK (codpersonaje REGEXP '^1.*'),
constraint ck_nombre CHECK (nombre REGEXP '^[A-Z][a-z]*')
);



insert into personaje values ('101','Solaire',1.70,80,'humano');
insert into personaje values ('102','Artorias',1.90,90,'hueco');
insert into personaje values ('103','Gargola',3.10,680,'Gárgola');
```

Al terminar, nos vamos al fichero de log `cat /var/log/mysql/mysql.log` y podemos ver cada uno de las acciones que hemos realizado:

![image](/images/audit-20.png)

Ahora vamos a realizar el ejercicio 4 de manera que solo audite cada vez que el usuario inserte o actualice el salario de un empleado que sea mayor de 2000:

```
CREATE TABLE auditoria_emp (
  id INT NOT NULL AUTO_INCREMENT,
  EMPNO INT NOT NULL,
  SALARIO DECIMAL(7, 2) NOT NULL,
  ACCION VARCHAR(10) NOT NULL,
  FECHA_HORA TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);
```

Vamos a explicar esta tabla:

- La tabla contiene un id que es autoincremental, lo cual nos permitirá identificar cada uno de los registros de la tabla aunque se repitan datos, la cual haremos primary key.
- el código del empleado el cual nos servirá para identificar el empleado que ha sufrido el cambio.
- el salario que será mayor de 2000 para que se pueda auditar.
- La fecha y hora del sistema en ese momento.


Ahora vamos a crear el disparador bastante sencillo que nos permitirá auditar cada vez que el salario de un empleado sea mayor de 2000:

```
DELIMITER //

CREATE TRIGGER tr_insert_emp
AFTER INSERT ON EMP
FOR EACH ROW
BEGIN
  IF NEW.SAL > 2000 THEN
    INSERT INTO auditoria_emp (EMPNO, SALARIO, ACCION) VALUES (NEW.EMPNO, NEW.SAL, 'INSERT');
  END IF;
END //

DELIMITER ;
```

```
DELIMITER //

CREATE TRIGGER tr_update_emp
AFTER UPDATE ON EMP
FOR EACH ROW
BEGIN
  IF NEW.SAL > 2000 THEN
    INSERT INTO auditoria_emp (EMPNO, SALARIO, ACCION) VALUES (NEW.EMPNO, NEW.SAL, 'UPDATE');
  END IF;
END //

DELIMITER ;
```

Insertamos y actualizamos registros:

```
INSERT INTO EMP VALUES(7911, 'JAMES', 'CLERK', 7698,'1981-12-03', 2950, NULL, 30);
UPDATE EMP SET SAL = 3000 WHERE EMPNO = 7369;
```



Y aquí el resultado junto con la acción de insertar o actualizar un salario a más de 2000:

![image](/images/audit-21.png)

## 9.  Averigua las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento. Demuestra su funcionamiento.

Las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento son las siguientes:



Podemos activar las auditorías en un fichero JSON desde el fichero de configuración:
`nano /etc/mongod.conf`


```
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/auditLog.json
```

También podremos activar las auditorías en el syslog desde el fichero de configuración:
nano /etc/mongod.conf

```
auditLog:
  destination: syslog
```

También podemos habilitar las auditorías en un fichero BSON desde el fichero de configuración:
nano /etc/mongod.conf

```
auditLog:
  destination: file
  format: BSON
  path: /var/log/mongodb/auditLog.bson
```

Habilitar las auditorías en la consola desde el fichero de configuración:
nano /etc/mongod.conf

```
auditLog:
  destination: console
```



Vamos a empezar a crear el usuario en la base de datos, la cual más tarde importará unos 140 documentos que poseo.

```
db.createUser(
   {
     user: "admin", 
     pwd: "admin", 
     roles: [ { role: "dbOwner", db: "souls" } ]
   }
 )
```


En la documentación oficial de mongo especifica que recomienda la auditoría en formato BSON, ya que los JSON degradan la eficiencia de la auditoría.

![image](/images/audit-25.png)

Por ende elijo mostrar la auditoría en formato BSON:

```
nano /etc/mongod.conf

auditLog:
  destination: file
  format: BSON
  path: /var/log/mongodb/auditLog.bson
```

ATENCIÓN: mongodb trabaja con ficheros bson, el cual es un lenguaje interpretado por la máquina específicamente creado por mongodb, por lo que no podremos visualizarlo con el comando `cat` o `less` como normalmente se hace con con los ficheros de texto plano, para ello utilizaremos el comando `bsondump` que nos permite visualizar el contenido de un fichero BSON y lo transforma a JSON.

```
bsondump /var/log/mongodb/auditLog.bson | jq
```

Entonces... por qué no he elegido el formato JSON si al final lo transformo a JSON? Pues porque al elegir el JSON directamente haces que lmongo sea la que traduzca directamente el BSON a JSON, lo cual hace que la base de datos tenga que hacer un trabajo extra que no es necesario, por lo que es mejor que la base de datos realice la auditoría con BSON y nosotros con la computación de la máquina del cliente lo transformemos a JSON cuando queramos visualizarlo.

En nuestro caso con 140 documentos es una diferencia mínima, pero si tuviéramos millones de documentos, la diferencia sería notable.

Ahora vamos a proceder a importar los documentos a la base de datos souls:


```
mongoimport --db souls --type=json --file DarkSoulsWeapons.json --jsonArray
```

![image](/images/audit-24.png)

Ahora vamos a comprobar que se ha registrado la importación en el fichero de auditoría:

![image](/images/audit-26.png)


Como podemos ver se ha realizado la acción de mongoimport y ha actuado sobre la base de datos y colección souls/soulsweapons.

Han pasado varios días, me levanto una mañana y descubro que alguien ha borrado todos los documentos de la colección soulsweapons, por lo que voy a proceder a realizar una auditoría de la base de datos para descubrir quien ha sido el culpable:

Resulta que un usuario con un nick un tanto sospechoso ha entrado en la base de datos souls, lo podemos ver a través del ActionType y user,db:

![image](/images/audit-27.png)

Pero esto no dice nada sobre lo que el usuario ha podido hacer, debemos indagar un poco más de su actividad una vez logueado en la base de datos, por lo que podemos filtrar ahora que conocemos los valores que hay que observar:

```
bsondump /var/log/mongodb/auditLog.bson | jq | egrep 'user|atype|ns''
```

OJO!! Este usuario ha hecho un drop de la colección soulsweapons, ahora sí que podemos decir que esta persona ha sido la culpable.

![image](/images/audit-29.png)



## 10. Averigua si en MongoDB se pueden auditar los accesos a una colección concreta. Demuestra su funcionamiento.

Como buenos administradores de sistemas tenemos la copia de la información guardada, entonces vamos a comenzar auditando la colección soulsweapons:

En el ejercicio anterior entre los valores que hemos filtrado se encuentra el campo ns, este es el campo del namespace, el cual es "nombrebasededatos.coleccion" así que vamos a establecer un filtro en el archivo de configuración de mongo para que comience a auditar la colección soulsweapons:


Entramos como administrador en la base de datos admin y ejecutamos el siguiente comando:

```
use souls
db.setProfilingLevel(2)
```

A partir de ahora todas las acciones que se realicen sobre la colección, podrán ser auditadas de manera que sabremos incluso qué se está insertando:


```
db.system.profile.find({
   "ns": "souls.DarkSoulsWeapons",
   "op": {
      "$in": [
         "insert",
         "update",
         "delete"
      ]
   }
})
```

![image](/images/audit-31.png)


Como podemos ver, el usuario admin ha insertado un documento en la colección soulsweapons.

