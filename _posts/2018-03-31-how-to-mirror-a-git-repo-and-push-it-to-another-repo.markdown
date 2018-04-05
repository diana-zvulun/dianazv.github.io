---
layout: post
title: "How to Mirror a git repo and push it to another repo"
date: 2018-03-31T20:00:50+03:00
comments: true
categories: git
---
Create a new repo in github - add name,description but uncheck the initialize button

Clone the remote repository to your local filesystem
```
git clone repo.git
```

Then change the git configuration to remove the default fetch and ask to fetch only heads,tags and changes:

```
cd repo directory
git config -e
```

Change - bare = true in [core]
```
[core]
bare = true
```

In [remote "origin"] section remove the default fetch and add the following to avoid an error when trying to mirror pull requests:
```
[remote "origin"]
        fetch = +refs/heads/*:refs/heads/*
        fetch = +refs/tags/*:refs/tags/*
        fetch = +refs/change/*:refs/change/*
```	

add a section of [remote "mirrors"]:
```
[remote "mirrors"]
        url = github.com/repo.git
        mirror = true
        skipDefaultUpdate = true
```

Run git remote update to get a full clone of your git repository (a bare clone):
```
git remote update
```

And run git push mirrors to push to your new repository:
```
git push mirrors
```

### **Sources** ###

* [https://help.github.com/articles/duplicating-a-repository/](https://help.github.com/articles/duplicating-a-repository/)
* [https://stackoverflow.com/questions/34265266/remote-rejected-errors-after-mirroring-a-git-repository](https://stackoverflow.com/questions/34265266/remote-rejected-errors-after-mirroring-a-git-repository)


