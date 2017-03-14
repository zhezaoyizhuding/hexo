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

TreeSet是SortedSet的实现类，它的底层实现是红黑树，可以确保集合中的元素一直处于排序状态。TreeSet中的排序分为自然排序和定制排序。

自然排序默认是升序，这也是TreeSet默认的排序方式。这种方式TreeSet中的元素要实现Comparable接口，并重写它的compareTo()方法，TreeSet在放入元素时会自动调用该元素中compareTo方法与集合中的元素比较大小。同时TreeSet中亦不允许重复元素，因此当compareTo返回0时，元素不会被放入集合中（返回正整数则说明a大约b，负整数则相反）。
应当注意：当把一个对象放入TreeSet中时，应当保证该对象的equals方法与compareTo方法得到的结果一致，即若equals方法返回true，则compareTo方法应当返回0。

定制排序：要实现定制排序，如降序排列，需要在创建TreeSet时传入一个Comparator对象负责集合元素的排序逻辑，由于Comparator也是函数式接口，因此可以使用Lambda表达式来代替Comparator对象。

##### 4. EnumSet



### 四 Queue

### 五 Map

### 六 线程安全的集合类

### 六 总结

