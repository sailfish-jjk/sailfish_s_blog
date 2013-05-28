---
layout: post
title: Learning About ThreadPool
category: notes
---

工作中遇到了消息队列的发送，之前都是用数据库作为中转和暂存的。这次考虑用多线程的方式进行消息的发送，于是学习了一下线程池的应用。说实话，实践中对Java高级特性的应用真的不多，对多线程的理解也就一直停留在理论层面。借着实践的机会好好整理一下。

准备从以下几个方面总结：

* 线程池的使用

* 消息队列——生产者消费者模式

* 定时任务Quartz原理

* 线程池的大小、队列大小设置

**这个部分是有关线程池的使用：**

1.  为什么要用线程池？

    在Java中，如果每当一个请求到达就创建一个新线程，开销是相当大的。在实际使用中，每个请求创建新线程的服务器在创建和销毁线程上花费的时间和消耗的系统资源，甚至可能要比花在实际处理实际的用户请求的时间和资源要多的多。除了创建和销毁线程的开销之外，活动的线程也需要消耗系统资源。如果在一个JVM中创建太多的线程，可能会导致系统由于过度消耗内存或者“切换过度”而导致系统资源不足。为了防止资源不足，服务器应用程序需要一些办法来限制任何给定时刻处理的请求数目，尽可能减少创建和销毁线程的次数，特别是一些资源耗费比较大的线程的创建和销毁，尽量利用已有对象来进行服务，这就是“池化资源”技术产生的原因。

    线程池主要用来解决线程生命周期开销问题和资源不足问题，通过对多个任务重用线程，线程创建的开销被分摊到多个任务上了，而且由于在请求到达时线程已经存在，所以消除了创建所带来的延迟。这样，就可以立即请求服务，使应用程序响应更快。另外，通过适当的调整线程池中的线程数据可以防止出现资源不足的情况。



2. ThreadPoolExecutor类

	JDK 1.5以后，Java提供一个线程池ThreadPoolExecutor类。下面从构造函数来分析一下这个线程池的使用方法。

		public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)

	参数名|说明|
	------|----|
	corePoolSize|线程池维护线程的最少数量|
	maximumPoolSize|线程池维护线程的最大数量|
	keepAliveTime|线程池维护线程所允许的空闲时间|
	workQueue|任务队列，用来存放我们所定义的任务处理线程|
	threadFactory|线程创建工厂|
	handler|线程池对拒绝任务的处理策略|

    ThreadPoolExecutor 将根据 corePoolSize和 maximumPoolSize 设置的边界自动调整池大小。当新任务在方法execute(Runnable) 中提交时， 如果运行的线程少于 corePoolSize，则创建新线程来处理请求，即使其他辅助线程是空闲的。

    当正在运行的线程等于corePoolSize时，ThreadPoolExecutor 优先往队列中添加任务，直到队列满了，并且没有空闲线程时才创建新的线程。如果设置的corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。

    ThreadPoolExecutor是Executors类的实现，Executors类里面提供了一些静态工厂，生成一些常用的线程池，主要有以下几个：

    newSingleThreadExecutor：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。  

     newFixedThreadPool：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

     newCachedThreadPool：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

    在实际的项目中，我们会使用得到比较多的是newFixedThreadPool，创建固定大小的线程池，但是这个方法在真实的线上环境中还是会有很多问题，这个将会在下面一节中详细讲到。

     当任务源源不断的过来，而我们的系统又处理不过来的时候，我们要采取的策略是拒绝服务。RejectedExecutionHandler接口提供了拒绝任务处理的自定义方法的机会。在ThreadPoolExecutor中已经包含四种处理策略。

      1）CallerRunsPolicy：线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
             if (!e.isShutdown()) {
                 r.run();
            }
        }

	这个策略显然不想放弃执行任务。但是由于池中已经没有任何资源了，那么就直接使用调用该execute的线程本身来执行。

     2）AbortPolicy：处理程序遭到拒绝将抛出运行时 RejectedExecutionException

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
              throw new RejectedExecutionException();
        }

	这种策略直接抛出异常，丢弃任务。

      3）DiscardPolicy：不能执行的任务将被删除

          public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {}

    这种策略和AbortPolicy几乎一样，也是丢弃任务，只不过他不抛出异常。

     4）DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }

      该策略就稍微复杂一些，在pool没有关闭的前提下首先丢掉缓存在队列中的最早的任务，然后重新尝试运行该任务。这个策略需要适当小心。