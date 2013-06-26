---
layout: post
title: JAVA线程池学习以及队列拒绝策略
category: notes
---

工作中遇到了消息队列的发送，之前都是用数据库作为中转和暂存的。这次考虑用多线程的方式进行消息的发送，于是学习了一下线程池的应用。说实话，实践中对Java高级特性的应用真的不多，对多线程的理解也就一直停留在理论层面。借着实践的机会好好整理一下。

准备从以下几个方面总结：

* 线程池的使用

* 消息队列——生产者消费者模式

* 定时任务Quartz原理

* 线程池的大小、队列大小设置

**这个部分是有关线程池的使用：**

**1.  为什么要用线程池？**

在Java中，如果每当一个请求到达就创建一个新线程，开销是相当大的。在实际使用中，每个请求创建新线程的服务器在创建和销毁线程上花费的时间和消耗的系统资源，甚至可能要比花在实际处理实际的用户请求的时间和资源要多的多。除了创建和销毁线程的开销之外，活动的线程也需要消耗系统资源。如果在一个JVM中创建太多的线程，可能会导致系统由于过度消耗内存或者“切换过度”而导致系统资源不足。为了防止资源不足，服务器应用程序需要一些办法来限制任何给定时刻处理的请求数目，尽可能减少创建和销毁线程的次数，特别是一些资源耗费比较大的线程的创建和销毁，尽量利用已有对象来进行服务，这就是“池化资源”技术产生的原因。

线程池主要用来解决线程生命周期开销问题和资源不足问题，通过对多个任务重用线程，线程创建的开销被分摊到多个任务上了，而且由于在请求到达时线程已经存在，所以消除了创建所带来的延迟。这样，就可以立即请求服务，使应用程序响应更快。另外，通过适当的调整线程池中的线程数据可以防止出现资源不足的情况。

**2. ThreadPoolExecutor类**

JDK 1.5以后，Java提供一个线程池ThreadPoolExecutor类。下面从构造函数来分析一下这个线程池的使用方法。

	public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
        BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler);

参数名、说明：

|参数名|说明|
|------|----|
|corePoolSize|线程池维护线程的最少数量|
|maximumPoolSize|线程池维护线程的最大数量|
|keepAliveTime|线程池维护线程所允许的空闲时间|
|workQueue|任务队列，用来存放我们所定义的任务处理线程|
|threadFactory|线程创建工厂|
|handler|线程池对拒绝任务的处理策略|


ThreadPoolExecutor将根据*corePoolSize*和*maximumPoolSize*设置的边界自动调整池大小。当新任务在方法execute(Runnable) 中提交时， 如果运行的线程少于corePoolSize，则创建新线程来处理请求。

如果正在运行的线程等于corePoolSize时，ThreadPoolExecutor优先往队列中添加任务，直到队列满了，并且没有空闲线程时才创建新的线程。如果设置的corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。

*keepAliveTime*：当线程数达到maximumPoolSize时，经过某段时间，发现多出的线程出于空闲状态，就进行线程的回收。keepAliveTime就是线程池内最大的空闲时间。

*workQueue*：当核心线程不能都在处理任务时，新进任务被放在Queue里。

线程池中任务有三种排队策略：

*a. 直接提交。*直接提交策略表示线程池不对任务进行缓存。新进任务直接提交给线程池，当线程池中没有空闲线程时，创建一个新的线程处理此任务。这种策略需要线程池具有无限增长的可能性。实现为：SynchronousQueue

*b. 有界队列。*当线程池中线程达到corePoolSize时，新进任务被放在队列里排队等待处理。有界队列（如ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。
    
*c. 无界队列。*使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

*拒绝策略*：当任务源源不断的过来，而我们的系统又处理不过来的时候，我们要采取的策略是拒绝服务。RejectedExecutionHandler接口提供了拒绝任务处理的自定义方法的机会。在ThreadPoolExecutor中已经包含四种处理策略。

1）CallerRunsPolicy：线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
         if (!e.isShutdown()) {
             r.run();
        }
    }

这个策略显然不想放弃执行任务。但是由于池中已经没有任何资源了，那么就直接使用调用该execute的线程本身来执行。（开始我总不想丢弃任务的执行，但是对某些应用场景来讲，很有可能造成当前线程也被阻塞。如果所有线程都是不能执行的，很可能导致程序没法继续跑了。需要视业务情景而定吧。）

2）AbortPolicy：处理程序遭到拒绝将抛出运行时 RejectedExecutionException

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
          throw new RejectedExecutionException();
    }

这种策略直接抛出异常，丢弃任务。（jdk默认策略，队列满并线程满时直接拒绝添加新任务，并抛出异常，所以说有时候放弃也是一种勇气，为了保证后续任务的正常进行，丢弃一些也是可以接收的，记得做好记录）

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

**3. Executors 工厂**

ThreadPoolExecutor是Executors类的实现，Executors类里面提供了一些静态工厂，生成一些常用的线程池，主要有以下几个：

newSingleThreadExecutor：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

newFixedThreadPool：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。(我用的就是这个，同上所述，相当于创建了相同corePoolSize、maximumPoolSize的线程池)

newCachedThreadPool：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

**4. 举个栗子**

测试类：ThreadPool，创建了一个 newFixedThreadPool，最大线程数为3的固定大小线程池。然后模拟10个任务丢进去。主线程结束后会打印一句：主线程结束。

    public class ThreadPool {
        public static void main(String[] args) {
            //创建固定大小线程池
            ExecutorService executor = Executors.newFixedThreadPool(3);
            for (int i = 0; i < 10; i++) {
                SendNoticeTask task = new SendNoticeTask();
                task.setCount(i);
                executor.execute(task);
            }
            System.out.println("主线程结束");
        }
    }

测试类：SendNoticeTask，执行任务类，就是打印一句当前线程名+第几个任务。为了方便观察，每个线程执行完以后睡10s。

    public class SendNoticeTask implements Runnable {

        private int count;

        public void setCount(int count) {
            this.count = count;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " start to" + " send " + count + " ...");
            try {
                Thread.currentThread().sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("finish " + Thread.currentThread().getName());
        }
    }

执行结果：

    主线程结束
    pool-1-thread-3 start to send 2 ...
    pool-1-thread-1 start to send 0 ...
    pool-1-thread-2 start to send 1 ...
    finish pool-1-thread-3
    finish pool-1-thread-2
    pool-1-thread-2 start to send 3 ...
    finish pool-1-thread-1
    pool-1-thread-3 start to send 4 ...
    pool-1-thread-1 start to send 5 ...
    finish pool-1-thread-3
    finish pool-1-thread-1
    pool-1-thread-1 start to send 6 ...
    pool-1-thread-3 start to send 7 ...
    finish pool-1-thread-2
    pool-1-thread-2 start to send 8 ...
    finish pool-1-thread-1
    pool-1-thread-1 start to send 9 ...
    finish pool-1-thread-3
    finish pool-1-thread-2
    finish pool-1-thread-1

由上可见执行顺序是这样的：线程池创建了三个线程，分别执行任务0、1、2，由于线程创建需要一定时间，因此前三个线程的执行顺序具有一定随机性。此时主线程接着往线程池中塞任务，线程池已达到最大线程数（3），于是开始往队列里放。当某个线程执行完任务后，直接从队列里拉出新的任务执行，队列具有先进先出的特性，因此后面的任务执行是有序的。

这个看一下 Executors 类的源码就更明白了。

    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

实际上是创建了一个具有固定线程数、无界队列的 ThreadPoolExecutor。队列无界，不会拒绝任务提交，因此使用此方法时，需要注意资源被耗尽的情况。

测试类：TestThreadPool，采用有界队列（队列大小2），和默认拒绝策略的ThreadPoolExecutor。

    public class TestThreadPool {

        private static final int corePoolSize = 2;
        private static final int maximumPoolSize = 4;
        private static final int keepAliveTime = 1000;
        private static BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<Runnable>(2);

        public static void main(String[] args) {
            //创建线程池
            ThreadPoolExecutor executor = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime,
                    TimeUnit.MILLISECONDS, workQueue);
            for (int i = 0; i < 10; i++) {
                SendNoticeTask task = new SendNoticeTask();
                task.setCount(i);
                executor.execute(task);
            }
            System.out.println("主线程结束:" + Thread.currentThread().getName());
        }
    }

执行结果：

    pool-1-thread-1 start to send 0 ...
    Exception in thread "main" java.util.concurrent.RejectedExecutionException
        at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:1768)
        at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:767)
        at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:658)
        at com.dylanviviv.pool.TestThreadPool.main(TestThreadPool.java:24)
    pool-1-thread-4 start to send 5 ...
    pool-1-thread-2 start to send 1 ...
    pool-1-thread-3 start to send 4 ...
    finish pool-1-thread-1
    finish pool-1-thread-3
    pool-1-thread-1 start to send 2 ...
    finish pool-1-thread-2
    pool-1-thread-3 start to send 3 ...
    finish pool-1-thread-4
    finish pool-1-thread-3
    finish pool-1-thread-1

线程池创建了2个线程，分别执行任务0、1，线程池达到corePoolSize，新进任务2、3被放入队列中等待处理，此时队列满，而线程池中线程没有执行完任务0、1，线程池创建新的线程，执行新进任务4、5，达到maximumPoolSize。此时所有任务都没有执行结束，主线程又继续提交任务，线程池进入默认异常策略（AbortPolicy）拒绝服务。

--EOF--