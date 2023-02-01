---
title: Configurar un entorno DNS con vistas.
categories: Servicios de Red e Internet
tags: Servicios de Red e Internet
---


![Descripción de la imagen](/images/vistas-dns-1.png)



Instalaremos un servidor DNS en charlie, de tal forma que todos los nombres de las máquinas deben tener resolución unas con otras al preguntar a charlie.

![Descripción de la imagen](/images/Untitled-2023-01-10-0143.png)

Las vistas sirven para que cada máquina en un entorno pueda ver las correspondientes salidas a nombres que se hallan dentro del entorno, para esto debemos hacer una configuración en el /etc/bind/named.conf.local, que quedaría de la siguiente manera:

```
view interna {
    match-clients { 192.168.0.0/24; 127.0.0.1; };
    allow-recursion { any; };
        zone "antonio.gonzalonazareno.org"
        {
               type master;
               file "db.interna.antonio.gonzalonazareno.org";
        };
        zone "0.168.192.in-addr.arpa"
        {
               type master;
               file "db.0.168.192";
        };
        zone "16.172.in-addr.arpa"
        {
               type master;
               file "db.16.172";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};

view externa {
    match-clients { 172.22.0.0/16; 192.168.202.2; 172.29.0.0/16;};
    allow-recursion { any; };
        zone "antonio.gonzalonazareno.org"
        {
               type master;
               file "db.externa.antonio.gonzalonazareno.org";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};

view dmz {
    match-clients {172.22.0.0/16; 172.16.0.0/16; 172.29.0.0/16;};
    allow-recursion { any; };
        zone "antonio.gonzalonazareno.org"
        {
               type master;
               file "db.dmz.antonio.gonzalonazareno.org";
        };
        zone "16.172.in-addr.arpa"
        {
               type master;
               file "db.16.172";
        };
        zone "0.168.192.in-addr.arpa"
        {
               type master;
               file "db.0.168.192";
        };
        include "/etc/bind/zones.rfc1918";
        include "/etc/bind/named.conf.default-zones";
};

```

Por cada rango de ips, tenemos una vista, ¿por qué tenemos 5 vistas? pues la respuesta sería la siguiente:
- La vista interna que será la vista que tendrán alfa, charlie y delta que contendrá 192.168.0.0
- La vista externa que será la que se muestre al exterior con 172.22.0.0
- La vista dmz que tendrá a bravo con 172.16.0.0
- Tanto la vista dmz como la interna tendrán respuesta sobre resoluciones inversas, pero en la externa no la tenemos, esto es debido a que no debemos proporcionar más información de la esencial a las preguntas desde la vista externa, ya que proviene del exterior y por tanto es menos seguro.

Una vez creada las zonas debemos crear los ficheros en /var/cache/bind/

## Zona externa:

```
$TTL    86400
@       IN      SOA     alfa.antonio.gonzalonazareno.org. root.antonio.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              alfa.antonio.gonzalonazareno.org.

$ORIGIN antonio.gonzalonazareno.org.

alfa                    IN      A               172.22.200.193
dns                     IN      CNAME           alfa
www                     IN      CNAME           alfa
```

Como podemos ver, hay varios cname, esto significa que en las preguntas dns que se realizarán desde el exterior a alfa, la respuesta sería tanto que alfa es el dns como el www, ya que no hay que proporcionar la información del interior.

## Zona interna:

```
$TTL    86400
@       IN      SOA     charlie.antonio.gonzalonazareno.org. root.antonio.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              charlie.antonio.gonzalonazareno.org.

$ORIGIN antonio.gonzalonazareno.org.

alfa                    IN      A               192.168.0.1
bravo                   IN      A               172.16.0.200
charlie                 IN      A               192.168.0.2
delta                   IN      A               192.168.0.3
bd                      IN      CNAME           delta
dns                     IN      CNAME           charlie
www                     IN      CNAME           bravo
```

Ahora en la vista interna podemos ver la información de las ip de las máquinas ya que es un entorno de producción seguro.

## Zona interna inversa:

```
$TTL    86400
@       IN      SOA     charlie.antonio.gonzalonazareno.org. root.antonio.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              charlie.antonio.gonzalonazareno.org.

$ORIGIN 0.168.192.in-addr.arpa.

1                       IN      PTR             alfa.antonio.gonzalonazareno.org.
2                       IN      PTR             charlie.antonio.gonzalonazareno.org.
3                       IN      PTR             delta.antonio.gonzalonazareno.org.
```

Es importante el '.' después del nombre del fdqn de la máquina ya que si no lo hacemos lo concatenará en la resolución con la ip inversa.


## Zona DMZ:

```
$TTL    86400
@       IN      SOA     charlie.antonio.gonzalonazareno.org. root.antonio.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              charlie.antonio.gonzalonazareno.org.

$ORIGIN antonio.gonzalonazareno.org.

alfa                    IN      A               172.16.0.1
bravo                   IN      A               172.16.0.200
charlie                 IN      A               192.168.0.2
delta                   IN      A               192.168.0.3
bd                      IN      CNAME           delta
dns                     IN      CNAME           charlie
www                     IN      CNAME           bravo
```

La diferencia que podemos apreciar de esta vista es que alfa posee el gateway 172.16.0.1, que es la que verá bravo.

## Zona DMZ inversa:

```
$TTL    86400
@       IN      SOA     charlie.antonio.gonzalonazareno.org. root.antonio.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              charlie.antonio.gonzalonazareno.org.

$ORIGIN 16.172.in-addr.arpa.

0.200                   IN      PTR             bravo.antonio.gonzalonazareno.org
```

Una vez hecho esto debemos hacer systemctl restart bind9 y comprobamos que no haya errores de sintaxis, después podemos porbar a realizar consultas con dig.

Ahora una vez realizadas las configuraciones, podemos jugar un poco con esto! y es que por ejemplo si tenemos una base de datos en delta y el cliente de esta base de datos está en bravo, podemos entrar poniendo el cname que especificamos en el dns.


![Descripción de la imagen](/images/prueba-dns-mysql.png)


Y si en el caso que en el /etc/resolv.conf ponemos un search con el nombre del dominio `search antonio.gonzalonazareno.org` pues no haría falta especificar siquiera el dominio, como en el siguiente ejemplo:

![Descripción de la imagen](/images/prueba-dns-mysql2.png)

Por último dejamos la web de alfa que realmente estará alojada en bravo:

![Descripción de la imagen](/images/web-alfa.png)


## Tips

Si queremos servir una web que está en un servidor que no pasa por alfa pero no es alfa, debemos poner una regla de iptables que nos permita dirigir el tráfico por el puerto que especifiquemos a bravo, para poder así acceder desde fuera.

Si necesitamos acceder a este entorno desde un servidor dns que lo controle, debemos editar el archivo /etc/bind/named.conf.options

```
 forwarders {
      192.168.202.2;
 };

```