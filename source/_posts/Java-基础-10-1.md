---
title: Java JDK各版本新特性
date: 2019-04-06 22:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
Java语法糖系列一：可变长度参数和foreach循环
 [http://www.jianshu.com/p/628568f94ef8](https://www.jianshu.com/p/628568f94ef8)

Java语法糖系列二：自动装箱/拆箱和条件编译
 [http://www.jianshu.com/p/946b3c4a5db6](https://www.jianshu.com/p/946b3c4a5db6)

Java语法糖系列三：泛型与类型擦除
 [http://www.jianshu.com/p/4de08deb6ba4](https://www.jianshu.com/p/4de08deb6ba4)

Java语法糖系列四：枚举类型
 [http://www.jianshu.com/p/ae09363fe734](https://www.jianshu.com/p/ae09363fe734)

Java语法糖系列五：内部类和闭包
 [http://www.jianshu.com/p/f55b11a4cec2](https://www.jianshu.com/p/f55b11a4cec2)

### Java 5 特性

从JDK 1.5开始，Jav的开发就出现了许多新特性，有部分的新特性已经使用了，只不过随着不同版本的提升，新特性也越来越多。

Java5开发代号为Tiger(老虎),于2004-09-30发行。
<!--more-->

#### 可变参数

如果现在要求设计一个方法，这个方法可以接收任意多个整型数据的相加。那么最早的设计就是利用数组来解决问题。

```java
//范例：最初的解决方案
public class TestDemo {

	public static void main(String[] args) {
		System.out.println(add(new int[]{1,2,3}));//传递三个整形数据
		System.out.println(add(new int[]{12,34}));//传递两个整形数据
	}

	/**
	 * 实现任意多个整型数据的相加操作
	 *
	 * @param data 由于要接收多个整型数据，所以使用数组完成接收
	 * @return 多个整形数据的累加结果
	 */
	public static int add(int[] data) {
		int sum = 0;
		for(int x=0;x<data.length;x++){
			sum+=data[x];
		}
		return sum;
	}
}
```
以上的代码之所以使用数组，是因为多个参数方法上无法描述，所以利用数组整合成多个参数，但是严格来讲这样的实现并不标准。要求是可以接收任意多个整型数据：

理想的调用形式：add(1,2,3)、add(10,20)、add(10,10,10,10,10）；

这一功能从JDK 1.5之后正式的登陆到了Java之中，它主要是在方法上使用，其定义形式：

```java
[public | protected | private ] [static] [final] [abstract] 返回值类型 方法名称（参数类型 ... 变量）{
           [return [返回值];]
}
```

此时给出的参数不再是一个内容，而是多个内容，但是尽管参数的定义形式变了，可是参数的访问却没有改变，也就是说在进行参数访问的时候，按照数组的形式操作。

```java
public class TestDemo {

 public static void main(String[] args) {
   // 可变参数支持接收数组
   System.out.println(add(new int[] { 1, 2, 3 }));// 传递三个整形数据
   System.out.println(add(new int[] { 12, 34 }));// 传递两个整形数据
   // 或者使用","区分不同的参数，接收的时候还是数组
   System.out.println(add(1, 2, 3));
   System.out.println(add(1, 3));
   System.out.println(add(2, 3));
   System.out.println(add());
 }

 /**
  * 实现任意多个整型数据的相加操作
  *
  * @param data
  *            由于要接收多个整型数据，所以使用数组完成接收
  * @return 多个整形数据的累加结果
  */
 public static int add(int... data) {
   int sum = 0;
   for (int x = 0; x < data.length; x++) {
     sum += data[x];
   }
   return sum;
 }
}
```
在大部分的开发情况下，应该要求参数的个数是准确的，所以对于这样的开发往往不会用于应用型的开发上，可能会用于一些程序相关系统类的设计使用上。

总结：在设计一个类的时候可变参数绝对不是优先的选择；可变参数就属于数组的变形应用。

#### for each循环

增强型for循环的使用。foreach输出是由C#最早引入的概念。其目的就是进行数组或是集合数据的输出。在最早如果要进行数组输出肯定使用for循环，而后利用下标进行数据的访问。

```java
//范例：传统的做法
public class TestDemo {

	public static void main(String[] args) {
		int data[] = new int[] { 1, 2, 3, 4, 5 };
		for (int x = 0; x < data.length; x++) {
			System.out.println(data[x]);
		}
	}
}
```
有人会认为以上的输出需要使用索引会比较麻烦。

for/in循环办不到的事情：

- 遍历同时获取index
- 集合逗号拼接时去掉最后一个
- 遍历的同时删除元素

从JDK1.5之后增加的foreach循环形式就可以取消掉索引的操作形式。语法如下：

```java
for(类型 变量 ： 数组 | 集合){
//每一次循环会自动的将数组的内容设置给变量
}
```
范例：观察增强型的for循环

```java
public class TestDemo {

	public static void main(String[] args) {
		int data[] = new int[] { 1, 2, 3, 4, 5 };
		for (int x : data) {// 循环次数由数组长度决定
			// 每一次循环实际上都表示数组的脚标增长，会取得每一个数组的内容，并且将其设置给x
			System.out.println(x);// x就是每一个数组元素的内容
		}
	}
}
```
用数组直接通过索引访问会比较麻烦。而有了这样的形式代码就避免了索引的麻烦。foreach循环支持数组的直接访问，避免了索引访问带来的麻烦。

#### 静态导入

如果说现在某一个类中定义的方法全部都属于static型的方法，那么其它类要引用此类时必须使用“类名称 . 方法()”进行调用。

```java
//范例：传统的做法
public class MyMath {
	public static int add(int x, int y) {
		return x + y;
	}

	public static int div(int x, int y) {
		return x / y;
	}
}
```
此时MyMath类里面的方法都是static型的方法，随后在其它类使用这些方法。

```java
//范例：基本使用形式
public class TestDemo {
	public static void main(String[] args) {
		System.out.println("加法操作：" + MyMath.add(10, 20));
		System.out.println("除法操作：" + MyMath.div(10, 2));
	}
```

如果在主类中定义的是static方法，那么可以直接调用static方法，而现在的MyMath类里面都是static方法，那么觉得前面加上类名称实在是多余。于是从JDK1.5之后开始增加了静态导入。

```java
//范例：静态导入
//将MyMath类中的全部static方法导入，这些方法就好比在主类中定义的static方法一样
import static MyMath.*;

public class TestDemo {

	public static void main(String[] args) {
		// 直接使用代码名称访问
		System.out.println("加法操作：" + add(10, 20));
		System.out.println("除法操作：" + div(10, 2));
	}
}
```
从道理上来讲，如果在前面增加上了类名称，认为反而能够更加清楚的表示出具体方法属于哪个类。

#### 泛型

所谓类型擦除指的就是Java源码中的范型信息只允许停留在编译前期，而编译后的字节码文件中将不再保留任何的范型信息。也就是说，范型信息在编译时将会被全部删除，其中范型类型的类型参数则会被替换为Object类型，并在实际使用时强制转换为指定的目标数据类型。而C++中的模板则会在编译时将模板类型中的类型参数根据所传递的指定数据类型生成相对应的目标代码。

```java
Map<Integer, Integer> squares = new HashMap<Integer, Integer>();
```
通配符类型：避免unchecked警告，问号表示任何类型都可以接受

```java
public void printList(List<?> list, PrintStream out) throws IOException {
    for (Iterator<?> i = list.iterator(); i.hasNext(); ) {
      out.println(i.next().toString());
    }
  }
```
限制类型
```java
public static <A extends Number> double sum(Box<A> box1,Box<A> box2){
    double total = 0;
    for (Iterator<A> i = box1.contents.iterator(); i.hasNext(); ) {
      total = total + i.next().doubleValue();
    }
    for (Iterator<A> i = box2.contents.iterator(); i.hasNext(); ) {
      total = total + i.next().doubleValue();
    }
    return total;
  }
```

#### 枚举

EnumMap

```java
public void testEnumMap(PrintStream out) throws IOException {
    // Create a map with the key and a String message
    EnumMap<AntStatus, String> antMessages =
      new EnumMap<AntStatus, String>(AntStatus.class);
    // Initialize the map
    antMessages.put(AntStatus.INITIALIZING, "Initializing Ant...");
    antMessages.put(AntStatus.COMPILING,    "Compiling Java classes...");
    antMessages.put(AntStatus.COPYING,      "Copying files...");
    antMessages.put(AntStatus.JARRING,      "JARring up files...");
    antMessages.put(AntStatus.ZIPPING,      "ZIPping up files...");
    antMessages.put(AntStatus.DONE,         "Build complete.");
    antMessages.put(AntStatus.ERROR,        "Error occurred.");
    // Iterate and print messages
    for (AntStatus status : AntStatus.values() ) {
      out.println("For status " + status + ", message is: " +
                  antMessages.get(status));
    }
  }
```

switch枚举

```java
public String getDescription() {
    switch(this) {
      case ROSEWOOD:      return "Rosewood back and sides";
      case MAHOGANY:      return "Mahogany back and sides";
      case ZIRICOTE:      return "Ziricote back and sides";
      case SPRUCE:        return "Sitka Spruce top";
      case CEDAR:         return "Wester Red Cedar top";
      case AB_ROSETTE:    return "Abalone rosette";
      case AB_TOP_BORDER: return "Abalone top border";
      case IL_DIAMONDS:   
        return "Diamonds and squares fretboard inlay";
      case IL_DOTS:
        return "Small dots fretboard inlay";
      default: return "Unknown feature";
    }
  }
```
#### 自动拆箱/装箱

将primitive类型转换成对应的wrapper类型：Boolean、Byte、Short、Character、Integer、Long、Float、Double

#### 注解

Inherited表示该注解是否对类的子类继承的方法等起作用

```java
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface InProgress { }
```
Target类型

Rentation表示annotation是否保留在编译过的class文件中还是在运行时可读。

#### print输出格式化
```java
System.out.println("Line %d: %s%n", i++, line);
```

#### 并发支持（JUC）

- 线程池

- uncaught exception（可以抓住多线程内的异常）

```java
class SimpleThreadExceptionHandler implements
    Thread.UncaughtExceptionHandler {
  public void uncaughtException(Thread t, Throwable e) {
    System.err.printf("%s: %s at line %d of %s%n",
        t.getName(),
        e.toString(),
        e.getStackTrace()[0].getLineNumber(),
        e.getStackTrace()[0].getFileName());
  }
```

- blocking queue(BlockingQueue)

- JUC类库

#### Arrays、Queue、线程安全StringBuilder


Arrays工具类

```java
Arrays.sort(myArray);
Arrays.toString(myArray)
Arrays.binarySearch(myArray, 98)
Arrays.deepToString(ticTacToe)
Arrays.deepEquals(ticTacToe, ticTacToe3)
```
Queue

避开集合的add/remove操作，使用offer、poll操作（不抛异常）


JDK5是java史上最重要的升级之一，具有非常重要的意义，虽然语法糖非常多。但可以使得我们的代码更加健壮，更加优雅。

### Java 6 特性
Java6开发代号为Mustang(野马),于2006-12-11发行.

评价：鸡肋的版本，有JDBC4.0更新、Complier API、WebSevice支持的加强等更新。

#### Web Services

优先支持编写 XML web service 客户端程序。你可以用过简单的annotaion将你的API发布成.NET交互的web services. Mustang 添加了新的解析和 XML 在 Java object-mapping APIs中, 之前只在Java EE平台实现或者Java Web Services Pack中提供

#### Scripting（开启JS的支持，算是比较有用的）

现在你可以在Java源代码中混入JavaScript了，这对开发原型很有有用，你也可以插入自己的脚本引擎。

#### Database

Mustang 将联合绑定 Java DB (Apache Derby). JDBC 4.0 增加了许多特性例如支持XML作为SQL数据类型，更好的集成Binary Large OBjects (BLOBs) 和 Character Large OBjects (CLOBs) .

#### More Desktop APIs

GUI 开发者可以有更多的技巧来使用 SwingWorker utility ，以帮助GUI应用中的多线程。, JTable 分类和过滤，以及添加splash闪屏。

很显然，这对于主攻服务器开发的Java来说，并没有太多吸引力

#### Monitoring and Management.

绑定了不是很知名的 memory-heap 分析工具Jhat 来查看内核导出。

#### Compiler Access（这个很厉害）

compiler API提供编程访问javac，可以实现进程内编译，动态产生Java代码。

#### Pluggable Annotation

一部分是进行注解处理的javax.annotation.processing，另一部分是对程序的静态结构进行建模的javax.lang.model

#### Desktop Deployment.

Swing拥有更好的 look-and-feel , LCD 文本呈现, 整体GUI性能的提升。Java应用程序可以和本地平台更好的集成，例如访问平台的系统托盘和开始菜单。Mustang将Java插件技术和Java Web Start引擎统一了起来。

#### Security

XML-数字签名(XML-DSIG) APIs 用于创建和操纵数字签名); 新的方法来访问本地平台的安全服务

#### The -ilities（很好的习惯）

质量，兼容性，稳定性。 80,000 test cases 和数百万行测试代码(只是测试活动中的一个方面). Mustang 的快照发布已经被下载15个月了，每一步中的Bug都被修复了，表现比J2SE 5还要好。

### Java 7 特性

Java7开发代号是Dolphin(海豚),于2011-07-28发行.

评价：不温不火

#### switch中添加对String类型的支持

```java
public String generate(String name, String gender) {  
       String title = "";  
       switch (gender) {  
           case "男":  
               title = name + " 先生";  
               break;  
           case "女":  
               title = name + " 女士";  
               break;  
           default:  
               title = name;  
       }  
       return title;  
}
```
编译器在编译时先做处理：
- case仅仅有一种情况。直接转成if。
- 假设仅仅有一个case和default，则直接转换为if…else…。
- 有多个case。先将String转换为hashCode，然后相应的进行处理，JavaCode在底层兼容Java7曾经版本号。

#### 数字字面量的改进
数字文字绝对是对眼睛的一种考验。我相信，如果你给了一个数字，比如说，十个零，你就会像我一样数零。如果不计算从右到左的位置，识别一个文字的话，就很容易出错，而且很麻烦。Not anymore。

Java7前支持十进制（123）、八进制（0123）、十六进制（0X12AB）。Java7添加二进制表示（0B11110001、0b11110001）

数字中可加入分隔符。Java7中支持在数字量中间添加`_`作为分隔符。更直观，如（12_123_456）。下划线仅仅能在数字中间。编译时编译器自己主动删除数字中的下划线。
```java
int one_million = 1_000_000;
```
请注意，这个版本中也引入了二进制文字-例如“0b1”-因此开发人员不必再将它们转换为十六进制。

#### 异常处理（捕获多个异常） try-with-resources

Java中有一些资源需要手动关闭，例如Connections，Files，Input/OutStreams等。通常我们使用 try-finally 来关闭资源。catch子句能够同一时候捕获多个异常

```java
public void testSequence() {  
    try {  
        Integer.parseInt("Hello");  
    }  
    catch (NumberFormatException | RuntimeException e) {  //使用'|'切割，多个类型，一个对象e  

    }  
}  
```
try-with-resources语句,Java7之前须要在finally中关闭socket、文件、数据库连接等资源；

Java7中在try语句中申请资源，实现资源的自己主动释放（资源类必须实现java.lang.AutoCloseable接口，一般的文件、数据库连接等均已实现该接口，close方法将被自己主动调用）。

```java
public void read(String filename) throws IOException {  

      try (
        BufferedReader reader = new BufferedReader(new FileReader(filename))
        ) {  
          StringBuilder builder = new StringBuilder();  
 String line = null;  
 while((line=reader.readLine())!=null){  
     builder.append(line);  
     builder.append(String.format("%n"));  
 }  
 return builder.toString();  
      }   
  }  
```
#### 增强泛型推断
之前
```java
Map<String, List<String>> map = new HashMap<String, List<String>>();   
```
Java7之后可以简单的这么写
```java
Map<String, List<String>> anagrams = new HashMap<>();
```

#### NIO2.0（AIO）新IO的支持

bytebuffer
```java
public class ByteBufferUsage {
    public void useByteBuffer() {
        ByteBuffer buffer = ByteBuffer.allocate(32);
        buffer.put((byte)1);
        buffer.put(new byte[3]);
        buffer.putChar('A');
        buffer.putFloat(0.0f);
        buffer.putLong(10, 100L);
        System.out.println(buffer.getChar(4));
        System.out.println(buffer.remaining());
    }
    public void byteOrder() {
        ByteBuffer buffer = ByteBuffer.allocate(4);
        buffer.putInt(1);
        buffer.order(ByteOrder.LITTLE_ENDIAN);
        buffer.getInt(0); //值为16777216
    }
    public void compact() {
        ByteBuffer buffer = ByteBuffer.allocate(32);
        buffer.put(new byte[16]);
        buffer.flip();
        buffer.getInt();
        buffer.compact();
        int pos = buffer.position();
    }
    public void viewBuffer() {
        ByteBuffer buffer = ByteBuffer.allocate(32);
        buffer.putInt(1);
        IntBuffer intBuffer = buffer.asIntBuffer();
        intBuffer.put(2);
        int value = buffer.getInt(); //值为2
    }
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        ByteBufferUsage bbu = new ByteBufferUsage();
        bbu.useByteBuffer();
        bbu.byteOrder();
        bbu.compact();
        bbu.viewBuffer();
    }
}
```

filechannel

```java
public class FileChannelUsage {
    public void openAndWrite() throws IOException {
        FileChannel channel = FileChannel.open(Paths.get("my.txt"), StandardOpenOption.CREATE, StandardOpenOption.WRITE);
        ByteBuffer buffer = ByteBuffer.allocate(64);
        buffer.putChar('A').flip();
        channel.write(buffer);
    }
    public void readWriteAbsolute() throws IOException {
        FileChannel channel = FileChannel.open(Paths.get("absolute.txt"), StandardOpenOption.READ, StandardOpenOption.CREATE, StandardOpenOption.WRITE);
        ByteBuffer writeBuffer = ByteBuffer.allocate(4).putChar('A').putChar('B');
        writeBuffer.flip();
        channel.write(writeBuffer, 1024);
        ByteBuffer readBuffer = ByteBuffer.allocate(2);
        channel.read(readBuffer, 1026);
        readBuffer.flip();
        char result = readBuffer.getChar(); //值为'B'
    }
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) throws IOException {
        FileChannelUsage fcu = new FileChannelUsage();
        fcu.openAndWrite();
        fcu.readWriteAbsolute();
    }
}
```
#### JSR292与InvokeDynamic

JSR 292: Supporting Dynamically Typed Languages on the JavaTM Platform，支持在JVM上运行动态类型语言。在字节码层面支持了InvokeDynamic。

方法句柄MethodHandle
```java
public class ThreadPoolManager {
    private final ScheduledExecutorService stpe = Executors
            .newScheduledThreadPool(2);
    private final BlockingQueue<WorkUnit<String>> lbq;
    public ThreadPoolManager(BlockingQueue<WorkUnit<String>> lbq_) {
        lbq = lbq_;
    }
    public ScheduledFuture<?> run(QueueReaderTask msgReader) {
        msgReader.setQueue(lbq);
        return stpe.scheduleAtFixedRate(msgReader, 10, 10, TimeUnit.MILLISECONDS);
    }
    private void cancel(final ScheduledFuture<?> hndl) {
        stpe.schedule(new Runnable() {
            public void run() {
                hndl.cancel(true);
            }
        }, 10, TimeUnit.MILLISECONDS);
    }
    /**
     * 使用传统的反射api
     */
    public Method makeReflective() {
        Method meth = null;
        try {
            Class<?>[] argTypes = new Class[]{ScheduledFuture.class};
            meth = ThreadPoolManager.class.getDeclaredMethod("cancel", argTypes);
            meth.setAccessible(true);
        } catch (IllegalArgumentException | NoSuchMethodException
                | SecurityException e) {
            e.printStackTrace();
        }
        return meth;
    }
    /**
     * 使用代理类
     * @return
     */
    public CancelProxy makeProxy() {
        return new CancelProxy();
    }
    /**
     * 使用Java7的新api，MethodHandle
     * invoke virtual 动态绑定后调用 obj.xxx
     * invoke special 静态绑定后调用 super.xxx
     * @return
     */
    public MethodHandle makeMh() {
        MethodHandle mh;
        MethodType desc = MethodType.methodType(void.class, ScheduledFuture.class);
        try {
            mh = MethodHandles.lookup().findVirtual(ThreadPoolManager.class,
                    "cancel", desc);
        } catch (NoSuchMethodException | IllegalAccessException e) {
            throw (AssertionError) new AssertionError().initCause(e);
        }
        return mh;
    }
    public static class CancelProxy {
        private CancelProxy() {
        }
        public void invoke(ThreadPoolManager mae_, ScheduledFuture<?> hndl_) {
            mae_.cancel(hndl_);
        }
    }
}
```
调用invoke

```java
public class ThreadPoolMain {
    /**
     * 如果被继承，还能在静态上下文寻找正确的class
     */
    private static final Logger logger = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
    private ThreadPoolManager manager;
    public static void main(String[] args) {
        ThreadPoolMain main = new ThreadPoolMain();
        main.run();
    }
    private void cancelUsingReflection(ScheduledFuture<?> hndl) {
        Method meth = manager.makeReflective();
        try {
            System.out.println("With Reflection");
            meth.invoke(hndl);
        } catch (IllegalAccessException | IllegalArgumentException
                | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
    private void cancelUsingProxy(ScheduledFuture<?> hndl) {
        CancelProxy proxy = manager.makeProxy();
        System.out.println("With Proxy");
        proxy.invoke(manager, hndl);
    }
    private void cancelUsingMH(ScheduledFuture<?> hndl) {
        MethodHandle mh = manager.makeMh();
        try {
            System.out.println("With Method Handle");
            mh.invokeExact(manager, hndl);
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
    private void run() {
        BlockingQueue<WorkUnit<String>> lbq = new LinkedBlockingQueue<>();
        manager = new ThreadPoolManager(lbq);
        final QueueReaderTask msgReader = new QueueReaderTask(100) {
            @Override
            public void doAction(String msg_) {
                if (msg_ != null)
                    System.out.println("Msg recvd: " + msg_);
            }
        };
        ScheduledFuture<?> hndl = manager.run(msgReader);
        cancelUsingMH(hndl);
        // cancelUsingProxy(hndl);
        // cancelUsingReflection(hndl);
    }
}
```
#### Path接口(重要接口更新)
Path
```java
public class PathUsage {
    public void usePath() {
        Path path1 = Paths.get("folder1", "sub1");
        Path path2 = Paths.get("folder2", "sub2");
        path1.resolve(path2); //folder1\sub1\folder2\sub2
        path1.resolveSibling(path2); //folder1\folder2\sub2
        path1.relativize(path2); //..\..\folder2\sub2
        path1.subpath(0, 1); //folder1
        path1.startsWith(path2); //false
        path1.endsWith(path2); //false
        Paths.get("folder1/./../folder2/my.text").normalize(); //folder2\my.text
    }
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        PathUsage usage = new PathUsage();
        usage.usePath();
    }
}
```
DirectoryStream

```java
public class ListFile {
    public void listFiles() throws IOException {
        Path path = Paths.get("");
        try (DirectoryStream<Path> stream = Files.newDirectoryStream(path, "*.*")) {
            for (Path entry: stream) {
                //使用entry
                System.out.println(entry);
            }
        }
    }
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) throws IOException {
        ListFile listFile = new ListFile();
        listFile.listFiles();
    }
}
```
Files
```java
public class FilesUtils {
    public void manipulateFiles() throws IOException {
        Path newFile = Files.createFile(Paths.get("new.txt").toAbsolutePath());
        List<String> content = new ArrayList<String>();
        content.add("Hello");
        content.add("World");
        Files.write(newFile, content, Charset.forName("UTF-8"));
        Files.size(newFile);
        byte[] bytes = Files.readAllBytes(newFile);
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        Files.copy(newFile, output);
        Files.delete(newFile);
    }
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) throws IOException {
        FilesUtils fu = new FilesUtils();
        fu.manipulateFiles();
    }
}
```

WatchService
```java
public class WatchAndCalculate {
    public void calculate() throws IOException, InterruptedException {
        WatchService service = FileSystems.getDefault().newWatchService();
        Path path = Paths.get("").toAbsolutePath();
        path.register(service, StandardWatchEventKinds.ENTRY_CREATE);
        while (true) {
            WatchKey key = service.take();
            for (WatchEvent<?> event : key.pollEvents()) {
                Path createdPath = (Path) event.context();
                createdPath = path.resolve(createdPath);
                long size = Files.size(createdPath);
                System.out.println(createdPath + " ==> " + size);
            }
            key.reset();
        }
    }
    /**
     * @param args the command line arguments
     */
    public static void main(String[] args) throws Throwable {
        WatchAndCalculate wc = new WatchAndCalculate();
        wc.calculate();
    }
}
```
#### fork/join计算框架

Java7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

### Java 8 特性

你真的开始用JDK8了吗？ 如果你没有用上一些新特性，请别说自己使用了Java8。

Java 8可谓是自Java 5以来最具革命性的版本了，她在语言、编译器、类库、开发工具以及Java虚拟机等方面都带来了不少新特性。我们来一一回顾一下这些特性。

#### Lambda表达式

Lambda表达式可以说是Java 8最大的卖点，她将函数式编程引入了Java。Lambda允许把函数作为一个方法的参数，或者把代码看成数据。

一个Lambda表达式可以由用逗号分隔的参数列表、–>符号与函数体三部分表示。例如：
```java
Arrays.asList( "p", "k", "u","f", "o", "r","k").forEach( e -> System.out.println( e ) );
```

为了使现有函数更好的支持Lambda表达式，Java 8引入了函数式接口的概念。函数式接口就是只有一个方法的普通接口。java.lang.Runnable与java.util.concurrent.Callable是函数式接口最典型的例子。为此，Java 8增加了一种特殊的注解@FunctionalInterface

#### 接口的默认方法与静态方法

我们可以在接口中定义默认方法，使用default关键字，并提供默认的实现。所有实现这个接口的类都会接受默认方法的实现，除非子类提供的自己的实现。例如：

```java
 public interface DefaultFunctionInterface {
     default String defaultFunction() {
         return "default function";
     }
 }
```

我们还可以在接口中定义静态方法，使用static关键字，也可以提供实现。例如：
```java
 public interface StaticFunctionInterface {
     static String staticFunction() {
         return "static function";
     }
 }
```

接口的默认方法和静态方法的引入，其实可以认为引入了C＋＋中抽象类的理念，以后我们再也不用在每个实现类中都写重复的代码了。

#### 方法引用（含构造方法引用）

通常与Lambda表达式联合使用，可以直接引用已有Java类或对象的方法。一般有四种不同的方法引用：

1. 构造器引用。语法是Class::new，或者更一般的Class< T >::new，可无参，可有参数。方法签名保持一致；
2. 静态方法引用。语法是Class::static_method，要求方法签名保持一致；
3. 特定类的任意对象方法引用。它的语法是Class::method。要求方法签名保持一致；
4. 特定对象的方法引用，它的语法是instance::method。要求方法签名保持一致。与3不同的地方在于，3是在列表元素上分别调用方法，而4是在某个对象上调用方法，将列表元素作为参数传入；

#### 重复注解

在Java 5中使用注解有一个限制，即相同的注解在同一位置只能声明一次。Java 8引入重复注解，这样相同的注解在同一地方也可以声明多次。重复注解机制本身需要用@Repeatable注解。Java 8在编译器层做了优化，相同注解会以集合的方式保存，因此底层的原理并没有变化。

#### 扩展注解的支持（类型注解）

Java 8扩展了注解的上下文，几乎可以为任何东西添加注解，包括局部变量、泛型类、父类与接口的实现，连方法的异常也能添加注解。
```java
private @NotNull String name;
```
#### Optional

Java 8引入Optional类来防止空指针异常，Optional类最先是由Google的Guava项目引入的。Optional类实际上是个容器：它可以保存类型T的值，或者保存null。使用Optional类我们就不用显式进行空指针检查了。
#### Stream

Stream API是把真正的函数式编程风格引入到Java中。其实简单来说可以把Stream理解为MapReduce，当然Google的MapReduce的灵感也是来自函数式编程。她其实是一连串支持连续、并行聚集操作的元素。从语法上看，也很像linux的管道、或者链式编程，代码写起来简洁明了，非常酷帅！
#### Date/Time API (JSR 310)

Java 8新的Date-Time API (JSR 310)受Joda-Time的影响，提供了新的java.time包，可以用来替代 java.util.Date和java.util.Calendar。一般会用到Clock、LocaleDate、LocalTime、LocaleDateTime、ZonedDateTime、Duration这些类，对于时间日期的改进还是非常不错的。
#### JavaScript引擎Nashorn

Nashorn允许在JVM上开发运行JavaScript应用，允许Java与JavaScript相互调用。
#### Base64

在Java 8中，Base64编码成为了Java类库的标准。Base64类同时还提供了对URL、MIME友好的编码器与解码器。

除了这十大新特性之外，还有另外的一些新特性：

**更好的类型推测机制：** Java 8在类型推测方面有了很大的提高，这就使代码更整洁，不需要太多的强制类型转换了。

**编译器优化：** Java 8将方法的参数名加入了字节码中，这样在运行时通过反射就能获取到参数名，只需要在编译时使用-parameters参数。

**并行（parallel）数组：** 支持对数组进行并行处理，主要是parallelSort()方法，它可以在多核机器上极大提高数组排序的速度。

**并发（Concurrency）：** 在新增Stream机制与Lambda的基础之上，加入了一些新方法来支持聚集操作。

**Nashorn引擎jjs：** 基于Nashorn引擎的命令行工具。它接受一些JavaScript源代码为参数，并且执行这些源代码。

**类依赖分析器jdeps：** 可以显示Java类的包级别或类级别的依赖。

### Java 9 特性

java 9 提供了超过 150 项新功能特性，包括备受期待的模块化系统、可交互的 REPL 工具：jshell，JDK 编译工具，Java 公共 API 和私有代码，以及安全增强、扩展提升、性能管理改善等。可以说 Java 9 是一个庞大的系统工程，完全做了一个整体改变。但本博文只介绍最重要的十大新特性

#### 平台级modularity（原名：Jigsaw） 模块化系统

模块化系统Java7开始筹备，Java8进行了大量工作，Java9才落地。首先带来最直观的感受，就是目录结构的感受：

java 7：

![2.png](https://i.loli.net/2019/04/14/5cb2be3e6b7cb.png)

java 9：

![1.png](https://i.loli.net/2019/04/14/5cb2bdc3e7f6e.png)

 对目录做相应的介绍：

![1.png](https://i.loli.net/2019/04/14/5cb2be792c774.png)

Java 9 的定义功能是一套全新的模块系统。当代码库越来越大，创建复杂，盘根错节的 **“意大利面条式代码”的几率呈指数级的增长** 。这时候就得面对两个基础的问题: 很难真正地对代码进行封装, 而系统并没有对不同部分（也就是 JAR 文件）之间的依赖关系有个明确的概念。每一个公共类都可以被类路径之下任何其它的公共类所访问到, 这样就会导致无意中使用了并不想被公开访问的 API。此外，类路径本身也存在问题: 你怎么知晓所有需要的 JAR 都已经有了, 或者是不是会有重复的项呢? 模块系统把这俩个问题都给解决了。

在模块的 src 下创建 module-info.java 文件，来描述依赖和导出（暴露）。
```
requires：指明对其它模块的依赖。
exports：控制着哪些包可以被其它模块访问到。所有不被导出的包默认都被封装在模块里面。
```

#### Java 的 REPL 工具： jShell 命令

REPL：read - evaluate - print - loop
这个简单的说就是能想脚本语言那样，所见即所得。之前我们用java，哪怕只想输出一句hello world，都是非常麻烦的。需要建文件、写代码、编译、运行等等。现在有了jShell工具，实在太方便了

 - 即写即得、快速运行
 - jShell 也可以从文件中加载语句或者将语句保存到文件中（使用Open命令）
 - jShell 也可以是 tab 键进行自动补全和自动添加分号
 - 列出当前 session 里所有有效的代码片段：/list

#### 多版本兼容 jar 包（这个在处理向下兼容方面，非常好用）

当一个新版本的 Java 出现的时候，你的库用户要花费数年时间才会切换到这个新的版本。这就意味着库得去向后兼容你想要支持的最老的 Java 版本（许多情况下就是 Java 6 或者 Java7）。这实际上意味着未来的很长一段时间，你都不能在库中运用 Java 9 所提供的新特性。

幸运的是，多版本兼容 jar 功能能让你创建仅在特定版本的 Java 环境中运行库程序选择使用的 class 版本

#### 语法改进：接口的私有方法

在 Java 9 中，接口更加的灵活和强大，连方法的访问权限修饰符
都可以声明为 private 的了，此时方法将不会成为你对外暴露的 API
的一部分（个人认为，这肯定是JDK8遗漏了的一个点，哈哈）

```java
public static String staticFun() {
       privateFun();
       return "";
   }

   default String defaultFun() {
       privateFun();
       return "";
   }

   private static void privateFun() {
       System.out.println("我是私有方法~");
   }
```
这样子是没有问题，可以正常调用和使用的。但是需要注意一下两点

1. 私有方法可以是static，也可以不是。看你需要default方法调用还是static方法调用
2. 私有方法只能用private修饰，不能用protected。若不写，默认就是public，就是普通静态方法了。

```java
default String defaultFun() {
        privateFun();
        return "";
    }

    private void privateFun() {
        System.out.println("我是私有方法~");
    }
```

#### 语法改进:钻石操作符(Diamond Operator)使用升级 泛型
在 java 8 中如下的操作是会报错的：
```java
public static void main(String[] args) {
       Set<String> set1 = new HashSet<>(); //最常用的初始化
       //Set<String> set2 = new HashSet<>(){}; //在JDK8中报错
       Set<String> set2 = new HashSet<String>(){}; //这样在JDK8中也正常
       Set<String> set3 = new HashSet<String>(){{}}; //这样也都是正常的
   }
```
报错的那种情况是因为在JDK8中，还不能直接推断出钻石操作符里面的类型而报错。而我们在JDK9以后，就可以直接这么写了：

```java
public static void main(String[] args) {
        Set<String> set1 = new HashSet<>(); //最常用的初始化
        Set<String> set2 = new HashSet<>(){}; //在JDK8中报错
        Set<String> set3 = new HashSet<>(){{}}; //这样也都是正常的
    }
```
这样写都是不会报错，可以直接书写使用的。相当于直接创建了一个HashMap的子类。

#### 语法改进：UnderScore(下划线)使用的限制

这个点非常的小。距离说明就懂了。在Java8中，我们给变量取名直接就是_

```java
public static void main(String[] args) {
       String _ = "hello";
       System.out.println(_); //hello
    }
```
我们很清晰的看到，Java8其实给出了提示，但是编译运行都是能通过的，而到了Java9， 直接就提示_是关键字，编译都过不了了。

#### 底层结构：String 存储结构变更（这个很重要）

UTF-8表示一个字符是个动态的过程，可以能用1、2、3个字节都是有可能的。但是UTF-16明确的就是不管你是拉丁文、中文等，都是恒定的用两个字节表示

JDK8的字符串存储在char类型的数组里面，不难想象在绝大多数情况下，char类型只需要一个字节就能表示出来了，比如各种字母什么的，两个字节存储势必会浪费空间，JDK9的一个优化就在这，内存的优化。

Java8:

```java
private final char value[];
```
Java9:

```java
private final byte[] value;
```

结论：String 再也不用 char[] 来存储啦，改成了 byte[] 加上编码标
记，节约了不少空间。由于底层用了字节数组byte[]来存储，所以遇上非拉丁文，JDK9配合了一个encodingFlag来配合编码解码的

so，相应的StringBuffer 和 StringBuilder 也对应的做出了对应的变化。

有的人担心，这会不会影响到我的charAt方法呢？那我们来看看：

```java
public static void main(String[] args) {
        String str = "hello";
        String china = "方世享";
        System.out.println(str.charAt(1)); //e
        System.out.println(china.charAt(1)); //世
    }
```
显然，这个对上层的调用者是完全透明的，完全是底层的数据结构存储而已。但是有必要对比一下源码，还是有非常大的区别的：
java8的charAt方法源码： 实现起来简单很多吧

```java
public char charAt(int index) {
       if ((index < 0) || (index >= value.length)) {
           throw new StringIndexOutOfBoundsException(index);
       }
       return value[index];
   }
```
java9的charAt方法源码：
```java
public char charAt(int index) {
        if (isLatin1()) {
            return StringLatin1.charAt(value, index);
        } else {
            return StringUTF16.charAt(value, index);
        }
    }
```

#### 集合工厂方法：快速创建只读集合

为了保证数据的安全性，有时候我们需要创建一个只读的List。在JDK8的时候，我们只能这么做：
```java
Collections.unmodifiableList(list)
Collections.unmodifiableSet(set)
Collections.unmodifiableMap(map)
```
>Tips：Arrays.asList(1,2,3)创建的List也是只读的，不能添加删除,但是一般我们并不会把他当作只读来用。

可以说是比较繁琐的一件事。Java 9 因此引入了方便的方法，这使得类似的事情更容易表达。调用集合中静态方法 of()，可以将不同数量的参数传输到此工厂方法。此功能可用于 Set 和 List，也可用于 Map 的类似形式。此时得到
的集合，是不可变的：

```java
List<String> list = List.of("a", "b", "c");
        Set<String> set = Set.of("a", "b", "c");
        //Map的两种初始化方式，个人喜欢第二种，语意更加清晰些，也不容易错
        Map<String, Integer> map1 = Map.of("Tom", 12, "Jerry", 21,
                "Lilei", 33, "HanMeimei", 18);
        Map<String, Integer> map2 = Map.ofEntries(
                Map.entry("Tom", 89),
                Map.entry("Jim", 78),
                Map.entry("Tim", 98)
        );
```
处于好奇心，可以让大家再对比一下类型，看看怎么实现的：
```java
public static void main(String[] args) {
        List<String> list = List.of("a", "b", "c");
        List<String> listOld = Collections.unmodifiableList(Arrays.asList("a", "b", "c"));
        System.out.println(list.getClass().getName()); //java.util.ImmutableCollections$ListN
        System.out.println(listOld.getClass().getName()); //java.util.Collections$UnmodifiableRandomAccessList
    }
```

#### 增强的 Stream API

在 Java 9 中，Stream API 变得更好，Stream 接口中添加了 4 个新的方法：dropWhile, takeWhile, ofNullable，还有个 iterate 方法的新重载方法，可以让你提供一个 Predicate (判断条件)来指定什么时候结束迭代。

除了对 Stream 本身的扩展，Optional 和 Stream 之间的结合也
得到了改进。现在可以通过 Optional 的新方法 stream() 将一个
Optional 对象转换为一个(可能是空的) Stream 对象
```
takeWhile()：返回从开头开始的尽量多的元素
dropWhile() ：行为与 takeWhile 相反，返回剩余的元素
ofNullable()：Stream 不能全为 null，否则会报空指针异常。而 Java 9 中的ofNullable健壮性就比of强很多。可以包含一个非空元素，也可以创建一个空 Stream
iterator()重载方法。如下，相当于不仅仅是limit，而是可以写逻辑来判断终止与否了
```
ofNullable()
```java
//报 NullPointerException   因为Of方法不允许全为null的
//Stream<Object> stream1 = Stream.of(null);
//System.out.println(stream1.count());

//ofNullable()：允许值为 null
Stream<Object> stream1 = Stream.ofNullable(null);
System.out.println(stream1.count());//0
```
#### 全新的 HTTP 客户端 API

HTTP，用于传输网页的协议，早在 1997 年就被采用在目前的 1.1
版本中。直到 2015 年，HTTP2 才成为标准。

Java 9 中有新的方式来处理 HTTP 调用。它提供了一个新的 HTTP客户端（ HttpClient ）， 它 将 替代仅适用于 blocking 模式的HttpURLConnection （HttpURLConnection是在HTTP 1.0的时代创建的，并使用了协议无关的方法），并提供对 WebSocket 和 HTTP/2 的支持。

此外，HTTP 客户端还提供 API 来处理 HTTP/2 的特性，比如流和
服务器推送等功能。全新的 HTTP 客户端 API 可以从 jdk.incubator.httpclient 模块中获取。因为在默认情况下，这个模块是不能根据 classpath 获取的，需要使用 add modules 命令选项配置这个模块，将这个模块添加到 classpath中。

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder(URI.create("http://www.baidu.com")).GET().build();
HttpResponse<String> response = client.send(req,
HttpResponse.BodyHandler.asString());
System.out.println(response.statusCode());
System.out.println(response.version().name());
System.out.println(response.body());
```
#### 其它特性

**Deprecated 废弃了相关 API**

Java 9 废弃或者移除了几个不常用的功能。其中最主要的是Applet API，现在是标记为废弃的。随着对安全要求的提高，主流浏览器已经取消对 Java 浏览器插件的支持

**智能 Java 编译工具**

智能 java 编译工具( sjavac )的第一个阶段始于 JEP139 这个项目，用于在多核处理器情况下提升 JDK 的编译速度

JDK 9 还更新了 javac 编译器以便能够将 java 9 代码编译运行在低版本 Java 中

**统一的 JVM 日志系统**

**javadoc 的 HTML 5 支持**

Nashorn 项目在 JDK 9 中得到改进，它为 Java 提供轻量级的Javascript 运行时。JDK 9 包含一个用来解析 Nashorn 的 ECMAScript 语法树的API。这个 API 使得 IDE 和服务端框架不需要依赖 Nashorn 项目的内部实现类，就能够分析 ECMAScript 代码。Javascript 引擎升级：Nashorn（该引擎在8中首次引入，非常好用）

**java 的动态编译器**

**JIT（Just-in-time）** 编译器可以在运行时将热点编译成本地代码，
速度很快。但是 Java 项目现在变得很大很复杂，因此 JIT 编译器需
要花费较长时间才能热身完，而且有些 Java 方法还没法编译，性能
方面也会下降。AoT 编译就是为了解决这些问题而生的


Java9有一个重大的变化，就是垃圾回收器默认采用了G1。

Java 9 移除了在 Java 8 中 被废弃的垃圾回收器配置组合，同时把G1设为默认的垃圾回收器实现。替代了之前默认使用的Parallel GC，对于这个改变，evens的评论是酱紫的：这项变更是很重要的，因为相对于Parallel来说，G1会在应用线程上做更多的事情，而Parallel几乎没有在应用线程上做任何事情，它基本上完全依赖GC线程完成所有的内存管理。这意味着切换到G1将会为应用线程带来额外的工作，从而直接影响到应用的性能

CMS收集器与G1收集器的区别，参考：[CMS收集器与G1收集器](https://blog.csdn.net/u011546953/article/details/78994882)


### Java 10 特性

JDK10的升级幅度其实主要还是以优化为主，并没有带来太多对使用者惊喜的特性。

#### 局部变量的类型推断 var关键字

这个新功能将为Java增加一些语法糖 - 简化它并改善开发者体验。新的语法将减少与编写Java相关的冗长度，同时保持对静态类型安全性的承诺。

这可能是Java10给开发者带来的最大的一个新特性。下面主要看例子：
```java
public static void main(String[] args) {
      var list = new ArrayList<String>();
      list.add("hello，world！");
      System.out.println(list);
  }
```
这是最平常的使用。注意赋值语句右边，最好写上泛型类型，否则会有如下情况：
```java
public static void main(String[] args) {
       var list = new ArrayList<>();
       list.add("hello，world！");
       list.add(1);
       list.add(1.01);
       System.out.println(list);
   }
```
list什么都可以装，非常的不安全了。和js等语言不同的是，毕竟Java还是强类型的语言，所以下面语句是编译报错的：

```java
public static void main(String[] args) {
        var list = new ArrayList<String>();
        list.add("hello，world！");
        System.out.println(list);

        list = new ArrayList<Integer>(); //编译报错
    }
```
**注意：注意：注意：** 下面几点使用限制

1. 局部变量初始化
2. for循环内部索引变量
3. 传统的for循环声明变量

```java
public static void main(String[] args) {
        //局部变量初始化
        var list = new ArrayList<String>();
        //for循环内部索引变量
        for (var s : list) {
            System.out.println(s);
        }
        //传统的for循环声明变量
        for (var i = 0; i < list.size(); i++) {
            System.out.println(i);
        }
    }
```
下面这几种情况，都是不能使用var的：方法参数、全局变量、构造函数参数、方法返回类型、字段、捕获表达式（或任何其他类型的变量声明）

#### GC改进和内存管理 并行全垃圾回收器 G1

JDK 10中有2个JEP专门用于改进当前的垃圾收集元素。
Java 10的第二个JEP是针对G1的并行完全GC（JEP 307），其重点在于通过完全GC并行来改善G1最坏情况的等待时间。G1是Java 9中的默认GC，并且此JEP的目标是使G1平行。

#### 垃圾回收器接口

这不是让开发者用来控制垃圾回收的接口；而是一个在 JVM 源代码中的允许另外的垃圾回收器快速方便的集成的接口。

#### 线程-局部变量管控

这是在 JVM 内部相当低级别的更改，现在将允许在不运行全局虚拟机安全点的情况下实现线程回调。这将使得停止单个线程变得可能和便宜，而不是只能启用或停止所有线程。

#### 合并 JDK 多个代码仓库到一个单独的储存库中

在 JDK9 中，有 8 个仓库： root、corba、hotspot、jaxp、jaxws、jdk、langtools 和 nashorn 。在 JDK10 中这些将被合并为一个，使得跨相互依赖的变更集的存储库运行 atomic commit （原子提交）成为可能。
#### 新增API：ByteArrayOutputStream

String toString(Charset): 重载 toString()，通过使用指定的字符集解码字节，将缓冲区的内容转换为字符串。
#### 新增API：List、Map、Set

这3个接口都增加了一个新的静态方法，copyOf(Collection）。这些函数按照其迭代顺序返回一个不可修改的列表、映射或包含给定集合的元素的集合。
#### 新增API：java.util.Properties

增加了一个新的构造函数，它接受一个 int 参数。这将创建一个没有默认值的空属性列表，并且指定初始大小以容纳指定的元素数量，而无需动态调整大小。还有一个新的重载的 replace 方法，接受三个 Object 参数并返回一个布尔值。只有在当前映射到指定值时，才会替换指定键的条目。
#### 新增API： Collectors收集器
```java
toUnmodifiableList():
toUnmodifiableSet():
toUnmodifiableMap(Function, Function):
toUnmodifiableMap(Function, Function, BinaryOperator):
```
这四个新方法都返回 Collectors ，将输入元素聚集到适当的不可修改的集合中。
#### 其它特性

线程本地握手（JEP 312）、其他Unicode语言 - 标记扩展（JEP 314）、基于Java的实验性JIT编译器、根证书颁发认证（CA）、删除工具javah（JEP 313）

从JDK中移除了javah工具，这个很简单并且很重要。

### Java 11 特性

Java11 带来了 ZGC、Http Client 等重要特性，一共包含 17 个 JEP（JDK Enhancement Proposals，JDK 增强提案）。

JDK 更新很重要吗？答：非常重要

```
    最新的安全更新，如，安全协议等基础设施的升级和维护，安全漏洞的及时修补，这是 Java 成为企业核心设施的基础之一。安全专家清楚，即使开发后台服务，而不是前端可直接接触，编程语言的安全性仍然是重中之重。
    大量的新特性、Bug 修复，例如，容器环境支持，GC 等基础领域的增强。很多生产开发中的 Hack，其实升级 JDK 就能解决了。
    不断改进的 JVM，提供接近零成本的性能优化
    …
    “Easy is cheap”? Java 的进步虽然“容易”获得，但莫忽略其价值，这得益于厂商和 OpenJDK 社区背后的默默付出。
```

作为发布计划的一部分，某些版本会被指定为长期支持版本(LTS)，它们会获得四年或更长时间的技术支持和安全补丁。所以这些版本通常会被称为“主要版本” —— 不是因为它们拥有更多的功能特性，而是因为它们具有长期的技术支持。

预计 Java 11 的更新补丁(11.0.1, 11.0.2, 11.0.3 等)将比 Java 8 的补丁(8u20, 8u40, 8u60)更小更简单。因为 Java 11 的更新将更加集中在安全补丁上，不会像 Java 8 的更新那样带来内部的功能增强。因为 Oracle 希望将 Java 12, 13, 14 等这些版本当做是小更新版本，类比成 Java 8 的话，即是 Java 11u20, 11u40。

Oracle 高级员工一再认为像 8u20 和 8u40 这样的更新常常会带来破坏性的变更，但本文作者表示这不是自己的经历，他记得的唯一有破坏性的变化是为 Javadoc 添加了 --allow-script-in-comments，但它也不是 Java 的核心部分。因此，他从不担心升级到最新版本带来的影响 —— 因为这是 Java 平台的核心优势。

下面深入了解一下为什么在旧的发布模式下，升级版本不会导致任何问题。先看一下新旧发布模式之间的差异：

![2(1).png](https://i.loli.net/2019/04/14/5cb2d146d8191.png)

Oracle 的官方观点认为：与 Java 7->8->9 相比，Java 9->10->11的升级和 8->8u20->8u40 更相似。

表格清楚地显示新模式下的 Java 版本发布都会包含许多变更，包括语言变更和 JVM 变更，这两者都会对 IDE、字节码库和框架产生重大影响。此外，不仅会新增其他 API，还会有 API 被删除（这在 Java 8 之前没有发生过）。

Oracle 的观点是，因为每个版本仅在前一个版本发布后的 6 个月推出，所以不会有太多新的“东西”，因此升级并不困难。虽然如此，但这不是重点。重要的是升级是否有可能会破坏代码。很明显，从 11 -> 12 -> 13 开始，代码遭受破坏的可能性要大于 8 -> 8u20 -> 8u40。

11 -> 12 -> 13 与 8u20 -> 8u40 等这样的更新主要区别在于对字节码版本的更改以及对规范的更改，对字节码版本的更改往往特别具有破坏性，大多数框架都大量使用与每个字节码版本密切相关的 ASM 或 ByteBuddy 等库。而 8u20 -> 8u40 仍然使用相同的 Java SE 规范，具有所有相同的类和方法，不同于从 Java 12 移动到 13。

除此之外，Oracle 的另一个声明也十分值得我们关注。声明透露出的消息是，如果坚持使用 Java 11 并计划在下一个 LTS 版本(即 Java 17)发布时再进行升级，开发者可能会发现自己的项目代码无法通过编译。所以请记住，Java 新的开发规则现在声明可以在一个版本中弃用某个 API 方法，并在下一个版本中删除它。

#### 采用新版本 Java 的注意事项

在本节中，将概述在采用新版本 Java 之前必须考虑的一些注意事项/风险。

**被新版本系列“绑定”**

如果采用了 Java 12 并使用新的语言特性或新的 API，这意味着实际上你已将项目绑定到 Java 的新版本系列。接下来你必须采用 Java 13, 14, 15, 16 和 17，并且必须在下一个版本发布后的一个月内采用每个新版本。

使用了新版本，每个版本的使用寿命为六个月，并且在发布后仅七个月就过时了。这是因为每个版本只有在六个月内提供安全补丁，发布后1个月的第一个补丁和发布后4个月的第二个补丁。7个月后，下一组安全补丁会发布，但旧版本不能获取更新。

因此，你要判断自身的开发流程是否允许升级 Java 版本，时间窗口方面会不会太狭窄？

**升级的“绊脚石”**

实际使用中有很多阻止我们升级 Java 的因素，下面列出一些常见的：

- 开发资源不足：你的团队可能会非常忙碌或规模太小，你能保证两年后从 Java 15 升级到 16 的开发时间吗？

- 构建工具和 IDE：你使用的 IDE 是否会在发布当天支持每个新版本？Maven? Gradle 呢? 如果不是，你有后备计划吗？请记住，你只有1个月的时间来完成升级、测试并将其发布到生产环境中。此外还包括 Checkstyle，JaCoCo，PMD，SpotBugs 等等其他工具。

- 依赖关系：你的依赖关系是否都准备好用于每个新版本？请记住，它不仅仅是直接依赖项，而是技术堆栈中的所有内容。字节码操作库尤其受到影响，例如 ByteBuddy 和 ASM。

- 框架：这是另一种依赖，但是一个大而重要的依赖。在一个月的狭窄时间窗口内，Spring 会每六个月发布一个新版本吗？ Jakarta EE(以前的 Java EE)会吗？如果它们不这样做会怎么样？

- 云 / 托管 / 部署

你是否可以控制代码在生产环境中的运行位置和方式？例如，如果你在 AWS Lambda 中运行代码，则无法控制。AWS Lambda 没有采用 Java 9或10，甚至没有采用 Java 11。所以除非 AWS 提供公共保证以支持每个新的 Java 版本，否则根本无法采用 Java 12。

如何托管你的 CI 系统？Jenkins, Travis, Circle, Shippable, GitLab 会快速更新吗？如果不是，你会怎么做？

**对未来的预测**

如果已经阅读了上面的列表，并且你的代码和流程可以应对。这十分好，但更重要的是要明白，你也在限制未来进行改变的能力。例如，你的代码可能今天不在 AWS Lambda 上运行，但未来三年呢？

说了这么多，作者当然不是鼓励大家不进行升级，新语言特性带来的好处以及性能增强会让开发者受益，但升级背后的风险也应该考虑进去。

**其他第三方产商的声明**

Spring 框架已经在视频中表达了对 Java 12 的策略。关键部分是：

>“Java 8 和 11 作为 LTS 版本会持续获得我们的正式支持，对于过渡版本，我们也会尽最大努力支持。如果你升级到 Java 11，我们非常愿意和你合作，但它们不会获得正式的生产环境支持。因为长期支持版本才是我们关注的重心，对于 Java 12 及更高版本我们会尽最大的努力。”

参考：

1. [Java11的新特性](https://www.jianshu.com/p/6e2438e4bac8)
2. [我该用 Java 12 还是坚持 Java 11？](https://blog.csdn.net/csdnnews/article/details/83753246)

### Java SE


jdk 主要包含了 java development tools 和 jre。

jre 主要包含了 javaSE 核心类库 和 jvm。

下面就是 javase 核心类库介绍，Java SE中包含的主要技术如下：
Deployment Tecknologies: 和部署相关的技术

#### Deployment、Java Web Start、 Java Plug-in

(1) Java Web Start：允许用户通过一次单击操作下载并启动特性完整的应用程序(比如电子表格)，而不需要进行安装，从而简化了Java应用程序的部署。
User Interface ToolKits：用户接口工具套件

#### AWT、Swing、Java 2D、Accessibility、Drag’n Drop、Input Methods、Image IO、Print Service、Sound

(1) Java Foundation Classes(Swing)(JFC)是一套Java类库，支持为基于Java的客户机应用程序构建GUI(Graphical User Interface，图形用户界面)和图形化功能。

(2) Java 2D API是一套用于高级2D图形和图像的类(为图像组合和alpha通道图像提供丰富的支持)，一套提供精确的颜色空间定义和转换的类以及一套面向显示的图像操作符。

#### Integration Libraries、Other Base Libraries：集成库

IDL、JDBC、JNDI、RMI、RMI-IIOP

(1) Java Database Connectivity(JDBC)是一个API，它使用户能够从Java代码中访问大多数表格式数据源，提供了对许多SQL数据库的跨DBMS连接能力，并可以访问其他表格式数据源，比如电子表格或平面文件。

(2) Java Naming and Directory Interface(JNDI)为Java应用程序提供一个连接到企业中的多个命名和目录服务的统一接口，可以无缝地连接结构不同的企业命名和目录服务。

#### lang & util Base Libraries：语言和通用的基础库
Beans、Int‘l Support、IO、New IO、JMX、JNI、Math、Networking、Std
Override Mechanism、Security、Serialization、Extension Mechanism、XML JAXP、Lang&Util、Collections、Concurrency Utilities、JAR、Logging、management、Preferences、Ref
Objects、Reflection、Regular Expressions、Versioning、Zip

(1) Java Beans Component Architecture是一个为Java平台定义可重用软件组件的框架，可以在图形化构建工具中设计这些组件。
(2) Java Native Interface(JNI)是JVM中运行的Java代码，可以与用其他编程语言编写的应用程序和库进行互操作。
(3) Java Help是一个独立于平台的可扩展的帮助系统，开发人员可以使用它将在线帮助集成到Applet、组件、应用程序、操作系统和设备中，还可以提供基于Web的在线文档。

(4) Java API for XML Processing(JAXP)允许Java应用程序独立于特定的XML处理，实现对XML文档进行解析和转换，允许灵活地在XML处理程序之间进行切换，而不需要修改应用程序代码。Java API for XML Binding(JAXB)允许在XML文档和Java对象之间进行自动的映射。

(5) Concurrency Utilities是一套中级实用程序，提供了并发程序中常用的功能。

(6) Java Platform Debugger Architecture(JPDA)是用于Java SE调试支持的基础结构。
(8) Certification Path API提供了一套用于创建、构建和检验认证路径(也称为"认证链")的API，可以安全地建立公共密钥到主体的映射。
(10) Java Advanced Imaging(JAI)是一个API，提供了一套面向对象的接口，这些接口支持一个简单的高级编程模型，使开发人员能够轻松地操作图像。
(11) Java Authentication and Authorization Service(JAAS)是一个包，实现了标准的Pluggable Authentication Module(PAM)框架的Java版本并支持基于用户的授权，能够对用户进行身份验证和访问控制。
(12) Java Cryptography Extension(JCE)是一组包，提供了用于加密、密钥生成和协商以及Message Authentication Code(MAC)算法的框架和实现。JCE给对称、不对称、块和流密码提供加密支持，它还支持安全流和密封的对象。
(13) Java Data Objects(JDO)是一种基于标准接口的持久化Java模型抽象，使程序员能够将Java领域模型实例直接保存到数据库(持久化存储器)中，这可以替代直接文件 I/O、串行化、JDBC以及EJB、BMP(Bean Managed Persistence)或CMP(Container Managed Persistence)实体Bean等方法。
(14) Java Management Extensions(JME)提供了用于构建分布式、基于Web、模块化且动态的应用程序的工具，这些应用程序可以用来管理和监视设备、应用程序和服务驱动的网络。
(15) Java Media Framework(JMF)可以将音频、视频和其他基于时间的媒体添加到Java应用程序和Applet中。
(17) Java Secure Socket Extensions(JSSE)是一组包，它们支持安全的互联网通信，实现了SSL(Secure Sockets Layer)和TLS(Transport Layer Security)的Java版本，包含了数据加密、服务器身份验证、消息完整性和可选的客户机身份验证等功能。
(18) Java Speech API(JSAPI)包含Java Speech Grammar Format(JSGF)和Java Speech Markup Language(JSML)规范，使Java应用程序能够将语音技术集成到用户界面中。JSAPI定义了一个跨平台的 API，支持命令和控制识别器、听写系统及语音识别器。
(19) Java 3D 是一个 API，它提供了一套面向对象的接口，这些接口支持一个简单的高级编程模型，开发人员可以使用这个API轻松地将可伸缩的独立于平台的3D图形集成到 Java应用程序中。
(20) Metadata Facility 允许给类、接口、字段和方法标上特定的属性，从而使开发工具、部署工具和运行时能够以特殊方式处理它们。
(21) Java Content Repository API 是一个用于访问Java SE中独立于实现的内容存储库的 API。内容存储库是一个高级信息管理系统，是传统数据存储库的超集。
(22) Enumerations(枚举)是一种类型，允许以类型安全的方式将特定的数据表示为常量。
(23) Generics(泛型)允许定义具有抽象类型参数的类，可以在实例化时指定这些参数。
(26) SOAP with Attachments API for Java(SAAJ)使开发人员能够按照SOAP1.1规范和 SOAP with Attachments note生成和消费消息。


### 总结

说了那么多特性，其实我们该选择使用那些版本的JDK呢？

其实可以这么说，长期支持版优于升级版，所以说可以选择Java 8和Java 11

**但是，Java 8在2019年开始不再更新商业版，所以说希望各位尽快升级到java 11，也可以选择OpenJDK。**

官方公告：

>Oracle will not post further updates of Java SE 8 to its public download sites for commercial use after January 2019. Customers who need continued access to critical bug fixes and security fixes as well as general maintenance for Java SE 8 or previous versions can get long term support through Oracle Java SE Advanced, Oracle Java SE Advanced Desktop, or Oracle Java SE Suite. For more information, and details on how to receive longer term support for Oracle JDK 8, please see the Oracle
