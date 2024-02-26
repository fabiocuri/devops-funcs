This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 0**

```
git clone git@gitlab.com:twn-devops-bootcamp/latest/04-build-tools/build-tools-exercises.git
cd build-tools-exercises

rm -rf .git
git init
git add .
git commit -m "Initial commit"

git remote add origin git@github.com:fabiocuri/04-build-tools.git
git push -u origin master
```

-------------

**Exercise 1**

```
gradle build
```

-------------

**Exercise 2**

Manually change the value.

```
gradle test
```

-------------

**Exercise 3**

```
gradle clean 
gradle build
```

-------------

**Exercise 4**

```
java -jar build/libs/build-tools-exercises-1.0-SNAPSHOT.jar
```

-------------

**Exercise 5**

Change file manually.

```
gradle build
java -jar build/libs/build-tools-exercises-1.0-SNAPSHOT.jar fabio curi
```