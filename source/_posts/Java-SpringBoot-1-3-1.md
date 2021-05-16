---
title: Spring boot 单元测试
date: 2019-06-30 16:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

本文使用的测试工具请参看：<https://hanyunpeng0521.github.io/categories/Java/测试/>

本文代码：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-3-test>

一个测试方法主要包括三部分：

1. setup

2. 执行操作

3. 验证结果

<!--more-->

```java
public class CalculatorTest {
    Calculator mCalculator;

    @Before // setup
    public void setup() {
        mCalculator = new Calculator();
    }

    @Test //assert 部分可以帮助我们验证一个结果
    public void testAdd() throws Exception {
        int sum = mCalculator.add(1, 2);
        assertEquals(3, sum);  //为了简洁，往往会static import Assert里面的所有方法。
    }

    @Test
    @Ignore("not implemented yet") // 测试时忽略该方法
    public void testMultiply() throws Exception {
    }

    // 表示验证这个测试方法将抛出 IllegalArgumentException 异常，若没抛出，则测试失败
    @Test(expected = IllegalArgumentException.class)
    public void test() {
        mCalculator.divide(4, 0);
    }
}
```

**基本注解介绍**

- `@BeforeClass` 在所有测试方法执行前执行一次，一般在其中写上整体初始化的代码。

- `@AfterClass` 在所有测试方法后执行一次，一般在其中写上销毁和释放资源的代码。

  ```java
  // 注意这两个都是静态方法
  @BeforeClass
  public static void test(){
      
  }
  @AfterClass
  public static void test(){
  }
  ```

- `@Before` 在每个方法测试前执行，一般用来初始化方法（比如我们在测试别的方法时，类中与其他测试方法共享的值已经被改变，为了保证测试结果的有效性，我们会在@Before注解的方法中重置数据）

- `@After` 在每个测试方法执行后，在方法执行完成后要做的事情。
- `@Test(timeout = 1000)` 测试方法执行超过1000毫秒后算超时，测试将失败。
- `@Test(expected = Exception.class)` 测试方法期望得到的异常类，如果方法执行没有抛出指定的异常，则测试失败。
- `@Ignore("not ready yet")` 执行测试时将忽略掉此方法，如果用于修饰类，则忽略整个类。
- `@Test` 编写一般测试用例用。
- `@RunWith` 在 Junit 中有很多个 Runner，他们负责调用你的测试代码，每一个 Runner 都有各自的特殊功能，你根据需要选择不同的 Runner 来运行你的测试代码。

如果我们只是简单的做普通 Java 测试，不涉及 Spring Web 项目，你可以省略 `@RunWith` 注解，你要根据需要选择不同的 Runner 来运行你的测试代码。

**一个单元测试类执行顺序为：**
 `@BeforeClass` -> `@Before` -> `@Test` -> `@After`  -> `@AfterClass`

**每一个测试方法的调用顺序为：**
 `@Before` -> `@Test` -> `@After`

**测试方法执行顺序**

按照设计，Junit不指定test方法的执行顺序。

- `@FixMethodOrder(MethodSorters.JVM)`:保留测试方法的执行顺序为JVM返回的顺序。每次测试的执行顺序有可能会所不同。
- `@FixMethodOrder(MethodSorters.NAME_ASCENDING`) :根据测试方法的方法名排序,按照词典排序规则(ASC,从小到大,递增)。

Failure  是测试失败，Error 是程序出错。

**测试方法命名约定**

Maven本身并不是一个单元测试框架，它只是在构建执行到特定生命周期阶段的时候，通过插件来执行JUnit或者TestNG的测试用例。这个插件就是maven-surefire-plugin，也可以称为测试运行器(Test Runner)，它能兼容JUnit 3、JUnit 4以及TestNG。

在默认情况下，maven-surefire-plugin的test目标会自动执行测试源码路径（默认为src/test/java/）下所有符合一组命名模式的测试类。这组模式为：

- `*/Test.java`：任何子目录下所有命名以Test开关的Java类。
- `*/Test.java`：任何子目录下所有命名以Test结尾的Java类。
- `*/TestCase.java`：任何子目录下所有命名以TestCase结尾的Java类。

### 基于 Spring 的单元测试编写

首先我们项目一般都是 MVC 分层的，而单元测试主要是在 Dao 层和 Service 层上进行编写。从项目结构上来说，Service 层是依赖 Dao 层的，但是从单元测试角度，对某个 Service 进行单元的时候，他所有依赖的类都应该进行Mock。而 Dao 层单元测试就比较简单了，只依赖数据库中的数据。

SpringBoot 提供了许多实用工具和注解来帮助测试应用程序，主要包括以下两个模块。

- spring-boot-test： 支持测试的核心内容。
- spring-boot-test-autoconfigure：支持测试的自动化配置。

开发进行只要引入spring-boot-starter-test的依赖 就能引入这些SpringBoot测试模块，还能引入一些像Junit，AssertJ，Hamcrest及其他一些有用的类库，具体如下所示。

> - Junit: Java应用程序单元测试标准类库。
> - Spring Test & Spring Boot Test: Spring Boot 应用程序功能集成化测试支持。
> - AssertJ: 一个轻量级断言类库。
> - Hamcrest: 一个对象匹配器类库。
> - Mockito: 一个java Mock测试框架。
> - JSONassert: 一个用于JSON的断言库。
> - JsonPath: 一个Json操作类库。

#### Mockito

Mockito是mocking框架，它让你用简洁的API做测试。而且Mockito简单易学，它可读性强和验证语法简洁。
 Mockito 是一个针对 Java 的单元测试模拟框架，它与 EasyMock 和 jMock 很相似，都是为了简化单元测试过程中测试上下文 ( 或者称之为测试驱动函数以及桩函数 ) 的搭建而开发的工具

相对于 EasyMock 和 jMock，Mockito 的优点是通过在执行后校验哪些函数已经被调用，消除了对期望行为（expectations）的需要。其它的 mocking 库需要在执行前记录期望行为（expectations），而这导致了丑陋的初始化代码。

SpringBoot 中的 `pom.xml` 文件需要添加的依赖：

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
```

进入 `spring-boot-starter-test-2.1.3.RELEASE.pom` 可以看到该依赖中已经有单元测试所需的大部分依赖，如：

- JUnit：Java 应用程序单元测试标准类库。
- Spring Test & Spring Boot Test：Spring Boot 应用程序功能集成化测试支持。
- Mockito：一个Java Mock测试框架。
- AssertJ：一个轻量级的断言类库。
- Hamcrest：一个对象匹配器类库。
- JSONassert：一个用于JSON的断言库。
- JsonPath：一个JSON操作类库。

若为其他 spring 项目，需要自己添加 Junit 和 mockito 项目。

**常用的 Mockito 方法：**

| **方法名**                                                   | **描述**                                  |
| ------------------------------------------------------------ | ----------------------------------------- |
| Mockito.mock(classToMock)                                    | 模拟对象                                  |
| Mockito.verify(mock)                                         | 验证行为是否发生                          |
| Mockito.when(methodCall).thenReturn(value1).thenReturn(value2) | 触发时第一次返回value1，第n次都返回value2 |
| Mockito.doThrow(toBeThrown).when(mock).[method]              | 模拟抛出异常。                            |
| Mockito.mock(classToMock,defaultAnswer)                      | 使用默认Answer模拟对象                    |
| Mockito.when(methodCall).thenReturn(value)                   | 参数匹配                                  |
| Mockito.doReturn(toBeReturned).when(mock).[method]           | 参数匹配（直接执行不判断）                |
| Mockito.when(methodCall).thenAnswer(answer))                 | 预期回调接口生成期望值                    |
| Mockito.doAnswer(answer).when(methodCall).[method]           | 预期回调接口生成期望值（直接执行不判断）  |
| Mockito.spy(Object)                                          | 用spy监控真实对象,设置真实对象行为        |
| Mockito.doNothing().when(mock).[method]                      | 不做任何返回                              |
| Mockito.doCallRealMethod().when(mock).[method] //等价于Mockito.when(mock.[method]).thenCallRealMethod(); | 调用真实的方法                            |
| reset(mock)                                                  | 重置mock                                  |

**示例**：

- 验证行为是否发生

  ```java
  //模拟创建一个List对象
  List<Integer> mock =  Mockito.mock(List.class);
  //调用mock对象的方法
  mock.add(1);
  mock.clear();
  //验证方法是否执行
  Mockito.verify(mock).add(1);
  Mockito.verify(mock).clear();
  ```

- 多次触发返回不同值

  ```java
  //mock一个Iterator类
  Iterator iterator = mock(Iterator.class);
  //预设当iterator调用next()时第一次返回hello，第n次都返回world
  Mockito.when(iterator.next()).thenReturn("hello").thenReturn("world");
  //使用mock的对象
  String result = iterator.next() + " " + iterator.next() + " " + iterator.next();
  //验证结果
  Assert.assertEquals("hello world world",result);
  ```

- 模拟抛出异常

  ```java
  @Test(expected = IOException.class)//期望报IO异常
  public void when_thenThrow() throws IOException{
        OutputStream mock = Mockito.mock(OutputStream.class);
        //预设当流关闭时抛出异常
        Mockito.doThrow(new IOException()).when(mock).close();
        mock.close();
    }
  ```

- 使用默认Answer模拟对象

  RETURNS_DEEP_STUBS 是创建mock对象时的备选参数之一
  `以下方法deepstubsTest和deepstubsTest2是等价的`

  ```java
    @Test
    public void deepstubsTest(){
        A a=Mockito.mock(A.class,Mockito.RETURNS_DEEP_STUBS);
        Mockito.when(a.getB().getName()).thenReturn("Beijing");
        Assert.assertEquals("Beijing",a.getB().getName());
    }
  
    @Test
    public void deepstubsTest2(){
        A a=Mockito.mock(A.class);
        B b=Mockito.mock(B.class);
        Mockito.when(a.getB()).thenReturn(b);
        Mockito.when(b.getName()).thenReturn("Beijing");
        Assert.assertEquals("Beijing",a.getB().getName());
    }
    class A{
        private B b;
        public B getB(){
            return b;
        }
        public void setB(B b){
            this.b=b;
        }
    }
    class B{
        private String name;
        public String getName(){
            return name;
        }
        public void setName(String name){
            this.name = name;
        }
        public String getSex(Integer sex){
            if(sex==1){
                return "man";
            }else{
                return "woman";
            }
        }
    }
  ```

- 参数匹配

  ```java
  @Test
  public void with_arguments(){
      B b = Mockito.mock(B.class);
      //预设根据不同的参数返回不同的结果
      Mockito.when(b.getSex(1)).thenReturn("男");
      Mockito.when(b.getSex(2)).thenReturn("女");
      Assert.assertEquals("男", b.getSex(1));
      Assert.assertEquals("女", b.getSex(2));
      //对于没有预设的情况会返回默认值
      Assert.assertEquals(null, b.getSex(0));
  }
  class B{
      private String name;
      public String getName(){
          return name;
      }
      public void setName(String name){
          this.name = name;
      }
      public String getSex(Integer sex){
          if(sex==1){
              return "man";
          }else{
              return "woman";
          }
      }
  }
  ```

- 匹配任意参数

  `Mockito.anyInt()` 任何 int 值 ；
  `Mockito.anyLong()` 任何 long 值 ；
  `Mockito.anyString()` 任何 String 值 ；

  `Mockito.any(XXX.class)` 任何 XXX 类型的值 等等。

  ```java
  @Test
  public void with_unspecified_arguments(){
      List list = Mockito.mock(List.class);
      //匹配任意参数
      Mockito.when(list.get(Mockito.anyInt())).thenReturn(1);
      Mockito.when(list.contains(Mockito.argThat(new IsValid()))).thenReturn(true);
      Assert.assertEquals(1,list.get(1));
      Assert.assertEquals(1,list.get(999));
      Assert.assertTrue(list.contains(1));
      Assert.assertTrue(!list.contains(3));
  }
  class IsValid extends ArgumentMatcher<List>{
      @Override
      public boolean matches(Object obj) {
          return obj.equals(1) || obj.equals(2);
      }
  }
  ```

  注意：使用了参数匹配，那么所有的参数都必须通过matchers来匹配
  Mockito继承Matchers，anyInt()等均为Matchers方法
  当传入两个参数，其中一个参数采用任意参数时，指定参数需要matchers来对比

  ```java
  Comparator comparator = mock(Comparator.class);
  comparator.compare("nihao","hello");
  //如果你使用了参数匹配，那么所有的参数都必须通过matchers来匹配
  Mockito.verify(comparator).compare(Mockito.anyString(),Mockito.eq("hello"));
  //下面的为无效的参数匹配使用
  //verify(comparator).compare(anyString(),"hello");
  ```

- 自定义参数匹配

  ```java
  @Test
  public void argumentMatchersTest(){
     //创建mock对象
     List<String> mock = mock(List.class);
     //argThat(Matches<T> matcher)方法用来应用自定义的规则，可以传入任何实现Matcher接口的实现类。
     Mockito.when(mock.addAll(Mockito.argThat(new IsListofTwoElements()))).thenReturn(true);
     Assert.assertTrue(mock.addAll(Arrays.asList("one","two","three")));
  }
  
  class IsListofTwoElements extends ArgumentMatcher<List>
  {
     public boolean matches(Object list)
     {
         return((List)list).size()==3;
     }
  }
  ```

- 预期回调接口生成期望值

  ```java
  @Test
  public void answerTest(){
        List mockList = Mockito.mock(List.class);
        //使用方法预期回调接口生成期望值（Answer结构）
        Mockito.when(mockList.get(Mockito.anyInt())).thenAnswer(new CustomAnswer());
        Assert.assertEquals("hello world:0",mockList.get(0));
        Assert.assertEquals("hello world:999",mockList.get(999));
    }
    private class CustomAnswer implements Answer<String> {
        @Override
        public String answer(InvocationOnMock invocation) throws Throwable {
            Object[] args = invocation.getArguments();
            return "hello world:"+args[0];
        }
    }
  等价于：(也可使用匿名内部类实现)
  @Test
   public void answer_with_callback(){
        //使用Answer来生成我们我们期望的返回
        Mockito.when(mockList.get(Mockito.anyInt())).thenAnswer(new Answer<Object>() {
            @Override
            public Object answer(InvocationOnMock invocation) throws Throwable {
                Object[] args = invocation.getArguments();
                return "hello world:"+args[0];
            }
        });
        Assert.assertEquals("hello world:0",mockList.get(0));
       Assert. assertEquals("hello world:999",mockList.get(999));
    }
  ```

- 预期回调接口生成期望值（直接执行）

  ```java
  @Test
  public void testAnswer1(){
  List<String> mock = Mockito.mock(List.class);  
        Mockito.doAnswer(new CustomAnswer()).when(mock).get(Mockito.anyInt());  
        Assert.assertEquals("大于三", mock.get(4));
        Assert.assertEquals("小于三", mock.get(2));
  }
  public class CustomAnswer implements Answer<String> {  
    public String answer(InvocationOnMock invocation) throws Throwable {  
        Object[] args = invocation.getArguments();  
        Integer num = (Integer)args[0];  
        if( num>3 ){  
            return "大于三";  
        } else {  
            return "小于三";   
        }  
    }
  }
  ```

- 修改对未预设的调用返回默认期望（指定返回值）

  ```java
  //mock对象使用Answer来对未预设的调用返回默认期望值
  List mock = Mockito.mock(List.class,new Answer() {
       @Override
       public Object answer(InvocationOnMock invocation) throws Throwable {
           return 999;
       }
   });
   //下面的get(1)没有预设，通常情况下会返回NULL，但是使用了Answer改变了默认期望值
   Assert.assertEquals(999, mock.get(1));
   //下面的size()没有预设，通常情况下会返回0，但是使用了Answer改变了默认期望值
   Assert.assertEquals(999,mock.size());
  ```

- 用spy监控真实对象,设置真实对象行为

  ```java
      @Test(expected = IndexOutOfBoundsException.class)
      public void spy_on_real_objects(){
          List list = new LinkedList();
          List spy = Mockito.spy(list);
          //下面预设的spy.get(0)会报错，因为会调用真实对象的get(0)，所以会抛出越界异常
          //Mockito.when(spy.get(0)).thenReturn(3);
  
          //使用doReturn-when可以避免when-thenReturn调用真实对象api
          Mockito.doReturn(999).when(spy).get(999);
          //预设size()期望值
          Mockito.when(spy.size()).thenReturn(100);
          //调用真实对象的api
          spy.add(1);
          spy.add(2);
          Assert.assertEquals(100,spy.size());
          Assert.assertEquals(1,spy.get(0));
          Assert.assertEquals(2,spy.get(1));
          Assert.assertEquals(999,spy.get(999));
      }
  ```

- 不做任何返回

  ```java
  @Test
  public void Test() {
      A a = Mockito.mock(A.class);
      //void 方法才能调用doNothing()
      Mockito.doNothing().when(a).setName(Mockito.anyString());
      a.setName("bb");
      Assert.assertEquals("bb",a.getName());
  }
  class A {
      private String name;
      private void setName(String name){
          this.name = name;
      }
      private String getName(){
          return name;
      }
  }
  ```

- 调用真实的方法

  ```java
  @Test
  public void Test() {
      A a = Mockito.mock(A.class);
      //void 方法才能调用doNothing()
      Mockito.when(a.getName()).thenReturn("bb");
      Assert.assertEquals("bb",a.getName());
      //等价于Mockito.when(a.getName()).thenCallRealMethod();
      Mockito.doCallRealMethod().when(a).getName();
      Assert.assertEquals("zhangsan",a.getName());
  }
  class A {
      public String getName(){
          return "zhangsan";
      }
  }
  ```

- 重置 mock

  ```java
      @Test
      public void reset_mock(){
          List list = mock(List.class);
          Mockito. when(list.size()).thenReturn(10);
          list.add(1);
          Assert.assertEquals(10,list.size());
          //重置mock，清除所有的互动和预设
          Mockito.reset(list);
          Assert.assertEquals(0,list.size());
      }
  ```

- @Mock 注解

  ```java
  public class MockitoTest {
      @Mock
      private List mockList;
      //必须在基类中添加初始化mock的代码，否则报错mock的对象为NULL
      public MockitoTest(){
          MockitoAnnotations.initMocks(this);
      }
      @Test
      public void AnnoTest() {
              mockList.add(1);
          Mockito.verify(mockList).add(1);
      }
  }
  ```

- 指定测试类使用运行器：MockitoJUnitRunner

  ```java
  @RunWith(MockitoJUnitRunner.class)
  public class MockitoTest2 {
      @Mock
      private List mockList;
  
      @Test
      public void shorthand(){
          mockList.add(1);
          Mockito.verify(mockList).add(1);
      }
  }
  ```

**@MockBean**

使用 `@MockBean` 可以解决单元测试中的一些依赖问题，示例如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ServiceWithMockBeanTest {
    @MockBean
    SampleDependencyA dependencyA;
    @Autowired
    SampleService sampleService;

    @Test
    public void testDependency() {
        when(dependencyA.getExternalValue(anyString())).thenReturn("mock val: A");
        assertEquals("mock val: A", sampleService.foo());
    }
}
```

`@MockBean` 只能 mock 本地的代码——或者说是自己写的代码，对于储存在库中而且又是以 Bean 的形式装配到代码中的类无能为力。

`@SpyBean` 解决了 SpringBoot 的单元测试中 `@MockBean` 不能 mock 库中自动装配的 Bean 的局限（目前还没需求，有需要的自己查阅资料）。

#### MockMvc

MockMvc是由spring-test包提供，实现了对Http请求的模拟，能够直接使用网络的形势，转换到Controller的调用，是的测试速度快，不依赖网络环境。同时提供了一套验证的工具，结果的验证十分方便。
 接口MockMvcBuilder，一共一个唯一的build方法，用来构造MockMvc。主要有两个实现：`StandaloneMockMvcBuilder`和`DefaultMockMvcBuilder`,分别对应两种测试方式，即独立安装和继承web环境测试(并不会集成真正的web环境，而是通过相应的Mock API进行模拟测试，无需启动服务)。MockMvcBuilders提供了对应的创建方法`standaloneSetup` 方法和`webAppContextSetup`方法,在使用时直接调用即可。

```java
private MockMvc mockMvc;

@Autowire
private WebApplicationContext webApplicationContext;

@Before
public void setup() {
    /*实例化方式一*/
    mockMvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
    /*实例化方式二*/
    mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
}
```

单元测试方法：

```csharp
  @Test
    public void testHello() throws Exception {
/*
        1.mockMvc.perform 执行一个请求
        2.MockMvcRequestBuilders.get("XXX")构造一个请求
        3.ResultActions.param()
        4.ResultActions.accept()
        5.ResultActions.andExpect
        6.ResultActions.andDo 添加一个结果处理器，表示要对结果做点什么事情。
        7.ResultActions.andReturn 表示执行完成后，返回响应的结果。
*/
        mockMvc.perform(MockMvcRequestBuilders.get("/mock-mvc/test-get")
                /*设置返回类型为utf-8,否则默认为ISO-8859-1*/
                .accept(MediaType.APPLICATION_JSON_UTF8_VALUE)
                .param("name","tom"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string("hello"))
                .andDo(MockMvcResultHandlers.print());
    }
```

**整个过程如下**

1. 准备测试环境

2. 通过MockMvc执行请求

3. 添加验证断言

4. 添加结果处理器
5. 得到MvcResult进行分自定义断言/进行下一步异步请求

6. 卸载测试环境

注意事项：如果使用DefaultMockMvcBuilder进行MockMvc实例化时需在SpringBoot启动类上添加组件扫描的package的指定，否则会出现404

```css
@ComponentScan(basePackages = "com.creators")
```

**相关API**

- RequestBuilder提供了一个方法buildRequest(ServletContext servletContext)用于构建
- MockHttpServletRequest；其中有两个子类MockHttpServletRequestBuilder和MockMultipartHttpServletRequestBuilder(文件上传使用) 。

- MockMvcRequestBuilders提供get、post等多种方法用来实例化RequestBuilder。

- ResultActions,MockMvc.perform(RequestBuilder requestBuilder)的返回值，提供三种能力：`andExpect` 添加断言判断结果是否达到预期；`andDo`,添加结果处理器，比如示例中的打印。`andReturn`返回验证成功后的MvcResult,用于自定义验证/下一步的异步处理。

**一些常用的测试**
 测试普通控制器

```dart
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}",1))
        /*验证存储模型数据*/
       .andExpect(MockMvcResultMatchers.model().attributeExists("user"))
        /*验证viewName*/
       .andExpect(MockMvcResultMatchers.view().name("user/view"))
        /*验证视图渲染时forward到的jsp*/
       .andExpect(MockMvcResultMatchers.forwardedUrl("/WEB-INF/jsp/user/view/jsp"))
        /*验证状态码*/
       .andExpect(MockMvcResultMatchers.status().isOk())
        /*输出MvcResult到控制台*/
       .andDo(MockMvcResultHandlers.print());
```

得到MvcResult自定义验证

```csharp
  MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get( "/user/{id}",1)) 
        .andReturn();
  Assert.assertNotNull(result.getModelAndView().getModel().get("user"));
```

验证请求参数绑定到模型数据及flash属性

```kotlin
mockMvc.perform(MockMvcRequestBuilders.post("/user").param("name","wang"))
                /*验证执行控制器类*/
                .andExpect(MockMvcResultMatchers.handler().handlerType(UserController.class))
                /*验证执行控制器方法名*/
                .andExpect(MockMvcResultMatchers.handler().methodName("create"))
                /*验证页面没有错误*/
                .andExpect(MockMvcResultMatchers.model().hasNoErrors())
                /*验证存在flash属性*/
                .andExpect(MockMvcResultMatchers.flash().attributeExists("success"))
                /*验证视图名称*/
                .andExpect(MockMvcResultMatchers.view().name("redirect:/user"));
```

文件上传

```csharp
 byte[] bytes = new byte[]{1,2};
        mockMvc.perform(MockMvcRequestBuilders.multipart("/user/{id}/icon",1L).file("icon",bytes))
                .andExpect(MockMvcResultMatchers.model().attribute("icon",bytes))
                .andExpect(MockMvcResultMatchers.view().name("success"));
```

JSON请求/响应验证

```dart
String requestBody = "{\"id\":1,\"name\":\"wang\"}";
        mockMvc.perform(MockMvcRequestBuilders.post("/user")
                .contentType(MediaType.APPLICATION_JSON_UTF8_VALUE)
                .content(requestBody)
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(MockMvcResultMatchers.content().contentType(MediaType.APPLICATION_JSON))
            /*检查返回JSON数据中某个值的内容： 请参考http://goessner.net/articles/JsonPath/*/
            .andExpect(MockMvcResultMatchers.jsonPath("$.id").value(1));

        String errorBody = "{id:1,name:wang}";
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.post("/user")
                .contentType(MediaType.APPLICATION_JSON).content(errorBody)
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isBadRequest())
                .andReturn();
        Assert.assertTrue(HttpMessageNotReadableException.class.isAssignableFrom(mvcResult.getResolvedException().getClass()));
```

异步测试

```kotlin
  MvcResult mvcResult1 = mockMvc.perform(MockMvcRequestBuilders.get("/user/async?id=1&name=wang"))
                .andExpect(MockMvcResultMatchers.request().asyncStarted())
                .andExpect(MockMvcResultMatchers.request().asyncResult(CoreMatchers.instanceOf(User.class)))
                .andReturn();
        mockMvc.perform(MockMvcRequestBuilders.asyncDispatch(mvcResult1))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").value(1));
```

使用MultiValueMap构建参数

```csharp
 MultiValueMap<String,String> params = new LinkedMultiValueMap<>();
        params.add("name","wang");
        params.add("hobby","sleep");
        params.add("hobby","eat");
        mockMvc.perform(MockMvcRequestBuilders.post("/user").params(params));
```

模拟session和cookie

```csharp
 mockMvc.perform(MockMvcRequestBuilders.get("/index").sessionAttr("name", "value"));
        mockMvc.perform(MockMvcRequestBuilders.get("/index").cookie(new Cookie("name", "value")));
```

全局配置

```dart
  mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .defaultRequest(MockMvcRequestBuilders.get("/user/1").requestAttr("default",true))
                .alwaysDo(MockMvcResultHandlers.print())
                .alwaysExpect(MockMvcResultMatchers.request().attribute("default",true))
                .build();
```

如果是测试Service层代码 可以在单元测试方法上加上@Transactional注解，在测试完毕后，数据能自动回滚。

### 参考

1. [SpringBoot 单元测试详解（Mockito、MockBean）](https://www.jianshu.com/p/ecbd7b5a2021)
2. [Springboot 单元测试详解](https://www.jianshu.com/p/81fc2c98774f)