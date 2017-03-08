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
如果Intent对象中既包含Uri又包含Type，那么，在<intent-filter>中也必须二者都包含才能通过测试。Type属性用于明确指定Data属性的数据类型或MIME类型，但是通常来说，当Intent不指定Data属性时，Type属性才会起作用，否则Android系统将会根据Data属性值来分析数据的类型，所以无需指定Type属性。data和type属性一般只需要一个，通过setData方法会把type属性设置为null，相反设置setType方法会把data设置为null，如果想要两个属性同时设置，要使用Intent.setDataAndType()方法    


- extras（扩展信息）：扩展信息     
是其它所有附加信息的集合。以键值对的形式放入Intent中    


- Flags（标志位）：期望这个意图的运行模式      
用于指定Activity与task之间的关系。

### 三 Intent发现组件的两种方式
   
Intent启动组件有两种方式：显示启动和隐式启动。

上面Intent的Intent的7个属性中，只有component属于显示启动，action，category，data皆为隐式启动。

##### 1.显示启动

代码示例：

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

##### 2.隐式启动

- action

在Androidmanifest.xml中定义

```java
         <activity 
             android:name=".SecondActivity">
             <intent-filter>
                  <action android:name="com.example.smyh006intent01.MY_ACTION"/>
                  <category android:name="android.intent.category.DEFAULT" />
             </intent-filter>            
         </activity>
```

java代码调用

```java
          button1.setOnClickListener(new OnClickListener() {            
              @Override
              public void onClick(View v) {
                  //启动另一个Activity，（通过action属性进行查找）
                  Intent intent = new Intent();
                  //设置动作（实际action属性就是一个字符串标记而已）
                  intent.setAction("com.example.smyh006intent01.MY_ACTION"); //方法：Intent android.content.Intent.setAction(String action)
                  startActivity(intent);        
              }
         });
```

上面代码也可简写成

```java
         button1.setOnClickListener(new OnClickListener() {            
             @Override
             public void onClick(View v) {
                 //启动另一个Activity，（通过action属性进行查找）
                 Intent intent = new Intent("com.example.smyh006intent01.MY_ACTION");//方法： android.content.Intent.Intent(String action)                
                 startActivity(intent);        
             }
         });
```

- action+categoty

Androidmanifest.xml文件：

```java
         <activity
             android:name=".SecondActivity">
             <intent-filter>
                  <action android:name="com.example.smyh006intent01.MY_ACTION"/>
                  <category android:name="android.intent.category.DEFAULT" />
                  <category android:name="com.example.smyh006intent01.MY_CATEGORY" />
             </intent-filter>            
         </activity>
```

java文件中：

```java
         button1.setOnClickListener(new OnClickListener() {            
              @Override
              public void onClick(View v) {
                  //启动另一个Activity，（通过action属性进行查找）
                  Intent intent = new Intent();
                  //设置动作（实际action属性就是一个字符串标记而已）
                  intent.setAction("com.example.smyh006intent01.MY_ACTION"); //方法：Intent android.content.Intent.setAction(String action)
                  intent.addCategory("com.example.smyh006intent01.MY_CATEGORY");
                  startActivity(intent);        
             }
         });
```

- action+data

示例：打开指定网页

```java
         <activity 
             android:name=".SecondActivity">
             <intent-filter>
                  <action android:name="android.intent.action.VIEW" />
                  <category android:name="android.intent.category.DEFAULT" />
                  <data android:scheme="http" android:host="www.baidu.com"/>                 
             </intent-filter>            
         </activity>
```

java文件中：

```java
          button1.setOnClickListener(new OnClickListener() {            
              @Override
              public void onClick(View v) {
                  Intent intent = new Intent();
                  intent.setAction(Intent.ACTION_VIEW);
                  Uri data = Uri.parse("http://www.baidu.com");
                  intent.setData(data);                
                  startActivity(intent);        
              }
         });
```

上面代码也可简写成：

```java
         button1.setOnClickListener(new OnClickListener() {            
             @Override
             public void onClick(View v) {
                 Intent intent = new Intent(Intent.ACTION_VIEW);
                 intent.setData(Uri.parse("http://www.baidu.com"));                
                 startActivity(intent);        
             }
         });
```

### 四 IntentFilter

IntentFilter用于与Intent中的数据对比，来挑选出符合条件的组件。它分为静态注册（推荐）和动态注册两种方式。Intent与IntentFilter比较的步骤如下：

1. 加载安装所有的IntentFilter到一个列表中
2. 剔除所有action匹配失败的IntentFilter
3. 剔除URI数据匹配失败的IntentFilter
4. 剔除category匹配失败的IntentFilter
5. 剩下的IntentFilter是否为0。是，查找失败，抛出异常；否将剩下的IntentFilter按优先级排列，返回优先级最高的IntentFilter

代码示例：

静态注册

```java
<receiver android:name=".MyBroadCastReceiver">  
            <!-- android:priority属性是设置此接收者的优先级（从-1000到1000） -->
            <intent-filter android:priority="20">
            <actionandroid:name="android.provider.Telephony.SMS_RECEIVED"/>  
            </intent-filter>  
</receiver>
```

动态注册

```java
IntentFilter intentFilter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
```


### 五 总结