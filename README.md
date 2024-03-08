This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 1**

I created a VM in GCP with enough RAM and disk storage for this exercise, then manually installed java and nexus.

------------
**Exercise 2**

Manual steps.

------------
**Exercise 3**

Manual steps.

------------
**Exercise 4**

```
npm publish --registry=http://X.X.X.X:8081/repository/npm-packages/ nodejs-app-1.0.0.tgz

```

------------
**Exercise 5**

Manual steps.

------------
**Exercise 6**

Manual steps.

------------
**Exercise 7**

```
gradle build
gradle publish
```

------------
**Exercise 8**

```
curl -u admin:XXXX -X GET 'http://X.X.X.X:8081/service/rest/v1/components?repository=npm-packages&sort=version' | jq "." > artifact.json
artifactDownloadUrl=$(jq '.items[].assets[].downloadUrl' artifact.json --raw-output)
wget --http-user=admin --http-password=XXXX $artifactDownloadUrl
npm install nodejs-app-1.0.0.tgz
npm run start
```

------------
**Exercise 9**

```
#!/bin/bash
curl -u admin:XXXX -X GET 'http://X.X.X.X:8081/service/rest/v1/components?repository=npm-packages&sort=version' | jq "." > artifact.json
artifactDownloadUrl=$(jq '.items[].assets[].downloadUrl' artifact.json --raw-output)
wget --http-user=admin --http-password=XXXX $artifactDownloadUrl
npm install nodejs-app-1.0.0.tgz
npm run start
```

