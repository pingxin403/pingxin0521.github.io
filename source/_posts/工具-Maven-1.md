---
title: Maven入门和配置
date: 2019-03-26 10:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
### 前言
>工欲善其事必先利其器             -- 孔子

Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的项目管理工具软件。

<!--more-->


![maven(1).jpeg](https://i.loli.net/2019/04/02/5ca370d950d86.jpeg)

Maven 除了以程序构建能力为特色之外，还提供高级项目管理工具。由于 Maven 的缺省构建规则有较高的可重用性，所以常常用两三行 Maven 构建脚本就可以构建简单的项目。由于 Maven 的面向项目的方法，许多 Apache Jakarta 项目发文时使用 Maven，而且公司项目采用 Maven 的比例在持续增长。

Maven是一个项目管理工具，它包含了一个项目对象模型 (Project Object Model)，一组标准集合，一个项目生命周期(Project Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。当你使用Maven的时候，你用一个明确定义的项目对象模型来描述你的项目，然后Maven可以应用横切的逻辑，这些逻辑来自一组共享的（或者自定义的）插件。
在多个开发团队环境时，Maven可以设置按标准在非常短的时间里完成配置工作。由于大部分项目的设置都很简单，并且可重复使用，Maven让开发人员的工作更轻松，同时创建报表，检查，构建和测试自动化设置。

Maven提供了开发人员的方式来管理：** Builds（构建）、 Documentation（文档） 、Reporting（报告）、 Dependencies（依赖） 、test（测试）、Releases（发布）、 部署**等。

Maven 有一个生命周期，当你运行 mvn install 的时候被调用。这条命令告诉 Maven 执行一系列的有序的步骤，直到到达你指定的生命周期。遍历生命周期旅途中的一个影响就是，Maven 运行了许多默认的插件目标，这些目标完成了像编译和创建一个 JAR 文件这样的工作。

此外，Maven能够很方便的帮你管理项目报告，生成站点，管理JAR文件，等等。

概括地说，Maven简化和标准化项目建设过程。处理编译，分配，文档，团队协作和其他任务的无缝连接。 Maven增加可重用性并负责建立相关的任务。最大化地消除构建的**重复**，并且依照约定优于配置的规则免去额外的学习成本。

### 作用
**作用一：**

个人理解maven主要是用来解决导入java类依赖的jar,编译java项目主要问题。(最早手动导入jar，使用Ant之类的编译java项目)
以pom.xml文件中dependency属性管理依赖的jar包，而jar包包含class文件和一些必要的资源文件。

当然它可以构建项目，管理依赖，生成一些简单的单元测试报告，像现在公司的持续集成都广泛的使用maven，当你接触一些项目以后你就会有更深的体会。

**作用二：**

比如之前项目导入jar。是通过copy方式导入项目中，而且还会存在jar之间的依赖和冲突。而maven解决了这些问题，只是网速不好的时候有点麻烦。只需要下载-bin.zip就可以了。

**作用三：**

jar 包管理，防止jar之间依赖起冲突 。小组之间建立个私服务，大家都用通用的maven配置文件，不用自己手动去下载jar ，pom.xml文件会自动管理下载好的jar包。

**作用四：**

Maven是基于项目对象模型，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。Maven能够很方便的帮你管理项目报告，生成站点，管理jar文件，等等。例如：项目开发中第三方jar引用的问题，开发过程中合作成员引用的jar版本可能不同，还有可能重复引用相同jar的不同版本，使用maven关联jar就可以配置引用jar的版本，避免冲突。

### 安装
>前提是电脑上配置有java开发环境

从[官网](http://maven.apache.org/)下载最新的包，linux上下载 `Binary tar.gz archive`包，Windows上下载 `Binary zip archive`包。

Windows下载完成后，解压到安装位置，配置PATH环境变量。
linux上下载完成后，解压到`/opt/`下，设置PATH环境变量，如下：
```(shell)
$ tar zxf  <包名>.tar.gz  /opt/xauto/maven
$ vim /etc/profile
#在文件最下面添加以下配置
export M2_HOME=/opt/xauto/maven
export PATH=${PATH}:/${MAVEN_HOME}/bin
```
测试是否可以使用
```（shell）
$ mvn --version
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /opt/xauto/maven
Java version: 11.0.3, vendor: Oracle Corporation, runtime: /opt/java/java-11
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "4.18.0-17-generic", arch: "amd64", family: "unix"
```
我们必须知道的站点：[Maven公共库查询](https://mvnrepository.com/)。我们可以通过这个站点查询需要用到的库文件和依赖文件，非常重要！！
此外，我们还需要了解Maven的远程仓库站点，我们的Maven项目会从该仓库下载依赖的库文件，默认远程仓库为`https://repo.maven.apache.org/`，下面我们会讲到设置远程仓库，以此来提高下载速率。

### 常用概念
下面我们将学习一下Maven中常用的概念。
#### Maven生命周期
我们在开发项目的时候，不断地在编译、测试、打包、部署等过程，maven的生命周期就是对所有构建过程抽象与统一，生命周期包含项目的清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成等几乎所有的过程。

Maven有三套相互独立的生命周期，请注意这里说的是“三套”，而且“相互独立”，初学者容易将Maven的生命周期看成一个整体，其实不然。这三套生命周期分别是：
1.  CleanLifecycle 在进行真正的构建之前进行一些清理工作。
2.   DefaultLifecycle 构建的核心部分，编译，测试，打包，部署等等。
3.  SiteLifecycle 生成项目报告，站点，发布站点。
![Maven生命周期(1).png](https://i.loli.net/2019/04/04/5ca5ee46037ac.png)

再次强调一下它们是相互独立的，可以仅仅调用clean来清理工作目录，仅仅调用site来生成站点。当然也可以直接运行 “mvn clean install site” 运行所有这三套生命周期。

每套生命周期都由一组阶段(Phase)组成，我们平时在命令行输入的命令总会对应于一个特定的阶段。maven中所有的执行动作(goal)都需要指明自己在这个过程中的执行位置，然后maven执行的时候，就依照过程的发展依次调用这些goal进行各种处理。这个也是maven的一个基本调度机制。

每套生命周期还可以细分成多个阶段。

**Clean生命周期**

Clean生命周期一共包含了三个阶段：
![Clean生命周期.png](https://i.loli.net/2019/04/04/5ca5f07deb91d.png)


执行一些需要在clean之后立刻完成的工作

命令“mvn clean”中的就是代表执行上面的clean阶段，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，“mvn clean” 等同于 “mvn pre-clean clean” ，如果我们运行“mvn post-clean” ，那么 “pre-clean”，“clean” 都会被运行。这是Maven很重要的一个规则，可以大大简化命令行的输入。



**Default生命周期**

Maven最重要就是的Default生命周期，也称构建生命周期，绝大部分工作都发生在这个生命周期中，每个阶段的名称与功能如下:

![Default生命周期.png](https://i.loli.net/2019/04/04/5ca5f07e2515d.png)


可见，构建生命周期被细分成了22个阶段，但是我们没必要对每个阶段都了如指掌，经常关联使用的只有process-test-resources、test、package、install、deploy等几个阶段而已。

一般来说，位置稍后的过程都会依赖于之前的过程。这也就是为什么我们运行“mvn install” 的时候，代码会被编译，测试，打包。当然，maven同样提供了配置文件，可以依照用户要求，跳过某些阶段。比如有时候希望跳过测试阶段而直接install，因为单元测试如果有任何一条没通过，maven就会终止后续的工作。


**Site生命周期**

![Site生命周期.png](https://i.loli.net/2019/04/04/5ca5f07dd5c01.png)

将生成的站点文档部署到特定的服务器上

这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这是Maven相当强大的功能。

#### Maven 关键词
在使用Maven做开发中常用到的关键词。
  -  Project:
      -  任何你想 build 的事物，Maven都会把它们当作是一个 Project。
      - 这些 Project 被定义为 POM(Project Object Model)。
      - 一个 Project 可以依赖其他的project，一个 project 也可以有多个子project组成。
  - POM：
       - POM(pom.xml) 是 Maven 的核心文件，它是指示 Maven 如何工作的元数据文件，类似 ant 的 build.xml 文件。
       - pom.xml 文件应该位于每个 Project 的根目录。
 - GroupId:
       - 顾名思义，这个应该是公司名或组织名。
  - ArtifactId：
       - 构建出来的文件名，一般来说或，这个也是project名。
  - Packaging：
       -  项目打包的类型，可以是将jar、war、rar、ear、pom，默认是jar。
  - Version：
       -  项目的版本，项目的唯一标识由 groupId+artifactId+packaging+versionz 组成。
  - Dependency:
       - 为了能够 build 或运行，一个典型的java project会依赖其他的包，在Maven中，这些被依赖的包就被称为 dependency。
  - Plug-in：
      -  Maven是有插件组织的，它的每一个功能都是由插件提供的，主要的插件是由 java 来写的，但是他也支持 beanshell 和 ant 脚本编写的插件。
  - Repository：
       - 仓库用来存放artifact的，可以是本地仓库，也可以是远程仓库，Maven是由一个默认的仓库
  - Snapshot：
       - 工程中可以（也应该）有这样一个特殊的版本：这个版本可以告诉Maven，该工程正在处于开发阶段，会经常更新（但还为发布）。当其他工程依赖此类型的artifact时，Maven会在仓库中寻找该artifact的最新版本，并自动下载、使用该最新版本。


#### Maven常用命令

maven的命令格式如下：

`mvn [plugin-name]:[goal-name]`

**mvn常用参数**

```(shell)
mvn -e 显示详细错误
mvn -U 强制更新snapshot类型的插件或依赖库（否则maven一天只会更新一次snapshot依赖）
mvn -o 运行offline模式，不联网更新依赖
mvn -N仅在当前项目模块执行命令，关闭reactor
mvn -pl module_name在指定模块上执行命令
mvn -ff 在递归执行命令过程中，一旦发生错误就直接退出
mvn -Dxxx=yyy指定java全局属性
mvn -Pxxx引用profile xxx
```

下面是不同生命周期使用的不同命令。
![Maven命令.gif](https://i.loli.net/2019/04/03/5ca463da35c8a.gif)
常用命令的功能。

|命令 | 作用|
| --- | ----|
|mvn archetype:generate |	反向生成 maven 项目的骨架|
|mvn archetype：create|创建项目|
|mvn compile 	|编译源代码|
|mvn test-compile 	|编译测试代码|
|mvn test |	运行应用程序中的单元测试|
|mvn install| 	在本地Respository中安装jar，例：installing D:\xxx\xx.jar to D:\xx\xxxx|
|mvn eclipse:eclipse |	生成eclipse项目文件|
|mvn jetty:run |	启动jetty服务|
|mvn clean |	清除项目目录中的生成结果|
|mvn site |	生成项目相关信息的网站|
|mvn package |	根据项目生成的jar|
|mvn help:system|打印出所有的java系统属性和环境变量 |
|mvn eclipse:eclipse| 生成eclipse项目|
|mvn idea:idea|生成idea项目|
|mvn jar:jar |  只打jar包|
|mvn deploy |上传到私服|
|mvn clean install-U |强制检查更新，由于快照版本的更新策略(一天更新几次、隔段时间更新一次)存在，如果想强制更新就会用到此命令|
|mvn source:jar 或 mvn source:jar-no-fork|源码打包|

**mvn compile与mvn install、mvn deploy的区别**

   mvn compile，编译类文件
   mvn install，包含mvn compile，mvn package，然后上传到本地仓库
   mvn deploy,包含mvn install,然后，上传到私服

> 后面我们会专门来讲一下Maven的命令。

示例：
**help:describe**

maven有各种插件，插件又有各种目标。我们不可能记得每个插件命令。maven提供了查询各类插件参数的命令：`mvn help:describe`。

例如：`mvn help:describe -Dplugin=help`

代表查询help 插件的命令规范，然后maven就会告诉你该命令有几个goal，各种参数的的意义以及配置方法




上面就是Maven中常用的概念，包括生命周期、命令和关键字，我们可以新建一个项目来测试一下。
``` （shell）
$ mvn archetype:generate
# 会要求输入对应的模板编号，GroupId、artifactId和版本号等信息。
...
Define value for property 'groupId': test
Define value for property 'artifactId': test01
Define value for property 'version' 1.0-SNAPSHOT: :
Define value for property 'package' test: : com.hyp.test
Confirm properties configuration:
groupId: test
artifactId: test01
version: 1.0-SNAPSHOT
package: com.hyp.test
 Y: : y
...

$ ls
test01

$ tree test01/
test01/
├── pom.xml
└── src
    └── main
        ├── resources
        └── webapp
            ├── index.jsp
            └── WEB-INF
                └── web.xml

5 directories, 3 files

```

好了，现在我们创建了一个项目，上面的概念和命令以后会经常用到。

### 配置

在了解Maven的配置之前，首先先了解一下Maven的安装目录结构,具体如下：

```（shell）
$ tree /opt/apache/maven/
/opt/apache/maven/
├── bin #Maven运行的搅拌
│   ├── m2.conf #classworlds配置文件
│   ├── mvn #unix平台命令脚本
│   ├── mvn.cmd #Windows平台的脚本
│   ├── mvnDebug #unix平台debug模式启动命令脚本
│   └── mvnDebug.cmd #mvnDebug命令Windows平台的脚本
├── boot #包含类加载框架，使用该框架加载自己的类库
│   └── plexus-classworlds-2.5.2.jar
├── conf #全局配置文件目录
│   ├── logging #日志文件配置
│   │   └── simplelogger.properties
│   ├── settings.xml #全局配置，推荐复制到~/.m2/目录下，在用户范围内控制
│   └── toolchains.xml
├── lib #Maven运行时需要类库
├── LICENSE #Maven的Apache证书许可
├── NOTICE #记录Maven包含的第三方软件
└── README.txt #Maven的简短介绍，包括安装需求和安装的简要指令。

14 directories, 76 files
```

#### 仓库

在Maven中，任何一个依赖、插件或者项目构建的输出，都可以称之为构件。

Maven在某个统一的位置存储所有项目的共享的构件，这个统一的位置，我们就称之为仓库。（仓库就是存放依赖和插件的地方）
任何的构件都有唯一的坐标，Maven根据这个坐标定义了构件在仓库中的唯一存储路径，

**解读Maven在仓库中的存储路径：**

1. 基于groupId准备路径，将句点分隔符转成路径分隔符，就是将  "."  转换成 "/" ; example： `org.testng --->org/testng`
2. 基于artifactId准备路径，将artifactId连接到后面：`org/testng/testng`
3. 使用version准备路径，将version连接到后面：`org/testng/testng/5.8`
4. 将artifactId于version以分隔符连字号连接到后面：`org/testng/testng/5.8/tesng-5.8`
5. 判断如果构件有classifier，就要在 第4项 后增加 分隔符连字号 再加上 classifier，`org/testng/testng/5.8/tesng-5.8-jdk5`
6. 检查构件的extension，如果extension存在，则加上句点分隔符和extension，而extension是由packing决定的，`org/testng/testng/5.8/tesng-5.8-jdk5.jar`

到这里我们就明白了Maven 对于构件存储的细节。

**Maven是根据怎样的规则从仓库解析并使用依赖构件的呢？**

当本地仓库没有依赖构件的时候，Maven会自动从远程仓库下载。当依赖版本为快照版本的时候，Maven会自动找到最新的的快照。这背后的依赖解析机制可以概括如下：

1. 当依赖的范围是system的时候，Maven直接从本地文件系统解析构件。
2. 根据依赖坐标计算仓库路径后，尝试直接从本地仓库寻找构件，如果发现相应构件，则解析成功。
3. 在本地仓库不存在相应构件的情况下，如果依赖的版本是显式的发布版本构件，如：1.2，2.1等，则遍历所有的远程仓库，发现后，下载并解析使用。
4. 如果依赖的版本是RELEASE或者LATEST，则基于更新策略读取所有远程仓库的元数据groupId/artifactId/mavenmetadata.xml，将其与本地仓库的对应元数据合并后，计算出RELEASE或者LATEST真实的值，然后基于这个真实的值检查本地和远程仓库，如步骤1和3。
5. 如果依赖的版本是SNAPSHOT，则基于更新策略读取所有远程仓库的元数据groupId/artifactId/version/mavenmetadata.xml，将其与本地仓库的对应元数据合并后，得到最新快照版本的值，然后基于该值检查本地仓库，或者从远程仓库下载。
6. 如果最后解析得到的构件版本是时间戳格式的快照，如：1.4-20091104.121450-121,则复制其时间戳格式的文件到非时间戳格式，如：SNAPSHOT，并使用该非时间戳格式的构件。

当依赖的版本不明晰的时候，如:RELEASE,LATEST和SNAPSHOT，Maven就需要基于更新远程仓库的更新策略来检查更新。在前面的仓库配置blog中，有一些配置与此有关；首先是<releases\><enabled\>和<snapshots\><enabled\>，只有仓库开启了对于发布版本的支持时，才能访问该仓库的发布版本构件信息，对于快照版本也是同理；其次要注意的是<releases\>和<snapshots\>的子元素<updatePolicy\>，该元素配置了检查更新的频率。最后，用户还可以从命令行加入参数-U,强制检查更新，使用参数后，Maven就会忽略<updatePolicy\>的配置。

当Maven检查完更新策略，并决定检查依赖更新的时候，就需要检查仓库元数据maven-metadata.xml。

#### 更改本地仓库位置

针对不同的用户，Maven会在其家目录下创建.m2目录，推荐将用户的Maven配置复制到该目录下。`~/.m2/`目录除了包含配置文件`setting.xml`，默认将本地仓库设在`~/.m2/repository/`下，我们所使用的库文件都会从远程仓库下载到该目录下，并依据groupID、包名、版本号来分类存放。如果想设置其他位置存放仓库，可以配置`setting.xml`文件（M2_HOME/conf/setting.xml使用全局，~/.m2/setting.xml适用该用户）：

```（xml）
<!--在配置文件中找到注释的localRepository节点，在下面添加如下-->
<localRepository>/path/to/youRepository</localRepository>
```

这样，Maven会将从远程仓库下载的依赖包存储到你指定的目录下。

Maven一个很突出的功能就是jar包管理，一旦工程需要依赖哪些jar包，只需要在Maven的pom.xml配置一下，该jar包就会自动引入工程目录。初次听来会觉得很神奇，下面我们来探究一下它的实现原理。

首先，这些jar包肯定不是没爹没娘的孩子，它们有来处，也有去处。集中存储这些jar包（还有插件等）的地方被称之为仓库（Repository）。

不管这些jar包从哪里来的，必须存储在自己的电脑里之后，你的工程才能引用它们。类似于电脑里有个客栈，专门款待这些远道而来的客人，这个客栈就叫做本地仓库。

比如，工程中需要依赖spring-core这个jar包，在`pom.xml`中声明之后，maven会首先在本地仓库中找，如果找到了很好办，自动引入工程的依赖lib库即可。可是，万一找不到呢？实际上这种情况经常发生，尤其初次使用maven的时候，本地仓库肯定是空无一物的，这时候就要靠maven大展神通，去远程仓库去下载。

Maven的父POM中也是有内置一个插件仓库的，我现在用的电脑安装的是Maven 3.6.0版本，我们可以找到这个文件：`${M2_HOME}/lib/maven-model-builder-3.6.0.jar`，打开该文件，能找到超级父POM：`/org/apache/maven/model/pom-4.0.0.xml`，它是所有Maven POM的父POM，所有Maven项目都继承该配置。

```（xml）
<repositories>
   <repository>
     <id>central</id>
     <name>Central Repository</name>
     <url>https://repo.maven.apache.org/maven2</url>
     <layout>default</layout>
     <snapshots>
       <enabled>false</enabled>
     </snapshots>
   </repository>
 </repositories>
...
```

#### 设置远程仓库

maven的仓库只有两大类：

![Maven仓库.png](https://i.loli.net/2019/04/04/5ca5f41f3264d.png)

说到远程仓库，先从最核心的中央仓库开始，中央仓库是默认的远程仓库，maven在安装的时候，自带的默认中央仓库地址为`http://repo1.maven.org/maven2/`，此仓库由Maven社区管理，包含了绝大多数流行的开源Java构件，以及源码、作者信息、SCM、信息、许可证信息等。一般来说，简单的Java项目依赖的构件都可以在这里下载到。Maven社区提供了一个中央仓库的搜索地址:`http://search.maven.org/#browse`，可以查询到所有可用的库文件。

除了中央仓库，还有其它很多公共的远程仓库，如中央仓库的镜像仓库。在世界各地还有很多中央仓库的镜像仓库。镜像仓库可以理解为仓库的副本，会从原仓库定期更新资源，以保持与原仓库的一致性。从仓库中可以找到的构件，从镜像仓库中也可以找到，直接访问镜像仓库，更快更稳定。

除此之外，还有很多各具特色的公共仓库，如果需要都可以在网上找到，比如Apache Snapshots仓库，包含了来自于Apache软件基金会的快照版本。

实际开发中，一般不会使用maven默认的中央仓库，现在业界使用最广泛的仓库地址为：` http://mvnrepository.com/`，比默认的中央仓库更快、更全、更稳定。

一般来讲，公司都会通过自己的私有服务器在局域网内架设一个仓库代理。私服可以看作一种特殊的远程仓库，代理广域网上的远程仓库，供局域网内的Maven用户使用。当Maven需要下载构件的时候，先从私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。

上面提到的中央仓库、中央仓库的镜像仓库、其他公共仓库、私服都属于远程仓库的范畴。
如果maven没有在本地仓库找到想要的东西，就会自动去配置文件中指定的远程仓库寻找，找到后将它下载到你的本地仓库。如果连远程仓库都找不到想要的东西，肯定是你配置写错了。

远程仓库的设置：

```(xml)
<repositories>
<repository>

   <!--仓库唯一标识 -->
  <id>repoId </id>

   <!--远程仓库名称  -->
  <name>repoName</name>

<!--远程仓库URL，如果该仓库配置了镜像，这里的URL就没有意义了，因为任何下载请求都会交由镜像仓库处理，前提是镜像（也就是设置好的私服）需要确保该远程仓库里的任何构件都能通过它下载到  -->
  <url>http://……</url>

   <!--如何处理远程仓库里发布版本的下载 -->
  <releases>

      <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。   -->
    <enabled>false</enabled>

      <!-- 该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：-->
      <!-- always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。  -->
    <updatePolicy>always</updatePolicy>

      <!--当Maven验证构件校验文件失败时该怎么做:-->
      <!--ignore（忽略），fail（失败），或者warn（警告）。 -->
    <checksumPolicy>warn</checksumPolicy>

  </releases>

   <!--如何处理远程仓库里快照版本的下载，与发布版的配置类似 -->
  <snapshots>     
    <enabled/>
    <updatePolicy/>
    <checksumPolicy/>      
  </snapshots>

</repository>    
</repositories>
```

可以配置多个远程仓库，用<id\>加以区分。

除此之外，还有一个与<repositories\>并列存在<pluginRepositories\>节点，用来配置插件的远程仓库。

仓库主要存储两种构件。第一种构件被用作其它构件的依赖，最常见的就是各类jar包。这是中央仓库中存储的大部分构件类型。另外一种构件类型是插件，Maven插件是一种特殊类型的构件。由于这个原因，插件仓库独立于其它仓库。<pluginRepositories\>节点与<repositories\>节点除了根节点的名字不一样，子元素的结构与配置方法完全一样：

```(xml)
<pluginRepositories>
      <pluginRepository>

             <id />
             <name />
             <url />

             <releases>
                    <enabled />
                    <updatePolicy />
                    <checksumPolicy />
             </releases>

             <snapshots>
                    <enabled />
                    <updatePolicy />
                    <checksumPolicy />
             </snapshots>     

      </pluginRepository>             
</pluginRepositories>
```

远程仓库有releases和snapshots两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。例如，有时候会只为开发目的开启对快照版本下载的支持，就需要把<releases\>中的<enabled \>设为“false”，而<snapshots\>中的<enabled \>设为“true”。

由于远程仓库的配置是挂在<profile\>节点下面，如果配置有多个<profile\>节点，那么就可能有多种远程仓库的设置方案，该方案是否生效是由它的父节点<profile\>是否被激活决定的。

**maven的setting.xml不支持直接配置repositories和pluginRepositories。所幸maven还提供了profile机制**，能让用户将仓库配置放到setting.xml 中的profile里，如下：

示例：jboss远程仓库配置

```
<profiles>
    <profile>
        <id>nexus</id>
        <repositories>
            <repository>
                <id>jboss</id>
                <name>JBoss Repository</name>
                <url>http://repository.jboss.com/maven2/</url>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>daily</updatePolicy>
                </releases>
                <snapshots>
                    <enabled>false</enabled>
                    <checksumPolicy>warn</checksumPolicy>
                </snapshots>
                <layout>default</layout>
            </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>jboss</id>
            <name>JBoss Repository</name>
            <url>http://repository.jboss.com/maven2/</url>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>daily</updatePolicy>
            </releases>
            <snapshots>
                <enabled>false</enabled>
                <checksumPolicy>warn</checksumPolicy>
            </snapshots>
            <layout>default</layout>
        </pluginRepository>
    </pluginRepositories>
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>nexus</activeProfile>
</activeProfiles>
```

当执行maven构建的时候，激活的profile会将仓库配置应用到项目中去。

**maven会区别对待依赖的远程仓库与插件远程仓库**

插件的仓库使用pluginRepositories来配置，除了pluginRepositories和pluginRepositorie标签不同之外，其余所有子元素和配置依赖仓库完全一样。

**私服**不是maven的核心概念，它是一种衍生出来的特殊的maven仓库。

maven官方区别依赖仓库和插件仓库，虽然id、url所有的元素值都相同；nexus的 maven-central 代理仓库不区别依赖仓库和插件仓库，从这里即可以下载普通依赖，也可以下载maven插件。

#### 设置镜像

默认情况我们会从`https://repo.maven.apache.org/`获取库文件，有些情况下网速并不是很快，我们可以选择国内的一些公开的镜像来代替。
设置位置及配置参数如下（推荐使用阿里云）：

1. 阿里云的远程仓库

```（xml）
<settings>
...
<mirrors>
<mirror>
    <id>nexus-aliyun</id>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
</mirrors>
...
</settings>
```

2. maven官方运维的2号仓库

```（xml）
<settings>
...
<mirrors>
<mirror>
    <id>repo2</id>
    <name>Mirror from Maven Repo2</name>
    <url>http://repo2.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
</mirrors>
...
</settings>
```

3. maven在UK架设的仓库

```（xml）
<settings>
...
<mirrors>
<mirror>
    <id>ui</id>
    <name>Mirror from UK</name>
    <url>http://uk.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
</mirrors>
...
</settings>
```

4. JBoss的仓库

```（xml）
<settings>
...
<mirrors>
<mirror>
    <id>jboss-public-repository-group</id>
    <mirrorOf>central</mirrorOf>
    <name>JBoss Public Repository Group</name>
   <url>http://repository.jboss.org/nexus/content/groups/public</url>
</mirror>
</mirrors>
...
</settings>
```

节点<mirrors\>下面可以配置多个镜像，<mirrorOf\>用于指明是哪个仓库的镜像，上例中使用通配符"\*"表明该私服是所有仓库的镜像，不管本地使用了多少种远程仓库，需要下载构件时都会从私服请求。

central表示该配置为中央仓库镜像，任何对于中央仓库的请求都会转至此镜像。另外三个元素与一般仓库配置无异，表示该镜像仓库的唯一标识符、名称及地址。

如果只想将私服设置成某一个远程仓库的镜像，使用<mirrorOf\>指定该远程仓库的ID即可。

```
<mirrorOf>*</mirrorOf> 表示该配置是所有Maven仓库的镜像，对于所有远程仓库的请求都会被转移到http://
<mirrorOf>external:*</mirrorOf> 匹配所有远程仓库，除localhost、file://协议。
<mirrorOf>repo1,repo2</mirrorOf> 匹配仓库1和仓库2
<mirrorOf>*,!repo1</mirrorOf> 匹配所有远程仓库，repo1除外。
```

设置完成后，我们来运行一下`mvn help:system`来看一下设置是否成功。可以看到使用的是阿里的镜像。

#### 设置http代理

使用Http代理的原因是：

1. 从远程仓库上获得的资源比较慢；
2. 当出现网络问题或者其它问题时，下载到不完整资源导致下载的资源不可用

简单来说，就是使用代理来访问远程仓库。

1. 检测本地网络是否不能直接访问Maven的远程仓库，命令为`ping repo1.maven.org`
2. 要检查代理服务器是否畅通，比如现在有一个IP地址为`192.168.10.117`，端口为3267的代理服务，我们需要先运行telnet  192.168.10.117 3267来检查该地址的该端口是否畅通，如果得道出错信息需要先获取正确的代理服务器信息，如果telnet连接正确，则输入`ctrl+]`，然后q，回车，退出即可。
3. 检查完毕之后，编辑`~/.m2/settings.xml`文件，代码如下：
   添加代理配置如下：

```（xml）
<settings>
   ...
   <proxies>
      <proxy>
         <id>my-proxy</id>
         <active>true</active>
         <protocol>http</protocol>
         <host>192.168.10.117</host>
         <port>3267</port>
         <!--
         <username>shihuan</username>
         <password>123456</password>
         <nonProxyHosts>repository.mycom.com|*.google.com</nonProxyHosts>
         -->
      </proxy>
    </proxies>
   ...
</settings>
```

这段配置十分简单，proxies下可以有多个proxy元素，如果你声明了多个proxy元素，则默认情况下第一个被激活的proxy会生效。这里声明 了一个id为my-proxy的代理，active的值为true表示激活该代理，protocol表示使用的代理协议，这里是http。当然，最重要的 是指定正确的主机名（host元素）和端口（port元素）。

上述XML配置中注释掉了username、password、nonProxyHost 几个元素，当你的代理服务需要认证时，就需要配置username和password。nonProxyHost元素用来指定哪些主机名不需要代理，可以 使用 | 符号来分隔多个主机名。

此外，该配置也支持通配符，如`*.google.com`表示所有以`google.com`结尾的域名访问都不要通过代理。

#### 最佳实践

1. 设置`MAVEN_OPTS`环境变量，设置值为 `-Xms128m -Xmx512m`，设置maven运行时占有内存大小。
2. 配置用户范围`setting.xml`, 避免影响其他用户，还有就是防止Maven在升级时覆盖配置文件。
3. 不要使用IDE内嵌的Maven。如Eclipse、NetBeans、Intellij IDEA等，尽量在使用时选择自己配置的Maven。


> 下篇我们会学习如何使用Maven构建Hello World项目

