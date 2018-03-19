---
title: 浅析HashMap
date: 2018-03-19 10:41:22
categories: Java源码浅析
tags:
- HashMap
- 源码
- 底层实现
- 原理
---

JDK1.8开始HashMap的源码有了一些改变，这篇博客主要分别介绍下JDK1.7和JDK1.8中HashMap的底层实现，并比较一下二者的不同。

### JDK1.7中的HashMap

HashMap是我们很常用的集合框架，对它的用法我们都很熟悉。但是如果要更好的使用它，我们可能需要理解HashMap的内部实现，同时这也是面试比较常问的一个问题。下面的源码使用的是Java SE Development Kit 7u80，主要介绍下HashMap的底层存储结构，扩容机制和一些常用的方法。

###### JDK1.7中的HashMap的存储结构

在JDk1.7中或者之前的一些版本中，HashMap的底层存储结构都是采用“数组+链表”的形式。数组的每一节被称为桶（bucket），每个bucket存储这一个或多个entry（Map中的每一个键值对），如果有冲突，冲突的地方开一个链表，链式存储这些entry，所以性能差的HashMap最终会退化成一个链表。JDK1.7中HashMap的存储结构大致可表述为下图：

{% asset_img JDK1.7中HashMap的底层实现.png JDK1.7中HashMap的底层实现 %}

table数组中的索引通过key的hashCode再取模获得。获取hashCode的hash算法如下(对hashCode进行rehash，尽量减少冲突)：

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

获取数组的索引的方法如下：

```java
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

###### 扩容

HashMap的成员变量中有一个负载因子loadFactor，值为0.75（不可变）。还有一个阈值变量threshold，这个变量的值为capacity * loadFactor。源码中定义如下：

```java
/**
     * The next size value at which to resize (capacity * load factor).
     * @serial
     */
    // If table == EMPTY_TABLE then this is the initial capacity at which the
    // table will be created when inflated.
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;
```

当table数组中所存储的bucket数大于等于这个值，并且当前位置的bucket已经有值时就会发生扩容。源码如下：

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}
```

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

扩容时会新建一个原先数组两倍的新数组，将原先数组的数据重新hash到新的数组，如果原先数组的某个bucket是链表，则链表顺序会反转。rehash后重设table引用和threshold的值。源码如下：

```java
/**
    * Transfers all entries from current table to newTable.
    */
   void transfer(Entry[] newTable, boolean rehash) {
       int newCapacity = newTable.length;
       for (Entry<K,V> e : table) {
           while(null != e) {
               Entry<K,V> next = e.next;
               if (rehash) {
                   e.hash = null == e.key ? 0 : hash(e.key);
               }
               int i = indexFor(e.hash, newCapacity);
               e.next = newTable[i];
               newTable[i] = e;
               e = next;
           }
       }
   }
```

同时我们要知道上面的addEntry方法是在put方法中调用的，HashMap在初始化时并没有创建table数组（只是创建一些基本的属性值，比如capacity），而是在真正放入数据才开始创建，并在这里判断是否需要扩容。
