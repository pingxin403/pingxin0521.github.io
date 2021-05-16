---
title: Maven 常见问题
date: 2019-04-02 20:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
### Maven [ERROR] 不再支持源选项 5。请使用 6 或更高版本

#### 在settings.xml文件中指定jdk版本

既可以修改全局的settings.xml文件（`apache/conf/settings.xml`）

也可以修改用户的settings.xml文件（`~/.m2/settings.xml`）

在settings.xml文件中找到<profiles\>标签，在里面新建一个字标签<profile\> 在里面指定jdk版本
<!--more-->

我的jdk版本是11.0.3 所以写的是低版本的8 根据你自己的jdk版本写 1.7/1.8
```
<profile>  
     <id>jdk-11</id>  
     <activation>  
         <activeByDefault>true</activeByDefault>  
         <jdk>11</jdk>  
     </activation>
     <properties>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
         <maven.compiler.source>8</maven.compiler.source>  
         <maven.compiler.target>8</maven.compiler.target>   
     </properties>   
</profile>
```
#### 在项目的pom.xml文件中指定jdk版本

我的jdk版本是11.0.3 所以写的是低版本的8根据你自己的jdk版本写 1.7/1.8~~~~

```
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
</properties>
```

### maven web报错:org.apache.jasper.JasperException: Unable to compile class for JSP

maven web项目启动没问题,访问页面就报错: 

>  `org.apache.jasper.JasperException: Unable to compile class for JSP  `

查找了很多文章，原因是jar包冲突，只要在pom.xml中添加下面的依赖就可以了

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.0.1</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>
```

注意：需要放在<dependencies\></dependencies\>中，不然会报错 

可是问题就来了，我添加了上面的依赖依旧是报同样的错，最后发现是启动的时候存在一些问题
我用的是Goals: tomcat:run，这样会导致一个问题：尽管我配置的是tomcat8.5，但默认使用tomcat6，而tomcat6不支持jdk1.8版本
这里就需要添加tomcat7-maven-plugin的插件
注意：如果你的版本是tomcat7-maven-plugin 2.0 的话，由于它不支持 jdk 1.8，所以把它换成 tomcat7-maven-plugin 2.2就行了

在pom.xml里添加如下代码：

```
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
</plugin>
```

这样就配置好了，记得把`Goals`改成`tomcat7:run`，便可以使用`tomcat9` 
`注意：官方并没有tomcat8-maven-plugin，这里只要使用tomcat7-maven-plugin就可以使用tomcat 9`

### 参考：

1. [Maven [ERROR] 不再支持源选项 5。请使用 6 或更高版本](https://www.cnblogs.com/0820LL/p/10586593.html)
2. [maven web报错:org.apache.jasper.JasperException: Unable to compile class for JSP](https://blog.csdn.net/ken1583096683/article/details/80837281)
