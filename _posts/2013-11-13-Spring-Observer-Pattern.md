---
layout: post
title: Spring 观察者模式
category: notes
---

根据需求，把用户回复事件作为被观察者，而观察者则是存储、匹配关键词、发邮件等具体的业务。

观察者模式的类图：



EventListener为JavaSE提供的事件监听者接口，任何自定义的事件监听者都实现了该接口。Spring中提供了该接口的子类ApplicationListener接口。

EventObject，为JavaSE提供的事件类型基类，任何自定义的事件都继承自该类，例如上图中右侧灰色的各个事件。Spring中提供了该接口的子类ApplicationEvent。

Spring中提供一些Aware相关的接口，BeanFactoryAware、 ApplicationContextAware、ResourceLoaderAware、ServletContextAware等等，其中最常用到的是ApplicationContextAware。实现ApplicationContextAware的Bean，Bean被初始化后，会被注入ApplicationContext的实例。ApplicationContextAware提供了publishEvent()方法，实现Observer(观察者)设计模式的事件传播机，提供了针对Bean的事件传播功能。通过Application.publishEvent方法，用来将事件通知系统内所有的ApplicationListener。

Spring事件处理一般过程：

* 定义Event类，继承org.springframework.context.ApplicationEvent。

* 编写发布事件类Publisher，实现org.springframework.context.ApplicationContextAware接口。

* 覆盖方法setApplicationContext(ApplicationContext applicationContext)和发布方法publish(Object obj)。

* 定义时间监听类EventListener，实现ApplicationListener接口，实现方法onApplicationEvent(ApplicationEvent event)。


代码如下：

######创建发布事件：ChatMsgEvent
	
		/**
		 * 用户上行事件，
		 * 收到用户上行的时候触发本事件。
		 * @author dylan
		 *
		 */
		public class ChatMsgEvent extends ApplicationEvent{
		 
			/**
			 * 
			 */
			private static final long serialVersionUID = 1L;
		 
			/**
			 * 会话消息。
			 * @param msgChat
			 */
			public ChatMsgEvent(MsgChat msgChat) {
				super(msgChat);
			}
		 
		}

上行事件内置MsgChat对象，属性包括上行时间，回复内容等。被发布的事件需要继承 ApplicationEvent 类，事件的类型是区分不同订阅者的重要参数之一。

######事件订阅者，负责具体业务逻辑实现：

		/**
		 * 聊天会话消息事件监听器。
		 * 只处理回复关键词事件……
		 * @author dylan
		 *
		 */
		@Service
		public class ChatMsgListener implements ApplicationListener<ChatMsgEvent> {
		 
			private static Logger logger = Logger.getLogger(ChatMsgListener.class);
		 
			//创建固定为10个的线程池。
			private ExecutorService service = Executors.newFixedThreadPool(10);
		 
			@Autowired
			private MessageService messageService;
		 
			public void setMessageService(MessageService messageService) {
				this.messageService = messageService;
			}
		 
			@Override
			public void onApplicationEvent(ChatMsgEvent event) {
		 
				if (event == null) {
					return;
				}
				//获得事件参数值。
				Object arg = event.getSource();
		 
				//若参数为会话消息，则处理。
				if (arg != null) {
					final MsgChat msgChat = (MsgChat) arg;
					//异步调用远程接口发送消息。
					service.execute(new Runnable() {
		 
						@Override
						public void run() {
							//业务逻辑
						}
					});
				}
		 
			}
		 
		}

可以通过实现 ApplicationListener 来进行监听，并通过泛型确定监听事件。通过泛型指定监听事件的话，就不用 arg instanceof MsgChat 来判断了。

业务逻辑在onApplicationEvent里实现。

当需要扩展功能时，只需新增一个Listener，并监听回复事件即可实现，无需对以前的代码进行修改。

######事件发布：

		//收到上行，并且是真实用户发送的，则触发触发消息发送事件，关联的监听器执行事件更新。
		if (msgChat.getIsFromPublic() != null && msgChat.getIsFromPublic().intValue() == 0) {
			//通知所有监听器处理监听事件,发布事件。
			if (applicationEventPublisher != null) {
				applicationEventPublisher.publishEvent(new ChatMsgEvent(msgChat));
			}
		}

收到上行消息，进行一系列判断后，通过applicationEventPublisher.publishEvent() 来发布消息。Spring发布事件之后，所有注册的事件监听器，都会收到该事件，因此，事件监听器在处理事件时，需要先判断该事件是否是自己关心的，本系统通过泛型确定监听事件的类型。

观察者模式，是一种一对多的关系，即多个观察者监听一个主题。可以很好的解耦业务逻辑，把原来同步的事件变为异步处理，加快了响应速度。并且观察者模式支持广播通讯。被观察者会向所有的登记过的观察者发出通知，更加方便扩展。

PS：好吧，其实没写的东西还挺多，要不是开始写论文了，几乎没时间总结这半年多的学到的东西。本来最初接触观察者模式的时候就想总结一下，结果可恶的拖延症…… 同样被耽搁的还有 maven 的 assembly plugin，用来自定义打包和发布的插件，很好用。以及 JMS ActiveMQ 相关知识点。还有持续集成的应用神马的。一些JS的神奇插件，swfUpload.js、ckeditor.js…… 哈哈，这么一说感觉还学到不少东西。不过不总结和看原理很容易变成一次性知识。

所谓“一次性知识”，就是遇到的时候Google一下，找到解决方法。我觉得判断一个人能力的高低，其中一个因素就是解决问题的能力。面对这种“一次性知识”如果Google不到，怎么样通过分解关键词，找到相关的知识作为下手点，又应该如何设计测试的参数来确定问题出现的原因，逐步找出答案。做完这些，如果加以总结梳理，“一次性知识”就可以变成自己的东西了，如果通过博客或其他方式分享出来，那就更好了。

这些体现了一个人解决问题的思路，也直接锻炼了学习能力、分析问题的能力，如果每次遇到问题都是直接请教大牛，没有经过思考的过程，所学到的东西可能就会少很多了。所以我喜欢先思考，准备出自己的解决方案再去请大家评审哪个更好，或者有什么更好的解决方案。如果实在没思路，就先找大牛给几个关键词，再去想解决方案。

一般来说，学习可以分为主动学习和被动学习，主动学习的过程就是自己拿起一本技术书籍开始各种啃，被动学习则是遇到问题了想办法去解决的过程。主动学习决定了知识的边界，可以更好的帮助我们在被动学习的过程中提取和分析问题。而被动学习则锻炼了解决问题的能力，毕竟生活中遇到问题不都是可以直接获取的。而单凭读书读来的知识，没有应用到实际中，也是空口白话罢了。

杨绛说，年轻的时候以为不读书不足以了解人生，直到后来才发现如果不了解人生，其实是读不懂书的。写程序也是这样的吧，我们读过的，看到过的，都是“知识”，在实际中用过的，才是“知道”。
