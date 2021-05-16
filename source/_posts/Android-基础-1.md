---
title: Android 应用程序构成
date: 2019-04-25 18:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### 项目目录
一个Android应用程序以一个项目目录的形式组织。Android程序由 java代码和xml 属性声明共同设计完成。

<!--more-->

![](https://i.loli.net/2019/05/06/5cd041bc3a6a6.png)

#### Project 视图

我们选择Project视图，就会有以下的项目文档结构：

![1.png](https://i.loli.net/2019/04/29/5cc663d2863c2.png)

**外层结构**

1. .gradle和.idea:目录下是Android Studio 自动创建的一些文件

2. app:项目内代码、资源均存放在这个目录下。

3. gradle:包含gradle wrapper 配置文件，使用gradle wrapper的方式不需要提前下载gradle,而是会根据本地的缓存情况来判断是否需要进行下载。

4. .gitignore:用来记录特定的目录或文件来排除在版本控制之外，具体参考Git

5. bulid.gradle:全局的gradle构建脚本，通常不需要进行修改。

6. gradle.properties:全局gradle配置文件，其中的属性会影响项目中所有gradle编译脚本。

7. gradlew和gradlew.bat:用来在命令行中执行gradle命令，前者在Linux或Mac中使用，后者在windows中使用。

8. local.properties:用于指定本机中的Android SDK路径

9. MyApplication.iml:用于标识Intellij IDEA 项目

10. setting.gradle:用于指定项目中所有引入的模块。

**app目录内结构**

1. build:编译时生成的文件

2. libs:使用第三方jar包时直接添加入该目录下，jar包会自动地被添加进构建路径里。

3. src:项目的主要代码和资源所在。

4. androidTest:用来编写Android Test测试用例，可以对项目进行一些自动化测试。

5. main:存放所有Java代码，可以构建子目录来规范管理代码

6. res:几乎是所有的资源都存放在这里，每个文件夹都有自己的功用。比如drawable内存放图片文件，layout为所有的布局xml文件，

7. AndroidManifest.xml:当前Android项目的配置文件，程序中定义的四大组件都在其中进行注册，以及程序的权限声明。

8. test:用来编写Unit Test测试用例

9. .gitignore:用于版本控制

10. app.iml:用于标识Intellij IDEA 项目

11. build.gradle:内层build.gradle为app模块内的gradle构建脚本，会指定项目构建相关配置。

12. proguard-rules.pro:用于指定项目代码的混淆规则，防止项目完成后生成的安装包文件被人破解。

#### Android视图

![1.png](https://i.loli.net/2019/04/29/5cc6627bca057.png)

1. manifests

  - AndroidManifest.xml：项目清单文件，包含对App的一系列配置，如：应用名、所需权限、包名、所有的Activity信息等

2. java

  存放项目的 Java 源代码
  R.java文件:由IDE自动生成，不能直接修改；用资源id的形式标注drawable、layout、values文件夹中的资源信息。

3. res
  - drawable

    存放不同分辨率的图片资源
  - layout

    存放项目的布局文件,在XML文件中声明Android应用程序界面布局和组件。
    优点：短小易维护。符合MVC原则：UI与程序逻辑相分离。
  - menu

    也用于存放项目的布局文件，不过一般只存放menu的布局文件
  - mipmap

    也用于存放不同分辨率的图片资源，不过在图片缩放的优化和性能上，mipmap比drawble更好
  - raw

    可选，一般用于存放数据库相关的资源
  - transition

    可选，一般用于存放slide文件
  - values
    -  colors.xml

      用于存放项目会用到的颜色数据,颜色值由RGB（三位16进制数）或RRGGBB （六位16进制数）表示，以“# ”符号开头。例如：#00f（蓝色）， #00ff00（绿色）。定义透明色，表示透明度的alpha通道值紧随“#”之后。例如： #600f（透明蓝色）， #7700ff00（透明绿色） 。

    -  dimens.xml

      用于存放项目会用到的长度和大小等数据,维度通常用于创建布局常量，在样式和布局资源中定义边界、高度和尺寸大小时经常用到维度。
      用标识符表示维度单位：
      ```
       px（像素）：屏幕上的像素。
       in （英寸）：长度单位。
       mm（毫米）：长度单位。
       pt （磅）：1/72英寸。
       dp（与密度无关的像素）：一种基于屏幕密度的抽象单位。
       sp （与刻度无关的像素）：与pd类似。
      ```
      建议：使用sp作为文字的单位，使用dp作为其他元素的单位。

    -  strings
      用于存放项目中会使用到的字符串数据，可根据系统语言不同（本地化），适配不同的strings.xml
      ```
      <?xml version="1.0" encoding="utf-8"?>
      <resources>
          <string name="hello">Hello World, Hello!</string>
          <string name="app_name">Hello, Android</string>
      </resources>
      ```

    -  styles
      用于存放项目中会使用到的样式数据，可根据Android不同版本，适配不同的style.xml
      ```
      <?xml version=”1.0” encoding=”utf-8”?>
        <resources>
          <style name=”BaseText”>
             <item name=”android:textSize”>14sp</item>
             <item name=”android:textColor”>#111</item>
         </style>
        <style name=”SmallText” parent=”BaseText”>
            <item name=”android:textSize”>8sp</item>
        </style>
      </resources>
      ```

    ![](https://i.loli.net/2019/04/29/5cc6c109cf162.png)

4. Gradle Scripts

  -  build.gradle (Project: [project name])
    包含当前项目正在使用的gradle版本号信息
    ```
    // Top-level build file where you can add configuration options common to all sub-projects/modules.

    // 用于设置驱动构建过程的代码
    buildscript {
        // 远程仓库
        repositories
            // 这里依赖的远程仓库是jcenter
            google()
           jcenter()
        }
        // 声明了使用Android Studio gradle插件版本。
        dependencies {
            classpath 'com.android.tools.build:gradle:3.2.0'

            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
        }
    }

    allprojects {
        repositories {
            google()
            jcenter()
        }
    }

    task clean(type: Delete) {
        delete rootProject.buildDir
    }
    ```
  -  build.gradle (Module: app)
    包含当前项目的applicationId、最小适配的Android版本、目标适配的Android版本、编译序号、应用版本号、所有依赖的包等信息
    ```
    // 表示使用com.android.application插件。
    apply plugin: 'com.android.application'

    // 配置所有android构建过程需要的参数
    android {
        // 用于编译的SDK版本
        compileSdkVersion 28
        // 用于Gradle编译项目的工具版本
        buildToolsVersion "28.0.2"

        // 项目默认配置
        defaultConfig {
            // 应用程序包名
            applicationId "com.hyp.learn"
            // 最低支持Android版本
            minSdkVersion 19
            // 目标版本
            targetSdkVersion 28
            // 版本号
            versionCode 1
            // 版本名称
            versionName "1.0"
            testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        }
        // 编译类型。默认两个：release和debug
        buildTypes {
            release {
                // 设置是否使用混淆
                minifyEnabled false
                // 混淆文件
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
        }
    }

    // 用于配置引用的依赖
    dependencies {
        implementation fileTree(dir: 'libs', include: ['*.jar'])
        implementation 'com.android.support:appcompat-v7:28.0.0-alpha1'
        testImplementation 'junit:junit:4.12'
        androidTestImplementation 'com.android.support.test:runner:1.0.2'
        androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
    }
    ```
  -  gradle-wrapper.properties
  ```
  distributionBase=GRADLE_USER_HOME
  distributionPath=wrapper/dists
  distributionUrl=https\://services.gradle.org/distributions/gradle-4.6-all.zip
  zipStoreBase=GRADLE_USER_HOME
  zipStorePath=wrapper/dists
  ```
  -  proguard-rules.pro

  -  setting.gradle

  -  local.properties
    包含当前Android Studio编译器所使用的外部SDK路径信息

### 基本组件

应用程序组件是一个Android应用程序的基本构建块。这些组件由应用清单文件松耦合的组织。AndroidManifest.xml描述了应用程序的每个组件，以及他们如何交互。

以下是可以在Android应用程序中使用的四个主要组件。


#### Activity

一个活动标识一个具有用户界面的单一屏幕。举个例子，一个邮件应用程序可以包含一个活动用于显示新邮件列表，另一个活动用来编写邮件，再一个活动来阅读邮件。当应用程序拥有多于一个活动，其中的一个会被标记为当应用程序启动的时候显示。

应用程序可以定义一个或多个Activity。

一个活动是Activity类的一个子类，如下所示：

```
public class MainActivity extends Activity {

}
```
Activity的显示内容由View对象提供。每个视图（视图组）对象都有它们自己的多种xml属性。每个视图（视图组）对象都有一个ID属性。

例：在HelloAndroid的main.xml中的TextView对象：
```
<TextView  
android:id=”@+id/tvStr”
android:layout_width="fill_parent"
android:layout_height="wrap_content"
android:text="@string/hello"
/>
```
ID属性有时被定义为字符串，编译后为整型值。
ID的定义：
```
android:id=“@+id/my_button1”
```
`@`告诉xml解析器，解析并展开id后的其余部分作为ID资源。
`@`后使用“+”表示定义一个新资源。
```
Android:id=“@android:id/my_button1”
```
`@`后不使用“+”表示引用Android的一个资源。
此时要加上Android包名字空间，通过它可以从android.R资源类中引用ID。

**启动Activity方式：**

1. 在onCreate()方法内调用setContentView()方法，用来指定将要启动的res/layout目录下的布局文件，例如setContentView(R.layout.main)。
2. 第二种方法是调用startActivity()，用于启动一个新的Activity。
3. 第三种方法是调用startActivityforResult()，用于启动一个Activity，并在该Activity结束时会返回信息。

**关闭Activity方式：**

1. 通常调用finish()方法来关闭一个Activity。
2. 调用setResult()方法，则可以返回数据给上一级的Activity。
3. 使用startActivityforResult()启动的Activity时，则需要调用finishActivity()方法，来关闭其父Activity。
4. System.exit()退出应用。

#### Services

服务是运行在后台，执行长时间操作的组件。举个例子，服务可以是用户在使用不同的程序时在后台播放音乐，或者在活动中通过网络获取数据但不阻塞用户交互。

没有用户界面显示。具有较长的生命周期。常用于播放背景音乐的应用设计。一般由Activity启动，但不依赖于Activity 。

一个服务是Service类的子类，如下所示：
```
public class MyService extends Service {

}
```

**启动（结束）方式：**

- startService方法：启动，会依次调用onCreate和onStart方法；
- stopService方法：结束，会调用onDestroy方法。
- bindService方法：启动，会依次调用onCreate和onBind方法；
- unbindService方法：结束，会依次调用onUnbind和onDestroy方法。

#### Broadcast Receivers

广播接收器简单地响应从其他应用程序或者系统发来的广播消息。举个例子，应用程序可以发起广播来让其他应用程序知道一些数据已经被下载到设备，并且可以供他们使用。因此广播接收器会拦截这些通信并采取适当的行动。

广播接收器是BroadcastReceiver类的一个子类，每个消息以Intent对象的形式来广播。
```
public class MyReceiver  extends  BroadcastReceiver {
}
```
**使用过程：**

- 将需要广播的消息封装到Intent中。
- 然后通三种发送方法中的一种将Intent广播出去 。
- 通过IntentFilter对象来过滤所发送的实体Intent。
- 实现一个重写了onReceive方法的BroadcastReceiver。

**注册方式：**

- 在AndroidManifest.xml中，放在<receiver> </receiver>中，通过<intent-filter>设置过滤条件。
- 在java代码中，先创建IntentFilter对象，在IntentFilter对象内设置Intent过滤条件。

#### Content Providers

内容提供者组件通过请求从一个应用程序到另一个应用程序提供数据。这些请求由ContentResolver类的方法来处理。这些数据可以是存储在文件系统、数据库或者其他其他地方。

ContentProvider为所有需要共享的数据创建一个数据表;ContentProvider会对外提供一个公开的URI(通用资源标识符:Uniform Resource Identifier)来标识数据集。

URI主要分三个部分：scheme, authority 和 path。其中authority又分为host和port。
格式：`scheme://host:port/path`

内容提供者是ContentProvider类的子类，并实现一套标准的API，以便其他应用程序来执行事务。
```
public class MyContentProvider extends  ContentProvider {

}
```

#### 附件组件

有一些附件的组件用于以上提到的实体、他们之间逻辑、及他们之间连线的构造。这些组件如下：

![](https://i.loli.net/2019/04/29/5cc65d9889555.png)

说明：

1. Intent

  是一种运行时的绑定机制，运行时连接两个不同的组件。  Activity、Service、BroadcastReceiver之间的通信由Intent协助完成。不同类型的组件有不同的 Intent传送方法。  
  Intent组成：组件名称， Action， Data，Category等。

2. Intent过滤器（IntentFilter ）

  当Intent没有指定组件名（隐性）时，使用IntentFilter 来找与Intent最合适的组件。
  工作机制：通过Intent向Android发出请求，然后查询各组件声明的IntentFilter，找到需要的组件并运行它。
  用<Intent-filter\>标签声明指定组件支持的 Intent值。
  IntentFilter可以设置多个过滤值（即元素值）。

3. Notification

  Notification用来在不需要焦点或不中断它们当前Activity的情况下提示用户。它们是Service或Broadcast Receiver获得用户注意的首选方式。
  例如：当设备收到文本信息或外部来电时，它通过闪光，发声，显示图标或显示对话框信息来提醒你。

### AndroidManifest 应用清单

> 先了解，后面会用到，慢慢理解


由gradle构建的Android应用，只需要最关键的3个文件(build.gradle AndroidManifest.xm Activity.java)


- build.gradle 用来构建项目脚本文件，编译打包

- AndroidManifest 清单文件向 Android 系统提供应用的必要信息，系统必须具有这些信息方可运行应用的任何代码

- Activity.java 源代码文件

每个应用的根目录中都必须包含一个 AndroidManifest.xml 文件，主要作用：

- 确定应用的Java软件包名。软件包名称充当应用的唯一标识符
- 声明应用的组件，包括 Activity、服务、广播接收器和内容提供程序。
- 声明应用必须具备哪些权限才能访问 API 中受保护的部分并与其他应用交互。
- 应用程序兼容的最低版本。
- 声明应用程序需要的链接库。
- 其他应用程序访问该应用程序时应该具有的权限。

该文件的结构大致如下：

```
<manifest>
<uses-permission/>
<application>
<activity/>
<service/>
<reciever/>
<provider/>
</application>
</manifest>
```

在安装一个应用程序的APK到设备中时，包管理服务（PackageManagerService）会调用自己的解析器(PackageParser)去解析应用程序的"AndroidManifest.xml"文件，从而形成包信息。当启动一个应用的时候，会哦那个到这些包信息，孵化(zygote)出应用程序的进程

1. manifest 根节点:
根节点manifest必须指定xmlns:android和package属性


2. application 应用程序的根节点:

  application 节点是AndroidManifest文件必须要定义的节点，声明了应用所用到的组件，即我们常说的Android四大组件。

  + activity 页面声明周期管理
    ```
    <activity android:name=”.Hello” android:label=”@string/app_name”>
        <intent-filter>
           <action android:name=”android.intent.action.MAIN” />
           <category android:name=”android.intent.category.LAUNCHER” />
       </intent-filter>
     </activity>
    ```
    - <intent-filter\>标签：声明组件所支持的一组Intent值。
        + <action\>：声明该组件支持的Intent action 。
        + <category\>：声明该组件支持的Intent目录。
        + <data\>：声明该组件支持的Intent 数据的URI及类型。
    - <meta-data\>标签：向活动添加新的元数据。一个活动可以有1个或多个meta-data。其他客户端可以取得这些meta-data以获得关于这个活动的信息。
  + service 后台服务
    ```
    <service
              android:enabled=”true” android:name=”.MyService”>
      	</service>
    ```
    定义一个Service组件。是后台操作，可运行任意长时间的组件 。除了没有屏幕显示，与activity组件一样，可以包含1个或多个该服务支持的元素和值。该节点下可以包含<intent-filter\>和<meta-data\>声明。通过Intent可实现与其他组件的通信。

  + reciever 广播接收器，亦可在代码中注册
    ```
    <receiver android:enabled=”true”
      android:label=”My Broadcast Receiver”
      android:name=”.MyBroadcastReceiver”>
    </receiver>
    ```
    定义一个BroadcastReceiver组件。是使应用来监听全局事件，并对外部事件作出响应的组件 。可以包含1个或多个该Receiver支持的元素和值，该节点下可以包含<intent-filter\>和<meta-data\>声明。通过Intent可实现与其他组件的通信。

  + provide 内容提供者，用于进程间数据交互和共享，即跨进程通信，底层采用Binder机制
    ```
    <provider android:permission=”com.paad.MY_PERMISSION”
        android:name=”.MyContentProvider”
        android:enabled=”true”
        android:authorities=”com.paad.myapp.MyContentProvider”>
    </provider>
    ```
    定义一个ContentProvider组件，是一组标准的方法接口，用来管理和发布持久数据的组件 。可以按照活动中的描述附加1个或多个<meta-data>值。

|属性名|作用|
| --- | --- |
|android:label|它是Android标签属性，是应用程序全局的一个用户可读的标签，也是该应用程序所有组件的默认标签。|
|android:icon|它是应用程序全局的一个图标，也是该应用程序所有组件的默认图标。|
|android:allowBackup|允许应用程序参与备份，默认true。|
|android:logo|logo属性用于配置应用程序的商标。自Android3.0以后，应用程序窗口多了一个标题栏，而应用程序的logo将会出现在那里。|
|android:debuggable|指示应用程序在用户模式的设备上是否可以调试。true，可以调试，false，不能调试，默认值false。需要注意的是，它只在用户模式的机器上生效，用户模式既是买着用的android手机，而虚拟机一般都是工程模式。|
|android:enabled|默认情况下，Android系统会自行实例化每一个应用程序的组件，包括Android四大组件，但如果我们需要自己完成这些事情的话，就需要使用android:enabled属性来限制Android系统的行为。如果这个属性定义在<application>节点中，那么它会默认将每个组件的enabled属性设置为相同的值。如果每一个组件单独定义了这个属性，那么<application>节点上定义的属性对此组件不再生效，就由自己的enabled属性决定。|
|android:hardwareAccelerated|硬件加速渲染功能是否对应用程序中的所有Activity和View启用，如果启用，则为true，否则为false，其默认值是false。从Android 3.0开始，硬件加速的OpenGL渲染器对所有应用程序都有效，这样做的目的是改善大多数2D图形操作的性能。当硬件加速渲染器被启用时，大多数操作（包括Canvas,Paint,Xfermode,ColorFilter,Shader和Camera）都会被加速，这样产生的结果是更顺滑的动画效果，更顺滑的滚动效果以及整体响应的改进。当没有设置这个标志的时候，它的默认值取决于是否配置了android:targetSdkVersion。如果没有配置，则Android默认将android:targetSdkVersion作为当前设备系统的SDK版本。当android:targetSdkVersion属性的值大于或者等于当前系统版本时，则启用硬件加速，反之则禁用硬件加速。|
|android:persistent|用来表明应用程序是否应该在任何时候都保持运行状态, 若为true，则表示应该，false则表示不应该，其默认值为false。通常，应用程序不应该设置本属性，而持续模式仅仅对于某些系统应用程序才有意义。例如电话模块，它在系统启动的时候就处于运行状态。|
|android:process|应用程序所有组件运行的进程名。默认值是当前的应用程序包名。当应用程序的第一个组件运行的时候，Android就会生成一个进程，所有组件全部运行在该进程中。该属性设置为一个与其他应用程序共享的进程名，就可以将两个应用程序的组件说运行在相同的进程里。能这样做的前提是仅在两个应用程序共享一个用户ID并且被赋予相同证书时。如果该属性里设置的名字以冒号开头: 那么在需要的时候它将生成该应用程序的一个私有新进程。如果进程名以小写字母开头，则生成以该进程名命名的一个全局进程。全局进程可以用来与其他应用程序分享，以便降低资源消耗。|
|android:taskAffinity|任务栈|
|android:largeHeap|此属性指示应用程序是否使用一个比较大的堆创建，它是一个布尔值，在没有配置的情况下，它的默认值是false|

3. <uses-permission\>
    
    ```
    <uses-permission  
        android:name=”android.permission.ACCESS_LOCATION”>
    </uses-permission>
    ```
    声明应用程序正常工作所需要的权限。
程序包必须被授予该权限才能进行正确的操作。
    
4. <permission\>
    
    ```
    <permission  
         android:name=”android.permission.ACCESS_LOCATION”>
    </permission>
    ```
    声明一个安全授权。
    用来限制哪些应用可以访问指定的程序包内的组件和特有功能。


#### 在清单文件中声明Activity

每次新建的Activity都需要在AndroidManifest文件中添加如下内容，并将元素添加为元素的子项

```
<manifest ... >
  <application ... >
      <activity android:name=".MainActivity" />
      ...
  </application ... >
  ...
</manifest >
```

**<activity\>元素主要属性**

|属性名|作用|
| --- | --- |
|android:name|组建名，Activity源文件所在的相对路径|
|android:screenOrientation|Activity在屏幕当中显示的方向|
|android:configChanges|Activity捕捉设备状态变化。当所指定属性(Configuration Changes)发生改变时，回调 onConfigurationChanged()函数。1. 设备旋转(orientation),2. 键盘隐藏(keyboardHidden), 3. 屏幕尺寸(screenSize)|
|android:exported|表示该组件是否能够被其它应用程序组件调用或者交互。该属性默认值依赖于它所包含得过滤器(intent-filter)。如果没有intent-filter，则表明该组件只能通过显示Intent调用，所以默认值为false。如果含有intent-filter,则表明该组件能通过隐式Intent调用，所以默认值为true。|
|android:launchMode|activity启动模式。四种：Standard,SingleTop,SingleTask,SingleInstance|
|android:taskAffinity|Activity所吸附的任务栈，该属性一般在lauchMode为singleTask模式才生效。默认taskAffinity于根Activity相同，即应用的包名。|
|android:windowSoftInputMode|键盘弹出后，界面调整模式 1. 当获取焦点时是否弹出键盘 2. 是否减少Activity主窗口大小以腾出空间给软键盘|
|android:allowTaskReparenting|当某个拥有相同affinity任务栈即将返回前台时，当前Activity是否能从启动它得任务栈中转移到与它相同Affinity相同任务栈中去。默认值，false，只能停留在启动它得任务栈。 利用该值的特性，可以强行让一个不再显示的Activity转移到另外的任务栈中。典型的应用，就是让一个程序的Activity转移到另外的应用程序中。|
|android:alwaysRetainTaskState|系统是否一直维持任务栈的状态。该属性只对任务栈根Activity有效，默认值false。通常，用户从launch界面重新打开应用的时候，系统会清空任务栈中根Activity以上所有Activity，比如说，用户停留在主界面很长时间没有操作了。当将该值设置为true的时候，总是返回任务的最后状态，比如说，浏览器应用打开了多个页面，用户不期望下次打开浏览器的时候，之前的页面全部清空了。|
|android:clearTaskOnLaunch|每次从launch启动应用时，是否清楚根Activity以上所有的Activity，默认值false|
|android:enabled|该组件是否能够被实例化。默认true|
|android:excludeFromRecents|该Activity是否排除在用户最近任务列表。默认值false。|
|android:finishOnTaskLaunch|当从launch再次启动任务时，是否finish该实例，默认值false。|
|android:multiprocess|是否可以把Activity新实例放入到启动它的组件所在的进程中。|


#### 启动方式

Activity的启动分为显式和隐式启动，原则上二者不该同时存在，同时存在以显式为主。显式需要指定被启动对象的组件信息（包名、类名），隐式不需要指定目标组件信息，只要Intent可以匹配上目标Activity的IntentFilter设置的过滤信息。


Intent是一个消息传递对象。Intent作用

- 指定当前组件的动作
- 组件之间传递数据

Intent 有两种,一种是显示Intent，一种是隐式Intent。

- 显示Intent 。按名称指定要启动的组件。在自己应用内使用代码显示Intent来启动组件。
新建一个secondActivity，在FirstActivity的onCreate()方法中添加如下代码：
```
Button button1 = (Button)findViewById(R.id.firstButton);
 button1.setOnClickListener(new View.OnClickListener() {
     @Override
     public void onClick(View v) {
         Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
         startActivity(intent);
     }
 });
```
在AndroidManifest中声明Activity：
```
    <activity android:name=".SecondActivity"></activity>
```

- 隐式Intent。不指定特定的组件，而是在AndroidManife.xml文件中声明要执行的常规操作，从而容许其它应用中的组件来处理它。
IntentFilter的过滤信息有action、category、data三种。一个Activity里可以有多组IntentFilter，每组IntentFilter里可以包含多个action、category、data，每组IntentFilter的任意三个及以上action、category、data形成一个匹配约束，只要Java代码里能够匹配一个匹配约束，即可启动目标Activity

**intent-filter**

在使用隐式Intent启动组件的时候，Android系统会通过Intent中的三个信息action data category 与在AndroidManifest.xml文件中注册的组件中的intent-filter进行比较，当匹配一致的时候，启动对应的组件。

在清单文件中intent-filter有三个元素action data category，Intent的过滤器(intent-filter),按照以下优先关系查找：action->data->category

**action**

intent-filter必须包含一个，或者多个action元素，否则该组件匹配不到任何的Intent。Intent中的action与其中的任一个action在字符串形式上完全相同，action方面就匹配成功。系统定义了一些标准的action属性，比如ACTION_MAIN 代表Application入口，ACTION_VIEW,ACTION_WEB_SEARCH等

action是一个字符串，系统预定义了一些action，我们也可以在应用定义自己的action。Java代码里的Intent必须要有action，且这个字符串必须和xml中的action一致，区分大小写。

**data**

data由两部分组成mimeType和URI,data可以仅设置其中一项，或者都设置。

mimeType 是指媒体类型，例如imgage/jpeg，auto/mpeg4和viedo/\*等
URI 则是由scheme、host、port、path | pathPattern | pathPrefix这4部分组成
```
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern >]
```

Scheme：URI的模式，如http、file、content等，如果URI没有指定scheme，那么那么其他参数无效，即整个URI无效。
Host：URI的主机名，如www.baidu.com，如果URI没有指定Host，那么那么其他参数无效，即整个URI无效。
Port：URI中的端口号，比如80。
Path：URI中的完整路径信息。
PathPattern：URI中的完整路径信息，但是可以包含通配符“\*”，要表示字符，\*要写成\\\*，\\要写成\\\\。
PathPrefix：URI中表示路径的前缀信息。


**category**

Intent中如果出现了category，不管几个category，对于每个category来说，他必须是过滤器中定义的category，才能匹配成功。系统默认设置了Intent的category为android.intent.category.DEFAULT

##### 隐式Intent实例

1. 通过浏览器打开网页

  ```
  Intent intent = new Intent();
  intent.setAction(Intent.ACTION_VIEW);
  intent.setData(Uri.parse("http://www.google.com"));
  intent.addCategory(Intent.CATEGORY_BROWSABLE);
  intent.addCategory(Intent.CATEGORY_DEFAULT);
  startActivity(intent);
  ```
2. 通过网页打开本地应用

  ```
  </html>
      <body>
          <div id="header">
           <a href="my.special.scheme://192.168.2.126/param1/param2">link open app
           </a>
          </div>
      </body>
  </html>
  ```
  当点击网页上超链接"link open app"时，会隐式启动我们在清单文件中注册的Activity，清单文件AndroidManifest.xml内容如下：
  ```
  <activity android:name=".intentfilter.IntentFilterTestActivity" >
      <intent-filter>
          <action android:name="android.intent.action.VIEW" />
          <category android:name="android.intent.category.DEFAULT" />
          <category android:name="android.intent.category.BROWSABLE" />
          <data  android:scheme="my.special.scheme"/>
      </intent-filter>
  </activity>
  ```
3. 图片

  ```
  <intent-filter>
      <action android:name="android.intent.action.SEND"/>
      <category android:name="android.intent.category.DEFAULT"/>
      <data android:mimeType="image/*"/>
  </intent-filter>
  ```
  这种规则指定了媒体类型为所有类型的图片，Intent中mimeType必须为“image/\*”才能匹配，xml中的过滤规则虽然没有指定URI但是scheme有默认值为content或file，所以intent中URI的scheme必须为content或file
  ```
  intent1.setDataAndType(Uri.parse("file://asd"),"image/*");
  ```
  更详细的：
  ```
  <intent-filter>
      <action android:name="android.intent.action.SEND"/>
      <category android:name="android.intent.category.DEFAULT"/>
      <data android:mimeType="image/*" android:scheme="file" android:host="asd"/>
  </intent-filter>
  ```
  像这种指定了完整的data的xml，Java代码不可以先用setData方法，再用setType方法指定URI和mimeType，因为这两个方法会将对方的值互相清空，应该用setDataAndType方法。

  此外，以下两种data过滤规则写法等价
  ```
  <data android:scheme="file" android:host="asd"/>
  或
  <data android:scheme="file"/>
  <data android:host="asd"/>
  ```

  判断是否有目标Activity可以响应我们的Intent，从而 避免crash的两种方法。

  - Intent的resolveActivity方法。
  - PackageManager的resolveActivity方法或queryIntentActivities方法，后者不是返回最佳的匹配Activity信息，而是返回所有的匹配Activity信息。MATCH_DEFAULT_ONLY参数是为了将不含DEFAULT的category的Activity排除，因为没有这个的category的Activity无法接收隐式Intent启动的。
  ```
  Button button = (Button)findViewById(R.id.openButton);
  button.setOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View v) {
          Intent intent1 = new Intent(Intent.ACTION_SEND);
          intent1.setDataAndType(Uri.parse("file://asd"),"image/jpg");
          PackageManager packageManager = getApplicationContext().getPackageManager();
          ResolveInfo info = packageManager.resolveActivity(intent1,PackageManager.MATCH_DEFAULT_ONLY);
          if(info != null){
              startActivity(intent1);
          }else{
              Toast.makeText(getApplicationContext(),info.toString()+" == info",Toast.LENGTH_LONG);
          }
      }
  });
  ....
  <activity android:name=".Activity2">
      <intent-filter>
          <action android:name="android.intent.action.VIEW"/>
          <category android:name="android.intent.category.DEFAULT"/>
          <category android:name="com.weihy"/>
          <category android:name="android.intent.category.BROWSABLE"/>
          <data android:scheme="http"/>
      </intent-filter>
      <intent-filter>
          <action android:name="android.intent.action.SEND"/>
          <category android:name="android.intent.category.DEFAULT"/>
          <data android:mimeType="image/jpg" android:scheme="file" android:host="asd"/>
      </intent-filter>
  </activity>
  ```
  值得注意的是，Android7.0以下我们这么写可以让Activity2匹配到Intent里的信息，但是Android7.0的权限问题，这么写会报错`android.os.FileUriExposedException: file://asd exposed beyond app through Intent.getData()`。

  这样的action和category是用来标识这个Activity是应用的入口Activity，并且应用可以出现在应用列表中，二者缺一不可

  ```
  <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>

  ```

### 参考

1. [第一次使用Android Studio时你应该知道的一切配置](https://www.cnblogs.com/smyhvae/p/4390905.html)
2. [Android Studio 项目结构完全解析](https://blog.csdn.net/lote921203/article/details/80257949)
3. [Android Studio 项目目录结构简析](https://www.jianshu.com/p/49b891091810)
