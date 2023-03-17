---
title: Procedimientos en Oracle y Postgres
categories: Bases de Datos
---

![remoto](/images/oraclepostgres.jpg)

## ORACLE

Comenzaremos adjuntando el script de creación de tablas e inserciones de Oracle:

```
CREATE TABLE DEPT
(
 DEPTNO NUMBER(2),
 DNAME VARCHAR2(14),
 LOC VARCHAR2(13),
 CONSTRAINT PK_DEPT PRIMARY KEY (DEPTNO)
);
CREATE TABLE EMP
(
 EMPNO NUMBER(4),
 ENAME VARCHAR2(10),
 JOB VARCHAR2(9),
 MGR NUMBER(4),
 HIREDATE DATE,
 SAL NUMBER(7, 2),
 COMM NUMBER(7, 2),
 DEPTNO NUMBER(2),
 CONSTRAINT FK_DEPTNO FOREIGN KEY (DEPTNO) REFERENCES DEPT (DEPTNO),
 CONSTRAINT PK_EMP PRIMARY KEY (EMPNO)
);
INSERT INTO DEPT VALUES (10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO DEPT VALUES (20, 'RESEARCH', 'DALLAS');
INSERT INTO DEPT VALUES (30, 'SALES', 'CHICAGO');
INSERT INTO DEPT VALUES (40, 'OPERATIONS', 'BOSTON');
INSERT INTO EMP VALUES(7369, 'SMITH', 'CLERK', 7902,TO_DATE('17-DIC-1980', 'DD-MON-YYYY'), 800, NULL, 20);
INSERT INTO EMP VALUES(7499, 'ALLEN', 'SALESMAN', 7698,TO_DATE('20-FEB-1981', 'DD-MON-YYYY'), 1600, 300, 30);
INSERT INTO EMP VALUES(7521, 'WARD', 'SALESMAN', 7698,TO_DATE('22-FEB-1981', 'DD-MON-YYYY'), 1250, 500, 30);
INSERT INTO EMP VALUES(7566, 'JONES', 'MANAGER', 7839,TO_DATE('2-ABR-1981', 'DD-MON-YYYY'), 2975, NULL, 20);
INSERT INTO EMP VALUES(7654, 'MARTIN', 'SALESMAN', 7698,TO_DATE('28-SEP-1981', 'DD-MON-YYYY'), 1250, 1400, 30);
INSERT INTO EMP VALUES(7698, 'BLAKE', 'MANAGER', 7839,TO_DATE('1-MAY-1981', 'DD-MON-YYYY'), 2850, NULL, 30);
INSERT INTO EMP VALUES(7782, 'CLARK', 'MANAGER', 7839,TO_DATE('9-JUN-1981', 'DD-MON-YYYY'), 2450, NULL, 10);
INSERT INTO EMP VALUES(7788, 'SCOTT', 'ANALYST', 7566,TO_DATE('09-DIC-1982', 'DD-MON-YYYY'), 3000, NULL, 20);
INSERT INTO EMP VALUES(7839, 'KING', 'PRESIDENT', NULL,TO_DATE('17-NOV-1981', 'DD-MON-YYYY'), 5000, NULL, 10);
INSERT INTO EMP VALUES(7844, 'TURNER', 'SALESMAN', 7698,TO_DATE('8-SEP-1981', 'DD-MON-YYYY'), 1500, 0, 30);
INSERT INTO EMP VALUES(7876, 'ADAMS', 'CLERK', 7788,TO_DATE('12-ENE-1983', 'DD-MON-YYYY'), 1100, NULL, 20);
INSERT INTO EMP VALUES(7900, 'JAMES', 'CLERK', 7698,TO_DATE('3-DIC-1981', 'DD-MON-YYYY'), 950, NULL, 30);
INSERT INTO EMP VALUES(7902, 'FORD', 'ANALYST', 7566,TO_DATE('3-DIC-1981', 'DD-MON-YYYY'), 3000, NULL, 20);
INSERT INTO EMP VALUES(7934, 'MILLER', 'CLERK', 7782,TO_DATE('23-ENE-1982', 'DD-MON-YYYY'), 1300, NULL, 10);

COMMIT;
```



### 1. Hacer un procedimiento que muestre el nombre y el salario del empleado cuyo código es 7082
```
create or replace procedure mostrar_7082
IS
    v_nombre emp.ename%type;
    v_sal emp.sal%type;
BEGIN
    select ename,sal into v_nombre,v_sal 
    from emp
    where empno= 7782;
    dbms_output.put_line('El nombre del empleado 7082 es ' || v_nombre || ' y su salario es ' || v_sal );
END;
/
```

![remoto](/images/oracle-ejercicio1.png)

### 2. Hacer un procedimiento que reciba como parámetro un código de empleado y devuelva su nombre
```
Create or replace procedure codigoanombreemp (p_codempleado emp.empno%type)
IS
    v_nombre emp.ename%type;
BEGIN
    select ename into v_nombre from emp where empno=p_codempleado;
    dbms_output.put_line ('El empleado con el código ' || p_codempleado || ' es ' || v_nombre);
END;
/
```

![remoto](/images/oracle-ejercicio2.png)

### 3. Hacer un procedimiento que devuelva los nombres de los tres empleados más antiguos
```
create or replace procedure tresempleadosmasantiguos
IS
cursor c_empleados is
select ename from emp WHERE ROWNUM <= 3 order by hiredate asc;
BEGIN
FOR v_empleados in c_empleados loop
    dbms_output.put_line (v_empleados.ename);
end loop;
END;
/
```

![remoto](/images/oracle-ejercicio5.png)

### 4. Hacer un procedimiento que reciba el nombre de un tablespace y muestre los nombres de los usuarios que lo tienen como tablespace por defecto (Vista DBA_USERS)
```
create or replace procedure tablespacedefecto (p_tablespace DBA_USERS.DEFAULT_TABLESPACE%type)
IS
cursor c_tablespace is
SELECT USERNAME from DBA_USERS where DEFAULT_TABLESPACE=p_tablespace;
BEGIN
FOR v_usuarios in c_tablespace loop
    dbms_output.put_line (v_usuarios.USERNAME);
end loop;
END;
/
```

![remoto](/images/oracle-ejercicio3.png)

### 5. Modificar el procedimiento anterior para que haga lo mismo pero devolviendo el número de usuarios que tienen ese tablespace como tablespace por defecto. Nota: Hay que convertir el procedimiento en función
```
create or replace function tablespacedefecto (p_tablespace DBA_USERS.DEFAULT_TABLESPACE%type)
return number
IS
v_num number (4);
BEGIN
SELECT count(USERNAME) into v_num from DBA_USERS where DEFAULT_TABLESPACE=p_tablespace;
return v_num;
END;
/

create or replace procedure mostrarfunciontablespacedefecto (p_tablespace DBA_USERS.DEFAULT_TABLESPACE%type)
is
v_num number;
BEGIN
    v_num:=tablespacedefecto(p_tablespace);
    dbms_output.put_line ('El tablespace ' || p_tablespace || ' lo tienen ' || v_num || ' usuarios.');
end;
/
```

![remoto](/images/oracle-ejercicio4.png)

### 6. Hacer un procedimiento llamado mostrar_usuarios_por_tablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios de cada uno y el número de los mismos, así: (Vistas DBA_TABLESPACES y DBA_USERS)

En  este ejercicio en concreto utilizaremos una función creada en el anterior ejercicio para este propósito.

```
create or replace procedure mostrar_usuarios_por_tablespace
is
cursor c_tablespace is
select DISTINCT DEFAULT_TABLESPACE from DBA_USERS;
BEGIN
for v_tablespace in c_tablespace loop
dbms_output.put_line('Tablespace ' || v_tablespace.DEFAULT_TABLESPACE || ':');
MOSTRARUSUARIOS(v_tablespace.DEFAULT_TABLESPACE);
dbms_output.put_line('Total Usuarios Tablespace '|| v_tablespace.DEFAULT_TABLESPACE || ': ' || tablespacedefecto(v_tablespace.DEFAULT_TABLESPACE));
end loop;
end;
/


CREATE OR REPLACE PROCEDURE MOSTRARUSUARIOS (p_tablespace in out varchar2)
is
cursor c_nombre is
select USERNAME FROM DBA_USERS WHERE DEFAULT_TABLESPACE=p_tablespace;
BEGIN
FOR v_nombre in c_nombre loop
    dbms_output.put_line('Usuario ' || v_nombre.USERNAME );
end loop;
end;
/
```

![remoto](/images/oracle-ejercicio6.png)

### 7. Hacer un procedimiento llamado mostrar_codigo_fuente  que reciba el nombre de otro procedimiento y muestre su código fuente. (DBA_SOURCE)
```
create or replace procedure mostrar_codigo_fuente (p_procedimiento varchar2)
IS
cursor c_fuente is
select text from DBA_SOURCE where NAME=p_procedimiento;
BEGIN
FOR v_fuente in c_fuente loop
    dbms_output.put_line(v_fuente.text);
end loop;
end;
/
```

![remoto](/images/oracle-ejercicio7.png)

### 8. Hacer un procedimiento llamado mostrar_privilegios_usuario que reciba el nombre de un usuario y muestre sus privilegios de sistema y sus privilegios sobre objetos. (DBA_SYS_PRIVS y DBA_TAB_PRIVS)

```
CREATE OR REPLACE PROCEDURE mostrar_privilegios_usuario (p_usuario DBA_USERS.USERNAME%TYPE)
is
cursor c_sistema is
SELECT GRANTEE,PRIVILEGE FROM DBA_SYS_PRIVS WHERE GRANTEE=p_usuario;
cursor c_objeto is
SELECT GRANTEE,PRIVILEGE FROM DBA_TAB_PRIVS WHERE GRANTEE=p_usuario;
BEGIN
FOR v_sistema in c_sistema loop
    dbms_output.put_line('Privilegio de sistema: ' || v_sistema.PRIVILEGE);
end loop;
FOR v_objeto in c_objeto loop
    dbms_output.put_line('Privilegiode objetos: ' || v_objeto.PRIVILEGE);
end loop;
END;
/
```

![remoto](/images/oracle-ejercicio8.png)

### 9. Realiza un procedimiento llamado listar_comisiones que nos muestre por pantalla un listado de las comisiones de los empleados agrupados según la localidad donde está ubicado su departamento con el siguiente formato:
```
CREATE OR REPLACE PROCEDURE Listar_comisiones
IS
cursor c_localidad is
select dname,loc from dept;
v_total number(5);
BEGIN
select sum(comm) into v_total from emp; 
FOR v_localidad in c_localidad loop
    dbms_output.put_line('Localidad: ' || v_localidad.loc);
    dbms_output.put_line('Departamento: ' || v_localidad.dname);
    MOSTRARCOMISIONES(v_localidad.loc);
end loop;
dbms_output.put_line('Total comisiones en la empresa es de: ' || v_total);
exception
WHEN NO_DATA_FOUND then
dbms_output.put_line('Hay tablas vacías');
end;
/

CREATE OR REPLACE PROCEDURE MOSTRARCOMISIONES (p_localidad in out varchar2)
IS
cursor c_comisiones is
select ename,comm from emp where deptno in (select deptno from dept where loc = p_localidad);
v_total number(5);
BEGIN
select sum(comm) into v_total from emp where deptno in  (select deptno from dept where loc = p_localidad);
FOR v_empleado in c_comisiones loop
    dbms_output.put_line('Empleado: ' || v_empleado.ename || '..................' || v_empleado.comm);
    if v_empleado.comm > 10000 then
        raise_application_error(-20001,'Hay algún empleado con más de 10.000 de comisión');
    end if;
end loop;
dbms_output.put_line('Total comisiones departamento: ' || p_localidad || ' es de: ' || v_total);
END;
/
```

Podemos ver en la imagen que se produce un raise ya que hay un usuario con más de 10.000 de comm.

![remoto](/images/oracle-ejercicio9.png)

### 10. Realiza un procedimiento que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna afectan y en qué consisten exactamente. (DBA_TABLES, DBA_CONSTRAINTS, DBA_CONS_COLUMNS)
```
SELECT CONSTRAINT_NAME,CONSTRAINT_TYPE,SEARCH_CONDITION_VC FROM DBA_CONSTRAINTS WHERE TABLE_NAME='EQUIPAR';
SELECT CONSTRAINT_NAME,COLUMN_NAME FROM DBA_CONS_COLUMNS WHERE TABLE_NAME = 'PERSONAJE';

CREATE OR REPLACE PROCEDURE MOSTRARRESTRICCIONES (p_tabla varchar2)
is
v_tabla varchar2(50);
BEGIN
v_tabla:=p_tabla;
MOSTRARCONSSYS (v_tabla);
MOSTRARCONSTABS (v_tabla);
end;
/

CREATE OR REPLACE PROCEDURE MOSTRARCONSSYS (p_tabla in out varchar2)
IS
cursor c_system is
SELECT CONSTRAINT_NAME,CONSTRAINT_TYPE,SEARCH_CONDITION_VC FROM DBA_CONSTRAINTS WHERE TABLE_NAME=p_tabla;
BEGIN
FOR v_system in c_system loop
    dbms_output.put_line(v_system.CONSTRAINT_NAME || 'Tipo de restricción: ' || v_system.CONSTRAINT_TYPE || v_system.SEARCH_CONDITION_VC);
end loop;
END;
/


CREATE OR REPLACE PROCEDURE MOSTRARCONSTABS (p_tabla in out varchar2)
IS
cursor c_tabs is
SELECT CONSTRAINT_NAME,COLUMN_NAME FROM DBA_CONS_COLUMNS WHERE TABLE_NAME = p_tabla;
BEGIN
FOR v_tabs in c_tabs loop
    dbms_output.put_line('Restricción: ' || v_tabs.CONSTRAINT_NAME || ' y la columna a la que hace referencia es: ' || v_tabs.COLUMN_NAME);
end loop;
END;
/
```

![remoto](/images/oracle-ejercicio10.png)

## POSTGRESQL

Comenzaremos adjuntando el script de creación de tablas e inserciones de Postgres:

```
create table dept(
  deptno   decimal(2,0) not null,
  dname    varchar(14),
  loc      varchar(13));
create table emp(
  empno    decimal(4,0) not null,
  ename    varchar(10),
  job      varchar(9),
  mgr      decimal(4,0),
  hiredate date,
  sal      decimal(7,2),
  comm     decimal(7,2),  
  deptno   decimal(2,0) not null);
create table bonus(
  ename    varchar(10),
  job      varchar(9),
  sal      decimal,
  comm     decimal);
create table salgrade(
  grade    decimal,
  losal    decimal,
  hisal    decimal);
create table dummy (
  dummy    decimal);
insert into dummy values (0);
insert into DEPT (DEPTNO, DNAME, LOC)
  select 10, 'ACCOUNTING', 'NEW YORK' from dummy union all
  select 20, 'RESEARCH',   'DALLAS'   from dummy union all
  select 30, 'SALES',      'CHICAGO'  from dummy union all
  select 40, 'OPERATIONS', 'BOSTON'   from dummy;
insert into emp (EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO)
  select 7839, 'KING',   'PRESIDENT', cast(null as integer), to_date('17-11-1981','dd-mm-yyyy'),    5000, cast(null as integer), 10 from dummy union all
  select 7698, 'BLAKE',  'MANAGER',   7839, to_date('1-5-1981','dd-mm-yyyy'),      2850, cast(null as integer), 30 from dummy union all
  select 7782, 'CLARK',  'MANAGER',   7839, to_date('9-6-1981','dd-mm-yyyy'),      2450, cast(null as integer), 10 from dummy union all
  select 7566, 'JONES',  'MANAGER',   7839, to_date('2-4-1981','dd-mm-yyyy'),      2975, cast(null as integer), 20 from dummy union all
  select 7788, 'SCOTT',  'ANALYST',   7566, to_date('13-7-87','dd-mm-rr') - 85,  3000, cast(null as integer), 20 from dummy union all
  select 7902, 'FORD',   'ANALYST',   7566, to_date('3-12-1981','dd-mm-yyyy'),     3000, cast(null as integer), 20 from dummy union all
  select 7369, 'SMITH',  'CLERK',     7902, to_date('17-12-1980','dd-mm-yyyy'),     800, cast(null as integer), 20 from dummy union all
  select 7499, 'ALLEN',  'SALESMAN',  7698, to_date('20-2-1981','dd-mm-yyyy'),     1600,  300, 30 from dummy union all
  select 7521, 'WARD',   'SALESMAN',  7698, to_date('22-2-1981','dd-mm-yyyy'),     1250,  500, 30 from dummy union all
  select 7654, 'MARTIN', 'SALESMAN',  7698, to_date('28-9-1981','dd-mm-yyyy'),     1250, 1400, 30 from dummy union all
  select 7844, 'TURNER', 'SALESMAN',  7698, to_date('8-9-1981','dd-mm-yyyy'),      1500,    0, 30 from dummy union all
  select 7876, 'ADAMS',  'CLERK',     7788, to_date('13-7-87', 'dd-mm-rr') - 51, 1100, cast(null as integer), 20 from dummy union all
  select 7900, 'JAMES',  'CLERK',     7698, to_date('3-12-1981','dd-mm-yyyy'),      950, cast(null as integer), 30 from dummy union all
  select 7934, 'MILLER', 'CLERK',     7782, to_date('23-1-1982','dd-mm-yyyy'),     1300, cast(null as integer), 10 from dummy;
insert into salgrade
  select 1,  700, 1200 from dummy union all
  select 2, 1201, 1400 from dummy union all
  select 3, 1401, 2000 from dummy union all
  select 4, 2001, 3000 from dummy union all
  select 5, 3001, 9999 from dummy;
commit;
```




### 1. Hacer un procedimiento que muestre el nombre y el salario del empleado cuyo código es 7082
```
CREATE or replace PROCEDURE mostrar_7082() 
AS $$
DECLARE
    v_nombre emp.ename%type;
    v_salario emp.sal%type;
BEGIN
    select ename,sal into v_nombre,v_salario from emp where empno=7782;
    RAISE NOTICE 'El nombre del empleado es %, y su salario es %', v_nombre,v_salario;
END;
$$ LANGUAGE plpgsql;
```

![remoto](/images/postgres-ejercicio2.png)

### 2. Hacer un procedimiento que reciba como parámetro un código de empleado y devuelva su nombre
```sql
Create or replace procedure codigoanombreemp (p_codempleado emp.empno%type) 
AS $$
DECLARE
    v_nombre emp.ename%type;
BEGIN
    select ename into v_nombre from emp where empno=p_codempleado;
    RAISE NOTICE 'El empleado con el código %, es %', p_codempleado,v_nombre;
END;
$$ LANGUAGE plpgsql;
```

![remoto](/images/postgres-ejercicio1.png)