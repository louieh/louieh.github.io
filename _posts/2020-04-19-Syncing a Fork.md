---
title: "Syncing a Fork"
layout: post
date: 2020-04-19
headerImage: false
tag:
- note
- git
star: true
category: blog
---



Sync a fork of a repository to keep it up-to-date with the upstream repository.

1. configure a remote that points to the upstream repository

   git remote -v

   ```bash
   origin  git@github.com:louieh/indigo.git (fetch)
   origin  git@github.com:louieh/indigo.git (push)
   ```

2. git remote add upstream

   ```bash
   # 添加上游源
   git remote add upstream <https://github.com/sergiokopplin/indigo.git>
   ```

   ```bash
   # 可以看到目前已经有两个源，origin为此repo的上游，upstream为fork的repo的上游
   git remote -v
   
   origin  git@github.com:louieh/indigo.git (fetch)
   origin  git@github.com:louieh/indigo.git (push)
   upstream        <https://github.com/sergiokopplin/indigo.git> (fetch)
   upstream        <https://github.com/sergiokopplin/indigo.git> (push)
   ```

3. Fetch the remote branch

   ```bash
   # 获取fork的repo的分支
   git fetch upstream
   
   From <https://github.com/sergiokopplin/indigo>
    * [new branch]      gh-pages   -> upstream/gh-pages
   
   # 可以看到远程分支多了fork的repo的分支
   git branch -a
   
   * gh-pages
     remotes/origin/HEAD -> origin/gh-pages
     remotes/origin/gh-pages
     remotes/upstream/gh-pages
   ```

4. Merge the changes from upstream/master into local master branch

   ```bash
   git merge upstream/gh-pages  # 直接将远程分支与所在分支合并
   
   git checkout -b upstream/gh-pages upstream/gh-pages  # 也可以根据远程分支创建一个本地分支 
   ```



远程分支，本地分支，源的关系

```bash
# 查看本地分支与远程分支关联关系，3条命令
git branch -vv
git remote show origin
cat .git/config

# 设置本地分支的上游
git branch --set-upstream
```