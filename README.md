This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 1**

```
minikube start --driver=docker
```

**Exercise 2**

Create a file called sql-values.yaml with the following content:

```
architecture: replication
auth:
  rootPassword: secret-root-pass
  database: my-app-db
  username: my-user
  password: my-pass

secondary:
  replicaCount: 2
  persistence:
    storageClass: standard
```

And then run:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-mysql bitnami/mysql -f sql-values.yaml
```

**Exercise 3**

Create the docker-registry secret:

```
kubectl create secret docker-registry docker-credentials \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD \
--docker-email=$DOCKER_EMAIL
```

And the DB secret:

```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_USER: bXktdXNlcg== 
  DB_PWD: bXktcGFzcw==
  DB_NAME: bXktYXBwLWRi
  DB_ROOT_PWD: c2VjcmV0LXJvb3QtcGFzcw==
```

And the ConfigMap:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  db_server: my-mysql-primary.default
```

Create the app java-app.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
  labels:
    app: java-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      imagePullSecrets:
      - name: docker-credentials
      containers:
      - name: service
        image: nanatwn/demo-app:mysql-app
        ports:
        - containerPort: 8080
        env:
         - name: DB_USER
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: DB_USER
         - name: DB_PWD
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: DB_PWD
         - name: DB_NAME
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: DB_NAME
         - name: DB_SERVER
           valueFrom:
             configMapKeyRef:
              name: db-config
              key: DB_SERVER
---
apiVersion: v1
kind: Service
metadata:
  name: java-app
spec:
  selector:
    app: java-app
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```

And run:

```
kubectl apply -f db-secret.yaml
kubectl apply -f db-config.yaml
kubectl apply -f java-app.yaml
```

**Exercise 4**

```
```

**Exercise 5**

```
```

**Exercise 6**

```
```

**Exercise 7**

```
```

**Exercise 8**

```
```
