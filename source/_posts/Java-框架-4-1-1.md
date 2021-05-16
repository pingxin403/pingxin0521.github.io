---
title: Spring  依赖注入(一)
date: 2019-05-22 13:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

### Spring容器

容器是Spring框架的核心。Spring容器使用DI管理构成应用的组件,它会创建相互协作的组件之间的关联。毫无疑问,这些对象更简单干净,更易于理解,更易于重用并且更易于进行单元测试。

<!--more-->

Spring容器并不是只有一个。Spring自带了多个容器实现,可以归为两种不同的类型。bean工厂(由`org.springframework.beans.factory.BeanFactory`接口定义)是最简单的容器,提供基本的DI支持。应用上下文(由`org.springframework.context.ApplicationContext`接口定义)基于BeanFactory构建,并提供应用框架级别的服务,例如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者。

虽然我们可以在bean工厂和应用上下文之间任选一种,但bean工厂对大多数应用来说往往太低级了,因此,应用上下文要比bean工厂更受欢迎。我们会把精力集中在应用上下文的使用上,不再浪费时间讨论bean工厂。

**ApplicationContext 容器包括 BeanFactory 容器的所有功能，所以通常建议超过 BeanFactory。BeanFactory 仍然可以用于轻量级的应用程序，如移动设备或基于 applet 的应用程序，其中它的数据量和速度是显著。**

**Spring容器初始化两种方式**

1. ApplicationContext（子类）

```java
//默认加载文件系统的配置文件，主要配置文件放在项目下、本地上、类路径下（3种位置）
ApplicationContext ac=new FileSystemXmlApplicationContext("classpath:beans.xml");

//默认加载ClassPath路径下的配置文件
ApplicationContext ac=new ClassPathXmlApplicationContext("classpath:beans.xml");
```

2. BeanFactory（父类）

```java
Resource resource=new ClassPathResource("beans.xml"); 

//初始化spring容器的代码
BeanFactory beanFactory=new XmlBeanFactory(resource); 

//调用Bean    
Person person=(Person)beanFactory.getBean("person");//person为配置文件bean中自定义的id名称
```

1. XmlBeanFactory通过Resource装载Spring配置信息冰启动IoC容器，然后就可以通过factory.getBean从IoC容器中获取Bean了。
2. 通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化动作发生在第一个调用时。
3. 对于单实例（singleton）的Bean来说，BeanFactory会缓存Bean实例，所以第二次使用getBean时直接从IoC容器缓存中获取Bean。

以上两种方法是可以转换的。

在配置文件中，如果在bean标签内添加  lazy-init="true",那么bean在容器初始化的时候将不会被实例化 ，则变成懒加载。如果lazy-init="false",那么bean在容器初始化的时候将自动实例化，不再是懒加载。 （当遇到多个bean，如配置文件中有100个bean的话，这样每个bean添加lazy-init="true",效率太低下，可以在配置文件xml的标签头部添加default-lazy-init="false" ，则配置文件中的所有bean都不再是懒加载）。

**BeanFactory**

BeanFactory 是 Spring 的“心脏”。它就是 Spring IoC 容器的真面目。Spring 使用 BeanFactory 来实例化、配置和管理 Bean。

BeanFactory：是IOC容器的核心接口， 它定义了IOC的基本功能，我们看到它主要定义了getBean方法。getBean方法是IOC容器获取bean对象和引发依赖注入的起点。方法的功能是返回特定的名称的Bean。

BeanFactory 是初始化 Bean 和调用它们生命周期方法的“吃苦耐劳者”。注意，BeanFactory 只能管理单例（Singleton）Bean 的生命周期。它不能管理原型(prototype,非单例)Bean 的生命周期。这是因为原型 Bean 实例被创建之后便被传给了客户端,容器失去了对它们的引用。

```java
package org.springframework.beans.factory;
//典型的工厂模式的工厂接口。
public interface BeanFactory {

    /**
     * 用来引用一个实例，或把它和工厂产生的Bean区分开，就是说，如果一个FactoryBean的名字为a，那么，&a会得到那个Factory
     */
    String FACTORY_BEAN_PREFIX = "&";

    /*
     * 四个不同形式的getBean方法，获取实例
     */
    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException;

    boolean containsBean(String name); // 是否存在

    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;// 是否为单实例

    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;// 是否为原型（多实例）

    boolean isTypeMatch(String name, Class<?> targetType)
            throws NoSuchBeanDefinitionException;// 名称、类型是否匹配

    Class<?> getType(String name) throws NoSuchBeanDefinitionException; // 获取类型

    String[] getAliases(String name);// 根据实例的名字获取实例的别名

}
```

1. XmlBeanFactory通过Resource装载Spring配置信息冰启动IoC容器，然后就可以通过factory.getBean从IoC容器中获取Bean了。
2. 通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化动作发生在第一个调用时。
3. 对于单实例（singleton）的Bean来说，BeanFactory会缓存Bean实例，所以第二次使用getBean时直接从IoC容器缓存中获取Bean。

**ApplicationContext**

如果说BeanFactory是Spring的心脏，那么ApplicationContext就是完整的躯体了，ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的功能。在BeanFactory中，很多功能需要以编程的方式实现，而在ApplicationContext中则可以通过配置实现。

BeanFactorty接口提供了配置框架及基本功能，但是无法支持spring的aop功能和web应用。而ApplicationContext接口作为BeanFactory的派生，因而提供BeanFactory所有的功能。而且ApplicationContext还在功能上做了扩展，相较于BeanFactorty，ApplicationContext还提供了以下的功能： 

1. MessageSource, 提供国际化的消息访问  
2. 资源访问，如URL和文件  
3. 事件传播特性，即支持aop特性
4. 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层 

ApplicationContext：是IOC容器另一个重要接口， 它继承了BeanFactory的基本功能，  同时也继承了容器的高级功能，如：MessageSource（国际化资源接口）、ResourceLoader（资源加载接口）、ApplicationEventPublisher（应用事件发布接口）等。

Spring自带了多种类型的应用上下文。下面罗列的几个是你最有可能遇到的。

- AnnotationConfigApplicationContext:从一个或多个基于Java的配置类中加载Spring应用上下文。
- AnnotationConfigWebApplicationContext:从一个或多个基于Java的配置类中加载Spring Web应用上下文。
- ClassPathXmlApplicationContext:从类路径下的一个或多个XML配置文件中加载上下文定义,把应用上下文定义文件作为类资源。
- FileSystemXmlapplicationcontext:从文件系统下的一个或多个XML配置文件中加载上下文定义。
- XmlWebApplicationContext:从Web应用下的一个或多个XML配置文件中加载上下文定义。

无论是从文件系统中装载应用上下文还是从类路径下装载应用上下文,将bean加载到bean工厂的过程都是相似的。例如,如下代码展示了如何加载一个FileSystemXmlApplicationContext:

```java
ApplicationContext context=new 
    FileSystemXmlApplicationContext("/home/hyp/knight.xml");
ApplicationContext context=new 
    ClassPathXmlApplicationContext("knight.xml");
```

使用FileSystemXmlApplicationContext和使用ClassPathXmlApplicationContext的区别在于:

- FileSystemXmlApplicationContext在指定的文件系统路径下查找knight.xml文件;

- 而ClassPathXmlApplicationContext是在所有的类路径(包含JAR文件)下查找knight.xml文件。

如果你想从Java配置中加载应用上下文,那么可以使用AnnotationConfigApplicationContext:

```java
ApplicationContext context=new 
    AnnotationConfigApplicationContext(com.hyp.learn.config.KnightConfig.class);
```

XmlWebApplicationContext：从web应用的根目录读取配置文件，需要先在web.xml中配置，可以配置监听器或者servlet来实现

```xml
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

或(spring 5已去除)

```xml
<servlet>
<servlet-name>context</servlet-name>
<servlet-class>org.springframework.web.context.ContextLoaderServlet</servlet-class>
<load-on-startup>1</load-on-startup>
</servlet>
```

这两种方式都默认配置文件为web-inf/applicationContext.xml，也可使用context-param指定配置文件

```xml
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>/WEB-INF/myApplicationContext.xml</param-value>
</context-param>
```

**两种容器的区别:**

1. BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化，这样，我们就不能发现一些存在的Spring的配置问题。而ApplicationContext则相反，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误。 相对于基本的BeanFactory，ApplicationContext  唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

   BeanFacotry延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；而ApplicationContext则在初始化自身是检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用  ApplicationContext。
   应用上下文则会在上下文启动后预载入所有的单实例Bean。通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

2. BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

   （Applicationcontext比  beanFactory 加入了一些更好使用的功能。而且 beanFactory 的许多功能需要通过编程实现而  Applicationcontext 可以通过配置实现。比如后处理 bean ， Applicationcontext 直接配置在配置文件即可而  beanFactory 这要在代码中显示的写出来才可以被容器识别。 ）

3. beanFactory主要是面对与 spring 框架的基础设施，面对 spring 自己。而  Applicationcontex 主要面对与 spring 使用的开发者。基本都会使用 Applicationcontex 并非  beanFactory 。

#### 总结

作用：

1. BeanFactory负责读取bean配置文档，管理bean的加载，实例化，维护bean之间的依赖关系，负责bean的声明周期。

2. ApplicationContext除了提供上述BeanFactory所能提供的功能之外，还提供了更完整的框架功能：
   a. 国际化支持
   b. 资源访问：Resource rs = ctx. getResource(“classpath:config.properties”), “file:c:/config.properties”
   c. 事件传递：通过实现ApplicationContextAware接口

### Spring Bean

Spring Bean是事物处理组件类和实体类（POJO）对象的总称，Spring Bean被Spring IOC容器初始化，装配和管理。

Spring 容器会自动完成bean对象的实例化。

创建应用对象之间的协作关系的行为称为：**装配(wiring)**，这就是依赖注入的本质。

#### Bean配置

主要包括：

- 基于XML的配置方式

- 基于注解的配置方式

- 基于Java类的配置方式

##### 基于XML的配置

```xml
<bean id=“loginUserDao” class=“com.chinalife.dao.impl.LoginUserDaoImpl”  
        lazy-init=“true” init-method=“myInit” destroy-method=“myDestroy”  
        scope=“prototype”>  
        ……   
</bean>
```

在XML配置中，通过<bean\> </bean\>来定义Bean，通过id或name属性定义Bean的名称，如果未指定id和name属性，Spring则自动将全限定类名作为Bean的名称。通过<property\>子元素或者p命名空间的动态属性为Bean注入值。还可以通过<bean\>的init-method和destory-method属性指定Bean实现类的方法名来设置生命过程方法（最多指定一个初始化方法和销毁方法）。通过<bean\>的scope指定Bean的作用范围。听过<bean\>的lazy-init属性指定是否延迟初始化。

**当Bean的实现类来源于第三方类库**，比如DataSource、HibernateTemplate等，无法在类中标注注解信息，只能通过XML进行配置；而且命名空间的配置，比如aop、context等，也只能采用基于XML的配置。(也可以通过配置类进行配置)

##### 基于注解的配置

```java
@Scope(“prototype”)   
@Lazy(true)   
@Component(“loginUserDao”)   
public class LoginUserDao {   
    ……   
    // 用于设置初始化方法   
    @PostConstruct  
    public void myInit() {   
  
    }   
  
    // 用于设置销毁方法   
    @PreDestroy  
    public void myDestroy() {   
    }   
}

```

在Bean实现类中通过一些Annotation来标注Bean类：

- @Component：标注一个普通的Spring Bean类（可以指定Bean名称，未指定时默认为小写字母开头的类名）
- @Controller：标注一个控制器类
- @Service：标注一个业务逻辑类

- @Repository：标注一个DAO类

通过在成员变量或者方法入参处标注@Autowired按类型匹配注入，也可以使用@Qualifier按名称配置注入。通过在方法上标注@PostConstrut和PreDestroy注解指定的初始化方法和销毁方法（可以定义任意多个）。通过@Scope(“prototype”)指定Bean的作用范围。通过在类定义处标注@Lazy(true)指定Bean的延迟加载。

**当Bean的实现类是当前项目开发的**，可以直接在Java类中使用基于注解的配置，配置比较简单。

##### 基于Java类配置

```java
@Configuration  
public class Conf {   
    	//给容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id
    @Scope(“prototype”)   
    @Bean(“loginUserDao”)   
    public LoginUserDao loginUserDao() {   
        return new LoginUserDao();   
    }   
}
```

在标注了@Configuration的java类中，通过在类方法标注@Bean定义一个Bean。方法必须提供Bean的实例化逻辑。通过@Bean的name属性可以定义Bean的名称，未指定时默认名称为方法名。

在方法处通过@Autowired使方法入参绑定Bean，然后在方法中通过代码进行注入；也可以调用配置类的@Bean方法进行注入。通过@Bean的initMethod或destroyMethod指定一个初始化或者销毁方法。通过Bean方法定义处标注@Scope指定Bean的作用范围。通过在Bean方法定义处标注@Lazy指定Bean的延迟初始化。

**当实例化Bean的逻辑比较复杂时**，则比较适合基于Java类配置的方式。

#### Bean注入

Spring框架的核心功能之一就是通过依赖注入的方式来管理Bean之间的依赖关系。

了解过设计模式的朋友肯定知道工厂模式吧，即所有的对象的创建都交给工厂来完成，是一个典型的面向接口编程。这比直接用new直接创建对象更合理，因为直接用new创建对象，会导致调用者与被调用者的硬编码耦合；而工厂模式，则把责任转向了工厂，形成调用者与被调用者的接口的耦合，这样就避免了类层次的硬编码耦合。这样的工厂模式确实比传统创建对象好很多。但是，正如之前所说的，工厂模式只是把责任推给了工厂，造成了调用者与被调用者工厂的耦合。

Spring框架则避免了调用者与工厂之间的耦合，通过spring容器“宏观调控”，调用者只要被动接受spring容器为调用者的成员变量赋值即可，而不需要主动获取被依赖对象。这种被动获取的方式就叫做依赖注入，又叫控制反转。依赖注入又分为设值注入和构造注入。而spring框架则负责通过配置xml文件来实现依赖注入。而设值注入和构造注入则通过配置上的差异来区分。

Spring 框架支持三种依赖注入方法：属性注入、构造函数注入以及工厂方法注入。

##### 属性注入

属性注入即通过 setXxx()  方法注入 Bean 的属性值或者依赖对象，由于属性注入方式具有可选择性和灵活性高的优点，因此属性注入是实际项目中最常采用的注入方式 。

设值注入是IoC容器使用成员变量的setter方法来注入被依赖对象。实现过程如下所示。整个过程采用面向接口编程，尽量降低类层次的耦合。

1. 首先建立一个Creature接口，然后实现Person类，用于操作其它Bean类，便于setter操作。

   ```java
   
   public interface Creature {
    
   	public void useTool();
   }
   
   public class Person implements Creature
   {
   	private Axe axe;
   	private Metal metal;
   	// 设值注入所需的setter方法
   	public void setMetal(Metal metal)
   	{
   		this.metal=metal;
   		
   	}
   	public void setAxe(Axe axe)
   	{
   		this.axe=axe;
   	}
   	public void useTool()
   	{
   		System.out.println("我打算去砍点柴火！");
   		// 调用axe的chop()方法，metal的make()方法
   		// 表明Person对象依赖于axe对象，metal对象
   		System.out.println(metal.make());
   		System.out.println(axe.chop());
   	}
   }
   ```

   Person类中有两个setter函数，属性都是组件属性，因此还需要分别建立Axe接口和Metal接口以及实现类，便于Person类的对象引用。另外Person是实现类，而且Axe接口和Metal接口以及它们的实现类，这里若传入组件参数，则类型应当是超类或者接口类型，这里用接口类型，实现面向接口编程。

   **注意：**默认构造函数是不带参数的构造函数 。Java 语言规定，如果类中没有定义任何构造函数，则 JVM 会自动为其生成一个默认的构造函数 。 反之，如果类中显式定义了构造函数，那么 JVM 就不会为其生成默认的构造函数 。

2. 建立Axe接口及其实现类StealMetal和Metal接口及其实现类Steal

   ```java
   
   public interface Axe {
   	public String chop();
    
   }
   
   public class StealAxe implements Axe {
    
   	@Override
   	public String chop() {
   		// TODO Auto-generated method stub
   		return "使用斧头砍柴";
   	}
    
   }
   
   public interface Metal {
   	public String make();
    
   }
   
   public class Steal implements Metal {
   	public String make()
   	{
   		return "使用铁做斧头";
   	}
    
   }
   
   
   ```

3. 进行xml文件配置

   观察Person类可以得知，有两个setter函数，说明有两个属性，因此进行如下配置。

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       <bean class="com.hyp.learn.demo03.StealAxe" id="stealAxe"/>
       <bean class="com.hyp.learn.demo03.Steal" id="steal"/>
       <bean class="com.hyp.learn.demo03.Person" id="person">
           <property name="axe" ref="stealAxe"/>
           <property name="metal" ref="steal"/>
       </bean>
   </beans>
   ```

   观察配置文件得知，该文件根元素是<beans.../>，而有多个子元素<bean.../>，每个<bean.../>对应一个类实例，对于每个类实例，若对应的类有setter函数，则用<property...>元素进行配置，如果有多个setter函数，则如上述配置文件一样，可配置多个<property.../>元素。其中<property.../>name是指属性名，即setter函数参数名，ref则是指传入的参数，ref一般是配置文件中出现的id。

   而<bean.../>配置中，id则是代表该类实例的标识属性，class则是该实例对应的类一般包含包路径，并且只能是实现类，不能是接口。另外还可以设置延迟初始化，因为依赖注入有预初始化功能，如果不需要则可以设置lazy-init="true"来实现。若限定bean的作用域，则用scope元素，默认的作用域是singleton即单例模式，而prototype是指每次通过容器的getBean()方法获得的bean都是一个新的实例，常用的作用域就是这两种。

   **注意：** Spring 只会检查 Bean 中是否有对应的 Setter 方法，至于 Bean 中是否有对应的属性变量则不做要求，因为它底层是采用 Java 的反射方法来设置值的呀O(∩_∩)O哈哈~

4. 接下来编写测试类

   ```java
   public class TestDemo03 {
   
       @Test
       public void demo03(){
           ApplicationContext context=new ClassPathXmlApplicationContext(
                   "applicationContext03.xml"
           );
           Creature person = context.getBean("person", Creature.class);
           person.useTool();
       }
   
   }
   ```

5. 运行结果如下：

   ```
   我打算去砍点柴火！
   使用铁做斧头
   使用斧头砍柴
   ```

**Java Bean 属性命名规范**

Java Bean 属性命名规范是 xxx 的属性，它对应着的是 setXxx() 方法。

一般情况下，Java 的属性变量名都以小写字母起头。但也存在特殊情况 , 比如一些具有特定意义的大写英文缩略词（如 JSON、AJAX等）。 Java Bean 也允许大写字母起头的属性变量名 , 不过必须满足 " **变量的前两个字母要么全部大写 , 要么全部小写** " 的要求 。如 :   IDCard 的属性名就是合法的，但 iDCard 属性名则是非法的，虽然 Spring 框架仍然可以正确注入，但不推荐。

所以，为了简便起见，对于以大写英文缩略词出现的专业术语，一律将其调整为小写形式，如 id 等，这样可以保证命名的统一性。

##### 构造注入

构造注入是IoC容器使用构造器来注入被依赖对象。具体操作如下所示。还是使用上面那个示例。

使用构造函数注入的前提是 Bean 必须提供带参的构造函数。

构造注入与设值注入主要区别就是setter函数变成了构造函数，所以上面示例只需修改Person类即可。用构造函数替代setter函数。

```java
public class Person implements Creature {
    private Axe axe;
    private Metal metal;

    public Person(Axe axe, Metal metal) {
        this.axe = axe;
        this.metal = metal;
    }



    public void useTool()
    {
        System.out.println("我打算去砍点柴火！");
        // 调用axe的chop()方法，metal的make()方法
        // 表明Person对象依赖于axe对象，metal对象
        System.out.println(metal.make());
        System.out.println(axe.chop());
    }
}

```

另外还需要作出修改的便是beans.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.hyp.learn.demo04.StealAxe" id="stealAxe"/>
    <bean class="com.hyp.learn.demo04.Steal" id="steal"/>
    <bean class="com.hyp.learn.demo04.Person" id="person">
       <constructor-arg name="axe"  index="0" ref="stealAxe"/>
        <constructor-arg name="metal"  index="1" ref="steal"/>
    </bean>

</beans>
```

如上配置文件所示：设值注入是用<property.../>元素配置将参数传入从而激活相应属性的setter函数，而构造注入则使用<constructor-arg.../>进行配置激活构造函数进行初始化。若出现多个构造函数，则只需对person类多构造几个实例即可。

1. 按类型匹配入参

   在 <constructor-arg\> 的元素中有一个 type 属性，它表示构造函数中参数的类型，这是 spring 用来判断配置项与构造函数入参对应关系的 “桥梁”。

   ```xml
   <constructor-arg type="com.hyp.learn.demo04.Axe" ref="stealAxe"/>
    <constructor-arg type="com.hyp.learn.demo04.Metal" ref="steal"/>
   ```

2. 按索引匹配入参

   Spring 底层是采用 Java 反射能力来实现依赖注入的，但 Java 反射无法获知构造函数的参数名，所以只能通过入参类型与索引信息来间接地确定构造函数的配置项与入参之间的对应关系。

   构造函数的第一个参数索引为 0，第二个为 1，以此类推。

   ```xml
    <constructor-arg index="0" ref="stealAxe"/>
   <constructor-arg index="1" ref="steal"/>
   ```

3. 联合使用类型匹配入参与索引匹配入参

   有时需要联合使用 type 和 index 属性才能确定匹配项和构造函数入参的对应关系：

   对于由于参数数目相同而类型不同所引起的潜在配置歧义问题， Spring 容器可以正确启动但不会给出报错信息，它将随机采用一个匹配的构造函数来实例化 Bean ，而被选择的构造函数可能并不是我们所希望的 。 因此，在配置时必须小心谨慎，以避免发生意外。

   ```xml
    <constructor-arg index="0" ref="stealAxe"/>
   <constructor-arg index="1" type="com.hyp.learn.demo04.Metal" ref="steal"/>
   ```

   建议显式地指定 index 与 type，因为这样可以明确指定 Bean 中的一个构造函数。

4. 通过自身类型反射匹配入参

   如果 Bean 构造函数的入参类型不是基础数据类型，而且入参类型各异，那么可以通过 Java 反射机制获取构造函数的入参类型。

   ```xml
   <constructor-arg  ref="stealAxe"/>
   <constructor-arg  ref="steal"/>
   ```

**注意：**Spring 的配置采用了与元素标签顺序有关的策略，所以上面的多个  `<constructor-arg>` 位置并会对配置效果产生影响。

其余函数以及接口都不变，包括测试类类都不变，输出结果也相同。

**循环依赖**

实例化构造函数配置的 Bean 的前提条件是：这个 Bean 构造函数的入参对象必须已准备就绪（已实例化）。如果存在两个 Bean，它们都使用构造函数的注入方式，并且它们的入参互相引用的话，就会发生循环依赖的问题。

假设 Book 中拥有 Author：

```java
public class Book {
    private String name;
    /**
     * 作者
     */
    private Author author;

    /**
     * 出版社
     */
    private String press;

    public Book() {
    }

    public Book(String name, String press) {
        this.name = name;
        this.press = press;
    }

    public Book(String name, Author author) {
        this.name = name;
        this.author = author;
    }


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Author getAuthor() {
        return author;
    }

    public void setAuthor(Author author) {
        this.author = author;
    }

     @Override
    public String toString() {
        return "Book{" +
                "name='" + name + '\'' +
                ", author=" + author +
                ", press='" + press + '\'' +
                '}';
    }

    public String getPress() {
        return press;
    }

    public void setPress(String press) {
        this.press = press;
    }
}

```

而 Author 中拥有 Book：

```java
public class Author {
    private Book book;
    private String name;

    public Author() {
    }

    public Author(String name, Book book) {
        this.name = name;
        this.book = book;
    }

    public Book getBook() {
        return book;
    }

    public void setBook(Book book) {
        this.book = book;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }


    @Override
    public String toString() {
        return "Author{" +
                "book=" + book.getName() +
                ", name='" + name + '\'' +
                '}';
    }
}

```

配置文件中，Book 与 Author 都采用构造函数的依赖注入方式：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.hyp.learn.demo05.Book" id="book">
        <constructor-arg name="author" ref="author"/>
        <constructor-arg name="name" value="郭与于"/>
    </bean>
    <bean class="com.hyp.learn.demo05.Author" id="author">
        <constructor-arg name="name" value="王钢蛋"/>
        <constructor-arg name="book" ref="book"/>
    </bean>

</beans>
```

测试文件：

```java
public class TestDemo05 {
    @Test
    public void Demo05(){
        ApplicationContext context=new ClassPathXmlApplicationContext(
                "applicationContext05.xml"
        );
        Book book = context.getBean("book", Book.class);
        System.out.println(book);
    }

}
```

报错，因为存在循环依赖的问题。

我们可以把它们的注入方式都改为属性注入方式，就可以解析上述的问题啦O(∩_∩)O哈哈~

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.hyp.learn.demo05.Book" id="book">
        <property name="name" value="《郭与于》"/>
        <property name="author" ref="author"/>
    </bean>
    <bean class="com.hyp.learn.demo05.Author" id="author">
        <property name="book" ref="book"/>
        <property name="name" value="王钢蛋"/>
    </bean>

</beans>
```

##### 工厂方法注入

工厂方法是被经常使用到的设计模式，它也是控制反转与单实例设计思想的主要实现方式。它已经成为 Spring 框架的底层基础设施的一部分。

**非静态工厂方法**

非静态，即必须实例化工厂类之后，才能调用其方法。

```java
public class BookFactory {

    public Book create() {
        Book book = new Book("面纱", "重庆出版社");
        Author author = new Author("王钢蛋", book);
        book.setAuthor(author);
        return book;
    }
}
```

工厂类负责创建目标类实例，它对外屏蔽了目标类的实例化细节，返回的类型一般是接口或抽象类的形式。

```xml
 <bean class="com.hyp.learn.demo05.BookFactory" id="bookFactory"/>
<!-- factory-bean：指定工厂类；factory-method：指定工厂方法-->
<bean id="book11" factory-bean="bookFactory" factory-method="create"/>
```

**静态工厂方法**

很多工厂方法是以静态的方式实现的，因为更易使用，直接调用方法就可以啦。

```java
public class BookFactory {
    public Book create() {
        return new Book("面纱", "重庆出版社");
    }

    public static Book createBook() {
        return new Book("面纱", "重庆出版社");
    }
}
```

xml配置为：

```xml
<!--class：指定工厂类； factory-method：指定静态工厂方法 -->
<bean id="book12" class="com.hyp.learn.demo05.BookFactory" factory-method="createBook"/>
```

**区别**

设值注入：

- 与传统的Javabean的写法更相似，通过setter方法设定依赖关系显得更加直观自然

- 对于复杂的依赖关系，如果采用构造注入，会导致构造器过于臃肿

- 多参数情况下使得构造器变得更加笨重

构造注入：

- 构造注入可以在构造器中决定依赖关系的注入顺序

- 对于依赖关系无须变化的bean，构造注入更有用处，无须担心后续代码对依赖关系的破坏

- 依赖关系只能在构造器中设定，更符合高内聚的原则

**选择**

支持构造函数注入的理由有这些：

- 构造函数可以保证一些重要的属性在 Bean 实例化时就设置好，避免因为一些重要属性没有提供而导致一个无用 Bean 实例出现的情况 。
- 不需要为每个属性提供 Setter 方法，减少了类的方法总数 。
- 可以更好的封装变量，不需要为每个属性指定 Setter 方法，避免外部错误的调用 。

更多的开发者更倾向于使用属性注入的方式，他们反对构造函数注入方式的理由是：

- 如果一个类的属性众多，那么构造函数的签名将变成一个庞然大物，可读性与可维护性都很差。
- 灵活性不强，在有些属性是可选的情况下，如果通过构造函数注入，也需要为非必填的参数提供一个 null 值。
- 如果有多个构造函数，那么需要考虑配置文件和具体构造函数匹配歧义的问题，配置上相对复杂。
- 构造函数不利于类的继承和扩展，因为子类需要引用父类复杂的构造函数。
- 构造函数注入有时会造成循环依赖的问题。

**建议一般情况下，使用属性注入；必要情况下，使用构造函数注入；只有在不得已的情况下，才使用工厂方法注入。**

#### bean获取

- 在初始化时保存ApplicationContext对象
- 通过Spring提供的utils类获取ApplicationContext对象 
- 继承自抽象类ApplicationObjectSupport 
- 继承自抽象类WebApplicationObjectSupport 
- 实现接口ApplicationContextAware 
- 通过Spring提供的ContextLoader 

 **获取spring中bean的方式总结：**  

1. 在初始化时保存ApplicationContext对象 

   ```java
   ApplicationContext ac = new FileSystemXmlApplicationContext("applicationContext.xml"); 
   ac.getBean("beanId");
   ```

   说明：这种方式适用于采用Spring框架的独立应用程序，需要程序通过配置文件手工初始化Spring的情况。 

2. 通过Spring提供的工具类获取ApplicationContext对象 

   ```java
   ApplicationContext ac1 = WebApplicationContextUtils.getRequiredWebApplicationContext(ServletContext sc); 
   ApplicationContext ac2 = WebApplicationContextUtils.getWebApplicationContext(ServletContext sc); 
   ac1.getBean("beanId"); 
   ac2.getBean("beanId");
   ```

   说明：这种方式适合于采用Spring框架的B/S系统，通过ServletContext对象获取ApplicationContext对象，然后在通过它获取需要的类实例。上面两个工具方式的区别是，前者在获取失败时抛出异常，后者返回null。 

3. 继承自抽象类ApplicationObjectSupport

   说明：抽象类ApplicationObjectSupport提供getApplicationContext()方法，可以方便的获取ApplicationContext。

   Spring初始化时，会通过该抽象类的setApplicationContext(ApplicationContext context)方法将ApplicationContext 对象注入。

4. 继承自抽象类WebApplicationObjectSupport

   说明：类似上面方法，调用getWebApplicationContext()获取WebApplicationContext

5. 实现接口ApplicationContextAware

   说明：实现该接口的setApplicationContext(ApplicationContext context)方法，并保存ApplicationContext 对象。Spring初始化时，会通过该方法将ApplicationContext对象注入。

   以下是实现ApplicationContextAware接口方式的代码，前面两种方法类似： 

   ```java
       public class SpringContextUtil implements ApplicationContextAware {  
           // Spring应用上下文环境  
           private static ApplicationContext applicationContext;  
           /** 
            * 实现ApplicationContextAware接口的回调方法，设置上下文环境 
            *  
            * @param applicationContext 
            */  
           public void setApplicationContext(ApplicationContext applicationContext) {  
               SpringContextUtil.applicationContext = applicationContext;  
           }  
           /** 
            * @return ApplicationContext 
            */  
           public static ApplicationContext getApplicationContext() {  
               return applicationContext;  
           }  
           /** 
            * 获取对象 
            *  
            * @param name 
            * @return Object
            * @throws BeansException 
            */  
           public static Object getBean(String name) throws BeansException {  
               return applicationContext.getBean(name);  
           }  
       }
   ```

   虽然，spring提供的后三种方法可以实现在普通的类中继承或实现相应的类或接口来获取spring 
   的ApplicationContext对象，但是在使用是一定要注意实现了这些类或接口的普通java类一定要在Spring 
   的配置文件applicationContext.xml文件中进行配置。否则获取的ApplicationContext对象将为null。 

6. 通过Spring提供的ContextLoader 

   ```java
   WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
   wac.getBean(beanID);
   ```

   最后提供一种不依赖于servlet,不需要注入的方式。但是需要注意一点，在服务器启动时，Spring容器初始化时，不能通过以下方法获取Spring 容器，细节可以查看spring源码org.springframework.web.context.ContextLoader。 