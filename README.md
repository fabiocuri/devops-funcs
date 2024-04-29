This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi Paixao 

E-mail: fcuri91@gmail.com

**Exercise 1**

```
eksctl create cluster --name=eks-tutorial --nodes=2 --kubeconfig=./kube/config
eksctl create fargateprofile --cluster eks-tutorial --name eks-fargate-tutorial --namespace my-app
export KUBECONFIG=./kube/config
```

**Exercise 2**

Create sql-values.yaml as below:

```
architecture: replication
auth:
  rootPassword: secret-root-pass
  database: my-app-db
  username: my-user
  password: my-pass
volumePermissions:
  enabled: true
primary:
  persistence:
    enabled: false
secondary:
  replicaCount: 2
  persistence:
    enabled: false
```

And run:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-mysql bitnami/mysql -f sql-values.yaml
```

And create the following YAMLs for phpmyadmin:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  db_server: my-mysql-primary.default
```

```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  db_user: bXktdXNlcg== 
  db_pwd: bXktcGFzcw==
  db_name: bXktYXBwLWRi
  db_root_pwd: c2VjcmV0LXJvb3QtcGFzcw==
```

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
                  key: db_server
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: db_root_pwd
---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
spec:
  selector:
    app: phpmyadmin
  ports:
  - protocol: TCP
    port: 8081
    targetPort: 80
```

And finally run:

```
kubectl apply -f db-config.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f phpmyadmin.yaml
kubectl port-forward svc/phpmyadmin-service 8081:8081
```

**Exercise 3**

```

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
