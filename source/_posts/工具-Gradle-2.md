---

title: Gradle 构建Java库
date: 2019-05-06 18:18:59
tags:
 - 工具
 - Gradle
categories:
 - 工具
 - Gradle
---

### 构建Java库

#### 你要建造什么

您将使用标准布局生成Java库。

#### 你需要什么

-  大约11分钟
-  文本编辑器或IDE
-  Java Development Kit（JDK）版本8或更高版本
-  Gradle分发版，5.4.1或更高版本

<!--more-->

####  创建一个库项目

Gradle附带一个名为Build Init插件的内置插件。 它在Gradle用户手册中有记录。 该插件有一个名为init的任务，用于生成项目。 init任务调用（也是内置的）包装器任务来创建Gradle包装器脚本gradlew。

第一步是为新项目创建一个文件夹并将目录更改为该文件夹。

```
$ mkdir building-java-libraries
$ cd building-java-libraries
```

####  运行init任务

从新项目目录中，使用java-library参数运行init任务。

```
> gradle init --type java-library --project-name jvm-library --dsl groovy --test-framework  junit
```

init任务首先运行包装器任务，生成gradlew和gradlew.bat包装器脚本。 然后它使用以下结构创建新项目：

```bash
$ tree
.
├── build.gradle
├── gradle
│   └── wrapper #生成包装文件的文件夹
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │   ├── java # 默认Java源文件夹 
    │   │   └── jvm
    │   │       └── library
    │   │           └── Library.java
    │   └── resources
    └── test
        ├── java # 默认Java测试文件夹
        │   └── jvm
        │       └── library
        │           └── LibraryTest.java
        └── resources

13 directories, 8 files
```

您现在拥有了一个简单Java库项目的必要组件。

#### 查看生成的项目文件

设置文件有很多注释，但只有一个活动行：

生成的 settings.gradle

```groovy
rootProject.name = 'building-java-libraries'  //这将分配根项目的名称。
```

生成的build.gradle文件也有很多注释。 此处重现活动部分（注意依赖项的版本号可能会在更高版本的Gradle中更新）：

生成的  build.gradle

```groovy
plugins {
    id 'java-library'
}

repositories {
    jcenter() // Public Bintray Artifactory存储库
}

dependencies {
    // 这是导出到使用者的依赖关系的一个示例，也就是说在他们的编译类路径中找到。
    api 'org.apache.commons:commons-math3:3.6.1' 
	// 这是在内部使用的依赖关系的示例，而不是在自己的编译类路径上向使用者公开。
    implementation 'com.google.guava:guava:27.0.1-jre' 
	// JUnit测试库
    testImplementation 'junit:junit:4.12' 
}
```

构建脚本添加了java-library插件。 这是java-base插件的扩展，并添加了用于编译Java源代码的其他任务。

 文件`src/main/java/jvm/library/Library.java`如下所示：

```java


package jvm.library;

public class Library {
    public boolean someLibraryMethod() {
        return true;
    }
}
```

生成的JUnit测试类，`src/test/java/jvm/library/LibraryTest.java`如下所示：

```java
package jvm.library;

import org.junit.Test;
import static org.junit.Assert.*;

public class LibraryTest {
    @Test public void testSomeLibraryMethod() {
        Library classUnderTest = new Library();
        assertTrue("someLibraryMethod should return 'true'", classUnderTest.someLibraryMethod());
    }
}
```

####  组装库JAR

要构建项目，请运行构建任务。 您可以使用常规gradle命令，但是当项目包含包装器脚本时，它被认为是使用它的好形式。

```
$ ./gradlew build
> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes
> Task :jar
> Task :assemble
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> Task :check
> Task :build

BUILD SUCCESSFUL in 10s
4 actionable tasks: 4 executed
```

 第一次运行包装器脚本gradlew时，在下载该gradle版本并将其本地存储在`～/ .gradle / wrapper / dists`文件夹中时可能会有延迟。

 第一次运行构建时，Gradle将检查您的~/缓存中是否已经有JUnit库和其他列出的依赖项。格雷尔目录。如果没有，这些库将被下载并存储在那里。下次运行构建时，将使用缓存版本。构建任务编译类、运行测试并生成测试报告。

您可以通过打开位于`build/reports/tests/test/index.HTML`的HTML输出文件来查看测试报告。

![](https://i.loli.net/2019/05/12/5cd7ec943bbd6.png)

您可以在`build/libs`目录中找到名为`jvm-library.jar`的新打包的JAR文件。请通过运行以下命令验证归档文件是否有效:

```
$ jar tf build/libs/jvm-library.jar 
META-INF/
META-INF/MANIFEST.MF
jvm/
jvm/library/
jvm/library/Library.class

```

 您应该看到所需的清单文件-MANIFEST.MF-和已编译的Library类。

所有这些都在构建脚本中没有任何其他配置的情况下发生，因为Gradle的java库插件假定您的项目源是按照传统的项目布局排列的。 如果您愿意，可以按照用户手册中的说明自定义项目布局。

 恭喜，您刚刚完成了创建Java库的第一步！ 您现在可以根据自己的项目需求进行自定义。

#### 自定义库JAR

 您通常希望JAR文件的名称包含库版本。 通过在构建脚本中设置顶级版本属性可以轻松实现这一点，如下所示：

build.gradle

```groovy
version = '0.1.0'
```

现在运行jar任务：

```
$ ./gradlew jar
```

请注意，生成的JAR文件在`build/libs/jvm-library-0.1.0.jar`中包含了预期的版本。

另一个常见的需求是定制清单文件，通常是通过添加一个或多个属性。让我们通过配置jar任务将库名称和版本包含在清单文件中。将以下内容添加到构建脚本的末尾:

```groovy
jar {
    manifest {
        attributes('Implementation-Title': project.name,
                   'Implementation-Version': project.version)
    }
}
```

 要确认这些更改按预期工作，请再次运行jar任务，这次也从JAR解压缩清单文件：

```
$ ./gradlew jar
$ jar xf build/libs/jvm-library-0.1.0.jar META-INF/MANIFEST.MF
```

现在查看META-INF / MANIFEST.MF文件的内容，您应该看到以下内容：

```
Manifest-Version: 1.0
Implementation-Title: jvm-library
Implementation-Version: 0.1.0
```

 现在，您可以通过尝试编译一些使用您刚构建的库的Java代码来完成此练习。

#### 添加API文档

`java-library`插件通过javadoc任务内置了对Java API文档工具的支持。

Build Init插件生成的代码已经对`Library.java`文件发表了评论。 修改注释以成为javadoc标记。

**src/main/java/Library.java**

```java
package jvm.library;

public class Library {
    public boolean someLibraryMethod() {
        return true;
    }
}
```

运行javadoc任务。

```
$ ./gradlew javadoc
```

通过打开位于`build/docs/javadoc/index.html`的HTML文件，可以查看生成的javadoc文件

####  总结

 您现在已经成功构建了一个Java库项目，将其打包为JAR并在单独的应用程序中使用它。 一路上，你已经学会了如何：

-  生成Java库

-  调整生成的build.gradle和示例Java文件的结构

-  运行构建并查看测试报告

-  自定义JAR文件的名称及其清单的内容

-  生成API文档。

### java库插件

 Java Library插件通过提供有关Java库的特定知识来扩展Java插件的功能。 特别是，Java库向消费者公开API（即，使用Java或Java库插件的其他项目）。 使用此插件时，Java插件公开的所有源集，任务和配置都是隐式可用的。

 **用法**

要使用Java Library插件，请在构建脚本中包含以下内容：

build.gradle

```
plugins {
    id 'java-library'
}
```

**API和实现分离**

标准Java插件和Java Library插件之间的主要区别在于后者引入了向消费者公开的API的概念。 库是一个Java组件，旨在供其他组件使用。 这是多项目构建中非常常见的用例，但只要您有外部依赖项。

该插件公开了两种可用于声明依赖关系的配置：api和实现。 api配置应该用于声明由库API导出的依赖项，而实现配置应该用于声明组件内部的依赖项。

build.gradle

```
dependencies {
    api 'org.apache.httpcomponents:httpclient:4.5.7'
    implementation 'org.apache.commons:commons-lang3:3.5'
}
```

出现在api配置中的依赖关系将传递给库的消费者，因此将出现在消费者的编译类路径中。 另一方面，在实现配置中找到的依赖关系不会暴露给消费者，因此不会泄漏到消费者的编译类路径中。 这有几个好处：

- 依赖关系不再泄漏到消费者的编译类路径中，因此您永远不会意外地依赖于传递依赖

- 由于减少了类路径大小，编译速度更快

- 当实现依赖性发生变化时，重新编译的次数减少：消费者不需要重新编译

- 清理发布：当与新的maven-publish插件结合使用时，Java库会生成POM文件，这些文件准确区分编译库所需的内容以及在运行时使用库所需的内容（换句话说，不要 混合编译库本身所需的内容以及编译库所需的内容。

编译配置仍然存在但不应使用，因为它不会提供api和实现配置提供的保证。

如果您的构建使用带有POM元数据的已发布模块，则Java和Java库插件都会通过pom中使用的作用域来尊重api和实现分离。 这意味着编译类路径仅包含编译范围的依赖项，而运行时类路径也添加了运行时范围的依赖项。

这通常不会对使用Maven发布的模块产生影响，其中定义项目的POM直接作为元数据发布。 在那里，编译范围包括编译项目所需的依赖关系（即实现依赖关系）和针对已发布库编译所需的依赖关系（即API依赖关系）。 对于大多数已发布的库，这意味着所有依赖项都属于编译范围。 但是，如上所述，如果使用Gradle发布库，则生成的POM文件仅将api依赖项放入编译范围，将剩余的实现依赖项放入运行时范围。

默认情况下，Gradle 5.0+中的模块的编译和运行时范围是分开的。 在Gradle 4.6+中，您需要通过在settings.gradle中添加enableFeaturePreview（'IMPROVED_POM_SUPPORT'）来激活它。

#### 识别API和实现依赖性

本节将使用简单的经验法则帮助您识别代码中的API和实现依赖项。 第一个是：

-  在可能的情况下，首选api上的实现配置

这使依赖关系不再依赖于使用者的编译类路径。 此外，如果任何实现类型意外泄漏到公共API中，消费者将立即无法编译。

那么什么时候应该使用api配置？ API依赖关系是包含至少一种在库二进制接口中公开的类型的依赖关系，通常称为其ABI（应用程序二进制接口）。 这包括但不限于：

-  超类或接口中使用的类型

-  公共方法参数中使用的类型，包括泛型参数类型（其中public是编译器可见的东西。即，Java世界中的public，protected和package私有成员）

-  公共领域中使用的类型

-  公共注释类型

相比之下，以下列表中使用的任何类型都与ABI无关，因此应声明为实现依赖项：

-  方法体中专门使用的类型

-  私人会员专用的类型

-  内部类中唯一的类型（Gradle的未来版本将允许您声明哪些包属于公共API）

下面的类使用了几个第三方库，其中一个在类的公共API中公开，另一个仅在内部使用。 import语句无法帮助我们确定哪个是哪个，因此我们必须查看字段，构造函数和方法：

**示例：区分API和实现**

```java
// The following types can appear anywhere in the code
// but say nothing about API or implementation usage
import org.apache.commons.lang3.exception.ExceptionUtils;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.HttpStatus;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.UnsupportedEncodingException;

public class HttpClientWrapper {

    private final HttpClient client; // private member: implementation details

    // HttpClient is used as a parameter of a public method
    // so "leaks" into the public API of this component
    public HttpClientWrapper(HttpClient client) {
        this.client = client;
    }

    // public methods belongs to your API
    public byte[] doRawGet(String url) {
        HttpGet request = new HttpGet(url);
        try {
            HttpEntity entity = doGet(request);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            entity.writeTo(baos);
            return baos.toByteArray();
        } catch (Exception e) {
            ExceptionUtils.rethrow(e); // this dependency is internal only
        } finally {
            request.releaseConnection();
        }
        return null;
    }

    // HttpGet and HttpEntity are used in a private method, so they don't belong to the API
    private HttpEntity doGet(HttpGet get) throws Exception {
        HttpResponse response = client.execute(get);
        if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
            System.err.println("Method failed: " + response.getStatusLine());
        }
        return response.getEntity();
    }
}
```

HttpClientWrapper的公共构造函数使用HttpClient作为参数，因此它向消费者公开，因此属于API。 请注意，HttpGet和HttpEntity用于私有方法的签名，因此它们不计入使HttpClient成为API依赖项。

另一方面，来自commons-lang库的ExceptionUtils类型仅用于方法体（不在其签名中），因此它是一个实现依赖项。

因此，我们可以推断出httpclient是一个API依赖，而commons-lang是一个实现依赖。 此结论转换为构建脚本中的以下声明：

```
//声明API和实现依赖关系
//build.gradle
dependencies {
    api 'org.apache.httpcomponents:httpclient:4.5.7'
    implementation 'org.apache.commons:commons-lang3:3.5'
}
```

#### Java库插件配置

下图描述了使用Java Library插件时的主要配置设置。

![2.png](https://i.loli.net/2019/05/12/5cd7f6d132595.png)

-  绿色配置是用户用来声明依赖关系的配置

-  粉色的配置是组件编译或运行库时使用的配置

-  蓝色配置是组件内部的，供自己使用

-  白色配置是从Java插件继承的配置

下一个图描述了测试配置设置

![](https://i.loli.net/2019/05/12/5cd7f70d55c33.png)

从Java插件继承的compile，testCompile，runtime和testRuntime配置仍然可用，但已弃用。 您应该避免使用它们，因为它们仅用于向后兼容。

每个配置的角色在下面的表中描述:

Java Library插件 - 用于声明依赖关系的配置

| 配置名称             | 角色                   | 可消费的 | 可分解的 | 描述                                                         |
| -------------------- | ---------------------- | -------- | -------- | ------------------------------------------------------------ |
| `api`                | 宣布API依赖关系        | 没有     | 没有     | 这就是你应该声明依赖性的轨迹上出口到消费者,编译。            |
| `implementation`     | 宣布实现依赖关系       | 没有     | 没有     | 这就是你应该声明依赖性,它纯粹是内部和不是为了暴露在消费者。  |
| `compileOnly`        | 声明只编译依赖关系     | 是的     | 是的     | 这就是你应该声明依赖性仅需要在编译时,但不应该泄漏到运行时。这通常包括在运行时发现阴影的依赖关系。 |
| `runtimeOnly`        | 宣布运行时依赖关系     | 没有     | 没有     | 这就是你应该声明依赖性仅需要在运行时,而不是在编译时。        |
| `testImplementation` | 测试依赖关系           | 没有     | 没有     | 这就是你应该声明依赖用于编译测试。                           |
| `testCompileOnly`    | 宣布测试编译只依赖关系 | 是的     | 是的     | 这是您应该声明仅在测试编译时需要的依赖项，但不应泄漏到运行时。 这通常包括在运行时找到时被着色的依赖项。 |
| `testRuntimeOnly`    | 宣布测试运行时依赖     | 没有     | 没有     | 这就是你应该声明依赖性仅需要在测试运行时,而不是在编译时进行测试。 |

消费者使用的Java库插件配置:

| 配置名称          | 角色         | 可消费的 | 可分解的 | 描述                                                         |
| ----------------- | ------------ | -------- | -------- | ------------------------------------------------------------ |
| `apiElements`     | 对这个库编译 | 是的     | 没有     | 这个配置是使用的消费者,获取所有必要的元素对这个库编译。不像` default`配置,这并不泄漏实现或运行时依赖关系。 |
| `runtimeElements` | 为执行这个库 | 是的     | 没有     | 这个配置是使用的消费者,获取所有必要的元素对这个库运行。      |

Java Library插件 - 库本身使用的配置:

| 配置名称             | 角色               | 可消费的? | 可分解的? | 描述                                                         |
| -------------------- | ------------------ | --------- | --------- | ------------------------------------------------------------ |
| compileClasspath     | 来编译这个库       | 没有      | 是的      | 该配置包含编译这个库的类路径中,因此使用时调用java编译器来编译它。 |
| runtimeClasspath     | 为执行这个库       | 没有      | 是的      | 该配置包含这个库的运行时类路径                               |
| testCompileClasspath | 来编译这个库的测试 | 没有      | 是的      | 该配置包含测试编译这个库的类路径。                           |
| testRuntimeClasspath | 这个库的执行测试   | 没有      | 是的      | 该配置包含这个库的测试运行时类路径                           |

#### 已知的问题

**与其他插件的兼容性**

目前，Java Library插件通过java，groovy和kotlin插件正确运行。 在Gradle 5.3或更早版本中，某些插件（如Groovy插件）可能无法正常运行。 特别是，如果除了java-library插件之外还使用了Groovy插件，那么消费者在使用库时可能无法获得Groovy类。 要解决此问题，您需要显式连接Groovy编译依赖项，如下所示：

在Gradle 5.3或更早的版本中配置Groovy插件来使用Java库:

```
configurations {
    apiElements {
        outgoing.variants.getByName('classes').artifact(
            file: compileGroovy.destinationDir,
            type: ArtifactTypeDefinition.JVM_CLASS_DIRECTORY,
            builtBy: compileGroovy)
    }
}
```

**增加消费者的内存使用量**

当项目使用Java Library插件时，如果项目使用Java插件，则消费者将直接在其编译类路径上使用此项目的输出类目录，而不是jar文件。 间接的结果是，最新的检查将需要更多的内存，因为Gradle将对单个类文件而不是单个jar进行快照。 这可能会导致大型项目的内存消耗增加。





### 参考

1. [Gradle学习](https://blog.csdn.net/lastsweetop/column/info/18566)
2. [Gradle构建多模块项目](https://www.cnblogs.com/heart-king/p/5909225.html)
3. [Gradle复合构建](https://segmentfault.com/a/1190000010129596)