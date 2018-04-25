---
title: 浅析ThreadLocal
date: 2018-04-23 20:31:54
categories: Java源码浅析
tags:
- ThreadLocal
---

在一次面试被问到了ThreadLocal，其间被问到了一个问题：ThreadLocal有什么缺陷。笔者不缺定的回答：内存泄漏？面试官随即问道了：为什么会造成内存泄漏？笔者懵逼了，不知道该如何回答。下面在这片博客中通过源码（JDK1.8）来简单了解一下ThreadLocal的内部实现，好好看一下它为什么会造成内存泄露，以及在什么情况下会造成内存泄露。

### 概要

我们都知道ThreadLcoal可以将一个变量与当前线程绑定，使这个变量变成线程私有，以此来实现线程安全。通常我们会用它来避免多个函数或组件之间公用变量传递的麻烦。下面我们看一下javaDoc中对ThreadLocal的介绍。

```java
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 *
 * <p>For example, the class below generates unique identifiers local to each
 * thread.
 * A thread's id is assigned the first time it invokes {@code ThreadId.get()}
 * and remains unchanged on subsequent calls.
 * <pre>
 * import java.util.concurrent.atomic.AtomicInteger;
 *
 * public class ThreadId {
 *     // Atomic integer containing the next thread ID to be assigned
 *     private static final AtomicInteger nextId = new AtomicInteger(0);
 *
 *     // Thread local variable containing each thread's ID
 *     private static final ThreadLocal&lt;Integer&gt; threadId =
 *         new ThreadLocal&lt;Integer&gt;() {
 *             &#64;Override protected Integer initialValue() {
 *                 return nextId.getAndIncrement();
 *         }
 *     };
 *
 *     // Returns the current thread's unique ID, assigning it if necessary
 *     public static int get() {
 *         return threadId.get();
 *     }
 * }
 * </pre>
 * <p>Each thread holds an implicit reference to its copy of a thread-local
 * variable as long as the thread is alive and the {@code ThreadLocal}
 * instance is accessible; after a thread goes away, all of its copies of
 * thread-local instances are subject to garbage collection (unless other
 * references to these copies exist).
 *
 * @author  Josh Bloch and Doug Lea
 * @since   1.2
 */
```

上面注释对ThreadLocal介绍的还是比较详细的，与我们了解的也差不多。我们主要看下最后一句，它说每个线程都会持有一个它的ThreadLocal变量副本的引用，如果这个线程一直存活，那么这个ThreadLocal实例就是可达的。这句话其实就透露了ThreadLocal内存泄露的原因，这里我们先按下不表，先来看看ThreadLocal的内部实现。

### 源码分析

ThreadLocal只有一个构造方法，同时也没有其他的工厂方法，我们先来看一下它的这个构造方法。

```java
/**
 * Creates a thread local variable.
 * @see #withInitial(java.util.function.Supplier)
 */
public ThreadLocal() {
}
```

可以看到这个构造方法是个空方法，只是用于new一个ThreadLocal对象，并没有进行数据的装配和底层存储结构的构造，这点上和HashMap差不多，采用了一种懒加载的方式，只有在真正在使用的时候才进行相关结构的构造。我么都知道ThreadLocal核心的额有三个方法set，get，和remove方法，那么它底层结构的初始化应该是在调用这些方法时进行的。我们来看看这些方法的源码。

##### set方法

set方法的源码如下：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

我们看到它首先获得了一个当前线程的引用，并把它传递进了一个getMap函数，我们来看看这个getMap函数。

```java
/**
 * Get the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param  t the current thread
 * @return the map
 */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

可见这个方法只是返回一个当前线程持有的ThreadLocalMap对象，这里我们要明白一点，当前线程其实是持有这个ThreadLocalMap的引用的。下面再回到上面的set方法，它先得到当前线程持有的ThreadLocalMap对象，而在刚开始这个map肯定是空的，所以它会调用createMap方法，我们来看看这个方法。

```java
/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

这里面new了一个ThreadLocalMap对象，并把它付给了当前线程持有的一个threadLocals。那么这个ThreadLocalMap是什么呢？它其实是ThreadLocal的静态内部类，ThreadLocal的数据存储和核心操作都是委托给它来实现的。我们来看一下上面这个ThreadLocalMap的构造方法里做了些什么。

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

首先它new了一个Entry类型的数组，初始容量是INITIAL_CAPACITY，这个值默认是16，我们再来看看Entry是什么。源码如下：

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

Entry是ThreadLocalMap的静态内部类，其实就是一个用来封装数据的结构体，类似于map中的桶的概念。这里需要注意的一点是，在这里ThreadLocalMap的key被包裹成了一个弱引用。我们再回到上面的构造方法，可以看到它new了一个数组后，然后对这个key进行了一顿操作，又是hash又是位运算的，但我们只需要知道它最终得到一个索引值作为上面数组的下标，通过这个我们可以索引到具体数据的位置，读者可以想想HashMap，java中的map都是这样干的。下面就是new一个Entry方法上面的数据中并初始化size和threshold。这里面size表示数组中entry的数目，threshold表示ThreadLocalMap扩容的阈值，这里面与HashMap不同的是这个map的负载因子是2/3。

上面这个初始化的问题说完了，下面看看具体的set方法。通过上面的set方法我们知道如果这个ThreadLocalMap没有初始化，调用set方法会初始化它；如果已经初始化了，再调用set方法会委托给ThreadLocalMap的set方法处理。这个set方法的源码如下：

```java
private void set(ThreadLocal<?> key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```
