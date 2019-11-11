---
title: git操作
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: git的一些仓用操作，更新回滚代码提交
categories: git
tags: git
keywords: git操作
abbrlink: 46390b34
date: 2019-08-26 13:17:49
password:
copyright: true
---

# Git操作

在github上创建一个仓库repository:  例如  config-repo

接下来在本地创建一个有文件的目录，cd到目录下：

## git命令

### 初始化仓库：  git init

cd  目录-------》git init

### 添加要传入仓库的文件: git add

添加全部： git add .

添加某个文件： git add 某个文件例如：git add README.md

### git提交：git commit

```shell
git commit -m "first commit"  （first commit可随意填写）
```

### git 连接远程仓库

```shell
git remote add origin （仓库git地址）例如：https://github.com/JunTech/ms-weather-forecast-sys.git
```

### git上传文件到仓库

```shell
git push -u origin master
```

如果连接了远程仓库并提交过文件

可直接使用:

```shell
git remote add origin https://github.com/JunTech/ms-weather-forecast-sys.git
git push -u origin master
```

### git 绑定github

​	创建ssh key:

```shell
ssh-keygen -t rsa -C ["your_email@youremail.com](mailto:%22your_email@youremail.com)"
```

​	l连按3次回车，然后成功后会在User文件夹对应的用户下创建.ssh文件夹，其中有一个id_rsa.pub文件，我们复制其中的key:到github仓库的设置页面的ssh key填写就行了。

验证是否绑定本地成功，在git-bash中验证，输入指令：

```shell
ssh -T git@github.com
```

如果第一次执行该指令，则会提示是否continue继续，如果我们输入yes就会看到成功信息：

由于GitHub每次执行commit操作时，都会记录username和email，所以要设置它们：

```shell
git config --global user.name "your name"
git config --global user.email  "your email"
```



### git查看状态

```shell
git status
```

### git撤销和回滚

 git的撤销与回滚在平时使用中还是比较多的，比如说我们想将某个修改后的文件撤销到上一个版本，或者是想撤销某次多余的提交，都要用到git的撤销和回滚操作。撤销分两种情况，一个是commit之前，一个是commit之后，下面具体看下这两种情况。

  **一.git commit之前**

​        未添加到暂存区的撤销(没有git add)

​        添加进暂存区的撤销(git add后)

```shell
$ git status
On branch test_git
Changes not staged for commit:     没有添加到暂存区
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
```

可以通过

```
 git checkout -- filename来撤销修改
 例如： git checkout -- dsnsjd.md
```

再次查看文件状态发现选择的文件已经成功被撤销了。

```shell
$ git status
On branch test_git
Changes not staged for commit:  
  (use "git add <file>..." to update what will be committed)
```

如果想将多个文件一次性撤销可以用

```shell
 git checkout -- .
```

上面是 未添加到暂存区的撤销

下面是添加到暂存区的

```shell
git status
On branch test_git
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
```

```shell
$ git add -A
$ git status
On branch test_git
Changes to be committed:    已经添加暂存区了
  (use "git reset HEAD <file>..." to unstage)
```

从暂存区撤销

```shell
git reset HEAD filename
```

如果想一次性将所有暂存区文件撤销回来，还是上面的命令，不过不用加文件路径。

```shell
git reset HEAD
```

**二.git commit之后**

如果当commit提交后想撤销的话，这就需要revert命令。git revert 命令是撤销某次操作，而在此次操作之前和之后的提交记录都会保留。

先修改了几个文件然后commit 再用git log查看提交记录。

```
commit 2842c8065322085c31fb7b8207b6296047c4ea3
Author: songguojun <songgj@kingnet.sh>
Date:   Sat Apr 28 11:21:30 2018 +0800

    add content
```

然后使用revert  后面跟上git提交的commitid

```
git  revert 2842c8065322085c31fb7b8207b6296047c4ea3
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Revert "add content"

This reverts commit 2842c8065ffe2085c31fb7b8207b6296047c4ea3.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch test_git
# Changes to be committed:
#       modified:   new_src/app/Http/Controllers/Frontend/KyHome/KyHomeApp.php
#       modified:   new_src/app/Http/Controllers/Frontend/KyHome/KyHomeComment.php
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

然后在推送到远端更新远程仓库代码，修改的文件就撤销回来了。注意的是revert奇数次生效，偶数次又回到之前的修改状态。比如一个文件内容是a，那么修改为ab，revert后文件变成了a，如果在revert后文件又还原成ab了。

 

还有就是如果想回到之前某个版本，可以用reset命令，可以回退到某次提交，那该提交之后的提交都会回滚，不过这种覆盖是不可逆的，之前的提交记录都没有了。所以平时开发中尽量注意，避免使用reset。

用法:

```
git  reset --hard  commit_id
```

- --hard – 强制将缓存区和工作目录都同步到你指定的提交

```
 git  reset --hard fdeb212a5418cc8e31f32d63cf197550297468ec
HEAD is now at fdeb212 增加mysql端口配置
```

### git branch        -----查看当前分支

### git pull             -----更新代码到本地