---
title: JSON框架--Gson 原理
date: 2019-05-19 14:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

### Gson原理

#### TypeAdapter

运用了JsonSerializer和JsonDeserializer进行JSON和java实体类之间的相互转化。这里利用`TypeAdapter`来更加高效的完成这个需求。

`JsonSerializer`和`JsonDeserializer`解析的时候都利用到了一个中间件-`JsonElement`，比如下方的序列化过程。可以看到我们在把Java对象转化为JSON字符串的时候都会用到这个中间件`JsonElement`

<!--more-->

![1.png](https://i.loli.net/2019/07/13/5d29de456feed11726.png)

而`TypeAdapter`的使用正是去掉了这个中间层，直接用流来解析数据，极大程度上提高了解析效率。

> New applications should prefer TypeAdapter, whose streaming API is more efficient than this interface’s tree API.
>  应用中应当尽量使用`TypeAdapter`，它流式的API相比于之前的树形解析API将会更加高效。

`TypeAdapter`作为一个抽象类提供两个抽象方法。分别是`write()`和`read()`方法,也对应着序列化和反序列化。如下图所示：

![2.png](https://i.loli.net/2019/07/13/5d29de6f61a4f37380.png)

下面就让我们来一起使用和了解TypeAdapter吧：

**实例**

`Book.java`实体类：

```java
public class Book {

  private String[] authors;
  private String isbn;
  private String title;

//为了代码简洁，这里移除getter和setter方法等
}
```

直接贴代码，具体序列化和反序列化的`TypeAdapter`类，这里是`BookTypeAdapter.java`：

```java
import java.io.IOException;

import org.apache.commons.lang3.StringUtils;

import com.google.gson.TypeAdapter;
import com.google.gson.stream.JsonReader;
import com.google.gson.stream.JsonWriter;

public class BookTypeAdapter extends TypeAdapter {

  @Override
  public Book read(final JsonReader in) throws IOException {
    final Book book = new Book();

    in.beginObject();
    while (in.hasNext()) {
      switch (in.nextName()) {
      case "isbn":
        book.setIsbn(in.nextString());
        break;
      case "title":
        book.setTitle(in.nextString());
        break;
      case "authors":
        book.setAuthors(in.nextString().split(";"));
        break;
      }
    }
    in.endObject();

    return book;
  }

  @Override
  public void write(final JsonWriter out, final Book book) throws IOException {
    out.beginObject();
    out.name("isbn").value(book.getIsbn());
    out.name("title").value(book.getTitle());
    out.name("authors").value(StringUtils.join(book.getAuthors(), ";"));
    out.endObject();
  }
}
```

同样这里设置`TypeAdapter`之后还是需要配置（注册）,可以注意到的是`gsonBuilder.registerTypeAdapter(xxx)`方法进行注册在我们之前的`JsonSerializer`和`JsonDeserializer`中也有使用：

```java
    final GsonBuilder gsonBuilder = new GsonBuilder();
    gsonBuilder.registerTypeAdapter(Book.class, new BookTypeAdapter());
    final Gson gson = gsonBuilder.create();
```

下面对两个write方法和read方法进行分别的阐述：

1. `write()`方法中会传入`JsonWriter`，和需要被序列化的`Book`对象的实例，采用和`PrintStream`类似的方式 写入到`JsonWriter`中。

   ```java
    @Override
     public void write(final JsonWriter out, final Book book) throws IOException {
       out.beginObject();
       out.name("isbn").value(book.getIsbn());
       out.name("title").value(book.getTitle());
       out.name("authors").value(StringUtils.join(book.getAuthors(), ";"));
       out.endObject();
     }
   
   ```

   下面是上面代码的步骤：

   -  `out.beginObject()`产生`{`,如果我们希望产生的是一个数组对象，对应的使用`beginArray()` 
   -  `out.name("isbn").value(book.getIsbn()); out.name("title").value(book.getTitle());`分别获取book中的isbn和title字段并且设置给Json对象中的isbn和title。也就是说上面这段代码，会在json对象中产生：

   ```
   "isbn": "978-0321336781",
    "title": "Java Puzzlers: Traps, Pitfalls, and Corner Cases",
   ```

   -  `out.name("authors").value(StringUtils.join(book.getAuthors(), ";"));`则会对应着：

   ```
    "authors": "Joshua Bloch;Neal Gafter"
   ```

   - 同理  `out.endObject()`则对应着`}` 
   - 那么整个上面的代码也就会产生JSON对象：

   ```
   {
     "isbn": "978-0321336781",
     "title": "Java Puzzlers: Traps, Pitfalls, and Corner Cases",
     "authors": "Joshua Bloch;Neal Gafter"
   }
   ```

   > 这里需要注意的是，如果没有调用 `out.endObject()`产生`}`,那么你的项目会报出 `JsonSyntaxException`错误

2. `read()`方法将会传入一个`JsonReader`对象实例并返回反序列化的对象。

   ```java
     @Override
     public Book read(final JsonReader in) throws IOException {
       final Book book = new Book();
   
       in.beginObject();
       while (in.hasNext()) {
         switch (in.nextName()) {
         case "isbn":
           book.setIsbn(in.nextString());
           break;
         case "title":
           book.setTitle(in.nextString());
           break;
         case "authors":
           book.setAuthors(in.nextString().split(";"));
           break;
         }
       }
       in.endObject();
       return book;
     }
   ```

   下面是这段代码的步骤：

   - 同样是通过`in.beginObject();`和`in.endObject();`对应解析`{`,`}` 
   - 通过

   ```java
       while (in.hasNext()) {
         switch (in.nextName()) {
         }
       }
   ```

   来完成每个`JsonElement`的遍历,并且通过`switch...case`的方法获取Json对象中的键值对。并通过我们`Book实体类`的`Setter`方法进行设置。

   ```java
       while (in.hasNext()) {
         switch (in.nextName()) {
         case "isbn":
           book.setIsbn(in.nextString());
           break;
         case "title":
           book.setTitle(in.nextString());
           break;
         case "authors":
           book.setAuthors(in.nextString().split(";"));
           break;
         }
       }
   
   ```

   同样需要注意的是,如果没有执行`in.endObject()`，将会出现`JsonIOException`的错误

   下面给出使用`TypeAdapter`的完整代码：

   ```java
   
   import java.io.IOException;
   
   import com.google.gson.Gson;
   import com.google.gson.GsonBuilder;
   
   public class Main {
     public static void main(final String[] args) throws IOException {
       final GsonBuilder gsonBuilder = new GsonBuilder();
       gsonBuilder.registerTypeAdapter(Book.class, new BookTypeAdapter());
       gsonBuilder.setPrettyPrinting();
   
       final Gson gson = gsonBuilder.create();
   
       final Book book = new Book();
       book.setAuthors(new String[] { "Joshua Bloch", "Neal Gafter" });
       book.setTitle("Java Puzzlers: Traps, Pitfalls, and Corner Cases");
       book.setIsbn("978-0321336781");
   
       final String json = gson.toJson(book);
       System.out.println("Serialised");
       System.out.println(json);
   
       final Book parsedBook = gson.fromJson(json, Book.class);
       System.out.println("\nDeserialised");
       System.out.println(parsedBook);
     }
   }
   
   ```

   对应的编译结果为：

   ```
   Serialised
   {
     "isbn": "978-0321336781",
     "title": "Java Puzzlers: Traps, Pitfalls, and Corner Cases",
     "authors": "Joshua Bloch;Neal Gafter"
   }
   
   Deserialised
   Java Puzzlers: Traps, Pitfalls, and Corner Cases [978-0321336781]
   Written by:
     >> Joshua Bloch
     >> Neal Gafter
   ```

##### TypeAdapter处理简洁的JSON数据

为了简化JSON数据，其实我们上面的JSON数据可以这么写：

```
["978-0321336781","Java Puzzlers: Traps, Pitfalls, and Corner Cases","Joshua Bloch","Neal Gafter"]
```

可以看到的是，这样采用的直接是值的形式。当然这样操作简化了JSON数据但是可能就让整个数据的稳定性下降了许多的，你需要按照一定的顺序来解析这个数据。
 对应的`write`和`read`方法如下：

```java
  @Override
  public void write(final JsonWriter out, final Book book) throws IOException {
    out.beginArray();
    out.value(book.getIsbn());
    out.value(book.getTitle());
    for (final String author : book.getAuthors()) {
      out.value(author);
    }
    out.endArray();
  }
  @Override
  public Book read(final JsonReader in) throws IOException {
    final Book book = new Book();

    in.beginArray();
    book.setIsbn(in.nextString());
    book.setTitle(in.nextString());
    final List authors = new ArrayList<>();
    while (in.hasNext()) {
      authors.add(in.nextString());
    }
    book.setAuthors(authors.toArray(new String[authors.size()]));
    in.endArray();

    return book;
  }
```

这里的解析原理和上面一致，不再赘述。

##### TypeAdapter解析内置对象

（这里将nested objects翻译为内置对象，其实就是在Book类）

这里对上面的Book实体类进行修改如下，添加Author作者类，每本书可以有多个作者。

```java

public class Book {

  private Author[] authors;
  private String isbn;
  private String title;

class Author {

  private int id;
  private String name;

//为了代码简洁，这里移除getter和setter方法等
}
//为了代码简洁，这里移除getter和setter方法等
}
```

这里提供JSON对象，

```json
{
  "isbn": "978-0321336781",
  "title": "Java Puzzlers: Traps, Pitfalls, and Corner Cases",
  "authors": [
    {
      "id": 1,
      "name": "Joshua Bloch"
    },
    {
      "id": 2,
      "name": "Neal Gafter"
    }
  ]
}
```

下面分别展示write和read方法：

```java
  @Override
  public void write(final JsonWriter out, final Book book) throws IOException {
    out.beginObject();
    out.name("isbn").value(book.getIsbn());
    out.name("title").value(book.getTitle());
    out.name("authors").beginArray();
    for (final Author author : book.getAuthors()) {
      out.beginObject();
      out.name("id").value(author.getId());
      out.name("name").value(author.getName());
      out.endObject();
    }
    out.endArray();
    out.endObject();
  }
 @Override
  public Book read(final JsonReader in) throws IOException {
    final Book book = new Book();

    in.beginObject();
    while (in.hasNext()) {
      switch (in.nextName()) {
      case "isbn":
        book.setIsbn(in.nextString());
        break;
      case "title":
        book.setTitle(in.nextString());
        break;
      case "authors":
        in.beginArray();
        final List authors = new ArrayList<>();
        while (in.hasNext()) {
          in.beginObject();
          final Author author = new Author();
          while (in.hasNext()) {
            switch (in.nextName()) {
            case "id":
              author.setId(in.nextInt());
              break;
            case "name":
              author.setName(in.nextString());
              break;
            }
          }
          authors.add(author);
          in.endObject();
        }
        book.setAuthors(authors.toArray(new Author[authors.size()]));
        in.endArray();
        break;
      }
    }
    in.endObject();

    return book;
  }
```

TypeAdapter对JSON和Java对象之间的序列化和反序列化可以通过上面的方法进行操作。其实在解决解析内置对象的序列化和反序列化的时候我们也可以通过JsonDeserializer或者JsonSerializer进行操作，序列化过程如下：

```java
@Override
  public JsonElement serialize(final Book book, final Type typeOfSrc, final JsonSerializationContext context) {
    final JsonObject jsonObject = new JsonObject();
    jsonObject.addProperty("isbn", book.getIsbn());
    jsonObject.addProperty("title", book.getTitle());

    final JsonElement jsonAuthros = context.serialize(book.getAuthors());
    jsonObject.add("authors", jsonAuthros);

    return jsonObject;
  }
```

这里通过`JsonSerializationContext`提供的context对象直接解析，一定程度上提供了JSON对象序列化（反序列化）的一致性。

#### 如何记录泛型信息

Gson是用来处理json格式文本数据的工具，提供fromJson()和toJson()的方法来实现反序列化和序列化。在反序列化的方法中，Gson提供fromJson(String json, Type typeOfT)重载方法来处理泛型。简单介绍下其使用方法，假设有如下json格式的数据A和B，A中s的属性是一个对象，B中s的属性是一个数据或集合，那么在定义反序列化的目标类型时，可以定义为如Result的类型。

json格式的数据：

```
String jsonDataA = "{\"succ\":{\"message\":\"success\"}}";
String jsonDataB = "{\"succ\":[{\"message\":\"success\"}]}";
```

反序列化的目标类型Result：

```java
public class Result<T> {
    T succ;

    public T getSucc() {
        return succ;
    }

    public void setSucc(T succ) {
        this.succ = succ;
    }
}
```

Gson使用：

```java
// jsonDataA的解析
Result<Success> result = new Gson().fromJson(jsonDataA, new TypeToken<Result<Success>>(){}.getType());
// jsonDataB的解析
Result<List<Success>> result = new Gson().fromJson(jsonDataA, new TypeToken<Result<List<Success>>>(){}.getType());
```

通过以上例子可以看到，Gson对泛型的支持使得开发人员只需要定义一个泛型类型的Result对象即可满足上述不同格式下的json数据解析。
但是我们都知道，java中的泛型类型是运行期擦除的，需要通过合适的方法来处理泛型信息。本文就讲讲gson是如何记录泛型信息的。

**TypeToken**

Gson提供TypeToken类在编译期处理泛型信息。TypeToken的构造函数是protected修饰的，可通过如下方法定义一个泛型信息为List<String\>的typeToken对象。

```java
TypeToken<List<String>> list = new TypeToken<List<String>>() {};
```

在TypeToken中，用以下3个变量用来保存泛型信息。

```java
  final Class<? super T> rawType;
  final Type type;
  final int hashCode;
```

在执行构造函数的时候，对以上3个变量进行赋值。

```java
  protected TypeToken() {
    this.type = getSuperclassTypeParameter(getClass());
    this.rawType = (Class<? super T>) $Gson$Types.getRawType(type);
    this.hashCode = type.hashCode();
  }
```

1. type

   type变量的值是由getSuperclassTypeParameter(getClass())语句获得的。

   ```java
     /**
      * Returns the type from super class's type parameter in {@link $Gson$Types#canonicalize
      * canonical form}.
      */
     static Type getSuperclassTypeParameter(Class<?> subclass) {
       // 获得带有泛型信息的父类
       Type superclass = subclass.getGenericSuperclass();
       // 如果没有泛型信息，则抛出异常
       if (superclass instanceof Class) {
         throw new RuntimeException("Missing type parameter.");
       }
       ParameterizedType parameterized = (ParameterizedType) superclass;
       // 通过$Gson$Types.canonicalize把泛型信息封装为合适的类型
       return $Gson$Types.canonicalize(parameterized.getActualTypeArguments()[0]);
     }
   ```

   在getSuperclassTypeParameter()函数中，会尝试获取带有泛型信息的父类型。如果获取到，就通过ParameterizedType.getActualTypeArguments()获得泛型的实际类型。其返回结果是一个数组，按泛型参数的顺序排列。此处TypeToken只有一个泛型参数，故取索引位置为0的值。 

   获得泛型的实际类型之后，接着调用$Gson$Types.canonicalize()方法来标准化泛型的类型。

   ```java
     /**
      * Returns a type that is functionally equal but not necessarily equal
      * according to {@link Object#equals(Object) Object.equals()}. The returned
      * type is {@link java.io.Serializable}.
      */
     public static Type canonicalize(Type type) {
       //普通的Class类型
       if (type instanceof Class) {
         Class<?> c = (Class<?>) type;
         // 如果是数组类型，则包装成GenericArrayTypeImpl对象，并对数组的元素类型调用canonicalize递归进行处理
         // 如果是普通的Class类型，则直接返回
         return c.isArray() ? new GenericArrayTypeImpl(canonicalize(c.getComponentType())) : c;
   
       } 
       // 如果是泛型类型
       else if (type instanceof ParameterizedType) {
         ParameterizedType p = (ParameterizedType) type;
         // 则包装成ParameterizedTypeImpl类型
         return new ParameterizedTypeImpl(p.getOwnerType(),
             p.getRawType(), p.getActualTypeArguments());
   
       } 
       // 如果泛型参数是数组类型
       else if (type instanceof GenericArrayType) {
         GenericArrayType g = (GenericArrayType) type;
         // 包装成GenericArrayTypeImpl，主要是进一步处理数组的元素类型
         return new GenericArrayTypeImpl(g.getGenericComponentType());
   
       } 
       // 如果泛型参数是通配符
       else if (type instanceof WildcardType) {
         WildcardType w = (WildcardType) type;
         return new WildcardTypeImpl(w.getUpperBounds(), w.getLowerBounds());
   
       } else {
         // type is either serializable as-is or unsupported
         return type;
       }
     }
   ```

   在canonicalize()方法中，对可能的Type类型进行枚举处理。对于泛型参数，可能的类型有Class、ParameterizedType、GenericArrayType、WildcardType，其他不识别的类型则默认返回。

   在处理Class的时候，要对Array和普通类型分别处理。如果是数组类型，则要对数组元素进一步处理，比如List<String\>[]形式的数组，则要对List<String\>进一步包装处理。

   在处理ParameterizedType的时候，比如List<String\>，则要进一步处理泛型类型中的泛型参数。观察Gson中自定义实现的ParameterizedTypeImpl类型。

   ```java
       public ParameterizedTypeImpl(Type ownerType, Type rawType, Type... typeArguments) {
         // require an owner type if the raw type needs it
         if (rawType instanceof Class<?>) {
           Class<?> rawTypeAsClass = (Class<?>) rawType;
           checkArgument(ownerType != null || rawTypeAsClass.getEnclosingClass() == null);
           checkArgument(ownerType == null || rawTypeAsClass.getEnclosingClass() != null);
         }
   
         // 对ownerType调用canonicalize()方法进一步处理
         this.ownerType = ownerType == null ? null : canonicalize(ownerType);
         // 对rawType调用canonicalize()方法进一步处理
         this.rawType = canonicalize(rawType);
         this.typeArguments = typeArguments.clone();
         for (int t = 0; t < this.typeArguments.length; t++) {
           checkNotNull(this.typeArguments[t]);
           checkNotPrimitive(this.typeArguments[t]);
           // 对泛型参数的实际类型调用canonicalize()方法进一步处理
           this.typeArguments[t] = canonicalize(this.typeArguments[t]);
         }
       }
   ```

   通过自定义实现的ParameterizedTypeImpl对象，则将泛型参数转化为合适的对象方便后续程序进一步处理。
   理解了ParameterizedTypeImpl的处理方法，GenericArrayTypeImpl、WildcardType的处理看上去就简单多了。

   ```java
       public GenericArrayTypeImpl(Type componentType) {
         // 进一步处理元素类型
         this.componentType = canonicalize(componentType);
       }
       
           public WildcardTypeImpl(Type[] upperBounds, Type[] lowerBounds) {
         checkArgument(lowerBounds.length <= 1);
         checkArgument(upperBounds.length == 1);
   
         if (lowerBounds.length == 1) {
           checkNotNull(lowerBounds[0]);
           checkNotPrimitive(lowerBounds[0]);
           checkArgument(upperBounds[0] == Object.class);
           // 对通配符的下限进一步处理
           this.lowerBound = canonicalize(lowerBounds[0]);
           this.upperBound = Object.class;
   
         } else {
           checkNotNull(upperBounds[0]);
           checkNotPrimitive(upperBounds[0]);
           this.lowerBound = null;
           // 对通配符的上限进一步处理
           this.upperBound = canonicalize(upperBounds[0]);
         }
       }
   ```

   以上便完成了TypeToken中type变量的赋值过程。可以看到type类型是经过转化后的自定义的Type实现类。 

2. rawType 

   rawType类型是通过以下语句进行赋值的，依赖于上文得到的type类型。

   ```
    this.rawType = (Class<? super T>) $Gson$Types.getRawType(type);
   ```

   对getRawType()函数进一步查看。

   ```java
     public static Class<?> getRawType(Type type) {
       // 如果是基本Class类型，则直接返回
       if (type instanceof Class<?>) {
         // type is a normal class.
         return (Class<?>) type;
   
       } 
       // 如果是泛型，则返回未指明泛型参数的类型
       else if (type instanceof ParameterizedType) {
         ParameterizedType parameterizedType = (ParameterizedType) type;
   
         // I'm not exactly sure why getRawType() returns Type instead of Class.
         // Neal isn't either but suspects some pathological case related
         // to nested classes exists.
         Type rawType = parameterizedType.getRawType();
         checkArgument(rawType instanceof Class);
         return (Class<?>) rawType;
   
       } 
       // 对于泛型数组，则递归处理元素类型直至返回最终的类型。
       // 比如List<String>[]，则返回的是List[].class
       else if (type instanceof GenericArrayType) {
         Type componentType = ((GenericArrayType)type).getGenericComponentType();
         return Array.newInstance(getRawType(componentType), 0).getClass();
   
       } 
       // 如果是泛型的形参，则返回Object.class
       else if (type instanceof TypeVariable) {
         // we could use the variable's bounds, but that won't work if there are multiple.
         // having a raw type that's more general than necessary is okay
         return Object.class;
   
       } 
       // 如果是通配符，则返回通配符的上界
       else if (type instanceof WildcardType) {
         return getRawType(((WildcardType) type).getUpperBounds()[0]);
   
       } else {
         String className = type == null ? "null" : type.getClass().getName();
         throw new IllegalArgumentException("Expected a Class, ParameterizedType, or "
             + "GenericArrayType, but <" + type + "> is of type " + className);
       }
     }
   ```

   至此，便赋值rawType类型。 

3. hashCode 

   对于hashCode，其赋值是type的hashCode()值。

   ```
   this.hashCode = type.hashCode();
   ```

**总结：**

在工具类实现反序列化的过程中，由于泛型有运行期擦除的特性，因此如何通过特定的方式记录下泛型信息是一个比较困难的问题。Gson通过让开发者匿名实现TypeToken类，并传递泛型参数类型，然后通过Class.getGenericSuperclass()从而获得泛型参数类型。在得到泛型参数类型之后，通过将泛型参数类型转化为合适的类型以方便后续进行处理。

#### ConstructorConstructor

在Gson反序列化过程中，通过反射生成实例对象是必不可少的。面对不同的实例类型，一系列的if else判断难免使得代码显得繁杂、冗余，而且也不符合面向对象的思想。在Gson中，通过定义ConstructorConstructor类来封装实例的创建过程。接下来直接查看ConstructorConstructor的实现。 

1. 构造函数

   在ConstructorConstructor类的构造函数中，传入Map

   ```
     public ConstructorConstructor(Map<Type, InstanceCreator<?>> instanceCreators) {
       this.instanceCreators = instanceCreators;
     }
   ```

   该变量的实现ConstructorConstructor的可扩展性。通过观察后续获得InstanceCreator的代码，发现首先会从instanceCreators变量中检索InstanceCreator对象，如果获得就直接返回。因此，通过在外部传入自定义的instanceCreators集合，可以让ConstructorConstructor返回自定义的InstanceCreator，从而实现该类功能上的可扩展性。具体细节在后续贴出的代码中可以看到。

2. get()方法

   get()方法是ConstructorConstructor对外提供的方法，通过传入一个TypeToken类型(TypeToken中存储反序列化目标类型的信息，具体在**Gson是如何记录泛型信息**的中有详细阐述)的值，可以返回相应的ObjectConstructor，其是Gson定义的获得对象实例的接口规范，只有一个方法需要实现，即construct()，返回一个新建的实例对象

   ```java
     public <T> ObjectConstructor<T> get(TypeToken<T> typeToken) {
       final Type type = typeToken.getType();
       final Class<? super T> rawType = typeToken.getRawType();
   
       // first try an instance creator
   
       // 此代码块是处理type类型
       // 根据外部传递的instanceCreators获得ObjectConstructor对象
       @SuppressWarnings("unchecked") // types must agree
       final InstanceCreator<T> typeCreator = (InstanceCreator<T>) instanceCreators.get(type);
       if (typeCreator != null) {
         return new ObjectConstructor<T>() {
           public T construct() {
             return typeCreator.createInstance(type);
           }
         };
       }
   
       // 此代码块是处理raw type类型
       // 根据外部传递的instanceCreators获得ObjectConstructor对象
       // Next try raw type match for instance creators
       @SuppressWarnings("unchecked") // types must agree
       final InstanceCreator<T> rawTypeCreator =
           (InstanceCreator<T>) instanceCreators.get(rawType);
       if (rawTypeCreator != null) {
         return new ObjectConstructor<T>() {
           public T construct() {
             return rawTypeCreator.createInstance(type);
           }
         };
       }
   
       // 按rawType类型来生成实例
       ObjectConstructor<T> defaultConstructor = newDefaultConstructor(rawType);
       if (defaultConstructor != null) {
         return defaultConstructor;
       }
   
       // rawType生成实例失败，说明rawType是接口等不能实例化的对象
       // 此时需要依次尝试去实例化
       ObjectConstructor<T> defaultImplementation = newDefaultImplementationConstructor(type, rawType);
       if (defaultImplementation != null) {
         return defaultImplementation;
       }
   
       // finally try unsafe
       return newUnsafeAllocator(type, rawType);
     }
   
   
   /**
    * 定义ObjectConstructor规范
    */
   public interface ObjectConstructor<T> {
   
     /**
      * Returns a new instance.
      */
     public T construct();
   }
   ```

   在get()方法中，newDefaultConstructor(rawType)创建普通对象的ObjectConstructor.

   ```java
     private <T> ObjectConstructor<T> newDefaultConstructor(Class<? super T> rawType) {
       try {
         //对于接口、虚类等，不存在声明的构造函数，故抛出异常
         final Constructor<? super T> constructor = rawType.getDeclaredConstructor();
         if (!constructor.isAccessible()) {
           constructor.setAccessible(true);
         }
         return new ObjectConstructor<T>() {
           @SuppressWarnings("unchecked") // T is the same raw type as is requested
           public T construct() {
             try {
               // 通过constructor.newInstance()创建实例
               Object[] args = null;
               return (T) constructor.newInstance(args);
             } catch (InstantiationException e) {
               // TODO: JsonParseException ?
               throw new RuntimeException("Failed to invoke " + constructor + " with no args", e);
             } catch (InvocationTargetException e) {
               // TODO: don't wrap if cause is unchecked!
               // TODO: JsonParseException ?
               throw new RuntimeException("Failed to invoke " + constructor + " with no args",
                   e.getTargetException());
             } catch (IllegalAccessException e) {
               throw new AssertionError(e);
             }
           }
         };
       } catch (NoSuchMethodException e) {
         return null;
       }
     }
   ```

   当面对比较复杂的类型时，newDefaultConstructor(）方法将不在满足。会进入newDefaultImplementationConstructor()方法去实现。在该方法中，罗列了几种常见的对象的类型进行处理。

   ```java
     private <T> ObjectConstructor<T> newDefaultImplementationConstructor(
         final Type type, Class<? super T> rawType) {
       // 如果是集合类型
       if (Collection.class.isAssignableFrom(rawType)) {
         // 如果是有序Set
         if (SortedSet.class.isAssignableFrom(rawType)) {
           return new ObjectConstructor<T>() {
             public T construct() {
               // 则创建TreeSet对象
               return (T) new TreeSet<Object>();
             }
           };
         } 
         // 如果是枚举类型的Set
         else if (EnumSet.class.isAssignableFrom(rawType)) {
           return new ObjectConstructor<T>() {
             @SuppressWarnings("rawtypes")
             public T construct() {
               if (type instanceof ParameterizedType) {
                 Type elementType = ((ParameterizedType) type).getActualTypeArguments()[0];
                 if (elementType instanceof Class) {
                   // 创建枚举类型的Set
                   return (T) EnumSet.noneOf((Class)elementType);
                 } else {
                   throw new JsonIOException("Invalid EnumSet type: " + type.toString());
                 }
               } else {
                 throw new JsonIOException("Invalid EnumSet type: " + type.toString());
               }
             }
           };
         } 
         //如果是普通Set
         else if (Set.class.isAssignableFrom(rawType)) {
           return new ObjectConstructor<T>() {
             public T construct() {
               // 创建LinkedHashSet
               return (T) new LinkedHashSet<Object>();
             }
           };
         } 
         // 如果是队列
         else if (Queue.class.isAssignableFrom(rawType)) {
           return new ObjectConstructor<T>() {
             public T construct() {
               // 创建链表集合
               return (T) new LinkedList<Object>();
             }
           };
         } else {
           return new ObjectConstructor<T>() {
             public T construct() {
               // 创建数组集合
               return (T) new ArrayList<Object>();
             }
           };
         }
       }
       // 如果是Map类型
       if (Map.class.isAssignableFrom(rawType)) {
         // 有序的Map
         if (SortedMap.class.isAssignableFrom(rawType)) {
           return new ObjectConstructor<T>() {
             public T construct() {
               // 则创建TreeMap
               return (T) new TreeMap<Object, Object>();
             }
           };
         } 
         // 处理key不是String的情况
         else if (type instanceof ParameterizedType && !(String.class.isAssignableFrom(
             TypeToken.get(((ParameterizedType) type).getActualTypeArguments()[0]).getRawType()))) {
           return new ObjectConstructor<T>() {
             public T construct() {
               return (T) new LinkedHashMap<Object, Object>();
             }
           };
         } 
         // 处理key是String
         else {
           return new ObjectConstructor<T>() {
             public T construct() {
               return (T) new LinkedTreeMap<String, Object>();
             }
           };
         }
       }
   
       return null;
     }
   ```

   上面这个函数是Gson用来处理几种常见复杂类型的实例创建的。在Java中，最常见的就是Collection和Map类型的。在处理Collection时，依次判断各种集合类型，有SortedSet、EnumSet、Set、Queue、List，其分别有默认的实现类；在处理Map时，有SortedMap以及Key为String和非String的情况。这几种类型已经可以处理绝大部分的Json反序列化了。可以看到，所有的集合实现类都是有序的，这意味着集合中数据存放的数据和反序列化的顺序是一致的。
   当以上类型都不能满足实例的创建时，不要忘了instanceCreators变量带来的可扩展性。

**总结：**

Gson通过ConstructorConstructor封装实例对象的创建过程，这使得逻辑代码变得简单、直观。要封装一个实例对象的创建，需要处理普通类型和复杂类型。对于普通类型，通过反射的constructor.newInstance(null)可以得到；对于负责类型，需要依次去判断尝试，并生成合适的实现类型。

#### 如何在运行时处理字段的泛型信息

上面讲述了Gson是如何通过特定的方式记录下序列化处理过程中的泛型信息。那么，在序列化的过程中，又是如何进行泛型信息的处理的？比如在处理Map类型中，该如何找到key和value的实际类型。

```
class Map<K, V> {
    K key;
    V value;
}
```

思考反序列化的过程，我们需要处理反序列化目标类型中的每一个字段。假设遇到某一字段Field，此时Gson通过$Gson$Types的resolve()函数进行处理。resolve()函数处理字段类型中的泛型信息，将其转化为实际类型。

```java
   // 反序列过程中的执行语句
   Type fieldType = $Gson$Types.resolve(type.getType(), raw, field.getGenericType());

   // resolve函数
   public static Type resolve(Type context, Class<?> contextRawType, Type toResolve) {
    while (true) {
      if (toResolve instanceof TypeVariable) {
        TypeVariable<?> typeVariable = (TypeVariable<?>) toResolve;
        toResolve = resolveTypeVariable(context, contextRawType, typeVariable);
        if (toResolve == typeVariable) {
          return toResolve;
        }

      } else if (toResolve instanceof Class && ((Class<?>) toResolve).isArray()) {
        Class<?> original = (Class<?>) toResolve;
        Type componentType = original.getComponentType();
        Type newComponentType = resolve(context, contextRawType, componentType);
        return componentType == newComponentType
            ? original
            : arrayOf(newComponentType);

      } else if (toResolve instanceof GenericArrayType) {
        GenericArrayType original = (GenericArrayType) toResolve;
        Type componentType = original.getGenericComponentType();
        Type newComponentType = resolve(context, contextRawType, componentType);
        return componentType == newComponentType
            ? original
            : arrayOf(newComponentType);

      } else if (toResolve instanceof ParameterizedType) {
        ParameterizedType original = (ParameterizedType) toResolve;
        Type ownerType = original.getOwnerType();
        Type newOwnerType = resolve(context, contextRawType, ownerType);
        boolean changed = newOwnerType != ownerType;

        Type[] args = original.getActualTypeArguments();
        for (int t = 0, length = args.length; t < length; t++) {
          Type resolvedTypeArgument = resolve(context, contextRawType, args[t]);
          if (resolvedTypeArgument != args[t]) {
            if (!changed) {
              args = args.clone();
              changed = true;
            }
            args[t] = resolvedTypeArgument;
          }
        }

        return changed
            ? newParameterizedTypeWithOwner(newOwnerType, original.getRawType(), args)
            : original;

      } else if (toResolve instanceof WildcardType) {
        WildcardType original = (WildcardType) toResolve;
        Type[] originalLowerBound = original.getLowerBounds();
        Type[] originalUpperBound = original.getUpperBounds();

        if (originalLowerBound.length == 1) {
          Type lowerBound = resolve(context, contextRawType, originalLowerBound[0]);
          if (lowerBound != originalLowerBound[0]) {
            return supertypeOf(lowerBound);
          }
        } else if (originalUpperBound.length == 1) {
          Type upperBound = resolve(context, contextRawType, originalUpperBound[0]);
          if (upperBound != originalUpperBound[0]) {
            return subtypeOf(upperBound);
          }
        }
        return original;

      } else {
        return toResolve;
      }
    }
  }
```

在上述代码中，单独一行是反序列化过程中对字段类型的处理语句。其中field.getGenericType返回字段的泛型类型。在resolve()函数，则枚举进行处理字段的各种可能类型。以下类中的各字段代表resolve()中处理的各种类型。

```java
public class Result<T> {
    // TypeVariable类型，需要找到T的实际类型
    T succ;

    // 数组类型
    Integer[] integers;

    // GenericArrayType类型
    T[] ts;

    // ParameterizedType类型
    List<T> list;

    ...
}
```

接着拆开阐述resolve中各种类型的处理过程。 

1. TypeVariable类型的处理

   ```java
         if (toResolve instanceof TypeVariable) {
           TypeVariable<?> typeVariable = (TypeVariable<?>) toResolve;
           // 处理过程
           toResolve = resolveTypeVariable(context, contextRawType, typeVariable);
           if (toResolve == typeVariable) {
             return toResolve;
           }
         } 
   ```

   在TypeVariable类型的处理过程中，resolveTypeVariable()函数封装了整个处理，用来找到TypeVariable变量的实际类型。

   ```java
     static Type resolveTypeVariable(Type context, Class<?> contextRawType, TypeVariable<?> unknown) {
       // 找到泛型变量声明的类型
       Class<?> declaredByRaw = declaringClassOf(unknown);
   
       // we can't reduce this further
       if (declaredByRaw == null) {
         return unknown;
       }
   
       // 获得带有泛型实际类型的类
       Type declaredBy = getGenericSupertype(context, contextRawType, declaredByRaw);
       if (declaredBy instanceof ParameterizedType) {
         // 查询泛型变量在declaredByRaw中声明的位置
         int index = indexOf(declaredByRaw.getTypeParameters(), unknown);
         // 根据泛型变量声明的位置返回实际类型
         return ((ParameterizedType) declaredBy).getActualTypeArguments()[index];
       }
   
       return unknown;
     }
   
     /**
      * 查找泛型变量在哪个类进行声明的
      */
     private static Class<?> declaringClassOf(TypeVariable<?> typeVariable) {
       GenericDeclaration genericDeclaration = typeVariable.getGenericDeclaration();
       return genericDeclaration instanceof Class
           ? (Class<?>) genericDeclaration
           : null;
     } 
   
     /**
      * 返回带有泛型实际类型信息的类型
      */
     static Type getGenericSupertype(Type context, Class<?> rawType, Class<?> toResolve) {
       if (toResolve == rawType) {
         return context;
       }
   
       // we skip searching through interfaces if unknown is an interface
       // 在resolveTypeVariable调用该函数之前，如果toResolve是接口，在直接返回了
       if (toResolve.isInterface()) {
         Class<?>[] interfaces = rawType.getInterfaces();
         for (int i = 0, length = interfaces.length; i < length; i++) {
           if (interfaces[i] == toResolve) {
             return rawType.getGenericInterfaces()[i];
           } else if (toResolve.isAssignableFrom(interfaces[i])) {
             return getGenericSupertype(rawType.getGenericInterfaces()[i], interfaces[i], toResolve);
           }
         }
       }
   
       // check our supertypes
       if (!rawType.isInterface()) {
         while (rawType != Object.class) {
           Class<?> rawSupertype = rawType.getSuperclass();
           if (rawSupertype == toResolve) {
             return rawType.getGenericSuperclass();
           } else if (toResolve.isAssignableFrom(rawSupertype)) {
             return getGenericSupertype(rawType.getGenericSuperclass(), rawSupertype, toResolve);
           }
           rawType = rawSupertype;
         }
       }
   
       // we can't resolve this further
       return toResolve;
     }
   ```

   从上面可以看到，resolveTypeVariable()处理泛型信息的步骤如下：
   1. 获得字段声明的类(因为字段有可能是在父类或父接口中声明的)
   2. 通过getSuperClass()和getGenericSuperclass()查找带有泛型实参的类
   3. 获得泛型形参在类中声明的索引
   4. 通过该索引在带有泛型实参的类中找到泛型的实际参数类型

2. 数组类型的处理 

   resolve()函数对数组的处理关键是找到数组参数的实际类型。

   ```java
   if (toResolve instanceof Class && ((Class<?>) toResolve).isArray()) {
           Class<?> original = (Class<?>) toResolve;
           Type componentType = original.getComponentType();
           // 获得数组元素的实际类型
           Type newComponentType = resolve(context, contextRawType, componentType);
           // 返回
           return componentType == newComponentType
               ? original
               : arrayOf(newComponentType);
   
         }
   ```

3. GenericArrayType类型的处理

   可以看到，对于泛型数组的处理的关键同样是找到泛型数组元素的实际类型。

   ```java
   if (toResolve instanceof GenericArrayType) {
           GenericArrayType original = (GenericArrayType) toResolve;
           Type componentType = original.getGenericComponentType();
            // 获得数组元素的实际类型
           Type newComponentType = resolve(context, contextRawType, componentType);
           return componentType == newComponentType
               ? original
               : arrayOf(newComponentType);
   
   }
   ```

4. ParameterizedType类型的处理

   在ParameterizedType类型处理中，有两个泛型信息需要处理，分别是ownerType和泛型参数，同样通过递归调用resolve()方法解决。

   ```java
   if (toResolve instanceof ParameterizedType) {
           ParameterizedType original = (ParameterizedType) toResolve;
           Type ownerType = original.getOwnerType();
           // 处理ownerType的泛型
           Type newOwnerType = resolve(context, contextRawType, ownerType);
           boolean changed = newOwnerType != ownerType;
   
           Type[] args = original.getActualTypeArguments();
           for (int t = 0, length = args.length; t < length; t++) {
             // 处理ParameterizedType中的泛型类型
             Type resolvedTypeArgument = resolve(context, contextRawType, args[t]);
             if (resolvedTypeArgument != args[t]) {
               if (!changed) {
                 args = args.clone();
                 changed = true;
               }
               args[t] = resolvedTypeArgument;
             }
           }
   
           // 返回类型转化之后的对象
           return changed
               ? newParameterizedTypeWithOwner(newOwnerType, original.getRawType(), args)
               : original;
   
         }
   ```

5. WildcardType类型的处理

   对于通配符类型，需要处理是通配符的上界和下界，处理方法采用递归调用resolve()，此处不再详细展开。

**总结：**

在Gson进行序列化/反序列的过程中，如何正确地找到字段定义的泛型参数类型是比较重要的一个步骤。不管是ParameterizedType、还是GenericArrayType类型，其最终需要处理的都是TypeVariable类型。在完成TypeVariable解析之后，都可以通过递归的方法解析出其他几种字段类型。TypeVariable类型解析的主要步骤如下：

1. 获得字段声明的类(因为字段有可能是在父类或父接口中声明的)
2. 通过getSuperClass()和getGenericSuperclass()查找带有泛型实参的类
3. 获得泛型形参在类中声明的索引
4. 通过该索引在带有泛型实参的类中找到泛型的实际参数类
#### 反序列化的实现原理

在分析Gson反序列化实现之前，我们先简单回顾一下json的语法。json是以键值对存储的数据格式，数据之间以逗号隔开，花括号保存对象，方括号保存数据。常见的json格式如下：

```json
{
"employer":{"name":"Jordan"},
"employees": [
{ "name":"John" },
{ "name":"Anna" },
{ "name":"Peter" }
]
}
```

json的反序列化处理，就是将上面的数据信息映射成java bean对象。对象中的每一个字段都对应json中的一个键值对。字段值的话是一个对象或是一个数组。比如上面的employer字段，其值就是一个包括人员信息的对象；employees字段，其值是包括多个人员信息的数组；在人员信息对象中，name字段的值则是一个字符串对象。

通过对json的分析，可以发现json的解析其实是一个递归过程。当对其中一个字段进行解析时，其值如果是花括号保存的对象，则递归解析该对象；其值如果是数组，则处理数组后递归解析数组中的各个值。递归的终止条件便是找到字符串表示的值。以下通过伪代码来描述整个过程：

```java
object parseJson(json) {
    if(json不是对象或数组) {
        return 格式化后的json值
    } else if(json是数组) {
        // 创建数组集合
        Collection collection = new Collection();
        for(数组中的每一条记录perChildJson : json) {
            // 递归处理每一个子json
            parseJson(perChildJson);
        }
        // 返回集合
        return collection ;
    } else {
        // 如果是json对象
        Object obj = new Object
        for(对象中的每一个键值对perChildJson : json) {
            // 递归处理键值对中的值，并返回处理后的对象
            Object value = parseJson(perChildJson.value);
            // 把处理结果赋值给当前对象
            obj.set(value);
            return obj;
        }
    }
}
```

以上代码描述json反序列化的递归处理过程。在实际处理中，会遇到各种更加复杂的问题，比如对象的创建、各种数据类型转化、集合类型的处理等。接下来就分析一下Gson是怎么来处理这些复杂问题的。

1. fromJson()

   观察Gson反序列化的入口函数，其核心代码如下：

   ```java
   public <T> T fromJson(JsonReader reader, Type typeOfT) {
       ...
       // 获得TypeToken
       TypeToken<T> typeToken = (TypeToken<T>) TypeToken.get(typeOfT);
       // 根据TypeToken获得合适的TypeAdapter
       TypeAdapter<T> typeAdapter = getAdapter(typeToken);
       // 通过TypeAdapter的read()方法来反序列化Json数据
       T object = typeAdapter.read(reader);
       return object;
   }
   ```

   可以看到，TypeAdapter是Gson中序列化/反序列化的核心工具。

2. TypeAdapter 

   TypeAdapter是Gson定义的序列化和反序列化的虚基类。其中有write()和read()方法需要实现。

   ```java
     // 序列化
     public abstract void write(JsonWriter out, T value) throws IOException;
     // 反序列化
     public abstract T read(JsonReader in) throws IOException;
   ```

   同时，TypeAdapter存在多个子类，以分别处理不同类型的json数据。在Gson中，用抽象工厂方法完成不同TypeAdapter的创建。抽象工厂则定义如下：

   ```java
   public interface TypeAdapterFactory {
     <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type);
   }
   ```

   接下来将详细分析下普通对象类型的TypeAdapter实现，以窥探整个Gson反序列的架构处理。

   - ReflectiveTypeAdapterFactory 

     在json中，以花括号来保存普通对象的信息。对于这种数据类型，Gson实现ReflectiveTypeAdapterFactory工厂来创建对应的TypeAdapter，从而处理普通对象的数据信息。工厂的create()方法如下：

     ```java
       public <T> TypeAdapter<T> create(Gson gson, final TypeToken<T> type) {
         // 获得类型信息
         Class<? super T> raw = type.getRawType();
         // 如果是基本数据类型，则返回
         if (!Object.class.isAssignableFrom(raw)) {
           return null; // it's a primitive!
         }
         // 获得ObjectConstructor
         ObjectConstructor<T> constructor = constructorConstructor.get(type);
         return new Adapter<T>(constructor, getBoundFields(gson, type, raw));
       }
     ```

     在create()中，需要关注的是ObjectConstructor的创建以及getBoundFields()方法。对于getBoundFields()方法，实现如下：

     ```java
       private Map<String, BoundField> getBoundFields(Gson context, TypeToken<?> type, Class<?> raw) {
         // 创建map对象
         Map<String, BoundField> result = new LinkedHashMap<String, BoundField>();
         if (raw.isInterface()) {
           return result;
         }
         // 在type中，包含类型信息以及泛型参数信息
         Type declaredType = type.getType();
         while (raw != Object.class) {
           // 获得当前类中所有字段
           Field[] fields = raw.getDeclaredFields();
           for (Field field : fields) {
             //　判断对象中的字段是否需要序列化（Gson支持通过expose注解来标注字段是否支持序列化和反序列）
             boolean serialize = excludeField(field, true);
             //　判断对象中的字段是否需要反序列化
             boolean deserialize = excludeField(field, false);
             if (!serialize && !deserialize) {
               continue;
             }
             field.setAccessible(true);
             // 在处理泛型对象时，需要获得泛型的实际类型
             Type fieldType = $Gson$Types.resolve(type.getType(), raw, field.getGenericType());
             // 创建BoundField对象，BoundField是对json数据操作的一个封装
             BoundField boundField = createBoundField(context, field, getFieldName(field),
                 TypeToken.get(fieldType), serialize, deserialize);
             // 加入缓存
             BoundField previous = result.put(boundField.name, boundField);
             if (previous != null) {
               throw new IllegalArgumentException(declaredType
                   + " declares multiple JSON fields named " + previous.name);
             }
           }
           type = TypeToken.get($Gson$Types.resolve(type.getType(), raw, raw.getGenericSuperclass()));
           raw = type.getRawType();
         }
         return result;
       }
     ```

     create()方法完成TypeAdapter类型实例的创建。

   - TypeAdapter 

     在处理普通类型的时候，我们需要处理类中的每一个字段，然后将字段的值通过反射赋值给当前对象。其TypeAdapter的read()实现如下：

     ```java
         @Override public T read(JsonReader in) throws IOException {
          // 校验，数据为空的时候返回null
           if (in.peek() == JsonToken.NULL) {
             in.nextNull();
             return null;
           }
           // 创建实例，constructor由ConstructorConstructor创建，封装了实例创建的过程
           T instance = constructor.construct();
     
           try {
             in.beginObject();
             //　处理json对象中的所有属性
             while (in.hasNext()) {
               // 获得属性名
               String name = in.nextName();
               // 根据属性名获得BoundField对象
               BoundField field = boundFields.get(name);
               if (field == null || !field.deserialized) {
                 in.skipValue();
               } else {
                 // 处理当前属性
                 field.read(in, instance);
               }
             }
           } catch (IllegalStateException e) {
             throw new JsonSyntaxException(e);
           } catch (IllegalAccessException e) {
             throw new AssertionError(e);
           }
           in.endObject();
           return instance;
         }
     ```

     上述的read流程主要是读取Json中的属性信息，然后根据属性信息查找到对应的BoundField，然后交由BoundField进行信息处理。在上面的ReflectiveTypeAdapterFactory工厂创建过程中，已经知道BoundField是在哪一步创建的。现在讲下BoundField创建的具体过程。

     ```java
      private ReflectiveTypeAdapterFactory.BoundField createBoundField(
           final Gson context, final Field field, final String name,
           final TypeToken<?> fieldType, boolean serialize, boolean deserialize) {
         final boolean isPrimitive = Primitives.isPrimitive(fieldType.getRawType());
     
         // special casing primitives here saves ~5% on Android...
         return new ReflectiveTypeAdapterFactory.BoundField(name, serialize, deserialize) {
           // 根据字段类型获得TypeAdapter
           final TypeAdapter<?> typeAdapter = context.getAdapter(fieldType);
           ... 省略序列化部分
     
           // 反序列化函数
           @Override void read(JsonReader reader, Object value)
               throws IOException, IllegalAccessException {
             //获得字段值
             Object fieldValue = typeAdapter.read(reader);
             //反射赋值
             if (fieldValue != null || !isPrimitive) {
               field.set(value, fieldValue);
             }
           }
         };
       }
     ```

     由于field属性可能是Collection、Map、Integer等类型，因此在BoundField中根据field的属性创建typeAdapter，并调用typeAdapter进一步反序列化属性值。可以猜想到，字段类型是基本类型的TypeAdapter是递归反序列化的终止条件。以下是Long类型的TypeAdapter，其返回的是Long类型的数值，递归反序列化在遇到Long类型后便会返回。

     ```java
       public static final TypeAdapter<Number> LONG = new TypeAdapter<Number>() {
         @Override
         public Number read(JsonReader in) throws IOException {
           if (in.peek() == JsonToken.NULL) {
             in.nextNull();
             return null;
           }
           try {
             return in.nextLong();
           } catch (NumberFormatException e) {
             throw new JsonSyntaxException(e);
           }
         }
         @Override
         public void write(JsonWriter out, Number value) throws IOException {
           out.value(value);
         }
       };
     
     ```

通过以上流程分析，可以看到Gson反序列化的实现原理如下：

1. 解析对象类型，缓存对象字段信息
2. 解析json，获得json中的键值对信息
3. 根据json中的键名寻找对象中对应的字段
4. 如果字段是非基本类型，则回到流程1处理该字段信息和json中的值，否则继续5
5. 字段是基本类型，则把值信息转化为基本类型返回
6. 反射赋值

**总结：** 

Gson反序列化的过程本质上是一个递归过程。当对其中一个字段进行解析时，其值如果是花括号保存的对象，则递归解析该对象；其值如果是数组，则处理数组后递归解析数组中的各个值。递归的终止条件是反序列的字段类型是java的基本类型信息。

### 参考

1. [GSON](https://www.jianshu.com/p/75a50aa0cad1)
2. [Google-Gson注解使用详解](https://juejin.im/post/59e5663f51882546b15b92f0)
3. [Gson基本用法](https://www.cnblogs.com/baiqiantao/p/7512336.html)
4. [Gson完全教程：基础篇](https://www.jianshu.com/p/923a9fe78108)
5. [Gson全解析（中）-TypeAdapter的使用](https://www.jianshu.com/p/8cc857583ff4)
6. [Gson的实现原理](https://blog.csdn.net/jiangjiajian2008/article/category/6530319)