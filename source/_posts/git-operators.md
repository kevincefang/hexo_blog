---
title: Git常用命令
date: 2017-11-29 18:30:10
tags: 
- git
categories: git
---

> 理解这些指令，觉得最重要的是理解Git的内部原理，比如Git的分布式版本控制，分清楚工作区、暂存区、版本库，还有就是理解Git跟踪并管理的是修改，而非文件。

![1](http://fhaoer.com/images/20171129/1.jpg)

## 第一步是要获得一个GIT仓库

有两种获得GIT仓库的方法，一是在需要用GIT管理的项目的根目录执行：

```
git init
```

执行后可以看到，仅仅在项目目录多出了一个.git目录，关于版本等的所有信息都在这个目录里面。
　　另一种方式是克隆远程目录，由于是将远程服务器上的仓库完全镜像一份至本地，而不是取某一个特定版本，所以用clone而不是checkout：

```
git clone <url>
```

## 设置

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

ssh -v git@github.com //检验下Git安装是否正确，
显示hi github用户名！You’ve successfully authenticated 说明Git安装正确

## 提交

git tracked的是修改，而不是文件
![2](http://fhaoer.com/images/20171129/2.jpg)

```Shell
#将“当前修改”移动到暂存区(stage)
$ git add somfile.txt
#将暂存区修改提交
$ git commit -m "Add somfile.txt."
```

## 状态

```shell
$ git status
$ git diff
```

## 回退

```shell
# 放弃工作区修改
$ git checkout -- file.name
$ git checkout -- .

# 取消commit(比如需要重写commit信息)
$ git reset --soft HEAD

# 取消commit、add(重新提交代码和commit)
$ git reset HEAD
$ git reset --mixed HEAD

# 取消commit、add、工作区修改(需要完全重置)
$ git reset --hard HEAD
```

## 记录

```shell
$ git reflog
$ git log
```

## 删除

```shell
$ rm file.name
$ git rm file.name
$ git commit -m "Del"
```

## Git 远程分支管理

```shell
git pull                         # 抓取远程仓库所有分支更新并合并到本地
git pull --no-ff                 # 抓取远程仓库所有分支更新并合并到本地，不要快进合并
git fetch origin                 # 抓取远程仓库更新
git merge origin/master          # 将远程主分支合并到本地当前分支
git checkout --track origin/branch     # 跟踪某个远程分支创建相应的本地分支
git checkout -b <local_branch> origin/<remote_branch>  # 基于远程分支创建本地分支，功能同上
 
git push                         # push所有分支
git push origin master           # 将本地主分支推到远程主分支
# 第一次推送，-u(--set-upstream)指定默认上游
git push -u origin master        # 将本地主分支推到远程(如无远程主分支则创建，用于初始化远程仓库)
git push origin <local_branch>   # 创建远程分支， origin是远程仓库名
git push origin <local_branch>:<remote_branch>  # 创建远程分支
git push origin :<remote_branch>  #先删除本地分支(git branch -d <branch>)，然后再push删除远程分支
```

## Git 远程仓库管理

```shell
git remote -v                    # 查看远程服务器地址和仓库名称
git remote show origin           # 查看远程服务器仓库状态
git remote add origin git@github.com:whuhacker/Unblock-Youku-Firefox.git         # 添加远程仓库地址
git remote set-url origin git@github.com:whuhacker/Unblock-Youku-Firefox.git # 设置远程仓库地址(用于修改远程仓库地址)
git remote rm <repository>       # 删除远程仓库
```

## 克隆

```shell
$ git clone https://github.com/Yikun/yikun.github.com.git path
$ git clone git@github.com:Yikun/yikun.github.com.git path
```

## 分支操作

![3](http://fhaoer.com/images/20171129/3.jpg)

```shell
# 查看当前分支
$ git branch

# 创建分支
$ git branch dev
# 切换分支
$ git checkout dev

# 创建并checkout分支
$ git checkout -b dev

# 合并分支
$ git merge dev

# 删除分支
$ git branch -d dev
```

## 标签

```shell
$ git tag 0.1.1
$ git push origin --tags
```

目前使用git主要流程就是先在本地clone一个git仓库，然后设置一下看git是否连接成功，接着git remote add origin git@github.com:michaelliao/learngit.git，添加要连接的远程仓库地址，进行push（将本地主分支推到远程主分支）还是pull（抓取远程仓库所有分支更新并合并到本地）操作即可。