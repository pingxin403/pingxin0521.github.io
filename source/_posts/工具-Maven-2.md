---
title: Maven初步构建
date: 2019-03-26 13:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
### 前言
上一篇我们已经安装并基本配置了Maven，本章我们将学习构建一个基本的Hello world项目。话不多说，开始吧！！
<!--more-->
> 这章我们选择不使用IDE来构建。


### 项目结构
首先我们先了解一下，使用Maven构建的项目的通用结构。
![maven结构.gif](https://i.loli.net/2019/04/03/5ca463d9d541b.gif)

|  路径   | 作用    |
| --- | --- |
|src/main/java |Application/Library sources|
|src/main/resources|Application/Library resources|
|src/main/filters 	|Resource filter files|
|src/main/assembly 	|Assembly descriptors|
|src/main/config 	|Configuration files|
|src/main/scripts 	|Application/Library scripts|
|src/main/webapp 	|Web application sources|
|src/test/java 	|Test sources|
|src/test/resources 	|Test resources|
|src/test/filters 	|Test resource filter files|
|src/site 	|Site|
|LICENSE.txt 	|Project's license|
|NOTICE.txt 	|Notices and attributions required by libraries that the project depends on|
|README.txt 	|Project's readme|

使用目录模板，可以使 pom.xml 更简洁。因为 Maven2 已经根据缺省目录，预定义了相关的动作，而无需人工的干预。以 resources 目录为例：

src/main/resources，负责管理项目主体的资源。在使用Maven2执行compile之后，这个目录中的所有文件及子目录，会复制到target/classes目录中，为以后的打包提供了方便。

src/test/resources，负责管理项目测试的资源。在使用Maven2执行test-compile之后，这个目录中的所有文件及子目录，会复制到target/test-classes目录中，为后续的测试做好了准备。

这些动作在 Maven1 中，是需要在 maven.xml 中使用<preGoal\>或<postGoal\>来完成的。如今，完全不需要在pom.xml中指定就能够自动完成。在src和test都使用resources，方便构建和测试，这种方式本就已是前人的经验。通过使用Maven2，使这个经验在开发团队中得到普及。


### 开发流程
新建一个文件夹hello-world，作为项目目录。
我们的项目目录如下：
``` （shell）
./
├── pom.xml #配置文件
├── src #程序存放位置
│   ├── main #主程序
│   │   └── java  #java代码位置
│   │       └── com
│   │           └── hyp
│   │               └── learn
│   │                   └── hello
│   │                       └── HelloWorld.java
│   └── test #测试程序
│       └── java 测试代码位置
│           └── com
│               └── hyp
│                   └── learn
│                       └── hello
│                           └── HelloWorldTest.java
└── target #构建目录
    ├── classes #编译目录
    │   └── com
    │       └── hyp
    │           └── learn
    │               └── hello
    │                   └── HelloWorld.class
    └── maven-status
        └── maven-compiler-plugin
            └── compile
                └── default-compile
                    ├── createdFiles.lst
                    └── inputFiles.lst

23 directories, 6 files

```

#### 编写pom文件

POM(Project Object Model,项目对象模型)定义了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。
在项目目录下创建一个pom.xml文件，输入内容如下。

```（xml）
<?xml version="1.0" encoding="UTF-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocaton="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/maven-v4_0_0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.hyp.learn</groupId>
        <artifactId>hello-world</artifactId>
        <version>1.0-SNAPSHOT</version>
        <name>Maven Hello World Project</name>
</project>
```
上面按照xml格式编写，project作为根节点，包括POM的命名空间；modelVersion是指当前POM模型的版本。
groupId定义项目属于哪个组，这个组和组织或公司有关，一般是`(com/cn/org).<公司名>.<项目名>`，artifactId是当前项目的唯一ID，用于定义子项目（模块）。version指定项目版本，SNAPSHOT是快照的意思，代表不稳定的项目。name是一个代称。
配置文件位于pom.xml内，没有加java代码，解耦了配置与代码之间的影响。

#### 编写主代码

编写src/main/java/com/hyp/learn/hello/HelloWorld.java文件，代码内容如下：
``` （java）
package com.hyp.learn.hello;
public class HelloWorld
{
        public String sayHello()
        {
                return "Hello Maven";
        }
        public static void main(String [] args)
        {
                System.out.println(new HelloWorld().sayHello());
        }

}
```
首先，Maven默认从src/main/java目录下找项目主代码，无需额外配置，其次，该java类的包名是com.hyp.learn.hello,与之前在pom中定义的groupId和artifactId相吻合。
代码编写完毕，使用Maven进行编译，在项目根目录下运行命令`mvn clean compile`，clean告诉Maven清理target/目录，compile告诉Maven编译项目主代码。默认情况下，Maven构建项目输出都在target/目录下。该命令分别进行clean：clean任务、resources：resources任务、compiler：compile任务。编译完成输出到target/classes目录，编译好的类为com/hyp/learn/hello/HelloWorld.class。
```（shell）
$ mvn clean compile
[INFO] Scanning for projects...
[INFO]
[INFO] ---------------------< com.hyp.learn:hello-world >----------------------
[INFO] Building Maven Hello World Project 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ hello-world ---
[INFO] Deleting /home/hyp/programes/IdeaProjects/hello-world/target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ hello-world ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/hyp/programes/IdeaProjects/hello-world/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ hello-world ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/hyp/programes/IdeaProjects/hello-world/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.881 s
[INFO] Finished at: 2019-04-03T21:28:58+08:00
[INFO] ------------------------------------------------------------------------

```

上面提到的任务都是对应插件和插件目标，例如clean：clean是clean插件的clean目标。

至此，Maven在没有多余的配置的情况下完成了项目的清理的编译任务，接下来是编写单元测试代码，并让Mavem执行自动化测试。

#### 测试代码

为了使项目结构保持清晰，主代码与测试代码分别位于独立的目录中，Maven默认测试代码目录是src/test/java，先创建目录。
使用单元测试的话，Junit是新手必不可少的助手，我们需要在pom.xml中添加一个依赖（在name节点后面，project节点内），具体如下：
``` （xml）
...
        <dependencies>
                <dependency>
                        <groupId>junit</groupId>
                        <artifactId>junit</artifactId>
                        <version>4.7</version>
                        <scope>test</scope>
                </dependency>
        </dependencies>

...
```
dependencies元素包含多个dependency元素，以声明项目的依赖，每个依赖的标识就是groupId、artifactId和version，Maven依照标识从远程仓库下载该依赖到本地仓库，如果本地仓库具有一个唯一版本的项目，可以省略version，Maven会直接使用本地仓库的依赖。scope时指依赖在什么周期对项目起作用，一般具有以下参数：
  - compile， 默认的scope，表示 dependency 都可以在生命周期中使用。而且，这些dependencies 会传递到依赖的项目中。适用于所有阶段，会随着项目一起发布
  - provided，跟compile相似，但是表明了dependency 由JDK或者容器提供，例如Servlet AP和一些Java EE APIs。这个scope 只能作用在编译和测试时，同时没有传递性。
  - runtime，表示dependency不作用在编译时，但会作用在运行和测试时，如JDBC驱动，适用运行和测试阶段。
  - test，表示dependency作用在测试时，不作用在运行时。 只在测试时使用，用于编译和运行测试代码。不会随项目发布。
  - system，跟provided 相似，但是在系统中要以外部JAR包的形式提供，maven不会在repository查找它。

接下来就是编写测试类，单元测试类一般是针对某个类，测试类的函数，以一个函数为单元进行测试，测试类尽量路径和主代码路径相同，测试类名称是`<被测试类>Test`。现在我们编写src/test/java/com/hyp/learn/hello/HelloWorldTest.java类。

```（java）
package com.hyp.learn.hello;
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloWorldTest
{
		//测试单元使用该标签标注
        @Test
        public void testSayHello()
        {
                HelloWorld helloWorld=new HelloWorld();
                String result=helloWorld.sayHello();
                assertEquals("Hello Maven",result);
        }

}
```
一个测试类包括三个步骤：准备测试类和数据、执行要测试的行为、检查结果。

现在使用`mvn clean test`来执行测试。输出如下：

```（shell）
$ mvn clean test
[INFO] Scanning for projects...
[INFO]
[INFO] ---------------------< com.hyp.learn:hello-world >----------------------
[INFO] Building Maven Hello World Project 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ hello-world ---
[INFO] Deleting /home/hyp/programes/IdeaProjects/hello-world/target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ hello-world ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/hyp/programes/IdeaProjects/hello-world/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ hello-world ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/hyp/programes/IdeaProjects/hello-world/target/classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ hello-world ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/hyp/programes/IdeaProjects/hello-world/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ hello-world ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/hyp/programes/IdeaProjects/hello-world/target/test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ hello-world ---
[INFO] Surefire report directory: /home/hyp/programes/IdeaProjects/hello-world/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.hyp.learn.hello.HelloWorldTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.057 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.101 s
[INFO] Finished at: 2019-04-03T21:56:44+08:00
[INFO] ------------------------------------------------------------------------

```

>注：有可能会出现不支持注解的错误，java 1.5 及以上版本支持注解，可以按照以下更改配置文件

``` （xml）
...
        <build>
                <plugins>
                        <plugin>
                                <groupId>org.apache.maven.plugins</groupId>
                                <artifactId>maven-complier-plugin</artifactId>
								<version>3.1</version>
                                <configuration>
                                        <source>1.6</source>
                                        <target>1.6</target>
                                </configuration>
                        </plugin>
                </plugins>
        </build>
...
```
上面是增加插件，在编译时指定Java的版本号。

如果执行成功，可以看到compiler：testCompile执行成功，接着输出测试报告。

#### 打包和运行
讲项目进行编译、测试之后，下一个重要目标就是打包（package）。POM中默认使用打包类型时Jar，可以使用`mvn clean package`进行打包。
``` (shell)
...
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ hello-world ---
[INFO] Building jar: /home/hyp/programes/IdeaProjects/hello-world/target/hello-world-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
...
```
可以看到执行成功，并且生成target/hello-world-1.0-SNAPSHOT.jar的包，他是根据fact-version.jar生成，可以使用finalName来更改。现在我们的jar包可以复制到别的项目使用，但是我们也可以将该jar包存放在本地仓库，然后使用标识和版本号引入到项目，具体命令是`mvn clean insatll`。安装完成后可以打开仓库进行查看。

但是默认打包的jar包是无法直接执行的，需要借用插件。配置如下：
``` （xml）
...
<plugins>
 <plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-shade-plugin</artifactId>
	<version>1.2.1</version>
	<executions>
			<execution>
					<phase>package</phase>
					<goals>
					<goal>shade</goal>
					</goals>
					<configuration>
							<transformers>
									<transformer implementation="org.apache.maven.plugins.shade.resoure.ManifestResourceTransformer">
											<mainClass>com.hyp.learn.hello.HelloWorld</mainClass>
									</transformer>
							</transformers>
					</configuration>
			</execution>
	</executions>

</plugin>
</plugins>
```
我们配置mainClass为HelloWorld.class，使用完全包路径。项目在打包时会将信息放在MANIFEST中。现在执行`mvn clean install`,构建完成后可以看到有两个jar文件，有original前缀的是原始的jar包，不带的是可执行的jar，我们来测试一下：

``` （shell）
$ ls target/
classes                       maven-status                           test-classes
hello-world-1.0-SNAPSHOT.jar  original-hello-world-1.0-SNAPSHOT.jar
maven-archiver                surefire-reports
$ java -jar  target/hello-world-1.0-SNAPSHOT.jar
Hello Maven
```

本小节介绍了Hello World项目，介绍了POM、Maven项目结构以及编译、测试、打包等。

### 使用 Archetype生成项目骨架
我们可以使用Maven来省略重复的步骤。简单运行如下：
```（shell）
$ mvn archetype:generate
#然后选择模板类型编号，我是选择`org.apache.maven.archetypes:maven-archetype-webapp `
#接着输入groupId、artifactId、version、包名等数据。
Define value for property 'groupId': learn
Define value for property 'artifactId': test02
Define value for property 'version' 1.0-SNAPSHOT: :
Define value for property 'package' learn: : com.hyp.learn
Confirm properties configuration:
groupId: learn
artifactId: test02
version: 1.0-SNAPSHOT
package: com.hyp.learn
 Y: : y
#然后输入y确认
```

之后我们就可以看到一个简单的项目就建立完成。
``` （shell）
$ tree test02
test02
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
然后我们就可以使用该项目进行构建。当然也可以使用自定义的Archetype，可以节省很多时间。

### IntelliJ IDEA使用Maven

当然，使用IDE构建会更加快捷方便，想要了解如何使用IDE的可以自行搜索，我们下一章将会使用IntelliJ IDEA来构建项目。

这里我们就来介绍一下idea的简单使用。
>前提是装有java环境、Maven也已经配置好、idea也安装完成;
>idea设置好Maven的配置，位于project defaults --》settings-->Builds-->build >Tools-->Maven,更改目录以及配置文件为你的配置即可
>idea配置好JDK，需要在project defaults --》Project staucture中设置。

#### 导入Maven项目
1. 使用命令来构建idea项目
在项目目录下使用`mvn idea：idea`来构建idea的maven项目
``` （shell）
$ ls
pom.xml  src  target
$ mvn idea:idea
$ ls
hello-world.iml  hello-world.ipr  hello-world.iws  pom.xml  src  target
```
可以发现多了一些文件，现在我们可以直接使用idea打开该项目了。提示是否当做Maven项目，选择是即可。
2. 使用idea导入Maven项目
这种就是直接选择导入项目，然后选择项目目录下的pom.xml文件导入。一路OK即可

#### 构建新的Maven项目

选择create new project-->Maven-->Next(可选，选择create from archetype，以及一个archetype，上面列的或自定义的都可)--》输入groupId、artifactId、version-->配置Maven--》配置项目位置和名字--》Finish--》开始构建、会从仓库（本地或远程）下载依赖，可以选择Enable Auto Import以节省操作。


###  约定优于配置

maven的配置文件看似很复杂，其实只需要根据项目的实际背景，设置个别的几个配置项而已。maven有自己的一套默认配置，使用者除非必要，并不需要去修改那些约定内容。这就是所谓的“约定优于配置”。

文件：`${M2_HOME}/lib/maven-model-builder-3.6.0.jar`，打开该文件，能找到超级父POM：`\org\apache\maven\model\pom-4.0.0.xml`，它是所有Maven POM的父POM，所有Maven项目都继承该配置。

#### 文件目录

maven默认的文件存放结构如下：
![Maven文件目录.jpeg](https://i.loli.net/2019/04/04/5ca5fc6658432.jpeg)
每一个阶段的任务都知道怎么正确完成自己的工作，比如compile任务就知道从src/main/java下编译所有的java文件，并把它的输出class文件存放到target/classes中。

超级父POM中定义了项目目录：

``` （xml）
...
<build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
  </build>
  <reporting>
     <outputDirectory>${project.build.directory}/site</outputDirectory>
   </reporting>
...
```

对maven来说，采用"约定优于配置"的策略可以减少修改配置的工作量，也可以降低学习成本，更重要的是，给项目引入了统一的规范。
#### 版本规范

maven有自己的版本规范，一般是如下定义：

` <majorversion>.<minor version>.<incremental version>-<qualifier>`

比如1.2.3-beta-01。要说明的是，maven自己判断版本的算法是major,minor,incremental部分用数字比较，qualifier部分用字符串比较，所以要小心 alpha-2和alpha-15的比较关系，最好用 alpha-02的格式。

maven在版本管理时候可以使用几个特殊的字符串 SNAPSHOT ,LATEST ,RELEASE 。比如"1.0-SNAPSHOT"。各个部分的含义和处理逻辑如下说明：

1. SNAPSHOT
如果一个版本包含字符串"SNAPSHOT"，Maven就会在安装或发布这个组件的时候将该符号展开为一个日期和时间值，转换为UTC时间。例如，"1.0-SNAPSHOT"会在2010年5月5日下午2点10分发布时候变成1.0-20100505-141000-1。
这个词只能用于开发过程中，因为一般来说，项目组都会频繁发布一些版本，最后实际发布的时候，会在这些snapshot版本中寻找一个稳定的，用于正式发 布，比如1.4版本发布之前，就会有一系列的1.4-SNAPSHOT，而实际发布的1.4，也是从中拿出来的一个稳定版。

2. LATEST
指某个特定构件的最新发布，这个发布可能是一个发布版，也可能是一个snapshot版，具体看哪个时间最后。

3.   RELEASE
指最后一个发布版。

#### Maven变量

除了在setting.xml以及pom.xml当中用properties定义的常量，maven还提供了一些隐式的变量，用来访问系统环境变量。
![Maven变量.png](https://i.loli.net/2019/04/04/5ca5fd0f1b50b.png)

Maven内置变量说明：

```
${basedir} 项目根目录(即pom.xml文件所在目录)
${project.build.directory} 构建目录，缺省为target目录
${project.build.outputDirectory} 构建过程输出目录，缺省为target/classes
project.build.finalName产出物名称，缺省为
{project.artifactId}-${project.version}
${project.packaging} 打包类型，缺省为jar
${project.xxx} 当前pom文件的任意节点的内容
env.xxx获取系统环境变量。例如,"env.PATH"指代了
path环境变量（在Windows上是%PATH%）。
settings.xxx指代了settings.xml中对应元素的值。例如：<settings><offline>false</offline></settings>通过
{settings.offline}获得offline的值。
Java System Properties: 所有可通过java.lang.System.getProperties()访问的属性都能在POM中使用，例如 ${JAVA_HOME}。
```

**一般和profile节点结合使用。**
