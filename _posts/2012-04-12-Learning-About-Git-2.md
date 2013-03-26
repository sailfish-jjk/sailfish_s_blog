---
layout: post
title: Git基础学习（二）
category: notes
---


**远程库操作**

*   提交到远程库

    $git push origin master

orign为默认远程分支

master为默认本地分支

<span style="color: #993300;">(上述语句：把本地master分支push到远程orign分支上，以后版本控制中，应用多分支时应注意分支的切换及合并)</span>

可手动指定, 也可在gitconfig中指定默认远程配置,&nbsp; 即可直接使用git push

*   获取更新

1. git fetch &amp; git merge

git fetch [remote-name] 从远程仓库中拉取所有你本地仓库中还没有的数据.

需要记住，fetch 命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支，只有当你确实准备好了，才能通过执行git merge命令合并

<span style="color: #993300;">(个人觉得这个还是在eclipse插件中操作比较方便，先commit本地修改，然后fetch&amp;merge远程文件，这样可以保证本地修改不会被覆盖~有冲突的时候要手动处理一下冲突~罗童鞋命令行用的很熟，每次看他手指纷飞敲各种命令刷了一屏又一屏的黑白命令行，一种小白仰望大虾的心情就油然而生了~)</span>

2. git pull (= git fetch + git merge)

git pull命令相当于git fetch + git merge, 即从远程仓库中拉取所有你本地仓库中还没有的数据, 并自动合并到当前工作分支.

**处理冲突**

*   希望合并修改内容

git fetch

git merge origin/master&nbsp; &lt;git status -r 即可查看本地对应的remote branch&gt;

a) 修改内容未提交时, 会提示不能merge, 需要先commit或移除文件或恢复修改 <span style="color: #993300;">(这个也是为啥我说在fetch前要先commit的原因~保证本地修改不被覆盖嘛，还有就是不提交git不让merge的~不过这样做的前提一定是在开发前已经更新了最新版本，否则很容易用本地老版本文件覆盖了别人的修改，最近开发中就总遇到这种乌龙事件……)</span>

b) 如果有冲突, 并且不能自动合并, 会提示冲突, 手工编辑解决冲突后, git add标记下, 然后即可git commit <span style="color: #993300;">(此时commit后会自动合并，冲突的文件git会标示出来，开发初期总遇到各种冲突，以至于同事一看到冲突就头大，现在反而少了，一个原因是切了分支，还一个是养成了比较好的开发习惯)</span>

*   希望替换(用远程版本替换本地)

还原本地修改分两种情况

a) 修改未提交: git checkout HEAD &lt;path&gt;<span style="color: #993300;">&nbsp;(在本地修改了代码只是为了测试用，后并不想保留，此情况直接checkout就好了)</span>

b) 修改已提交: git reset –hard HEAD 还原至上一个提交<span style="color: #993300;"> (本地修改了某个类，然后发现改错了……还原至上一次提交)</span>

**撤销操作**

*   修复未提交文件中的错误(重置)

如果你现在的工作目录(work tree)里搞的一团乱麻, 但是你现在还没有把它们提交; 你可以通过下面的命令, 让工作目录回到上次提交时的状态(last committed state):

    $ git reset --hard HEAD


这条件命令会把你工作目录中所有未提交的内容清空(当然这不包括未置于版控制下的文件 untracked files)

也可用git checkout HEAD &lt;path&gt;来恢复文件或目录, 命令将从HEAD中签出并且把它恢复成未修改时的样子

git checkout — &lt;path&gt;是从暂存区签出内容并恢复. 注意区别. <span style="color: #993300;">&lt;&lt;这句很关键</span>

<span style="color: #993300;">(此处这个HEAD为每次提交的SHA值，也就是说，想要确定还原到某次提交，用该提交的SHA值替代HAED即可)</span>

*   修复已提交文件中的错误

如果你已经做了一个提交(commit),但是你马上后悔了, 这里有两种截然不同的方法去处理这个问题:

1.撤销旧的提交 (会丢失此次提交中的修改内容, 但当前工作区的修改仍保留)

  $ git revert HEAD

2. 修改旧提交 (不能修改已提交文件的内容, 只能加入新的提交文件或修改提交注释)

如果你刚刚做了某个提交(commit), 但是你又想马上修改这个提交; git commit 现在支持一个叫–amend的参数，它能让你修改刚才的这个提交(HEAD commit). 这项机制能让你在代码发布前,添加一些新的文件或是修改你的提交注释(commit message).

**※Git — reset篇**

意义:

是要reset到指定commit位置的状态<span style="color: #993300;">.(reset到，这个“到”字很关键，要着重理解~)</span>

*   可以做到只消去commit 但不消去已更动的file内容
*   也可以同时消去commit和file的内容
*   可以从staging area拿回来，就是说add之后还可以取消

用法:

git reset 前一个SHA值(不要reset你要rollback的那个commit，是没用的)

git reset –soft HEAD^: 退到上上一个commit的状态, stage和working tree的修改都会保留<span style="color: #993300;"> (学过指针就知道^是指针前移，同理，^^就是指针前移两位~)</span>

git reset –mixed: 只保留working tree的修改，而stage和commit都会回退

git reset –hard: 不只退回commit, 连working tree的档案内容都会退回上一个commit <span style="color: #993300;">(所以可以用来清理工作区)</span>

git reset &lt;commit&gt; — &lt;path&gt;: 从指定commit中恢复指定&lt;path&gt;到暂存区, 本地修改不丢失

TIP：

*   根据–soft –mixed –hard，会对working tree和index和HEAD进行重置。
*   git reset [&lt;commit&gt;] — [&lt;path&gt;]可以对单个文件或目录使用, 恢复指定path在HEAD的内容. working tree的内容不变.
*   git reset无法在push以后作. 不管如何已经push出去的就不能取消了. 所以已经git push的就只能用git revert

![git-reset.png](http://lh3.ggpht.com/-bdbJqUhe6Xo/T124YiLLRYI/AAAAAAAAAU4/n_sDUjMU-Y4/git-reset.png?imgmax=512)

<span style="color: #993300;">上图很好理解reset的含义，被reset的commit就不存在了~</span>

思考:

    git reset –soft HEAD

为什么是无意义的操作?

<span style="color: #993300;">(因为soft只回退到指定commit，暂存区和工作区都不变，而HEAD就是上一次的commit，此次commit还没有发生，回退操作不会发生任何改变)</span>

**※Git — revert篇**

意义:

file rollback 并且自动产生一个roll back的commit (所以commit会往前不会退后)

<span style="color: #993300;">(用上图理解就是，在G处执行了revert C命令后，会产生一个新的commit节点H，而H和C的内容是一样的)</span>

用法:

*   git revert HEAD: 将这个commit的file内容回到前一次commit，会产生一个新的revert commit log
*   git revert HEAD^: 承上，回到前前一个的commit
*   git revert &lt;commit&gt;: 取消指定commit的提交，内容会变成此commit之前的状态，一样产生一个新的revert commit log
*   加上-n选项后, 可以避免强制提交.

TIP:

*   这个命令要求工作区是干净的(与指定commit中的文件无冲突)
*   工作区相对于指定commit新增文件的修改在revert后仍保留

**思考题 :修改指定commit中的指定文件**

Q:在一次commit中提交了a和b两个文件的修改, 但是实际上只应该提交对于a的修改，b的修改希望留在本地，以后再提交

A:

1. git reset. 但是已经PUSH的不能恢复

$git reset &lt;commit&gt; — &lt;path&gt;

从指定commit中恢复指定&lt;path&gt;到暂存区, 同时本地修改不会丢失.

这样再次commit之后HEAD中保存的内容实际上是暂存区已恢复的内容.

2. git revert, 这样做会导致b的本地修改丢失.

1).$git revert -n &lt;commit&gt;

2).$ git reset HEAD a.txt

3).$ git checkout — a.txt

思考: 这两种方法为什么能实现这样的结果.
