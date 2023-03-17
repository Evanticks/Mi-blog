---
title: Poblando un directorio LDAP con datos CSV
Categoria: Administración de sistemas
---

![ldap.png](/images/ldap-csv-logo.png)

# Poblando un directorio LDAP con datos CSV

Comenzaremos conectándonos a alfa y establecer un fichero csv que es simplemente un fichero separado por comas:

```
Belen,Nazareth,belennazareth@gmail.com,antonio,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC73j7AidXdLgiu5wJw7YgJuvOHyb6U8c04MuQyehYnMknMR8mTnWZr20npVHJ8VHYHDy8RlgbkMMBFgeVCgXJ+Im3A6Efp6HC4yj2SM+73hr1EKCLdRPzCzdtDSUtkqU9k+x2RdF3T6qD6H4Cg/nT8Sg3Qenqds4XORfDWOvntxFja2D0OhZv1MLPUD9pEj+a8D4erfiPx/gKW/Rtu89une+uiwVgK60B5CxnC8XXnXmPO3NhrgyQhVgzQZ658cUbLooxQURVlo1gnOmcqX5h+svUKN1SDbzTyy7HKSk7bbLHEhk7qDh7jSzcf80GLU0li8vXc2to8NpC00EOQ9POPivESz23gMNY8ooDtNU3Ll/xYvhtvXrJNTbuBiuVLzuopMvrQi6LVsQEWmPJzBiJ2qt8JW1KRLcnWRL4AezbxAPXuRYVnYBS3it6L0J4AZjZg63BkIIrfU7GYzrKb+z5mqUgDJhIZ4d5av+OAxPSSzNeVnyWEnWrI0k9kf9qmqhU= nazare@ThousandSunny 
Roberto,Rodriguez,roberroberto@gmail.com,antonio,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDV+kKxQNDAQzjU+v5vKBdDBryGRZVx8isCL+VIyZM44qVELXgSA8ZAKfv6YqOwF669WIGD7LKUAZ9PpNk0kWJCiDkHUMW1KmAgaqzo0NAAux0cdoMDfHDpREvlDwS5Ryr64H08oJsNh3IAqefrAU0rszFAkK2ycCLI4Dl0bqO4zNMAqkbDdrhk61jB3nZEAvDTvxZ/3c+mX4RlWBeCARPR9Hq0AuGcR5JAdQf74AySk6mbu0h8mw3DRnU9dxdrCkkhEuPlAVo7uh6i81NDfFLxkddjv46P9aDDvQxxhmv2jCFjx7PUuDvGb0MoCDXV+C4oZyJ0HzUkwX2YuqRTVcE8wvTIxjeQ/ABRgMBfYMr7qvwPV8v2lrylfmHw4A3vxDHmS3kQojnFUxc9lWTWRtT8Ab+3PIPPU+a09Mw7N9dEdusljhviLMZ83ni7qTtfd4IBpS3oVS9JbgkpxOxNJw9bQlFqm+hwKH2sRmjgZft3QcOy1JAwGTViI8eW1mCyl3U= roberto@portatil
```


Crearemos un fichero ldif que será el que se encargue de poblar el directorio:


```
dn: cn=openssh-lpk,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: openssh-lpk
olcAttributeTypes: ( 1.3.6.1.4.1.24552.500.1.1.1.13 NAME 'sshPublicKey'
  DESC 'MANDATORY: OpenSSH Public key'
  EQUALITY octetStringMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.40 )
olcObjectClasses: ( 1.3.6.1.4.1.24552.500.1.1.2.0 NAME 'ldapPublicKey' SUP top AUXILIARY
  DESC 'MANDATORY: OpenSSH LPK objectclass'
  MAY ( sshPublicKey $ uid )
  )
```

Importaremos la nueva configuración con el siguiente comando:

```
ldapadd -Y EXTERNAL -H ldapi:/// -f openssh-lpk.ldif
```

![img](/images/ldap-2-1.png)

Ahora crearemos un programa en python en el cual podremos leer el fichero csv y crear un fichero ldif que será el que se encargue de poblar el directorio:

Para ello instalaremos python y crearemos el entorno virtual:

```
apt install python3-venv -y

python3 -m venv ldap-csv

source ldap-csv/bin/activate
```


Seguidamente crearemos el programa en python:

```
#!/usr/bin/env python

import ldap3
from ldap3 import Connection, ALL
from getpass import getpass
from sys import exit

### VARIABLES

# Shell que se le asigna a los usuarios
shell = '/bin/bash'

# Ruta absoluta del directorio que contiene los directorios personales de los usuarios. Terminado en "/"
home_dir = '/home/ldap/'

# El valor inicial para los UID que se asignan al insertar usuarios. 
uid_number = 10000

# El GID que se le asigna a los usuarios. Si no se manda al anadir el usuario da error.
gid = 10000

### VARIABLES

# Leemos el fichero .csv de los usuarios y guardamos cada linea en una lista.
with open('usuarios.csv', 'r') as usuarios:
  usuarios = usuarios.readlines()


### Parametros para la conexion
ldap_ip = 'ldap://alfa.antonio.gonzalonazareno.org:389'
dominio_base = 'dc=antonio,dc=gonzalonazareno,dc=org'
user_admin = 'admin' 
contrasena = getpass('Contrasena: ')

# Intenta realizar la conexion.
conn = Connection(ldap_ip, 'cn={},{}'.format(user_admin, dominio_base),contrasena)

# conn.bind() devuelve "True" si se ha establecido la conexion y "False" en caso contrario.

# Si no se establece la conexion imprime por pantalla un error de conexion.
if not conn.bind():
  print('No se ha podido conectar con ldap') 
  if conn.result['description'] == 'invalidCredentials':
    print('Credenciales no validas.')
  # Termina el script.
  exit(0)

# Recorre la lista de usuarios
for user in usuarios:
  # Separa los valores del usuario usando como delimitador ",", y asigna cada valor a la variable correspondiente.
  user = user.split(',')
  cn = user[0]
  sn = user[1]
  mail = user[2]
  uid = user[3]
  ssh = user[4]

  #Anade el usuario.
  conn.add(
    'uid={},ou=Personas,{}'.format(uid, dominio_base),
    object_class = 
      [
      'inetOrgPerson',
      'posixAccount', 
      'ldapPublicKey'
      ],
    attributes =
      {
      'cn': cn,
      'sn': sn,
      'mail': mail,
      'uid': uid,
      'uidNumber': str(uid_number),
      'gidNumber': str(gid),
      'homeDirectory': '{}{}'.format(home_dir,uid),
      'loginShell': shell,
      'sshPublicKey': str(ssh)
      })

  if conn.result['description'] == 'entryAlreadyExists':
    print('El usuario {} ya existe.'.format(uid))

  # Aumenta el contador para asignar un UID diferente a cada usuario (cada vez que ejecutemos el script debemos asegurarnos de ante mano que no existe dicho uid en el directorio ldap, o se solaparian los datos)
  uid_number += 1

#Cierra la conexion.
conn.unbind()

```

Lo ejecutaremos:

```
python3 poblarusuarios-csv.py
```

![img](/images/ldap-2-2.png)

```
ldapsearch -x -D "cn=admin,dc=antonio,dc=gonzalonazareno,dc=org" -b "dc=antonio,dc=gonzalonazareno,dc=org" -W
```

Roberto:

![img](/images/ldap-2-3.png)

Nazareth:

![img](/images/ldap-2-4.png)

nano /etc/ldap/ldap.conf

```
BASE dc=antonio,dc=gonzalonazareno,dc=org
URI ldap://alfa.antonio.gonzalonazareno.org
```


Y hacemos esto para que cuando un usuario entre puedan crearse sus respectivos directorios:

```
echo "session    required        pam_mkhomedir.so" >> /etc/pam.d/common-session
```


Ahora vamos a hacer un script que luego adjuntaremos en el servicio de sshd config para que encuentre las claves públicas de los usuarios:

nano /opt/buscarclave.sh

```
#!/bin/bash
ldapsearch -x -u -LLL -o ldif-wrap=no '(&(objectClass=posixAccount)(uid='"$1"'))' 'sshPublicKey' | sed -n 's/^[ \t]*sshPublicKey::[ \t]*\(.*\)>
```

Le damos permisos de ejecución:

```
chmod u + x /opt/buscarclave.sh 
```


Vemos que el programa se ejecuta correctamente:

![img](/images/ldap-2-5.png)


nano /etc/ssh/sshd_config

```
AuthorizedKeysCommand /opt/buscarclave.sh
AuthorizedKeysCommandUser nobody
```

Y por último reiniciamos el servicio ssh:

```
systemctl restart sshd
```



Aquí comprobamos que Nazareth puede conectarse a alfa:

![img](/images/nazareth1.jpeg)


Y comprobamos que Roberto puede conectarse:

![img](/images/rober1.jpeg)