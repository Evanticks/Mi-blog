- La máquina Alfa tiene un servidor ssh escuchando por el puerto 22, pero al acceder desde el exterior habrá que conectar al puerto 2222.


```

iptables -A PREROUTING -t nat -i ens4 -p tcp --dport 22 -j REDIRECT --to-port 2222
```


- Desde Delta y Bravo se debe permitir la conexión ssh por el puerto 22 a la máquina Alfa.
- La máquina Alfa debe tener permitido el tráfico para la interfaz loopback.

```
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```
- A la máquina Alfa se le puede hacer ping desde la DMZ, pero desde la LAN se le debe rechazar la conexión (REJECT) y desde el exterior se rechazará de manera silenciosa.
- La máquina Alfa puede hacer ping a la LAN, la DMZ y al exterior.
- Desde la máquina Bravo se puede hacer ping y conexión ssh a las máquinas de la LAN.
- Desde cualquier máquina de la LAN se puede conectar por ssh a la máquina Bravo.
- Configura la máquina Alfa para que las máquinas de LAN y DMZ puedan acceder al exterior.
- Las máquinas de la LAN pueden hacer ping al exterior y navegar.
- La máquina Bravo puede navegar. Instala un servidor web, un servidor ftp y un servidor de correos si no los tienes aún.
- Configura la máquina Alfa para que los servicios web y ftp sean accesibles desde el exterior.
- El servidor web y el servidor ftp deben ser accesibles desde la LAN y desde el exterior.
- El servidor de correos sólo debe ser accesible desde la LAN.
- En la máquina Charlie instala un servidor mysql si no lo tiene aún. A este servidor se puede acceder desde la DMZ, pero no desde el exterior.
- Evita ataques DoS por ICMP Flood, limitando el número de peticiones por segundo desde una misma IP.
- Evita ataques DoS por SYN Flood.
- Evita que realicen escaneos de puertos a Alfa.