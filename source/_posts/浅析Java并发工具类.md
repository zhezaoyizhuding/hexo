---
title: 浅析Java并发工具类
date: 2018-03-30 10:59:06
categories: Java源码浅析
tags:
- CountDownLatch
- CyclicBarrier
- Exchanger
- Phaser
- Semaphore
- ThreadLocalRandom
---

在java.util.concurrent包下面有一些并发工具类，本博客通过源码简单介绍下这些并发工具类。

### CountDownLatch

CountDownLatch俗称闭锁，在JDk中的介绍是：它是一个同步辅助类，允许一个或者多个线程等待，直到其他的一系列线程均执行完毕。它采用一个可以指定的count变量来表示其他需要执行完毕的线程数，每当一个线程执行完毕时，可以调用countDown方法使count减一，直到count为0，被await阻塞的线程继续执行。同时在JDk的注释中还指出count无法被重置，如果需要count被重置，则可以考虑使用另一个同步工具类CyclicBarrier。CyclicBarrier之后再说，这里我们通过源码了解一下CountDownLatch的实现。

CountDownLatch只有一个构造函数，也没有其他的静态方法入口，我们要使用它时必须调用这个构造函数。它的源码如下：

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

从这个代码中我们可以看出count不允许小于0，同时它新建了一个Sync类。那么这个Sync类是干什么的呢？通过翻看代码我们发现CountDownLatch只有一个属性，就是Sync类型的。CountDownLatch的所有操作都是委托这个类实现的。下面我们来看看Sync的实现。

```java
/**
 * Synchronization control For CountDownLatch.
 * Uses AQS state to represent count.
 */
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

这个是CountDownLatch的静态内部类，从代码中可以看出它继承于AbstractQueuedSynchronizer，也就是我们常说的AQS。事实上，下面笔者要介绍的一些同步工具很多都是基于AQS实现的。至于AQS的底层实现原理，这里先按下不表，笔者打算之后还写一篇介绍AQS的博客。读者可以先查些其他资料，了解下AQS。否则，这些同步工具类很难理解透彻。

下面我们来看一下CountDownLatch中主要的两个方法await和countDown。await方法用于阻塞当前方法，知道其他所有线程执行完毕。它有两种实现，源码如下：

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

```java
public boolean await(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

第一个方式会一直阻塞直到所有的线程都执行完毕。第二个方法可以指定一个时间，当超出这个时间，还有线程没有执行完毕时，当前线程将不再等待，继续执行。如果这个时间小于或者等于0，则当前线程不会等待。

下面是countDown方法的源码，这个方法在其他线程中调用，用于在线程执行完毕时，使count减一。

```java
public void countDown() {
    sync.releaseShared(1);
}
```

下面看一下CountDownLatch的用法，下面代码摘自CountDownLatch的注释。

```java
class Driver { // ...
    void main() throws InterruptedException {
      CountDownLatch startSignal = new CountDownLatch(1);
      CountDownLatch doneSignal = new CountDownLatch(N);

      for (int i = 0; i < N; ++i) // create and start threads
        new Thread(new Worker(startSignal, doneSignal)).start();

      doSomethingElse();            // don't let run yet
      startSignal.countDown();      // let all threads proceed
      doSomethingElse2();
      doneSignal.await();
      doSomethingElse3();            // wait for all to finish
    }
}

class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;
    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
      this.startSignal = startSignal;
      this.doneSignal = doneSignal;
    }
    public void run() {
      try {
        startSignal.await();
        doWork();
        doneSignal.countDown();
      } catch (InterruptedException ex) {} // return;
    }

    void doWork() { ... }
}
```

我们来分析一下上面这段代码的执行，这里面有两个CountDownLatch，其中startSignal用于阻塞其他子线程的运行，直到主线程调用doSomethingElse()完毕，此时startSignal调用countDown使count为0。此时闸门放开，其他子线程继续执行，与此同时主线程也同时执行doSomethingElse()，并在执行完毕后，等待其他所有子线程执行完毕。CountDownLatch还有一个典型用法是将一个问题，划分成若干个子问题，然后这些子问题分别运行，直到所有子问题处理完毕，才重新运行当前线程。

### CyclicBarrier

CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）,它的功能是让一组线程到达一个屏障（也就叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会打开，所有被屏障拦截的线程继续运行。值得一提的是，所谓的循环是指CyclicBarrier中的parties（相当于CountDownLatch的count）是可以重置的，因此它可以被循环调用。
