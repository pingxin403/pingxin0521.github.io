---
title: JSON框架--Gson 基础使用
date: 2019-05-19 13:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

GSON是Google提供的用来在Java对象和JSON数据之间进行映射的Java类库。可以将一个Json字符转成一个Java对象，或者将一个Java转化为Json字符串。

<!--more-->

在开发领域中数据传递有很多形式，通常数据调用交互采用XML，JSON，数据流，纯文本等形式；越来越多数据调用采用JSON，因为JSON数据结构简单，数据字节长度短，既简单又快速何乐而不为呢？

从JSON的结构入手，所有json数据最终分为三种情况：

1. 标量（Scalar)，也就是单纯的字符串或则数字形式
2. 序列（Sequence)，也就是若干数据按照一定顺序并列在一起又称“数组”
3. 映射（Mapping)，也就是key/value键值对
    Json的规格非常简单,此文章就不一一描述

**特点**：

- 快速、高效
- 代码量少、简洁
- 面向对象
- 数据传递和解析方便

**基本概念**

- Serialization:序列化，使Java对象到Json字符串的过程。
- Deserialization：反序列化，字符串转换成Java对象。
- JSON数据中的`JsonElement`有下面这四种类型：
  - JsonPrimitive —— 例如一个字符串或整型
  -  JsonObject—— 一个以 JsonElement 名字（类型为           String）作为索引的集合。也就是说可以把 
  - JsonObject 看作值为 JsonElement 的键值对集合。
  -  JsonArray—— JsonElement 的集合。注意数组的元素可以是四种类型中的任意一种，或者混合类型都支持。
  -  JsonNull—— 值为null

**Gson解决的问题**

1. 提供一种像toString()和构造方法的很简单的机制，来实现Java 对象和Json之间的互相转换。
2. 允许已经存在的无法改变的对象，转换成Json，或者Json转换成已存在的对象。
3. 允许自定义对象的表现形式
4. 支持任意的复杂对象
5. 能够生成可压缩和可读的Json的字符串输

### 基本使用

**知识点**

![1.png](https://i.loli.net/2019/07/13/5d29ac91de61425202.png)

Gson的pom依赖：

```xml
   <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.0</version>
        </dependency>
```

**Gson的创建方式**

方式一：

```java
Gson gson = new gson();
```

方式二：通过GsonBuilder()，可以配置多种配置，**重要**。

```java
Gson gson = new GsonBuilder()
    .registerTypeAdapter(Id.class, new IdTypeAdapter()) //单独对Id类设置了独立解析方式
    .setLenient()// json宽松  
    .enableComplexMapKeySerialization()//支持Map的key为复杂对象的形式  
    .serializeNulls() //智能null  
    .setPrettyPrinting()// 调教格式  
    .setDateFormat(DateFormat.LONG)  //设置时间转换格式
    .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)//会把字段首字母大写
    .disableHtmlEscaping() //默认是GSON把HTML 转义的
    .setVersion(1.0)    
    .create();  
```

GsonBuilder方法解释

- setFieldNamingPolicy 设置序列字段的命名策略(UPPER_CAMEL_CASE,UPPER_CAMEL_CASE_WITH_SPACES,LOWER_CASE_WITH_UNDERSCORES,LOWER_CASE_WITH_DASHES)
- addDeserializationExclusionStrategy 设置反序列化时字段采用策略ExclusionStrategy，如反序列化时不要某字段，当然可以采用@Expore代替。
- excludeFieldsWithoutExposeAnnotation 设置没有@Expore则不序列化和反序列化
- addSerializationExclusionStrategy 设置序列化时字段采用策略，如序列化时不要某字段，当然可以采用@Expore代替。
- registerTypeAdapter 为某特定对象设置固定的序列和反序列方式，实现JsonSerializer和JsonDeserializer接口
- setFieldNamingStrategy 设置字段序列和反序列时名称显示，也可以通过@Serializer代替
- setPrettyPrinting 设置gson转换后的字符串为一个比较好看的字符串
- setDateFormat 设置默认Date解析时对应的format格式

**几个重要点**

1. 推荐把成员变量都声明称private的

2. 没有必要用注解（@Expose 注解）指明某个字段是否会被序列化或者反序列化，所有包含在当前类（包括父类）中的字段都应该默认被序列化或者反序列化

3. 如果某个字段被 transient 这个Java关键词修饰，就不会被序列化或者反序列化

4. 下面的实现方式能够正确的处理null
    1）当序列化的时候，如果对象的某个字段为null，是不会输出到Json字符串中的。
    2）当反序列化的时候，某个字段在Json字符串中找不到对应的值，就会被赋值为null

5. 如果一个字段是 synthetic的,他会被忽视，也即是不应该被序列化或者反序列化

6. 内部类（或者anonymous class（匿名类），或者local class(局部类，可以理解为在方法内部声明的类)）的某个字段和外部类的某个字段一样的话，就会被忽视，不会被序列化或者反序列化

#### Java-JSON的序列化和反序列化

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    private String name;
    private int age;
    private Date birth;
    private boolean isDeveloper;
}
```

**java 对象to JSON字符串**

*注意：*

如果成员变量对象值为空或未赋值，默认不参加转换，基本变量参加转换，若未初始化，则以默认值。

设置serializeNulls()后，则成员变量未赋值时以默认值转换，对象为null，基本数字以0，boolean类型为false，char为`''`。

```java
    @Test
    public void test01() {
        Person person = new Person("平心", 18, new Date(),true);
        Gson gson = new Gson();
        System.out.println(gson.toJson(person));
        System.out.println("---------------");
        Gson gson1 = new GsonBuilder()
                //.registerTypeAdapter(Id.class, new IdTypeAdapter()) //单独对Id类设置了独立解析方式
                .setLenient()// json宽松
                .enableComplexMapKeySerialization()//支持Map的key为复杂对象的形式
                .serializeNulls() //智能null
                .setPrettyPrinting()// 调整格式
                .setDateFormat(DateFormat.SHORT)  //设置时间转换格式
                .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)//会把字段首字母大写
                .disableHtmlEscaping() //默认是GSON把HTML 转义的
                .create();
       System.out.println(gson1.toJson(person));
    }

//结果
//{"name":"平心","age":18,"birth":"Jul 13, 2019, 6:19:04 PM","isDeveloper":true}
//---------------
//{
//    "Name": "平心",
//        "Age": 18,
//        "Birth": "Jul 13, 2019, 6:19:04 PM",
//        "IsDeveloper": true
//}
```

**JSON字符串 to java 对象**

可以这么认为，Gson先将映射类进行初始化，然后将Json字符串转化为类似map的结构，然后通过成员变量字段名或者用户指定字段转化名来进行一一对应，如果可以转化成对应类型变量值，则对对象的字段赋值，不能则不进行赋值。当然Gson的内部实现比这个要更复杂，后面后讲到，这里只是为了更好地理解。

```java
    @Test
    public void Test03()
    {
        Gson gson = new Gson();

        String s1 = "{\"name\":\"平心\",\"birth\":\"Jul 13, 2019, 6:19:04 PM\",\"isDeveloper\":true}";
        Person person1 = gson.fromJson(s1, Person.class);

        System.out.println("---->jsonStr convert javaBean \n" + person1);

        String s2 = "{\"name\":\"平心\",\"age\":18,\"birth\":\"Jul 13, 2019, 6:19:04 PM\",\"isDeveloper\":true}";
        Person person2 = gson.fromJson(s2, Person.class);
        System.out.println("---->jsonStr convert javaBean \n" + person2);

        String s3 = "{}";
        Person person3 = gson.fromJson(s3, Person.class);

        System.out.println("---->jsonStr convert javaBean \n" + person3);
    }
//输出
//---->jsonStr convert javaBean
//Person(name=平心, age=0, birth=Sat Jul 13 18:19:04 CST 2019, isDeveloper=true)
//---->jsonStr convert javaBean
//Person(name=平心, age=18, birth=Sat Jul 13 18:19:04 CST 2019, isDeveloper=true)
//---->jsonStr convert javaBean
//Person(name=null, age=0, birth=null, isDeveloper=false)
```

#### 嵌套对象

**java 对象to JSON字符串**

现在，我们的user还拥有家庭住址，家庭住址有它自己的类型为UserAddress：

```java
public class UserNested {  
    String name;
    String email;
    int age;
    boolean isDeveloper;

    // new, see below!
    UserAddress userAddress;
}

public class UserAddress {  
    String street;
    String houseNumber;
    String city;
    String country;
}
```

由UserNested模型来表示user，在这个模型中添加了一个一对一关系的住址对象。住址由UserAddress模型表示。

在Java中，这两个模型可以以类明确分离开来，然后通过创建UserAddress userAddress成员变量以保持一个引用。但是在Json中我们没有类也没有引用。唯一的方式（除了通过IDs然后将数据合并在一起的方式）就只能是将用户住址嵌入到user对象中了，在JSON中我们在域名后创建了一个用{}包围的新对象：

```json
{
    "age": 26,
    "email": "norman@futurestud.io",
    "isDeveloper": true,
    "name": "Norman",

    "userAddress": {
        "city": "Magdeburg",
        "country": "Germany",
        "houseNumber": "42A",
        "street": "Main Street"
    }
}
```

不像其他的属性（age，email，...）该新userAddress属性没有一个直接的值。相反，它包含一些子值并用{}包裹。理解域名后出现的大括号是非常重要的，这通常表示**这是一个嵌套对象**。

理论已经足够。是时候了解Gson通过一个UserNested对象创建了什么？你将可能认识该模型。Gson不需要任何的设置。它将会通过传入的class类型自动推断相应的数据结构。

```java
UserAddress userAddress = new UserAddress(  
    "Main Street", 
    "42A", 
    "Magdeburg", 
    "Germany"
);

UserNested userObject = new UserNested(  
    "Norman", 
    "norman@futurestud.io", 
    26, 
    true, 
    userAddress
);

Gson gson = new Gson();  
String userWithAddressJson = gson.toJson(userObject); 
```

userWithAddressJson字符串的值是有趣的：

```json
{
    "age": 26,
    "email": "norman@futurestud.io",
    "isDeveloper": true,
    "name": "Norman",

    "userAddress": {
        "city": "Magdeburg",
        "country": "Germany",
        "houseNumber": "42A",
        "street": "Main Street"
    }
}
```

是的，Gson对域按字母重排序了，但结果无疑是我们希望的。Gson正确的创建了包裹着userAddress的JSON对象。当然，我们也可以添加更多的被包裹对象，比如用户的付款方式或者工作地址。同样，被包裹的对象也可以包裹其它对象。

当成员变量未赋值时，仍然符合上面所说的转换方法。

**JSON字符串 to java 对象**

为了不使你感到啰嗦，我们将不适用user的例子了，而适用一个漂亮的小旅馆。

```json
{
  "name": "Future Studio Steak House",
  "owner": {
    "name": "Christian",
    "address": {
      "city": "Magdeburg",
      "country": "Germany",
      "houseNumber": "42A",
      "street": "Main Street"
    }
  },
  "cook": {
    "age": 18,
    "name": "Marcus",
    "salary": 1500
  },
  "waiter": {
    "age": 18,
    "name": "Norman",
    "salary": 1000
  }
}
```

这是调用我们的API所得到的结果，我们希望通过使用Gson自动创建匹配的Java对象。首先，你需要模型化一个基本的类，这个基本类包含了所有顶层的域：

```java
public class Restaurant {  
    String name;

    Owner owner;
    Cook cook;
    Waiter waiter;
}
```

看一下，我们是如何为name定义为字符型，而其他三个是如何定义类的？可能你会得出不同的结果。创建Java对象并不总是明确的。例如，基于该JSON，我们可以发现，cook和waiter的嵌套对象拥有相同的结构。你可以为这它们定义不同的类，就行上面那样，或者你也可以它们定义一个共同的类 —— Staff：

```
public class Restaurant {  
    String name;

    Owner owner;
    Staff cook;
    Staff waiter;
}
```

无论哪一种方式都是有效的。如果你不相信，我们通常倾向创建一个额外的类来避免将来的出现问题。比如，如果cook模型改变了但是waiter模型没有改变，那么你可能需要改变大量的代码。因此，现在我们抛弃Staff的解决方法。当然，我们依然需要为第二层的对象创建Java模型类：

```java
public class Owner {  
    String name;

    UserAddress address;
}

public class Cook {  
    String name;
    int age;
    int salary;
}

public class Waiter {  
    String name;
    int age;
    int salary;
}
```

好了，我们偷了一点懒，复用了开始的UserAddress。但是这仅仅是因为它能够完美的匹配。

尽管如此，我们希望你能够理解根据JSON字符串创建Java模型类的过程。你需要从最高层一直深入到最底层，直到你的嵌套JSON只剩下常规的类型。

我们已经做了主要的工作，可以放心的将接下来的事情交给Gson了。当然，只有我们正确的做了我们该做的，Gson才会只需要很少的代码就能优雅的创建Java对象。

```java
String restaurantJson = "{ 'name':'Future Studio Steak House', 'owner':{ 'name':'Christian', 'address':{ 'city':'Magdeburg', 'country':'Germany', 'houseNumber':'42', 'street':'Main Street'}},'cook':{ 'age':18, 'name': 'Marcus', 'salary': 1500 }, 'waiter':{ 'age':18, 'name': 'Norman', 'salary': 1000}}";

Gson gson = new Gson();

Restaurant restaurantObject = gson.fromJson(restaurantJson, Restaurant.class); 
```

该restaurantObject包含了JSON中的所有信息。

*提示：根据JSONs创建Java模型类是一件非常繁琐的工作。如果你已经意识到这一点那么你肯定希望有工具可以自动的完成这一流程。这里我们推荐[beJson](http://www.bejson.com/json2javapojo/new/)。*

#### Arrays和Lists的映射

**Arrays和Lists之间的不同**

在进入正题之前，我们想阐述一下Arrays和Lists这两种Java数据结构。他们的Java实现是不同的并且各有各的优势。在你的用例中采取哪种方式取决于软件需求以及你个人的喜好。有趣的是，什么是选择list还是array结构映射到JSON是无关紧要的。

在JSON的数据格式中，没有lists和arrays。是的，Java的实现造成了二者之间巨大的区别，但在就高层次来说，他们都是代表相同的列表结构。在接下来的博客中，我们将他们成为对象列表，但是在Java中他们又是不同的。如果你对此感到迷惑，不用担心，一些例子将会使你更加清晰。

**Arrays或者Lists数据的序列化**

这里使用餐馆和菜单。

```java
public class RestaurantWithMenu {  
    String name;

    List<RestaurantMenuItem> menu;
    //RestaurantMenuItem[] menu; // alternative, either one is fine
}

public class RestaurantMenuItem {  
    String description;
    float price;
}
```

Java处理嵌套对象的方式和JSON是不同的。Java可以将之分离到不同的类中，并由List或者Array的来持有其引用。JSON需要保持一个本地的、嵌套的列表。这意味着在高层次，我们希望JSON像下面这样：

```json
{
  "name": "Future Studio Steak House",
  "menu": [
    ...
  ]
}
```

就像嵌套对象一样，menu并不拥有一个直接的值。相反，JSON为其值定义了一个由[]包裹着的对象列表。如上面提到的，这是array还是list是没有区别的。在JSON的数据结构中它看起来是相同的。

menu由很多对象组成。在我们的例子中，它们是饭店菜单项。让我们运行Gson查看一个完整的JSON会是什么样子。

我们希望你现在已经知道常规步骤了。得到你的Java对象，初始化Gson然后让Gson创建相应的JSON：

```java
List<RestaurantMenuItem> menu = new ArrayList<>();  
menu.add(new RestaurantMenuItem("Spaghetti", 7.99f));  
menu.add(new RestaurantMenuItem("Steak", 12.99f));  
menu.add(new RestaurantMenuItem("Salad", 5.99f));

RestaurantWithMenu restaurant =  
        new RestaurantWithMenu("Future Studio Steak House", menu);

Gson gson = new Gson();  
String restaurantJson = gson.toJson(restaurant);
```

restaurantJson包含如下内容：

```json
{
  "menu": [
    {
      "description": "Spaghetti",
      "price": 7.99
    },
    {
      "description": "Steak",
      "price": 12.99
    },
    {
      "description": "Salad",
      "price": 5.99
    }
  ],
  "name": "Future Studio Steak House"
}
```

就跟通常一样，排序有点神奇，列表排在前面是因为按字母来说menu在name的前面。除了排序意外，其他一切都是我们希望的。列表（由[]包裹）包含多个对象（每一个对象由{}包裹）。

然而，我们并不总是发送一个嵌套了列表数据的单独对象，就像上面做的那样。有时候我们也希望发送一个列表。当然，Gson同样支持JSON列表的序列化。例如，如果我们希望序列化如下菜单项列表：

```java
List<RestaurantMenuItem> menu = new ArrayList<>();  
menu.add(new RestaurantMenuItem("Spaghetti", 7.99f));  
menu.add(new RestaurantMenuItem("Steak", 12.99f));  
menu.add(new RestaurantMenuItem("Salad", 5.99f));

Gson gson = new Gson();  
String menuJson = gson.toJson(menu);  
```

将会得到如下结果：

```json
[
  {
    "description": "Spaghetti",
    "price": 7.99
  },
  {
    "description": "Steak",
    "price": 12.99
  },
  {
    "description": "Salad",
    "price": 5.99
  }
]
```

让我们指出重要的不同之处：该JSON的首个字符为[，这就提示了接下来是对象列表！到目前为止，我们只看到由{开始的对象。你应该马上记住它们之间的差异。

**Arrays或者Lists的序列化和反序列化**

JSON数据中作为根列表和嵌套在对象中的列表的重要不同。

1. 列表作为根对象

   让我们做一个练习。考虑这样一种情况，在未来的环境中我们将会开启自己的API，并会提供一个端口GET /founders。该端口将返回三个对象的数组，每个对象包含一个name域和一个flowerCount域。该数组代表我们桌子上的植物。因此如下JSON列表：

   ```
   [
       {
         "name": "Christian",
         "flowerCount": 1
       },
       {
         "name": "Marcus",
         "flowerCount": 3
       },
       {
         "name": "Norman",
         "flowerCount": 2
       }
   ]
   ```

   是的，你是对的。这个想象的端口直接返回了一个列表。JSON以[]开始和结束。没错，Marcus也喜欢在绿植环绕的办公桌上工作。

   因此，我们如何使用Gson将此映射到Java对象呢？第一步是创建模型：

   ```
   public class Founder {  
       String name;
       int flowerCount;
   }
   ```

   第二部取决于你。你是希望使用Lists还是Arrays作为你的类型呢？

2. Arrays

   如果你想要使用Arrays，那太简单了。跟之前一样直接调用fromJson()方法并且传递数组模型的class，就像：gson.fromJson(founderGson, Founder[].class);

   ```java
   String founderJson = "[{'name': 'Christian','flowerCount': 1}, {'name': 'Marcus', 'flowerCount': 3}, {'name': 'Norman', 'flowerCount': 2}]";
   
   Gson gson = new Gson();  
   Founder[] founderArray = gson.fromJson(founderJson, Founder[].class); 
   ```

   这将会产生一个founder的Java数组对象，它们的属性都映射正确

3. Lists
   鉴于Lists可以扩展容量，因此现在的开发者更喜欢使用Java Lists。不幸的是，你不能直接传递List<Founder\>给Gson。为了使Gson知道List的准确结构，你需要得到它的Type。幸运的是，Gson有一个TypeToken类帮助你正确找到任何类的Type。我们的Founder类在一个ArrayList中，让我们看一下：

   ```java
   Type founderListType = new TypeToken<ArrayList<Founder>>(){}.getType(); 
   ```

   你可以使用该语句的结果作为type供Gson调用：

   ```java
   String founderJson = "[{'name': 'Christian','flowerCount': 1}, {'name': 'Marcus', 'flowerCount': 3}, {'name': 'Norman', 'flowerCount': 2}]";
   
   Gson gson = new Gson();
   
   Type founderListType = new TypeToken<ArrayList<Founder>>(){}.getType();
   
   List<Founder> founderList = gson.fromJson(founderJson, founderListType);
   ```

   结果跟使用Array得到的结果差不多,最后，映射你的数据到Array还是List取决于你的个人偏好和用例情况。

4. 列表作为一个对象的一部分

   我们已经扩展了我们想象的未来环境中的API，端口为GET /info。它返回了比founder更多的信息：

   ```json
   {
     "name": "Future Studio Dev Team",
     "website": "https://futurestud.io",
     "founders": [
       {
         "name": "Christian",
         "flowerCount": 1
       },
       {
         "name": "Marcus",
         "flowerCount": 3
       },
       {
         "name": "Norman",
         "flowerCount": 2
       }
     ]
   }
   ```

   我们希望你已经熟悉流程了。首先我们需要写一个与JSON响应匹配的模型。我们可以复用之前的Founder类：

   ```java
   public class GeneralInfo {  
       String name;
       String website;
       List<Founder> founders;
   }
   ```

   处理嵌套在一个对象中的列表是简单的，因为Gson不用使用TypeToken就可以简单地处理了。我们可以直接传入class：

   ```java
   String generalInfoJson = "{'name': 'Future Studio Dev Team', 'website': 'https://futurestud.io', 'founders': [{'name': 'Christian', 'flowerCount': 1 }, {'name': 'Marcus','flowerCount': 3 }, {'name': 'Norman','flowerCount': 2 }]}";
   
   Gson gson = new Gson();
   
   GeneralInfo generalInfoObject = gson.fromJson(generalInfoJson, GeneralInfo.class); 
   ```

   当然，你也可以使用Founder[]数组代替List<Founder\>。Gson同样可以处理。

   *注意：你注意到没有，GeneralInfo和Founder模型拥有相同的name属性，但是Gson却没有出现问题？在序列化和反序列化过程中不会出现任何问题。这真是太神奇了。*

5. 嵌套在列表里面的列表

   如果你正好奇，在处理嵌套在列表里面的列表时会不会有问题。例如，下面的模型将不会有问题：

   ```java
   public class GeneralInfo {  
       String name;
       String website;
       List<FounderWithPets> founders;
   }
   ```

   FounderWithPets类是由宠物列表扩展而来的：

   ```java
   public class FounderWithPets {  
       String name;
       int flowerCount;
       List<Pet> pets;
   }
   ```

   Pet类又拥有玩具的列表

   ```java
   public class Pet {  
       String name;
       List<Toy> toys;
   }
   ```

   Toy类又包含……好了，循环停止吧。我们希望这个推论可以带给你这样一个观点：你可以在列表中逐层的包含列表而不会有任何问题。Gson就像个冠军一样良好的处理序列化和反序列化。

   不过，Gson只能处理包含一致性的对象的列表。如果对象是完全不同的，Gson就不能映射了。尽管可以由多种类型组成列表。

#### Maps的映射

**Java Maps的序列化**

Java maps是一种非常具有弹性数据类型，它可以用于各种各样的场景。它使得我们开发者运用Java程序语言可以实现很多真实世界的场景。因为Java maps的使用范围如此之广，因此这里可能不会和你的用例相同，但方法时适合所有用例的。

让我们从这样一个场景开始，你的App拥有一个雇员姓名列表。你被要求实现这样一个View，它可以显示所有以某个特定字母开头的雇员。例如，用户可以选择字母A，然后你的应用将会返回三个匹配的雇员Andreas，Aden和Arnold。开始的迭代器仅仅是一个可以显示所有名字列表，性能并不良好。因此，我们将采用HashMap代替，它的key是首字母（比如A），它的值时一个名字列表。

我们的创建HashMap的Java代码如下所示：

```java
HashMap<String, List<String>> employees = new HashMap<>();  
employees.put("A", Arrays.asList("Andreas", "Arnold", "Aden"));  
employees.put("C", Arrays.asList("Christian", "Carter"));  
employees.put("M", Arrays.asList("Marcus", "Mary")); 
```

maps的序列化和其他类型是一样的。你仅仅需要把它扔给Gson，Gson将会执行一切：

```java
Gson gson = new Gson();  
String employeeJson = gson.toJson(employees);  
```

返回的JSON结果为：

```json
{
  "M": [
    "Marcus",
    "Mary"
  ],
  "C": [
    "Christian",
    "Carter"
  ],
  "A": [
    "Andreas",
    "Arnold",
    "Aden"
  ]
}
```

每一个键（A，C和M）都拥有一个名字列表，这正是我们想要的。

**反序列化Java Maps**

如果你查看之前的JSON结果，或者查看下面的JSON，你可能会提出这样一个问题：你怎样才能发现这是一个集合，还是很多个对象呢？答案是简单粗暴的：你不能。这就是JSON数据的意思是模棱两可的一个例子。让我们看一下下面这个例子：

```
{
  "1$": {
    "amount": 1,
    "currency": "Dollar"
  },
  "2$": {
    "amount": 2,
    "currency": "Dollar"
  },
  "3€": {
    "amount": 3,
    "currency": "Euro"
  }
}
```

用户可以把该JSON假设为三个对象，每个对象的名为1$, 2$, 3€。每个对象有一些值。但另一方面，也可以想象成是一个简单的Map，其中1$, 2$和3€是键。

没有任何简便方法可以使你估计出JSON的数据类型。一些要点可能能帮到你：

- 首要的是：环境信息！如果你由文档或者知道每个对象的描述信息，那么你可能就知道他们是分离的对象，还是一个map数据了。
- 数据类型是否是一致的？若是则倾向为map。
- 对象的名称或者是键值是否是动态的并且取名范围较广？这也是倾向为map的。

之前的博客中，我们已经向你展示如何映射常规对象，因此在在这篇博客中，我们假设JSON是map数据。那么我们该如何将上面的JSON映射为Java对象呢？

我们使用在列表对象博客中已经提到过的TypeToken方法。通过创建一个新的TypeToken来获得我们所希望的数据类型的type：

```java
public class AmountWithCurrency {  
    String currency;
    int amount;
}

String dollarJson = "{ '1$': { 'amount': 1, 'currency': 'Dollar'}, '2$': { 'amount': 2, 'currency': 'Dollar'}, '3€': { 'amount': 3, 'currency': 'Euro'} }";

Gson gson = new Gson();

Type amountCurrencyType =  
    new TypeToken<HashMap<String, AmountWithCurrency>>(){}.getType();

HashMap<String, AmountWithCurrency> amountCurrency =  
    gson.fromJson(dollarJson, amountCurrencyType);
```

amountCurrency变量正确地拥有该集合的所有键值对

#### Sets的映射

**序列化Java Sets**

Java的集合框架包括大量的数据结构。我们已经讨论过lists和maps，它们在JSON的表达中有些许不同。这周，我们探究Sets。HashSet可以使你的数据集合例子中的值保持唯一性。因为sets有其存在的理由并且应用有现实世界，Gson也需要有能力去处理它们。

因此，让我们来看这样一个例子。你的app拥有这样一个功能，可以使得众多的用户可以加入到一个团队中。当然，每一个用户只能加入一次，因此我们使用HashSet去保存用户的名字。

使用Java的实现如下：

```java
HashSet<String> users = new HashSet<>();  
users.add("Christian");  
users.add("Marcus");  
users.add("Norman");  
users.add("Marcus"); // would not be added again  
```

sets的序列化和其他类型是一样的。你仅仅需要把它扔给Gson：

```java
Gson gson = new Gson();  
String usersJson = gson.toJson(users);  
```

结果如下所示：

```json
[
  "Marcus",
  "Christian",
  "Norman"
]
```

正如你所看到的，JSON采用和list一样的表达方式来表达set。是的，Java处理二者之间的内在逻辑是完全不同的，但是对于高层来说，它们存储相同的数据。对于如JSON这样的不管如何实现的语言来说，内在细节是无关紧要的。

**反序列化Java Sets**

正如我们上面所提到的，lists和sets在JSON中的表达都是相同的。因此，因此，Gson乐意将一个宽泛的JSON反序列化为上面的无论哪一种数据类型。在我们可以使用JSON转换为无论哪一种之前，我们先查看一下：

```java
[
    {
      "name": "Christian",
      "flowerCount": 1
    },
    {
      "name": "Marcus",
      "flowerCount": 3
    },
    {
      "name": "Norman",
      "flowerCount": 2
    }
]
```

和处理lists是一样的。我们为Gson创建一个Type，然后让它来施展它的魔术：

```java
String founderJson = "[{'name': 'Christian','flowerCount': 1}, {'name': 'Marcus', 'flowerCount': 3}, {'name': 'Norman', 'flowerCount': 2}]";

Gson gson = new Gson();

Type founderSetType = new TypeToken<HashSet<Founder>>(){}.getType();

HashSet<Founder> founderSet = gson.fromJson(founderJson, founderSetType);  
```

founderSet变量拥有和之前博客中相同的内容，仅仅是数据类型不同而已

### 配置

#### Gson注解

1. 自定义字段的名字

   ```java
   @SerializedName("宽度", alternate = "width");
   int width
   ```

   **SerializeName**接收两个参数：**value**和**alternate**。前者使用了默认的参数。如果你仅仅传入了一个字符串，那么将该字符串设置给**value**而**alternate**设置为空值。但是你可以给这两个参数传递值

   强调一遍，**value**改变了序列化和反序列化的默认情况！因此，如果Gson根据你的Java模型类创建了一个JSON，它将会使用**value**作为该属性的名。

   **alternate**仅仅是作为**反序列化**中的代选项。Gson将会JSON中的所有名称并且尝试映射到被注解了的属性中的某一个。在上面的模型类中，Gson将会检查到来的JSON中是否含有**宽度**或者**width**。无论是哪一个，都会映射到**width**属性

   如果有多个域匹配一个属性，Gson会使用最后一个遇到的域。

2. 定义那些字段需要被序列化或者反序列化

   **注意，在Java中，所有用transient声明的字段，都不会被Gson序列化和反序列化**

   Gson提供了注解来分别的控制某一个字段是否需要被序列化或者反序列化。对序列化和反序列化分开控制。

   使用@Expose注解，我们可以对序列化和反序列化单独控制，该注解有两个值，分别是deserialize和serialize，比如如下的例子

   ```java
   public class Account {
   
     //声明该字段不参与反序列化
     @Expose(deserialize = false)
     private String accountNumber;
       
   
     //只要有一个字段使用了Expose注解，所有需要参与序列化和反序列化的字段都要有这个注解
     //因为这个注解要么不生效，如果生效的话，就只会对有Expose注解的字段进行处理。
     @Expose
     private String iban;
   
     //声明该字段不参与序列化
     @Expose(serialize = false)
     private String owner;
   
     //声明该字段序列化和反序列化都不参与
     @Expose(serialize = false, deserialize = false)
     private String address;
   
     private String pin;
   }
   ```

   要使该注解生效，必须对Gson进行配置，如下

   ```java
   GsonBuilder builder = new GsonBuilder();
   builder.excludeFieldsWithoutExposeAnnotation();
   Gson gson = builder.create();
   ```

   如果我们不对Gson进行配置的话，该注解就不会生效，这样就会默认所有的字段都会被序列化和反序列化。

   通过对Gson进行配置，只有带有Expose注解的字段才会被Gson进行序列化或者反序列化。

3. @Since 和 @Until

   这2个注解用于表示数据序列化的最早版本since（自从）,和最晚版本until(直到).
   也是搭配GsonBuilder使用的。

   对Java Bean进行版本控制，这个使用的很少，比如

   ```java
   public class SoccerPlayer {
   
     private String name;
   
     //表明这个属性是1.2版本之后才加入的,[1.2,+)
     @Since(1.2)
     private int shirtNumber;
   
     //表明这个属性在0.9版本上已经被移除了,(0,0.9)
     @Until(0.9)
     private String country;
   
     private String teamName;
   
     // Methods removed for brevity
   }
   ```

   和Expose一样，要使用这两个注解，也需要对Gson进行配置，如下

   ```java
   GsonBuilder builder = new GsonBuilder();
   //在这里，我们定义版本是1.0，由于shirtNumber在1.2才加入，所以不生效
   //country在0.9被移除，所以也不生效
   builder.setVersion(1.0);
   Gson gson = builder.create();
   ```

4. @JsonAdapter.

   这个注解的作用可以自定义序列化和反序列化。比如你想给你的HashMap数据自定义序列化和反序列化。
   作用范围： class 和 field. 就是说可以放在类和字段上.

### 构建

#### GsonBuilder基础以及命名策略

在之前的示例中，仅仅使用了**Gson gson = new Gson()**方式得到Gson的实例；这在你仅仅需要Gson的基础配置时是完全有效的。然而，你可以改变Gson的设置细节。如果你需要使用Gson的方式和基础配置有略微的不同，那么这将是非常方便的。为了改变某一确定的配置，你可以使用**GsonBuilder**创建Gson实例，并自定义你的配置。

```
// previously
Gson gson = new Gson();

// now using GsonBuilder
GsonBuilder gsonBuilder = new GsonBuilder();  
Gson gson = gsonBuilder.create();  
```

**GsonBuilder**类提供了**.Create()**方法，该方法会返回一个Gson实例。该Gson实例可以做任何之前已经向你展示的功能：映射任何从JSON来或者到JSON去的数据。

**命名策略**

我们想为你展示的第一个**GsonBuilder**的可选项是命名策略。我们经常想象Java模型文件和从API发送请求然后响应回来的JSON拥有一样的命名机制。我们已经向你展示了如何使用@SerializedName在序列化时改变一个单独的属性。然而，如果你的API和Java模型不同意对它进行命名，那么使用**@SerializedName**为100多个属性添加注解将是非常繁琐的。

因此，Gson提供了配置的和自定义的**FieldNamingPolicy**。为了完成指定的目标，我们调整了UserSimple模型，并且为某些属性给定了新名称：

```java
public class UserNaming {  
    String Name;
    String email_of_developer;
    boolean isDeveloper;
    int _ageOfDeveloper;
}
```

正如你所看到的，我们将所有命名标准都统一在了一个模型里面。这将导致它的JSON结果看起来很滑稽，但是，这很容易使我们发现多钟命名策略是如何影响它们的。你可以在**GsonBuilder**中添加策略：

```java
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.IDENTITY);  
Gson gson = gsonBuilder.create();  
```

所有用到以上Gson实例的转换，都将会遵循此处的域命名策略**FieldNamingPolicy.IDENTITY**。在下一部分，我们将会探索这究竟是什么意思，以及该预定义的域命名策略是如何起作用的。让我们先从简单的**IDENTITY**开始吧。

1. FieldNamingPolicy - IDENTITY：在序列化一个对象时，**IDENTITY**域命名策略将会使用和Java模型完全一样的名称。不论你使用什么样的命名标准设置你的Java模型，JSON会使用相同的。
2. FieldNamingPolicy - LOWER_CASE_WITH_UNDERSCORES：将会按照大写字母分离每一个属性名称，并且使用一个对应的小写字母和一个**_**符号代替。
3. FieldNamingPolicy - LOWER_CASE_WITH_DASHES：这和**LOWER_CASE_WITH_UNDERSCORES**的机制是相同的，但是域名的分隔符为**-**
4. FieldNamingPolicy - UPPER_CAMEL_CASE：它使得均以大写字母开头，即使是以**_**开头的属性名。尽管该策略并没有改变分隔符。它保留了下划线。
5. FieldNamingPolicy - UPPER_CAMEL_CASE_WITH_SPACESThe last policy：几乎和**UPPER_CAMEL_CASE**一样，唯一的不同是有两个域名，在它们拥有两个大写字母的单词之间加了多了空格。

**与@SerializedName进行交互**

你可能会好奇，策略是如何和**@SerializedName**进行交互的呢？

```java
public class UserNaming {  
    String Name;

    @SerializedName("emailOfDeveloper")
    String email_of_developer;

    boolean isDeveloper;
    int _ageOfDeveloper;
}
```

如果你现在申请命名策略会发生什么呢？答案是，它将不会应用到**@SerializedName**注解的属性上。例如，我们使用**UPPER_CAMEL_CASE**，JSON结果将会：

```json
{
  "Name": "Norman",
  "_AgeOfDeveloper": 26,
  "emailOfDeveloper": "norman@futurestud.io",
  "IsDeveloper": true
}
```

**emailOfDeveloper**和**@SerializedName**注解的完全一样，而不会使起始字母大写。

**自定义域名**

已有的策略以及**@SerializedName**可能还不能满足你的用例的需求。你可以使用**FieldNamingPolicy**实现你自己的版本。因为你只能为**.setFieldNamingPolicy**传递已经预定义的枚举值，因此，Gson为你提供了另一个方法**.setFieldNamingStrategy()**。

你可以传递**FieldNamingStrategy**的实例给相应的方法。**FieldNamingStrategy**类仅仅只有一个方法。例如，如果你想要移除所有的下划线，这没有任何预定义的策略做得到。下面的代码可以帮我们做到这点：

```java
FieldNamingStrategy customPolicy = new FieldNamingStrategy() {  
    @Override
    public String translateName(Field f) {
        return f.getName().replace("_", "");
    }
};

GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setFieldNamingStrategy(customPolicy);  
Gson gson = gsonBuilder.create();

UserNaming user = new UserNaming("Norman", "norman@futurestud.io", true, 26);  
String usersJson = gson.toJson(user);  
```

结果就不会包含任何下划线了：

```json
{
  "Name": "Norman",
  "ageOfDeveloper": 26,
  "emailOfDeveloper": "norman@futurestud.io",
  "isDeveloper": true
}
```

Gson仅仅只能接受一种策略（Strategy）。因此，你必须使用在单一的**FieldNamingStrategy**实现类中实现你的逻辑。如果你多次调用了以上我们所展示的方法，那么后面的将会覆盖前面的。

#### 强制序列化null值

忽略空值这一行为对于减少JSON字符串的体积来说通常是个好主意。然而，并不总是如此。有些API会强制要求该域存在或者**null**值对于某一属性来说有特定的含义（换句话说，某值的默认值不是为空；我们需要明确的设置它为空）。

Gson为改变这一默认的行为提供了选择。我们可以使用**GsonBuilder**来为序列化提供**null**值。

```java
public class UserSimple {  
    String name;
    String email;
    boolean isDeveloper;
    int age;
}
```

现在，我们为其创建一个email为空的用户实例。

```java
Gson gson = new Gson();  
UserSimple user = new UserSimple("Norman", null, 26, true);  
String usersJson = gson.toJson(user); 
```

默认设置下，**email**属性将不会在JSON结果中出现：

```json
{
  "age": 26,
  "isDeveloper": true,
  "name": "Norman"
}
```

如果你要求**email**域作为JSON的一部分,你需要调用**GsonBuilder**的**.serializeNulls()**方法。如果你这样做了，Gson将会序列化所有属性，即使属性设置为空：

```java
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.serializeNulls();  
Gson gson = gsonBuilder.create();

UserSimple user = new UserSimple("Norman", null, 26, true);  
String usersJson = gson.toJson(user);  
```

usersJson现在包括**email**域了：

```json
{
  "age": 26,
  "email": null,
  "isDeveloper": true,
  "name": "Norman"
}
```

#### Exclusion Strategies

你已经学习了transient以及@Expose了，它们可以改变序列化和反序列化过程中的单个属性。下面我们将探讨更加普遍的方法。Gson称它为ExclusionStrategies。当然，你需要通过GsonBuilder设置它。

在我们开始特定的实现之前，先创建一个测试模型。我们使用一个新的**UserDate**模型，它拥有一些属性：

```java
public class UserDate {  
    private String _name;
    private String email;
    private boolean isDeveloper;
    private int age;
    private Date registerDate = new Date();
}
```

请注意属性的类型和名称。这将是非常重要的。假设我们需要排除所有类型为**Date**以及**boolean**的属性，使用**ExclusionStrategies**很容易做到。你可以通过**GsonBuilder**实现：

```java
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setExclusionStrategies(new ExclusionStrategy() {  
    @Override
    public boolean shouldSkipField(FieldAttributes f) {
        return false;
    }

    @Override
    public boolean shouldSkipClass(Class<?> incomingClass) {
        return incomingClass == Date.class || incomingClass == boolean.class;
    }
});
Gson gson = gsonBuilder.create();

UserDate user = new UserDate("Norman", "norman@futurestud.io", 26, true);  
String usersJson = gson.toJson(user); 
```

**ExclusionStrategies**类提供了两个重写方法。上面的例子我们使用了第二个方法。我们检查该类是否是**Date**或者**boolean**之一。如果该属性是其中之一的类型，那么该方法就会返回true，Gson将会忽略该属性。你可以在该方法中检查任意类。JSON结果将只包含字符串型和整型：

```json
{
  "age": 26,
  "email": "norman@futurestud.io",
  "_name": "Norman"
}
```

另一个方法的排除功能基于属性的声明。例如，如果我们还想排除所有包含下划线**_**的属性，那么可以按如下代码：

```java
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setExclusionStrategies(new ExclusionStrategy() {  
    @Override
    public boolean shouldSkipField(FieldAttributes f) {
        return f.getName().contains("_");
    }

    @Override
    public boolean shouldSkipClass(Class<?> incomingClass) {
        return incomingClass == Date.class || incomingClass == boolean.class;
    }
});
Gson gson = gsonBuilder.create();

UserDate user = new UserDate("Norman", "norman@futurestud.io", 26, true);  
String usersJson = gson.toJson(user);  
```

JSON结果更短了：

```json
{
  "age": 26,
  "email": "norman@futurestud.io"
}
```

你可以在各种各样不同的场景中运用排除策略。如果你拥有一些具有特殊序列化和反序列化排除的普遍的机制，这将会非常容易。请注意，你可以传递多钟排除策略到一个参数中。

如果你此刻还看不出来排除策略的重要之处，那也没关系！伴随着更加复杂的类型，我们的内容将会更加深入。一旦我们开始书写我们的自定义适配器，那么你将会看到确定值。非常幸运，通过排除策略你可以使Gson忽略**任何类型**

**排除策略仅作用于序列化或者反序列化**

在之前的篇幅中，我们为序列化和反序列化都申请了排除策略。如果你仅仅需要应用于二者之一，你可以使用下面的方法：

- **addSerializationExclusionStrategy()**
- **addDeserializationExclusionStrategy().**

这二者的效果和之前的**setExclusionStrategies()**是相同的。你同样可以传递和实现**ExclusionStrategy**对象给它们。

**基于修饰词排除成员变量**

正如我们之前解释的，所有使用**transient**修饰的成员变量在序列化和反序列化过程中都会被忽略。

GsonBuilder允许你改变这一行为。使用**excludeFieldsWithModifiers()**可以选择在序列化和反序列化过程中排除哪些被特定修饰词修饰的成员变量。该方法需要传递[java.lang.reflect.Modifier](https://link.jianshu.com?t=https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/Modifier.html)类中的修饰词。

例如，你有下面的模型：

```
public class UserModifier {  
    private String name;
    private transient String email;
    private static boolean isDeveloper;
    private final int age;
}
```

如果你想排除所有**final**和**static**类型，但包括field，你需要按如下代码配置Gson：

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.excludeFieldsWithModifiers(Modifier.STATIC, Modifier.FINAL);  
Gson gson = gsonBuilder.create();

UserModifier user = new UserModifier("Norman", "norman@fs.io", 26, true);  
String usersJson = gson.toJson(user);
```

上面的代码将会创建如下JSON：

```
{
  "email": "norman@fs.io",
  "name": "Norman"
}
```

注意，Gson实例也会包含**email**域的，即使它被设置为**transient**。调用**excludeFieldsWithModifiers()**方法重写它的默认设置。我们仅仅传递了**static**和**final**，因此**transient**修饰词将不会被忽略。如果你想要包含所有域而不管修饰词，仅需要传递空列表参数给**excludeFieldsWithModifiers()**。

**排除没有被@Expose注解的域**

最后，有一个选择我们没有多大兴趣，但会提示你所有的选项：**excludeFieldsWithoutExposeAnnotation**。正如它的名字所暗示的，它将会排除所有没有被**@Expose**注解的成员变量。

你可以强制你的应用程序模型必须每处都用**@Expose**注解了，这样可以使得开发者思考哪些域需要序列化和反序列化。

#### 轻松使用仁慈的Gson（容错机制）

JSON内容的格式必须完全遵守一些标准规则。该标准是在**RFC4627**规范描述的。它所依赖的基础是键和值的分离，数组是如何结构化的等等。

首先，当Gson序列化一个Java对象到JSON时，该JSON会100%遵循标准。因此，我们感兴趣的部分是反序列化时Gson会如何行为！

在内部，Gson使用一个[JsonReader](https://link.jianshu.com?t=https://github.com/google/gson/blob/ee8d6be59ff6f2466d65be746b96ccf07ddb9ddf/gson/docs/javadocs/com/google/gson/stream/JsonReader.html)类。该类可以选择是否具有一定的仁慈性。默认情况是不具有的，这意味着只能接收遵循标准的JSON输入。如果JSON违反了任意一条标准规则，**JsonReader**以及随之的Gson将会抛出异常。然而，**JsonReader**也可以设置为具有仁慈性。然后，也仅有如此，它会吞下容忍这些错误，尽自己最大的努力去解析这一有格式问题的JSON。

让我们引用一些没有遵循JSON标准要素的[文档](https://link.jianshu.com?t=https://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/stream/JsonReader.html#method_detail)

> Streams that start with the non-execute prefix, ")]}'\n".
>  Streams that include multiple top-level values. With strict parsing, each stream must contain exactly one top-level value.
>  Top-level values of any type. With strict parsing, the top-level value must be an object or an array.
>  Numbers may be NaNs or infinities.
>  End of line comments starting with // or # and ending with a newline character.
>  C-style comments starting with /* and ending with */. Such comments may not be nested.
>  Names that are unquoted or 'single quoted'.
>  Strings that are unquoted or 'single quoted'.
>  Array elements separated by ; instead of ,.
>  Unnecessary array separators. These are interpreted as if null was the omitted value.
>  Names and values separated by = or => instead of :.
>  Name/value pairs separated by ; instead of ,.

所有这些会如何影响你的Gson呢？理论上，Gson（不像**JsonReader**）默认情况下是仁慈的。所以通常情况下你不需要启动它，即使你的JSON是错误的。不幸的是，Gson在面对这些问题时的行为表现的有点疑惑。

你一定记得Gson可以选择在反序列化JSON的过程中拥有更大的容忍度。如果你需要设置这一点，使用**GsonBuilder**：

```
GsonBuilder gsonBuilder = new GsonBuilder();  
gsonBuilder.setLenient();  
Gson gson = gsonBuilder.create();  
```

使你的Gson实例具有仁慈性可以解决一些具有结构错误的问题。然而，其仁慈性也仅仅能够如此。如果JSON违背了超过上面我们所实例的所有错误，Gson会抛出**MalformedJsonException**异常。如果是这样，那么你需要检查你的JSON看看它的结构是否合法。大部分情况下，这是你问题的根源。

推荐你使用像[beJson](http://www.bejson.com/)这样的工具检查你的JSON是否合法。

#### Float和Double类型的特殊值

在Java中，某些特殊情况下需要将值置为float型和double型。因此，在Java语言刚开始的时候，Float.NEGATIVE_INFINITY，Float.POSITIIVE_INFINITY以及Float.NaN以及他们对应的double型就以及存在了。遗憾的是，JSON标准不知道它们的价值，并没有将它们作为标准的一部分。

让我们引用Gson文档中关于该问题的描述：

> [JSON规范](https://link.jianshu.com?t=http://www.ietf.org/rfc/rfc4627.txt)的2.4章节不允许拥有特殊的double值（NaN，Infinity，-Infinity）。然而，[Javascript标准](https://link.jianshu.com?t=http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf)（查看章节4.3.20，4.3.22，4.3.23）允许它们作为Javascript的合法值。甚至，大部分的JavaScript引擎接收这些在JSON中的特殊值而不会发生任何问题。因此，从实践层面来看，即使JSON规范不支持它们，但接收这些值作为合法的JSON也是有理可寻的。

反序列化时Gson对于JSON标准具有很棒的灵活性，而在序列化时又是灵活的。前一部分依然是正确的，但是，这些特殊的float和double值是唯一的意外，它们使得Gson会内在的违反标准。

让我们讲得再透彻一点：如果你传入的JSON使用了一个或多个那样的float或double边缘值，它们会被默认反序列化，而不会有任何问题。

如果你传递一个**Float.POSITIVE_INFINITY**作为值，Gson将会抛出异常，因为Gson不能根据标准来进行转换。如果你需要传递这些特殊值，你需要使用**GsonBuilder**来明确的指定这些值可以转换。Gson为你提供了**serializeSpecialFloatingPointValues**来处理。

是时候来看一个例子了。我们来创建一个用户类，它拥有一个float类型的成员变量weight。

```java
public class UserFloat {  
    String name;
    Float weight;

    public UserFloat(String name, Float weight) {
        this.name = name;
        this.weight = weight;
    }
}
```

现在，复活节的巧克力兔女郎对我产生了负面影响，最后我的体重大增，因此我需要像下面那样创建我的用户实例：

```java
UserFloat user = new UserFloat("Norman", Float.POSITIVE_INFINITY);
```

如果你将之传递给Gson，它将会抛出异常：

```java
Gson gson = new Gson();

UserFloat user = new UserFloat("Norman", Float.POSITIVE_INFINITY);

String usersJson = gson.toJson(user); // will throw an exception  
```

Gson将会给你一个**IllegalArgumentException**异常，它的陈述如下：

> Infinity并不是JSON规范的有效double值。想要使用该行为，使用**serializeSpecialFloatingPointValues()**方法。

异常信息是非常有帮助的，并且已经提供给了我们解决办法。我们需要使用**GsonBuilder**：

```java
        Gson gson =
                new GsonBuilder()
                        .serializeSpecialFloatingPointValues()
                        .create();
        UserFloat user = new UserFloat("Norman", Float.POSITIVE_INFINITY);

        String usersJson = gson.toJson(user); // will throw an exception
        System.out.println(usersJson);
//输出
//{"name":"Norman","weight":Infinity}
```

#### 模型版本化

Gson可以通过**@Since**注解以及**@Until**注解来为你的Java对象设置版本控制，如此，则你的模型类里面被以上两个注解标记了的成员变量，将只有符合特定版本范围内时才会被序列化和反序列化。

这两个注解只有在通过GsonBuilder创建的Gson实例上才有效，我们需要通过**GsonBuilder.setVersion(double)**来激活。

**@Since**

该注解指示出某一成员或类型在这一特定的版本号之后才存在。例如有下面的模板类：

```java
public class User {
   private String firstName;
   private String lastName;
    //[1.0,+]
   @Since(1.0) private String emailAddress;
    //[1.0,+]
   @Since(1.0) private String password;
    //[1.1,+]
   @Since(1.1) private Address address;
 }
```

如果你使用**new Gson()**创建Gson实例，那么**toJson()**和**fromJson()**不会使用它们。然而，如果你使用**Gson gson = new GsonBuilder().setVersion(1.0).create()**来创建Gson实例，那么**toJson()**和**fromJson()**方法将排除**address**域，因为它的版本号被设置为了**1.1**。

**@Until**

**@Until**注解为某一成员或类型指定了一个版本号，代表该成员或类型在该版本号之前才存在。
 稍微改变User模型：

```java
 public class User {
   private String firstName;
   private String lastName;
     //[0,1.1)
   @Until(1.1) private String emailAddress;
     //[0,1.1)
   @Until(1.1) private String password;
 }
```

现在，我们使用**Gson gson = new GsonBuilder().setVersion(1.2).create()**来创建Gson实例，**toJson()**和**fromJson()**方将排除**emailAddress**和**password**域，这是因为传入的版本号为**1.2**，超过了我们给定的**1.1**。

#### 格式化日期和时间

正如你所想到的，该功能也需要配置**GsonBuilder**。Gson为我们提供了三个重载的方法。

- setDateFormat(String pattern):pattern遵循[SimpleDateFormat](https://link.jianshu.com?t=http://docs.oracle.com/javase/6/docs/api/java/text/SimpleDateFormat.html?is-external=true)类的惯例。
- setDateFormat(int style)：style必须是[DateFormat](https://link.jianshu.com?t=http://docs.oracle.com/javase/6/docs/api/java/text/DateFormat.html?is-external=true)的一个常量。
- setDateFormat(int dateStyle, int timeStyle)：类似于上面的，只不过将日期和时间分开了。

#### 漂亮输出

序列化得到的JSON是无空格的，所有字符都密密麻麻挤在了一起，这虽然节约空间，但对于人的理解却不友好。只需相应的设置**GsonBuilder**即可：

```java
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String jsonOutput = gson.toJson(someObject);
```

### 参考

1. [GSON](https://www.jianshu.com/p/75a50aa0cad1)
2. [Google-Gson注解使用详解](https://juejin.im/post/59e5663f51882546b15b92f0)
3. [Gson基本用法](https://www.cnblogs.com/baiqiantao/p/7512336.html)
4. [Gson完全教程：基础篇](https://www.jianshu.com/p/923a9fe78108)
5. [Gson全解析（中）-TypeAdapter的使用](https://www.jianshu.com/p/8cc857583ff4)
6. [Gson的实现原理](https://blog.csdn.net/jiangjiajian2008/article/category/6530319)