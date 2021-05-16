---
title: Maven 仓库搜索服务和私服搭建
date: 2019-03-27 20:18:59
tags:
 - 工具
 - Maven
categories:
 - 工具
 - Maven
---
### Maven 仓库搜索服务

使用maven进行日常开发的时候，一个常见问题就是如何寻找需要的依赖，我们可能只知道需要使用类库的项目名称，但是添加maven依赖要求提供确切的maven坐标，这时就可以使用仓库搜索服务来根据关键字得到maven坐标。

<!--more-->

![1.png](https://i.loli.net/2019/05/13/5cd8f718a1bb757665.png)
![2.png](https://i.loli.net/2019/05/13/5cd8f718abc3739094.png)

#### Sonatype Nexus

地址: http://repository.sonatype.org/

Nexus是当前最流行的开源Maven仓库管理软件，Nexus提供了关键字搜索，类名搜索，左边搜索，校验和搜索等功能。搜索后，页面清晰地列出了结果构件的左边及所属仓库。用户可以直接下载相应构件，还可以直接复制已经根据坐标自动生成的XML依赖声明


#### Jarvana

地址：http://www.jarvana.com/jarvana/

Jarvana提供了基于关键字，类名的搜索，构件下载，依赖声明片段等功能也一应俱全。值得一提的是，Jarvana还支持浏览构建内部的内容。此外，Jarvana还提供了便捷的Java文档浏览的功能。


#### MVNbrowser

地址：http://www.mvnbrowser.com

MVNbrowser只提供关键字搜索的功能，除了提供基于坐标的依赖声明代码片段等基本功能之外，MVNbrowser的一大特色就是，能够告诉用户该构件的依赖于其他那些构件（Dependencies）以及该构件给哪些其他构件依赖（Referenced By）

#### MVNrepository

地址：https://mvnrepository.com/

MVNrepository 的界面比较清晰，提供基于关键字的搜索、依赖声明代码片段、构件下载、依赖与被依赖关系信息、构件所含包信息等功能。还提供简单的图表，显示某个构件各版本的大小变化。


>选择最适合自己的搜索服务

### 使用Nexus创建私服

私服仅仅是一种衍生出来的特殊的Maven仓库，通过自己建立私服，可以降低中央仓库负荷、节省外网带宽、加速Maven构建、自己部署构件等，从而高效使用Maven。

有些公司都不提供外网给项目组人员，因此就不能使用maven访问远程的仓库地址，所以很有必要在局域网里找一台有外网权限的机器，搭建nexus私服，然后开发人员连到这台私服上，这样的话就可以通过这台搭建了nexus私服的电脑访问maven的远程仓库。

如果某个IP地址恶意的下载中央仓库内容，例如全公司100台机器使用同一个IP反复下载，这个IP（甚至是IP段）会进入黑名单，因此稍有规模的使用Maven时，应该用Nexus架设私服。总归主要是两点：

1. 自己maven私服更容易维护，公司开发从maven私服迁出jar到本地仓库更快

2. 有些公司未开放外网给开发人员

有三种专门的Maven仓库管理软件可以帮助建立私服：Apache的Archiva、JFrog的Artifactory和Sonatype的Nexus。其中，Archiva是开源的，另外两个和核心也是开源的，可以自由选择使用。

Nexus是一个强大的Maven仓库管理器，它极大地简化了自己内部仓库的维护和外部仓库的访问。利用Nexus你可以只在一个地方就能够完全控制访问 和部署在你所维护仓库中的每个Artifact。Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。Nexus 使用ExtJS来开发界面，利用Restlet来提供完整的REST APIs，通过m2eclipse与Eclipse集成使用。Nexus支持WebDAV与LDAP安全身份认证。

Nexus的[下载地址](https://help.sonatype.com/repomanager3/download)，到官网上将ZIP的压缩包下载下来即可，解压之后发现有两个文件夹，一个是nexus-<版本号\>，另一个是sonatype-work；第一个文件夹包含了Nexus运行所需要的文件，是运行Nexus必须的；第二个文件夹目录包含Nexus生成的配置文件、日志文件、仓库文件等，当需要备份Nexus的时候，默认备份的是此目录文件。

>环境：Maven 3.6.1 + Nexus 3.x

1. Nexus安装
下载完nexus之后，只需要将压缩包解压，将解压后的文件夹放到你想要安装的目录即可——我的为/opt/xauto。

2. 配置Nexus环境变量
将nexus的bin目录设置到path的环境变量中,修改/etc/profile
```
export PATH=${PATH}:/opt/xauto/nexus/nexus/bin
```
或者使用`sudo ln -s /opt/xauto/nexus/nexus/bin/nexus  /usr/bin/ `。

> 必须使用的是java8，如果常用其他的jdk版本，则需要下载jdk8，并设置环境变量INSTALL4J_JAVA_HOME

3. 配置Nexus
在 nexus的根目录etc/nexus-default.properties的文件在找到,可以修改端口。
在nexus的根目录bin/contrib/下有可以将nexus加入服务的脚本，如果想加入服务可以执行相应脚本，也有卸载服务的脚本。

4. Nexus的测试
现在可以使用了，在命令行输入`nexus start`就可以运行，可以从浏览器访问`http://<ip-addr>:<配置文件中的端口>/nexus`。

在网页上的右上角进行登录，默认用户名：admin，密码：admin123。

>登陆之后可以对其进行修改和配置。不用时，请使用`nexus stop`关闭运行，或者使用`service nexus stop`。

5. 创建宿主目录和代理仓库
    Hosted：本地仓库，通常我们会部署自己的构件到这一类型的仓库。包括3rd party仓库，Releases仓库，Snapshots仓库。 

  Proxy：代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。

  Group：仓库组，用来合并多个hosted/proxy仓库，通常我们配置maven依赖仓库组。

  ![3.png](https://i.loli.net/2019/05/13/5cd8f718becad54764.png)

  主要介绍一下三个本地仓库：

  1. Releases:用来部署管理内部的发布版本构件的宿主类型仓库，这里存放我们自己项目中发布的构建,通常是Release版本的, 比如我们自己做了一个FTP Server的项目, 生成的构件为ftpserver.war,我们就可以把这个构建发布到Nexus的Releases本地仓库。

  2.  Snapshots:用来部署管理内部的快照版本构件的宿主类型仓库，它的目的是让我们可以发布那些非release版本, 非稳定版本,比如我们在trunk下开发一个项目,在正式release之前你可能需要临时发布一个版本给你的同伴使用, 因为你的同伴正在依赖你的模块开发,那么这个时候我们就可以发布Snapshot版本到这个仓库, 你的同伴就可以通过简单的命令来获取和使用这个临时版本。

  3. 3rd Party:无法从公共仓库获得的第三方发布版本的构件仓库，比如有些构件在中央仓库是不存在的.比如你在中央仓库找不到Oracle 的JDBC驱动, 这个时候我们就需要自己添加到3rdparty仓库

6. 创建仓库组
    点击Public Repositories仓库，在Configurations栏中选取需要合并的仓库,点击箭头加到左边保存即可

5. 添加代理仓库

     以 Sonatype 为例，添加一个代理仓库，用于代理 Sonatype 的公共远程仓库。点击菜单 Add - Proxy Repository ：填写Repository ID - sonatype；Repository Name - Sonatype Repository；Remote Storage Location - http://repository.sonatype.org/content/groups/public/ ，save 保存。

     将添加的 Sonatype 代理仓库加入 Public Repositories 仓库组。选中 Public Repositories，在 Configuration 选项卡中，将 Sonatype Repository 从右侧 Available Repositories 移到左侧 Ordered Group Repositories，save 保存

8. 搜索构件

   为了更好的使用 Nexus 的搜索，我们可以设置所有 proxy 仓库的 Download Remote Indexes 为 true，即允许下载远程仓库索引。索引下载成功之后，在 Browse Index 选项卡下，可以浏览到所有已被索引的构件信息，包括坐标、格式、Maven 依赖的 xml 代码

   或者。

   由于索引文件很大，在线下载会很漫长，所以采用离线下载会很快。

   访问http://repo.maven.apache.org/maven2/.index/下载中心仓库最新版本的索引文件，我们需要下载如下两个文件nexus-maven-repository-index.gz和nexus-maven-repository-index.properties。

   进入nexus安装目录/sonatype-work/nexus3进入indexer目录，因为我们的代理名为central所以找到central-ctx ，将下载好的文件解压进去后。重新启动nexus，若能在central 的browse index中看到和remote一样的索引，即代表成功完成。

   

9. 配置所有构建均从私服下载，在~/.m2/setting.xml中配置如下
```（xml）
<settings>
 <mirrors>
          <mirror>
                   <!--此处配置所有的构建均从私有仓库中下载 *代表所有，也可以写central -->
                   <id>nexus</id>
                   <mirrorOf>*</mirrorOf>
                   <url>http://<你的IP>:8081/nexus/content/groups/public</url>
          </mirror>
 </mirrors>
 <profiles>
          <profile>
                   <id>nexus</id>
                   <!—所有请求均通过镜像 -->
                   <repositories>
                            <repository>
                                     <id>central</id>
                                     <url>http://central</url>
                                     <releases><enabled>true</enabled></releases>
                                    <snapshots><enabled>true</enabled></snapshots>
                            </repository>
                   </repositories>
                   <pluginRepositories>
                            <pluginRepository>
                                     <id>central</id>
                                     <url>http://central</url>
                                     <releases><enabled>true</enabled></releases>
                                     <snapshots><enabled>true</enabled></snapshots>
                            </pluginRepository>
                   </pluginRepositories>
          </profile>
 </profiles>
<activeProfiles>
 <!--make the profile active all the time-->
 <activeProfile>nexus</activeProfile>
 </activeProfiles>
```
9. 部署构建到Nexus，包含Release和Snapshot， 在项目根目录中pom.xml中配置：
```（xml）
<distributionManagement>

         <repository>
            <id>releases</id>
            <name>Internal Releases</name>
            <url>http://localhost:8081/nexus/content/repositories/releases/</url>
         </repository>
         <snapshotRepository>
             <id>snapshots</id>
            <name>Internal Snapshots</name>
            <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
         </snapshotRepository>
  </distributionManagement>
```

10. Nexus的访问权限控制，在~/m2/setting.xml中配置如下：
``` (xml)
<!-- 设置发布时的用户名 -->
 <servers>
    <server>
        <id> releases </id>
        <username>admin</username>
        <password>admin123</password>
        </server>
        <server>
        <id> snapshots </id>
        <username>admin</username>
        <password>admin123</password>
 	</server>
 </servers>
```

**注意下面几点说明**

1. component name的一些说明：
1）maven-central：maven中央库，默认从https://repo1.maven.org/maven2/拉取jar
2）maven-releases：私库发行版jar
3）maven-snapshots：私库快照（调试版本）jar
4）maven-public：仓库分组，把上面三个仓库组合在一起对外提供服务，在本地maven基础配置settings.xml中使用。
2. Nexus默认的仓库类型有以下四种：
1）group(仓库组类型)：又叫组仓库，用于方便开发人员自己设定的仓库；
2）hosted(宿主类型)：内部项目的发布仓库（内部开发人员，发布上去存放的仓库）；
3）proxy(代理类型)：从远程中央仓库中寻找数据的仓库（可以点击对应的仓库的Configuration页签下Remote Storage Location属性的值即被代理的远程仓库的路径）；
4）virtual(虚拟类型)：虚拟仓库（这个基本用不到，重点关注上面三个仓库的使用）；
3. Policy(策略):表示该仓库为发布(Release)版本仓库还是快照(Snapshot)版本仓库；
4. Public Repositories下的仓库
1）3rd party: 无法从公共仓库获得的第三方发布版本的构件仓库，即第三方依赖的仓库，这个数据通常是由内部人员自行下载之后发布上去；
2）Apache Snapshots: 用了代理ApacheMaven仓库快照版本的构件仓库
3）Central: 用来代理maven中央仓库中发布版本构件的仓库
4）Central M1 shadow: 用于提供中央仓库中M1格式的发布版本的构件镜像仓库
5）Codehaus Snapshots: 用来代理CodehausMaven 仓库的快照版本构件的仓库
6）Releases: 内部的模块中release模块的发布仓库，用来部署管理内部的发布版本构件的宿主类型仓库；release是发布版本；
7）Snapshots:发布内部的SNAPSHOT模块的仓库，用来部署管理内部的快照版本构件的宿主类型仓库；snapshots是快照版本，也就是不稳定版本
所以自定义构建的仓库组代理仓库的顺序为：Releases，Snapshots，3rd party，Central。也可以使用oschina放到Central前面，下载包会更快。
5. Nexus默认的端口是8081，可以在etc/nexus-default.properties配置中修改。
6. Nexus默认的用户名密码是admin/admin123
7. 当遇到奇怪问题时，重启nexus，重启后web界面要1分钟左右后才能访问。
8. Nexus的工作目录是sonatype-work（路径一般在nexus同级目录下）

Nexus仓库分类的概念：

1）Maven可直接从宿主仓库下载构件,也可以从代理仓库下载构件,而代理仓库间接的从远程仓库下载并缓存构件
2）为了方便,Maven可以从仓库组下载构件,而仓库组并没有时间的内容(下图中用虚线表示,它会转向包含的宿主仓库或者代理仓库获得实际构件的内容).

#### 上传jar包到远程仓库

**将已有的项目打成jar包上传到私服服务器**

首先需要在pom.xml中配置上传仓库的地址，配置distributionManagement元素，仓库地址指向前面自定义的仓库

```
<distributionManagement>
        <repository>
            <id>nexus</id>
            <name>bbsidrepository</name>
            <url>http://127.0.0.1:8081/nexus/content/repositories/releases</url>
        </repository>
</distributionManagement>
```

distributionManagement包含repository和snapshotRepository子元素，前者表示发布版本（稳定版本）jar包的仓库，后者表示快照版本（开发测试版本）的仓库。

这两个元素都需要配置id、name和url，id为远程仓库的唯一标识,很重要，name只是为了方便人阅读，关键的url表示该仓库的地址。

往远程仓库部署jar包的时候，需要认证，配置认证的方式为id,一定要与前面settings中server的id保持一致。

如果项目当前的版本是快照版本，则部署到快照版本的仓库地址，否则就部署到发布版本的仓库地址，因为这里只是测试演示，前面只创建了Release版本的仓库bbsid，所以省略了snapshotRepository。

配置正确后，定位到要上传的项目目录，运行命令**mvn clean deploy**，Maven就会直接将项目打包生成的jar包部署到配置对应的远程仓库中。

**上传第三方jar包到远程仓库**

**方式一  (假设发布仓库为bbsid,发布Jar包为zbb-sms-0.0.1.jar)：**

1. settings.xml中配置认证信息。
2. . 定位到要上传的jar包的目录，执行“ mvn deploy:deploy-file  -DgroupId=com.zxp.test -DartifactId=sms -Dversion=1.0 -Dpackaging=jar  -Dfile=zbb-sms-0.0.1.jar  -Durl=http://127.0.0.1:8081/nexus/content/repositories/bbsid  -DrepositoryId=bbsnexus”命令。

说明：deploy:deploy-file表示发布独立的文件。

groupId、artifactId和version可根据需要设定。（我们要传的包为**zbb-sms-0.0.1.jar**，但是命令里指定**-Dversion=1.0**，**-DartifactId=sms**，所以最终上传到仓库后的名称为**sms-1.0.jar**）

url为Nexus服务器中需要上传的仓库路径。

repositoryId与server的id必须一致。

**方式二 Nexus控制台直接上传jar包 (假设发布仓库为bbsid,发布Jar包为mysql-connector-java-5.1.43.jar)：**

 在Repositories列表中选择Releases，点Artifact Upload,指定GAV Definition为“GAV Parameters”，然后输入相应的groupId、artifactId和version、Packaging，再点击“Select Artifact(s) to Upload...”选择指定的Jar文件，点击“Add ActifactId”添加到Actifacts框，最后点“Upload Artifact(s)”即可。