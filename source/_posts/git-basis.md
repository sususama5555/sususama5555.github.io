---
title: Git原理及实践
date: 2019-11-12 22:45:15
tags: 
- Git
- 版本控制
categories: 
- Git
---

# Git是什么?
## 什么是版本控制？
版本控制是指对软件开发过程中各种程序代码、配置文件及说明文档等文件变更的管理，是软件配置管理的核心思想之一。

## 什么是Git?
![](/picture/git_logo.png)
Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
Git 与常用的版本控制工具 CVS， Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。
<!--more-->

## Git 与 SVN 区别
Git 不仅仅是个版本控制系统，它也是个内容管理系统(CMS)，工作管理系统等。
如果你是一个具有使用 SVN 背景的人，你需要做一定的思想转换，来适应 Git 提供的一些概念和特征。
Git 与 SVN 区别点：
1、Git 是分布式的，SVN 不是：这是 Git 和其它非分布式的版本控制系统，例如 SVN，CVS 等，最核心的区别。
2、Git 把内容按元数据方式存储，而 SVN 是按文件：所有的资源控制系统都是把文件的元信息隐藏在一个类似 .svn、.cvs 等的文件夹里。
3、Git 分支和 SVN 的分支不同：分支在 SVN 中一点都不特别，其实它就是版本库中的另外一个目录。
4、Git 没有一个全局的版本号，而 SVN 有：目前为止这是跟 SVN 相比 Git 缺少的最大的一个特征。
5、Git 的内容完整性要优于 SVN：Git 的内容存储使用的是 SHA-1 哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。

# Git的理论基础
* Git的四大工作区域
* Git的工作流程
* Git文件的四种状态
* Git的工作原理

## Git的四大工作区域
![avatar](/picture/git_4_workspace.png)

* <kbd style="color:#ff7600">Workspace</kbd>：你电脑本地看到的文件和目录，在Git的版本控制下，构成了工作区。
* <kbd style="color:#ff7600">Index/Stage</kbd>：暂存区，一般存放在 .git目录下，即.git/index，它又叫待提交更新区，用于临时存放你未提交的改动。比如，你执行git add，这些改动就添加到这个区域啦。
* <kbd style="color:#ff7600">Repository</kbd>：本地仓库，你执行git clone 地址，就是把远程仓库克隆到本地仓库。它是一个存放在本地的版本库，其中HEAD指向最新放入仓库的版本。当你执行git commit，文件改动就到本地仓库来了~
* <kbd style="color:#ff7600">Remote</kbd>：远程仓库，就是类似github，码云等网站所提供的仓库，可以理解为远程数据交换的仓库~

## Git的工作流程
![avatar](/picture/git_work_process.png)

## Git文件的四种状态
根据一个文件是否已加入版本控制，可以把文件状态分为：Tracked(已跟踪)和Untracked(未跟踪)，而tracked(已跟踪)又包括三种工作状态：Unmodified，Modified，Staged

![avatar](/picture/git_file_status.png)

* Untracked: 文件还没有加入到git库，还没参与版本控制，即未跟踪状态。这时候的文件，通过git add 状态，可以变为Staged状态
* Unmodified：文件已经加入git库， 但是呢，还没修改， 就是说版本库中的文件快照内容与文件夹中还完全一致。修改变为Modified. 可用git remove移出版本库， 变为Untracked。
* Modified：文件被修改了，就进入modified状态啦，通过stage命令进入staged状态
* staged：暂存状态. 执行git commit将修改同步到库中，这时库中的文件和本地文件变为一致，为Unmodified状态.

## Git的工作原理
![avatar](/picture/git_work_principle.png)

# Git基础命令

## Git命令流程
以下是命令使用的大致流程
![avatar](/picture/git_use_detail.png)

## Git常用命令集
遇事不决查文档，<kbd style="color:#ff7600">git -help</kbd> + <kbd style="color:#ff7600">git command -help</kbd>
### git init -初始化仓库

### git clone url -克隆远程版本库

### git remote add newRemote url -添加另一个远程仓库
```
git remote add [-t <branch>] [-m <master>] -添加仓库的高级版
git remote [-v | --verbose] -查看远程所有仓库
git remote rename <old> <new> -重命名仓库名
git remote remove <name> -移除本地绑定的远程仓库
```

### git checkout
```
git checkout -b newBranch  [origin/newBranch ] 创建开发分支dev，并切换到该分支下在上面基础上[并关联远程分支]
git checkout [file]  丢弃某个文件file(还未add进暂存区)
git checkout .  丢弃所有文件(还未add进暂存区)(已经add了则用reset)
```

### git add
```
git add .	添加当前目录的所有文件到暂存区
git add [dir]	添加指定目录到暂存区，包括子目录
git add [file1]	添加指定文件到暂存区
```

### git commit
```
git commit -m [message] 提交暂存区到仓库区，message为说明信息
git commit [file1] -m [message] 提交暂存区的指定文件到本地仓库
git commit --amend -m [message] 使用一次新的commit，替代上一次提交
```

### git log
```
git log  查看提交历史
git log --oneline 以精简模式显示查看提交历史
git log -p <file> 查看指定文件的提交历史
git blame <file> 一列表方式查看指定文件的提交历史
```

### git diff
```
git diff 显示暂存区和工作区的差异
git diff filepath   filepath路径文件中，工作区与暂存区的比较差异
git diff HEAD filepath 工作区与HEAD ( 当前工作分支)的比较差异
git diff branchName filepath 当前分支的文件与branchName分支的文件的比较差异
git diff commitId filepath 与某一次提交的比较差异
```

### git status
```
git status  查看当前工作区暂存区变动
git status -s  查看当前工作区暂存区变动，概要信息
git status  --show-stash 查询工作区中是否有stash（暂存的文件）
```

### git pull/git fetch
```
git pull  拉取远程仓库所有分支更新并合并到本地分支。
git pull origin master 将远程master分支合并到当前本地分支
git pull origin master:master 将远程master分支合并到当前本地master分支，冒号后面表示本地分支
git fetch --all  拉取所有远端的最新代码
git fetch origin master 拉取远程最新master分支代码
```

### git push
```
git push origin master 将本地分支的更新全部推送到远程仓库master分支。
git push origin -d <branchname>   删除远程branchname分支
git push --tags 推送所有标签
```

### git reset
_使用模式：_
![avatar](/picture/git_reset.png)
```
git reset HEAD --file 回退暂存区里的某个文件，回退到当前版本工作区状态
git reset –-soft 目标版本号 可以把版本库上的提交回退到暂存区，修改记录保留
git reset –-mixed 目标版本号 可以把版本库上的提交回退到工作区，修改记录保留
git reset –-hard  可以把版本库上的提交彻底回退，修改的记录全部revert。
```
代码git add到暂存区，并未commit提交，如何回退：
```
git reset HEAD file 取消暂存
git checkout file 撤销修改
```

### git 配置

```
git config --global user.name  "username"
git config --global user.email  "username"  
```

#### 配置密码

```
git config –system –unset credential.helper
git config –global http.emptyAuth true
```

#### Windows凭据管理 git 密码

进入控制面板 -> 用户账号 -> 凭据管理器 -> windows凭据 -> 普通凭据，在里面找到对应git域名，点开编辑密码，更新为最新密码之后就可以正常操作了。


******
参考链接：
[Git Reference](https://git-scm.com/docs) 
[Git Book](https://git-scm.com/book/zh/v2)  



