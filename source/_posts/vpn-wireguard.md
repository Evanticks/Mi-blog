---
title: VPN Wireguard
categories: Seguridad
tags: Seguridad
---

![wireguard-site-1.png](/images/logo-wireguard.png)

# Site to Site

## Linux

Primero vamos a cada cliente y vamos a establecer la ruta por defecto

```
Cliente 1:
    sudo ip route del default
    sudo ip route add default via 192.168.200.1
```

```
Cliente 2:

    sudo ip route del default
    sudo ip route add default via 192.168.100.1
```


Ahora nos conectarems a **cada servidor** de manera que en cada uno realizaremos los siguientes comandos:
```
Instalamos wireguard
    sudo apt update
    sudo apt install wireguard
```

Creamos la clave privada y pública

`wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key`

```
cat /etc/wireguard/server_private.key
cat /etc/wireguard/server_public.key
```
El resultado de los comandos será cada clave pública y privada de los servidores

Creo el fichero de configuración en el que la interfaz se llamará wg0.conf

    sudo nano /etc/wireguard/wg0.conf

Después de esto en vpn1 quedará el archivo de configuración de la siguiente manera:

```
[Interface]
Address = 10.99.99.1
PrivateKey = UKn7oartqeJjRPk8jUtACSTYn/e23wszOBxT/Ymo1Uk=
ListenPort = 51820

[Peer]
PublicKey = PWrfuUd1UODnjTPuFCJfy/eixRTEsCq/KioI74/vDFo=
AllowedIPs = 0.0.0.0/0
Endpoint = 10.0.0.1:51820
PersistentKeepAlive = 25

```


El siguiente archivo de configuración es el de vpn2:

```
[Interface]
Address = 10.99.99.2
PrivateKey = MCL13K+q/43eyF8qLtBxz9cNhvcFxQ2s98LOiseZy3w=
ListenPort = 51820

[Peer]
PublicKey = 2az5T5rW/7x1Z/YLRfX7saNteaOsoupFh07Z6VkeUjI=
AllowedIPs = 0.0.0.0/0
Endpoint = 10.0.0.10:51820
```

Ahora en **cada servidor** emplearemos 

`wg-quick up wg0` para activar el servicio
`wg-quick down wg0` para desactivar el servicio
`wg` para poder ver el estado del servicio

En los servidores hay que intercambiar las respectivas claves públicas e ingresarlo en PublicKey.
Address es la ip del túnel que se va a asignar en cada extremo del servidor.
AllowedIPs es las ip permitidas hacia el servidor
Endpoint es la ip pública del servidor y el puerto que se ha configurado en el servidor opuesto.
PersistentKeepAlive es el tiempo en el que se realizan las comprobaciones de conexión.


Ahora entraremos a cada cliente y realizaremos la comprobación de que pueden conectarse:


![wireguard-site-1.png](/images/wireguard-site-1.png)


## Cliente Windows


Ahora vamos a proceder a conectar un segundo cliente windows:


![wireguard-site-2.png](/images/wireguard-site-2.png)


Ahora cambiaremos la interfaz para establecer una ip estática:


![wireguard-site-2.png](/images/wireguard-site-3.png)

Una vez hecho esto probamos a hacer un ping para ver que podemos comunicarnos en remoto a través de la VPN

![wireguard-site-2.png](/images/wireguard-site-4.png)


# Point to Site

La misma configuración que en el site to site, solo que no activaremos el bit de forwarding en el cliente final ya que este mismo no hará de router.


```
[Interface]
Address = 10.99.99.1
PrivateKey = 4GLQzZT6G/Oq69aAitajxl7ZRtzuV1cFrsI0j4Cqjk4=
ListenPort = 51820

[Peer]
Publickey = fKWBm53DKrAAukk6nssSZ0DuVy9jEBIRKbqOS22sdg8=
AllowedIPs = 0.0.0.0/0
PersistentKeepAlive = 25
Endpoint = 192.188.188.20:51820

```


![wireguard-site-1.png](/images/vpn-cliente-C-lin.png)



Luego conectaremos la interfaz en windows como explicamos anteriormente y activaremos la configuración de la VPN como hemos hecho en linux, solo que esta vez su ip de la ruta será 
10.99.99.4/32

![wireguard-site-1.png](/images/vpn-cliente-C-win2.png)


Finalmente copiamos la configuración en el servidor con la clave pública de la vpn de windows y de esta manera reiniciando el servicio con wg-quick down wg0 y wg-quick up wg0 ya tendremos la vpn funcionando.

```
[Peer]
Publickey = I/Gj50Ue+qmv+a4Q+C1VcJ4VptBVnU3QfDgo//vq+Fw=
AllowedIPs = 0.0.0.0/0
PersistentKeepAlive = 25
Endpoint = 192.188.188.21:51820
```


![wireguard-site-1.png](/images/vpn-cliente-C-win.png)

