This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 1**

```
#!/bin/bash
git clone git@gitlab.com:twn-devops-bootcamp/latest/03-git/git-exercises.git
cd git-exercises

rm -rf .git
git init
git add .
git commit -m "Initial commit"

git remote add origin git@github.com:fabiocuri/03-git-devops.git
git push -u origin master
```

------------
**Exercise 2**

`touch .gitignore`

Add these:

    .idea 
    .DS_Store
    out 
    build

Run:
```
#!/bin/bash
git rm --cached .DS_Store

git rm -r --cached .idea
git rm -r --cached out
git rm -r --cached build

git add .
git commit -m "remove ignored files"
git push

```