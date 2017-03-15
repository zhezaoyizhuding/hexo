---
title: Java基础之集合
date: 2017-03-13 16:42:33
categories: Java
tags:
- Java
- List
- Map
- Set
- 集合
---

### 一 概述

为了保存数量不确定的数据，以及具有映射关心的数据（也被称为关联数组），java从1.2开始提供了集合类。集合与数组不同，数组必须保存有确定数目的数据，且数组即可以保存基本数据也可以保存对象，而集合只能保存对象。集合类主要负责保存，盛装其他数据，因此集合类也被称作容器类。所有的集合都保存在java.util包下，从java1.5开始还在java.util.concurrent下提供了一下线程安全的集合类。

java中的集合类主要由两个接口派生而出：Collection和Map，它们是java集合框架的根接口。其继承结构如下：

```java
Collection
├List
│├LinkedList
│├ArrayList
│└Vector
│　└Stack
├Queue
└Set
Map
├Hashtable
├HashMap
└WeakHashMap
```

### Collection集合

Collection是List，Set，Queue的父接口，因此在Collection中定义的接口在List，Set，Queue中均可使用。对集合的操作除了添加和删除之外，还有一个重要的操作就是遍历集合。下面来看一下集合中多种多样的遍历方法：

##### 1.使用Iterator遍历集合（推荐方法）

使用Iterator遍历集合是集合遍历的传统方法，也是推荐方法。Iterator也是java集合框架中的一员，但是它不是用来盛装数据，它的作用主要就是用来遍历（迭代访问）Collection中的元素。我们可以通过Collection下的iterator()来获得这个集合的迭代器对象，并使用它的方法对集合进行遍历。Iterator中包含以下方法:

- boolean hasNext()   
判断集合中的元素是否已遍历完，返回true，则还没遍历完
- E next()   
获取集合中的下一个元素
- void remove()  
删除集合中上一次next方法返回的元素
- void forEachRemaining(Consumer action)   
Java8新增的方法，可使用Lambda表达式遍历集合

Iterator遍历集合代码示例：



##### 2.使用foreach循环迭代集合

这是Java5新增的用于变异集合和数组的循环。

foreach遍历集合代码示例：



##### 3.使用Lambda表达式遍历集合

Java从Java8开始新增Lambda表达式，在很多方面都简化了操作。在遍历集合方面就可以使用Lambda表达式，如：

- forEach(Consumer action)   
Java8在Iterable接口中增加了一个forEach(Consumer action)方法，该方法的参数类型是一个函数式接口。而Iterable接口时Collection接口的父接口，因此可使用该方法遍历集合。

- forEachRemaining(Consumer action)   
Java8中为Iterator新增的方法，可用于遍历集合。

同时我们还可以用Lambda表达式对集合进行其他的操作，比如：

Java8中为Collection新增了一个removeIf（Predicate filter）方法，Predicate也是函数式接口，可用它来过滤集合。

Java8中新增了Stream，IntStream，LongStream，DoubleStream等流式API，这些API中有大量的聚集函数，如求和，去平均值等。而Collection中亦提供了一个stream()，可返回该集合对应的流，然后再通过这些流式API操作集合。

### 二 List

List是一个有序的，可重复的集合。List默认以元素的添加顺序来设置元素的索引（从0开始），因此可以通过索引来查找元素，也因此List允许重复的元素（不会无法分辨这两个元素）。

由于存在索引，List可以通过普通的for来迭代，同事List继承于Collection，可以使用Collection中的方法，同事它还有一个额外的方法listIterator用于List的迭代。

##### 1. ArrayList与Vector

ArrayList和Vector均是基于数组实现的List集合类，皆继承于List。它们的内部封装了一个动态的，允许再分配的Object[]数组，并使用initialCapacity参数来设置数组的长度（initialCapacity参数可以随着元素的增多而自动增长，默认是10），可通过ensureCapacity（int minCapacity）来一次性设置数组的initialCapacity值，减少重新分配内存的次数，提高性能。

Vector是一种古老的集合类，里面有一些命名很长的方法，现在已不推荐使用。它与ArrayList不同的是，它是线程安全的，而ArrayList不是。

Vector还有一个子类Stack，用于模拟栈这种数据结构，它也是线程安全的。但Stack也是一个古老的类，现已不推荐使用，要模拟栈可使用下文中ArrayDeque。

- 固定长度的ArrayList   
Java中有一个操作数组的工具类：Arrays，这个类中有一个方法asList可以将数组转换为一个List集合，但是这个ArrayList与上述介绍的不同，它不是继承于List，而是Arrays的内部类，这个集合只能遍历，而不能插入，删除。因此不推荐使用该方法将一个数组转换为集合。

##### 2. LinkedList

LinkedList也是List的实现类，但它同时还实现了Deque接口，提供额外的get，remove，insert方法在LinkedList的尾部和首部操作数据，因此LinkedList也可以被用作队列，双端队列，栈来使用。

LinkedList底层是用链表实现的，因此的它的访问性能较差，但插入、删除性能较好。一般迭代LinkedList时需要使用迭代器Iterator，而迭代ArrayList使用普通的for循环即可。

### 三 Set

Set集合是无序集合。它就像一个罐子，无法记住元素的添加顺序。通时它也不允许出现重复元素（因为Set是无序的，若出现重复元素，则无法区分），若试图将两个相同的元素添加到一个Set中，则添加操作失败，add()方法返回false，元素不会被加入。常见的Set有HashSet，TreeSet，EnumSet，它们均继承于Set。

##### 1. HashSet

HashSet继承于Set，因此它具有Set的所有特点。同时它是通过Hash算法来计算元素的存储位置，具有很好的存取和查找性能。因为集合中只能存放对象，因此它们的存储位置实际上是通过对象的hashCode方法得到的。

但是HashSet的判重和Set的不同，Set判重是通过equals()方法比较两个元素的值是否相同，HashSet的判重不只用equals比较两个元素的值，还比较这两个对象hashCode方法的返回值是否相等。只有这两者都满足，HashSet才会判断这两个元素重复。如果equals满足二hashCode不满足，则HashSet会在两个不同的hashCode位置存放这两个对象，而不会判定其重复；如果equals不满足而hashCode满足，则两个元素会被计算到同一个存储位置，此时该位置则采取链式结构来存放这两个元素。此时会严重影响HashSet的性能。

因此，如果我们要重写该对象对应类的equals方法，则也应该重写它的hashCode方法，使得：如果两个对象通过equals方法返回为true，则他们的hashCode返回值也应该相同。

##### 2. LinkedHashSet

LinkedHashSet继承于HashSet，它与HashSet的不同之处在于它以链表来维持元素的插入次序。即当遍历LinkedHashSet时，它会按照元素的添加顺序来访问集合里的元素。

因为LinkedHashSet要维护元素的插入顺序，因此它的性能要低于HashSet,但是当迭代访问集合里的全部元素时它有很好的性能。

##### 3. TreeSet

TreeSet是SortedSet的实现类（SortedSet是Set的子接口），它的底层实现是红黑树，可以确保集合中的元素一直处于排序状态。TreeSet中的排序分为自然排序和定制排序。

自然排序默认是升序，这也是TreeSet默认的排序方式。这种方式TreeSet中的元素要实现Comparable接口，并重写它的compareTo()方法，TreeSet在放入元素时会自动调用该元素中compareTo方法与集合中的元素比较大小。同时TreeSet中亦不允许重复元素，因此当compareTo返回0时，元素不会被放入集合中（返回正整数则说明a大约b，负整数则相反）。
应当注意：当把一个对象放入TreeSet中时，应当保证该对象的equals方法与compareTo方法得到的结果一致，即若equals方法返回true，则compareTo方法应当返回0。

定制排序：要实现定制排序，如降序排列，需要在创建TreeSet时传入一个Comparator对象负责集合元素的排序逻辑，由于Comparator也是函数式接口，因此可以使用Lambda表达式来代替Comparator对象。

##### 4. EnumSet

EnumSet是一个专门为枚举类设计的集合，其中的元素必须是指定枚举类型的枚举值。EnumSet集合是有序的，它安枚举类中枚举值的定义顺序来决定集合的元素顺序。EnumSet没有暴露任何构造器（即没有公有的构造器），要创建EnumSet需要调用它的类方法（静态方法）。

EnumSet在内部以位向量的方式存储，这种存储方式非常紧凑，高效，占用内存小，运行效率高。

##### 5. 各Set实现类的性能分析

- HashSet和TreeSet是常用实现，HashSet的性能要比TreeSet好一些，因为TreeSet需要额外的红黑树来维持元素的次序。


- LinkedHashSet与HashSet相比，在插入、删除方面，HashSet要快一些；但是在遍历时，LinkedHashSet要更快。

- EnumSet是所有Set实现类中性能最好的，但它功能有限，只能保存同一个枚举类的枚举值作为集合元素。

### 四 Queue

Queue集合用于模拟队列这种数据结构，队列是先进先出（FIFO）的，头部保存存放时间最长的元素，尾部保存存放时间最短的元素，队尾插入，队首出。Queue还有一个子接口Deque，Deque表示一个双端队列，因此既可以模拟队列也可以模拟栈。Deque有ArrayDeque和LinkedList两个实现类。

##### 1. PriorityQueue

PriorityQueue并不是严格的队列。它的队列中的元素的顺序并不是按插入队列的顺序，而是按照队列中元素的大小排序。PriorityQueue有两种排序方式：自然排序和定制排序。这点和TreeSet基本一致，可参考上文中TreeSet的简介。

##### 2. Deque和ArrayDeque

上文已经介绍了Deque，它是Queue的子接口，是一个双端队列，里面定义一些可以在队尾和队首操作元素的方法，因此它既可以模拟栈，也可以模拟队列。它主要有两个实现类：ArrayDeque和LinkedList。

ArrayDeque底层结构和ArrayList类似，一般用它来模拟栈，而不用Stack。

### 五 Map

Map是一个key-value键值对。它的每个单元包含key和value两个数据，由key可以找到value。key可以为空但不可重复，其实将它们分开来看，key就是一个Set集合，而value可以看做List，事实上Map中也有相应的方法。可以把Map看做value均为空的Set，事实上要想遍历Map，就需要获取key的Set集合。

##### 1. HashMap和HashTable

HashMap和Hashtable均为Map接口的实现类，它们之间的关系类似于ArrayList和Vector的关系。Hashtable是一个古老的实现类，现在已不推荐使用。

HashMap和Hashtable有两点区别：

- Hashtable是线程安全的，而HashMap不是
- Hashtable中的key和value均不能为空，而HashMap的可以

同时放入key中的对象需要被hash，所以key中的元素均要实现equals和hashCode方法，且二者结果应当一致，即若equals结果为true，则这两个元素的hashCode值也要相等。（与HashSet相同，Java的要hash类均要实现hashCode方法）。

##### 2. LinkedHashMap

LinkedHashMap是HashMap的子类，这俩的关系就像LinkedHashSet和HashSet的关系。使用一个链表来维持key的插入顺序。因此你可以在外部对元素排好序，然后让其顺序进入集合，就可以得到一个有序的HashMap，而不需要再对其进行排序。

##### 3. Properties

Properties是Hashtable的子类，也是一个古老的集合类，该类在处理属性文件是特别方便（属性文件即“属性名=属性值”的文件，如Windows上的ini文件）。可以把Properties看做一个key和value均为String的Map。该类可以与XML文件交互。

##### 4. SortedMap和TreeMap

TreeMap,SortedMap和Map的关心可以参照Set。TreeMap的实现与TreeSet类似，只是其中的元素换成了键值对，而所有对Set中元素的约束在TreeMap中均换成对key的约束。事实上在Java源码中Set的实现即是通过value均为null的Map完成的，所有Java中的Set和Map有很多相通之处。

##### 5. WeakHashMap

WeakHashMap即是key是弱引用的HashMap。若key被回收，则WeakHashMap会自动删除对应的key-value键值对。可使用它来做缓存。

##### 6. IdentityHashMap

IdentityHashMap与HashMap不同处在于：IdentityHashMap在处理key相等问题的方式是使用“==”，而不是equals。

##### 7. EnumMap

EnumMap是使用枚举类实现的Map，它的key是枚举类中的枚举值，创建它时必须传入一个枚举类（类的class对象）。它有以下几个特点：

- 内部以数组形式保存，紧凑，高效。
- 以枚举类中枚举值的定义顺序来维持key的顺序
- key不能为空

### 六 线程安全的集合类

##### 1. 集合工具类Collections

Java提供了一个操作集合的工具类：Collections，里面提供了大量的类方法用于对集合元素进行排序，查询，修改，同步，设置集合不可变等。（工具类中一般都是类方法，即静态方法）。

其中就有一种synchronizedXxx()方法，该方法可以将指定集合包装成线程同步的集合，从而解决了多线程并发访问集合时的线程安全问题。读者可以通过查看Collections的源码来了解它的用法。

##### 2. 线程安全的集合类

从Java5开始，Java在java.util.concurrent包下面提供了大量线程安全的集合类，主要可分为以下两类：

- 以Concurrent开头的集合类      
如ConcurrentHashMap，ConcurrentSkipListMap，ConcurrentSkipListSet，ConcurrentLinkedQueue，ConcurrentLinkedDeque等

- 以CopyOnWrite开头的集合类       
如CopyOnWriteArrayList，，CopyOnWriteArraySet等

### 六 总结

