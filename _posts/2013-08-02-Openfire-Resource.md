---
layout: post
title: Openfire消息推送指定资源解决方案
category: notes
---

好吧我就不找理由说最近各种忙所以没时间总结了，其实这段时间还是学到不少东西的。主要是学习了XMPP一些核心协议，以及Openfire的改造，编写插件神马的。

除此之外，由于做了一段时间的webim，对js的理解可以说是又进了一步。也接触到一个nb闪闪的名词“闭包”…… （这个“闭包”各种折磨人啊，要调的东西各种调不到，写起来的格式跟java代码似的，要new一个对象出来，然后再对象.方法这样调用……）

对于js，可以说是又爱又恨。由于工作中一直有专业前端并且木有审美，这东东最开始就没有深入学过，只是用的时候查查文档，然后套用各种类库就好了。许多东西都是知其然而不知其所以然，写一个纯js的项目的时候，基础不扎实的问题就立马暴露出来了。

有时间还是要好好学习一下的，毕竟相对于java的庞大，动态语言更能简单直观的做出更炫的效果。

-------------以上是汇报的分割线---------------

现在主要是为了挖个坑，以免下次写总结又是好久以后。下周要研究的问题是openfire消息推送时，只推送到指定资源上去。

* 场景：机器人账号要发送通知给手机端，由于webim上线，用户可能在多个终端同时在线。或者仅web端在线而手机端不在线。openfire默认的策略是没指定资源的情况下，哪个账号在线就推给最近一次活动的资源，指定资源的情况下，如果该资源在线则会推给相应资源。如果用户离线，则会在用户下一次登录的时候（不管哪个资源）直接当离线消息推过去。

* 问题：要求只推送到指定资源上去。

* 思路：还没开始查资料，目前的想法是如果没有现成解决方案的话，去看看message route部分的代码，加个拦截器什么的人为控制一下。

参考资料：

[http://community.igniterealtime.org/thread/38398](http://community.igniterealtime.org/thread/38398)

[RFC6121](http://xmpp.org/rfcs/rfc6121.html#rules-localpart-fulljid)

######消息递送规则的总结

下表总结了本章前面描述的消息（不是节）递送规则。左边的列显示了各种条件（不存在的帐号，没有活跃的资源，只有一个资源并且出席信息优先级为负， 只有一个资源并且出席信息优先级为非负，或多个资源并且每个的出席信息优先级都为非负）和'to'地址（纯JID，匹配一个可用资源的全JID， 或没有匹配的可用资源的全JID）的组合。 随后的列列出了四个主要的消息类型（normal, chat, groupchat, 或 headline） 以及六种可能的递送选项：存储消息到离线 （O），以一个节错误弹回该消息（E），安静地忽略该消息（S），递送该消息到'to'地址中指定的资源 （D），根据服务器实现特有的机制递送该消息到“最可用的”资源或资源们， 例如，认为拥有最高出席信息优先级的资源或资源们是“最可用的”（M）， 或递送该消息给所有拥有非负出席信息优先级的资源 （A -- 这里对于 chat 消息来说 “所有资源” 可能意味着一组显式地被选中接收每个chat消息的资源）。 '/' 字符代表“异或”。该服务器在给一个特定的消息选择哪种动作的时候，应该遵守如下规则：

######表1: 消息递送规则

|条件	|Normal	|Chat	|Groupchat	|Headline|
|帐号不存在 + 纯JID	|S/E	|S/E	|E	|S|
|帐号不存在 + 全JID	|S/E	|S/E	|S/E	|S/E|
|帐号存在但无活跃资源 + 纯JID	|O/E	|O/E	|E	|S|
|帐号存在但无活跃资源 + 全JID(不匹配)	|S/E	|O/E	|S/E	|S/E|
|多个负资源但无非负资源 + 纯JID	|O/E	|O/E	|E	|S|
|多个负资源但无非负资源 + 全JID(匹配)	|D	|D	|D	|D|
|多个负资源但无非负资源 + 全JID(不匹配)	|S/E	|O/E	|S/E	|S/E|
|一个非负资源 + 纯JID	|D	|D	|E	|D|
|一个非负资源 + 全JID(匹配)	|D	|D	|D	|D|
|一个非负资源 + 全JID(不匹配)	|S/E	|D	|S/E	|S/E|
|多个非负资源 + 纯JID	|M/A	|M/A*	|E	|A|
|多个非负资源 + 全JID(匹配)	|D	|D/A*	|D	|D|
|多个非负资源 + 全JID(不匹配)	|S/E	|M/A*	|S/E	|S/E|

对于类型为"chat"的消息，一个服务器不应该（SHOULD NOT）根据选项（A）来动作，除非客户端能显式地选择接收所有的chat消息；无论如何， 选择的方法超出了本协议的范围。

--EOF--