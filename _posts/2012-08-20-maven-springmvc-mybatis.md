---
layout : post
category: notes
title : 学习Maven搭框架ing
---

周五开会果断被忽悠了，决定整理一个 springmvc + mybatis 的框架。当然还是用maven管理，话说这个maven真是很神奇……

当初刚来的时候大牛同事问我之前用什么管理项目，我是各种不懂啊！还很傻很天真的问maven是不是用来管理jar包的……估计同事都瀑布汗了，说你理解到这儿就成了。话说时光匆匆，到现在才了解到maven的冰山一角，赶紧整理一下。

计划一下要做的事情：

* maven 搭建一个 springmvc + mybatis 的框架
* maven 搭建本地私服
* 把已有框架通过archetype生成模版，方便以后自动生成框架 

话不多说，开始：

系统环境 windows8 + eclipse 

1. 建立 maven project：
	new --> project --> maven --> maven project

2. 选择默认工作目录：	
	Use default Workspace location

3. 选择项目类型：(这是一个)
	maven-archetype-webapp

4. 输入groupId & artifactID



Profiles是maven的一个很关键的术语：profile是用来定义一些在build lifecycle中使用的environmental variations，profile可以设置成在不同的环境下激活不同的profile（例如：不同的OS激活不同的profile，不同的JVM激活不同的profile，不同的dabase激活不同的profile等等）。


mvn archetype:create -DgroupId=com.dylanvivi -DartifactId=dylan-archetype -DarchetypeArtifactId=maven-archetype-archetype

{% include references.md %}