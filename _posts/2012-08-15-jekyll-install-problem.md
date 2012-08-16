---
layout: post
title: github + jekyll 安装遇到问题
category: problems
tags: [github, jekyll]
---

决定把个人博客和技术分开了，放在一块儿乱糟糟的。在这里记录遇到的问题以及解决方案，先从搭建 github + jekyll 环境开始吧。

本地测试需要装[ruby](http://www.ruby-lang.org/en/downloads/)，然后装jekyll：

	$ gem install jekyll

安装Rdiscount，这个是用来解析Markdown标记的解析包。如果你使用Textile的话，就是安装Kramdown。

	$ gem install rdiscount

安装好后启动服务器

	$ jekyll --server --auto

windows用户伤不起啊伤不起，直接报错

> **Liquid error: invalid byte sequence in GBK** 

原因是控制台不支持UTF-8，用英文就没问题……

添加自定义环境变量：

> **LC_ALL=en_US.UTF-8**

> **LANG=en_US.UTF-8**

接下来这个好像是jekyll的bug……
	
> **lib/ruby/gems/1.9.1/gems/jekyll-0.11.2/lib/jekyll/convertible.rb:29:in ‘read_yaml’: invalid byte sequence in GBK (ArgumentError)** 

将 convertible.rb 中 self.content = File.read(File.join(base, name)) 改为 ：

> self.content = File.read(File.join(base, name), :encoding => "utf-8")

如果没问题了的话浏览器输入localhost:4000应该已经可以访问了。

github部分就不多说了，这里用的是[SimpleGray](https://github.com/mytharcher/SimpleGray)这个主题，直接clone下来，修改远程仓库：

	$ git remote rm origin
	$ git remote add origin git@github.com:dylanvivi/dylanvivi.github.com.git
	$ git remote -v # 查看

编写post，名称格式yyyy-MM-dd-title.md。 [Markdown语法](http://wowubuntu.com/markdown/#em)。

剩下的是git提交神马神马的了，问题不大，反正经常用忘不了~就先这样吧~

{% include references.md %}


