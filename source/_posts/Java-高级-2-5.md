---
title: Jdk动态代理和Cglib动态代理 底层源码分析
date: 2019-12-19 15:18:59
tags:
 - Java
categories:
 - Java
 - 高级
---

java动态代理主要有2种，Jdk动态代理、Cglib动态代理，本文主要讲解Jdk动态代理的使用、运行机制、以及源码分析。当spring没有手动开启Cglib动态代理，即：`<aop:aspectj-autoproxy proxy-target-class="true"/>`或`@EnableAspectJAutoProxy(proxyTargetClass = true)`，默认使用的就是Jdk动态代理。动态代理的应用范围很广，例如：日志、事务管理、缓存等。本文将模拟@Cacheable，即缓存在动态代理中的应用进行讲解。**需要注意的是，Jdk动态代理相比起cglib动态代理，Jdk动态代理的对象必须实现接口，否则将报错。我们也将带着这个问题在源码分析中寻找答案**

<!--more-->

当@Cacheable注解在方法上时

1. 在方法执行前，将调用Jdk动态代理优先查找Redis(或其他缓存)
2. 当缓存不存在时，执行方法，例如查询数据库
3. 在方法执行后，再次调用Jdk动态代理，将结果缓存到Redis中

### Jdk动态代理

#### 使用

1. 创建接口UserService

   ```java
   public interface UserService {
       public String getUserByName(String name);
   }
   ```

2. 创建接口实现类UserServiceImpl

   ```java
   public class UserServiceImpl implements UserService {
       @Override
       public String getUserByName(String name) {
           System.out.println("从数据库中查询到:" + name);
           return name;
       }
   }
   ```

3. 创建Jdk动态代理JdkCacheHandler，用于增强UserServiceImpl方法前后的缓存逻辑

   ```java
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
   ```

4. 创建测试类

   ```java
   public class JdkTest {
   
       @Test
       public void test() {
           //在调用测试代理的方法前加(此方法通知虚拟保存产生的代理文件)
           //System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true"); 
           //上面这段代码如果是放在了 JUnit 的单元测试方法中就不会生效，要放在 main 方法中才行。
           UserService userService = new UserServiceImpl();
           JdkCacheHandler jdkCacheHandler = new JdkCacheHandler(userService);
           UserService proxy = (UserService) jdkCacheHandler.createJDKProxy();
   
           System.out.println("==========================");
           proxy.getUserByName("px");
           System.out.println("==========================");
   
           System.out.println(proxy.getClass());
       }
   }
   ```

5. 输出

   ```
   ==========================
   查找数据库前，在缓存中查找是否存在:px
   从数据库中查询到:px
   查找数据库后，将px加入到缓存中
   ==========================
   class com.sun.proxy.$Proxy4
   ```

#### 调用机制

##### 查看$Proxy代码

可以看到当经过Jdk动态代理以后，生产的proxy已经不再是UserService类型了，而是$Proxy4类型，想要了解其调用机制，得先获取到proxy类的代码

```
System.out.println(proxy.getClass());

class com.sun.proxy.$Proxy4
```

1. 修改JVM运行参数，添加`-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true`

2. 运行test即可在com.sun.proxy查看代码，此时生产的是class，idea打开会自动反编译

3. 在上方输出中可以看到代理类是$Proxy4，至此获取到$Proxy4的源代码，接下去分析代理类的调用机制

   ```java
   package com.sun.proxy;
   
   import com.hyp.learn.proxy.UserService;
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   import java.lang.reflect.UndeclaredThrowableException;
   
   public final class $Proxy4 extends Proxy implements UserService {
       private static Method m1;
       private static Method m3;
       private static Method m2;
       private static Method m0;
   
       public $Proxy4(InvocationHandler var1) throws  {
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

##### 调用机制

1. 从proxy调用开始

   ```java
   // JdkTest.java
   proxy.getUserByName("px");
   ```

2. proxy是$Proxy4类型，因此进入$Proxy4的getUserByName方法

   ```java
   // $Proxy4.class
   public final class $Proxy4 extends Proxy implements UserService {
       ...
   
       // 构造器，传入JdkCacheHandler类的对象，正是下方调用的super.h属性
       public $Proxy4(InvocationHandler var1) throws  {
           super(var1);
       }
   
       public final String getUserByName(String var1) throws  {
           try {
               /**
               *   调用父类的h属性的invoke方法
               *   在下面的源码分析中，会发现h属性正是JdkCacheHandler类createJDKProxy方法中所传入的this
               *   Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
               *   而this指代的正是JdkCacheHandler类的对象，因此最后调用的是JdkCacheHandler的invoke方法
               */
               return (String)super.h.invoke(this, m3, new Object[]{var1});
           } catch (RuntimeException | Error var3) {
               throw var3;
           } catch (Throwable var4) {
               throw new UndeclaredThrowableException(var4);
           }
       }
   
       ...
   
       static {
           ...
           m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
           m3 = Class.forName("proxy.jdk.UserService").getMethod("getUserByName", Class.forName("java.lang.String"));
           m2 = Class.forName("java.lang.Object").getMethod("toString");
           m0 = Class.forName("java.lang.Object").getMethod("hashCode");
           ...
       }
   }
   ```

3. 因此`h.invok`实际调用的正是JdkCacheHandler类的invoke方法

   ```java
   // JdkCacheHandler.java
   public class JdkCacheHandler implements InvocationHandler {
       ...
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   
           System.out.println("查找数据库前，在缓存中查找是否存在:" + args[0]);
           // 触发目标类方法
           Object result = method.invoke(target, args);
           System.out.printf("查找数据库后，将%s加入到缓存中\r\n", result);
           return result;
       }
   }
   ```

4. 而`method.invoke(target, args)`中的method = getUserByName，target = 构造函数传进来的UserServiceImpl对象，args = "bugpool"

   ```java
   // UserServiceImpl.java
   public class UserServiceImpl implements UserService {
       @Override
       public String getUserByName(String name) {
           System.out.println("从数据库中查询到:" + name);
           return name;
       }
   }
   ```

#### 源码分析

了解完Jdk动态代理的调用机制，所有核心问题都落在了$Proxy4类的对象proxy是如何生成的上面？即下面这句代码上，这里先给出概述，有利于宏观上看源码。在开始之前先复习一下java的运行机制：

1. 所有.java文件经过编译生成.class文件 
2. 通过类加载器classLoad将.class中的字节码加载到JVM中 
3.  运行

```java
// 创建JDK代理
public Object createJDKProxy() {
    Class clazz = target.getClass();
    // 创建JDK代理需要3个参数
    // 目标类加载器：用于加载生成的字节码
    // 目标类接口：用于生成字节码，也就是说$Proxy的生产仅仅需要接口数组就可以完成
    // 代理类对象(即本身)：用于回调invoke方法，实现方法的增强
    return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
}
```

1. 通过clazz.getInterfaces()获取到所有接口，通过接口可以生成类似以下字节码（注意以下给出的是代码），细细观察会发现其实各个接口方法生成的代码都是一样的，只有`(String)super.h.invoke(this, m3, new Object[]{var1}`的m3和参数有可能是不同的。所以其实想生成$Proxy字节码，只需要接口数组就已经完全足够了

   ```java
   public final String getUserByName(String var1) throws  {
       try {
           return (String)super.h.invoke(this, m3, new Object[]{var1});
       } catch (RuntimeException | Error var3) {
           throw var3;
       } catch (Throwable var4) {
           throw new UndeclaredThrowableException(var4);
       }
   }
   ```

2. 此时已经获取到$Proxy4.class的字节码，但是此处的字节码还未加载到JVM中，因此需要调用clazz.getClassLoader()传进来的类加载器进行加载，并得到对应的class，也就是$Proxy类

3. 获取$Proxy类的构造函数，该构造函数有一个重要的参数h

4. 通过反射调用$Proxy类的构造函数，`cons.newInstance(new Object[]{h});`构造函数的h正是传入的this，也就是JdkCacheHandler类的对象

5. 将反射获取到的$Proxy对象放回

##### 源码分析

1. 从`Proxy.newProxyInstance`开始跟踪代码（注：①代表上方概述的步骤1）

   ```java
   // JdkCacheHandler.java
   // 创建JDK代理
   public Object createJDKProxy() {
       Class clazz = target.getClass();
       // 创建JDK代理需要3个参数，目标类加载器、目标类接口、代理类对象(即本身)
       return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
   }
   ```

2. 跟踪代码newProxyInstance，这里需要注意在①②过程结束后，③④过程调用前，即`Class cl = getProxyClass0(loader, intfs);`结束后，cl变量一直都只是class，即$Proxy4类，并未生成对应的对象，这里不要混淆类和对象

   ```java
   // Proxy.java
   // loader类加载器，interfaces目标类实现的所有接口，h即InvocationHandler类的对象
   public static Object newProxyInstance(ClassLoader loader,
                                             Class<?>[] interfaces,
                                             InvocationHandler h)
           throws IllegalArgumentException
       {
           // 校验InvocationHandler是否为空
           Objects.requireNonNull(h);
   
           // 该目标类实现的接口数组
           final Class<?>[] intfs = interfaces.clone();
           // 安全检查
           final SecurityManager sm = System.getSecurityManager();
           if (sm != null) {
               checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
           }
   
           /*
            * Look up or generate the designated proxy class.
            */
           // 当缓存中存在代理类则直接获取，否则生成代理类
           // ①②步骤，核心代码，即生成代理类字节码以及加载都在这里进行
           Class<?> cl = getProxyClass0(loader, intfs);
   
           /*
            * Invoke its constructor with the designated invocation handler.
            */
           try {
               if (sm != null) {
                   checkNewProxyPermission(Reflection.getCallerClass(), cl);
               }
   
               // ③ 从生成的代理类中获取构造函数
               // constructorParams = { InvocationHandler.class };
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
               // ④ 调用构造函数，将InvocationHandler作为参数实例化代理对象
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

3. 跟踪`Class<?> cl = getProxyClass0(loader, intfs);`

   ```java
   // Proxy.java
   private static Class<?> getProxyClass0(ClassLoader loader,
                                          Class<?>... interfaces) {
       if (interfaces.length > 65535) {
           throw new IllegalArgumentException("interface limit exceeded");
       }
   
       // If the proxy class defined by the given loader implementing
       // the given interfaces exists, this will simply return the cached copy;
       // otherwise, it will create the proxy class via the ProxyClassFactory
       // 如果在类加载器中已经存在实现了对应接口的代理类，则直接返回缓存中的代理类
       // 否则，通过ProxyClassFactory新建代理类
       return proxyClassCache.get(loader, interfaces);
   }
   ```

4. 跟踪`proxyClassCache.get(loader, interfaces);`（注：Jdk动态代理对已经生成加载过的代理类进行了缓存以提高性能，缓存的相关代码不是我们关心的重点，可以跳过相关代码）本段代码我们主要关心`V value = supplier.get();`其中supplier本质是factory，通过`new Factory(key, parameter, subKey, valuesMap)`创建

   ```java
   // WeakCache.java
   //K和P就是WeakCache定义中的泛型，key是类加载器，parameter是接口类数组
   public V get(K key, P parameter) {
           // 检查接口数组是否为空
           Objects.requireNonNull(parameter);
   
           expungeStaleEntries();
   
           Object cacheKey = CacheKey.valueOf(key, refQueue);
   
           // lazily install the 2nd level valuesMap for the particular cacheKey
           ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
           if (valuesMap == null) {
               ConcurrentMap<Object, Supplier<V>> oldValuesMap
                   = map.putIfAbsent(cacheKey,
                                     valuesMap = new ConcurrentHashMap<>());
               if (oldValuesMap != null) {
                   valuesMap = oldValuesMap;
               }
           }
   
           // create subKey and retrieve the possible Supplier<V> stored by that
           // subKey from valuesMap
           Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
           //通过sub-key得到supplier，实质就是factory
           Supplier<V> supplier = valuesMap.get(subKey);
           Factory factory = null;
   
           while (true) {
               if (supplier != null) {
                   // supplier might be a Factory or a CacheValue<V> instance
                   // ①②步骤都在这里，如果supplier不为空，则直接调用get方法返回代理类
                   V value = supplier.get();
                   if (value != null) {
                       return value;
                   }
               }
               // else no supplier in cache
               // or a supplier that returned null (could be a cleared CacheValue
               // or a Factory that wasn't successful in installing the CacheValue)
   
               // lazily construct a Factory
               if (factory == null) {
                   // 创建对应factory，此段代码在死循环中，下一次supplier.get()将会获取到代理类并退出循环
                   factory = new Factory(key, parameter, subKey, valuesMap);
               }
   
               if (supplier == null) {
                   supplier = valuesMap.putIfAbsent(subKey, factory);
                   if (supplier == null) {
                       // successfully installed Factory
                       // 赋值给supplier
                       supplier = factory;
                   }
                   // else retry with winning supplier
               } else {
                   if (valuesMap.replace(subKey, supplier, factory)) {
                       // successfully replaced
                       // cleared CacheEntry / unsuccessful Factory
                       // with our Factory
                       supplier = factory;
                   } else {
                       // retry with current supplier
                       supplier = valuesMap.get(subKey);
                   }
               }
           }
       }
   ```

5. 跟踪`V value = supplier.get();`即Factory类的get方法，这里大部分的工作还是在做校验和缓存，我们只关心核心逻辑`valueFactory.apply(key, parameter);`其中valueFactory是上一个步骤传入的ProxyClassFactory

   ```java
   // Factory.java    
   public synchronized V get() { // serialize access
           // re-check
           // 再次检查，supplier是否是当前对象
           Supplier<V> supplier = valuesMap.get(subKey);
           if (supplier != this) {
               // something changed while we were waiting:
               // might be that we were replaced by a CacheValue
               // or were removed because of failure ->
               // return null to signal WeakCache.get() to retry
               // the loop
               return null;
           }
           // else still us (supplier == this)
   
           // create new value
           V value = null;
           try {
               // valueFactory 是前序传进来的 new ProxyClassFactory()
               // ①②步骤，核心逻辑，调用valueFactory.apply生成对应代理类并加载
               value = Objects.requireNonNull(valueFactory.apply(key, parameter));
           } finally {
               if (value == null) { // remove us on failure
                   valuesMap.remove(subKey, this);
               }
           }
           // the only path to reach here is with non-null value
           assert value != null;
   
           // wrap value with CacheValue (WeakReference)
           CacheValue<V> cacheValue = new CacheValue<>(value);
   
           // put into reverseMap
           reverseMap.put(cacheValue, Boolean.TRUE);
   
           // try replacing us with CacheValue (this should always succeed)
           if (!valuesMap.replace(subKey, this, cacheValue)) {
               throw new AssertionError("Should not reach here");
           }
   
           // successfully replaced us with new CacheValue -> return the value
           // wrapped by it
           return value;
       }
   ```

6. 跟踪核心逻辑

   - Jdk动态代理通过拼凑的方式拼凑出$Proxy的全类名：com.sun.proxy.$proxy0.class
   - ③生产字节码`byte[] proxyClassFile = ProxyGenerator.generateProxyClass( proxyName, interfaces, accessFlags);`可以看出Jdk动态代理需要interfaces接口数组进行生成字节码，这也是文章开头提出为什么必须实现接口的原因。同时从参数也可以看出需要生成字节码其实只需要接口数组，不需要其他信息。其实实现原理大概也可以猜出，Jdk动态代理通过遍历所有接口方法，为方法生成对应的`return (String)super.h.invoke(this, m0~n, new Object[]{var1});`代码
   - ④加载字节码：在③中获取到了字节码的字节数组，接下去就是调用classLoader将所有的字节码读入到JVM中

   ```java
   // ProxyClassFactory.java
   private static final class ProxyClassFactory
           implements BiFunction<ClassLoader, Class<?>[], Class<?>>
       {
           // prefix for all proxy class names
           // 代理类名称前缀
           private static final String proxyClassNamePrefix = "$Proxy";
   
           // next number to use for generation of unique proxy class names
           // 代理类计数器
           private static final AtomicLong nextUniqueNumber = new AtomicLong();
   
           @Override
           public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
   
               Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
               // 校验代理类接口
               for (Class<?> intf : interfaces) {
                   /*
                    * Verify that the class loader resolves the name of this
                    * interface to the same Class object.
                    */
                   Class<?> interfaceClass = null;
                   try {
                       interfaceClass = Class.forName(intf.getName(), false, loader);
                   } catch (ClassNotFoundException e) {
                   }
                   if (interfaceClass != intf) {
                       throw new IllegalArgumentException(
                           intf + " is not visible from class loader");
                   }
                   /*
                    * Verify that the Class object actually represents an
                    * interface.
                    */
                   if (!interfaceClass.isInterface()) {
                       throw new IllegalArgumentException(
                           interfaceClass.getName() + " is not an interface");
                   }
                   /*
                    * Verify that this interface is not a duplicate.
                    */
                   if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                       throw new IllegalArgumentException(
                           "repeated interface: " + interfaceClass.getName());
                   }
               }
   
               // 代理类包名
               String proxyPkg = null;     // package to define proxy class in
               int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
   
               /*
                * Record the package of a non-public proxy interface so that the
                * proxy class will be defined in the same package.  Verify that
                * all non-public proxy interfaces are in the same package.
                */
               // 当接口修饰符是public，则所有包都可以使用
               // 当接口是非public，则生成的代理类必须和接口在与非public接口同一个包下
               // 如果非public的接口均在同一个包下，则生成的代理类放在非public接口同一个包下
               // 而如果非public的接口存在多个，且在不同包下，则抛出异常
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
                   // 如果都是公有的接口，则代理类默认放在com.sun.proxy package
                   proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
               }
   
               /*
                * Choose a name for the proxy class to generate.
                */
               // 生成计数器，例如$proxy0~n
               long num = nextUniqueNumber.getAndIncrement();
               // 代理类名，com.sun.proxy.$proxy0.class
               String proxyName = proxyPkg + proxyClassNamePrefix + num;
   
               /*
                * Generate the specified proxy class.
                */
               // ③生成代理类字节码
               byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                   proxyName, interfaces, accessFlags);
               try {
                   // ④使用传进来的classLoader将代理类字节码加载到JVM中
                   return defineClass0(loader, proxyName,
                                       proxyClassFile, 0, proxyClassFile.length);
               } catch (ClassFormatError e) {
                   /*
                    * A ClassFormatError here means that (barring bugs in the
                    * proxy class generation code) there was some other
                    * invalid aspect of the arguments supplied to the proxy
                    * class creation (such as virtual machine limitations
                    * exceeded).
                    */
                   throw new IllegalArgumentException(e.toString());
               }
           }
       }
   ```

### Cglib动态代理

CGLIB(Code Generation Library)是一个开源项目！是一个强大的，高性能，高质量的Code生成类库，

它可以在运行期扩展Java类与实现Java接口。Hibernate用它来实现PO(Persistent Object 持久化对象)字节码的动态生成。

CGLIB是一个强大的高性能的代码生成包。它广泛的被许多AOP的框架使用，例如Spring AOP为他们提供

方法的interception（拦截）。CGLIB包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。

除了CGLIB包，脚本语言例如Groovy和BeanShell，也是使用ASM来生成java的字节码。当然不鼓励直接使用ASM，

因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。

#### 使用

```java
public class CglibCacheHandler implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("查找数据库前，在缓存中查找是否存在:" + objects[0]);
        // 触发目标类方法
        Object result = methodProxy.invokeSuper(o, objects);
        System.out.printf("查找数据库后，将%s加入到缓存中\r\n", result);
        return result;
    }

    //疑问？
    //好像并没有持有被代理对象的引用
    public Object getInstance(Class clazz) throws Exception{

        Enhancer enhancer = new Enhancer();
        //把父类设置为谁？
        //这一步就是告诉cglib，生成的子类需要继承哪个类
        enhancer.setSuperclass(clazz);
        //设置回调
        enhancer.setCallback(this);

        //第一步、生成源代码
        //第二步、编译成class文件
        //第三步、加载到JVM中，并返回被代理对象
        return enhancer.create();
    }
}

public class CglibTest {
    @Test
    public void test() throws Exception {
		//保存生成类
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/home/hyp/Downloads/data");

        //设置代理类对象
        UserServiceImpl userService = (UserServiceImpl) new CglibCacheHandler().getInstance(UserServiceImpl.class);
        //在调用代理类中方法时会被我们实现的方法拦截器进行拦截

        System.out.println("==========================");
        userService.getUserByName("px");
        System.out.println("==========================");

        System.out.println(userService.getClass());
    }
}
```

输出

```
CGLIB debugging enabled, writing to '/home/hyp/Downloads/data'
==========================
查找数据库前，在缓存中查找是否存在:px
从数据库中查询到:px
查找数据库后，将px加入到缓存中
==========================
class com.hyp.learn.proxy.UserServiceImpl$$EnhancerByCGLIB$$14f99790

```

#### CGLIB动态代理源码分析

实现CGLIB动态代理必须实现MethodInterceptor(方法拦截器)接口，源码如下:

```java
package net.sf.cglib.proxy;
 
public interface MethodInterceptor
extends Callback
{
    public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,
                               MethodProxy proxy) throws Throwable;
 
}
```

这个接口只有一个intercept()方法，这个方法有4个参数：

1）obj表示增强的对象，即实现这个接口类的一个对象；

2）method表示要被拦截的方法；

3）args表示要被拦截方法的参数；

4）proxy表示要触发父类的方法对象；

在上面的Client代码中，通过Enhancer.create()方法创建代理对象，create()方法的源码：

```java
public Object create() {
        classOnly = false;
        argumentTypes = null;
        return createHelper();
    }
```

该方法含义就是如果有必要就创建一个新类，并且用指定的回调对象创建一个新的对象实例，

使用的父类的参数的构造方法来实例化父类的部分。核心内容在createHelper()中，源码如下:

```java
private Object createHelper() {
        preValidate();
        Object key = KEY_FACTORY.newInstance((superclass != null) ? superclass.getName() : null,
                ReflectUtils.getNames(interfaces),
                filter == ALL_ZERO ? null : new WeakCacheKey<CallbackFilter>(filter),
                callbackTypes,
                useFactory,
                interceptDuringConstruction,
                serialVersionUID);
        this.currentKey = key;
        Object result = super.create(key);
        return result;
    }
```

preValidate()方法校验callbackTypes、filter是否为空，以及为空时的处理。

通过newInstance()方法创建EnhancerKey对象，作为Enhancer父类AbstractClassGenerator.create()方法

创建代理对象的参数。

```java
protected Object create(Object key) {
        try {
            ClassLoader loader = getClassLoader();
            Map<ClassLoader, ClassLoaderData> cache = CACHE;
            ClassLoaderData data = cache.get(loader);
            if (data == null) {
                synchronized (AbstractClassGenerator.class) {
                    cache = CACHE;
                    data = cache.get(loader);
                    if (data == null) {
                        Map<ClassLoader, ClassLoaderData> newCache = new WeakHashMap<ClassLoader, ClassLoaderData>(cache);
                        data = new ClassLoaderData(loader);
                        newCache.put(loader, data);
                        CACHE = newCache;
                    }
                }
            }
            this.key = key;
            Object obj = data.get(this, getUseCache());
            if (obj instanceof Class) {
                return firstInstance((Class) obj);
            }
            return nextInstance(obj);
        } catch (RuntimeException e) {
            throw e;
        } catch (Error e) {
            throw e;
        } catch (Exception e) {
            throw new CodeGenerationException(e);
        }
    }
```

真正创建代理对象方法在nextInstance()方法中，该方法为抽象类AbstractClassGenerator的一个方法,在子类Enhancer中实现，实现源码如下：

```java
    protected Object firstInstance(Class type) throws Exception {
        return this.classOnly ? type : this.createUsingReflection(type);
    }

    protected Object nextInstance(Object instance) {
        Enhancer.EnhancerFactoryData data = (Enhancer.EnhancerFactoryData)instance;
        if (this.classOnly) {
            return data.generatedClass;
        } else {
            Class[] argumentTypes = this.argumentTypes;
            Object[] arguments = this.arguments;
            if (argumentTypes == null) {
                argumentTypes = Constants.EMPTY_CLASS_ARRAY;
                arguments = null;
            }

            return data.newInstance(argumentTypes, arguments, this.callbacks);
        }
    }
```

看看data.newInstance(argumentTypes, arguments, callbacks)方法，

第一个参数为代理对象的构成器类型，第二个为代理对象构造方法参数，第三个为对应回调对象。

最后根据这些参数，通过反射生成代理对象，源码如下：

```java
        public Object newInstance(Class[] argumentTypes, Object[] arguments, Callback[] callbacks) {
            this.setThreadCallbacks(callbacks);

            Object var4;
            try {
                if (this.primaryConstructorArgTypes == argumentTypes || Arrays.equals(this.primaryConstructorArgTypes, argumentTypes)) {
                    var4 = ReflectUtils.newInstance(this.primaryConstructor, arguments);
                    return var4;
                }

                var4 = ReflectUtils.newInstance(this.generatedClass, argumentTypes, arguments);
            } finally {
                this.setThreadCallbacks((Callback[])null);
            }

            return var4;
        }
```

最后生成代理对象`class com.hyp.learn.proxy.UserServiceImpl$$EnhancerByCGLIB$$14f99790`：

![GsfyTA.png](https://s1.ax1x.com/2020/04/06/GsfyTA.png)

反编译代码

```java
package com.hyp.learn.proxy;

import java.lang.reflect.Method;
import net.sf.cglib.core.ReflectUtils;
import net.sf.cglib.core.Signature;
import net.sf.cglib.proxy.Callback;
import net.sf.cglib.proxy.Factory;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class UserServiceImpl$$EnhancerByCGLIB$$14f99790 extends UserServiceImpl implements Factory {
    private boolean CGLIB$BOUND;
    public static Object CGLIB$FACTORY_DATA;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;
    private MethodInterceptor CGLIB$CALLBACK_0;
    private static Object CGLIB$CALLBACK_FILTER;
    private static final Method CGLIB$getUserByName$0$Method;
    private static final MethodProxy CGLIB$getUserByName$0$Proxy;
    private static final Object[] CGLIB$emptyArgs;
    private static final Method CGLIB$equals$1$Method;
    private static final MethodProxy CGLIB$equals$1$Proxy;
    private static final Method CGLIB$toString$2$Method;
    private static final MethodProxy CGLIB$toString$2$Proxy;
    private static final Method CGLIB$hashCode$3$Method;
    private static final MethodProxy CGLIB$hashCode$3$Proxy;
    private static final Method CGLIB$clone$4$Method;
    private static final MethodProxy CGLIB$clone$4$Proxy;

    static void CGLIB$STATICHOOK1() {
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
        CGLIB$emptyArgs = new Object[0];
        Class var0 = Class.forName("com.hyp.learn.proxy.UserServiceImpl$$EnhancerByCGLIB$$14f99790");
        Class var1;
        Method[] var10000 = ReflectUtils.findMethods(new String[]{"equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
        CGLIB$equals$1$Method = var10000[0];
        CGLIB$equals$1$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$1");
        CGLIB$toString$2$Method = var10000[1];
        CGLIB$toString$2$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$2");
        CGLIB$hashCode$3$Method = var10000[2];
        CGLIB$hashCode$3$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$3");
        CGLIB$clone$4$Method = var10000[3];
        CGLIB$clone$4$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$4");
        CGLIB$getUserByName$0$Method = ReflectUtils.findMethods(new String[]{"getUserByName", "(Ljava/lang/String;)Ljava/lang/String;"}, (var1 = Class.forName("com.hyp.learn.proxy.UserServiceImpl")).getDeclaredMethods())[0];
        CGLIB$getUserByName$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)Ljava/lang/String;", "getUserByName", "CGLIB$getUserByName$0");
    }

    final String CGLIB$getUserByName$0(String var1) {
        return super.getUserByName(var1);
    }

    public final String getUserByName(String var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? (String)var10000.intercept(this, CGLIB$getUserByName$0$Method, new Object[]{var1}, CGLIB$getUserByName$0$Proxy) : super.getUserByName(var1);
    }

    final boolean CGLIB$equals$1(Object var1) {
        return super.equals(var1);
    }

    public final boolean equals(Object var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var2 = var10000.intercept(this, CGLIB$equals$1$Method, new Object[]{var1}, CGLIB$equals$1$Proxy);
            return var2 == null ? false : (Boolean)var2;
        } else {
            return super.equals(var1);
        }
    }

    final String CGLIB$toString$2() {
        return super.toString();
    }

    public final String toString() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? (String)var10000.intercept(this, CGLIB$toString$2$Method, CGLIB$emptyArgs, CGLIB$toString$2$Proxy) : super.toString();
    }

    final int CGLIB$hashCode$3() {
        return super.hashCode();
    }

    public final int hashCode() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var1 = var10000.intercept(this, CGLIB$hashCode$3$Method, CGLIB$emptyArgs, CGLIB$hashCode$3$Proxy);
            return var1 == null ? 0 : ((Number)var1).intValue();
        } else {
            return super.hashCode();
        }
    }

    final Object CGLIB$clone$4() throws CloneNotSupportedException {
        return super.clone();
    }

    protected final Object clone() throws CloneNotSupportedException {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? var10000.intercept(this, CGLIB$clone$4$Method, CGLIB$emptyArgs, CGLIB$clone$4$Proxy) : super.clone();
    }

    public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
        String var10000 = var0.toString();
        switch(var10000.hashCode()) {
        case -1038061212:
            if (var10000.equals("getUserByName(Ljava/lang/String;)Ljava/lang/String;")) {
                return CGLIB$getUserByName$0$Proxy;
            }
            break;
        case -508378822:
            if (var10000.equals("clone()Ljava/lang/Object;")) {
                return CGLIB$clone$4$Proxy;
            }
            break;
        case 1826985398:
            if (var10000.equals("equals(Ljava/lang/Object;)Z")) {
                return CGLIB$equals$1$Proxy;
            }
            break;
        case 1913648695:
            if (var10000.equals("toString()Ljava/lang/String;")) {
                return CGLIB$toString$2$Proxy;
            }
            break;
        case 1984935277:
            if (var10000.equals("hashCode()I")) {
                return CGLIB$hashCode$3$Proxy;
            }
        }

        return null;
    }

    public UserServiceImpl$$EnhancerByCGLIB$$14f99790() {
        CGLIB$BIND_CALLBACKS(this);
    }

    public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] var0) {
        CGLIB$THREAD_CALLBACKS.set(var0);
    }

    public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] var0) {
        CGLIB$STATIC_CALLBACKS = var0;
    }

    private static final void CGLIB$BIND_CALLBACKS(Object var0) {
        UserServiceImpl$$EnhancerByCGLIB$$14f99790 var1 = (UserServiceImpl$$EnhancerByCGLIB$$14f99790)var0;
        if (!var1.CGLIB$BOUND) {
            var1.CGLIB$BOUND = true;
            Object var10000 = CGLIB$THREAD_CALLBACKS.get();
            if (var10000 == null) {
                var10000 = CGLIB$STATIC_CALLBACKS;
                if (var10000 == null) {
                    return;
                }
            }

            var1.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])var10000)[0];
        }

    }

    public Object newInstance(Callback[] var1) {
        CGLIB$SET_THREAD_CALLBACKS(var1);
        UserServiceImpl$$EnhancerByCGLIB$$14f99790 var10000 = new UserServiceImpl$$EnhancerByCGLIB$$14f99790();
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
        return var10000;
    }

    public Object newInstance(Callback var1) {
        CGLIB$SET_THREAD_CALLBACKS(new Callback[]{var1});
        UserServiceImpl$$EnhancerByCGLIB$$14f99790 var10000 = new UserServiceImpl$$EnhancerByCGLIB$$14f99790();
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
        return var10000;
    }

    public Object newInstance(Class[] var1, Object[] var2, Callback[] var3) {
        CGLIB$SET_THREAD_CALLBACKS(var3);
        UserServiceImpl$$EnhancerByCGLIB$$14f99790 var10000 = new UserServiceImpl$$EnhancerByCGLIB$$14f99790;
        switch(var1.length) {
        case 0:
            var10000.<init>();
            CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
            return var10000;
        default:
            throw new IllegalArgumentException("Constructor not found");
        }
    }

    public Callback getCallback(int var1) {
        CGLIB$BIND_CALLBACKS(this);
        MethodInterceptor var10000;
        switch(var1) {
        case 0:
            var10000 = this.CGLIB$CALLBACK_0;
            break;
        default:
            var10000 = null;
        }

        return var10000;
    }

    public void setCallback(int var1, Callback var2) {
        switch(var1) {
        case 0:
            this.CGLIB$CALLBACK_0 = (MethodInterceptor)var2;
        default:
        }
    }

    public Callback[] getCallbacks() {
        CGLIB$BIND_CALLBACKS(this);
        return new Callback[]{this.CGLIB$CALLBACK_0};
    }

    public void setCallbacks(Callback[] var1) {
        this.CGLIB$CALLBACK_0 = (MethodInterceptor)var1[0];
    }

    static {
        CGLIB$STATICHOOK1();
    }
}

```

从代理对象反编译源码可以知道，代理对象继承于UserServiceImpl，拦截器调用intercept()方法，

intercept()方法由自定义CglibCacheHandler实现，所以，最后调用CglibCacheHandler中

的intercept()方法，从而完成了由代理对象访问到目标对象的动态代理实现。

### 总结

jdk动态代理就是：

1. 代理对象是在程序运行时产生的，而不是编译期；
2. 对代理对象的所有接口方法调用都会转发到InvocationHandler.invoke()方法，在invoke()方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；之后我们通过某种方式执行真正的方法体，示例中通过反射调用了Hello对象的相应方法，还可以通过RPC调用远程方法。
3. 对于从Object中继承的方法，JDK Proxy会把hashCode()、equals()、toString()这三个非接口方法转发给InvocationHandler，其余的Object方法则不会转发。
4. 通过继承接口来对实现类进行增强

cglib代理

1. JDK代理要求被代理的类必须实现接口，有很强的局限性。而CGLIB动态代理则没有此类强制性要求。简单的说，CGLIB会让生成的代理类继承被代理类，并在代理类中对代理方法进行强化处理(前置处理、后置处理等)。但是如果被代理类被final修饰，那么它不可被继承，即不可被代理；同样，如果被代理类中存在final修饰的方法，那么该方法也不可被代理。
2. CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理，但是无法处理final的情况。
3. 通过在编译期生成ASM字节码，进行代理类的生成

