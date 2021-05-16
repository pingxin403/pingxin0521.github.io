---
title: Android Activity
date: 2019-04-26 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### Activity

Android应用程序由许多组件组成，其中Activity是程序中使用频率最高、最基本的组件，Activity中的内容在屏幕上的显示称作用户界面（User Interface，即UI）。

<!--more-->

UI是实现在屏幕上进行显示数据、选择和输入数据等操作的用户交互窗口。
UI的布局（Layout）为Activity构造用户界面的结构，定义各窗体控件的排列位置。

Android的UI元素:
- View类：是所有可视化窗体控件的基类。
- ViewGroup类：是控件的容器。其下层子控件既可以是View，也可以是ViewGroup。
- Widget是窗体控件的包，包含各种UI元素，大部分是可见的控件，如文本框、按钮、列表框、图片、进度条等。


每个布局都有一组布局参数，用于描述其内控件的分布属性 。Android界面布局设计有两种方法：xml声明法和程序代码设计法。

xml声明法：
- 应用程序的可视控件及其布局信息，由xml文件定义声明，此文件称为布局文件。
- 每个Activity对应一个布局文件。
- 所有布局文件都存放在工程文件夹下的“res\layout”子文件夹内。

通常使用xml声明法定义布局，使用java代码来控制Activity组件状态、执行UI交互操作。

Activity类的Java代码文件存放在应用项目的“src”目录的包内。

activity类处于android.app包中，继承体系如下： `java.lang.Object -> android.content.Context  -> android.app.ApplicationContext ->android.app.Activity`；

Activity类通常需要重载的方法：
- onCreate()：初始化Activity。
- onPause()：离开一个Activity时，提交用户所做的修改。

一个activity可以启动另外一个，甚至包括与它不在同一应用程序之中的activity。

**每一个Activity必须在Androidmanifest.xml文件中声明。**



 Android中提供的Activity类，与其子类的类图如下图所示。

![1.jpeg](https://i.loli.net/2019/05/11/5cd6cdd6eb693.jpeg)

从上图可以看到，Activity类间接或直接地继承了 Context、ContextWrapper、 ContextThemeWrapper等基类，因此Activity及其子类都可以直接调用它们的方法。其子类的作用分别为：

- AccountAuthenticatorActivity：帐号身份验证Activity，即一个用于实现账户身份验证的Activity。

- TabActivity：实现Tab界面的Activity，在上一期已经学习了解过。

- AliasActivity：存根Activity，根据组件自己Manifest文件中的meta-data的信息启动另一个指定的Activity，并且Finish自身。

- ExpandableListActivity：实现可展开列表界面的Activity。

- ListActivity：含有一个ListView组件的Activity，在之前也已经学过。

- LauncherActivity：实现一个列表界面的Activity，当单机列表项时，所对应的Activity被启动。

- PreferenceActivity：实现一个程序参数设置、存储功能的Activity。

- FragmentActivity：用来解决Android 3.0 之前没有Fragment的接口，方便在Activity中就能嵌入Fragment来实现你想要的布局效果。

一般创建Activity的步骤总结如下：

1. 定义一个类继承自 android.app.Activity或者其子类。

2. 在res/layout目录中创建一个xml文件，用于创建 Activity的布局。

3. 在 AndroidManifest.xml 文件中注册所创建的 Activity。

4. 重写 Activity的 onCreate()方法，并在该方法中使用 setContentView()加载指定的布局文件。


需要注意的是setContentView()方法既可以接收View对象为参数，也可以接收布局文件对应的资源id为参数。

**快速创建:**

 Android Studio开发工具非常便捷，其实上面创建MyActivity的几个步骤已经被封装好了，只需要几个简单操作即可完成。右击MainActivity所在的包名，依次选择New→Activity→Empty Activity，或者选择其他类型的Activity。

在弹出的New Android Activity对话框中输入相应配置，如下图所示，点击Finish按钮完成Activity的创建。

#### 任务栈

组件的动态运行，有一个最与众不同的概念 -- 任务.

任务（Task）：完成用户的一个目的的所有Activity 。

任务栈（Task Stack）：任务以一组栈的模式，将这些Activity组件聚集在一起的集合。Android系统用一个任务栈来记录一个任务。


以往基于应用（application）的程序开发中，程序具有明确的边界，一个程序就是一个应用，一个应用为了实现功能可以采用开辟新线程甚至新进程来辅助，但是应用与应用之间不能复用资源和功能。

而Android引入了基于组件开发的软件架构，虽然我们开发android程序，仍然使用一个apk工程一个Application的开发形式，但是对于Aplication的开发就用到了Activity、service等四大组件，其中的每一个组件，都是可以被跨应用复用的，这就是android的神奇之处。

虽然组件可以跨应用被调用，但是一个组件所在的进程必须是在组件所在的Aplication进程中。由于android强化了组件概念，弱化了Aplication的概念，所以在android程序开发中，A应用的A组件想要使用拍照或录像的功能就可以不用去针对Camera类进行开发，直接调用系统自带的摄像头应用（称其B应用）中的组件（称其B组件）就可以了，

但是这就引发了一个新问题，A组件运行在A应用中，B组件运行在B应用中，自然都不在同一个进程中，那么从B组件中返回的时候，如何实现正确返回到A组件呢？Task就是来负责实现这个功能的，它是从用户角度来理解应用而建立的一个抽象概念。因为用户所能看到的组件就是Activity，所以Task可以理解为实现一个功能而负责管理所有用到的Activity实例的栈。

栈是一个先进后出的线性表，根据Activity在当前栈结构中的位置，来决定该Activity的状态。正常情况下，当一个Activity启动了另一个Activity的时候，新启动的Activity就会置于任务栈的顶端，并处于活动状态，而启动它的Activity虽然成功身退，但依然保留在任务栈中，处于停止状态，当用户按下返回键或者调用finish()方法时，系统会移除顶部Activity，让后面的Activity恢复活动状态。当然，世界不可能一直这么“和谐”，可以给Activity设置一些“特权”，来打破这种“和谐”的模式，这种特权，就是通过在AndroidManifest文件中的属性andorid:launchMode来设置或者通过Intent的flag来设置的，下面就先介绍下Activity的几种启动模式。

**启动模式**

Android开发者可以在AndroidManifest文件中一共设计了四种启动模式：

1. standard

    默认的启动模式，如果不指定Activity的启动模式，则使用这种模式来启动Activity，每次点击standard模式创建Activity之后，都会创建新的MainActivity覆盖在原有的Activity上，如下图。

    ![1.jpeg](https://i.loli.net/2019/05/07/5cd0fd48c9f34.jpeg)
    如果以这种方式启动的Activity被跨进程调用，在5.0之前新启动的Activity实例会放入发送Intent的Task的栈的顶部，尽管它们属于不同的程序，这似乎有点费解看起来也不是那么合理，所以在5.0之后，上述情景会创建一个新的Task，新启动的Activity就会放入刚创建的Task中，这样就合理的多了。

2. singleTop

    栈顶复用模式，如果要开启的activity在任务栈的顶部已经存在，就不会创建新的实例，而是调用 onNewIntent() 方法。避免栈顶的activity被重复的创建。应用场景：在通知栏点击收到的通知，然后需要启动一个Activity，这个Activity就可以用singleTop，否则每次点击都会新建一个Activity。当然实际的开发过程中，测试妹纸没准给你提过这样的bug：某个场景下连续快速点击，启动了两个Activity。如果这个时候待启动的Activity使用 singleTop模式也是可以避免这个Bug的。
      ![2.jpeg](https://i.loli.net/2019/05/07/5cd0fd48dab70.jpeg)

     同standard模式，如果是外部程序启动singleTop的Activity，在Android 5.0之前新创建的Activity会位于调用者的Task中，5.0及以后会放入新的Task中。

3. singleTask

    栈内复用模式， activity只会在任务栈里面存在一个实例。如果要激活的activity，在任务栈里面已经存在，就不会创建新的activity，而是复用这个已经存在的activity，调用 onNewIntent() 方法，并且清空这个activity任务栈上面所有的activity。

    应用场景：大多数App的主页。对于大部分应用，当我们在主界面点击回退按钮的时候都是退出应用，那么当我们第一次进入主界面之后，主界面位于栈底，以后不管我们打开了多少个Activity，只要我们再次回到主界面，都应该使用将主界面Activity上所有的Activity移除的方式来让主界面Activity处于栈顶，而不是往栈顶新加一个主界面Activity的实例，通过这种方式能够保证退出应用时所有的Activity都能报销毁。

    在跨应用Intent传递时，如果系统中不存在singleTask Activity的实例，那么将创建一个新的Task，然后创建SingleTask Activity的实例，将其放入新的Task中。
    - 假如目前有个任务栈T1中的情况是ABC，这个时候ActivityD以singleTask模式请求启动，其所需要的任务栈正是T1，则系统会直接创建D的实例并将其入栈到T1中。
      ![3.jpeg](https://i.loli.net/2019/05/07/5cd0fd48e8886.jpeg)

    - 假如D Activity启动所需要的任务栈为T2,由于T2和D的实例均不存在，那么系统会先创建任务栈T2，然后再创建D的实例并将其入栈到T2中。我们可以通过设置Activity的taskAffinity属性来模拟这一场景。

    ```xml
    <activity android:name=".SingleTastActivtiy"
              android:label="singleTask launchmode"
              android:launchMode="signleTask"
              android:taskAffinity="">
    ```

    ![4.jpeg](https://i.loli.net/2019/05/07/5cd0fd48e69f3.jpeg)

    - 如果D所需的任务栈为T3，并且当前任务栈T3的情况为ADBC，根据栈内复用的原则，此时D不会重新创建，系统会把D切换到栈顶并调用其onNewIntent()方法，同时由于singleTask默认具有ClearTop的效果，会导致栈内所有在D上面的Activity全部出栈，于是最终T3的情况为AD。
      ![5.jpeg](https://i.loli.net/2019/05/07/5cd0fd48dce86.jpeg)

    - 假如目前有两个任务栈，前台任务栈T4的情况为AB,后台任务栈t4里存有CD,假设CD的启动模式均为singleTask，现在由B去启动D,那么整个后台任务都会被切换到前台，这个时候整个栈就变成了ABCD。
      ![6.jpeg](https://i.loli.net/2019/05/07/5cd0fd48f0a2d.jpeg)

    - 假如上面的其他条件不变，B启动的是C而不是D,那么整个栈的情况就变成了ABC,因为D在C上面，会被清理出栈。
      ![7.jpeg](https://i.loli.net/2019/05/07/5cd0fd48df04a.jpeg)


4. singleInstance

    单一实例模式，整个手机操作系统里面只有一个实例存在。不同的应用去打开这个activity 共享公用的同一个activity。他会运行在自己单独，独立的任务栈里面，并且任务栈里面只有他一个实例存在。应用场景：呼叫来电界面。这种模式的使用情况比较罕见，在Launcher中可能使用。或者你确定你需要使Activity只有一个实例。建议谨慎使用。



关于singleTop和singleInstance这两种启动模式还有一点需要特殊说明：

如果在一个singleTop或者singleInstance的Activity中通过startActivityForResult()方法来启动另一个ActivityB, 那么系统将直接返回Activity_RESULT_CANCELED而不会再去等待返回。

这是由于系统在framework层做了对这两种启动模式的限制，因为Android开发者认为，不同的Task中，默认是不能传递数据的。如果一定要传递数据的话，那么只能通过Intent去绑定数据。



**设置Intent的Flag**

系统提供了两种方式来设置一个Activity的启动模式，除了在AndroidManifest文件中设置以外，还可以通过Intent的Flag来设置一个Activity的启动模式，下面我们在简单介绍下一些Flag。

1. FLAG_ACTIVITY_NEW_TASK

    使用一个新的Task来启动一个Activity，但启动的每个Activity都讲在一个新的Task中。该Flag通常使用在从Service中启动Activity的场景，由于Service中并不存在Activity栈，所以使用该Flag来创建一个新的Activity栈，并创建新的Activity实例。

2. FLAG_ACTIVITY_SINGLE_TOP

   使用singletop模式启动一个Activity，与指定android：launchMode=“singleTop”效果相同。

3. FLAG_ACTIVITY_CLEAR_TOP

    使用SingleTask模式来启动一个Activity，与指定android：launchMode=“singleTask”效果相同。

4. FLAG_ACTIVITY_NO_HISTORY

     Activity使用这种模式启动Activity，当该Activity启动其他Activity后，该Activity就消失了，不会保留在Activity栈中。

**清空任务栈**

系统同样提供了清空任务栈的方法来让我们讲一个Task清空，通常情况下，我们可以在activity的标签上使用以下几种属性来清空任务栈。

- clearTaskOnLaunch：每次返回Activity的时候，都将该Activity上的所有Activity都清除，通过这个属性，可以让这个Task每次初始化的时候，都只有一个Activity。
- finishTaskOnLaunch：这个属性和clearTaskOnLaunch有点类似，只不过clearTaskOnLaunch作用在别人身上，而finishTaskOnLaunch作用在自己身上，通过这个属性，当离开这个Activity所处的Task，那么用户再返回的时候，该Activity会被finish掉。
- alwaysRetainTaskState：Task的一道免死金牌，如果将Activity这个属性设置为true，那么该Activity所在的Task将不接受任何清除命令，一直保持当前Task的状态

我们使用Activity任务栈的各种启动模式和清理方法，是为了更好地使用App中Activity，合理地设置Activity的启动模式会让程序运行更有效率，用户体验更好。

但任务栈虽好，却也不能滥用，如果过度地使用Activity任务栈，则会导致整个App的栈管理混乱，不利于以后程序的拓展，而且在容易出现由于任务栈导致的显示异常，这样的bug是很难调的。所以，在App中使用Activity任务栈一定要根据实际项目的需要，而不是为了使用任务栈而使用任务栈。


**LaunchMode与StartActivityForResult**

我们在开发过程中经常会用到StartActivityForResult方法启动一个Activity，然后在onActivityResult()方法中可以接收到上个页面的回传值，但你有可能遇到过拿不到返回值的情况，那有可能是因为Activity的LaunchMode设置为了singleTask。5.0之后，android的LaunchMode与StartActivityForResult的关系发生了一些改变。两个Activity，A和B，现在由A页面跳转到B页面，看一下LaunchMode与StartActivityForResult之间的关系：

![8.jpeg](https://i.loli.net/2019/05/07/5cd0fd490f4bd.jpeg)

![9.jpeg](https://i.loli.net/2019/05/07/5cd0fd4917274.jpeg)

这是因为ActivityStackSupervisor类中的startActivityUncheckedLocked方法在5.0中进行了修改。在5.0之前，当启动一个Activity时，系统将首先检查Activity的launchMode，如果为A页面设置为SingleInstance或者B页面设置为singleTask或者singleInstance,则会在LaunchFlags中加入FLAG_ACTIVITY_NEW_TASK标志，而如果含有FLAG_ACTIVITY_NEW_TASK标志的话，onActivityResult将会立即接收到一个cancle的信息，而5.0之后这个方法做了修改，修改之后即便启动的页面设置launchMode为singleTask或singleInstance，onActivityResult依旧可以正常工作，也就是说无论设置哪种启动方式，StartActivityForResult和onActivityResult()这一组合都是有效的。所以如果你目前正好基于5.0做相关开发，不要忘了向下兼容，这里有个坑请注意避让。


实际开发过程中如果采用比较合理的Activity启动模式来做好任务栈的管理，可以事半功倍。在launchMode的选择上首先要搞清楚当前的Activity的作用，以及实际使用场景来做出合理选择。

#### 进程和线程

进程是低级核心处理过程，用于运行应用程序代码。

在Android操作系统中，进程完全是应用程序的具体实现。组件运行的进程由Androidmanifest文件控制。
- 组件标签< activity>,< service>, < receiver>, 和< provider>包含一个process属性，这个属性可以设置组件运行的进程。
- < application>标签也包含process属性，用来设置程序中所有组件的默认进程。
- 所有的组件在默认进程的主线程中实例化，系统对这些组件的调用从主线程中分离。

**进程分类**

![](https://i.loli.net/2019/05/07/5cd0fe8db3198.png)

1. 前台进程：正在前台运行的进程,前台进程是必须的用户操作。

     前台进程包括：

   - 正运行着的，与用户交互的Activity。
   - 正运行着的Activity所使用的一个Service。
   - 服务。正在执行回调方法（如onStart()、onCreate()或 onDestroy())的Service对象。
   - 正在执行onReceive()方法的BroadcastReceiver对象。

   一般情况下，前台进程不会被“杀死”。



2. 可见进程：在屏幕中显示，但用户没有直接与之进行交互,可见进程为用户在屏幕上可见但不能与用户进行交互的进程。

    可见进程包括：

   - 一个不在前台但为用户可见的Activity（如在调用了方法onPause() 之后）。
   - 一个可视的Activity所绑定的Service。

   可见进程很重要，不到极端情况（如无法维持前台进程运行时），不会“销毁”可见进程。



3. 服务进程：不可见，在后台为用户服务; 一般不会被中断。

   服务进程包括：

   - 一个由startService()方法启动的Service。

   - 支持正在处理的不需要可见界面运行的Service。

   因为服务不是直接和用户打交道，它的优先级稍低于可见的活动。系统会尽量维持它们的运行，除非系统内存不足以维持前台进程和可见进程的运行需要。

4. 后台进程：对用户作用不大，可能会被系统中止。

     后台进程包括：

   - 目前不可见的Activity（即已调用了onStop()方法）。

   - 目前没有服务的Service。
     在一般情况下会有大量的后台进程，Android将会使用last-seen-first-killed模式来“杀死”进程来为前台进程获得资源。

5. 空进程：对用户没有任何作用，是首先被中止的进程。

    为了改善系统的整体性能， Android通常在内存中保留生命周期结束了的应用。
    Android使用这种缓存机制能够减少应用程序在再次启动时所需的启动时间。这些过程通常根据需要被杀死。

**线程**

每个进程有一到多个线程运行在其中。进程中的所有组件都在UI线程中实例化，以保证应用程序是单线程的,线程通过java的标准对象Thread 创建。

永远要记住：
- 不要阻塞UI线程！如果在UI线程中执行阻塞或者耗时操作会导致UI线程无法响应用户请求。
- 不能在非UI线程（也称为工作线程）中更新UI！这是因为android的UI控件都是线程不安全的。


#### Activity 生命周期

**五大状态:**

- Start：Activity被压入栈顶。
- Running状态：一个新的Activity启动入栈后，它在屏幕最前端，处于栈的最顶端，此时它处于可见并可和用户交互的激活状态。
- Paused状态：当Activity被另一个透明或者Dialog样式的Activity覆盖时的状态。此时它依然与窗口管理器保持连接，系统继续维护其内部状态，它仍然可见，但它已经失去了焦点，故不可与用户交互。
- Stopped状态：当Activity不可见时，Activity处于Stopped状态。当Activity处于此状态时，一定要保存当前数据和当前的UI状态，否则一旦Activity退出或关闭时，当前的数据和UI状态就丢失了。
- Killed状态：Activity被杀掉以后或者被启动以前，处于Killed状态。这是Activity已从Activity堆栈中移除，需要重新启动才可以显示和使用。

其中，Running状态和Paused状态是可见的，Stopped状态和Killed状态时不可见的。

![](https://i.loli.net/2019/05/07/5cd0f281ba3db.png)

**生命周期函数:**

![](https://i.loli.net/2019/05/07/5cd0f47118936.png)

1. onCreate()  

   在Activity生命周期开始时被调用，执行一次性的初始化工作。
   提供Bundle参数：如果Activity之前是被冻结状态，其状态由Bundle提供；接受参数为null或由onSaveInstanceState()方法保存的状态信息。
   其后调用onStart()或onRestart()方法。

2. onRestart()
    当activity从停止状态重新启动时调用

3. onStart()
    当activity对用户即将可见的时候调用。

4. onResume()
    当activity将要与用户交互时调用此方法，此时activity在activity栈的栈顶，用户输入已经 可以传递给它

5. onPause()
    当系统要启动一个其他的activity时调用（其他的activity显示之前），这个方法被用来提交那些持久数据的改变、停止动画、和其他占用CPU资源的东西。由于下一个activity在这个方法返回之前不会resumed，所以实现这个方法时代码执行要尽可能快。

6. onStop()
    当另外一个activity恢复并遮盖住此activity,导致其对用户不再可见时调用。一个新activity启动、其它activity被切换至前景、当前activity被销毁时都会发生这种场景。

7. onDestroy()
    在activity被销毁前所调用的最后一个方法，当进程终止时会出现这种情况；如果内存不足，系统会终止进程，可能不需要调用该方法。
8. onSaveInstanceState(Bundle)：
    调用该方法让活动可以保存每个实例的状态。
9. onRestoreInstanceState(Bundle)：
    使用onSaveInstanceState()方法保存的状态来重新初始化某个活动时调用该方法。

**理解**

![1.png](https://i.loli.net/2019/05/07/5cd0f857f11be.png)
1. Activity启动和销毁过程

   在系统调用onCreate方法之后，就会马上调用onStart，然后继续调用onResume来进图运行状态，最后都会停在onResume状态，完成启动，系统会调用onDestroy来结束一个Activity的生命周期让他毁掉kill状态。
   以上就是一个Activity的启动和销毁的过程。

   - onCreate中创建基本的UI元素。

   - onPause和onStop：清除Acvtivity的资源，避免浪费。

   - onDestroy:因为引用会在Activity销毁的时候销毁，而线程不会，所以清除开启的线程。

2. Activity的暂停和恢复过程

   当栈顶的Activity部分不可见的时候，就会倒置Activity进入onPause。

   - onPause：释放系统资源。

   - onResume：需要重新初始化onPause释放的资源。

3. Activity的停止过程

   栈顶的Activity部分不可见的时候，实际上后续会有两种可能，从部分不可见到可见，也就是恢复过程，从部分不可见到完全不可见，也就是停止过程，系统在当前Activity不可见的时候调用onPause。

4. Activity的重新创建过程

   最后我们来看看Activity是如何重新创建的，如果你的系统长时间处于stop的状态，而此时系统需要更多的内存或者系统内存比较紧张的时候，系统就会回收你的Activity，而系统为了补偿你，会将你的Activity状态通过onRestoreInstanceState()方法保存到Bundle中去，当然你也可以额外增加键值对去保存这些状态，当你重新需要创建这个Activity的时候，保存的Bundle对象就会传递到Activity的onRestoreInstanceState（）方法中去与onCreate方法中去，这也是onCreate的重要参数——saveInstanceState的来源。

   不过这里要注意的一点就是savedInstanceState方法并不是每次当Activity离开前台就会调用，如果用户使用finish方法结束，则不会调用，而且Android系统已经默认实现了控件的缓存状态，一次来减少开发者需要实现的缓存逻辑。


**举例：**

有两个界面Activity A和Activity B。先启动第一个界面Activity A，方法回调的次序是：
```
onCreate(A)-->onStart(A)-->onResume(A)
```
Activity A不关闭，跳转第二个Activity B，方法回调的次序是：
```
onFreeze(A)-->onPause(A)-->onCreate(B)-->onStart(B)-->onResume(B)--onStop(A)
```
在点击back回到第一个界面，这时方法回调的次序是
```
onPause(B)-->onActivityforResult(A)-->onRestart(A)-->onStart(A)-->onResume(A)-->onStop(B)-->onDestroy(B)
```
在点击Exit退出应用时，方法回调的次序是：
```
onPause(A)-->onStop(A)-->onDestroy(A)
```

#### 常用方法

 Activity启动其他Activity有如下两个方法。

- startActivity(Intent intent)：启动其他 Activity。

- startActivityForResult(Intent intent, int requestCode)：以指定的请求码（requestCode）启动Activity，而且程序将会获取新启动的Activity返回的结果（通过重写 onActivityResult(..)方法来获取）。其中requestCode参数代表了启动Activity的请求码，该请求码的值由开发者根据业务自行设置，用于标识请求来源。


上面两个方法都用到了 Intent参数，Intent是Android应用里各组件之间通信的重要方式，一个Activity通过Intent来表达自己“意图”——想要启动哪个组件，被启动的组件既可是 Activity组件，也可是Service组件。

```
// 方式一

// 创建Intent对象

Intentintent1=newIntent();

// 设置需要启动的Activity，以及要启动Activity的上下文环境

intent1.setClass(this,MyActivity.class);

// 方式二

// 直接创建Intent对象，包含要启动的Activity信息

Intentintent2=newIntent(this,MyActivity.class);
```

Android为关闭Activity准备了如下两个方法。

- finish()：结束当前 Activity。

- finishActivity(int requestCode)：结束以 startActivityForResult(Intent intent, int requestCode)方法启动的 Activity。



**主要的Activity属性有：**

1. launchMode
2. taskAffinity
3. allowTaskReparenting
4. clearTaskOnLaunch
5. alwaysRetainTaskState
6. finishOnTaskLaunch

个人理解

1. Activity launchMode设置不为standard,在跳转Activity时,如果新的Activity不再创建新的实例(不执行onCreate) 会调用onNewIntent方法。 （正常生命周期情况不会调用onNewIntent()）

2. Activity launchMode singleInstance实用场景—>关于浏览器的LaunchMode为singleTask，所以如果当你点击一个连接下载文件时(由一个activity来处理下载，launchmode为standard)，如果再次进入浏览器，那么下载页面就被Destory了，那么这里我们可以把下载页面LaunchMode设置为singleInstance可以解决这个问题.(即:在使用singleTask的情况下 不想关闭的界面可单独新起一个task)

3. taskAffinty对lanuchMode的影响 ：

   当LanuchMode设置为 standard 和singTop，即使 taskAffinty不同，也不会新起Task.

   当LanuchMode设置为 singleTask ，以A启动B来说

   1. 当A和B的taskAffinity相同时：第一次创建B的实例时，并不会启动新的task，而是直接将B添加到A所在的task；当B的实例已经存在时，将B所在task中位于B之上的全部Activity都删除，B就成为栈顶元素，实现跳转到B的功能。
   2. 当A和B的taskAffinity不同时：第一次创建B的实例时，会启动新的task，然后将B添加到新建的task中；当B的实例引进存在，将B所在task中位于B之上的全部Activity都删除，B就成为栈顶元素（也是root Activity），实现跳转到B的功能。

   当LanuchMode设置为singleInstance

   当第一次创建该Activity实例时，会新建一个task，并将该Activity添加到该task中。注意：该task只能容纳该Activity实例，不会再添加其他的Activity实例！如果该Activity实例已经存在于某个task，则直接跳转到该task。

4. allowTaskReparenting 这个属性用来标记一个Activity实例在当前应用退居后台后，是否能从启动它的那个task移动到有共同affinity的task，“true”表示可以移动，“false”表示它必须呆在当前应用的task中，默认值为false。

5. clearTaskOnLaunch

     程序回到home界面后，再次点击程序图标的效果。影响的是activity的生命周期。

     简单的：

     activity A（clearTaskOnLaunch设置为true)(为主界面)

     activity B

     程序启动A,在启动B。再点击HOME键回到桌面，再点击程序图标，效果是B执行onrestart，B执行ondestory。A界面显示。（如果clearTaskOnLaunch没设置，则是显示B界面）

     稍微复杂点的：

     activity A（clearTaskOnLaunch设置为true）,B（clearTaskOnLaunch设置为true）,C

     依次启动A,B,C，点击HOME,再在桌面点击图标。启动的是A（执行onrestart）,B、C执行（ondestory）。

     也就是说，优先启动第一个（A）已注册clearTaskOnLaunch为true的Activity，其余的后启动的activity(B、C)都销毁，除非前面A已经finish销毁，后面的已注册clearTaskOnLaunch为true的activity才会生效。

### Activity数据保存和横竖屏切换

#### 数据保存

当 Activity失去焦点时，首先必然会执行onPause()方法，因此当项目中需要保存数据时，可以在onPause()方法中保存。同时，当两个Activity跳转时， MainActivity会先失去焦点让SecondActivity得到焦点，等到SecondActivity完全显示在前台时MainActivity才会切换到后台。

可能有的同学已经发现，在 Activity的生命周期方法中，只有第一个 onCreate()回调方法带有参数 savedInstanceState，其他回调方法都是没有参数的，这个参数有什么作用呢？

经常会出现用户按到 HOME键退出了界面，或者安卓系统意外回收了应用，这种情况下，使用 savedInstanceState就可以让用户再次打开应用时恢复原来的状态。

在具体学习savedInstanceState之前，先来了解一下onSaveInstanceState() 和 onRestoreInstanceState() 方法，这两个方法并不是Activity的生命周期方法，它们不同于 onCreate()、onPause()等生命周期方法，它们并不一定会被回调。当应用遇到意外情况（如：内存不足、用户直接按HOME键）由系统销毁一个Activity时，onSaveInstanceState() 会被调用；但是当用户主动去销毁一个Activity时（如在应用中按返回键），onSaveInstanceState()就不会被调用。因为在这种情况下，用户的行为决定了不需要保存Activity的状态。通常onSaveInstanceState()只适合用于保存一些临时性的状态，而onPause()适合用于数据的持久化保存。

在Activity被杀掉之前调用保存每个实例的状态，以保证该状态可以在onCreate(Bundle)或者onRestoreInstanceState(Bundle) （传入的Bundle参数是由onSaveInstanceState封装好的）中恢复。这个方法在一个Activity被杀死前调用，当该Activity在将来某个时刻回来时可以恢复其先前状态。 例如，如果Activity B启用后位于Activity A的前端，在某个时刻Activity A因为系统回收资源的问题要被杀掉，A通过onSaveInstanceState将有机会保存其用户界面状态，使得将来用户返回到Activity A时能通过onCreate(Bundle)或者onRestoreInstanceState(Bundle)恢复界面的状态。

onSaveInstanceState()方法会在什么时候被执行，有这么几种情况：

- 当用户按下HOME键时。

- 长按HOME键，选择运行其他的程序时。

- 按下电源按键（关闭屏幕显示）时。

- 从Activity A中启动一个新的Activity时。

- 屏幕方向切换时，例如从竖屏切换到横屏时。


总而言之，onSaveInstanceState()的调用遵循一个重要原则，即当系统存在“未经你许可”时销毁了我们的Activity的可能时，则onSaveInstanceState()会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据。

onRestoreInstanceState()方法一般是在onStart()和onResume()之间执行。其被调用的前提是，Activity A确实被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示Activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到Activity A，这种情况下Activity A一般不会因为内存的原因被系统销毁，故Activity A的onRestoreInstanceState方法不会被执行。此也说明上二者，大多数情况下不成对被使用

#### 横竖屏切换

1. 可以横竖屏切换但禁止销毁重建 

   当横竖屏切换时Activity会依次回调onPause()、onStop()、 onDestory()、onCreate()、onStar()、onResume()方法。这种情况对实际开发肯定会有影响，那应该如何解决呢？

   如果不希望在横竖屏切换时Activity被销毁重建，可以在 AndroidManifest.xml文件中设置 Activity的android:conflgChanges的属性，这样无论怎样切换Activity都不会销毁重新创建，具体设置代码如下所示： 

   ```
   android:configChanges="orientation丨keyboardHidden丨screenSize"
   ```

2. 禁止横竖屏切换 

   如果希望某一个界面一直处于竖竖横屏状态，不随手机的晃动而改变，同样可以在清单文件中通过设置Activity的android:screenOrientation属性来完成。该属性主要有以下几个值：

   - ​	unspecified：默认值由系统来判断显示方向。判定的策略是和设备相关的，不同的设备会有不同的显示方向。


   - landscape：横屏显示（宽比高要长）。


   - portrait：竖屏显示(高比宽要长)。


   - user：用户当前首选的方向。


   - behind：和该Activity下面的那个Activity的方向一致(在Activity堆栈中的)。


   - sensor：有物理的感应器来决定。如果用户旋转设备这屏幕会横竖屏切换。


   - nosensor：忽略物理感应器，这样就不会随着用户旋转设备而更改了（"unspecified"设置除外）。


   比如设置为竖屏并不可切换方向，具体设置代码如下所示：

   ```
   android:screenOrieritation="portrait"
   ```

   除了在配置文件中配置，其实还可以在Java文件中设置，只要在onCreate方法中加一句代码即可，如保持竖屏代码为：

   ```
   setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
   ```

   如保持横屏，代码为：

   ```
   setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
   ```

   当然在实际开发中不可能每个Activity都这样多加一句，不仅维护麻烦而且容易出错。一般都是先定义一个BaseActivity，让其继承Activity，然后重写onCreate方法，并进行横竖屏设置，最后让其他的Activity继承BaseActivity即可。 

3. 横竖屏切换同时切换不同布局

   在开发中有时候希望横竖屏时加载不同的布局，可以通过两种方式来实现：

   准备两套不同的布局，Android会自己根据横竖屏加载不同布局。layout-land为横屏，layout-port为竖屏。然后把这两套布局文件放这两文件夹里，文件名一样，Android就会自行判断加载相应布局。

   在Java代码中进行判断，自己想加载什么就加载什么。一般是在onCreate()方法中加载布局文件，可以对横竖屏的状态做下判断，关键代码如下

   ```
   if(this.getResources().getConfiguration().orientation==Configuration.ORIENTATION_LANDSCAPE){ 
    
       setContentView(R.layout.横屏布局);
    
   }elseif(this.getResources().getConfiguration().orientation==Configuration.ORIENTATION_PORTRAIT){ 
    
      setContentView(R.layout.竖屏布局);
    
   }
   ```

   当然在横竖屏切换过程中，可以让Activity销毁重建，需要注意的就是相关数据的保存，可以参考前面数据保存部分的内容，当然还可以通过其他方式来完成，如数据持久化，后期再逐步来学习。