---
title: "DRBD"
categories: almacenamiento
---

![imagen](/images/drbd-logo.png)



# 1. La salida del comando drbdadm status wwwdata.

En el nodo 1 instalamos:

`apt install drbd-utils`

Ahora vamos a crear el recurso en /etc/drbd.d/wwwdata.res

```
resource wwwdata {
 protocol C;
 meta-disk internal;
 device /dev/drbd1;
 syncer {
  verify-alg sha1;
 }
 net {
  allow-two-primaries;
 }
 on nodo1 {
    disk /dev/vdb;
    address 10.1.1.101:7789;
 }
 on nodo2 {
    disk /dev/vdb;
    address 10.1.1.102:7789;
 }

```

Debemos crear el mismo recurso de manera simétrica en el nodo2 para que funcionen correctamente.


```
drbdadm create-md wwwdata
drbdadm up wwwdata
drbdadm primary --force wwwdata
apt install xfsprogs
mkfs.xfs /dev/drbd1
mount /dev/drbd1 /mnt
echo "<h1>Hola soy Antonio</h1>" > /mnt/index.html
```



`drbdadm status wwwdata`

```
wwwdata role:Primary
  disk:UpToDate
  peer role:Secondary
    replication:Established peer-disk:UpToDate
```

# 2. Prueba de funcionamiento del modo Single-primary.


Acabo de desmontar /mnt del nodo1 que funcionaba con normalidad, habiendo creado un fichero index.html.

A continuación vamos a hacerlo al reverso, siendo el nodo 2 el nodo primario y el nodo 1 secundrio, para eso ejecutamos en el nodo 1:

dbdadm secundary wwwdata

Y en el nodo 2:

dbdadm primary --force wwwdata

acto seguido comprobamos que funciona:

![imagen](/images/drbd-1.png)

Y si montamos el recurso podemos leer el fichero:

```
root@nodo2:/home/vagrant# mount /dev/drbd1 /mnt
root@nodo2:/home/vagrant# cat /mnt/index.html
<h1>Hola soy Antonio</h1>
```


## OCFS2

OCFS2 es un sistema de ficheros el cual nos permitirá leer y escribir de manera simultánea en un mismo disco. Para ello, se utiliza un sistema de ficheros distribuido, que permite que varios nodos puedan acceder al mismo disco, y que los cambios que se realicen en un nodo se reflejen en el resto de nodos.

para instalar OCFS2 en ambos nodos ejecutamos:

`apt install ocfs2-tools`

Ahora vamos a crear el recurso en /etc/drbd.d/dbdata.res en ambos nodos:

```
resource dbdata {
  protocol C;
  meta-disk internal;
  device /dev/drbd2;
  syncer {
    verify-alg sha1;
  }
  net {
    allow-two-primaries;
  }
  on nodo1 {
    disk /dev/vdc;
    address 10.1.1.101:7790;
  }
  on nodo2 {
    disk /dev/vdc;
    address 10.1.1.102:7790;
  }
}
```

```
drbdadm create-md dbdata 

drbdadm up dbdata

drbdadm primary --force dbdata
```

Ahora en el nodo1:

`o2cb add-cluster micluster`

seguidamente ingresamos estos parámetros:

```
o2cb add-node micluster nodo1 --ip 10.1.1.101
o2cb add-node micluster nodo2 --ip 10.1.1.102
```

ejecutando `o2cb list-cluster micluster` podemos ver que se ha añadido correctamente.
copiamos el contenido de /etc/ocfs2/cluster.conf en el nodo2 en el mismo directorio.
```
node:
	number = 0
	name = nodo1
	ip_address = 10.1.1.101
	ip_port = 7777
	cluster = micluster

node:
	number = 1
	name = nodo2
	ip_address = 10.1.1.102
	ip_port = 7777
	cluster = micluster

cluster:
	node_count = 2
	heartbeat_mode = local
	name = micluster
```

En el nodo 2 vamos a /etc/default/o2cb y establecemos los siguientes parámetros:

```
# O2CB_ENABLED: 'true' means to load the driver on boot.
O2CB_ENABLED=true

# O2CB_BOOTCLUSTER: If not empty, the name of a cluster to start.
O2CB_BOOTCLUSTER=micluster  # Cambiamos el valor de este parámetro por el nombre de nuestro cluster.
```

ingresamos en /etc/sysctl.conf  y añadimos la siguiente línea:

```
kernel.panic = 30
kernel.panic_on_oops = 1
```

sysctl -p

ejecutamos lo siguiente en ambos nodos:

```
o2cb register-cluster micluster
```

En nodo1 ejecutamos:

```
mkfs.ocfs2 --cluster-stack=o2cb --cluster-name=micluster /dev/drbd2
mount /dev/drbd2 /mnt
```

si ejecutamos el siguiente comando en nodo1:

```
for ((;;)) do date >> /mnt/fecha_nodo1.txt;sleep 1; done
```

Podemos ver en nodo2 con:

```
watch tail fecha_nodo1.txt
```


# 3. La salida del comando drbdadm status dbdata.

```
root@nodo1:~# drbdadm status dbdata
dbdata role:Primary
  disk:UpToDate
  peer role:Primary
    replication:Established peer-disk:UpToDate
```



# 4. Prueba de funcionamiento del modo Dual-primary.

![imagen](/images/peek.gif)

# 5. Muestra al profesor el funcionamiento del modo Dual-primary.


Al reiniciar los dos nodos deberemos de ejecutar el siguiente comando para que sincronice los datos:

`drbdadm up dbdata`
`drbdadm primary --force dbdata`

Si no es capaz de unirse al grupo de cluster:
`o2cb register-cluster micluster`