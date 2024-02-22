This repo contains all the code for the exercises of the DevOps bootcamp.

Author: Fabio Curi

**Exercise 1**

```
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
git rm --cached .DS_Store

git rm -r --cached .idea
git rm -r --cached out
git rm -r --cached build

git add .
git commit -m "remove ignored files"
git push

```

------------
**Exercise 3**

`git checkout -b feature/changes`

Change version manually.
Add image in HTML manually.

```
git diff
git add .
git commit -m "Upgrade logback library and add image url"

git push
```

------------
**Exercise 4**

`git checkout -b bugfix/changes`

Change line manually to log.info("Java app started");

```
git diff
git add .
git commit -m "Fix spelling error"
git push
```

------------
**Exercise 5**

```
git checkout master
git merge feature/changes 
git push
```

------------
**Exercise 6**

`git checkout bugfix/changes`

Change line manually to include new version.

```
git add .
git commit -m "upgrade logger library version"
git merge master
git status
git push
```

------------
**Exercise 7**

Change the typo manually.

```
git add .
git commit -m "Fix spelling error"
```

Change the image URL manually.

```
git add .
git commit -m "Set image url"
git push
git revert HEAD
git push
```

------------
**Exercise 8**

Change text manually.

<li>Bruno - DevOps engineer</li>

```
git add .
git commit -m "Adjust employee role description"
git reset HEAD~1
```