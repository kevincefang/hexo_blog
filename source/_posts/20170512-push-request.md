---
title: github提交代码到开源项目
date: 2017-05-12 12:30:10
tags: 
- GitHub
- git
categories: 工具
---

#### 首先,在github中把开源项目fork到自己的仓库

```shell
$ git clone https://github.com/kevincefang/test.git
$ cd test
$ git config user.name "yourname"
$ git config user.email "your email"
```

#### 更新内容提交后,并推送到自己的仓库

```shell
$ #do some change on the content
$ git add .
$ git commit -m "Fix issue"
$ git push
```

#### 最后，在 GitHub 网站上提交 pull request 即可。

> 另外，建议定期使用项目仓库内容更新自己仓库内容。

```Shell
$ git remote add upstream https://github.com/kevincefang/test.git
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master
```

