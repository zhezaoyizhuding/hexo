---
title: Java基础之反射与类加载器
date: 2017-03-19 22:48:27
categories: Java基础知识
tags:
- 类加载器
- 反射
---

### 一 概述

类加载即是编译好的class文件被加载进Java虚拟机的过程，而这就需要用到Java类加载器java.lang.ClassLoader。class文件被加载进虚拟机后会生成java.lang.Class对象，而这些是发生在Java虚拟机中的，除了类加载过程，其余的我们并不能看到。要想获得这个Class对象，我们就需要通过反射。

### 二 类加载

指将类的class文件读入内存，并为之创建一个java.lang.Class的对象。

类文件来源

- 从本地文件系统加载的class文件
- 从JAR包加载class文件
- 从网络加载class文件
- 把一个Java源文件动态编译，并执行加载

类加载器通常无须等到“首次使用”该类时才加载该类，JVM允许系统预先加载某些类

##### 类加载器的层次结构

下面是Java中类加载器的层次结构（不是继承结构）：

{% asset_img 类加载器的层次结构.png 类加载器的层次结构 %}

- 根类加载器（Bootstrap ClassLoader）：其负责加载Java的核心类，比如String、System这些类
- 拓展类加载器（Extension ClassLoader）：其负责加载JRE的拓展类库
- 系统类加载器（System ClassLoader）：其负责加载CLASSPATH环境变量所指定的JAR包和类路径
- 用户类加载器：用户自定义的加载器，以类加载器为父类

代码示例:

```java
    public static void main(String[] args) throws IOException {
        ClassLoader systemLoader = ClassLoader.getSystemClassLoader();
        System.out.println("系统类加载");
        Enumeration<URL> em1 = systemLoader.getResources("");
        while (em1.hasMoreElements()) {
            System.out.println(em1.nextElement());
        }
        ClassLoader extensionLader = systemLoader.getParent();
        System.out.println("拓展类加载器" + extensionLader);
        System.out.println("拓展类加载器的父" + extensionLader.getParent());
```

结果：

```java
系统类加载
file:/E:/gaode/em/bin/
拓展类加载器sun.misc.Launcher$ExtClassLoader@6d06d69c
拓展类加载器的父null
```

**注意**根类加载器并不是Java实现的，而且由于程序通常须访问根加载器，因此访问扩展类加载器的父类加载器时返回NULL

##### JVM类加载机制

- 全盘负责，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入

- 父类委托，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类

- 缓存机制，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效

##### URLClassLoader类

URLClassLoader为ClassLoader的一个实现类，该类也是系统类加载器和拓展类加载器的父类（继承关系）。它既可以从本地文件系统获取二进制文件来加载类，也可以远程主机获取二进制文件来加载类。

它的两个构造器：

- URLClassLoader(URL[] urls):使用默认的父类加载器创建一个ClassLoader对象，该对象将从urls所指定的路径来查询并加载类

- URLClassLoader(URL[] urls,ClassLoader parent):使用指定的父类加载器创建一个ClassLoader对象，其他功能与前一个构造器相同

代码示例：

```java
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLClassLoader;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

import com.mysql.jdbc.Driver;

public class GetMysql {
    private static Connection conn;
    public static Connection getConn(String url,String user,String pass) throws MalformedURLException, InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException{
        if(conn==null){
            URL[]urls={new URL("file:mysql-connector-java-5.1.18.jar")};
            URLClassLoader myClassLoader=new URLClassLoader(urls);
            Driver driver=(Driver) myClassLoader.loadClass("com.mysql.jdbc.Driver").newInstance();
            Properties pros=new Properties();
            pros.setProperty("user", user);
            pros.setProperty("password", pass);
            conn=driver.connect(url, pros);
        }
        return conn;
    }
    public static method1 getConn() throws MalformedURLException, InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException{

            URL[]urls={new URL("file:com.em")};
            URLClassLoader myClassLoader=new URLClassLoader(urls);
            method1 driver=(method1) myClassLoader.loadClass("com.em.method1").newInstance();

        return driver;
    }

    public static void main(String[] args) throws MalformedURLException, InstantiationException, IllegalAccessException, ClassNotFoundException, SQLException {
        System.out.println(getConn("jdbc:mysql://10.10.16.11:3306/auto?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true", "jiji", "jiji"));
        System.out.println(getConn());
    }
}
```

获得URLClassLoader对象后，调用loanClass()方法来加载指定的类

##### 自定义类加载器

JVM中除了根类加载器之外的所有类的加载器都是ClassLoader子类的实例，通过重写ClassLoader中的方法，实现自定义的类加载器，通常我们会使用以下两个方法中的一个（通常第二个）

- loadClass(String name,boolean resolve):为ClassLoader的入口点，根据指定名称来加载类，系统就是调用ClassLoader的该方法来获取制定累对应的Class对象

- findClass(String name):根据指定名称来查找类

代码示例：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.lang.reflect.Method;

public class CompileClassLoader extends ClassLoader

{

    // 读取一个文件的内容

    @SuppressWarnings("resource")
    private byte[] getBytes(String filename) throws IOException

    {

        File file = new File(filename);

        long len = file.length();

        byte[] raw = new byte[(int) len];

        FileInputStream fin = new FileInputStream(file);

        // 一次读取class文件的全部二进制数据

        int r = fin.read(raw);

        if (r != len)

            throw new IOException("无法读取全部文件" + r + "!=" + len);

        fin.close();
        return raw;

    }

    // 定义编译指定java文件的方法

    private boolean compile(String javaFile) throws IOException

    {

        System.out.println("CompileClassLoader:正在编译" + javaFile + "……..");

        // 调用系统的javac命令

        Process p = Runtime.getRuntime().exec("javac" + javaFile);

        try {

            // 其它线程都等待这个线程完成

            p.waitFor();

        } catch (InterruptedException ie)

        {

            System.out.println(ie);

        }

        // 获取javac 的线程的退出值

        int ret = p.exitValue();

        // 返回编译是否成功

        return ret == 0;

    }

    // 重写Classloader的findCLass方法

    protected Class<?> findClass(String name) throws ClassNotFoundException

    {

        Class clazz = null;

        // 将包路径中的.替换成斜线/

        String fileStub = name.replace(".", "/");

        String javaFilename = fileStub + ".java";

        String classFilename = fileStub + ".class";

        File javaFile = new File(javaFilename);

        File classFile = new File(classFilename);

        // 当指定Java源文件存在，且class文件不存在，或者Java源文件的修改时间比class文件//修改时间晚时，重新编译

        if (javaFile.exists() && (!classFile.exists())
                || javaFile.lastModified() > classFile.lastModified())

        {

            try {

                // 如果编译失败，或该Class文件不存在

                if (!compile(javaFilename) || !classFile.exists())

                {

                    throw new ClassNotFoundException("ClassNotFoundException:"
                            + javaFilename);

                }

            } catch (IOException ex)

            {

                ex.printStackTrace();

            }

        }

        // 如果class文件存在，系统负责将该文件转化成class对象

        if (classFile.exists())

        {

            try {

                // 将class文件的二进制数据读入数组

                byte[] raw = getBytes(classFilename);

                // 调用Classloader的defineClass方法将二进制数据转换成class对象

                clazz = defineClass(name, raw, 0, raw.length);

            } catch (IOException ie)

            {

                ie.printStackTrace();

            }

        }

        // 如果claszz为null,表明加载失败，则抛出异常

        if (clazz == null) {

            throw new ClassNotFoundException(name);

        }

        return clazz;

    }

    // 定义一个主方法

    public static void main(String[] args) throws Exception

    {

        // 如果运行该程序时没有参数，即没有目标类

        if (args.length < 1) {

            System.out.println("缺少运行的目标类，请按如下格式运行java源文件：");

            System.out.println("java CompileClassLoader ClassName");

        }

        // 第一个参数是需要运行的类

        String progClass = args[0];

        // 剩下的参数将作为运行目标类时的参数，所以将这些参数复制到一个新数组中

        String progargs[] = new String[args.length - 1];

        System.arraycopy(args, 1, progargs, 0, progargs.length);

        CompileClassLoader cl = new CompileClassLoader();

        // 加载需要运行的类

        Class<?> clazz = cl.loadClass(progClass);

        // 获取需要运行的类的主方法

        Method main = clazz.getMethod("main", (new String[0]).getClass());

        Object argsArray[] = { progargs };

        main.invoke(null, argsArray);

    }

}
```

##### 类的连接

当类被加载后，系统会为之生成一个Class对象，接着将会进入连接阶段，链接阶段负责把类的二进制数据合并到JRE中。它分为三个阶段：

- 验证：检验被加载的类是否有正确的内部结构，并和其他类协调一致
- 准备：负责为类的类变量分配内存。并设置默认初始值
- 解析：将类的二进制数据中的符号引用替换成直接引用

如下图：

{% asset_img Java类从加载到初始化的过程.png Java类从加载到初始化的过程 %}

##### 类的初始化

JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式：

- 声明类变量是指定初始值
- 使用静态代码块为类变量指定初始值

JVM初始化步骤

- 假如这个类还没有被加载和连接，则程序先加载并连接该类
- 假如该类的直接父类还没有被初始化，则先初始化其直接父类
- 假如类中有初始化语句，则系统依次执行这些初始化语句

类初始化时机

- 创建类实例。也就是new的方式
- 调用某个类的类方法
- 访问某个类或接口的类变量，或为该类变量赋值
- 使用反射方式强制创建某个类或接口对应的java.lang.Class对象
- 初始化某个类的子类，则其父类也会被初始化
- 直接使用java.exe命令来运行某个主类

### 三 反射

要想获得运行时对象和类的真实信息通常有两种方式：

- 第一种是在编译时和运行时都完全知道类型的具体信息，这种情况下，可以通过instanceof运算符进行判断，再使用强制类型转换将其转化成运行时类型的变量即可
- 第二种是编译时根本无法获知对象和类的类型信息，这种情况下只能通过反射来获知运行时对象和类的真实信息

##### 获取Class对象

通常有以下3种方法：

- 使用Class类的forName(String name)方法，如Class.forName("Person")获得person的Class对象
- 使用某个类的class属性，如Person.class
- 使用某个对象的getClass()方法，如Person.getClass()

通常使用第二种方法。

##### 从Class对象中获取信息

Java的java.lang.Class类提供了大量的实例方法来获得某个类具体信息，如成员变量，方法，构造器等。具体方法请参考java.lang.Class的源码，这里不再赘述。

##### 使用反射生成并操作对象

通过反射来生成对象有两种方式：

- 通过Class对象的newInstance()方法来获取Class对象对应类的实例。但是该方法需要这个对应类有默认构造函数，执行newInstance()方法实际上就是通过默认构造方法获取类的实例。
- 通过Class对象的获取指定的Constructor对象，在调用Constructor对象的newInstance()来获取Class对应类的实例。

Java的java.lang.reflect包下有对应的Constructor，Field，Method，Array等类，来通过反射获取相应类的具体信息。（Array用来动态的创建数组）

### 四 动态代理

##### JDK动态代理

在Java的java.lang.refect包下有一个Proxy类和一个InvocationHandler接口，可以实现动态代理。

Proxy是所有动态代理类的父类，它可以用于动态代理类和动态代理对象。如：

- static Class<?> getProxyClass(ClassLoader loader, Class<?>...interfaces):创建一个动态代理类对应的Class对象该代理类将实现interfaces所指定的多个接口，第一个参数指生成该动态代理类的类加载器。
- static Object newProxyInstance(ClassLoader loader, Class<?>...interfaces,InvocationHandler h):创建动态代理对象，**执行代理对象的每个方法时都会被替换成执行InvocationHandler的invoke方法**。

实际上即使采用第一种方法创建动态代理类后，如果需要创建动态代理实例，也需要传入一个InvocationHandler对象，每个动态代理实例都有一个与之对应的InvocationHandler对象。

代码示例：

被代理的接口即其实现类

```java
public interface Subject
{
    public void rent();

    public void hello(String str);
}

public class RealSubject implements Subject
{
    @Override
    public void rent()
    {
        System.out.println("I want to rent my house");
    }

    @Override
    public void hello(String str)
    {
        System.out.println("hello: " + str);
    }
}
```

代理类：

```java
public class DynamicProxy implements InvocationHandler
{
    //　这个就是我们要代理的真实对象
    private Object subject;

    //    构造方法，给我们要代理的真实对象赋初值
    public DynamicProxy(Object subject)
    {
        this.subject = subject;
    }

    @Override
    public Object invoke(Object object, Method method, Object[] args)
            throws Throwable
    {
        //　　在代理真实对象前我们可以添加一些自己的操作
        System.out.println("before rent house");

        System.out.println("Method:" + method);

        //    当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
        method.invoke(subject, args);

        //　　在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("after rent house");

        return null;
    }

}
```

主程序：

```java
public static void main(String[] args)
    {
        //    我们要代理的真实对象
        Subject realSubject = new RealSubject();

        //    我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
        InvocationHandler handler = new DynamicProxy(realSubject);

        /*
         * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
         * 第一个参数 handler.getClass().getClassLoader() ，我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
         * 第二个参数realSubject.getClass().getInterfaces()，我们这里为代理对象提供的接口是真实对象所实行的接口，表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
         * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 InvocationHandler 这个对象上
         */
        Subject subject = (Subject)Proxy.newProxyInstance(handler.getClass().getClassLoader(), realSubject
                .getClass().getInterfaces(), handler);

        System.out.println(subject.getClass().getName());
        subject.rent();
        subject.hello("world");
    }
```

##### CGLIB代理

JDK的动态代理只能是面向接口的，要想面向类可以使用CGLIB代理。
