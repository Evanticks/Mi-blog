---
title: Trabajando con los charts de Helm
Categoria: Orquestación
---


![helm](/images/logo-helm.png)

## Instalación de un CMS con Helm


Ahora vamos a trabajar con helm, para ello debemos descargarlo de su página oficial:

```
wget https://get.helm.sh/helm-v3.11.0-linux-amd64.tar.gz
```

Descomprimimos el archivo y lo movemos a la carpeta /usr/local/bin

```
tar zxvf helm-v3.11.0-linux-amd64.tar.gz
sudo install helm /usr/local/bin/
```

Ejecutamos el siguiente comando para que helm pueda acceder a los repositorios de charts:

```
helm version
```

Ahora vamos a añadir el siguiente repositorio en helm:

```
helm repo add "stable" "https://charts.helm.sh/stable" --force-update
```

Con el siguiente comando podemos ver los repositorios que tenemos añadidos:

```
helm repo list
```

![helm](/images/helm-1.png)

Ahora vamos a instalar un CMS con helm, para ello vamos a descargar el repositorio de bitnami:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

```
helm repo update
helm repo list
```

## Pantallazo con la búsqueda del chart con el comando helm

Vamos a buscar dentro de los repositorios el CMS que queremos instalar, que en nuestro caso es wordpress:

```
helm search repo wordpress
```

![helm](/images/helm-2.png)

Ahora vamos a instalar el CMS con helm, especificaremos el tipo de servicio que queremos que sea NodePort para poder acceder al recurso:

```
helm install wordpress bitnami/wordpress --set service.type=NodePort
```

## Pantallazo donde se compruebe que se ha desplegado de forma correcta.

Si seguimos las instrucciones que nos da helm, podremos acceder al CMS desde el navegador:



![helm](/images/helm-3.png)


Con las variables y echo podemos obtener la IP del nodo y el puerto que nos ha asignado el servicio:

![helm](/images/helm-4.png)



## Pantallazo donde se vean los Pods, ReplicaSets, Deployments y Services que se han creado.



![helm](/images/helm-6.png)



Aquí vemos como está funcionando el wordpress:

![helm](/images/helm-5.png)


Por último, vamos a eliminar el CMS que hemos instalado con helm:

```
helm delete wordpress
```


## Pantallazo donde se vea el acceso al blog y se vea tu nombre como título del blog.

![helm](/images/helm-7.png)