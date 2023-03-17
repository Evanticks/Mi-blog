---
title: Introducción a Kubernetes
categories: Contenedores
---


![Descripción de la imagen](/images/k8s-logo.png)



## Taller 1: Trabajando con Pods

Trabajar con pods nos permite crear contenedores que se ejecutan en un mismo nodo. Los pods son la unidad mínima de despliegue en Kubernetes.

1. Pantallazo del fichero yaml que has creado con la definición del Pod.

![Descripción de la imagen](/images/k8s-taller-1-1.png)

2. Pantallazo donde se comprueba que el Pod ha sido creado.

kubectl get pods

![Descripción de la imagen](/images/k8s-taller-1-2.png)

3. Pantallazo donde se ve la información detallada del Pod.

Podemos ver la información detallada del pod con `kubectl describe pod pod-nginx`

![Descripción de la imagen](/images/k8s-taller-1-3.png)

4. Pantallazo donde se ve el fichero index.html del DocumentRoot.

Podemos acceder al interior del pod con `kubectl exec -it pod-nginx -- /bin/bash`

![Descripción de la imagen](/images/k8s-taller-1-4.png)

5. Pantallazo del navegador accediendo a la aplicación con el port-forward.

Para poder acceder a través de un navegador, tenemos que hacer un port-forward al puerto 80 del pod.

`kubectl port-forward pod-nginx 8080:80`

![Descripción de la imagen](/images/k8s-taller-1-5.png)

6. Pantallazo donde se ve los logs de acceso del Pod.

`kubectl logs pod-nginx`

![Descripción de la imagen](/images/k8s-taller-1-6.png)

## Taller 2: Trabajando con ReplicaSets

Trabajar con ReplicaSets nos permite crear réplicas de un mismo pod.

iesgn/test_web:latest

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: iesgn-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image: iesgn/test_web:latest
          name: contenedor-nginx
```

![Descripción de la imagen](/images/k8s-taller-2-1.png)

Una vez creado el replicaset podemos crearlo con `kubectl apply -f iesgn.yaml`



También podemos crearlo con `kubectl create -f iesgn.yaml` pero en este caso no se puede modificar el replicaset una vez creado.



Para borrarlo emplearemos `kubectl delete -f iesgn.yaml`

![Descripción de la imagen](/images/k8s-taller-2-2.png)


Para ver los replicaset creados emplearemos  `kubectl get rs,pods`


```
╭─antonio@debian ~/Documentos/k8s/replicasets  
╰─➤  kubectl get rs,pods
NAME                        DESIRED   CURRENT   READY   AGE
replicaset.apps/iesgn-web   3         3         3       55s

NAME                      READY   STATUS    RESTARTS      AGE
pod/iesgn-web-g6s5g       1/1     Running   0             55s
pod/iesgn-web-sls5g       1/1     Running   0             55s
pod/iesgn-web-tnsff       1/1     Running   0             55s

```

Para ver la información detallada de un replicaset podemos emplear `kubectl describe rs iesgn-web`

![Descripción de la imagen](/images/k8s-taller-2-3.png)


POdemos ver la tolerancia a fallos borrando un pod con `kubectl delete pod iesgn-web-vjckt`


![Descripción de la imagen](/images/k8s-taller-2-4.png)


Kubernetes nos ofrece escalabilidad horizontal, es decir, podemos aumentar o disminuir el número de réplicas de un replicaset. Para ello emplearemos `kubectl scale --replicas=6 rs iesgn-web`

![Descripción de la imagen](/images/k8s-taller-2-5.png)

## Taller 3: Trabajando con Deployments

### Ejercicio 1: Trabajando con Deployments

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-iesgn
  labels:
    app: nginx
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate
  replicas: 2
  selector:
    matchLabels:
      app: iesgn
  template:
    metadata:
      labels:
        app: iesgn
    spec:
      containers:
      - image: iesgn/test_web:latest
        name: iesgn
        ports:
        - name: http
          containerPort: 80
```








Pantallazo del fichero yaml que has creado con la definición del Deployment.

![Descripción de la imagen](/images/k8s-taller-3-1.png)

Pantallazo donde se comprueba los recursos que se han creado.

`kubectl get deploy,rs,pod`

![Descripción de la imagen](/images/k8s-taller-3-2.png)

Pantallazo donde se ve la información detallada del Deployment.

`kubectl describe deployment/deployment-nginx`

![Descripción de la imagen](/images/k8s-taller-3-3.png)


Pantallazo donde se vea el acceso desde un navegador web a la aplicación usando el port-forward.

`kubectl port-forward deployment/deployment-iesgn 8080:80`


![Descripción de la imagen](/images/k8s-taller-3-4.png)

Pantallazo donde se vea los logs del despliegue después del acceso.

`kubectl logs deployment/deployment-iesgn`

![Descripción de la imagen](/images/k8s-taller-3-5.png)

### Ejercicio 2: Actualización y desactualización de nuestra aplicación

Pantallazo donde se vea el acceso desde un navegador web a la version 1 de la aplicación aplicación.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-iesgn-v1
  labels:
    app: contendor-iesgn
spec:
  replicas: 1
  selector:
    matchLabels:
      app: contenedor-iesgn
  template:
    metadata:
      labels:
        app: contenedor-iesgn
    spec:
      containers:
      - name: contenedor-iesgn
        image: iesgn/test_web:version1
        ports:
        - containerPort: 80
```

```
kubectl annotate deployment/deployment-iesgn-v1 kubernetes.io/change-cause="Primer despliegue de iesgn"
```

![Descripción de la imagen](/images/k8s-taller-3-2-1.png)


```
kubectl port-forward deployment/deployment-iesgn-v1 8081:80
```

Pantallazo donde se vea el acceso desde un navegador web a la version 2 de la aplicación aplicación.

```
kubectl apply -f deployment-iesgn-v1.yaml
```

![Descripción de la imagen](/images/k8s-taller-3-2-2.png)


Pantallazo donde se visualice el historial de actualización del despliegue después de actualizar a la versión 2.

![Descripción de la imagen](/images/k8s-taller-3-2-3.png)

Pantallazo donde se vea el acceso desde un navegador web a la version 3 de la aplicación (¡¡¡No se visualiza bien la 
hoja de estilos!!!).

![Descripción de la imagen](/images/k8s-taller-3-2-4.png)

Pantallazo donde se visualice el historial de actualización después de realizar el rollback del despliegue.

`kubectl rollout undo deployment/deployment-iesgn-v1`


![Descripción de la imagen](/images/k8s-taller-3-2-5.png)

Pantallazo donde se vea el acceso desde un navegador web a la version de la aplicación que queda después de hacer el rollout.

![Descripción de la imagen](/images/k8s-taller-3-2-6.png)






### Ejercicio 3: Despliegue de la aplicación GuestBook

guestbook-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: contenedor-guestbook
        image: iesgn/guestbook
        ports:
          - name: http-server
            containerPort: 5000

```

redis-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      containers:
        - name: contenedor-redis
          image: redis
          ports:
            - name: redis-server
              containerPort: 6379
```


Creamos el escenario ejecutando los siguientes comandos:

`kubectl apply -f guestbook-deployment.yaml`

`kubectl apply -f redis-deployment.yaml`

Tenemos dos deploys, uno con guestbook y otro con redis. Para acceder a la aplicación de guestbook, debemos hacer un port-forward al servicio de guestbook. Para ello, ejecutamos el siguiente comando:

`kubectl port-forward deployment/guestbook 8082:5000`


Podemos ver que los recursos están creados con el siguiente comando:

`kubectl get all`, `kubectl get all -o wide` o bien `kubectl get deploy,rs,pod`

![Descripción de la imagen](/images/k8s-taller-3-3-1.png)

Lo que ocurre es que no conecta a la base de datos porque los dos deployments no están relacionados entre sí.

![Descripción de la imagen](/images/k8s-taller-3-3-2.png)



## Taller 4: Trabajando con Services


### Ejercicio 1: Despliegue y acceso de la aplicación GuestBook

1. Pantallazo donde se vea el acceso desde un navegador web a la aplicación cuando sólo tenemos el servicio para acceder a la aplicación (tiene que aparecer el mensaje de error).

![Descripción de la imagen](/images/k8s-taller-4-1.png)

2. Pantallazo donde se vea el acceso desde un navegador web a la aplicación usando la ip del nodo master y el puerto asignado al Service.

![Descripción de la imagen](/images/k8s-taller-4-2.png)

3. Pantallazo donde se vea el acceso desde un navegador web a la aplicación usando el nombre que hemos configurado en el recurso Ingress.

Para crear el recurso ingress debemos tener el siguiente yaml:

nano ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: www.antonio.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: guestbook
            port:
              number: 80

```

![Descripción de la imagen](/images/k8s-taller-4-3.png)

### Ejercicio 2: Despliegue y acceso de la Aplicación Lets-Chat

1. Los ficheros yaml que has creado.

He creado varios yaml, voy a ordenarlos en sentido de conexión desde el frontend al backend:

ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: www.chat-antonio.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: letschat
            port:
              number: 8080

```

letschat-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: letschat
  labels:
    app: letschat
    tier: frontend
spec:
  type: NodePort
  ports: 
  - port: 8080
    targetPort: letschat-server
  selector:
    app: letschat
    tier: frontend
```

letschat-deployment.yaml


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: letschat
  labels:
    app: letschat
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: letschat
      tier: frontend
  template:
    metadata:
      labels:
        app: letschat
        tier: frontend
    spec:
      containers:
      - name: contenedor-letschat
        image: sdelements/lets-chat
        ports:
          - name: letschat-server
            containerPort: 8080
```

mongo-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app: mongo
    tier: backend
spec:
  type: ClusterIP
  ports: 
  - port: 27017
    targetPort: mongo-server
  selector:
    app: mongo
    tier: backend
```

mongo-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    app: mongo
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
      tier: backend
  template:
    metadata:
      labels:
        app: mongo
        tier: backend
    spec:
      containers:
        - name: contenedor-mongo
          image: mongo:4
          ports:
            - name: mongo-server
              containerPort: 27017
```


A través de este esquema podemos ver como los diferentes deploys y pods se conectan entre sí para ofrecer el servicio:


![Descripción de la imagen](/images/letschat-minikube-esquema.png)

2. Un pantallazo donde se vea el acceso desde un navegador web a la aplicación usando la ip del nodo master y el puerto asignado al Service.

Si usamos el comando `minikube service letschat` nos abre el navegador con la aplicación.

![Descripción de la imagen](/images/k8s-taller-4-4.png)

3. Un pantallazo donde se vea el acceso desde un navegador web a la aplicación usando el nombre que hemos configurado en el recurso Ingress.

![Descripción de la imagen](/images/k8s-taller-4-5.png)



## Taller 5: Trabajando con ConfigMaps

### Ejercicio 1: Configurando nuestra aplicación Temperaturas

Con el siguiente comando estaremos creando una variable de entorno en el cual indica el nombre SERVIDOR_TEMPERATURAS y el valor servidor-temperaturas:5000

```
kubectl create cm tempcm --from-literal=SERVIDOR_TEMPERATURAS=servidor-temperaturas:5000 -o yaml --dry-run=client > temperaturas_configmap.yaml
```

UNa vez ejecutado se crea un fichero que contiene en data el nombre y la clave de la variable de entorno:

temperaturas_configmap.yaml

```
apiVersion: v1
data:
  SERVIDOR_TEMPERATURAS: servidor-temperaturas:5000
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: tempcm

```

El nombre lo debemos especificar en el deployment, y la key el nombre que va a llevar la variable de entorno, en nuestro caso tempcm y SERVIDOR_TEMPERATURAS respectivamente:


Entonces en nuestro fichero yaml de nuestro deployment, quedaría de la siguiente manera:

tempers-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: temperaturas-frontend
  labels:
    app: temperaturas
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: temperaturas
      tier: frontend
  template:
    metadata:
      labels:
        app: temperaturas
        tier: frontend
    spec:
      containers:
      - name: contenedor-temperaturas
        image: iesgn/temperaturas_frontend
        ports:
          - name: http-server
            containerPort: 3000
        env:
          - name: TEMP_SERVER
            valueFrom:
              configMapKeyRef:
                name: tempcm
                key: SERVIDOR_TEMPERATURAS
```

Ahora vamos a crear el servicio para el frontend por el cual vamos a ver que se puede acceder a la aplicación a través de un puerto aleatorio:

temperaturas-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: temperaturas
  labels:
    app: temperaturas
    tier: frontend
spec:
  type: NodePort
  ports: 
  - port: 3000
    targetPort: http-server
  selector:
    app: temperaturas
    tier: frontend
```

Tomará el puerto 3000 y hará un proxy para que lo veamos a través de un puerto aleatorio, pero a través del port y el targetPort crearemos un ingress para que se pueda acceder a través de un dominio, que será www.antonio.org

ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: acceso-frontend
spec:
  rules:
  - host: www.antonio.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: temperaturas
            port:
              number: 3000

```

Dentro del confimap que hemos mencionado más arriba hace referencia al backend-srv.yaml, que toma el nombre de TEMP_SERVER y coge el valor de la key SERVIDOR_TEMPERATURAS, que es el la clave del valor servidor-temperaturas:5000, que se halla en el tempeaturas-deployment.yaml, tomado del configmap tempcm.

backend-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: servidor-temperaturas
  labels:
    app: temperaturas
    tier: backend
spec:
  type: ClusterIP
  ports:
  - port: 5000
    targetPort: api-server
  selector:
    app: temperaturas
    tier: backend

```

Pantallazo donde se vea la definición del recurso ConfigMap.

![Descripción de la imagen](/images/k8s-taller-5-1.png)

Pantallazo donde se vea la modificación del fichero frontend-deployment.yaml.

![Descripción de la imagen](/images/k8s-taller-5-2.png)

Pantallazo donde se vea la modificación del fichero backend-srv.yaml.

![Descripción de la imagen](/images/k8s-taller-5-3.png)

Pantallazo donde se compruebe que la aplicación está funcionando.

![Descripción de la imagen](/images/k8s-taller-5-4.png)

### Ejercicio 2: Despliegue y acceso de la aplicación Nextcloud


Primero vamos a generar las claves para que la base de datos mariadb pueda cambiar los valores por defecto del usuario:

En el configmap cambiaremos:

- nombre de usuario
- nombre de la base de datos
- nombre del host mariadb

En el secret que lo utilizaremos para convertir a base64 las contraseñas:

- contraseña de usuario



```
kubectl create cm bd-datos --from-literal=bd_user=antonio \
                           --from-literal=bd_dbname=nextcloud \
                           --from-literal=bd_host=mariadb-service \
                           -o yaml --dry-run=client > bd_datos_configmap.yaml



kubectl create secret generic bd-passwords --from-literal=bd_password=antonio \
                                           --from-literal=bd_rootpassword=root1234 \
                                           -o yaml --dry-run=client > bd_passwords_secret.yaml
``` 

Seguidamente, tanto el deploy de nextcloud como el de mariadb van a tomar cada uno de estos datos para poder crear la base de datos y el usuario, y nextcloud va a tomar el usuario y la contraseña para poder conectarse a la base de datos.

nextcloud-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud-deployment
  labels:
    app: nextcloud
    type: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
      type: frontend
  template:
    metadata:
      labels:
        app: nextcloud
        type: frontend
    spec:
      containers:
        - name: contenedor-nextcloud
          image: nextcloud
          ports:
            - containerPort: 80
              name: http-port
            - containerPort: 443
              name: https-port
          env:
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_user
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_dbname
            - name: MYSQL_HOST
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_host
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bd-passwords
                  key: bd_password
```


mariadb-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
  labels:
    app: nextcloud
    type: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
      type: database
  template:
    metadata:
      labels:
        app: nextcloud
        type: database
    spec:
      containers:
        - name: contenedor-mariadb
          image: mariadb:10.5
          ports:
            - containerPort: 3306
              name: db-port
          env:
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_user
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_dbname
            - name: MYSQL_HOST
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_host
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bd-passwords
                  key: bd_password
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bd-passwords
                  key: bd_rootpassword
```


Como podemos ver en el valor env tanto en nextcloud como en mariadb, se toman los valores de los configmaps y los secrets que hemos creado anteriormente, teniendo como '- name:' el valor que la aplicación toma por defecto.

maradb-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
  labels:
    app: nextcloud
    type: database
spec:
  selector:
    app: nextcloud
    type: database
  ports:
  - port: 3306
    targetPort: db-port
  type: ClusterIP
```

nextclod-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: nextcloud
  labels:
    app: nextcloud
    type: frontend
spec:
  selector:
    app: nextcloud
    type: frontend
  ports:
  - name: http-sv-port
    port: 80
    targetPort: http-port
  type: NodePort
```


Pantallazo donde se vea el contenido del fichero de despliegue de NextCloud.

![Descripción de la imagen](/images/k8s-taller-5-5.png)

Pantallazo donde se vean los recursos que se han creado.

![Descripción de la imagen](/images/k8s-taller-5-6.png)

Pantallazo donde se compruebe que la aplicación está funcionando.

![Descripción de la imagen](/images/k8s-taller-5-7.png)


## Taller 6: Almacenamiento en Kubernetes

### Ejercicio 1: Desplegando un servidor web persistente

Para empezar debemos pedir el recurso de almacenamiento que vamos a utilizar, en este caso vamos a utilizar un recurso de tipo persistent volume claim, que es un recurso que nos permite pedir un recurso de almacenamiento, en este caso un disco duro, y que nos lo asigne a un nodo.

cat apache-pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-webserver
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```

Como podemos ver, pide un recurso de almacenamiento de 2Gi

cat apache-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-taller6
  labels:
    app: apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      volumes:
        - name: pvc-webserver
          persistentVolumeClaim:
            claimName: pvc-webserver
      containers:
        - name: contenedor-apache
          image: php:7.4-apache
          ports:
            - name: http-server
              containerPort: 80
          volumeMounts:
            - mountPath: "/var/www/html"
              name: pvc-webserver
```

En el apartado spec nombra el volumen y lo monta en VolumeMounts.

Ahora vamos a crear el servicio nodeport por el cual vamos a entrar en la web:

cat apache-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: apache
spec:
  type: NodePort
  ports:
  - name: service-http
    port: 80
    targetPort: http-server
  selector:
    app: apache
```

Tras esto vamos a ejecutar un comando que es un info.php por bash y lo va a poner en /var/www/html que es donde está el recurso solicitado montado:

```
kubectl exec pod/apache-taller6-5f78fbb47d-7rlzd -- bash -c "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"
```

Pantallazo con la definición del recurso PersistentVolumenClaim.

![Descripción de la imagen](/images/k8s-taller-6-1.png)

Pantallazo donde se visualice los recursos pv y pvc que se han creado.

![Descripción de la imagen](/images/k8s-taller-6-2.png)

Pantallazo donde se vea el fichero yaml para el despliegue.

![Descripción de la imagen](/images/k8s-taller-6-3.png)

Pantallazo donde se vea el acceso a info.php.

![Descripción de la imagen](/images/k8s-taller-6-4.png)

Pantallazo donde se vea que se ha eliminado y se ha vuelto a crear el despliegue y se sigue sirviendo el fichero info.php.

Adjunto en una sola imagen la destrucción,creación y comprobación de que sigue funcionando a través de un curl:

![Descripción de la imagen](/images/k8s-taller-6-5.png)

### Ejercicio 2: Haciendo persistente la aplicación GuestBook

Vamos a modificar el recurso de despliegue de guestbook para que se pueda hacer persistente, a través de hacer una copia d ela base de datos redis en /data:

redis-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      volumes:
        - name: volumen-redis
          persistentVolumeClaim:
            claimName: pvc-guestbook
      containers:
        - name: contenedor-redis
          image: redis
          command: ["redis-server"]
          args: ["--appendonly", "yes"]
          ports:
            - name: redis-server
              containerPort: 6379
          volumeMounts:
            - mountPath: /data
              name: volumen-redis
```

Y este sería el guestbook-pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-guestbook
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```


Pantallazo con la definición del recurso PersistentVolumenClaim.

Mostrado arriba.

Pantallazo donde se visualicen los recursos pv y pvc que se han creado.

![Descripción de la imagen](/images/k8s-taller-6-6.png)

Pantallazo donde se vea el fichero yaml modificado para el despliegue de redis.

Mostrado arriba.

Pantallazo donde se vea el acceso a la aplicación con los mensajes escritos.

![Descripción de la imagen](/images/k8s-taller-6-7.png)

Pantallazo donde se vea que se ha eliminado y se ha vuelto a crear el despliegue de redis y que se sigue sirviendo la aplicación con los mensajes.

![Descripción de la imagen](/images/k8s-taller-6-8.gif)

### Ejercicio 3: Haciendo persistente la aplicación Nextcloud

Vamos a utilizar los ficheros que anteriormente creamos en nextcloud, lo que haremos será proporcionar almacenamiento tanto en nextcloud como mariadb, para ello vamos a modificar los ficheros de despliegue en relación a los pvc:

nextcloud-pvc.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: nextcloud-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

nextcloud-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud-deployment
  labels:
    app: nextcloud
    type: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
      type: frontend
  template:
    metadata:
      labels:
        app: nextcloud
        type: frontend
    spec:
      volumes:
        - name: nextcloud-pvc
          persistentVolumeClaim:
            claimName: nextcloud-pvc
      containers:
        - name: contenedor-nextcloud
          image: nextcloud
          volumeMounts:
            - mountPath: /var/www/html/
              name: nextcloud-pvc
          ports:
            - containerPort: 80
              name: http-port
            - containerPort: 443
              name: https-port
          env:
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_user
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_dbname
            - name: MYSQL_HOST
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_host
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bd-passwords
                  key: bd_password
```

Ahora vamos con mariadb:

mariadb-pvc.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
  labels:
    app: nextcloud
    type: database
spec:
  selector:
    app: nextcloud
    type: database
  ports:
  - port: 3306
    targetPort: db-port
  type: ClusterIP
```


cat mariadb-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
  labels:
    app: nextcloud
    type: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
      type: database
  template:
    metadata:
      labels:
        app: nextcloud
        type: database
    spec:
      volumes:
        - name: mariadb-pvc
          persistentVolumeClaim:
            claimName: mariadb-pvc
      containers:
        - name: contenedor-mariadb
          image: mariadb:10.5
          ports:
            - containerPort: 3306
              name: db-port
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              name: mariadb-pvc
          env:
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_user
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_dbname
            - name: MYSQL_HOST
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_host
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bd-passwords
                  key: bd_password
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: bd-passwords
                  key: bd_rootpassword

```


Pantallazo donde se vean los ficheros yaml modificados para los despliegues.

Se muestra arriba.

Pantallazo donde se vea el acceso a la aplicación con el fichero que has subido.

![Descripción de la imagen](/images/k8s-taller-6-8.png)

Pantallazo donde se vea que se han eliminado y se han vuelto a crear los despliegues y que la aplicación sigue sirviendo el fichero que habíamos subido.

![Descripción de la imagen](/images/k8s-taller-6-9.gif)


