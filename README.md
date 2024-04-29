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
kubectl create namespace my-app

kubectl create secret -n my-app docker-registry my-registry-key \
--docker-server=xxxx \
--docker-username=xxxx \
--docker-password=xxxx \
--docker-email=xxxx

kubectl apply -f db-secret.yaml -n my-app
kubectl apply -f db-config.yaml -n my-app
kubectl apply -f java-app.yaml -n my-app
```

**Exercise 4 and 5**

```
kubectl create secret -n my-app docker-registry my-ecr-registry-key --docker-server=xxxx --docker-username=xxxx --docker-password=xxxx

ssh -i ./ssh/id_rsa.pub {user}@{public-ip}

sudo docker exec -it xxxx -u 0 bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install

apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl

apt-get update
apt-get install -y gettext-base

```

Adapt the JenkinsFile and run the pipeline.

**Exercise 6**

```

```
