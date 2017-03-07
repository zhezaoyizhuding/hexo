---
title: android基础之Content Provider
date: 2017-02-27 14:32:42
categories: android
tags:
- android
- java
- Content Provider
---

### 一 概述

ContentProvider（内容提供者）是Android的四大组件之一，通过它可以向其他的应用程序共享其数据。虽然使用其他方法也可以对外共享数据，但数据访问方式会因数据存储的方式而不同，如：采用文件方式对外共享数据，需要进行文件操作读写数据；采用sharedpreferences共享数据，需要使用sharedpreferences API读写数据。而使用ContentProvider共享数据的好处是统一了数据访问方式。而且Android为常见的一些数据提供了默认的ContentProvider（包括音频、视频、图片和通讯录等）。

但注意ContentProvider它也只是一个中间人，真正操作的数据源可能是数据库，也可以是文件、xml或网络等其他存储方式。

### Uri类简介

```java
Uri uri = Uri.parse("content://com.changcheng.provider.contactprovider/contact")
```

在Content Provider中使用的查询字符串有别于标准的SQL查询。很多诸如select, add, delete, modify等操作我们都使用一种特殊的URI来进行，这种URI由3个部分组成， “content://”, 代表数据的路径，和一个可选的标识数据的ID。以下是一些示例URI:    

```java
content://media/internal/images  //这个URI将返回设备上存储的所有图片
content://contacts/people/  //这个URI将返回设备上的所有联系人信息
content://contacts/people/45  //这个URI返回单个结果（联系人信息中ID为45的联系人记录）
```

尽管这种查询字符串格式很常见，但是它看起来还是有点令人迷惑。为此，Android提供一系列的帮助类（在android.provider包下），里面包含了很多以类变量形式给出的查询字符串，这种方式更容易让我们理解一点，因此，如上面content://contacts/people/45这个URI就可以写成如下形式：

```java
Uri person = ContentUris.withAppendedId(People.CONTENT_URI,  45);
```

然后执行数据查询:

```java
Cursor cur = managedQuery(person, null, null, null);
```

这个查询返回一个包含所有数据字段的游标，我们可以通过迭代这个游标来获取所有的数据.

### 三 ContentProvider类简介

##### 1.主要方法

- public boolean onCreate()   
ContentProvider创建后　或　打开系统后其它应用第一次访问该ContentProvider时调用。      
- public Uri insert(Uri uri, ContentValues values)    
外部应用向ContentProvider中添加数据。    
- public int delete(Uri uri, String selection, String[] selectionArgs)     
外部应用从ContentProvider删除数据。    
- public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)         
外部应用更新ContentProvider中的数据。    
- public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)　    
供外部应用从ContentProvider中获取数据。 　    
- public String getType(Uri uri)     
该方法用于返回当前Url所代表数据的MIME类型  

##### 2.创建步骤

要创建我们自己的Content Provider的话，我们需要遵循以下几步：

1.创建一个继承了ContentProvider父类的类   
2.定义一个名为CONTENT_URI，并且是public static final的Uri类型的类变量，你必须为其指定一个唯一的字符串值，最好的方案是以类的全名称， 如:


```java
public static final Uri CONTENT_URI = Uri.parse( “content://com.google.android.MyContentProvider”);
```

3.定义你要返回给客户端的数据列名。如果你正在使用Android数据库，必须为其定义一个叫_id的列，它用来表示每条记录的唯一性。     
4.创建你的数据存储系统。大多数Content Provider使用Android文件系统或SQLite数据库来保持数据，但是你也可以以任何你想要的方式来存储。      
5.如果你要存储字节型数据，比如位图文件等，数据列其实是一个表示实际保存文件的URI字符串，通过它来读取对应的文件数据。处理这种数据类型的Content Provider需要实现一个名为_data的字段，_data字段列出了该文件在Android文件系统上的精确路径。这个字段不仅是供客户端使用，而且也可以供ContentResolver使用。客户端可以调用ContentResolver.openOutputStream()方法来处理该URI指向的文件资源；如果是ContentResolver本身的话，由于其持有的权限比客户端要高，所以它能直接访问该数据文件。    
6.声明public static String型的变量，用于指定要从游标处返回的数据列。    
7.查询返回一个Cursor类型的对象。所有执行写操作的方法如insert(), update() 以及delete()都将被监听。我们可以通过使用ContentResover().notifyChange()方法来通知监听器关于数据更新的信息。     
8.在AndroidMenifest.xml中使用<provider>标签来设置Content Provider。    
9.如果你要处理的数据类型是一种比较新的类型，你就必须先定义一个新的MIME类型，以供ContentProvider.geType(url)来返回。MIME类型有两种形式:一种是为指定的单个记录的，还有一种是为多条记录的。这里给出一种常用的格式：   
- vnd.android.cursor.item/vnd.yourcompanyname.contenttype （单个记录的MIME类型）   
比如, 一个请求列车信息的URI如content://com.example.transportationprovider/trains/122 可能就会返回typevnd.android.cursor.item/vnd.example.rail这样一个MIME类型。   
- vnd.android.cursor.dir/vnd.yourcompanyname.contenttype （多个记录的MIME类型）     
比如, 一个请求所有列车信息的URI如content://com.example.transportationprovider/trains 可能就会返回vnd.android.cursor.dir/vnd.example.rail这样一个MIME 类型。

下面是一个Content Provider代码示例：


> 代码

一个名为MyContentProvider的Content Provider创建完成了，它用于从Sqlite数据库中添加和读取记录。

### 四 ContentResolver

一个ContentProvider新建好了之后，我们需要一个ContentResolver类来获取其共享出的数据。

##### 1.ContentResolver的主要方法

- public Uri insert(Uri uri, ContentValues values)　//添加
- public int delete(Uri uri, String selection, String[] selectionArgs)　//删除
- public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)　//更新
- public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)//获取


示例代码：

```java
ContentResolver resolver =  getContentResolver();
Uri uri = Uri.parse("content://cn.scu.myprovider/user");

//插入
ContentValues values = new ContentValues();
values.put("name", "fanrunqi");
values.put("age", 24);
resolver.insert(uri, values);  

//查询
Cursor cursor = resolver.query(uri, null, null, null, "userid desc");
while(cursor.moveToNext()){
   //操作
}

//更新
ContentValues updateValues = new ContentValues();
updateValues.put("name", "finch");
Uri updateIdUri = ContentUris.withAppendedId(uri, 1);
resolver.update(updateIdUri, updateValues, null, null);

//删除
Uri deleteIdUri = ContentUris.withAppendedId(uri, 2);
resolver.delete(deleteIdUri, null, null);
```

##### 使用ContentResolver获取数据的步骤

1. 通过getContentResolver()方法得到ContentResol1.ver对象。
2. 调用ContentResolver类的query()方法查询数据，该方法会返回一个Cursor对象。
3. 对得到的Cursor对象进行分析，得到需要的数据。
4. 调用Cursor类的close()方法将Cursor对象关闭。

示例代码（与上面ContentProvider处的代码相连）：

> 代码


### 五 总结




      







