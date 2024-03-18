This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 0**

```
git clone https://gitlab.com/twn-devops-bootcamp/latest/07-docker/docker-exercises.git
cd docker-exercises

rm -rf .git
git init 
git add .
git commit -m "initial commit"

git remote add origin git@github.com:fabiocuri/07-docker.git
git push -u origin master

```

------------
**Exercise 1**

```
sudo docker run -p 3306:3306 \
--name mysql \
-e MYSQL_ROOT_PASSWORD=rootpass \
-e MYSQL_DATABASE=team-member-projects \
-e MYSQL_USER=admin \
-e MYSQL_PASSWORD=adminpass \
-d mysql mysqld --default-authentication-plugin=mysql_native_password

export DB_USER=admin
export DB_PWD=adminpass
export DB_SERVER=localhost
export DB_NAME=team-member-projects

gradle build

java -jar build/libs/docker-exercises-project-1.0-SNAPSHOT.jar
```

------------
**Exercise 2**

```
sudo docker run -p 8083:80 \
--name phpmyadmin \
--link mysql:db \
-d phpmyadmin/phpmyadmin
```

------------
**Exercise 3**

```
version: '3'
services:
  mysql:
    image: mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=team-member-projects
      - MYSQL_USER=admin    
      - MYSQL_PASSWORD=adminpass
    volumes:
    - mysql-data:/var/lib/mysql
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
  phpmyadmin:
    image: phpmyadmin
    environment:
      - PMA_HOST=mysql
    ports:
      - 8083:80
    container_name: phpmyadmin
volumes:
  mysql-data:
    driver: local
```

and then

```
sudo docker-compose -f docker_compose.yaml up
```
------------
**Exercise 4**

```
FROM openjdk:17.0.2-jdk
EXPOSE 8080
RUN mkdir /opt/app
COPY build/libs/docker-exercises-project-1.0-SNAPSHOT.jar /opt/app
WORKDIR /opt/app
CMD ["java", "-jar", "docker-exercises-project-1.0-SNAPSHOT.jar"]
```

------------
**Exercise 5**

```
gradle build
docker login
docker build -t fabiocuri/java-app-docker .
docker tag fabiocuri/java-app-docker:latest fabiocuri/java-app-docker:tagname
docker push fabiocuri/java-app-docker:tagname
```

------------
**Exercise 6**

```
version: '3'
services:
  my-java-app:
    image: java-mysql-app:1.0 # specify the full image name with repository name
    environment:
      - DB_USER=${DB_USER}
      - DB_PWD=${DB_PWD}
      - DB_SERVER=${DB_SERVER}
      - DB_NAME=${DB_NAME}
    ports:
    - 8080:8080
    container_name: my-java-app
    depends_on:
      - mysql
  mysql:
    image: mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PWD}
    volumes:
    - mysql-data:/var/lib/mysql
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8083:80
    environment:
      - PMA_HOST=${PMA_HOST}
      - PMA_PORT=${PMA_PORT}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    container_name: phpmyadmin
    depends_on:
      - mysql
volumes:
  mysql-data:
    driver: local
```

And then run

```
export DB_USER=admin
export DB_PWD=adminpass
export DB_SERVER=mysql
export DB_NAME=team-member-projects

export MYSQL_ROOT_PASSWORD=rootpass

export PMA_HOST=mysql
export PMA_PORT=3306

docker-compose -f docker-compose-with-app.yaml up    
```

------------
**Exercise 7**

Manually add the /etc/docker/daemon.json file and content.
Changed the HOST value in the HTML.

```
sudo service docker restart

gradle build
docker login
docker build -t fabiocuri/java-app-docker .
docker tag fabiocuri/java-app-docker:latest fabiocuri/java-app-docker:tagname
docker push fabiocuri/java-app-docker:tagname

scp docker_compose.yaml {server-user}:{server-ip}:/home/{server-user}

```

And follow the same steps as the last exercise.

------------
**Exercise 8**

Manual configuration of firewall.