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

现在我们已经知道什么是 repository（缩写 repo）了，假如我们现在建立一个仓库（repo），那么在建立仓库的这个目录中有一个“.git”的文件夹。这个文件夹非常重要，所有的版本信息，更新记录，以及 Git 进行仓库管理的相关信息全部保存在这个文件夹里面。所以，不要修改/删除其中的文件，以免造成数据的丢失。

进一步的讲解请参考下面一张图，大概展示出了我们需要了解的基本知识：

![git](/images/git/git.png）

根据上面的图片，下面给出了每个部分的简要说明：

-Directory：使用 Git 管理的一个目录，也就是一个仓库，包含我们的工作空间和 Git 的管理空间。
-WorkSpace：需要通过 Git 进行版本控制的目录和文件，这些目录和文件组成了工作空间。
-.git：存放 Git 管理信息的目录，初始化仓库的时候自动创建。
-Index/Stage：暂存区，或者叫待提交更新区，在提交进入 repo 之前，我们可以把所有的更新放在暂存区。
-LocalRepo：本地仓库，一个存放在本地的版本库；HEAD 会只是当前的开发分支（branch）。
-Stash：是一个工作状态保存栈，用于保存/恢复 WorkSpace 中的临时状态。

Git 常用命令

新建仓库:gitinit







添加：gitaddfilename

1）在目录下新建 text.tx 文件



2）通过"gitstatus"可以查看 WorkSpace 的状态，看到输出显示"test.txt"没有被 Git 跟踪，并且提示可以使用"gitadd<file>..."把该文件添加到待提交区（暂存区）。

注意，如果添加到暂存区，这时的更新只是在 WorkSpace 中



3）使用"gitaddtest.txt"或者"gitadd."，然后继续查看 WorkSpace 的状态。这时发现文件已经被放到暂存区。

这时的更新已经从 WorkSpace 保存到 Stage 中



4）最后通过“gitcommit-m”来提交更新了。-m 后面跟的是对 commit 的描述（message）。

这时的更新已经从 Stage 保存到了 LocalRepo 中。



更新

1）现在需要对"test.txt"进行更新，修改文件后，查看 WorkSpace 的状态，会发现提示文件有更新，但是更新只是在 WorkSpace 中，没有到暂存区中



2）通过 add、commit 的操作，我们可以把文件的更新先放到暂存区，然后从暂存区提交到 repo 中



注意，只有被 add 到暂存区的更新才会被提交进入 repo。提交前，如果对 WorkSpace 的文件进行修改，而没有被添加到暂存区，那么提交进 repo 中的只是暂存区的更新，WorkSpace 修改的部分不会提交进 repo 中的。

gitdiff

gitdiff 用于显示 WorkSpace 中的文件和暂存区文件的差异。

1）将 test.txt 文件中的内容 xi 修改后提交到 repo 中，内容为：text 修改



2）接下来修改成下面内容，我们通过"gitdiff"可以查看 WorkSpace 和 Stage 的 diff 情况，当我们把更新 add 到 Stage 中，diff 就不会有任何输出了。

text 修改:

111



3）也可以把 WorkSpace 中的状态和 repo 中的状态进行 diff，命令如下:gitdiffHEAD~n

撤销修改

1）根据前面对基本概念的了解，更新可能存在三个地方，WorkSpace 中，Stage 中和 repo 中。

撤销 WorkSpace 中的更新

1）先将 text.txt 中的文本设置如下内容，并提交。

text 修改:

1111

2）然后将 text.txt 中的内容进行修改

text 修改:

1111

2222



3）可以使用"gitcheckout<file>..."来撤销 WorkSpace 中的更新。

gitcheckout--text.txt



执行命令之后内容变成：

text 修改

1111

注意：使用这种方法撤销更新的时候一定要慎重，因为通过这种方式撤销后，更新将没有办法再找回

撤销 Stage 中的更新

1）上面的操作将 test.txt 修改撤销了。再次修改为下面内容，并且使用了"gitadd"把这个更新提交到了暂存区。这时，"gitstatus"的输出中提示我们可以通过"gitresetHEAD<file>..."把暂存区的更新移出到 WorkSpace 中。



撤销 repo 中的更新

1）介绍撤销 repo 中的更新之前，我们先看一下"gitlog"这个命令，通过这个命令可以查看 commit 的历史记录



2）撤销提交有两种方式：使用 HEAD 指针和使用 commitid

在 Git 中，有一个 HEAD 指针指向当前分支中最新的提交。当前版本，我们使用"HEAD^"，那么再上一个版本可以使用"HEAD^^"，如果想回退到更早的提交，可以使用"HEAD~n"。(也就是 HEAD^=

HEAD~1，HEAD^^=HEAD~2)

gitrest--hardHEAD^
gitrest--hardHEAD~1
gitrest--c2760c5512bc67a8b990c1da508d40cca623f23



3）再次查看，发现最新的提交已经被撤销了，查看 text.txt 中的文件，发现内容又变成空白,那么问题就来了，我现在又想恢复被撤销的提交，当然 Git 是支持这样的操作。

下面来看看"gitreflog"这个命令。"gitlog"只是包括了当前分支中的 commit 记录，而"gitreflog"中会记录这个仓库中所有的分支的所有更新记录，包括已经撤销的更新:



4）有了这个，我们就可以恢复撤销操作:

gitreset--hardHEAD@{3}
gitreset--harde92c687



text.txt 内容变成:

text 修改

1111

5）--hard 和--soft

在使用 reset 来撤销更新的时候，我们都是使用的"--head"选项，其实与之对应的还有一个"--soft"选项，区别如下：

--head：撤销并删除相应的更新

--soft：撤销相应的更新，把这些更新的内容放到 Stage 中

删除文件

1）在 Git 中，要删除一个文件，可以使用下面的命令，"gitrm"相比"rm"只是多了一步，把这次删除的更新发到 Stage 中

rm<file>

gitrm<file>

Git 常用命令总结：



gitclone,push,pull,fetch 命令详解



gitclone

1）远程操作的第一步，通常是从远程主机克隆一个版本库，这时就要用到 gitclone 命令:

$gitclone<版本库的网址>

比如，克隆 jQuery 的版本库:

$gitclonehttps://github.com/jquery/jquery.git

2）该命令会在本地主机生成一个目录，与远程主机的版本库同名。如果要指定不同的目录名，可以将目录名作为 gitclone 命令的第二个参数:

$gitclone<版本库的网址><本地目录名>

3）gitclone 支持多种协议，除了 HTTP(s)以外，还支持 SSH、Git、本地文件协议等：

$gitclonehttp[s]://example.com/path/to/repo.git/
$gitclonessh://example.com/path/to/repo.git/
$gitclonegit://example.com/path/to/repo.git/
$gitclone/opt/git/project.git
$gitclonefile:///opt/git/project.git
$gitcloneftp[s]://example.com/path/to/repo.git/
$gitclonersync://example.com/path/to/repo.git/

SSH 协议还有另一种写法：

gitclone[user@]example.com:path/to/repo.git/

通常来说，Git 协议下载速度最快，SSH 协议用于需要用户认证的场合

gitremote

1）为了便于管理，Git 要求每个远程主机都必须指定一个主机名。gitremote 命令就用于管理主机名，不带选项的时候，gitremote 命令列出所有远程主机:

$gitremote

origin

使用-v 选项，可以参看远程主机的网址

$gitremote-v

origingit@github.com:jquery/jquery.git(fetch)
origingit@github.com:jquery/jquery.git(push)

上面命令表示，当前只有一台远程主机，叫做 origin，以及它的网址。

克隆版本库的时候，所使用的远程主机自动被 Git 命名为 origin。如果想用其他的主机名，需要用 gitclone 命令的-o 选项指定:

$gitclone-ojQueryhttps://github.com/jquery/jquery.git
$gitremote

jQuery

上面命令表示，克隆的时候，指定远程主机叫做 jquery。

gitremoteshow 命令加上主机名，可以查看该主机的详细信息。

$gitremoteshow<主机名>

gitremoteadd 命令用于添加远程主机；

$gitremoteadd<主机名><网址>

gitremoterm 命令用于删除远程主机；

$gitremoterm<主机名>

gitremoterename 命令用于远程主机的改名；

$gitremoterename<原主机名><新主机名>

gitfetch

1）一旦远程主机的版本库有了更新(Git 术语叫做 commit)，需要将这些更新取回本地，这时就要用到 gitfetch 命令。

$gitfetch<远程主机名>

上面命令将某个远程主机的更新，全部取回本地。

gitfetch 命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。

默认情况下，gitfetch 取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。

$gitfetch<远程主机名><分支名>

比如，取回 origin 主机的 master 分支。

$gitfetchoriginmaster

所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如 origin 主机的 master，就

origin/master 读取。

gitbranch 命令的-r 选项，可以用来查看远程分支，-a 选项查看所有分支。

$gitbranch-r

origin/master

$gitbranch-a

master

remotes/origin/master

上面命令表示，本地主机的当前分支是 master，远程分支是 origin/master。

取回远程主机的更新以后，可以在它的基础上，使用 gitcheckout 命令创建一个新的分支。

$gitcheckout-bnewBrachorigin/master

上面命令表示，在 origin/master 的基础上，创建一个新分支。

此外，也可以使用 gitmerge 命令或者 gitrebase 命令，在本地分支上合并远程分支。

$gitmergeorigin/master

或者

$gitrebaseorigin/master

上面命令表示在当前分支上，合并 origin/master。

gitpull

1）gitpull 命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。

$gitpull<远程主机名><远程分支名>:<本地分支名>

比如，取回 origin 主机的 next 分支，与本地的 master 分支合并，需要写成下面这样。

$gitpulloriginnext:master

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

$gitpulloriginnext

上面命令表示，取回 origin/next 分支，再与当前分支合并。实质上，这等同于先做 gitfetch，再做 gitmerge。

$gitfetchorigin
$gitmergeorigin/next

在某些场合，Git 会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在 gitclone 的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的 master 分支自动"追踪"origin/master 分支。

Git 也允许手动建立追踪关系:

gitbranch--set-upstreammasterorigin/next

上面命令指定 master 分支追踪 origin/next 分支。

如果当前分支与远程分支存在追踪关系，gitpull 就可以省略远程分支名。

$gitpullorigin

上面命令表示，本地的当前分支自动与对应的 origin 主机"追踪分支"（remote-trackingbranch）进行合并。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

$gitpull

上面命令表示，当前分支自动与唯一一个追踪分支进行合并。

如果合并需要采用 rebase 模式，可以使用--rebase 选项。

$gitpull--rebase<远程主机名><远程分支名>:<本地分支名>

如果远程主机删除了某个分支，默认情况下，gitpull 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致 gitpull 不知不觉删除了本地分支。

但是，你可以改变这个行为，加上参数-p 就会在本地删除远程已经删除的分支。

$gitpull-p

等同于下面的命令

$gitfetch--pruneorigin
$gitfetch-p

gitpush

gitpush 命令用于将本地分支的更新，推送到远程主机。它的格式与 gitpull 命令相仿。

$gitpush<远程主机名><本地分支名>:<远程分支名>

注意，分支推送顺序的写法是<来源地>:<目的地>，所以 gitpull 是<远程分支>:<本地分支>，而 gitpush<本地分支>:<远程分支>。

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

$gitpushoriginmaster

上面命令表示，将本地的 master 分支推送到 origin 主机的 master 分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

$gitpushorigin:master

等同于

$gitpushorigin--deletemaster

上面命令表示删除 origin 主机的 master 分支。

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

$gitpushorigin

上面命令表示，将当前分支推送到 origin 主机的对应分支。

如果当前分支只有一个追踪分支，那么主机名都可以省略。

$gitpush

如果当前分支与多个主机存在追踪关系，则可以使用-u 选项指定一个默认主机，这样后面就可以不加任何参数使用 gitpush。

$gitpush-uoriginmaster

上面命令将本地的 master 分支推送到 origin 主机，同时指定 origin 为默认主机，后面就可以不加任何参数使用 gitpush 了。

不带任何参数的 gitpush，默认只推送当前分支，这叫做 simple 方式。此外，还有一种 matching 方式，会推送所有有对应的远程分支的本地分支。Git2.0 版本之前，默认采用 matching 方法，现在改为默认采用 simple 方式。如果要修改这个设置，可以采用 gitconfig 命令。

$gitconfig--globalpush.defaultmatching

或者

$gitconfig--globalpush.defaultsimple

还有一种情况，就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用--all 选项。

gitpush--allorigin

上面命令表示，将所有本地分支都推送到 origin 主机。

如果远程主机的版本比本地版本更新，推送时 Git 会报错，要求先在本地做 gitpull 合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用--force 选项。

gitpush--forceorigin

上面命令使用--force 选项，结果导致远程主机上更新的版本被覆盖。除非你很确定要这样做，否则应该尽量避免使用--force 选项。

最后，gitpush 不会推送标签（tag），除非使用--tags 选项。

gitpushorigin--tags