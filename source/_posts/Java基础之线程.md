---
title: Java基础之线程
date: 2017-03-16 09:58:21
categories: Java
tags:
- Java
- 线程
---

### 一 概述

所谓的多线程就是一个程序中存在多个顺序执行流，每个执行流便是一个线程。Java提供了优秀的多线程支持，下面将会详细介绍Java多线程编程的相关知识。

### 二 线程与进程的比较

现在的操作系统基本都支持多进程，即可以同时运行多个任务，每个任务通常是一个程序，每个运行的程序便是一个进程。而一个程序运行时往往有多个顺序执行流，这每个顺序执行流便是一个线程。

线程与进程的区别：

- 线程比进程更轻，并发性更高
- 线程可以共享进程的资源，便于实现相互之间的通信
- 创建线程的代价比创建进程的代价小得多
- java内置了多线程编程支持，可以非常方便的操作线程

并发与并行的区别：

并行是某一时刻同时运行，并发是某一时刻只有一个指令运行，但是多个指令之间切换过快，所以看起来像是同时运行。

### 三 创建线程的三种方式

##### 1.继承Thread类创建线程

使用Thread类创建线程的步骤如下：

1. 定义Thread的子类并重写它的run方法，run方法的方法体里面就是线程需要完成的任务
2. 创建Thread子类的实例，即创建了线程对象
3. 调用线程对象的start()方法来启动线程

代码示例：

抽空写

##### 2.实现Runnable接口创建线程

步骤如下：

1. 实现接口Runnable接口，并实现它的run方法。
2. 创建Runnable接口实现类的实例，并把它作为参数传给Thread的构造函数，创建Thread对象（即将该实例作为Thread类的Target），并调用它的start方法启动线程。（Runnable接口时函数式接口，可使用Lambda表达式）

代码示例：

- 使用Runnable创建线程的一个优势是多个线程之间可以共享线程类中的实例变量。这也是最常用的一种创建线程的方式，但一般都采用匿名内部类的方式简写。

##### 3. 使用Callable和Future创建线程

从java5开始，java提供了一个Callable接口，接口中提供了一个call()方法来替代run()方法，它比run方法的功能更强大。主要有以下两点：

- call方法可以有返回值
- call方法可以声明抛出异常

由于Callable接口没有继承Runnable接口，且call方法有返回值，因此实现Callable接口的类无法作为Target传入Thread类中。为此java5提供了Future接口来包装Callable接口创建线程，该接口有一个FutureTask实现类，该类同时也实现了Runnable接口，因此该类的对象可以作为Target传入Thread类中来创建线程。

创建并启动有返回值的线程的步骤如下：

1. 创建Callable接口的实现类，并实现它的call方法。、
2. 创建FutureTask类来包装Callable接口的实现类的对象（Callable是函数式接口，这一步可以是Lambda表达式）
3. 将FutureTask对象作为Target传入Thread类中，创建线程。
4. 使用FutureTask对象的get方法得到call的返回值

Callab接口实现线程与Runnable接口类似，因此它也有Runnable接口的优势。

##### 4. 创建线程的三种方式对比

使用Runnable和Callable接口创建线程的优势：

- 因为是实现接口，还可以继承其他类，避免由于Java的单继承特性而带来的局限。
- 这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而将CPU，代码和数据分开，更清晰。

劣势：

- 编程稍稍复杂，且要访问当前线程，需要使用Thread.currentThread()方法获取当前线程的实例

使用继承Thread类的方式优势：

- 编程简单

劣势：

因为已经继承了Thread类，无法再继承其他的类。

### 三 线程的生命周期

Java中的线程总结起来可以划分为5个生命周期：

- 新建：创建Thread类或其子类对象
- 就绪（可运行）：调用Thread类的start方法启动线程，此时线程处于可运行状态，但何时运行取决于JVM线程调度器的调度。一个就绪的线程只有获得CPU时间片才可以进入运行状态。
- 运行：获得CPU时间片开始运行。现在的操作系统基本上都是抢占式调度策略（当然也有些小型设备是协作式调度策略），即当前时间片用完，即会被其他线程抢占（取决于优先级）。
- 阻塞：有sleep、join或者IO造成的线程阻塞（调用suspend()也可以使线程挂起，但是该方法容易导致死锁，已不推荐使用）
- 死亡：run结束或者异常。（调用stop方法也可以结束线程，但该方法容易造成死锁，已不推荐使用）

下面是线程状态转换图：

{% asset_img Java线程状态转换图.png Java线程状态转换图 %}

### 四 Java中内置的用于线程控制的方法

常见的有以下几个方法：（详细请查看Thread源码）

- join：加入一个线程，是当前线程（调用join的线程所处的线程）阻塞，直到调用join的线程执行完。
- setDaemon：将一个线程设置为后台线程（守护线程），该方法必须在start方法前调用；isDaemon判断一个线程是否是后台线程。
- sleep：是当前正在执行的线程阻塞
- yield：使当前正在执行的线程进入就绪状态。将机会让给与它优先级相同或者比它高的线程，如其优先级很高，完全有可能被JVM的线程调度器再次调用
- setPriority：设置当前线程的优先级

### 五 线程同步

##### 1. 使用synchronized关键实现同步

- 同步代码块  
使用synchronized对对象进行加锁，如：

```java
synchronized（obj）{
   
   //TODO
   ...
}
```

- 同步方法   
使用synchronized对方法进行加锁

```java
public synchronized void lock（Object obj）{

    //TODO
}
```

synchronized关键字可以对代码块，方法加锁，但不能修饰构造器，成员变量等。

使用synchronized加锁后，线程什么情况下会释放锁：

- 同步方法，代码块执行完毕
- 在同步方法，代码块执行中遇到了break，return
- 出现了未处理的异常和Error
- 调用wait方法

下列情况下不会释放锁：

- 调用sleep，yield方法
- 调用suspend方法

##### 2. 同步锁

从java5开始java提供了接口Lock和ReadWriteLock来实现锁同步。Lock接口有个实现类ReentrantLock，ReadWriteLock有个实现类ReentrantReadWriteLock。使用这两个类可以实现锁同步。如：

```java
public void test(){
   lock.lock();
   try{
      //需要保证线程安全的代码
   }
   finally{
      lock.unlock();
   }
}
```

Java8又新增了一个StampedLock类，在大多数情况下它可以替代ReentrantReadWriteLock。

### 六 线程通信

##### 1.传统的线程通信

使用wait、notify、notifyAll实现线程通信，适用于使用synchronized关键字来保证线程同步的情况下。代码示例：

##### 2.使用Contition实现线程通信

若是使用Lock对象实现线程同步，则无法使用上述的线程通信的方法，java5提供了一个Condition类来实现线程通信。可通过Lock对象的newCondition()方法获得Condition对象，Condition对象中有await，signal，signalAll方法分别对应传统的wait，notify，notifyAll方法，用于实现线程通信。

##### 3.使用阻塞队列实现线程通信

java5提供了一个BlockingQueue接口，它有一个特征：当生产者线程试图向BlockingQueue里放入元素时，若队列已满，则该线程被阻塞；当消费者试图从BlockingQueue里取出元素时，若队列为空，这该线程被阻塞。利用此原理可以实现线程通信。

该接口有几个实现类，可用于构造线程池。

### 七 线程池

线程池即是一个对象池，传入的都是Runnable对象，底层使用数组或者集合来实现。java中内置了一个Executeors工厂类来实现线程池（它还有一个子类ForkjoinPool）,此外还可以使用其他的第三方框架，如ThreadPoolExecutor。

### 八 ThreadLocal类

在处理多线程并发问题上还有一个解决方案，是使用ThreadLocal类将需要并发的资源封装起来，这样当多个线程并发访问这些资源时，ThreadLocal类会为没一个线程均复制一份该资源的副本，从根本上解决了资源的共享冲突问题。但是此方式虽解决了冲突，但多个线程之间也无法共享资源。