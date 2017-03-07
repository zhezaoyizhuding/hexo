---
title: android基础之Intents与Intent Filters
date: 2017-02-27 14:32:09
categories: android
tags:
- android
- java
- Intent
- IntentFilter
---

### 一 概述

Intent是Android中组件之间传递消息的一种机制，这个组件可以是同一个应用程序的，也可以是不同的应用程序。Android四大组件中除了ContentProvider，其余三者皆可通过Intent来传递数据。而且Intent是一种运行时绑定机制。

Intent在Android的三个组件中传递的机制是不同的：

- 使用Context.startActivity() 或 Activity.startActivityForResult()，传入一个intent来启动一个activity。使用 Activity.setResult()，传入一个intent来从activity中返回结果。
- 将intent对象传给Context.startService()来启动一个service或者传消息给一个运行的service。将intent对象传给 Context.bindService()来绑定一个service。
- 将intent对象传给 Context.sendBroadcast()，Context.sendOrderedBroadcast()，或者Context.sendStickyBroadcast()等广播方法，则它们被传给 broadcast receiver。

### 二 Intent的结构

Intent主要包含以下属性：

- component(组件)：目的组件    
Component属性明确指定Intent的目标组件的类名称。（属于直接Intent）。组件的名字通过函数setComponent()、setClass()、setClassName()设置，通过函数读取getComponent()。   
- action（动作）：用来表现意图的行动  
Action主要用来描述应用程序的组件，一个组件可以用于多个action，也可以多个组件拥有同一个action。
- category（类别）：用来表现动作的类别     
category通常和Action放在一起用的，用于描述这个action。一个action可以包含多个categoty，必须所有的categoty都匹配成功才算是与这个action匹配成功。如果一个action只有一个默认的categoty DEFAULT，则不需向Intent中添加categoty，系统会自动添加。addCategory() 放置一个intent里的类别，removeCategory()删除之前添加的，getCategories()获取当前所有的类别。      
类别越多，动作越具体，意图越明确    
- data（数据）：表示与动作要操纵的数据      
Data属性是Android要访问的数据，和action和Category声明方式相同，也是在<intent-filter>中。多个组件匹配成功显示优先级高的； 相同显示列表。Data是用一个uri对象来表示的，uri代表数据的地址，属于一种标识符。通常情况下，我们使用action+data属性的组合来描述一个意图：做什么    
- type（数据类型）：对于data范例的描写   
描述data的类型，http://www.cnblogs.com/smyhvae/p/3959204.html
- extras（扩展信息）：扩展信息
- Flags（标志位）：期望这个意图的运行模式

##### 1.component：目的组件
   
如果 component这个属性有指定的话，将直接使用它指定的组件。指定了这个属性以后，Intent的其它所有属性都是可选的。组件的名字通过函数setComponent()、setClass()、setClassName()设置，通过函数读取getComponent()。       
例如，启动第二个Activity时，我们可以这样来写：

```java
    button1.setOnClickListener(new OnClickListener() {            
              @Override
              public void onClick(View v) {
                  //创建一个意图对象
                  Intent intent = new Intent();
                  //创建组件，通过组件来响应
                  ComponentName component = new ComponentName(MainActivity.this, SecondActivity.class);
                  intent.setComponent(component);                
                  startActivity(intent);                
             }
         });
```

如果写的简单一点，监听事件onClick()方法里可以这样写：

```java
      Intent intent = new Intent();
      //setClass函数的第一个参数是一个Context对象
      //Context是一个类，Activity是Context类的子类，也就是说，所有的Activity对象，都可以向上转型为Context对象
      //setClass函数的第二个参数是一个Class对象，在当前场景下，应该传入需要被启动的Activity类的class对象
      intent.setClass(MainActivity.this, SecondActivity.class);
      startActivity(intent);    
```

再简单一点，可以这样写：（当然，也是最常见的写法）

```java
                 Intent intent = new Intent(MainActivity.this,SecondActivity.class);
                 startActivity(intent);
```


##### 2.Action（动作）：用来表现意图的行动

### 三 Intent发现组件的两种方式

### 四 IntentFilter

### 五 总结