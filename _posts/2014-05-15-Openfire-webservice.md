---
layout: post
title: Openfire发布webservice接口
category: notes
---

之前碰到有人问怎么把openfire的接口发布成webservice，由于我们项目中采用的是Dubbo形式，还真就没实际操作过。不过估计原理差不多，得空研究一下。

首先还是注册插件：XMPPServer类里的loadModules方法中加入

    //加载数据集成模块。该模块为外部扩展模块，可能会不存在。
    loadModule("com.dylanvivi.openfire.common.DylanModule");

新建xmpp-plugin项目，通过jetty搭建webservice服务。[参考文章](http://blog.csdn.net/robert8803/article/details/8137461)

运行结果：

