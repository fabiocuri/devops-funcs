This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 1**

```
git clone https://gitlab.com/twn-devops-bootcamp/latest/08-jenkins/jenkins-exercises.git
cd jenkins-exercises

rm -rf .git
git init 
git add .
git commit -m "initial commit"

git remote add origin git@github.com:fabiocuri/08-jenkins.git
git push -u origin master

```

Then create Dockerfile:

```
FROM node:20-alpine

RUN mkdir -p /usr/app
COPY app/* /usr/app/

WORKDIR /usr/app
EXPOSE 3000

RUN npm install
CMD ["node", "server.js"]
```

------------
**Exercise 2**

Create credentials for Gitlab and Docker.

Install the NodeJS plugin and the JSON parser.

Write the following Jenkinsfile

```
#!/usr/bin/env groovy
pipeline {
    agent any
    tools {
        nodejs "21.7.1"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    dir("app") {
                        sh "npm version minor"
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }
                }
            }
        }
        stage('run tests') {
            steps {
               script {
                    dir("app") {
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                    sh "docker build -t fabiocuri/npm-app:${IMAGE_NAME} ."
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push fabiocuri/npm-app:${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'gitlab_credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh 'git remote set-url origin https://$USER:$PASS@gitlab.com/fabiocuri/jenkins-exercise.git'
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:master'
                    }
                }
            }
        }
    }
}
```
------------
**Exercise 3**

```
ssh root@XXXXXXXXX

docker login

docker run -p 3000:3000 fabiocuri/npm-app:1.2.0-8
```
------------
**Exercise 4**

The functions were added to https://gitlab.com/fabiocuri/jenkins-shared-library and the new code is here below:

```
#!/usr/bin/env groovy
library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
        [$class: 'GitSCMSource',
        remote: 'https://gitlab.com/fabiocuri/jenkins-shared-library.git',
        credentialsId: 'gitlab_credentials'])

pipeline {
  agent any
  tools {
    nodejs "21.7.1"
  }
  stages {
    stage('increment-version') {
      steps {
        incrementVersion()
      }
    }
    stage('run-test') {
      steps {
        runTest()
      }
    }
    stage('build-push-docker') {
      steps {
        buildPushDocker "fabiocuri/npm-app"
      }
    }
    stage('commit version update') {
      steps {
        script {
          commitGit "gitlab.com/fabiocuri/jenkins-exercise.git"
        }
      }
    }
  }
}
```