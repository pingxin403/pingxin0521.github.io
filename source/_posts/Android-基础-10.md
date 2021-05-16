---
title: Android 基本控件（三） 
date: 2019-04-30 16:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### GridView

前面学的ListView是列表， 这里的GridView就是显示网格，用于在界面上按行、列分布的方式来显示多个组件。

<!--more-->

GridView 和 ListView 有共同的父类：AbsListView，因此 GridView和ListView具有很高的相似性，它们都是列表项。GridView与ListView的唯一区别在于：ListView只显示一列；而GridView可以显示多列。从这个角度来看，ListView相当于一种特殊的GridView，如果让 GridView只显示一列，那么该GridView就变成了 ListView。 与ListView类似的是，GridView也需要通过Adapter来提供显示的数据：开发者可以采用上面介绍的几种方式中的任意一种来创建Adapter。不管使用哪种方式，GridView与ListView 的用法是基本一致的。 

 GridView提供的常用XML属性及相关方法如下表所示：

![1.jpeg](https://i.loli.net/2019/05/09/5cd40913be8ee.jpeg)

上表中android:stretchMode属性支持如下几个属性值。

- NO_STRETCH：不拉伸。

- STRETCH_SPACING:仅拉伸元素之间的间距。

- STRETCH_SPACING_UNIFORM：表格元素本身、元素之间的间距一起拉伸。

- STRETCH_COLUMN_WIDTH：仅拉伸表格元素本身。


另外需要注意的是使用GridView时一般都应该指定numColumns大于1；否则该属性的默认值为1。如果将该属性设为1，则意味着该GridView只有一列，那么GridView 就变成了 ListView。

gridview_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center_horizontal">
 
    <!-- 定义一个GridView组件 -->
    <GridView
        android:id="@+id/gridview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:horizontalSpacing="1dp"
        android:verticalSpacing="1dp"
        android:numColumns="4"
        android:gravity="center"/>
</LinearLayout>
```

定义GridView时指定了 android:numColumns="4"，这意味着该网格包含4列。该GridView包含的行是动态改变的——正如ListView到底包含多少行是由该ListView对应的Adapter所决定的，GridView到底包含多少行也是由Adapter决定的。

gridview_item.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center_horizontal"
              android:orientation="vertical"
              android:padding="5dp" >
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="48dp"
        android:layout_height="48dp" />
 
    <TextView
        android:id="@+id/name_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="5dp"/>
</LinearLayout>
```

接下来为GridView提供Adapter，具体实现方式有多种，这里使用SimpleAdapter决定GridView所要显示的内容。新建GridViewActivity.java文件，加载上面新建的布局文件，具体代码如下

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.AdapterView;
import android.widget.GridView;
import android.widget.SimpleAdapter;
import android.widget.Toast;
 
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
 
public class GridViewActivity extends AppCompatActivity {
    private GridView mAppGridView = null;
    // 应用图标
    private int[] mAppIcons = {
            R.drawable.app_1, R.drawable.app_2, R.drawable.app_3,
            R.drawable.app_4, R.drawable.app_5, R.drawable.app_6,
            R.drawable.app_7, R.drawable.app_8, R.drawable.app_9
    };
    // 应用名
    private String[] mAppNames = {
            "魔法棒", "点赞社群", "购物街区","蚂蚁社区","鑫鱻地图",
            "鑫鱻消息", "房品汇","商城","模型盒子"
    };
 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.gridview_layout);
 
        // 获取界面组件
        mAppGridView = (GridView) findViewById(R.id.gridview);
 
        // 初始化数据，创建一个List对象，List对象的元素是Map
        List<Map<String, Object>> listItems =
                new ArrayList<Map<String, Object>>();
        for (int i = 0; i < mAppIcons.length; i++) {
            Map<String, Object> listItem = new HashMap<String, Object>();
            listItem.put("icon", mAppIcons[i]);
            listItem.put("name",mAppNames[i]);
            listItems.add(listItem);
        }
        // 创建一个SimpleAdapter
        SimpleAdapter simpleAdapter = new SimpleAdapter(this,
                listItems,
                R.layout.gridview_item,
                new String[]{"icon", "name"},
                new int[]{R.id.icon_img, R.id.name_tv});
 
        // 为GridView设置Adapter
        mAppGridView.setAdapter(simpleAdapter);
 
        // 添加列表项被单击的监听器
        mAppGridView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                // 显示被单击的图片
                Toast.makeText(GridViewActivity.this, mAppNames[position],
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

上面的程序同样使用了 SimpleAdapter对象作为GridView的Adapter，这个SimpleAdapter底层保证了一个长度为9的List集合这意味着该GridView—共需要显示9个组件，GridView总共有4 列，因此该GridView —共包含3行。

单击界面中的图标，可以看到消息提示。

### 下拉框Spinner

Spinner其实就是一个列表选择框。不过Android的列表选择框并不需要显示下拉列表，而是相当于弹出一个菜单供用户选择。

Spinner 与 Gallery 都继承了AbsSpinner，AbsSpinner 继承了AdapterView，因此它也表现出AdapterView的特征：只要为AdapterView提供Adapter即可。

Spinner支持的常用XML属性及相关方法如下表所示：

![1.jpeg](https://i.loli.net/2019/05/09/5cd40b2f89985.jpeg)

如果开发者使用Spinner时己经可以确定列表选择框里的列表项，则完全不需要编写代码，只要为Spinner指定android:entries属性即可让Spinner正常工作；如果程序需要在运行时动态 地决定Spinner的列表项，或者程序需要对Spinner的列表项进行定制，则可使用Adapter为 Spinner提供列表项

spinner_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"
              android:padding="5dp" >
 
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="选择专业方向"
        android:textColor="#44BDED"
        android:textSize="18sp" />
 
    <Spinner
        android:id="@+id/spin_one"
        android:layout_width="100dp"
        android:layout_height="64dp"
        android:entries="@array/professionals"
        android:spinnerMode="dialog" />
 
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:text="选择教科书"
        android:textColor="#F5684A"
        android:textSize="18sp" />
 
    <Spinner
        android:id="@+id/spin_two"
        android:layout_width="wrap_content"
        android:layout_height="64dp" />
</LinearLayout>
```

在res/values/目录下新建arrays.xml文件，定义professionals数组资源，如下：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="professionals">
        <item>Android</item>
        <item>Java</item>
        <item>Python</item>
        <item>PHP</item>
        <item>.Net</item>
        <item>C++</item>
        <item>C</item>
    </string-array>
</resources>
```

 接下来为Spinner提供Adapter。新建SpinnerActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Spinner;
import android.widget.Toast;
 
public class SpinnerActivity extends AppCompatActivity
        implements AdapterView.OnItemSelectedListener {
    private Spinner mProSpinner = null;
    private Spinner mBookSpinner = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.spinner_layout);
 
        // 获取界面布局文件中的Spinner组件
        mProSpinner = (Spinner) findViewById(R.id.spin_one);
        mBookSpinner = (Spinner) findViewById(R.id.spin_two);
 
        String[] arr = { "初识Android开发", "Android初识开发", "Android中级开发",
                "Android高级开发", "Android开发进阶"};
        // 创建ArrayAdapter对象
        ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_1, arr);
        // 为Spinner设置Adapter
        mBookSpinner.setAdapter(adapter);
 
        // 为Spinner设置选中事件监听器
        mProSpinner.setOnItemSelectedListener(this);
        mBookSpinner.setOnItemSelectedListener(this);
    }
 
    @Override
    public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
        String content = parent.getItemAtPosition(position).toString();
        switch (parent.getId()){
            case R.id.spin_one:
                Toast.makeText(SpinnerActivity.this, "选择的专业是：" + content,
                        Toast.LENGTH_SHORT).show();
                break;
            case R.id.spin_two:
                Toast.makeText(SpinnerActivity.this, "选择的教材是：" + content,
                        Toast.LENGTH_SHORT).show();
                break;
            default:
                break;
        }
    }
 
    @Override
    public void onNothingSelected(AdapterView<?> adapterView) {
    }
}
```



Gallery与Spinner组件有共同的父类：AbsSpinner，表明Gallery和Spinner都是一个列表选择框。它们之间的区别在于，Spinner显示的是一个垂直的列表选择框，而Gallery显示的是一个水平的列表选择框。 Gallery与Spinner还有一个区别：Spinner的作用是供用户选择，而Gallery则允许用户通过拖动来查看上一个、下一个列表项。

Gallery本身的用法非常简单——基本上与Spinner的用法相似，只要为它提供一个内容 Adapter即可，该Adapter的getView()方法所返回的View将作为Gallery列表的列表项。如果程序需要监控到Gallery选择项的改变，通过为Gallery添加OnltemSelectedListener监听器即可实现。

Android已经不再推荐使用Gallery组件，而是推荐使用其他水平滚动组件，如HorizontalScrollView和ViewPager来代替Gallery组件，所以此处不做过多讲解。

### 自动完成文本框AutoCompleteTextView

 AutoCompleteTextView是自动完成文本框，从EditText派生而出，实际上它也是一个文本编辑框，但它比普通编辑框多了一个功能：当用户输入一定字符之后，自动完成文本框会显示一个下拉菜单，供用户从中选择，当用户选择某个菜单项之后，AutoCompleteTextView按用户选择自动填写该文本框。

AutoCompleteTextView除了可使用EditText提供的XML属性和方法之外，还支持如下表所示的常用XML属性及相关方法：

![1.jpeg](https://i.loli.net/2019/05/09/5cd40c48bd59e.jpeg)

使用AutoCompleteTextView很简单，只要为它设置一个Adapter即可，该Adapter封装了 AutoCompleteTextView预设的提示文本。

AutoCompleteTextView还派生了一个子类：MultiAutoCompleteTextView,该子类的功能与 AutoCompleteTextView基本相似，只是MultiAutoCompleteTextView允许输入多个提示项，多个提示项以分隔符分隔。MultiAutoCompleteTextView提供了 setTokenizer()方法来设置分隔符。

autocomplete_textview_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <!-- 定义一个自动完成文本框，指定输入一个字符后进行提示 -->
    <AutoCompleteTextView
        android:id="@+id/auto_actv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="输入联系人姓名"
        android:completionHint="选择联系人"
        android:dropDownHorizontalOffset="10dp"
        android:completionThreshold="1"/>
 
    <!-- 定义一个MultiAutoCompleteTextView组件 -->
    <MultiAutoCompleteTextView
        android:id="@+id/mauto_mactv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="输入联系人姓名，可多个分隔符分隔"
        android:completionThreshold="1"/>
</LinearLayout>
```

上面的界面布局文件中定义了 AutoCompleteTextView 和 MultiAutoCompleteTextView，接下来在程序中为它们绑定同一个Adapter，这意味着两个自动完成文本框的提示项完全相同，只是它们的表现行为略有差异。

新建AutoCompleteTextViewActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.ArrayAdapter;
import android.widget.AutoCompleteTextView;
import android.widget.MultiAutoCompleteTextView;
 
public class AutoCompleteTextViewActivity extends AppCompatActivity {
    private AutoCompleteTextView mAutoTv = null;
    private MultiAutoCompleteTextView mMultiAutoTv = null;
 
    // 定义字符串数组，作为提示的文本
    private String[] mContacts = new String[]{
            "test", "abc", "aaa", "aabbcc", "bac", "ok", "say", "aabbsd"
    };
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.autocomplete_textview_layout);
        mAutoTv = (AutoCompleteTextView)findViewById(R.id.auto_actv);
        mMultiAutoTv = (MultiAutoCompleteTextView)findViewById(R.id.mauto_mactv);
 
        // 创建一个ArrayAdapter，封装数组
        ArrayAdapter<String> aa = new ArrayAdapter<String>(this,
                android.R.layout.simple_dropdown_item_1line, mContacts);
 
        // 设置Adapter
        mAutoTv.setAdapter(aa);
        mMultiAutoTv.setAdapter(aa);
 
        // 为MultiAutoCompleteTextView设置分隔符
        mMultiAutoTv.setTokenizer(new MultiAutoCompleteTextView.CommaTokenizer());
    }
}
```

上面程序代码负责为AutoCompleteTextView、MultiAutoCompleteTextView 设置同一个 Adapter，并为 MultiAutoCompleteTextView 设置了分隔符。

### 可折叠列表ExpandableListView

ExpandableListView 是 ListView 的子类，它在普通ListView的基础上进行了扩展，它把应用中的列表项分为几组，每组里又可包含多个列表项。

ExpandableListView的用法与普通 ListView的用法非常相似，只是 ExpandableListView所显示的列表项应 该由 ExpandableListAdapter 提供。ExpandableListAdapter 也是一个接口，该接口下提供的类继承关系图如下图所示：

![1.jpeg](https://i.loli.net/2019/05/09/5cd40d7a131a2.jpeg)

与Adapter类似的是，实现 ExpandableListAdapter也有如下三种常用方式。

- 扩展 BaseExpandableListAdapter 实现 ExpandableListAdapter。

- 使用 SimpleExpandableListAdapter 将两个 List 集合包装成 ExpandableListAdapter。

- 使用 SimpleCursorTreeAdapter 将 Cursor 中的数据包装成 SimpleCursorTreeAdapter。 


ExpandableListView支持的常用XML属性如下：

- android:childDivider：指定各组内子类表项之间的分隔条，图片不会完全显示， 分离子列表项的是一条直线。

- android:childIndicator：显示在子列表旁边的Drawable对象，可以是一个图像。

- android:childIndicatorEnd：子列表项指示符的结束约束位置。

- android:childIndicatorLeft：子列表项指示符的左边约束位置。

- android:childIndicatorRight：子列表项指示符的右边约束位置。

- android:childIndicatorStart：子列表项指示符的开始约束位置。

- android:groupIndicator：显示在组列表旁边的Drawable对象，可以是一个图像。

- android:indicatorEnd：组列表项指示器的结束约束位置。

- android:indicatorLeft：组列表项指示器的左边约束位置。

- android:indicatorRight：组列表项指示器的右边约束位置。

- android:indicatorStart：组列表项指示器的开始约束位置。

expandlist_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent"  >
 
    <ExpandableListView
        android:id="@+id/expendlist"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```

expendlist_group.xml

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="70dp"
                android:gravity="center_vertical"
                android:padding="10dp">
 
    <ImageView
        android:id="@+id/group_img"
        android:layout_width="36dp"
        android:layout_height="36dp"
        android:layout_centerVertical="true"
        android:layout_marginLeft="20dp"
        android:src="@drawable/group_close" />
 
    <TextView
        android:id="@+id/groupname_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_toRightOf="@+id/group_img"
        android:layout_marginLeft="10dp"
        android:text="张三"
        android:textSize="18sp" />
</RelativeLayout>
```

expendlist_item.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="70dp"
              android:orientation="horizontal"
              android:gravity="center_vertical"
              android:padding="10dp" >
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="48dp"
        android:layout_height="48dp"
        android:layout_marginLeft="20dp"
        android:src="@drawable/item" />
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:gravity="center_vertical"
        android:orientation="vertical" >
 
        <TextView
            android:id="@+id/itemname_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="22sp"
            android:textStyle="bold"
            android:text="李大钊" />
 
        <TextView
            android:id="@+id/info_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            android:text="今天是个好日子啊!" />
    </LinearLayout>
</LinearLayout>
```

新建MyExpandableListViewAdapter类，继承ExpandableListViewAdapter，并重写其方法，代码如下：

```
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseExpandableListAdapter;
import android.widget.ImageView;
import android.widget.TextView;
 
 
import java.util.List;

public class MyExpandableListViewAdapter extends BaseExpandableListAdapter {
    private Context mContext = null;
    private List<String> mGroupList = null;
    private List<List<String>> mItemList = null;
 
    public MyExpandableListViewAdapter(Context context, List<String> groupList,
                                       List<List<String>> itemList) {
        this.mContext = context;
        this.mGroupList = groupList;
        this.mItemList = itemList;
    }
 
    /**
     * 获取组的个数
     *
     * @return
     * @see android.widget.ExpandableListAdapter#getGroupCount()
     */
    @Override
    public int getGroupCount() {
        return mGroupList.size();
    }
 
    /**
     * 获取指定组中的子元素个数
     *
     * @param groupPosition
     * @return
     * @see android.widget.ExpandableListAdapter#getChildrenCount(int)
     */
    @Override
    public int getChildrenCount(int groupPosition) {
        return mItemList.get(groupPosition).size();
    }
 
    /**
     * 获取指定组中的数据
     *
     * @param groupPosition
     * @return
     * @see android.widget.ExpandableListAdapter#getGroup(int)
     */
    @Override
    public String getGroup(int groupPosition) {
        return mGroupList.get(groupPosition);
    }
 
    /**
     * 获取指定组中的指定子元素数据。
     *
     * @param groupPosition
     * @param childPosition
     * @return
     * @see android.widget.ExpandableListAdapter#getChild(int, int)
     */
    @Override
    public String getChild(int groupPosition, int childPosition) {
        return mItemList.get(groupPosition).get(childPosition);
    }
 
    /**
     * 获取指定组的ID，这个组ID必须是唯一的
     *
     * @param groupPosition
     * @return
     * @see android.widget.ExpandableListAdapter#getGroupId(int)
     */
    @Override
    public long getGroupId(int groupPosition) {
        return groupPosition;
    }
 
    /**
     * 获取指定组中的指定子元素ID
     *
     * @param groupPosition
     * @param childPosition
     * @return
     * @see android.widget.ExpandableListAdapter#getChildId(int, int)
     */
    @Override
    public long getChildId(int groupPosition, int childPosition) {
        return childPosition;
    }
 
    /**
     * 获取显示指定组的视图对象
     *
     * @param groupPosition 组位置
     * @param isExpanded    该组是展开状态还是伸缩状态
     * @param convertView   重用已有的视图对象
     * @param parent        返回的视图对象始终依附于的视图组
     * @return
     * @see android.widget.ExpandableListAdapter#getGroupView(int, boolean, android.view.View,
     * android.view.ViewGroup)
     */
    @Override
    public View getGroupView(int groupPosition, boolean isExpanded, View convertView,
                             ViewGroup parent) {
        GroupHolder groupHolder = null;
        if (convertView == null) {
            convertView = LayoutInflater.from(mContext).inflate(R.layout.expendlist_group, null);
            groupHolder = new GroupHolder();
            groupHolder.groupNameTv = (TextView) convertView.findViewById(R.id.groupname_tv);
            groupHolder.groupImg = (ImageView) convertView.findViewById(R.id.group_img);
            convertView.setTag(groupHolder);
        } else {
            groupHolder = (GroupHolder) convertView.getTag();
        }
 
        if (isExpanded) {
            groupHolder.groupImg.setImageResource(R.drawable.group_open);
        } else {
            groupHolder.groupImg.setImageResource(R.drawable.group_close);
        }
        groupHolder.groupNameTv.setText(mGroupList.get(groupPosition));
 
        return convertView;
    }
 
    /**
     * 获取一个视图对象，显示指定组中的指定子元素数据。
     *
     * @param groupPosition 组位置
     * @param childPosition 子元素位置
     * @param isLastChild   子元素是否处于组中的最后一个
     * @param convertView   重用已有的视图(View)对象
     * @param parent        返回的视图(View)对象始终依附于的视图组
     * @return
     * @see android.widget.ExpandableListAdapter#getChildView(int, int, boolean, android.view.View,
     * android.view.ViewGroup)
     */
    @Override
    public View getChildView(int groupPosition, int childPosition, boolean isLastChild,
                             View convertView, ViewGroup parent) {
        ItemHolder itemHolder = null;
        if (convertView == null) {
            convertView = LayoutInflater.from(mContext).inflate(R.layout.expendlist_item, null);
            itemHolder = new ItemHolder();
            itemHolder.nameTv = (TextView) convertView.findViewById(R.id.itemname_tv);
            itemHolder.iconImg = (ImageView) convertView.findViewById(R.id.icon_img);
            convertView.setTag(itemHolder);
        } else {
            itemHolder = (ItemHolder) convertView.getTag();
        }
        itemHolder.nameTv.setText(mItemList.get(groupPosition).get(childPosition));
        itemHolder.iconImg.setBackgroundResource(R.drawable.item);
 
        return convertView;
    }
 
    /**
     * 组和子元素是否持有稳定的ID,也就是底层数据的改变不会影响到它们。
     *
     * @return
     * @see android.widget.ExpandableListAdapter#hasStableIds()
     */
    @Override
    public boolean hasStableIds() {
        return true;
    }
 
    /**
     * 是否选中指定位置上的子元素。
     *
     * @param groupPosition
     * @param childPosition
     * @return
     * @see android.widget.ExpandableListAdapter#isChildSelectable(int, int)
     */
    @Override
    public boolean isChildSelectable(int groupPosition, int childPosition) {
        return true;
    }
 
 
    class GroupHolder {
        public TextView groupNameTv;
        public ImageView groupImg;
    }
 
    class ItemHolder {
        public ImageView iconImg;
        public TextView nameTv;
    }
}
```

上面程序的关键代码就是扩展BaseExpandableListAdapter来实现ExpandableListAdapter, 当扩展BaseExpandableListAdapter时，关键是实现如下4个方法。

- getGroupCount()：该方法返回包含的组列表项的数量。

- getGroupView()：该方法返回的View对象将作为组列表项。

- getChildrenCount()：该方法返回特定组所包含的子列表项的数量。

- getChildView()：该方法返回的View对象将作为特定组、特定位置的子列表项。


新建ExpandableListActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.ExpandableListView;
import android.widget.Toast;
 
 
import java.util.ArrayList;
import java.util.List;
 
public class ExpandableListActivity extends AppCompatActivity {
    private ExpandableListView mExpandableListView = null;
    // 列表数据
    private List<String> mGroupNameList = null;
    private List<List<String>> mItemNameList = null;
    // 适配器
    private MyExpandableListViewAdapter mAdapter = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.expandlist_layout);
 
        // 获取组件
        mExpandableListView = (ExpandableListView) findViewById(R.id.expendlist);
        mExpandableListView.setGroupIndicator(null);
 
        // 初始化数据
        initData();
 
        // 为ExpandableListView设置Adapter
        mAdapter = new MyExpandableListViewAdapter(this, mGroupNameList, mItemNameList);
        mExpandableListView.setAdapter(mAdapter);
 
        // 监听组点击
        mExpandableListView.setOnGroupClickListener(new ExpandableListView.OnGroupClickListener() {
            @Override
            public boolean onGroupClick(ExpandableListView parent, View v,
                                        int groupPosition, long id) {
                if (mGroupNameList.get(groupPosition).isEmpty()) {
                    return true;
                }
                return false;
            }
        });
 
        // 监听每个分组里子控件的点击事件
        mExpandableListView.setOnChildClickListener(new ExpandableListView.OnChildClickListener() {
            @Override
            public boolean onChildClick(ExpandableListView parent, View v, int groupPosition,
                                        int childPosition, long id) {
                Toast.makeText(ExpandableListActivity.this,
                        mAdapter.getGroup(groupPosition) + ":"
                                +  mAdapter.getChild(groupPosition, childPosition) ,
                        Toast.LENGTH_SHORT).show();
                return false;
            }
        });
    }
 
    // 初始化数据
    private void initData(){
        // 组名
        mGroupNameList = new ArrayList<String>();
        mGroupNameList.add("历代帝王");
        mGroupNameList.add("华坛明星");
        mGroupNameList.add("国外明星");
        mGroupNameList.add("政坛人物");
 
        mItemNameList = new  ArrayList<List<String>>();
        // 历代帝王组
        List<String> itemList = new ArrayList<String>();
        itemList.add("唐太宗李世民");
        itemList.add("秦始皇嬴政");
        itemList.add("汉武帝刘彻");
        itemList.add("明太祖朱元璋");
        itemList.add("宋太祖赵匡胤");
        mItemNameList.add(itemList);
        // 华坛明星组
        itemList = new ArrayList<String>();
        itemList.add("范冰冰 ");
        itemList.add("梁朝伟");
        itemList.add("谢霆锋");
        itemList.add("章子怡");
        itemList.add("杨颖");
        itemList.add("张柏芝");
        mItemNameList.add(itemList);
        // 国外明星组
        itemList = new ArrayList<String>();
        itemList.add("安吉丽娜•朱莉");
        itemList.add("艾玛•沃特森");
        itemList.add("朱迪•福斯特");
        mItemNameList.add(itemList);
        // 政坛人物组
        itemList = new ArrayList<String>();
        itemList.add("唐纳德•特朗普");
        itemList.add("金正恩");
        itemList.add("奥巴马");
        itemList.add("普京");
        mItemNameList.add(itemList);
    }
}
```

上述代码为ExpandableListView设置Adapter，并为ExpandableListView设置事件监听器。点击组的时候，会将其子元素打开。

### AdapterViewFlipper图片轮播

 AdapterViewFilpper 继承 了AdapterViewAnimator，它也会显示 Adapter 提供的多个 View 组件，但它每次只能显示一个View组件，程序可通过showPrevious()和showNext()方法控制该组件显示上一个、下一个组件。

AdapterViewFilpper可以在多个View切换过程中使用渐隐渐显的动画效果。除此之外，还可以调用该组件的startFlipping()控制它“自动播放”下一个View组件。 

AdapterViewAnimator支持的XML属性如下：

- android:animateFirstView：设置显示组件的第一个View时是否使用动画。

- android:inAnimation：设置组件显示时使用的动画。

- android:loopViews：设置循环到最后一个组件时是否自动跳转到第一个组件。

- android:outAnimation：设置组件隐藏时使用的动画。


AdapterViewFilpper额外支持的XML属性及相关方法如下表所示。

![1.png](https://i.loli.net/2019/05/09/5cd40eb790b38.png)

adapterview_filpper_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8" ?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
    <AdapterViewFlipper
        android:id="@+id/flipper"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:flipInterval="5000"
        android:layout_alignParentTop="true"/>
    <Button
        android:id="@+id/prev_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:text="上一个"/>
    <Button
        android:id="@+id/next_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:text="下一个"/>
    <Button
        android:id="@+id/auto_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:text="自动播放"/>
</RelativeLayout>
```

创建一个MyFilpperAdapter类，继承BaseAdapter类，重写其4个主要方法，具体代码如下：

```
import android.content.Context;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;

public class MyFilpperAdapter extends BaseAdapter {
    private Context mContext = null;
    private int[] mImageIds = null;
 
    public MyFilpperAdapter(Context context, int[] imageIds) {
        this.mContext = context;
        this.mImageIds = imageIds;
    }
 
    @Override
    public int getCount() {
        return mImageIds.length;
    }
 
    @Override
    public Object getItem(int position) {
        return position;
    }
 
    @Override
    public long getItemId(int position) {
        return position;
    }
 
    // 该方法返回的View代表了每个列表项
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ImageView imageView = null;
        if(null == convertView) {
            // 创建一个ImageView
            imageView = new ImageView(mContext);
            // 设置ImageView的缩放类型
            imageView.setScaleType(ImageView.ScaleType.FIT_XY);
            // 为imageView设置布局参数
            imageView.setLayoutParams(new ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
            convertView = imageView;
        } else {
            imageView = (ImageView) convertView;
        }
 
        // 给ImageView设置图片资源
        imageView.setImageResource(mImageIds[position]);
 
        return imageView;
    }
}
```

接下来为AdapterViewFilpper提供Adapter，使用自定义的BaseAdapter。新建AdapterViewFilperActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.AdapterViewFlipper;
import android.widget.Button;
 
 
public class AdapterViewFilperActivity extends AppCompatActivity implements View.OnClickListener {
    private AdapterViewFlipper mFlipper = null;
    private Button mPrevBtn = null;
    private Button mNextBtn = null;
    private Button mAutoBtn = null;
    private int[] mImageIds = {
            R.drawable.image_01,R.drawable.image_02,R.drawable.image_03,
            R.drawable.image_04,R.drawable.image_05,R.drawable.image_06,
            R.drawable.image_07,R.drawable.image_08,R.drawable.image_09
    };
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.adapterview_filpper_layout);
 
        // 获取界面组件
        mFlipper = (AdapterViewFlipper) findViewById(R.id.flipper);
        mPrevBtn = (Button) findViewById(R.id.prev_btn);
        mNextBtn = (Button) findViewById(R.id.next_btn);
        mAutoBtn = (Button) findViewById(R.id.auto_btn);
 
        // 为AdapterViewFlipper设置Adapter
        MyFilpperAdapter adapter = new MyFilpperAdapter(this, mImageIds);
        mFlipper.setAdapter(adapter);
 
        // 为三个按钮设置点击事件监听器
        mPrevBtn.setOnClickListener(this);
        mNextBtn.setOnClickListener(this);
        mAutoBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.prev_btn:
                // 显示上一个组件
                mFlipper.showPrevious();
                // 停止自动播放
                mFlipper.stopFlipping();
                break;
            case R.id.next_btn:
                // 显示下一个组件。
                mFlipper.showNext();
                // 停止自动播放
                mFlipper.stopFlipping();
                break;
            case R.id.auto_btn:
                // 开始自动播放
                mFlipper.startFlipping();
                break;
            default:
                break;
        }
    }
}
```

上面程序代码调用了 AdapterViewFlipper 的 showPrevious()、 showNext()方法来控制该组件显示上一个、下一个组件，并调用了 startFlipping()方法控制自动播放。

单击上一个或下一个按钮可以切换显示的组件，单击自动播放按钮，将可以看到AdapterViewFlipper每隔5秒更换一个图片，切换图片时会使用渐隐渐显效果。

### StackView卡片堆叠

StackView也是AdapterView Animator的子类，它也用于显示Adapter提供的一系列View。 StackView将会以堆叠（Stack）的方式来显示多个列表项。

为了控制StackView显示的View组件，StackView提供了如下两种控制方式。

- 拖走StackView中处于顶端的View，下一个View将会显示出来。将上一个View拖进StackView，将使之显示出来。

- 通过调用StackView的showNext()、showPrevious()控制显示下一个、上一个组件。

stackview_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal">
        <Button
            android:id="@+id/prev_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="上一个" />
        <Button
            android:id="@+id/next_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="下一个" />
    </LinearLayout>
    <StackView
        android:id="@+id/stackview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:loopViews="true" />
</LinearLayout>
```

创建一个MyStackAdapter类，继承BaseAdapter类，重写其4个主要方法，具体代码如下：

```
import android.content.Context;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;

public class MyStackAdapter extends BaseAdapter {
    private Context mContext = null;
    private int[] mImageIds = null;
 
    public MyStackAdapter(Context context, int[] imageIds) {
        this.mContext = context;
        this.mImageIds = imageIds;
    }
 
    @Override
    public int getCount() {
        return mImageIds.length;
    }
 
    @Override
    public Object getItem(int position) {
        return position;
    }
 
    @Override
    public long getItemId(int position) {
        return position;
    }
 
    // 该方法返回的View代表了每个列表项
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ImageView imageView = null;
        if(null == convertView) {
            // 创建一个ImageView
            imageView = new ImageView(mContext);
            // 设置ImageView的缩放类型
            imageView.setScaleType(ImageView.ScaleType.FIT_XY);
            // 为imageView设置布局参数
            imageView.setLayoutParams(new ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
            convertView = imageView;
        } else {
            imageView = (ImageView) convertView;
        }
 
        // 给ImageView设置图片资源
        imageView.setImageResource(mImageIds[position]);
 
        return imageView;
    }
}
```

接下来为StackView提供Adapter，使用自定义的BaseAdapter。新建StackViewActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.StackView; 
public class StackViewActivity extends AppCompatActivity implements View.OnClickListener {
    private StackView mStackView = null;
    private Button mPrevBtn = null;
    private Button mNextBtn = null;
    private int[] mImageIds = {
            R.drawable.image_01, R.drawable.image_02, R.drawable.image_03, R.drawable.image_04,
            R.drawable.image_05, R.drawable.image_06, R.drawable.image_07, R.drawable.image_08
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.stackview_layout);
 
        // 获取界面组件
        mStackView = (StackView) findViewById(R.id.stackview);
        mPrevBtn = (Button) findViewById(R.id.prev_btn);
        mNextBtn = (Button) findViewById(R.id.next_btn);
 
        // 为AdapterViewFlipper设置Adapter
        MyStackAdapter adapter = new MyStackAdapter(this, mImageIds);
        mStackView.setAdapter(adapter);
 
        // 为三个按钮设置点击事件监听器
        mPrevBtn.setOnClickListener(this);
        mNextBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.prev_btn:
                // 显示上一个组件
                mStackView.showPrevious();
                break;
            case R.id.next_btn:
                // 显示下一个组件
                mStackView.showNext();
                break;
            default:
                break;
        }
    }
}
```

 点击上一个或下一个按钮时，StackView将会将组件分别显示出来。当拖动StackView的组件时，也可以实现同样的效果。

### 日历视图CalendarView

 日历视图（CalendarView）可用于显示和选择日期，用户既可选择一个日期，也可通过触 摸来滚动日历。如果希望监控该组件的日期改变，则可调用CalendarView的 setOnDateChangeListener()方法为此组件的点击事件添加事件监听器。 

用CalendarView时可指定如下表所示的常见XML属性及相关方法

![2.jpeg](https://i.loli.net/2019/05/09/5cd439802c56e.jpeg)

创建calendarview_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
    <CalendarView
        android:id="@+id/calendarView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:firstDayOfWeek="3"
        android:shownWeekCount="4"  />
</RelativeLayout>
```

修改MainActivity里面的代码，加载上述定义的布局文件，然后运行程序.



### 定时器Chronometer

 Chronometer是一个简单的定时器，可以通过setBase()来给它一个基准时间，并从该时间开始计数；如果不给基准时间，将使用调用start()方法时的时间。默认将显示当前"MM:SS"或 "H:MM:SS"格式的时间，当然也可以自定义字符串来格式化显示。

Chronometer的一个比较重要的XML属性如下：

- android:format：设置时间的格式如: hh:mm:ss。


Chronometer的一些常用方法如下：

- setBase(long base)：设置倒计时定时器。

- setFormat(String format)：设置显示时间的格式。

- start()：开始计时。

- stop()：停止计时。


在使用Chronometer时，如果希望监控该组件的时间，则可调用Chronometer的 setOnChronometerTickListener()方法为此组件的点击事件添加事件监听器。

创建chronnmeter_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"
              android:gravity="center_horizontal">
 
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onStart"
            android:text="开始计时" />
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onStop"
            android:text="停止计时" />
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onReset"
            android:text="重置" />
    </LinearLayout>
 
    <Chronometer
        android:id="@+id/chronometer"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="15dp"/>
</LinearLayout>
```

接下来在Activity中完成Chronometer格式化，并响应用户的操作。新建ChronometerActivity.java文件，加载上面新建的布局文件，初始化DatePicker并获取用户的选择，具体代码如下：

```
import android.os.Bundle;
import android.os.SystemClock;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Chronometer;

public class ChronometerActivity extends AppCompatActivity {
    private Chronometer mChronometer = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.chronnmeter_layout);
 
        mChronometer = (Chronometer) findViewById(R.id.chronometer);
        //setFormat设置用于显示的格式化字符串。
        //替换字符串中第一个“%s”为当前"MM:SS"或 "H:MM:SS"格式的时间显示。
        mChronometer.setFormat("计时：%s");
    }
 
    // 开始计时
    public void onStart(View view) {
        mChronometer.start();
    }
    // 停止计时
    public void onStop(View view) {
        mChronometer.stop();
    }
    // 重置
    public void onReset(View view) {
        //setBase 设置基准时间
        //设置参数base为SystemClock.elapsedRealtime()即表示从当前时间开始重新计时）。
        mChronometer.setBase(SystemClock.elapsedRealtime());
    }
}
```







### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)