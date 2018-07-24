---
title: 浅析Java并发工具类(2) - CyclicBarrier
date: 2018-07-23 22:18:41
categories: Java源码浅析
tags:
- CyclicBarrier
---

### CyclicBarrier

CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）,它的功能是让一组线程到达一个屏障（也就叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会打开，所有被屏障拦截的线程继续运行。值得一提的是，所谓的循环是指CyclicBarrier中的parties（相当于CountDownLatch的count）是可以重置的，因此它可以被循环调用。