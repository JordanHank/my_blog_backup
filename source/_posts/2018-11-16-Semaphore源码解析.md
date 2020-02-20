---
title: Semaphore源码解析
date: 2018-11-16 22:23:48
categories: blog
toc: true
tags:
    - java
    - 多线程
    - Semaphore
    - 源码
---
在java 多线程编程中常常需要处理同步问题，处理同步问题时常用的关键字就是synchronized，synchronized的含义就是同步的、互斥的锁，表示同一时刻只能有一个线程能获取执行代码的锁，但是实际情况和应用场景往往是需要多个线程获取锁，并发执行代码，这个时候使用synchronized就不合适了。而java并发工具类中的Semaphore类，就是专门用来处理这种情况的。

<!--more-->

### Semaphore含义
[Semaphore](https://baike.baidu.com/item/semaphore/1322231) 是一种在多线程环境下使用的设施，该设施负责协调各个线程，以保证它们能够正确、合理的使用公共资源的设施，也是操作系统中用于控制进程同步互斥的量。简单来说，Semaphore就是线程允许执行代码的信号量，或者说是许可证，每个acquire方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个release方法增加一个许可证，这可能会释放一个阻塞的acquire方法。然而，其实并没有实际的许可证这个对象，Semaphore只是维持了一个可获得许可证的数量。 

### Semaphore购票示例
```
public class Ticket {

    public static void main(String [] args) {

        List<User> userList = new ArrayList<>(6);
        User user1 = new User("张三");
        userList.add(user1);
        User user2 = new User("李四");
        userList.add(user2);
        User user3 = new User("王五");
        userList.add(user3);
        User user4 = new User("小明");
        userList.add(user4);
        User user5 = new User("小李");
        userList.add(user5);
        User user6 = new User("韩梅梅");
        userList.add(user6);

        //模拟6个用户同时买票
        for (User user: userList) {
           user.start();
        }
    }
}

class User extends Thread{

    // 声明4个窗口窗口
    final static Semaphore windows = new Semaphore(4);

    private String name;

    public User(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        try {
            // 占用窗口
            windows.acquire();
            System.out.println(name + ": 开始买票");
            // 睡2秒，模拟买票流程
            sleep(2000);
            System.out.println(name + ": 购票成功");
            // 释放窗口
            windows.release();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
从上面可以看到，创建了4个购票窗口，6个用户同时购买，每次购买会调用acquire()获取一个许可证，在购买结束后调用release()释放一个许可证，同一时刻最多支持4人买票，直到所有用户购买结束。

运行结果：
![semaphore示例结果](/assets/img/semaphoreResult.png)

### Semaphore构造函数
默认构造函数，只需要指定许可数量，默认非公平模式。
```
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
} 
 ```
指定许可数量和是否为公平模式的构造函数：
```
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```
Semaphore内部基于AQS（AbstractQueuedSynchronizer）的共享模式，所以实现都委托给了Sync类。 看一下NonfairSync的源码可以发现，其实是直接调用了父类Sync类的构造函数。
```
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;

    NonfairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}
```
查看Sync类的源码发现，其实Sync类的构造函数是直接调用了AQS（AbstractQueuedSynchronizer）的setState()方法，也就是AQS中的设置许可数量方法。
```
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 1192457210091910933L;

    Sync(int permits) {
        setState(permits);
    }
    ...
}
```

### 获取许可
Semaphore默认构造函数是使用的非公平模式，所以我们先看一下非公平模式的获取许可方法acquire()源码。
```
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
发现其实是调用的Sync类继承自AQS的acquireSharedInterruptibly()方法。
```
public final void acquireInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```
AQS子类如果要使用共享模式的话，需要实现tryAcquireShared方法，下面看NonfairSync的该方法实现：
```
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        //获取剩余许可数量
        int available = getState();
        //计算给完这次许可数量后的个数
        int remaining = available - acquires;
        //如果许可不够或者可以将许可数量重置的话，返回
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```
从上面可以看到每次获取许可都会重新计算许可剩余数量，当剩余数量为0的时候后面的线程将会阻塞，无法执行。

公平模式的获取许可：
```
protected int tryAcquireShared(int acquires) {
    for (;;) {
        //如果前面有线程再等待，直接返回-1
        if (hasQueuedPredecessors())
            return -1;
        //获取剩余许可数量
        int available = getState();
        //计算给完这次许可数量后的个数
        int remaining = available - acquires;
        //如果许可不够或者可以将许可数量重置的话，返回
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```
可以看出公平模式除了会首先判断当前队列中有没有线程在等待以外，其他和非公平模式一样，所以公平模式有线程已经在等待，那么下一个线程只能进入等待队列，直到上一个线程执行结束。

### 释放许可
查看源码可以知道释放许可，最终是调用的AQS的releaseShared()方法。
```
public void release() {
    sync.releaseShared(1);
}
```
releaseShared方法在AQS中，如下：
```
public final boolean releaseShared(int arg) {
    //判断是否改变许可成功
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
首先会执行tryReleaseShared()方法判断是否改变许可成功：
```
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        //获取当前许可数量
        int current = getState();
        //计算回收后的数量
        int next = current + releases;
        //CAS改变许可数量成功，返回true
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
```
一旦CAS改变许可数量成功，那么就会调用doReleaseShared()方法释放阻塞的线程。
```
 private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

### 查看许可数量
可以通过availablePermits()查看可用许可数量，用于判断是否还有许可可以使用。
```
public int availablePermits() {
    return sync.getPermits();
}
```
除了availablePermits()方法，Semaphore还可以一次将剩余的许可数量全部取走，那就是drainPermits()方法。
```
public int drainPermits() {
    return sync.drainPermits();
}
```
调用了Sync类的实现：
```
final int drainPermits() {
    for (;;) {
        //获取当前许可数量
        int current = getState();
        //许可数量为0直接返回0
        if (current == 0 || compareAndSetState(current, 0))
            return current;
    }
}
```

## 总结
Semaphore就是用于管理许可的信号量，其内部是基于AQS的共享模式，AQS的状态表示许可证的数量，在许可证数量不够时，线程将会被挂起；而一旦有一个线程释放一个资源，那么就有可能重新唤醒等待队列中的线程继续执行。所以应用场景中需要限制获取某种资源的线程数量的时候往往使用Semaphore。