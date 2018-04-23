---
title: 浅析ThreadLocal
date: 2018-04-23 20:31:54
categories: Java源码浅析
tags:
- ThreadLocal
---

在一次面试被问到了ThreadLocal，其间被问到了一个问题：ThreadLocal有什么缺陷。笔者不缺定的回答：内存泄漏？面试官随即问道了：为什么会造成内存泄漏？笔者懵逼了，不知道该如何回答。因为笔者记忆里现在的ThreadLocal好像并不会造成内存泄漏。但是并没有深入了解过，所以当时笔者完全不敢确定，也不知道该如何回答。下面在这片博客中通过源码（JDK1.8）来简单了解一下ThreadLocal的内部实现。
