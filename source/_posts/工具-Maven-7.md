---
title: Maven插件
date: 2019-03-31 20:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
###  Maven插件

编写Maven插件的主要步骤：

1. 创建Maven-plugin项目：插件和其他项目区别就是packaging必须是maven-plugin，用户可以使用maven-archetype-plugin 快速创建一个插件项目。
<!--more-->
2. 为插件编写目标：每个插件包含一个或多个目标，Maven称为Mojo（Maven Old Java Object）。编写插件时必须提供一个或多个继承自AbstractMojo的类。

3. 为目标提供配置点：注意为插件和插件目标配置可配置的参数。

4. 编写代码实现目标行为：根据实际的需要实现Mojo

5. 错误处理以及日志：控制Mojo异常情况；在代码编写必要的日志。

6. 测试插件：编写自动化的测试代码测试行为，然后再实际运行插件以验证其行为。

#### 代码统计插件

1. 创建插件项目：使用`mvn archetype:generate`， 然后选择maven-archetype-plugin ，输入坐标信息

``` （xml）
$ tree count/
count/
├── pom.xml
└── src
    ├── it
    │   ├── settings.xml
    │   └── simple-it
    │       ├── pom.xml
    │       └── verify.groovy
    └── main
        └── java
            └── com
                └── hyp
                    └── learn
                        └── count
                            └── MyMojo.java
```
2. 修改pom.xml文件，加上2个依赖：分别是maven-plugin-api和maven-plugin-annotations，前者是插件开发API，后者是插件中使用的注解定以的包，注意打包方式为：<packaging>maven-plugin</packaging>。完整的pom.xml文件如下，一定要把自动生成那些没用的东西删掉，只留下下面的内容，否则运行插件的时候有可能报错。
```(xml)
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hyp.learn</groupId>
    <artifactId>count-maven-plugin</artifactId>
    <version>1.0-RELEASE</version>
    <packaging>maven-plugin</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-plugin-api</artifactId>
            <version>2.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugin-tools</groupId>
            <artifactId>maven-plugin-annotations</artifactId>
            <version>3.2</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```
3. 删掉默认的包，自己新建一个包com.hyp.learn.count,在这个包下面创建一个类叫做Car，继承AbstractMojo类。重写里面的execute方法。
```(Java)
package com.hyp.learn.count;
import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugin.MojoFailureException;
import org.apache.maven.plugins.annotations.LifecyclePhase;
import org.apache.maven.plugins.annotations.Mojo;
import org.apache.maven.plugins.annotations.Parameter;
@Mojo(name = "drive")
public class Car extends AbstractMojo {
    @Override
    public void execute() throws MojoExecutionException, MojoFailureException {
        System.out.println("Car drive...");
    }
}
```


4. 这样插件就开发完成了。我们将插件install到本地仓库。然后在项目组引入，可以是在本插件项目中引入，也可以在其他项目中引入。
使用`mvn clean install`安装到本地仓库.
```(xml)
<build>
        <plugins>
            <plugin>
                <groupId>com.hyp.learn</groupId>
                <artifactId>count-maven-plugin</artifactId>
                <version>1.0-RELEASE</version>
            </plugin>
        </plugins>
    </build>
```

#### 注意

1. 注解@Mojo是必须要的，这是定义插件对象的启动方法，由于该类只有一个方法，所以启动方法和启动类是一致的。在Maven 3之前是使用注释注解：@goal xxx这种方式。现在已经不使用这种方式了。

2. 我们平时在使用Maven的各种插件的时候往往都能在配置文件中传入属性的值，比如tomcat-maven-plugin插件我们可以随意指定tomcat的端口号。这里插件的处理方式是在Car类中定义一些属性，比如下面这样。然后我们重新将插件install到本地仓库。再次运行。
```（xml）
@Mojo(name = "drive")
public class Car extends AbstractMojo {
@Parameter(defaultValue = "8080")
private Integer port;
@Override
public void execute() throws MojoExecutionException, MojoFailureException {
    System.out.println("Car drive...");
    System.out.println(port);
}
}
```
那么，在插件的配置中增加Configuration标签，加上子标签<port>，如下：
``` （xml）
      <plugin>
                <groupId>com.mook.plugin</groupId>
                <artifactId>gr-maven-plugin</artifactId>
                <version>1.0-RELEASE</version>
                <configuration>
                    <port>8090</port>
                </configuration>
            </plugin>
```

那么运行时会输出8090，这就是插件的参数设置方式。

#### mojo参数

如可使用@parameter将mojo的某个字段标注为可配置参数，即mojo参数。支持boolean,int,float,String,Date,File,Url,数组,Collection,map,Propertes

boolean(boolean和Boolean)：

```
/**
*@parameter
*/
private boolean sampleBoolean
//对应配置<sampleBoolean>true</sampleBoolean>
```

int(Integer,long.Long,short,Short,byte,Byte)：

```
/**
*@parameter
*/
private int sampleInt
//对应配置<sampleInt>8</sampleInt>
```

float(Float,Double,double)：

```
/**
*@parameter
*/
private float sampleFloat
//对应配置<sampleFloat>8.2</sampleFloat>

```

String(StringBuffer,char,Character)：
```
/**
*@parameter
*/
private String sampleString
//对应配置<sampleString>heoll</sampleString>
```

Date(yyyy-MM-dd HH:mm:ss.S a或yyyy-MM-dd HH:mm:ssa)：

```
/**
*@parameter
*/
private Date sampleDate
//对应配置<sampleDate>2010-06-09 3:14:55.1 PM或2010-06-09 3:14:55 PM</sampleDate>
```

File

```
/**
*@parameter
*/
private File sampleFile
//对应配置<sampleFile>c:tem</sampleFile>
```

URL

```
/**
*@parameter
*/
private URL sampleUrl
//对应配置<sampleUrl>http;//www.baidu.com</sampleUrl>
```

数组

```
/**
*@parameter
*/
private String[] includes
//对应配置<includes><include>ee</include><include>dd</include></includes>

```

Collection(任何实现Collection接口的类)

```
/**
*@parameter
*/
private List includes
//对应配置<includes><include>ee</include><include>dd</include></includes>

```

Map

```
/**
*@parameter
*/
private Map includes

//对应配置<includes><key1>ee</key1><key2>dd</include></key2></includes>

```

Properties

```
/**
*@parameter
*/
private Properties includes


//对应配置

<includes>
  <property>
    <name>ee</name>
    <value>22</value>
  </property>
  <property>
    <name>dd</name>
    <value>11</value>
  </property>
</includes>
```


`@parameter`额外属性：
```
@parameter alias="<aliasName>":为mojo参数使用别名
@parameter expression="${aSystemProperty}":使用系统属性表达式对mojo参数进行赋值
@parameter defaultValue="aValue/${anExpression}":提供一个默认值
```

可获取`mvn -Dxxx` 中的参数值


另外

```
@readonly:只读属性，不允许配置
@required:必需的属性，未配置，会报错
```
