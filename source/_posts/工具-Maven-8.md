---
title: Maven 常用archetype
date: 2019-04-01 20:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---

### archetype
archetype的意思就是模板原型的意思，原型是一个Maven项目模板工具包。一个原型被定义为从其中相同类型的所有其它事情是由一个原始图案或模型。名称配合，因为我们正在努力提供一种系统，该系统提供了一种生成Maven项目的一致的手段。原型将帮助作者为用户创建Maven项目模板，并为用户提供了手段，产生的这些项目模板参数化的版本。
<!--more-->


#### 使用archetype的一般步骤

**交互式**
`mvn archetype:generate`,mvn会列出一个列表供用户选择

这个列表来自archetype-catalog.xml文件，在后续操作中，用户需要提供一些通用的基本参数，主要有groupId,artifactId,version,package，这样archetype插件就可以生成项目的骨架了

**批处理方式**

批处理方式不同在于使用上述命令时直接把参数给出来,同时使用-B参数要求archetype插件以批处理的方式运行，不过需要用户显示给出archetype的坐标信息
```
mvn archetype:generate -B \
-DarchetypeGroupId=org.apache.maven.archetypes \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DarchetypeVersion=1.0 \
-DgroupId=com.zheng.mavenstudy\
-DartifactId=archetype-test \
-Dversion=1.0-SNAPSHOT \
-Dpackage=com.zheng.mavenstudy
```

#### 常用的archetype介绍

`maven-archetype-quickstart`默认值 常用于一般javase项目结构
`maven-archetype-webapp` 用于web项目架构
`appfuse archetype` appfuse是一个集成了很多开源工具的项目，在于帮助程序员快速高效的创建项目，它提供了大量archetype，方便用户使用各种类型的项目

#### 编写自己的archetype项目

使用`mvn archetype:generate`创建项目,选择`org.apache.maven.archetypes:maven-archetype-archetype`类型。

一个archetype-maven项目需要包含以下几个重要部分
```
pom.xml archetype自身的pom
src/main/resources/archetype-resources/pom.xml 基于该archetype生成的项目pom原型
src/main/resources/META-INF/maven/archetype-metadata.xml， archetype的描述符文件
src/main/resources/archetype-resources/其他需要包含在archetype项目中的内容
```

基本结构如下
``` (shell)
$ tree archetype-maven/
archetype-maven/
├── pom.xml
└── src
    └── main
        └── resources
            ├── archetype-resources
            │   ├── pom.xml
            │   └── src
            │       ├── main
            │       │   └── java
            │       │       └── App.java
            │       └── test
            │           └── java
            │               └── AppTest.java
            └── META-INF
                └── maven
                    └── archetype.xml

11 directories, 5 files

```

archetype的pom.xml文件，用于定义archetype项目的坐标

 archetype-test/pom.xml
 ``` （xml）
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.hyp.learn</groupId>
   <artifactId>archetype-maven</artifactId>
   <version>1.0-SNAPSHOT</version>
   <name>Archetype - archetype-maven</name>
   <url>http://maven.apache.org</url>
 </project>
 ```

 生成的项目pom.xml,用于描述生成项目中包含的一些配置，包括参数、依赖等，其中groupId,artifactId,version使用了属性，替换文件位于`archetype-test/src/main/resources/archetype-resources/pom.xml`


```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>$com.hyp.learn</groupId>
  <artifactId>$archetype-maven</artifactId>
  <version>$1.0-SNAPSHOT</version>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>

```

archetype-metadata.xml文件，用于描述archetype项目哪些文件夹及目录需要被引入到项目中位于

`src/main/resources/META-INF/maven/archetype-metadata.xml`

其中通过fileSets设置了需要被用于项目中的文件，通过filtered,packaged分别设置指定目录下包含的文件是否需要属性值替换，同时对应的目录是否需要生成包目录

```
<?xml version="1.0" encoding="UTF-8"?>
<archetype-catalog xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0 http://maven.apache.org/xsd/archetype-catalog-1.0.0.xsd"
                   xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <fileSets>
        <fileSet filtered="true" packaged="true">
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.java</include>
            </includes>
        </fileSet>
        <fileSet filtered="true" packaged="true">
            <directory>src/test/java</directory>
            <includes>
                <include>**/*.java</include>
            </includes>
        </fileSet>
        <fileSet filtered="true" packaged="false">
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
            </includes>
        </fileSet>
    </fileSets>
    <requiredProperties>
        <requiredProperty key="groupId">
            <defaultValue>com.zheng.archetypestudy</defaultValue>
        </requiredProperty>
    </requiredProperties>
</archetype-catalog>

```

上面配置的src/main/java,src/test/java,src/main/resources，分别指向目标项目的`src/main/java,src/test/java,src/main/resources`对应于archetype-test项目的

```
archetype-test/src/main/resources/archetype-resources/src/main/java
archetype-test/src/main/resources/archetype-resources/src/test/java
archetype-test/src/main/resources/archetype-resources/src/main/resources
```

其中配置的src/main/java目录，采用通配符方式包含所有当前包及其子包下的java文件，最终出现在目标项目`project-name/src/main/java/[package-name]/**/*.java`

自定义的一些项目文件`src/main/resources/archetyperesources/src/main/java/service/BaseService.java`

```
package ${package}.service;

import java.io.Serializable;
import java.util.List;

/**
 * 基础服务接口
 * Created by zhenglian on 2017/8/21.
 */
public interface BaseService<T> {
    void save(T t);
    void update(T t);
    int delete(Serializable id);
    T findById(Serializable id);
    List<T> findAll();
}

```
配置完成后，项目总体结构如上面所示，通过mvn clean install将archetype项目打包到本地仓库中如此就可以通过mvn archetype:generate命令使用本地创建的archetype项目架构了
`mvn archetype:generate -DarchetypeGroupId=com.hyo.learn  -DarchetypeArtifactId=archetype-maven -DarchetypeVersion=1.0-SNAPSHOT`

注意在运行该命令的时候，如果是在archetype-test项目根目录下，那么会报错，提示Unable to add module to the current project as it is not of packaging type 'pom',说明当前创建的项目为模块项目，而父级目录不是一个pom类型的项目，所以报错在上一级目录下运行即可

#### archetypeCatalog

在使用maven-archetype-plugin插件时，会得到一个列表供选择，这个列表的信息来源于一个名为archetype-catalog.xml的文件用户可以自定义这个archetype-catalog.xml中的内容，当然也可以通过扫描本地仓库自动生成基于该仓库的archetype-catalog.xml文件那么archetype-catalog.xml配置文件是从哪里来的呢，maven中有以下几种选择：
```
internal maven内置的archetypeCatalog

local 指向用户本地的archetype catalog,其位置为~/.m2/archetype-catalog.xml,但是该文件默认是不存在的

remote 指向了maven中央仓库的archetype catalog,具体地址为http://repo1.maven.org/maven2/archetype-catalog.xml

file://... 用户可以指定本机任何位置的archetype-catalog.xml文件

http://... 用户可以使用http协议指定远程的archetype-catalog.xml文件
```
上面几种方式可以通过mvn archetype:generate命令的时候，使用archetypeCatalog指定插件使用的catalog,例如：
`mvn archetype:generate -DarchetypeCatalog=local`

```
<?xml version="1.0" encoding="UTF-8"?>
<archetype-catalog xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0 http://maven.apache.org/xsd/archetype-catalog-1.0.0.xsd"
    xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-catalog/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <archetypes>
    <archetype>
      <groupId>org.apache.maven.archetypes</groupId>
      <artifactId>maven-archetype-mojo</artifactId>
      <version>1.0</version>
      <description>plugin</description>
    </archetype>
    <archetype>
      <groupId>org.apache.maven.archetypes</groupId>
      <artifactId>maven-archetype-quickstart</artifactId>
      <version>1.0</version>
      <description>quickstart</description>
    </archetype>
    <archetype>
      <groupId>org.apache.maven.archetypes</groupId>
      <artifactId>maven-archetype-quickstart</artifactId>
      <version>1.1</version>
      <description>quickstart</description>
    </archetype>
    <archetype>
      <groupId>org.apache.maven.archetypes</groupId>
      <artifactId>maven-archetype-webapp</artifactId>
      <version>1.0</version>
      <description>webapp</description>
    </archetype>
    <archetype>
      <groupId>org.appfuse.archetypes</groupId>
      <artifactId>appfuse-modular-spring</artifactId>
      <version>2.0</version>
      <description>appfuse-modular-spring</description>
    </archetype>
  </archetypes>
</archetype-catalog>
```

上面的archetype-catalog.xml是通过扫描本地仓库自动生成的，可以通过mvn archetype:crawl来浏览当前本地仓库的项目结构，默认会生成到本地仓库(localRepository)配置的根目录下

当然也可以通过-Drepository指定本地仓库位置，-Dcatalog指定生成的archetype-catalog.xml生成的方式
`mvn archetype:crawl -Drepository=G:/workspace/repository -Dcatalog=C:/Users/Administrator/.m2/archetype-catalog.xml`

如果没有指定-Drepository参数时，maven会通过settings.xml中配置的本地仓库进行解析

根据项目生成archetype的命令

1.mvn archetype:create-from-project     //生成archetype项目文件
2.cd target/generated-sources/archetype/   //切换到archetype项目跟目录
3.mvn install   //对archetype项目打包安装
4.mvn archetype:generate -DarchetypeCatalog=local  //利用local本地提供的archetype创建项目

#### 常用archetype

建立Maven项目时，网上建议的分别是

1、cocoon-22-archetype-webapp
2、maven-archetype-quickstart
3、maven-archetype-webapp


maven提供的41个骨架原型分别是：

1: appfuse-basic-jsf (创建一个基于Hibernate，Spring和JSF的Web应用程序的原型)
2: appfuse-basic-spring(创建一个基于Hibernate，Spring和Spring MVC的Web应用程序的原型)
3: appfuse-basic-struts(创建一个基于Hibernate，Spring和Struts 2的Web应用程序的原型)
4: appfuse-basic-tapestry(创建一个基于Hibernate，Spring 和 Tapestry 4的Web应用程序的原型)
5: appfuse-core(创建一个基于Hibernate，Spring 和 XFire的jar应用程序的原型)
6: appfuse-modular-jsf(创建一个基于Hibernate，Spring和JSF的模块化应用原型)
7: appfuse-modular-spring(创建一个基于Hibernate, Spring 和 Spring MVC 的模块化应用原型)
8: appfuse-modular-struts(创建一个基于Hibernate, Spring 和 Struts 2 的模块化应用原型)
9: appfuse-modular-tapestry (创建一个基于 Hibernate, Spring 和 Tapestry 4 的模块化应用原型)
10: maven-archetype-j2ee-simple(一个简单的J2EE的Java应用程序)
11: maven-archetype-marmalade-mojo(一个Maven的 插件开发项目 using marmalade)
12: maven-archetype-mojo(一个Maven的Java插件开发项目)
13: maven-archetype-portlet(一个简单的portlet应用程序)
14: maven-archetype-profiles()
15:maven-archetype-quickstart()
16: maven-archetype-site-simple(简单的网站生成项目)
17: maven-archetype-site(更复杂的网站项目)
18:maven-archetype-webapp(一个简单的Java Web应用程序)
19: jini-service-archetype(Archetype for Jini service project creation)
20: softeu-archetype-seam(JSF+Facelets+Seam Archetype)
21: softeu-archetype-seam-simple(JSF+Facelets+Seam (无残留) 原型)
22: softeu-archetype-jsf(JSF+Facelets 原型)
23: jpa-maven-archetype(JPA 应用程序)
24: spring-osgi-bundle-archetype(Spring-OSGi 原型)
25: confluence-plugin-archetype(Atlassian 聚合插件原型)
26: jira-plugin-archetype(Atlassian JIRA 插件原型)
27: maven-archetype-har(Hibernate 存档)
28: maven-archetype-sar(JBoss 服务存档)
29: wicket-archetype-quickstart(一个简单的Apache Wicket的项目)
30: scala-archetype-simple(一个简单的scala的项目)
31: lift-archetype-blank(一个 blank/empty liftweb 项目)
32: lift-archetype-basic(基本（liftweb）项目)
33: cocoon-22-archetype-block-plain([http://cocoapacorg2/maven-plugins/])
34: cocoon-22-archetype-block([http://cocoapacorg2/maven-plugins/])
35:cocoon-22-archetype-webapp([http://cocoapacorg2/maven-plugins/])
36: myfaces-archetype-helloworld(使用MyFaces的一个简单的原型)
37: myfaces-archetype-helloworld-facelets(一个使用MyFaces和Facelets的简单原型)
38: myfaces-archetype-trinidad(一个使用MyFaces和Trinidad的简单原型)
39: myfaces-archetype-jsfcomponents(一种使用MyFaces创建定制JSF组件的简单的原型)
40: gmaven-archetype-basic(Groovy的基本原型)
41: gmaven-archetype-mojo(Groovy mojo 原型)

#### Maven根据一个现有的项目创建模板

1.首先需要一个现成的项目，预览一下

```
localhost:didi-standard-project didi$ ll /home/hyp/work/git/project
total 32
drwxr-xr-x  11 didi  staff   352  7 21 18:14 .
drwxr-xr-x  20 didi  staff   640  7 21 18:12 ..
drwxr-xr-x  12 didi  staff   384  7 21 18:10 .idea
-rw-r--r--   1 didi  staff    51  7 21 12:33 README.md
drwxr-xr-x   6 didi  staff   192  7 21 18:14 dao
drwxr-xr-x   5 didi  staff   160  7 21 18:14 domain
drwxr-xr-x   5 didi  staff   160  7 21 18:14 service
drwxr-xr-x   5 didi  staff   160  7 21 18:14 util
drwxr-xr-x   5 didi  staff   160  7 21 18:14 web
-rw-r--r--   1 didi  staff  2938  7 21 16:34 project.iml
-rw-r--r--   1 didi  staff  5353  7 21 16:44 pom.xml
```

2.进入现有项目根目录，运行命令

```
$ cd /home/hyp/work/git/project
$ mvn clean compile
$ mvn install
$ mvn archetype:create-from-project
#执行成功后会在现有项目根目录新增一个文件夹，名字叫做target,预览下该目录
```

3.进一步执行mvn install命令

```
$ cd /home/hyp/work/git/project/target/generated-sources/archetype/
$ mvn install
#正常情况下maven仓库目录下会新建一个名叫archetype-catalog.xml的文件
#得到maven仓库路径
$ mvn help:evaluate -Dexpression=settings.localRepository | grep -v '\[INFO\]'
/Users/didi/.m2/repository

```

4.大功告成，接下来创建项目测试一下

```
$ cd /home/hyp/work/temp
$ mvn archetype:generate -DarchetypeCatalog=local 
mvn archetype:generate -DarchetypeCatalog=local 
接下来依次输入：
1: local -> com.xxx:project-archetype (项目脚手架，初衷是方便大家快速创建Java工程，有兴趣的小伙伴可以加入进来一起完善。)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 1
Define value for property 'groupId': com.hyp
Define value for property 'artifactId': hello3
Define value for property 'version' 1.0-SNAPSHOT: : 1.0.0-SNAPSHOT
Define value for property 'package' com.alioo: : com.hyp.hello
Confirm properties configuration:
groupId: com.hyp
artifactId: hello3
version: 1.0.0-SNAPSHOT
package: com.hyp.hello
 Y: 
```

至此已经根据模板生成好新项目了，快来看看吧

5.如果还想将这个模板工程分享供其它人使用呢

在前面执行mvn install改成mvn deploy即可上传到私服，上传私服需要注意在pom.xml（pom.xml路径：/home/hyp/work/git/didi-standard-project/target/generated-sources/archetype/pom.xml）中添加如下内容

```
<distributionManagement>
    <repository>
      <id>central</id>
      <name>artifactory-main-releases</name>
      <url>http://artifactory.xxx.com:80/artifactory/libs-release</url>
    </repository>
    <snapshotRepository>
      <id>snapshots</id>
      <name>artifactory-main-snapshots</name>
      <url>http://artifactory.xxx.com:80/artifactory/libs-snapshot</url>
    </snapshotRepository>
  </distributionManagement>
```

观察日志deploy是否成功，可以进一步到私服控制台（[一般情况私服网址是artifactory.xxx.com/）上去检查确认](http://xn--artifactory-km8qr99avqtnu6axds1ldn00jcdwau7m.xxx.com/）上去检查确认)

6.其它人如何利用私服上的模板来生成新项目呢

```
mvn archetype:generate -DarchetypeGroupId=com.xxx -DarchetypeArtifactId=didi-standard-project-archetype -DarchetypeVersion=1.0.0-SNAPSHOT -DarchetypeRepository=local
```

小提示：

- 上面的archetypeVersion值为1.0.0-SNAPSHOT，不同的私服实现方式上有些区别，我了解到有些私服就没有生成1.0.0-SNAPSHOT相关的pom文件，所以只好写具体的版本号了，比如1.0.0-20180721.093712-3之类的
- 可以把这个命令保存一个bat/Shell脚本，毕竟记这么长的命令还是比较麻烦的
- 由于是前往私服去下载模板来创建工程，请确保$MVN_HOME/conf/setting.xml中已经配置好了私服地址信息，示例：

```xml
<profiles>
  <profile>
    <id>baseprofile</id>
    <repositories>
        <repository>
            <id>central</id>
            <name>Nexus Repository</name>
            <url>http://artifactory.xxx.com/artifactory/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
  </profile>
</profiles>

  <localRepository>${user.home}/.m2/repository</localRepository>

  <activeProfiles>
    <activeProfile>baseprofile</activeProfile>
  </activeProfiles>
```

