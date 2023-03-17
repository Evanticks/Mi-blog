---
title: Migración CentOS
Categoría: Administración de sistemas
---



# Migración CentOS
![](/images/centos-26.png)



## Analiza el desencadenante de la retirada de centOS 8 del mercado

La retirada de Centos8 del mercado por parte de Redhat fue debido a que cambiaría la estrategia de desarrollo de CentOS.

En su lugar, Red Hat presentó CentOS Stream, que es una distribución continua de código abierto basada en el mismo código fuente que RHEL y que se actualiza más rápidamente que CentOS. Red Hat declaró que CentOS Stream se centraría en proporcionar a los desarrolladores una vista previa temprana del próximo lanzamiento de RHEL y en ofrecer una plataforma para el desarrollo de aplicaciones basadas en RHEL.

CentOS 8 tenía un ciclo de vida útil previsto hasta 2029. 

Con la retirada, los usuarios ya no recibirán actualizaciones de seguridad ni parches de errores, lo que significa que tendrán que buscar alternativas para mantener sus sistemas seguros.


Mi opinión es que la retirada de centos 8 produce una desconfianza en la continuidad de los productosde redhat, aunque la nueva distribución de Rocky Linux puede ser una buena alternativa al cambio.




## RHEL9

Descargamos la iso de RHEL9 desde la página de redhat:

![image](/images/centos-15.png)

Luego procedemos a instalarlo en nuestra máquina virtual:

![image](/images/centos-14.png)

Una vez instalado, si ejecutamos subscription-manager register nos pedirá que introduzcamos el usuario y la contraseña de nuestra cuenta de redhat:

![image](/images/centos-16.png)

Ahora podemos ver que la máquina está asociada a la cuenta, entonces nos hemos dado de alta.

Si ahora realizamos un yum update pues tendremos a nuestra disposición los paquetes de RHEL9:

![image](/images/centos-17.png)

Red Hat Enterprise Linux 9 (RHEL9) es un sistema operativo de código abierto desarrollado y mantenido por Red Hat. Está diseñado para ser una plataforma estable y segura para entornos empresariales y de servidor.


## CentOS Stream


Primero descargamos la iso de CentOS Stream desde la página de centos:

![image](/images/centos-18.png)



Procedemos a iniciar la máquina virtual con la iso de CentOS Stream:

![image](/images/centos-19.png)


Instalamos CentOS Stream:

![image](/images/centos-20.png)


Y aquí podemos ver como está instalada:

![image](/images/centos-21.png)


Ahora vamos a hablar un poco de lo que es Centos Stream y lo que lo diferencia de RHEL9:
Centos Stream es una distribución que tiene apoyo de la comunidad, se actualiza constantemente entonces puede ser menos estable para un entorno de producción, pero es una buena opción para entornos de desarrollo y pruebas.

Aunque sea gratis, no cuenta con el soporte a largo plazo con la que cuenta RHEL.


## Descarga de iso

Alma Linux, soporte a largo plazo: Si necesita una distribución que ofrezca soporte a largo plazo, entonces AlmaLinux y VZLinux pueden ser buenas opciones. Ambas distribuciones ofrecen soporte hasta 2029, lo que significa que puede contar con actualizaciones de seguridad y parches de errores durante muchos años.

Rocky Linux, comunidad de usuarios: Si está buscando una distribución con una gran comunidad de usuarios, entonces Rocky Linux puede ser una buena opción. Esta distribución fue creada por la comunidad de usuarios de CentOS y tiene una gran cantidad de seguidores y desarrolladores.

eurolinux, enfoque en la estabilidad: Si necesita una distribución que se centre en la estabilidad y la confiabilidad, entonces euroLinux es una buena opción. Esta distribución se basa en CentOS, pero con enfoque en estabilidad y seguridad, lo que significa que puede ser una buena opción para entornos empresariales y de producción críticos.

VZlinux, adaptabilidad y escalabilidad: Si necesita una distribución que pueda adaptarse a diferentes entornos y escalar según sea necesario, entonces VZLinux es una buena opción. Esta distribución es conocida por su capacidad para ejecutarse en diferentes entornos de virtualización y contenedores, lo que la hace ideal para proyectos de infraestructura en la nube y para empresas que buscan una distribución escalable.

En nuestro caso vamos a elegir eurolinux ya que nos vamos a basar en la simulación de un entorno de producción estable y seguro.

Descargamos eurolinux desde la página oficial:

![image](/images/centos-22.png)

Ahora vamos a instalar la iso de eurolinux en nuestra máquina virtual:

![image](/images/centos-23.png)

Podemos apreciar que tiene el mismo instalador que centos, ya que se basa en la misma distribución:

![image](/images/centos-24.png)

![image](/images/centos-25.png)


Una vez instalado, podemos ver la distribución de eurolinux:

![image](/images/centos-27.png)

No la he configurado con interfaz gráfica ya que sería para entornos de producción y sería aumentar el consumo de recursos, eurolinux es una alternativa muy buena para un entorno de producción estable y seguro como podría serlo debian o ubuntu.



## Migración Centos - Rocky Linux


### Migración de Centos 7 a 8

usamos uname -r para ver si tenemos la última versión:

![image](/images/centos-1.png)


sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0

Ahora que tenemos conexión nos vamos a la terminal y nos conectamos a través de ssh:

```
yum install epel-release -y
```

![image](/images/centos-2.png)

```

rpmconf -a

sudo package-cleanup --leaves

sudo package-cleanup --orphans


sudo yum install -y dnf

dnf remove -y yum yum-metadata-parser

rm -Rf /etc/yum

dnf -y makecache

dnf -y upgrade
```

![image](/images/centos-3.png)


tras varios minutos, ejecutaremos el siguiente comando:

dnf install http://vault.centos.org/8.5.2111/BaseOS/x86_64/os/Packages/{centos-linux-repos-8-3.el8.noarch.rpm,centos-linux-release-8.5-1.2111.el8.noarch.rpm,centos-gpg-keys-8-3.el8.noarch.rpm}

![image](/images/centos-5.png)

dnf upgrade -y epel-release

![image](/images/centos-6.png)

```
cd /etc/yum.repos.d
sudo mkdir backups
sudo mv CentOS-* backups
```


removemos las dependencias sin utilizar:

```
rpm -e `rpm -q kernel` --nodeps
rpm -e `rpm -q kernel-devel` --nodeps
```

```
dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
```

instalamos kernel-core:

```
dnf -y install kernel-core
```

El kernel-core incluye los controladores y módulos del kernel para la administración de dispositivos de hardware, el sistema de archivos, la gestión de procesos y otros componentes esenciales del sistema operativo.



```
dnf -y groupupdate "Core" "Minimal Install" "Servidor con GUI"


systemctl set-default graphical.target
```

```
reboot
```


 ### Migración de Centos 8 a Rocky Linux

Ahora al iniciar sesión podremos ver que tenemos el escritorio de GNOME de centos 8:

![image](/images/centos-8.png)


Y ya tendríamos centos 8:

![image](/images/centos-9.png)


Ahora vamos a proceder a migrar de Centos 8 que es una solución sin soporte a Rocky Linux:

```
wget https://raw.githubusercontent.com/rocky-linux/rocky-tools/main/migrate2rocky/migrate2rocky.sh


chmod u+x migrate2rocky.sh
./migrate2rocky.sh -r
```

Con la opción -r especificamos que va a migrarse a rocky linux

Tras esperar unos minutos, nos pedirá que reiniciemos el sistema:

![image](/images/centos-10.png)

Aquí podemos ver el entorno gráfico

![image](/images/centos-11.png)

Y si ejecutamos el comando `cat /etc/redhat-release` nos dirá que tenemos Rocky Linux 8.

![image](/images/centos-13.png)

Con esto ya podemos dar por finalizada la migración de centos 7 a 8, y de 8 a rocky linux.