---
title: 注解框架--lombok
date: 2019-05-15 12:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

Project Lombok是一个java库，可以自动插入编辑器并构建工具，为您的java增添色彩。 永远不要再写另一个getter或equals方法，使用一个注释，您的类具有一个功能齐全的构建器，自动化您的日志记录变量等等。

**不要滥用**

[github项目](https://github.com/rzwitserloot/lombok)

[官网](https://projectlombok.org/)

<!--more-->

Lombok能以简单的注解形式来简化java代码，提高开发人员的开发效率。例如开发中经常需要写的javabean，都需要花时间去添加相应的getter/setter，也许还要去写构造器、equals等方法，而且需要维护，当属性多时会出现大量的getter/setter方法，这些显得很冗长也没有太多技术含量，一旦修改属性，就容易出现忘记修改对应方法的失误。

Lombok能通过注解的方式，在编译时自动为属性生成构造器、getter/setter、equals、hashcode、toString方法。出现的神奇就是在源码中没有getter和setter方法，但是在编译生成的字节码文件中有getter和setter方法。这样就省去了手动重建这些代码的麻烦，使代码看起来更简洁些。

Lombok的使用跟引用jar包一样，可以在官网（<https://projectlombok.org/download>）下载jar包，也可以使用maven添加依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.8</version>
    <scope>provided</scope>
</dependency>
```

使用lombok的好处是:

1. 减少大量的模板代码,get和set方法,从代码封装维度看,将大量的模板代码进行封装,不需要其他人员来不断编写,哪怕是IDE可以生成的代码,这也是重复代码,减少重复的出现;
2. 从代码可读性角度来看,可以专注于查看类的属性,尤其编写代码的风格不一致,比如为了防止代码冲突,新增加的代码都在最后下面编写,之前总会看到类似的问题,新增加的属性和set和get方法写道后面,让其他人无法专注查看类本身的关键内容;
3. 第三减少代码冲突的可能,尤其增加新属性的时候,哪怕处理冲突也非常简单的,大量的bean,model,vo中减少非常多的代码维护;
4. lombok处理的toString方法,hash,equal方法等内容,当增加新属性的时候,以上方法都不需要重新编写,而是lombok直接帮助处理的,不会出现遗漏的问题和情况.就好比修改增加或修改某个属性,那么你就要处理该属性对其相关内容的变化,现在你修改后,其他内容不需要你去善后,而是直接有人给你善后一样,这一点尤其在新增加属性的时候非常容易忘记处理.

使用lombok的不足:

1. 要求IDE增加对lombok的支持,比如IDEA中需要安装lombok的插件;如果你对外提供的服务使用lombok,那么可能引用jar的其他项目需要增加对lombok的支持,但是并不麻烦.

2. 如果你想确认某个set或get方法是否在程序中被调用,你无法找到哪里使用的,但是我更认为这样的操作是违背了bean使用的初衷,bean尤其数据库和java类的映射bean,java对bean的定义和使用就是无参数的构造方法和set和get方法,而不应该在bean中处理任何和业务有任何关系的逻辑.

### 使用

需要在IDE上添加相应版本的插件，请从官网下载适合自己IDE的插件。

常用注释：

```java
@Data注解在类上，自动为所有字段添加@ToString，@EqualsAndHashCode，@Getter为非final字段添加@Setter和@RequiredArgsConstructor本质上相当于几个注解的综合效果
@Getter注解在属性（类）上，为属性（所有非静态成员变量）提供get()方法
@Setter注解在属性（类）上，为属性（所有非静态成员变量）提供set()方法
@ToString 该注解的作用是为类自动生成toString()方法
@AllArgsConstructor，@RequiredArgsConstructor，@NoArgsConstructor顾名思义，为类自动生成对应参数的构造器
@Log，@Log4j，@Log4j2，@Slf4j，@XSlf4j，@CommonsLog，@JBossLog注解在类上，自动为类添加对应的日志支持
@NonNull注解在方法参数上，用于自动生成空值参数检查，自动帮助我们避免空指针
@Cleanup自动帮我们调用close()方法，作用在局部变量上，在作用域结束时会自动调用close()方法释放资源，可以关闭流
@Builder注解在类上，被注解的类加个构造者模式
@Synchronized 注解在类上，加个同步锁
@SneakyThrows等同于try/catch捕获异常
@NonNull : 如果给参数加个这个注解 参数为null会抛出空指针异常
@Value : 注解和@Data类似，区别在于它会把所有成员变量默认定义为private final修饰，并且不会生成set方法。
@toString：注解在类上；为类提供toString方法（可以添加排除和依赖）；
@EqualsAndHashCode注解：在JavaBean或类JavaBean中使用，使用此注解会自动重写对应的equals方法和hashCode方法；
```

官方文档<https://projectlombok.org/features/index.html>

在使用以上注解需要处理参数时，处理方法如下（以@ToString注解为例，其他注解同@ToString注解）：

```
@ToString(exclude="column")

意义：排除column列所对应的元素，即在生成toString方法时不包含column参数；

@ToString(exclude={"column1","column2"})

意义：排除多个column列所对应的元素，其中间用英文状态下的逗号进行分割，即在生成toString方法时不包含多个column参数；

@ToString(of="column")

意义：只生成包含column列所对应的元素的参数的toString方法，即在生成toString方法时只包含column参数；

@ToString(of={"column1","column2"})

意义：只生成包含多个column列所对应的元素的参数的toString方法，其中间用英文状态下的逗号进行分割，即在生成toString方法时只包含多个column参数
```

#### @Data

@Data注解在类上，会为类的所有属性自动生成setter/getter、equals、canEqual、hashCode、toString方法，如为final属性，则不会为该属性生成setter方法。

```java
import lombok.AccessLevel;
import lombok.Setter;
import lombok.Data;
import lombok.ToString;

@Data public class DataExample {
  private final String name;
  @Setter(AccessLevel.PACKAGE) private int age;
  private double score;
  private String[] tags;
  
  @ToString(includeFieldNames=true)
  @Data(staticConstructor="of")
  public static class Exercise<T> {
    private final String name;
    private final T value;
  }
}
```

如不使用Lombok，则实现如下：

```java
import java.util.Arrays;

public class DataExample {
  private final String name;
  private int age;
  private double score;
  private String[] tags;
  
  public DataExample(String name) {
    this.name = name;
  }
  
  public String getName() {
    return this.name;
  }
  
  void setAge(int age) {
    this.age = age;
  }
  
  public int getAge() {
    return this.age;
  }
  
  public void setScore(double score) {
    this.score = score;
  }
  
  public double getScore() {
    return this.score;
  }
  
  public String[] getTags() {
    return this.tags;
  }
  
  public void setTags(String[] tags) {
    this.tags = tags;
  }
  
  @Override public String toString() {
    return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
  }
  
  protected boolean canEqual(Object other) {
    return other instanceof DataExample;
  }
  
  @Override public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof DataExample)) return false;
    DataExample other = (DataExample) o;
    if (!other.canEqual((Object)this)) return false;
    if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
    if (this.getAge() != other.getAge()) return false;
    if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
    if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
    return true;
  }
  
  @Override public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final long temp1 = Double.doubleToLongBits(this.getScore());
    result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
    result = (result*PRIME) + this.getAge();
    result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
    result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
    return result;
  }
  
  public static class Exercise<T> {
    private final String name;
    private final T value;
    
    private Exercise(String name, T value) {
      this.name = name;
      this.value = value;
    }
    
    public static <T> Exercise<T> of(String name, T value) {
      return new Exercise<T>(name, value);
    }
    
    public String getName() {
      return this.name;
    }
    
    public T getValue() {
      return this.value;
    }
    
    @Override public String toString() {
      return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
    }
    
    protected boolean canEqual(Object other) {
      return other instanceof Exercise;
    }
    
    @Override public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof Exercise)) return false;
      Exercise<?> other = (Exercise<?>) o;
      if (!other.canEqual((Object)this)) return false;
      if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
      if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
      return true;
    }
    
    @Override public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
      result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
      return result;
    }
  }
}
```

#### @Getter/@Setter

如果觉得@Data太过残暴（因为@Data集合了@ToString、@EqualsAndHashCode、@Getter/@Setter、@RequiredArgsConstructor的所有特性）不够精细，可以使用@Getter/@Setter注解，此注解在属性上，可以为相应的属性自动生成Getter/Setter方法

**属性**

- onMethod：在方法上添加中注解
- onParam：在方法的参数上添加注解
- value：访问权限修饰符

```
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {

  @Getter @Setter private int age = 10;
  
  @Setter(AccessLevel.PROTECTED) private String name;
  
  @Override public String toString() {
    return String.format("%s (age: %d)", name, age);
  }
}
```

#### @NonNull

该注解用在属性或构造器上，Lombok会生成一个非空的声明，可用于校验参数，能帮助避免空指针。

```
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

#### @Cleanup

该注解能帮助我们自动调用close()方法，很大的简化了代码。

```
import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup InputStream in = new FileInputStream(args[0]);
    @Cleanup OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}
```

#### @EqualsAndHashCode

默认情况下，会使用所有非静态（non-static）和非瞬态（non-transient）属性来生成equals和hasCode，也能通过exclude注解来排除一些属性。

**参数**

- callSuper：是否调用父类的hashCode()，默认：false
- doNotUseGetters：是否不调用字段的getter，默认如果有getter会调用。设置为true，直接访问字段，不调用getter
- exclude：此处列出的任何字段都不会在生成的equals和hashCode中使用。
- of：与exclude相反，设置of，exclude失效
- onParam：添加注解，参考@Getter#onMethod

```
import lombok.EqualsAndHashCode;

@EqualsAndHashCode(exclude={"id", "shape"})
public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.name;
  }
  
  @EqualsAndHashCode(callSuper=true)
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

#### @ToString

类使用@ToString注解，Lombok会生成一个toString()方法，默认情况下，会输出类名、所有属性（会按照属性定义顺序），用逗号来分割。

通过将`includeFieldNames`参数设为true，就能明确的输出toString()属性。这一点是不是有点绕口，通过代码来看会更清晰些。

```
import lombok.ToString;

@ToString(exclude="id")
public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;
  
  public String getName() {
    return this.getName();
  }
  
  @ToString(callSuper=true, includeFieldNames=true)
  public static class Square extends Shape {
    private final int width, height;
    
    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

#### @NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor

无参构造器、部分参数构造器、全参构造器。Lombok没法实现多种参数构造器的重载。

**NoArgsConstructor参数**

- access：访问权限修饰符
- force：为true时，强制生成构造器，final字段初始化为null
- onConstructor：添加注解，参考@Getter#onMethod

```
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample<T> {
  private int x, y;
  @NonNull private T description;
  
  @NoArgsConstructor
  public static class NoArgsExample {
    @NonNull private String field;
  }
}
```

#### @Slf4j

```
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
/**
 *   日志测试
 */


@Slf4j
public class LoggerTest {

    private  final Logger logger = LoggerFactory.getLogger(LoggerTest.class);
    /**
     * 一、传统方式实现日志
     */
    @Test
    public  void test1(){
        logger.debug("debug message");
        logger.warn("warn message");
        logger.info("info message");
        logger.error("error message");
        logger.trace("trace message");
    }

    /**
     * 二、注解方式实现日志
     */
    @Test
   public  void test2(){
        log.debug("debug message");
        log.warn("warn message");
        log.info("info message");
        log.error("error message");
        log.trace("trace message");
    }

}
```

#### @Builder

生成构建者(Builder)模式

示例：

```
@Builder
public class Example {

    private int foo;
    private final String bar;
}
```

生成：

```
public class Example {
    private int foo;
    private final String bar;

    Example(int foo, String bar) {
        this.foo = foo;
        this.bar = bar;
    }

    public static Example.ExampleBuilder builder() {
        return new Example.ExampleBuilder();
    }

    public static class ExampleBuilder {
        private int foo;
        private String bar;

        ExampleBuilder() {
        }

        public Example.ExampleBuilder foo(int foo) {
            this.foo = foo;
            return this;
        }

        public Example.ExampleBuilder bar(String bar) {
            this.bar = bar;
            return this;
        }

        public Example build() {
            return new Example(this.foo, this.bar);
        }

        public String toString() {
            return "Example.ExampleBuilder(foo=" + this.foo + ", bar=" + this.bar + ")";
        }
    }
}
```



#### @Singular

这个注解和@Builder一起使用，为Builder生成字段是集合类型的add方法，字段名不能是单数形式，否则需要指定value值

```
@Builder
public class Example {

    @Singular
    @Setter
    private List<Integer> foos;
}
```

生成

```
public class Example {
    private List<Integer> foos;

    Example(List<Integer> foos) {
        this.foos = foos;
    }

    public static Example.ExampleBuilder builder() {
        return new Example.ExampleBuilder();
    }

    public void setFoos(List<Integer> foos) {
        this.foos = foos;
    }

    public static class ExampleBuilder {
        private ArrayList<Integer> foos;

        ExampleBuilder() {
        }
        
        // 这方法是@Singular作用生成的
        public Example.ExampleBuilder foo(Integer foo) {
            if (this.foos == null) {
                this.foos = new ArrayList();
            }

            this.foos.add(foo);
            return this;
        }

        public Example.ExampleBuilder foos(Collection<? extends Integer> foos) {
            if (this.foos == null) {
                this.foos = new ArrayList();
            }

            this.foos.addAll(foos);
            return this;
        }

        public Example.ExampleBuilder clearFoos() {
            if (this.foos != null) {
                this.foos.clear();
            }

            return this;
        }

        public Example build() {
            List foos;
            switch(this.foos == null ? 0 : this.foos.size()) {
            case 0:
                foos = Collections.emptyList();
                break;
            case 1:
                foos = Collections.singletonList(this.foos.get(0));
                break;
            default:
                foos = Collections.unmodifiableList(new ArrayList(this.foos));
            }

            return new Example(foos);
        }

        public String toString() {
            return "Example.ExampleBuilder(foos=" + this.foos + ")";
        }
    }
}
```

#### @SneakyThrows

用try{}catch{}捕捉异常

示例

```
public class Example {

    @SneakyThrows(UnsupportedEncodingException.class)
    public String utf8ToString(byte[] bytes) {
        return new String(bytes, "UTF-8");
    }
}
```

生成后：

```
public class Example {
    public Example() {
    }

    public String utf8ToString(byte[] bytes) {
        try {
            return new String(bytes, "UTF-8");
        } catch (UnsupportedEncodingException var3) {
            throw var3;
        }
    }
}
```

#### @Synchronized

生成Synchronized(){}包围代码

示例：

```
public class Example {

    @Synchronized
    public String utf8ToString(byte[] bytes) {
        return new String(bytes, Charset.defaultCharset());
    }
}
```

生成后：

```
public class Example {
    private final Object $lock = new Object[0];

    public Example() {
    }

    public String utf8ToString(byte[] bytes) {
        Object var2 = this.$lock;
        synchronized(this.$lock) {
            return new String(bytes, Charset.defaultCharset());
        }
    }
}
```

#### @val

变量声明类型推断

```
public class ValExample {
    public String example() {
        val example = new ArrayList<String>();
        example.add("Hello, World!");
        val foo = example.get(0);
        return foo.toLowerCase();
    }

    public void example2() {
        val map = new HashMap<Integer, String>();
        map.put(0, "zero");
        map.put(5, "five");
        for (val entry : map.entrySet()) {
            System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
        }
    }
}
```

生成后：

```
public class ValExample {
    public ValExample() {
    }

    public String example() {
        ArrayList<String> example = new ArrayList();
        example.add("Hello, World!");
        String foo = (String)example.get(0);
        return foo.toLowerCase();
    }

    public void example2() {
        HashMap<Integer, String> map = new HashMap();
        map.put(0, "zero");
        map.put(5, "five");
        Iterator var2 = map.entrySet().iterator();

        while(var2.hasNext()) {
            Entry<Integer, String> entry = (Entry)var2.next();
            System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
        }

    }
}
```

#### @Value

把类声明为final，并添加toString()、hashCode()等方法，相当于 `@Getter @FieldDefaults(makeFinal=true, level=AccessLevel.PRIVATE) @AllArgsConstructor @ToString @EqualsAndHashCode`.

```
@Value
public class Example {

    private Integer foo;
}
```

生成后：

```
public final class Example {
    private final Integer foo;

    public Example(Integer foo) {
        this.foo = foo;
    }

    public Integer getFoo() {
        return this.foo;
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Example)) {
            return false;
        } else {
            Example other = (Example)o;
            Object this$foo = this.getFoo();
            Object other$foo = other.getFoo();
            if (this$foo == null) {
                if (other$foo != null) {
                    return false;
                }
            } else if (!this$foo.equals(other$foo)) {
                return false;
            }

            return true;
        }
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $foo = this.getFoo();
        int result = result * 59 + ($foo == null ? 43 : $foo.hashCode());
        return result;
    }

    public String toString() {
        return "Example(foo=" + this.getFoo() + ")";
    }
}
```

#### @var

和val一样，官方文档中说区别就是var不加final修饰，但测试的效果是一样的

#### @Accessors

默认情况下，没什么作用，需要设置参数

参数

- chain：为true时，setter链式返回，即setter的返回值为this
- fluent：为true时，默认设置chain为true，setter的方法名修改为字段名

### 总结

Lombok本质上就是一个实现了“[JSR 269 API](https://www.jcp.org/en/jsr/detail?id=269)”的程序。在使用javac的过程中，它产生作用的具体流程如下：

1. javac对源代码进行分析，生成了一棵抽象语法树（AST）
2. 运行过程中调用实现了“JSR 269 API”的Lombok程序
3. 此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和setter方法定义的相应树节点
4. javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）

Lombok虽然有很多优点，但Lombok更类似于一种IDE插件，项目也需要依赖相应的jar包。Lombok依赖jar包是因为编译时要用它的注解，为什么说它又类似插件？因为在使用时，eclipse或IntelliJ IDEA都需要安装相应的插件，在编译器编译时通过操作AST（抽象语法树）改变字节码生成，变向的就是说它在改变java语法。它不像spring的依赖注入或者mybatis的ORM一样是运行时的特性，而是编译时的特性。这里我个人最感觉不爽的地方就是对插件的依赖！因为Lombok只是省去了一些人工生成代码的麻烦，但IDE都有快捷键来协助生成getter/setter等方法，也非常方便。

知乎上有位大神发表过对Lombok的一些看法：

```
这是一种低级趣味的插件，不建议使用。JAVA发展到今天，各种插件层出不穷，如何甄别各种插件的优劣？能从架构上优化你的设计的，能提高应用程序性能的 ，
实现高度封装可扩展的...， 像lombok这种，像这种插件，已经不仅仅是插件了，改变了你如何编写源码，事实上，少去了代码你写上去又如何？ 
如果JAVA家族到处充斥这样的东西，那只不过是一坨披着金属颜色的屎，迟早会被其它的语言取代。
```

虽然话糙但理确实不糙，试想一个项目有非常多类似Lombok这样的插件，个人觉得真的会极大的降低阅读源代码的舒适度。

虽然非常不建议在属性的getter/setter写一些业务代码，但在多年项目的实战中，有时通过给getter/setter加一点点业务代码，能极大的简化某些业务场景的代码。所谓取舍，也许就是这时的舍弃一定的规范，取得极大的方便。

我现在非常坚信一条理念，任何编程语言或插件，都仅仅只是工具而已，即使工具再强大也在于用的人，就如同小米加步枪照样能赢飞机大炮的道理一样。结合具体业务场景和项目实际情况，无需一味追求高大上的技术，适合的才是王道。

Lombok有它的得天独厚的优点，也有它避之不及的缺点，熟知其优缺点，在实战中灵活运用才是王道。

### 参考：

1. [Lombok的使用详解](https://blog.csdn.net/f641385712/article/details/82081900)