---
title: Gradle 依赖管理
date: 2019-05-06 16:18:59
tags:
 - 工具
 - Gradle
categories:
 - 工具
 - Gradle
---

### 依赖关系管理

**什么是依赖管理**？

软件项目很少孤立地工作。 在大多数情况下，项目依赖于库中可重用的功能，或者分解为单独的组件以组成模块化系统。 依赖关系管理是一种以自动方式声明，解析和使用项目所需的依赖关系的技术。

<!--more-->

**Gradle中的依赖管理**

Gradle内置了对依赖管理的支持，并且完成了履行现代软件项目中遇到的典型场景的任务。 我们将在示例项目的帮助下探索主要概念。 下图应该为您提供所有运动部件的概述。

![1.png](https://i.loli.net/2019/05/12/5cd7f6786c7bf.png)

示例项目构建Java源代码。 一些Java源文件从Google Guava导入类，这是一个提供丰富实用功能的开源库。 除了Guava之外，该项目还需要JUnit库来编译和执行测试代码。

Guava和JUnit代表了这个项目的依赖关系。 构建脚本开发人员可以声明不同范围的依赖关系，例如 只是为了编译源代码或执行测试。 在Gradle中，依赖关系的范围称为配置。 

通常，依赖关系以模块的形式出现。 您需要告诉Gradle在哪里找到这些模块，以便它们可以被构建使用。 存储模块的位置称为存储库。 通过为构建声明存储库，Gradle将知道如何查找和检索模块。 存储库可以采用不同的形式：作为本地目录或远程存储库。 存储库类型的参考提供了有关此主题的广泛报道。

在运行时，Gradle将根据操作特定任务的需要找到声明的依赖项。 可能需要从远程存储库下载依赖项，从本地目录检索依赖项或者需要在多项目设置中构建另一个项目。 此过程称为依赖性解析。 您可以在依赖项解析的工作原理中找到详细的讨论。

解析后，解析机制将依赖项的基础文件存储在本地缓存中，也称为依赖性缓存。 Future构建重用存储在缓存中的文件以避免不必要的网络调用。

模块可以提供其他元数据。 元数据是更详细地描述模块的数据，例如 在存储库中查找它的坐标，有关项目的信息或其作者。 作为元数据的一部分，模块可以定义需要其他模块才能正常工作。 例如，JUnit 5平台模块还需要平台commons模块。 Gradle自动解析那些附加模块，即所谓的传递依赖项。 如果需要，您可以自定义处理传递依赖项的行为以满足项目的要求。

具有数十或数百个声明的依赖项的项目很容易受到依赖地狱的影响。 Gradle提供了足够的工具，可以借助构建扫描或内置任务来可视化，导航和分析项目的依赖关系图。 了解有关检查依赖关系的更多信息。

#### 依赖解析如何工作

Gradle接受您的依赖项声明和存储库定义，并尝试通过称为依赖项解析的过程下载所有依赖项。 以下是此过程如何工作的简要概述。

- 给定所需的依赖关系，Gradle尝试通过搜索依赖关系指向的模块来解决依赖关系。 按顺序检查每个存储库。 根据存储库的类型，Gradle会查找描述模块（.module，.pom或ivy.xml文件）的元数据文件，或直接查找工件文件。
  - ​     如果依赖项被声明为动态版本（如1. +，[1.0，），[1.0,2.0）），Gradle将把它解析为存储库中最高的可用具体版本（如1.2）。 对于Maven存储库，这是使用maven-metadata.xml文件完成的，而对于Ivy存储库，这是通过目录列表完成的。
  - ​     如果模块元数据是已声明父POM的POM文件，则Gradle将递归尝试解析POM的每个父模块。
- 一旦检查了每个存储库的模块，Gradle将选择要使用的“最佳”存储库。 这是使用以下标准完成的：
  - ​     对于动态版本，“更高”的具体版本优于“较低”版本。
  - ​     由模块元数据文件（.module，.pom或ivy.xml文件）声明的模块优先于仅具有工件文件的模块。
  - ​     早期存储库中的模块优先于后续存储库中的模块。
  - ​     当具体版本声明依赖关系并且在存储库中找到模块元数据文件时，不需要继续搜索以后的存储库，并且该过程的其余部分被短路。
- 然后，从上面的过程中选择的相同存储库请求模块的所有工件。

依赖项解析过程可高度自定义以满足企业要求。 

#### HTTP重试

Gradle会多次尝试连接到给定的存储库。 如果失败，Gradle将重试，增加每次重试之间等待的时间。 在最大次数的失败尝试之后，存储库将被列入黑名单以用于整个构建。

#### 解析典型的构建脚本

让我们看一下基于Java的项目的一个非常简单的构建脚本。 它应用Java库插件，该插件自动引入标准项目布局，提供执行典型工作的任务以及对依赖关系管理的充分支持。

基于Java的项目的依赖性声明:

```
//build.gradle
   id 'java-library'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.hibernate:hibernate-core:3.6.7.Final'
    api 'com.google.guava:guava:23.0'
    testImplementation 'junit:junit:4.+'
}
```

 Project.dependencies {}代码块声明需要Hibernate核心3.6.7.Final来编译项目的生产源代码。 它还指出编译项目测试需要junit> = 4.0。 应该在Project.repositories {}定义的Maven Central存储库中查找所有依赖项。 以下部分更详细地解释了每个方面。

#### 声明模块依赖项

您可以声明各种类型的依赖项。 一种这样的类型是模块依赖性。 模块依赖性表示对具有在当前构建之外构建的特定版本的模块的依赖性。 模块通常存储在存储库中，例如Maven Central，公司Maven或Ivy存储库，或本地文件系统中的目录。

要定义模块依赖关系，请将其添加到依赖关系配置：

```
//build.gradle
dependencies {
    implementation 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

####  使用依赖配置

Configuration是一组命名的依赖项和工件。 配置有三个主要目的：

**声明依赖项**

 插件使用配置使构建作者可以轻松地在执行插件定义的任务期间声明其他各种子项目或外部工件用于各种目的。 例如，插件可能需要Spring Web框架依赖项来编译源代码。

**解决依赖关系**

 插件使用配置来查找（并可能下载）它定义的任务的输入。 例如，Gradle需要从Maven Central下载Spring Web框架JAR文件。

**暴露artifacts以供消费**

 插件使用配置来定义它为其他项目生成的工件。 例如，项目希望将打包在JAR文件中的已编译源代码发布到内部Artifactory存储库。

考虑到这三个目的，让我们看一下Java Library Plugin定义的一些标准配置。

**implementation**

 编译项目的生产源所需的依赖项，这些依赖项不是项目公开的API的一部分。 例如，该项目使用Hibernate进行内部持久层实现。

**API**

 编译项目的生产源所需的依赖项，这是项目公开的API的一部分。 例如，项目使用Guava并在方法签名中公开具有Guava类的公共接口。

**testImplementation**

 编译和运行项目测试源所需的依赖项。 例如，项目决定使用测试框架JUnit编写测试代码。

各种插件增加了进一步的标准配置。 您还可以通过Project.configurations {}在构建中定义自己的自定义配置。 有关定义和自定义依赖关系配置的详细信息。

#### 声明常见的Java repositories

Gradle如何知道在哪里可以找到外部依赖项的文件？ Gradle在存储库中查找它们。 存储库是模块的集合，按组，名称和版本组织。 Gradle了解不同的存储库类型，例如Maven和Ivy，并支持通过HTTP或其他协议访问存储库的各种方法。

默认情况下，Gradle不定义任何存储库。 在使用模块依赖项之前，需要在Project.repositories {}的帮助下定义至少一个。 一种选择是使用Maven Central存储库：

```
//build.gradle
repositories {
    mavenCentral()
}
```

您还可以在本地文件系统上拥有存储库。 这适用于Maven和Ivy存储库。

```
//build.gradle
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

 一个项目可以有多个存储库。 Gradle将按照指定的顺序在每个存储库中查找依赖项，并在包含所请求模块的第一个存储库中停止。

**发布artifacts**

依赖关系配置也用于发布文件。 Gradle调用这些文件发布工件，或者通常只是工件。 作为用户，您需要告诉Gradle在哪里发布工件。 您可以通过为uploadArchives任务声明存储库来完成此操作。 以下是发布到Maven存储库的示例：

```
//build.gradle
plugins {
    id 'maven'
}

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

现在，当您运行gradle uploadArchives时，gradle将构建JAR文件，生成.pom文件并上载工件。

### 声明存储库

#### 声明一个公开可用的存储库

构建软件的组织可能希望利用公共二进制存储库来下载和使用开源依赖项。 流行的公共存储库包括Maven Central，Bintray JCenter和Google Android存储库。 Gradle为最广泛使用的存储库提供内置的快捷方法。

![1.png](https://i.loli.net/2019/05/12/5cd7fccd69b57.png)

要将JCenter声明为存储库，请将此代码添加到构建脚本中：

```
//build.gradle
repositories {
    jcenter()
}
```

 在底层，Gradle从快捷方法定义的公共存储库的相应URL解析依赖关系。 所有快捷方法都可以通过RepositoryHandler API获得。 或者，您可以拼出存储库的URL以进行更细粒度的控制。

####  按URL声明自定义存储库

大多数企业项目都设置了仅在Intranet中可用的二进制存储库。 内部存储库使团队能够发布内部二进制文件，设置用户管理和安全措施，并确保正常运行时间和可用性。 如果要声明不太流行但可公开使用的存储库，则指定自定义URL也很有用。

添加以下代码以声明可通过自定义URL访问的内部存储库。

```
//build.gradle
repositories {
    maven {
      url 'http://repo.mycompany.com/maven2'
    }
}
```

通过调用RepositoryHandler API上提供的相应方法，可以将具有自定义URL的存储库指定为Maven或Ivy存储库。 Gradle支持除http或https之外的其他协议作为自定义URL的一部分，例如 文件，sftp或s3。 有关完整报道，请参阅支持的传输协议的参考手册。

您还可以使用ivy {}存储库定义自己的存储库布局，因为它们在存储库中如何组织模块方面非常灵活。

####  声明多个存储库

您可以定义多个存储库以解决依赖关系。 如果某些依赖项仅在一个存储库中可用而在另一个存储库中不可用，则声明多个存储库很有用。 您可以混合参考部分中描述的任何类型的存储库。

此示例演示如何为项目声明各种快捷方式和自定义URL存储库：

```
//build.gradle
repositories {
    jcenter()
    maven {
        url "https://maven.springframework.org/release"
    }
    maven {
        url "https://maven.restlet.com"
    }
}
```

声明顺序决定了Gradle如何在运行时检查依赖项。 如果Gradle在特定存储库中找到模块描述符，它将尝试从同一存储库下载该模块的所有工件。

#### 将存储库与依赖项匹配

Gradle公开API以声明存储库可能包含或不包含的内容。 它有不同的用例：

-  性能，当您知道在特定存储库中永远不会找到依赖项时

-  安全性，避免泄漏私有项目中使用的依赖项

-  可靠性，当某些存储库包含损坏的元数据或工件时

考虑到存储库的顺序很重要，这一点更为重要。

```
//build.gradle
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
        content {
            // this repository *only* contains artifacts with group "my.company"
            includeGroup "my.company"
        }
    }
    jcenter {
        content {
            // this repository contains everything BUT artifacts with group starting with "my.company"
            excludeGroupByRegex "my\\.company.*"
        }
    }
}
```

默认情况下，存储库包含所有内容而不排除任何内容：

-  如果您声明一个包含，那么它会排除除了包含的内容之外的所有内容。

-  如果您声明一个排除，那么它包括除排除的内容之外的所有内容。

-  如果您同时声明包含和排除，则它仅包括明确包含但未排除的内容。

可以通过显式组，模块或版本严格过滤或使用正则表达式过滤。 

#### Maven存储库过滤

对于Maven存储库，通常情况下存储库将包含版本或快照。 Gradle允许您使用此DSL声明在存储库中找到哪种工件

```
//build.gradle
repositories {
    maven {
        url "http://repo.mycompany.com/releases"
        mavenContent {
            releasesOnly()
        }
    }
    maven {
        url "http://repo.mycompany.com/snapshots"
        mavenContent {
            snapshotsOnly()
        }
    }
}
```

### Repository Types

#### Flat目录存储库

某些项目可能更喜欢将依赖项存储在共享驱动器上，或者作为项目源代码的一部分而不是二进制存储库产品。 如果要将（平面）文件系统目录用作存储库，只需键入：

```
//build.gradle
repositories {
    flatDir {
        dirs 'lib'
    }
    flatDir {
        dirs 'lib1', 'lib2'
    }
}
```

 这会添加存储库，这些存储库会查找一个或多个目录以查找依赖项。 请注意，此类型的存储库不支持任何元数据格式，如Ivy XML或Maven POM文件。 相反，Gradle将根据工件的存在动态生成模块描述符（没有任何依赖性信息）。 但是，由于Gradle更喜欢使用其描述符是从真实元数据创建而不是生成的模块，因此不能使用平面目录存储库来覆盖来自其他存储库的真实元数据的工件。 例如，如果Gradle在平面目录存储库中仅找到jmxri-1.2.1.jar，而在另一个支持元数据的存储库中找到jmxri-1.2.1.pom，则它将使用第二个存储库来提供模块。

对于使用本地覆盖远程工件的用例，请考虑使用Ivy或Maven存储库，而不是其URL指向本地目录。 如果仅使用平面目录存储库，则无需设置依赖关系的所有属性。

####  Maven Central存储库

Maven Central是托管开源库的流行存储库，供Java项目使用。

要为您的构建声明中央Maven存储库，请将其添加到您的脚本中：

```
//build.gradle
repositories {
    mavenCentral()
}
```

####  JCenter Maven存储库

Bintray的JCenter是所有流行的Maven OSS工件的最新集合，包括直接发布给Bintray的工件。

要声明JCenter Maven存储库，请将其添加到构建脚本中：

```
//build.gradle
repositories {
    jcenter()
}
```

#### Google Maven存储库

Google存储库托管Android特定的工件，包括Android SDK。 要声明Google Maven存储库，请将其添加到构建脚本中：

```
repositories {
    google()
}
```

#### 本地Maven存储库

Gradle可以使用本地Maven存储库中可用的依赖项。 声明此存储库对于使用一个项目发布到本地Maven存储库并在另一个项目中使用Gradle的工件的团队是有益的。

 Gradle将已解析的依赖项存储在自己的缓存中。 即使您从基于Maven的远程存储库中解析依赖关系，构建也不需要声明本地Maven存储库

```
repositories {
    mavenLocal()
}

```

 Gradle使用与Maven相同的逻辑来标识本地Maven缓存的位置。 如果在settings.xml中定义了本地存储库位置，则将使用此位置。 USER_HOME `/.m2`中的settings.xml优先于M2_HOME /conf中的settings.xml。 如果没有settings.xml可用，Gradle将使用默认位置`USER_HOME/.m2/repository`。

#### 自定义Maven存储库

许多组织在只能在公司网络中访问的内部Maven存储库中托管依赖项。 Gradle可以通过URL声明Maven存储库。

要添加自定义Maven存储库，您可以执行以下操作：

```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

 有时，存储库会将POM发布到一个位置，并在另一个位置发布JAR和其他工件。 要定义这样的存储库，您可以：

```
repositories {
    maven {
        // Look for POMs and artifacts, such as JARs, here
        url "http://repo2.mycompany.com/maven2"
        // Look for artifacts here if not found at the above location
        artifactUrls "http://repo.mycompany.com/jars"
        artifactUrls "http://repo.mycompany.com/jars2"
    }
}
```

 Gradle将查看POM和JAR的第一个URL。 如果在那里找不到JAR，则工件URL用于查找JAR。

#### 自定义Ivy存储库

组织可能决定在内部Ivy存储库中托管依赖项。 Gradle可以通过URL声明Ivy存储库。

使用标准布局定义Ivy存储库

要使用标准布局声明Ivy存储库，不需要额外的自定义。 您只需声明URL即可。

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

 为Ivy存储库定义命名布局

您可以使用命名布局指定存储库符合Ivy或Maven默认布局。

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        layout "maven"
    }
}
```

 为存储库指定的每个工件或常春藤都会添加其他模式以供使用。 模式按照定义的顺序使用。

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        patternLayout {
            artifact "3rd-party-artifacts/[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"
            artifact "company-artifacts/[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"
            ivy "ivy-files/[organisation]/[module]/[revision]/ivy.xml"
        }
    }
}
```

 可选地，具有模式布局的存储库可以使其“组织”部分以Maven样式布局，正斜杠将点替换为分隔符。 例如，组织my.company将表示为`my/company`。

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
        patternLayout {
            artifact "[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"
            m2compatible = true
        }
    }
}
```

访问受密码保护的常春藤存储库

您可以为基本身份验证保护的Ivy存储库指定凭据。

```
repositories {
    ivy {
        url "http://repo.mycompany.com"
        credentials {
            username "user"
            password "password"
        }
    }
}
```

#### 支持的元数据源

在存储库中搜索模块时，默认情况下，Gradle会检查该存储库中支持的元数据文件格式。 在Maven存储库中，Gradle查找.pom文件，在常春藤存储库中查找ivy.xml文件，在平面目录存储库中，它直接查找.jar文件，因为它不期望任何元数据。 从5.0开始，Gradle还会查找.module（Gradle模块元数据）文件。

但是，如果定义自定义存储库，则可能需要配置此行为。 例如，您可以定义不包含.pom文件的Maven存储库，但只能定义jar。 为此，您可以为任何存储库配置元数据源。

```
repositories {
    maven {
        url "http://repo.mycompany.com/repo"
        metadataSources {
            mavenPom()
            artifact()
        }
    }
}
```

您可以指定多个源来告诉Gradle继续查找是否找不到文件。 在这种情况下，预定义检查源的顺序。

支持以下元数据源：

| Metadata source    | Description                     | Order | Maven | Ivy / flat dir |
| ------------------ | ------------------------------- | ----- | ----- | -------------- |
| `gradleMetadata()` | Look for Gradle `.module` files | 1st   | yes   | yes            |
| `mavenPom()`       | Look for Maven `.pom` files     | 2nd   | yes   | yes            |
| `ivyDescriptor()`  | Look for `ivy.xml` files        | 2nd   | no    | yes            |
| `artifact()`       | Look directly for artifact      | 3rd   | yes   | yes            |

#### 支持的存储库传输协议

Maven和Ivy存储库支持使用各种传输协议。 目前支持以下协议：

| Type    | Credential types                                             |
| ------- | ------------------------------------------------------------ |
| `file`  | none                                                         |
| `http`  | username/password                                            |
| `https` | username/password                                            |
| `sftp`  | username/password                                            |
| `s3`    | access key/secret key/session token or Environment variables |
| `gcs`   | default application credentials sourced from well known files, Environment variables etc. |

绝不应以纯文本格式将用户名和密码作为构建文件的一部分检入版本控制。 您可以将凭据存储在本地gradle.properties文件中，并使用其中一个开源Gradle插件来加密和使用凭据，例如 [凭证插件](https://plugins.gradle.org/plugin/nu.studer.credentials)。

传输协议是存储库的URL定义的一部分。 以下构建脚本演示了如何创建基于HTTP的Maven和Ivy存储库：

```groovy
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }

    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

下面的示例展示了如何声明SFTP存储库

```groovy
repositories {
    maven {
        url "sftp://repo.mycompany.com:22/maven2"
        credentials {
            username "user"
            password "password"
        }
    }

    ivy {
        url "sftp://repo.mycompany.com:22/repo"
        credentials {
            username "user"
            password "password"
        }
    }
}
```

 使用AWS S3支持的存储库时，您需要使用AwsCredentials进行身份验证，提供访问密钥和私钥。 以下示例显示如何声明S3支持的存储库并提供AWS凭据：

```groovy
repositories {
    maven {
        url "s3://myCompanyBucket/maven2"
        credentials(AwsCredentials) {
            accessKey "someKey"
            secretKey "someSecret"
            // optional
            sessionToken "someSTSToken"
        }
    }

    ivy {
        url "s3://myCompanyBucket/ivyrepo"
        credentials(AwsCredentials) {
            accessKey "someKey"
            secretKey "someSecret"
            // optional
            sessionToken "someSTSToken"
        }
    }
}
```

 您还可以使用AwsImAuthentication将所有凭据委派给AWS sdk。 以下示例显示了如何：

```groovy
repositories {
    maven {
        url "s3://myCompanyBucket/maven2"
        authentication {
           awsIm(AwsImAuthentication) // load from EC2 role or env var
        }
    }

    ivy {
        url "s3://myCompanyBucket/ivyrepo"
        authentication {
           awsIm(AwsImAuthentication)
        }
    }
}
```

 使用Google云端存储支持的存储库时，将使用默认应用程序凭据，无需进一步配置：

```groovy
repositories {
    maven {
        url "gcs://myCompanyBucket/maven2"
    }

    ivy {
        url "gcs://myCompanyBucket/ivyrepo"
    }
}
```

 **S3配置属性**

以下系统属性可用于配置与s3存储库的交互：

org.gradle.s3.endpoint： 用于在使用非AWS，S3 API兼容的存储服务时覆盖AWS S3端点。

org.gradle.s3.maxErrorRetry： 指定在S3服务器使用HTTP 5xx状态代码响应时重试请求的最大次数。 未指定时，使用默认值3。

**S3 URL格式**

S3 URL是“虚拟托管样式”，必须采用以下格式

```
s3://<bucketName>[.<regionSpecificEndpoint>]/<s3Key>


e.g. s3://myBucket.s3.eu-central-1.amazonaws.com/maven/release
```

- myBucket是AWS S3存储桶名称。

- s3.eu-central-1.amazonaws.com是可选的区域特定端点。

- / maven / release是AWS S3密钥（存储桶中对象的唯一标识符）

**S3代理设置**

可以使用以下系统属性配置S3的代理：

- `https.proxyHost`
- `https.proxyPort`
- `https.proxyUser`
- `https.proxyPassword`
- `http.nonProxyHosts`

 如果使用http（而非https）URI指定了“org.gradle.s3.endpoint”属性，则可以使用以下系统代理设置：

- `http.proxyHost`
- `http.proxyPort`
- `http.proxyUser`
- `http.proxyPassword`
- `http.nonProxyHosts`

**AWS S3 V4签名（AWS4-HMAC-SHA256）**

某些AWS S3区域（eu-central-1 - Frankfurt）要求所有HTTP请求都根据AWS的签名版本4进行签名。建议在使用需要V4签名的存储桶时指定包含区域特定端点的S3 URL。 例如

```
s3://somebucket.s3.eu-central-1.amazonaws.com/maven/release
```

 **AWS S3跨账户访问**

某些组织可能拥有多个AWS账户，例如 每个团队一个。 存储桶拥有者的AWS账户通常与工件发布者和消费者不同。 存储桶拥有者需要能够授予消费者访问权限，否则工件只能由发布者的帐户使用。 这是通过将bucket-owner-full-control Canned ACL添加到上载的对象来完成的。 Gradle将在每次上传时执行此操作。 确保发布者具有所需的IAM权限，PutObjectAcl（如果启用了桶版本，则为PutObjectVersionAcl），可以直接或通过假定的IAM角色（具体取决于您的情况）。 您可以在AWS S3 Access权限中阅读更多内容。

**Google云端存储配置属性**

以下系统属性可用于配置与Google云存储存储库的交互：

org.gradle.gcs.endpoint

 用于在使用非Google Cloud Platform，兼容Google云端存储API的存储服务时覆盖Google云端存储终端。

org.gradle.gcs.servicePath

 用于覆盖Google云端存储客户端构建请求的Google云端存储根服务路径，默认为/。

 **Google云端存储网址格式**

 Google云端存储网址是“虚拟托管式”，必须采用以下格式：gcs：// <存储桶名称> / <对象密钥>

e.g. `gcs://myBucket/maven/release`

myBucket是Google云端存储桶名称。

/ maven / release是Google Cloud Storage密钥（存储桶中对象的唯一标识符）

 **配置HTTP身份验证方案**

使用HTTP或HTTPS传输协议配置存储库时，可以使用多种身份验证方案。 默认情况下，Gradle将尝试使用此处记录的Apache HttpClient库支持的所有方案。 在某些情况下，最好明确指定在与远程服务器交换凭据时应使用哪些身份验证方案。 显式声明时，仅在对远程存储库进行身份验证时使用这些方案。

您可以使用api：org.gradle.api.credentials.PasswordCredentials []为基本身份验证保护的Maven存储库指定凭据。

```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
        credentials {
            username "user"
            password "password"
        }
    }
}
```

下面的示例演示如何配置存储库，使其只使用api:org.gradle.authentication.http.DigestAuthentication

```groovy
repositories {
    maven {
        url 'https://repo.mycompany.com/maven2'
        credentials {
            username "user"
            password "password"
        }
        authentication {
            digest(DigestAuthentication)
        }
    }
}
```

 目前支持的认证方案是：

BasicAuthentication:HTTP上的基本访问身份验证。 使用此方案时，会先发送凭据。

DigestAuthentication:通过HTTP摘要访问身份验证。

HttpHeaderAuthentication: 基于任何自定义HTTP标头的身份验证，例如 私人代币，OAuth代币等

**使用抢先身份验证**

Gradle的默认行为是仅在服务器以HTTP 401响应的形式响应身份验证质询时才提交凭据。 在某些情况下，服务器将使用不同的代码进行响应（例如，对于在GitHub上托管的存储库，将返回404），从而导致依赖性解析失败。 为了解决此问题，可以抢先将凭据发送到服务器。 要启用抢先身份验证，只需将存储库配置为显式使用BasicAuthentication方案：

```groovy
repositories {
    maven {
        url 'https://repo.mycompany.com/maven2'
        credentials {
            username "user"
            password "password"
        }
        authentication {
            basic(BasicAuthentication)
        }
    }
}
```

 **使用HTTP头身份验证**

您可以使用api：org.gradle.api.credentials.HttpHeaderCredentials []和api：org.gradle.authentication.http.HttpHeaderAuthentication []为需要令牌，OAuth2或其他基于HTTP头的身份验证的安全Maven存储库指定任何HTTP头.

```groovy
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
        credentials(HttpHeaderCredentials) {
            name = "Private-Token"
            value = "TOKEN"
        }
        authentication {
            header(HttpHeaderAuthentication)
        }
    }
}
```

