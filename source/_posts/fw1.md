---
title: Firewall 1
Categories: firewall
---

![ssh](/images/firewall1-logo.png)


Ejecutaremos primero las reglas de entrada y salida para poder hacer conexiones ssh a la propia máquina.

```
iptables -A INPUT -s 172.22.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 172.22.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

iptables -A INPUT -s 172.29.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 172.29.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

![ssh](/images/fw1-1.png)

Ponemos como política por defecto una lista blanca, con lo cual rechazaremos todas las conexiones:

```
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

Si hacemos un ping a la ip de la máquina, no nos responde, ya que no hemos permitido el acceso a la ip de la máquina de ninguna forma, solo a través de ssh.

![ssh](/images/fw1-2.png)


Ahora dentro de la máquina permitiremos el ping del localhost:

```
iptables -A INPUT -i lo -p icmp -j ACCEPT
iptables -A OUTPUT -o lo -p icmp -j ACCEPT
```

![ssh](/images/fw1-3.png)

Ahora vamos a permitir el ping tanto de entrada como de salida de la máquina, para ello haremos un ping a una ip pública:

```
iptables -A INPUT -i ens3 -p icmp --icmp-type echo-reply -j ACCEPT
iptables -A OUTPUT -o ens3 -p icmp --icmp-type echo-request -j ACCEPT
```

![ssh](/images/fw1-4.png)


Ahora vamos a permitir las consultas DNS de entrada y salida de la máquina al exterior:

```
iptables -A OUTPUT -o ens3 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens3 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

![ssh](/images/fw1-5.png)


A continuación permitiremos el tráfico web de entrada y salida de la máquina al exterior por http y https:

```
iptables -A OUTPUT -o ens3 -p tcp --match multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens3 -p tcp --match multiport --sports 80,443 -m state --state ESTABLISHED -j ACCEPT
```

Muestro la prueba de que puedo navegar al puerto 80 y a mi web que es a través de https:

![ssh](/images/fw1-6.png)



Ahora vamos a permitir el tráfico hacia el servidor apache instalado:

```
iptables -A INPUT -i ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```


![ssh](/images/fw1-7.png)

# Ejercicios

## Permite poder hacer conexiones ssh al exterior.

```
iptables -A INPUT -i ens3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
```

Nos conectamos a otra máquina que tenemos en la red de Openstack

![ssh](/images/fw1-8.png)

## Deniega el acceso a tu servidor web desde una ip concreta.

Vamos a denegar el acceso web a mi máquina local, con lo cual vamos a bloquear la ip de la vpn:

```
iptables -A INPUT ! -s 172.29.0.14 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT ! -d 172.29.0.14 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

![ssh](/images/fw1-9.png)


## Permite hacer consultas DNS sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.


Primero vamos a borrar las reglas que permite el acceso a todos los DNS:

```
iptables -L --line-numbers
```


```
iptables -D INPUT 6
iptables -D OUTPUT 6
```


```
iptables -A INPUT -s 192.168.202.2 -p udp --sport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 192.168.202.2 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
```


![ssh](/images/fw1-10.png)


## No permitir el acceso al servidor web de www.josedomingo.org (Tienes que utilizar la ip). ¿Puedes acceder a fp.josedomingo.org?

```
dig www.josedomingo.org

37.187.119.60
```

Ahora vamos a establecer una regla por encima a la de permitir todas las conexiones al puerto 80, porque cuando lee esa regla, ignora la del bloqueo que podamos ponerle, entonces establecemos la regla por encima de la que permite todas las conexiones al puerto 80:

Primero averiguamos la posición de la regla con iptaules -L --line-numbers

![ssh](/images/fw1-11.png)

Vemos que se encuentra en la posición 6, entonces vamos a insertar la regla en la posición 5:

```
iptables -I OUTPUT 6 -d 37.187.119.60 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j DROP
```

Ahora vemos que la regla queda por encima:

![ssh](/images/fw1-12.png)


Podemos acceder a mi web, pero no a la de www.josedomingo.org ni a la de fp.josedomingo.org ya que se encuentran bajo la misma ip

![ssh](/images/fw1-13.png)

## Permite mandar un correo usando nuestro servidor de correo: babuino-smtp. Para probarlo ejecuta un telnet bubuino-smtp.gonzalonazareno.org 25.

apt update && apt install telnet -y

dig babuino-smtp.gonzalonazareno.org

```
iptables -A INPUT -s 192.168.203.3 -p tcp --sport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 192.168.203.3 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
```

Ahora podemos comprobar que podemos mandar un correo a babuino-smtp.gonzalonazareno.org por el puerto 25:

![ssh](/images/fw1-14.png)


## Instala un servidor mariadb, y permite los accesos desde la ip de tu cliente. Comprueba que desde otro cliente no se puede acceder.

Instalamos el servidor mariadb:

```
apt install mariadb-server -y
```

Luego vamos al archivo de configuración de mariadb:

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

y ponemos el bind-address:

```
bind-address            = 0.0.0.0
```

Luego establecemos las reglas dando paso a la ip de mi cliente:

```
iptables -A INPUT -s 10.0.0.112 -p tcp --dport 3306 -j ACCEPT
iptables -A OUTPUT -d 10.0.0.112 -p tcp --sport 3306 -j ACCEPT
```

En la izquierda vemos que la máquina cliente puede acceder al servidor mariadb, pero en la derecha no, ya que no le hemos dado paso a delta de nuestro escenario:

![ssh](/images/fw1-15.png)


Pero si le damos permiso a delta:

Para ello debemos darle permiso a la ip pública del escenario, que sería alfa:

```
iptables -A INPUT -s 10.0.0.175 -p tcp --dport 3306 -j ACCEPT
iptables -A OUTPUT -d 10.0.0.175 -p tcp --sport 3306 -j ACCEPT
```

Ahora podemos ver como delta puede conectar con el servidor mariadb:

![ssh](/images/fw1-16.png)