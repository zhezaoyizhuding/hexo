---
title: Java基础之注解
date: 2017-03-17 10:26:21
categories: JAva
tags:
- Java
- 注解
- 注释
---

### 一 概述

从Java5开始，Java增加了对元数据的支持，也就是Annotation（注解）。Annotation其实就是代码中的特殊标记，这些标记可以在编译，类加载，运行时被读取，并执行相应的处理。通过使用注解，程序开发人员可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充信息。代码分析工具，开发工具和部署工具可以通过这些补充信息进行验证或者进行部署。

注解在定义时类似一个接口，程序可以通过反射来获取指定程序元素的注解对象，在使用时注解就像修饰符一样，可用于修饰类，方法，变量参数等。这些信息被存储在注解的“name=value”对中。

### 二 Java中内置注解

下面是5个Java中内置的注解：

- @Override    
用于标记重写方法，被标记的方法说明是重写父类或父接口的方法

- @Deprecated    
标记方法已过时，是不推荐使用的方法

- SuppressWarning   
抑制某个编译器警告，被它标记的方法或者类，在执行时编译器将不再显示其表明抑制的警告。如@SuppressWarning（value=“unchecked”），注解后面的是一些可以设置的值，若只有一个value值，可以省略为@SuppressWarning（“unchecked”）

- @SafeVarargs      
Java7新增的注解，它的效果类似于@SuppressWarning（“unchecked”）

- @FunctionalInterface    
该注解标记该接口为函数式接口。Java8规定：如果如果接口中只有一个抽象方法，则该方法为函数式接口（可以包含多个静态方法或者多个默认方法）。函数式接口就是Java8中为Lambda表达式定义的，Lambda表达式可以直接创建函数式接口的实例。默认方法是Java8中新增的特性，使用default关键字修饰，默认方法同static方法一样可以直接在接口中实现，实际上默认方法就是为了解耦接口和它的实现类而设计的（实现了接口就必须实现它的抽象方法，若接口改变，则所有的实现类都要改变）

### 三 元注解

元注解即是修饰注解的注解，用在注解定义的地方。Java中内置了以下几个常用的元注解。

- @Retention     
该注解只能用于修饰注解的定义，说明该注解可以保留多长时间。它定义了一个value变量，可以有3中取值，对应类加载时的3中状态。  
1.RetentionPolicy.CLASS：该属性是默认值，说明该注解信息被编译在class文件中，但无法在运行时获取注解信息。   
2.RetentionPolicy.RUNTIME：表明该注解不止被编译在了class文件中，而且可以在程序运行时通过反射获取注解的信息。    
3.RetentionPolicy.SOURCE：说明该注解信息只放在源文件中，编译时直接丢弃。   

- @Target
用于指定被修饰的注解能用于修饰哪些程序单元。它也包含一个名为value的成员变量，该变量包含以下几种取值：   
1.ElementType.ANNOTATION_TYPE：指定该策略的注解只能修饰注解。    
2.ElementType.CONSTRUCTOR：标记该注解只能修饰构造器    
3.ElementType.FIELD：标记该注解只能修饰成员变量     
4.ElementType.LOCAL_VARIABLE：标记该注解只能修饰局部变量    
5.ElementType.METHOD：标记该注解只能修饰方法    
6.ElementType.PACKAGE：标记该注解只能修饰包     
7.ElementType.PARAMETER：标记该注解只能修饰参数    
8.ElementType.TYPE：标记该注解能修饰类，接口（包括注解），枚举类型    

- @Documented    
表明被该元注解修饰的注解将被javadoc提取为文档，如果一个注解在定义时使用该元注解修饰，那么所有被该注解修饰的程序元素的API文档中将会包含该注解信息。

- @Inherited   
表明被该元注解修饰的注解具有继承性，即若该被修饰的注解修饰了某个类，则它的子类会自动被该注解修饰。

### 四 自定义注解

定义注解：

```java
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.TYPE)
	
	public @interface MyAnnotation {
	  public String name();
	  public String value();
	}
```

上面是注解的定义过程，可以看到它类似一个接口。起方法名和返回值定义了该成员变量的名字和类型，在定义了这些成员变量后，使用注解时，必须给他们赋值；同时它也可以使用default关键字来定义默认值，这样在使用时就不必须给它赋值。如：

```java
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.TYPE)
	
	public @interface MyAnnotation {
	  public String name() default "Tom";
	  public String value() default "jeri";
	}
```

使用注解：

```java
	@MyAnnotation(name="someName",  value = "Hello World")
	public class TheClass {
	}
```

### 五 提取注解信息

在使用注解修饰了类，方法，成员变量后这些注解不会自己生效，必须有开发者提供相应的工具类来提取并处理这些注解信息。

Java为此提供了一个Annotation接口，它是所有注解的父接口，Java同时还在java.lang.reflect包下新增了AnnotatedElement接口，该接口代表了程序中可以接受注解的程序元素，里面定义一些获取相应元素的注解的方法。它有以下几个实现类：

- Class
- Constructor
- Field
- Method
- Package

### 六 Java8新增的重复注解

从Java8开始为简化两个相同注解修饰同一个程序元素的情况，定义了重复注解，这个注解也是个元注解--@Repeatable。
代码示例：

可重复的注解：

```java
	@Retention(Retention.RUNTIME)
	@Target(ElementType.TYPE)
	@Repeatable（examples.class）
	public @interface example{
		String name() default "Tom";
		int age();
	}
```

作为父容器的注解：

```java
	@Retention(Retention.RUNTIME)
	@Target(ElementType.TYPE)
	public @interface examples{
		example[] value();
	}
```

使用：

```java
	@example(age=5)
	@example(name="jeri",age=9)
	public class RepeatTest{

	}
```

而不必写成：

```java
	@examples({@example(age=5),@example(name="jeri",age=9)})
	public class RepeatTest{

	}
```

实际上两种情况都对，Java8就是为了简化第二种形式，才设计了重复注解。

### 七 Java8新增的类型注解

Java8为ElementType枚举增加了TYPE_PARAMETER和TYPE_USE两个枚举值，这样定义注解时就允许使用@Target(ElementType.TYPE_USE)修饰，这种注解被称为类型注解，类型注解可用在任何使用类型的地方，如：

- 创建对象（用new关键字创建）

```java
Object str = new @NotNull Object();
```

- 类型转换

```java
String str = (@NotNull String)Object;
```

- 使用implement实现接口

```java
puclic class example implements @NotNull Serializable{}
```

- 使用throws声明抛出异常

```java
public void example(int i)  throws @NotNull FlieNotFoundException{}
```

### 八 注解处理工具APT




 






