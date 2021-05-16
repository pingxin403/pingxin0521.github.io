---
title: Maven测试和Web项目
date: 2019-03-28 20:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
### Maven 测试

在默认情况下，“maven-surefire-plugin”插件将自动执行项目“src/test/java”路径下的测试类，但测试类需要遵从以下命名模式，Maven才能自动执行它们：　　
```
    Test*.java ：以Test开头的Java类；
    *Test.java ：以Test结尾的Java类;
    *TestCase.java：以TestCase结尾的Java类；
```

<!--more-->

 如果要求Maven跳过测试，在Maven命令行中加入skipTests参数即可：

`mvn package -DskipTests`

如果不仅要跳过测试类的执行，还要临时性的跳过测试代码编译，可以执行以下命令：

`mvn package -Dmaven.test.skip=true`

Dmaven.test.skip=true 参数控制了“maven-compiler-plugin” 和“maven-surefire-plugin” 两个插件的行为，在项目构建过程中跳过了测试代码的编译和测试。此外，还可以通过配置“pom.xml”文件，完成与上述执行Maven命令同样的效果

**基本测试报告**

除了命令行输出，maven用户还可以以文件的方式生成更丰富的测试报告
默认情况下，maven-surefire-plugin会在target/surefire-reports目录下生成两种格式的错误报告：

简单文本格式（比如com.zheng.SayHelloTest.txt）
与junit兼容的xml格式（比如TEST-com.zheng.SayHelloTest.xml）


**动态指定要运行的测试类：**
在进行单元测试时，有时候不需要执行项目中所有的测试类，“maven-surefire-plugin”插件支持命令中设置Dtest参数，来执行指定的测试类。例如只执行“ATest” 测试类:

`mvn -Dtest=ATest test`

”maven-surefire-plugin“插件还支持使用星号匹配测试类名的方式以指定运行特定的测试类，星号表示匹配零个或多个字符。例如只执行”A“开头的测试类：

`mvn -Dtest=A* test`

除了使用星号匹配，还可以使用逗号指定多个测试类：

`mvn test -Dtest=ATest,BTest`

同时匹配类名以A开头的测试类和类名为BTest的测试类：

`mvn -Dtest=A*,BTest test`

当test命令通过参数匹配不到任何测试类时，项目将会构建失败。配置参数DfailIfNoTests=false可以在匹配不到测试类时依旧构建成功：

`mvn -Dtest=AATest -DfailIfNoTests=false  test`

使用DfailIfNoTests=false参数也是另一种跳过测试的方法。“maven-surefire-plugin”插件并没有提供通过指定测试类的类名来跳过执行某个测试类的命令，但可以通过在“pom.xml”文件中进行排除指定的测试类。

`mvn cobertura:cobertura`
生成覆盖率报告，接着打开项目目录target/site/cobertura/下的index.html，就能看到测试率报告。

引入以下插件依赖：

```（xml）
<plugins>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>cobertura-maven-plugin</artifactId>
        <version>2.7</version>
    </plugin>
</plugins>

```

**包含与排除测试用例：**

<include/>元素配置的测试类名将会被执行,其中`**/*Tests.java`匹配以Tests结尾的Java类，其中两个星号用来表示匹配任意路径，一个星号用来匹配路径风格符除外的零个或多个字符。
使用<excludes/>元素中排除执行匹配到的测试类。

``` （xml）
<project>
    [...]
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.20</version>
                <configuration>
                <includes>
                        <include>**/*Tests.java</include>
                    </includes>
                    <excludes>
                        <exclude>**/CTest.java</exclude>
                        <exclude>**/DTest.java</exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
   [...]
</project>
```

**测试代码重用**

mvn package 会打包项目主代码和资源文件代码，没有包含测试代码。如果想一起打包测试用例，供依赖方使用， 使用 maven-jar-plugin 插件

``` (xml)
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.4</version>
    <executions>
        <execution>
            <goals>
                <goal>test-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
maven-jar-plugin 有两个目标 jar ，test-jar,
jar 内置绑定在 default 生命周期的 package 阶段。
test-jar没有内置绑定。

依赖方引入时 dependency,需要设置 type 和 scope。

``` (xml)
<dependency>
    <groupId>org.A</groupId>
    <artifactId>A</artifactId>
    <version>5.0.0</version>
    <type>test-jar</type>
    <scope>test</scope>
</dependency>
```
### Web项目
基于java的Web应用，其标准的打包方式是WAR，WAR与JAR类似，只不过包含更多内容，包括web的资源文件。

![](https://i.loli.net/2019/04/06/5ca8b0d620b75.png)

Maven项目使用打包方式为war时，才可以生成war包，Maven的Web项目也有默认的项目目录结构：

![](https://i.loli.net/2019/04/06/5ca8b220ba80d.png)

可以发现WAR包中有一个lib目录，包含依赖的jar包，但是Maven项目中没有这样的目录，因为依赖都配置在POM中，Maven在打包成WAR包时，会根据POM的配置从本地仓库复制相应的JAR文件。


示例项目：链接：https://pan.baidu.com/s/1aFUQpEb3MhfDqOfZiKcPsg 密码：dogt


在accout-web中内置了jetty服务器插件：`jetty-maven-plugin`,并且accout-web的打包方式为war。web页面测试在web中不可缺少，但是能够使用单元测试进行测试的尽量不要留在页面测试时进行。jetty能够帮助我们省去打包和部署的步骤，

``` （xml）
<plugin>
        <groupId>org.mortbay.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>7.1.0.RC1</version>
        <configuration>
              <!--扫描项目时间间隔，默认为0，不扫描-->
                <scanIntervalSeconds>10</scanIntervalSeconds>
                <webAppConfig>
                      <!--项目部署后的context path-->
                        <contextPath>/account</contextPath>
                </webAppConfig>
        </configuration>
</plugin>
```
在主项目目录下编译完成并安装，在web项目下运行jetty之前，我们需要在setting.xml文件中添加pluginGroup中添加jetty的GroupId，以便能够使用简短命令。

`<pluginGroup>org.mortbay.jetty</pluginGroup>`

现在可以使用一下命令启动`jetty-maven-plugin`。
```
mvn jetty:run
```
然后就会启动jetty，并且默认使用8080端口，并将项目部署到容器中，并且根据配置扫描代码改动。可以使用`-Djetty.port=<端口号>`使用其他端口。

#### Cargo实现自动化部署


**部署到本地Web容器**

 在standalone模式，Cargo会从Web容器的安装目录复制一份配置到用户指定的目录，然后在此基础上部署应用，每次重新构建的时候，这个目录都会被清空，所有配置被重新生成

```（xml）
<plugin>
 <groupId>org.codehaus.cargo</groupId>
 <artifactId>cargo-maven2-plugin</artifactId>
 <version>1.7.1</version>
 <configuration>
   <container>
     <containerId>tomcat7x</containerId>
     <home>/usr/local/devtools/apache-tomcat-7.0.55</home>
   </container>
   <configuration>
     <type>standalone</type>
     <home>${project.build.directory}/tomcat7x</home>
     <properties>
       <!-- 更改监听端口 -->
       <cargo.servlet.port>8088</cargo.servlet.port>
     </properties>
   </configuration>
 </configuration>
</plugin>
```
 然后用mvn cargo:run启动。

 在existing模式下，用户需要指定现有的web容器配置目录，然后Cargo会直接使用这些配置并将应用部署到其对应的位置

```（xml）
<plugin>
	<groupId>org.codehaus.cargo</groupId>
	<artifactId>cargo-maven2-plugin</artifactId>
	<version>1.7.1</version>
	<configuration>
		<container>
			<containerId>tomcat7x</containerId>
			<home>/usr/local/devtools/apache-tomcat-7.0.55</home>
		</container>
		<configuration>
			<type>existing</type>
			<home>/usr/local/devtools/apache-tomcat-7.0.55</home>
		</configuration>
	</configuration>
</plugin>
```
 然后运行cargo:run之后在对应的tomcat的webapps目录下能够看到被部署的应用

**部署到远程Web容器**

这里注意在远程部署模式下，container元素的type子元素的值必须为remote，如果不指定，Cargo会默认使用installed，并寻找对应的容器安装目录或者安装包，一般我们远程部署的服务器上都有设定好的web容器了，并不需要再区安装。

``` (xml)
<!-- tomcat7 -->
<plugin>
	<groupId>org.apache.tomcat.maven</groupId>
	<artifactId>tomcat7-maven-plugin</artifactId>
	<version>2.2</version>
	<configuration>
		<url>http://localhost:8080/manager/text</url>
		<URIEncoding>UTF-8</URIEncoding>
		<server>tomcat7x</server>
		<username>admin</username>
		<password>password</password>
		<path>/${project.artifactId}</path>
	</configuration>
</plugin>

....

<plugin>
	<groupId>org.codehaus.cargo</groupId>
	<artifactId>cargo-maven2-plugin</artifactId>
	<version>1.7.1</version>
	<configuration>
		<container>
			<containerId>tomcat7x</containerId>
			<type>remote</type>
		</container>
		<configuration>
			<type>runtime</type>
			<properties>
				<cargo.tomcat.manager.url>http://localhost:8080/manager/text</cargo.tomcat.manager.url>
                <cargo.remote.username>admin</cargo.remote.username>
                <cargo.remote.password>password</cargo.remote.password>
            </properties>
		</configuration>
        <deployables>
            <deployable>          
                <groupId>io.steveguoshao</groupId>  
                <artifactId>webapp</artifactId>  
                <type>war</type>  
                <properties>  
                    <context>/${project.artifactId}</context>
                </properties>  
                <!-- 可选：验证是否部署成功 -->
                <pingURL>http://localhost:8080/webapp</pingURL>
                <!-- 可选：验证超时时间，默认是120000 毫秒-->
				<pingTimeout>60000</pingTimeout>
            </deployable>
        </deployables>
	</configuration>
	<executions>
		<execution>
			<id>verify-deployer</id>
			<phase>install</phase>
			<goals>
				<goal>deployer-redeploy</goal>
			</goals>
		</execution>
		<execution>
			<id>clean-deployer</id>
			<phase>clean</phase>
			<goals>
				<goal>deployer-undeploy</goal>
			</goals>
		</execution>  
	</executions>  
</plugin>


```
在tomcat7的conf/tomcat-users.xml中增加角色和用户， 不然会报403，没法访问

``` (xml)
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>  
<role rolename="manager-status"/>
<role rolename="admin-gui"/>
<user username="admin" password="password" roles="admin-gui,manager-gui,manager-script,manager-status"/>
```

另外还有一点要注意的是url,tomcat7是

`http://localhost:8080/manager/text`

而tomcat6是

`http://localhost:8080/manager/html`

配置好之后就可以运行mvn cargo:redeploy 来部署应用了（必须保证tomcat是running状态，否则没法部署），如果容器中已经部署的当前应用，Cargo会先卸载掉原来的应用，然后再重新部署。


**Cargo插件中各个命令的之间的异同**

![](https://i.loli.net/2019/04/06/5ca8c58c78e35.png)

从上面可以看出，cargo:start于cargo:run的不同之处了吧？cargo:start的生命周期依赖于maven实例的生命周期，也就是说，maven构建成功或者失败之后，cargo插件的生命周期也自动停止了；而cargo:run不同，不管maven是否构建成功或者失败，都必须手工去按Ctrl + C来停止。

### 版本管理

**如今所说的maven版本号不同于SVN的版本号**

之前我们说过Maven的版本号分为快照和稳定版本号，快照版本号使用在开发的过程中，方便于团队内部交流学习。而所说的稳定版本号，理想状态下是项目到了某个比較稳定的状态。这个稳定包括了源码和构建都要稳定。

怎样衡量项目的稳定状态呢？

1. 所有的自己主动化測试应当所有通过

2. 项目没有配置不论什么快照版本号的依赖

3. 项目没有配置不论什么快照版本号的插件

4. 项目所包括的代码都已经所有提交到了版本号控制系统中

5. 我们应当再一次运行Maven构建，以确保项目的状态是OK的

6. 我们将这一次变更提交到版本号控制的主干中，并打上标签

仅仅有满足了上述6个条件, 我们就能够将这一个快照版本号更新至公布版本号

**在开发的过程中，版本要怎样进行变更呢？Maven是否有潜在的约定？**

我们在开发的过程中。下载jar包的时候常常会发现某个jar类似这样：1.2.3-beat-4.jar,多么复杂的一个名称。

以下来解释一下。这里每一个数字的含义：

“ 1 ” :  表示该版本号的第一个重大版本号

“ 2 ” :  表示这是基于重大版本号的第二个次要版本号

“ 3 ” :  表示该次要版本号的第三个增量

" beat-4" : 表示该增量的一个里程碑

用一个图来描写叙述：

< 主版本号 >  ------   < 次版本号 > ------ < 增量版本号 > ------ < 里程碑版本号 >

主版本号：表示了项目的重大架构变更  struts1 --  struts2

次版本号：表示较大范围的功能添加和变化  Nexus1.5 ----   Nexus1.4

增量版本号：一般表示重大Bug修复  

里程碑版本号：指某一个版本号的里程碑   *.*-alpha-1  *.*-beat-1

看起来有点麻烦啊。 可是在一般来说，我们仅仅会声明主版本号和次版本号，增量版本号和里程碑版本号就不一定了。


>注：maven中约定的版本号次序：对于主版本号、次版本号、增量版本号来说他们的比較是基于数字的。因此：1.5>1.4>1.3>1.2.11>1.2.8
>对于里程碑版本号来说，比較是基于字符串的。因此：1.5>1.4>1.3>1.2.3>1.2.11

**主干、分支、标签**

上面的笔记中提到了主干和标签，究竟怎样理解主干、分支、标签呢？

主干: 项目开发代码的主体，是从项目開始到当前都处于活动的状态，从这里能够获得项目最新的源码和差点儿全部的变更历史

分支: 从主干的某个点分离出来的代码拷贝。通常能够在不影响主干的前提下。在这里进行重大的bug修复或者实验性质的开发。假设达到了预期的目的，通常将这里的变更合并到主干中去。

标签: 用来标识主干或者分支的某个点的状态，以代表项目的某个稳定状态，也就是通常说的公布状态

这三个元素，能够清晰的描写叙述出项目的版本号管理，并且也已经成了一个默认的行业标准。


**自己主动化版本号公布**

用久了手动版本号公布之后。我们会想到是否能进行自己主动化的公布版本号。答案是肯定的，这将引入一个新的插件：Maven Release Plugin

通过一些必要的配置。就能够完毕版本号公布

Maven Release Plugin 插件简单介绍：

该插件主要有三个目标：**release: prepare,  release: rollback,  release: perform** ，在介绍分支自己主动化的时候还会引入branch目标

**release:prepare**   准备版本号公布。依次运行下列操作

1. 检查项目是否有未提交的代码

2. 检查项目是否有快照版本号依赖

3. 依据用户的输入将快照版本号升级为公布版

4. 将POM中的SCM信息更新为标签地址

5. 基于改动后的POM运行maven构建

6. 提交POM变更

7. 基于用户输入为代码打标签

8. 将代码从公布版升级为新的快照版

9. 提交POM变更

**release: rollback**

回退release: prepare所运行的操作。
将POM回退至release:prepare之前的状态。并提交。

>注：该步骤不会删除release:prepare生成的标签，须要用户手动删除

**release: perform**

运行版本号公布,签出release:prepare生成的标签中的源码，并在此基础上运行mvn deploy命令打包并部署构件至仓库

>注：要为项目公布版本号，首先须要为其加入正确的版本号控制系统信息(这是由于Maven Release Plugin须要知道版本号控制系统的主干、标签等地址后才干运行相关操作)

分支的自己主动化创建

先看一下Maven Release Plugin 的branch目标能帮助我们做哪些事情

1. 检查本地有无未提交的代码

2. 将分支更改POM的版本号。如：1.1.0-SNAPSHOT改成1.1.1-SNAPSHOT

3. 将POM中的SCM信息更新为分支地址

4. 提交以上更改

5. 将主干代码拷贝到分支中

6. 改动本地代码使其回退到分支前的版本号(用户能够指定新的版本号)

7. 提交本地更改

>注：此时也必须正确的配置SCM信息


### GPG签名

GPG主要验证软件文件的合法性，时PGP算法的具体实现。最常用于给电子有效进行加密、解密以及提供签名。

GnuPG时PGP标准的一个免费实现，可以帮助我们为文件生成签名、管理秘钥以及验证秘钥。

从官网下载：`https://www.gnupg.org/`，安装后使用，在使用之前先生成一个秘钥对。之后使用私钥对文件进行签名，并且将公钥分发到公钥服务器供其他用户下载，用户使用公钥对签名进行验证。

``` （shell）
$ gpg   --genkey
```

按照真是情况输入信息，之后可以使用`gpg --list-keys`查看秘钥对信息，使用`gpg --list-secret-keys`查看私钥。

使用`gpg -ab  <文件名>`生成ascii格式签名，生成`<文件名>.asc`签名,将公钥分发到公钥服务器：`gpg --keyserver <服务器> -- send-keys  <公钥ID>`。

用户导入公钥命令：`gpg --keyserver <服务器> --recv-keys <公钥ID>`，使用`gpg --verify <文件名>.asc`验证

**Maven GPG Plugin**

Maven自动生成GPG签名

``` （xml）
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-gpg-plugin</artifactId>
	<version>1.0</version>
	<executions>
	     <execution>
		<id>sign-artifacts</id>
		<phase>verify</phase>
		<goals>
		      <goal>sign</goal>
		</goals>
	     </execution>
	</executions>
</plugin>
```
一般的mvn命令签名并发布项目构件：`mvn clean deploy  -Dgpg.passphrase=<你的密码>`，不提供`-Dgpg.passphrase`参数，运行时要求输入密码。
```（xml）
<distributionManagement>
    <repository>
        <id>proficio-repository</id>
        <name>Proficio Repository</name>
        <url>file://${basedir}/target/deploy</url>
    </repository>
</distributionManagement>
```

### 资源过滤

自定义变量在POM文件中定义时一般使用properties节点，但是对不同环境中的同名变量需要重新修改，所以就有了profile元素，就是使用一个额外的profile将元素包括。

profile可以让我们定义一系列的配置信息，然后指定其激活条件。这样我们就可以定义多个profile，然后每个profile对应不同的激活条件和配置信息，从而达到不同环境使用不同配置信息的效果。

**profile定义的位置**

1. 针对于特定项目的profile配置我们可以定义在该项目的pom.xml中。（下面举例是这种方式）

2.  针对于特定用户的profile配置，我们可以在用户的settings.xml文件中定义profile。该文件在用户家目录下的“.m2”目录下。

3.  全局的profile配置。全局的profile是定义在Maven安装目录下的“conf/settings.xml”文件中的。

``` （xml）
<profiles>

    <profile>
        <!-- 本地开发环境 -->
        <id>dev</id>
        <properties>
            <profiles.active>dev</profiles.active>
        </properties>
        <activation>
            <!-- 设置默认激活这个配置 -->
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>

    <profile>
        <!-- 发布环境 -->
        <id>release</id>
        <properties>
            <profiles.active>release</profiles.active>
        </properties>
    </profile>

    <profile>
        <!-- 测试环境 -->
        <id>beta</id>
        <properties>
            <profiles.active>beta</profiles.active>
        </properties>
    </profile>
</profiles>
```
这里定义了三个环境，分别是dev（开发环境）、beta（测试环境）、release（发布环境），其中开发环境是默认激活的（activeByDefault为true），这样如果在不指定profile时默认是开发环境，也在package的时候显示指定你要选择哪个开发环境。

**<profile\>节点**

在仓库的配置一节中，已经对setting.xml中的常用节点做了详细的说明。在这里需要特别介绍一下的是<profile\>节点的配置，profile是maven的一个重要特性。

<profile\>节点包含了激活(activation)，仓库(repositories)，插件仓库(pluginRepositories)和属性(properties)共四个子元素元素。profile元素仅包含这四个元素是因为他们涉及到整个的构建系统，而不是个别的项目级别的POM配置。

profile可以让maven能够自动适应外部的环境变化，比如同一个项目，在linux下编译linux的版本，在win下编译win的版本等。一个项目可以设置多个profile，也可以在同一时间设置多个profile被激活（active）的。自动激活的 profile的条件可以是各种各样的设定条件，组合放置在activation节点中，也可以通过命令行直接指定。如果认为profile设置比较复杂，可以将所有的profiles内容移动到专门的 profiles.xml 文件中，不过记得和pom.xml放在一起。

activation节点是设置该profile在什么条件下会被激活，常见的条件有如下几个：

1.   os
判断操作系统相关的参数，它包含如下可以自由组合的子节点元素
message - 规则失败之后显示的消息
arch - 匹配cpu结构，常见为x86
family - 匹配操作系统家族，常见的取值为：dos，mac，netware，os/2，unix，windows，win9x，os/400等
name - 匹配操作系统的名字
version - 匹配的操作系统版本号
display - 检测到操作系统之后显示的信息

2.   jdk
检查jdk版本，可以用区间表示。

3.   property
检查属性值，本节点可以包含name和value两个子节点。

4.   file
检查文件相关内容，包含两个子节点：exists和missing，用于分别检查文件存在和不存在两种情况。
如果settings中的profile被激活，那么它的值将覆盖POM或者profiles.xml中的任何相等ID的profiles。
如果想要某个profile默认处于激活状态，可以在<activeProfiles\>中将该profile的id放进去。这样，不论环境设置如何，其对应的 profile都会被激活。

**profile存在pom.xml、用户setting.xml和全局setting.xml。**

>Profile可以促使POM在开发过程拥有更多快捷选择，比如：在不同操作系统使用不同的构件等等。

资源过滤就是使用profile设置不同条件的有效资源。
