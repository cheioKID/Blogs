---
date: 2017-11-21
title: "Git note"
tags:
    - Git
    - Software Engineer
categories:
    - Git
comment: true
---

# git笔记

- 版本控制系统Git，代码托管采用Github仓库
### 初始化
- 在目录下通过`git init` 初始化空仓库Initialized empty Git repository
### 暂存区
- 通过`git add`指令将修改添加到暂存区

- 通过`git status`查看状态**是否commit**

- 通过`git rm <filename>`删除版本库中的文件，实现和工作区的同步

- 通过`git commit`提交修改，`git commit -m “commit name"` 为本次commit命名

```bash
git add . # add all
git add   # add changed files
# use git rm --cached <file> to unstage
git status
git commit -m 'Initial commit'
```

### 版本

- 通过`git log`查看版本控制记录，即**commit历史记录**

- git log --pretty=oneline breif info`
### 回退
- 通过`git reset --hard HEAD^`回退到上一个版本，撤销最近一次commit `HEAD`代表当前版本，`HEAD^`代表当前版本的上一个版本，`HEAD^^`代表当前版本的上一个版本的上一个版本

- 通过`git reset —hard <commit-id>`回退到指定id的版本

- 通过`git checkout -- <filename>`撤销工作区的修改

### 分支
- 通过`git checkout -b <branch-name>`创建并切换到该分支**branch**，`git branch`查看当前分支(带\*)，分支之间的修改不互相影响，通过`git merge branch-name`将某分支合并到当前分支

- 不同分支可以分别提交，但是无法直接合并，需要手动解决冲突每个组员可以在不同的分支上开发提交修改

- 通过`git stash`保存现场，创建新的分支，修复bug，然后合并到**master**分支，`git stash pop`回到现场，继续工作

- 通常增加新的功能，采取创建新的**feature**分支，在合并之前可以通过`git branch -D <name>`删除分支

### 远端仓库
- 通过`git push <remote-name> <branch-name>`推送分支到远程仓库

- 通过`git pull`抓取最新的提交，和本地合并，解决冲突，然后**push**

- 切换到某分支，通过`git tag <name>`，为分支最近一次的commit定义标签**tag**，`git tag`查看所有标签 

---

命令`git push origin <tagname>`可以推送一个本地标签； 

命令`git push origin --tags`可以推送全部未推送过的本地标签； 

命令`git tag -d <tagname>`可以删除一个本地标签； 

命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。 

#### 初始提交到远端仓库
```bash
#初始提交到远端仓库
git add .
git pull origin master --allow-unrelated-histories
```
### git --help
```
usage: git [--version][--help] [-C <path>][-c name=value]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | --no-pager][--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]
```