---
title: Android 用户界面状态保存
date: 2019-04-27 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### 用户界面状态保存

一个Activity被激活并运行，即为建立了一个Activity的实例。

Activity实例与用户交互，产生界面状态信息。如用户所选取的值，光标的位置等。

当Activity实例进入“暂停”功“停止”状态时，需要保存这些临时的状态信息。保存状态信息的方法：SharedPreferences对象、Bundle对象。
<!--more-->
#### SharedPreferences对象

SharedPreferences存储方式是Android中存储轻量级数据的一种方式。SharedPreferences存储主要用来存储一些简单的配置信息，内部以Map方式进行存储，因此需要使用键值对提交和保存数据，保存的数据以xml格式存放在本地的`/data/data/<package name>/shares_prefs`文件夹下。

1. 使用简单，便于存储轻量级的数据；
2. 只支持Java基本数据类型，不支持自定义数据类型；
3. SharedPreferences是单例对象，在整个应用内数据共享，无法在其他应用内共享数据；

SharedPreferences创建的时候使用的文件名不同，得到的对象不同，在存储位置会创建多个xml文件，不同文件名的SharedPreferences的数据不会共享；创建时采用相同的文件名，得到多个SharedPreferences引用，此时这多个引用共享同一个xml文件，它们操作的数据为相同的数据；

针对数据的增删改查操作只有在提交后操作才能生效，此步骤最容易被忽略，当执行了对数据的操作后得不到需要的效果时，请查看是否没有提交操作。

三种创建模式中最好采用MODE_PRIVATE，使用其他两种模式创建容易引起安全问题，就算采用MODE_PRIVATE，当别人活得root权限后也可能泄露用户的重要信息，因此建议使用SharedPreferences时，如果要存储用户名和密码时，不要明文存储，应该使用加密存储，防止重要隐私泄露，引起损失。

1.  **获得SharedPreferences对象**

   SharedPreferences对象必须使用上下文获得，使用时注意先要获得上下文。获得SharedPreferences对象方法为：

   ```
   SharedPreferences sharedPreferences = getSharedPreferences(参数一, 参数二);
   ```

   参数一为要保存的xml文件名，不同的文件名产生的对象不同，但同一文件名可以产生多个引用，从而可以保证数据共享。此处注意指定参数一时，不用加xml后缀，由系统自动添加。

   参数二为创建模式，共有三个值：

   ```
   MODE_PRIVATE: 私有方式存储,其他应用无法访问（默认方式）
   MODE_WORLD_READABLE: 表示当前文件可以被其他应用读取;不是一个安全的模式，不建议使用。再说，跨进程通信也不会使用SP。
   MODE_WORLD_WRITEABLE: 表示当前文件可以被其他应用写入
   ```

   第一个值使得SharedPreferences存储的数据只能在本应用内获得，第二个和第三个值分别使得其他应用可以读和读写本应用SharedPreferences存储的数据。由此可能带来安全问题，请参考本文二(3)部分。

2. **获得editor对象**
    使用以上获得的SharedPreferences对象产生editor，方法为：
  ```
  Editor editor = sharedPreferences.edit();
  ```
3. **对数据实现增删改查**
    添加数据使用以下方法：
  ```
  editor.putString(key, value);
  ```
  可以实现数据的更新只需添加同键的键值对，和操作Map集合一样。

  删除数据：
  ```
  editor.remove(key);
  ```
  删除参数部分键的键值对。

  查询数据：
  ```
  String result = sharedPreferences.getString(key1, key2);
  ```
  Key1是要查询的键，返回对应的值，当键不存在时，返回key2作为结果。

  清空数据
  ```
  editor.clear();
  ```
  注意：无论是否同一个sharedPreferences对象，若是产生多个editor，不同的editor之间对数据的操作不会相互影响，此处容易犯错误，例如,以下的程序：
  ```
  sharedPreferences.edit().putString(key1, value1);
  sharedPreferences.edit().putString(key2, value2);
  sharedPreferences.edit().putString(key3, value3);
  sharedPreferences.edit().commit();
  ```
  执行后这种方式无法存储，因为sp.edit()每次都会返回一个新的Editor对象，Editor的实现类EditorImpl里面会有一个缓存的Map，最后commit的时候先将缓存里面的Map写入内存中的Map，然后将内存中的Map写进XML文件中。使用上面的方式commit，由于sp.edit()又重新返回了一个新的Editor对象，缓存中的Map是空的，所以导致数据无法被存储。这里即需要注意，只有提交时，不同的editor无法完成既定的任务。而增加，删除，更新，查询，若是同一个sharedPreferences产生的editor，则对共享的数据有效，会按照既定的顺序工作。而不同sharedPreferences产生的editor则因为处理的xml文件不同而不能共享数据。

  再次强调：**所有使用editor操作的数据，必须经过同一个editor提交即commit后才能生效。**
4. 文件生成的时机
    SP有两个方法可以提交数据：
  ```
  editor.commit();//效率低，线程安全
  editor.apply();//效率高，线程不安全
  ```
  以上两个方法执行时，如果没有文件，则创建文件。

  区别：

  - apply没有返回值而commit返回boolean表明修改是否提交成功
  - apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。
  - apply方法不会提示任何失败的提示。
    由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。

**获取SP对象的三种方法**

（1）基于Context，需指定文件名和读写模式
```
SharedPreferences sharedPreferences = getSharedPreferences("wenjianming", Context.MODE_PRIVATE);//创建SP对象，指定文件名为wenjianming和指定读写模式为私有模式
```

（2）基于Context，获取默认SP
```
SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
```
默认SP的文件名和读写模式可以从源码中可以看出
```
public static String getDefaultSharedPreferencesName(Context context) {
    return context.getPackageName() + "_preferences";
}

private static int getDefaultSharedPreferencesMode() {
    return Context.MODE_PRIVATE;
}
```
（3）基于Activity获取SP对象，只需要指定模式
```
SharedPreferences sharedPreferences = getPreferences(Context.MODE_PRIVATE);
```
文件的命名方式可以从源码中可以看出
```
@NonNull
public String getLocalClassName() {
    final String pkg = getPackageName();
    final String cls = mComponent.getClassName();
    int packageLen = pkg.length();
    if (!cls.startsWith(pkg) || cls.length() <= packageLen
            || cls.charAt(packageLen) != '.') {
        return cls;
    }
    return cls.substring(packageLen+1);
}
```

#### 示例
养成良好的封装习惯

```
import android.content.Context;
import android.content.SharedPreferences;

public class SharePreferencesUtils {

    private SharedPreferences sharedPreferences;

    /**
     * 构造方法
     * @param context
     * @param fileName
     */
    public SharePreferencesUtils(Context context, String fileName) {
        sharedPreferences = context.getSharedPreferences(fileName, Context.MODE_PRIVATE);
    }

    /**
     * 创建一个内部类使用，里面有key和value这两个值
     */
    public static class ContentValue {
        String key;
        Object value;

        public ContentValue(String key, Object value) {
            this.key = key;
            this.value = value;
        }
    }

    /**
     * 传值
     * @param contentValues
     */
    public void putValues(ContentValue... contentValues) {

        SharedPreferences.Editor editor = sharedPreferences.edit();

        for (ContentValue contentValue : contentValues) {
            //如果是字符型类型
            if (contentValue.value instanceof String) {
                editor.putString(contentValue.key, contentValue.value.toString()).commit();
            }
            //如果是int类型
            if (contentValue.value instanceof Integer) {
                editor.putInt(contentValue.key, Integer.parseInt(contentValue.value.toString())).commit();
            }
            //如果是Long类型
            if (contentValue.value instanceof Long) {
                editor.putLong(contentValue.key, Long.parseLong(contentValue.value.toString())).commit();
            }
            //如果是布尔类型
            if (contentValue.value instanceof Boolean) {
                editor.putBoolean(contentValue.key, Boolean.parseBoolean(contentValue.value.toString())).commit();
            }
        }
    }


    public String getString(String key) {
        return sharedPreferences.getString(key, null);
    }

    public boolean getBoolean(String key) {
        return sharedPreferences.getBoolean(key, false);
    }

    public int getInt(String key) {
        return sharedPreferences.getInt(key, -1);
    }

    public long getLong(String key) {
        return sharedPreferences.getLong(key, -1);
    }

    //清除当前文件的所有的数据
    public void clear() {
        sharedPreferences.edit().clear().commit();
    }

}
```
SP是一个轻量级的存储类，因为考虑到内存问题，key和value字符不能太长，字符越长，文件占用的存储空间越大，读取配置文件的时间太长，占用的内存过大，导致手机卡顿。

### Bundle

Bundle主要用于传递数据；它保存的数据，是以key-value(键值对)的形式存在的。

我们经常使用Bundle在Activity之间传递数据，传递的数据可以是boolean、byte、int、long、float、double、string等基本类型或它们对应的数组，也可以是对象或对象数组。当Bundle传递的是对象或对象数组时，必须实现Serializable 或Parcelable接口。下面分别介绍Activity之间如何传递基本类型、传递对象。

Bundle提供了各种putXxx()/getXxx()方法，用于读写基本数据类型，Bundle用于读写基本数据类型的API有：

![1.png](https://i.loli.net/2019/05/07/5cd13c0bce396.png)

写数据的方法：
```
Bundle bundle=new Bundle();
bundle.putString("name","police");
bundle.putInt("years",8);
final Intent intent=new Intent().setClassName("police.myapp","police.myapp.Main2Activity");
intent.putExtras(bundle);
startActivity(intent);
```
执行后将bundle绑定到intent，传递到Mian2Activity

读数据的方法：
(Intent.getExtras()获取bundle对象)
```
Bundle bundle=this.getIntent().getExtras();
String bundleString=bundle.getString("name");
int bundleInt=bundle.getInt("years");
textView.setText(bundleString+bundleInt);
```


**Bundle与SharedPreferences的区别**

- SharedPreferences是简单的存储持久化的设置，它只是一些简单的键值对存储方式。它将数据保存在一个xml文件中。
- Bundle是将数据传递到另一个上下文中或保存或回复你自己状态的数据存储方式。它的数据不是持久化存储状态。

#### 传递Parcelable类型的对象

Parcelable是Android自定义的一个接口，它包括将数据写入Parcel和从Parcel中读出的API。
一个实体（用类来表示），如果需要封装到Bundle中去，可以通过实现Parcelable接口来完成。

![1.jpg](https://i.loli.net/2019/05/07/5cd13ccaa6433.jpg)

**Parcelable接口说明**
```
public interface Parcelable {  
  //内容描述接口，基本不用管  
  public int describeContents();  
  //写入接口函数，打包  
  public void writeToParcel(Parcel dest, int flags);  
  //读取接口，目的是要从Parcel中构造一个实现了Parcelable的类的实例处理。因为实现类在这里还是不可知的，所以需要用到模板的方式，继承类名通过模板参数传入。  
  //为了能够实现模板参数的传入，这里定义Creator嵌入接口,内含两个接口函数分别返回单个和多个继承类实例。  
  public interface Creator<T> {  
      public T createFromParcel(Parcel source);  
      public T[] newArray(int size);  
}  
}  
```
**Parcelable接口的实现方法**

从parcelable接口定义中，我们可以看到，实现parcelable接口，需要我们实现下面几个方法：
1. describeContents方法。内容接口描述，默认返回0就可以;

2. writeToParcel 方法。该方法将类的数据写入外部提供的Parcel中.即打包需要传递的数据到Parcel容器保存，以便从parcel容器获取数据，该方法声明如下：writeToParcel(Parcel dest, int flags)

3. 静态的Parcelable.Creator接口，本接口有两个方法：
    createFromParcel(Parcelin)  从Parcel容器中读取传递数据值，封装成Parcelable对象返回逻辑层。
    newArray(int size) 创建一个类型为T，长度为size的数组，仅一句话（returnnew T[size])即可。方法是供外部类反序列化本类数组使用。


#### 传递Serializable类型的对象
 Serializable是一个对象序列化的接口。一个类只有实现了Serializable接口，它的对象才是可序列化的。因此如果要序列化某些类的对象，这些类就必须实现Serializable接口。而实际上，Serializable是一个空接口，没有什么具体内容，它的目的只是简单的标识一个类的对象可以被序列化。

很简单，只要implements Serializable接口就可以了
