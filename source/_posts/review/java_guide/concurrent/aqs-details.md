---
title: aqs详解
description: aqs详解
categories:
  - 学习
tags:
  - 复习
  - 复习-javaGuide
  - 复习-javaGuide-并发
date: 2022-11-30 14:48:01
updated: 2022-11-30 14:48:01
---

> 转载自https://github.com/Snailclimb/JavaGuide

```Semaphore  [ˈseməfɔː(r)]```

> - 何为 AQS？AQS 原理了解吗？
> - `CountDownLatch` 和 `CyclicBarrier` 了解吗？两者的区别是什么？
> - 用过 `Semaphore` 吗？应用场景了解吗？
> - ......

# AQS简单介绍

AQS,AbstractQueueSyschronizer，即抽象队列同步器，这个类在java.util.concurrent.locks包下面

![image-20221130154309546](https://raw.githubusercontent.com/lwmfjc/lwmfjc.github.io.resource/main/img/image-20221130154309546.png)

AQS是一个抽象类，主要用来构建锁和同步器

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
} 
```

AQS **为构建锁和同步器提供了一些通用功能**的是实现，因此，使用 AQS 能简单且高效地**构造出应用广泛的大量的同步器**，比如我们提到的 **`ReentrantLock`**，**`Semaphore`**，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask`(jdk1.7) 等等皆是基于 AQS 的。

# AQS原理

**面试不是背题，大家一定要加入自己的思想，即使加入不了自己的思想也要保证自己能够通俗的讲出来而不是背出来**

AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 **CLH 队列锁**实现的，即**将暂时获取不到锁的线程加入到队列**中。 

  > CLH(Craig,Landin and Hagersten)队列是一个**虚拟的双向队列**（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是**将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点**（Node）来实现锁的分配。
  > 搜索了一下，CLH好像是人名

- AQS（AbstractQueuedSynchronized）原理图   
    ![image-20221120193141243](https://raw.githubusercontent.com/lwmfjc/lwmfjc.github.io.resource/main/img/image-20221120193141243.png)

  AQS使用一个**int成员变量来表示同步状态**，通过内置的FIFO队列来获取资源线程的排队工作。AQS**使用CAS对同步状态进行原子操作**并实现对其值的修改

  ```java
  private volatile int state;//共享变量，使用volatile修饰保证线程可见性
  ```

  状态信息的操作  

  ```java
  //返回同步状态的当前值
  protected final int getState() {
      return state;
  }
  //设置同步状态的值
  protected final void setState(int newState) {
      state = newState;
  }
  //原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
  protected final boolean compareAndSetState(int expect, int update) {
      return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
  }
  ```

- AQS对资源的共享方式

  - 包括Exclusive（独占）和Share（共享）

  - Exclusive（独占）
    **只有一个线程能执行，如ReentrantLock，又分为公平锁和非公平锁**，Reentrant同时支持两种所，定义：  

    - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁

      ```java
      //例子
          @Test
          public void tt() throws InterruptedException {
              Lock reLock=new ReentrantLock();
              //reLock.lock();
              for(int i=0;i<100;i++){
                  int finalI = i;
                  new Thread(()->{
                      reLock.lock();
                      try {
                          log.info("线程标志"+finalI+"即将停止10s");
                          TimeUnit.SECONDS.sleep(10);
                          log.info("线程标志"+finalI+"停止结束");
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      reLock.unlock();
                  }).start();
                  TimeUnit.SECONDS.sleep(1);
              }
              while (true){}
          }
      /* 结果
      2022-11-30 17:27:31 下午 [Thread: Thread-1] 
      INFO:线程标志0即将停止10s
      2022-11-30 17:27:41 下午 [Thread: Thread-1] 
      INFO:线程标志0停止结束
      2022-11-30 17:27:41 下午 [Thread: Thread-2] 
      INFO:线程标志1即将停止10s
      2022-11-30 17:27:51 下午 [Thread: Thread-2] 
      INFO:线程标志1停止结束
      2022-11-30 17:27:51 下午 [Thread: Thread-3] 
      INFO:线程标志2即将停止10s
      2022-11-30 17:28:01 下午 [Thread: Thread-3] 
      INFO:线程标志2停止结束
      2022-11-30 17:28:01 下午 [Thread: Thread-4] 
      INFO:线程标志3即将停止10s
      2022-11-30 17:28:11 下午 [Thread: Thread-4] 
      INFO:线程标志3停止结束
      2022-11-30 17:28:11 下午 [Thread: Thread-5] 
      INFO:线程标志4即将停止10s
      2022-11-30 17:28:21 下午 [Thread: Thread-5] 
      INFO:线程标志4停止结束
      2022-11-30 17:28:21 下午 [Thread: Thread-6] 
      INFO:线程标志5即将停止10s
      2022-11-30 17:28:31 下午 [Thread: Thread-6] 
      INFO:线程标志5停止结束
      2022-11-30 17:28:31 下午 [Thread: Thread-7] 
      INFO:线程标志6即将停止10s
      2022-11-30 17:28:41 下午 [Thread: Thread-7] 
      INFO:线程标志6停止结束
      2022-11-30 17:28:41 下午 [Thread: Thread-8] 
      INFO:线程标志7即将停止10s
      2022-11-30 17:28:51 下午 [Thread: Thread-8] 
      INFO:线程标志7停止结束
      2022-11-30 17:28:51 下午 [Thread: Thread-9] 
      INFO:线程标志8即将停止10s
      2022-11-30 17:29:01 下午 [Thread: Thread-9] 
      INFO:线程标志8停止结束
      2022-11-30 17:29:01 下午 [Thread: Thread-10] 
      INFO:线程标志9即将停止10s
      2022-11-30 17:29:11 下午 [Thread: Thread-10] 
      INFO:线程标志9停止结束
      2022-11-30 17:29:11 下午 [Thread: Thread-11] 
      INFO:线程标志10即将停止10s
      2022-11-30 17:29:21 下午 [Thread: Thread-11] 
      */
      ```

      

    - 非公平锁：当线程要获取锁时，**先通过两次CAS操作去抢锁**，如果没抢到，当前线程**再加入到队列**中等待唤醒

  - **`ReentrantLock` 中相关的源代码**
    ReentrantLock默认采用非公平锁，考虑获得更好的性能，通过boolean决定是否用公平锁（传入true用公平锁）

    ```java
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;
    public ReentrantLock() {
        // 默认非公平锁
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    } 
    ```

    ReentrantLock中公平锁的lock方法
    

- AQS底层使用了模板方法模式
  使用方式  

  1. 使用者继承AbstractQueueSynchronizer并重写指定方法（**无非是对于共享资源state的获取和释放**）

  2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而**这些模板方法会调用使用者重写的方法**

  3. 自定义同步器时，需要重写下面几个AQS提供的钩子方法

     ```java
     protected boolean tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
     protected boolean tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
     protected int tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
     protected boolean tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
     protected boolean isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。 
     ```

  4. **什么是钩子方法呢？** 钩子方法是一种被声明在抽象类中的方法，它可以是空方法（由子类实现），也可以是默认实现的方法。模板设计模式**通过钩子方法控制固定步骤的实现**。AQS类中除了钩子方法，其他方法都是final

  > 重点：以 `ReentrantLock` 为例，state 初始化为 0，表示未锁定状态。A 线程 `lock()` 时，会调用 `tryAcquire()` 独占该锁并将 `state+1` 。此后，其他线程再 `tryAcquire()` 时就会失败，直到 A 线程 `unlock()` 到 `state=`0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多少次，这样才能保证 state 是能回到零态的。
  >
  > 再以 `CountDownLatch` 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后` countDown()` 一次，state 会 CAS(Compare and Swap) 减 1。等到所有子线程都执行完后(即 `state=0` )，会 `unpark()` 主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后余动作。
  >
  > 一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。 


# Semaphore

# CountDownLatch

# CyclicBarrier
