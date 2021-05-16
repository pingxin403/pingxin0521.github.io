---
title: Gradle 入门
date: 2019-05-06 11:18:59
tags:
 - 工具
 - Gradle
categories:
 - 工具
 - Gradle
---

### 前言

简单的说，Gradle是一个构建工具，它是用来帮助我们构建app的，构建包括编译、打包等过程。我们可以为Gradle指定构建规则，然后它就会根据我们的“命令”自动为我们构建app。Android   Studio中默认就使用Gradle来完成应用的构建。有些同学可能会有疑问：”我用AS不记得给Gradle指定过什么构建规则呀，最后不还是能搞出来个apk。“   实际上，app的构建过程是大同小异的，有一些过程是”通用“的，也就是每个app的构建都要经历一些公共步骤。因此，在我们在创建工程时，Android  Studio自动帮我们生成了一些通用构建规则，很多时候我们甚至完全不用修改这些规则就能完成我们app的构建。

<!--more-->

有些时候，我们会有一些个性化的构建需求，比如我们引入了第三方库，或者我们想要在通用构建过程中做一些其他的事情，这时我们就要自己在系统默认构建规则上做一些修改。这时候我们就要自己向Gradle”下命令“了，这时候我们就需要用Gradle能听懂的话了，也就是Groovy。Groovy是一种基于JVM的动态语言。

 

我们在开头处提到“Gradle是一种构建工具”。实际上，当我们想要更灵活的构建过程时，Gradle就成为了一个编程框架——我们可以通过编程让构建过程按我们的意愿进行。也就是说，当我们把Gradle作为构建工具使用时，我们只需要掌握它的配置脚本的基本写法就OK了；而当我们需要对构建流程进行高度定制时，就务必要掌握Groovy等相关知识了。

### 特性

下面是一些 Gradle 特性的列表。

**基于声明的构建和基于约定的构建**

Gradle 的核心在于基于 Groovy 的丰富而可扩展的域描述语言(DSL)。 Groovy  通过声明性的语言元素将基于声明的构建推向下层，你可以按你想要的方式进行组合。 这些元素同样也为支持 Java， Groovy，OSGi，Web 和  Scala 项目提供了基于约定的构建。 并且，这种声明性的语言是可以扩展的。你可以添加新的或增强现有的语言元素。  因此，它提供了简明、可维护和易理解的构建。 

**为以依赖为基础的编程方式提供语言支持**

声明性语言优点在于通用任务图，你可以将其充分利用在构建中. 它提供了最大限度的灵活性，以让 Gradle 适应你的特殊需求。

**构建结构化**

Gradle 的灵活和丰富性最终能够支持在你的构建中应用通用的设计模式。  例如，它可以很容易地将你的构建拆分为多个可重用的模块，最后再进行组装，但不要强制地进行模块的拆分。  不要把原本在一起的东西强行分开（比如在你的项目结构里），从而避免让你的构建变成一场噩梦。  最后，你可以创建一个结构良好，易于维护，易于理解的构建。

**深度 API**

Gradle 允许你在构建执行的整个生命周期，对它的核心配置及执行行为进行监视并自定义。

**Gradle 的扩展**

Gradle 有非常良好的扩展性。 从简单的单项目构建，到庞大的多项目构建，它都能显著地提升你的效率。 这才是真正的结构化构建。通过最先进的增量构建功能，它可以解决许多大型企业所面临的性能瓶颈问题。

**多项目构建**

Gradle 对多项目构建的支持非常出色。项目依赖是首先需要考虑的问题。 我们允许你在多项目构建当中对项目依赖关系进行建模，因为它们才是你真正的问题域。 Gradle 遵守你的布局。

Gradle 提供了局部构建的功能。 如果你在构建一个单独的子项目，Gradle 也会帮你构建它所依赖的所有子项目。 你也可以选择重新构建依赖于特定子项目的子项目。 这种增量构建将使得在大型构建任务中省下大量时间。

**多种方式管理依赖**

不同的团队喜欢用不同的方式来管理他们的外部依赖。 从 Maven 和 Ivy 的远程仓库的传递依赖管理，到本地文件系统的 jar 包或目录，Gradle 对所有的管理策略都提供了方便的支持。

**Gradle 是第一个构建集成工具**

Ant tasks 是最重要的。而更有趣的是，Ant projects 也是最重要的。 Gradle 对任意的 Ant  项目提供了深度导入，并在运行时将 Ant 目标(target)转换为原生的 Gradle 任务(task)。 你可以从 Gradle  上依赖它们(Ant targets)，增强它们，甚至在你的 build.xml 上定义对 Gradle tasks 的依赖。Gradle  为属性、路径等等提供了同样的整合。

Gradle 完全支持用于发布或检索依赖的 Maven 或 Ivy 仓库。 Gradle 同样提供了一个转换器，用于将一个 Maven pom.xml 文件转换为一个 Gradle 脚本。Maven 项目的运行时导入的功能将很快会有。

#### 易于移植

Gradle 能适应你已有的任何结构。因此，你总可以在你构建项目的同一个分支当中开发你的 Gradle 构建脚本，并且它们能够并行进行。 我们通常建议编写测试，以保证生成的文件是一样的。 这种移植方式会尽可能的可靠和减少破坏性。这也是重构的最佳做法。

**Groovy**

Gradle 的构建脚本是采用 Groovy 写的，而不是用 XML。  但与其他方法不同，它并不只是展示了由一种动态语言编写的原始脚本的强大。 那样将导致维护构建变得很困难。 Gradle  的整体设计是面向被作为一门语言，而不是一个僵化的框架。 并且 Groovy 是我们允许你通过抽象的 Gradle 描述你个人的 story  的黏合剂。 Gradle 提供了一些标准通用的 story。这是我们相比其他声明性构建系统的主要特点。 我们的 Groovy  支持也不是简单的糖衣层，整个 Gradle 的 API 都是完全 groovy 化的。只有通过 Groovy才能去运用它并对它提高效率。

**The Gradle wrapper**

Gradle Wrapper 允许你在没有安装 Gradle 的机器上执行 Gradle 构建。  这一点是非常有用的。比如，对一些持续集成服务来说。 它对一个开源项目保持低门槛构建也是非常有用的。 Wrapper  对企业来说也很有用，它使得对客户端计算机零配置。 它强制使用指定的版本，以减少兼容支持问题。

**自由和开源**

Gradle 是一个开源项目，并遵循 ASL 许可。

### 安装

环境：Ubuntu 18.04 +JDK 11.0.3

Gradle 需要 1.5 或更高版本的 JDK。Gradle 自带了 Groovy 库，所以不需要安装 Groovy。Gradle 会忽略已经安装的 Groovy。Gradle 会使用 ptah中的 JDK(可以使用 java -version 检查)。当然，你可以配置 JAVA_HOME 环境变量来指向 JDK 的安装目录。 

从 Gralde 官方网站下载 [Gradle 的最新发行包](http://services.gradle.org/distributions/)。Gradle 发行包是一个 ZIP 文件。

```
$ mkdir /opt/gradle
$ unzip -d /opt/gradle gradle-5.4.1-bin.zip
$ ls /opt/gradle/gradle-5.4.1
LICENSE  NOTICE  bin  getting-started.html  init.d  lib  media
```

**配置环境变量**

运行 gradle 必须将 GRADLE_HOME/bin 加入到你的 PATH 环境变量中。

```
 $ export PATH=$PATH:/opt/gradle/gradle-5.4.1/bin
```

运行如下命令来检查是否安装成功.该命令会显示当前的 JVM 版本和 Gradle 版本。

```
$ gradle -v

------------------------------------------------------------
Gradle 5.4
------------------------------------------------------------

Build time:   2019-04-16 02:44:16 UTC
Revision:     a4f3f91a30d4e36d82cc7592c4a0726df52aba0d

Kotlin:       1.3.21
Groovy:       2.5.4
Ant:          Apache Ant(TM) version 1.9.13 compiled on July 10 2018
JVM:          11.0.3 (Oracle Corporation 11.0.3+12-LTS)
OS:           Linux 4.18.0-17-generic amd64

```

或者从包装Gradle中升级,也可以不指定版本，会在使用时自动下载gradle.properties中指定的版本。

```
$ ./gradlew wrapper --gradle-version=5.4.1 --distribution-type=<bin/all/src>
```

**Gradle使用Maven的仓库**

如果没有安装Maven不要配置，配置`GRADLE_USER_HOME=~/.m2/repository/`

Gradle 目录结构(和Go类似)

```
├── com.alibaba
│   └── fastjson
│       └── 1.2.47
├── com.fasterxml.jackson.core
│   └── jackson-annotations
│       └── 2.9.5
├── com.github.kuangcp
│   └── JavaToolKit
│       └── 0.0.4-SNAPSHOT
├── com.google.code.gson
│   └── gson
│       └── 2.8.5
└── org.projectlombok
    └── lombok
        └── 1.18.2
```

Maven目录结构：

```
└── org
    └── projectlombok
        └── lombok
            └── 1.18.2
```



Gradle 构件的目录结构就和maven不同,只是去那个目录下去找找有没有对应的构件, 有就复制过来(~/.gradle/caches/modules-2/files-2.1/), 并建立一个新的目录结构,并不打算复用

所以, 如果你是Maven和Gradle 混着用的话, 两个本地仓库是互相独立和冗余的。

后面讲配置文件时会讲到Gradle使用Maven的仓库时配置文件的配置。

**JVM 参数配置**

Gradle 运行时的 JVM 参数可以通过 GRADLE_OPTS 或 JAVA_OPTS 来设置.这些参数将会同时生效。  JAVA_OPTS 设置的参数将会同其它 JAVA 应用共享，一个典型的例子是可以在 JAVA_OPTS 中设置代理和 GRADLE_OPTS  设置内存参数。同时这些参数也可以在 gradle 或者 gradlew 脚本文件的开头进行设置。



### Groovy语法入门

为什么我们学习gradle需要学习groovy? groovy是google推出的一款基于java的脚本语言。只要简单几句代码就可以完成java程序。虽然groovy看起来是脚本编程语言，但是究其更本也是运行在jvm上面的java语言，知识groovy对java进行了二层的封装。因为这个语言的灵活性，gradle选择了groovy。

#### helloWorld

让我们使用最简单的helloWorld代码来入门。首先要确保gradle安装正确，安装正确后，我们新建一个文件夹，在这个文件夹内部新建一个build.gradle文件。

```groovy
 println("Hello World!");
```

接着我们在命令行执行 gradle 后就可以看到相应的结果了

#### 基础语法

```
def a = 1
def b = "abc"
def int a
```

上面是gradle如何定义一个变量的写法。注意gradle没有要求每行必须以；结尾，一行就是一句代码

```
def oneMeathod(arg1){

}

int twoMeathod(arg1,arg2){
     arg1+arg2
}
```

上面是定义方法的写法，如果是无返回的方法。需使用def定义。如果指定了相应的返回类型的话，可不使用def关键字。

注意如果有返回值的话，需在最后一行将需要返回的值返回。

#### 字符串

```
def a ='huangkai'
def b = "i am $a"
def lines = ''' abc


       abc

       end '''

```

字符串有三种格式

- '' 代表完全按照符号内部的内容
- "" 如果内部有$变量值的话，先取变量值
- ''' '''代表文本支持换行

#### 容器类型

在Java里提供了多种的容器类型，常见的有List Map Set。他们分别适合不同场景。为不同需求提供解决能力。在grrovy内部也提供了对应的容器类型。

以下内容不用死记硬背，学会[查询即可](http://docs.groovy-lang.org/latest/html/groovy-jdk/)

**List**

与Java数组类型相似，与java不一样的，Java需要类型相同的对象。可以理解为定义了一个ArrayList

```
def a = ["a",5,6]
```

上面是定义的方法。

可以在上面的网站查询到list提供的方法

```
def a = [5,'string',true]

println(a.getAt(2));
```

**Map**

与java中的map类似，map提供的键值对的能力。

```
 def map = ["a":100,"b":200]
```

定义map的方法，key:vaule key必须为字符串 value可以为任何值。 多个值之间使用,分开

简单的使用map

```
 def map = ["a":100,"b":200]

 println(map.getAt("b"));
```

#### 闭包（Closure）

闭包是groovy里面的特殊语法，我们在很多文件都可以看到闭包的身影。闭包是类似回调的一个代码块。因为groovy是脚本语言，所以有很多约定的写法，下面我们来看看闭包是如何定义与书写的。

```
 def aClosure ={
           println("helloWorld");
      }

```

带参数的闭包

```
def xxx = {a , b ->

   println(a);
   println(b);

 }
```

-> 符号前代表输入的参数，可以不使用类型

带默认参数的闭包

```
def xxx ={
    println(it);
}
```

每个闭包在定义的时候都带了一个默认的参数，这个参数的名称叫it。我们可以直接获取他不需要定义。以上闭包的几种定义的方式。

闭包的使用通常有两种：

- 直接用，我们把闭包当做一个函数使用。

  ```
   def abc = {time,name ->
     println(time);
     println(name);
   }
  abc.call(100,"huangkai");
   def xxx ={
      println(it);
   }
  xxx.call(10086)
  ```

  

- 作为一个参数使用，我们把闭包作为一个回调使用。

  ```
  List    each(Closure closure)
  //list定义了一个方法，这个方法是遍历list的方法，方法接收一个参数，参数的类型就是一个闭包。
  def a = [5,'string',true]
  
  def b ={
   println(it);
   }
  
  a.each(b);
  ```

### 构建块

每个Gradle构建都包括三个基本构建块：project、task、property，每个构建包至少一个project，进而又包含一个或多个task，project和task暴露的属性可以用来控制构建

![](https://i.loli.net/2019/05/11/5cd65627ac139.png)

#### project

一个project代表一个正在构建的组件，或者一个想要完成的目标，Gradle的build.gradle相当于Maven的pom.xml，每个Gradle构建脚本至少定义一个project，当构建进程启动后，Gradle基于build.gradle中的配置实例化org.gradle.qpi.Project类，并且能够通过project变量使其隐式可用。

![project接口](https://i.loli.net/2019/05/11/5cd656f344922.png)

一个project可以创建新的task，添加依赖关系和配置,并应用插件和其他的构建脚本。他的许多属性可以通过getter的setter方法访问。

当访问属性和方法时，不需要使用project变量，默认指向project示例：

```
setDescription("MyProject")
println "Description of project $name :" + description
```

#### task

task任务包括一些重要功能：task action 、task dependency。任务动作定义一个当任务执行时最小的工作单元。task的接口如下：

![](https://i.loli.net/2019/05/11/5cd67b9e24356.png)

#### properties

每个Project和Task示例都提供可以通过getter和setter方法访问属性。

**扩展属性**：Gradle很多领域模型类提供了特别的属性支持，以键值对的形式储存，为了添加属性，需要使用ext命名空间

```groovy
project.ext.myProp='myValue' //初始声明扩展属性时需要使用ext命名空间
ext {
	someOtherProp = 123
}
assert myProp == 'myValue'
println project.someOtherProp
ext.someOtherProp = 567  //ext命名空间访问属性可选
```

**Gradle属性**：通过在gradle.properties文件中声明直接添加到项目中，

**其他方式**：项目属性通过-P命令行提供、系统属性通过-D命令行提供、环境变量提供

### 使用

 `<<` 在gradle 在5.1 之后废弃了   

#### 创建一个简单的Gradle项目

首先创建一个目录，即项目的所在目录

```
>mkdir basic-demo
> cd basic-demo
```

然后，我们可以使用Gradle `init`命令来生成一个简单的项目。我们将探索所有产生的项目文件，以确切知道发生了什么。

> `gradle init`命令可以生成[不同类型的项目](https://docs.gradle.org/4.6/userguide/build_init_plugin.html?_ga=2.56416747.1372490660.1523501630-709977496.1497512105#sec:build_init_types),甚至可以知道如何将简单`pom.xml`文件转换为Gradle。

```
> gradle init 
Select type of project to generate:
  1: basic
  2: cpp-application
  3: cpp-library
  4: groovy-application
  5: groovy-library
  6: java-application
  7: java-library
  8: kotlin-application
  9: kotlin-library
  10: scala-library
Enter selection (default: basic) [1..10] 1
Select build script DSL:
  1: groovy
  2: kotlin
Enter selection (default: groovy) [1..2] 1
Project name (default: basic-demo): 

BUILD SUCCESSFUL in 7s
2 actionable tasks: 2 executed

```

下面就是Gradle生成的文件目录：

```
> tree 
.
├── build.gradle #项目配置脚本，用于配置当前项目中的任务
├── gradle
│   └── wrapper 
│       ├── gradle-wrapper.jar #Gradle Wrappe可执行JAR
│       └── gradle-wrapper.properties #Gradle Wrapper配置属性
├── gradlew #用于基于Unix系统的Gradle Wrapper脚本
├── gradlew.bat #用于基于Windows的Gradle Wrapper脚本
└── settings.gradle #设置配置脚本，用于配置哪些项目参与构建

2 directories, 6 files
```

gradle版本会不断更新，每个人使用的版本可能会不同，而gradlew（wrapper）可以算是gradle的一层包装。

让我们使用相同版本的gradle进行构建，我们在gradle -> wrapper中可以看到gradle-wrapper.properties文件。

打开，可以看到配置的gradle版本的信息：

```
distributionBase=GRADLE_USER_HOME #GRADLE_USER_HOME表示用户目录
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-5.4-bin.zip #指定某个版本gradle下载地址
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

我们使用gradlew命令的时候，会根据这个文件来使用对应的gradle进行构建，没有则会下载

> gradle init可以生成各种不同类型的项目，甚至知道如何将简单的pom.xml文件转换成gradle。

#### 创建任务

Gradle提供了通过Groovy或Kotlin的DSL来创建和配置任务的的API。每个Project有一系列执行基本操作的Task。

Gradle附带一个用于配置项目的任务库。例如，有个叫做`Copy`的核心类，它将文件从一个位置复制到另一个位置。`Copy`任务非常的有用。但是，在这里，我们再一次只是简单的使用它。

执行以下步骤：

1. 创建名为`src`的文件夹

2. 在文件夹`src`中添加`myfile.txt`。内容是任意的(甚至可以为空)，但为了方便起见，添加一行内容`Hello, World!`。

3. 在主构建文件`build.gradle`中定一个名为`copy`的`Copy`类型任务。它将`src`目录复制到一个名为`dest`的新目录中。(你不必创建`dest`文件夹，任务将替你创建)

   ```
   task copy(type: Copy, group: "Custom", description: "Copies sources to the dest directory") {
       from "src"
       into "dest"
   }
   ```

在这里，group和description是可以任意设置的。你甚至可以忽略它们，但是，如果这么做，`tasks`报告中，也会忽略它们，过会我们会用到它们。

现在执行新创建的`./gradlew copy`任务，通过检查在`dest`文件夹中有名为`myfile.txt`的文件，并且里面的内容与`src`中的`myfile.txt`内容一致来检查该任务是否按照预期执行。

#### 应用插件

Gradle包含一系列插件，[ the Gradle plugin portal](https://plugins.gradle.org)中提供了非常多的插件。这个发行版中包含的一个名为`base`的插件。与核心类`Zip`一起使用，可以使用配置的名称和位置创建项目的zip压缩文件。

使用`plugins`脚本将`base`插件添加到`build.gradle`中。确保在文件顶部添加`plugins {}`代码块。

```
plugins {
    id "base"
}

... rest of the build file ...
```

现在添加一个创建`src`文件夹的zip压缩文件的任务。

```
task zip(type: Zip, group: "Archive", description: "Archives sources in a zip file") {
    from "src"
}
```

`base`插件与设置一起完成任务，在`build/distributions`文件夹下创建名为`basic-demo-1.0.zip`的压缩文件。

在这种情况下，执行任务`zip`并且查看生成的压缩文件是您所期望的。

```
 ./gradlew zip
```

生成的文件位于：

```
.
└── build
   └── distributions
       └── basic-demo.zip
```

#### 探索和调试构建

**查看可用的`tasks`**

`tasks`命令列出你可调用的Gradle任务，包括base插件添加的任务以及刚刚添加的自定义任务。

```
>./gradlew tasks
```

**分析和调试你的构建**

Gradle还为您的构建提供了一个丰富的，基于Web的视图，称为构建审视。

通过使用`--scan`命令选项或通过显示声明将构建审视插件应用到项目中，您可以免费在链接scans.gradle.com上创建构建审视。构建审视发布到scans.gradle.com 并将这些数据上传到Gradle的服务器。要将数据保存在您自己的服务器上，请查看[Gradle Enterprise](https://gradle.com/enterprise?_ga=2.143345461.1063945172.1523501794-431069063.1523242361).

在执行任务时，通过添加 `--scan`命令选项生成构建审视。

```
>./gradlew zip --scan
```

在[Build Scan Plugin用户手册](https://docs.gradle.com/build-scan-plugin/?_ga=2.243588101.1414919168.1523501302-1348449306.1523242263)中详细了解如何配置和使用构建审视。

#### 发现可用属性

properties命令告诉您项目的属性。

```
> ./gradlew properties
```

默认情况下，项目的名称与文件夹的名称匹配。 您还可以指定组和版本属性，但目前它们采用默认值，如描述。

buildFile属性是构建脚本的完全限定路径名，该脚本位于projectDir中 - 默认情况下。

您可以更改许多属性。 例如，您可以尝试将以下行添加到构建脚本文件中，然后重新执行gradle属性。

```
description = "A trivial Gradle build"
version = "1.0"
```

