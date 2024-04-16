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
  DB_SERVER: my-mysql-primary.default
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

Create php-myadmin.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin
  labels:
    app: phpmyadmin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
        - name: phpmyadmin
          image: phpmyadmin/phpmyadmin:5
          ports:
            - containerPort: 80
              protocol: TCP
          env:
            - name: PMA_HOST
              valueFrom:
                configMapKeyRef:
                  name: db-config
                  key: DB_SERVER
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: DB_ROOT_PWD
---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin
spec:
  selector:
    app: phpmyadmin
  ports:
  - protocol: TCP
    port: 8081
    targetPort: 80
```

and run

```
kubectl apply -f php-myadmin.yaml
```

**Exercise 5**

```
minikube addons enable ingress
```

**Exercise 6**

Create java-app-ingress.yaml:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: java-app-ingress
spec:
  rules:
  - host: my-java-app.com
    http:
      paths:
      - backend:
          service:
            name: java-app
            port: 
              number: 8080
        pathType: Prefix
        path: /
  ingressClassName: nginx
```

and run

```
kubectl apply -f java-app-ingress.yaml
minikube tunnel
```

**Exercise 7**

```
kubectl port-forward svc/phpmyadmin 8081:8081
```

**Exercise 8**

```
helm create java-app
```

Create the manual YAML files with values-override.yaml and run:

```
helm install my-cool-java-app java-app -f java-app/values-override.yaml --dry-run --debug
```

