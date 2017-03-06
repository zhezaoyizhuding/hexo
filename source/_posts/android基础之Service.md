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

##### 1.startService()启动    
通过startService()启动的Service，Service一旦启动即与它的调用者（通常是某个activity）不再关联，即使调用它的组件被销毁，它也能在后台一直运行下去。通常，该类服务执行单一的操作并且不会向调用者返回结果，比如它可以通过网络下载或上传文件。此类服务应该在其工作完成后通过重写stopSelf()使其自行终止，或者由其他组件调用stopService()来终止。       
通过startService()启动的服务需要实现它的生命周期中onStartCommand()方法        



##### 2.bindService()启动      
通过bindService()启动的服务，该服务的生命周期将与启动它的组件相关联，即如果启动它的组件被销毁则该服务也会被销毁，或者通过调用unBindService()与服务解绑，解绑后服务被销毁。多个组件可以同时与一个服务绑定，当所有组件都与服务解绑后，该服务才会被销毁。    
bind服务提供了一个客户端/服务器接口，允许组件与服务进行交互、发送请求、获取结果，甚至进行进程间通信（IPC）。       
通过bindService()启动的服务需要实现Service中的onBind()方法。   




### 三 Service的生命周期     
两种Service的生命周期如下图：     
{% asset_img Service生命周期图.png Service生命周期图 %}     
其回调方法如下：     

   
```java       
public class ExampleService extends Service {
    int mStartMode;       // indicates how to behave if the service is killed
    IBinder mBinder;      // interface for clients that bind
    boolean mAllowRebind; // indicates whether onRebind should be used

    @Override
    public void onCreate() {
        // The service is being created
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }
    @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService()
        return mBinder;
    }
    @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }
    @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }
    @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```         



### 四 Service与线程的区别    
- Thread：Thread 是程序执行的最小单元，它是分配CPU的基本单位。可以用 Thread 来执行一些异步的操作。     
- Service：Service 是android的一种机制，当它运行的时候如果是Local Service，那么对应的 Service 是运行在主进程的 main 线程上的。如：onCreate，onStart 这些函数在被系统调用的时候都是在主进程的 main 线程上运行的。如果是Remote Service，那么对应的 Service 则是运行在独立进程的 main 线程上。       
其实谈Service和线程的区别毫无意义，它们本来就是毫无关联的两样东西，我觉得应该谈的是在什么情况下，什么地方使用线程更合适。   
比如：如果你的线程主要用于对UI的更新，那么在Activity中新建即可。但是如果你的线程用于app和服务器之间的消息推送，你需要隔一段时间就要连接服务器做数据同步的话，那么就不能在Activity中新建线程。因为此时的线程在Activity被销毁时，它仍然需要运行。此时将面对一个问题，如果你是Activity中创建得线程，那么当该Activity被销毁时，你将失去该线程的实例。另一方面，你没办法在不同的Activity中对同一线程进行控制。这种情况你就需要在Service中创建它，一方面Service在Activity被销毁时仍然可以运行，另一方面其他的Activity重建后也可以通过与Service进行绑定获得Service的实例，进而控制该线程。

### 五 保证Service不被杀死    
- onStartCommand方法，返回START_STICKY   
- 提升service优先级   
- 提升service进程优先级,将其提升为前台服务    
- onDestroy方法里重启service  
- 监听系统广播判断Service状态    
- 将APK安装到/system/app，变身系统级应用      


### 六 IntentService    

因为大多数started服务都不需要同时处理多个请求（这实际上是一个危险的多线程情况），所以最佳方式也许就是用IntentService类来实现你的服务。   
IntentService将执行以下步骤：    　　
- 创建一个缺省的工作（worker）线程，它独立于应用程序主线程来执行所有发送到onStartCommand()的intent。　　　
- 创建一个工作队列，每次向你的onHandleIntent()传入一个intent，这样你就永远不必担心多线程问题了。　
- 在处理完所有的启动请求后，终止服务，因此你就永远不需调用stopSelf()了。　　
- 提供缺省的onBind()实现代码，它返回null。
- 提供缺省的onStartCommand()实现代码，它把intent送入工作队列，稍后会再传给你的onHandleIntent()实现代码。　　　

以上所有步骤将汇成一个结果：你要做的全部工作就是实现onHandleIntent()的代码，来完成客户端提交的任务。（当然你还需要为服务提供一小段构造方法。）       

以下是个IntentService的实现例程：    

```java    

public class HelloIntentService extends IntentService {

  /** 
   * 构造方法是必需的，必须用工作线程名称作为参数
   * 调用父类的[http://developer.android.com/reference/android/app/IntentService.html#IntentService(java.lang.String) IntentService(String)]构造方法。
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * IntentService从缺省的工作线程中调用本方法，并用启动服务的intent作为参数。 
   * 本方法返回后，IntentService将适时终止这个服务。
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // 通常我们会在这里执行一些工作，比如下载文件。
      // 作为例子，我们只是睡5秒钟。
      long endTime = System.currentTimeMillis() + 5*1000;
      while (System.currentTimeMillis() < endTime) {
          synchronized (this) {
              try {
                  wait(endTime - System.currentTimeMillis());
              } catch (Exception e) {
              }
          }
      }
  }
}  

```    

所有你需要做的就是：一个构造方法和一个onHandleIntent()方法的实现。

如果你还决定重写其它的回调方法，比如onCreate()、onStartCommand()、onDestroy()， 请确保调用一下父类的实现代码，以便IntentService能正确处理工作线程的生命周期。

比如说，onStartCommand()必须返回缺省实现代码的结果（缺省代码实现了如何获取传给onHandleIntent()的intent）：    

```java  
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
    return super.onStartCommand(intent,flags,startId);
}
```   

除了onHandleIntent()以外，唯一不需要调用父类实现代码的方法是onBind()（不过如果你的服务允许绑定，你还是需要实现它）。    


### 七 扩展Service   

如上节所述，利用IntentService来实现一个started服务非常简单。 不过，假如你的服务需要多线程运行（而不是通过一个工作队列来处理启动请求），那你可以扩展Service类来完成每个intent的处理。

作为对照，以下例程实现了 Service 类，它执行的工作与上述使用IntentService的例子相同。确切地说，对于每一个启动请求，它都用一个工作线程来完成处理工作，并且每次只处理一个请求。    

```java    
public class HelloService extends Service {
  private Looper mServiceLooper;
  private ServiceHandler mServiceHandler;

  // 处理从线程接收的消息
  private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
          super(looper);
      }
      @Override
      public void handleMessage(Message msg) {
          // 通常我们在这里执行一些工作，比如下载文件。
          // 作为例子，我们只是睡个5秒钟。
          long endTime = System.currentTimeMillis() + 5*1000;
          while (System.currentTimeMillis() < endTime) {
              synchronized (this) {
                  try {
                      wait(endTime - System.currentTimeMillis());
                  } catch (Exception e) {
                  }
              }
          }
          // 根据startId终止服务，这样我们就不会在处理其它工作的过程中再来终止服务
          stopSelf(msg.arg1);
      }
  }

  @Override
  public void onCreate() {
    // 启动运行服务的线程。
    // 请记住我们要创建一个单独的线程，因为服务通常运行于进程的主线程中，可我们不想阻塞主线程。
    // 我们还要赋予它后台运行的优先级，以便计算密集的工作不会干扰我们的UI。
    HandlerThread thread = new HandlerThread("ServiceStartArguments",
            Process.THREAD_PRIORITY_BACKGROUND);
    thread.start();
    
    // 获得HandlerThread的Looper队列并用于Handler
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
  }

  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();

      // 对于每一个启动请求，都发送一个消息来启动一个处理
      // 同时传入启动ID，以便任务完成后我们知道该终止哪一个请求。
      Message msg = mServiceHandler.obtainMessage();
      msg.arg1 = startId;
      mServiceHandler.sendMessage(msg);
      
      // 如果我们被杀死了，那从这里返回之后被重启
      return START_STICKY;
  }

  @Override
  public IBinder onBind(Intent intent) {
      // 我们不支持绑定，所以返回null
      return null;
  }
  
  @Override
  public void onDestroy() {
    Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show(); 
  }
}
```    

如你所见，它要干的事情比用IntentService时多了很多。

不过，因为是自行处理每个onStartCommand()调用，你可以同时处理多个请求。 本例中没有这么去实现，但只要你愿意，你就可以为每个请求创建一个新的线程，并立即运行它们（而不是等待前一个请求处理完毕）。

请注意onStartCommand()方法必须返回一个整数。这个整数是描述系统在杀死服务之后应该如何继续运行（上一节中缺省的 IntentService 实现代码会替你处理这一点，当然那样你就无法修改这个处理过程）。onStartCommand()的返回值必须是以下常量之一：


- START_NOT_STICKY: 如果系统在onStartCommand()返回后杀死了服务，则不会重建服务了，除非还存在未发送的intent。 当服务不再是必需的，并且应用程序能够简单地重启那些未完成的工作时，这是避免服务运行的最安全的选项。 


- START_STICKY: 如果系统在onStartCommand()返回后杀死了服务，则将重建服务并调用onStartCommand()，但不会再次送入上一个intent， 而是用null intent来调用onStartCommand() 。除非还有启动服务的intent未发送完，那么这些剩下的intent会继续发送。 这适用于媒体播放器（或类似服务），它们不执行命令，但需要一直运行并随时待命。 


- START_REDELIVER_INTENT: 如果系统在onStartCommand()返回后杀死了服务，则将重建服务并用上一个已送过的intent调用onStartCommand()。任何未发送完的intent也都会依次送入。这适用于那些需要立即恢复工作的活跃服务，比如下载文件。    