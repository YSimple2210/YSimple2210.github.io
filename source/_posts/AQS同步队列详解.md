---
title: AQS同步队列详解
date: 2019-12-16 15:00:01
tags: [AQS]
categories: -Java
---



# AQS同步队列

## 同步锁的本质 - 排队

>   -   同步的方式：独享锁-单个队列窗口 、共享锁-多个队列窗口
>   -   抢锁的方式：插队锁（不公平锁） 、先来后到抢锁（公平锁）
>   -   没抢到锁的处理方式：快速尝试多次（CAS自旋锁）、阻塞等待
>   -   唤醒阻塞线程的方式（叫号器）：全部通知、通知下一个

## AQS抽象队列同步器

>   提供了对资源占用、释放、线程的等待、唤醒等等接口和具体实现
>
>   可以用在各种需要控制资源争用的场景中。（ReentrantLock/CountDownLatch/Semphore）

<img src="">

| 定义                         | 描述                                                   |
| ---------------------------- | ------------------------------------------------------ |
| acquire、acquireShared       | 定义了资源争用的逻辑，如果没有拿到，则等待             |
| tryAcquire、tryAcquireShared | 实际执行占用资源的操作、如何判定一个由使用者具体去实现 |
| release、releaseShared       | 定义释放资源的逻辑、释放之后，通知后续节点进行争抢     |
| tryRelease、tryReleaseShared | 实际执行资源释放的操作，具体有AQS使用者去实现          |

# Semaphore

## 介绍

>   称之为“信号量”，控制多个线程争抢许可。
>
>   -   acquire ：获取一个许可。如果没有就等待
>   -   release  ：释放一个许可
>   -   availablePermits：方法得到可用的许可数目

## 应用场景

>   1.  代码并发处理限流

```java
public class SemphoreTest {
    public static void main(String[] args) {
        SemphoreTest semphoreTest = new SemphoreTest();
        Semaphore semaphore =new Semaphore(5); // 最大只有5个可以进来
        for (int i = 0; i < 8; i++) {
            String vipNo = "vip" + i;
            new Thread(()->{
                   try {
                       semaphore.acquire();// 获得许可
                       semphoreTest.service(vipNo);
                       semaphore.release();
                   }catch (Exception e){
                   }
            }).start();
        }
    }
    public void service(String vipNo) throws Exception{
        System.out.println("进来一位贵宾：" + vipNo);
        Thread.sleep(new Random().nextInt(3000));
        System.out.println("欢送贵宾：" + vipNo);
    }
}
```

# CountDownLatch

## 介绍

>倒计数器
>
>创建对象时,传入指定数值作为线程参与的数量
>
>await：方法等待计数器值变为0，在这之前，线程进入等待状态
>
>countdown：计算器数值减一，直到为0
>
>经常用于等待其它线程执行到某一节点，再继续执行当前线程代码

## 应用场景

>1.  统计线程执行的情况
>2.  压力测试中，使用countDownLatch实现最大程度的并发处理
>3.  多个线程之间，相互通信，比如线程异步调用完接口，结果告知。

```java
CountDownLatch countDownLatch = new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(() ->{
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("我是" + Thread.currentThread() + ".我执行了接口-" + finalI + "调用了");
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("执行最后一次");
```

# CyclicBarrier

## 介绍

>   称为“线程栅栏”
>
>   创建对象时候，指定栅栏线程数量
>
>   await：等指定数量的线程都处于等待状态时，继续执行后续代码
>
>   barrierAction：线程数量到了指定量之后，自动触发执行指定任务

>   和countDownLatch的区别在于，CyclicBarrier对象可以多次执行触发

```java
 LinkedBlockingQueue<String> sqls = new LinkedBlockingQueue<>();
        CyclicBarrier barrier = new CyclicBarrier(4,() -> {
            System.out.println("有4个线程执行了： " + Thread.currentThread());
            for (String sql : sqls) {
                System.out.println(sqls.poll());;
            }
        });
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                sqls.add("data - " + Thread.currentThread() );
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println("插入完毕");
            }).start();
        }
```

