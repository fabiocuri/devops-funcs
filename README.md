**Exercise 1**

Go to https://www.virtualbox.org/wiki/Downloads
Install Windows version.
Click on "New", add name and select Linux, Ubuntu 64 bit.
Select base memory and processors.
Create virtual hard disk.
Click on "Finish" and the VM is created.
Go to https://linuxmint.com/ and download an ISO for Linux Mint.
Upload ISO in the Settings/Storage.
Linux Mint is installed.

In order to check the distribution, run` cat /etc/os-release`
Given that this is a Debian-based distribution, the package manager is APT and APT-GET. I also double-checked it by running `which apt` and `which apt-get`, which returned the paths. "yum" is not installed because `which yum` returns empty.
The CLI editors that were installed are "vi" and "nano", and not "vim", also by checking their correspondings "which" commands.
The software manager is found through echo `$XDG_CURRENT_DESKTOP` and it gives X-Cinnamon.
Finally, the "bash" shell is used, which is given by `echo $SHELL`.

------------

**Exercise 2**
```bash
#!/bin/bash

sudo apt update

sudo apt install -y default-jdk

java_version=$(java -version 2>&1 > /dev/null  | grep "java version\|openjdk version" | awk '{print substr($3,2,2)}')

if [ "$java_version" == "" ]
then
    echo Installing Java has failed. No java version found.
elif [ "$java_version" -l "11" ]
then
    echo An old version of Java installation found.
elif [ "$java_version" -ge 11 ]
then
    echo Java version 11 or greater installed successfully.
fi
```

------------

**Exercise 3**

`touch file.sh`
`vim file.sh`

Add the code:

```bash
#!/bin/bash

ps aux | grep -i $USER
```

Press ESC and then write :wq

------------

**Exercise 4**

`vim file.sh`

Add the code:
```bash
#!/bin/bash

echo -n "Would you like to sort the processes output by memory or CPU? (m/c) "
read sortby

if [ "$sortby" = "m" ]
then
    ps aux --sort -rss | grep -i $USER
elif [ "$sortby" = "c" ]
then
    ps aux --sort -%cpu | grep -i $USER
else
    echo "No input provided. Exiting"
fi

```
Press ESC and then write :wq

------------

**Exercise 5**

`vim file.sh`

```bash
#!/bin/bash

echo -n "Would you like to sort the processes output by memory or CPU? (m/c) "
read sortby
echo -n "How many results do you want to display? "
read lines

if [ "$sortby" = "m" ]
then
    ps aux --sort -rss | grep -i `whoami` | head -n "$lines"
elif [ "$sortby" = "c" ]
then
    ps aux --sort -%cpu | grep -i `whoami` | head -n "$lines"
else
    echo "No input provided. Exiting"
fi
```
Press ESC and then write :wq

------------

**Exercise 6**

```bash
#!/bin/bash

sudo apt update

sudo apt install -y nodejs npm curl net-tools

node_version=$(node --version)
echo "NodeJS version $node_version installed"

npm_version=$(npm --version)
echo "NPM version $npm_version installed"

wget https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz

tar zxvf ./bootcamp-node-envvars-project-1.0.0.tgz

export APP_ENV=dev
export DB_PWD=mysecret
export DB_USER=myuser

cd package

npm install

node server.js &

```

------------

**Exercise 7**

```bash
#!/bin/bash

sudo apt update

sudo apt install -y nodejs npm curl net-tools

node_version=$(node --version)
echo "NodeJS version $node_version installed"

npm_version=$(npm --version)
echo "NPM version $npm_version installed"

wget https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz

tar zxvf ./bootcamp-node-envvars-project-1.0.0.tgz

export APP_ENV=dev
export DB_PWD=mysecret
export DB_USER=myuser

cd package

npm install

node server.js &

ps aux | grep node | grep -v grep

netstat -ltnp | grep :3000
```

------------

**Exercise 8**

```bash
#!/bin/bash

sudo apt update

sudo apt install -y nodejs npm curl net-tools

node_version=$(node --version)
echo "NodeJS version $node_version installed"

npm_version=$(npm --version)
echo "NPM version $npm_version installed"

wget https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz

tar zxvf ./bootcamp-node-envvars-project-1.0.0.tgz

export APP_ENV=dev
export DB_PWD=mysecret
export DB_USER=myuser

echo -n "Set log directory location for the application (absolute path): "
read LOG_DIRECTORY
if [ -d $LOG_DIRECTORY ]
then
  echo "$LOG_DIRECTORY already exists"
else
  mkdir -p $LOG_DIRECTORY
  echo "A new directory $LOG_DIRECTORY has been created"
fi

cd $LOG_DIRECTORY
export LOG_DIR=$(pwd)
cd ..

cd package

npm install

node server.js & > $LOG_DIR/app.log

ps aux | grep node | grep -v grep

netstat -ltnp | grep :3000
```

------------

**Exercise 9**

```bash
#!/bin/bash

NEW_USER=myapp

useradd $NEW_USER -m

sudo apt update

sudo apt install -y nodejs npm curl net-tools

node_version=$(node --version)
echo "NodeJS version $node_version installed"

npm_version=$(npm --version)
echo "NPM version $npm_version installed"

runuser -l $NEW_USER -c "wget https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz"

runuser -l $NEW_USER -c "tar zxvf ./bootcamp-node-envvars-project-1.0.0.tgz"

echo -n "Set log directory location for the application (absolute path): "
read LOG_DIRECTORY
if [ -d $LOG_DIRECTORY ]
then
  echo "$LOG_DIRECTORY already exists"
else
  mkdir -p $LOG_DIRECTORY
  echo "A new directory $LOG_DIRECTORY has been created"
fi

cd $LOG_DIRECTORY
runuser -l $NEW_USER -c "export LOG_DIR=$(pwd)"
cd ..

chown $NEW_USER -R $LOG_DIRECTORY

runuser -l $NEW_USER -c "
    export APP_ENV=dev && 
    export DB_PWD=mysecret && 
    export DB_USER=myuser && 
    cd package && 
    npm install && 
    node server.js & > $LOG_DIR/app.log"

ps aux | grep node | grep -v grep

netstat -ltnp | grep :3000
```
