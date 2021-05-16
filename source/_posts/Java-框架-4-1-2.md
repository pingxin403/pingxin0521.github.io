---
title: Spring  依赖注入（二）
date: 2019-05-22 14:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

#### Bean 生命周期

<!--more-->

![lJXNVJ.png](https://s2.ax1x.com/2020/01/02/lJXNVJ.png)

可以简述为以下九步

1. **BeanFactoryPostProcessor**的postProcessorBeanFactory()方法：若某个IoC容器内添加了实现了BeanFactoryPostProcessor接口的实现类Bean，那么在该容器中实例化任何其他Bean之前可以回调该Bean中的postPrcessorBeanFactory()方法来对Bean的配置元数据进行更改，比如从XML配置文件中获取到的配置信息。
2. Bean的实例化：Bean的实例化是使用反射实现的。
3. Bean属性注入：Bean实例化完成后，利用反射技术实现属性及依赖Bean的注入。
4. **BeanNameAware**的setBeanName()方法：如果某个Bean实现了BeanNameAware接口，那么Spring将会将Bean实例的ID传递给setBeanName()方法，在Bean类中新增一个beanName字段，并实现setBeanName()方法。
5. **BeanFactoryAware**的setBeanFactory()方法：如果某个Bean实现了BeanFactoryAware接口，那么Spring将会将创建Bean的BeanFactory传递给setBeanFactory()方法，在Bean类中新增了一个beanFactory字段用来保存BeanFactory的值，并实现setBeanFactory()方法。
6. **ApplicationContextAware**的setApplicationContext()方法：如果某个Bean实现了ApplicationContextAware接口，那么Spring将会将该Bean所在的上下文环境ApplicationContext传递给setApplicationContext()方法，在Bean类中新增一个ApplicationContext字段用来保存ApplicationContext的值，并实现setApplicationContext()方法。
7. **BeanPostProcessor**预初始化方法：如果某个IoC容器中增加的实现BeanPostProcessor接口的实现类Bean，那么在该容器中实例化Bean之后，执行初始化之前会调用BeanPostProcessor中的postProcessBeforeInitialization()方法执行预初始化处理。
8. **InitializingBean**的afterPropertiesSet()方法：如果Bean实现了InitializingBean接口，那么Bean在实例化完成后将会执行接口中的afterPropertiesSet()方法来进行初始化。
9. 自定义的inti-method指定的方法：如果配置文件中使用init-method属性指定了初始化方法，那么Bean在实例化完成后将会调用该属性指定的初始化方法进行Bean的初始化。
10. **BeanPostProcessor**初始化后方法：如果某个IoC容器中增加的实现BeanPostProcessor接口的实现类Bean，那么在该容器中实例化Bean之后并且完成初始化调用后执行该接口中的postProcessorAfterInitialization()方法进行初始化后处理。
11. 使用Bean：此时有关Bean的所有准备工作均已完成，Bean可以被程序使用了，它们将会一直驻留在应用上下文中，直到该上下文环境被销毁。
12. **DisposableBean**的destory()方法：如果Bean实现了DisposableBean接口，Spring将会在Bean实例销毁之前调用该接口的destory()方法，来完成一些销毁之前的处理工作。
13. 自定义的destory-method指定的方法：如果在配置文件中使用destory-method指定了销毁方法，那么在Bean实例销毁之前会调用该指定的方法完成一些销毁之前的处理工作。

**注意**：

1. BeanFactoryPostProcessor接口与BeanPostProcessor接口的作用范围是整个上下文环境中，使用方法是单独新增一个类来实现这些接口，那么在处理其他Bean的某些时刻就会回调响应的接口中的方法。

2. BeanNameAware、BeanFactoryAware、ApplicationContextAware的作用范围的Bean范围，即仅仅对实现了该接口的指定Bean有效，所有其使用方法是在要使用该功能的Bean自己来实现该接口。

3. 第8点与第9点所述的两个初始化方法作用是一样的，我们完全可以使用其中的一种即可，一般情况我们使用第9点所述的方式，尽量少的去来Bean中实现某些接口，保持其独立性，低耦合性，尽量不要与Spring代码耦合在一起。第12和第13也是如此。

**先看一个最简单的一生(没有使用Bean的后置处理器)**

```java
/**
 *    Student.java
 * @Description:一个学生类(Bean)，能体现其生命周期的Bean
 */
public class Student implements BeanNameAware {
	private String name;

	//无参构造方法
	public Student() {
		super();
	}

	/** 设置对象属性
	 * @param name the name to set
	 */
	public void setName(String name) {
		System.out.println("设置对象属性setName()..");
		this.name = name;
	}
	
	//Bean的初始化方法
	public void initStudent() {
		System.out.println("Student这个Bean：初始化");
	}
	
	//Bean的销毁方法
	public void destroyStudent() {
		System.out.println("Student这个Bean：销毁");
	}
	
	//Bean的使用
	public void play() {
		System.out.println("Student这个Bean：使用");
	}

	/* 重写toString
	 * @see java.lang.Object#toString()
	 */
	@Override
	public String toString() {
		return "Student [name = " + name + "]";
	}

	//调用BeanNameAware的setBeanName()
	//传递Bean的ID。
	@Override
	public void setBeanName(String name) {
		System.out.println("调用BeanNameAware的setBeanName()..." ); 
	}
	
}

```

xml配置applicationContext.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
 <!-- init-method：指定初始化的方法
         destroy-method：指定销毁的方法 -->
    <bean id="student" class="com.hyp.learn.demo06.Student" 
          init-method="initStudent" destroy-method="destroyStudent">
        <property name="name" value="pingxin"/>
    </bean>
</beans>
```

测试类:

```java
    @Test
 	public void test() {
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		Student student = (Student) context.getBean("student");
		//Bean的使用
		student.play();
		System.out.println(student);
		//关闭容器
		((AbstractApplicationContext) context).close();
	}

```

输出如下：

```
6月 08, 2019 5:19:55 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@6cc7b4de: startup date [Sat Jun 08 17:19:55 CST 2019]; root of context hierarchy
6月 08, 2019 5:19:56 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [applicationContext.xml]
设置对象属性setName()..
调用BeanNameAware的setBeanName()...
Student这个Bean：初始化
Student这个Bean：使用
Student [name = pingxin]
6月 08, 2019 5:19:56 下午 org.springframework.context.support.ClassPathXmlApplicationContext doClose
信息: Closing org.springframework.context.support.ClassPathXmlApplicationContext@6cc7b4de: startup date [Sat Jun 08 17:19:55 CST 2019]; root of context hierarchy
Student这个Bean：销毁
```

可以在输出结果看出bean的一生，完全与之前的一生过程图相符(除了bean后置处理器部分)，这里还需要提及的是在xml配置中的两个属性

- init-method：指定初始化的方法
- destroy-method：指定销毁的方法

说到init-method和destroy-method，当然也要提及一下在< beans>的属性

- default-init-method：为应用上下文中所有的Bean设置了共同的初始化方法
- default-destroy-method：为应用上下文中所有的Bean设置了共同的销毁方法

**Bean的后置处理器**

上面bean的一生其实已经算是对bean生命周期很完整的解释了，然而bean的后置处理器，是为了对bean的一个增强

分别在Bean的初始化前后对Bean对象提供自己的实例化逻辑

```
- 实现BeanPostProcessor接口
	- postProcessBeforeInitialization方法
	- postProcessAfterInitialization方法
```

接上面的Student Demo，增加    MyBeanPostProcessor.java（实现BeanPostProcessor接口）

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

/**
 * bean的后置处理器
 * 分别在bean的初始化前后对bean对象提供自己的实例化逻辑
 * postProcessAfterInitialization：初始化之后对bean进行增强处理
 * postProcessBeforeInitialization：初始化之前对bean进行增强处理
 *
 */
public class MyBeanPostProcessor implements BeanPostProcessor {

    //对初始化之后的Bean进行处理
    //参数：bean：即将初始化的bean
    //参数：beanname：bean的名称
    //返回值：返回给用户的那个bean,可以修改bean也可以返回一个新的bean
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanname) throws BeansException {
        Student stu = null;
        System.out.println("对初始化之后的Bean进行处理,将Bean的成员变量的值修改了");
        if ("name".equals(beanname) || bean instanceof Student) {
            stu = (Student) bean;
            stu.setName("Jack");
        }
        return stu;
    }

    //对初始化之前的Bean进行处理
    //参数：bean：即将初始化的bean
    //参数：beanname：bean的名称
    //返回值：返回给用户的那个bean,可以修改bean也可以返回一个新的bean
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanname) throws BeansException {
        System.out.println("对初始化之前的Bean进行处理,此时我的名字" + bean);
        return bean;
    }

}
```

更改    applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- init-method：指定初始化的方法
         destroy-method：指定销毁的方法 -->
    <bean id="student" class="com.hyp.learn.demo06.Student"
          init-method="initStudent" destroy-method="destroyStudent">
        <property name="name" value="pingxin"/>
    </bean>

    <!-- 配置bean的后置处理器,不需要id，IoC容器自动识别是一个BeanPostProcessor -->
    <bean class="com.hyp.learn.demo06.MyBeanPostProcessor"/>
</beans>
```

测试类不变，运行后输出：

```
6月 08, 2019 5:28:52 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@6cc7b4de: startup date [Sat Jun 08 17:28:52 CST 2019]; root of context hierarchy
6月 08, 2019 5:28:52 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [applicationContext.xml]
设置对象属性setName()..
调用BeanNameAware的setBeanName()...
对初始化之前的Bean进行处理,此时我的名字Student [name = pingxin]
Student这个Bean：初始化
对初始化之后的Bean进行处理,将Bean的成员变量的值修改了
设置对象属性setName()..
Student这个Bean：使用
Student [name = Jack]
6月 08, 2019 5:28:52 下午 org.springframework.context.support.ClassPathXmlApplicationContext doClose
信息: Closing org.springframework.context.support.ClassPathXmlApplicationContext@6cc7b4de: startup date [Sat Jun 08 17:28:52 CST 2019]; root of context hierarchy
Student这个Bean：销毁
```

可以在applicationContext.xml中看到配置Bean后置处理器，不需要ID，只需要其全类名，因为IoC容器自动识别一个BeanPostProcessor

在控制台显示结果可以看出,Bean的后置处理器强大之处，可以对Bean实现自己想要做的事情，比如我这里的Demo就是在postProcessAfterInitialization方法中将成员变量name偷偷修改了，最后输出的就是偷偷修改之后的值

好了以上就是bean的一生，在控制台下将bean的一生映射出来，对理解bean的一生(生命周期)更加直观咯

##### Spring Aware接口

org.springframework.beans.factory.Aware使得自定义Bean可以识别利用Spring容器的资源，比如，

- BeanNameAware.setBeanName(String)，可以在Bean中得到它在IOC容器中的Bean的实例的名字。
- BeanFactoryAware.setBeanFactory(BeanFactory)，可以在Bean中得到Bean所在的IOC容器，从而直接在Bean中使用IOC容器的服务。
- ApplicationContextAware.setApplicationContext(ApplicationContext)，可以在Bean中得到Bean所在的应用上下文，从而直接在Bean中使用上下文的服务。
- MessageSourceAware，在Bean中可以得到消息源。
- ApplicationEventPublisherAware，在bean中可以得到应用上下文的事件发布器，从而可以在Bean中发布应用上下文的事件。
- ResourceLoaderAware，在Bean中可以得到ResourceLoader，从而在bean中使用ResourceLoader加载外部对应的Resource资源。

如非必要，Spring官方不推荐自定义Bean实现Aware接口，这会增加代码与Spring 框架的耦合性。

##### Spring PostProcessor

- BeanFactoryPostProcessor可以对bean的配置信息进行操作。结合Bean的生命周期，Spring IOC容器允许BeanFactoryPostProcessor读取配置信息并且能够在容器实例化任何其他bean之前改变配置信息。
- BeanPostProcessor可以在Spring完成Bean实例化，配置，初始化之后实现自己的业务逻辑。

BeanPostProcessor是存在于对象实例化阶段，而BeanFactoryPostProcessor则是存在于容器启动阶段

Spring Aware和PostProcessor都是提供了用户利用Spring IoC添加业务逻辑进一步定制Bean。

![UTOOLS1576404780606.png](https://i.loli.net/2019/12/15/1qHKSU8JFaRh5o4.png)

##### 补充

**Q:** Bean在什么时候会被创建？
**A:** 容器启动之后，并不会马上就实例化相应的bean定义。容器现在仅仅拥有所有对象的BeanDefinition来保存实例化阶段将要用的必要信息。只有当请求方通过BeanFactory的getBean()方法来请求某个对象实例的时候，才有可能触发Bean实例化阶段的活动。BeanFactory的getBean()方法可以被客户端对象显式调用，也可以在容器内部隐式地被调用。隐式调用有如下两种情况:

- BeanFactory来说，对象实例化默认采用延迟初始化。通常情况下，当对象A被请求而需要第一次实例化的时候，如果它所依赖的对象B之前同样没有被实例化，那么容器会先实例化对象A所依赖的对象。这时容器内部就会首先实例化对象B，以及对象 A依赖的其他还没有实例化的对象。这种情况是容器内部调用getBean()，对于本次请求的请求方是隐式的。
- ApplicationContext启动之后会实例化所有单例bean定义。但ApplicationContext在实现的过程中依然遵循Spring容器实现流程的两个阶段，只不过它会在启动阶段的活动完成之后，紧接着调用注册到该容器的所有bean定义的实例化方法getBean()。这就是为什么当你得到ApplicationContext类型的容器引用时，容器内所有对象已经被全部实例化完成。

之所以说getBean()方法是有可能触发Bean实例化阶段的活动，是因为只有当对应某个bean定义的getBean()方法第一次被调用时，不管是显式的还是隐式的，Bean实例化阶段的活动才会被触发，第二次被调用则会直接返回容器缓存的第一次实例化完的对象实例（prototype类型bean除外）。当getBean()方法内部发现该bean定义之前还没有被实例化之后，会通过createBean()方法来进行具体的对象实例化.

**Q:** Spring中的单例模式是否时线程安全的？

**A:** 关于单例bean的多线程行为，Spring框架没有做任何事情。开发人员有责任处理单例bean的并发问题和线程安全问题。 实际上，大多数Spring bean都没有可变状态，因此非常简单。但是如果你的bean具有可变状态，那么你需要确保线程安全。解决此问题最简单明了的方法是将可变bean的bean范围从“singleton”更改为“prototype".

**一些建议**

不建议使用InitializingBean, DisposableBean，因为这个会增加代码与Spring框架的耦合性。

@PostConstruct，@PreDestroy是JavaX的标准，而非Java，Spring定义的注解，使用时应该注意。

将Bean从Spring IoC中移除之前需要释放持有的资源，建议在destroy-method中写释放资源的代码

**应用**

了解Bean的生命周期有利于我们根据业务需要对Bean进行相关的拓展工作。

举个例子，在Bean初始化前希望用户名和密码。倘若将这些信息硬编码到工厂代码中显然是不安全的，一般地，公司会有一个统一的密码存储服务，通过调用开放的API获取用户名和密码。此时，我们可以在Bean的init-method中加入这样的逻辑：调用公司的密码存储服务API获取用户名和密码，加载至Bean的相关字段中。

再举个例子，在Bean被销毁之前，释放Bean与数据库的连接，那么我们可以把释放数据库连接这段逻辑放到destroy-method中。

**示例**

使用到的bean：

```java
public class MyBean implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, BeanClassLoaderAware,
        InitializingBean, DisposableBean {
    private String name;
    private String description;

    public MyBean() {
        System.out.println("**MyBean** construct.");
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("**MyBean** BeanClassLoaderAware.setBeanClassLoader: " + classLoader.getClass());
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("**MyBean** BeanFactoryAware.setBeanFactory: " + beanFactory.getClass());
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("**MyBean** BeanNameAware.getBeanName: " + s);
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("**MyBean** DisposableBean.destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("**MyBean** InitializingBean.afterPropertiesSet");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("**MyBean** ApplicationContextAware.setApplicationContext");
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("**MyBean** MyBean finalized.");
    }

    @PostConstruct
    public void springPostConstruct() {
        System.out.println("**MyBean** @PostConstruct");
    }

    // xml文件中的init-method
    public void myInitMethod() {
        System.out.println("**MyBean** init-method");
    }

    @PreDestroy
    public void springPreDestroy() {
        System.out.println("**MyBean** @PreDestroy");
    }

    // xml文件中的destroy-method
    public void mydestroyMethod() {
        System.out.println("**MyBean** destory-method");
    }

    @Autowired
    public void setName(String name) {
        this.name = name;
        System.out.println("**MyBean** setName");
    }

    @Autowired
    public void setDescription(String description) {
        this.description = description;
        System.out.println("**MyBean** setDescription");
    }

    @Override
    public String toString() {
        return "MyBean{" +
                "name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}

public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    public MyBeanFactoryPostProcessor() {
        super();
        System.out.println("[MyBeanFactoryPostProcessor] constructor");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("[MyBeanFactoryPostProcessor] postProcessBeanFactory");
    }
}

public class MyBeanPostProcessor implements BeanPostProcessor {
    public MyBeanPostProcessor() {
        super();
        System.out.println("[MyBeanPostProcessor] constructor.");
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("[MyBeanPostProcessor] postProcessBeforeInitialization: " + bean.getClass() + ": " + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("[MyBeanPostProcessor] postProcessAfterInitialization: " + bean.getClass() + ": " + beanName);
        return bean;
    }
}

public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
    public MyInstantiationAwareBeanPostProcessor() {
        super();
        System.out.println("[MyInstantiationAwareBeanPostProcessor] constructor.");
    }

    // 接口方法、实例化Bean之前调用
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        System.out.println("[MyInstantiationAwareBeanPostProcessor] postProcessBeforeInstantiation");
        return null;
    }

    // 接口方法、实例化Bean之前调用
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        System.out.println("[MyInstantiationAwareBeanPostProcessor] postProcessAfterInstantiation");
        return true;
    }

    // 接口方法、设置某个属性时调用
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
        System.out.println("[MyInstantiationAwareBeanPostProcessor] postProcessPropertyValues");
        // 注意返回结果，否则会无法正确Bean的属性
        return pvs;
    }
}

public class LifeCycleApplication {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("lifecycle.xml");
        System.out.println("[Application] before get bean");
        MyBean bean = (MyBean) context.getBean("myBean");
        System.out.println("[Application] after get bean");
        System.out.println(bean);
        ((ClassPathXmlApplicationContext) context).registerShutdownHook();
    }
}
```

配置文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.hyp.learn.spring.demo4"/>
    <bean id="myBean" class="com.hyp.learn.spring.demo4.MyBean" init-method="myInitMethod"
          destroy-method="mydestroyMethod">
         <property name="name" value="平心"/>
        <property name="description" value="lifecycle"/>
    </bean>
    <bean class="com.hyp.learn.spring.demo4.MyBeanPostProcessor"/>
    <bean class="com.hyp.learn.spring.demo4.MyBeanFactoryPostProcessor"/>
    <bean class="com.hyp.learn.spring.demo4.MyInstantiationAwareBeanPostProcessor"/>
</beans>

```

运行结果：

```
[MyBeanFactoryPostProcessor] constructor
[MyBeanFactoryPostProcessor] postProcessBeanFactory
[MyBeanPostProcessor] constructor.
[MyInstantiationAwareBeanPostProcessor] constructor.
[MyInstantiationAwareBeanPostProcessor] postProcessBeforeInstantiation
**MyBean** construct.
[MyInstantiationAwareBeanPostProcessor] postProcessAfterInstantiation
[MyInstantiationAwareBeanPostProcessor] postProcessPropertyValues
**MyBean** setName
**MyBean** setDescription
**MyBean** BeanNameAware.getBeanName: myBean
**MyBean** BeanClassLoaderAware.setBeanClassLoader: class sun.misc.Launcher$AppClassLoader
**MyBean** BeanFactoryAware.setBeanFactory: class org.springframework.beans.factory.support.DefaultListableBeanFactory
**MyBean** ApplicationContextAware.setApplicationContext
[MyBeanPostProcessor] postProcessBeforeInitialization: class com.hyp.learn.spring.demo4.MyBean: myBean
**MyBean** @PostConstruct
**MyBean** InitializingBean.afterPropertiesSet
**MyBean** init-method
[MyBeanPostProcessor] postProcessAfterInitialization: class com.hyp.learn.spring.demo4.MyBean: myBean
[Application] before get bean
[Application] after get bean
MyBean{name='平心', description='lifecycle'}
**MyBean** @PreDestroy
**MyBean** DisposableBean.destroy
**MyBean** destory-method
```

对照输出来理解，多想一些为什么。

![UTOOLS1576404780606.png](https://i.loli.net/2019/12/15/1qHKSU8JFaRh5o4.png)

#### Bean Scope

scope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入其相应的scope之前，生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁Spring 的 IoC 容器这些对象。

如果你不指定bean的scope，singleton便是容器默认的scope.

- singleton: 在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。

  标记为singleton的bean是由容器来保证这种类型的bean在同一个容器中只存在一个共享实例；而Singleton模式则是保证在同一个Classloader中只存在一个这种类型的实例。

- propertype: 针对声明为拥有propertype scope的bean定义，容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。

  对于那些请求方不能共享使用的对象类型，应该将其bean定义的scope设置为prototype。这样，每个请求方可以得到自己对应的一个对象实例。通常，声明为prototype的scope的bean定义类型，都是一些有状态的，比如保存每个顾客信息的对象。

另外三种scope类型，即request、session和global session类型,只有在支持Web应用的ApplicationContext中才能使用这三个scope。

- request: 为每一个HTTP请求创建一个全新的bean，当请求结束之后，该对象的生命周期即告结束。
- session: 为每一个独立的session创建一个全新的bean对象，session结束之后，该bean对象的生命周期即告结束。
- global session: global session只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的 session。如果在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session类型的scope对待。

#### p命名空间和c命名空间

在通过构造方法或`set`方法给`bean`注入关联项时通常是通过`constructor-arg`元素和`property`元素来定义的。在有了`p`命名空间和`c`命名空间时我们可以简单的把它们当做`bean`的一个属性来进行定义。

P名称空间用于简化set方法的属性赋值

C名称空间用于简化构造器的属性赋值

##### p命名空间

使用`p`命名空间时需要先声明使用对应的命名空间，即在`beans`元素上加入`xmlns:p="http://www.springframework.org/schema/p"`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

     <bean id="user" class="com.test.spring.aop.User"></bean>
    <bean id="userDao" class="com.test.spring.aop.UserDao"></bean>
    <bean id="userService" class="com.test.spring.aop.UserService">
        <property name="userName" value="小王"></property>
        <property name="userDao" ref="userDao"></property>
    </bean> 
    <!--<bean id="userService" class="com.test.spring.aop.UserService" p:userName="小王" p:userDao-ref="userDao"></bean>-->
</beans>
```

1. 引入schema

2. 引入属性值和引入一个对应不同，引入属性直接写 p:[属性名]=[属性值]；引入引用对象写：p[属性名-ref]=[属性值]

#####  c命名空间

c命名空间的用法和p命名空间类似，其对应于constructor-arg，即可以将constructor-arg元素替换为bean的一个以c命名空间前缀开始的属性。使用c命名空间之前也需要通过`xmlns:c=”http://www.springframework.org/schema/c”`进行声明。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans

         http://www.springframework.org/schema/beans/spring-beans.xsd">
        <!--配置年级对应的bean-->
    <bean id="grade" class="cn.bdqn.bean.Grade">
        <!--01.设置注入（推荐使用，便于阅读）在对应的类中必须有set方法，
              因为底层执行反射机制查阅类中的setXxx()-->
            <property name="gradeId" value="1"/>
            <property name="name" value="一年级"/>
        </bean>
   <!--
   配置 学生对应的bean 02.p命名空间赋值
   <bean id="student" class="cn.bdqn.bean.Student"
          p:age="18" p:name="小明" p:grade-ref="grade">
    </bean>-->
    <!--
    03.通过构造方法给属性赋值， 前提是  必须有对应的构造方法 不需要set（）
    <bean id="student" class="cn.bdqn.bean.Student">
            001:使用参数的下标从0开始
       <constructor-arg index="0" value="小花"/>
         <constructor-arg index="1" value="19"/>
         <constructor-arg index="2" ref="grade"/>
         002:使用参数的属性名成
         <constructor-arg name="name" value="小白"/>
         <constructor-arg name="age" value="20"/>
         <constructor-arg name="grade" ref="grade"/>
         003:使用参数的默认顺序
        <constructor-arg value="heiheihei"/>
        <constructor-arg value="19"/>
        <constructor-arg ref="grade"/>
    </bean>-->


    <!--04.通过c命名空间（构造方法）给属性赋值 前提是必须有构造方法-->
    <bean id="student" class="cn.bdqn.bean.Student"
          c:_0="小妞" c:_1="25" c:grade-ref="grade">

    </bean>
</beans>
```

#### Bean 自动注入

通常情况下都是在XML配置文件中手动声明Bean和组件的。不过Spring也可以自动扫描组件实例化Bean，这样就可以避免在XML文件中繁琐的Bean声明。

**手动声明Bean：**

这里不再啰嗦，就是简单地在XML文件中配置Bean:

```xml
<bean id="car" class="com.Car">
    <property name="wheel" ref="wheel" />
</bean>

<bean id="wheel" class="com.Wheel" />
```

**自动组件扫描**

1. @Component注释表示这是类是一个自动扫描组件。

   ```java
   @Component
   public class Wheel{
       //...
   }
   
   @Component
   public class Car{
       @Autowired
       Wheel wheel;
   
       //...
   } 
   ```

2. 启动自动扫描功能

   方法是在xml配置文件中引入“context:component”。其中base-package 指明存储组件，Spring将扫描该文件夹，找出Bean(注解为@Component)然后注册到 Spring 容器。

   ```xml
   <context:component-scan base-package="<your package>" />
   
   
   
   <context:component-scan base-package="com.hyp.learn.spring"/>
   
   //注解类上
   @ComponentScans(
   		value = {
   				@ComponentScan(value="com.hyp.learn.spring")	
   		}
   		)
   ```

3. 在应用程序中获取Bean

   这里有一个问题，在程序中没有定义Bean的id或者name,那么如何通过IoC容器获得相应的Bean呢？默认情况下Spring将小写组件的第一个字符作为Bean的Id。比如获取Car Bean的方法如下：

   ```java
   Car car = (Car)context.getBean("car");
   ```

   当然也可以自定义组件名称，自定义组件名称的方法很简单：

   ```
   @Component("plane")
   public class Car{
       //...
   }
   ```

   这样Car Bean的名称就被定义为plane了。

**自动组件扫描的注释类型**

一共有四种注释类型:

- @Component – 指示自动扫描组件。
- @Repository – 表示在持久层DAO组件。
- @Service – 表示在业务层服务组件。
- @Controller – 表示在表示层控制器组件。

其实四种注释功能上没有任何取别，查看源码可以知道 @Repository, @Service 或 @Controller 都被注解为 @Component。但为了程序的可读性，还是按照上面的区分来添加注释。

**自动组件扫描--过滤器**

通过过滤器可以 扫描并注册那些满足要求的Bean，即使这些类上面并没有@Component注解。

1. ”包含“过滤器

   同样是上面两个类Car和Wheel,现在去掉上面的@Component注解，也可以通过包含过滤器扫描这两个Bean。

   ```xml
   <context:component-scan base-package="com.CarPackage" >
       <context:include-filter type="regex" 
            expression="com.*Wheel*" />
   
       <context:include-filter type="regex" 
            expression="com.*Car*" />
   </context:component-scan>
   ```

   `<context:include-filter\>`标签表示包含过滤器，其中有两个属性：type和expression。type表示匹配方式，expression指定匹配表达式，type有5个取值。

   | **Filter Type** | **Examples Expression**    | **Description**                                |
   | --------------- | -------------------------- | ---------------------------------------------- |
   | annotation      | org.example.SomeAnnotation | 符合SomeAnnoation的target class                |
   | assignable      | org.example.SomeClass      | 指定class或interface的全名，自身，子类和实现类 |
   | aspectj         | org.example..*Service+     | AspectJ語法                                    |
   | regex           | org\.example\.Default.*    | Regelar Expression                             |
   | custom          | org.example.MyTypeFilter   | Spring3新增Type                                |

2. “不包含”过滤器

   ```xml
   <context:component-scan base-package="com.CarPackage" >
       <context:exclude-filter type="regex" 
            expression="com.*Wheel*" />
   </context:component-scan>
   ```

#### 注入bean

1. 包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]

2. @Bean[导入的第三方包里面的组件]

3. @Import导入组件，id默认是组件的全类名；实现ImportSelector接口，并使用@import导入，返回需要导入的组件的全类名数组；实现ImportBeanDefinitionRegistrar接口，并使用@import导入，手动注册bean到容器中

   ```java
   //car定义
   public class Car {
   
       public void speakOnself(){
           System.out.println("Car");
       }
   }
   
   //cat定义
   
   public class Cat {
   
       public void speakOneself(){
           System.out.println("cat");
       }
   }
   
   //person定义
   import lombok.Data;
   
   @Data
   public class Person {
   
       private String name;
   
       private Integer age;
   
       public void speakOneself(){
           System.out.println("person");
       }
   
   }
   
   //MyImportSelector实现
   import org.springframework.context.annotation.ImportSelector;
   import org.springframework.core.type.AnnotationMetadata;
   
   public class MyImportSelector implements ImportSelector {
       @Override
       public String[] selectImports(AnnotationMetadata annotationMetadata) {
           // 返回值为要实例化bean的全类名，数组
           return new String[]{"com.hyp.learn.spring.model.Person"};
       }
   }
   
   //MyImportBeanDefinition实现
   import org.springframework.beans.factory.support.BeanDefinitionRegistry;
   import org.springframework.beans.factory.support.RootBeanDefinition;
   import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
   import org.springframework.core.type.AnnotationMetadata;
   
   public class MyImportBeanDefinition implements ImportBeanDefinitionRegistrar {
   
       @Override
       public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
   
           RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Cat.class);
           beanDefinitionRegistry.registerBeanDefinition("cat", rootBeanDefinition);
       }
   }
   
   
   //配置类导入
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.Import;
   
   @Import({Car.class, MyImportSelector.class, MyImportBeanDefinition.class})
   @Configuration
   public class ImportAnnoExampleConfig {
   }
   
   //验证：
   
   @RestController
   public class DemoController {
   
       @Autowired
       private Car car;
   
       @Autowired
       private Cat cat;
   
       @Autowired
       private Person person;
   
       @PostConstruct
       public void run(){
           car.speakOnself();
           person.speakOneself();
           cat.speakOneself();
       }
   }
   
   //结果：
   //Car
   //person
   //cat
   ```

4. 实现Condition进行注入,配合@Conditional注解，可用于类或者方法上

   ```java
   //判断是否linux系统
   public class LinuxCondition implements Condition {
   
   	/**
   	 * ConditionContext：判断条件能使用的上下文（环境）
   	 * AnnotatedTypeMetadata：注释信息
   	 */
   	@Override
   	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
   		// TODO是否linux系统
   		//1、能获取到ioc使用的beanfactory
   		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
   		//2、获取类加载器
   		ClassLoader classLoader = context.getClassLoader();
   		//3、获取当前环境信息
   		Environment environment = context.getEnvironment();
   		//4、获取到bean定义的注册类
   		BeanDefinitionRegistry registry = context.getRegistry();
   
   		String property = environment.getProperty("os.name");
   
   		//可以判断容器中的bean注册情况，也可以给容器中注册bean
   		boolean definition = registry.containsBeanDefinition("person");
   		if(property.contains("linux")){
   			return true;
   		}
   
   		return false;
   	}
   
   }
   
   //判断是否windows系统
   public class WindowsCondition implements Condition {
   
   	@Override
   	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
   		Environment environment = context.getEnvironment();
   		String property = environment.getProperty("os.name");
   		if(property.contains("Windows")){
   			return true;
   		}
   		return false;
   	}
   
   }
   
   //配置类使用
       /**
        * @Conditional({Condition}) ： 按照一定的条件进行判断，满足条件给容器中注册bean
        * <p>
        * 如果系统是windows，给容器中注册("bill")
        * 如果是linux系统，给容器中注册("linus")
        */
   
       @Conditional(WindowsCondition.class)
       @Bean("bill")
       public Person person01() {
           return new Person("Bill Gates", 62);
       }
   
       @Conditional(LinuxCondition.class)
       @Bean("linus")
       public Person person02() {
           return new Person("linus", 48);
       }
   ```

5. 使用Spring提供的 FactoryBean（工厂Bean），默认获取到的是工厂bean调用getObject创建的对象，要获取工厂Bean本身，我们需要给id前面加一个&

   ```java
   //创建一个Spring定义的FactoryBean
   public class ColorFactoryBean implements FactoryBean<Color> {
   
   	//返回一个Color对象，这个对象会添加到容器中
   	@Override
   	public Color getObject() throws Exception {
   		// TODO Auto-generated method stub
   		System.out.println("ColorFactoryBean...getObject...");
   		return new Color();
   	}
   
   	@Override
   	public Class<?> getObjectType() {
   		// TODO Auto-generated method stub
   		return Color.class;
   	}
   
   	//是单例？
   	//true：这个bean是单实例，在容器中保存一份
   	//false：多实例，每次获取都会创建一个新的bean；
   	@Override
   	public boolean isSingleton() {
   		// TODO Auto-generated method stub
   		return false;
   	}
   
   }
   
   //配置类中注入
       @Bean
       public ColorFactoryBean colorFactoryBean() {
           return new ColorFactoryBean();
       }
   
   //使用
   //getBean("id")实际上是 factoryBean 的 getObject() 生产的 Bean
   //getBean("&id")获取工厂类本身
   
   ```

#### Bean 自动装配

Spring从两个角度来实现自动化装配:

- 组件扫描(component scanning):Spring会自动发现应用上下文中所创建的bean。
- 自动装配(autowiring):Spring自动满足bean之间的依赖。

组件扫描和自动装配组合在一起就能发挥出强大的威力,它们能够将你的显式配置降低到最少。

##### XML配置里的Bean自动装配

Spring IOC容器可以自动装配Bean，需要做的仅仅是在的autowire属性里指定自动装配的模式

- no：默认方式，手动装配方式，需要通过ref设定bean的依赖关系
- byType（根据类型自动装配）：若IOC容器中有多个与目标Bean类型一致的Bean，在这种情况下，Spring将无法判定哪个Bean最合适该属性，所以不能执行自动装配
- byName（根据名称自动装配）：必须将目标Bean的名称和属性名设置的完全相同
- constructor（通过构造器自动装配）：当Bean中存在多个构造器时，此种自动装配方式将会很复杂，不推荐使用
- autodetect：如果有默认构造器，则以constructor方式进行装配，否则以byType方式进行装配

添加一个Person类

```java
public class Person
{
	private String name;
	
	private Address address;
	
	private Car car;

	public String getName()
	{
		return name;
	}

	public void setName(String name)
	{
		this.name = name;
	}

	public Address getAddress()
	{
		return address;
	}

	public void setAddress(Address address)
	{
		this.address = address;
	}

	public Car getCar()
	{
		return car;
	}

	public void setCar(Car car)
	{
		this.car = car;
	}

	@Override
	public String toString()
	{
		return "Person [name=" + name + ", address=" + address + ", car=" + car + "]";
	}
}
```

再添加一个Address类和Car类

```java
public class Address
{
	private String city;
	private String street;
	public String getCity()
	{
		return city;
	}
	public void setCity(String city)
	{
		this.city = city;
	}
	public String getStreet()
	{
		return street;
	}
	public void setStreet(String street)
	{
		this.street = street;
	}
	@Override
	public String toString()
	{
		return "Address [city=" + city + ", street=" + street + "]";
	}
	
	
}

public class Car
{
	private String brand;
	
	private double price;

	public String getBrand()
	{
		return brand;
	}

	public void setBrand(String brand)
	{
		this.brand = brand;
	}

	public double getPrice()
	{
		return price;
	}

	public void setPrice(double price)
	{
		this.price = price;
	}

	@Override
	public String toString()
	{
		return "Car [brand=" + brand + ", price=" + price + "]";
	}
	
	
}


```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="address" class="com.hyp.learn.demo07.Address"
          p:city="BeiJing" p:street="HuiLongGuan">
    </bean>

    <bean id="car" class="com.hyp.learn.demo07.Car"
          p:brand="Audi" p:price="300000">
    </bean>

    <bean id="person" class="com.hyp.learn.demo07.Person"
          p:name="Tom" autowire="byName">
    </bean>

</beans>
```

测试类：

```java
    @Test
    public void demo07()
    {
        ApplicationContext context=new ClassPathXmlApplicationContext(
                "applicationContext07.xml"
        );
        Person person = (Person) context.getBean("person");
        System.out.println(person);
    }
```

可以使用autowire属性指定自动装配的方式，
byName：根据bean的名字和当前bean 的setter风格的属性名进行自动装配，若有匹配的，则进行自动装配，若没有，则不匹配

byType：根据bean 的类型和当前bean的属性的类型进行自动装配，若IOC容器中有一个以上的类型匹配的bean，则抛异常

如果在容器中存在多个类型相同的bean怎么办呢？spring提供了另外两种选择，可以设置一个首选bean，或者排除一些bean。

元素的primary属性代表是否是首选bean，如果标注为true，那么该bean将成为首选bean。
但是spring默认每个bean的primary属性都是true，所以如果需要设置首选bean需要将那些非首选bean的primary属性标注为false。

**缺点**

- 在Bean配置文件里设置aotowire属性进行自动装配会装配Bean的所有属性，然而，若只希望装配个别属性时，aotuwire属性不够灵活了
- autowire属性要么根据类型自动装配，要么根据名称自动装配，不能两者兼而有之
- 一般情况下，在实际的项目中很少使用自动装配功能，因为和自动装配功能所带来的好处比起来，明确清晰的配置文档更有说服力一些

**默认自动装配**
在元素中添加一个default-autowire属性，该配置文件当中的所有bean将会进行自动装配，如果有特定的bean需要使用其他的方式，在该bean上直接设置autowire属性就可以了，会覆盖掉默认自动装配的配置，代码如下。

```xml
<beans ... default-autowire="byType">
</beans>
```

**自动装配侯选者**
 XML配置中默认所有的bean都是自动装配的侯选者。如果设置元素的autowire-candidate属性为false，该bean将不用于自动装配。autowire-candidate默认值为true。

元素的default-autowire-candidates属性的值允许使用通配符，例如我们制定`default-autowire-candidates=“*abc”`，则所有以“abc”结尾的Bean都将被包含到自动装配的待选类中。该属性可以指定多个匹配字符串，匹配任一字符串的Bean都将作为侯选者。

##### 使用注解自动装配

如果不想在xml文件中使用autowire属性来启用自动装配，还可以直接在类定义中使用@Autowired或@Resource来装配bean。

在使用注解装配之前，首先要开启注解装配的方式，在配置文件中加上下面这句话

```xml
    <context:annotation-config/>
```

当然还要在xml文件添加context命名空间并指定schema

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context.xsd">
```

spring支持多种注解装配的方式，这里主要介绍spring自带的Autowired注解装配

注：@Resource也可用于自动装配。但@Resource并不是Spring的注解，他的包是javax.annotation.Resource。Spring支持该注解的注入，@Resource通过设置可以按byName和byType 方式注入，如果都没有写，默认按byName方式。

**使用@Autowired注解**
@Autowired注解可以用在任何方法上，不一定非得是setter方法，只要方法有需要自动装配的参数都可以，但是一般都是使用在setter方法和构造器上的。

支持以下三种注入方式

- Match by Type
- Match by Qualifier
- Match by Name
  注入的方式也是支持属性注入和setter方法注入两种形式。

注意：@Autowired注解默认使用的是byType的方式向Bean里面注入相应的Bean。

1. 用于setter方法：

   ```java
   @Autowired
   public void setNotifyservice(NotifyService notifyservice) {
       this.notifyservice = notifyservice;
   }
   ```

   以上代码是把@Autowired注解在setter方法上，在spring创建该类的bean的时候，就会自动寻找匹配的参数注入到该bean当中。

2. 用于构造函数

   ```java
   public class Order {
   @Autowired
   public Order(NotifyService notifyservice) {
       this.notifyservice = notifyservice;
   }
   //......省略部分代码
   ```

3. 直接注解在属性(最常用)

   @Autowired还有一种用法就是直接注解在属性上，从而去掉setter方法

   ```java
   @Autowired
   private NotifyService notifyservice;
   ```

   使用@Autowired自动装配时，容器中只能有一个适合的Bean待选，否则的话，spring会抛出异常。(因为@Autowired默认是使用byType的方式装配)
   如果在应用上下文当中找不到相应的bean去自动装配，那么spring也会抛出异常（NoSuchBeanDefinitionException）。
   如果想避免这种情况发生，而且需要装配的属性也不是必须要装配的话，可以使用如下代码来使用注解：

   ```java
   @Autowired(required=false)
   private Instrument instrument;
   ```

   在这里添加@Autowired的required属性，将这个属性设置为false，意思就是在创建bean的时候该属性不是必须的

**@Resource**

它具有三种注入的模式

- Match by Name(按变量名注入)
- Match by Type(按类型注入)
- Match by Qualifier(使用Qualifier显式注入)
  这些注入方式都支持通过setter方法注入或者是通过属性的方式注入。

装配顺序：

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

**@Inject**

@Inject同样不是Spring自带的注解，它是属于JSP-330的，同样有三种注入方式

- Match by Name
- Match by Type
- Match by Qualifier
  同样，也可以使用属性注入或者是setter方法注入

想要使用@Inject注解，需要在maven中引入相关依赖

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>

```

@Inject这个注解自身是不带有name属性的，需要另外使用一个@Named注解来告诉Spring框架使用什么名字对属性进行注入。

**@Qualifier注解**

刚才提到，如果在容器中出现了两个适合的bean，就会出错。怎么解决呢？这个时候可以使用@Qualifier注解指定一个Bean来装配，这样就不会报异常了。@Qualifier注解采用的是byName的方式。

```java
@Autowired
@Qualifier("CellPhoneNotifyserviceImpl")
private NotifyService notifyservice;
```

括号中的字符串标注的是需要自动装配进来的Bean的id 在注解注入中使用表达式

**@Value注解**

在使用注解自动装配的过程当中，如果想要自动装配基本类型的或者是字面值常量的参数的话，可以是用@Value注解

```java
@Value("Messi")
private String username;
```

上例为一个String类型的属性装配了一个String类型的值，同样可以装配int，boolean等基本类型的属性。

在@Value注解中，还可以使用表达式来动态的计算并装配属性的值。(经常用来从properties文件中获取值)

比如使用spel表达式从某个对象属性中取得一个值()

```java
@Value("#{systemConfig.UploadPath}")
private String savePath;
```

##### @Resource和@Autowired对比

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

**共同点**

两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。

**不同点**

- @Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照byType注入。

- @Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。

- @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

  注：最好是将@Resource放在setter方法上，因为这样更符合面向对象的思想，通过set、get去操作属性，而不是直接去操作属性。

个人在使用上，更偏重使用@Inject，这是jsr330规范的实现，而@Autowired是spring的实现，如果不用spring一般用不上这个，而@Resource则是jsr250的实现，这是多年前的规范。

参考：

1. [Spring中@Autowire、@Resource和@Inject的使用和区别](https://blog.csdn.net/turbo_zone/article/details/8 204815)
2. [@Inject 注解初识](https://blog.csdn.net/zhaoyishi/article/details/83964634)
3. XXXAware讲解：https://www.baeldung.com/spring-bean-name-factory-aware
4. 后置处理器XXXPostProcessor讲解：http://wiki.jikexueyuan.com/project/spring/bean-post-processors.html
5. https://www.jianshu.com/p/fb39f568cd5e
