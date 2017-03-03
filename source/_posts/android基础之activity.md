---
title: android基础之activity
date: 2017-02-27 10:35:34
categories: android
tags:
- java
- android
- activity
---

### 一 Activity简介   
activity是android四大组件之一，用于android app的界面显示。activity中包含一个window，window中则包含包含相应的view。

### 二 Activity类的层次结构    
继承的抽象类： ContextThemeWrapper
实现的接口： ComponentCallbacks2 KeyEvent.Callback LayoutInflater.Factory2 View.OnCreateContextMenuListener Window.Callback      

```java   
java.lang.Object   
	android.content.Context
		android.content.ContextWrapper
			android.content.ContextThemeWrapper
				android.app.Activity
```

直接子类： AccountAuthenticatorActivity, ActivityGroup, AliasActivity, ExpandableListActivity, FragmentActivity, ListActivity, NativeActivity 　　　
间接子类： ActionBarActivity, LauncherActivity, PreferenceActivity, TabActivity 　　　

### 三 Activity的生命周期   　
Activity在它的一生中有以下四种状态：  
- running：此时Activity处于屏幕的前台并且拥有焦点，位于回退栈的顶部（回退栈将在下面介绍）。   
- paused： 此时另一个Activity位于前台并拥有焦点，也就是说另一个Activity覆盖在本Activity的上面，但是这个Activity是透明的或者没有占据全屏，即从屏幕上还可以看到下面的Activity（比如弹出个红包）。一个paused的activity是完全存活的（Activity 对象仍然保留在内存里，它保持着所有的状态和成员信息，并且保持与window manager的联接），但在系统内存严重不足的情况下它能被杀死。    
- stopped： 本Activity被其他Activity完全覆盖了，看不见了。一个stopped的activity也仍然是存活的（Activity 对象仍然保留在内存中，它保持着所有的状态和成员信息，但是不再与window manager联接了）。 但是，对于用户而言它已经不再可见了，并且当其它地方需要内存时它将会被杀死。    
- killed： 当Activity处于paused或者stopped时，若内存不足，它将可能被干掉。当它被再次创建处于前台时，需要调用onCreate方法重新创建并恢复之前的状态    
下面是activity的生命周期图：      
{% asset_img activity生命周期图.png activity生命周期图 %}       
由上图可以看出activity有三种不同的生命周期：　　
- 完整生命周期： onCreate--onDestroy    
- 可见生命周期： onResume到onPause之间循环    
- 前台生命周期： onStart-onStop-onRestart三者之间循环    
Activity中的回调方法    

```java
public class ExampleActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // The activity is being created.
    }
    @Override
    protected void onStart() {
        super.onStart();
        // The activity is about to become visible.
    }
    @Override
    protected void onResume() {
        super.onResume();
        // The activity has become visible (it is now "resumed").
    }
    @Override
    protected void onPause() {
        super.onPause();
        // Another activity is taking focus (this activity is about to be "paused").
    }
    @Override
    protected void onStop() {
        super.onStop();
        // The activity is no longer visible (it is now "stopped")
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // The activity is about to be destroyed.
    }
}
```
 
Activity回调方法汇总：       
{% asset_img activity回调方法汇总.png activity回调方法汇总 %}       
标为“之后可否被杀死？”的列指明了系统是否可以在这个方法返回之后的任意时刻杀掉这个activity的宿主进程， 而不再执行其它流程上的activity代码。 有三个方法是标为“可以”：（ onPause()、 onStop()、 和onDestroy()）。 因为onPause()是三个方法中的第一个， 一旦activity被创建， onPause() 就是进程可以被杀死之前最后一个能确保被调用的方法 ——如果系统在某种紧急情况下必须回收内存，则 onStop() 和onDestroy() 可能就不会被调用了。因此，你应该使用 onPause() 来把至关重要的需长期保存的数据写入存储器（比如用户所编辑的内容）。 不过，应该对必须通过 onPause() 方法进行保存的信息有所选择，因为该方法中所有的阻塞操作都会让切换到下一个activity的停滞，并使用户感觉到迟缓。

“之后可否被杀死？”列中标为“否”的方法，在它们被调用时的那一刻起，就会保护本activity的宿主进程不被杀掉。 因此，只有在 onPause() 方法返回时至 onResume() 方法被调用时之间，activity才会被杀掉。直到 onPause() 再次被调用并返回时，activity都不会再次允许被杀死。

Note:表1中标明的技术上不“可杀”的activity仍然有可能会被系统杀死——但那只有在没有任何资源的极端情况下才会发生。     

### 四 保存Activity的状态    
上一节中已简单提到，当一个activity被paused或者stopped时，activity的状态可以被保存。 的确如此，因为 Activity 对象在paused或者stopped时仍然被保留在内存之中——它所有的成员信息和当前状态都仍然存活。 这样用户在activity里所作的改动全都还保存着，所以当activity返回到前台时（当它“resume“），那些改动仍然有效。

不过，如果系统是为了回收内存而销毁activity，则这个 Activity 对象就会被销毁，这样系统就无法简单地resume一下就能还原完整状态的activity。 如果用户要返回到这个activity的话，系统必须重新创建这个Activity 对象。可是用户并不知道系统是先销毁activity再重新创建了它的，所以，他很可能希望activity完全保持原样。 这种情况下，你可以保证activity状态的相关重要信息都由另一个回调方法保存下来了，此方法让你能保存activity状态的相关信息： onSaveInstanceState()。

在activity变得很容易被销毁之前，系统会调用 onSaveInstanceState()方法。 调用时系统会传入一个Bundle对象， 你可以利用 putString() 之类的方法，以键值对的方式来把activity状态信息保存到该Bundle对象中。 然后，如果系统杀掉了你的application进程并且用户又返回到你的activity，系统就会重建activity并将这个 Bundle 传入onCreate() 和onRestoreInstanceState() 中，你就可以从 Bundle 中解析出已保存信息并恢复activity状态。如果没有储存状态信息，那么传入的 Bundle 将为null（当activity第一次被创建时就是如此）。     
下图是Activity状态保存示意图：      
{% asset_img Activity状态保存示意图.png activity状态保存示意图 %}  
### 五 配置改动后的处理   
设备的某些配置可能会在运行时发生变化（比如屏幕方向、键盘可用性以及语言）。 当发生这些变化时，Android会重建这个运行中的activity（系统会调用 onDestroy() ，然后再马上调用 onCreate() ）。这种设计有助于应用程序适用新的参数配置，通过把你预置的可替换资源（比如对应各种屏幕方向和尺寸的layout）自动重新装载进入应用程序的方式来实现。

如果你采取了适当的设计，让activity能够正确地处理这些因为屏幕方向而引起的重启，并能如上所述地恢复activity状态， 那么你的应用程序将对生命周期中其它的意外事件更具适应能力。

处理这类重启的最佳方式，就是利用 onSaveInstanceState() 和onRestoreInstanceState() （或者 onCreate() ）进行状态的保存和恢复，如上节所述。   

### 六 Activity的启动方式   
antivity的启动方式可以通过两种方式定义：     

##### Androidmanifest文件       
Androidmanifest文件是android的清单文件，android的四大组件与相应权限皆需在这里设置。在这里你可以利用 <activity> 元素的 launchMode 属性来设定 activity 与 task 的关系。       
可通过Androidmanifest文件设置的启动模式有：      
- standard
- singleTop
- singleTask
- singleInstance       

##### Intent标志     
Intent标志中有以下几种Activity的启动方式:       
- FLAG_ACTIVITY_NEW_TASK
- FLAG_ACTIVITY_SINGLE_TOP
- FLAG_ACTIVITY_CLEAR_TOP       
通过Intent标志的方式来启动Activity，优先级比manifest的高。    

> **警告**： 大多数应用不应该改变 activity 和 task 默认的工作方式。 如果你确定有必要修改默认方式，请保持
谨慎，并确保 activity 在启动和从其它 activity 用回退键返回时的可用性。 请确保对可能与用户预期的导航方式
相冲突的地方进行测试。      

### 七 启动Activity  
要启动一个activity需要使用Intent，实际上android中组件之前的通信皆由Intent完成（如果你不了解Intent，请看我之后的博客[android基础之Intents与Intent-Filters.md](android基础之Intents与Intent-Filters.md "android基础之Intents与Intent-Filters")），而使用Intent启动activity有显示和隐式两种方式。   


##### 显示启动      

     
```java
Intent intent = new Intent(this, SignInActivity.class);
startActivity(intent);//显示启动一个叫SignInActivity的Activity    

```   



#####  隐式启动    
要想隐式启动一个activity，你需要使用intent-filter来暴露你的activity。使用filter有java代码和xml文件两种方式，推荐使用第二种方式，你只需在Androidmanifest.xml文件中的activit元素中加入intent-filter属性，并声明一个action即可。但是如果你想使你的activity更独立，那么就不要为你的activity设置filter。那么什么时候使用这种方式呢？    
如果，你的程序可能想要展示某些动作，例如发邮件，短信，微博，或者使用你activity中的数据。 这时候，你就不应该使用自己的activity来做这些工作。你应该调用系统中其他程序提供的响应功能。 这是intent真正体现其价值的地方。你可以创建一个描述了响应动作的intent,然后系统来为你挑选完成任务的程序。 如果有多个选择，系统会提示用户进行选择。例如你想让用户发邮件，你可以创建下面的intent：        


```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);//recipientArray即你想发送过去的联系人信息
startActivity(intent);    

```       

##### 启动一个带返回结果的Activity    
有时候，你想要启动一个activity,并从这个activty获得一个结果。 这时，要通过 startActivityForResult() (取代startActivity()) 来启动activity。 然后通过实现onActivityResult()回调方法来获得返回后的结果。 当这个后续的activity被关闭，它将发送一个 Intent 给 onActivityResult() 方法。     
  
例如，你可能想要取一个联系人的信息。下面介绍怎么创建intent并处理结果:         


```java
private void pickContact() {
    // Create an intent to "pick" a contact, as defined by the content provider URI
    Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
    startActivityForResult(intent, PICK_CONTACT_REQUEST);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
    if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
        // Perform a query to the contact's content provider for the contact's name
        Cursor cursor = getContentResolver().query(data.getData(),
        new String[] {Contacts.DISPLAY_NAME}, null, null, null);
        if (cursor.moveToFirst()) { // True if the cursor is not empty
            int columnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
            String name = cursor.getString(columnIndex);
            // Do something with the selected contact's name...
        }
    }
}       

```      


这个例子展示了使用onActivityResult() 来获取结果的基本方法。 第一步要判断请求是否被成功响应，通过判断resultCode 是不是RESULT_OK—， 然后判断这个响应是不是针对相应的请求— ，此时只要判断requestCode 和发送时提供的第二个参数 startActivityForResult() 是否相匹配。 最后，查询 Intent中的data信息。 (data 参数)。

这个过程中，ContentResolver 开启了一个查询而不是content provider, 它返回一个 Cursor ，这将允许数据被读取。更多content provider相关信息，请阅读我之后的博客[android基础之Content-Providers.md](android基础之Content-Providers.md "android基础之Content-Providers")。     

##### 关闭activity    
你可以通过调用finish() 来终止activity。 你也可以调用finishActivity() 来终止你之前启动了的一个独立activity。    

> **注意**: 多数情况下，你不应该明确地通过这些方式来关闭acitivity。 就像下面要讨论的activity的生命周期。系统会为你管理。所以你不必关闭他们。 调用这些方法将有悖于用户体验。它们仅用于你绝对不想让用户再返回这个activity的实例。   


### 八 Task和back stack    
一个app通常包含多个activity，android使用task来管理这些activity。当你在手机上没点开一个app，android即为这个app创建一个task，task中有一个back stack，你多点开的activity将依次入栈或者出栈，若你的activity调用别的应用的activity，它也会被压入本activity中。（activity与back stack的关系可阅读上文的activity的启动方式，它定义了activity在back stack中的存在方式）。    
一个app即对应一个task，若你点开一个新的app，则该新app会进入前台，而之前的会被调入后台。当android的内存不足时，后台的task则可能被销毁。    

其实，activity所处的task可以通过<activity>元素的taskAffinity属性指定，taskAffinity 属性是一个字符串值，必须与<manifest> 元素定义的包名称保证唯一性，因为系统把这个包名称用于标识应用的默认 task affinity值。但是，注意，一般情况下如无必要，不要去修改taskAffinity的值。    
### 九 总结    
到这里，android的activity相关的知识已经介绍的差不多了，但是这其中还有很多细节需要读者自己去发现。我所说的并不全面，有些地方可能还有谬误，万不可全信，这些仅仅是我个人的理解。其实博客的功能也只是让你对一个陌生的事物知道个大概，要想最终精通还需要多写多做。这些东西你只有自己经历了，研究了它才能成为你自己的东西，否则不过是镜花水月。   
作为一个程序员最重要的是勤劳，最后以一句话与大家共勉：慈不掌兵，义不行贾，懒不撸码。
   

