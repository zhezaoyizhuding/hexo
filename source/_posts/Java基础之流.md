---
title: Java基础之流
date: 2017-03-17 16:06:59
categories: Java
tags:
- Java
- IO
---

### 一 概述

Java在java.io包下提供了相应的类和接口来对IO进行支持。Java中主要包含输入输出两种流，没中输入输出流又分为字节流和字符流两种流。其中字节流以字节为单位处理输入输出操作，字符流以字符来处理输入输出操作。从Java7开始Java在java.nio包下提供了一种全新的API，被称作NIO，它可以更高效的进行输入、输出操作。

### 二 Java IO流相关的类

- File类：系统中文件或者目录的实例，里面有很多方法用于获取文件和目录信息，可以通过查看File的源码获知。
- 字节流：InputStream/OutputStream输入输出字节流的基类
- 字符流：Reader/Writer 输入输出字符流的基类
- 处理流：如PrintStream，可以通过传入节点流来简化读写操作。上面的字节流和字符流就叫做节点流。
- 转换流：如InputStreamReader，字节流转换成字符流，由于字符流比字节流处理方便，所以Java中只有字节流转字符流的类，没有字符流转字节流的类。
- 缓冲流：如BufferedReader，用于包装字符流。


### 三 RandomAccessFile类

一个强大的文件内容访问类，可以直接跳转到文件的任意位置读写数据。

### 四 序列化

Java中的序列化是将Java对象转化为二进制流，以便写入磁盘或者在网络上传输。要序列化某个对像需要实现Serializable或者Externalizable（一般是实现Serializable），这在JavaEE中非常普遍，因为JavaEE张系统之间要相互远程调用，一般建议将每一个JavaBean都要序列化。

如果一个类中注入了另一个对象，若想这个类可以序列化，则这个对象所在的类也必须序列化。

##### 自定义序列化

- 使用transient关键字    
有时候我们序列化一个对象，却不想它的某个成员变量被序列化，可以使用transient关键字修饰该变量。

- 实现特殊的签名方法    
自定义序列化的类要实现这些方法：writeObject，readObject，readObjectNoData

- 实现Externalizable接口，自定义序列化

##### 版本

反序列化必须提供对象的class文件，但是随着项目的升级，class文件也会改变，为了保证两个class文件的兼容性，Java要求需要序列化的类要是设置一个private static final变量serialVersionUID。只要改值不变，则被认为是同一个序列化版本。

### 五 Java NIO（Java4新增，Java7增强）

采用内存映射文件的方式来处理输入/输出，效率很高。
