---
title: android基础之Service
date: 2017-02-27 14:06:22
categories: android
tags:
- android
- java
- service
---

### 一 Service简介    
Service也是Android的四大组件之一，它并不提供用户界面，主要做一些即使该应用程序被调入后台也需要运行的事务。比如下载，音乐播放，执行文件I/O或者与content provider进行交互等。Service主要做的就是上面这些比较耗时的操作，但是由于Service也是运行在主线程中，因此最好在其中另开一个线程做这些耗时操作，否者可能会引起UI阻塞。       

### 二 Service的两种启动方式     
Service有started和bound两种启动方式，当然也可以两种都启用。你可通过startService()启动服务后再通过bindService()将activity于Service绑定。     

##### 1.startService()    
    

### 三 Service的生命周期     


### 四 Service与线程的区别    


### 五 保证Service不被杀死    

   