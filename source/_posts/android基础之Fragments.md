---
title: android基础之Fragments
date: 2017-02-27 14:31:31
categories:　android
tags: 
- android
- java
- Fragment
---

### 一 概述   
Fragment表现Activity中用户界面的一个行为或者是一部分。你可以在一个单独的activity上把多个fragment组合成为一个多区域的UI，并且可以在多个activity中再使用。你可以认为fragment是activity的一个模块零件，它有自己的生命周期，接收它自己的输入事件，并且可以在activity运行时添加或者删除。

Fragment必须总是被嵌入到一个activity之中，并且fragment的生命周期直接受其宿主activity的生命周期的影响。例如，一旦activity被暂停，它里面所有的fragment也被暂停，一旦activity被销毁，它里面所有的fragment也被销毁。然而，当activity正在运行时（处于resumed的生命周期状态），你可以单独的操控每个fragment，比如添加或者删除。当你执行这样一项事务时，可以将它添加到后台的一个栈中，这个栈由activity管理着——activity里面的每个后台栈内容实体是fragment发生过的一条事务记录。这个后台栈允许用户通过按BACK键回退一项fragment事务（往后导航）。

当你添加一个fragment作为某个activity布局的一部分时，它就存在于这个activity视图体系内部的ViewGroup之中，并且定义了它自己的视图布局。你可以通过在activity布局文件中声明fragment，用<fragment>元素把fragment插入到activity的布局中，或者是用应用程序源码将它添加到一个存在的ViewGroup中。然而，fragment并不是一个定要作为activity布局的一部分；fragment也可以为activity隐身工作。   

### 二 Fragment的继承结构   
继承 Object
实现 ComponentCallbacks2 View.OnCreateContextMenuListener 
      
```java 
java.lang.Object
   ↳ 	android.app.Fragment
```
直接子类：
DialogFragment, ListFragment, PreferenceFragment, WebViewFragment      

### 三 Fragment设计哲学  
Fragment被设计出来主要为了适应平板等大屏幕移动设备。有了fragment，你可以不必去管理视图体系的复杂变化。通过将activity的布局分割成若干个fragment，可以在运行时编辑activity的呈现，并且那些变化会被保存在由activity管理的后台栈里面。  
如下图所示，一个Activity中的两个Fragment分别在手机和平板的显示情况：    
{% asset_img Fragment显示图.png Fragment显示图 %}     

### 四 Fragment的生命周期   


### 五 将Fragment添加到Activity中    

### 六 Fragment事务后台栈     

### 七 与Activity交互    

