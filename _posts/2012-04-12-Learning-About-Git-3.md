---
layout: post
title: Git基础学习（三）
category: notes
---

**Git diff比较版本差异**

Git 比较不同版本文件差异的常用命令格式：

*   git diff &lt;path&gt; &nbsp; 查看尚未暂存的文件更新了哪些部分
*   git diff –cached&nbsp; &lt;path&gt;&nbsp; 查看已暂存的文件和上次提交的版本之间的差异
*   git diff HEAD&nbsp; &lt;path&gt;&nbsp; 查看未暂存的文件和上次提交的版本之间的差异
*   git diff ffd98b b8e7b0 &lt;path&gt;查看某两个版本之间的差异

流程分析:

将 Current working directory 记为 (1)

将 Index file 记为 (2)

将 Git repository 记为 (3)

他们之间的提交层次关系是 (1) -&gt; (2) -&gt; (3)

git add完成的是(1) -&gt; (2)

git commit完成的是(2) -&gt; (3)

git commit -a两者的直接结合

从时间上看，可以认为(1)是最新的代码，(2)比较旧，(3)更旧

按时间排序就是 (1) &lt;- (2) &lt;- (3)

git diff得到的是从(2)到(1)的变化

git diff –cached得到的是从(3)到(2)的变化

git diff HEAD得到的是从(3)到(1)的变化

**文件忽略**

*   要忽略文件，只要把文件名加入到.gitignore文件中就可以了。git会忽略此文件中列出的文件, 不进行跟踪和提示.

    在加入.gitignore之前已跟踪的文件, 需要先在版本库中删除索引(rm –cached).

    支持通配符, 每行一个.
*   如果只是想忽略在当前本地的repository的文件，则可以把文件名字加入到.git/info/exclude就可以了. 内容规则同.gitignore

**GIT文件机制**

在项目根目录中有一个隐藏目录.git, 其中有几个比较重要的文件和目录需要解释一下：

*   HEAD文件存放根节点的信息，其实目录结构就表示一个树型结构，Git采用这种树形结构来存储版本信息，那么HEAD就表示根；
*   refs目录存储了你在当前版本控制目录下的各种不同引用（引用指的是你本地和远程所用到的各个树分支的信息），它有heads、remotes、stash、tags四个子目录，分别存储对不同的根、远程版本库、Git栈和标签的四种引用，你可以通过命令’git show-ref’更清晰地查看引用信息；
*   logs目录根据不同的引用存储了日志信息。
*   因此，Git只需要代码根目录下的这一个.git目录就可以记录完整的版本控制信息

<span style="color: #993300;">由于没用过相当有名的SVN\CVS，之前用的版本控制工具叫&nbsp;ClearCase(CC) (怎么跟同事说起来的时候他们好像都没听过，囧)，然后就是现在的git了~同事说不好用，但好不好用我是没啥评价资格滴，看着原理虽然复杂但是并不难接受~既然用了就应该好好学习，好好总结，对不？</span>

<span style="color: #993300;">一下发了那么多晕菜了，好吧，也许就只有我晕了。本来对着ppt一边敲命令练习，一边把自己的理解加到注释里面。结果一口气吃不出胖子，关于以上部分的就没怎么用心看~其实我觉得那个用eclipse的插件就挺好使的了~</span>

<span style="color: #993300;">嘿嘿~~接下来开始进行分支部分。</span>

<span style="color: #993300;">**GIT分支管理**</span>

<span style="color: #993300;">分支其实就是指向某种代码状态的一个指针。而合并其实就是将两种代码状态合并到另一种代码状态中。</span>

<span style="color: #993300;">在Git中，正确的使用方法中，无处不在使用分支。比如，提交实际上就是本地分支合并到远程分支，更新实际上就是将远程分支合并到本地分支，在开发过程中，每加入一个功能或特性，都加入一个分支，当实验成功后合并到主分支…</span>

[http://www.open-open.com/lib/view/open1328069889514.html](http://www.open-open.com/lib/view/open1328069889514.html)

<span style="color: #993300;">以上这篇文章我觉得讲得挺清楚的，存下来以后看吧~</span>

<span style="color: #993300;">一个很好的分支管理模型：</span>

[http://blog.leezhong.com/translate/2010/10/30/a-successful-git-branch.html](http://blog.leezhong.com/translate/2010/10/30/a-successful-git-branch.html)
