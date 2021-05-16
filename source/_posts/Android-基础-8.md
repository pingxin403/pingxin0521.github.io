---
title: Android 基本控件（二）
date: 2019-04-29 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### ListView

 在Android开发中，ListView是比较常用的控件，它以列表的形式显示具体内容，并且能够根据数据的长度自适应显示。在学习ListView之前，先来一起了解AdapterView。

<!--more-->

AdapterView是一组重要的组件，AdapterView本身是一个抽象基类，它派生的子类在用法上十分相似，只是显示界面有一定的区别，因此把它们归为一类，针对它们的共性集中讲解，并突出介绍它们的区别。AdapterView主要具有以下几个特征。

- AdapterView继承了 ViewGroup，它的本质是容器。

- AdapterView可以包括多个“列表项”，并将多个“列表项”以合适的形式显示出来。

- AdapterView显示的多个“列表项”由Adapter提供。调用AdapterView的 setAdapter(Adapter)方法设置 Adapter 即可。

![3.jpeg](https://i.loli.net/2019/05/08/5cd2951e2adfc.jpeg)

从上图可以看出，AdapterView派生了三个子类：AbsListView、AbsSpinner 和 AdapterViewAnimator，这三个子类依然是抽象的，实际使用时往往采用它们的子类。

先从比较简单的子类ListView的使用方法开始学习，使用ListView主要有以下两种方式。

- 直接使用ListView进行创建。
- 让 Activity 继承 ListActivity （相当于该 Activity 显示的组件为 ListView，后续再进行学习）。

 一旦在程序中获得了 ListView之后，接下来就需要为ListView设置它要显示的列表项了。 
在这一点上，ListView显示出AdapterView的特征：通过setAdapter(Adapter)方法为之提供 
Adapter，并由Adapter提供列表项即可。

ListView提供的常用XML属性如下所示：

- android:divider：设置 List 列表项的分隔条（即可用颜色分隔，也可用 Drawable 分隔）。

- android:dividerHeight：设置分隔条的高度。

- android:entries：指定一个数组资源，Android 将根据该数组资源来生成 ListView。

- android:footerDividerEnabled：如果设置为 false，则不在 footer View 之前绘制分隔条。

- android:footerDividerEnabled：如果设置为 false，则不在 header View 之后绘制分隔条。

```
//xml文件
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <!-- 直接使用数组资源给出列表项 -->
    <!-- 设置使用蓝色的分隔条 -->
    <ListView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:entries="@array/countries"
        android:divider="#00f"
        android:dividerHeight="2px"
        android:headerDividersEnabled="false"/>
</LinearLayout>
//arrays.xml文件
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="counties">
        <item>中华人民共和国</item>
        <item>美利坚合众国</item>
        <item>俄罗斯联邦</item>
        <item>比利时王国</item>
        <item>德意志共和国</item>
    </string-array>
</resources>

```

 从上述示例程序发现，使用数组创建ListView非常简单，但这种ListView 能定制的内容很少，甚至连每个列表项的字号大小、颜色都不能改变。 如果想对ListView的外观、行为进行定制，就需要把ListView作为AdapterView使用，通过Adapter控制每个列表项的外观和行为。

### Adapter

#### MVC

 在Android开发中，比较流行的开发框架模式采用的是MVC框架模式，采用MVC模式的好处是便于UI界面部分的显示和业务逻辑，数据处理分开。那么Android项目中哪些代码来充当M、V、C角色呢？

Android 鼓励弱耦合和组件的重用，Android 中MVC的具体体现如下：

- 模型（model）：是应用程序的主题部分，所有的业务逻辑都应在该层（对数据库的操作、对网络等的操作都应该在model里面处理，当然对计算等操作也是必须放在该层的）。

- 视图层（view）：是应用程序中负责生成用户界面的部分。也是整个MVC架构中用户唯一可以看到的一层，接收用户的输入，显示用户的处理结果。一般用XML文件进行界面的描述，使用的时候可以非常方便的引入。

- 控制层（controller）：是根据用户的输入，控制用户界面数据显示及更新model对象状态的部分。Android的控制层的重任通常落在了众多Activity的肩上，这句话也就暗含了不要在Activity中写过多代码，要通过Activity交给model业务逻辑处理层处理，这样做的另外一个原因是Android中的Activity的响应时间是5秒，如果耗时的操作放在这里，程序很容易无响应。


在MVC模式中其实控制器Activity主要是起到解耦作用，将View视图和Model模型分离，虽然Activity起到交互作用，但是一般在Activity中有很多关于视图UI的显示代码，因此View视图和Activity控制器并不是完全分离的，也就是说一部分View视图和Contronller控制器Activity是绑定在一个类中的。

使用MVC模式的优点：

- 耦合性低。所谓耦合性就是模块代码之间的关联程度。利用MVC框架使得View（视图）层和Model（模型）层可以很好的分离，这样就达到了解耦的目的，所以耦合性低，减少模块代码之间的相互影响。

- 可扩展性好。由于耦合性低，添加需求，扩展代码就可以减少修改之前的代码，降低bug的出现率。

- 模块职责划分明确。主要划分层M、V、C三个模块，利于代码的维护。


什么时候适合使用MVC设计模式？当然一个小的项目且无需频繁修改需求就不用MVC框架来设计了，那样反而觉得代码过度设计，代码臃肿。一般在大型项目中，且业务逻辑处理复杂，页面显示比较多，需要模块化设计的项目使用MVC就有足够的优势了。

#### Adapter

Adapter是连接后端数据和前端显示的适配器接口，是数据和UI（View）之间一个重要的纽带。在常见的View（ListView、GridView）等地方都需要用到Adapter。

Android的适配器负责为列表组件提供数据源，也负责将单独的数据元素转换为显示在列表组件中的特定视图，如ListView的适配器关系如下图所示。

![](https://i.loli.net/2019/07/02/5d1ac118c6a2f88649.jpg)

Adapter本身只是一个接口，它派生了 ListAdapter和SpinnerAdapter两个子接口，其中 ListAdapter 为 AbsListView 提供列表项，而 SpinnerAdapter 为 AbsSpinner 提供列表项。Adapter接口及其实现类的继承关系图如下图所示。

![5.jpeg](https://i.loli.net/2019/05/08/5cd297c185fb4.jpeg)

上图中标红粗线框标出的是比较常用的Adapter。从图中可以看出几乎所有的Adapter都继承了 BaseAdapter，而BaseAdapter同时实现了 ListAdapter、SpinnerAdapter 两个接口，因此 BaseAdapter 及其子类可以同时为 AbsListView、AbsSpinner提供列表项。

Adapter的几个常用实现类如下。

- ArrayAdapter：简单、易用的Adapter，通常用于将数组或List集合的多个值包装成多个列表项。

- SimpleAdapter：并不简单、功能强大的Adapter，可用于将List集合的多个对象包装成多个列表项。

- SimpleCursorAdapter：与SimpleAdapter基本相似，只是用于包装Cursor提供的数据。

- BaseAdapter：通常用于被扩展，扩展BaseAdapter可以对各列表项进行最大限度的定制。

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <!-- 设置使用绿色的分隔条 -->
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:divider="#0f0"
        android:dividerHeight="2px"
        android:headerDividersEnabled="false"/>
</LinearLayout>
```

java文件

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;
 
public class ArrayAdapterActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.arrayadapter_layout);
 
        // 获取界面ListView组件
        ListView listView = (ListView) findViewById(R.id.listview);
 
        // 定义一个数组
        final String[] books = {"初识Android开发", "Android初级开发", "Android中级开发",
                "Android高级开发", "Android开发进阶", "Android项目实战", "Android企业级开发"};
        // 将数组包装成ArrayAdapter
        ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_1, books);
 
        // 为ListView设置Adapter
        listView.setAdapter(adapter);
 
        // 为ListView绑定列表项点击事件监听器
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long id) {
                Toast.makeText(ArrayAdapterActivity.this, "点击了" + books[position],
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

 上面的程序中前面两行粗体代码创建了一个ArrayAdapter，创建ArrayAdapter时必须指定如下三个参数。

- context：要使用的上下文环境，几乎创建所有组件都需要传入Context对象。

- resource： 要使用的视图资源 ID，该视图将作为ArrayAdapter的列表项组件。这里使用了Android系统中自带的视图资源，系统预定义的视图资源主要有以下几种：
  - ​    android.R.layout.simple_list_item_1: 单独一行的文本框。
  - ​    android.R.layout.simple_list_item_2: 两个文本框组成。
  - ​    android.R.layout.simple_list_item_checked: 每项都是由一个已选中的列表项。
  - ​    android.R.layout.simple_list_item_multiple_choice: 都带有一个复选框。
  - ​    android.R.layout.simple_list_item_single_choice: 都带有一个单选钮。
- objects：要实际显示的数组或List，将负责为多个列表项提供数据。 该数组或List包含多少个元素，就将生成多少个列表项。


上面的程序中后面几行粗体代码为ListView列表项添加点击事件监听器，当用户点击某列表项的时候，就会收到onItemClick事件，然后做消息提示或者其他需要的处理。

### ListActivity

如果程序的窗口仅仅需要显示一个列表，则可以直接让Activity继承ListActivity来实现， ListActivity的子类无须调用setContentView()方法来显示某个界面，而是可以直接传入一个内容Adapter，ListActivity的子类就呈现出一个列表。

```
 import android.app.ListActivity;
import android.os.Bundle;
import android.widget.ArrayAdapter;
 
public class MyListActivity extends ListActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        // 定义一个数组
        String[] persons = {"蜡笔小新", "皮卡丘", "蜘蛛侠", "灰太狼", "黑猫警长",
                "孙悟空", "忍者神龟", "米老鼠", "HelloKitty", "樱桃小丸子"};
 
        // 将数组包装成ArrayAdapter
        ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_single_choice, persons);
 
        // 设置该窗口显示列表
        setListAdapter(adapter);
    }
}
```

### 自定义列表项

前面学习ListView都是使用的Android系统自定义列表项资源，基本都是一些纯文本的资源，界面不够炫目，也没有办法定制。在实际开发中，列表经常包括图标、按钮等组件，这就需要开发者自定义列表项来完成了。关键是需要给适配器Adapter提供足够的数据，让Adapter能够用更丰富的View对象来填充列表的每一行。

custom_item_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

custom_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="horizontal"
              android:gravity="center_vertical">
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:padding="5dp"
        android:src="@mipmap/ic_launcher"/>
 
    <TextView
        android:id="@+id/content_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="24sp"
        android:textColor="#00f"/>
</LinearLayout>
```

这个布局使用LinearLayout作为一行，其中图标位于左侧，文本位于右侧。接下来为ListView提供Adapter，Adapter决定了ListView所要显示的列表项。

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;
 
import java.util.ArrayList;
import java.util.List;
 
public class CustomItemActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.custom_item_layout);
 
        // 获取界面ListView组件
        ListView listView = (ListView) findViewById(R.id.listview);
 
        // 定义一个List集合
        final List<String> components = new ArrayList<>();
        components.add("TextView");
        components.add("EditText");
        components.add("Button");
        components.add("CheckBox");
        components.add("RadioButton");
        components.add("ToggleButton");
        components.add("ImageView");
 
        // 将List包装成ArrayAdapter
        ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
                R.layout.custom_item, R.id.content_tv, components);
 
        // 为ListView设置Adapter
        listView.setAdapter(adapter);
 
        // 为ListView列表项绑定点击事件监听器
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long id) {
                Toast.makeText(CustomItemActivity.this, components.get(position),
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

使用了自定义列表布局R.layout.list_item。创建ArrayAdapter必须指定如下四个参数。

- context：要使用的上下文环境，几乎创建所有组件都需要传入Context对象。

- resource： 要使用的自定义列表项布局资源 ID。

- textViewResourceId：自定义列表布局中TextView的ID，该TextView组件将作为ArrayAdapter的列表项组件。

- objects：要实际显示的数组或List，将负责为多个列表项提供数据。 该数组或List包含多少个元素，就将生成多少个列表项。

 但是在这个示例中，所有的图标都是相同的，往往不能满足实际开发需求

### 自定义ArrayAdapter

如果需要每个列表项的图标根据内容动态表示，Android系统的ArrayAdapter就无能为力了，就只能使用自定义ArrayAdapter来实现啦。

做法就是创建一个ArrayAdapter的子类，重写其getView()方法，再构建不同的列表项。其中getView()方法返回的是一个View，也就是与Adapter数据对应的相应位置的行。

在学习自定义ArrayAdapter前，一起先来学习一下LayoutInflater类。在实际开发中LayoutInflater这个类还是非常有用的，它的作用类似于findViewById()。不同点是LayoutInflater是用来找res/layout/下的xml布局文件并实例化；而findViewById()是找xml布局文件下的具体widget控件（如Button、TextView等）。

LayoutInflater 是一个抽象类，获得 LayoutInflater 实例有以下三种方式。

```
// 通过Activity获取
LayoutInflaterinflater=getLayoutInflater();
 
// 通过静态方法获取
LayoutInflaterinflater=LayoutInflater.from(context);
 
// 通过系统服务获取
LayoutInflaterinflater= (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

其实这三种方式最终本质是都是调用的Context.getSystemService()，关于该方法的使用会在后续内容进行学习。

获得LayoutInflater 实例后，就可以调用inflater.inflater()方法来查找并实例化布局文件了，常用于获得ListView的每个Item布局

在上面示例的基础下更改：

新建MyArrayAdapter类，继承ArrayAdapter类，重写getView()方法

```
 
import android.app.Activity;
import android.support.annotation.NonNull;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.TextView;

public class MyArrayAdapter extends ArrayAdapter {
    private Activity mContext = null; // 上下文环境
    private int mResourceId; // 列表项布局资源ID
    private String[] mItems; // 列表内容数组
 
    public MyArrayAdapter(Activity context, int resId, String[] items){
        super(context, resId, items);
        // 保存参数
        mContext = context;
        mResourceId = resId;
        mItems = items;
    }
 
    @NonNull
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // 获取LayoutInflater对象
        LayoutInflater inflater = mContext.getLayoutInflater();
        // 装载列表项视图
        View itemView = inflater.inflate(mResourceId, null);
 
        // 获取列表项之组件
        TextView contentTv = (TextView) itemView.findViewById(R.id.content_tv);
        ImageView letterImg = (ImageView) itemView.findViewById(R.id.icon_img);
 
        // 取出要显示的数据
        String content = mItems[position].trim();
 
        // 给TextView设置显示值
        contentTv.setText(content);
        // 根据内容首字母判断要显示的图标
        if(content.startsWith("a") || content.startsWith("A")) {
            letterImg.setImageResource(R.drawable.letter_a);
        } else if(content.startsWith("b") || content.startsWith("B")) {
            letterImg.setImageResource(R.drawable.letter_b);
        } else if(content.startsWith("c") || content.startsWith("C")) {
            letterImg.setImageResource(R.drawable.letter_c);
        } else if(content.startsWith("d") || content.startsWith("D")) {
            letterImg.setImageResource(R.drawable.letter_d);
        } else if(content.startsWith("e") || content.startsWith("E")) {
            letterImg.setImageResource(R.drawable.letter_e);
        }
 
        // 返回列表项视图
        return itemView;
    }
}
```

 在上述代码中，重写了getView()方法，以便根据要显示的对象返回列表项，其中对象是用Adapter中的位置索引来表示的。

通过LayoutInflater获取到的View对象，实际上就有由列表项布局文件，包含ImageView和TextView的LinearLayout。然后找到ImageView和TextView组件，填充内容给TextView，并根据内容的首字母来判断ImageView要显示的字母图标。

将上面的Activity类更改：

```
ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,R.layout.custom_item, R.id.content_tv, components);
改为
// 将数组包装为自定义MyArrayAdapter
MyArrayAdapter adapter = new MyArrayAdapter(this,R.layout.custom_item, R.id.content_tv, components);
```

### SimpleAdapter

通过ArrayAdapter实现Adapter虽然简单、易用，但ArrayAdapter的功能比较有限，它的每个列表项只能给一个TextView动态填充内容。如果开发者需要实现更复杂的列表项，则可以考虑使用 SimpleAdapter。

在使用SimpleAdapter之前，先来一起学习SimpleAdapter的构造方法，其构造方法如下：

```
SimpleAdapter(Contextcontext,List<?extendsMap<String,?>>data,intresource,String[]from,int[]to)
```

从SimpleAdapter的构造方法可以看到，一共需要5个参数，这也是很多开发者觉得使用SimpleAdapter比较难的原因，其实就是没有很好的理解这5个参数。这个5个参数的含义如下：

- context：要使用的上下文环境。

- data：是一个`List<? extends Map<String, ?>>`类型的集合对象，该集合中每个Map<String, ?>对象生成一个列表项。

- resource：界面布局文件的ID，对应的布局文件作为列表项的组件。

- from：是一个String[]类型的参数，该参数决定提取Map<String, ?>对象中哪些key对应的value来生成列表项。

- to：该参数是一个int[]类型的参数，该参数决定填充哪些组件

```
 
 // 获取界面组件
 ListView listView = (ListView) findViewById(R.id.listview);

// 创建一个SimpleAdapter
SimpleAdapter adapter = new SimpleAdapter(this,
getData(),
R.layout.simpleadapter_item,
new String[]{"img", "title", "info"},
new int[]{R.id.icon_img, R.id.title_tv, R.id.info_tv});

// 为ListView设置Adapter
listView.setAdapter(adapter);

//getData函数

 /**
 *  创建一个List集合，其元素为Map
 * @return 返回列表项的集合对象
 */
 private List<Map<String, Object>> getData() {
 List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();

Map<String, Object> map = new HashMap<String, Object>();
map.put("img", R.drawable.item_01);
map.put("title", "小宗");
map.put("info", "电台DJ");
list.add(map);

map = new HashMap<String, Object>();
map.put("img", R.drawable.item_02);
map.put("title", "貂蝉");
map.put("info", "四大美女");
list.add(map);

map = new HashMap<String, Object>();
map.put("img", R.drawable.item_03);
map.put("title", "奶茶");
map.put("info", "清纯妹妹");
list.add(map);

map = new HashMap<String, Object>();
map.put("img", R.drawable.item_04);
map.put("title", "大黄");
map.put("info", "是小狗");
list.add(map);

map = new HashMap<String, Object>();
map.put("img", R.drawable.item_05);
map.put("title", "hello");
map.put("info", "every thing");
list.add(map);

map = new HashMap<String, Object>();
map.put("img", R.drawable.item_06);
map.put("title", "world");
map.put("info", "hello world");
list.add(map);

return list;
}

//simpleadapter_item.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="horizontal">
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:padding="5dp" />
 
    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
 
        <TextView
            android:id="@+id/title_tv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="16sp" />
 
        <TextView
            android:id="@+id/info_tv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="10sp" />
    </LinearLayout>
</LinearLayout>
```



SimpleAdapter 同样可作为 ListActivity 的内容Adapter，这样可以让用户方便地定制ListActivity所显示的列表项。

同ArrayAdapter创建ListView一样，如果需要监听用户单击、选中某个列表项的事件，则可以通过AdapterView的setOnltemClickListener()方法为单击事件添加监听器，或者通过 setOnItemSelectedListener()方法为列表项的选中事件添加监听器。

### 自定义BaseAdapter

在ListView的使用中，有时候还需要在里面加入按钮等控件，实现单独的操作。也就是说，这个ListView不再只是展示数据，也不仅仅是这一行要来处理用户的操作，而是里面的控件要获得用户的焦点。读者可以试试用SimpleAdapter添加一个按钮到ListView的条目中，会发现可以添加，但是却无法获得焦点，点击操作被ListView的Item所覆盖。这时候最方便的方法就是使用灵活的适配器BaseAdapter了。

BaseAdapter是Android应用程序中经常用到的基础数据适配器的基类，它实现了Adapter接口。其主要用途是将一组数据传到像ListView、Spinner、Gallery及GridView等UI显示组件进行显示。

由于BaseAdapter是一个抽象类，所以使用BaseAdapter时必须有一个类继承它，并实现它的方法。BaseAdapter的灵活性就在其要重写的很多方法，常会重写的几个方法如下。

- int getCount()：主要是获得列表项的数量。

- Object getItem(int position)：主要是获得当前列表项。

- long getItemId(int position)：主要是获得当前列表项的ID。

- View getView(int position, View convertView, ViewGroup parent)：主要是获得第position处的列表项组件。

在上面示例的基础上，创建Data类

```
class Data {
    private int icon; // 图标数据
    private String title; // 标题数据
    private String info; // 信息描述数据
 
    // 构造方法
    public Data() {
    }
    public Data(int icon, String title, String info) {
        this.icon = icon;
        this.title = title;
        this.info = info;
    }
 
    // Getter和Setter方法
    public int getIcon() {
        return icon;
    }
    public void setIcon(int icon) {
        this.icon = icon;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public String getInfo() {
        return info;
    }
    public void setInfo(String info) {
        this.info = info;
    }
}
```

创建MyBaseAdapter类，继承BaseAdapter类，重写其4个主要方法：

```java

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;
 
 
import java.util.List;

public class MyBaseAdapter extends BaseAdapter {
    private Context mContext; // 上下文环境
    private List<Data> mDatas; // 列表数据集合
    private int mResId; // 列表项布局文件ID
 
    // 构造方法
    public MyBaseAdapter(Context context, List<Data> datas, int resId) {
        this.mContext = context;
        this.mDatas = datas;
        this.mResId = resId;
    }
 
    // 获得列表项的数量
    @Override
    public int getCount() {
        return mDatas.size();
    }
 
    // 获得当前列表项
    @Override
    public Data getItem(int position) {
        return mDatas.get(position);
    }
 
    // 获得当前列表项的ID
    @Override
    public long getItemId(int position) {
        return position;
    }
 
    // 获得第position处的列表项组件
    @Override
    public View getView(int position, View convertView, ViewGroup viewGroup) {
        // 获取LayoutInflater对象
        LayoutInflater inflater = LayoutInflater.from(mContext);
        // 装载列表项视图
        View itemView = inflater.inflate(mResId, null);
 
        // 获取列表项组件
        ImageView iconImg = (ImageView) itemView.findViewById(R.id.icon_img);
        TextView titleTv = (TextView) itemView.findViewById(R.id.title_tv);
        TextView infoTv = (TextView) itemView.findViewById(R.id.info_tv);
 
        // 给列表项赋值
        Data data = getItem(position);
        if(null != data) {
            iconImg.setImageResource(data.getIcon());
            titleTv.setText(data.getTitle());
            infoTv.setText(data.getInfo());
        }
 
        return itemView;
    }
}
```

更改Activity中的代码

```java
// 获取界面组件
ListView listView = (ListView) findViewById(R.id.listview);

// 将数组包装为自定义MyBaseAdapter
MyBaseAdapter adapter = new MyBaseAdapter(this, getData(), R.layout.custom_baseadapter_item);

// 为ListView设置Adapter
listView.setAdapter(adapter);

/**
* 获取列表数据
* @return
*/
private List<Data> getData() {
List<Data> datas = new ArrayList<>();
datas.add(new Data(R.drawable.item_01, "小宗", "电台DJ"));
datas.add(new Data(R.drawable.item_02, "貂蝉", "四大美女"));
datas.add(new Data(R.drawable.item_03, "奶茶", "清纯妹妹"));
datas.add(new Data(R.drawable.item_04, "大黄", "是小狗"));
datas.add(new Data(R.drawable.item_05, "hello", "every thing"));
datas.add(new Data(R.drawable.item_06, "world", "hello world"));
return datas;
}

```

可以发现使用自定义BaseAdapter创建ListView，界面交互代码非常简洁，却可以实现非常复杂的界面。

### ListView优化和列表首尾使用

#### 使用convertView

前面讲的自定义ArrayAdapter和自定义BaseAdapter，都会重写getView()方法，虽然可以正常使用，但其实效率非常低。当列表项很多时，用户每次滚动屏幕，都会创建一批新的View对象，以填充新出现的列表项，这样势必会影响用户体验。

我们可以看到getView()方法中传入了一个参数convertView，可以验证该convertView的值有时候是null，有时候又不是null，特别是当用户滚动ListView的时候。其实这是适配器使用相同组件动态绑定数据的方式进行了优化，这是为何呢？

大家可以想想，如果列表项有成百上千个，Android系统会为每个列表项新建一个列表项组件吗？当然这是不可能的，毕竟Android系统的内存有限，不可能无限新建列表项组件。实际上Android缓存了视图组件，由于Android系统中有一个Recycler构件，其工作原理如下图所示。

![6.jpeg](https://i.loli.net/2019/05/08/5cd2a15261cd8.jpeg)

如果有很多个列表项，其中只有可见的列表项组件保存在内存中，其他的都在Recycler中。其实Recyler可以理解为就是一个队列，用来存储不在屏幕范围内的item，如果item完全滚粗屏幕范围，那么该item就保存在队列中；如果新的item要滚动出来，那么就会首先查看Recyler是否含有可以重复使用的View，如果有就直接重新设置该View 的数据源，然后显示出来。

其实Recycler缓存的item就是getView()方法中的参数convertView。所以会发现convertView有时候为null，有时候不为null。那么我们是否可以利用这一点来优化我们的ListView运行效率呢？答案是肯定的。

接下来就在“自定义BaseAdapter”的基础上来开始优化，除了MyBaseAdapter类的getView()方法代码会发生改变，其他不变。

```java
// 获得第position处的列表项组件
    @Override
    public View getView(int position, View convertView, ViewGroup viewGroup) {
        if(null == convertView) {
            // 获取LayoutInflater对象
            LayoutInflater inflater = LayoutInflater.from(mContext);
            // 装载列表项视图
            convertView = inflater.inflate(mResId, null);
        }
 
        // 获取列表项组件
        ImageView iconImg = (ImageView) convertView.findViewById(R.id.icon_img);
        TextView titleTv = (TextView) convertView.findViewById(R.id.title_tv);
        TextView infoTv = (TextView) convertView.findViewById(R.id.info_tv);
 
        // 给列表项赋值
        Data data = getItem(position);
        if(null != data) {
            iconImg.setImageResource(data.getIcon());
            titleTv.setText(data.getTitle());
            infoTv.setText(data.getInfo());
        }
 
        return convertView;
    }

```

经过这样的改造后，getView()方法首先检查convertView是否为空，如果是则新装填一个列表项组件，否则就重用它，就可以避免多余的装载导致的内存开销。

#### 使用持有者模式

与创建列表项组件的另一个代价较大的操作，就是调用findViewById()方法。这个方法会深入到已装填的行，根据指定的标识符取出对应的组件，便于修改列表项组件的内容，如修改TextView的文本。由于findViewById()方法可以从行所在根视图的所有子组件中找到组件，因此可能需要执行相当多的指令，而在重复取的相同组件的情况下则更是如此。

在某些GUI工具包中，可以通过在程序代码中整体性地声明复合的View对象来避免这个问题。因为在访问这个组件时，无非就是调用getter方法或访问字段。当然，在Android中也可以做到这一点，只不过代码会复杂繁琐一些。一个比较理想的方案就是，仍然使用XML布局，但是又可以缓存行中的关键子组件，也就是只需要查找一次即可，就意味着要使用持有者模式了。

在前面学习View的时候，知道每个View对象都有一个getTag()和setTag()方法，通过这两个方法可以在任何对象与组件之间建立联系。在持有者模式中，Tag标签用来保存对象，而对象又用来保存要使用的子组件。在将持有者添加到视图后，只要用到了行，就可以轻而易举的访问其子组件，而不必再调用findViewById()方法了。

接下来继续在“自定义BaseAdapter”的基础上来开始优化，除了MyBaseAdapter类中增加一个持有者类和修改getView()方法代码，其他不变。

```java
   // 获得第position处的列表项组件
    @Override
    public View getView(int position, View convertView, ViewGroup viewGroup) {
        ViewHolder holder = null;
        if(null == convertView) {
            // 获取LayoutInflater对象
            LayoutInflater inflater = LayoutInflater.from(mContext);
            // 装载列表项视图
            convertView = inflater.inflate(mResId, null);
 
            // 新建持有者ViewHolder
            holder = new ViewHolder();
            // 获取列表项组件
            holder.iconImg = (ImageView) convertView.findViewById(R.id.icon_img);
            holder.titleTv = (TextView) convertView.findViewById(R.id.title_tv);
            holder.infoTv = (TextView) convertView.findViewById(R.id.info_tv);
 
            // 将ViewHolder对象存储到convertView中
            convertView.setTag(holder);
        } else {
            // 从convertView取出ViewHolder对象
            holder = (ViewHolder) convertView.getTag();
        }
 
        // 给列表项组件设置内容
        Data data = getItem(position);
        if(null != data) {
            holder.iconImg.setImageResource(data.getIcon());
            holder.titleTv.setText(data.getTitle());
            holder.infoTv.setText(data.getInfo());
        }
 
        return convertView;
    }
 
    // 持有者类
    private class ViewHolder {
        ImageView iconImg; // 图标
        TextView titleTv; // 标题
        TextView infoTv; // 内容
    }
```

 这里ViewHolder作为持有者类，此处比较简单直接使用没有给出getter和setter方法。当convertView 为空的时候，装填一个列表项组件，并同时创建相应的ViewHolder对象；当convertView 不为空，只需要从其中取出ViewHolder对象，即可轻松给子组件填充内容。

#### 列表头和列表尾的使用

 在实际使用ListView时，经常会有这样的需求：当位于ListView最顶部的时候，显示一个搜索框可以搜索列表内容，或者显示下拉刷新；当位于ListView最底部的时候，显示一个上拉加载更多的功能。由于这显示的内容同ListView列表项内容不同，可以通过控制position来实现效果，但是非常繁琐，当然Android中提供了ListView的列表头和列表尾功能。

给ListView添加HeadView和FootView，当ListView滑动至列表第一项时使HeadView滑动出现，当ListView滑动至列表最后一项时使FootView滑动出现。

接下来就通过一个示例来学习如何使用ListView列表头和列表尾。仍然在“自定义BaseAdapter”的基础上来完成。

首先设计一个ListView列表头布局list_headview_layout.xml，主要是一个搜索框，代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <EditText
        android:id="@+id/search_et"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="搜索"
        android:padding="10dp"/>
</LinearLayout>
```

接着设计一个ListView列表尾布局list_footview_layout.xml，主要是提示用户上拉加载更多，代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:gravity="center_horizontal">
 
    <TextView
        android:id="@+id/prompt_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="10dp"
        android:text="上拉加载更多"/>
</LinearLayout>
```

最后是将上面定义的列表头布局额列表尾布局添加到ListView列表，主要修改Activity类的onCreate方法，其他不变，代码如下：

```java
// 获取界面组件
ListView listView = (ListView) findViewById(R.id.listview);

// 获取列表和列表尾
View hearderView = getLayoutInflater().inflate(R.layout.list_headview_layout, null);
View footView = getLayoutInflater().inflate(R.layout.list_footview_layout, null);

// 给ListView添加列表和列表尾
listView.addHeaderView(hearderView);
listView.addFooterView(footView);

// 将数组包装为自定义MyBaseAdapter
MyBaseAdapter adapter = new MyBaseAdapter(this, getData(), R.layout.custom_baseadapter_item);

// 为ListView设置Adapter
listView.setAdapter(adapter);

```

这里需要注意的是，给ListView添加列表和列表尾的代码必须放在设置Adapter代码之前，否则会报错。

关于列表搜索和加载的功能此处不做过多学习，后期根据需要再进行学习。

### ListView数据动态更新

updatedata_layout.xml文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical" >
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_horizontal">
 
        <Button
            android:id="@+id/add_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="添加"/>
        <Button
            android:id="@+id/update_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="更新"/>
        <Button
            android:id="@+id/delete_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="删除"/>
        <Button
            android:id="@+id/clear_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="清空"/>
    </LinearLayout>
 
    <TextView
        android:id="@+id/empty_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:padding="15dp"
        android:textSize="15sp"
        android:text="暂无数据"/>
 
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

updatedata_item.xml文件

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center_vertical"
              android:orientation="horizontal">
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="48dp"
        android:layout_height="48dp"
        android:padding="5dp"/>
 
    <TextView
        android:id="@+id/content_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp" />
</LinearLayout>
```

创建数据实体类UpdateData.java

```java

public class UpdateData {
    private int imgId;
    private String content;
 
    public UpdateData() {}
 
    public UpdateData(int imgId, String content) {
        this.imgId = imgId;
        this.content = content;
    }
 
    public int getImgId() {
        return imgId;
    }
 
    public String getContent() {
        return content;
    }
 
    public void setImgId(int imgId) {
        this.imgId = imgId;
    }
 
    public void setContent(String content) {
        this.content = content;
    }
}
```

创建MyUpdateAdapter类，继承BaseAdapter，再另外添加几个方法，便于操作ListView。

```java
 import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;
 
import com.jinyu.cqkxzsxy.android.listviewsample.R;
import com.jinyu.cqkxzsxy.android.listviewsample.entity.UpdateData;
 
import java.util.LinkedList;
import java.util.List;

public class MyUpdateAdapter extends BaseAdapter {
    private Context mContext;
    private List<UpdateData> mUpdateData;
 
    public MyUpdateAdapter() {}
    public MyUpdateAdapter( Context context, List<UpdateData> UpdateDatas) {
        this.mUpdateData = UpdateDatas;
        this.mContext = context;
    }
 
    /**
     * 添加列表项
     * @param position
     * @param UpdateData
     */
    public void add(int position, UpdateData UpdateData){
        if (null == mUpdateData) {
            mUpdateData = new LinkedList<>();
        }
        mUpdateData.add(position,UpdateData);
        notifyDataSetChanged();
    }
 
    /**
     * 更新列表内容
     * @param UpdateDatas
     */
    public void update(List<UpdateData> UpdateDatas){
        if (null == mUpdateData) {
            mUpdateData = new LinkedList<>();
        }
        mUpdateData.clear();
        mUpdateData.addAll(UpdateDatas);
 
        notifyDataSetChanged();
    }
 
    /**
     * 更新列表项
     * @param position
     * @param UpdateData
     */
    public void update(int position,UpdateData UpdateData){
        if(mUpdateData != null && position < mUpdateData.size()) {
            mUpdateData.set(position, UpdateData);
        }
        notifyDataSetChanged();
    }
 
    /**
     * 移除指定列表项
     * @param position
     */
    public void remove(int position) {
        if(mUpdateData != null && 0 != getCount()) {
            mUpdateData.remove(position);
        }
        notifyDataSetChanged();
    }
 
    /**
     * 清空列表数据
     */
    public void clear() {
        if(mUpdateData != null) {
            mUpdateData.clear();
        }
        notifyDataSetChanged();
    }
 
    @Override
    public int getCount() {
        return mUpdateData.size();
    }
 
    @Override
    public UpdateData getItem(int position) {
        return mUpdateData.get(position);
    }
 
    @Override
    public long getItemId(int position) {
        return position;
    }
 
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        if(null == convertView){
            convertView = LayoutInflater.from(mContext).inflate(R.layout.updatedata_item, null);
            holder = new ViewHolder();
            holder.img_icon = (ImageView) convertView.findViewById(R.id.icon_img);
            holder.txt_content = (TextView) convertView.findViewById(R.id.content_tv);
            convertView.setTag(holder);
        }else{
            holder = (ViewHolder) convertView.getTag();
        }
 
        UpdateData UpdateData = getItem(position);
        holder.img_icon.setImageResource(UpdateData.getImgId());
        holder.txt_content.setText(UpdateData.getContent());
        return convertView;
    }
 
    private class ViewHolder{
        ImageView img_icon;
        TextView txt_content;
    }
}
```

 接下来为ListView提供Adapter，使用自定义的BaseAdapter决定ListView所要显示的列表项，然后为4个按钮设置监听监听器。新建UpdateDataActivity.java文件，加载上面新建的布局文件，具体代码如下：

```java
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.ListView;
 
import com.jinyu.cqkxzsxy.android.listviewsample.adapter.MyUpdateAdapter;
import com.jinyu.cqkxzsxy.android.listviewsample.entity.UpdateData;
 
import java.util.LinkedList;
import java.util.List;
 
public class UpdateDataActivity extends AppCompatActivity implements View.OnClickListener {
    private ListView mListView = null; // 列表
    private Button mAddBtn = null; // 添加列表项按钮
    private Button mUpdateBtn = null; // 更新列表按钮
    private Button mDeleteBtn = null; // 删除列表项按钮
    private Button mClearBtn = null; // 清空列表数据
    private MyUpdateAdapter mAdapter = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.updatedata_layout);
 
        // 获取界面组件
        mListView = (ListView) findViewById(R.id.listview);
        mAddBtn = (Button) findViewById(R.id.add_btn);
        mUpdateBtn = (Button) findViewById(R.id.update_btn);
        mDeleteBtn = (Button) findViewById(R.id.delete_btn);
        mClearBtn = (Button) findViewById(R.id.clear_btn);
 
        // 添加列表控内容视图
        View emptyView = findViewById(R.id.empty_tv);
        mListView.setEmptyView(emptyView);
 
        // 初始化列表
        List<UpdateData> datas = new LinkedList<UpdateData>();
        mAdapter = new MyUpdateAdapter(this, datas);
        mListView.setAdapter(mAdapter);
 
        // 设置按钮点击事件监听器
        mAddBtn.setOnClickListener(this);
        mUpdateBtn.setOnClickListener(this);
        mDeleteBtn.setOnClickListener(this);
        mClearBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.add_btn:
                addItem();
                break;
            case R.id.update_btn:
                updateData();
                break;
            case R.id.delete_btn:
                deleteItem();
                break;
            case R.id.clear_btn:
                clearData();
                break;
            default:
                break;
        }
    }
 
    /**
     * 添加列表项
     */
    private void addItem() {
        int position = getRandomPosition();
        mAdapter.add(position, new UpdateData(R.mipmap.ic_launcher, "随机添加" + position));
    }
 
    /**
     * 更新列表内容
     */
    private void updateData() {
        int position = getRandomPosition();
        mAdapter.update(position, new UpdateData(R.mipmap.ic_launcher, "更新" + getRandomNumber()));
    }
 
    /**
     * 删除列表项
     */
    private void deleteItem() {
        int position = getRandomPosition();
        mAdapter.remove(position);
    }
 
    /**
     * 清空列表数据
     */
    private void clearData(){
        mAdapter.clear();
    }
 
    /**
     * 获取列表随机位置
     * @return
     */
    private int getRandomPosition() {
        int count = mAdapter.getCount();
        return (int) (Math.random() * count);
    }
 
    /**
     * 获取100以内的随机数
     * @return
     */
    private double getRandomNumber(){
        return Math.random() * 100;
    }
}
```

修改启动的Activity，运行程序，然后点击添加按钮，在列表中随机添加一些列表项，可以看到列表数据动态更新，然后再点击更新按钮，可以随机更新列表数据，再点击删除按钮，可以看到将会从列表中删除随机列表项，点击清空按钮，可以将列表所有数据全部清空，显示启动时的页面。

从以上几个操作，可以看到动态更新时离不开每次调用notifyDataSetChanged()方法，这个方法的主要作用就是当适配器里面的内容发生改变时需要强制调用getView()方法来刷新每个Item的内容。



### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)