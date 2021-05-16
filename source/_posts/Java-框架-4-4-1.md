---
title: Spring 测试
date: 2019-05-22 16:18:59
tags:
 - Java
 - 框架
 - Spring
categories:
 - Java
 - Spring
---

### 单元测试

单元测试的作用以及重要性其实不用在这里不断的重复了.为了保证软件的可靠性,单元测试可能是最容易执行的一个可靠的手段了.一方面，程序员通过编写单元测试来验证自己程序的有效性,另外一方面,管理者通过持续自动的执行单元测试和分析单元测试的覆盖率等来确保软件本身的质量.

<!--more-->

JAVA生态圈里面说起单元测试一般都会使用`JUnit`或者`TestNG`.其中`JUnit`可能使用的更加频繁一些,`JUnit4`使用注解以及各种其他框架对它的支持,形成了一个完善的单元测试的生态圈.

本着“不写单元测试的程序员不是好程序员”原则，我在坚持写着单元测试，不敢说所有的Java web应用都基于Spring，但至少一半以上都是基于Spring的。

发现通过Spring进行bean管理后，做测试会有各种不足。

例如，很多人做单元测试的时候，还要在Before方法中，初始化Spring容器，导致容器被初始化多次。

```java

    @Before  
     public void init() {  
          ApplicationContext ctx = new FileSystemXmlApplicationContext( "classpath:spring/spring-basic.xml");  
          baseDao = (IBaseDao) ctx.getBean("baseDao");  
          assertNotNull(baseDao);  
     }    

```

 在开发基于Spring的应用时，如果你还直接使用Junit进行单元测试，那你就错过了Spring满汉全席中最重要的一道硬菜。 

再说这道菜之前，我们先来讨论下，在基于Spring的javaweb项目中使用Junit直接进行单元测试有什么不足

1. 导致多次Spring容器初始化问题 

   根据JUnit测试方法的调用流程，每执行一个测试方法都会创建一个测试用例的实例并调用setUp()方法。由于一般情况下，我们在setUp()方法中初始化Spring容器，这意味着如果测试用例有多少个测试方法，Spring容器就会被重复初始化多次。虽然初始化Spring容器的速度并不会太慢，但由于可能会在Spring容器初始化时执行加载Hibernate映射文件等耗时的操作，如果每执行一个测试方法都必须重复初始化Spring容器，则对测试性能的影响是不容忽视的； 

   使用Spring测试套件，Spring容器只会初始化一次！

2. 需要使用硬编码方式手工获取Bean 

   在测试用例类中我们需要通过ctx.getBean()方法从Spirng容器中获取需要测试的目标Bean，并且还要进行强制类型转换的造型操作。这种乏味的操作迷漫在测试用例的代码中，让人觉得烦琐不堪； 

   使用Spring测试套件，测试用例类中的属性会被自动填充Spring容器的对应Bean ,无须在手工设置Bean！ 

3. 数据库现场容易遭受破坏 

   测试方法对数据库的更改操作会持久化到数据库中。虽然是针对开发数据库进行操作，但如果数据操作的影响是持久的，可能会影响到后面的测试行为。举个例子，用户在测试方法中插入一条ID为1的User记录，第一次运行不会有问题，第二次运行时，就会因为主键冲突而导致测试用例失败。所以应该既能够完成功能逻辑检查，又能够在测试完成后恢复现场，不会留下“后遗症”； 

   使用Spring测试套件，Spring会在你验证后，自动回滚对数据库的操作，保证数据库的现场不被破坏，因此重复测试不会发生问题！ 

4. 不方便对数据操作正确性进行检查 
   假如我们向登录日志表插入了一条成功登录日志，可是我们却没有对t_login_log表中是否确实添加了一条记录进行检查。一般情况下，我们可能是打开数据库，肉眼观察是否插入了相应的记录，但这严重违背了自动测试的原则。试想在测试包括成千上万个数据操作行为的程序时，如何用肉眼进行检查？ 

   只要你继承Spring的测试套件的用例类，你就可以通过jdbcTemplate在同一事务中访问数据库，查询数据的变化，验证操作的正确性！ 

下面，让我们看看，使用Spring测试套件后，代码是如何变优雅的。

#### Spring Test+JUnit

现在的JAVA WEB项目中,起码一半以上的项目是使用了Spring的.因此,单纯的使用`JUnit`来进行单元测试并不是十分的好用,对于由Spring管理的`Bean`要进行单元测试,首先需要实例化Spring上下文,然后又需要手动的去注入依赖的`Bean`,比较麻烦.特别是对于有事务的单元测试,或数据库数据测试,单独使用`JUnit`几乎无法完成.

所幸,SpringFramework也意识到了这点,于是推出了`spring-test`模块,他能完成Spring环境与Junit单元测试环境的对接.让我们只专注于单元测试本身进行书写,而由它来完成Spring容器的初始化、`Bean`的获取、数据库事务的管理、数据操作正确性检查等等。

依赖：

```

<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>4.9</version>  
    <scope>test</scope>  
</dependency>   
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>${spring.version}</version>  
    <scope>provided</scope>  
</dependency>   

```

测试基类：

```java

@RunWith(SpringJUnit4ClassRunner.class)  //使用junit4进行测试  
@ContextConfiguration   
({"/spring/app*.xml","/spring/service/app*.xml"}) //加载配置文件  

//------------如果加入以下代码，所有继承该类的测试类都会遵循该配置，也可以不加，在测试类的方法上///控制事务，参见下一个实例  
//这个非常关键，如果不加入这个注解配置，事务控制就会完全失效！  
//@Transactional  
//这里的事务关联到配置文件中的事务控制器（transactionManager = "transactionManager"），同时//指定自动回滚（defaultRollback = true）。这样做操作的数据才不会污染数据库！  
//@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)  
//------------  
public class BaseJunit4Test {  
}  

```

接着是我们自己的测试类：

```java
public class UserAssignServiceTest extends BaseJunit4Test{  
@Resource  //自动注入,默认按名称  
private IBaseDao baseDao;  
@Test   //标明是测试方法  
@Transactional   //标明此方法需使用事务  
@Rollback(false)  //标明使用完此方法后事务不回滚,true时为回滚  
public void insert( ) {  
            String sql="insert into user(name,password) values(?,?)";  
            Object[] objs=new Object[]{"00","000"};  
            baseDao.insert( sql , objs );  
            String sql1="select * from user where name=? and password=? ";  
            List<Map<String,Object>> list=baseDao.queryForList( sql1 , objs );  
            System.out.println(list);  
            assertTrue(list.size( )>0);   
         }  
}  
```

也可以在测试数据库时使用事务回滚，避免测试数据保留。其余的测试注解和Junit4相同。只是引入了Spring管理的bean。

在JUnit4的测试方法中打事务注解@Transactional，默认会按照@Rollback(true)来进行处理，无论如何都会回滚，打不打注解@Rollback或@Rollback(true)已经不重要了。而在@Transactional的基础上加上@Rollback(false)之后，效果就好像是单单这个测试函数这一层没有打事务似的，而不会传播到嵌套的service层的事务内，即如果在service层的事务中，插入数据后又发生异常，最终在service层里还是会进行rollback，数据并不会插入到数据库中。 

下面再理一下@TransactionConfiguration过时与替代写法

```java
//过时的写法
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)
@Transactional


//替代写法，在高版本的Spring框架中（Spring4.2以后）
@RunWith(SpringJUnit4ClassRunner.class)
//测试Spring MVC时添加
@WebAppConfiguration
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)
@Transactional(transactionManager = "transactionManager")
@Rollback(value = true)
```

这里需要说明的是：
1. 原来的defaultRollback属性现在由专门的注解@Rollback（新增注解）代替，其中只有一个属性就是boolean型的value，作用没变，值为true表示测试时如果涉及了数据库的操作，那么测试完成后，该操作会回滚，也就是不会改变数据库内容；值为false则与此相反，表示你测试的内容中对数据库的操作会真实的执行到数据库中，不会回滚。官方文档中还给出了一个新注解@Commit，该注解与@Rollback只能使用一个，同时用貌似可能出现问题，@Commit注解中无属性需要设置，不像@Rollback中还有一个value属性，用了@Commit，你的测试操作会改变数据库，不会回滚，等同于@Rollback(value=false)。这里建议使用@Rollback，不要用@Commit，这样起码你有两种选择可以选。
2. 原来放在@TransactionConfiguration注解中的transactionManager属性现在放在了@Transactionl注解中。

顺带提一下@RunWith注解的作用:

首先我们写了一个测试类，该类中会有许多测试方法，测试方法上面会利用@BeforeClass、@before、@Test、@after、@AfterClass这5个注解进行测试，类有了，方法有了，那么当你执行某个测试方法时，是由谁来调用的这个测试方法呢，或者说，你执行的测试方法是在哪里运行的呢，答案就是@RunWith注解里面标注的类，也就是说这个注解的作用是告诉系统你执行测试方法时，调用者是谁，这里就是SpringJunit4ClassRunner类。默认情况下，也就是假如你省略了@RunWith注解，测试类上面不写它，系统默认的是相当于你注解了@RunWith(BlockJUnit4ClassRunner.class)，实际上SpringJunit4ClassRunner继承了BlockJUnit4ClassRunner。因为你的项目中使用了Spring，那么测试方法中要测试的内容一般都会用到Spring管理的bean，此时你只能用SpringJunit4ClassRunner而不能用BlockJUnit4ClassRunner了，否则Spring环境中管理的东西你是无法在测试方法中使用的，测试方法拿不到时会报空指针异常。

#### Mockito和SpringTest

经过上面所说的`JUnit`+`SpringTest`,基本上可以满足80%的单元测试了。但是，由于现在的系统越来越复杂，相互之间的依赖越来越多。特别是微服务化以后的系统，往往一个模块的代码需要依赖几个其他模块的东西。因此，在做单元测试的时候，往往很难构造出需要的依赖。一个单元测试，我们只关心一个小的功能，但是为了这个小的功能能跑起来，可能需要依赖一堆其他的东西，这就导致了单元测试无法进行。所以，我们就需要再测试过程中引入`Mock`测试。

所谓的`Mock`测试就是在测试过程中，对于一些不容易构造的、或者和这次单元测试无关但是上下文又有依赖的对象，用一个虚拟的对象（Mock对象）来模拟，以便单元测试能够进行。

比如有一段代码的依赖为：

![1.jpg](https://i.loli.net/2019/06/09/5cfc7af37ac4d69718.jpg)

当我们要进行单元测试的时候，就需要给`A`注入`B`和`C`,但是`C`又依赖了`D`，`D`又依赖了`E`。这就导致了，A的单元测试很难得进行。

但是，当我们使用了Mock来进行模拟对象后，我们就可以把这种依赖解耦，只关心A本身的测试，它所依赖的B和C，全部使用**Mock出来**的对象，并且给`MockB`和`MockC`指定一个明确的行为。就像这样：

![2.jpg](https://i.loli.net/2019/06/09/5cfc7b30d5be875158.jpg)

因此，当我们使用Mock后，对于那些难以构建的对象，就变成了个模拟对象，只需要提前的做`Stubbing`（桩）即可，所谓做桩数据，也就是告诉Mock对象，当与之交互时执行何种行为过程。比如当调用B对象的b()方法时，我们期望返回一个`true`，这就是一个设置桩数据的预期。

在JAVA中，Mock测试框架主要有[Mockito](http://mockito.org/),[Jmock](http://www.jmock.org/),[EsayMock](http://easymock.org/),[PowerMock](https://github.com/jayway/powermock)等等。其中`Mockito`最为方便和简单，用的人也最多。而`PowerMock`是对`Mockito`的一个增强,增加了对静态、final、私有方法的Mock，但是基本用法和`Mockito`大致相同。对因此，我们使用`Mockito`作为Mock的框架。

要在项目中使用`Mockito`非常的简单，只需要在项目的Maven中引入：

```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.1.0</version>
    <scope>test</scope>
</dependency>
```

其实这两者的整合也非常的简单。和他们单独使用的时候并没有什么区别。

```java
@RunWith(SpringJUnit4ClassRunner.class) // 整合Spring-Test与JUnit
@ContextConfiguration(locations = {"classpath:application_bean_commons.xml",
        "classpath:application_bean_rmdbaccess.xml",
        "classpath:application_pm_service_poolmanage.xml"}) // 加载需要的配置配置
@Transactional          //事务回滚,便于重复测试
public class TestPoolService{
    @Autowired
    private IPoolService poolService;
    
    @Mock
    private IPoolDao poolDao;
    @Before
    public void setUp() {
		  //可以在每一次单元测试前准备执行一些东西
		  MockitoAnnotations.initMocks(this);
		  
		  //把Spring上下文注入的对象给替换掉
		  ReflectionTestUtils.setField(AopTargetUtils.getTarget(poolService), ”poolDao“,poolDao);
		  
		  /*对void的方法设置模拟*/
        Mockito.doAnswer(invocationOnMock -> {
            System.out.println("进入了Mock");
            return null;
        }).when(poolDao).insert(Mockito.any());
    }
    
    @Test
    public void testAddPool() {
    	 
        Pool pool = new Pool();
        pool.setPoolName("test_pool_001");
        pool.setDescription("test add pool");
        pool = poolService.addPool(pool);
        //进行结果的验证
        assertNotNull(pool.getId());
        assertNotNull(pool.getPoolCode());
    }
}
```

与单独使用Mockito相比，最大的不同其实就是在*setUp()*方法中调用的`ReflectionTestUtils.setField(AopTargetUtils.getTarget(poolService), ”poolDao“,poolDao);`这个方法。*ReflectionTestUtils*是*Spring-test*提供的一个用于反射处理测试类的工具，通过这个，我们可以替换某一个被spring所管理的bean的成员变量。把他换成我们Mock出来的模拟对象。

当然这又引出了一个问题，就是如果依赖的对象的依赖对象需要被Mock，那么手动的不断重复的找需要被Mock的成员变量非常的麻烦。因此，我们可以写一个`AbstractTestExecutionListener`监听器，当注入依赖的时候，找到被Mock的变量，以及需要被注入的变量，然后做关系的依赖。这样就能自动的对成员变量做替换了。

```java
public class MockitoDependencyInjectionTestExecutionListener extends DependencyInjectionTestExecutionListener {
    private static final Map<Class,Object> mockObject = new HashMap<>();
    
    @Override
    protected void injectDependencies(final TestContext testContext) throws Exception {
        super.injectDependencies(testContext);
        init(testContext);
    }
    
    private void injectMock(Object bean) throws Exception {
        Field[] fields;
        /*找到所有的测试用例的字段*/
        if (AopUtils.isAopProxy(bean)){
            // 如果是代理的话，找到真正的对象
            if(AopUtils.isJdkDynamicProxy(bean)) {
                Class targetClass = AopTargetUtils.getTarget(bean).getClass();
                if (targetClass == null) {
                    // 可能是远程实现  
                    return;
                }
                fields = targetClass.getDeclaredFields();
            } else { //cglib
                /*CGLIB的代理 不支持*/
                return;
            }
            
        }else {
            fields = bean.getClass().getDeclaredFields();
        }
        
        List<Field> injectFields = new ArrayList<>();
        
        /*判断字段上的注解*/
        for (Field field : fields) {
            Annotation[] annotations = field.getAnnotations();
            for (Annotation antt : annotations) {
                /*如果是Mock字段的,就直接注入Mock的对象*/
                if (antt instanceof org.mockito.Mock) {
                    // 注入mock实例  
                    Object mockObj = mock(field.getType());
                    mockObject.put(field.getType(),mockObj);
                    field.setAccessible(true);
                    field.set(bean, mockObj);
                } else if (antt instanceof Autowired) {
                    /*需要把所有标注为autowired的找到*/
                    injectFields.add(field);
                }
            }
        }
        
        /*访问每一个被注入的实例*/
        for (Field field : injectFields) {
            field.setAccessible(true);
            
            /*找到每一个字段的值*/
            Object object = field.get(bean);
            Class targetClass = field.getType();
            if (!replaceInstance(targetClass,bean,field.getName())){
                //如果没有被mock过.那么这个字段需要再一次的做递归
                injectMock(object);
            }
        }
    }
    
    private boolean replaceInstance(Class targetClass, Object bean, String fieldName) throws Exception {
        boolean beMocked = false;
        for (Map.Entry<Class, Object> classObjectEntry : mockObject.entrySet()) {
            Class type = classObjectEntry.getKey();
            if (type.isAssignableFrom(targetClass)){
                //如果这个字段是被mock了的对象.那么就使用这个mock的对象来替换
                ReflectionTestUtils.setField(AopTargetUtils.getTarget(bean), fieldName, classObjectEntry.getValue());
                beMocked = true;
                break;
            }
        }
        return beMocked;
    }
    
    private void init(final TestContext testContext) throws Exception {
        Object bean = testContext.getTestInstance();
        injectMock(bean);
    }
}
```

以上代码即为Mock初始化的监听器。它会查询这个测试用例的所有的成员变量。找到被标记为`@Mock`的变量，然后模拟出来。而后，又找到所有被标注为`@Autowired`的成员变量，判断变量类型是否是需要被Mock的。

当需要使用这个监听器的时候，只需要增加一个注解`@TestExecutionListeners`即可：

```java
@RunWith(SpringJUnit4ClassRunner.class) // 整合Spring-Test与JUnit
/*如果要加事物,那么当手动设置TestExecutionListeners的时候,需要把TransactionalTestExecutionListener这个也加上*/
@TestExecutionListeners({ MockitoDependencyInjectionTestExecutionListener.class, TransactionalTestExecutionListener.class })
@ContextConfiguration(locations = {"classpath:application_bean_commons.xml",
        "classpath:application_bean_rmdbaccess.xml",
        "classpath:application_pm_service_poolmanage.xml"}) // 加载需要的配置配置
@Transactional          //事务回滚,便于重复测试
public class TestPoolService{
    @Autowired
    private IPoolService poolService;
    
    @Mock
    private IPoolDao poolDao;
    @Before
    public void setUp() {		  
		  /*对void的方法设置模拟*/
        Mockito.doAnswer(invocationOnMock -> {
            System.out.println("进入了Mock");
            return null;
        }).when(poolDao).insert(Mockito.any());
    }
    
    @Test
    public void testAddPool() {
    	 
        Pool pool = new Pool();
        pool.setPoolName("test_pool_001");
        pool.setDescription("test add pool");
        pool = poolService.addPool(pool);
        //进行结果的验证
        assertNotNull(pool.getId());
        assertNotNull(pool.getPoolCode());
    }
}
```





参考：[使用Mockito和SpringTest进行单元测试](http://sunxiang0918.cn/2016/03/28/%E4%BD%BF%E7%94%A8Mockito%E5%92%8CSpringTest%E8%BF%9B%E8%A1%8C%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95/)