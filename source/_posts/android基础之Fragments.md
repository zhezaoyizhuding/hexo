---
title: android基础之Fragments
date: 2017-02-27 14:31:31
categories: android
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
下面是Activity的生命周期：     
{% asset_img Fragment生命周期图.png Fragment生命周期图 %}    
管理fragment生命周期与管理activity生命周期很相像。像activity一样，fragment也有三种状态：    
- Resumed    
fragment在运行中的activity可见。    
- Paused    
 另一个activity处于前台且得到焦点，但是这个fragment所在的activity仍然可见（前台activity部分透明，或者没有覆盖全屏）。   
- Stopped   
fragment不可见。要么宿主activity已经停止，要么fragment已经从activity上移除，但已被添加到后台栈中。一个停止的fragment仍然活着（所有状态和成员信息仍然由系统保留着）。但是，它对用户来讲已经不再可见，并且如果activity被杀掉，它也将被杀掉。     
同activity类似的还有，你也可以用Bundle保存fragment状态，万一activity的进程被杀掉了，并且在activity被重新创建时，你需要恢复fragment状态。在回调执行fragment的onSaveInstanceState()期间可以保存状态，在onCreate()，onCreateView()，或者onActvityCreate()中可以恢复状态。      
在生命周期方面,activity与fragment之间一个很重要的不同，就是在各自的后台栈中是如何存储的。当activity停止时，默认情况下，activity被安置在由系统管理的activity后台栈中（因此用户可以按BACK键回退导航，就像在Tasks和后台栈中讨论的那样）。但是，仅当你在一个事务被移除时，通过显式调用addToBackStack()请求保存的实例，该fragment才被置于由宿主activity管理的后台栈。     
除此之外，管理fragment的生命周期与管理activity的生命周期非常相似。所以，管理activity生命周期的实践同样也适用于fragment。你需要了解的，仅仅是activity的生命周期如何影响fragment的的。    

### 五 与Activity生命周期协调合作 
下面是Activity与Fragment生命周期图对比：    
{% asset_img Fragment与Activity生命周期对比.png Fragment与Activity生命周期对比 %}      
fragment所生存的activity生命周期直接影响着fragment的生命周期，由此针对activity的每一个生命周期回调都会引发一个fragment类似的回调。例如，当activity接收到onPause()时，这个activity之中的每个fragment都会接收到onPause()。     
Fragment有一些额外的生命周期回调方法，然而，为了处理像是创建和销毁fragment界面，它与activity进行独特的交互。这些额外的回调方法是：     
- onAttach()     
 当fragment被绑定到activity时调用（Activity会被传入）。 
-  onCreateView()    
创建与fragment相关的视图体系时被调用。     
- onActivityCreated()     
当activity的onCreate()函数返回时被调用。     
- onDestroyView()     
当与fragment关联的视图体系正被移除时被调用。     
- onDetach()    
当fragment正与activity解除关联时被调用。      

fragment的生命周期流程实际上是受其宿主activity影响，如上图所示。在这张图中，可以看到activity的每个连续状态是如何决定fragment可能接收到哪个回调函数的。例如，当activity接收到它的onCreate()回调时，activity之中的fragment接收到的仅仅是onActivityCreated()回调。

一旦activity处于resumed状态，则可以在activity中自由的添加或者移除fragment。因此，只有当activity处于resumed状态时，fragment的生命周期才可以独立变化。

然而，当activity离开恢复状态时，fragment再一次被activity推入它的生命周期中。

### 六 向Fragment中添加用户界面    
fragment常被用作activity用户界面的一部分，并且将本身的布局构建到activity中去。     
为了给fragment提供一个布局，你必须实现onCreateView()回调函数，在绘制fragment布局时Android系统会调用它。实现这个函数时需要返回fragment所属的根View。     

> 注意：如果你的fragment时ListFragment的子类，默认实现从onCreateView()返回一个ListView，所以你不需要实现它。    
   
为了从onCreateView()返回一个布局，你可以从layout resource定义的XML文件inflate它。为了便于你这样做，onCreateView()提供一个LayoutInflater对象。   
例如，下面是一个fragment子类，它的布局从example_fragment.xml载入的：    

```java  
public static class ExampleFragment extends Fragment {
		@Override
		public View onCreateView(LayoutInflater inflater, ViewGroup container,
								 Bundle savedInstanceState) {
			// Inflate the layout for this fragment
			return inflater.inflate(R.layout.example_fragment, container, false);
		}
	}
```

传入onCreateView()的参数container 是你的frament布局将要被插入的父ViewGroup（来自activity的布局）。如果fragment处于resumed状态（恢复状态在操纵fragment生命周期一节中将作更多讨论），参数savedInstanceState是属于Bundle类，它提供了fragment之前实例的相关数据。     

inflate()函数需要以下三个参数：     
- 要inflate的布局的资源ID。      
- 被inflate的布局的父ViewGroup。传入container很重要，这是为了让系统将布局参数应用到被inflate的布局的根view中去，由其将要嵌入的父view指定。    
- 一个布尔值，表明在inflate期间被infalte的布局是否应该附上ViewGroup（第二个参数）。（在这个例子中传入的是false，因为系统已经将被inflate的布局插入到容器中（container）——传入true会在最终的布局里创建一个多余的ViewGroup。）       

现在你已经知道了如何创建一个有布局的fragment。下一步，则需要将fragment添加到activity中。      

### 七 将Fragment添加到Activity中    
通常，fragment构建了其宿主activity的部分界面，它被作为activity全部视图层次体系的一部分被嵌入进去。在acitivity布局中添加fragment有两种方法：     

##### 在activity的布局文件里声明fragment

像这样，你可以像为view一样为fragment指定布局属性。例如，下面含有两个fragment的布局文件：

```java
<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:orientation="horizontal"
		android:layout_width="match_parent"
		android:layout_height="match_parent">
		<fragment android:name="com.example.news.ArticleListFragment"
				android:id="@+id/list"
				android:layout_weight="1"
				android:layout_width="0dp"
				android:layout_height="match_parent" />
		<fragment android:name="com.example.news.ArticleReaderFragment"
				android:id="@+id/viewer"
				android:layout_weight="2"
				android:layout_width="0dp"
				android:layout_height="match_parent" />
	</LinearLayout>
```

<fragment>中的android:name 属性指定了布局中实例化的Fragment类。

当系统创建activity布局时，它实例化了布局文件中指定的每一个fragment，并为它们调用onCreateView()函数，以获取每一个fragment的布局。系统直接在<fragment>元素的位置插入fragment返回的View。

> 注意：每个fragment都需要一个唯一的标识，如果重启activity，系统可用来恢复fragment（并且可用来捕捉fragment的事务处理，例如移除）。为fragment提供ID有三种方法：   
- 用android:id属性提供一个唯一的标识。   
- 用android:tag属性提供一个唯一的字符串。   
- 如果上述两个属性都没有，系统会使用其容器视图（view）的ID。 

##### 通过java代码将fragment添加到已存在的ViewGroup中

在activity运行的任何时候，你都可以将fragment添加到activity布局中。你仅需要简单指定用来放置fragment的ViewGroup。

你应当使用FragmentTransaction的API来对activity中的fragment进行操作（例如添加，移除，或者替换fragment）。你可以像下面这样从Activity中取得FragmentTransaction的实例：

```java
FragmentManager fragmentManager = getFragmentManager()
	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
```

可以用add()函数添加fragment，并指定要添加的fragment以及要将其插入到哪个视图（view）之中：

```java
ExampleFragment fragment = new ExampleFragment();
	fragmentTransaction.add(R.id.fragment_container, fragment);
	fragmentTransaction.commit();
```

传入add()函数的第一个参数是fragment被放置的ViewGroup，它由资源ID（resource ID）指定，第二个参数就是要添加的fragment。一旦通过FragmentTransaction 做了更改，都应当使用commit()使变化生效。

##### 添加无界面的Fragment

上面的例子是如何将fragment添加到activity中去，目的是提供一个用户界面。然而，也可以使用fragment为activity提供后台动作，却不呈现多余的用户界面。

想要添加没有界面的fragment ，可以使用add(Fragment, String)（为fragment提供一个唯一的字符串“tag”，而不是视图（view）ID）。这样添加了fragment，但是，因为还没有关联到activity布局中的视图（view） ，收不到onCreateView()的调用。所以不需要实现这个方法。

想要添加没有界面的fragment ，可以使用add(Fragment, String)（为fragment提供一个唯一的字符串“tag”，而不是视图（view）ID）。这样添加了fragment，但是，因为还没有关联到activity布局中的视图（view） ，收不到onCreateView()的调用。所以不需要实现这个方法。  


### 八 Fragment事务后台栈     

在activity中使用fragment的一大特点是具有添加、删除、替换，和执行其它动作的能力，以响应用户的互动。提交给activity的每一系列变化被称为事务，并且可以用FragmentTransaction 中的APIs处理。你也可以将每一个事务保存在由activity管理的后台栈中，并且允许用户导航回退fragment变更（类似于activity的导航回退）。  

你可以从FragmentManager中获取FragmentTransaction实例，像这样：

```java
FragmentManager fragmentManager = getFragmentManager();
 FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
```

每项事务是在同一时间内要执行的一系列的变更。你可以为一个给定的事务用相关方法设置想要执行的所有变化，例如add()，remove()，和replace()。然后，用commit()将事务提交给activity。然而，在调用commit()之前，为了将事务添加到fragment事务后台栈中，你可能会想调用addToBackStatck()。这个后台栈由activity管理，并且允许用户通过按BACK键回退到前一个fragment状态。   
举个例子，下面的代码是如何使用另一个fragment代替一个fragment，并且将之前的状态保留在后台栈中：

```java
// Create new fragment and transaction
 Fragment newFragment = new ExampleFragment();
 FragmentTransaction transaction = getFragmentManager().beginTransaction();

 // Replace whatever is in the fragment_container view with this fragment,
 // and add the transaction to the back stack
 transaction.replace(R.id.fragment_container, newFragment);
 transaction.addToBackStack(null);

 // Commit the transaction
 transaction.commit();
```

在这个例子中，newFragment替换了当前在布局容器中用R.id.fragment_container标识的所有的fragment（如果有的话），替代的事务被保存在后台栈中，因此用户可以回退该事务，可通过按BACK键还原之前的fragment。如果添加多个变更事务（例如另一个add()或者remove()）并调用addToBackStack()，那么在调用commit()之前的所有应用的变更被作为一个单独的事务添加到后台栈中，并且BACK键可以将它们一起回退。

将变更添加到FragmentTransaction中的顺序注意以下两点：     
- 必须要在最后调用commit()   
- 如果你正将多个fragment添加到同一个容器中，那么添加顺序决定了它们在视图层次（view hierarchy）里显示的顺序。 

在执行删除fragment事务时，如果没有调用addToBackStack()，那么事务一提交fragment就会被销毁，而且用户也无法回退它。然而，当移除一个fragment时，如果调用了addToBackStack()，那么之后fragment会被停止，如果用户回退，它将被恢复过来。     
> 提示：对于每一个fragment事务，在提交之前通过调用setTransition()来应用一系列事务动作。     

调用commit()并不立刻执行事务，相反，而是采取预约方式，一旦activity的界面线程（主线程）准备好便可运行起来。然而，如果有必要的话，你可以从界面线程调用executePendingTransations()立即执行由commit()提交的事务。但这样做，通常是没有必要的，除非其它线程的工作依赖与该项事务。

> 警告：只能在activity保存状态（当用户离开activity时）之前用commit()提交事务。如果你尝试在那时之后提交，会抛出一个异常。这是因为如果activity需要被恢复，提交后的状态会被丢失。对于这类丢失提交的情况，可使用commitAllowingStateLoss() 


### 九 与Activity交互      
##### 1.Fragment可以得到宿主Activity的引用
尽管Fragment被实现为一个对象，它独立于Activity并可以在多个Activity中使用，一个给定的fragment实例直接被捆绑在包含它的Activity中。特别是，fragment可以通过getActivity()函数访问Activity，并且很容易的执行类似于查找activity布局中的视图的任务：

```java
View listView = getActivity().findViewById(R.id.list);
```

##### 2.Activity获取Fragment的引用
同样的，activity能够调用fragment的函数findFragmentById()或者findFragmentByTag()，从FragmentManager中获取Fragment的索引，例如：

```java
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
```

##### 3.创建Activity时间回调函数

在一些情况下，你可能需要fragment与activity共享事件。这样做的一个好方法是在fragment内部定义一个回调接口，并需要宿主activity实现它。当activity通过接口接收到回调时，可以在必要时与布局中的其它fragment共享信息。    
举个例子，如果新闻应用的actvity中有两个fragment——一个显示文章列表（fragment A），另一个显示一篇文章（fragment B）——然后fragment A 必须要告诉activity列表项何时被选种，这样，activity可以通知fragment B显示这篇文章。这种情况下，在fragment A内部声明接口OnArticleSelectedListener：

```java
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}
```

然后fragment的宿主activity实现了OnArticleSelectedListener接口，并且重写onArticleSelected()以通知fragment B来自于fragment A的事件。为了确保宿主activity实现了这个接口，fragment A的onAttach()回调函数（当添加fragment到activity中时系统会调用它）通过作为参数传入onAttach()的activity的类型转换来实例化一个OnArticleSelectedListener实例。

```java
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnArticleSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...
}
```

如果activity没有实现这个接口，那么fragment会抛出一个ClassCaseException异常。一旦成功，mListener成员会保留一个activity的OnArticleSelectedListener实现的引用，由此fragment A可以通过调用由OnArticleSelectedListener接口定义的方法与activity共享事件。例如，如果fragment A是ListFragment的子类，每次用户点击列表项时，系统都会调用fragment的onListItemClick()事件，然后fragment调用onArticleSelected()来与activity共享事件。

```java
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Append the clicked item's row ID with the content provider Uri
        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
        // Send the event and Uri to the host activity
        mListener.onArticleSelected(noteUri);
    }
    ...
}
```

传递给onListItemClick()的参数id是点击的列表项行id，activity（或者其它fragment）用以从应用的ContentProvider获取文章。

#####　添加items到Action Bar

你的fragments可以通过实现onCreateOptionsMenu()来构建菜单项到activity的Options Menu（因此Action Bar也一样）。为了使用这个方法接收到调用，不管怎样，你必须在onCreate()期间调用setHasOptionsMenu()，来指明想要添加项目到Options Menu的那个fragment（否则，fragment将接收不到onCreateOptionsMenu()的调用）。     
任何想要在fragment中的Options Menu添加的项目都追加到已有的菜单项后面。当菜单项被选中时，fragment也会接收到对onOptionsItemSelected()的回调。    
你也可以通过调用registerForContextMenu()在fragment布局中注册一个view以提供一个context menu。当用户打开context menu时，fragment接收到对onCreateContextMenu()的回调。当用户选中一个项目时，fragment接收到对onContextItemSelected()的回调。

> 注意：尽管你的fragment会接收到为添加到每个菜单项被选择菜单项的回调，但当用户选择一个菜单项时，activity会首先接收到对应的回调。如果activity的选择菜单项回调的实现没有处理被选中的项目，那么该事件被传递给fragment的回调。这同样适用于Options Menu和context menu。

### 十 总结



