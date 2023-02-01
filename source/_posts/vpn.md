---
title: Cliente-Servidor Openvpn
categories: Seguridad
tags: Seguridad
---


![Descripción de la imagen](/images/vpn-logo.png)


# Cliente-Servidor Openvpn



## Servidor

Tendremos un escenario en vagrant en el cual constará de 3 máquinas, un cliente externo, el servidor vpn 




Entramos en el servidor vpn y haremos apt update, y después instalaremos openvpn que servirá tanto para el cliente como para el servidor.


Activamos el bit de forwarding en `/etc/sysctl.conf`  net.ipv4.ip_forward=1
Ejecutamos `sudo sysctl -p` para que lea el archivo sysctl.conf


`sudo cp -r /usr/share/easy-rsa /etc/openvpn`


`cd /etc/openvpn/easy-rsa`

Activamos el servicio de infraestructura pública:

`sudo ./easyrsa init-pki`

Tras esto generamos la entidad certificadora con su respectiva clave privada gracias al binario easyrsa.

`./easyrsa build-ca`

Ahora vamos a montar el servidor de certificaciones:

`./easyrsa build-server-full server nopass`

Generamos parámetros de Diffie-Helman, el objetivo del algoritmo de cifrado Diffie-Hellman es lograr el intercambio de una clave secreta por medio de un canal inseguro como es internet. Para que este algoritmo funcione, los datos informáticos deben traducirse a números, lo cual es posible por medio de sistemas como el código ASCII, combinado con otros.

`sudo ./easyrsa gen-dh`

Generamos el par de claves del cliente:
`./easyrsa build-client-full clientevpn nopass`

Ahora pasamos las claves al servidor cliente

```
sudo cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/clientevpn.crt,private/clientevpn.key} /home/vagrant/
cd /home/vagrant
sudo chown vagrant:vagrant {ca.crt,clientevpn.crt,clientevpn.key}
scp {ca.crt,clientevpn.crt,clientevpn.key} vagrant@vpn-cli:
```

Ahora configuraremos los parámetros del servidor y sus saltos:


`/etc/openvpn/server/servidor.conf`

``` 
port 1194
proto udp
dev tun


ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem


topology subnet


server 9.8.7.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt


push "route 192.168.0.0 255.255.255.0"


keepalive 10 120
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1

```


`sudo systemctl enable --now openvpn-server@servidor`


## Cliente

```
sudo apt update
sudo apt install openvpn
```

Movemos las claves al directorio de openvpn y le cambiamos el propietario:

```
sudo mv {ca.crt,clientevpn.crt,clientevpn.key} /etc/openvpn/client
sudo chown root: /etc/openvpn/client/*
```

Ahora vamos a crear el fichero /etc/openvpn/client/cliente.conf

```
client
dev tun
proto udp

remote 192.22.23.1 1194
resolv-retry infinite
nobind

persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/clientevpn.crt
key /etc/openvpn/client/clientevpn.key

remote-cert-tls server
cipher AES-256-CBC
verb 3
```

E iniciamos la el servicio de la configuración:

sudo systemctl enable --now openvpn-client@cliente


## Máquina interna


Quitamos la ip de la ruta por defecto que viene en vagrant:

sudo ip route del default


Añadimos la ruta que creamos hacia la vpn:

sudo ip route add default via 192.168.0.1



comprobaciones


![Descripción de la imagen](/images/vpn-acceso-remoto-ssh-openvpn.png)


![Descripción de la imagen](/images/traceroute-openvpn.png)



# Site-To-Site Openvpn


![Descripción de la imagen](/images/vpn-site-esquema.png)




## Servidor

Entramos en el servidor vpn y haremos apt update, y después instalaremos openvpn que servirá tanto para el cliente como para el servidor.


Activamos el bit de forwarding en `/etc/sysctl.conf`  net.ipv4.ip_forward=1
Ejecutamos `sudo sysctl -p` para que lea el archivo sysctl.conf


`sudo cp -r /usr/share/easy-rsa /etc/openvpn`


`cd /etc/openvpn/easy-rsa`

Activamos el servicio de infraestructura pública:

`sudo ./easyrsa init-pki`
Tras esto generamos la entidad certificadora con su respectiva clave privada gracias al binario easyrsa.

`./easyrsa build-ca`
Ahora vamos a montar el servidor de certificaciones:

`./easyrsa build-server-full server nopass`


`sudo ./easyrsa gen-dh`

Generamos el par de claves del cliente:
`./easyrsa build-client-full clientevpn nopass`

Ahora pasamos las claves al servidor cliente

```
sudo cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/clientevpn.crt,private/clientevpn.key} /home/vagrant/
cd /home/vagrant
sudo chown vagrant:vagrant {ca.crt,clientevpn.crt,clientevpn.key}
scp {ca.crt,clientevpn.crt,clientevpn.key} vagrant@vpn-cli:
```

A partir de aquí la configuración varía respecto a la parte de cliente-servidor:

``` 
dev tun
ifconfig 10.99.99.1 10.99.99.2
route 192.168.100.0 255.255.255.0
tls-server

dh /etc/openvpn/easy-rsa/pki/dh.pem
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key

comp-lzo
keepalive 10 60

verb 3
```


`sudo systemctl enable --now openvpn-server@servidor`


## Cliente

``` 
sudo apt update
sudo apt install openvpn
sudo mv {ca.crt,clientevpn.crt,clientevpn.key} /etc/openvpn/client
sudo chown root: /etc/openvpn/client/*
```

Ahora vamos a crear el fichero /etc/openvpn/client/cliente.conf

``` 
dev tun
remote 10.0.0.1
ifconfig 10.99.99.2 10.99.99.1
route 192.168.200.0 255.255.255.0
tls-client

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/clientevpn.crt
key /etc/openvpn/client/clientevpn.key

comp-lzo
keepalive 10 60

verb 3
```

sudo systemctl enable --now openvpn-client@cliente



## Cliente 1

sudo ip route del default
sudo ip route add default via 192.168.200.1


![Descripción de la imagen](/images/vpn-cliente2-traceroute.png)
![Descripción de la imagen](/images/vpn-cliente1.png)

## Cliente 2

sudo ip route del default
sudo ip route add default via 192.168.100.1


![Descripción de la imagen](/images/vpn-cliente2-traceroute.png)
![Descripción de la imagen](/images/vpn-cliente2.png)

