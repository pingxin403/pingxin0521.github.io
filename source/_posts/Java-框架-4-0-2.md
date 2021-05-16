---
title: Spring 配置
date: 2019-05-22 15:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring
---

#### XML配置文件

Spring的配置文件是用于指导Spring工厂进行Bean的生产、依赖关系注入及Bean实例分发的“图纸”，它是一个或多个标准的XML文档，J2EE程序员必须学会并灵活应用这份“图纸”，准确的表达自己的“生产意图”。

<!--more-->

1. 声明

   随便打开一份Spring工程的配置文件，第一行基本上都如下所示：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   ```

   从字面意义基本就能知道大概是关于版本和编码信息的，而事实也的确如此。1998年，W3C就发布了XML1.0规范，也许将来会发布新版本，但是目前仍然是1.0版本。encoding是编码声明，代表xml文件采用utf-8的编码格式。

   需要注意的是，XML 声明通常在 XML 文档的第一行出现。 XML 声明不是必选项，但是如果使用 XML 声明，必须在文档的第一行，前面不得包含任何其他内容或空白。

2. 正文

   声明信息之后，就是配置文件的正文了，内容基本上如下所示：

   ```xml
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context-2.5.xsd">
   
                               //具体配置信息
   
   </beans>
   ```

   `beans标签是整个配置文件的根节点，包含一个或者多个bean元素`，而我们的学习也从这里展开。

3. 命名空间

   - xmlns
     XML NameSpace的缩写，因为XML文件的标签名称都是自定义的，自己写的和其他人定义的标签很有可能会重复命名，而功能却不一样，Spring默认的命名空间就是`http://www.springframework.org/schema/beans`。Spring容器在解析xml文件时，会获取标签的命名空间来跟上述url比较，判断是否为默认命名空间。
   - xmlns:xsi
     全名xml schema instance，是指具体用到的schema资源文件里定义的元素所准守的规范。即`http://www.w3.org/2001/XMLSchema-instance`这个文件里定义的元素遵守什么标准 。
   - xsi:schemaLocation
     本文档里的xml元素所遵守的规范，这些规范都是由官方制定的，可以进你写的网址里面看版本的变动。xsd的网址还可以帮助你判断使用的代码是否合法。

   那么问题来了，感觉上述信息都要在线验证，如果我的应用离线运行，那怎么办呢？Spring也考虑到了这个问题，所以都会在jar包中附带上相应的的xsd文件。通常，META-INF目录下有如下spring.handlers和spring.schemas两个文件。

   两个文件的内容都是常量的定义，值都是jar包的路径，打开对应的文件可以知道，xsd文件是命名空间schema的定义文件，而*Namespacehandler文件则负责将定义的xml文件信息注册Spring容器中，具体的逻辑其实可以参考spring容器的启动过程，此处不再赘述。

4. import标签

   在实际开发过程中，一些大型项目会将配置信息按照不同的模块划分成多个配置文件，spring import标签就可以达到此目的，我们会经常看到如下的配置信息：

   ```xml
   <import resource="file:..."/>
   <import resource="classpath:..."/>
   ```

   - file：表示使用文件系统的方式寻找后面的文件（文件的完整路径）
   - classpath：相当于/WIN-INF/classes/，如果使用了classpath，那就表示只会到你的class路径中去查找文件
   - `classpath*`：表示不仅会在class路径中去查找文件，还会在jar中去查找文件

   需要注意的是，Spring采取递归的方式解析import标签，很可能会出现变量无法解析的情况，如果存在变量引用的情况，需要注意。

5. context标签

   spring从2.5版本开始支持注解注入，注解注入可以省去很多的xml配置工作。由于注解是写入java代码中的，所以注解注入会失去一定的灵活性，我们要根据需要来选择是否启用注解注入。

   ```xml
   <context:annotation-config />
   <context:component-scan base-package="xxxxxxxxx"/>
   ```

   前者的作用是隐式地向Spring容器注册AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor 这 4 个BeanPostProcessor。

   后者启用了对类包进行扫描以实施注释驱动 Bean 定义的功能，同时还启用了注释驱动自动注入的功能。当使用 < context:component-scan/ > 后，就可以将 < context:annotation-config/ > 移除了。base-package 属性指定了需要扫描的类包，类包及其递归子包中所有的类都会被处理。

6. aop标签

   ```xml
       <aop:aspectj-autoproxy/>
       <aop:config>
           <aop:aspect id = "aspectXML" ref="actionAspectXML">
               <aop:pointcut id="anyMethod" expression="execution(* com.maowei.learning.aop.ActionImpl.*(..))"/>
               <aop:before method="beforeMethod" pointcut-ref="anyMethod"/>
               <aop:after method="afterMethod" pointcut-ref="anyMethod"/>
               <aop:after-returning method="afterReturningMethod" pointcut-ref="anyMethod"/>
               <aop:after-throwing method="afterThrowMethod" pointcut-ref="anyMethod"/>
               <aop:around method="aroundMethod" pointcut-ref="anyMethod"/>
           </aop:aspect>
       </aop:config>
   ```

   aop:aspectj-autoproxy声明自动为spring容器中那些配置@aspectJ切面的bean创建代理，织入切面。它有一个proxy-target-class属性，默认为false，表示使用jdk动态代理织入增强，当配为true时，表示使用CGLib动态代理技术织入增强。不过即使proxy-target-class设置为false，如果目标类没有声明接口，则spring将自动使用CGLib动态代理。

   aop:aconfig便是具体的AOP信息了，具体内容可以查看相关内容，不再赘述。

7. bean标签

   `bean标签`在配置文件中最常见，具体的格式有如下几种：

   ```xml
   <!-- id是bean的标识符,必须唯一,如果没有配置id,name默认为标识符
     　　  如果配置了id,有配置了name,那么name为别名
        name可以设置多个别名,分隔符可以是空格 逗号 分号
        class是bean的全限定名,即包名加类名
           如果不配置id和name,那么可以根据applicationContext.getbean(Class)获取对象,
   　　　scope:bean的作用域,
   　　　　　　取值:singleton:单例的,整个容器只产生一个对象,默认是单例
   　　　　　　　　 prototype:原型,每次获取bean都创建一个新对象
   　　　　　　　　 request:每次请求时创建一个新的对象
   　　　　　　　　 session:在一个会话范围内只产生一个对象
   　　　　　　　　 application:在应用范围内是一个对象
       autowire:自动装配 用于简化spring的配置
   　　　　　　取值:byname:根据名称(根据set方法中set后面的内容)去查找相应的bean,发现了则装载上
   　　　　　　　　 bytype:根据类型自动装配,不用去管id,但同一种类型的bean只能有一个,f否则报错
   　　　　　　　　 constructor,当通过构造器注入实例化bean时,装配构造方法
   　　　　　　　　　
            -->
   <bean id = "bean实例名" class="bean类全名" />
   
   <bean id = "bean实例名" class="bean类全名" scope="prototype" />
   
   <bean id = "bean实例名" class="bean类全名" init-method="初始化时调用的方法" destory-method="对象销毁时调用方法"/>
   
   <bean id = "bean实例名" class="bean类全名" >
       <property name="bean类的属性名称" ref="要引用的bean名称"/>
       <property name="bean类的属性名称" value="直接指定属性值"/>
       ……
   </bean>
   
   <bean id = "bean实例名" class="bean类全名" >
       <constructor-arg index="构造方法中参数的序号，从0开始计算" type="构造方法参数类型" value="参数值" />
       <constructor-arg index="构造方法中参数的序号，从0开始计算" type="构造方法参数类型" ref="引用的bean名称" />
   </bean>
   ```

   - bean的关联(depend-on属性):

     要求在配置类A的bean时,必须有一个关联的类B的bean,换句话说类A的bean依赖于类B的bean,这时可以在类A的bean标签中设置depend-on="bBean".

     这样设置的话,类B的bean会先初始化,　

   - 抽象bean(abstract属性):

     当在bean标签中设置属性abstract="true",即指定该bean为抽象bean,不会被实例化,一般仅供被其他的bean继承.

     抽象bean,可以不指定class属性,而是在继承它的子Bean中设置class属性.

   - bean的继承(parent属性):

     如果car1和car2对象同属于类Car,在配置文件中,他们之间就可以使用parent属性来简化代码

     ```xml
     <bean id="car1" class="com.wang.entity.Car">
             <property name="brand" value="Audi"/>
             <property name="price" value="1000000"/>
       </bean>
       <bean id="car2" parent="car1">
             <property name="price" value="3000"></property>
       </bean>
     ```

     这样的配置,car2就继承了car1中的class属性和brand的属性.简化了代码,car1称为父bean,car2称为子bean.car2可以覆盖从car1继承过来的属性,比如price.

8. tx标签

   `tx标签`一般用于事务管理，常见的用法如下：

   ```xml
       <tx:advice id="txAdvice" transaction-manager="transactionManager">
           <tx:attributes>
               <tx:method name="*" 
                          isolation="READ_COMMITTED"
                          propagation="REQUIRED" 
                          timeout="100"/>
               <tx:method name="get*" 
                          read-only="100"/>
           </tx:attributes>
       </tx:advice>
   ```

9. 使用外部属性文件

   在配置文件中配置Bean时,有时候需要在bean的配置里混入一些系统部署的细节信息(例如文件路径,数据源配置信息),而这些部署细节实际上需要和bean配置相分离.

   在配置c3p0数据源连接池时,我们可以这样写:

   ```xml
   <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <property name="user" value="root"></property>
           <property name="password" value="123"></property>
           <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
           <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test"></property>
        <!-- <property name="initPoolSize" value="3"></property>
           <property name="maxPoolSize" value="10"></property>  -->
       </bean>
   ```

   为了使数据库配置信息和spring的配置文件分离,便于维护,更好的方法是这样:

   在resources目录下,新建一个db.properties文件,内容如下:

   ```properties
   jdbc.user=root
   jdbc.password=123
   jdbc.driverClass=com.mysql.jdbc.Driver
   jdbc.jdbcUrl=jdbc:mysql://localhost:3306/hibernate
   
   jdbc.initialPoolSize=5
   jdbc.maxPoolSize=10
   ```

   在beans.xml中:

   ```xml
   <!-- 导入资源文件 -->
       <context:property-placeholder location="classpath:db.properties"/>
       <!-- 配置c3p0连接池 -->
       <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <property name="user" value="${jdbc.user}"></property>
           <property name="password" value="${jdbc.password}"></property>
           <property name="driverClass" value="${jdbc.driverClass}"></property>
           <property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
        <!-- <property name="initialPoolSize" value="${jdbc.initPoolSize}"></property>
           <property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>  -->
       </bean>
   ```

     注意使用context的标签,需要在头文件中添加支持context的信息,这里不再给出

10. 引用其他Bean

    **引用外部Bean**

    ```xml
    <bean id="car" class="com.hzg.spring.beans.Car">
        <constructor-arg value="Audi" type="java.lang.String"></constructor-arg>
        <constructor-arg value="120" type="int"></constructor-arg>
        <constructor-arg value="black" type="java.lang.String"></constructor-arg>
    </bean>
    
    <bean id="person" class="com.hzg.spring.beans.Person">
        <property name="name" value="hzg"></property>
        <property name="age" value="32"></property>
        <!-- 使用 property的ref属性建立Bean之间的引用关系-->
        <property name="car" ref="car"></property>
    </bean>
    ```

    使用 property的ref属性建立Bean之间的引用关系

    **引用内部Bean**

    ```xml
    <bean id="person" class="com.hzg.spring.beans.Person">
        <property name="name" value="hzg"></property>
        <property name="age" value="32"></property>
        <!-- 内部Bean，不能被外部引用，只能在内部使用-->
        <property name="car">
            <bean class="com.hzg.spring.beans.Car">
                <constructor-arg value="Audi" type="java.lang.String"></constructor-arg>
                <constructor-arg value="120" type="int"></constructor-arg>
                <constructor-arg value="black" type="java.lang.String"></constructor-arg>
            </bean>
        </property>
    </bean>
    ```

    因为id是为了让外部引用，而内部Bean内部只用一次，不能被外部引用，随意不写id。

11. 级联属性

    ```xml
    <bean id="person" class="com.hzg.spring.beans.Person">
        <property name="name" value="hzg"></property>
        <property name="age" value="32"></property>
        <property name="car" ref="car"></property>
        <!-- 级联属性赋值 -->
        <property name="car.price" value="30000"></property>
    </bean>
    ```

    注意：属性想用car.price的级联，首先car必须要先初始化，否则会有异常

12. 集合属性

    当我们的Bean中有集合元素的时候，Spring中可以通过一组内置的xml标签（`<list>、<set>、<map>`）来配置集合属性。

    现在如果你想传递多个值，如 Java Collection 类型 List、Set、Map 和 Properties，应该怎么做呢。为了处理这种情况，Spring 提供了四种类型的集合的配置元素，如下所示： 

    | 元素     | 描述                                                        |
    | -------- | ----------------------------------------------------------- |
    | <list\>  | 它有助于连线，如注入一列值，允许重复。                      |
    | <set\>   | 它有助于连线一组值，但不能重复。                            |
    | <map\>   | 它可以用来注入名称-值对的集合，其中名称和值可以是任何类型。 |
    | <props\> | 它可以用来注入名称-值对的集合，其中名称和值都是字符串类型。 |

    **List集合**

    刚才的Person类有一个car属性，现在我们修改一下，Person类中有一个List<car>,即一个人拥有多辆车

    ```java
    public class Person {
        private String name;
        private int age;
        private List<Car> cars;
        
        //为上面的3个属性设置getter和setter方法
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public int getAge() {
            return age;
        }
        public void setAge(int age) {
            this.age = age;
        }
        public List<Car> getCars() {
            return cars;
        }
        public void setCars(List<Car> cars) {
            this.cars = cars;
        }
    }
    ```

    配置文件修改如下:

    ```xml
    <bean id="car1" class="com.hzg.spring.beans.Car">
        <constructor-arg value="Audi" type="java.lang.String"></constructor-arg>
        <constructor-arg value="120" type="int"></constructor-arg>
        <constructor-arg value="black" type="java.lang.String"></constructor-arg>
    </bean>
    <bean id="car2" class="com.hzg.spring.beans.Car">
        <constructor-arg value="Ford" type="java.lang.String"></constructor-arg>
        <constructor-arg value="100" type="int"></constructor-arg>
        <constructor-arg value="black" type="java.lang.String"></constructor-arg>
    </bean>
    <bean id="car3" class="com.hzg.spring.beans.Car">
        <constructor-arg value="Toyota" type="java.lang.String"></constructor-arg>
        <constructor-arg value="95" type="int"></constructor-arg>
        <constructor-arg value="black" type="java.lang.String"></constructor-arg>
    </bean>
    <bean id="person" class="com.hzg.spring.beans.Person">
        <property name="name" value="hzg"></property>
        <property name="age" value="32"></property>
        <!-- 集合属性（List），使用ref来配置子节点信息 -->
        <property name="cars">
            <list>
                <ref bean="car1"></ref>
                <ref bean="car2"></ref>
                <ref bean="car3"></ref>
            </list>
        </property>
    </bean>
    ```

    **Map集合**

    如果是**<map\>集合**的话，只需要把list换成map即可。

    ```xml
    <!-- 集合属性（Map），使用entry来配置map的子节点信息 -->
    <property name="cars">
    　　<map>
    　　　<entry key="A" value-ref="car1" ></entry>
    　　　<entry key="B" value-ref="car2" ></entry>
       </map>
    </property>
    ```

     **现在这么多集合Bean能不能抽出来，以供多个Bean进行引用，那么就需要使用util**

    ```xml
    <bean id="person" class="com.hzg.spring.beans.Person">
        <property name="name" value="hzg"></property>
        <property name="age" value="32"></property>
        <!-- 引用util -->
        <property name="cars" ref="cars"></property>
    </bean>
    <util:list id="cars">
        <ref bean="car1"></ref>
        <ref bean="car2"></ref>
    </util:list>
    ```

    **使用p命名空间** 

    Sping从2.5版本开始引入一个新的P的命名空间，通过使用P命名空间后，使得Xml的配置方式进一步简化

    ```xml
    <!-- 通过P命名空间为Bean属性赋值，相对于传统的配置方式更加简洁 -->
    <bean id="person" class="com.hzg.spring.beans.Person" p:name="hzg" p:age="32" p:cars-ref="cars">
    </bean>
    ```

13. 使用autowire属性进行Bean的自动装配

    当我们要往一个bean的某个属性里注入另外一个bean，我们会使用`<property ref="">`标签的形式。但是对于大型项目这种方式其实是不可取的。

    ```xml
    <!-- 
         大型项目中我们一般使用autowire属性自动装备Bean，autowire属性有两个值：byName和byType
            byName：根据Bean的名字和当前的Bean的setter属性名称进行装配
            byType：根据Bean的类型进行装配，若有1个以上的类型，则抛异常
    -->
    <bean id="person" class="com.hzg.spring.beans.Person" autowire="byType">
         <property name="name" value="hzg"></property>
         <property name="age" value="32"></property>
    </bean>
    ```

    自动装配的特点：

    - 只有引用类型才可以自动装配。

    - autowire属性是在bean的级别上，一旦指定，当前bean的所有引用类型都必须使用自动装配。

    - byType和byName的只能选择其一。

14. 配置上的继承（parent属性）

    当我们有两个或者以上的同一个类型的Bean做实例化的时候，我们可以将一些公共的提成通用Bean。这样其他的Bean继承它即可以了。

    ```xml
    <bean id="address1" class="com.hzg.Address" p:provice="Hebei" p:city="LangFang" p:street="YingBing">
    
    </bean>
    <bean id="address2" class="com.hzg.Address" p:provice="Hebei" p:city="LangFang" p:street="YanLing">
    
    </bean>
    <bean id="address3" class="com.hzg.Address" p:provice="Hebei" p:city="LangFang" p:street="YanShun">
    
    </bean>
    ```

    以上Bean中用的都是address这个类，都是省市街道的赋值，只是在街道上每个不一样，所以我们完全可以把通用的提出来。

    ```xml
    <bean id="address" class="com.hzg.Address" p:provice="Hebei" p:city="LangFang" p:street="YingBing">
    
    </bean>
    <bean id="address1" parent="address" p:street="YanLing">
    
    </bean>
    <bean id="address2" parent="address" p:street="YanShun">
    
    </bean>
    ```

15. Bean的作用域（scope属性）

    ```xml
    <bean id="address" class="com.hzg.Address" p:provice="Hebei" p:city="LangFang" p:street="YanShun">
    </bean>
    ```

    ```java
    ApplicationContext ctx = new ClassPathXmlApplicationContext("configspring.xml");
    Address address1 = (Address) ctx.getBean("address");
    Address address2 = (Address) ctx.getBean("address");
    System.out.println(address1==address2);
    ```

    输出结果为：true

    说明默认为单例模式的，它只要是看scope属性的。

    使用Bean的scope属性来配置Bean的作用域的，scope有两个重要的属性值。

    - singleton：默认值，容器初始化时创建Bean实例，在整个容器的生命周期内只创建一个Bean，单例的。

    - prototype：原型的，容器初始化时不创建Bean的实例，而在每次请求时都创建一个新的Bean的实例，并返回。

#### Spring表达式SPEL表达式

Spring 表达式语言(简称SpEL)：是一个支持运行时查询和操作对象图的强大的表达式语言。

**语法类似于 EL：**SpEL 使用 #{...} 作为定界符 , 所有在大括号中的字符都将被认为是 SpEL , SpEL 为 bean 的属性进行动态赋值提供了便利。

**通过 SpEL 可以实现：**

- 通过 bean 的 id 对 bean 进行引用。
- 调用方式以及引用对象中的属性。
- 计算表达式的值
- 正则表达式的匹配。

首先，SpEL表达式是放到“#{ … }”之中，而属性占位符是放到“${ … }”之中的。下面是几种比较基础的使用方式。

**使用方式**
如果通过组件扫描（在类上添加@Component）创建bean的话， 在注入属性和构造器参数时， 可以使用@Value注解。这时，使用的不再是占位符，而是SpEL表达式。例如：@Value(“#{systemProperties[‘user.language’]}”)是用来获取用户语言系统的属性。

在XML配置中， 可以将SpEL表达式传入或的value属性中， 或者将其作为p-命名空间、c-命名空间条目的值。

示例：

Car.java文件

```java
public class Car {

    private String brand;

    private double price;

    private String perimeter;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public String getPerimeter() {
        return perimeter;
    }

    public void setPerimeter(String perimeter) {
        this.perimeter = perimeter;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                ", perimeter='" + perimeter + '\'' +
                '}';
    }
}
```

Address.java文件：

```java

public class Address {

    private String province;

    private String city;

    private String area;

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getArea() {
        return area;
    }

    public void setArea(String area) {
        this.area = area;
    }

    @Override
    public String toString() {
        return "Address{" +
                "province='" + province + '\'' +
                ", city='" + city + '\'' +
                ", area='" + area + '\'' +
                '}';
    }
}
```

Person.java文件：

```java
public class Person {

    private String name;

    private int age;

    private boolean marriage;

    private Car car;

    private String socialStatus;

    private String address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public boolean isMarriage() {
        return marriage;
    }

    public void setMarriage(boolean marriage) {
        this.marriage = marriage;
    }

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    public String getSocialStatus() {
        return socialStatus;
    }

    public void setSocialStatus(String socialStatus) {
        this.socialStatus = socialStatus;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", marriage=" + marriage +
                ", car=" + car +
                ", socialStatus='" + socialStatus + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="car" class="com.itdjx.spring.spel.Car">
        <property name="brand" value="#{'玛莎拉蒂'}"></property>
        <property name="price" value="#{32000.78}"></property>
        <property name="perimeter" value="#{T(java.lang.Math).PI * 75.8f}"></property>
    </bean>

    <bean id="person" class="com.itdjx.spring.spel.Person">
        <property name='name' value='#{"平心"}'></property>
        <property name="age" value="#{25}"></property>
        <property name="marriage" value="#{car.price > 400000 and age > 30}"></property>
        <property name="car" value="#{car}"></property>
        <property name="socialStatus" value="#{car.price > 30000 ? '金领' : '白领'}"></property>
        <property name="address" value="#{address.province + '省' + address.city + '市' + address.area + '区'}"/>
    </bean>

    <bean id="address" class="com.itdjx.spring.spel.Address">
        <property name="province" value="#{'河南'}"/>
        <property name="city" value="#{'郑州'}"/>
        <property name="area" value="#{'紫荆山'}"/>
    </bean>

</beans>
```

测试文件：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext app = new ClassPathXmlApplicationContext("spel-beans.xml");
        Person person = (Person) app.getBean("person");
        System.out.println(person);

    }
}
```

控制台输出：

```
Person{name='平心', age=25, marriage=false, car=Car{brand='玛莎拉蒂', price=32000.78, perimeter='238.13273272948624'}, socialStatus='金领', address='河南省郑州市紫荆山区'}
```

**SpEL 字面量：**

- 整数：#{8}
- 小数：#{8.8}
- 科学计数法：#{1e4}
- String：可以使用单引号或者双引号作为字符串的定界符号。
- Boolean：#{true}

**SpEL引用bean , 属性和方法：**

- 引用其他对象:#{car}
- 引用其他对象的属性：#{car.brand}
- 调用其它方法 , 还可以链式操作：#{car.toString()}
- 调用静态方法静态属性：#{T(java.lang.Math).PI}

 **SpEL支持的运算符号：**

- 算术运算符：+，-，*，/，%，^(加号还可以用作字符串连接)
- 比较运算符：< , > , == , >= , <= , lt , gt , eg , le , ge
- 逻辑运算符：and , or , not , |
- if-else 运算符(类似三目运算符)：？:(temary), ?:(Elvis)
- 正则表达式：#{admin.email matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,4}'}

#### 常用注解

##### java配置类相关注解

@Configuration 声明当前类为配置类，相当于xml形式的Spring配置（类上）

@Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）

@Configuration 声明当前类为配置类，其中内部组合了@Component注解，表明这个类是一个bean（类上）

@ComponentScan 用于对Component进行扫描，相当于xml中的（类上）

@WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解

@Configuration把一个类作为一个IoC容器，它的某个方法头上如果注册了@Bean，就会作为这个Spring容器中的Bean。

```java
//@ComponentScan 用于对Component进行扫描，相当于xml中的（类上）
@ComponentScan
//@Configuration 声明当前类为配置类，相当于xml形式的Spring配置（类上）   
@Configuration  
//@WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解
    public class MainConfig {  
      
    //在properties文件里配置  
        @Value(“${wx_appid}”)  
    public String appid;  
        
         protected MainConfig(){}  
      //@Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）
        @Bean  
        public WxMpService wxMpService() {  
            WxMpService wxMpService = new WxMpServiceImpl();  
            wxMpService.setWxMpConfigStorage(wxMpConfigStorage());  
            return wxMpService;  
    }  
    }  
```



##### 声明bean的注解

@Controller, @Service, @Repository,@Component:目前4种注解意思是一样，并没有什么区别，区别只是名字不同。

@Component泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

@Service用于标注业务层组件、 

@Controller用于标注控制层组件（如struts中的action）

@Repository用于标注数据访问组件，即DAO组件。

##### @Bean的属性支持

1. @PostConstruct用于指定初始化方法（用在方法上），@PreDestory用于指定销毁方法（用在方法上），实现初始化和销毁bean之前进行的操作，只能有一个方法可以用此注释进行注释，方法不能有参数，返回值必需是void,方法需要是非静态的。

   ```java
       public class TestService {   
         
           @PostConstruct    
           public void  init(){    
               System.out.println(“初始化”);    
           }    
               
           @PreDestroy    
           public void  dostory(){    
               System.out.println(“销毁”);    
           }    
       }  
   ```

   @PostConstruct：在构造方法和init方法（如果有的话）之间得到调用，且只会执行一次。

   @PreDestory：注解的方法在destory()方法调用后得到执行。

   ![1.png](https://i.loli.net/2019/06/09/5cfc694b3825b58523.png)

   引深一点，Spring 容器中的 Bean 是有生命周期的，Spring 允许在 Bean 在初始化完成后以及 Bean 销毁前执行特定的操作，常用的设定方式有以下三种：

   1.通过实现 InitializingBean/DisposableBean 接口来定制初始化之后/销毁之前的操作方法；

   2.通过 <bean\> 元素的 init-method/destroy-method属性指定初始化之后 /销毁之前调用的操作方法；

   3.在指定方法上加上@PostConstruct 或@PreDestroy注解来制定该方法是在初始化之后还是销毁之前调用

   但他们之前并不等价。即使3个方法都用上了，也有先后顺序.

   ```
   @Constructor > @PostConstruct >InitializingBean > init-method
   ```

2. @Primary：自动装配时当出现多个Bean候选者时，被注解为@Primary的Bean将作为首选者，否则将抛出异常

   ```java
       @Component    
       public class Apple implements Fruit{    
           
           @Override    
           public String hello() {    
               return ”我是苹果”;    
           }    
       }  
         
       @Component    
       @Primary  
       public class Pear implements Fruit{    
           
           @Override    
           public String hello(String lyrics) {    
               return ”梨子”;    
           }    
       }  
          
       public class FruitService {   
           
         //Fruit有2个实例子类，因为梨子用@Primary，那么会使用Pear注入  
           @Autowired    
           private Fruit fruit;    
           
           public String hello(){    
               return fruit.hello();    
           }    
       }  
   ```

3. @Lazy(true) 表示延迟初始化,用于注解类

4. @Singleton。只要在类上加上这个注解，就可以实现一个单例类，不需要自己手动编写单例实现类。

5. @Scope注解 作用域

   @Scope用于指定scope作用域的（用在类上）

   Singleton （单例,一个Spring容器中只有一个bean实例，默认模式）, Protetype （每次调用新建一个bean）, Request （web项目中，给每个http request新建一个bean）, Session （web项目中，给每个http session新建一个bean）, GlobalSession（给每一个 global http session新建一个Bean实例）

6. @DependsOn：定义Bean初始化及销毁时的顺序

##### 注入bean的注解

使用注解之前要开启自动扫描功能，其中base-package为需要扫描的包(含子包)。

```xml
<context:component-scan base-package="com.hyp"/> 
```

**@Autowired** 默认按类型装配，如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。如下：

`@Autowired @Qualifier("personDaoBean")` 存在多个实例配合使用

Autowired默认先按byType，如果发现找到多个bean，则，又按照byName方式比对，如果还有多个，则报出异常。

```
//通过此注解完成从spring配置文件中 查找满足Fruit的bean,然后按//@Qualifier指定pean
@Autowired
@Qualifier(“pean”)
public Fruit fruit;
```

如果要允许null 值，可以设置它的required属性为false，如：

```
@Autowired(required=false) 
public Fruit fruit;
```

**@Resource**

@Resource默认按名称装配，当找不到与名称匹配的bean才会按类型装配。

默认按 byName自动注入,如果找不到再按byType找bean,如果还是找不到则抛异常，无论按byName还是byType如果找到多个，则抛异常。

可以手动指定bean,它有2个属性分别是name和type，使用name属性，则使用byName的自动注入，而使用type属性时则使用byType自动注入。

```
@Resource(name=”bean名字”)

或

@Resource(type=”bean的class”)
```

这个注解是属于J2EE的，减少了与spring的耦合。

**@Inject**

使用@Inject需要引用javax.inject.jar，它与Spring没有关系，是jsr330规范。与@Autowired有互换性。

@Named和Spring的@Component功能相同。@Named可以有值，如果没有值生成的Bean名称默认和类名相同。比如

```
@Named public class Person 
或
@Named(“cc”) public class Person
```

##### @Value注解

@Value注解,为了简化从properties里取配置，可以使用@Value, 可以properties文件中的配置值。

在dispatcher-servlet.xml里引入properties文件。

```xml
    <context:property-placeholder location=“classpath:test.properties” />  
```

在程序里使用@Value:

```java
        @Value(“${wx_appid}”)  
    public String appid;  
```

即使给变量赋了初值也会以配置文件的值为准。

@Value 为属性注入值（属性上） 支持如下方式的注入：

```java
//》注入普通字符
@Value("Michael Jackson")String name;
//》注入操作系统属性
@Value("#{systemProperties['os.name']}")String osName;
//》注入表达式结果
@Value("#{ T(java.lang.Math).random() * 100 }") String randomNumber;
//》注入其它bean属性
@Value("#{domeClass.name}")String name;
//》注入文件资源
@Value("classpath:com/hgs/hello/test.txt")String Resource file;
//》注入网站资源
@Value("http://www.cznovel.com")Resource url;
//》注入配置文件
Value("${book.name}")String bookName;
```

使用java类注入配置使用方法：

- 编写配置文件（test.properties）

  ```
  book.name=《三体》
  ```

- @PropertySource 加载配置文件(类上)

  ```
  @PropertySource("classpath:com/hgs/hello/test/test.propertie")
  ```

- 还需配置一个PropertySourcesPlaceholderConfigurer的bean。

**@Value注入List、Map**

```java

@Value("#{'${blackList}'.split(',')}")
private List<String> blackList;


@Value("#{${paramMaps}}")
private Map<String,String> paramMaps;
```

配置文件

```properties
blackList=127.0.0.1,192.0.0.1
paramMaps="{userId: 'u123', seckillId: 'g00001'}"
```

@Value至此springEL,但是@ConfigurationProperties是不支持的。

##### 切面（AOP）相关注解

Spring支持AspectJ的注解式切面编程。

@Aspect 声明一个切面（类上） 使用@After、@Before、@Around定义建言（advice），可直接将拦截规则（切点）作为参数。

@After 在方法执行之后执行（方法上） @Before 在方法执行之前执行（方法上） @Around 在方法执行之前与之后执行（方法上）

@PointCut 声明切点 在java配置类中使用@EnableAspectJAutoProxy注解开启Spring对AspectJ代理的支持（类上）

##### 环境切换

@Profile 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）

@Conditional Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

##### 异步相关

@EnableAsync 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）

@Async 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要@EnableAsync开启异步任务）

**@Async异步方法调用**

 java里使用线程用3种方法：

- 继承Thread，重写run方法

- 实现Runnable,重写run方法

- 使用Callable和Future接口创建线程，并能得到返回值。

而使用@Async可视为第4种方法。基于@Async标注的方法，称之为异步方法,这个注解用于标注某个方法或某个类里面的所有方法都是需要异步处理的。被注解的方法被调用的时候，会在新线程中执行，而调用它的方法会在原来的线程中执行。

第一步配置XML:

```xml
    <!–扫描注解，其中包括@Async –>  
    <context:component-scan base-package=“com.test”/>  
    <!– 支持异步方法执行, 指定一个缺省的executor给@Async使用–>  
    <task:annotation-driven executor=“defaultAsyncExecutor”  />   
    <!—配置一个线程执行器–>  
    <task:executor id=” defaultAsyncExecutor ”pool-size=“100-10000” queue-capacity=“10” keep-alive =”5”/>  
```

参数解读：

- <task:executor />配置参数：

- id：当配置多个executor时，被@Async(“id”)指定使用；也被作为线程名的前缀。

- pool-size：

  - core size：最小的线程数，缺省：1

  - max size：最大的线程数，缺省：Integer.MAX_VALUE

- queue-capacity：当最小的线程数已经被占用满后，新的任务会被放进queue里面，当这个queue的capacity也被占满之后，pool里面会创建新线程处理这个任务，直到总线程数达到了max size，这时系统会拒绝这个任务并抛出TaskRejectedException异常（缺省配置的情况下，可以通过rejection-policy来决定如何处理这种情况）。缺省值为：Integer.MAX_VALUE

- keep-alive：超过core size的那些线程，任务完成后，再经过这个时长（秒）会被结束掉

- rejection-policy：当pool已经达到max size的时候，如何处理新任务

  - ABORT（缺省）：抛出TaskRejectedException异常，然后不执行DISCARD：不执行，也不抛出异常

  - DISCARD_OLDEST：丢弃queue中最旧的那个任务

  - CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行

第二步在类或方法上添加@Async，当调用该方法时，则该方法即是用异常执行的方法单独开个新线程执行。

```java
    @Async(“可以指定执行器id，也可以不指定”)  
        public static void testAsyncVoid (){  
            try {  
                //让程序暂停100秒，相当于执行一个很耗时的任务  
        System.out.println(“异常执行打印字符串”);  
                Thread.sleep(100000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
```

当在外部调用testAsync方法时即在新线程中执行，由上面<task: annotation-driven/>执行器去维护线程。

总结：先用context:component-scan去扫描注解，让spring能识别到@Async注解，然后task:annotation-driven去驱动@Async注解，并可以指定默认的线程执行器executor。那么当用@Async注解的方法或类得到调用时，线程执行器会创建新的线程去执行。

上面方法是无返回值的情况，还有异常方法有返回值的例子。

```java
    @Async  
    public Future<String> testAsyncReturn () {    
        System.out.println(“Execute method asynchronously - ”    
          + Thread.currentThread().getName());    
        try {    
            Thread.sleep(5000);    
            return new AsyncResult<String>(“hello world !!!!”);    
        } catch (InterruptedException e) {    
            //    
        }    
        return null;    
    }  
```

返回的数据类型为Future类型，接口实现类是AsyncResult.

调用方法如下：

```
    public void test(){  
        Future<String> future = cc.testAsyncReturn();    
        while (true) {  ///这里使用了循环判断，等待获取结果信息    
            if (future.isDone()) {  //判断是否执行完毕    
                System.out.println(“Result from asynchronous process - ” + future.get());    
                break;    
            }    
            System.out.println(“Continue doing something else. ”);    
            Thread.sleep(1000);    
        }    
    }  
```

通过不停的检查Future的状态来获取当前的异步方法是否执行完毕

编程的方式使用@Async:

```java
    @Configuration    
    @EnableAsync   
//@EnableAsync 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）
    public class SpringConfig {    
          
        private int corePoolSize = 10;    
        private int maxPoolSize = 200;   
        private int queueCapacity = 10;    
        private String ThreadNamePrefix = “MyLogExecutor-“;    
        
        @Bean    
        public Executor logExecutor() {    
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();    
            executor.setCorePoolSize(corePoolSize);    
            executor.setMaxPoolSize(maxPoolSize);    
            executor.setQueueCapacity(queueCapacity);    
            executor.setThreadNamePrefix(ThreadNamePrefix);    
            // rejection-policy：当pool已经达到max size的时候，如何处理新任务    
            // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行    
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());    
            executor.initialize();    
            return executor;    
        }  
    }  
```

##### 定时任务相关

@EnableScheduling 在配置类上使用，开启计划任务的支持（类上）

@Scheduled 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

##### @Enable*注解说明

这些注解主要用来开启对xxx的支持。 @EnableAspectJAutoProxy 开启对AspectJ自动代理的支持

@EnableAsync 开启异步方法的支持

@EnableScheduling 开启计划任务的支持

@EnableWebMvc 开启Web MVC的配置支持

@EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持

@EnableJpaRepositories 开启对SpringData JPA Repository的支持

@EnableTransactionManagement 开启注解式事务的支持

@EnableTransactionManagement 开启注解式事务的支持

@EnableCaching 开启注解式的缓存支持

##### 测试相关注解

@RunWith 运行器，Spring中通常用于对JUnit的支持

```
@RunWith(SpringJUnit4ClassRunner.class)
```

@ContextConfiguration 用来加载配置ApplicationContext，其中classes属性用来加载配置类

```
@ContextConfiguration(classes={TestConfig.class})
```

##### SpringMVC部分注解

@EnableWebMvc 在配置类中开启Web MVC的配置支持，如一些ViewResolver或者MessageConverter等，若无此句，重写WebMvcConfigurerAdapter方法（用于对SpringMVC的配置）。

@Controller 声明该类为SpringMVC中的Controller

@RequestMapping 用于映射Web请求，包括访问路径和参数（类或方法上）

@ResponseBody 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）

@RequestBody 允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）

@PathVariable 用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。

@RestController 该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。

@ControllerAdvice 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了@Controller的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上， 这对所有注解了 @RequestMapping的控制器内的方法有效。

@ExceptionHandler 用于全局处理控制器里的异常

@InitBinder 用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。

@ModelAttribute 本来的作用是绑定键值对到Model里，在@ControllerAdvice中是让全局的@RequestMapping都能获得在此处设置的键值对。