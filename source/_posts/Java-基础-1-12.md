---
title: Java 注解
date: 2019-04-06 14:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
注解(Annotation)很重要，未来的开发模式都是基于注解的，JPA是基于注解的，Spring2.5以上都是基于注解的，Hibernate3.x以后也是基于注解的，现在的Struts2有一部分也是基于注解的了，注解是一种趋势，现在已经有不少的人开始用注解了，注解是JDK1.5之后才有的新特性

在java 8之前，注解只能是在声明的地方所使用，java8开始，注解可以应用在任何地方。

<!--more-->

#### 为什么要引入注解？

使用Annotation之前(甚至在使用之后)，XML被广泛的应用于描述元数据。不知何时开始一些应用开发人员和架构师发现XML的维护越来越糟糕了。他们希望使用一些和代码紧耦合的东西，而不是像XML那样和代码是松耦合的(在某些情况下甚至是完全分离的)代码描述。

如果你在Google中搜索“XML  vs.  annotations”，会看到许多关于这个问题的辩论。最有趣的是XML配置其实就是为了分离代码和配置而引入的。上述两种观点可能会让你很疑惑，两者观点似乎构成了一种循环，但各有利弊。下面我们通过一个例子来理解这两者的区别。

假如你想为应用设置很多的常量或参数，这种情况下，XML是一个很好的选择，因为它不会同特定的代码相连。如果你想把某个方法声明为服务，那么使用Annotation会更好一些，因为这种情况下需要注解和方法紧密耦合起来，开发人员也必须认识到这点。

另一个很重要的因素是Annotation定义了一种标准的描述元数据的方式。在这之前，开发人员通常使用他们自己的方式定义元数据。例如，使用标记interfaces，注释，transient关键字等等。每个程序员按照自己的方式定义元数据，而不像Annotation这种标准的方式。

**目前，许多框架将XML和Annotation两种方式结合使用，平衡两者之间的利弊。**

注解的主要用途:
- 生成文档，通过代码里标识的元数据生成javadoc文档。
- 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例

JDK1.5之后内部提供的三个注解
```
@Deprecated 意思是“废弃的，过时的”
@Override 意思是“重写、覆盖”
@SuppressWarnings 意思是“压缩警告”
```

常用注释：

- @Deprecated`。这个元素是用来标记过时的元素，想必大家在日常开发中经常碰到。编译器在编译阶段遇到这个注解时会发出提醒警告，告诉开发者正在调用一个过时的元素比如过时的方法、过时的类、过时的成员变量。
- `@Override`。这个大家应该很熟悉了，提示子类要复写父类中被 `@Override `修饰的方法。

- `@SuppressWarnings`。阻止警告的意思。之前说过调用被 `@Deprecated` 注解的方法后，编译器会警告提醒，而有时候开发者会忽略这种警告，他们可以在调用的地方通过 `@SuppressWarnings` 达到目的。

- `@SafeVarargs`。参数安全类型注解。它的目的是提醒开发者不要用参数做一些不安全的操作,它的存在会阻止编译器产生 unchecked 这样的警告。它是在 Java 1.7 的版本中加入的。Java 官方文档说，未来的版本会授权编译器对这种不安全的操作产生错误警告。
- `@FunctionalInterface`。函数式接口注解，这个是 Java 1.8 版本引入的新特性。函数式编程很火，所以 Java 8 也及时添加了这个特性。函数式接口 (Functional Interface) 就是一个具有一个方法的普通接口

我们进行线程开发中常用的 Runnable 就是一个典型的函数式接口，上面源码可以看到它就被 @FunctionalInterface 注解。

可能有人会疑惑，函数式接口标记有什么用，这个原因是函数式接口可以很容易转换为 Lambda 表达式(java 8特性)。

#### 分类

按照运行机制划分：
【**源码注解→编译时注解→运行时注解**】

 - 源码注解：只在源码中存在，编译成.class文件就不存在了。

 - 编译时注解：在源码和.class文件中都存在。像前面的@Override、@Deprecated、@SuppressWarnings，他们都属于编译时注解。

 - 运行时注解：在运行阶段还起作用，甚至会影响运行逻辑的注解。像@Autowired自动注入的这样一种注解就属于运行时注解，它会在程序运行的时候把你的成员变量自动的注入进来。

按照来源划分：

【**来自JDK的注解——来自第三方的注解——自定义注解**】

元注解：

元注解是给注解进行注解，可以理解为注解的注解就是元注解。

**注解的应用：**

```Java
/**
 * 此类是用来演示注解(Annotation)的应用的，注解也是JDK1.5新增加的特性之一
 * JDK1.5内部提供的三种注解是：@SuppressWarnings(":deprecation")、@Deprecated、@Override
 *
 */
/**
 * 类名的命名是有讲究的，类名、属性名、变量名一般是名词，或者是形容词+名词，方法一般是动词，或者是动词+名词，
 * 以AnnotationTest作为类名和以TestAnnotation作为类名是有区别的，
 * 前者是注解的测试，符合名词的特征，后者是测试注解，听起来就是一个动作名称，是方法的命名特征
 */
public class AnnotationTest {
    /**
     * @param args
     */
    @SuppressWarnings("deprecation")
    //这里就是注解，称为压缩警告，这是JDK内部自带的一个注解，一个注解就是一个类，在这里使用了这个注解就是创建了SuppressWarnings类的一个实例对象
    public static void main(String[] args) {
         sayHello();
        //这里的 sayHello()方法画了一条横线表示此方法已经过时了，不建议使用了
    }
    @Deprecated //这也是JDK内部自带的一个注解，意思就是说这个方法已经废弃了，不建议使用了
    public static void sayHello(){
        System.out.println("hi,pingxin");
    }
    @Override //这也是JDK1.5之后内部提供的一个注解，意思就是要重写(覆盖)JDK内部的toString()方法
    public String toString(){
        return "pingxin";
    }
}
```

总结：**注解(Annotation)相当于一种标记，在程序中加入注解就等于为程序打上某种标记，没有加，则等于没有任何标记，以后，javac编译器、开发工具和其他程序可以通过反射来了解你的类及各种元素上有无何种标记，看你的程序有什么标记，就去干相应的事，标记可以加在包、类，属性、方法，方法的参数以及局部变量上。**

#### 注解使用原理

![](https://i.loli.net/2019/04/13/5cb196a63588a.jpg)

注解就相当于一个你的源程序要调用一个类，在源程序中应用某个注解，得事先准备好这个注解类。就像你要调用某个类，得事先开发好这个类。


#### 元注解

元注解是什么意思呢？

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。

如果难于理解的话，你可以这样理解。元注解也是一张标签，但是它是一张特殊的标签，它的作用和目的就是给其他普通的标签进行解释说明的。

元标签有 @Retention、@Documented、@Target、@Inherited、@Repeatable 5 种。

![注解.jpg](https://i.loli.net/2019/05/18/5cdf9362b480d68737.jpg)

##### `@Retention`

Retention 的英文意为保留期的意思。当 `@Retention `应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

它的取值如下：

 - RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
 - RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
 - RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

我们可以这样的方式来加深理解，`@Retention` 去给一张标签解释的时候，它指定了这张标签张贴的时间。`@Retention` 相当于给一张标签上面盖了一张时间戳，时间戳指明了标签张贴的时间周期。

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
}
```
上面的代码中，我们指定 TestAnnotation 可以在程序运行周期被获取到，因此它的生命周期非常的长。

##### `@Documented`

顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去。
##### `@Target`

Target 是目标的意思，@Target 指定了注解运用的地方。

你可以这样理解，当一个注解被 `@Target `注解时，这个注解就被限定了运用的场景。

类比到标签，原本标签是你想张贴到哪个地方就到哪个地方，但是因为 `@Target `的存在，它张贴的地方就非常具体了，比如只能张贴到方法上、类上、方法参数上等等。@Target 有下面的取值

```
ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
ElementType.CONSTRUCTOR 可以给构造方法进行注解
ElementType.FIELD 可以给属性进行注解
ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
ElementType.METHOD 可以给方法进行注解
ElementType.PACKAGE 可以给一个包进行注解
ElementType.PARAMETER 可以给一个方法内的参数进行注解
ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举
ElementType.TYPE_PARAMETER @since 1.8，输入参数声明，表示该注解能写在类型变量的声明语句中
ElementType.TYPE_USE @since 1.8 表示该注解能写在使用类型的任何语句中（eg：声明语句、泛型和强制转换语句中的类型）。
ElementType.MODULE @since 9 模块声明
```

##### `@Inherited`

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被` @Inherited` 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}

@Test
public class A {}
public class B extends A {}
```
注解 Test 被 `@Inherited` 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。

##### `@Repeatable`

Repeatable 自然是可重复的意思。`@Repeatable` 是 Java 1.8 才加进来的，所以算是一个新的特性。

什么样的注解会多次应用呢？通常是注解的值可以同时取多个。

举个例子，一个人他既是程序员又是产品经理,同时他还是个画家。

```java
@interface Persons {
	Person[]  value();
}

@Repeatable(Persons.class)
@interface Person{
	String role() default "";
}

@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
}
```
注意上面的代码，`@Repeatable` 注解了 Person。而` @Repeatable `后面括号中的类相当于一个容器注解。

什么是容器注解呢？就是用来存放其它注解的地方。它本身也是一个注解。

我们再看看代码中的相关容器注解。

```java
@interface Persons {
	Person[]  value();
}
```

按照规定，它里面必须要有一个 value 的属性，属性类型是一个被 @Repeatable 注解过的注解数组，注意它是数组。

如果不好理解的话，可以这样理解。Persons 是一张总的标签，上面贴满了 Person 这种同类型但内容不一样的标签。把 Persons 给一个 SuperMan 贴上，相当于同时给他贴了程序员、产品经理、画家的标签。

我们可能对于 @Person(role=“PM”) 括号里面的内容感兴趣，它其实就是给 Person 这个注解的 role 属性赋值为 PM ，大家不明白正常，马上就讲到注解的属性这一块。

#### 注解的属性

注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	int id();
	String msg();
}
```
上面代码定义了 TestAnnotation 这个注解中拥有 id 和 msg 两个属性。在使用的时候，我们应该给它们进行赋值。

赋值的方式是在注解的括号内以 value="" 形式，多个属性之前用 ，隔开。

```java
@TestAnnotation(id=3,msg="hello annotation")
public class Test {
}
```

需要注意的是，**java8之前**，在注解中定义属性时它的类型必须是 8 种基本数据类型外加 类、接口、注解及它们的数组。

**返回类型必须是基本类型，*String*，*Class*，*Enum*或以前类型之一的数组。否则，编译器将抛出错误。**

注解中属性可以有默认值，默认值需要用 default 关键值指定。比如：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	public int id() default -1;
	public String msg() default "Hi";
}
```
TestAnnotation 中 id 属性默认值为 -1，msg 属性默认值为 Hi。它可以这样应用。

```java
@TestAnnotation()
public class Test {}
```

因为有默认值，所以无需要再在 `@TestAnnotation` 后面的括号里面进行赋值了，这一步可以省略。

另外，还有一种情况。如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接接属性值填写到括号内。

```java
public @interface Check {
	String value();
}
```
上面代码中，Check 这个注解只有 value 这个属性。所以可以这样应用。

```java
@Check("hi")
int a;
```
这和下面的效果是一样的
```java
@Check(value="hi")
int a;
```

最后，还需要注意的一种情况是一个注解没有任何属性。比如

```java
@Perform
public void testMethod(){}
```

增加枚举类型的属性：
```java
EumTrafficLamp lamp() default EumTrafficLamp.RED;
```
应用枚举类型的属性：
```java
@MyAnnotation(lamp=EumTrafficLamp.GREEN)
```

#### 自定义注解

自定义一个最简单的注解：

```java
/**
 * 这是一个自定义的注解(Annotation)类 在定义注解(Annotation)类时使用了另一个注解类Retention
 * 在注解类上使用另一个注解类，那么被使用的注解类就称为元注解
 *
 */
@Retention(RetentionPolicy.RUNTIME)
//Retention注解决定MyAnnotation注解的生命周期
@Target( { ElementType.METHOD, ElementType.TYPE })
//Target注解决定MyAnnotation注解可以加在哪些成分上，如加在类身上，或者属性身上，或者方法身上等成分
/*
 * @Retention(RetentionPolicy.SOURCE)
 * 这个注解的意思是让MyAnnotation注解只在java源文件中存在，编译成.class文件后注解就不存在了
 * @Retention(RetentionPolicy.CLASS)
 * 这个注解的意思是让MyAnnotation注解在java源文件(.java文件)中存在，编译成.class文件后注解也还存在，
 * 被MyAnnotation注解类标识的类被类加载器加载到内存中后MyAnnotation注解就不存在了
 */
/*
 * 这里是在注解类MyAnnotation上使用另一个注解类，这里的Retention称为元注解。
 * Retention注解括号中的"RetentionPolicy.RUNTIME"意思是让MyAnnotation这个注解的生命周期一直程序运行时都存在
 */
public @interface MyAnnotation {
}
```
把自定义的注解加到某个类上：

```java
@MyAnnotation
public class AnnotationUse{
}
```
用反射测试进行测试AnnotationUse的定义上是否有@MyAnnotation

```java
@MyAnnotation
//这里是将新创建好的注解类MyAnnotation标记到AnnotaionTest类上
public class AnnotationUse {
    public static void main(String[] args) {
        // 这里是检查Annotation类是否有注解，这里需要使用反射才能完成对Annotation类的检查
        if (AnnotationUse.class.isAnnotationPresent(MyAnnotation.class)) {
            /*
             * MyAnnotation是一个类，这个类的实例对象annotation是通过反射得到的，这个实例对象是如何创建的呢？
             * 一旦在某个类上使用了@MyAnnotation，那么这个MyAnnotation类的实例对象annotation就会被创建出来了
             * 假设很多人考驾照，教练在有些学员身上贴一些绿牌子、黄牌子，贴绿牌子的表示送礼送得比较多的，
             * 贴黄牌子的学员表示送礼送得比较少的，不贴牌子的学员表示没有送过礼的，通过这个牌子就可以标识出不同的学员
             * 教官在考核时一看，哦，这个学员是有牌子的，是送过礼给他的，优先让有牌子的学员过，此时这个牌子就是一个注解
             * 一个牌子就是一个注解的实例对象，实实在在存在的牌子就是一个实实在在的注解对象，把牌子拿下来(去掉注解)注解对象就不存在了
             */
            MyAnnotation annotation = (MyAnnotation) AnnotationUse.class
                    .getAnnotation(MyAnnotation.class);
            System.out.println(annotation);// 打印MyAnnotation对象，这里输出的结果为：@cn.itcast.day2.MyAnnotation()
        }
    }
}
```
#### 注解的提取

用标签来比作注解，前面的内容是讲怎么写注解，然后贴到哪个地方去，而现在我们要做的工作就是检阅这些标签内容。 形象的比喻就是你把这些注解标签在合适的时候撕下来，然后检阅上面的内容信息。

要想正确检阅注解，离不开一个手段，那就是**反射**。

注解通过反射获取。**首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解。然后通过 getAnnotation() 方法来获取 Annotation 对象。或者是 getAnnotations() 方法。前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解。如果获取到的 Annotation 如果不为 null，则就可以调用它们的属性方法了**。比如

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	public int id() default -1;
	public String msg() default "Hi";
}


public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}

public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
//或
 public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}

//调用它们的属性方法

 @TestAnnotation()
 public class Test {

 	public static void main(String[] args) {

 		boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);

 		if ( hasAnnotation ) {
 			TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);

 			System.out.println("id:"+testAnnotation.id());
 			System.out.println("msg:"+testAnnotation.msg());
 		}
 	}
 }
```
运行结果:

```
id:-1
msg:
```
这个正是 TestAnnotation 中 id 和 msg 的默认值。

上面的例子中，只是检阅出了注解在类上的注解，其实属性、方法上的注解照样是可以的。同样还是要假手于反射。


```java
@TestAnnotation(msg="hello")
public class Test {

	@Check(value="hi")
	int a;


	@Perform
	public void testMethod(){}


	@SuppressWarnings("deprecation")
	public void test1(){
		Hero hero = new Hero();
		hero.say();
		hero.speak();
	}


	public static void main(String[] args) {

		boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);

		if ( hasAnnotation ) {
			TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
			//获取类的注解
			System.out.println("id:"+testAnnotation.id());
			System.out.println("msg:"+testAnnotation.msg());
		}


		try {
			Field a = Test.class.getDeclaredField("a");
			a.setAccessible(true);
			//获取一个成员变量上的注解
			Check check = a.getAnnotation(Check.class);

			if ( check != null ) {
				System.out.println("check value:"+check.value());
			}

			Method testMethod = Test.class.getDeclaredMethod("testMethod");

			if ( testMethod != null ) {
				// 获取方法中的注解
				Annotation[] ans = testMethod.getAnnotations();
				for( int i = 0;i < ans.length;i++) {
					System.out.println("method testMethod annotation:"+ans[i].annotationType().getSimpleName());
				}
			}
		} catch (NoSuchFieldException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println(e.getMessage());
		} catch (SecurityException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println(e.getMessage());
		} catch (NoSuchMethodException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println(e.getMessage());
		}
	}
}
```
它们的结果如下：
```
id:-1
msg:hello
check value:hi
method testMethod annotation:Perform
```

需要注意的是，如果一个注解要在运行时被成功提取，那么 @Retention(RetentionPolicy.RUNTIME) 是必须的。

#### 使用场景

注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。

注解有许多用处，主要如下：

 - 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息
 - 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。
 - 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取

值得注意的是，注解不是代码本身的一部分。

当开发者使用了Annotation 修饰了类、方法、Field 等成员之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT（Annotation Processing Tool)。

现在，我们可以给自己答案了，注解有什么用？给谁用？给 编译器或者 APT 用的。

示例：

```java
public class NoBug {

	@Jiecha
	public void suanShu(){
		System.out.println("1234567890");
	}
	@Jiecha
	public void jiafa(){
		System.out.println("1+1="+1+1);
	}
	@Jiecha
	public void jiefa(){
		System.out.println("1-1="+(1-1));
	}
	@Jiecha
	public void chengfa(){
		System.out.println("3 x 5="+ 3*5);
	}
	@Jiecha
	public void chufa(){
		System.out.println("6 / 0="+ 6 / 0);
	}

	public void ziwojieshao(){
		System.out.println("我写的程序没有 bug!");
	}

}

//注解类
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Jiecha {
}

//测试类 TestTool 就可以测试 NoBug 相应的方法

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
public class TestTool {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		NoBug testobj = new NoBug();

		Class clazz = testobj.getClass();

		Method[] method = clazz.getDeclaredMethods();
		//用来记录测试产生的 log 信息
		StringBuilder log = new StringBuilder();
		// 记录异常的次数
		int errornum = 0;

		for ( Method m: method ) {
			// 只有被 @Jiecha 标注过的方法才进行测试
			if ( m.isAnnotationPresent( Jiecha.class )) {
				try {
					m.setAccessible(true);
					m.invoke(testobj, null);

				} catch (Exception e) {
					// TODO Auto-generated catch block
					//e.printStackTrace();
					errornum++;
					log.append(m.getName());
					log.append(" ");
					log.append("has error:");
					log.append("\n\r  caused by ");
					//记录测试过程中，发生的异常的名称
					log.append(e.getCause().getClass().getSimpleName());
					log.append("\n\r");
					//记录测试过程中，发生的异常的具体信息
					log.append(e.getCause().getMessage());
					log.append("\n\r");
				}
			}
		}
		log.append(clazz.getSimpleName());
		log.append(" has  ");
		log.append(errornum);
		log.append(" error.");
		// 生成测试报告
		System.out.println(log.toString());
	}
}
```
运行结果：

```
1+1=11
1-1=0
3 x 5=15
1234567890
chufa has error:
  caused by ArithmeticException
/ by zero
NoBug has  1 error.
```

所以，再问我注解什么时候用？我只能告诉你，这取决于你想利用它干什么用。

#### 使用反射和注解完成简单的ORM功能

**ORM原理简介**

ORM是对象关系映射的意思。他建立起了以下映射关系：

- 类对应于表

- 对象对应于表中的记录

- 对象的属性对应于表的字段

有了这种映射关系,我们在编写代码时就可以通过操作对象来映射对数据库表的操作,比如添加记录,更新记录,删除记录等等。常见的Mybatis,Hibernate就是ORM框架。而实现ORM功能最常用的手段就是注解+反射。由注解维护这种映射关系,然后运行期通过反射技术解析注解,完成对应关系的转换,从而形成一句完整的sql去执行。

下面以建表为例,实现简单的ORM功能。

**ORM实战**

- 自定义表注解,完成类和表的映射。

  ```java
      /**
       * 自定义表注解,完成类和表的映射
       */
      @Retention(RetentionPolicy.RUNTIME) //因为要使用到反射,故注解信息必须保留到运行时
      @Target(ElementType.TYPE)//只能用在类上
      public @interface MyTable {
          //表名
          String value();
      }
  ```

- 自定义字段注解

  ```java
      /**
       * 自定义字段注解,完成类属性和表字段的映射
       */
      @Retention(RetentionPolicy.RUNTIME)//要反射,故注解信息需要保留到运行期
      @Target(ElementType.FIELD)//只能用在类属性上
      public @interface MyColumn {
          //字段名
          String value();
          //字段类型,默认为字符串类型
          String type() default "VARCHAR(30)";//字段类型,默认为VARCHAR类型
          //类型为注解类型的字段约束,默认的约束为:非主键，非唯一字段，不能为null
          Constraints constraint() default @Constraints;
      }
  ```

- 自定义字段约束注解

  ```java
      /**
       * 约束注解:主键,是否为空,是否唯一等信息。
       */
      @Retention(RetentionPolicy.RUNTIME)//运行期
      @Target(ElementType.FIELD)//只能在类属性上使用
      public @interface Constraints {
          //字段是否为主键约束
          boolean primaryKey() default false;
          //字段是否允许为null
          boolean nullable() default false;
          //字段是否唯一
          boolean unique() default false;
      }
  ```

- 带注解的实体类

  ```java
      /**
       * 带注解的实体类,建立了对象和表的映射关系,可以再运行时被解析
       */
      @MyTable("t_user")
      public class User {
          //主键,对应表字段id,类型为VARCHAR
          @MyColumn(value = "id", constraint = @Constraints(primaryKey = true))
          private String id;
          //对应表字段name,类型为类型为VARCHAR
          @MyColumn(value = "name")
          private String name;
          //对应表字段age,类型为INT,且可为null
          @MyColumn(value = "age", type = "INT", constraint = @Constraints(nullable = true))
          private int age;
          //对应表字段phone_number,类型为VARCHAR,且有唯一约束
          @MyColumn(value = "phone_number", constraint = @Constraints(unique = true))
          private String phoneNumber;
      }
  ```

- 运行时注解解析器

  ```java
      public class TableGenerator {
          /**
           * 运行时解析注解生成对应的建表语句
           *
           * @param clazz 与表对应的实体的Class对象
           * @return
           */
          public static String genSQL(Class clazz) {
              String table;//表名
              List<String> columnSegments = new ArrayList<>();
              //获取表注解
              MyTable myTable = (MyTable) clazz.getAnnotation(MyTable.class);
              if (myTable == null) {
                  throw new IllegalArgumentException("表注解不能为空!");
              }
              //获取表名
              table = myTable.value();
              //获取所有字段
              Field[] fields = clazz.getDeclaredFields();
              for (Field field : fields) {
                  MyColumn column = field.getAnnotation(MyColumn.class);
                  if (column == null) {
                      continue;//为null说明该字段不为映射字段,也就是没有加上字段注解
                  }
                  StringBuilder columnSegement = new StringBuilder();//字段分片,eg:"id varchar(50) primary key"
                  String columnType = column.type().toUpperCase();//字段类型
                  String columnName = column.value().toUpperCase();//字段名
                  columnSegement.append(columnName).append(" ").append(columnType).append(" ");
                  Constraints constraint = column.constraint();
                  boolean primaryKey = constraint.primaryKey();
                  boolean nullable = constraint.nullable();
                  boolean unique = constraint.unique();
                  if (primaryKey) {
                      //主键唯一且不为空
                      columnSegement.append("PRIMARY KEY ");
                  } else if (!nullable) {
                      //字段不为null
                      columnSegement.append("NOT NULL ");
                  }
                  if (unique) {
                      //有唯一键
                      columnSegement.append("UNIQUE ");
                  }
                  columnSegments.add(columnSegement.toString());
              }
              if (columnSegments.size() < 1) {
                  //没有映射任何表字段,抛出异常
                  throw new IllegalArgumentException("没有映射任何表字段!");
              }
              StringJoiner joiner = new StringJoiner(",", "(", ")");
              for (String segement : columnSegments) {
                  joiner.add(segement);
              }
              //生成SQL语句
              return String.format("CREATE TABLE %s", table) + joiner.toString();
          }
      }
  ```

  通过该解析器的genSQL方法在运行时生成建表SQL,通过传入的Class参数在运行时解析类和属性上的注解,分别得到表名,字段名,字段类型，约束条件等信息，然后拼装成SQL。由于只是为了做演示,对SQL语法的支持比较弱,只允许字段为int和varchar类型。且解析语法时也没有考虑一些边界情况。但是通过这段代码演示可以知道ORM框架在解析注解时的大概工作和流程是怎么样的。

- 测试代码

  ```java
      public class TableGeneratorTest {
          public static void main(String[] args) {
              String sql = TableGenerator.genSQL(User.class);
              System.out.println(sql);
          }
      }
  ```

最后得到的建表语句如下

```mysql
CREATE TABLE t_user(ID VARCHAR(30) PRIMARY KEY ,NAME VARCHAR(30) NOT NULL ,AGE INT ,PHONE_NUMBER VARCHAR(30) NOT NULL UNIQUE )
```

最后我们验证下生成的建表SQL语法是否有问题,在mysql客户端上执行该sql

#### 总结

- 如果注解难于理解，你就把它类同于标签，标签为了解释事物，注解为了解释代码。
- 注解的基本语法，创建如同接口，但是多了个 @ 符号。
- 注解的元注解。
- 注解的属性。
- 注解主要给编译器及工具类型的软件用的。
- 注解的提取需要借助于 Java 的反射技术，反射比较慢，所以注解使用时也需要谨慎计较时间成本。

注解的好处：
1. 能够读懂别人写的代码，特别是框架相关的代码。
2. 本来可能需要很多配置文件，需要很多逻辑才能实现的内容，就可以使用一个或者多个注解来替代，这样就使得编程更加简洁，代码更加清晰。
3. （重点）刮目相看。（但是怎么样才能让别人刮目相看呢？会用注解不是目的，最重要的是要使用自定义注解来解决问题。）

#### java 8 注解新特性

对于注解（也被称做元数据），Java 8 主要有两点改进：类型注解和重复注解。

##### 类型注解

1. Java 8 的类型注解扩展了注解使用的范围。

   在java 8之前，注解只能是在声明的地方所使用，java8开始，注解可以应用在任何地方。

   eg：

   ```
   创建类实例
   new@Interned MyObject();
   类型映射
   myString = (@NonNull String) str;
   implements 语句中
   class UnmodifiableList<T> implements@Readonly List<@Readonly T> { ... }
   throw exception声明
   void monitorTemperature() throws@Critical TemperatureException { ... }
   ```

   Note：

   在Java 8里面，当类型转化甚至分配新对象的时候，都可以在声明变量或者参数的时候使用注解。

   Java注解可以支持任意类型。

   类型注解只是语法而不是语义，并不会影响java的编译时间，加载时间，以及运行时间，也就是说，编译成class文件的时候并不包含类型注解。

2. 新增ElementType.TYPE_USE 和ElementType.TYPE_PARAMETER（在Target上）

   新增的两个注释的程序元素类型 ElementType.TYPE_USE 和 ElementType.TYPE_PARAMETER用来描述注解的新场合。

   ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中。

   ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中（eg：声明语句、泛型和强制转换语句中的类型）。

   eg：

   ```
   @Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})  
   @interface MyAnnotation {}  
   ```

3. 类型注解的作用

   类型注解被用来支持在Java的程序中做强类型检查。配合第三方插件工具Checker Framework（注：此插件so easy,这里不介绍了），可以在编译的时候检测出runtime error（eg：UnsupportedOperationException； NumberFormatException；NullPointerException异常等都是runtime error），以提高代码质量。这就是类型注解的作用。

   Note：
   使用Checker Framework可以找到类型注解出现的地方并检查。

   eg:

   ```
   import checkers.nullness.quals.*;  
   public class TestDemo{  
       void sample() {  
           @NonNull Object my = new Object();  
       }  
   }  
   ```

   使用javac编译上面的类：（当然若下载了Checker Framework插件就不需要这么麻烦了）

   ```
   javac -processor checkers.nullness.NullnessChecker TestDemo.java
   ```

   上面编译是通过的，但若修改代码：

   ```
   @NonNull Object my = null;
   ```

   但若不想使用类型注解检测出来错误，则不需要processor，正常javac TestDemo.java是可以通过编译的，但是运行时会报 NullPointerException 异常。

   为了能在编译期间就自动检查出这类异常，可以通过类型注解结合 Checker Framework 提前排查出来错误异常。

   注意java 5,6,7版本是不支持注解@NonNull，但checker framework 有个向下兼容的解决方案，就是将类型注解@NonNull 用`/**/`注释起来。

   这样javac编译器就会忽略掉注释块，但用checker framework里面的javac编译器同样能够检测出@NonNull错误。
   通过 类型注解 + checker framework 可以在编译时就找到runtime error。

##### 重复注解

允许在同一声明类型（类，属性，或方法）上多次使用同一个注解。

Java8以前的版本使用注解有一个限制是相同的注解在同一位置只能使用一次，不能使用多次。

Java 8 引入了重复注解机制，这样相同的注解可以在同一地方使用多次。重复注解机制本身必须用 @Repeatable 注解。

实际上，重复注解不是一个语言上的改变，只是编译器层面的改动，技术层面仍然是一样的。

1. 自定义一个包装类Hints注解用来放置一组具体的Hint注解

   ```
   @interface MyHints {  
       Hint[] value();  
   }  
      
   @Repeatable(MyHints.class)  
   @interface Hint {  
       String value();  
   }  
   ```

   使用包装类当容器来存多个注解（旧版本方法）

   ```
   @MyHints({@Hint("hint1"), @Hint("hint2")})  
   class Person {}  
   ```

   使用多重注解（新方法）

   ```
   @Hint("hint1")  
   @Hint("hint2")  
   class Person {}  
   ```

2. ```java
   public class RepeatingAnnotations {  
       @Target(ElementType.TYPE)  
       @Retention(RetentionPolicy.RUNTIME)  
       public @interface Filters {  
           Filter[] value();  
       }  
         
       @Target(ElementType.TYPE)  
       @Retention(RetentionPolicy.RUNTIME)  
       @Repeatable(Filters.class)  
       public @interface Filter {  
           String value();  
       }  
       @Filter("filter1")  
       @Filter("filter2")  
       public interface Filterable {  
       }  
       public static void main(String[] args) {  
           for (Filter filter : Filterable.class.getAnnotationsByType(Filter.class)) {  
               System.out.println(filter.value());  
           }  
       }  
   }  
   ```

   输出结果：

   ```
   filter1
   filter2
   ```

   分析：

   注释Filter被@Repeatable( Filters.class )注释。Filters 只是一个容器，它持有Filter, 编译器尽力向程序员隐藏它的存在。通过这样的方式，Filterable接口可以被Filter注释两次。

   另外，反射的API提供一个新方法getAnnotationsByType() 来返回重复注释的类型（注意Filterable.class.getAnnotation( Filters.class )将会返回编译器注入的Filters实例）。

3. java 8之前也有重复使用注解的解决方案，但可读性不好。

   ```java
   public @interface MyAnnotation {    
        String role();    
   }    
      
   public @interface Annotations {    
       MyAnnotation[] value();    
   }    
      
   public class RepeatAnnotationUseOldVersion {    
           
       @Annotations({@MyAnnotation(role="Admin"),@MyAnnotation(role="Manager")})    
       public void doSomeThing(){    
       }    
   }  
   ```

   Java8的实现方式（由另一个注解来存储重复注解，在使用时候，用存储注解Authorities来扩展重复注解），可读性更强。

   ```java
   @Repeatable(Annotations.class)   
   public @interface MyAnnotation {    
        String role();    
   }    
      
   public @interface Annotations {    
       MyAnnotation[] value();    
   }    
      
   public class RepeatAnnotationUseOldVersion {    
       @MyAnnotation(role="Admin")    
       @MyAnnotation(role="Manager")  
       public void doSomeThing(){    
       }    
   }   
   ```

#### 面试

1. 是否可以扩展注释？

   注释。注释总是扩展*java.lang.annotation.Annotation，*如[Java语言规范中所述](http://docs.oracle.com/javase/specs/jls/se7/html/jls-9.html#jls-9.6)。

   如果我们尝试在注释声明中使用*extends*子句，我们将得到一个编译错误：

   ```java
   public @interface AnAnnotation extends OtherAnnotation {
      // Compilation error}
   ```

2. 下面的代码会编译吗？

   ```java
   @Target({ ElementType.FIELD, ElementType.TYPE, ElementType.FIELD })
    
   public @interface TestAnnotation {
    
       int[] value() default {};
    
   }
   ```

   不。如果在*@Target*注释中多次出现相同的枚举常量，那么这是一个编译时错误。

   删除重复常量将使代码成功编译：

   ```java
   @Target({ ElementType.FIELD, ElementType.TYPE})
   ```

#### 参考

1. [秒懂，Java 注解 （Annotation）你可以这样学](https://blog.csdn.net/briblue/article/details/73824058/)

