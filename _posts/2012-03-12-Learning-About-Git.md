---
layout: post
title: Git基础学习
category: notes
---

总说整理一下git的培训，一直被拖延症困扰。这下好，趁着“被加班”，好好总结整理一下培训成果。嗯，其实总结和整理比盲目的学新东西要重要的多了。

以下摘自罗童鞋培训ppt：

**Git 的特性**

*   直接记录快照，而非差异比较<span style="color: #993300;">（但是由于算法比较精妙，文件很小。。。）</span>
*   去中心化. 近乎所有操作都是本地执行 (离线操作)
*   时刻保持数据完整性(SHA-1 哈希值)
*   多数操作仅添加数据
*   分支功能简单实用
*   快速
*   文件流转的三个工作区域: 工作目录，暂存区域，以及本地仓库<span style="color: #993300;">（这个在eclipse的插件中看不出来，要用命令行才有比较明显的体现。）</span>


**文件的三种状态**

![2011061711422217.png](http://lh3.ggpht.com/-tDcwkCnlg0Q/T1m3Ybn0n3I/AAAAAAAAAT4/D6YGMnAtHwo/2011061711422217.png?imgmax=400)

已提交（committed）– 本地仓库(repository)

已暂存（staged）– 暂存区域(staging area)

已修改（modified）– 工作目录(working directory)


**文件流转区域**

![git-transport.png](http://lh6.ggpht.com/-GmCcLzkdd2E/T1m37IlA29I/AAAAAAAAAUA/j-scJpnEtI8/git-transport.png?imgmax=400)

**文件的流转过程**

1.  在工作目录中修改某些文件。<span style="color: #993300;">(对应状态为已修改，用git status命令查看为modified下列出文件，此时如要保存修改，应add进暂存区)</span>
2.  对修改后的文件进行快照，然后保存到暂存区域。
3.  提交更新，将保存在暂存区域的文件快照永久转储到 Git 目录中。<span style="color: #993300;">(commit命令提交的是暂存区中文件，没有add进暂存区的修改不会被提交)</span>


**Git常用操作：**

流程：取代码 → 每次工作前更新代码到最新版本 → 修改代码 → 提交代码到服务器 <span style="color: #993300;"> (这个应该是要养成的习惯吧，否则很容易造成冲突^^)</span>

_配置git的用户信息_

    $ git config --global user.name
    $ git config --global user.email

此时配置的分别是你个人的用户名称和电子邮件地址, 将在commit log中体现.

–global选项为全局配置, 将会成为默认设置, 在所有git项目中生效. 此时设定信息保存在用户主目录的.gitconfig文件中.

如果只想在单个git项目中配置, 去掉–global即可, 此时设定信息保存在.git/config文件中


_查看配置信息_

要查看已有的git配置信息, 可以使用 git config –list命令，或者查看具体属性值.

    $git config user.name  #查看配置的用户名称

**创建一个版本库：git-init-db**

    $ mkdir gittutorcn
    $ cd gittutorcn
    $ git-init-db

要对现有的某个项目开始用git管理, 只需在项目的顶级目录执行git init

    $git init 或 git init –bare

加上–bare选项时只初始化一个git目录, 不保存项目文件. 通常用来作为中心库.<span style="color: #993300;">(即远程服务器端用于控制大家版本同步的那个库)</span>

**clone一个远程仓库**

    $git clone @:/.git []

如果不指定&lt;projectname&gt;, 默认使用&lt;path&gt;命名

**检查当前文件状态**

要确定哪些文件当前处于什么状态，可以用 git status 命令.

在取得一个git仓库后, 执行git status会看到如下界面:

![git-status.png](http://lh5.ggpht.com/-LsuGZdO0OMw/T12vRk55yPI/AAAAAAAAAUU/Crma0Qucwnw/git-status.png?imgmax=512)

存在以下三种状态

*   Changes to be commited >> 已暂存尚未提交 <<
*   Changes not staged for commit  >>已跟踪的文件修改后尚未暂存 <<
*   Untracked files <span style="color: #993300;">(没有列入版本控制的文件们，通常是在项目里新建了文件就是这个状态。此时对该文件进行的任何操作是不会受版本控制影响的~也就是说项目中的其他同事无法看到，需add进暂存区方可)</span>


**查看提交历史**

_git log查看提交历史_

在提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，可以使用 git log 命令查看。

默认不用任何参数的话，git log 会按提交时间列出所有的更新，最近的更新排在最上面

![git-log.png](http://lh3.ggpht.com/-tFSjH2QwFHM/T12y8Zx0_KI/AAAAAAAAAUs/XdUA2uSMw6I/git-log.png?imgmax=320)

包含以下信息内容:

1.  commit hash(SHA-1 校验和)<span style="color: #993300;"> (这个hash值可以看做此次commit的唯一标示，以后进行reset、checkout的时候可以通过此id定位到具体哪次提交)</span>
2.  作者的名字和电子邮件地址
3.  提交时间
4.  最后缩进一个段落显示提交说明<span style="color: #993300;"> (也就是每次提交时的那个附注信息~应该养成好习惯把提交信息说明清楚，以方便进行版本控制)</span>

使用图形化工具查阅提交历史：

*   git自带的工具:在项目工作目录中输入 gitk 命令
*   eclipse插件使用show history

**Git的基本操作：**

*   git add  将文件加入到暂存区域<span style="color: #993300;"> (只有暂存区的文件才会被提交哦~)</span>
*   git commit  将暂存区域的文件提交到版本库, 加上-a选项则将已跟踪但未暂存的文件也包括在内.<span style="color: #993300;"> (通过-a这个选项可以省去了add这个命令哦~不过未列入版本控制的文件`<Untracked files>`不在此列中哦~)</span>
*   git rm   删除文件. –cached选项只删除索引, 保留文件, 即恢复成untracked状态
