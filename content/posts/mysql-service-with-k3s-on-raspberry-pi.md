---
title: "Mysql Service With K3s on Raspberry Pi"
date: 2020-06-24T11:26:40+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Kubernetes
  - Raspberry Pi
  - Docker
  - MySQL
---

## Dependences

* Installed [K3s](https://k3s.io/) on a Raspberry Pi[^1]
* Installed [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) on your local computer.
* A MySQL client. (for testing)

## Install MySQL on Kubernetes

We will create configures and apply them to create a MySQL service. All configure files below must be applied after created with command:

```shell
kubectl apply -f <config-name>.yaml
```

All commands below are omitted.

### Prepare MySQL Data Storage

Let us create a volume for our MySQL service to store data.[^2]

At first, we need create a directory on node with command:

```shell
mkdir /mnt/data
```

Then, create a configure file for persistent volume.

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-data-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"  # the directory created on node
```

After created, we can claim it on Kubernetes. Here is the config file:

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi  # the storage should be less than capacity in persistent volume
```

### Adding A Secret with MySQL Password

The passsword of mysql we will use later should be injected into the pod environment by using the Secret.

_Note that the value must be encoded using base64[^3]_.

Says that the password is "mypassword".

```shell
echo -n 'mypassword' | base64
```

The output `bXlwYXNzd29yZA==` should be put into our secret configure file.

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: bXlwYXNzd29yZA==  # encoded base64 password
```

### Creating Deployment with a MySQL Container

* Specifying image to create a container in pod.
* Using PersistentVolumeClaim as volume.
* Using Secret as environment variables.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      volumes:
        - name: mysql-data-storage
          persistentVolumeClaim:
            claimName: mysql-data-pv-claim
      containers:
        - name: mysql-container
          image: jsurf/rpi-mariadb
          ports:
            - containerPort: 3306
              name: "mysql-port"
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: password
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              name: mysql-data-storage
```

### Expose the Service

After the deployment applied and the pod is running. The MySQL service is not exposed to the node. We need to create a Service to listen the port on node.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      nodePort: 30006
```

## Have a Try

Now, you can connet to your new database with a MySQL client.

```shell
mysql -P 30006 -h <raspberry-pi-host> -u root -p
```

All done!

[^1]: An article can help for your [installation](https://opensource.com/article/20/3/kubernetes-raspberry-pi-k3s).
[^2]: You can reference [here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod) to configure persistent volume storage.
[^3]: Follow the [example](https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually) in the official document.

