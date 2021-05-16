---
title: Spring AOP和aspectj
date: 2019-05-22 18:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented  Programming，面向对象编程）的补充和完善。OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross  cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

<!--more-->

AOP技术恰恰相反，它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：**核心关注点**和**横切关注点**。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

首先，在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

-  **所谓的核心业务**，比如登陆，增加数据，删除数据都叫核心业务
-  **所谓的周边功能**，比如性能统计，日志，事务管理等等

周边功能在 Spring 的面向切面编程AOP思想里，即被定义为切面

在面向切面编程AOP的思想里面，核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP

在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 **Aspect(切面)**

![1.png](https://i.loli.net/2019/07/09/5d23fec5e325b89516.png)

**AOP 的目的**

AOP能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**核心概念**

1. 横切关注点

   对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2. 切面（aspect）

   类是对物体特征的抽象，切面就是对横切关注点的抽象

   ` aspect` 由 `pointcount` 和 `advice` 组成, 它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP就是负责实施切面的框架, 它将切面所定义的横切逻辑织入到切面所指定的连接点中.
   AOP的工作重心在于如何将增强织入目标对象的连接点上, 这里包含两个工作:

   1. 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
   2. 如何在 advice 中编写切面代码.

   **可以简单地认为, 使用 @Aspect 注解的类就是切面.**

3. 连接点（joinpoint）

   被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

   在 Spring AOP 中, join point 总是方法的执行点, 即只有方法连接点.

4. 切入点（pointcut）

   对连接点进行拦截的定义

   匹配 join point 的谓词(a predicate that matches join points).
   Advice 是和特定的 point cut 关联的, 并且在 point cut 相匹配的 join point 中执行. 
   `在Spring 中, 所有的方法都可以认为是 joinpoint, 但是我们并不希望在所有的方法上都添加 Advice, 而pointcut 的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配joinpoint, 给满足规则的 joinpoint 添加 Advice.`

   advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice

5. 通知（advice）

   所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

   许多 AOP框架, 包括 Spring AOP, 会将 advice 模拟为一个拦截器(interceptor), 并且在 join point 上维护多个 advice, 进行层层拦截.

   例如 HTTP 鉴权的实现, 我们可以为每个使用 RequestMapping 标注的方法织入 advice, 当 HTTP 请求到来时, 
   首先进入到 advice 代码中, 在这里我们可以分析这个 HTTP 请求是否有相应的权限, 如果有, 则执行Controller, 如果没有, 则抛出异常. 这里的 advice 就扮演着鉴权拦截器的角色了.

6. 目标对象

   织入 advice 的目标对象. 目标对象也被称为 `advised object`.
   `因为 Spring AOP 使用运行时代理的方式来实现 aspect, 因此 adviced object 总是一个代理对象(proxied object)`
   `注意, adviced object 指的不是原来的类, 而是织入 advice 后所产生的代理类.`

7. 织入（weave）

   将切面应用到目标对象并导致代理对象创建的过程

   - 编译器织入, 这要求有特殊的Java编译器.
   - 类装载期织入, 这需要有特殊的类装载器.
   - 动态代理织入, 在运行期为目标类添加增强(Advice)生成子类的方式.

   Spring 采用动态代理织入, 而AspectJ采用编译器织入和类装载期织入.

8. 引入（introduction）

   在不修改代码的前提下，引入可以在**运行期**为类动态地添加一些方法或字段。
   
   为一个类型添加额外的方法或字段. Spring AOP 允许我们为 `目标对象` 引入新的接口(和对应的实现). 例如我们可以使用 introduction 来为一个 bean 实现 IsModified 接口, 并以此来简化 caching 的实现.
   
9. AOP proxy

   一个类被 AOP 织入 advice, 就会产生一个结果类, 它是融合了原类和增强逻辑的代理类.
   在 Spring AOP 中, 一个 AOP 代理是一个 JDK 动态代理对象或 CGLIB 代理对象.

通知(Advice)的类型：

- 前置通知（Before advice）：在某个连接点（Join point）之前执行的通知，但这个通知不能阻止连接点的执行（除非它抛出一个异常）。
- 返回后通知（After returning advice）：在某个连接点（Join point）正常完成后执行的通知。例如，一个方法没有抛出任何异常正常返回。
- 抛出异常后通知（After throwing advice）：在方法抛出异常后执行的通知。
- 后置通知（After（finally）advice）：当某个连接点（Join point）退出的时候执行的通知（不论是正常返回还是发生异常退出）。
- 环绕通知（Around advice）：包围一个连接点（Join point）的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。

**一个例子**

为了更好的说明 AOP 的概念，我们来举一个实际中的例子来说明：

![2.png](https://i.loli.net/2019/06/10/5cfdc8ff722b064954.png)

在上面的例子中，包租婆的核心业务就是签合同，收房租，那么这就够了，灰色框起来的部分都是重复且边缘的事，交给中介商就好了，这就是 **AOP 的一个思想：让关注点代码与业务代码分离！**

**流程图**

![1.png](https://i.loli.net/2019/06/10/5cfdc71da53e794597.png)

上图为我个人对AOP流程的一个理解。我把面向对象的过程从一个HttpRquest到访问数据库DB的整个流程看做是一条直线。AOP定义了一个切面（Aspect），一个切面包含了切入点，通知，引入，这个切面上定义了许多的切入点(Pointcut)，一旦访问过程中有对象的方法跟切入点匹配那么就会被AOP拦截。此时该对象就是目标对象（Target Object）而匹配的方法就是连接点（Join Point）。

紧接着AOP会用过JDK动态代理或者CGLIB生成一个目标对象的代理对象(AOP proxy)，这个过程就是织入（Weaving）。这个时候我们就可以按照我们的需求对连接点进行一些拦截处理。可以看到，我们可以引入（Introduction）一个新的接口，让代理对象来实现这个接口来，以实现额外的方法和字段。也可以在连接点上进行通知（Advice），通知的类型包括了前置通知，返回后通知，抛出异常后通知，后置通知，环绕通知。

最后也是最骚的是整个过程不会改变代码原有的逻辑。

#### 彻底理解 aspect, join point, point cut, advice

让我们来假设一下, 从前有一个叫爪哇的小县城, 在一个月黑风高的晚上, 这个县城中发生了命案. 作案的凶手十分狡猾,  现场没有留下什么有价值的线索. 不过万幸的是, 刚从隔壁回来的老王恰好在这时候无意中发现了凶手行凶的过程, 但是由于天色已晚, 加上凶手蒙着面,  老王并没有看清凶手的面目, 只知道凶手是个男性, 身高约七尺五寸. 爪哇县的县令根据老王的描述, 对守门的士兵下命令说:  凡是发现有身高七尺五寸的男性, 都要抓过来审问. 士兵当然不敢违背县令的命令, 只好把进出城的所有符合条件的人都抓了起来.

来让我们看一下上面的一个小故事和 AOP 到底有什么对应关系.
首先我们知道, 在 Spring AOP 中 **join point  指代的是所有方法的执行点**, 而 **point cut 是一个描述信息, 它修饰的是 join point**, 通过 point cut,  我们就可以确定哪些 join point 可以被织入 Advice. 对应到我们在上面举的例子, 我们可以做一个简单的类比, join  point 就相当于 **爪哇的小县城里的百姓**, point cut 就相当于 **老王所做的指控, 即凶手是个男性, 身高约七尺五寸**, 而 advice 则是施加在符合老王所描述的嫌疑人的动作: **抓过来审问**.
为什么可以这样类比呢?

- join point --> 爪哇的小县城里的百姓: 因为根据定义, join point 是所有可能被织入 advice  的候选的点, 在 Spring AOP中, 则可以认为所有方法执行点都是 join point. 而在我们上面的例子中, 命案发生在小县城中,  按理说在此县城中的所有人都有可能是嫌疑人.
- point cut --> 男性, 身高约七尺五寸: 我们知道, 所有的方法(joint point) 都可以织入  advice, 但是我们并不希望在所有方法上都织入 advice, 而 pointcut 的作用就是提供一组规则来匹配joinpoint,  给满足规则的 joinpoint 添加 advice. 同理, 对于县令来说, 他再昏庸, 也知道不能把县城中的所有百姓都抓起来审问, 而是根据`凶手是个男性, 身高约七尺五寸`, 把符合条件的人抓起来. 在这里 `凶手是个男性, 身高约七尺五寸` 就是一个修饰谓语, 它限定了凶手的范围, 满足此修饰规则的百姓都是嫌疑人, 都需要抓起来审问.
- advice --> 抓过来审问, advice 是一个动作, 即一段 Java 代码, 这段 Java 代码是作用于 point cut 所限定的那些 join point 上的. 同理, 对比到我们的例子中, `抓过来审问` 这个动作就是对作用于那些满足 `男性, 身高约七尺五寸` 的`爪哇的小县城里的百姓`.
- aspect: aspect 是 point cut 与 advice 的组合, 因此在这里我们就可以类比: **"根据老王的线索, 凡是发现有身高七尺五寸的男性, 都要抓过来审问"** 这一整个动作可以被认为是一个 aspect.

------

或则我们也可以从语法的角度来简单类比一下. 我们在学英语时, 经常会接触什么 `定语`, `被动句` 之类的概念, 那么可以做一个不严谨的类比, 即 `joinpoint` 可以认为是一个 `宾语`, 而 `pointcut` 则可以类比为修饰 `joinpoint` 的定语, 那么整个 `aspect` 就可以描述为: `满足 pointcut 规则的 joinpoint 会被添加相应的 advice 操作.`

#### Spring对AOP的支持

**Spring中AOP代理由Spring的IOC容器负责生成、管理，其依赖关系也由IOC容器负责管理**。因此，AOP代理可以直接使用容器中的其它bean实例作为目标，这种关系可由IOC容器的依赖注入提供。Spring创建代理的规则为：

1. **默认使用Java动态代理来创建AOP代理**，这样就可以为任何接口实例创建代理了

2. **当需要代理的类不是代理接口的时候，Spring会切换为使用CGLIB代理**，也可强制使用CGLIB

AOP编程其实是很简单的事情，纵观AOP编程，程序员只需要参与三个部分：

1. 定义普通业务组件

2. 定义切入点，一个切入点可能横切多个业务组件

3. 定义增强处理，增强处理就是在AOP框架为普通业务组件织入的处理动作

所以进行AOP编程的关键就是定义切入点和定义增强处理，一旦定义了合适的切入点和增强处理，AOP框架将自动生成AOP代理，即：**代理对象的方法=增强处理+被代理对象**的方法。

下面给出一个Spring AOP的.xml文件模板，名字叫做aop.xml，之后的内容都在aop.xml上进行扩展：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">
            
</beans>
```

**简单使用**

maven引用：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

Landlord.java

```java
import org.springframework.stereotype.Component;

@Component("landlord")
public class Landlord {

    public void service() {
        // 仅仅只是实现了核心的业务功能
        System.out.println("签合同");
        System.out.println("收房租");
    }
}
```

Broker.java

```java
@Component
@Aspect
public class Broker {
    @Before("execution(* com.hyp.learn.demo10.Landlord.service())")
    public void before(){
        System.out.println("带租客看房");
        System.out.println("谈价格");
    }

    @After("execution(* com.hyp.learn.demo10.Landlord.service())")
    public void after(){
        System.out.println("交钥匙");
    }
}
```

applicationContext.xml 中配置自动注入，并告诉 Spring IoC 容器去哪里扫描这两个 Bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:annotation-config/>
    <context:component-scan base-package="com.hyp.learn.demo10" />
    <aop:aspectj-autoproxy/>
</beans>

```

测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:applicationContext.xml"})
public class Demo10Test {

    @Autowired
    private ApplicationContext context;
    @Test
    public void demo10(){

        Landlord landlord = (Landlord) context.getBean("landlord", Landlord.class);
        landlord.service();
    }

}
```

运行结果：

```
带租客看房
谈价格
签合同
收房租
交钥匙
```

这个例子使用了一些注解，现在看不懂没有关系，但我们可以从上面可以看到，我们在 Landlord 的 service() 方法中仅仅实现了核心的业务代码，其余的关注点功能是根据我们设置的切面**自动补全**的。

#### SpringAOP与Aspectj的不同

1. SpingAop与Aspectj在面向切面编程是关注的点不同

   Spring AOP的目的并不是为了提供最完整的AOP实现（虽然Spring AOP具有相当的能力）；而是为了要帮助解决企业应用中的常见问题，提供一个AOP实现与Spring IOC之间的紧密集成，能够处理业务中的横切关注点。
   Aspectj提供了非常完善的AOP能力，几乎能在java class的任何时刻使用织入功能。

2. 注入时期

   AspectJ 的AOP，可以在三种时期进行代理，或者注入

   - **编译时织入**
      利用ajc编译器替代javac编译器，直接将源文件(java或者aspect文件)编译成class文件并将切面织入进代码。

   - **编译后织入**
      利用ajc编译器向javac编译期编译后的class文件或jar文件织入切面代码。

   - **加载时织入**
      不使用ajc编译器，利用aspectjweaver.jar工具，使用java agent代理在类加载期将切面织入进代码。
      （aspectjweaver.jar中，存在着@Before,@After等注解，并且有他们织入的实现(ASM动态代理)）

   SpringAop仅仅依赖动态代理实现：

   spring 的 AOP却完全只会使用Cglib或者JDK动态代理，在类加载时通过动态代理织入。因此，代理bean时，会受到bean的生命周期、声明方式而且产生一些制约问题。例如，JDK动态代理无法代理没有接口的JavaBean。

3. Aspectj影响了SpringAOP哪些东西

   AspectJ对AOP技术提供了两种概念：

   - 切面语法：这个和spring aop 是类似的，切点表达式、连接点等术语和概念
   - 织入工具：自己的AJC编译器，还有aspectjweaver.jar这种运行时代理工具

   也就是说，SpingAOP具体的面向切面具体实现，和AspectJ是没有太大关系的，但是，这里强调一下，SpringAOP借用了AspectJ的概念，可以拿aspectjweaver中的一些功能，来为SpringAOP服务。例如切点、连接点、切点表达式、前置通知等等

   并且，仿照AspectJ实现了自己的AOP功能。实际上，SpringAOP在功能上弱于AspectJ，因此，Spring也提供了对AspectJ的扩展Spring-aspects.jar和 Spring-instrument.jar，用来使用完整的AspectJ功能（在拥有IOC的前提下使用AspectJ，如果只用AspectJ提供的工具，就没有IOC了）。

#### 使用注解来开发 Spring AOP

使用注解的方式已经逐渐成为了主流，所以我们利用上面的例子来说明如何用注解来开发 Spring AOP

1. 选择连接点

   Spring 是方法级别的 AOP 框架，我们主要也是以某个类额某个方法作为连接点，另一种说法就是：**选择哪一个类的哪一方法用以增强功能。**

   我们在这里就选择上述 Landlord 类中的 service() 方法作为连接点。

2. 创建切面

   选择好了连接点就可以创建切面了，我们可以把切面理解为一个拦截器，当程序运行到连接点的时候，被拦截下来，在开头加入了初始化的方法，在结尾也加入了销毁的方法而已，在 Spring 中只要使用 `@Aspect` 注解一个类，那么 Spring IoC 容器就会认为这是一个切面了。

   -  **注意：** 被定义为切面的类仍然是一个 Bean ，需要 `@Component` 注解标注

   代码部分中在方法上面的注解看名字也能猜出个大概，下面来列举一下 Spring 中的 AspectJ 注解：

   | 注解              | 说明                                                         |
   | :---------------- | :----------------------------------------------------------- |
   | `@Before`         | 前置通知，在连接点方法前调用                                 |
   | `@Around`         | 环绕通知，它将覆盖原有方法，但是允许你通过反射调用原有方法，后面会讲 |
   | `@After`          | 后置通知，在连接点方法后调用                                 |
   | `@AfterReturning` | 返回通知，在连接点方法执行并正常返回后调用，要求连接点方法在执行过程中没有发生异常 |
   | `@AfterThrowing`  | 异常通知，当连接点方法异常时调用                             |

   有了上表，我们就知道 before() 方法是连接点方法调用前调用的方法，而 after() 方法则相反，这些注解中间使用了定义切点的正则式，也就是告诉 Spring AOP 需要拦截什么对象的什么方法，下面讲到。

   ![3.png](https://i.loli.net/2019/06/10/5cfdd70d7164243310.png)

3. 定义切点

   在上面的注解中定义了 execution 的正则表达式，Spring 通过这个正则表达式判断具体要拦截的是哪一个类的哪一个方法。

   ```
   execution(* com.hyp.learn.demo10.Landlord.service())
   ```

   依次对这个表达式作出分析：

   - execution：代表执行方法的时候会触发
   -  `*` ：代表任意返回类型的方法
   - com.hyp.learn.demo10.Landlord：代表类的全限定名
   - service()：被拦截的方法名称

   通过上面的表达式，Spring 就会知道应该拦截 pojo.Lnadlord 类下的 service() 方法。上面的演示类还好，如果多出都需要写这样的表达式难免会有些复杂，我们可以通过使用 `@Pointcut` 注解来定义一个切点来避免这样的麻烦：

   ```java
   @Component
   @Aspect
   class Broker {
   
       @Pointcut("execution(* com.hyp.learn.demo10.Landlord.service())")
       public void lService() {
       }
   
       @Before("lService()")
       public void before() {
           System.out.println("带租客看房");
           System.out.println("谈价格");
       }
   
       @After("lService()")
       public void after() {
           System.out.println("交钥匙");
       }
   }
   ```

4. 测试 AOP

#### 环绕通知

我们来探讨一下环绕通知，这是 Spring AOP 中最强大的通知，因为它集成了前置通知和后置通知，它保留了连接点原有的方法的功能，所以它及强大又灵活，让我们来看看：

```java
@Component
@Aspect
public class Broker {

    @Pointcut("execution(* com.hyp.learn.demo10.Landlord.service())")
    public void IService(){

    }

//    @Before("IService()")
//    public void before(){
//        System.out.println("带租客看房");
//        System.out.println("谈价格");
//    }
//
//    @After("IService()")
//    public void after(){
//        System.out.println("交钥匙");
//    }

    @Around("IService()")
    public Object around(ProceedingJoinPoint joinPoint)
    {
        System.out.println("带租客看房");
        System.out.println("谈价格");
        Object result=null;
        try {
            result=joinPoint.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }

        System.out.println("交钥匙");
        return result;
    }
}
```

#### 使用 XML 配置开发 Spring AOP

注解是很强大的东西，但基于 XML 的开发我们仍然需要了解，我们先来了解一下 AOP 中可以配置的元素：

| AOP 配置元素          | 用途                             | 备注                       |
| :-------------------- | :------------------------------- | :------------------------- |
| `aop:advisor`         | 定义 AOP 的通知其                | 一种很古老的方式，很少使用 |
| `aop:aspect`          | 定义一个切面                     | ——                         |
| `aop:before`          | 定义前置通知                     | ——                         |
| `aop:after`           | 定义后置通知                     | ——                         |
| `aop:around`          | 定义环绕通知                     | ——                         |
| `aop:after-returning` | 定义返回通知                     | ——                         |
| `aop:after-throwing`  | 定义异常通知                     | ——                         |
| `aop:config`          | 顶层的 AOP 配置元素              | AOP 的配置是以它为开始的   |
| `aop:declare-parents` | 给通知引入新的额外接口，增强功能 | ——                         |
| `aop:pointcut`        | 定义切点                         | ——                         |

有了之前通过注解来编写的经验，并且有了上面的表，我们将上面的例子改写成 XML 配置很容易（去掉所有的注解）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--<context:annotation-config/>-->
    <!--<context:component-scan base-package="com.hyp.learn.demo10"/>-->
    <!--<aop:aspectj-autoproxy/>-->
    
    <bean class="com.hyp.learn.demo10.Landlord" id="landlord"/>
    <bean class="com.hyp.learn.demo10.Broker" id="broker"/>
    
    <aop:config>
        <aop:pointcut id="landlordPoint"
                      expression="execution(* com.hyp.learn.demo10.Landlord.service())"/>
        <aop:aspect id="logAspect" ref="broker">
            <aop:around method="around" pointcut-ref="landlordPoint"/>
        </aop:aspect>
    </aop:config>

</beans>
```

#### 使能 @AspectJ 支持

**@AspectJ** 是一种使用 Java 注解来实现 AOP 的编码风格.
@AspectJ 风格的 AOP 是 AspectJ Project 在 AspectJ 5 中引入的, 并且 Spring 也支持@AspectJ 的 AOP 风格.

@AspectJ 可以以 XML 的方式或以注解的方式来使能, 并且不论以哪种方式使能@AspectJ, 我们都必须保证 aspectjweaver.jar 在 classpath 中.

##### 使用 Java Configuration 方式使能@AspectJ

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
}
```

##### 使用 XML 方式使能@AspectJ

```xml
<aop:aspectj-autoproxy/>
```

#### 定义 aspect(切面)

当使用注解 **@Aspect** 标注一个 Bean 后, 那么 Spring 框架会自动收集这些 Bean, 并添加到 Spring AOP 中, 例如:

```java
@Component
@Aspect
public class MyTest {
}
```

注意, 仅仅使用@Aspect 注解, 并不能将一个 Java 对象转换为 Bean, 因此我们还需要使用类似 @Component 之类的注解.`
`注意, 如果一个 类被@Aspect 标注, 则这个类就不能是其他 aspect 的 **advised object** 了, 因为使用 @Aspect 后, 这个类就会被排除在 auto-proxying 机制之外.

#### 声明 pointcut

一个 pointcut 的声明由两部分组成:

- 一个方法签名, 包括方法名和相关参数
- 一个 pointcut 表达式, 用来指定哪些方法执行是我们感兴趣的(即因此可以织入 advice).

在@AspectJ 风格的 AOP 中, 我们使用一个方法来描述 pointcut, 即:

```java
@Pointcut("execution(* com.xys.service.UserService.*(..))") // 切点表达式
private void dataAccessOperation() {} // 切点前面
```

`这个方法必须无返回值.`
`这个方法本身就是 pointcut signature, pointcut 表达式使用@Pointcut 注解指定.`
上面我们简单地定义了一个 pointcut, 这个 pointcut 所描述的是: 匹配所有在包 **com.xys.service.UserService** 下的所有方法的执行.

其实很好理解，aop的切入单位就是方法，所以我们需要理解方法的全称签名即可。其中除了返回类型模式、方法名模式和参数模式外，其它项都是可选的。

```java
//<修饰符模式>?<返回类型模式><方法名模式>(<参数模式>)<异常模式>?
//方法名模式:[<包名>.]<类名>.<方法名>
//如
public int com.hyp.learn.MyService.login(String name,String pass) throws
```

然后，pointcut基于正则的语法

```
* 表示任何数量的字符，除了(.)
.. 表示任何数量的字符包括任何数量的(.)
+ 描述指定类型的任何子类或者子接口

同java一样，提供了一元和二元的条件表达操作符。
一元操作符：!
二元操作符：||和&&
优先权同java
```

##### 切点标志符(designator)

AspectJ5 的切点表达式由标志符(designator)和操作参数组成. 如 "execution( *greetTo(..))" 的切点表达式, **execution** 就是 标志符, 而圆括号里的*  greetTo(..) 就是操作参数

##### execution

匹配 join point 的执行, 例如 "execution(* hello(..))" 表示匹配所有目标类中的 hello() 方法. 这个是最基本的 pointcut 标志符.

##### within

匹配**特定包下**的所有 join point, 例如 `within(com.xys.*)` 表示 com.xys 包中的所有连接点, 即包中的所有类的所有方法. 而 `within(com.xys.service.*Service)` 表示在 com.xys.service 包中所有以 Service 结尾的类的所有的连接点.

相当于：

```
execution(* com.xys.service.*Service.*.(..))
```

##### this 与 target

this 的作用是**匹配一个 bean**, 这个 bean(Spring AOP proxy) 是一个给定类型的**实例(instance  of)**. 而 target 匹配的是一个目标对象(target object, 即需要织入 advice 的**原始的类**),  此对象是一个给定类型的实例(instance of).

##### bean

匹配 bean 名字为**指定值的 bean 下的所有方法**, 例如:

```
bean(*Service) // 匹配名字后缀为 Service 的 bean 下的所有方法
bean(myService) // 匹配名字为 myService 的 bean 下的所有方法
//相当于
execution(* *.(<参数模式>))
```

##### args

匹配参数**满足要求的的方法**.

```
args(<参数模式>)
```

相当于

```
execution(* *.(<参数模式>))
```

例如:

```java
@Pointcut("within(com.xys.demo2.*)")
public void pointcut2() {
}

@Before(value = "pointcut2()  &&  args(name)")
public void doSomething(String name) {
    logger.info("---page: {}---", name);
}
@Service
public class NormalService {
    private Logger logger = LoggerFactory.getLogger(getClass());

    public void someMethod() {
        logger.info("---NormalService: someMethod invoked---");
    }


    public String test(String name) {
        logger.info("---NormalService: test invoked---");
        return "服务一切正常";
    }
}
```

当 NormalService.test 执行时, 则 advice `doSomething` 就会执行, test 方法的参数 name 就会传递到 `doSomething` 中.

常用例子:

```java
// 匹配只有一个参数 name 的方法
@Before(value = "aspectMethod()  &&  args(name)")
public void doSomething(String name) {
}

// 匹配第一个参数为 name 的方法
@Before(value = "aspectMethod()  &&  args(name, ..)")
public void doSomething(String name) {
}

// 匹配第二个参数为 name 的方法
Before(value = "aspectMethod()  &&  args(*, name, ..)")
public void doSomething(String name) {
}
```

##### @annotation

匹配由指定注解所标注的方法, 例如:

```java
@Pointcut("@annotation(com.xys.demo1.AuthChecker)")
public void pointcut() {
}
```

则匹配由注解 `AuthChecker` 所标注的方法.

##### 常见的切点表达式

##### 匹配方法签名

```java
// 匹配指定包中的所有的方法
execution(* com.xys.service.*(..))

// 匹配当前包中的指定类的所有方法
execution(* UserService.*(..))

// 匹配指定包中的所有 public 方法
execution(public * com.xys.service.*(..))

// 匹配指定包中的所有 public 方法, 并且返回值是 int 类型的方法
execution(public int com.xys.service.*(..))

// 匹配指定包中的所有 public 方法, 并且第一个参数是 String, 返回值是 int 类型的方法
execution(public int com.xys.service.*(String name, ..))
```

##### 匹配类型签名

```java
// 匹配指定包中的所有的方法, 但不包括子包
within(com.xys.service.*)

// 匹配指定包中的所有的方法, 包括子包
within(com.xys.service..*)

// 匹配当前包中的指定类中的方法
within(UserService)


// 匹配一个接口的所有实现类中的实现的方法
within(UserDao+)
```

##### 匹配 Bean 名字

```java
// 匹配以指定名字结尾的 Bean 中的所有方法
bean(*Service)
```

##### 切点表达式组合

```java
// 匹配以 Service 或 ServiceImpl 结尾的 bean
bean(*Service || *ServiceImpl)

// 匹配名字以 Service 结尾, 并且在包 com.xys.service 中的 bean
bean(*Service) && within(com.xys.service.*)
```

#### 切面优先级

我们常常会遇到这样一个问题， 如果有两个或多个切面同时对应同一个目标对象时，那么它的**优先级**是怎样的呢？ 

不管是基于哪种方式，设置**优先级**的数值越小，**优先级**越高（负数也可以）。设置了**优先级**数值的切面要比没有设置**优先级**数值的切面的**优先级**高，如果两个切面都没有设置**优先级**数值或**优先级**数值相同时，他们的**优先级**是不确定的。

配制方法

1. 在XMl中设置order属性，time优先级高于logger

   ```xml
   <!-- 配置切面类 -->
   <aop:config>
       <!-- 配置切面 -->
       <aop:aspect ref="time" order="1">
           <aop:before method="beforeTime" pointcut="execution(public int com.demo.HelloWorld.hello())" />
       </aop:aspect>
       <aop:aspect ref="logger" order="2">
           <aop:before method="beforeLogger" pointcut="execution(public int com.demo.HelloWorld.hello())" />
       </aop:aspect>
   </aop:config>
   ```

2. 如果是基于注解配置，切面的优先级可以通过实现 Ordered接口 或利用 Order注解 指定。

   ```java
   //实现Ordered接口
   public class Time implements Ordered {
       public void beforeTime() {
           System.out.println("计时开始~~~~~ 我是前置通知，会在目标方法前执行");
       }
       @Override
       public int getOrder() {
           return -2;
       }   
   }
   
   //Order 注解指定
   @Order(2)
   @Aspect
   @Component
   public class Time {
       public void beforeTime() {
           System.out.println("计时开始~~~~~ 我是前置通知，会在目标方法前执行");
       }   
   }
   ```

扩展阅读：

1. [Spring【AOP模块】就这么简单](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483954&idx=1&sn=b34e385ed716edf6f58998ec329f9867&chksm=ebd74333dca0ca257a77c02ab458300ef982adff3cf37eb6d8d2f985f11df5cc07ef17f659d4#rd) 

2. [关于 Spring AOP(AspectJ)你该知晓的一切（慎独读，有些深度...）](https://zhuanlan.zhihu.com/p/25522841)

3. [AspectJ之this和target的区别（四）](https://blog.csdn.net/wenyingzhi/article/details/84069529)

   