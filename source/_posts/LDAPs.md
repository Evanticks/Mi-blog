---
title: LDAPs
Categoría: Sistemas Operativos
---

![ldaps](/images/ldaps-logo.png)

# LDAPs

## LDAPs en Alfa

Primero vamos a generar tanto el certificado público como el privado:

```
openssl genrsa 4096 > /etc/ssl/private/alfa.key
openssl req -new -key /etc/ssl/private/alfa.key -out alfa.csr
```

Ahora vamos a descargar el csr, lo enviaremos a la entidad certificadora del IES Gonzalo Nazareno, y cuando nos devuelvan el certificado público, lo guardaremos en el servidor como alfa.crt.

Una vez tengamos el certificado público, lo importaremos en el servidor:


```
mv alfa.crt /etc/ssl/certs/

mv gonzalonazareno.crt /etc/ssl/certs/

chown root:root /etc/ssl/certs/alfa.crt

chown root:root /etc/ssl/certs/gonzalonazareno.crt
```


Ahora debemos instalar acl para que openldap pueda acceder a los certificados:

```
apt install acl

setfacl -m u:openldap:r-x /etc/ssl/private

setfacl -m u:openldap:r-x /etc/ssl/private/alfa.key

getfacl /etc/ssl/private

getfacl /etc/ssl/private/alfa.key
```


![getfacl](/images/ldaps-1.png)


nano ldaps.diff

```
dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/gonzalonazareno.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/alfa.key
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/alfa.crt
```

Ahora importamos el diff:

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f ldaps.ldif
```

![ldapmodify](/images/ldaps-2.png)

Sustituímos la línea en:

nano /etc/default/slapd

```
SLAPD_SERVICES="ldaps:///"
```

```
cp /etc/ssl/certs/gonzalonazareno.crt /usr/local/share/ca-certificates/
update-ca-certificates
```

Podemos comprobar que todo funciona correctamente desde alfa ejecutando el siguiente comando:

```
ldapsearch -x -b "dc=antonio,dc=gonzalonazareno,dc=org" -H ldaps://localhost:636
```

![ldapsearch](/images/ldaps-4.png)


## LDAPs en Delta

Ahora desde nuestro local enviamos a delta el crt de la CA que es el gonzalonazareno.crt:

```
scp gonzalonazareno.crt antonio@delta:
```

![scp](/images/ldaps-3.png)

Ahora entramos a delta:

```
mv gonzalonazareno.crt /usr/local/share/ca-certificates/
chown root:root /usr/local/share/ca-certificates/gonzalonazareno.crt
update-ca-certificates
```



Una vez hecho esto ejecutamos el siguiente comando para comprobar que todo funciona correctamente:

```
ldapsearch -x -b "dc=antonio,dc=gonzalonazareno,dc=org" -H ldaps://alfa.antonio.gonzalonazareno.org:636
```


![ldapsearch](/images/ldaps-5.png)





Tras esto forzamos a delta a que utilice solamente LDAPS:

```
nano /etc/ldap/ldap.conf

URI     ldaps://192.168.0.1
```


## LDAPs en Bravo:


Volvemos a mandar el certificado de la CA de local a bravo:

```
scp gonzalonazareno.crt antonio@bravo:
```

![scp](/images/ldaps-6.png)

Ahora entramos a bravo:

```
mv gonzalonazareno.crt /etc/pki/ca-trust/source/anchors/
chown root:root /etc/pki/ca-trust/source/anchors/gonzalonazareno.crt
update-ca-trust
```

volvemos a hacer lo mismo en bravo para forzar el uso de LDAPS:

```
nano /etc/openldap/ldap.conf

URI ldaps://alfa.antonio.gonzalonazareno.org
```

Ahora comprobamos que funcina ejecutando el mismo comando que en delta ya que el host sigue siendo alfa:

![ldapsearch](/images/ldaps-7.png)




Hacemos que conecte el servidor de bravo con el nfs que se encuentra en alfa:


nano /etc/systemd/system/home.mount

```
[Unit]
Description= Montaje de carpeta home para NFS

[Mount]
What=172.16.0.1:/home/antonio/nfs/
Where=/home
Type=nfs
Options=defaults

[Install]
WantedBy=multi-user.target

```

luego vamos a reninicar el servicio de ldap en rocky:

```
systemctl restart sssd
```




Ahora comprobamos que el nfs con el usuario macarena creado en alfa a través de ldap puede acceder a la carpeta home de bravo, leer y crear ficheros:

![ldapsearch](/images/ldaps-8.gif)