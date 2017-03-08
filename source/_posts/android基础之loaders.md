---
title: android基础之loaders
date: 2017-02-27 14:39:21
categories: android
tags:
- android
- java
- loader
---

### 一 概述

Loaders从Android3.0开始引进（Loaders被翻译为装载器，它是一个异步加载数据的框架），它能在Activity或Fragment中异步加载数据；装载器具有如下特性：

- 它们对每个Activity和Fragment都有效
- 它们支持数据的异步加载
- 它们监视数据源的改变，并在数据源改变时传送新的结果
- 当由于配置改变而被重新创建后，它们会自动重连到上一个装载器的游标，所以不必重新查询数据

### 二 loader使用相关简介

##### LoaderManager

一个和Activity或Fragment关联抽象类，管理一个或多个装载器的实例，它帮助应用管理那些与Activity或Fragment生命周期相关的长时间运行的操作。最常见的方式是与一个CursorLoader一起使用，你也可以实现自己的装载器以加载其它类型的数据。 每个Activity或Fragment只有一个LoaderManager，但是一个LoaderManager可以拥有多个装载器。

##### Loader

一个执行异步数据加载的抽象类，它是加载器的基类。你可以使用典型的CursorLoader，但是你也可以实现你自己的子类。一旦装载器被激活，它们应该监视它们的数据源并且在数据改变时传送新的结果。

##### AsyncTaskLoader

一个使用AsyncTask来执行异步加载工作的抽象类。继承于Loader

##### CursorLoader

一个AsyncTaskLoader的子类，它查询ContentResolver然后返回一个Cursor。这个装载器类的实现遵循查询游标数据源的标准，它的游标查询是通过AsyncTaskLoader在后台线程中执行，从而不会阻塞UI线程。使用这个装载器是从ContentProvider异步加载数据的最好方式。

##### LoaderManager.LoaderCallbacks

一个用于客户端与LoaderManager交互的回调接口。主要有三个回调方法：onCreateLoader()，onLoadFinished()，onLoaderReset()。


### 三 使用Loader

一个使用加载器的典型的应用包含以下几个组件：

- 一个Activity或Fragment；
-  一个LoaderManager的实例；
-  一个依靠ContentProvider加载数据的CursorLoader；当然，你也可以继承Loader或AsyncTaskLoader实现你自己的装载器来从其它数据源加载数据；
- 一个LoaderManager.LoaderCallbacks的实现，这是你创建新的装载器以及管理已有装载器的地方；
-  一个用于展示装载器的返回数据的方式，例如使用一个SimpleCursorAdapter；
-  一个数据源，例如ContentProvider（使用CursorLoader加载数据）。

代码示例：





### 四 总结




