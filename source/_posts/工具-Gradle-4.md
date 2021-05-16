---
title: Gradle Project
date: 2019-05-06 11:18:59
tags:
 - 工具
 - Gradle
categories:
 - 工具
 - Gradle
---

### Project

这个接口是用于从构建文件与Gradle交互的主要API。从项目中，您可以通过编程访问Gradle的所有特性。

<!--more-->

####  生命周期

Project和build.gradle文件之间存在一对一的关系。 在构建初始化期间，Gradle为每个参与构建的项目组装一个Project对象，如下所示：

-  为构建创建一个Settings实例。

-  根据Settings对象评估settings.gradle脚本（如果存在）以进行配置。

-  使用配置的Settings对象创建Project实例的层次结构。

-  最后，通过对项目执行build.gradle文件（如果存在）来评估每个项目。 项目按广度顺序进行评估，以便在项目之前评估项目。 可以通过调用Project.evaluationDependsOnChildren（）或使用Project.evaluationDependsOn（java.lang.String）添加显式评估依赖项来覆盖此顺序。

####  任务

项目本质上是Task对象的集合。 每个任务执行一些基本工作，例如编译类，运行单元测试或压缩WAR文件。 您可以使用TaskContainer上的某个create（）方法向项目添加任务，例如TaskContainer.create（java.lang.String）。 您可以使用TaskContainer上的查找方法之一找到现有任务，例如TaskCollection.getByName（java.lang.String）。

#### 依赖

项目通常具有许多依赖性，以便完成其工作。 此外，项目通常会生成许多工件，其他项目可以使用。 这些依赖项按配置分组，可以从存储库中检索和上载。 您使用Project.getConfigurations（）方法返回的ConfigurationContainer来管理配置。 Project.getDependencies（）方法返回的DependencyHandler来管理依赖项。 Project.getArtifacts（）方法返回ArtifactHandler来管理工件。 Project.getRepositories（）方法返回的RepositoryHandler来管理存储库。

#### 多项目构建

项目按层次排列为项目。 项目具有名称和完全限定的路径，该路径在层次结构中唯一标识它。

#### 插件

插件可用于模块化和重用项目配置。 可以使用PluginAware.apply（java.util.Map）方法或使用PluginDependenciesSpec插件脚本块应用插件。

#### 属性

Gradle针对Project实例执行项目的构建文件以配置项目。 脚本使用的任何属性或方法都会委托给关联的Project对象。 这意味着，您可以直接在脚本中使用Project接口上的任何方法和属性。

```
defaultTasks('some-task')  // Delegates to Project.defaultTasks()
reportsDir = file('reports') // Delegates to Project.file() and the Java Plugin
```

您还可以使用project属性访问Project实例。 这可以使脚本在某些情况下更清晰。 例如，您可以使用project.name而不是name来访问项目的名称。

一个项目有5个属性“范围”，它搜索属性。 您可以在构建文件中按名称访问这些属性，也可以通过调用项目的Project.property（java.lang.String）方法来访问这些属性。 范围是：

- Project对象本身。 此范围包括Project实现类声明的任何属性getter和setter。 例如，Project.getRootProject（）可作为rootProject属性访问。 此范围的属性是可读写的，具体取决于是否存在相应的getter或setter方法。

- 项目的额外属性。 每个项目都维护一个额外属性的映射，它可以包含任意名称 - >值对。 定义后，此作用域的属性是可读写的。 有关详细信息，请参阅其他属性

- 插件添加到项目中的扩展。 每个扩展名都可以作为只读属性使用，其名称与扩展名相同。

- 插件添加到项目中的约定属性。 插件可以通过项目的Convention对象向项目添加属性和方法。 此范围的属性可以是可读的或可写的，具体取决于约定对象。

- 项目的任务。 可以使用其名称作为属性名称来访问任务。 此范围的属性是只读的。 例如，一个名为compile的任务可以作为编译属性访问。

在读取属性时，项目按顺序搜索上述范围，并从找到该属性的第一个范围返回值。如果未找到，则抛出异常。Project.property（java.lang.String）。

在编写属性时，项目按顺序搜索上述范围，并在找到该属性的第一个范围内设置该属性。如果未找到，则抛出异常。 Project.setProperty（java.lang.String，java.lang.Object）。

#### 额外的属性

必须通过“ext”命名空间定义所有额外属性。 一旦定义了额外的属性，它就可以直接在拥有对象上获得（在下面的例子中分别是Project，Task和子项目），并且可以读取和更新。 只需要通过命名空间完成的初始声明。

```
project.ext.prop1 = "foo"
task doStuff {
    ext.prop2 = "bar"
}
subprojects { ext.${prop3} = false }
```

获取额外的属性是通过“ext”或通过拥有对象。

```
ext.isSnapshot = version.endsWith("-SNAPSHOT")
if (isSnapshot) {
    // do snapshot stuff
}
```

```

```

#### 动态方法

 一个项目有5个方法'范围'，它搜索方法：

-  Project对象本身。

-  构建文件。 该项目搜索构建文件中声明的匹配方法。

-  插件添加到项目中的扩展。 每个扩展都可用作将闭包或Action作为参数的方法。

-  插件添加到项目中的约定方法。 插件可以通过项目的Convention对象向项目添加属性和方法。

-  项目的任务。 为每个任务添加一个方法，使用任务名称作为方法名称并使用单个闭包或Action参数。 该方法使用提供的闭包为相关任务调用Task.configure（groovy.lang.Closure）方法。 例如，如果项目有一个名为compile的任务，则会添加一个带有以下签名的方法：void compile（Closure configureClosure）。

-  父项目的方法，递归到根项目。

-  项目的属性，其值为闭包。 闭包被视为一种方法，并使用提供的参数进行调用。

参考：[Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:dependencies(groovy.lang.Closure))



### Tasks

Gradle提供了几个内置的任务，通过这些任务可以查看构建的详细信息。可以帮助理解构建的依赖结构，debug错误。

系统默认设置的task：

1. Build tasks
   - assemble - 组装此项目的输出
   - build - 组装和测试此项目
   - clean - 删除build项目.
2. Build Setup tasks
   - init - 初始化一个新的Gradle项目
   - wrapper - 生成Gradle包装文件
3. Custom tasks
   - copy - 将源复制到dest目录
4. Help tasks
   - buildEnvironment -  列出选定项目构建脚本需要的依赖。 
   - components -  显示根项目生成的组件。  
   - dependencies -  会根据配置列出选定项目的依赖，针对每个配置，直接依赖和传递依赖都会被列出。 
   - dependencyInsight -  显示对根项目中特定依赖项的深入了解。 
   - dependentComponents -  显示根项目中组件的依赖组件。 
   - help -  可以列出所选任务的详细信息，如果有多个任务匹配给出的名字，那么将多个任务同时列出：`./gradlew help --task <task>`
   - model -  显示根项目的配置模型。 
   - projects -  可以查看所选项目及子项目的列表信息，结果以层级结构输出。 
   - properties -  可以列出选定项目的所有的属性。 
   - tasks -  可以查看所选项目的主要任务列表，显示方式和项目列表一样。 
5. Verification tasks
   - check -  执行所有检查。 

#### Init 插件

Gradle Build Init插件可用于引导创建新Gradle构建的过程。 它支持创建不同类型的全新项目，以及将现有构建（例如，Apache Apache Maven构建）转换为Gradle构建。

Gradle插件通常需要在可以使用之前应用于项目（请参阅使用插件）。  Build  Init插件是一个自动应用的插件，这意味着您不需要显式应用它。 要使用该插件，只需执行名为init的任务，即可在其中创建Gradle构建。  无需创建“存根”build.gradle文件即可应用插件。

它还利用包装器任务为项目生成Gradle Wrapper文件。

**任务**：init、wrapper

init支持不同的构建设置类型。 通过提供--type参数值来指定类型。 例如，要创建Java库项目，只需执行：`gradle init --type java-library`。

如果未提供--type参数，Gradle将尝试从环境中推断出类型。 例如，如果找到要转换为Gradle构建的pom.xml，它将推断类型值“pom”。

如果无法推断类型，将使用“基本”类型。

init插件还支持使用Gradle Groovy DSL或Gradle Kotlin DSL生成构建脚本。 要使用的构建脚本DSL默认为Groovy DSL，并通过提供--dsl参数值来指定。 例如，要使用Kotlin DSL构建脚本创建Java库项目，只需执行：`gradle init --type java-library --dsl kotlin`。

所有构建设置类型都包括Gradle Wrapper的设置。

请注意，从Maven构建版本的迁移仅支持用于生成的构建脚本的Groovy DSL。

 **pom（Maven转换）**

“pom”类型可用于将Apache Maven构建转换为Gradle构建。 这可以通过将POM转换为一个或多个Gradle文件来实现。 只有在调用init任务的目录中有一个有效的“pom.xml”文件时，或者，如果通过“-p”命令行选项在指定的项目目录中调用，它才能被使用。 如果存在这样的文件，将自动推断出这种“pom”类型。

Maven转换实现的灵感来自最初由Gradle社区成员开发的maven2gradle工具。

转换过程具有以下功能：

- 使用有效的POM和有效设置（支持POM继承，依赖关系管理，属性）
- 支持单模块和多模块项目
- 支持自定义模块名称（与目录名称不同）
- 生成常规元数据 - id，描述和版本
- 应用maven，java和war插件（根据需要）
- 如果需要，支持将战争项目打包为罐子
- 生成依赖项（外部和模块间）
- 生成下载存储库（包括本地Maven存储库）
- 调整Java编译器设置
- 支持包装和测试
- 支持TestNG跑步者
- 从Maven enforcer插件设置生成全局排除项

**java-application**

“java-application”构建init类型是不可推断的。 必须明确指定。

它具有以下功能：

- 使用“application”插件生成使用Java实现的命令行应用程序
- 使用“jcenter”依赖库
- 使用JUnit进行测试
- 在常规位置具有源代码的目录
- 如果没有现有的源文件或测试文件，则包含样本类和单元测试

可以通过提供--test-framework参数值来指定备用测试框架。 要使用其他测试框架，请执行以下命令之一：

```
 gradle init --type java-application --test-framework spock：使用Spock进行测试而不是JUnit

 gradle init --type java-application --test-framework testng：使用TestNG代替JUnit进行测试
```

**java-library**

“java-library”构建init类型是不可推断的。 必须明确指定。

它具有以下功能：

- 使用“java”插件生成一个库Jar
- 使用“jcenter”依赖库
- 使用JUnit进行测试
- 在常规位置具有源代码的目录
- 如果没有现有的源文件或测试文件，则包含样本类和单元测试

可以通过提供--test-framework参数值来指定备用测试框架。 要使用其他测试框架，请执行以下命令之一：

```
 gradle init --type java-library --test-framework spock：使用Spock代替JUnit进行测试

 gradle init --type java-library --test-framework testng：使用TestNG代替JUnit进行测试
```

**scala-library**

“scala-library”构建init类型是不可推断的。 必须明确指定。

它具有以下功能：

- 使用“scala”插件生成一个库Jar
- 使用“jcenter”依赖库
- 使用Scala 2.10
- 使用ScalaTest进行测试
- 在常规位置具有源代码的目录
- 如果没有现有的源或测试文件，则包含示例scala类和关联的ScalaTest测试套件
- 默认情况下使用Zinc Scala编译器

**groovy-library**

“groovy-library”构建init类型是不可推断的。 必须明确指定。

它具有以下功能：

- 使用“groovy”插件生成一个库Jar
- 使用“jcenter”依赖库
- 使用Groovy 2.x.
- 使用Spock测试框架进行测试
- 在常规位置具有源代码的目录
- 如果没有现有的源文件或测试文件，则包含示例Groovy类和关联的Spock规范

**groovy-application**

“groovy-application”构建init类型是不可推断的。 必须明确指定。

它具有以下功能：

- 使用“groovy”插件
- 使用“application”插件生成使用Groovy实现的命令行应用程序
- 使用“jcenter”依赖库
- 使用Groovy 2.x.
- 使用Spock测试框架进行测试
- 在常规位置具有源代码的目录
- 如果没有现有的源文件或测试文件，则包含示例Groovy类和关联的Spock规范

**basic**

“basic”构建init类型对于创建全新的Gradle项目非常有用。 它创建了一个示例build.gradle文件，其中包含注释和链接以帮助您入门。

如果未明确指定类型，则使用此类型，并且不能推断任何类型。