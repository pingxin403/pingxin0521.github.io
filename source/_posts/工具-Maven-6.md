---
title: Maven生成站点
date: 2019-03-30 20:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---

### 生成站点

用户可以让Maven自动生成一个Web站点，以Web的形式发布如项目描述、版本控制系统地址、缺陷跟踪系统地址等，更便捷、更快速地为团队提供项目当前的状态信息；

<!--more-->

1. 在pom.xml文件中，配置maven-site-plugin插件（Maven的site生命周期如果默认绑定了site插件就可以不配置）：
``` （xml）
<project>
    ... ...  
    <build>
        <plugins>
      <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7</version>
          <dependencies>
            <dependency>
              <groupId>org.apache.maven.doxia</groupId>
              <artifactId>doxia-site-renderer</artifactId>
              <version>1.8</version>
            </dependency>
          </dependencies>
        </plugin>
        </plugins>
    </build>
    ... ...  
</project>
```
2. 配置正确版本之后，在项目之下运行mvn site就能直接生成一个最简单的站点；
可以使用Maven生成站点，默认位于target/site目录下，可以使用`mvn site`生成一个简单的站点，可以使用`-DstagingDirectory=<路径>`设置站点位置。

3. 待Maven运行完成后，可以再项目的target/site/目录下找到Maven生成的站点文件，包括dependencies.html、dependency-convergence.html、index.html和css、images文件夹；

4. 点击index.html文件，打开生成的站点；


#### 丰富项目的信息

1. 默认情况下，Maven生成的站点包含了很多项目的信息链接，这其实是由一个名为maven-project-info--reports-plugin的插件（Maven3中，该插件内置在maven-site-plugin中，Maven2内置在核心源码中）生成的；该插件会基于POM配置生成下列项目信息报告（
  关于（about）：项目描述；
  持续集成（Continuous Integeration）：项目持续化集成服务器信息；
  依赖（Dependencies）：项目依赖信息，包括传递性依赖、依赖图、依赖许可证以及依赖文件的大小、所包含的类的数目；
  依赖收敛（Dependency Convergence）：针对多个模块项目生成，提供一些依赖健康状况分析；
  依赖管理（Dependency Management）：基于项目的 依赖管理生成的报告；
  问题追踪（Issue Tracking）：项目问题追踪系统信息；
  邮件列表（Mailing Lists）：项目的邮件列表信息；
  插件管理（Plugin Management）：项目所有项目插件的列表；
  项目许可证（Project License）：项目许可证信息；
  项目概述（Project Summary）：项目概述包括坐标、名称、描述等；
  项目团队（Project Team）：项目团队信息；
  源码仓库（Source Repository）：项目的源码仓库信息；

2. Maven不会凭空生成信息，只有用户在POM中提供了相关配置后，站点才可能包含这些信息的报告。为了让站点包含完整的项目信息，需要配置POM如下：
 ``` （xml）
 <project>
     ... ...  
     <url>http://phoneproject.qunar.com</url>
     <!--项目描述信息-->
     <description>check phone description.</description>
     <!--源码仓库信息-->
     <scm>
         <connection>scm:git:http://gitlab.corp.chengxiang.com/mobile_hotel_res/phone_spider_project</connection>
         <developerConnection>scm:git:git@gitlab.corp.chengxiang.com:mobile_hotel_res/phone_spider_project.git
         </developerConnection>
         <url>http://gitlab.corp.chengxiang.com/mobile_hotel_res/phone_spider_project</url>
     </scm>
     <!--持续化集成服务信息-->
     <ciManagement>
         <system>Jenkins</system>
         <url>http://ci.chengxiang.com/phoneproject</url>
     </ciManagement>
     <!--项目成员信息-->
     <developers>
         <developer>
             <id>chengxiang.peng</id>
             <name>chengxiang.peng</name>
             <email>chengxiang.peng@chengxiang.com</email>
             <timezone>8</timezone>
         </developer>
     </developers>
     <!--问题跟踪信息-->
     <issueManagement>
         <system>JIRA</system>
         <url>http://jira.chengxiang.com/phoneproject</url>
     </issueManagement>
     ... ...  
 </project>
 ```

3. 执行mvn site重新生成站点，发现对比简单站点，多生成了如 "CI Management" report、"Source Code Management" report等；

4. 有些时候，我们并不需要生成某些项目信息，如可能不想公开源码仓库信息。可以通过maven-project-info-reports-plugin选择性生成信息项目，pom.xml配置如下：
```（xml）
<project>
    ... ...  
    <!--报告配置-->
    <reporting>
        <plugins>
            <!--maven-project-info-reports-plugin报告插件集成在site中，故不同配置build/plugins/plugin。配置生成什么报告，
            如下配置只生成dependencies、project-team和mailing-list-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-project-info-reports-plugin</artifactId>
                <version>2.9</version>
                <reportSets>
                    <reportSet>
                        <reports>
                            <report>dependencies</report>
                            <report>project-team</report>
                        </reports>
                    </reportSet>
                </reportSets>
            </plugin>
        </plugins>
    </reporting>
    ... ...
</project>
```

#### 项目报告插件

除了默认的项目信息报告，Maven社区还提供了大量报告插件，只要稍加配置，用户就能让Maven自动生成各种内容丰富的报告。一般的插件在<project><build><plugins>配置，报告插件在<project><reporting><plugins>配置：
1. JavaDocs：使用JDK的javadoc工具，基于项目源代码生成JavaDocs文档；
pom.xml配置如下:
``` （xml）
<project>
    ... ...  
    <!--报告配置-->
    <reporting>
        <plugins>
            <!--生成Javadoc文档-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <version>2.7</version>
            </plugin>
        </plugins>
    </reporting>
    ... ...  
</project>
```
执行mvn site生成站点中，包含javadoc文档Project Reports->JavaDocs如下

2. Source Xref：能够随时随地打开浏览器访问项目的最新源代码；
pom.xml配置：
```（xml）
<project>
    ... ...  
    <!--构建配置-->
    <build>
        <plugins>
            <!--引用插件配置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jxr-plugin</artifactId>
                <version>2.2</version>
            </plugin>
        </plugins>
    </build>
    <!--报告配置-->
    <reporting>
        <plugins>
            <!--报告生成配置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jxr-plugin</artifactId>
                <version>2.2</version>
            </plugin>
        </plugins>
    </reporting>
    ... ...  
</project>
```
执行mvn site生成站点中，包含源码文档Project Reports->Source Xref。

3. CheckStyle：帮助开发人员遵循编码规范的工具，能根据一套规则自动检查Java代码，使得团队能够方便地定义自己的编码规范；
pom.xml配置如下：
```（xml）
<project>
    ... ...  
    <!--构建配置-->
    <build>
        <plugins>
            <!--代码规范检测插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>2.5</version>
                <!--在插件的目标mvn site:site中有效，配置checkstyle检测规则-->
                <configuration>
                    <configLocation>${project.basedir}/config/checkstyle/checkstyle.xml</configLocation>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <!--报告配置-->
    <reporting>
        <plugins>
            <!--代码规范检测报告-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>2.5</version>
                <!--在mvn site生命周期有效，配置checkstyle检测规则-->
                <configuration>
                    <configLocation>${project.basedir}/config/checkstyle/checkstyle.xml</configLocation>
                </configuration>
            </plugin>
        </plugins>
    </reporting>
    ... ...  
</project>
```
Checkstyle.xml配置如下（详细配置信息，查看官方文档：http://checkstyle.sourceforge.net/checks.html）：
``` （xml）
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
        "-//Puppy Crawl//DTD Check Configuration 1.3//EN"
        "http://www.puppycrawl.com/dtds/configuration_1_3.dtd">
<module name="Checker">
    <module name="TreeWalker">
        <!--包名检测-->
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z0-9]*)*$"/>
        </module>
       <!--类名检测-->
        <module name="TypeName">
            <property name="format" value="^[A-Z][a-zA-Z0-9]*$"/>
            <property name="tokens" value="CLASS_DEF"/>
        </module>
        … ….
        <!--成员变量检查-->
         <module name="LocalVariableName">
            <property name="format" value="^[a-z][a-zA-Z_0-9]*$"/>
            <!--该属性在IEAD中不包裹，在mvn site报错，mvn site -X可以查看详细信息-->
            <property name="allowOneCharVarInForLoop" value="false"/>
        </module>
    </module>
    … …..
</module>
```
执行mvn site生成站点中，包含源码文档Project Reports->CheckStyle。
>当然，使用IDE也可以进行代码检查，比如在IDEA上使用阿里巴巴Java代码规约等。

4. PMD：Java源代码分析工具，能够寻找代码中的问题，包括潜在的bug、无用的代码、可优化的代码、重复代码以及过于复杂的表达式；
pom.xml配置如下：
```（xml）
<project>
    ... ...
    <!--构建配置-->
    <build>
        <plugins>
            <!--代码问题分析插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-pmd-plugin</artifactId>
                <version>3.7</version>
            </plugin>
        </plugins>
    </build>
    <!--报告配置-->
    <reporting>
        <plugins>
            <!--代码问题分析报告-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-pmd-plugin</artifactId>
                <version>3.7</version>
            </plugin>
        </plugins>
    </reporting>
    ... ...  
</project>
```
执行mvn site生成站点中，包含源码文档Project Reports->PMD。
关于PMD自定义规则详情查看（http://pmd.sourceforge.net/pmd-4.3.0/howtowritearule.html）

5. ChangeLog：基于版本控制系统中就近的变更记录生成三份报告（貌似只支持github）；

6. Cobertura：生成测试覆盖率报告；
pom.xml配置：
``` （xml）
<project>
    ... ...  
    <!--构建配置-->
    <build>
        <plugins>
            <!--代码覆盖率检测插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>cobertura-maven-plugin</artifactId>
                <version>2.7</version>
            </plugin>
        </plugins>
    </build>
    <!--报告配置-->
    <reporting>
        <plugins>
            <!--代码覆盖检测报告-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>cobertura-maven-plugin</artifactId>
                <version>2.7</version>
            </plugin>
        </plugins>
    </reporting>
    ... ...  
</project>
```
执行mvn site生成站点中，包含源码文档Project Reports->Coberatura Test。

#### 自定义站点外观

Maven生成的站点非常灵活，除了前面提到的标准信息报告和其它创建生成的报告，还能够自定义站点的外观和布局；

分别创建target/site/site.xml、target/site/fml/faqtest.fml和target/site/apt/apttest.apt文件：
src/site/site.xml文件（定义了站点描述符，头部内容及外观-1、2、3、4，皮肤-7，导航边栏-5，创建自定义页面-5）：

``` (xml)
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!--1.左边栏-->
    <bannerLeft>
        <name>Phone Site</name>
        <!--访问url图片-->
        <src>http://tika.apache.org/asf-logo.gif</src>
        <href>http://www.baidu.com</href>
    </bannerLeft>
     
    <!--2.右边栏-->
    <bannerRight>
        <name>java</name>
        <!--访问本地图片，注意图片路径/site/resources/images/right.jpg-->
        <src>images/right.jpg</src>
        <href>http://www.hao123.com</href>
    </bannerRight>


    <!--3.显示发布时间位置-->
    <publishDate position="right"/>


    <body>
        <!--4.导航栏链接-->
        <breadcrumbs>
            <item name="youku" href="www.youku.com"/>
            <item name="iqiyi" href="www.iqiyi.com"/>
        </breadcrumbs>


        <!--5.导航边栏-->
        <menu name="${project.name}">
            <!--href-引用自定义页面，apttest.apt文件生成-->
            <item name="apttest" href="apttest.html"/>
        </menu>
        <menu name="Examples">
            <!--href-引用自定义页面，faqtest.faq文件生成-->
            <item name="faltest" href="faqtest.html"/>
        </menu>


        <!--6.ref-引用默认自动生成的报告页面-->
        <menu ref="reports"/>
    </body>


    <!--7.站点皮肤：如下图与如上有区别-->
    <skin>
        <groupId>com.googlecode.fluido-skin</groupId>
        <artifactId>fluido-skin</artifactId>
        <version>1.3</version>
    </skin>
</project>
```

src/site/fml/faqtest.fml文件（一种用来创建FAQ页面的XML文档格式）
``` (xml)
<?xml version="1.0" encoding="UTF-8"?>
<faqs xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd
      http://maven.apache.org/xsd/fml-1.0.1.xsd"
      title="fqltest title"   toplink="false">     
  <part id="install">    
      <title>title1</title>     
     <faq id="download">         
     <question>what?</question>     
         <answer>           
       <p>answer1</p>   
               <p>answer2</p>          
    </answer>       
   </faq>  
       </part>
   </faqs>
```
src/site/apt/apttest.apt文件（一种类似于维基的文档格式，用它来快速创建简单而又结构丰富的文档）

what is apt?
* aaaa
* bbbb

#### 国际化

1. 要正确的生成简单中文站点，首先要确保项目所有的源码，包括pom.xml、site.xml以及apt文档等，都是使用UTF-8编码保存；

2. 接下来我们配置pom.xml，配置编码格式，本地语言；
```（xml）
<project>
    ... ...  
    <!--属性，在POM的其它地方使用${属性名称}的方式引用属性-->
    <properties>
        <!--国际化，告诉maven-site-plugin使用UTF-8编码读取所有源码及文档-->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>
    <!--构建配置-->
    <build>
        <plugins>
            <!--生成站点插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-site-plugin</artifactId>
                <version>3.3</version>
                <!--国际化定义当地语言-->
                <configuration>
                    <locales>zh_CN</locales>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ... ....  
</project>
```
3.执行mvn site生成站点中，生成本地化站点.

>生成的站点可以通过部署上传到服务器，上传方法参考POM详解。
