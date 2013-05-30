---
layout: post
title: Quartz以及线程池大小设置
category: notes
---

不说不知道，总结一下真觉得一个项目里能学到的东西挺多的。可是我能做的还是太少了……T-T…… 

主要想说的是对上面那个消息服务进行压力测试的情况。其实我一直以为这件事情应该是测试童鞋做的，也没什么方法论的，于是……上周末自己开了一个客户端，while(true) 一下，一直给服务器发消息，看看服务器的反应如何。基本上测试数据是一条通知对7000条数据，客户端丢一个消息过去，服务器就转发7000次。设置了下班开始发，第二天早晨6点结束，也就是跑12个小时…… 

话说，走之前我那个小心脏跳的啊…… 从没有做过这样的事，也不知道后果会是啥。晚上睡觉都没睡好，做的梦都是 while(true) 了太多次，直接导致机器爆炸…… 然后第二天早晨6点多自己就醒了…… 阿门，原谅我这个土鳖的不懂编译原理的妹纸吧…… 

事实证明，对于不了解的东西瞎弄是一定会出问题的。这个压力测试果然把测试环境跑挂了。定时器Quartz拒绝添加任务，抛异常，然后就挂了。

其实在测试之前就知道线程池和队列的大小不会设计，会导致问题。大的线程池能提高对CPU的利用率，加快消费者消费，大的队列则可以加快任务的提交速度。然而大能大到多大，没有一个具体的量化的理论指导。

为了解决这个问题，看了Spring Quartz的源码，又了解了一下队列大小和内存之间的关系。

    占用内存 = 队列中对象大小（所有类型占用空间最大值） * 队列大小

例如对象 User 有两个字段，int id 和 String name。int占4个字节，name最大值为5的话，全中国字占10个字节，那么一个size为500的User队列，所占用的最大内存为：14 * 500 = 7000。约6k+。

队列大小可以量化以后，就可以根据部署环境的实际情况进行队列大小的设置了。

**Spring Quartz**

 Quartz 调度器以多线程的方式执行调度任务JobDetail,最大线程池大小为maxPoolSize，也就是说若调度器中已有maxPoolSize个Job在工作（线程没有结束），那么即使有JobDetail到了被触发的时间，新的JobDetail不会被执行，也就是说阻塞的条件是，调度器中正在运行的JobDetail数量达到了设定值maxPoolSize。

        <!-- 线程执行器配置，用于任务注册  -->
        <bean id="executor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
            <property name="corePoolSize" value="6" />
            <property name="maxPoolSize" value="16" />
            <property name="queueCapacity" value="500" />
        </bean>

这个executor里就有上文提到过的threadPoolExecutor，相当于创建了一个大小为500的LinkedBlockingQueue作为任务队列，线程池的线程大小由corePoolSize和maxPoolSize决定。看下代码就明白了。

    public class ThreadPoolTaskExecutor extends ExecutorConfigurationSupport implements SchedulingTaskExecutor {
        private final Object poolSizeMonitor = new Object();
        private int corePoolSize = 1;
        private int maxPoolSize = Integer.MAX_VALUE;
        private int keepAliveSeconds = 60;
        private boolean allowCoreThreadTimeOut = false;
        private int queueCapacity = Integer.MAX_VALUE;
        //这个就是核心的那个线程池
        private ThreadPoolExecutor threadPoolExecutor;
        ...
        /**
        * 初始化线程池
        *
        **/
        protected ExecutorService initializeExecutor(
            ThreadFactory threadFactory, RejectedExecutionHandler rejectedExecutionHandler) {

            BlockingQueue<Runnable> queue = createQueue(this.queueCapacity);
            ThreadPoolExecutor executor  = new ThreadPoolExecutor(
                    this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
                    queue, threadFactory, rejectedExecutionHandler);
            if (this.allowCoreThreadTimeOut) {
                executor.allowCoreThreadTimeOut(true);
            }

            this.threadPoolExecutor = executor;
            return executor;
        }

        /**
        * 初始化队列
        */
        protected BlockingQueue<Runnable> createQueue(int queueCapacity) {
            if (queueCapacity > 0) {
                return new LinkedBlockingQueue<Runnable>(queueCapacity);
            }
            else {
                return new SynchronousQueue<Runnable>();
            }
        }
    }

我的job由于生产者生产速度远大于消费者消费速度，导致任务队列爆满被阻塞。此时生产者还在继续发送消息，于是执行了异常策略，抛出异常。然后又由于很无脑的把线程数开的太大，资源耗尽，测试服务器跑挂……


**举个栗子：**

配置：JobA 触发时间为 每秒运行一次，每个Job执行时间为30秒

运行：

1、 10个JobA将连续启动

2、 到第10个JobA启动后，线程池中所有线程被耗尽，调度器出现了阻塞，即没有新的JobA启动，尽管设置为每秒执行一次。

3、 30秒后，将有1个以上JobA执行完毕，在短时间内，新的10个JobA又被启动，再次进入2的阻塞状态

    2状态可以称做调度器阻塞状态，没有新的Job能执行，导致一些诸如定时读取数据的操作无法继续下去。除非有JobA执行完毕，新的JobA才能被执行。实际运行中，假设调度器中有一个JobA线程的执行时间大于两次启动间隔，则经过若干次操作后，将耗尽所有10个线程资源，导致其他的调度任务阻塞。

后来查了一下，推荐线程数 =  CPU核数 + 1 （计算型任务） 或者 推荐线程数 = CPU核数 * 请求的等待时间（WT）与服务时间（ST）之间的比例（I/O边界型任务）。[地址](http://dongdong1314.blog.51cto.com/389953/225380)

--EOF--