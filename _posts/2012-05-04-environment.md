---
layout: post
title: 搞坏了配置文件于是重新配环境
category: notes
---

悲催的我为了6G内存换了win8预览版的64位系统，然后就经常各种死机。。。终于在前两天某次死机直接按电源的情况下，配置文件损坏了。。。

oracle也不能用了，每次重配都要各种百度google，主要原因还是配完了就忘。今天本姑娘就把你记下来，下次用的时候也方便查找不是……

首先，使用Oracle的连接插件：instantClient，下载地址如下：[http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html?ssSourceSiteId=ocomen](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html?ssSourceSiteId=ocomen)

操作系统版本有32位和64位之分（想用plsql developer连接的要用32位版本）

解压完了放文件夹里，配置环境变量：


    NLS_LANG : SIMPLIFIED CHINESE_CHINA.ZHS16GBK   --语言选项
    TNS_ADMIN : D:\PTOOL\instantclient_11_2_32\network\admin   --tnsnames.ora文件路径
    PATH : D:\PTOOL\instantclient_11_2_32  --安装路径，都懂的


然后配置plsql。。。Tool -&gt; Preferences

    OracleHome: D:\PTOOL\instantclient_11_2_32
    OciLibrary: D:\PTOOL\instantclient_11_2_32\oci.dll

确定，重启客户端，就好啦~

Tips: plsql连接远程数据库

      sever =
    (DESCRIPTION =
    (ADDRESS_LIST =
    (ADDRESS = (PROTOCOL = TCP)(HOST = HOSTIP)(PORT = 1521))
    )
    (CONNECT_DATA =
    (SERVER = server)
    (SERVICE_NAME = servername)
    )
    )

———————————分割线—————————————

改几个快捷键方便使用 O(∩_∩)O

SQL关键字全部大写

Tools –&gt; Preferences –&gt; Editor –&gt; Keyword Case –&gt; Uppercase

SQL Window中根据光标位置自动选择语句（根据分号分隔语句）

Preferences –&gt; Window Types –&gt; SQL Window –&gt;&nbsp;AutoSelectStatement

自定义快捷键

Tools –&gt; Preferences –&gt; Key Configuration

习惯设置：格式化SQL(format) ctrl+shift+f

执行：ctrl+enter

新建sqlwindow: &nbsp;alt+N
