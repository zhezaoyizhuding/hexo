---
title: java并发框架浅析
date: 2018-03-12 17:25:57
categories: Java基础知识
tags:
- 并发框架
- 多线程
---

本博客主要介绍JDK5.0后引入的java.util.cocurrent包下的相应并发框架类。并发编程是复杂的，而使用这些并发框架类可以有效降低并发编程的开发难度，提升编码效率与代码健壮性。

### Executor框架

这是一个线程池框架，主要成员包括Executor，Executors，ExecutorService，CompletionService，ThreadPoolExecutor，Future，Callable等。

###### Executor

框架的顶级接口，提供了一个execute方法，接受一个Runnable实例，用来执行一个任务（即继承了Runnable的类）。

###### ExecutorService

继承自Executor，提供了更加丰富的功能，比如shutdown，submit等。

###### Executors

创建线程池的工厂类，提供了一系列创建各类线程池的方法，返回类型均为ExecutorService。

- newSingleThreadExecutor

创建一个单线程化的Executor。

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

- newFixedThreadPool

创建指定线程数量的线程池。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

- newCachedThreadPool

创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有60秒钟未被使用的线程。

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

- newScheduledThreadPool

创建一个支持定时或周期性的任务执行的线程池，多数情况下可用来替代Timer类。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

###### ThreadPoolExecutor自定义线程池

通过查看源码可以发现上面的工厂方法底层都是通过ThreadPoolExecutor实现的。下面详细介绍一个类。

ThreadPoolExecutor继承于AbstractExecutorService，而AbstractExecutorService实现了ExecutorService接口，提供了四个构造方法：

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```

下面介绍一下各个参数的含义：

- corePoolSize：核心线程池大小。在线程池被创建后，池里是没有线程的，只有当有任务进来时才会创建线程去执行任务（当然也可以调用prestartAllCoreThreads去预先创建线程）。而核心线程大小表示当池中线程少于这个值时，每有任务进来，都会创建一个新的线程去执行它；当线程数量等于这个值时，即池里没有空闲线程，此时会把任务暂存在任务队列中。此外，核心线程也表示池中常驻的线程，即当没有需要执行的任务时，这些线程不会被清除。（当然你也可以设置allowCoreThreadTimeOut(true)使核心线程在超出keepAliveTime后被清除，默认是false）

- maxinumPoolSize：池中存在的最大线程数量。当核心线程都在忙碌，并且任务队列也已经满时；此时，再有新的任务进来，线程池就会临时创建线程去执行这个任务。

- keepAliveTime：线程的保活时间，默认只有当池中线程数量超出核心线程数时该参数才会起作用。

- unit：时间单位

- workQueue：任务队列。根据情况的选择，有多种队列，这个下文再介绍。

- threadFactory：线程工厂

- handler：任务拒绝策略。

当有任务到来时，线程池的工作内容如下：

如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，直至线程池中的线程数目不大于corePoolSize；如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被终止。

任务队列workQueue：

- SynchronousQueue：同步队列。它不会保存提交的任务，而是直接将任务交给线程执行，若当前没有空闲线程，则会创建一个新的线程来执行任务。因此使用这个队列时，通常会将maxinumPoolSize设置的非常大。

- ArrayBlockingQueue：有界队列。基于数组，在创建时必须指定大小。

- LinkedBlockingQueue：无界队列。基于链表，如果没有指定大小，默认值为Integer.MAX_VALUE。

任务拒绝策略handler：

当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：

- ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
- ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
- ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
- ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

线程池状态：

```java
// runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

- RUNNING：当创建线程池后，初始时，线程池处于RUNNING状态；此时，能接受新提交的任务，并且也能处理阻塞队列中的任务。
- SHUTDOWN：关闭状态，不再接受新提交的任务，但正在执行的任务和阻塞队列中已保存的任务会继续执行完毕。在线程池处于 RUNNING 状态时，调用 shutdown()方法会使线程池进入到该状态。
- STOP：不再接受新的任务，并且会清除任务队列中任务和尝试中断当前正在运行的任务。在线程池处于 RUNNING 或 SHUTDOWN 状态时，调用 shutdownNow() 方法会使线程池进入到该状态。
- TIDYING：如果所有的任务都已终止了，工作线程为0，任务队列为空，线程池进入该状态，并且之后会调用 terminated() 方法进入TERMINATED 状态。
- TERMINATED：在terminated() 方法执行完后进入该状态，默认terminated()方法中什么也没有做。

###### Future

用于获取线程返回的结果。自从JDK1.5之后，java增加了一个Callable接口，该接口与Runnbale一样，都可用来封装任务，但不同的是Callable有返回值并且可以抛出异常。Future便是被设计出来接收Callable的返回结果的。它有一个实现类FutureTask，该类同时又实现了Runnable。因此它可以作为一个任务传入池中。

###### CompletionService

这是一个批处理任务接口，它有一个实现类ExecutorCompletionService，该类中内置了一个BlockingQueue，用于存放任务执行结果返回的所有Future（实际类型是FutureTask）。可以通过队列的take或者poll方法获取完成的任务。

### 并发集合框架

为了提高在多线程环境下的吞吐率，java从JDK1.5开始增加了很多并发集合框架。下面简单介绍一下这些并发集合类，感兴趣的同学要想进一步了解它们的底层实现机制，可以查看相关资料或者研究一下源码。

###### Map

- ConcurrentHashMap：为了在并发环境下替换HashMap而设计的一个类，在JDK1.8之前采用的是分段锁的机制来实现线程安全性，并保证很好的伸缩性；JDK1.8中对它进行了较大的改变，摒弃了原先的Segment（分段锁）的 概念，利用了CAS算法，采用了一种全新的方式实现。

- ConcrrentSkipListMap：为了在并发环境下替换TreeMap而设计的一个类，底层采用跳表（SkipList）的机制实现而来数据的有序性。

###### Collection

- ConcurrentSkipListSet：用于在并发环境中替换TreeSet。
- CopyOnWriteArrayList：用于在并发环境下替换List，在以遍历为主的情况下具有很好的性能。
- CopyOnWriteArraySet：用于在并发环境下替换Set，底层是通过CopyOnWriteArrayList实现的。

###### Queue

- ArrayBlockingQueue：是一个用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原则对元素进行排序。默认情况下不保证访问者公平的访问队列，所谓公平访问队列是指阻塞的所有生产者线程或消费者线程，当队列可用时，可以按照阻塞的先后顺序访问队列，即先阻塞的生产者线程，可以先往队列里插入元素，先阻塞的消费者线程，可以先从队列里获取元素。通常情况下为了保证公平性会降低吞吐量。
- DelayQueue：是一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。
- LinkedBlockingQueue：是一个用链表实现的有界阻塞队列。此队列的默认和最大长度为Integer.MAX_VALUE。此队列按照先进先出的原则对元素进行排序。
- LinkedTransferQueue：是一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列LinkedTransferQueue多了tryTransfer和transfer方法。
- PriorityBlockingQueue：是一个支持优先级的无界队列。默认情况下元素采取自然顺序排列，也可以通过比较器comparator来指定元素的排序规则。元素按照升序排列。
- SynchronousQueue：是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素。
- LinkedBlockingDeque：是一个由链表结构组成的双向阻塞队列。所谓双向队列指的你可以从队列的两端插入和移出元素。
- ConcurrentLinkedQueue：并发无界队列
- ConcurrentLinkedDeque：并发无界双端队列

### Fork/Join

### 其他框架

###### CountDownLatch
###### CyclicBarrier
###### Exchanger
###### Phaser
###### Semaphore
###### ThreadLocalRandom
