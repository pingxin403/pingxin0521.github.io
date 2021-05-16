---
title: JSON框架--fastJSON
date: 2019-05-19 11:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

Java对于处理JSON数据的序列化与反序列化目前常用的类库有Gson、FastJSON、Jackson。

fastjson用于将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。

JSON协议使用方便，越来越流行,JSON的处理器有很多,这里我介绍一下FastJson,FastJson是阿里的开源框架,被不少企业使用,是一个极其优秀的Json框架,Github地址: [FastJson](https://github.com/alibaba/fastjson)

[教程](https://www.w3cschool.cn/fastjson/fastjson-serializefilter.html)

<!--more-->

**特点:**

1. FastJson数度快,无论序列化和反序列化,都是当之无愧的fast  
2. 功能强大(支持普通JDK类包括任意Java Bean Class、Collection、Map、Date或enum)  
3. 零依赖(没有依赖其它任何类库)

**简单说明:**

FastJson对于json格式字符串的解析主要用到了下面三个类：  

1. JSON：fastJson的解析器，用于JSON格式字符串与JSON对象及javaBean之间的转换  
2. JSONObject：fastJson提供的json对象    
3. JSONArray：fastJson提供json数组对象

**依赖：**

通过maven引入相应的json包

```xml
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.49</version>
        </dependency>
    </dependencies>
```

定义一个需要转换所实体类User，代码如下：

```java
package com.ivan.json.entity;

import java.util.Date;

import com.alibaba.fastjson.annotation.JSONField;

public class User {

    private Long   id;

    private String name;

    @JSONField(format = "yyyy-MM-dd HH:mm:ss")
    private Date   createTime;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }
    
}
```

写个简单的测试类用于测试fastjson的序列化与反序列化，代码如下：

```java
package com.ivan.json;

import java.util.Date;

import com.alibaba.fastjson.JSON;
import com.ivan.json.entity.User;

public class SimpleTest {

    public static void main(String[] args) {
        serialize();
        deserialize();
    }

    public static void serialize() {
        User user = new User();
        user.setId(11L);
        user.setName("西安");
        user.setCreateTime(new Date());
        String jsonString = JSON.toJSONString(user);
        System.out.println(jsonString);
    }

    public static void deserialize() {
        String jsonString = "{\"createTime\":\"2018-08-17 14:38:38\",\"id\":11,\"name\":\"西安\"}";
        User user = JSON.parseObject(jsonString, User.class);
        System.out.println(user.getName());
        System.out.println(user.getCreateTime());
    }
}
```

#### SerializerFeature特性的使用

fastjson通过SerializerFeature对生成的json格式的数据进行一些定制，比如可以输入的格式更好看，使用单引号而非双引号等。例子程序如下：

```java
package com.ivan.json;

import java.util.Date;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.ivan.json.entity.User;

public class SerializerFeatureTest {

    public static void main(String[] args) {
        User user = new User();
        user.setId(11L);
        user.setCreateTime(new Date());
        String jsonString = JSON.toJSONString(user, SerializerFeature.PrettyFormat, 
                SerializerFeature.WriteNullStringAsEmpty, SerializerFeature.UseSingleQuotes);
        System.out.println(jsonString);

    }

}
```

SerializerFeature常用属性

| 名称                           | 含义                                                         |
| ------------------------------ | :----------------------------------------------------------- |
| QuoteFieldNames                | 输出key时是否使用双引号,默认为true                           |
| UseSingleQuotes                | 使用单引号而不是双引号,默认为false                           |
| WriteMapNullValue              | 是否输出值为null的字段,默认为false                           |
| WriteEnumUsingToString         | Enum输出name()或者original,默认为false                       |
| UseISO8601DateFormat           | Date使用ISO8601格式输出，默认为false                         |
| WriteNullListAsEmpty           | List字段如果为null,输出为[],而非null                         |
| WriteNullStringAsEmpty         | 字符类型字段如果为null,输出为”“,而非null                     |
| WriteNullNumberAsZero          | 数值字段如果为null,输出为0,而非null                          |
| WriteNullBooleanAsFalse        | Boolean字段如果为null,输出为false,而非null                   |
| SkipTransientField             | 如果是true，类中的Get方法对应的Field是transient，序列化时将会被忽略。默认为true |
| SortField                      | 按字段名称排序后输出。默认为false                            |
| WriteTabAsSpecial              | 把\t做转义输出，默认为false不推荐设为true                    |
| PrettyFormat                   | 结果是否格式化,默认为false                                   |
| WriteClassName                 | 序列化时写入类型信息，默认为false。反序列化是需用到          |
| DisableCircularReferenceDetect | 消除对同一对象循环引用的问题，默认为false                    |
| WriteSlashAsSpecial            | 对斜杠’/’进行转义                                            |
| BrowserCompatible              | 将中文都会序列化为\uXXXX格式，字节数会多一些，但是能兼容IE 6，默认为false |
| WriteDateUseDateFormat         | 全局修改日期格式,默认为false。                               |
| DisableCheckSpecialChar        | 一个对象的字符串属性中如果有特殊字符如双引号，将会在转成json时带有反斜杠转移符。如果不需要转义，可以使用这个属性。默认为false |
| BeanToArray                    | 将对象转为array输出                                          |

#### JSONField与JSONType注解的使用

fastjson提供了JSONField对序列化与反序列化进行定制，比如可以指定字段的名称，序列化的顺序。JSONField用于属性，方法方法参数上。JSONField的源码如下：

```java
package com.alibaba.fastjson.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.serializer.SerializerFeature;

@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
public @interface JSONField {
// 配置序列化和反序列化的顺序
    int ordinal() default 0;
// 指定字段的名称
    String name() default "";
// 指定字段的格式，对日期格式有用
    String format() default "";
 // 是否序列化
    boolean serialize() default true;
// 是否反序列化
    boolean deserialize() default true;
//字段级别的SerializerFeature
    SerializerFeature[] serialzeFeatures() default {};
//
    Feature[] parseFeatures() default {};
   //给属性打上标签， 相当于给属性进行了分组
    String label() default "";
    
    boolean jsonDirect() default false;
    
//制定属性的序列化类
    Class<?> serializeUsing() default Void.class;
 //制定属性的反序列化类
    Class<?> deserializeUsing() default Void.class;

    String[] alternateNames() default {};

    boolean unwrapped() default false;
}
```

其中serializeUsing与deserializeUsing可以用于对字段的序列化与反序列化进行定制化。比如我们在User实体上加上个sex属性，类型为boolean。下面分别定义了序列化类与反序列化类，序列化类代码如下：

```java
package com.ivan.json.converter;

import java.io.IOException;
import java.lang.reflect.Type;

import com.alibaba.fastjson.serializer.JSONSerializer;
import com.alibaba.fastjson.serializer.ObjectSerializer;

public class SexSerializer implements ObjectSerializer {

    public void write(JSONSerializer serializer,
                      Object object,
                      Object fieldName,
                      Type fieldType,
                      int features)
            throws IOException {
        Boolean value = (Boolean) object;
        String text = "女";
        if (value != null && value == true) {
            text = "男";
        }
        serializer.write(text);
    }

}
```

反序列化类代码如下：

```java
package com.ivan.json.converter;

import java.lang.reflect.Type;


import com.alibaba.fastjson.parser.DefaultJSONParser;
import com.alibaba.fastjson.parser.JSONToken;
import com.alibaba.fastjson.parser.deserializer.ObjectDeserializer;

public class SexDeserialize implements ObjectDeserializer {

    public <T> T deserialze(DefaultJSONParser parser,
                                        Type type,
                                        Object fieldName) {
        


        String sex = parser.parseObject(String.class);
        if ("男".equals(sex)) {
            return (T) Boolean.TRUE;
        } else {
            return (T) Boolean.FALSE;
        }
    }

    public int getFastMatchToken() {
        return JSONToken.UNDEFINED;
    }

}
```

fastjosn提供了JSONType用于类级别的定制化, JSONType的源码如下：

```java
package com.alibaba.fastjson.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import com.alibaba.fastjson.PropertyNamingStrategy;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.serializer.SerializeFilter;
import com.alibaba.fastjson.serializer.SerializerFeature;

@Retention(RetentionPolicy.RUNTIME)
//需要标注在类上
@Target({ ElementType.TYPE })
public @interface JSONType {

    boolean asm() default true;
//这里可以定义输出json的字段顺序
    String[] orders() default {};
//包含的字段
    String[] includes() default {};
//不包含的字段
    String[] ignores() default {};
//类级别的序列化特性定义
    SerializerFeature[] serialzeFeatures() default {};
    Feature[] parseFeatures() default {};
    //按字母顺序进行输出
    boolean alphabetic() default true;
    
    Class<?> mappingTo() default Void.class;
    
    Class<?> builder() default Void.class;
    
    String typeName() default "";

    String typeKey() default "";
    
    Class<?>[] seeAlso() default{};
    //序列化类
    Class<?> serializer() default Void.class;
    //反序列化类
    Class<?> deserializer() default Void.class;

    boolean serializeEnumAsJavaBean() default false;

    PropertyNamingStrategy naming() default PropertyNamingStrategy.CamelCase;

    Class<? extends SerializeFilter>[] serialzeFilters() default {};
}
```

#### SerializeFilter

fastjson通过SerializeFilter编程扩展的方式定制序列化fastjson支持以下SerializeFilter用于不同常景的定制序列化：

- PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化,接口定义如下：

```java
package com.alibaba.fastjson.serializer;

/**
 * @author wenshao[szujobs@hotmail.com]
 */
public interface PropertyFilter extends SerializeFilter {

    /**
     * @param object the owner of the property
     * @param name the name of the property
     * @param value the value of the property
     * @return true if the property will be included, false if to be filtered out
    * 根据 属性的name与value判断是否进行序列化
     */
    boolean apply(Object object, String name, Object value);
}
```

- PropertyPreFilter根据PropertyName判断是否序列化

```java
package com.alibaba.fastjson.serializer;

public interface PropertyPreFilter extends SerializeFilter {

//根据 object与name判断是否进行序列化
    boolean apply(JSONSerializer serializer, Object object, String name);
}
```

- NameFilter 序列化时修改Key

```java
package com.alibaba.fastjson.serializer;

public interface NameFilter extends SerializeFilter {
//根据 name与value的值，返回json字段key的值
    String process(Object object, String name, Object value);
}
```

- ValueFilter 序列化时修改Value

```java
package com.alibaba.fastjson.serializer;

public interface ValueFilter extends SerializeFilter {
  //根据name与value定制输出json的value
    Object process(Object object, String name, Object value);
}
```

- BeforeFilter 在序列化对象的所有属性之前执行某些操作

```java
package com.alibaba.fastjson.serializer;

public abstract class BeforeFilter implements SerializeFilter {

    private static final ThreadLocal<JSONSerializer> serializerLocal = new ThreadLocal<JSONSerializer>();
    private static final ThreadLocal<Character>      seperatorLocal  = new ThreadLocal<Character>();

    private final static Character                   COMMA           = Character.valueOf(',');

    final char writeBefore(JSONSerializer serializer, Object object, char seperator) {
        serializerLocal.set(serializer);
        seperatorLocal.set(seperator);
        writeBefore(object);
        serializerLocal.set(null);
        return seperatorLocal.get();
    }

    protected final void writeKeyValue(String key, Object value) {
        JSONSerializer serializer = serializerLocal.get();
        char seperator = seperatorLocal.get();
        serializer.writeKeyValue(seperator, key, value);
        if (seperator != ',') {
            seperatorLocal.set(COMMA);
        }
    }
//需要实现的方法，在实际实现中可以调用writeKeyValue增加json的内容
    public abstract void writeBefore(Object object);
}
```

- AfterFilter 在序列化对象的所有属性之后执行某些操作

```java
package com.alibaba.fastjson.serializer;

/**
 * @since 1.1.35
 */
public abstract class AfterFilter implements SerializeFilter {

    private static final ThreadLocal<JSONSerializer> serializerLocal = new ThreadLocal<JSONSerializer>();
    private static final ThreadLocal<Character>      seperatorLocal  = new ThreadLocal<Character>();

    private final static Character    COMMA           = Character.valueOf(',');

    final char writeAfter(JSONSerializer serializer, Object object, char seperator) {
        serializerLocal.set(serializer);
        seperatorLocal.set(seperator);
        writeAfter(object);
        serializerLocal.set(null);
        return seperatorLocal.get();
    }

    protected final void writeKeyValue(String key, Object value) {
        JSONSerializer serializer = serializerLocal.get();
        char seperator = seperatorLocal.get();
        serializer.writeKeyValue(seperator, key, value);
        if (seperator != ',') {
            seperatorLocal.set(COMMA);
        }
    }
//子类需要实现的方法，实际使用的时候可以调用writeKeyValue增加内容
    public abstract void writeAfter(Object object);
}
```

- LabelFilter根据 JsonField配置的label来判断是否进行输出

```java
package com.alibaba.fastjson.serializer;

//根据 JsonField配置的label来判断是否进行输出
public interface LabelFilter extends SerializeFilter {
    boolean apply(String label);
}
```

#### 泛型反序列化

fastjson通过TypeReference来实现泛型的反序列化，以下是一个简单的例子程序。首先定义了BaseDTO用于所有DTO的父类，代码如下：

```java
package com.ivan.frame.dto.common;

import java.io.Serializable;

import com.alibaba.fastjson.JSONObject;

public class BaseDTO implements Serializable{

    private static final long  serialVersionUID = 2230553030766621644L;

    @Override
    public String toString() {
        return JSONObject.toJSONString(this);
    }

}
```

RequestDTO用于抽像所有的请求DTO，里面有个泛型参数，代码如下：

```java
package com.ivan.frame.dto.common;


public final class RequestDTO<T extends BaseDTO> extends BaseDTO {

    private static final long serialVersionUID = -2780042604928728379L;

    /**
     * 调用方的名称
     */
    private String            caller;

    /**
     * 请求参数
     */
    private T                 param;
    

 
    public String getCaller() {
        return caller;
    }

    public void setCaller(String caller) {
        this.caller = caller;
    }

    /**
     * 获取请求参数
     */
    public T getParam() {
        return param;
    }

    /**
     * 设置请求参数
     * 
     * @param param 请求参数
     */
    public void setParam(T param) {
        this.param = param;
    }

}
```

定义一个具体的业务对象， PersonDTO代码如下：

```java
package com.ivan.frame.dto;

import com.ivan.frame.dto.common.BaseDTO;

public class PersonDTO extends BaseDTO {
    
    private static final long serialVersionUID = 4637634512292751986L;
    
    private int id;
    private int age;
    private String name;
    
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    
}
```

通过JSON.parseObject传入TypeReference对象进行泛型转换，代码如下：

```java
package com.ivan.json;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
import com.ivan.frame.dto.PersonDTO;
import com.ivan.frame.dto.common.RequestDTO;

public class GenericTest {

    public static void main(String[] args) {
        RequestDTO<PersonDTO> requestDTO = new RequestDTO<PersonDTO>();
        requestDTO.setCaller("callerId");
        PersonDTO personDTO = new PersonDTO();
        personDTO.setAge(11);
        personDTO.setName("张三");
        requestDTO.setParam(personDTO);
        
        String jsonString = JSON.toJSONString(requestDTO);
        System.out.println(jsonString);
        //这行是关键代码
        requestDTO = JSON.parseObject(jsonString, new TypeReference<RequestDTO<PersonDTO>>(){});
        
        
        System.out.println(requestDTO.getParam().getName());
    }
}
```

#### fastjson各种概念

- JSON：本身是Abstract，提供了一系统的工具方法方便用户使用的API。

###### 序列化相关的概念

- SerializeConfig：内部是个map容器主要功能是配置并记录每种Java类型对应的序列化类。
- SerializeWriter 继承自Java的Writer，其实就是个转为FastJSON而生的StringBuilder，完成高性能的字符串拼接。
- SerializeFilter: 用于对对象的序列化实现各种定制化的需求。
- SerializerFeature：对于对输出的json做各种格式化的需求。
- JSONSerializer：相当于一个序列化组合器，集成了SerializeConfig， SerializeWriter ， SerializeFilter与SerializerFeature。
  序列化的入口代码如下，上面提到的各种概念都包含了

```java
    public static String toJSONString(Object object, // 
                                      SerializeConfig config, // 
                                      SerializeFilter[] filters, // 
                                      String dateFormat, //
                                      int defaultFeatures, // 
                                      SerializerFeature... features) {
        SerializeWriter out = new SerializeWriter(null, defaultFeatures, features);

        try {
            JSONSerializer serializer = new JSONSerializer(out, config);
            
            if (dateFormat != null && dateFormat.length() != 0) {
                serializer.setDateFormat(dateFormat);
                serializer.config(SerializerFeature.WriteDateUseDateFormat, true);
            }

            if (filters != null) {
                for (SerializeFilter filter : filters) {
                    serializer.addFilter(filter);
                }
            }

            serializer.write(object);

            return out.toString();
        } finally {
            out.close();
        }
    }
```

##### 反序列化相关的概念

- ParserConfig：内部通过一个map保存各种ObjectDeserializer。
- JSONLexer : 与SerializeWriter相对应，用于解析json字符串。
- JSONToken：定义了一系统的特殊字符，这些称为token。
- ParseProcess ：定制反序列化，类似于SerializeFilter。
- Feature：用于定制各种反序列化的特性。
- DefaultJSONParser：相当于反序列化组合器，集成了ParserConfig，Feature， JSONLexer 与ParseProcess。
   反序列化的入口代码如下，上面的概念基本都包含了：

```java
    @SuppressWarnings("unchecked")
    public static <T> T parseObject(String input, Type clazz, ParserConfig config, ParseProcess processor,
                                          int featureValues, Feature... features) {
        if (input == null) {
            return null;
        }

        if (features != null) {
            for (Feature feature : features) {
                featureValues |= feature.mask;
            }
        }

        DefaultJSONParser parser = new DefaultJSONParser(input, config, featureValues);

        if (processor != null) {
            if (processor instanceof ExtraTypeProvider) {
                parser.getExtraTypeProviders().add((ExtraTypeProvider) processor);
            }

            if (processor instanceof ExtraProcessor) {
                parser.getExtraProcessors().add((ExtraProcessor) processor);
            }

            if (processor instanceof FieldTypeResolver) {
                parser.setFieldTypeResolver((FieldTypeResolver) processor);
            }
        }

        T value = (T) parser.parseObject(clazz, null);

        parser.handleResovleTask(value);

        parser.close();

        return (T) value;
    }
```

#### 与Spring MVC整合

fastjson提供了FastJsonHttpMessageConverter用于将Spring mvc里的body数据(必须是json格式)转成Controller里的请求参数或者将输出的对象转成json格式的数据。spring mvc里的核心配置如下：

```xml
    <mvc:annotation-driven conversion-service="conversionService">
        <mvc:message-converters register-defaults="true">
            <bean
                class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
                <property name="features">
                    <array>
                        <value>WriteMapNullValue</value>
                        <value>WriteNullStringAsEmpty</value>
                    </array>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

这里有一个注意点，当你用Spring 3或者fastjson使用的是1.1.x的版本，在转换带有泛型参数类型的时候无法进行转换，而在Spring4配合fastjson1.2.X的版本可以解决这个问题。FastJsonHttpMessageConverter read的核心代码如下：

```java
public class FastJsonHttpMessageConverter extends AbstractHttpMessageConverter<Object>//
        implements GenericHttpMessageConverter<Object> {

//将json转成javabean的时候会调用。这里的type
    public Object read(Type type, //
                       Class<?> contextClass, //
                       HttpInputMessage inputMessage //
    ) throws IOException, HttpMessageNotReadableException {
        return readType(getType(type, contextClass), inputMessage);
    }

//这里会通过Spring4TypeResolvableHelper得到类型参数，
    protected Type getType(Type type, Class<?> contextClass) {
        if (Spring4TypeResolvableHelper.isSupport()) {
            return Spring4TypeResolvableHelper.getType(type, contextClass);
        }

        return type;
    }

}
```

#### Jackson序列化LocalDate与Springboot集成

Java8的date API一经推出便广受好评，今日也准备用一用，然后就用出问题了。

**问题**

LocalDate可以很友好的toString为`YYYY-MM-dd`的格式，很适合我当前的业务，但当我把它丢到json的时候，瞬间解体了：

```
{
  "year": 2018,
  "month": "AUGUST",
  "era": "CE",
  "dayOfMonth": 1,
  "dayOfWeek": "TUESDAY",
  "dayOfYear": 213,
  "leapYear": false,
  "monthValue": 8,
  "chronology": {
      "id":"ISO",
      "calendarType":"iso8601"
   }
}
```

可我想要的是yyyy-MM-dd啊。加上jackson format试一试，也不行。

```
@JsonFormat(shape = JsonFormat.Shape.STRING,
        pattern = "yyyy-MM-dd", timezone = "Asia/Shanghai")
```

难道要手动实现JsonSerializer? google之，果然有人解决了。

##### 解决

添加

```xml
<dependency>
  <groupId>com.fasterxml.jackson.datatype</groupId>
  <artifactId>jackson-datatype-jsr310</artifactId>
  <version>2.8.6</version>
</dependency>
```

用法

```java
@Test
public void testJsonFormat() throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JavaTimeModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

    LocalDate date = LocalDate.of(2018, 5, 5);
    String dateStr = mapper.writeValueAsString(date);
    Assert.assertEquals("\"2018-05-05\"", dateStr);

    LocalDateTime dateTime = LocalDateTime.of(2018, 5, 5, 1, 1, 1);
    Assert.assertEquals("\"2018-05-05T01:01:01\"", mapper.writeValueAsString(dateTime));
}
```

然而，在Springboot中，默认提供了ObjectMapper，我又不想自定义。

1. Springboot中使用

   同样把上述jar加入依赖。然后修改配置文件，新增

   ```
   spring:
     jackson:
       serialization:
         WRITE_DATES_AS_TIMESTAMPS: false
   ```

   这样可以直接使用LocalDate，不用单独JsonFormat就可以实现自己的功能了。

2. LocalDateTime序列化

   ```java
   @Configuration
   public class LocalDateTimeSerializerConfig {
       @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
       private String pattern;
   
       @Bean
       public LocalDateTimeSerializer localDateTimeDeserializer() {
           return new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern));
       }
   
       @Bean
       public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
           return builder -> builder.serializerByType(LocalDateTime.class, localDateTimeDeserializer());
       }
   
   }
   ```

#### FastJson配置命名规则导致spring boot admin或者 swagger-ui无法正常使用

> 由于`fastjson`配置了命名规则,导致`/actuator/metrics`接口和接口返回的数据,键值转换为下划线格式,进而导致前端页面无法正确解析.

通过修改fastjson配置,将`/actuator/metrics`和`/actuator/env`接口返回的数据配置为序列化时不进行命名格式的转换.

```java
@Bean
    public FastJsonHttpMessageConverter getFastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();

        List<MediaType> fastMediaTypes = new ArrayList<>();
        fastMediaTypes.add(MediaType.APPLICATION_JSON);
        fastMediaTypes.add(MediaType.APPLICATION_FORM_URLENCODED);
        fastMediaTypes.add(MediaType.APPLICATION_OCTET_STREAM);
        fastMediaTypes.add(MediaType.TEXT_HTML);
        fastMediaTypes.add(new MediaType("application", "vnd.spring-boot.actuator.v2+json"));
        fastConverter.setSupportedMediaTypes(fastMediaTypes);
        FastJsonConfig fastJsonConfig = geFastJsonConfig();
        fastConverter.setFastJsonConfig(fastJsonConfig);

        return fastConverter;
    }
    
    @Bean
    public FastJsonConfig geFastJsonConfig() {
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        SerializeConfig serializeConfig = new SerializeConfig();
        //增加过滤
        NameFilter nameFilter = (object, name, value) -> name;
        serializeConfig.addFilter(UiConfiguration.class, nameFilter);
        serializeConfig.addFilter(SwaggerResource.class, nameFilter);
        serializeConfig.addFilter(MetricsEndpoint.MetricResponse.class, nameFilter);
        serializeConfig.addFilter(EnvironmentEndpoint.EnvironmentDescriptor.class, nameFilter);
        serializeConfig.propertyNamingStrategy = PropertyNamingStrategy.SnakeCase;
        fastJsonConfig.setSerializeConfig(serializeConfig);
        fastJsonConfig.setDateFormat(JSON.DEFFAULT_DATE_FORMAT);
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
        return fastJsonConfig;
    }
```

`NameFilter nameFilter = (object, name, value) -> name`;表示序列化的时候将直接返回原来的键值而不做任何处理

### 参考

1. [教程](https://www.w3cschool.cn/fastjson/fastjson-serializefilter.html)