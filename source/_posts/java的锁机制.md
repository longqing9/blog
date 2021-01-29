---
title: java的锁机制
date: 2021-01-25 10:11:51
tags: [java,锁机制,CAS算法]
categories: java
---

1、乐观锁和悲观锁

1、乐观锁

> 乐观锁认为在使用数据时，不会有线程修改数据，所以不会添加锁。

乐观锁其实是一种无锁的状态，是通过**CAS算法（对比与交换）**实现的，只是在更新数据时判断是否有线程修改了数据，如果数据没有更新，则将线程数据写入；如果数据被别的线程修改，则进行重试。

2、悲观锁

> 悲观锁认为自己在使用数据时，一定会存在其他的线程在修改数据，所以他会在使用数据前先加锁，等到使用完毕释放锁资源。

在java中synchronized关键字和lock的实现类都属于悲观锁。

3、CAS算法

CAS算法，是一种实现并发算法常用到的技术，CAS具体包含三个参数：当前内存值V，旧的内存值A，即将更新的值B，借助循环，当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

> CAS操作的实现是在native层的`compareAndSwapInt()`中，JNI里是借助于CPU指令`cmpxchg`完成的，该指令是一个原子操作，可以保证变量的可见性。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    //通过循环重试比较新值与旧值，直到两者相等说明此时数据未被其他线程修改，之后更新内存中的变量值
    do {
        var5 = this.getIntVolatile(var1, var2);
    //compareAndSwapInt这个方法是native方法具体分析见底下
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

CAS带来的问题：

- ABA问题，因为CAS算法在更新前是通过检查变量是否可以更新的，如果变量A更新为B然后又被更新为A，那么在检查时就会认为值是没有变化的，但实际上是有变化的，使得线程安全策略变得不可靠。解决，每次更新都记录一个版本号，即1A->2B->3C，这样就可以完美的解决ABA问题了。
- 循环策略导致CPU开销高。

2、公平锁和非公平锁

1、公平锁

> 公平锁，线程按照申请锁的顺序来持有锁。优点是等待的线程不会饥饿。缺点是吞吐率比非公平锁低。

通过ReentrantLock可以公平锁的创建： ReentrantLock reentrantLock = new ReentrantLock(true);

2、非公平锁

> 非 公平锁线程获取锁是无序的，存在线程插队获取锁的情况。优点是吞吐量效率高，因为线程有几率不被阻塞就获取到了锁，但缺点是可能会导致线程一直等待，处于饥饿状态。

常见的非公平锁：synchronized和ReentrantLock；

```java
    private void test(){
        //创建公平锁锁
        ReentrantLock reentrantLock = new ReentrantLock(true);

        //创建非公平锁锁
        ReentrantLock noLock = new ReentrantLock(true);
        
        // 加锁
        reentrantLock.lock();
        // 解锁
        reentrantLock.unlock();
    }
```

3、可重入锁（递归锁）

> 是指同一线程外层函数获得锁之后，内层的递归函数依然能获得该锁的代码。在同一线程的外层函数获取锁的时候，进入到内层函数自动获取锁。即：线程可以进入任何一个它已经拥有的锁所同步的代码锁。

ReentrantLock和synchronized就是一个典型的可重入锁。可重入锁的最大作用是避免死锁。

4、自旋锁

在共享数据锁定的状态下，有很多方法都是只会持有很短的一段时间，为了这么一小段时间而让线程挂起和恢复很不值得。那么jvm就让等待锁的线程稍等一下，但不放弃相应的执行时间。以此看等待的线程是否很快释放，如此就减少了线程调度的压力。如果锁被占用时间很短，这个效果就很好，如果时间过长，就白白浪费了循环的资源，而且会带来资源浪费。

5、独享锁和共享锁

> 独享锁：该锁只能被一个线程所持有；synchronized,ReentrantLock(写锁)，ReentrantLock
>
> 共享锁：该锁可以被多个线程所共同持有；可以保证并发读ReentrantLock(读锁)

```java
ReetrantReadWriteLock readWriteLock = new ReetrantReadWriteLock();
// 获取读锁  加锁
readWriteLock.readLock().lock();
// 读锁解锁 
readWriteLock.readLock().unlock();

// 获取写锁 并加锁
readWriteLock.writeLock().lock();
// 写锁解锁 
readWriteLock.writeLock.unlock();
```

