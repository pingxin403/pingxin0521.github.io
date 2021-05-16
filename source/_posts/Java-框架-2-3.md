---
title: JSON框架--Jackson
date: 2019-05-19 15:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

Jackson是一个简单基于Java应用库，Jackson可以轻松的将Java对象转换成json对象和xml文档，同样也可以将json、xml转换成Java对象。Jackson所依赖的jar包较少，简单易用并且性能也要相对高些，并且Jackson社区相对比较活跃，更新速度也比较快。

<!--more-->

以Avro， BSON， CBOR， CSV， Smile， （Java）Properties， Protobuf， XML 或YAML编码的处理数据 ; 甚至还有大量的数据格式模块，以支持广泛使用的数据类型的数据类型，例如 Guava， Joda， PCollections 等等。

**特点**

- 容易使用 - jackson API提供了一个高层次外观，以简化常用的用例。
- 无需创建映射 - API提供了默认的映射大部分对象序列化。
- 性能高 - 快速，低内存占用，适合大型对象图表或系统。
- 干净的JSON - jackson创建一个干净和紧凑的JSON结果，这是让人很容易阅读。
- 不依赖 - 库不需要任何其他的库，除了JDK。
- 开源代码 - jackson是开源的，可以免费使用。

**jackson主要的包**

- jackson-core——核心包（必须），提供基于“流模式”解析的API。核心包：JsonPaser（json流读取），JsonGenerator（json流输出）。
- jackson-databind——数据绑定包（可选），提供基于“对象绑定”和“树模型”相关API。数据绑定包：ObjectMapper（构建树模式和对象绑定模式），JsonNode（树节点）。
- jackson-annotations——注解包（可选），提供注解功能。
- jackson-datatype-joda-2.1.5.jar——日期转换

**三种方式处理JSON**

提供了三种不同的方法来处理JSON

- 流式API - 读取并将JSON内容写入作为离散事件。 JsonParser读取数据，而JsonGenerator写入数据。它是三者中最有效的方法，是最低的开销和最快的读/写操作。它类似于Stax解析器XML。流式API是一套比较底层的API，速度快，但是使用起来特别麻烦。使用通常从创建可重用（并且在配置后是线程安全的）JsonFactory实例开始

  ```java
  JsonFactory factory =  new JsonFactory();
  // 配置（如有必要）： 
  factory.enable(JsonParser.Feature.ALLOW_COMMENTS);
  // 另外，您也可以方便地ObjectMapper（从Jackson-Databind软件包中获得）。如果是这样，您可以执行以下操作：
  JsonFactory factory = objectMapper.getFactory();
  
  // 首先写入
  File jsonFile = new File("test.json");
  JsonGenerator g = f.createGenerator(jsonFile);
  // 写入 JSON: { "message" : "Hello world!" }
  g.writeStartObject();
  g.writeStringField("message", "Hello world!");
  g.writeEndObject();
  g.close();
  
  // 然后读取
  JsonParser p = f.createParser(jsonFile);
  
  JsonToken t = p.nextToken(); 
  t = p.nextToken(); 
  if ((t != JsonToken.FIELD_NAME) || !"message".equals(p.getCurrentName())) {
     // handle error
  }
  t = p.nextToken();
  if (t != JsonToken.VALUE_STRING) {
     // similarly
  }
  String msg = p.getText();
  System.out.printf("My message to you is: %s!\n", msg);
  p.close();
  ```

  官方使用示例：[Reading and Writing Event Streams](http://www.cowtowncoder.com/blog/archives/2009/01/entry_132.html)

- 树模型 - 准备JSON文件在内存里以树形式表示。 ObjectMapper构建JsonNode节点树。这是最灵活的方法。它类似于XML的DOM解析器。

  ```java
  JsonNode root = mapper.readTree(rerenjson);
  JsonNode user = root.get("user");
  root.with("other").put("type", "student");
  int id = user.get("id").asInt();
  String name = user.get("name").asText();
  JsonNode avators = user.get("avatar");
  if (avators.isArray()) {
      for (Iterator it = avators.getElements(); it.hasNext(); ){
          JsonNode avator = it.next();
          if ("tiny".equals(avator.get("type").asText())) {
              String ava = avator.get("url").asText();
              break;
          }
      }
  }
  
  ```

-  			数据绑定 - 转换JSON并从POJO（普通Java对象）使用属性访问或使用注释。它有两个类型。

  -  					简单的数据绑定 - 转换JSON和Java Maps, Lists, Strings, Numbers, Booleans 和null 对象。
  -  					全部数据绑定 - 转换为JSON从任何JAVA类型。

   ObjectMapper读/写JSON两种类型的数据绑定。数据绑定是最方便的方式是类似XML的JAXB解析器。
  
  ```java
  //创建一个com.fasterxml.jackson.databind.ObjectMapper用于所有数据绑定的实例，ObjectMapper是线程安全的，应该尽量的重用。
  ObjectMapper mapper = new ObjectMapper();
  //  字符串反序列化为对象
  Grade grade = mapper.readValue(jsonStr, Grade.class);
  //  Jackson是基于JavaBean来序列化属性的，属性要有GETTER方法，才会输出该属性
  Map<String, ResultValue> results = mapper.readValue(jsonSource, new TypeReference<Map<String, ResultValue>>() { } );
  
  // 对象序列号化为json字符串
  String gradeStr = mapper.writeValueAsString(grade);
  // 对象序列号化为byte数组
  byte[] jsonBytes = mapper.writeValueAsBytes(grade);
  // 写入文件
  mapper.writeValue(new File("result.json"), grade);
  
  ```
  
  jackson-databind软件包依赖于jackson-core和jackson-annotations软件包，但是当使用Maven或Gradle之类的构建工具时，依赖项会自动包括在内。

**类型转换**

Jackson的一项有用（但不是很广为人知）的功能是它能够进行任意的POJO到POJO转换。一般做法，首先将POJO编写为JSON，其次将JSON绑定到另一种POJO中。Jackson实现跳过JSON的实际生成，并使用更高效的中间实现。

```java
// 基本使用
ResultType result = mapper.convertValue(sourceObject, ResultType.class);

// List<Integer> 转 int[]
List<Integer> sourceList = ...;
int[] ints = mapper.convertValue(sourceList, int[].class);
// POJO 转 Map!
Map<String,Object> propertyMap = mapper.convertValue(pojoValue, Map.class);
// POJO 转 back
PojoType pojo = mapper.convertValue(propertyMap, PojoType.class);
// 解码Base64！（缺省字节[]表示是base64编码字串）
String base64 = "TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5IGhpcyByZWFzb24sIGJ1dCBieSB0aGlz";
byte[] binary = mapper.convertValue(base64, byte[].class);


//As Array:

MyClass[] myObjects = mapper.readValue(json, MyClass[].class);

//As List:
List<MyClass> myObjects = mapper.readValue(jsonInput, new TypeReference<List<MyClass>>(){});

//Another way to specify the List type:
List<MyClass> myObjects = mapper.readValue(jsonInput, mapper.getTypeFactory().constructCollectionType(List.class, MyClass.class));

```

#### Jackson注解

1. 属性重命名时使用的注解

  最常见的使用方式之一就是改变某个成员属性所使用的JSON名称。例如：

  ```java
   class Name {
       @JsonProperty("firstName") 
       public String _first_name; 
   }
  ```

2. 忽略属性使用的注解

  有时POJO包括了一些你不希望输出的属性，在这种情况下，你可以进行如下操作：

  ```java
  public class Value {
    public int value;
    @JsonIgnore
     public int internalValue;
  }
  ```

  或者，你可能忽略掉某些从JSON数据中得到的属性，如果是这样，你可以使用：

  ```java
  @JsonIgnoreProperties({ "extra", "uselessValue" })
  public class Value {
    public int value;
  }
  ```

  或者，更粗暴点的，忽略掉从JSON（由于在应用中没有完全匹配的POJO）中获得的所有“多余的”属性。

  ```java
  @JsonIgnoreProperties(ignoreUnknown=true)
  public class PojoWithAny {
    public int value;
  }
  ```

3. Date类型直接转化为想要的格式:`@JsonFormat(pattern = “yyyy-MM-dd HH-mm-ss”)`

4. `@JsonInclude`注解来标记属性的返回值，如忽略null字段的序列化

   - Include.Include.ALWAYS （Default / 都参与序列化）
   - Include.NON_DEFAULT（当Value 为默认值的时候不参与，如Int a; 当 a=0 的时候不参与）
   - Include.NON_EMPTY（当Value 为“” 或者null 不输出）
   - Include.NON_NULL（当Value 为null 不输出）

5. 反序列化（json转对象时）使用别名`@JsonAlias`

   ```java
   @Data
   public class PlayerStar {
   
     @JsonAlias({"starName", "playerName" })
     private String name;
   
   ```

   下面三种JSON格式数据都可以被正确的反序列化为PlayerStar对象，并为name成员变量赋值

   ```java
   String jsonInString = "{\"name\":\"乔丹\",\"age\":45,\"hobbies\":[\"高尔夫球\",\"棒球\"]}";
   String jsonInString = "{\"starName\":\"乔丹\",\"age\":45,\"hobbies\":[\"高尔夫球\",\"棒球\"]}";
   String jsonInString = "{\"playerName\":\"乔丹\",\"age\":45,\"hobbies\":[\"高尔夫球\",\"棒球\"]}";
   ```

6. 序列化字段排序`@JsonPropertyOrder`,使用时直接在内部填写顺序，或者使用`alphabetic=true`自动按照字母升序排序

   ```java
   @JsonPropertyOrder({ "id", "name" })
       
   @JsonPropertyOrder(alphabetic=true)
   ```

更多参考：https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations

#### Mapper的一些实用配置

```java
// （空对象是否抛出异常）
mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
// （日期改为时间戳）
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
// （特殊字符和打印符，这在FastJson曾是个bug）
mapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
// （单引号）
mapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
```

#### SpringBoot中使用Jackson

SpringMVC内置json解析器就是Jackson，以SpringBoot技术栈开发的话,用默认的Jackson是最好的。

```yml
spring:
    jackson:
      # 设置属性命名策略,对应jackson下PropertyNamingStrategy中的常量值，SNAKE_CASE-返回的json驼峰式转下划线，json body下划线传到后端自动转驼峰式
      property-naming-strategy: SNAKE_CASE
      # 全局设置@JsonFormat的格式pattern
      date-format: yyyy-MM-dd HH:mm:ss
      # 当地时区
      locale: zh
      # 设置全局时区
      time-zone: GMT+8
      # 常用，全局设置pojo或被@JsonInclude注解的属性的序列化方式
      default-property-inclusion: NON_NULL #不为空的属性才会序列化,具体属性可看JsonInclude.Include
      # 常规默认,枚举类SerializationFeature中的枚举属性为key，值为boolean设置jackson序列化特性,具体key请看SerializationFeature源码
      serialization:
        WRITE_DATES_AS_TIMESTAMPS: true # 返回的java.util.date转换成timestamp
        FAIL_ON_EMPTY_BEANS: true # 对象为空时是否报错，默认true
      # 枚举类DeserializationFeature中的枚举属性为key，值为boolean设置jackson反序列化特性,具体key请看DeserializationFeature源码
      deserialization:
        # 常用,json中含pojo不存在属性时是否失败报错,默认true
        FAIL_ON_UNKNOWN_PROPERTIES: false
      # 枚举类MapperFeature中的枚举属性为key，值为boolean设置jackson ObjectMapper特性
      # ObjectMapper在jackson中负责json的读写、json与pojo的互转、json tree的互转,具体特性请看MapperFeature,常规默认即可
      mapper:
        # 使用getter取代setter探测属性，如类中含getName()但不包含name属性与setName()，传输的vo json格式模板中依旧含name属性
        USE_GETTERS_AS_SETTERS: true #默认false
      # 枚举类JsonParser.Feature枚举类中的枚举属性为key，值为boolean设置jackson JsonParser特性
      # JsonParser在jackson中负责json内容的读取,具体特性请看JsonParser.Feature，一般无需设置默认即可
      parser:
        ALLOW_SINGLE_QUOTES: true # 是否允许出现单引号,默认false
      # 枚举类JsonGenerator.Feature枚举类中的枚举属性为key，值为boolean设置jackson JsonGenerator特性，一般无需设置默认即可
      # JsonGenerator在jackson中负责编写json内容,具体特性请看JsonGenerator.Feature
```

##### 优雅序列化Java枚举类

为了便于统一处理和规范统一的风格，建议指定一个统一的抽象接口，例如：

```java
/**
 * The interface Enumerator.
 */
public interface Enumerator {
    /**
     * Code integer.
     *
     * @return the integer
     */
    Integer code();

    /**
     * Description string.
     *
     * @return the string
     */
    String description();
}
```

我们来写一个实现来标识性别：

```java
public enum GenderEnum implements Enumerator {
   
    UNKNOWN(0, "未知"),

    MALE(1, "男"),

    FEMALE(2, "女");


    private final Integer code;
    private final String description;

    GenderEnum(Integer code, String description) {
        this.code = code;
        this.description = description;
    }


    @Override
    public Integer code() {
        return code;
    }

    @Override
    public String description() {
        return description;
    }
}
```

如果我们直接使用**Jackson**对枚举进行序列化，将只能简单的输出枚举的`String`名称。在**Spring Boot**应用中我们希望能全局配置。**Spring Boot**的自动配置为我们提供了一个个性化定制`ObjectMapper`的可能性，你只需要声明一个`Jackson2ObjectMapperBuilderCustomizer`并注入**Spring IoC**：

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer enumCustomizer(){
    return jacksonObjectMapperBuilder -> jacksonObjectMapperBuilder.serializerByType(Enumerator.class, new JsonSerializer<Enumerator>() {
        @Override
        public void serialize(Enumerator value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
                    gen.writeStartObject();
                    gen.writeNumberField("code",value.code());
                    gen.writeStringField("description",value.description());
                    gen.writeEndObject();


        }
    });
}
```

##### 提供对LocalDate的支持

上面的日期配置对于java8新提供的日期APILocalDate、LocalDateTime等无效。

解决办法：

1. 引入依赖

   ```xml
   
   <dependency>
         <groupId>com.fasterxml.jackson.datatype</groupId>
         <artifactId>jackson-datatype-jsr310</artifactId>
         <version>2.8.9</version>
   </dependency>
   ```

2. 添加如下代码

   ```java
        /**
        * DateTime格式化字符串
        */
       private static final String DEFAULT_DATETIME_PATTERN = "yyyy-MM-dd HH:mm:ss";
   
       /**
        * Date格式化字符串
        */
       private static final String DEFAULT_DATE_PATTERN = "yyyy-MM-dd";
   
       /**
        * Time格式化字符串
        */
       private static final String DEFAULT_TIME_PATTERN = "HH:mm:ss";
   
   /**
     * 配置一个Jackson2ObjectMapperBuilderCustomizerBean
     * Jackson序列化和反序列化转换器，用于转换Post请求体中的json以及将对象序列化为返回响应的json
     */
   @Bean
   public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
       return builder -> builder
               .serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATETIME_PATTERN)))
               .serializerByType(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_PATTERN)))
               .serializerByType(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_PATTERN)))
               .serializerByType(Date.class, new DateSerializer(false, new SimpleDateFormat(DEFAULT_DATETIME_PATTERN)))
               .deserializerByType(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATETIME_PATTERN)))
               .deserializerByType(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_PATTERN)))
               .deserializerByType(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_PATTERN)))
               .deserializerByType(Date.class, new DateDeserializers.DateDeserializer(DateDeserializers.DateDeserializer.instance, new SimpleDateFormat(DEFAULT_DATETIME_PATTERN), DEFAULT_DATETIME_PATTERN))
               ;
   }
   
   //或者自定义一个MappingJackson2HttpMessageConverter
   
   @Bean
   public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
       MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
       ObjectMapper objectMapper = new ObjectMapper();
       // 指定时区
       objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8:00"));
       // 日期类型字符串处理
       objectMapper.setDateFormat(new SimpleDateFormat(DEFAULT_DATETIME_PATTERN));
   
       // Java8日期日期处理
       JavaTimeModule javaTimeModule = new JavaTimeModule();
       javaTimeModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATETIME_PATTERN)));
       javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_PATTERN)));
       javaTimeModule.addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_PATTERN)));
       javaTimeModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATETIME_PATTERN)));
       javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_PATTERN)));
       javaTimeModule.addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_PATTERN)));
       objectMapper.registerModule(javaTimeModule);
   
       converter.setObjectMapper(objectMapper);
       return converter;
   }
   ```

   

### 参考

1. [jackson](https://github.com/FasterXML/jackson)
2. [jackson-core](https://github.com/FasterXML/jackson-core)
3. [jackson-databind](https://github.com/FasterXML/jackson-databind)
4. [如何在Spring Boot应用中优雅的使用Date和LocalDateTime](https://www.jianshu.com/p/ccab14aef228)