---
layout: _post
title: git 使用
date: 2017-07-31 10：26
tags: 
    - git
categories: 
    - 工具
---

## Git 简介
+ Git 是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
+ Git 是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。

## Git 基本概念
在 Git 中，我们将需要进行版本控制的文件目录叫做一个仓库（repository），每个仓库可以简单理解成一个目录，这个目录里面的所有文件都通过 Git 来实现版本管理，Git 都能跟踪并记录在该目录中发生的所有更新。

假如我们现在建立一个仓库（repo），那么在建立仓库的这个目录中有一个.git”的文件夹。这个文件夹非常重要，所有的版本信息，更新记录，以及 Git 进行仓库管理的相关信息全部保存在这个文件夹里面。所以，不要修改/删除其中的文件，以免造成数据的丢失。
进一步的讲解请参考下面一张图，大概展示出了我们需要了解的基本知识：

![git](git.png)

根据上面的图片，下面给出了每个部分的简要说明：
- Directory：使用 Git 管理的一个目录，也就是一个仓库，包含我们的工作空间和 Git 的管理空间。
- WorkSpace：需要通过 Git 进行版本控制的目录和文件，这些目录和文件组成了工作空间。
- .git：存放 Git 管理信息的目录，初始化仓库的时候自动创建。
- Index/Stage：暂存区，或者叫待提交更新区，在提交进入 repo 之前，我们可以把所有的更新放在暂存区。
- LocalRepo：本地仓库，一个存放在本地的版本库；HEAD 会只是当前的开发分支（branch）。
- Stash：是一个工作状态保存栈，用于保存/恢复 WorkSpace 中的临时状态。

## Git 常用命令
+ git init：新建仓库：
+ git add <filename>：添加文件
+ 通过`git status`可以查看 WorkSpace 的状态。**注意，如果添加到暂存区，这时的更新只是在 WorkSpace 中**
+ git diff：显示 WorkSpace 中的文件和暂存区文件的差异
+ git checkout：撤销 WorkSpace 中的更新
+ git log：查看当前分支中 commit 的历史记录
+ git reset：撤销 Stage 中的更新
  撤销提交有两种方式：使用 HEAD 指针和使用 commit id
  在 Git 中，有一个 HEAD 指针指向当前分支中最新的提交。当前版本，我们使用"HEAD^"，那么再上一个版本可以使用"HEAD^^"，如果想回退到更早的提交，可以使用"HEAD~n"。(也就是 HEAD^=HEAD~1，HEAD^^=HEAD~2)
**--hard 和--soft
在使用 reset 来撤销更新的时候，我们都是使用的"--head"选项，其实与之对应的还有一个"--soft"选项，区别如下：
--hard：撤销并删除相应的更新
--soft：撤销相应的更新，把这些更新的内容放到 Stage 中**

+ git reflog：会记录这个仓库中所有的分支的所有更新记录，包括已经撤销的更新，使用它就可以恢复撤销操作
+ git rm <file>：删除文件
+ git clone <版本库的网址> <本地目录名>：从远程主机克隆一个版本库
比如，克隆 jQuery 的版本库:`git clone https://github.com/jquery/jquery.git`
> gitclone 支持多种协议，除了 HTTP(s)以外，还支持 SSH、Git、本地文件协议等
> git clone http[s]://example.com/path/to/repo.git/
> git clone ssh://example.com/path/to/repo.git/
> git clone git://example.com/path/to/repo.git/
> git clone /opt/git/project.git
> git clone file:///opt/git/project.git
> git clone ftp[s]://example.com/path/to/repo.git/
> git clone rsync://example.com/path/to/repo.git/

+ git remote：用于管理主机名，为了便于管理，Git 要求每个远程主机都必须指定一个主机名。
  1. git remote不带选项的时候，列出所有远程主机
  2. git remote -v ：查看远程主机的网址
    origin git@github.com:jquery/jquery.git(fetch)
    origin git@github.com:jquery/jquery.git(push)
    上面命令表示，当前只有一台远程主机，叫做 origin，以及它的网址```
  3. git 克隆版本库的时候，所使用的远程主机自动被 Git 命名为 origin。如果想用其他的主机名，需要用 git   clone 命令的-o 选项指定:
    git clone -o jQuery https://github.com/jquery/jquery.git
    git remote
    jQuery
    上面命令表示，克隆的时候，指定远程主机叫做 jquery```
  4. git remote show <主机名>：查看主机的详细信息
  5. git remote add <主机名> <网址>：命令用于添加远程主机
  6. git remote rm <主机名>：删除远程主机
  7. git remote rename <原主机名> <新主机名>：远程主机的改名
+ git fetch <远程主机名>：将某个远程主机所有分支（branch）的更新，全部取回本地，gitfetch 命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。
+ git fetch <远程主机名> <分支名>：想取回特定分支的更新，所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如 origin 主机的 master，就origin/master 读取。
+ git branch -r：查看远程分支
+ git branch -a：查看所有分支。
+ git checkout -b <branch>：创建一个新的分支。
+ git merge 命令或者 gitrebase 命令，在本地分支上合并远程分支。
  git merge origin/master：在当前分支上，合并 origin/master
  git rebase origin/master：在当前分支上，合并 origin/master
+ git pull <远程主机名> <远程分支名>:<本地分支名>：取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。
  比如取回 origin 主机的 next 分支，与本地的 master 分支合并，需要写成下面这样。
  git pull origin next:master
+ git push <主机名>：将当前分支推送到主机的对应分支。
  如果当前分支只有一个追踪分支，那么主机名都可以省略：git push
  如果当前分支与多个主机存在追踪关系，则可以使用 -u 选项指定一个默认主机，这样后面就可以不加任何参数使用 git push -u origin master 将本地的 master 分支推送到 origin 主机，同时指定 origin 为默认主机，后面就可以不加任何参数使用 gitpush 了。
  不带任何参数的 git push，默认只推送当前分支，这叫做 simple 方式。此外，还有一种 matching 方式，会推送所有有对应的远程分支的本地分支。Git2.0 版本之前，默认采用 matching 方法，现在改为默认采用 simple 方式。如果要修改这个设置，可以采用 gitconfig 命令。
  git config --global push.default matching 或者 git config --global push.default simple
  还有一种情况，就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用 --all 选项。git push --all origin：将所有本地分支都推送到 origin 主机。
**如果远程主机的版本比本地版本更新，推送时 Git 会报错，要求先在本地做 git pull 合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用 --force 选项，
git push --force origin：结果导致远程主机上更新的版本被覆盖。除非你很确定要这样做，否则应该尽量避免使用 --force 选项。
git push 不会推送标签（tag），除非使用 --tags 选项
git push origin --tags
```

```