---
title: Gradle 范例
date: 2019-05-08 11:18:59
tags:
 - 工具
 - Gradle
categories:
 - 工具
 - Gradle
---

### 前言

本章会通过一个案例来进行说明Gradle的使用：一个To Do应用程序。先是一个没有界面的控制台应用，再使用基于Web的java项目来说明。

<!--more-->

To Do帮助您来记录您的计划。

所有代码在[这里](https://github.com/hanyunpeng0521/ToDo)。

### 应用程序

使用To Do来实现典型的增删改查（CRUD）功能。

其中的主要类如下，包括类之间的交互。

![有序图.png](https://i.loli.net/2019/05/11/5cd65f88b9531.png)

新建项目文件夹：

```
>mkdir ToDos
>cd ToDos
```

创建Gradle项目：

```
>gradle init

Select type of project to generate:
  1: basic
  2: cpp-application
  3: cpp-library
  4: groovy-application
  5: groovy-library
  6: java-application
  7: java-library
  8: kotlin-application
  9: kotlin-library
  10: scala-library
Enter selection (default: basic) [1..10] 6

Select build script DSL:
  1: groovy
  2: kotlin
Enter selection (default: groovy) [1..2] 1

Select test framework:
  1: junit
  2: testng
  3: spock
Enter selection (default: junit) [1..3] 1

Project name (default: ToDos): 
Source package (default: ToDos): com.hyp.learn

BUILD SUCCESSFUL in 20s
2 actionable tasks: 2 executed
```

使用tree查看项目结构：

```bash
> tree
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── hyp
    │   │           └── learn
    │   │               ├── model
    │   │               │   └── ToDoItem.java
    │   │               ├── repository
    │   │               │   ├── InMemoryToDoRepository.java
    │   │               │   └── ToDoRepository.java
    │   │               ├── ToDoApp.java
    │   │               └── utils
    │   │                   ├── CommandLineInputHandler.java
    │   │                   └── CommandLineInput.java
    │   └── resources
    └── test
        ├── java
        │   └── com
        │       └── hyp
        │           └── learn
        └── resources

```



TO Do 模型类,位于src/main/java/com/hyp/learn/model文件夹下：

```java
package com.hyp.learn.model;
//ToDo模型类
public class ToDoItem implements Comparable<ToDoItem> {
    private Long id;
    private String name;
    private boolean completed;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isCompleted() {
        return completed;
    }

    public void setCompleted(boolean completed) {
        this.completed = completed;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        ToDoItem that = (ToDoItem) o;

        if (completed != that.completed) return false;
        if (id != null ? !id.equals(that.id) : that.id != null) return false;
        if (name != null ? !name.equals(that.name) : that.name != null) return false;

        return true;
    }

    @Override
    public int hashCode() {
        int result = id != null ? id.hashCode() : 0;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        result = 31 * result + (completed ? 1 : 0);
        return result;
    }

    @Override
    public int compareTo(ToDoItem toDoItem) {
        return this.getId().compareTo(toDoItem.getId());
    }

    @Override
    public String toString() {
        return id + ": " + name + " [completed: " + completed + "]";
    }
}
```

仓库类接口，位于src/main/java/com/hyp/learn/repository文件夹下：

```java
package com.hyp.learn.repository;
import com.hyp.learn.model.ToDoItem;
import java.util.List;
//CURD接口
public interface ToDoRepository {
    List<ToDoItem> findAll();

    ToDoItem findById(Long id);

    Long insert(ToDoItem toDoItem);

    void update(ToDoItem toDoItem);

    void delete(ToDoItem toDoItem);
}
```

仓库类接口实现类，位于src/main/java/com/hyp/learn/repository文件夹下：

```java
package com.hyp.learn.repository;

import com.hyp.learn.model.ToDoItem;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.atomic.AtomicLong;
//CURD实现类，
public class InMemoryToDoRepository implements ToDoRepository {
    private AtomicLong currentId = new AtomicLong();//线程安全的标识符序列号
    private ConcurrentMap<Long, ToDoItem> toDos = new ConcurrentHashMap<Long, ToDoItem>();//保存ToDos项目的数据

    @Override
    public List<ToDoItem> findAll() {
        List<ToDoItem> toDoItems = new ArrayList<ToDoItem>(toDos.values());
        Collections.sort(toDoItems);//排序
        return toDoItems;
    }

    @Override
    public ToDoItem findById(Long id) {
        return toDos.get(id);
    }

    @Override
    public Long insert(ToDoItem toDoItem) {
        Long id = currentId.incrementAndGet();
        toDoItem.setId(id);
        toDos.putIfAbsent(id, toDoItem);//插入项目，如果不存在
        return id;
    }

    @Override
    public void update(ToDoItem toDoItem) {
        toDos.replace(toDoItem.getId(), toDoItem);//如果存在则替换到项目
    }

    @Override
    public void delete(ToDoItem toDoItem) {
        toDos.remove(toDoItem.getId());//如果存在就移除。
    }
}
```

通用工具类命令行输入类CommandLineInput，位于src/main/java/com/hyp/learn/utils文件夹下：

```
package com.hyp.learn.utils;

import java.util.HashMap;
import java.util.Map;
//枚举类，用户可选择的操作
public enum CommandLineInput {
    FIND_ALL('a'), FIND_BY_ID('f'), INSERT('i'), UPDATE('u'), DELETE('d'), EXIT('e');

    private final static Map<Character, CommandLineInput> INPUTS;

    static {
        INPUTS = new HashMap<Character, CommandLineInput>();

        for (CommandLineInput input : values()) {
            INPUTS.put(input.getShortCmd(), input);
        }
    }

    private final char shortCmd;

    private CommandLineInput(char shortCmd) {
        this.shortCmd = shortCmd;
    }

    public char getShortCmd() {
        return shortCmd;
    }

    public static CommandLineInput getCommandLineInputForInput(char input) {
        return INPUTS.get(input);
    }
}
```

通用工具类命令行输入处理类CommandLineInputHandler，位于src/main/java/com/hyp/learn/utils文件夹下：

```java
package com.hyp.learn.utils;


import com.hyp.learn.model.ToDoItem;
import com.hyp.learn.repository.InMemoryToDoRepository;
import com.hyp.learn.repository.ToDoRepository;

import java.util.Collection;
//处理用户输入
public class CommandLineInputHandler {
    private ToDoRepository toDoRepository = new InMemoryToDoRepository();

    public void printOptions() {
        System.out.println("\n--- To Do Application ---");
        System.out.println("Please make a choice:");
        System.out.println("(a)ll items");
        System.out.println("(f)ind a specific item");
        System.out.println("(i)nsert a new item");
        System.out.println("(u)pdate an existing item");
        System.out.println("(d)elete an existing item");
        System.out.println("(e)xit");
    }

    public String readInput() {
        return System.console().readLine("> ");
    }

    public void processInput(CommandLineInput input) {
        if (input == null) {
            handleUnknownInput();
        } else {
            switch (input) {
                case FIND_ALL:
                    printAllToDoItems();
                    break;
                case FIND_BY_ID:
                    printToDoItem();
                    break;
                case INSERT:
                    insertToDoItem();
                    break;
                case UPDATE:
                    updateToDoItem();
                    break;
                case DELETE:
                    deleteToDoItem();
                    break;
                case EXIT:
                    break;
                default:
                    handleUnknownInput();
            }
        }
    }

    private Long askForItemId() {
        System.out.println("Please enter the item ID:");
        String input = readInput();
        return Long.parseLong(input);
    }

    private ToDoItem askForNewToDoAction() {
        ToDoItem toDoItem = new ToDoItem();
        System.out.println("Please enter the name of the item:");
        toDoItem.setName(readInput());
        return toDoItem;
    }

    private void printAllToDoItems() {
        Collection<ToDoItem> toDoItems = toDoRepository.findAll();

        if (toDoItems.isEmpty()) {
            System.out.println("Nothing to do. Go relax!");
        } else {
            for (ToDoItem toDoItem : toDoItems) {
                System.out.println(toDoItem);
            }
        }
    }

    private void printToDoItem() {
        ToDoItem toDoItem = findToDoItem();

        if (toDoItem != null) {
            System.out.println(toDoItem);
        }
    }

    private ToDoItem findToDoItem() {
        Long id = askForItemId();
        ToDoItem toDoItem = toDoRepository.findById(id);

        if (toDoItem == null) {
            System.err.println("To do item with ID " + id + " could not be found.");
        }

        return toDoItem;
    }

    private void insertToDoItem() {
        ToDoItem toDoItem = askForNewToDoAction();
        Long id = toDoRepository.insert(toDoItem);
        System.out.println("Successfully inserted to do item with ID " + id + ".");
    }

    private void updateToDoItem() {
        ToDoItem toDoItem = findToDoItem();

        if (toDoItem != null) {
            System.out.println(toDoItem);
            System.out.println("Please enter the name of the item:");
            toDoItem.setName(readInput());
            System.out.println("Please enter the done status the item:");
            toDoItem.setCompleted(Boolean.parseBoolean(readInput()));
            toDoRepository.update(toDoItem);
            System.out.println("Successfully updated to do item with ID " + toDoItem.getId() + ".");
        }
    }

    private void deleteToDoItem() {
        ToDoItem toDoItem = findToDoItem();

        if (toDoItem != null) {
            toDoRepository.delete(toDoItem);
            System.out.println("Successfully deleted to do item with ID " + toDoItem.getId() + ".");
        }
    }

    private void handleUnknownInput() {
        System.out.println("Please select a valid option!");
    }
}
```

应用程序入口ToDoApp类，位于src/main/java/com/hyp/learn文件夹下

```java
package com.hyp.learn;
import com.hyp.learn.utils.CommandLineInput;
import com.hyp.learn.utils.CommandLineInputHandler;

public class ToDoApp {
    public static final char DEFAULT_INPUT = '\u0000';

    public static void main(String args[]) {
        CommandLineInputHandler commandLineInputHandler = new CommandLineInputHandler();
        char command = DEFAULT_INPUT;

        while (CommandLineInput.EXIT.getShortCmd() != command) {
            commandLineInputHandler.printOptions();
            String input = commandLineInputHandler.readInput();
            char[] inputChars = input.length() == 1 ? input.toCharArray() : new char[]{DEFAULT_INPUT};
            command = inputChars[0];
            CommandLineInput commandLineInput =
                    CommandLineInput.getCommandLineInputForInput(command);
            commandLineInputHandler.processInput(commandLineInput);
        }
    }
}
```

上面就是主要的代码，下面将讲述构建java项目。

#### 构建java项目

在根目录下配置文件build.gradle的内容如下：

```groovy
//依赖插件
plugins {
    // 使用 Java插件
    id 'java'

    // 支持生成应用
    id 'application'
}

//组名
group 'com.hyp.learn'
version '1.0'  //项目版本

sourceCompatibility = 1.8 //java编译兼容1.8

repositories {//声明仓库
	mavenLocal()//Maven本地仓库
	mavenCentral()//Maven远程仓库
}

//外部依赖
dependencies {
    // This dependency is found on compile classpath of this component and consumers.
    implementation 'com.google.guava:guava:27.0.1-jre'

    // Use JUnit test framework
    testImplementation 'junit:junit:4.12'
}

// 为应用程序声明主类
mainClassName = 'com.hyp.learn.ToDoApp'

jar {
    manifest {//将MainClass添加到jar文件代码清单中
        attributes 'Main-Class': 'com.hyp.learn.ToDoApp'
    }
    //from configuration.compile.collect { zipTree it}//将引用的包打进 jar 包
}//可以运行jar包：java -jar <name>.jar


sourceSets {
    main {
        java{
            srcDirs= ['src/main/java'] //源代码目录
        }
    }
    test {
        java {
            srcDirs = ['src/test/java']//测试代码目录
        }
    }
}
buildDir = 'out' //项目输出属性
```

构建项目,在根目录下执行：

```bash
$ ./gradlew build -i

> Configure project :
Evaluating root project 'ToDos' using build file '/home/hyp/programer/IdeaProjects/ToDos/build.gradle'.
All projects evaluated.
Selected primary task 'build' from project :
Tasks to be executed: [task ':compileJava', task ':processResources', task ':classes', task ':jar', task ':startScripts', task ':distTar', task ':distZip', task ':assemble', task ':compileTestJava', task ':processTestResources', task ':testClasses', task ':test', task ':check', task ':build']
:compileJava (Thread[Execution worker for ':',5,main]) started.

> Task :compileJava #编译java代码
> Task :processResources #操作资源
> Task :classes #生成class文件
> Task :jar  #构建jar文件
> Task :startScripts  #执行脚本
> Task :distTar #生成发行的tar文件
> Task :distZip #生成发行的zip文件
> Task :assemble  #合并
> Task :compileTestJava #编译java测试代码
> Task :processTestResources #操作测试资源
> Task :testClasses #生成class测试文件
> Task :test #执行测试示例
> Task :check #检查
> Task :build #构建

# 被标记为UP-TO-DATE消息的任务被跳过，构建文件位于out文件夹下
```

运行主类：

```bash
$ java -cp out/classes/java/main com.hyp.learn.ToDoApp

--- To Do Application ---
Please make a choice:
(a)ll items
(f)ind a specific item
(i)nsert a new item
(u)pdate an existing item
(d)elete an existing item
(e)xit
> e
```



### Web开发

一个war文件被用来绑定Web组件、编译class文件以及其他资源文件，将该war包部署到Web容器中即可运行。Gradle提供开箱即用的插件，用来组装War文件和将Web应用部署到本地Servlet容器中。

![请求.png](https://i.loli.net/2019/05/11/5cd65f889d386.png)

在src/main/java/com/hyp/learn/web文件夹下添加文件ToDoServlet类

```
package com.hyp.learn.web;


import com.hyp.learn.model.ToDoItem;
import com.hyp.learn.repository.InMemoryToDoRepository;
import com.hyp.learn.repository.ToDoRepository;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class ToDoServlet extends HttpServlet {
    public static final String FIND_ALL_SERVLET_PATH = "/all";
    public static final String INDEX_PAGE = "/jsp/todo-list.jsp";
    private ToDoRepository toDoRepository = new InMemoryToDoRepository();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String servletPath = request.getServletPath();
        String view = processRequest(servletPath, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(view);
        dispatcher.forward(request, response);
    }

    private String processRequest(String servletPath, HttpServletRequest request) {
        if (servletPath.equals(FIND_ALL_SERVLET_PATH)) {
            List<ToDoItem> toDoItems = toDoRepository.findAll();
            request.setAttribute("toDoItems", toDoItems);
            request.setAttribute("stats", determineStats(toDoItems));
            request.setAttribute("filter", "all");
            return INDEX_PAGE;
        } else if (servletPath.equals("/active")) {
            List<ToDoItem> toDoItems = toDoRepository.findAll();
            request.setAttribute("toDoItems", filterBasedOnStatus(toDoItems, true));
            request.setAttribute("stats", determineStats(toDoItems));
            request.setAttribute("filter", "active");
            return INDEX_PAGE;
        } else if (servletPath.equals("/completed")) {
            List<ToDoItem> toDoItems = toDoRepository.findAll();
            request.setAttribute("toDoItems", filterBasedOnStatus(toDoItems, false));
            request.setAttribute("stats", determineStats(toDoItems));
            request.setAttribute("filter", "completed");
            return INDEX_PAGE;
        }
        if (servletPath.equals("/insert")) {
            ToDoItem toDoItem = new ToDoItem();
            toDoItem.setName(request.getParameter("name"));
            toDoRepository.insert(toDoItem);
            return "/" + request.getParameter("filter");
        } else if (servletPath.equals("/update")) {
            ToDoItem toDoItem = toDoRepository.findById(Long.parseLong(request.getParameter("id")));

            if (toDoItem != null) {
                toDoItem.setName(request.getParameter("name"));
                toDoRepository.update(toDoItem);
            }

            return "/" + request.getParameter("filter");
        } else if (servletPath.equals("/delete")) {
            ToDoItem toDoItem = toDoRepository.findById(Long.parseLong(request.getParameter("id")));

            if (toDoItem != null) {
                toDoRepository.delete(toDoItem);
            }

            return "/" + request.getParameter("filter");
        } else if (servletPath.equals("/toggleStatus")) {
            ToDoItem toDoItem = toDoRepository.findById(Long.parseLong(request.getParameter("id")));

            if (toDoItem != null) {
                boolean completed = "on".equals(request.getParameter("toggle")) ? true : false;
                toDoItem.setCompleted(completed);
                toDoRepository.update(toDoItem);
            }

            return "/" + request.getParameter("filter");
        } else if (servletPath.equals("/clearCompleted")) {
            List<ToDoItem> toDoItems = toDoRepository.findAll();

            for (ToDoItem toDoItem : toDoItems) {
                if (toDoItem.isCompleted()) {
                    toDoRepository.delete(toDoItem);
                }
            }

            return "/" + request.getParameter("filter");
        }

        return FIND_ALL_SERVLET_PATH;
    }

    private List<ToDoItem> filterBasedOnStatus(List<ToDoItem> toDoItems, boolean active) {
        List<ToDoItem> filteredToDoItems = new ArrayList<ToDoItem>();

        for (ToDoItem toDoItem : toDoItems) {
            if (toDoItem.isCompleted() != active) {
                filteredToDoItems.add(toDoItem);
            }
        }

        return filteredToDoItems;
    }

    private ToDoListStats determineStats(List<ToDoItem> toDoItems) {
        ToDoListStats toDoListStats = new ToDoListStats();

        for (ToDoItem toDoItem : toDoItems) {
            if (toDoItem.isCompleted()) {
                toDoListStats.addCompleted();
            } else {
                toDoListStats.addActive();
            }
        }

        return toDoListStats;
    }

    public class ToDoListStats {
        private int active;
        private int completed;

        private void addActive() {
            active++;
        }

        private void addCompleted() {
            completed++;
        }

        public int getActive() {
            return active;
        }

        public int getCompleted() {
            return completed;
        }

        public int getAll() {
            return active + completed;
        }
    }
}
```

在根目录下配置文件build.gradle的内容如下：

```
plugins {
    // 使用 Java插件
    id 'java'

	//web插件
    id 'war'
	//嵌入式jetty容器
    id "org.akhikhl.gretty" version "2.0.0"

    // 支持生成应用
    id 'application'
}


group 'com.hyp.learn'
version '1.0'  //项目版本

sourceCompatibility = 1.8 //java编译兼容1.8

repositories {//声明仓库
	mavenLocal()//Maven本地仓库
	mavenCentral()//Maven远程仓库
}

dependencies {
    // This dependency is found on compile classpath of this component and consumers.
    implementation 'com.google.guava:guava:27.0.1-jre'

    // Use JUnit test framework
    testImplementation 'junit:junit:4.12'
    providedCompile 'javax.servlet:servlet-api:2.5',
            'javax.servlet.jsp:jsp-api:2.1'
    runtime 'javax.servlet:jstl:1.1.2',
            'taglibs:standard:1.1.2'
}

// 为应用程序声明主类
mainClassName = 'com.hyp.learn.ToDoApp'

jar {
    manifest {//将MainClass添加到jar文件代码清单中
        attributes 'Main-Class': 'com.hyp.learn.ToDoApp'
    }
    //from configuration.compile.collect { zipTree it}//将引用的包打进 jar 包
}//可以运行jar包：java -jar <name>.jar


sourceSets {
    main {
        java{
            srcDirs= ['src/main/java'] //源代码目录
        }
    }
    test {
        java {
            srcDirs = ['src/test/java']//测试代码目录
        }
    }
}
buildDir = 'out' //项目输出属性

webAppDirName = 'webfiles' //web app 的根目录

war { //指定war的属性
    from 'static' //指定资源的目录
}
```

构建项目：

```
$ ./gradlew build -i
# 在assemble任务前有war任务执行
```

执行web项目：

```
$ ./gradlew jettyRun

```



