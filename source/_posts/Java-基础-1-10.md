---
title: Java  反射和动态代理
date: 2019-04-06 13:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---

要想理解反射的原理，首先要了解什么是类型信息。Java让我们在运行时识别对象和类的信息，主要有2种方式：一种是传统的RTTI(Runtime Type Information)运行时类型信息，它假定我们在编译时已经知道了所有的类型信息；另一种是反射机制，它允许我们在运行时发现和使用类的信息。

<!--more-->

[浅谈 RTTI](https://www.jianshu.com/p/b7fea01bb64f)

**用途**

Java反射机制主要提供了以下功能：

**在运行时构造一个类的对象；判断一个类所具有的成员变量和方法；调用一个对象的方法；生成动态代理。反射最大的应用就是框架**

在日常的第三方应用开发过程中，经常会遇到某个类的某个成员变量、方法或是属性是私有的或是只对系统应用开放，这时候就可以利用Java的反射机制通过反射来获取所需的私有成员或是方法。当然，也不是所有的都适合反射，之前就遇到一个案例，通过反射得到的结果与预期不符。阅读源码发现，经过层层调用后在最终返回结果的地方对应用的权限进行了校验，对于没有权限的应用返回值是没有意义的缺省值，否则返回实际值起到保护用户的隐私目的。

反射的应用很多，很多框架都有用到

```
spring 的 ioc/di 也是反射....
javaBean和jsp之间调用也是反射....
struts的 FormBean 和页面之间...也是通过反射调用....
JDBC 的 classForName()也是反射.....
hibernate的 find(Class clazz) 也是反射....
```
反射还有一个不得不说的问题，就是性能问题，大量使用反射系统性能大打折扣。

### 相关类

Java 反射机制是在运行状态中，对于任意一个类，都能够获得这个类的所有属性和方法，对于任意一个对象都能够调用它的任意一个属性和方法。这种在运行时动态的获取信息以及动态调用对象的方法的功能称为Java 的反射机制。

Class 类与java.lang.reflect 类库一起对反射的概念进行了支持，该类库包含了Field,Method,Constructor类(每个类都实现了Member 接口)。这些类型的对象时由JVM 在运行时创建的，用以表示未知类里对应的成员。

这样你就可以使用Constructor 创建新的对象，用get() 和set() 方法读取和修改与Field  对象关联的字段，用invoke() 方法调用与Method 对象关联的方法。另外，还可以调用getFields() getMethods() 和  getConstructors()  等很便利的方法，以返回表示字段，方法，以及构造器的对象的数组。这样匿名对象的信息就能在运行时被完全确定下来，而在编译时不需要知道任何事情。

与Java反射相关的类如下：

![](https://i.loli.net/2019/04/13/5cb1747abe004.png)


#### Class

理解RTTI在Java中的工作原理，首先需要知道类型信息在运行时是如何表示的，这是由Class对象来完成的，它包含了与类有关的信息。Class对象就是用来创建所有“常规”对象的，Java使用Class对象来执行RTTI，即使你正在执行的是类似类型转换这样的操作。

Class是一个类，封装了当前对象所对应的类的信息：

1. 一个类中有属性，方法，构造器等，比如说有一个Person类，一个Order类，一个Book类，这些都是不同的类，现在需要一个类，用来描述类，这就是Class，它应该有类名，属性，方法，构造器等。Class是用来描述类的类

2. Class类是一个对象照镜子的结果，对象可以看到自己有哪些属性，方法，构造器，实现了哪些接口等等

3. 对于每个类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个类的有关信息。

4. Class 对象只能由系统建立对象，一个类（而不是一个对象）在 JVM 中只会有一个Class实例

每个类都会产生一个对应的Class对象，也就是保存在.class文件。所有类都是在对其第一次使用时，动态加载到JVM的，当程序创建一个对类的静态成员的引用时，就会加载这个类。Class对象仅在需要的时候才会加载，static初始化是在类加载时进行的。

```（java）
public class TestMain {
    public static void main(String[] args) {
        System.out.println(XYZ.name);
    }
}

class XYZ {
    public static String name = "luoxn28";

    static {
        System.out.println("xyz静态块");
    }

    public XYZ() {
        System.out.println("xyz构造了");
    }
}
```
运行结果：

```
$ java TestMain
xyz静态块
luoxn28
```
类加载器首先会检查这个类的Class对象是否已被加载过，如果尚未加载，默认的类加载器就会根据类名查找对应的.class文件。

想在运行时使用类型信息，必须获取对象(比如类Base对象)的Class对象的引用，使用功能Class.forName(“Base”)可以实现该目的，或者使用base.class。注意，有一点很有趣，**使用功能”.class”来创建Class对象的引用时，不会自动初始化该Class对象，使用forName()会自动初始化该Class对象**。为了使用类而做的准备工作一般有以下3个步骤：

 - 加载：由类加载器完成，找到对应的字节码，创建一个Class对象
 - 链接：验证类中的字节码，为静态域分配空间
 - 初始化：如果该类有超类，则对其初始化，执行静态初始化器和静态初始化块

```java
public class Base {
    static int num = 1;

    static {
        System.out.println("Base " + num);
    }
}
public class Main {
    public static void main(String[] args) {
        // 不会初始化静态块
        Class clazz1 = Base.class;
        System.out.println("------");
        // 会初始化
        Class clazz2 = Class.forName("zzz.Base");
    }
}
```

Class类常用方法：

**获得类相关的方法**

![](https://i.loli.net/2019/04/13/5cb1756d7b24e.png)

**获得类中属性相关的方法**

![](https://i.loli.net/2019/04/13/5cb1756d48cd6.png)

**获得类中注解相关的方法**

![](https://i.loli.net/2019/04/13/5cb1756d4c13a.png)

**获得类中构造器相关的方法**

![](https://i.loli.net/2019/04/13/5cb175dd24045.png)

**获得类中方法相关的方法**

![](https://i.loli.net/2019/04/13/5cb1765070f71.png)

**类中其他重要的方法**

![](https://i.loli.net/2019/04/13/5cb176505664b.png)


#### Field类

Field代表类的成员变量（成员变量也称为类的属性）。

![](https://i.loli.net/2019/04/13/5cb176b239540.png)


#### Method类

Method代表类的方法。

![](https://i.loli.net/2019/04/13/5cb1775b636b3.png)

#### Constructor类

Constructor代表类的构造方法。

![](https://i.loli.net/2019/04/13/5cb1775b0d7d7.png)



在阅读Class类文档时发现一个特点，以通过反射获得Method对象为例，一般会提供四种方法，getMethod(parameterTypes)、getMethods()、getDeclaredMethod(parameterTypes)和getDeclaredMethods()。getMethod(parameterTypes)用来获取某个公有的方法的对象，getMethods()获得该类所有公有的方法，getDeclaredMethod(parameterTypes)获得该类某个方法，getDeclaredMethods()获得该类所有方法。

**带有Declared修饰的方法可以反射到私有的方法，没有Declared修饰的只能用来反射公有的方法。** 其他的Annotation、Field、Constructor也是如此。

### 使用

实体类：

```java
@Resource
public class Book implements Serializable {
    private final static String TAG = "BookTag";
    @NotNull
    public  String ID;
    @NotNull
    private String name;
    @Nullable
    private String author;

    @Override
    public String toString() {
        return "Book{" +
                "name='" + name + '\'' +
                ", author='" + author + '\'' +
                '}';
    }

    public Book() {
    }

    public Book(String name, String author) {
        this.name = name;
        this.author = author;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    private String declaredMethod(int index) {
        String string = null;
        switch (index) {
            case 0:
                string = "I am declaredMethod 1 !";
                break;
            case 1:
                string = "I am declaredMethod 2 !";
                break;
            default:
                string = "I am declaredMethod 1 !";
        }
        return string;
    }
}
```

在Java 中可以通过三种方法获取类的字节码(Class)对象

- 通过Object 类中的getClass() 方法，想要用这种方法必须要明确具体的类并且创建该类的对象。
- 所有数据类型都具备一个静态的属性.class 来获取对应的Class 对象。但是还是要明确到类，然后才能调用类中的静态成员。
- 只要通过给定类的字符串名称就可以获取该类的字节码对象，这样做扩展性更强。通过Class.forName()  方法完成，必须要指定类的全限定名，由于前两种方法都是在知道该类的情况下获取该类的字节码对象，因此不会有异常，但是Class.forName()  方法如果写错类的路径会报 ClassNotFoundException 的异常。

示例：
```java
public class ReflectClass {
    private final static String TAG = "peter.log.ReflectClass";

    public static void main(String [] args)
    {
        Class clazz=null;

        //1.通过类名
        clazz=Book.class;

        //2.通过对象名
        //这种方式是用在传进来一个对象，却不知道对象类型的时候使用
        Book book=new Book();
        clazz=book.getClass();

        //上面这个例子的意义不大，因为已经知道book类型是Book类，再这样写就没有必要了
        //如果传进来是一个Object类，这种做法就是应该的
        Object obj = new Book();
        clazz = obj.getClass();

        //3.通过全类名(会抛出异常)
        //一般框架开发中这种用的比较多，
        // 因为配置文件中一般配的都是全类名，通过这种方式可以得到Class实例

        String clazzName="com.hyp.learn.reflect.Book";
        try {
            clazz=Class.forName(clazzName);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

**类实例**

通过Constructor获取类实例。

```java
public class ReflectClass {
    private final static String TAG = "peter.log.ReflectClass";

    public static void main(String [] args)
    {
        Class clazz=null;
    String clazzName="com.hyp.learn.reflect.Book";
        try {
        clazz=Class.forName(clazzName);

            //1.使用Constructor获取Book类实例,绑定对应的构造函数，可以使用所有可访问的构造函数
            Constructor constructor1 = clazz.getConstructor();
            Constructor constructor2 = clazz.getConstructor(String.class, String.class);

            //只能使用绑定的构造函数
            Book book1=(Book) constructor1.newInstance();

            System.out.println(book1.toString());

            Book book2=(Book)constructor2.newInstance("Java从入门到成神","YY");

            System.out.println(book2);

            //2. 使用Class的newInstance，只能创建无参数构造函数
            Book book3=(Book)clazz.newInstance();
            System.out.println(book3.toString());


        } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：

```
Book{name='null', author='null'}
Book{name='Java从入门到成神', author='YY'}
Book{name='null', author='null'}
```

**属性**

获取类中属性。

```java
public class ReflectClass {
    private final static String TAG = "peter.log.ReflectClass";
    public static void main(String [] args)
    {
        Class clazz=null;
    String clazzName="com.hyp.learn.reflect.Book";
        try {
        clazz=Class.forName(clazzName);

        //1. 获取可访问属性
            Field[] fields = clazz.getFields();
            System.out.println("clazz.getFields");
            if (null!=fields) {
                for (Field field : fields) {
                    System.out.println(field.getModifiers() + ":" + field.getType() + ":" + field.getName());
                }
            }
            //2. 获取所有属性
            Field[] declaredFields = clazz.getDeclaredFields();
            System.out.println("clazz.getDeclaredFields");
            if (null!=declaredFields) {
                for (Field field : declaredFields) {
                    System.out.println(field.getModifiers() + ":" + field.getType() + ":" + field.getName());
                }
            }
            //3. 通过Class函数获取
          clazz.getFields();
          clazz.getDeclaredFields();
       } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
    }    
}
```
运行结果：

```
clazz.getFields
1:class java.lang.String:ID
clazz.getDeclaredFields
26:class java.lang.String:TAG
1:class java.lang.String:ID
2:class java.lang.String:name
2:class java.lang.String:author
```
**接口**

```java
public class ReflectClass {
    private final static String TAG = "peter.log.ReflectClass";
    public static void main(String [] args)
    {
        Class clazz=null;
    String clazzName="com.hyp.learn.reflect.Book";

        try {
            clazz=Class.forName(clazzName);

            Class[] interfaces = clazz.getInterfaces();

            for (Class a : interfaces) {
                System.out.println(a);
            }

            AnnotatedType[] annotatedInterfaces = clazz.getAnnotatedInterfaces();

            for (AnnotatedType anInterface : annotatedInterfaces) {
                System.out.println(anInterface.getType());
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

运行结果：

```
interface java.io.Serializable
interface java.io.Serializable
```

**注解**

```java
public class ReflectClass {
    private final static String TAG = "peter.log.ReflectClass";
    public static void main(String [] args)
    {
        Class clazz=null;
    String clazzName="com.hyp.learn.reflect.Book";

        try {
            clazz=Class.forName(clazzName);

            Annotation[] annotations = clazz.getAnnotations();

            for (Annotation annotation : annotations) {
                System.out.println(annotation.toString());
            }

            Annotation[] annotations1 = clazz.getDeclaredAnnotations();
            for (Annotation annotation : annotations1) {
                System.out.println(annotation.toString());
            }

            System.out.println(clazz.getDeclaredAnnotation(Resource.class));


        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果
```
@javax.annotation.Resource(shareable=true, lookup=, name=, description=, authenticationType=CONTAINER, type=class java.lang.Object, mappedName=)
@javax.annotation.Resource(shareable=true, lookup=, name=, description=, authenticationType=CONTAINER, type=class java.lang.Object, mappedName=)
@javax.annotation.Resource(shareable=true, lookup=, name=, description=, authenticationType=CONTAINER, type=class java.lang.Object, mappedName=)
```


**方法**

获取类中方法：

```java
public class ReflectClass {
    private final static String TAG = "peter.log.ReflectClass";
    public static void main(String [] args)
    {
        Class clazz=null;
    String clazzName="com.hyp.learn.reflect.Book";

        try {
            clazz=Class.forName(clazzName);
            Method[] methods = clazz.getMethods();
            if (null!=methods) {
                for (Method method : methods) {
                    System.out.println(method.toString());
                }
            }
            Method method = clazz.getMethod("setName", String.class);
            Book book=new Book();
            System.out.println(book);
            method.invoke(book, "hello");
            System.out.println(book);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：

```
public java.lang.String com.hyp.learn.reflect.Book.getAuthor()
public void com.hyp.learn.reflect.Book.setAuthor(java.lang.String)
public java.lang.String com.hyp.learn.reflect.Book.toString()
public java.lang.String com.hyp.learn.reflect.Book.getName()
public void com.hyp.learn.reflect.Book.setName(java.lang.String)
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
Book{name='null', author='null'}
Book{name='hello', author='null'}
```

method.invoke(owner, args)：执行该Method.invoke方法的参数是执行这个方法的对象owner（方法拥有类对象），和参数数组args（方法内参数），可以这么理解：owner对象中带有参数args的method方法。返回值是Object，也既是该方法的返回值。

invoke回调流程示例

1. 由Class对象动态构造对应类型对象

2. Class对象的getMethod方法，由方法名和形参构造Method对象

3. Method对象的invoke方法来委托动态构造的对应类型对象，使其执行对应形参的add方法，这是回调函数（方法）的功能

“回调函数就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。”

### Reflections

Reflections 通过扫描 classpath，索引元数据，允许在运行时查询这些元数据，也可以保存收集项目中多个模块的元数据信息。

使用 Reflections 可以查询以下元数据信息： 

- 获得某个类型的所有子类型
- 获得标记了某个注解的所有类型／成员变量，支持注解参数匹配。
- 使用正则表达式获得所有匹配的资源文件
- 获得所有特定签名（包括参数，参数注解，返回值）的方法

Reflections 依赖 Google 的 [Guava](http://www.oschina.net/p/guava) 库和 [Javassist](http://www.oschina.net/p/javassist) 库。

Maven 项目导入

```
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.9.10</version>
</dependency>
```

通常用法：

```java
Reflections reflections = new Reflections("my.project");

Set<Class<? extends SomeType>> subTypes = reflections.getSubTypesOf(SomeType.class);

Set<Class<?>> annotated = reflections.getTypesAnnotatedWith(SomeAnnotation.class);
```

Reflections 初始化代码。

```java
//scan urls that contain 'my.package', include inputs starting with 'my.package', use the default scanners
Reflections reflections = new Reflections("my.package");

//or using ConfigurationBuilder
new Reflections(new ConfigurationBuilder()
     .setUrls(ClasspathHelper.forPackage("my.project.prefix"))
     .setScanners(new SubTypesScanner(), 
                  new TypeAnnotationsScanner().filterResultsBy(optionalFilter), ...),
     .filterInputsBy(new FilterBuilder().includePackage("my.project.prefix"))
     ...);
```

以下是一些使用例子代码。

```java
//SubTypesScanner
Set<Class<? extends Module>> modules = 
    reflections.getSubTypesOf(com.google.inject.Module.class);
//TypeAnnotationsScanner 
Set<Class<?>> singletons = 
    reflections.getTypesAnnotatedWith(javax.inject.Singleton.class);
//ResourcesScanner
Set<String> properties = 
    reflections.getResources(Pattern.compile(".*\\.properties"));
//MethodAnnotationsScanner
Set<Method> resources =
    reflections.getMethodsAnnotatedWith(javax.ws.rs.Path.class);
Set<Constructor> injectables = 
    reflections.getConstructorsAnnotatedWith(javax.inject.Inject.class);
//FieldAnnotationsScanner
Set<Field> ids = 
    reflections.getFieldsAnnotatedWith(javax.persistence.Id.class);
//MethodParameterScanner
Set<Method> someMethods =
    reflections.getMethodsMatchParams(long.class, int.class);
Set<Method> voidMethods =
    reflections.getMethodsReturn(void.class);
Set<Method> pathParamMethods =
    reflections.getMethodsWithAnyParamAnnotated(PathParam.class);
//MethodParameterNamesScanner
List<String> parameterNames = 
    reflections.getMethodParamNames(Method.class)
//MemberUsageScanner
Set<Member> usages = 
    reflections.getMethodUsages(Method.class)
```

更多请查看[文档](http://ronmamo.github.io/reflections/index.html?org/reflections/ReflectionUtils.html)

#### 用例

以下是使用Reflections的一些有用用例：

-  多模块环境中的Bootstrap

-  收集预扫描的元数据

-  将Reflections序列化为java源文件，并使用它静态引用java元素

-  直接查询商店，避免在类加载器中定义类型

-  在类路径中查找资源（例如所有属性文件）

1. 多模块环境中的Bootstrap

   在一个多模块项目中，每个模块负责它的属性，jpa实体和guice模块，使用Reflections来收集元数据并引导应用程序

   ```java
   Reflections reflections = new Reflections(new ConfigurationBuilder() 
       .addUrls(ClasspathHelper.forPackage("your.package.here"), ClasspathHelper.forClass(Entity.class), ClasspathHelper.forClass(Module.class)) 
       .setScanners(new ResourcesScanner(), new TypeAnnotationsScanner(), new SubTypesScanner()));
   
   Set<String> propertiesFiles = reflections.getResources(Pattern.compile(".*\\.properties"));
   Properties allProperties = createOneBigProperties(propertiesFiles);
   
   Set<Class<?>> jpaEntities = reflections.getTypesAnnotatedWith(Entity.class);
   SessionFactory sessionFactory = createOneBigSessionFactory(jpaEntities, allProperties);
   
   Set<Class<? extends Module>> guiceModules = reflections.getSubTypesOf(Module.class);
   Injector injector = createOneBigInjector(guiceModules);
   ```

2. 收集预扫描的元数据

   虽然扫描可以很容易地在应用程序的引导时间完成，而且不会花很长时间，但有时在编译后将所有扫描的元数据保存到xml/json文件中是个好主意，稍后，当您的项目正在引导时，您可以让Reflections收集所有这些资源并避免扫描。

   因此，首先确保反射扫描的元数据保存到源/资源文件夹中的文件中。这可以使用您的构建工具(首选)或编程实现:

   ```
   reflections.save("src/main/resources/META-INF/reflections/resource1-reflections.xml");
   ```

   然后，在运行时，收集这些预先保存的元数据并实例化Reflections

   ```
   Reflections reflections = isProduction() ? Reflections.collect() : new Reflections("your.package.here");
   ```

3. 将Reflection序列化为Java源文件，并使用它静态引用java元素

   Reflection可以将类型和类型元素序列化为接口，分别为完全限定名称。 首先，使用保存Reflections元数据

   ```
     reflections.save(filename, new JavaCodeSerializer());
   ```

   文件名应该在模式中：`path/path/path/package.package.classname`

   保存的文件应该如下所示

   ```java
   public interface MyModel {
     public interface my {
      public interface package1 {
       public interface MyClass1 {
        public interface fields {
         public interface f1 {}
         public interface f2 {}
        }
        public interface methods {
         public interface m1 {}
         public interface m2 {}
        }
   ...
   }
   ```

   然后，您可以以静态类型的方式引用方法/字段/注释描述符:

   ```
     Class m1Ref = MyModel.my.package1.MyClass1.methods.m1.class;
   ```

   并使用JavaCodeSerializer中的帮助器方法解析为运行时元素：

   ```
    Method method = JavaCodeSerializer.resolve(m1Ref);
   ```

4. 直接查询商店，避免在类加载器中定义类型

   通过Reflection查询会导致类加载器定义的类。 这通常不是问题，但是在不希望类定义的情况下，您可以仅使用字符串直接查询商店

   ```java
   Reflections reflections = new Reflections(...);` //see in other use cases 
   Set<String> serializableFqns = reflections.getStore().getSubTypesOf("java.io.Serializable"); 
   ```

   另外，您可以通过直接查询商店来创建专门的查询方法

   ```java
   Map<String, Multimap<String, String>> storeMap = reflections.getStore().getStoreMap(); 
   //or
   Multimap<String, String> scannerMap = reflections.getStore().get(ResourcesScanner.class);
   ```

5. 在您的类路径中找到的资源

   ```java
   Reflections reflections = new Reflections(new ConfigurationBuilder() 
     .setUrls(ClasspathHelper.forPackage("your.package.here")) 
     .setScanners(new ResourcesScanner());
   
   Set<String> propertiesFiles = reflections.getResources(Pattern.compile(".*\\.properties"));
   Set<String> hibernateCfgFiles = reflections.getResources(Pattern.compile(".*\\.cfg\\.xml"));
   
   ```


### JDK动态代理

代理模式是23种设计模式的一种，他是指一个对象A通过持有另一个对象B，可以具有B同样的行为的模式。为了对外开放协议，B往往实现了一个接口，A也会去实现接口。但是B是“真正”实现类，A则比较“虚”，他借用了B的方法去实现接口的方法。A虽然是“伪军”，但它可以增强B，在调用B的方法前后都做些其他的事情。Spring AOP就是使用了动态代理完成了代码的动态“织入”。

使用代理好处还不止这些，一个工程如果依赖另一个工程给的接口，但是另一个工程的接口不稳定，经常变更协议，就可以使用一个代理，接口变更时，只需要修改代理，不需要一一修改业务代码。从这个意义上说，所有调外界的接口，我们都可以这么做，不让外界的代码对我们的代码有侵入，这叫防御式编程。代理其他的应用可能还有很多。

上述例子中，类A写死持有B，就是B的静态代理。如果A代理的对象是不确定的，就是动态代理。动态代理目前有两种常见的实现，jdk动态代理和cglib动态代理。

1. 静态代理：由程序员或者自动生成工具生成代理类，然后进行代理类的编译和运行。在代理类、委托类运行之前，代理类已经以.class的格式存在。
2. 动态代理：在程序运行时，由反射机制动态创建而成。
   - JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理
   - 使用CGLib实现动态代理，完全不受代理类必须实现接口的限制，而且CGLib底层采用ASM字节码生成框架，使用字节码技术生成代理类，比使用Java反射效率要高。**唯一需要注意的是，CGLib不能对声明为final的方法进行代理，因为CGLib原理是动态生成被代理类的子类。**

java动态代理类位于java.lang.reflect包下，一般主要涉及到以下两个类：

1. Interface InvocationHandler：该接口中仅定义了一个方法Object：invoke(Object obj,Method method, Object[] args)。在实际使用时，第一个参数obj一般是指代理 类，method是被代理的方法，如上例中的request()，args为该方法的参数数组。 这个抽 象方法在代理类中动态实现。

2. Proxy：该类即为动态代理类，作用类似于上例中的ProxySubject。

3. Protected Proxy(InvocationHandler h)：构造函数，估计用于给内部的h赋值。

4. Static Class getProxyClass (ClassLoader loader, Class[] interfaces)：获得一个 代理类，其中loader是类装载器，interfaces是真实类所拥有的全部接口的数组。

5. Static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)：返回代理类的一个实例，返回后的代理类可以当作被代理类使用 (可使用被代理类的在Subject接口中声明过的方法)。

在使用动态代理类时，我们必须实现InvocationHandler

**对于JDK 的Proxy,有以下几点：**

-   1）Interface：对于JDK proxy，业务类是需要一个Interface的，这也是一个缺陷
  
- 2）Proxy，Proxy 类是动态产生的，这个类在调用Proxy.newProxyInstance(targetCls.getClassLoader, targetCls.getInterface,InvocationHander)之后，会产生一个Proxy类的实例。实际上这个Proxy类也是存在的，不仅仅是类的实例。这个Proxy类可以保存到硬盘上。
  
- 3） Method:对于业务委托类的每个方法，现在Proxy类里面都不用静态显示出来
  
- 4） InvocationHandler: 这个类在业务委托类执行时，会先调用invoke方法。invoke方法再执行相应的代理操作，可以实现对业务方法的再包装

代码：

```java
//接口
public interface UserService {
    public String getUserByName(String name);
}
 
//实现类
public class UserServiceImpl implements UserService {
    @Override
    public String getUserByName(String name) {
        System.out.println("从数据库中查询到:" + name);
        return name;
    }
}
//代理类
public class JdkCacheHandler implements InvocationHandler {

    // 目标类对象
    private Object target;

    // 获取目标类对象
    public JdkCacheHandler(Object target) {
        this.target = target;
    }

    // 创建JDK代理
    public Object createJDKProxy() {
        Class clazz = target.getClass();
        // 创建JDK代理需要3个参数，目标类加载器、目标类接口、代理类对象(即本身)
        return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("查找数据库前，在缓存中查找是否存在:" + args[0]);
        // 触发目标类方法
        Object result = method.invoke(target, args);
        System.out.printf("查找数据库后，将%s加入到缓存中\r\n", result);
        return result;
    }
}
//测试类
public class JdkTest {

    @Test
    public void test() {
        UserService userService = new UserServiceImpl();
        JdkCacheHandler jdkCacheHandler = new JdkCacheHandler(userService);
        UserService proxy = (UserService) jdkCacheHandler.createJDKProxy();

        System.out.println("==========================");
        proxy.getUserByName("px");
        System.out.println("==========================");

        System.out.println(proxy.getClass());
    }
}
 
//输出

==========================
查找数据库前，在缓存中查找是否存在:px
从数据库中查询到:px
查找数据库后，将px加入到缓存中
==========================
class com.sun.proxy.$Proxy0
```

Proxy（jdk类库提供）根据B的接口生成一个实现类，我们称为C，它就是动态代理类（该类型是 $Proxy+数字 的“新的类型”）。生成过程是：由于拿到了接口，便可以获知接口的所有信息（主要是方法的定义），也就能声明一个新的类型去实现该接口的所有方法，这些方法显然都是“虚”的，它调用另一个对象的方法。当然这个被调用的对象不能是对象B，如果是对象B，我们就没法增强了，等于饶了一圈又回来了。

所以它调用的是B的包装类，这个包装类需要我们来实现，但是jdk给出了约束，它必须实现InvocationHandler，上述例子中就是AppleProxy，这个接口里面有个方法，它是所有Target的所有方法的调用入口（invoke），调用之前我们可以加自己的代码增强。

整个JDK动态代理的秘密也就这些，简单一句话，动态代理就是要生成一个包装类对象，由于代理的对象是动态的，所以叫动态代理。由于我们需要增强，这个增强是需要留给开发人员开发代码的，因此代理类不能直接包含被代理对象，而是一个InvocationHandler，该InvocationHandler包含被代理对象，并负责分发请求给被代理对象，分发前后均可以做增强。从原理可以看出，JDK动态代理是“对象”的代理

#### 原理分析

1. Proxy.newProxyInstance使用Proxy类的方法生成代理类

   ```java
   //   类加载器定义代理类
   //   代理类要实现的接口列表
   //	调用处理程序，将方法调用分派给InvocationHandler的invoke方法
   public static Object newProxyInstance(ClassLoader loader,
                                             Class<?>[] interfaces,
                                             InvocationHandler h)
           throws IllegalArgumentException
       {
   
           //如果h为null则抛出NUllPointerException异常
           //之后所有的判断时候为null都用此方法
           Objects.requireNonNull(h);
           // 拷贝类实现的所有接口
           final Class<?>[] intfs = interfaces.clone();
       //安全管理器
           final SecurityManager sm = System.getSecurityManager();
           if (sm != null) {
                           //检查创建Proxy类所需的权限
               checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
           }
   
           /*
            * Look up or generate the designated proxy class.
             查找或产生代理类
            */
           Class<?> cl = getProxyClass0(loader, intfs);
   
           /*
            * Invoke its constructor with the designated invocation handler.
             * 调用它的构造函数
            */
           try {
               if (sm != null) {
                   checkNewProxyPermission(Reflection.getCallerClass(), cl);
               }
   //constructorParams:
               //    private static final Class<?>[] constructorParams =
               //{ InvocationHandler.class };
               // 获取代理类的构造器对象
               final Constructor<?> cons = cl.getConstructor(constructorParams);
               final InvocationHandler ih = h;
               if (!Modifier.isPublic(cl.getModifiers())) {
                   AccessController.doPrivileged(new PrivilegedAction<Void>() {
                       public Void run() {
                           cons.setAccessible(true);
                           return null;
                       }
                   });
               }
               //根据代理类的构造器对象创建代理类对象并返回            
               return cons.newInstance(new Object[]{h});
           } catch (IllegalAccessException|InstantiationException e) {
               throw new InternalError(e.toString(), e);
           } catch (InvocationTargetException e) {
               Throwable t = e.getCause();
               if (t instanceof RuntimeException) {
                   throw (RuntimeException) t;
               } else {
                   throw new InternalError(t.toString(), t);
               }
           } catch (NoSuchMethodException e) {
               throw new InternalError(e.toString(), e);
           }
       }
   ```

   可以看到主要做了3件事情：

   - 生成代理类
   - 获取代理类的构造器对象
   - 根据构造器创建代理类对象

2. 查找或生成代理类

   ```java
   //java.lang.reflect.Proxy
   /**
        * Generate a proxy class.  Must call the checkProxyAccess method
        * to perform permission checks before calling this.
        */
       private static Class<?> getProxyClass0(ClassLoader loader,
                                              Class<?>... interfaces) {
    
           if (interfaces.length > 65535) {//接口数不能超过65535
               throw new IllegalArgumentException("interface limit exceeded");
           }
    
           // If the proxy class defined by the given loader implementing
           // the given interfaces exists, this will simply return the cached copy;
           // otherwise, it will create the proxy class via the ProxyClassFactory
           // 如果缓存命中代理类则直接返回，否则用代理类工厂ProxyClassFactory创建代理类
           return proxyClassCache.get(loader, interfaces);
       }
    
   /**
        * 缓存代理类，使用弱引用避免内存泄露
        */
       private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
           proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
   
   // java.lang.reflect.WeakCache
   
   // the key type is Object for supporting null key
   private final ConcurrentMap<Object, ConcurrentMap<Object, Supplier<V>>> map
       = new ConcurrentHashMap<>();
   
   /**
    * @param key       类加载器（可能为null）
    * @param parameter 接口数组（不能为null）
    */
   public V get(K key, P parameter) {
    
       // 在缓存中没有找到会创建新的代理类
       Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
       Supplier<V> supplier = valuesMap.get(subKey);
    
       V value = supplier.get();
       return value;
   }
   ```

3. apply方法的实现源码如下：

   ```java
   //java.lang.reflect.Proxy.ProxyClassFactory#apply
   public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
    
       Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
       // 验证
       for (Class<?> intf : interfaces) {
           Class<?> interfaceClass = null;
           try {
               interfaceClass = Class.forName(intf.getName(), false, loader);
           } catch (ClassNotFoundException e) {
           }
           if (interfaceClass != intf) {
               throw new IllegalArgumentException(
                   intf + " is not visible from class loader");
           }
           if (!interfaceClass.isInterface()) {
               throw new IllegalArgumentException(
                   interfaceClass.getName() + " is not an interface");
           }
           if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
               throw new IllegalArgumentException(
                   "repeated interface: " + interfaceClass.getName());
           }
       }
    
       String proxyPkg = null;     // 代理类所在包
       int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
    
       // 验证所有非公共的接口在同一个包内，公共的就无需处理
       for (Class<?> intf : interfaces) {
           int flags = intf.getModifiers();
           if (!Modifier.isPublic(flags)) {
               accessFlags = Modifier.FINAL;
               String name = intf.getName();
               int n = name.lastIndexOf('.');
               String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
               if (proxyPkg == null) {
                   proxyPkg = pkg;
               } else if (!pkg.equals(proxyPkg)) {
                   throw new IllegalArgumentException(
                       "non-public interfaces from different packages");
               }
           }
       }
    
       if (proxyPkg == null) {
           // if no non-public proxy interfaces, use com.sun.proxy package
           proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
       }
    
       // 生成代理类名，以$Proxy开头
       long num = nextUniqueNumber.getAndIncrement();
       String proxyName = proxyPkg + proxyClassNamePrefix + num;
    
       // 生成代理类字节码
       byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
           proxyName, interfaces, accessFlags);
       try {
           // 加载字节码，生成代理类对象
           return defineClass0(loader, proxyName,
                               proxyClassFile, 0, proxyClassFile.length);
       } catch (ClassFormatError e) {
           throw new IllegalArgumentException(e.toString());
       }
   }
   ```

4. 可以看到通过generateProxyClass方法生成代理类字节码，跟进该方法，发现又调用了generateClassFile()方法生成字节码，源码如下：

   ```java
   private byte[] generateClassFile() {
       // 添加hashCode、equals、toString方法
       this.addProxyMethod(hashCodeMethod, Object.class);
       this.addProxyMethod(equalsMethod, Object.class);
       this.addProxyMethod(toStringMethod, Object.class);
       Class[] var1 = this.interfaces;
       int var2 = var1.length;
    
       int var3;
       Class var4;
       // 添加接口中的方法
       for(var3 = 0; var3 < var2; ++var3) {
           var4 = var1[var3];
           Method[] var5 = var4.getMethods();
           int var6 = var5.length;
    
           for(int var7 = 0; var7 < var6; ++var7) {
               Method var8 = var5[var7];
               this.addProxyMethod(var8, var4);
           }
       }
    
       Iterator var11 = this.proxyMethods.values().iterator();
    
       List var12;
       while(var11.hasNext()) {
           var12 = (List)var11.next();
           checkReturnTypes(var12);
       }
    
       Iterator var15;
       try {
           // 生成代理类的构造函数
           this.methods.add(this.generateConstructor());
           var11 = this.proxyMethods.values().iterator();
    
           while(var11.hasNext()) {
               var12 = (List)var11.next();
               var15 = var12.iterator();
    
               while(var15.hasNext()) {
                   ProxyGenerator.ProxyMethod var16 = (ProxyGenerator.ProxyMethod)var15.next();
                   this.fields.add(new ProxyGenerator.FieldInfo(var16.methodFieldName, "Ljava/lang/reflect/Method;", 10));
                   this.methods.add(var16.generateMethod());
               }
           }
    
           this.methods.add(this.generateStaticInitializer());
       } catch (IOException var10) {
           throw new InternalError("unexpected I/O Exception", var10);
       }
    
       if (this.methods.size() > 65535) {
           throw new IllegalArgumentException("method limit exceeded");
       } else if (this.fields.size() > 65535) {
           throw new IllegalArgumentException("field limit exceeded");
       } else {
           // 编写最终类文件
           this.cp.getClass(dotToSlash(this.className));
           this.cp.getClass("java/lang/reflect/Proxy");
           var1 = this.interfaces;
           var2 = var1.length;
    
           for(var3 = 0; var3 < var2; ++var3) {
               var4 = var1[var3];
               this.cp.getClass(dotToSlash(var4.getName()));
           }
    
           this.cp.setReadOnly();
           ByteArrayOutputStream var13 = new ByteArrayOutputStream();
           DataOutputStream var14 = new DataOutputStream(var13);
    
           try {
               var14.writeInt(-889275714);
               var14.writeShort(0);
               var14.writeShort(49);
               this.cp.write(var14);
               var14.writeShort(this.accessFlags);
               var14.writeShort(this.cp.getClass(dotToSlash(this.className)));
               var14.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
               var14.writeShort(this.interfaces.length);
               Class[] var17 = this.interfaces;
               int var18 = var17.length;
    
               for(int var19 = 0; var19 < var18; ++var19) {
                   Class var22 = var17[var19];
                   var14.writeShort(this.cp.getClass(dotToSlash(var22.getName())));
               }
    
               var14.writeShort(this.fields.size());
               var15 = this.fields.iterator();
    
               while(var15.hasNext()) {
                   ProxyGenerator.FieldInfo var20 = (ProxyGenerator.FieldInfo)var15.next();
                   var20.write(var14);
               }
    
               var14.writeShort(this.methods.size());
               var15 = this.methods.iterator();
    
               while(var15.hasNext()) {
                   ProxyGenerator.MethodInfo var21 = (ProxyGenerator.MethodInfo)var15.next();
                   var21.write(var14);
               }
    
               var14.writeShort(0);
               return var13.toByteArray();
           } catch (IOException var9) {
               throw new InternalError("unexpected I/O Exception", var9);
           }
       }
   }
   ```

   可以看到主要做了3件事：

   - 为所有方法生成代理调度代码，将代理方法对象集合起来
   - 为类中的方法生成字段信息和方法信息
   - 编写最终类

5. 生成的代理类class文件为$Proxy0.class.通过对其反编译得到其源码如下：

   ```java
   package com.sun.proxy;
   
   import com.hyp.learn.proxy.UserService;
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   import java.lang.reflect.UndeclaredThrowableException;
   
   public final class $Proxy0 extends Proxy implements UserService {
       private static Method m1;
       private static Method m3;
       private static Method m2;
       private static Method m0;
   
       public $Proxy0(InvocationHandler var1) throws  {
           super(var1);
       }
   
       public final boolean equals(Object var1) throws  {
           try {
               return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
           } catch (RuntimeException | Error var3) {
               throw var3;
           } catch (Throwable var4) {
               throw new UndeclaredThrowableException(var4);
           }
       }
   
       public final String getUserByName(String var1) throws  {
           try {
               return (String)super.h.invoke(this, m3, new Object[]{var1});
           } catch (RuntimeException | Error var3) {
               throw var3;
           } catch (Throwable var4) {
               throw new UndeclaredThrowableException(var4);
           }
       }
   
       public final String toString() throws  {
           try {
               return (String)super.h.invoke(this, m2, (Object[])null);
           } catch (RuntimeException | Error var2) {
               throw var2;
           } catch (Throwable var3) {
               throw new UndeclaredThrowableException(var3);
           }
       }
   
       public final int hashCode() throws  {
           try {
               return (Integer)super.h.invoke(this, m0, (Object[])null);
           } catch (RuntimeException | Error var2) {
               throw var2;
           } catch (Throwable var3) {
               throw new UndeclaredThrowableException(var3);
           }
       }
   
       static {
           try {
               m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
               m3 = Class.forName("com.hyp.learn.proxy.UserService").getMethod("getUserByName", Class.forName("java.lang.String"));
               m2 = Class.forName("java.lang.Object").getMethod("toString");
               m0 = Class.forName("java.lang.Object").getMethod("hashCode");
           } catch (NoSuchMethodException var2) {
               throw new NoSuchMethodError(var2.getMessage());
           } catch (ClassNotFoundException var3) {
               throw new NoClassDefFoundError(var3.getMessage());
           }
       }
   }
   ```
   
   可以看到以下特点：
   
   - 该类继承了Proxy实现了UserService接口
   - 该类在static代码块中定义了所有该类包含的方法的Method实例，其中m3即通过反射获取的Fruit接口中的方法
   - 该类有一个构造器$Proxy0(InvocationHandler var1)传入调用处理器
   - 该类所有方法都将执行super.h.invoke并返回其结果，就是调用JdkCacheHandler类中invoke方法。

   