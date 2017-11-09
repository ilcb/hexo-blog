---
layout:_post
title:Git使用
date:2017-07-30 11:33:12
type:"tags"
categories:"工具"
---
## Git简介


​Git是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
​Git是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。

## Git基本概念

在Git中，我们将需要进行版本控制的文件目录叫做一个**仓库（repository）**，每个仓库可以简单理解成一个目录，这个目录里面的所有文件都通过Git来实现版本管理，Git都能跟踪并记录在该目录中发生的所有更新。

现在我们已经知道什么是repository（缩写repo）了，假如我们现在建立一个仓库（repo），那么在建立仓库的这个目录中有一个“.git”的文件夹。这个文件夹非常重要，所有的版本信息，更新记录，以及Git进行仓库管理的相关信息全部保存在这个文件夹里面。所以，不要修改/删除其中的文件，以免造成数据的丢失。
进一步的讲解请参考下面一张图，大概展示出了我们需要了解的基本知识：
![git](/images/git/git.png）
根据上面的图片，下面给出了每个部分的简要说明：

- Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
- WorkSpace：需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。
- .git：存放Git管理信息的目录，初始化仓库的时候自动创建。
- Index/Stage：暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。
- Local Repo：本地仓库，一个存放在本地的版本库；HEAD会只是当前的开发分支（branch）。
- Stash：是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

## Git常用命令

### 新建仓库:git init
![1](/images/git/1.png)
![2](/images/git/2.png)
![3](/images/git/3.png)
### 添加：git add filename
1）在目录下新建text.tx文件
![4](/images/git/4.png)
2）通过"git status"可以查看WorkSpace的状态，看到输出显示"test.txt"没有被Git跟踪，并且提示可以使用"git add <file>..."把该文件添加到待提交区（暂存区）。
**注意，如果添加到暂存区，这时的更新只是在WorkSpace中**
![5](/images/git/5.png)
3）使用"git add test.txt"或者"git add ."，然后继续查看WorkSpace的状态。这时发现文件已经被放到暂存区。
**这时的更新已经从WorkSpace保存到Stage中**
![6](/images/git/6.png)
4）最后通过“git commit -m”来提交更新了。-m后面跟的是对commit的描述（message）。
**这时的更新已经从Stage保存到了Local Repo中。**
![7](/images/git/7.png)
### 更新
1）现在需要对"test.txt"进行更新，修改文件后，查看WorkSpace的状态，会发现提示文件有更新，**但是更新只是在WorkSpace中，没有到暂存区中**
![8](/images/git/8.png)
2）通过add、commit的操作，我们可以把文件的更新先放到暂存区，然后从暂存区提交到repo中
![9](/images/git/9.png)
**注意，只有被add到暂存区的更新才会被提交进入repo。提交前，如果对WorkSpace的文件进行修改，而没有被添加到暂存区，那么提交进repo中的只是暂存区的更新，WorkSpace修改的部分不会提交进repo中的。**

### git diff
git diff用于显示WorkSpace中的文件和暂存区文件的差异。
1）将test.txt文件中的内容xi修改后提交到repo中，内容为：text修改
![10](/images/git/10.png)
2）接下来修改成下面内容，我们通过"git diff"可以查看WorkSpace和Stage的diff情况，当我们把更新add到Stage中，diff就不会有任何输出了。
text修改:
111
![11](/images/git/11.png)
3）也可以把WorkSpace中的状态和repo中的状态进行diff，命令如下:git diff HEAD~n
### 撤销修改
1）根据前面对基本概念的了解，更新可能存在三个地方，WorkSpace中，Stage中和repo中。
#### 撤销WorkSpace中的更新
1）先将text.txt中的文本设置如下内容，并提交。
text修改:
1111
2）然后将text.txt中的内容进行修改
text修改:
1111
2222
![12](/images/git/12.png)
3）可以使用"git checkout <file>..."来撤销WorkSpace中的更新。
`git checkout --text.txt`
![13](/images/git/13.png)
执行命令之后内容变成：
text修改
1111
**注意：使用这种方法撤销更新的时候一定要慎重，因为通过这种方式撤销后，更新将没有办法再找回**
#### 撤销Stage中的更新
1）上面的操作将test.txt修改撤销了。再次修改为下面内容，并且使用了"git add"把这个更新提交到了暂存区。这时，"git status"的输出中提示我们可以通过"git reset HEAD <file>..."把暂存区的更新移出到WorkSpace中。
![14](/images/git/14.png)
#### 撤销repo中的更新
1）介绍撤销repo中的更新之前，我们先看一下"git log"这个命令，通过这个命令可以查看commit的历史记录
![15](/images/git/15.png)
2）撤销提交有两种方式：使用HEAD指针和使用commit id
在Git中，有一个HEAD指针指向当前分支中最新的提交。当前版本，我们使用"HEAD^"，那么再上一个版本可以使用"HEAD^^"，如果想回退到更早的提交，可以使用"HEAD~n"。(也就是HEAD^=
HEAD~1，HEAD^^=HEAD~2)
```
git rest --hard HEAD^
git rest --hard HEAD~1
git rest --c2760c5512bc67a8b990c1da508d40cca623f23
```
![16](/images/git/16.png)
3）再次查看，发现最新的提交已经被撤销了，查看text.txt中的文件，发现内容又变成空白,那么问题就来了，我现在又想恢复被撤销的提交，当然Git是支持这样的操作。
下面来看看"git reflog"这个命令。"git log"只是包括了当前分支中的commit记录，而"git reflog"中会记录这个仓库中所有的分支的所有更新记录，包括已经撤销的更新:
![17](/images/git/17.png)
4）有了这个，我们就可以恢复撤销操作:
```
git reset --hard HEAD@{3}
git reset --hard e92c687
```
![18](/images/git/18.png)
text.txt内容变成:
text修改
1111
5）--hard和--soft
在使用reset来撤销更新的时候，我们都是使用的"--head"选项，其实与之对应的还有一个"--soft"选项，区别如下：
**--head：撤销并删除相应的更新**
**--soft：撤销相应的更新，把这些更新的内容放到Stage中**
### 删除文件
1）在Git中，要删除一个文件，可以使用下面的命令，"git rm"相比"rm"只是多了一步，把这次删除的更新发到Stage中
`rm <file>`
`git rm <file>`
### Git常用命令总结：
![19](/images/git/19.png)
## git clone,push,pull,fetch命令详解
![22](/images/git/22.jpg)
### git clone
1）远程操作的第一步，通常是从远程主机克隆一个版本库，这时就要用到git clone命令:
`$ git clone <版本库的网址>`
比如，克隆jQuery的版本库:
`$ git clone https://github.com/jquery/jquery.git`
2）该命令会在本地主机生成一个目录，与远程主机的版本库同名。如果要指定不同的目录名，可以将目录名作为git clone命令的第二个参数:
`$ git clone <版本库的网址> <本地目录名>`
3）git clone支持多种协议，除了HTTP(s)以外，还支持SSH、Git、本地文件协议等：
```
$ git clone http[s]://example.com/path/to/repo.git/
$ git clone ssh://example.com/path/to/repo.git/
$ git clone git://example.com/path/to/repo.git/
$ git clone /opt/git/project.git 
$ git clone file:///opt/git/project.git
$ git clone ftp[s]://example.com/path/to/repo.git/
$ git clone rsync://example.com/path/to/repo.git/
```
SSH协议还有另一种写法：
` git clone [user@]example.com:path/to/repo.git/`
通常来说，Git协议下载速度最快，SSH协议用于需要用户认证的场合
### git remote
1）为了便于管理，Git要求每个远程主机都必须指定一个主机名。git remote命令就用于管理主机名，不带选项的时候，git remote命令列出所有远程主机:
`$ git remote`
origin
使用-v选项，可以参看远程主机的网址
`$ git remote -v`
```
origin  git@github.com:jquery/jquery.git (fetch)
origin  git@github.com:jquery/jquery.git (push)
```
上面命令表示，当前只有一台远程主机，叫做origin，以及它的网址。
克隆版本库的时候，所使用的远程主机自动被Git命名为origin。如果想用其他的主机名，需要用git clone命令的-o选项指定:
```
$ git clone -o jQuery https://github.com/jquery/jquery.git
$ git remote
```
jQuery
上面命令表示，克隆的时候，指定远程主机叫做jquery。
git remote show 命令加上主机名，可以查看该主机的详细信息。
`$ git remote show <主机名>`
git remote add 命令用于添加远程主机；
`$ git remote add  <主机名> <网址>`
git remote rm 命令用于删除远程主机；
$ git remote rm <主机名>
git remote rename 命令用于远程主机的改名；
$ git remote rename <原主机名> <新主机名>
### git fetch
1）一旦远程主机的版本库有了更新(Git术语叫做commit)，需要将这些更新取回本地，这时就要用到git fetch命令。
`$ git fetch <远程主机名>`
上面命令将某个远程主机的更新，全部取回本地。
git fetch命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。
默认情况下，git fetch取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。
`$ git fetch <远程主机名> <分支名>`
比如，取回origin主机的master分支。
`$ git fetch origin master`
所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如origin主机的master，就
origin/master读取。
git branch命令的-r选项，可以用来查看远程分支，-a选项查看所有分支。
`$ git branch -r`
origin/master
`$ git branch -a`
master
remotes/origin/master
上面命令表示，本地主机的当前分支是master，远程分支是origin/master。
取回远程主机的更新以后，可以在它的基础上，使用git checkout命令创建一个新的分支。
`$ git checkout -b newBrach origin/master`
上面命令表示，在origin/master的基础上，创建一个新分支。
此外，也可以使用git merge命令或者git rebase命令，在本地分支上合并远程分支。
$ git merge origin/master
或者
$ git rebase origin/master
上面命令表示在当前分支上，合并origin/master。
### git pull
1）git pull命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。
`$ git pull <远程主机名> <远程分支名>:<本地分支名>`
比如，取回origin主机的next分支，与本地的master分支合并，需要写成下面这样。
`$ git pull origin next:master`
如果远程分支是与当前分支合并，则冒号后面的部分可以省略。
`$ git pull origin next`
上面命令表示，取回origin/next分支，再与当前分支合并。实质上，这等同于先做git fetch，再做git merge。
```
$ git fetch origin
$ git merge origin/next
```
在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在git clone的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的master分支自动"追踪"origin/master分支。
Git也允许手动建立追踪关系:
`git branch --set-upstream master origin/next`
上面命令指定master分支追踪origin/next分支。
如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名。
`$ git pull origin`
上面命令表示，本地的当前分支自动与对应的origin主机"追踪分支"（remote-tracking branch）进行合并。
如果当前分支只有一个追踪分支，连远程主机名都可以省略。
`$ git pull`
上面命令表示，当前分支自动与唯一一个追踪分支进行合并。
如果合并需要采用rebase模式，可以使用--rebase选项。
`$ git pull --rebase <远程主机名> <远程分支名>:<本地分支名>`
如果远程主机删除了某个分支，默认情况下，git pull 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致git pull不知不觉删除了本地分支。
但是，你可以改变这个行为，加上参数 -p 就会在本地删除远程已经删除的分支。
`$ git pull -p`
等同于下面的命令
```
$ git fetch --prune origin 
$ git fetch -p
```
### git push
git push命令用于将本地分支的更新，推送到远程主机。它的格式与git pull命令相仿。
`$ git push <远程主机名> <本地分支名>:<远程分支名>`
**注意，分支推送顺序的写法是<来源地>:<目的地>，所以git pull是<远程分支>:<本地分支>，而git push <本地分支>:<远程分支>。**
如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。
`$ git push origin master`
上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。
如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。
`$ git push origin :master`
等同于
`$ git push origin --delete master`
上面命令表示删除origin主机的master分支。
如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。
`$ git push origin`
上面命令表示，将当前分支推送到origin主机的对应分支。
如果当前分支只有一个追踪分支，那么主机名都可以省略。
`$ git push`
如果当前分支与多个主机存在追踪关系，则可以使用-u选项指定一个默认主机，这样后面就可以不加任何参数使用git push。
`$ git push -u origin master`
上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。
不带任何参数的git push，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。如果要修改这个设置，可以采用git config命令。
`$ git config --global push.default matching`
或者
`$ git config --global push.default simple`
还有一种情况，就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用--all选项。
` git push --all origin`
上面命令表示，将所有本地分支都推送到origin主机。
如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做git pull合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用--force选项。
`git push --force origin `
上面命令使用--force选项，结果导致远程主机上更新的版本被覆盖。除非你很确定要这样做，否则应该尽量避免使用--force选项。
最后，git push不会推送标签（tag），除非使用--tags选项。
` git push origin --tags`