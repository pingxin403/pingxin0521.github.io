---
title: Android  Intent和Application
date: 2019-04-27 15:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### Application 

每个App里面都有一个Application，但是我们为什么还要自定义一个Application类呢？我们知道当App启动的时候，系统会自动加载并初始化Application类。其实Google 并不希望我们去自定义Application类。基本上只有需要做一些全局初始化的时候可能才需要用到自定义Application。

<!--more-->

通常不需要子类化应用程序。在大多数情况下，静态单例可以以一种更模块化的方式提供相同的功能。如果你需要一个全局上下文(例如注册广播接收器),包括Context . getApplicationContext()作为一个上下文参数调用您的单例的getInstance()方法。

市场上的App基本上都会创建自定义App，创建的自定义的Application类和一些单例的工具类差不多。可是使用归使用，有不少项目对自定义Application的用法并不到位，正如官方文档中所表述的一样，多数项目只是把自定义Application当成了一个通用工具类，而这个功能并不需要借助Application来实现，使用单例可能是一种更加标准的方式。不过使用自定义的Application类并没有什么副作用，它和单例模式一样都能实现全局功能。

#### 自定义Application

**用途**

- 得到一个Application对象
- 封装一些全局性的操作
- 初始化全局性的数据

 **创建**

自定义Application 方法很简单，首先创建一个类继承Application类就可以了。

```java
public class MyApplication extends Application {
  @Override  
    public void onCreate() {  
        super.onCreate();  
       
    }  
}
```

然后在AndroidManifest.xml文件中的application节点上，引用就可以了

```xml
 <application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
```

指定完成后，当我们的程序启动时Android系统就会创建一个MyApplication的实例，如果这里不指定的话就会默认创建一个Application的实例。剩下的就是自己创建你需要的方法和实例了。

**单例**

```java
public class MyApplication extends Application {
    public  static  MyApplication  app;

    public  static  MyApplication  getInstance(){
        return  app;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        app = this;

    }
    
}
```

getInstance()方法可以照常提供，但是里面不要做任何逻辑判断，直接返回app对象就可以了，而app对象又是什么呢？在onCreate()方法中我们将app对象赋值成this，this就是当前Application的实例，那么app也就是当前Application的实例了。

[app启动流程](https://www.jianshu.com/p/e931bebe1a18)

### Intent

Android中提供了Intent机制来协助应用间的交互与通讯，或者采用更准确的说法是，Intent不仅可用于应用程序之间，也可用于应用程序内部的activity, service和broadcast receiver之间的交互。Intent这个英语单词的本意是“目的、意向、意图”。

Intent是一种运行时绑定（runtime binding)机制，它能在程序运行的过程中连接两个不同的组件。通过Intent，你的程序可以向Android表达某种请求或者意愿，Android会根据意愿的内容选择适当的组件来响应。

activity、service和broadcast receiver之间是通过Intent进行通信的，而另外一个组件Content Provider本身就是一种通信机制，不需要通过Intent。我们来看下面这个图就知道了：

![1(1).png](https://i.loli.net/2019/05/07/5cd100e2e7cca.png)

如果Activity1需要和Activity2进行联系，二者不需要直接联系，而是通过Intent作为桥梁。通俗来讲，Intnet类似于中介、媒婆的角色。

尽管 Intent 可以通过多种方式促进组件之间的通信，但其基本用例主要包括以下三个：

1. 启动 Activity

Activity 表示应用中的一个屏幕。通过将 Intent 传递给 startActivity()，可以启动新的 Activity 实例。Intent 描述了要启动的 Activity，并携带了任何必要的数据。

如果希望在 Activity 完成后收到结果，则可以调用 startActivityForResult()。在 Activity 的 onActivityResult() 回调中，Activity 将结果作为单独的 Intent 对象接收。

2. 启动服务

Service 是一个不使用用户界面而在后台执行操作的组件。通过将 Intent 传递给 startService()，可以启动服务执行一次性操作（例如，下载文件）。Intent 描述了要启动的服务，并携带了任何必要的数据。

如果服务旨在使用客户端-服务器接口，则通过将 Intent 传递给 bindService()，可以从其他组件绑定到此服务。

3. 发送广播

广播是任何应用均可接收的消息。系统将针对系统事件（例如：系统启动或设备开始充电时）传递各种广播。通过将 Intent 传递给 sendBroadcast()、sendOrderedBroadcast() 或 sendStickyBroadcast()，可以将广播传递给其他应用



对于向这三种组件发送intent有不同的机制：

- 使用Context.startActivity() 或 Activity.startActivityForResult()，传入一个intent来启动一个activity。使用 Activity.setResult()，传入一个intent来从activity中返回结果。
- 将intent对象传给Context.startService()来启动一个service或者传消息给一个运行的service。将intent对象传给 Context.bindService()来绑定一个service。
- 将intent对象传给 Context.sendBroadcast()，Context.sendOrderedBroadcast()，或者Context.sendStickyBroadcast()等广播方法，则它们被传给 broadcast receiver。

接下来通过一个表来列举Intent启动组件的常用的方法，具体如下表所示：

![2.jpeg](https://i.loli.net/2019/05/11/5cd6d1572e93f.jpeg)

### Intent类型

Android中Intent寻找目标组件的方式分为两种，一种是显式Intent，另一种是隐式Intent。

#### 显式Intent 

显式Intent，即在通过Intent启动Activity时，需要明确指定激活组件的名称。在程序中，如果需要在本应用中启动其他的Activity时，可以使用显式意图来启动Activity，其本例代码具体如下

```
// 创建Intent对象
Intentintent=newIntent(this, SecondActivity.class);
// 开启Activity
startActivity(intent);
```

 在上述示例代码中，通过Intent的构造方法来创建Intent对象。构造方法接收两个参数，第一个参数Context要求提供一个启动Activity的上下文，第二个参数Class则是指定要启动的目标Activity，通过构造方法就可以构建出Intent对象。

除了通过指定类名开启组件外，显式Intent还可以根据目标组件的包名、全路径名来指定开启组件，代码如下所示：

```
Intentintent=newIntent();
intent.setClassName("com.android.intent", "com.intent.SecondActivity");
startActivity(intent);
```

在上述实例代码中，通过setClassName（包名，类全路径名）方法指定要开启组件的包名和全路径名来启动另一个组件。

Activity类中提供了一个startActivity ( Intent intent )方法，该方法专门用于开启Activity，它接收一个Intent参数，这里将构建好的Intent传入该方法即可启动目标Activity。

使用这种方式开启的Activity，意图非常明显，因此称之为显式Intent，也叫做显式意图。

#### 隐式Intent 

没有明确指定组件名的Intent称为隐式Intent，又叫隐式意图。Android系统会根据隐式Intent中设置的动作（action )、类别（category )、数据（Uri和数据类型）找到最合适的组件。具体代码如下所示： 

```
<activityandroid:name=".SecondActivity">
   <intent-filter>
       <!-- 设置action属性，需要在代码中根据所设置的name打开指定的组件 -->
       <actionandroid:name="com.jinyu.cqkxzsxy.android.intent.action.xxx"/>
       <categoryandroid:name="android.intent.category.DEFAULT"/>
   </intent-filter>
</activity>
```

在上述代运中，<action\>标明了当前Activity可以响应的动作为“com.jinyu.cqkxzsxy.android.intent.action.xxx”，而<category\>标签则包含了一些类别信息，只有当<action\>和<category\>中的内容同时匹配时，Activity才会被开启。 

使用隐式Intent开启Activity的示例代码如下所示

```
Intentintent=newIntent();
intent.setAction("com.intent.action.xxx");
startActivity(intent);
```

 在上述代码中，Intent置顶了setAction(“com.jinyu.cqkxzsxy.android.intent.action.xxx”)；这个动作并没有指定category，这是因为清单文件中配置的“android.intent.category.DEFAULT”是一种默认的category，在调用startActivity()方法时，会自动将这个category添加到Intent中。

隐式 Intent启动Activity的示意图如下图所示。

![3.jpeg](https://i.loli.net/2019/05/11/5cd6d2687d266.jpeg)

在上图中，Activity A 创建包含操作描述的 Intent，并将其传递给 startActivity()。 Android 系统搜索所有应用中与 Intent 匹配的 Intent 过滤器。 找到匹配项之后，该系统通过调用匹配 Activity（Activity B）的 onCreate() 方法并将其传递给 Intent，以此启动匹配 Activity。

在上述两种Intent中，显式Intent开启组件时必须要指定组件的名称，一般只在本应用程序切换组件时使用。而隐式Intent的功能要比显示Intent更加强大，不仅可以开启本应用的组件，还可以开启其他应用的组件，例如打开系统自带的照相机、浏览器等。

#### 相关属性

Intent由以下各个组成部分：
- component(组件)：目的组件
- action（动作）：用来表现意图的行动
- category（类别）：用来表现动作的类别
- data（数据）：表示与动作要操纵的数据
- type（数据类型）：对于data范例的描写
- extras（扩展信息）：扩展信息
- Flags（标志位）：期望这个意图的运行模式

Intent类型分为显式Intent（直接类型）、隐式Intent（间接类型）。官方建议使用隐式Intent。上述属性中，component属性为直接类型，其他均为间接类型。

相比与显式Intent，隐式Intnet则含蓄了许多，它并不明确指出我们想要启动哪一个活动，而是指定一系列更为抽象的action和category等信息，然后交由系统去分析这个Intent，并帮我们找出合适的活动去启动。

Activity 中 Intent Filter 的匹配过程 ：

![1.png](https://i.loli.net/2019/05/07/5cd10257546f7.png)

1. component(组件)：目的组件

   Component属性明确指定Intent的目标组件的类名称。（属于直接Intent）
    如果 component这个属性有指定的话，将直接使用它指定的组件。指定了这个属性以后，Intent的其它所有属性都是可选的。
   例如，启动第二个Activity时，我们可以这样来写：

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
  //OR
  Intent intent = new Intent();
  //setClass函数的第一个参数是一个Context对象
  //Context是一个类，Activity是Context类的子类，也就是说，所有的Activity对象，都可以向上转型为Context对象
  //setClass函数的第二个参数是一个Class对象，在当前场景下，应该传入需要被启动的Activity类的class对象
  intent.setClass(MainActivity.this, SecondActivity.class);
  startActivity(intent);

  //OR
  Intent intent = new Intent(MainActivity.this,SecondActivity.class);
  startActivity(intent);
  ```

2. Action（动作）：用来表现意图的行动

   当日常生活中，描述一个意愿或愿望的时候，总是有一个动词在其中。比如：我想“做”三个俯卧撑；我要“写” 一封情书，等等。在Intent中，Action就是描述做、写等动作的，当你指明了一个Action，执行者就会依照这个动作的指示，接受相关输入，表现对应行为，产生符合的输出。在Intent类中，定义了一批量的动作，比如ACTION_VIEW，ACTION_PICK等， 基本涵盖了常用动作。加的动作越多，越精确。

   Action 是一个用户定义的字符串，用于描述一个 Android 应用程序组件，一个 Intent Filter 可以包含多个 Action。在 AndroidManifest.xml 的Activity 定义时，可以在其 <intent-filter \>节点指定一个 Action列表用于标识 Activity 所能接受的“动作”。
   SDk中定义了一些标准的动作，包括：

   | **Constant**            | **Target component** | **Action**                                                   |
   | ----------------------- | -------------------- | ------------------------------------------------------------ |
   | ACTION_CALL             | activity             | Initiate a phone call.                                       |
   | ACTION_EDIT             | activity             | Display data for the user to edit.                           |
   | ACTION_MAIN             | activity             | Start up as the initial activity of a task, with no data input and no returned output. |
   | ACTION_SYNC             | activity             | Synchronize data on a server with data on the mobile device. |
   | ACTION_BATTERY_LOW      | broadcast receiver   | A warning that the battery is low.                           |
   | ACTION_HEADSET_PLUG     | broadcast receiver   | A headset has been plugged into the device, or unplugged from it. |
   | ACTION_SCREEN_ON        | broadcast receiver   | The screen has been turned on.                               |
   | ACTION_TIMEZONE_CHANGED | broadcast receiver   | The setting for the time zone has changed.                   |

   当然，也可以自定义动作（自定义的动作在使用时，需要加上包名作为前缀,如"com.example.project.SHOW_COLOR”），并可定义相应的Activity来处理我们的自定义动作。



3. category（类别）：用来表现动作的类别

    Category属性也是作为<intent-filter\>子元素来声明的。例如：

      ```
      <intent-filter>
      　　<action android:name="com.vince.intent.MY_ACTION"></action>
      　　<category android:name="com.vince.intent.MY_CATEGORY"></category>
      　　<category android:name="android.intent.category.DEFAULT"></category>
      </intent-filter>   
      ```

      在自定义动作时，使用activity组件时，必须添加一个默认的类别

      ```
      Intent intent = new Intent(); //设置动作（实际action属性就是一个字符串标记而已）
      intent.setAction("com.example.smyh006intent01.MY_ACTION"); //方法：Intent android.content.Intent.setAction(String action)
      //OR
       Intent intent = new Intent("com.example.MY_ACTION");//方法： android.content.Intent.Intent(String action)  
      ```

      具体的实现为：

      ```
      <intent-filter>
                     <action android:name="com.example.action.MY_ACTION"/>
                     <category android:name="android.intent.category.DEFAULT"/>
      </intent-filter>
      ```

    如果有多个组件被匹配成功，就会以对话框列表的方式让用户进行选择。

    每个Intent中只能指定一个action，但却能指定多个category；类别越多，动作越具体，意图越明确（类似于相亲时，给对方提了很多要求）。
    目前我们的Intent中只有一个默认的category，现在可以通过intent.addCategory()方法来实现。

      ```
      intent.addCategory("com.example.smyh006intent01.MY_CATEGORY");
      ```

    既然在Intent中增加了一个category，那么我们要在清单文件中去声明这个category，不然程序将无法运行。代码如下：

      ```
      <intent-filter>
      　　<action android:name="com.vince.intent.MY_ACTION"></action>
      　　<category android:name="com.vince.intent.MY_CATEGORY"></category>
      　　<category android:name="android.intent.category.DEFAULT"></category>
      </intent-filter>   
      ```

    自定义类别： 在Intent添加类别可以添加多个类别，那就要求被匹配的组件必须同时满足这多个类别，才能匹配成功。操作Activity的时候，如果没有类别，须加上默认类别

4. data（数据）：表示与动作要操纵的数据

     Data属性是Android要访问的数据，和action和Category声明方式相同，也是在<intent-filter>中。多个组件匹配成功显示优先级高的； 相同显示列表。

     Data是用一个uri对象来表示的，uri代表数据的地址，属于一种标识符。通常情况下，我们使用action+data属性的组合来描述一个意图：做什么

     使用隐式Intent，我们不仅可以启动自己程序内的活动，还可以启动其他程序的活动，这使得Android多个应用程序之间的功能共享成为了可能。比如应用程序中需要展示一个网页，没有必要自己去实现一个浏览器（事实上也不太可能），而是只需要条用系统的浏览器来打开这个网页就行了。

     ```
     intent.setData(Uri.parse("http://www.baidu.com"));                
     ```

     此时， 调用的是系统默认的浏览器，也就是说，只调用了这一个组件。现在如果有多个组件得到了匹配，应该是什么情况呢？

     ```
     <activity android:name=".SecondActivity">
         <intent-filter>
              <action android:name="android.intent.action.VIEW" />
              <category android:name="android.intent.category.DEFAULT" />
              <data android:scheme="http" android:host="www.baidu.com"/>                 
         </intent-filter>            
     </activity>
     ```

     现在，SecondActivity也匹配成功了，我们运行程序，点击MainActicity的按钮时，弹出选择界面。
     我们可以总结如下：

   当Intent匹配成功的组件有多个时，显示优先级高的组件，如果优先级相同，显示列表让用户自己选择

   优先级从-1000至1000，并且其中一个必须为负的才有效

   注：系统默认的浏览器并没有做出优先级声明，其优先级默认为正数。

   - 优先级的配置如下：

   - 在清单文件中修改对SecondAcivity的声明，即增加一行代码，通过来android:priority设置优先级，如下：

     ```
     <activity
         android:name=".SecondActivity">
         <intent-filter android:priority="-1">
              <action android:name="android.intent.action.VIEW" />
              <category android:name="android.intent.category.DEFAULT" />
              <data android:scheme="http" android:host="www.baidu.com"/>                                  
         </intent-filter>            
     </activity>
     ```

   注：  Data属性的声明中要指定访问数据的Uri和MIME类型。可以在<data>元素中通过一些属性来设置：

   android:scheme、android:path、android:port、android:mimeType、android:host等，通过这些属性来对应一个典型的Uri格式`scheme://host:port/path`。例如：`http://www.google.com`。

5. type（数据类型）：对于data范例的描写

    如果Intent对象中既包含Uri又包含Type，那么，在<intent-filter\>中也必须二者都包含才能通过测试。

    Type属性用于明确指定Data属性的数据类型或MIME类型，但是通常来说，当Intent不指定Data属性时，Type属性才会起作用，否则Android系统将会根据Data属性值来分析数据的类型，所以无需指定Type属性。

    data和type属性一般只需要一个，通过setData方法会把type属性设置为null，相反设置setType方法会把data设置为null，如果想要两个属性同时设置，要使用Intent.setDataAndType()方法。
    播放指定路径的mp3文件。

      ```
      button.setOnClickListener(new OnClickListener(){
          @Override
          public void onClick(View v) {
              Intent intent = new Intent();
              intent.setAction(Intent.ACTION_VIEW);
              Uri data = Uri.parse("file:///storage/sdcard0/平凡之路.mp3");
              //设置data+type属性
              intent.setDataAndType(data, "audio/mp3"); //方法：Intent android.content.Intent.setDataAndType(Uri data, String type)
              startActivity(intent);                
          }            
      });
      ```

6. extras（扩展信息）：扩展信息

    是其它所有附加信息的集合。使用extras可以为组件提供扩展信息，比如，如果要执行“发送电子邮件”这个动作，可以将电子邮件的标题、正文等保存在extras里，传给电子邮件发送组件。

7. Flags（标志位）：期望这个意图的运行模式

    一个程序启动后系统会为这个程序分配一个task供其使用，另外同一个task里面可以拥有不同应用程序的activity。那么，同一个程序能不能拥有多个task？这就涉及到加载activity的启动模式，这个需要单独讲一下。

    注：android中一组逻辑上在一起的activity被叫做task，自己认为可以理解成一个activity堆栈。

#### Intent数据传递

启动四大组件通过Intent对象来实现，Intent的功能包括启动四大组件以及相关信息+传递数据。

其中传递数据Intent提供了putExtra和对应的getExtra方法来实现：

putExtra和getExtra 其实是和Bundle  put和get方法一一对应的，在Intent类中有一个Bundle的mExtras成员变量

所有的putExtra和getExtra方式实际是调用mExtras对象的put和get方法进行存取。

所以正常情况下 四大组件间传递数据直接通过putExtra和getExtra方法存取即可，无需再创建一个bundle对象。

Intent  putExtra方法：

```
Intent  putExtra(String name, Bundle value)
Intent  putExtra(String  name, Parcelable[] value)
Intent  putExtra(String name, Serializable  value)
Intent  putExtra(String name, Parcelable value)
Intent  putExtra(String  name, int[] value)
Intent  putExtra(String name, float value)
Intent  putExtra(String name, byte[] value)
Intent  putExtra(String name, long[]  value)
Intent  putExtra(String name, float[] value)
Intent  putExtra(String name, long value)
Intent  putExtra(String name, String[]  value)
Intent  putExtra(String name, boolean value)
Intent  putExtra(String name, boolean[] value)
Intent  putExtra(String name, short  value)
Intent  putExtra(String name, double value)
Intent  putExtra(String  name, short[] value)
Intent  putExtra(String name, String value)
Intent  putExtra(String name, byte value)
Intent  putExtra(String name, char[]  value)
Intent  putExtra(String name, CharSequence[] value)
Intent  putExtras(Intent src)
Intent  putExtras(Bundle extras)
Intent  putIntegerArrayListExtra(String name, ArrayList<Integer> value)
Intent  putParcelableArrayListExtra(String name, ArrayList<? extends Parcelable>  value)
Intent  putStringArrayListExtra(String name, ArrayList<String>  value)
```

Intent getExtra方法：

```
double[]  getDoubleArrayExtra(String name)
double  getDoubleExtra(String  name, double defaultValue)
Bundle  getExtras()
int  getFlags()
float[]  getFloatArrayExtra(String name)
float  getFloatExtra(String name, float  defaultValue)
int[]  getIntArrayExtra(String name)
int  getIntExtra(String  name, int defaultValue)
ArrayList<Integer>  getIntegerArrayListExtra(String name)
long[]  getLongArrayExtra(String  name)
long  getLongExtra(String name, long defaultValue)
Parcelable[]   getParcelableArrayExtra(String name)
<T extends Parcelable>  ArrayList<T>  getParcelableArrayListExtra(String name)
<T extends  Parcelable> T  getParcelableExtra(String name)
Serializable   getSerializableExtra(String name)
short[]  getShortArrayExtra(String  name)
short  getShortExtra(String name, short defaultValue)
String[]   getStringArrayExtra(String name)
ArrayList<String>  getStringArrayListExtra(String name)
String  getStringExtra(String  name)
```

#### intent大全

1. 从google搜索内容

```
Intent intent = new Intent();
intent.setAction(Intent.ACTION_WEB_SEARCH);
intent.putExtra(SearchManager.QUERY,"searchString")
startActivity(intent);
```



2. 浏览网页

```
Uri uri = Uri.parse("http://www.google.com");
Intent it = new Intent(Intent.ACTION_VIEW,uri);
startActivity(it);
```



3. 显示地图

```
Uri uri = Uri.parse("geo:38.899533,-77.036476");
Intent it = new Intent(Intent.Action_VIEW,uri);
startActivity(it);
```



4. 路径规划

```
Uri uri = Uri.parse("http://maps.google.com/maps?f=dsaddr=startLat%20startLng&daddr=endLat%20endLng&hl=en");
Intent it = new Intent(Intent.ACTION_VIEW, uri);
startActivity(it);
```



5. 拨打电话

```
Uri uri = Uri.parse("tel:xxxxxx");
Intent it = new Intent(Intent.ACTION_DIAL, uri);
startActivity(it);
```



6. 调用发短信的程序

```
Intent it = new Intent(Intent.ACTION_VIEW);
it.putExtra("sms_body", "The SMS text");
it.setType("vnd.android-dir/mms-sms");
startActivity(it);
```



7. 发送短信

```
Uri uri = Uri.parse("smsto:0800000123");
Intent it = new Intent(Intent.ACTION_SENDTO, uri);
it.putExtra("sms_body", "The SMS text");
startActivity(it);



String body="this is sms demo";
Intent mmsintent = new Intent(Intent.ACTION_SENDTO, Uri.fromParts("smsto", number, null));
mmsintent.putExtra(Messaging.KEY_ACTION_SENDTO_MESSAGE_BODY, body);
mmsintent.putExtra(Messaging.KEY_ACTION_SENDTO_COMPOSE_MODE, true);
mmsintent.putExtra(Messaging.KEY_ACTION_SENDTO_EXIT_ON_SENT, true);
startActivity(mmsintent);
```



8. 发送彩信

```
Uri uri = Uri.parse("content://media/external/images/media/23");
Intent it = new Intent(Intent.ACTION_SEND);
it.putExtra("sms_body", "some text");
it.putExtra(Intent.EXTRA_STREAM, uri);
it.setType("image/png");
startActivity(it);


StringBuilder sb = new StringBuilder();
sb.append("file://");
sb.append(fd.getAbsoluteFile());
Intent intent = new Intent(Intent.ACTION_SENDTO, Uri.fromParts("mmsto", number, null));
// Below extra datas are all optional.
intent.putExtra(Messaging.KEY_ACTION_SENDTO_MESSAGE_SUBJECT, subject);
intent.putExtra(Messaging.KEY_ACTION_SENDTO_MESSAGE_BODY, body);
intent.putExtra(Messaging.KEY_ACTION_SENDTO_CONTENT_URI, sb.toString());
intent.putExtra(Messaging.KEY_ACTION_SENDTO_COMPOSE_MODE, composeMode);
intent.putExtra(Messaging.KEY_ACTION_SENDTO_EXIT_ON_SENT, exitOnSent);
startActivity(intent);
```



9. 发送Email

```
Uri uri = Uri.parse("mailto:xxx@abc.com");
Intent it = new Intent(Intent.ACTION_SENDTO, uri);
startActivity(it);



Intent it = new Intent(Intent.ACTION_SEND);
it.putExtra(Intent.EXTRA_EMAIL, "me@abc.com");
it.putExtra(Intent.EXTRA_TEXT, "The email body text");
it.setType("text/plain");
startActivity(Intent.createChooser(it, "Choose Email Client"));



Intent it=new Intent(Intent.ACTION_SEND);
String[] tos={"me@abc.com"};
String[] ccs={"you@abc.com"};
it.putExtra(Intent.EXTRA_EMAIL, tos);
it.putExtra(Intent.EXTRA_CC, ccs);
it.putExtra(Intent.EXTRA_TEXT, "The email body text");
it.putExtra(Intent.EXTRA_SUBJECT, "The email subject text");
it.setType("message/rfc822");
startActivity(Intent.createChooser(it, "Choose Email Client"));



Intent it = new Intent(Intent.ACTION_SEND);
it.putExtra(Intent.EXTRA_SUBJECT, "The email subject text");
it.putExtra(Intent.EXTRA_STREAM, "file:///sdcard/mysong.mp3");
sendIntent.setType("audio/mp3");
startActivity(Intent.createChooser(it, "Choose Email Client"));


```



10. 播放多媒体

```
Intent it = new Intent(Intent.ACTION_VIEW);
Uri uri = Uri.parse("file:///sdcard/song.mp3");
it.setDataAndType(uri, "audio/mp3");
startActivity(it);


Uri uri = Uri.withAppendedPath(MediaStore.Audio.Media.INTERNAL_CONTENT_URI, "1");
Intent it = new Intent(Intent.ACTION_VIEW, uri);
startActivity(it);
```



11. uninstall apk

```
Uri uri = Uri.fromParts("package", strPackageName, null);
Intent it = new Intent(Intent.ACTION_DELETE, uri);
startActivity(it);
```



12. install apk

```
Uri installUri = Uri.fromParts("package", "xxx", null);
returnIt = new Intent(Intent.ACTION_PACKAGE_ADDED, installUri);
```

13.  打开照相机

```
Intent i = new Intent(Intent.ACTION_CAMERA_BUTTON, null);
this.sendBroadcast(i);



long dateTaken = System.currentTimeMillis();
String name = createName(dateTaken) + ".jpg";
fileName = folder + name;
ContentValues values = new ContentValues();
values.put(Images.Media.TITLE, fileName);
values.put("_data", fileName);
values.put(Images.Media.PICASA_ID, fileName);
values.put(Images.Media.DISPLAY_NAME, fileName);
values.put(Images.Media.DESCRIPTION, fileName);
values.put(Images.ImageColumns.BUCKET_DISPLAY_NAME, fileName);
Uri photoUri = getContentResolver().insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
Intent inttPhoto = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
inttPhoto.putExtra(MediaStore.EXTRA_OUTPUT, photoUri);
startActivityForResult(inttPhoto, 10);
```



14. 从gallery选取图片

```
Intent i = new Intent();
i.setType("image/*");
i.setAction(Intent.ACTION_GET_CONTENT);
startActivityForResult(i, 11);
```



15.  打开录音机

```
Intent mi = new Intent(Media.RECORD_SOUND_ACTION);
startActivity(mi);
```



16. 显示应用详细列表

```
Uri uri = Uri.parse("market://details?id=app_id");
Intent it = new Intent(Intent.ACTION_VIEW, uri);
startActivity(it);

//where app_id is the application ID, find the ID
//by clicking on your application on Market home
//page, and notice the ID from the address bar

//刚才找app id未果，结果发现用package name也可以
Uri uri = Uri.parse("market://details?id=<packagename>");
//这个简单多了
```



17. 寻找应用

```
Uri uri = Uri.parse("market://search?q=pname:pkg_name");
Intent it = new Intent(Intent.ACTION_VIEW, uri);
startActivity(it);
//where pkg_name is the full package path for an application
```



18. 打开联系人列表

```
Intent i = new Intent();
i.setAction(Intent.ACTION_GET_CONTENT);
i.setType("vnd.android.cursor.item/phone");
startActivityForResult(i, REQUEST_TEXT);


Uri uri = Uri.parse("content://contacts/people");
Intent it = new Intent(Intent.ACTION_PICK, uri);
startActivityForResult(it, REQUEST_TEXT);
```



19. 打开另一程序

```
Intent i = new Intent();
ComponentName cn = new ComponentName("com.yellowbook.android2", "com.yellowbook.android2.AndroidSearch");
i.setComponent(cn);
i.setAction("android.intent.action.MAIN");
startActivityForResult(i, RESULT_OK);
```



20. 调用系统编辑添加联系人（高版本SDK有效）：

```
Intent it = newIntent(Intent.ACTION_INSERT_OR_EDIT);
it.setType("vnd.android.cursor.item/contact");
//it.setType(Contacts.CONTENT_ITEM_TYPE);
it.putExtra("name","myName");
it.putExtra(android.provider.Contacts.Intents.Insert.COMPANY, "organization");
it.putExtra(android.provider.Contacts.Intents.Insert.EMAIL,"email");
it.putExtra(android.provider.Contacts.Intents.Insert.PHONE,"homePhone");
it.putExtra(android.provider.Contacts.Intents.Insert.SECONDARY_PHONE, "mobilePhone");
it.putExtra( android.provider.Contacts.Intents.Insert.TERTIARY_PHONE, "workPhone");
it.putExtra(android.provider.Contacts.Intents.Insert.JOB_TITLE,"title");
startActivity(it);
```



21. 调用系统编辑添加联系人（全有效）：

```
Intent intent = newIntent(Intent.ACTION_INSERT_OR_EDIT);
intent.setType(People.CONTENT_ITEM_TYPE);
intent.putExtra(Contacts.Intents.Insert.NAME, "My Name");
intent.putExtra(Contacts.Intents.Insert.PHONE, "+1234567890");
intent.putExtra(Contacts.Intents.Insert.PHONE_TYPE,Contacts.PhonesColumns.TYPE_MOBILE);
intent.putExtra(Contacts.Intents.Insert.EMAIL, "com@com.com");
intent.putExtra(Contacts.Intents.Insert.EMAIL_TYPE,Contacts.ContactMethodsColumns.TYPE_WORK);
startActivity(intent);
```
