---
title: android基础之Handler与AsycTask
date: 2017-02-27 14:47:53
categories: android
tags:
- android
- java
- Handler
- AsycTask
---

### 一 概述

Handler是Android中用于线程间通信的机制。

当程序第一次启动的时候，Android会同时启动一条主线程（ Main Thread）来负责处理与UI相关的事件，我们叫做UI线程。Android的UI操作并不是线程安全的（出于性能优化考虑），意味着如果多个线程并发操作UI线程，可能导致线程安全问题。为了解决Android应用多线程问题—Android平台只允许UI线程修改Activity里的UI，这样如果在主线程中执行耗时操作就会导致UI阻塞，若超过5秒就会造成ANR。

因此我们需要另开线程来处理这些耗时操作，而处理的结果我们可能需要传给主线程用于UI更新，这是就需要线程间的通信。Handler即是Android中用于线程间通信的机制，现在市场一些android的线程通信框架基本上都是基于Handler实现的，如下文将要介绍的已经置于android源码中AsycTask，和现在比较流行的EventBus。

### 二 Handler的主要作用

通过翻看的Handler的源码可知，Handler主要有两个作用。

##### 1.线程延时

Handler中内置了线程延时的方法：

- final boolean postAtTime(Runnable r, long uptimeMillis)
- final boolean postDelayed(Runnable r, long delayMillis)

##### 2.线程通信

主要步骤：

- 在新启动的线程中发送消息     
使用Handler对象的sendMessage()方法或者SendEmptyMessage()方法发送消息。   

- 在主线程中获取处理消息   
重写Handler类中处理消息的方法（void handleMessage(Message msg)），当新启动的线程发送消息时，消息发送到与之关联的MessageQueue。而Hanlder不断地从MessageQueue中获取并处理消息。

### 三 Handler与UI线程通信示例

- 首先要进行Handler 申明，复写handleMessage方法( 放在主线程中)

```java
private Handler handler = new Handler() {

        @Override
        public void handleMessage(Message msg) {
            // TODO 接收消息并且去更新UI线程上的控件内容
            if (msg.what == UPDATE) {
                // 更新界面上的textview
                tv.setText(String.valueOf(msg.obj));
            }
            super.handleMessage(msg);
        }
    };
```

- 子线程发送Message给ui线程表示自己任务已经执行完成，主线程可以做相应的操作了。

```java
new Thread() {
            @Override
            public void run() {
                // TODO 子线程中通过handler发送消息给handler接收，由handler去更新TextView的值
                try {
                       //do something

                        Message msg = new Message();
                        msg.what = UPDATE;                  
                        msg.obj = "更新后的值" ;
                        handler.sendMessage(msg);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }.start();
```

### 四 Handler原理分析

##### 1.Handler的构造函数

1. public　Handler() 
2. public　Handler(Callbackcallback)
3. public　Handler(Looperlooper)
4. public　Handler(Looperlooper, Callbackcallback) 


- 第1个和第2个构造函数都没有传递Looper，这两个构造函数都将通过调用Looper.myLooper()获取当前线程绑定的Looper对象，然后将该Looper对象保存到名为mLooper的成员字段中。 　   
下面来看1,2个函数源码：

```java
    public Handler() {
        this(null, false);
    }

    public Handler(Callback callback) {
        this(callback, false);
    }

    //他们会调用Handler的内部构造方法

    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
      final Class<? extends Handler> klass = getClass();
      if ((klass.isAnonymousClass() ||klass.isMemberClass()
         || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
     /************************************
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

通过Looper.myLooper()获取了当前线程保存的Looper实例，又通过这个Looper实例获取了其中保存的MessageQueue（消息队列）。每个Handler 对应一个Looper对象，产生一个MessageQueue

- 第3个和第4个构造函数传递了Looper对象，这两个构造函数会将该Looper保存到名为mLooper的成员字段中。 　   
下面来看3、4个函数源码： 

```java
    public Handler(Looper looper) {
        this(looper, null, false);
    }　

    public Handler(Looper looper, Callback callback) {
        this(looper, callback, false);
    }
   //他们会调用Handler的内部构造方法

    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

- 第2个和第4个构造函数还传递了Callback对象，Callback是Handler中的内部接口，需要实现其内部的handleMessage方法，Callback代码如下:

```java
     public interface Callback {
         public boolean More ...handleMessage(Message msg);
     }
```

Handler.Callback是用来处理Message的一种手段，如果没有传递该参数，那么就应该重写Handler的handleMessage方法，也就是说为了使得Handler能够处理Message，我们有两种办法： 　　 

　
1. 向Hanlder的构造函数传入一个Handler.Callback对象，并实现Handler.Callback的handleMessage方法  　    
2. 无需向Hanlder的构造函数传入Handler.Callback对象，但是需要重写Handler本身的handleMessage方法 　

也就是说无论哪种方式，我们都得通过某种方式实现handleMessage方法，这点与Java中对Thread的设计有异曲同工之处。

##### Handler发送消息的几个方法的源码

```java
   public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }

   public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageDelayed(msg, delayMillis);
    }

 public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

 public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }

```

我们可以看出他们最后都调用了sendMessageAtTime（），然后返回了enqueueMessage方法，下面看一下此方法源码：

```java
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
　　　　　　//把当前的handler作为msg的target属性
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

在该方法中有两件事需要注意： 

- msg.target = this   
该代码将Message的target绑定为当前的Handler
- queue.enqueueMessage   
变量queue表示的是Handler所绑定的消息队列MessageQueue，通过调用queue.enqueueMessage(msg, uptimeMillis)我们将Message放入到消息队列中。

### 五 Looper原理分析

我们一般在主线程申明Handler，有时我们需要继承Thread类实现自己的线程功能，当我们在里面申明Handler的时候会报错。其原因是主线程中已经实现了两个重要的Looper方法，下面看一看ActivityThread.java中main方法的源码：

```java
public static void main(String[] args) {
            //......省略
        Looper.prepareMainLooper();//>

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        AsyncTask.init();

        if (false) {
            Looper.myLooper().setMessageLogging(new
   LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();//>

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
}
```

首先看prepare()方法

```java
     public static void prepare() {
         prepare(true);
     }
 
     private static void prepare(boolean quitAllowed) {
　　　　　//证了一个线程中只有一个Looper实例
         if (sThreadLocal.get() != null) {
             throw new RuntimeException("Only one Looper may be created per thread");
         }
         sThreadLocal.set(new Looper(quitAllowed));
     }
```

该方法会调用Looper构造函数同时实例化出MessageQueue和当前thread.

```java
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    } 

    public static MessageQueue myQueue() {
        return myLooper().mQueue;
    }
```

prepare()方法中通过ThreadLocal对象实现Looper实例与线程的绑定。

再看loop()方法

```java
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
               
                return;
            }

            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
       //重点****
            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

首先looper对象不能为空，就是说loop()方法调用必须在prepare()方法的后面。

Looper一直在不断的从消息队列中通过MessageQueue的next方法获取Message，然后通过代码msg.target.dispatchMessage(msg)让该msg所绑定的Handler（Message.target）执行dispatchMessage方法以实现对Message的处理。 

Handler的dispatchMessage的源码如下：

```java
     public void dispatchMessage(Message msg) {
         if (msg.callback != null) {
             handleCallback(msg);
         } else {
             if (mCallback != null) {
                 if (mCallback.handleMessage(msg)) {
                     return;
                }
            }
            handleMessage(msg);
        }
    }
```

我们可以看到Handler提供了三种途径处理Message，而且处理有前后优先级之分：首先尝试让postXXX中传递的Runnable执行，其次尝试让Handler构造函数中传入的Callback的handleMessage方法处理，最后才是让Handler自身的handleMessage方法处理Message。

### 六 如何在子线程中使用Handler

Handler本质是从当前的线程中获取到Looper来监听和操作MessageQueue，当其他线程执行完成后回调当前线程。

子线程需要先prepare（）才能获取到Looper的，是因为在子线程只是一个普通的线程，其ThreadLoacl中没有设置过Looper，所以会抛出异常，而在Looper的prepare（）方法中sThreadLocal.set(new Looper())是设置了Looper的。

示例代码：

定义一个类实现Runnable接口或继承Thread类（一般不继承）。

```java
class Rub implements Runnable {  

        public Handler myHandler;  
        // 实现Runnable接口的线程体 
        @Override  
        public void run() {  

         /*①、调用Looper的prepare()方法为当前线程创建Looper对象并，
          创建Looper对象时，它的构造器会自动的创建相对应的MessageQueue*/
            Looper.prepare();  

            /*.②、创建Handler子类的实例，重写HandleMessage()方法，该方法处理除当前线程以外线程的消息*/
             myHandler = new Handler() {  
                @Override  
                public void handleMessage(Message msg) {  
                    String ms = "";  
                    if (msg.what == 0x777) {  

                    }  
                }  

            };  
            //③、调用Looper的loop()方法来启动Looper让消息队列转动起来
            Looper.loop();  
        }
    }
```

注意分成三步：

- 调用Looper的prepare()方法为当前线程创建Looper对象，创建Looper对象时，它的构造器会创建与之配套的MessageQueue。 　
- 有了Looper之后，创建Handler子类实例，重写HanderMessage()方法，该方法负责处理来自于其他线程的消息。
- 调用Looper的looper()方法启动Looper。然后使用这个handler实例在任何其他线程中发送消息，最终处理消息的代码都会在你创建Handler实例的线程中运行。

### 七 Handler总结

- Handler：     
发送和处理消息，它把消息发送给Looper管理的MessageQueue，并负责处理Looper分发给它的消息。
      
- Message：    
Handler接收和处理的消息对象。

- Looper：     
每个线程只有一个Looper，它负责管理对应的MessageQueue，会不断地从MessageQueue取出消息，并将消息分给对应的Hanlder处理。主线程中，系统已经初始化了一个Looper对象，因此可以直接创建Handler即可，就可以通过Handler来发送消息、处理消息。若是在子线程，必须手动创建一个Looper对象，并启动它，调用Looper.prepare()方法。

- prapare()：    
保证每个线程最多只有一个Looper对象。

- looper()：
启动Looper，使用一个死循环不断取出MessageQueue中的消息，并将取出的消息分给对应的Handler进行处理。 　

- MessageQueue：    
由Looper负责管理，它采用先进先出的方式来管理Message。Handler的构造方法，会首先得到当前线程中保存的Looper实例，进而与Looper实例中的MessageQueue想关联。Handler的sendMessage方法，会给msg的target赋值为handler自身，然后加入MessageQueue中。

### 八 Android中另一个线程通信机制AsycTask

##### 1.AsycTask简介

AsycTask是Android中另一个处理异步任务的机制，我们上面说了可以用Handler来处理异步任务，但如果耗时的操作太多，那么我们需要开启太多的子线程，这就会给系统带来巨大的负担，随之也会带来性能方面的问题。在这种情况下我们就可以考虑使用类AsyncTask来异步执行任务，不需要子线程和handler，就可以完成异步操作和刷新UI。

AsyncTask主要有二个部分：一个是与主线程的交互，另一个就是线程的管理调度。虽然可能多个AsyncTask的子类的实例，但是AsyncTask的内部Handler和ThreadPoolExecutor都是进程范围内共享的，其都是static的（所以AsycTask是串行的，不支持并发），也即属于类的，类的属性的作用范围是CLASSPATH，因为一个进程一个VM，所以是AsyncTask控制着进程范围内所有的子类实例。AsyncTask内部会创建一个进程作用域的线程池来管理要运行的任务，也就就是说当你调用了AsyncTask的execute()方法后，AsyncTask会把任务交给线程池，由线程池来管理创建Thread和运行Therad。

Android的AsyncTask比Handler更轻量级一些（只是代码上轻量一些，而实际上要比handler更耗资源），适用于简单的异步处理。

注意：不要随意使用AsyncTask,除非你必须要与UI线程交互.默认情况下使用Thread即可,要注意需要将线程优先级调低.AsyncTask适合处理短时间的操作,长时间的操作,比如下载一个很大的视频,这就需要你使用自己的线程来下载,不管是断点下载还是其它的.

##### 2.AsycTask使用步骤

AsyncTask对线程间的通讯做了包装，使后台线程和UI线程可以简易通讯。后台线程执行异步任务，将result告知UI线程。

使用AsycTask分为两步：

- 继承AsyncTask类实现自己的类    

```java
public abstract class AsyncTask<Params, Progress, Result> {

    /* Params: 输入参数，对应excute()方法中传递的参数。如果不需要传递参数，则直接设为void即可。

    ** Progress：后台任务执行的百分比

    ** Result：返回值类型，和doInBackground（）方法的返回值类型保持一致。*/
}
```

- 复写方法

最少要重写以下这两个方法：

**a.**doInBackground(Params…)      
在子线程（其他方法都在主线程执行）中执行比较耗时的操作，不能更新ＵＩ，可以在该方法中调用publishProgress(Progress…)来更新任务的进度。Progress方法是AsycTask中一个final方法只能调用不能重写。

**b.**onPostExecute(Result)     
使用在doInBackground 得到的结果处理操作UI， 在主线程执行，任务执行的结果作为此方法的参数返回。

有时根据需求还要实现以下三个方法：

**c.**onProgressUpdate(Progress…)     
可以使用进度条增加用户体验度。 此方法在主线程执行，用于显示任务执行的进度。

**d.**onPreExecute()     
这里是最终用户调用Excute时的接口，当任务执行之前开始调用此方法，可以在这里显示进度对话框。  

**e.**onCancelled()    
用户调用取消时，要做的操作

##### 3.AsycTask使用示例

按照上面的步骤定义自己的异步类：

```java
public class MyTask extends AsyncTask<String, Integer, String> {  
    //执行的第一个方法用于在执行后台任务前做一些UI操作  
    @Override  
    protected void onPreExecute() {  

    }  

    //第二个执行方法,在onPreExecute()后执行，用于后台任务,不可在此方法内修改UI
    @Override  
    protected String doInBackground(String... params) {  
         //处理耗时操作
        return "后台任务执行完毕";  
    }  

   /*这个函数在doInBackground调用publishProgress(int i)时触发，虽然调用时只有一个参数  
    但是这里取到的是一个数组,所以要用progesss[0]来取值  
    第n个参数就用progress[n]来取值   */
    @Override  
    protected void onProgressUpdate(Integer... progresses) {  
        //"loading..." + progresses[0] + "%"
        super.onProgressUpdate(progress);  
    }  

    /*doInBackground返回时触发，换句话说，就是doInBackground执行完后触发  
    这里的result就是上面doInBackground执行后的返回值，所以这里是"后台任务执行完毕"  */
    @Override  
    protected void onPostExecute(String result) { 

    }  

    //onCancelled方法用于在取消执行中的任务时更改UI  
    @Override  
    protected void onCancelled() {  

    }  
}
```

在主线程申明该类的对象，调用对象的execute（）函数开始执行。

```java
MyTask ｔ= new MyTask();
t.execute();//这里没有参数
```

##### 4.使用AsyncTask需要注意的地方 

- AsnycTask内部的Handler需要和主线程交互，所以AsyncTask的实例必须在UI线程中创建

- AsyncTaskResult的doInBackground(mParams)方法执行异步任务运行在子线程中，其他方法运行在主线程中，可以操作UI组件。

- 一个AsyncTask任务只能被执行一次。

- 运行中可以随时调用AsnycTask对象的cancel(boolean)方法取消任务，如果成功，调用isCancelled()会返回true，并且不会执行 onPostExecute() 方法了，而是执行 onCancelled() 方法。

- 对于想要立即开始执行的异步任务，要么直接使用Thread，要么单独创建线程池提供给AsyncTask。默认的AsyncTask不一定会立即执行你的任务，除非你提供给他一个单独的线程池。如果不与主线程交互，直接创建一个Thread就可以了。