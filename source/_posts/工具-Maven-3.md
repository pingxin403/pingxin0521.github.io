---
title: Maven坐标和依赖
date: 2019-03-27 10:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
### 前言
本章主要是讲Maven如何根据pom里的依赖和插件配置，从中央远程仓库获取所需要的库和依赖库。其实，库之间的依赖问题是很困扰人的，因为有可能会出现依赖同一个库的不同版本，或者环形依赖等等，后面我们会讲到Maven是如何处理依赖并获取到合适的库的。
<!--more-->

### 坐标
坐标就是确定位置，Maven坐标就是Maven如何确定唯一的库文件。和在平面坐标系中通过（x，y）确定点的位置相似，Maven也是通过一系列坐标来确定库文件。只有确定库文件之后，Maven才会从远程仓库下载到本地。

Maven的坐标元素包括：groupId、artifactId、version、packaging、classifier。先看一组坐标：

``` （xml）
	<groupId>org.springframework</groupId>
	<artifactId>spring-context-support</artifactId>
	<version>2.5.6</version>
  <packaging>jar</packaging>
```
这是spring-context-support的坐标定义，他是springframework的一个子项目，一般可以通过groupId、artifactId、version就可以确定一个库文件，默认packaging就是jar，classifier是不能直接定义的，项目构建的文件名与坐标相对应，一般为`artifactId-version[-classifier].packaging`,其中`[-classifier]`是可选的。

1. groupId 。定义当前Maven项目隶属的实际项目。一般可以称作项目名，包含公司或组织名，例如：com.alibaba，是指阿里的项目，一般与公司网站域名对应，`alibaba.com`。项目名和Maven项目名并不是一一对应的关系，例如,阿里的 druid连接池项目，项目名是com.alibaba，Maven项目名是druid，同时阿里还有其他的Maven项目的项目名是com.alibaba。（其实，我更愿意叫它组名）。

2. artifactId。定义实际项目的Maven项目（模块），推荐是实际项目名作为artifactId的前缀，方便寻址实际构件。比如上面就是`spring-context-support`,spring就是实际项目名，context-support才是Maven项目名。

3. version。定义Maven项目当前所处的版本，Maven定义了一套完全的版本规范，以及快照的概念，我们后面会学习到。

4. packaging。定义Maven项目的打包方式。打包方式对应生成的构件文件扩展名，打包方式会影响构建的生命周期。默认为jar。packaging并非一定与扩展名相同，比如packaging为maven-plugin的构建扩展名为jar。

5. classifier。帮助定义输出的一些附属构件。附属构件不是项目直接默认生成的，而是附加的插件帮助生成。


#### 坐标规划的原则

滥用坐标、错用坐标的样例比比皆是，在中央仓库中我们能看到SpringFramework有两种坐标，其一是直接使用springframework作为groupId，如springframework:spring-beans:1.2.6，另一种是用org.springframework作为groupId，如org.springframework:spring-beans:2.5。细心看看，前一种方式显得比较随意，后一种方式则是基于域名衍生出来的，显然后者更合理，因为用户能一眼根据域名联想到其Maven坐标，方便寻找。因此新版本的SpringFramework构件都使用org.springframework作为groupId。由这个例子我们可以看到坐标规划一个原则是基于项目域名衍生。其实很多流行的开源项目都破坏了这个原则，例如JUnit，这是因为Maven社区在最开始接受构件并部署到中央仓库的时候，没有很很严格的限制，而对于这些流行的项目来说，一时间更改坐标会影响大量用户，因此也算是个历史遗留问题了。

还有一个常见的问题是将groupId直接匹配到公司或者组织名称，因为乍一看这是显而易见的。例如组织是zoo.com，有个项目是dog，那有些人就直接使用groupId com.zoo了。如果项目只有一个模块，这是没有什么问题的，但现实世界的项目往往会有很多模块，Maven的一大长处就是通过多模块的方式管理项目。那dog项目可能会有很多模块，我们用坐标的哪个部分来定义模块呢？groupId显然不对，version也不可能是，那只有artifactId。因此要这里有了另外一个原则，用artifactId来定义模块，而不是定义项目。接下来，很显然的，项目就必须用groupId来定义。因此对于dog项目来说，应该使用groupId com.zoo.dog，不仅体现出这是zoo.com下的一个项目，而且可以与该组织下的其他项目如com.zoo.cat区分开来。

除此之外，artifactId的定义也有最佳实践，我们常常可以看到一个项目有很多的模块，例如api，dao，service，web等等。Maven项目在默认情况下生成的构件，其名称不会是基于artifactId，version和packaging生成的，例如api-1.0.jar，dao-1.0.jar等等，他们不会带有groupId的信息，这会造成一个问题，例如当我们把所有这些构件放到Web容器下的时候，你会发现项目dog有api-1.0.jar，项目cat也有api-1.0.jar，这就造成了冲突。更坏的情况是，dog项目有api-1.0.jar，cat项目有api-2.0.jar，其实两者没什么关系，可当放在一起的时候，却很容易让人混淆。为了让坐标更加清晰，又出现了一个原则，即在定义artiafctId时也加入项目的信息，例如dog项目的api模块，那就使用artifactId dog-api，其他就是dog-dao，dao-service等等。虽然连字号是不允许出现在Java的包名中的，但Maven没这个限制。现在dog-api-1.0.jar，cat-2.0.jar被放在一起时，就不容易混淆了。

读者可以从[Maven: The Complete Guide](http://www.sonatype.com/books/mvnref-book/reference/pom-relationships-sect-pom-syntax.html#pom-relationships-sect-version-build-numbers)中找到详细的解释，简言之就是使用这样一个格式：

`<主版本>.<次版本>.<增量版本>-<限定符>`

其中主版本主要表示大型架构变更，次版本主要表示特性的增加，增量版本主要服务于bug修复，而限定符如alpha、beta等等是用来表示里程碑。当然不是每个项目的版本都要用到这些4个部分，根据需要选择性的使用即可。在此基础上Maven还引入了SNAPSHOT的概念，用来表示活动的开发状态，由于不涉及坐标规划，这里不进行详述。不过有点要提醒的是，由于SNAPSHOT的存在，自己显式地在version中使用时间戳字符串其实没有必要。


理解清楚Maven坐标后，我们就能开始讨论Maven的依赖管理了。

### 依赖

在maven的管理体系中，各个项目组成了一个复杂的关系网，但是每个项目都是平等的，是个没有贵贱高低，众生平等的世界，全球每个项目从理论上来说都可以相互依赖。就是说，你跟开发Spring的大牛们平起平坐，你的项目可以依赖Spring项目，Spring项目也可以依赖你的项目（虽然现实中不太会发生，你倒贴钱人家也不敢引用）。

项目的依赖关系主要分为三种：**依赖**，**继承**，**聚合**

#### 依赖关系

依赖关系是最常用的一种，就是你的项目需要依赖其他项目，比如Apache-common包，Spring包等等。

``` (xml)
<dependency>
   <groupId>junit</groupId>
   <artifactId>junit</artifactId>
   <version>4.11</version>
   <scope>test</scope>
   <type >jar</ type >
   <optional >true</ optional >
 </dependency>
```

任意一个外部依赖说明包含如下几个要素：groupId, artifactId, version, scope, type, optional。其中前3个是必须的。

这里的version可以用区间表达式来表示，比如(2.0,)表示>2.0，`[2.0,3.0)`表示2.0<=ver<3.0；多个条件之间用逗号分隔，比如`[1,3],[5,7]`。

type 一般在pom引用依赖时候出现，其他时候不用。

maven认为，程序对外部的依赖会随着程序的所处阶段和应用场景而变化，所以maven中的依赖关系有作用域(scope)的限制。在maven中，scope包含如下的取值:

![Scope选项.png](https://i.loli.net/2019/04/04/5ca5fe7314e19.png)

scope默认为compile，不同scope在生命周期的有效阶段不一样。
![Scope有效范围.png](https://i.loli.net/2019/04/04/5ca5fecfe8feb.png)

dependency中的type一般不用配置，默认是jar。当type为pom时，代表引用关系：
```（xml）
<dependency>
     <groupId>org.sonatype.mavenbook</groupId>
     <artifactId>persistence-deps</artifactId>
     <version>1.0</version>
     <type>pom</type>
</dependency>
```
#### 继承关系

继承就是避免重复，maven的继承也是这样，它还有一个好处就是让项目更加安全。项目之间存在上下级关系时就属于继承关系。

父项目的配置如下：
``` （xml）
<project>  
	  <modelVersion>4.0.0</modelVersion>  
	  <groupId>org.clf.parent</groupId>  
	  <artifactId>my-parent</artifactId>  
	  <version>2.0</version>  
	  <packaging>pom</packaging>

	  <!-- 该节点下的依赖会被子项目自动全部继承 -->
	  <dependencies>
  	    <dependency>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-api</artifactId>
               <version>1.7.7</version>
               <type>jar</type>
               <scope>compile</scope>
        </dependency>
	  </dependencies>

	  <dependencyManagement>
        <!-- 该节点下的依赖关系只是为了统一版本号，不会被子项目自动继承，-->
        <!--除非子项目主动引用，好处是子项目可以不用写版本号 -->
        <dependencies>
           <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
        </dependencies>
       </dependencyManagement>

       <!-- 这个元素和dependencyManagement相类似，它是用来进行插件管理的-->
       <pluginManagement>  
       ......
       </pluginManagement>
       ...
</project>
```

**注意，此时<packaging\>必须为pom。**

为了项目的正确运行，必须让所有的子项目使用依赖项的统一版本，必须确保应用的各个项目的依赖项和版本一致，才能保证测试的和发布是相同的结果。

Maven 使用dependencyManagement 元素来提供了一种管理依赖版本号的方式。通常会在一个组织或者项目的最顶层的父POM 中看到dependencyManagement 元素。使用pom.xml 中的dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven 会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement 元素中指定的版本号。

 父项目在dependencies声明的依赖，子项目会从全部自动地继承。而父项目在dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

pluginManagement管理插件，和dependencyManagement相类似。

如果某个项目需要继承该父项目，基础配置应该这样：

``` (xml)
<project>  
	  <modelVersion>4.0.0</modelVersion>  
	  <groupId>org.clf.parent.son</groupId>  
	  <artifactId>my-son</artifactId>  
	  <version>1.0</version>
	  <!-- 声明将父项目的坐标 -->
	  <parent>
		  <groupId>org.clf.parent</groupId>  
		  <artifactId>my-parent</artifactId>  
		  <version>2.0</version>  
		  <!-- 父项目的pom.xml文件的相对路径。相对路径允许你选择一个不同的路径。 -->
		  <!--	默认值是../pom.xml。Maven首先在构建当前项目的地方寻找父项目的pom， -->
		  <!--	其次在文件系统的这个位置（relativePath位置）， -->
		  <!--	然后在本地仓库，最后在远程仓库寻找父项目的pom。 -->
		  <relativePath>../parent-project/pom.xml</relativePath>
	  </parent>

	  <!-- 声明父项目dependencyManagement的依赖，不用写版本号 、scope-->
	  <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-beans</artifactId>
          </dependency>
      </dependencies>

</project>
```
#### 聚合关系

随着技术的飞速发展和各类用户对软件的要求越来越高，软件本身也变得越来越复杂，然后软件设计人员开始采用各种方式进行开发，于是就有了我们的分层架构、分模块开发，来提高代码的清晰和重用。针对于这一特性，maven也给予了相应的配置。

maven的多模块管理也是非常强大的。一般来说，maven要求同一个工程的所有模块都放置到同一个目录下，每一个子目录代表一个模块，比如:
``` (shell)
总项目/

     pom.xml 总项目的pom配置文件

     子模块1/

           pom.xml 子模块1的pom文件

     子模块2/

          pom.xml子模块2的pom文件
```

**注意，此时<packaging\>必须为pom。**

总项目的配置如下：

``` (xml)
<project>
       <modelVersion>4.0.0</modelVersion>
       <groupId>org.clf.parent</groupId>
       <artifactId>my-parent</artifactId>
       <version>2.0</version>

       <!-- 打包类型必须为pom -->
       <packaging>pom</packaging>

       <!-- 声明了该项目的直接子模块 -->
       <modules>
       <!-- 这里配置的不是artifactId，而是这个模块的目录名称-->
        <module>module-1</module>
        <module>module-2</module>
        <module>module-3</module>
    </modules>

       <!-- 聚合也属于父子关系，总项目中的dependencies与dependencyManagement、pluginManagement用法与继承关系类似 -->
       <dependencies>
        ......
       </dependencies>

       <dependencyManagement>
        ......
       </dependencyManagement>

       <pluginManagement>
       ......
       </pluginManagement>
</project>
```
子模块的配置如下：

``` (xml)
<project>
       <modelVersion>4.0.0</modelVersion>
       <groupId>org.clf.parent.son</groupId>
       <artifactId>my-son</artifactId>
       <version>1.0</version>
       <!-- 声明将父项目的坐标 -->
       <parent>
              <groupId>org.clf.parent</groupId>
              <artifactId>my-parent</artifactId>
              <version>2.0</version>
       </parent>
</project>
```
####  继承与聚合的关系
首先，继承与聚合都属于父子关系，并且，聚合 POM与继承关系中的父POM的packaging都是pom。

不同的是，对于聚合模块来说，它知道有哪些被聚合的模块，但那些被聚合的模块不知道这个聚合模块的存在。对于继承关系的父 POM来说，它不知道有哪些子模块继承与它，但那些子模块都必须知道自己的父 POM是什么。

![](https://i.loli.net/2019/04/05/5ca701c6ee14c.png)

在实际项目中，一个 POM往往既是聚合POM，又是父 POM，它继承了某个项目，本身包含几个子模块，同时肯定会存在普通的依赖关系，就是说，依赖、继承、聚合这三种关系是并存的。

Maven可继承的POM 元素列表如下：


``` (shell)

groupId ：项目组 ID ，项目坐标的核心元素；

version ：项目版本，项目坐标的核心元素；

description ：项目的描述信息；

organization ：项目的组织信息；

inceptionYear ：项目的创始年份；

url ：项目的 url 地址

develoers ：项目的开发者信息；

contributors ：项目的贡献者信息；

distributionManagerment：项目的部署信息；

issueManagement ：缺陷跟踪系统信息；

ciManagement ：项目的持续继承信息；

scm ：项目的版本控制信息；

mailingListserv ：项目的邮件列表信息；

properties ：自定义的 Maven 属性；

dependencies ：项目的依赖配置；

dependencyManagement：醒目的依赖管理配置；

repositories ：项目的仓库配置；

build ：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等；

reporting ：包括项目的报告输出目录配置、报告插件配置等。
```

**反应堆**
反应堆时指所有模块组成的一个构建结构，对于单模块项目，反应堆就是该模块本身，对于多模块项目来说，反应堆就是包含了个模块之间继承与依赖的关系。
``` （xml）
<modules>
<!-- 这里配置的不是artifactId，而是这个模块的目录名称-->
 <module>module-1</module>
 <module>module-2</module>
 <module>module-3</module>
</modules>
```
上面写了POM的读取顺序，构建顺序与模块之间的继承和依赖关系有关，与POM读取顺序无关。


#### 依赖调解

**传递性依赖** 就是引用的库依赖文件又依赖别的库文件。其中传递性依赖和scope有关。



**依赖调解原则：如果依赖同一库，选择依赖路径最短的库；如果依赖路径相同，选择最先声明的库。**

例如：
项目A具有以下依赖关系：A->B->C->X(1.0)和A->D->X(2.0),Maven会选择X(2.0)，因为该依赖路径更短。
项目B具有以下依赖关系：B->C->X(1.0)和B->D->X(2.0),Maven会选择最先声明的库，如果C在D之前声明，Maven会选择X（1.0）。

2019-04-05 15-19-27 的屏幕截图.png
#### 最佳实践

Maven 中的依赖关系是有传递性的。例如：项目B依赖项目C（B —> C），如果有一个项目A依赖项目B（A —> B）的话，那么项目A也会依赖项目C（A —> C）。虽然，这种依赖的自动传递性给我们维护项目的必要依赖关系带来了极大地帮助，但当我们在某些情况下，需要在项目A中排除对项目C的依赖时，这时又该怎么做呢？Maven 为我们提供了两种解决方案，分别是可选依赖（Optional Dependencies）和依赖排除（Dependency Exclusions）。


虽然项目A依赖项目B，但当项目A不是完全依赖项目B的时候，即项目A只用到了项目B的一部分功能，而正巧项目B这部分功能的实现，并不需要依赖于项目C，这个时候，项目A就应该排除对项目C的依赖。

**可选依赖**


一个项目A依赖另一个项目B时，项目A可能很少一部分功能用到了项目B，此时就可以在A中配置对B的可选依赖。举例来说，一个类似hibernate的项目，它支持对mysql、oracle等各种数据库的支持，但是在引用这个项目时，我们可能只用到其对mysql的支持，此时就可以在这个项目中配置可选依赖。     配置可选依赖的原因：1、节约磁盘、内存等空间；2、避免license许可问题；3、避免类路径问题，等等。
配置方法就是在添加依赖时添加：
``` （xml）
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>sample.ProjectC</groupId>
      <artifactId>Project-C</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <optional>true</optional>
    </dependency>
  </dependencies>
</project>
```

在项目A中按照正常的方式引入对项目B的依赖即可，这个时候，项目A就不会依赖项目C了。如果项目A用到了项目B中涉及项目C的功能，则需要在项目A的 pom.xml 文件中额外配置对项目C的依赖。

应用场景：假设有一个项目X，它实现了类似Hibernate的功能，支持很多种数据库驱动，例如：mysql，oracle等。我们在构建项目X的时候，确实需要所有这些依赖。但设想，假如有一个项目Y想使用项目X提供的功能，那么它极有可能只使用其中的一种数据库驱动。这个时候，就需要我们在项目X中将所有和驱动相关的依赖设置为可选依赖。只有这样，在项目Y中声明项目X为直接依赖的时候，才不会将项目X的所有关于驱动的依赖自动引入。这时，项目Y只需要额外声明自己真正需要依赖的驱动即可。


**排除依赖**
如果项目B没有把项目C设置为可选依赖（Optional Dependencies），而项目A又想在依赖项目B的同时不依赖项目C，应该如何操作呢？这个时候，我们使用依赖排除（Dependency Exclusions）就可以达到目的。此时，项目B按照正常的方式引入对项目C的依赖，项目A在引入对项目B的依赖时，配置如下：
``` (xml)
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>sample.ProjectB</groupId>
      <artifactId>Project-B</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <exclusions>
        <exclusion>
          <groupId>sample.ProjectC</groupId>
          <artifactId>Project-C</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
</project>
```
应用场景：接上文例子，假如项目X并没有将所有对驱动的依赖设置为可选依赖（Optional Dependencies），那么项目Y直接引入对项目X的依赖时，就会将项目X中所有对驱动的依赖一起引入。这时候，就需要我们在引入项目X时通过依赖排除（Dependency Exclusions）去除掉我们不需要的那些依赖。

需要注意的是，依赖排除（Dependency Exclusions）是基于单个依赖的，并不是POM级别的。也就是说，虽然，项目A在引入对项目B的依赖时，排除了对项目C的依赖。但，并不影响项目A在其他地方引入对项目C的依赖。例如，项目A又引入了对项目D的依赖，而项目D依赖项目C，这个时候，项目A就又产生了对项目C的依赖。

另外，在Maven引入的jar包发生冲突时，也往往需要借助依赖排除（Dependency Exclusions）来解决。


> 另外，我们也可以在Pom中使用自定义属性，来同一修改版本号

**归类依赖**

使用常量不仅可以使代码变得更加简洁，更重要的是可以避免重复，只需要更改一处，降低了错误的概率。
使用maven属性归类依赖：

``` (xml)
<project>
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.juven.mvnbook.account</groupId>
  <artifactId>account-email</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <name>Account Email</name>


  <properties>
     <springframework.version>2.5.6</springframework.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${springframework.version}</version>
    </dependency>

     <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${springframework.version}</version>
    </dependency>

     <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>${springframework.version}</version>
    </dependency>
  </dependencies>
</project>
```

这里简单使用到了Maven的属性，首先使用properties元素定义了Maven的属性，该例子中定义了一个springframework.version子元素，其值为2.5.6，有了这个属性之后，Maven运行的时候，会将POM中所有${springframework.version}替换成2.5.6，也就是说，可以使用${}的方式来引用Maven的属性，然后将所有Spring Framework依赖的版本值用这一属性引用。这个和在java中常量PI替换3.14是同样的道理，只是语法不同而已。

**优化依赖**

在软件开发的过程中，程序员会通过重构等方式不断地优化自己的代码，使其变得更简洁、更灵活。同理，程序员也应该能够对maven项目的依赖了然于胸，并对其优化，如去除多余的依赖，显式声明某些必要的依赖。
maven会自动解析所有项目的直接依赖和传递性依赖，并且根据规则正确判断每个依赖的范围。对于一些依赖冲突，也能进行调解，以确保任何一个构件只能唯一的版本在依赖中存在。在这些工作之后，最后得到的那些依赖被称为已解析依赖。
可以通过运行如下命令查看当前项目的已解析依赖：`mvn dependency:list `


将直接在当前POM声明的依赖定义为顶层依赖，而这些顶层依赖的依赖则定义为第二层依赖，以此类推，有第三、第四层依赖。当这些依赖经过Maven解析之后，就会构成一个依赖树，通过这颗依赖树，就能够很清楚地看到某个依赖是通过哪条路径引入的。可以运行如下命令查看当前项目的依赖树：`mvn dependency:tree `

使用`mvn dependency:list`和`mvn dependency:tree`可以帮助我们详细了解项目中所有的依赖的具体详细，在此基础上，还有`mvn dependency:analyze`工具可以帮助分析当前项目的依赖




### Maven插件
Maven本质上是一个插件框架，它的核心并不执行任何具体的构建任务，所有这些任务都交给插件来完成，像编译是通过maven-compile-plugin实现的、测试是通过maven-surefire-plugin实现的，maven也内置了很多插件，所以我们在项目进行编译、测试、打包的过程是没有感觉到。
![Maven插件(1).png](https://i.loli.net/2019/04/04/5ca5f1b9a60e3.png)

进一步说，每个任务对应了一个插件目标（goal），每个插件会有一个或者多个目标，例如maven-compiler-plugin的compile目标用来编译位于src/main/java/目录下的主源码，testCompile目标用来编译位于src/test/java/目录下的测试源码。对于插件本身，为了能够复用代码，往往能够完成多个任务，maven-compiler-plugin有十多个目标，插件目标通用写法：`<插件名前缀>:<goal>` 。


认识上述Maven插件的基本概念能帮助你理解Maven的工作机制，不过要想更高效率地使用Maven，了解一些常用的插件还是很有必要的，这可以帮助你避免一不小心重新发明轮子。多年来Maven社区积累了大量的经验，并随之形成了一个成熟的插件生态圈。Maven官方有两个插件列表，第一个列表的GroupId为org.apache.maven.plugins，这里的插件最为成熟，具体地址为：`http://maven.apache.org/plugins/index.html`。第二个列表的GroupId为org.codehaus.mojo，这里的插件没有那么核心，但也有不少十分有用，其地址为：`http://mojo.codehaus.org/plugins.html`。

下面列举了一些常用的核心插件，每个插件的如何配置，官方网站都有详细的介绍。

一个插件通常提供了一组目标，可使用以下语法来执行：

`mvn [plugin-name]:[goal-name]`

例如，一个Java项目使用了编译器插件，通过运行以下命令编译

`mvn compiler:compile`

Maven提供以下两种类型的插件：
1.  构建插件  。在生成过程中执行，并应在pom.xml中的<build/>元素进行配置
2.  报告插件  。在网站生成期间执行的，应该在pom.xml中的<reporting/>元素进行配置。

这里仅列举几个常用的插件，每个插件的如何进行个性化配置在官网都有详细的介绍。
```(xml)
<plugins>
      <plugin>
             <!-- 编译插件 -->
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-compiler-plugin</artifactId>
             <version>2.3.2</version>
             <configuration>
                    <source>1.5</source>
                    <target>1.5</target>
             </configuration>
      </plugin>
      <plugin>
             <!-- 发布插件 -->
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-deploy-plugin</artifactId>
             <version>2.5</version>
      </plugin>
      <plugin>
             <!-- 打包插件 -->
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-jar-plugin</artifactId>
             <version>2.3.1</version>
      </plugin>
      <plugin>
             <!-- 安装插件 -->
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-install-plugin</artifactId>
             <version>2.3.1</version>
      </plugin>
      <plugin>
             <!-- 单元测试插件 -->
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-surefire-plugin</artifactId>
             <version>2.7.2</version>
             <configuration>
                    <skip>true</skip>
             </configuration>
      </plugin>
      <plugin>
             <!-- 源码插件 -->
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-source-plugin</artifactId>
             <version>2.1</version>
             <!-- 发布时自动将源码同时发布的配置 -->
             <executions>
                <execution>
                       <id>attach-sources</id>
                             <goals>
                                   <goal>jar</goal>
                            </goals>
                      </execution>
                 </executions>
      </plugin>
</plugins>
```

#### 插件绑定

Maven的生命周期与插件相互绑定，用以完成实际的构建任务。例如，项目编译这一任务，对应default生命周期的compile这一阶段，而maven-compiler-plugin这一插件的compile目标能够完成这一任务，因此将它们绑定。
![Maven插件绑定.png](https://i.loli.net/2019/04/05/5ca6dfcdecb78.png)

为了让用户减少配置，因此Maven为主要的生命周期绑定了很多插件的目标，当用户调用生命周期阶段的时候，对应的插件目标就会执行相应的任务。

clean生命周期和插件目标的绑定关系：
![2019-04-05 13-01-12 的屏幕截图.png](https://i.loli.net/2019/04/05/5ca6e15f3880f.png)


site生命周期与插件目标绑定关系：

![2019-04-05 13-01-21 的屏幕截图.png](https://i.loli.net/2019/04/05/5ca6e15f2382e.png)

default生命周期与插件目标绑定关系：

![2019-04-05 13-04-36 的屏幕截图.png](https://i.loli.net/2019/04/05/5ca6e21129bcc.png)

default生命周期还有很多其他阶段默认没有绑定任何插件，因此也没有任何实际行为。

除了默认的打包类型jar外，常见的打包类型还有war、pom、maven-plugin、ear等。

**自定义绑定**

除了内置绑定以外，用户还能够自己选择将某个插件目标绑定到生命周期的某个阶段。

``` （xml）
<build>
  <plugins>
    <plugin>
           <!-- 源码插件 -->
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-source-plugin</artifactId>
           <version>2.1</version>
           <!-- 发布时自动将源码同时发布的配置 -->
           <executions>
              <execution>
                     <id>attach-sources</id>
                     <phase>verify</phase>
                           <goals>
                                 <goal>jar</goal>
                          </goals>
                    </execution>
               </executions>
    </plugin>
  </plugins>
</build>
```
添加的插件，在executions下的每个execution子元素可以用来配置执行一个任务，该例中有配置了一个id为attach-sources的任务，在phase中配置绑定的生命周期，再通过goals配置指定要执行的插件目标，至此，自定义插件绑定完成。运行`mvn verify`查看：

``` （shell）
...
[INFO] >>> maven-source-plugin:2.1:jar (attach-sources) > generate-sources @ hello-world >>>
[INFO]
[INFO] <<< maven-source-plugin:2.1:jar (attach-sources) < generate-sources @ hello-world <<<
...
```
可以在target目录下以-sources.jar结尾的文件。有时不需要在phase定义生命周期，插件目标在编写时就已经定义了默认绑定阶段，可以使用maven-help-plugin查看详细信息。

``` (shell)
$ mvn help:describe -Dplugin=org.apache.maven.plugins:maven-source-plugin:2.1 -Ddetail
```

#### 插件配置

用户可以配置目标的参数，进一步调整插件目标所执行的任务，用户可以通过命令行和POM配置来配置这些参数。

可以在命令行配置`-D参数名=参数值`，其实参数-D时java自带的，其功能就是通过命令行设置一个java系统属性，maven简单重用了该参数。例如：

``` （shell）
$ mvn install -Dmaven.test.skip=true
```

可以在安装时跳过测试。

POM中插件全局配置。需要在configuration中配置插件的参数，可以配置插件plugin元素中，也可以在execution元素中配置。

``` （xml）
<plugin>
       <!-- 编译插件 -->
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-compiler-plugin</artifactId>
       <version>2.3.2</version>
       <configuration>
              <source>1.5</source>
              <target>1.5</target>
       </configuration>
</plugin>
```

#### 获取插件信息

在网上获取在线信息：http://maven.apache.org/plugins/

使用本地帮助获取插件信息：`mvn help:describe  -Dplugin=<groupId>:<artifactId>[:version]`

``` (shell)
$ mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin:2.1
```
也可以使用`-Ddetail`显示更详细信息，或者使用`-Dgoal=<目标名>`输出目标信息。


#### 插件解析

与我们所依赖的构件一样，插件也是基于坐标保存在我们的Maven仓库当中的。在用到插件的时候会先从本地仓库查找插件，如果本地仓库没有则从远程仓库查找插件并下载到本地仓库。

与普通的依赖构件不同的是，Maven会区别对待普通依赖的远程仓库与插件的远程仓库。前面提到的配置远程仓库只会对普通的依赖有效果。当Maven需要的插件在本地仓库不存在时是不会去我们以前配置的远程仓库查找插件的，而是需要有专门的插件远程仓库，我们来看看怎么配置插件远程仓库，在pom.xml加入如下内容：

``` （xml）
<pluginRepositories>
        <pluginRepository>
            <id>nexus</id>
            <name>nexus</name>
            <url>http://192.168.0.70:8081/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
</pluginRepositories>
```

大家可以发现，除了pluginRepositories和pluginRepository与以前配置远程仓库不同以外，其他的都是一样的，所代表的含义也是一样的。


Maven的父POM中也是有内置一个插件仓库的，我现在用的电脑安装的是Maven 3.6.0版本，我们可以找到这个文件：`${M2_HOME}/lib/maven-model-builder-3.6.0.jar`，打开该文件，能找到超级父POM：`\org\apache\maven\model\pom-4.0.0.xml`，它是所有Maven POM的父POM，所有Maven项目都继承该配置。

我们来看看默认的远程插件仓库配置的是啥：

``` (xml)
<pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
```

默认插件仓库的地址就是中央仓库咯，它关闭了对snapshots的支持，防止引入snapshots版本的插件而导致不稳定的构件。一般来说，中央仓库所包含的插件完全能够满足我们的需要，只有在少数情况下才要配置，比如项目的插件无法在中央仓库找到，或者自己编写了插件才会配置自己的远程插件仓库。

我们来看这样一个命令：

`mvn compiler:compiler`

这个命令会调用maven-compiler-plugin插件并执行compiler目标，大家有木有觉得很神奇？我们在pom.xml中配置插件往往是这样：

``` (xml)
  <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.7</source> <!-- 源代码使用的开发版本 -->
                <target>1.7</target> <!-- 需要生成的目标class文件的编译版本 -->
            </configuration>
        </plugin>
```

maven-compiler-plugin插件默认执行的目标为compiler，那么命令的完整写法应该是：`mvn org.apache.maven.plugins:maven-compiler-plugin:3.1:compiler`才对啊，为什么mvn compiler:compiler也能完美的执行？

我们来看看Maven到底干了些神马来做到如此功能：

1. 插件默认groupId
Maven默认以org.apache.maven.plugins作为groupId，到这里我们的命令应该是长这样的：
`mvn org.apache.maven.plugins:compiler:compiler`
我们也可以配置自己默认的groupId，在Maven的settings.xml中添加如下内容，前面提过最好将settings.xml放在用户目录的.m2下：
``` (xml)
<pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
    <pluginGroup>com.your.plugins</pluginGroup>
</pluginGroups>
```
不过说实在的，没必要动他就别去动他了，我们用Maven只是解决一些刚需的问题，**没必要的设置就尽量不去动他，别把Maven搞得太复杂，虽然Maven的却有点小复杂，只是希望大家能够对maven理解的更深入那么一点点，并不是建议大家一定要去使用某些东西，大家在平时的开发中要谨记这一点。**

2. 我们来看看Maven插件远程仓库的元数据`org/apache/maven/plugins/maven-metadata.xml`，Maven默认的远程仓库是`http://repo.maven.apache.org/maven2/`，所有插件元数据路径则是：`http://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-metadata.xml`，我们找到compiler插件的元数据。
这里会根据prefix指定的前缀找到对应的artifactId，到这里我们的命令应该长成了这样：
`mvn org.apache.maven.plugins:maven-compiler-plugin:compiler`

3. 我们再根据groupId和artifactId找到maven-compiler-plugin插件单个的元数据，路径为`http://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-compiler-plugin/maven-metadata.xml`。maven将所有的远程插件仓库及本地仓库元数据归并后，就能找到release的版本（maven3后为了保证项目构建的稳定性默认使用release版本），到这里命令就被扩展成为这样：
`mvn org.apache.maven.plugins:maven-compiler-plugin:3.6.0:compiler`
如果执行的是mvn compiler:compiler命令，由于maven-compiler-plugin的最新版本已经到了3.6.0，则默认会使用此版本。最后的compiler则是插件要执行的目标咯，看到这里大家应该明白mvn compiler:compiler命令为什么能够得到完美的运行了吧。



> 强调，如果我们没有在自己的pom.xml中配置相应的内容，则默认会使用超级父POM配置的内容。

很多插件是超级父POM当中并没有配置的，如果用户使用某个插件时没有设定版本，那么则会根据我上述所说的规则去仓库中查找可用的版本，然后做出选择。在Maven2中，插件的版本会被解析至latest。也就是说，当用户使用某个非核心插件且没有声明版本的时候，Maven会将版本解析为所有可用仓库中的最新版本，latest表示的就是最新版本，而这个版本很有可能是快照版本。

当插件为快照版本时，就会出现潜在的问题。昨天还好好的，可能今天就出错了，其原因是这个快照版本发生了变化导致的。为了防止这类问题，Maven3调整了解析机制，当插件没有声明版本的时候，不再解析至latest，而是使用release。这样就避免了由于快照频繁更新而导致的不稳定问题。但是这样就好了吗？不写版本号其实是不推荐的做法，例如，我使用的插件发布了一个新版本，而这个release版本与之前的版本的行为发生了变化，这种变化依然可能导致我们项目的瘫痪。所以使用插件的时候，应该一直显式的设定版本，这也解释了Maven为什么要在超级父POM中为核心插件设定版本咯。
