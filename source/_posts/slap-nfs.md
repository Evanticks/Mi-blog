---
title: Directorios centralizados con LDAP y NFS
categories: Administración de Sistemas
---


![Descripción de la imagen](/images/LOGO-LDAP.png)


# Directorios centralizados con LDAP y NFS

## Introducción

En este post vamos a ver como crear un directorio centralizado con LDAP y NFS.

## Instalación de LDAP

Para instalar LDAP vamos a usar el paquete slapd, que es el servidor LDAP.

```bash
apt install slapd
```

Una vez instalado vamos a configurarlo, para ello ejecutamos el comando:

```bash
dpkg-reconfigure slap-utils
```

Nos pedirá que introduzcamos la contraseña de del usuario root del servidor slapd, y nos preguntará si queremos usar el dominio de la máquina o no, en este caso vamos a usar el dominio de la máquina.

Con el siguiente comando podremos ver la información del usuario root en el dominio gonzanonazareno.org

```bash
ldapsearch -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -b "dc=antonio,dc=gonzalonazareno,dc=org" -W
```


![Descripción de la imagen](/images/slap1-1.png)

Ahora crearemos una unidad organizativa, en este caso la llamaremos Organizacion.ldif, y le daremos el siguiente contenido:

```bash
dn: ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Personas

dn: ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Grupos
```

"dn" significa Distinguished Name, e indica el nombre distintivo que el objeto tendrá dentro de la jerarquía de directorios, y "objectClass" es la clase de objeto, en este caso es una unidad organizativa.

Procedemos a ejecutar el siguiente comando para inyectar los nuevos objetos en el árbol de slapd:

```bash
ldapadd -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -f UnidadesOrganizativas.ldif -W
```

![Descripción de la imagen](/images/slap1-2.png)


Ahora si volvemos a ejecutar el comando 
```bash
ldapsearch -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -b "dc=antonio,dc=gonzalonazareno,dc=org" -W
```

Podremos ver que se han creado los nuevos objetos llamados Personas y Grupos.

![Descripción de la imagen](/images/slap1-3.png)

Ahora vamos a crear un usuario, para ello crearemos un fichero llamado usuarios.ldif, y le daremos el siguiente contenido:

```bash
dn: uid=prueba,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: top
cn: prueba
gidNumber: 2001
homeDirectory: /home/prueba
loginShell: /bin/bash
sn: prueba
uid: prueba
uidNumber: 2001
userPassword: {SSHA}sfqp8j+/1HHe7N6qcbDIrLkf3T2c4wIW

dn: uid=macarena,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: top
cn: macarena
gidNumber: 2002
homeDirectory: /home/macarena
loginShell: /bin/bash
sn: macarena
uid: macarena
uidNumber: 2002
userPassword: {SSHA}sCwMT7CqCloD4bzvED/1QB9xvUnZ0N5P

```

Para los hash de contraseñas la hemos creado con la utilidad slappasswd, que viene instalada con slapd.

```bash
root@alfa:/home/antonio# slappasswd
New password: 
Re-enter new password: 
{SSHA}sfqp8j+/1HHe7N6qcbDIrLkf3T2c4wIW
```

Para agregar al usuario prueba y al usuario macarena debemos de volver a inyectar el fichero Usuario.ldif en el árbol de slapd, para ello ejecutamos el siguiente comando:

```bash
ldapadd -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -f usuarios.ldif -W
```

![Descripción de la imagen](/images/slap1-4.png)

Lo que hemos hecho ha sido insertar registros en el objeto de personas, ahora vamos a crear un grupo, para ello crearemos un fichero llamado grupos.ldif, y le daremos el siguiente contenido:

```bash
dn: cn=desarrollo,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: desarrollo
member:

dn: cn=sistemas,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: sistemas
member:
```

A continuación vamos a inyectar el fichero grupos.ldif en el árbol de slapd, para ello ejecutamos el siguiente comando:

```bash
ldapadd -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -f grupos.ldif -W
```

![Descripción de la imagen](/images/slap1-5.png)

Luego vamos a añadir a los usuarios a los grupos, para ello crearemos un fichero llamado personas-grupos.ldif, y le daremos el siguiente contenido:

```bash
# Al grupo de desarrollo
dn: cn=desarrollo,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=prueba,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org

# Al grupo de desarrollo y sistemas
dn: cn=desarrollo,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=macarena,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org

dn: cn=sistemas,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
changetype:modify
add: member
member: uid=macarena,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org
```

A continuación vamos a inyectar el fichero personas-grupos.ldif en el árbol de slapd, para ello ejecutamos el siguiente comando:

```bash
ldapmodify -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -f personas-grupos.ldif -W
```

![Descripción de la imagen](/images/slap1-6.png)

Podríamos usar también ldapmodify que es una herramiente enfocada a la modificación, pero para nuestro caso ambas nos sirven.

Usando el comando ldapsearch podemos ver que los usuarios están en los grupos:

```bash
ldapsearch -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -b "cn=desarrollo,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org" -W
```

```bash
root@alfa:/home/antonio# ldapsearch -x -b ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
# extended LDIF
#
# LDAPv3
# base <ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Grupos, antonio.gonzalonazareno.org
dn: ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Grupos

# sistemas, Grupos, antonio.gonzalonazareno.org
dn: cn=sistemas,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: sistemas
member:
member: uid=macarena,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org

# desarrollo, Grupos, antonio.gonzalonazareno.org
dn: cn=desarrollo,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: groupOfNames
cn: desarrollo
member:
member: uid=prueba,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org
member: uid=macarena,ou=Personas,dc=antonio,dc=gonzalonazareno,dc=org

# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 5
```

Si queremos borrar un elemento del árbol podemos realizarlo gracias a la siguiente instrucción:

```bash
ldapdelete -x -D 'cn=admin,dc=antonio,dc=gonzalonazareno,dc=org' -W cn=reabastecimiento,ou=Grupos,dc=antonio,dc=gonzalonazareno,dc=org
```


Desplegar árbol a través de NFS:


apt-get install nfs-kernel-server -y

Nos vamos a /etc/exportfs y añadimos la siguiente línea:
```bash
/home/antonio/nfs *(rw,fsid=0,subtree_check,no_root_squash)
```

Ahora ejecutamos `exportfs -a`

```bash
root@alfa:/home/antonio# mkdir nfs
root@alfa:/home/antonio# mkdir nfs/almacenamiento
```


Comenzamos con la configuración del paquete nscd. Este es un demonio de caché para el servicio de nombres, lo cual evitará repetir consultas de las bases de datos de passwd, group y hosts.


```bash
apt install libnss-ldapd libpam-ldapd nscd -y
```

Luego de configurar el instalador de forma que capture la ip 127.0.0.1 y el nombre dc=antonio,dc=gonzalonazareno,dc=org, debemos editar el fichero /etc/nsswitch.conf, y añadir las siguientes líneas:

![Descripción de la imagen](/images/slap1-7.png)


## Cliente Ubuntu

```bash
apt-get install libnss-ldapd libpam-ldapd nscd nfs-kernel-server -y
```

Introducimos la dirección IP del servidor LDAP y el nombre del dominio:

![Descripción de la imagen](/images/slap1-8.png)

En el fichero /etc/ldap/ldap.conf se establecen los parámetros de configuración del cliente LDAP. En este fichero se establece la dirección del servidor LDAP, el puerto, el dominio, el tipo de autenticación, etc.

```bash
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE    dc=antonio,dc=gonzalonazareno,dc=org
URI     ldap://192.168.0.1

SIZELIMIT       12
TIMELIMIT       15
DEREF           never

# TLS certificates (needed for GnuTLS)
TLS_CACERT      /etc/ssl/certs/ca-certificates.crt
```


Ahora necesitaremos configurar el cliente para que al iniciar sesión cree el home del usuario. Para ello editamos el fichero /etc/pam.d/common-account y añadimos la siguiente línea:

```bash
session required pam_mkhomedir.so skel=/etc/skel/ umask=0022

```

Vamos a probar montando de manera efímera el directorio en el cual se va a crear el home del usuario macarena:

```bash
mount -t nfs 192.168.0.1:/home/antonio/nfs/ /home
```


Ahora cuando el usuario macarena inicie sesión este se verá reflejado en nuestro servidor nfs, probaremos creando un documento txt:


![Descripción de la imagen](/images/slap1-9.png)

Ahora vamos a hacer que este recurso funcione de manera permanente, para ello podríamos editar el fichero fstab, o bien crear una unidad de systemd, que en este caso es lo que vamos a emplear.


nano /etc/systemd/system/home.mount
```bash
[Unit]
Description= Montaje de carpeta home para NFS

[Mount]
What=192.168.0.1:/home/antonio/nfs
Where=/home   
Type=nfs
Options=defaults

[Install]
WantedBy=multi-user.target

```

systemctl daemon-reload
systemctl enable home.mount
systemctl start home.mount
systemctl status home.mount

![Descripción de la imagen](/images/slap1-10.png)

## Cliente Rocky Linux

```bash
dnf install openldap-clients sssd sssd-ldap oddjob-mkhomedir sssd-tools -y
```

```bash
authselect list
```

```bash
[root@bravo antonio]# authselect list
- minimal	 Local users only for minimal installations
- sssd   	 Enable SSSD for system authentication (also for local users only)
- winbind	 Enable winbind for system authentication

```

```bash
authselect select sssd with-mkhomedir --force
```

volvemos a especificar la dirección IP del servidor LDAP y el nombre del dominio en /etc/openldap/ldap.conf:

```bash
BASE dc=antonio,dc=gonzalonazareno,dc=org
URI ldap://172.16.0.1


SIZELIMIT       12
TIMELIMIT       15
DEREF           never
```

sudo nano /etc/sssd/sssd.conf

```bash
[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldap://172.0.16.1
ldap_search_base = dc=antonio,dc=gonzalonazareno,dc=org
ldap_id_use_start_tls = True
ldap_tls_cacertdir = /etc/openldap/cacerts
cache_credentials = True
ldap_tls_reqcert = allow

[sssd]
services = nss, pam, autofs
domains = default

[nss]
homedir_substring = /home/nfs

```


```bash
chmod 0600 /etc/sssd/sssd.conf
systemctl restart sssd
```

authconfig --enablemkhomedir --updateall



```bash
authconfig --enableldap \
--enableldapauth \
--ldapserver=antonio.gonzalonazareno.org \
--ldapbasedn="dc=antonio,dc=gonzalonazareno,dc=org" \
--enablemkhomedir \
--update
```

![Descripción de la imagen](/images/slap1-11.png)

