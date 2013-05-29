---
layout: post
title: 消息队列——生产者消费者模式
category: notes
---

生产者消费者问题（英语：Producer-consumer problem），也称有限缓冲问题（英语：Bounded-buffer problem），是一个多线程同步问题的经典案例。该问题描述了两个共享固定大小缓冲区的线程——即所谓的“生产者”和“消费者”——在实际运行时会发生的问题。生产者的主要作用是生成一定量的数据放到缓冲区中，然后重复此过程。与此同时，消费者也在缓冲区消耗这些数据。该问题的关键就是要保证生产者不会在缓冲区满时加入数据，消费者也不会在缓冲区中空时消耗数据。

要解决该问题，就必须让生产者在缓冲区满时休眠（要么干脆就放弃数据），等到下次消费者消耗缓冲区中的数据的时候，生产者才能被唤醒，开始往缓冲区添加数据。同样，也可以让消费者在缓冲区空时进入休眠，等到生产者往缓冲区添加数据之后，再唤醒消费者。通常采用进程间通信的方法解决该问题，常用的方法有信号灯法等。如果解决方法不够完善，则容易出现死锁的情况。出现死锁时，两个线程都会陷入休眠，等待对方唤醒自己。该问题也能被推广到多个生产者和消费者的情形。[via WIKI]

讨论如何发消息时，组长就指出我设计的发送这一块儿的逻辑实际上就是典型的“生产者-消费者”模型。好吧，我承认在此之前对于设计模式我只知道一个“工厂模式”。不过就像曾经大牛说过的，很多经典的东西你在不知道的时候就已经用了。

    生产者 -->  缓冲区（队列） --> 消费者

我设计的消息发送的流程是这样的：

![img](https://lh4.googleusercontent.com/-olb30ccL-Fs/UaW6l9DJcHI/AAAAAAAABJI/eIKEDg2LshQ/w1110-h363-no/%25E9%2580%259A%25E7%259F%25A5%25E9%2598%259F%25E5%2588%2597%25E5%258E%259F%25E7%2590%2586+%25283%2529.jpg)

现在知道了，读线程担任着生产者角色，而写线程则承担着消费者的角色。中间的队列（缓冲区）承担着仓库角色。对于消息的处理，生产者生产一条消息，消费者要消费7000条，为保证生产消费的相对平衡和动态，以及资源的合理利用，一个缓冲区是很有必要的。生产者源源不断的读消息，放到缓冲区里，消费者则负责把消息发出去。

这里的关键问题就是队列满时的处理，上文提到的 BlockingQueue 即为一种解决方案。首先作为仓库肯定是唯一的，生产者只能放在这个仓库里，消费者也只能从这个仓库里取数据。当仓库满时生产者不再生产，挂起等待。仓库空时消费者停止消费。

栗子没写，直接贴段网上的：[地址](http://www.oschina.net/code/snippet_113883_12675)

        import java.util.concurrent.BlockingQueue;
        import java.util.concurrent.ExecutorService;
        import java.util.concurrent.Executors;
        import java.util.concurrent.LinkedBlockingQueue;

        /**
         * 阻塞队列BlockingQueue
         * 
         * 下面是用BlockingQueue来实现Producer和Consumer的例子
         */
        public class BlockingQueueTest2 {

            /**
             * 定义装苹果的篮子(仓库类)
             */
            public static class Basket {
                // 篮子，能够容纳3个苹果
                BlockingQueue<String> basket = new LinkedBlockingQueue<String>(3);

                // 生产苹果，放入篮子
                public void produce() throws InterruptedException {
                    // put方法放入一个苹果，若basket满了，等到basket有位置
                    basket.put("An apple");
                }

                // 消费苹果，从篮子中取走
                public String consume() throws InterruptedException {
                    // get方法取出一个苹果，若basket为空，等到basket有苹果为止
                    return basket.take();
                }
            }

            // 　测试方法
            public static void testBasket() {

                // 建立一个装苹果的篮子
                final Basket basket = new Basket();

                // 定义苹果生产者
                class Producer implements Runnable {
                    public String instance = "";

                    public Producer(String a) {
                        instance = a;
                    }

                    public void run() {
                        try {
                            while (true) {
                                // 生产苹果
                                System.out.println("生产者准备生产苹果：" + instance);
                                basket.produce();
                                System.out.println("! 生产者生产苹果完毕：" + instance);
                                // 休眠300ms
                                Thread.sleep(300);
                            }
                        } catch (InterruptedException ex) {
                        }
                    }
                }

                // 定义苹果消费者
                class Consumer implements Runnable {
                    public String instance = "";

                    public Consumer(String a) {
                        instance = a;
                    }

                    public void run() {
                        try {
                            while (true) {
                                // 消费苹果
                                System.out.println("消费者准备消费苹果：" + instance);
                                basket.consume();
                                System.out.println("! 消费者消费苹果完毕：" + instance);
                                // 休眠1000ms
                                Thread.sleep(1000);
                            }
                        } catch (InterruptedException ex) {
                        }
                    }
                }

                ExecutorService service = Executors.newCachedThreadPool();
                Producer producer = new Producer("P1");
                Producer producer2 = new Producer("P2");
                Consumer consumer = new Consumer("C1");
                service.submit(producer);
                service.submit(producer2);
                service.submit(consumer);

                // 程序运行3s后，所有任务停止
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                }

                service.shutdownNow();
            }

            public static void main(String[] args) {
                BlockingQueueTest2.testBasket();
            }
        }


--EOF--