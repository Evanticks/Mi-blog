---
title: Práctica Kubernetes
Categoría: Contenedores
---


![kubernetes](/images/k8s-logo-2.png)


## Archivos de configuración de Kubernetes

```
kubectl create cm bd-datos --from-literal=bd_user=bookmedik \
                           --from-literal=bd_dbname=bookmedik \
                           --from-literal=bd_host=mariadb-service \
                           -o yaml --dry-run=client > bd_datos_configmap.yaml



kubectl create secret generic bd-passwords --from-literal=bd_password=bookmedik \
                                           --from-literal=bd_rootpassword=root \
                                           -o yaml --dry-run=client > bd_passwords_secret.yaml
```


bookmedik-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookmedik-deployment
  labels:
    app: bookmedik
    type: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookmedik
      type: frontend
  template:
    metadata:
      labels:
        app: bookmedik
        type: frontend
    spec:
      volumes:
        - name: bookmedik-pvc
          persistentVolumeClaim:
            claimName: bookmedik-pvc
      containers:
        - name: contenedor-bookmedik
          image: evanticks/bookmedik:v1_3
#          volumeMounts:
#            - mountPath: /var/www/html/
#              name: bookmedik-pvc
          ports:
            - containerPort: 80
              name: http-port
          env:
            - name: USUARIO_BOOKMEDIK
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_user
            - name: NOMBRE_DB
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_dbname
            - name: DATABASE_HOST
              valueFrom:
                configMapKeyRef:
                  name: bd-datos
                  key: bd_host
            - name: CONTRA_BOOKMEDIK
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
    app: bookmedik
    type: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookmedik
      type: database
  template:
    metadata:
      labels:
        app: bookmedik
        type: database
    spec:
      volumes:
        - name: mariadb-pvc
          persistentVolumeClaim:
            claimName: mariadb-pvc
      containers:
        - name: contenedor-mariadb
          image: mariadb
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
#            - name: MARIADB_HOST
#              valueFrom:
#                configMapKeyRef:
#                  name: bd-datos
#                  key: bd_host
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


bookmedik-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: bookmedik
  labels:
    app: bookmedik
    type: frontend
spec:
  selector:
    app: bookmedik
    type: frontend
  ports:
  - name: http-sv-port
    port: 80
    targetPort: http-port
  type: NodePort
```

maria-srv.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
  labels:
    app: bookmedik
    type: database
spec:
  selector:
    app: bookmedik
    type: database
  ports:
  - port: 3306
    targetPort: db-port
  type: ClusterIP
```

bookmedik-pvc.yaml 

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: bookmedik-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

mariadb-pvc.yaml  

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mariadb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

cat ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookmedik
spec:
  rules:
  - host: www.antonio.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bookmedik
            port:
              number: 80
```

## Entrega de la práctica

- Salida de los comando que nos posibilitan ver los recursos que has creado en el cluster.

```
kubectl get all,pv,pvc,cm,secret

NAME                                        READY   STATUS    RESTARTS   AGE
pod/bookmedik-deployment-57988bfdd9-6zljr   1/1     Running   0          9m12s
pod/mariadb-deployment-7cd7fb9cd8-2tsts     1/1     Running   0          36m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/bookmedik         NodePort    10.100.41.241   <none>        80:30814/TCP   63m
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        3h21m
service/mariadb-service   ClusterIP   10.103.3.28     <none>        3306/TCP       63m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/bookmedik-deployment   1/1     1            1           55m
deployment.apps/mariadb-deployment     1/1     1            1           36m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/bookmedik-deployment-57988bfdd9   1         1         1       9m12s
replicaset.apps/bookmedik-deployment-85bff5fdfd   0         0         0       55m
replicaset.apps/mariadb-deployment-7cd7fb9cd8     1         1         1       36m

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/pvc-0f3db0b9-81d5-4574-abeb-2da92aa82a44   4Gi        RWO            Delete           Bound    default/bookmedik-pvc   standard                63m
persistentvolume/pvc-fe594844-a9e4-4390-986a-7950611dee65   4Gi        RWO            Delete           Bound    default/mariadb-pvc     standard                63m

NAME                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/bookmedik-pvc   Bound    pvc-0f3db0b9-81d5-4574-abeb-2da92aa82a44   4Gi        RWO            standard       63m
persistentvolumeclaim/mariadb-pvc     Bound    pvc-fe594844-a9e4-4390-986a-7950611dee65   4Gi        RWO            standard       63m

NAME                         DATA   AGE
configmap/bd-datos           3      63m
configmap/kube-root-ca.crt   1      3h21m

NAME                  TYPE     DATA   AGE
secret/bd-passwords   Opaque   2      63m

```

- Pantallazo accediendo a la aplicación utilizando el servicio.

```
minikube get service bookmedik
```

![bookmedik-0](/images/bookmedik-7.png)

- Pantallazo accediendo a la aplicación utilizando el ingress.

![bookmedik-1](/images/bookmedik-1.png)

- Elimina el despliegue de la base datos, vuelve a crearla y comprueba que la aplicación no ha perdido los datos.

![bookmedik-2](/images/bookmedik-2.gif)



- Escala la aplicación con 3 replicas. Muestra la salida oportuna para ver los pods que se han creado.

```
kubectl get pods

kubectl scale deployment bookmedik-deployment --replicas=3
```

![bookmedik-3](/images/bookmedik-2.png)


- Modifica la aplicación, vuelve a crear una imagen con la nueva versión y actualiza el despliegue. No te olvide de anotar la modificación. Muestra la salida del historial de despliegue, la salida de kubectl get all y un pantallazo donde se vea la modificación que has realizado.

Nos vamos a bookmedik/core/app/view/pacients-view.php y cambiamos el texto Nombre completo por Nombre y Apellidos.


![bookmedik-4](/images/bookmedik-3.png)

```
docker build -t evanticks/bookmedik:v1_3 .
docker push evanticks/bookmedik:v1_3
```

Ahora editamos la el bookmedik-deployment.yaml y cambiamos la imagen por la nueva.

![bookmedik-5](/images/bookmedik-4.png)

```
kubectl apply -f bookmedik-deployment.yaml
```

```
kubectl annotate deployments.apps/bookmedik-deployment kubernetes.io/change-cause="He cambiado el título del nombre del paciente"
```

![bookmedik-6](/images/bookmedik-5.png)

Y aquí podemos ver como ha cambiado la aplicación:

![bookmedik-7](/images/bookmedik-6.png)

Y aquí la salida de kubectl get all:

```
NAME                                        READY   STATUS    RESTARTS   AGE
pod/bookmedik-deployment-57988bfdd9-6zljr   1/1     Running   0          8m24s
pod/mariadb-deployment-7cd7fb9cd8-2tsts     1/1     Running   0          35m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/bookmedik         NodePort    10.100.41.241   <none>        80:30814/TCP   62m
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        3h20m
service/mariadb-service   ClusterIP   10.103.3.28     <none>        3306/TCP       62m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/bookmedik-deployment   1/1     1            1           54m
deployment.apps/mariadb-deployment     1/1     1            1           35m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/bookmedik-deployment-57988bfdd9   1         1         1       8m24s
replicaset.apps/bookmedik-deployment-85bff5fdfd   0         0         0       54m
replicaset.apps/mariadb-deployment-7cd7fb9cd8     1         1         1       35m

```

- Entrega la url del repositorio donde están los ficheros yaml.

https://github.com/Evanticks/kubernetes-practica


## K3S

Tendremos 4 nodos, un maestro y 3 esclavos. El maestro tendrá el rol de controlador y los esclavos de nodos de trabajo, lo haremos a través del siguiente vagrantfile:

```
Vagrant.configure("2") do |config|
    config.vm.box = "debian/bullseye64"
    config.vm.box_check_update = false
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.provider "libvirt" do |v|
      v.memory = 1024
      v.cpus = 1
    end
    config.vm.define "master" do |master|
      master.vm.hostname = "master"
      master.vm.network "private_network",
        :libvirt__network_name => "k3s-vagrant",
        :ip => "10.10.10.10",
        :libvirt__dhcp_enabled => false,
        :libvirt__forward_mode => "veryisolated"
    end
    config.vm.define "nodo1" do |nodo1|
      nodo1.vm.hostname = "nodo1"
      nodo1.vm.network "private_network",
        :libvirt__network_name => "k3s-vagrant",
        :ip => "10.10.10.20",
        :libvirt__dhcp_enabled => false,
        :libvirt__forward_mode => "veryisolated"
    end
    config.vm.define "nodo2" do |nodo2|
      nodo2.vm.hostname = "nodo2"
      nodo2.vm.network "private_network",
        :libvirt__network_name => "k3s-vagrant",
        :ip => "10.10.10.30",
        :libvirt__dhcp_enabled => false,
        :libvirt__forward_mode => "veryisolated"
    end
    config.vm.define "nodo2" do |nodo2|
        nodo2.vm.hostname = "nodo2"
        nodo2.vm.network "private_network",
          :libvirt__network_name => "k3s-vagrant",
          :ip => "10.10.10.40",
          :libvirt__dhcp_enabled => false,
          :libvirt__forward_mode => "veryisolated"
    end
  end
```

Tras esto entramos en el nodo maestro y ejecutamos el siguiente comando como administrador:

```
apt update

apt install curl -y

curl -sfL https://get.k3s.io | sh -
```

Tras esto ejecutamso un `cat /var/lib/rancher/k3s/server/node-token` para que ese token lo podamos usar en los nodos esclavos.

En los nodos esclavos utilizamos el siguiente comando:

```
apt update

apt install curl -y

curl -sfL https://get.k3s.io | K3S_URL=https://10.10.10.10:6443 K3S_TOKEN=K10cd69b71c10ac263c200b4bfb2747bff1d5563fdce2076454533940bc0be0b848::server:fdf97632c55a66fff13d217e2a5ca407 sh -
```

Tras esto, si lo hemos hecho en los 3 nodos esclavos, veremos en el nodo maestro que tenemos 4 nodos:

```
kubectl get nodes
```

![k3s-1](/images/bookmedik-8.png)

Ahora vamos a clonar el repositorio con nuestros yaml:

```
git clone https://github.com/Evanticks/kubernetes-practica.git
```

editamos el ingress.yaml para adaptarlo a nuestra configuración:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookmedik
spec:
  rules:
  - host: www.bookantonio.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bookmedik
            port:
              number: 80
```

Ahora ejecutamos un `kubectl apply -f .` para que se creen los recursos.

Tan solo faltaría ingresar en /etc/hosts la ip de la máquina maestra y luego www.bookantonio.org.

![k3s-2](/images/bookmedik-9.png)