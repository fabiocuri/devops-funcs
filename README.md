This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 0**

```
git clone git@gitlab.com:twn-devops-bootcamp/latest/05-cloud/cloud-basics-exercises.git
cd cloud-basics-exercises/

rm -rf .git
git init 
git add .
git commit -m "initial commit"

git remote add origin git@github.com:fabiocuri/05-cloud.git
git push -u origin master
```

------------
**Exercise 1**

```
cd app
npm pack
```

------------
**Exercise 2, 3**

I hid the IP number below for security reasons.

```
ssh root@{XXXXXXXXXXXXXX}
apt install -y nodejs npm
```

------------
**Exercise 4**

```
scp bootcamp-node-project-1.0.0.tgz fabio@{XXXXXXXXXXXXXX}:/home/fabio
```

------------
**Exercise 5**

```
ssh -i ~/id_rsa root@{XXXXXXXXXXXXXX}
tar -zxvf bootcamp-node-project-1.0.0.tgz
cd package

npm install
node server.js
```

------------
**Exercise 6**

Manual step.