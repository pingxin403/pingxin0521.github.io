---
title: Android 入门与开发配置
date: 2019-04-24 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### 前言

>用一盏灯点燃另一盏灯     ---戈特弗里德·威廉·莱布尼茨

Android是一个开源的，基于Linux的移动设备操作系统，主要使用于移动设备，如智能手机和平板电脑。Android是由谷歌及其他公司带领的开放手机联盟开发的。

<!--more-->

#### 系统架构

![1.png](https://i.loli.net/2019/04/29/5cc64bdc24139.png)

这是Android系统架构图，够一目了然了吧，Android大致可以分为四层架构，五块区域。

- Linux内核层(Linux Kernel)
- 系统运行层
- 应用框架层(Application Framework)
- 应用层(Applications)

**五块区域**

1. Linux Kernel：我们知道Android其实就是一个操作系统，其底层是基于Linux Kernel的，这一层主要完成的是操作系统所具有的功能，比如这一层有许多的驱动程序，正是通过这些驱动程序来驱动我们设备上的硬件设备的。

2. Android Runtime：Android的运行环境，我们学过java的都知道，java程序的运行需要java的核心包的支持，然后通过JVM虚拟机来运行我们的应用程序，这里Android Runtime里的Core Libraries就相当于java的JDK，是运行android应用程序所需要的核心库，Dalvik Virtual Machine就相当于JVM，这时Google专为Android开发的运行android应用程序所需的虚拟机。

3. Liberaries：这里面都是Android的库文件，例如我们访问SQLite数据库的库文件等等。
```
android.app - 提供应用程序模型的访问，是所有 Android 应用程序的基石。
android.content - 方便应用程序之间，应用程序组件之间的内容访问，发布，消息传递。
android.database - 用于访问内容提供者发布的数据，包含 SQLite 数据库管理类。
android.opengl - OpenGL ES 3D 图片渲染 API 的 Java 接口。
android.os - 提供应用程序访问标注操作系统服务的能力，包括消息，系统服务和进程间通信。
android.text - 在设备显示上渲染和操作文本。
android.view - 应用程序用户界面的基础构建块。
android.widget - 丰富的预置用户界面组件集合，包括按钮，标签，列表，布局管理，单选按钮等。
android.webkit - 一系列类的集合，允许为应用程序提供内建的 Web 浏览能力。
```

4. Application Framework：应用程序的框架，这个是非常的重要的，相信Framework这个词大家都应该非常的熟悉了，我们学习Android也主要学的就是这一层，我们通过这些各种各样的框架来实现我们的Application。
```
活动管理者 - 控制应用程序生命周期和活动栈的所有方面。
内容提供者 - 允许应用程序之间发布和分享数据。
资源管理器 - 提供对非代码嵌入资源的访问，如字符串，颜色设置和用户界面布局。
通知管理器 - 允许应用程序显示对话框或者通知给用户。
视图系统 - 一个可扩展的视图集合，用于创建应用程序用户界面。
```

5. Application：顶层中有所有的 Android 应用程序。你写的应用程序也将被安装在这层。这些应用程序包括通讯录，浏览器，游戏等。都使用Java语言编写。

#### Android应用特色

![1.jpg](https://i.loli.net/2019/04/29/5cc65a4c15b7e.jpg)

**四大组件**

什么是四大组件？分别是活动(Activity)、服务(Service)、广播接收器(BroadCast Receiver)和内容提供器(Content Provider)。

其中活动(Activity)就是Android应用程序中看得东西，也是用户打开一个应用程序的门面，并且与用户交互的界面，比较高调。服务(Service)，则比较低调了，一直在后台默默的付出，即使用户退出了，服务仍然是可以继续运行的。

广播接收器(BroadCast Receiver)，则允许你的应用接收来自各处的广播消息，比如电话、短信等，可以根据广播名称不同，做相应的操作处理，当然了， 除了可以接受别人发来的广播消息，自身也可以向外发出广播消息，自产自销。

内容提供器(Content Provider)，则为应用程序之间共享数据提供了可能，比如你想要读取系统电话本中的联系人，就需要通过内容提供器来实现。

**丰富的系统控件**

Android系统为开发者提供了丰富的系统控件，我们可以编写漂亮的界面，也可以通过扩展系统控件，自定义控件来满足自我的需求，常见控件有：TextView、Buttion、EditText、一些布局控件等。

**持久化技术**

Android系统还自带了SQLite数据库，SQLite数据库是一种轻量级、运算速度极快的嵌入式关系型数据库。它不仅支持标准的SQL语法，还可以通过Android封装好的API进行操作，让存储和读取数据变得非常方便。

**地理位置定位**

移动设备和PC相比，地理位置定位是一大亮点，现在基本Android手机都内置了GPS，我们可以通过GPS，结合我们的创意，打造一款基于LBS的产品，是不是很酷的事情啊，再说，目前火热的LBS应用也不是空穴来风的，不过在天朝，因为可恶的GFW，只能用些本土化的地图API，比如百度地图、高德地图。要是哪天能用上大谷歌的地图，那才是高大上啊。

**强大的多媒体**

Android系统提供了丰富的多媒体服务，比如音乐、视频、录音、拍照、闹铃等，这一切都可以在程序中通过代码来进行控制，让你的应用变得更加丰富多彩。

**传感器**

Android手机中内置了多种传感器，比如加速传感器、方向传感器，这是移动设备的一大特点，我们可以灵活地使用这些传感器，可以做出很多在PC上无法实现的应用。比如“微信摇一摇"\_你懂得，“搜歌摇一摇”等功能。


### 开发环境配置

环境：Ubuntu 18.04

#### 安装JDK

JVM ：英文名称（Java Virtual Machine），就是我们耳熟能详的 Java 虚拟机。它只认识 xxx.class 这种类型的文件，它能够将 class 文件中的字节码指令进行识别并调用操作系统向上的 API 完成动作。所以说，jvm 是 Java 能够跨平台的核心，具体的下文会详细说明。

JRE ：英文名称（Java Runtime Environment），我们叫它：Java 运行时环境。它主要包含两个部分，jvm 的标准实现和 Java 的一些基本类库。它相对于 jvm 来说，多出来的是一部分的 Java 类库。

JDK ：英文名称（Java Development Kit），Java 开发工具包。jdk 是整个 Java 开发的核心，它集成了 jre 和一些好用的小工具。例如：javac.exe，java.exe，jar.exe 等。

我们配置java环境需要安装JDK，从[官网](https://www.oracle.com/technetwork/java/javase/overview/index.html)下载对应版本。

![](https://i.loli.net/2019/04/08/5caaca86967e8.png)

我们下载java 11 和java 8 。

![](https://i.loli.net/2019/04/08/5caaca86a817a.png)

选择java 11 安装包。

![](https://i.loli.net/2019/04/08/5caaca86a3daf.png)

选择java 8 安装包。

![](https://i.loli.net/2019/04/08/5caaca86b3d70.png)

把安装包下载后，解压并重命名，移动到`/opt/java/`下，如下：

``` （shell）
$ tree -L 2  /opt/java/
/opt/java/
├── java-11
│   ├── bin
│   ├── conf
│   ├── include
│   ├── jmods
│   ├── legal
│   ├── lib
│   ├── README.html
│   └── release
└── java-8
     ├── bin
     ├── COPYRIGHT
     ├── include
     ├── javafx-src.zip
     ├── jre
     ├── lib
     ├── LICENSE
     ├── man
     ├── README.html
     ├── release
     ├── src.zip
     ├── THIRDPARTYLICENSEREADME-JAVAFX.txt
     └── THIRDPARTYLICENSEREADME.txt

```
我们为java 11 配置环境变量：

``` （shell）
$ sudo vim /etc/profile
#在文件最后面添加以下，保存并退出

#java
export JAVA_HOME=/opt/java/java-11
export JER_HOME=${JAVA_HOME}/jre
export PATH=${PATH}:${JAVA_HOME}/bin

$ source /etc/profile
#验证是否设置成功
$ java -version
java version "11.0.3" 2019-04-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.3+12-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.3+12-LTS, mixed mode)
```

**解决sudo情况下PATH无效**
```（shell）
$ sudo java --version
#会出现没有找到java命令
$ vim ~/.bashrc
#找到alias声明位置，在后面添加如下，保存并退出

alias sudo="sudo env PATH=$PATH"

$ source  ~/.bashrc

$sudo java --version
hyp@hyp-HP-Notebook:~$ sudo java -version
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
```

#### 下载安装SDK

需要一个网速较快的地方，硬盘剩余空间大于30 G

从[国内网站](http://tools.android-studio.org/index.php/sdk)下载适合的Sdk Tools，

更改文件夹权限：
```
sudo mkdir /opt/android
sudo chmod 777 /opt/android
sudo mkdir /opt/android/as
sudo chmod 777 /opt/android/as
sudo mkdir /opt/android/sdk
sudo chmod 777 /opt/android/sdk
```

解压后放置`/opt/android/sdk`中,运行`/opt/android/sdk/tools/android`,然后下载最新版的Android SDK（28），安装Android SDK Platform-tools（28.0.2）和Android Build-Tools（28.0.2）。

之后选择Install--》同意--》安装，就会开始下载SDK和工具。

#### 安装Android Studio

从[官网](https://developer.android.google.cn/studio#downloads)下载3.4.0的Android Studio，完成后解压到`/opt/android/as`下。

#### 运行

```
/opt/android/as/bin/studio.sh
```
启动后设置proxy-》Manual proxy configuration，从下面找到自己能用的

- 南阳理工学院镜像服务器地址:
```
mirror.nyist.edu.cn 端口:80
```
- 中国科学院开源协会镜像站地址:
```
  IPV4/IPV6: mirrors.opencas.cn 端口:80

  IPV4/IPV6: mirrors.opencas.org 端口:80

  IPV4/IPV6: mirrors.opencas.ac.cn 端口:80
```
- 上海GDG镜像服务器地址:
```
sdk.gdgshanghai.com 端口:8000
```
- 北京化工大学镜像服务器地址:
```
  IPv4: ubuntu.buct.edu.cn/ 端口:80

  IPv4: ubuntu.buct.cn/ 端口:80

  IPv6: ubuntu.buct6.edu.cn/ 端口:80
```
- 大连东软信息学院镜像服务器地址:
```
mirrors.neusoft.edu.cn 端口:80
```
- 腾讯Bugly 镜像:
```
android-mirror.bugly.qq.com 端口:8080
```
腾讯镜像使用方法:<http://android-mirror.bugly.qq.com:8080/include/usage.html>

然后在选择SDK的时候选择刚刚的地址`/opt/android/sdk`。

#### 离线配置（可选）

从[官网](https://developer.android.google.cn/studio#downloads)下载离线组件，Android Gradle Plugin 和 Google Maven dependencies，下载完成后解压，然后移动到 `%USER_HOME%/.android/manual-offline-m2/`.

```bash
$ unzip -O CP936 offline-android-gradle-plugin-preview.zip
$ unzip -O CP936 offline-gmaven-stable.zip
$ mkdir ~/.android/manual-offline-m2/
$ ls
android-gradle-plugin-3.5.0-beta01
gmaven_stable
$ cd gmaven_stable
$ cp -frap ./* ~/.android/manual-offline-m2/
$ cd ../android-gradle-plugin-3.5.0-beta01/
$ cp -frap ./* ~/.android/manual-offline-m2/
$ rm -rf gmaven_stable/ android-gradle-plugin-3.5.0-beta01/
```

以后更新时直接替代即可。

下载并解压缩脱机组件后，需要创建一个脚本告诉Gradle使用时包含下载的脱机组件搜索项目的Android Gradle插件和Google Maven依赖项。

要创建脚本，请按照下面的说明继续。记住，你需要即使在脱机更新之后，也只能创建和保存此脚本一次。组件。

这个脚本适用于本机所有的Gradle项目：

```bash
$ mkdir -p ~/.gradle/init.d/
$ vim ~/.gradle/init.d/offline.gradle
def reposDir = new File(System.properties['user.home'], ".android/manual-offline-m2")
def repos = new ArrayList()
reposDir.eachDir {repos.add(it) }
repos.sort()

allprojects {
  buildscript {
    repositories {
      for (repo in repos) {
        maven {
          name = "injected_offline_${repo.name}"
          url = repo.toURI().toURL()
        }
      }
    }
  }
  repositories {
    for (repo in repos) {
      maven {
        name = "injected_offline_${repo.name}"
        url = repo.toURI().toURL()
      }
    }
  }
}
```



#### Gradle配置

新建一个Hello World项目，会发现构建时提示构建失败，打开`～/.gradle/wrapper/dists/`,查看下面的是否有gradle-<version\>-all的文件夹，如果有，请从[官网](http://services.gradle.org/distributions/)上下载对应版本的zip包，**不要解压**，直接放在`～/.gradle/wrapper/dists/gradle-<version>-all/<乱码>/`文件夹下，并删除其中的`.part`和`.lck`文件，再次打开项目，选择构建，就会发现构建成功。

如果没有文件夹，查看项目下的`./gradle/wrapper/gradle-wrapper.properties`文件，查看distributionUrl后的gradle版本，操作和上面相同。

至此，开发环境算是构建完成，注意SDK最好通过SDK Tools安装，使用Android Studio容易出现网络问题。

##### (AndroidStudio)gradle配置多个代码仓库repositories

```
repositories {
    maven { url "https://jitpack.io" }
    maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
    maven { url 'http://maven.oschina.net/content/groups/public/' } 
    maven { url 'https://oss.sonatype.org/content/repositories/snapshots/' } 
    maven { url "http://maven.springframework.org/release" } 
    maven { url "http://maven.restlet.org" } 
    maven { url "http://mirrors.ibiblio.org/maven2" }
    maven {
        url "http://repo.baichuan-android.taobao.com/content/groups/BaichuanRepositories/"
    }
    maven { url 'https://maven.fabric.io/public' }
    mavenCentral()
    jcenter()
    google()
}
```

当然 jcenter在国内的话，基本没人使用了,当然作为android开发者24小时翻墙，是可以用得。。这里非常推荐使用阿里，速度，那叫一个快。。。千万别把这些一股脑地都放到自己的项目中。用不了，还耽误搜索时间。

在用户目录中的.gradle文件夹中新建init.gradle文件，我的MAC目录是：/home/hyp/.gradle，将下面代码粘贴进去，不过需要建立一个环境变量GRADLE_USER_HOME，指向你gradle的家目录，并且重启计算机

配置环境变量，指向maven仓库：`export GRADLE_USER_HOME=~/.gradle`

```
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            def url = repo.url.toString()
            if ((repo instanceof MavenArtifactRepository) && (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com'))) {
                project.logger.lifecycle 'Repository ${repo.url} replaced by $REPOSITORY_URL .'
                remove repo
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```



> 注意：用户/用户名/目录下找到.gradle文件夹里的gradle.properties 文件注释掉prox使用


#### Hello World项目

上面已经可以构建项目，我们就来运行一下这个项目吧，可以选择AVD（安卓虚拟机）或者真机测试，AVD的话新建即可（需要/dev/vmmon）

真机的话需要连接到电脑，开启真机的开发者模式和调试模式。

选择开启即可，如果期间遇到问题，请百度或者谷歌，因为我不能够完全预料您所遇到的问题。


Android Studio更多配置：[第一次使用Android Studio时你应该知道的一切配置](https://www.cnblogs.com/smyhvae/p/4390905.html)

### Tools详解

Android SDK包含了各种各样的定制工具,位于`/opt/android/sdk/platform-tools/`，简介如下：

**Android模拟器（Android Emulator）**

它是在你的计算机上运行的一个虚拟移动设备。你可以使用模拟器来在一个实际的Android运行环境下设计，调试和测试你的应用程序。

**Android调试桥（Android Debug Bridge (adb) ）**

Adb 工具可以让你在模拟器或设备上安装应用程序的.apk文件，并从命令行访问模拟器或设备。你也可以用它把Android模拟器或设备上的应用程序代码和一个标准的调试器连接在一起。

**层级观察器 （Hierarchy Viewer）**

层级观察器工具允许你调试和优化你的用户界面。它用可视的方法把你的视图（view）的布局层次展现出来，此外还给当前界面提供了一个具有像素栅格(grid)的放大镜观察器，这样你就可以正确地布局了。

**9-patch**

Draw 9-patch工具允许你使用所见即所得（WYSIWYG）的编辑器轻松地创建NinePatch图形。它也可以预览经过拉伸的图像，高亮显示内容区域。


**Dalvik 调试监视器服务（Dalvik Debug Monitor Service (ddms)）**

这个工具集成了Dalvik（为Android平台定制的虚拟机（VM）），能够让你在模拟器或者设备上管理进程并协助调试。你可以使用它杀死进程，选择某个特定的进程来调试，产生跟踪数据，观察堆（heap）和线程信息，截取模拟器或设备的屏幕画面，还有更多的功能。

**Android Asset Packaging Tool (aapt)**

Aapt工具可以让你创建包含Android应用程序二进制文件和资源文件的.apk文件。Android接口描述语言（Android Interface Description Language (aidl)）,可以让你生成进程间的接口的代码，诸如service可能使用的接口。

**sqlite3**

这个工具能够让你方便地访问SQLite 数据文件。这些数据文件是由Android 应用程序创建并使用的。

**Traceview**

这个工具可以将你的Android 应用程序产生的跟踪日志（trace log）转换为图形化的分析视图。

**mksdcard**

帮助你创建磁盘映像（disk image），你可以在模拟器环境下使用磁盘映像来模拟外部存储卡（例如SD 卡）。

**dx**

Dx gongju 将.class字节码（bytecode）转换为Android字节码（保存在.dex文件中） 。

**UI/Application Exerciser Monkey**

Monkey是在模拟器上或设备上运行的一个小程序，它能够产生为随机的用户事件流，例如点击(click)，触摸(touch)，挥手（gestures），还有一系列的系统级事件。你可以使用Monkey来给你正在开发的程序做随机的，但可重复的压力测试。

**activitycreator**

一个可以产生Ant build 文件的脚本，你可以使用它编译你的android 应用程序。如果你正在Eclipse上开发，并使用ADT插件，你不必使用这个脚本。

### 三种移动APP（应用程序）开发方式比较

#### 名词介绍

1. Native APP

   Native APP 指的是原生程序，一般依托于操作系统，有很强的交互，是一个完整的App，可拓展性强，需要用户下载安装使用。（简单来说，原生应用是特别为某种操作系统开发的，比如iOS、Android、黑莓等等，它们是在各自的移动设备上运行的）

   该模式通常是由“云服务器数据+APP应用客户端”两部份构成，APP应用所有的UI元素、数据内容、逻辑框架均安装在手机终端上。

   原生应用程序是某一个移动平台（比如iOS或安卓）所特有的，使用相应平台支持的开发工具和语言（比如iOS平台支持Xcode和Objective-C，安卓平台支持Eclipse和Java）。原生应用程序看起来（外观）和运行起来（性能）是最佳的。

2. Web APP

   Web App 指采用Html5语言写出的App，不需要下载安装。类似于现在所说的轻应用。生存在浏览器中的应用，基本上可以说是触屏版的网页应用。（Web应用本质上是为移动浏览器设计的基于Web的应用，它们是用普通Web开发语言开发的，可以在各种智能手机浏览器上运行）

   Web App开发即是一种框架型APP开发模式（HTML5 APP 框架开发模式），该开发具有跨平台的优势，该模式通常由“HTML5云网站+APP应用客户端”两部份构成，APP应用客户端只需安装应用的框架部份，而应用的数据则是每次打开APP的时候，去云端取数据呈现给手机用户。

   HTML5应用程序使用标准的Web技术，通常是HTML5、JavaScript和CSS。这种只编写一次、可到处运行的移动开发方法构建的跨平台移动应用程序可以在多个设备上运行。虽然开发人员单单使用HTML5和JavaScript就能构建功能复杂的应用程序，但仍然存在一些重大的局限性，具体包括会话管理、安全离线存储以及访问原生设备功能（摄像头、日历和地理位置等）。

3. Hybrid APP

   Hybrid APP指的是半原生半Web的混合类App。需要下载安装，看上去类似Native App，但只有很少的UI Web View，访问的内容是 Web 。

   混合应用程序让开发人员可以把HTML5应用程序嵌入到一个细薄的原生容器里面，集原生应用程序和HTML5应用程序的优点（及缺点）于一体。

   混合应用大家都知道是原生应用和Web应用的结合体，采用了原生应用的一部分、Web应用的一部分，所以必须在部分在设备上运行、部分在Web上运行。不过混合应用中比例很自由，比如Web 占90%，原生占10%；或者各占50%。

   有些应用最开始就是包了个原生客户端的壳，其实里面是HTML5的网页，后来才推出真正的原生应用。比较知名的APP，比如手机百度和淘宝客户端 Android版，走的也是Hybrid App的路线，不过手机百度里面封装的不是WebView，而是自己的浏览内核，所以体验上更像客户端，更高效。

#### 3种APP技术特性

**1.Native APP**

**优点：**

- 能够与移动硬件设备的底层功能，比如个人信息，摄像头以及重力加速器等等。
- 可访问手机所有功能（GPS、摄像头）。
- 速度更快、性能高、整体用户体验不错。
- 可线下使用（因为是在跟Web相对地平台上使用的）。
- 支持大量图形和动画
- 容易发现（在App Store里面和应用商店里面）和重新发现（应用图标会一直在主页上），对于苹果而言，应用下载能创造盈利（当然App Store抽取20-30% 的营收）
- 比移动Web App运行快
- 一些商店与卖场会帮助用户寻找原生App
- 官方卖场的应用审核流程会保证让用户得到高质量以及安全的App
- 官方会发布很多开发工具或者人工支持来帮助你的开发
- 页面存放于本地

**缺点：**

- 开发成本高，尤其是当需要多种移动设备来测试时
- 因为是不同的开发语言，所以开发，维护成本也高
- 因为用户使用的App版本不同，所以你维护起来很困难
- 支持设备非常有限（一般是哪个系统就在哪个平台专属设备上用）
- 官方卖场审核流程复杂且慢，会严重影响你的发布进程
- 上线时间不确定（App Store审核过程不一）
- 内容限制（App Store限制）
- 获得新版本时需重新下载应用更新（提示用户下载跟新，用户体验差）

**2.Web APP**

**优点：**

- 跨平台开发、用户不需要去卖场来下载安装App,开发速度快
- 任何时候都可以发布App，因为根本不需要官方卖场的审核
- 纯H5 APP快速开发、低成本、多平台，与很多APP开发方式不同的是-图文混合的排版（正是这些复杂多变的CSS样式消耗了性能，但是它带来了排版的多样性，能够细致到每一个字宽行高和风格的像素级处理，才是H5的优异之处）
- 支持设备广泛
- 较低的开发成本
- 可即时上线
- 无内容限制
- 用户可以直接使用最新版本（自动更新，不需用户手动更新）
- 跨平台开发
- 用户不需要去卖场来下载安装App
- 如果你已经有了一个Web App，你可以使用 responsive web design来辅助改进
- 页面存放于web服务器（受限于UIwebview）（减少了内存，但是会增加服务器的压力）

**缺点：**

- 只能使用有限的移动硬件设备功能,无法使用很多移动硬件设备的独特功能
- 要同时支持多种移动设备的浏览器让开发维护的成本也不低（也要适配不同的浏览器），如果用户使用更多的新型浏览器，那问题就更不好处理了
- 对于用户来说，这种App很难被用户发现
- 这里的数据获取都是在资源页面上异步完成的，因为只有这样才能让这些资源页面完成预加载或者渲染。（异步的话都涉及到耗时的问题）
- 表现差（对联网的要求比较大）
- 用户体验没那么炫
-  图片和动画支持性不高
- 没法在App Store中下载、无法通过应用下载获得盈利机会
- 对手机特点有限制（摄像头、GPS等）
- 无法体会包括会话管理、安全离线存储以及访问原生设备功能（摄像头、日历和地理位置等）
- 页面跳转更加费力，不稳定感更强
- 更小的页面空间（由于浏览器的导航本身占用一部分屏幕空间），更大的信息记忆负担
- 导航不明显，原有底部导航消失，有效的导航遇到挑战
- 交互动态效果收到限制，影响一些页面场景、逻辑的理解。比如登录注册流程的弹出、完成及异常退出，做好文字提示。

**3.Hybrid APP**

**（1）第一种方案：Web架构为重**

**优点：**

- 全Web开发，一定程度上有利于Web前端技术人员快速地构建页面样式
- 有利于在不同的平台上面展示同一个交互层
- 便于调试，开发的时候可以通过浏览器的
- 方式进行调试，工具丰富。
- 兼容多平台
- 顺利访问手机的多种功能
- App Store中可下载（Wen应用套用原生应用的外壳）
- 可线下使用
- 页面存放于本地和服务器两种方式，部署应用程序(受限于UIwebview)

**缺点：**

- 不确定上线时间
- 虽然说你可以专注在界面以及交互开发上了，但是这页会成为一个缺点，比如说要仿造一个iOS的默认设置界面，就需要大量的html以及css代码了，而且效果不一定和iPhone上面的界面一样好
- 用户体验不如本地应用
- 性能稍慢（需要连接网络）
- 技术还不是很成熟（比如Facebook现在的应用属于混合应用它可以在许多App Store畅通无阻，但是掺杂了大量Web特性，所以它运行速度比较慢，而现在为了提高性能FB又决定采用原生应用）

**（2）第二种方案：编译转换方式**

**优点：**

- 利用自己熟悉的语言进行应用开发。

**缺点：**

- 严重依赖于其工具厂商提供的工具包，调试的时候就要有全套的工具。

**（3）第三种方案：Native架构为重（主流）**

**优点：**

- 最稳定的Hybrid App开发方式了，交互层的效率上由Native的东西解决了，而且架构上基本就是在App内写网页，连App Store都是采用了该种方案；

**缺点：**

- 团队至少需要两个工程师，一个是Web的，一个是iOS或者Android的。当然如果开发人员会两种技术也可独立承担；还是运行效率，要权衡好多少界面采用Web来渲染，毕竟WebView的效率会相对降低，以前Facebook就是因为Web的渲染效率低下，把整个应用改为原生的解决方案。当然这里面可以通过优化来解决，但是优化也是有限度的。

#### 3种APP对比分析

对用户来讲差别主要是用户体验，如果WebApp做得好也能接近原生App的效果；

对于开发人员，WebApp更加易于移植到多个平台，减少非常多的工作量。

**1.主要区别**

**原生APP中：**

- 每一种移动操作系统都需要独立的开发项目；
- 每种平台都需要独立的开发语言。Java(Android), Objective-C(iOS)以及Visual C++(Windows Mobile)等等，需要使用各自的软件开发包，开发工具以及各自的控件。
- Native App（原生型APP）需要开发“云服务器数据中心”和“APP客户端”
- 每次获取最新的APP功能，需要升级APP应用
- 原生型APP应用的安装包相对较大，包含UI元素、数据内容、逻辑框架；
- 手机用户无法上网也可访问APP应用中以前下载的数据
- 原生型的APP可以调用手机终端的硬件设备（语音、摄像头、短信、GPS、蓝牙、重力感应等）
- APP应用更新新功能，涉及到每次要向各个应用商店进行
- 提交审核。
- 适用企业：游戏、电子杂志、管理应用、物联网等无需经常更新程序框架的APP应用。

**WebAPP中：**

- 因为运行在移动设备的浏览器上，所以只需要一个开发项目
- 这种应用可以使用HTML5,CSS3以及JavaScript以及服务器端语言来完成（PHP,Ruby  on Rails,Python），这里可没有标准的SDK，基本任意选择别忘了有一些跨平台的开发工具，比如PhoneGap, Sencha  Touch 2，APPcan以及Appcelerator Titanium等等。
- Web APP需开发“html5云网站”和“APP客户端”
- 每次打开APP，都要通过APP框架向云网站取UI及数据
- 手机用户无法上网则无法访问APP应用中的数据
- 框架型的APP无法调用手机终端的硬件设备，（语音、摄像头、短信、GPS、蓝牙、重力感应等）
- 框架型APP的访问速度受手机终端上网的限制，每次使用均会消耗一定的手机上网流量
- 框架型APP应用的安装包小巧，只包含框架文件，而大量的UI元素、数据内容刚存放在云端
- APP用户每次都可以访问到实时的最新的云端数据
- APP用户无须频繁更新APP应用，与云端实现的是实时数据交互
- 适用企业：电子商务、金融、新闻资讯、企业集团，需经常更新内容的APP应用。

**2.开发难度区别**

移动web和混合App开发难度对于web开发者来说相对较低，而且可以充分利用现有的web开发工具和工作流程

**3.发布渠道和更新方式**

混合App可以在应用商店App Store发布，但可以自主更新，而原生App的更新必须通过应用商店App Store。

**4.移动设备本地API访问**

混合App可以通过JavaScript API访问到移动设备的摄像头、GPS；而原生App可以通过原生编程语言访问设备所有功能。

**5.跨平台和可移植性**

基于浏览器的移动web最好的可移植性和跨平台表现；混合App也能节省跨平台的时间和成本，只需编写一次核心代码就可部署到多个平台，而原生App的跨平台性能最差。

**6.搜索引擎友好**

只有移动web对搜索引擎友好，可与在线营销无缝整合。

**7.货币化**

混合App除广告外，还支持付费下载及程序内购买；原生App的程序内购买金额2012年首次超过下载收费。

**8.消息推送**

只有混合App和原生App支持消息推送，这能增加用户忠诚度。

**9.获取方法区别**

**原生APP中：**

- 直接下载到设备
- 以独立的应用程序运行(并不需要浏览器)
- 用户必须手动去下载并安装这些原生App
- 有一些商店与卖场来帮助用户寻找你的App，

**WebAPP中：**

- 从移动设备上的浏览器访问
- 不需要安装额外的软件
- 软件更新只需要服务器就够了
- 因为现在没有什么商品或卖场提供这种App，所以如何搜索这些移动Web App相当不简单

**10.版本控制区别**

**原生APP中：**

- 用户可以自由地选择是否更新软件版本，所以会出现不同用户同时使用不同版本的情况

**WebAPP中：**

- 所有的用户都是用同样的版本

#### 如何判断一个混合APP开发的页面形式

1. 断网检查不是绝对的，web app并不一定是在远程服务器上的， 也能pack在程序里，load本地的资源也能算是web app。

2. 在系统设置里进入“开发者选项”，勾选“显示布局边界”，然后就可以看得出来了。（比较靠谱）

3. 一般web界面有明显的加载的过程，你看页面的最上面一般有一个加载的进度条，不过这个进度条一般加载也比较快，有些应用在这样的说明页面会有刷新操作、这样你断网再刷新就会提示网址找不到

4. 网页的一般就在手机的当前界面加载一个url地址。

5. （快速）滚动起来是否比较卡

6. 图片加载失败的图标

#### 怎样选择开发模式（视情况而定）

近年来随着移动设备类型的变多，操作系统的变多，用户需求的增加，对于每个项目启动前，大家都会考虑到的成本，团队成员，技术成熟度，时间，项目需求等一堆的因素。

因此，开发App的方案已经变得越来越多了。无数的人参与或者看到过一个讨论：原生开发还是混合开发，又或者是Web开发？要结实践和自身的情况。

1. 比如，你的预算是多少？预算充足的话可以开发几个本地应用加一个Web应用

2. 你的应用需要什么时候面市？Web应用可以很快地开发然后直接推出来

3. 你的应用需要包含什么特点和功能？如果跟手机的某些功能深度整合了，比如摄像头，需要呈现大量图形和动画就选原生
   应用好点

4. 你的应用是否一定需要网络

5. 你的应用的目标硬件设备是所有的移动设备还是仅仅只是一部分而已

6. 你自己已经熟悉的开发语言，或者说现有资源

7. 这个应用对于性能要求是否苛刻

8. 如何靠这个应用赢利我想这几个问题应该能让你做出明智的选择

9. 你的应用是否需要使用某些设备的特殊功能，比如摄像头，摄像头闪光灯或者重力加速器

10. 移动Web无所不在，移动Web是目前唯一的支持各种设备访问的平台，与桌面Web一样，移动Web支持各种标准的协议。移动Web也是唯一一个可供开发者发布移动应用的，平台，它将各种移动交互与桌面任务有效地连接了起来；而开发Native App可以充分利用设备的特性，而这一点往往是Web浏览器做不到的，所以对一个产品本身而言，Native App是最佳的选择。

11. 为应用收费(人们的观念webApp是不收费的)用原生开发模式

12. Web Apps是唯一一个经久不衰的移动内容、服务、应用开发平台。

13. 使用定位功能、使用摄像头、使用感应器、访问文件系统、离线用户、多点触控：双击、缩放及其他组合的用户界面（UI）手势；快速图形API：原生平台为你提供了显示最快速的图形。如果你显示只有寥寥几个元素的静态屏幕，这个功能可能不太重要，但如果你使用大量数据，需要快速刷新，这项功能却很重要；流畅动画：与快速图形API有关的是实现流畅动画的功能。这在动画、高度交互的报表或者转换照片和声音的计算密集型算法中显得尤为重要；内置部件：摄像头、地址簿、地理位置及设备的其他原生功能可以无缝地整合到移动应用程序中。另一个重要的内置部件是加密的存储装置，这方面稍后会有详细介绍；易于使用：原生平台是人们耳熟能详的平台，所以如果你在这个熟悉的平台上添加人们期望的所有原生功能，也就拥有了一款使用起来完全更容易的应用程序时用原生

14. 是原生App还是移动Web App，主要受商业目标，目标用户，以及技术需要这些因素影响的。其实更多时候你也不要为选择那种App模式烦恼，正如本文提到，类似Facebook这样的公司就为用户提供了两种选择。然而对于大部分人来说，预算，资源限制将会逼迫我们只能选择其中一种（或者只能以其中一种为重点

#### WebAPP和原生APP交互区别

**1.Web APP受限因素**

相比Native App，Web App体验中受限于以上5个因素：网络环境，渲染性能，平台特性，浏览器限制，系统限制。

**（1）网络环境，渲染性能**

Web APP对网络环境的依赖性较大，因为Web  APP中的H5页面，当用户使用时，去服务器请求显示页面。如果此时用户恰巧遇到网速慢，网络不稳定等其他环境时，用户请求页面的效率大打折扣，在用户使用中会出现不流畅，断断续续的不良感受。同时，H5技术自身渲染性能较弱：对复杂的图形样式，多样的动效，自定义字体等的支持性不强。
 因此，基于网络环境和渲染性能的影响，在设计H5页面时，应注意以下几点：

- **简化不重要的动画/动效**
- **简化复杂的图形文字样式**
- **减少页面渲染的频率和次数**

**具体案例：**设计Web APP要去除冗余的功能，回溯本源，只给用户提供最初的本质需求。既符合H5精简功能又达到了突出核心功能的设计原则。

- **切记重要的并不是我们提供的信息量有多大，而是我们能否给他们提供真正需要的信息。**
- **切记要减少功能入口，增强用户的专注度，不要分散用户的注意力。**

**（2）浏览器限制**

通常Web App生存于浏览器里，宿主是浏览器。不同的浏览器自身的属性不尽相同，如：浏览器自带的手势，页面切换方式，链接跳转方式，版本兼容问题等等。

**具体案例1：**UC 浏览器和百度浏览器自身支持手势切换页面。手指从左侧滑动页面，返回至上一级。百度手机助手H5页面，顶部Banner支持手势左右滑动切换。这一操作与浏览器自身手势是冲突的。

**具体案例2：**基于浏览器的Web  APP在打开新的模块中的页面时，大多会新开窗口来展现。例如用户在使用购物类APP时，浏览每日精选模块时，每当打开新的商品时，默认新开一个窗口。这样的优劣势显而易见：优势是能够记录用户浏览过的痕迹，浏览过的商品，以便后续横向对比；劣势是过多的页面容易使用户迷失在页面中。

正如Google开发手册里描述：当用户打开一个Web App的时候，他们期待这个应用就像是一个单个应用，而不是一系列网页的结合。然而，什么情况下需要跳转页面，什么情况下在当前页面展示则需要设计师细致考量。

因此，Web App基于浏览器的特性，从设计角度应该遵循以下了两点：

- **少用手势，避免与浏览器手势冲突。**
- **减少页面跳转次数，尽量在当前页面显示。**

**（3）系统限制，平台特性**

由于Html5语言的技术特性，无法调用系统级别的权限。例如，系统级别的弹窗，系统级别的通知，地理信息，通讯录，语音等等。且与系统的兼容性也会存在一些问题。以上限制通常导致APP的拓展性不强，体验相对较差。**具体案例：百度网页地图与百度APP地图。**

Web版地图基于浏览器展现，因此，不能全屏显示地图，给用户的眼界带来局限感；相反，Native 版地图以全屏展现的形式，很好的拓展了用户的视野。整个界面干净简洁，首页去除冗余功能。

Web 版地图耗费的流量大于Native版，且不能预先缓存离线地图。对于地理位置的判断也是基于宿主浏览器，而非Web地图本身。获取路线后，对于更换到达方式，相对来说是不便利的。

相反，Native 版地图，能够直接访问用户的地理位置，能够很清晰的为用户展现App规划的路线，并能轻松的查看多种路线方案，以便做出符合自己的最佳方案。对于切换公交，走路，自驾等路线方式也是只需一键操作。

Native 版地图相对于 Web版地图增加更多情感化，易用的功能，如：能够记录用户的生活轨迹，记录用户的点滴足迹，能够享受躲避拥堵方案等。而Web版地图基于技术框架，很难实现以上功能，从用户体验角度来看，弱于Native版地图。

**2.Web APP设计要点**

**（1）简化**

- 简化不重要的动画/动效
- 简化复杂的图形文字样式

**（2）少用**

- 少用手势，避免与浏览器手势冲突
- 少用弹窗

**（3）减少**

- 减少页面内容
- 减少控件数量
- 减少页面跳转次数，尽量在当前页面显示

**（4）增强**

- 增强Loading时的趣味性
- 增强页面主次关系
- 增强控件复用性

**3.有效的WebAPP产品设计**

有效的导航设计：基本的快捷导航中包括返回常用页面（如首页、我的等）的快捷方式

出现深层架构时，及时补充返回重要层级页面的快捷方式。

情境式导航，方便用户快捷跳转到ta想去的页面，如购买结束时提供查看订单详情的按钮。

WebAPP更加需要画页面跳转的流程图，摸清各个页面的入口，尤其是页面返回的流程；有些简化的返回按钮，可以特殊注明返回到的页面。




### 参考：

1. [Android SDK中tools详解](https://blog.csdn.net/weixin_40251830/article/details/82734797)
2. ["浅谈Android"第一篇：Android系统简介](https://www.cnblogs.com/cr330326/p/4229026.html)
