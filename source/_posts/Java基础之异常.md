---
title: Java基础之异常
date: 2017-03-17 16:06:28
categories: Java基础知识
tags:
- Java
- 异常
---

### 一 概述

异常处理已经成为衡量一门语言是否成熟的标准之一，目前主流的C++，C#，Ruby，Python等大都提供了异常处理机制。异常处理可以使正常业务代码与异常代码分离开，使程序具有更好的容错性和健壮性。一个好的程序员不只能做好”对“的事情，“错”的事情也要做好。

### 二 Java异常继承体系

Java中的异常皆继承于Exception，Exception与Error共同继承于Throwable。下面是Java的异常继承体系图：

{% asset_img Java异常继承体系图.jpg Java异常继承体系图 %}

### 三 Java中的异常处理

Java中的异常分为Checked（编译时异常）和RuntimeException（运行时异常）两种，同时规定Checked异常必须被处理否则无法通过编译，这虽进一步增强了Java程序的健壮性，但是也使开发者烦不胜烦，是否应该存在该类异常一直是备受争议的问题。

Java定义了5个关键字来处理异常：try，catch，finally，throw，throws。

##### 1.使用try...catch...finally捕获异常

Java提供了try...catch...finally的方式来捕获异常，在try块中处理业务代码，在catch块中捕获异常并处理异常，在finally块中关闭在try中打开的物理资源，如文件，数据库等。如下面代码示例：

```java
    try{
       System.out.println("try") ;
    }catch(ArrayIndexOutOfBoundsException e){
       System.out.println("Exception thrown  :" + e);
    }
    finally{
       System.out.println("The finally statement is executed");
    }
```

如果没有在捕获异常后还需要处理的问题，则可以省略finally块；若有多个异常，可以写多个catch块（但是只能执行一个），在Java7之前一个catch块只能捕获一个异常，同时要注意子异常应该写在父异常前面，否者编译不通过。

**注意finally是无论如何都会执行的模块（除非虚拟机崩了），尽量不要再finally中添加return，它会覆盖try的return。**

##### 2.访问异常信息

前面说到Java中的异常最终皆继承于Throwable，而Throwable有以下几个方法在捕获异常后经常用到：

- getMessage()： 返回该异常的详细描述字符串。
- printStackTrace()：将该异常的跟踪栈信息打印出来
- printStackTrace(PrintStream s)：将该异常的跟踪栈信息输出到指定输出流
- getStackTrace()：返回该异常的跟踪栈信息

##### 3.Java中Checked异常和Runtime异常

Java中增加了一个Checked异常，而其他语言都没有该异常。这些异常必须被显示处理，否则编译不通过。这体现了java
设计的哲学--没有完善错误处理的代码根本不会被执行。对checked异常的处理有以下两种：

- 使用try...catch捕获异常
- 使用throws抛出异常

而Runtime异常很灵活，不需要显示声明抛出，当然也可以使用try-catch捕获该异常并处理，而Checked异常就是个老流氓，你要么捕获并处理它，要么抛出它让你的上一层调用者处理它。

##### 4.使用throws声明抛出异常

throws只能在方法签名后声明抛出异常，可以抛出多个异常类。

使用throws抛出异常的思路是：当前方法不知道如果处理该异常（如果能处理就用try-catch捕获它），将它抛给上层调用者处理，如果上层调用者也无法处理，继续向上抛出，知道main方法，如果main方法也不能处理，抛给JVM虚拟机。JVM对异常的处理方式是：打印异常的跟踪栈信息，并中止程序。

使用throws抛出异常也有限制:子类方法声音抛出的异常应当是父类抛出异常的子类（或者二者想同），子类声明抛出的异常不允许比父类抛出的多。

throws一般用来抛出Checked异常。

##### 5.使用throw抛出异常

异常这个东西很有哲学性，你可以把它理解为与预想不符的东西，在这个地方它不是异常，而在另一个地方它可能就是异常。所以很多时候，系统是否要抛出异常，可能要根据具体的业务来决定，而这些有具体业务需求而产生的异常，系统无法自动抛出，不需有程序员手动抛出。这时就使用throw。
throw抛出的是一个异常实例，可单独成句，自行抛出异常，但每次只能抛出一个异常。示例：

```java
	throw new Exception("你要打开的文件已经存在了");
```

使用throw抛出的异常，在catch中亦可以捕获到。但throw一般不这样是使用，它一般与throws结合使用。因为有时候底层的异常比较敏感，我们不希望程序的调用者或者使用者看到这些异常，这时候我们就可以在catch中捕获这些敏感异常（一般将其输入到日志中），然后再用throw抛出一个新的异常（一般是对发生的异常的描述）给它的调用者，最后通过throws抛给它的调用者。这样我们就对底层进行了很好的封装。

在Java7之前我们通过throw抛出的是什么异常，那方法签名后的throws也要抛出相应异常。Java7之后，系统可以判断出具体发生的异常。比如，如果发生的是FileOutputException，而捕获并throw的是Exception异常，在Java7之前throws必须抛出Exception异常，而Java7之后throws FileOutputException即可。

### 四 Java7增强的异常

##### 1.Java7增强功能点一

在Java7之前一个catch只能捕获一个异常，但Java7新增了一个功能是的一个catch可以捕获多个异常。使用一个catch捕获多个异常应该注意一下两个方面：

- 在捕获多异常时多个异常之间用“|”分隔
- 捕获多异常时，异常对象有隐式的final属性，无法对它重新赋值（单异常时可以）

##### 2.Java7增强功能点二

在Java7之前如果在try中打开了物理资源，就必须在finally中关闭，Java7新增了一个功能，可以使try中打开的资源自动关闭。如：

```java
	try(
		BufferedReader br = new BufferedReader(
			new FileReader("test.java"));
		PrintStream ps = new PrintStream(
			new FileOutputStream("a.txt"))
	){
		System.out.println(br.readLine());
		ps.println("我若成佛，天下无魔；我若成魔，佛奈我何");
	}
```

需要指出的是要想自动关闭资源，这些资源实现类必须实现AutoCloseable或Closeable接口，实现它们的close方法。放心这些工作不需要你来做，Java7基本把所有的资源类进行了改写，使它们实现了AutoCloseable或Closeable接口。

### 五 自定义异常

有时候为了抛出明确的异常，我们需要自定义异常类。

自定义异常类继承于Exception类，如果希望自定义Runtime异常，就继承于RuntimeException类。定义自定义异常类需要提供了两个构造器：

- 一个是无参构造器
- 一个是带一个字符串参数的构造器，该字符串传入该异常对象的描述信息

下面是代码示例：

```java
	public class CustomException extends Exception{

		public CustomException(){}
		puvlic CustomException(String msg){
			super(msg);
		}
	}
```

自定义异常类应该取一个足以描述异常的类名。

### 六 异常处理规则

在Java中使用异常应该遵守一定的规则：

- 不要过度使用异常，只对可能发生异常的语句捕获异常
- 不要使用过大的try块，不要将可能发生异常的代码与正常代码都放在一个try块中
- 捕获的异常尽量具体，不要捕获的均是Exception
- 不要忽略捕获的异常
