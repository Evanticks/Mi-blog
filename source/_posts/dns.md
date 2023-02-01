---
title: Configurar un servidor DNS.
categories: Servicios de Red e Internet
tags: Servicios de Red e Internet
---

![Descripción de la imagen](/images/obelix-4.jpg)


## Servidor maestro

Primero comenzaremos actualizando el sistema e instalando el servidor dns bind9, la máquina deberá tener un nombre full qualificated como en mi caso que sería **dns1.antonio.org**

`apt update && apt install bind9 -y`

Vamos a forzar la salida de las peutciones por ipv4, ya que al no estar configurado de esta forma pueden perderse paquetes al tratar de viajar a través de ipv6, esto lo haremos en /etc/default/named, especificando esta sentencia en  `OPTIONS="-4 -f -u bind"`.

Primero debemos permitir el tráfico desde las diferentes ip en el fichero /etc/bind/named.conf.options.

```
allow-query {172.29.0.0/16; 172.22.0.0/16;};
allow-transfer { none ;};
```

Allow-query se encargará de permitir las consultas a través de los distintos rangos de ip, yo al estar conectado a una vpn debo poner también ese rango para así poder realizar las consultas desde casa.

Allow-transfer none hará que no se realicen transpasos completos en la información de la zona, esto nos permitirá que no podamos exponer información sensible dentro del servidor dns al realizar las consultas con dig.

Tras esto, comenzaremos editando el fichero de configuración de zonas, el cual se encargará de administrar las plantillas a las que se les va a realizar una consulta dns con dig.

/etc/bind/named.conf.local

```shell
include "/etc/bind/zones.rfc1918";
zone "antonio.org" {
    type master;
    file "db.antonio.org";
    allow-transfer { 172.22.7.171; };
    notify yes;
};

zone "22.172.in-addr.arpa" {
    type master;
    file "db.172.22.0.0";
    allow-transfer { 172.22.7.171; };
    notify yes;
};

```

Nuestra zona será antonio.org, el cual tendrá el fichero db.antonio.org, hasta ahí será necesario para ralizar solo un dns, para realizar el dns respaldado por un esclavo necesitaremos conceder el traspaso a la ip 172.22.7.171 que será la ip de la máquina esclava, más abajo nos econtraremos con el fichero de zona de resolución inversa, el cual tendrá el nombre del rango de ip que abarcará, y la misma transferencia de zona y el notify que será lo que avise al esclavo que ha habido un cambio en el maestro.

Si nos vamos al fichero /var/cache/bind/db.antonio.org podemos ver los siguientes componentes:

```shell
$TTL    86400
@       IN      SOA     dns1.antonio.org. root.antonio.org. (
                              6         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;

@       IN      NS              dns1.antonio.org.
@       IN      NS              dns2.antonio.org.
@       IN      MX      10      correo.antonio.org.


$ORIGIN antonio.org.

dns1                    IN      A       172.22.7.155
dns2                    IN      A       172.22.7.171
correo                  IN      A       172.22.200.101
asterix                 IN      A       172.22.200.102
obelix                  IN      A       172.22.200.103
prueba                  IN      A       172.22.200.120
www                     IN      CNAME   asterix
;informatica             IN      CNAME   asterix
ftp                     IN      CNAME   obelix
entrebytes              IN      A       172.22.200.166

$ORIGIN informatica.antonio.org.
@       IN      NS    dns
dns     IN      A     172.22.7.177

```

Pues bien, deberemos cumplimentar de la siguiente forma el fichero, el cual tiene las siguientes definiciones:
La primera parte es el SOA, que es fdqn de la máquina maestra, acompañado del correo del administrador, el cual será con puntos.

En la segunda parte podemos ver los nameservers, el cual tendrá el @, esto mostrará las máquinas a disposición del fdqn que hemos elegido, en nuestro caso tendremos el de la máquina maestra, la del esclavo y la del correo.

Origin es la variable que va a tener antonio.org. para no tener que repetir continuamente el nombre del dominio en los registros.

Los registros de tipo A tendrán una ip asociada, estos se encargan de darle nombre a la ip de dentro del dominio, como dns1 que tiene una 172.22.7.155 o la del esclavo, que tendrá la 172.22.7.171

Los registros de tipo CNAME son nombres relacionados a otros nombres, como por ejemplo ftp, que va relacionado al nombre obelix y que tendrá la ip 172.22.200.103. entonces al realizar la consulta en el apartado ANSWER nos daría el nombre completo al que está asociado.

![Descripción de la imagen](/images/obelix.png)


Ahora vamos con el fichero de zona de resolución inversa, el cual crearemos en /var/cache/bind/db.172.22.0.0

```shell
$TTL    86400
@       IN      SOA     dns1.antonio.org. root.antonio.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@      IN      NS              dns1.antonio.org.
@      IN      NS              dns2.antonio.org.

$ORIGIN 22.172.in-addr.arpa.

155.7                  IN      PTR             dns1.antonio.org.
171.7                  IN      PTR             dns2.antonio.org.
101.200                IN      PTR             correo.antonio.org.
102.200                IN      PTR             asterix.antonio.org.
200.103                IN      PTR             obelix.antonio.org.
200.166                IN      PTR             entrebytes.antonio.org.

```

El fichero es muy similar al de la resolución directa, pero en este caso especificaremos la IP de la manera opuesta, con PTR conseguiremos obtener el nombre gracias a una petición inversa a la ip que hemos configurado, también pondremos la del servidor esclavo.

![Descripción de la imagen](/images/obelix-2.png)

Por último, debemos editar el fichero /etc/bind/zones.rfc1918 que se encarga de proporcionar la plantilla empty para las resoluciones no configuradas, de manera que debamos comentar el rango de ips de `//zone "22.172.in-addr.arpa"  { type master; file "/etc/bind/db.empty"; };` ya que nosotros hemos configurado anteriormente.

ahora solo nos queda realizar un `systemctl restart bind9` y `systemctl status bind9` para comprobar de que el servicio está activo y sin errores.

## Servidor esclavo

Primero comenzaremos actualizando el sistema e instalando el servidor dns bind9, la máquina deberá tener un nombre full qualificated como en mi caso que sería **dns2.antonio.org**

`apt update && apt install bind9 -y`

Vamos a forzar la salida de las peutciones por ipv4, ya que al no estar configurado de esta forma pueden perderse paquetes al tratar de viajar a través de ipv6, esto lo haremos en /etc/default/named, especificando esta sentencia en  `OPTIONS="-4 -f -u bind"`.

/etc/bind/named.conf.local

```shell
include "/etc/bind/zones.rfc1918";
zone "antonio.org" {
    type slave;
    file "db.antonio.org";
    masters { 172.22.7.155; };
};
zone "22.172.in-addr.arpa" {
    type slave;
    file "db.172.22.0.0";
    masters { 172.22.7.155; };
};
```

Hacemos un `systemctl restart bind9` y `systemctl status bind9` para comprobar de que el servicio está activo y sin errores.

Tras esto, cuando modifiquemos el servidor maestro subimos el serial de tal forma que sea mayor al de la máquina esclava, y de esta manera realizará la transferencia.

![Descripción de la imagen](/images/obelix-3.png)

## Delegación de subdominio

Pra delegar un subdominio primero comenzaremos actualizando el sistema e instalando el servidor dns bind9, la máquina deberá tener un nombre full qualificated como en mi caso que sería **dns.informatica.antonio.org**

`apt update && apt install bind9 -y`

Vamos a forzar la salida de las peutciones por ipv4, ya que al no estar configurado de esta forma pueden perderse paquetes al tratar de viajar a través de ipv6, esto lo haremos en /etc/default/named, especificando esta sentencia en  `OPTIONS="-4 -f -u bind"`.

Una vez hecho esto, como podemos ver en la máquina maestra, tenemos un apartado en db.antonio.org en el cual especificamos los siguientes parámetros:

```shell
$ORIGIN informatica.antonio.org.
@       IN      NS    dns
dns     IN      A     172.22.7.177
```

Con esto estaremos delegando el subdominio para que lo gestione dns.informatica.antonio.org, si nos vamos a esta máquina, debemos configurar el dominio de la siguiente forma:

/etc/bind/named.conf.local
```
zone "informatica.antonio.org" {
    type master;
    file "db.informatica.antonio.org";
};
```

Luego nos vamos a /var/cache/bind/db.informatica.antonio.org

```shell
$TTL    86400
@       IN      SOA     dns.informatica.antonio.org. root.informatica.antonio.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              dns.informatica.antonio.org.
@       IN      MX      10      mail.informatica.antonio.org.

$ORIGIN informatica.antonio.org.

dns                     IN      A               172.22.7.177
mail                    IN      A               172.22.7.177
web                     IN      A               172.22.7.177
www                     IN      CNAME           web
entrebytes              IN      CNAME           web

```


## Tips

Si realizamos cambios a veces no se verán reflejados ya que el servicio DNS la almacena en caché, para borrarla necesitaremos ejecutar `rndc flush` en la máquina maestra.

Para poder realizar las consultas sin especificar la ip a través de @, necesitaremos agregar un nameserver XXX.XXX.XXX.XXX en /etc/resolv.conf

En el caso de dns.informatica.antonio.org no es necesario descomentar el include "/etc/bind/zones.rfc1918"; ya que se ocupará de gestionarlo la máquina maestra.

Para detectar con información más detallada los errores de plantillas db, necesitaremos ejecutar `named-checkzone antonio.org /var/cache/bind/db.antonio.org`

Si queremos detectar los errores del fichero de zonas, necesitaremos ejecutar `named-checkconf`

