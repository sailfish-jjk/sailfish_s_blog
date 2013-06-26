---
layout: post
title: Maven + tomcat 报错 ClassNotFoundException
category: problems
---

具体问题描述如下：

[http://www.iteye.com/problems/15791](http://www.iteye.com/problems/15791)

跟这个人出现的问题症状一模一样的，google了也发现了类似的问题，但是大家都是throw exception，木有人回答……没有仔细看问题描述干脆回答不到点子上。不是WEB-INF/lib下少jar包，jar包都在Maven Dependency里，但就是不加载。我遇到的问题是这样解决的：

项目目录下的.tomcatplugin文件里有这么几行：

	<webClassPathEntry>/test/src/main/webapp/classes</webClassPathEntry>
	<webClassPathEntry>/test/target/classes</webClassPathEntry>
	<webClassPathEntry>/test/target/test-classes</webClassPathEntry>


分别是对应的web项目编译完的class路径，问题出现了：/test/src/main/webapp/classes这个路径是不存在的，我在build path里class输出的路径是/test/target/classes。于是把`<webClassPathEntry>/test/src/main/webapp/classes</webClassPathEntry>` 这行去掉，重启tomcat，然后就o了~
