---
title: Android  高级控件（二）
date: 2019-04-30 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### 滚动视图ScrollView

在默认情况下，ScrollView只是为其他组件添加垂直滚动条，如果应用需要添加水平滚动条，则可借助于另一个滚动视图HorizontalScrollView来实现。ScrollView与HorizontalScrollView的功能基本相似，只是前者添加垂直滚动条，后者添加水平滚动条。

<!--more-->

ScrollView由FrameLayout派生而出，它就是一个用于为普通组件添加滚动条的组件。 ScrollView里最多只能包含一个组件，而ScrollView的作用就是为该组件添加垂直滚动条。

ScrollView支持的XML属性如下：

- android:scrollX：以像素为单位设置水平方向滚动的的偏移值。

- android:scrollY：以像素为单位设置垂直方向滚动的的偏移值。

- android:scrollbarAlwaysDrawHorizontalTrack：设置是否始终显示垂直滚动条。

- android:scrollbarAlwaysDrawVerticalTrack：设置是否始终显示垂直滚动条。

- android:scrollbarDefaultDelayBeforeFade：设置N毫秒后开始淡化，以毫秒为单位。

- android:scrollbarFadeDuration：设置滚动条淡出效果（从有到慢慢的变淡直至消失）时间，以毫秒为单位。

- android:scrollbarSize：设置滚动条的宽度。

- android:scrollbarStyle：设置滚动条的风格和位置。属性值有以下几个：

- outsideInset：该ScrollBar显示在视图(view)的边缘,增加了view的padding. 如果可能的话,该ScrollBar仅仅覆盖这个view的背景。

- outsideOverlay：该ScrollBar显示在视图(view)的边缘,不增加view的padding,该ScrollBar将被半透明覆盖。

- insideInset：该ScrollBar显示在padding区域里面,增加了控件的padding区域,该ScrollBar不会和视图的内容重叠。

- insideOverlay：该ScrollBar显示在内容区域里面,不会增加了控件的padding区域,该ScrollBar以半透明的样式覆盖在视图(view)的内容上。

- android:scrollbarThumbHorizontal：设置水平滚动条的drawable。

- android:scrollbarThumbVertical：设置垂直滚动条的drawable。

- android:scrollbarTrackHorizontal：设置水平滚动条背景（轨迹）的色drawable。

- android:scrollbarTrackVertical：设置垂直滚动条背景（轨迹）的drawable。

- android:scrollbars：设置滚动条显示。属性值有：none、horizontal、vertical。


ScrollView的几个常用方法有：

- addView (View child)：添加子视图。如果事先没有给子视图设置layout参数，会采用当前ViewGroup的默认参数来设置子视图。

- addView (View child, int index)：添加子视图。如果事先没有给子视图设置layout参数，会采用当前ViewGroup的默认参数来设置子视图。

- arrowScroll (int direction)：响应点击上下箭头时对滚动条滚动的处理。

- fling (int velocityY)：滚动视图的滑动（fling）手势。

创建scrollview_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="10dp"
            android:fillViewport="false">
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <ImageView
            android:id="@+id/imageView"
            android:layout_width="wrap_content"
            android:layout_height="200dp"
            android:scaleType="centerCrop"
            android:src="@drawable/image" />
        <Button
            android:id="@+id/more_btn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="more" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ScrollView"
            android:textAppearance="?android:attr/textAppearanceLarge" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/description"
            android:textAppearance="?android:attr/textAppearanceSmall" />
    </LinearLayout>
</ScrollView>
```

其中description为定义的字符串，由于内容较多，此处不在给出。

### 搜索框组件SearchView

 SearchView是搜索框组件，它可以让用户在文本框内输入文字，并允许通过监听器监控用户输入，当用户输入完成后提交搜索时，也可通过监听器执行实际的搜索。

SearchView默认是展示一个search的icon，点击icon展开搜索框，也可以自己设定图标。用SearchView时可指定如下表所示的常见XML属性及相关方法

![1.jpeg](https://i.loli.net/2019/05/09/5cd43df1cc641.jpeg)

如果为SearchView增加一个配套的ListView，则可以为SearchView增加自动完成的功能。

创建searchview_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:layout_margin="15dp"
              android:orientation="vertical" >
 
    <SearchView
        android:id="@+id/searchView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="请输入搜索内容" />
 
    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
</LinearLayout>
```

上面的布局文件中定义了一个SearchView组件，并为该SearchView组件定义了一个 ListView组件，该ListView组件用于为SearchView组件显示不自动完成列表。

接下来为SearchView编写操作控制代码，并为其添加监听器。新建SearchViewActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.text.TextUtils;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SearchView;

public class SearchViewActivity extends AppCompatActivity {
    private SearchView mSearchView = null;
    private ListView mListView = null;
    private String[] mDatas = {"aaa", "bbb", "ccc", "airsaid"};
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.searchview_layout);
 
        mSearchView = (SearchView) findViewById(R.id.searchView);
        mListView = (ListView) findViewById(R.id.listView);
        mListView.setAdapter(new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_1, mDatas));
        mListView.setTextFilterEnabled(true);
 
        // 设置搜索文本监听
        mSearchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            // 当点击搜索按钮时触发该方法
            @Override
            public boolean onQueryTextSubmit(String query) {
                return false;
            }
 
            // 当搜索内容改变时触发该方法
            @Override
            public boolean onQueryTextChange(String newText) {
                if (!TextUtils.isEmpty(newText)){
                    mListView.setFilterText(newText);
                }else{
                    mListView.clearTextFilter();
                }
                return false;
            }
        });
    }
}
```

### 选项卡TabHost

TabHost是一种非常实用的组件，TabHost可以很方便地在窗口上放置多个标签页，每个标签页相当于获得了一个与外部容器相同大小的组件摆放区域。通过这种方式，就可以在一个容器里放置更多组件。

TabHost是整个Tab的容器，包含TabWidget和FrameLayout两个部分，TabWidget是每个Tab的标签，FrameLayout是Tab内容。与TabHost结合使用的有如下2个组件。

- TabWidget：代表选项卡的标题条。

- TabSpec：代表选项卡的一个Tab页面。


TabHost仅仅是一个简单的容器，它提供了如下两个方法来创建、添加标签页。

- newTabSpec(String tag)：创建选项卡。

- addTab(TabHost.TabSpec tabSpec)：添加选项卡。


实现TabHost有两种方式：

- 直接让一个Activity程序继承TabActivity类。

- 定义XML布局文件利用findViewById()方法取得TagHost组件，通过setup()方法实例化并进行配置。



#### 继承TabActivity实现

通过继承TabActivity类，使用TabHost的一般步骤如下。

1. 在界面布局文件中定义TabHost组件，并为该组件定义该选项卡的内容。

2. Activity 应该继承 TabActivity。

3. 调用 TabActivity 的 getTabHost()方法获取 TabHost 对象。

4. 通过TabHost对象的方法来创建、添加选项卡。


除此之外，TabHost还提供了一些方法来获取当前选项卡。如果程序需要监控TabHost里当前标签页的改变，则可以为它设置 TabHost.OnTabChangeListener 监听器。

```
<?xml version="1.0" encoding="utf-8"?>
<TabHost xmlns:android="http://schemas.android.com/apk/res/android"
         android:id="@android:id/tabhost"
         android:layout_width="match_parent"
         android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
 
        <TabWidget
            android:id="@android:id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
 
        <FrameLayout
            android:id="@android:id/tabcontent"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1">
            <LinearLayout
                android:id="@+id/widget_layout_red"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#ff0000"
                android:orientation="vertical"/>
            <LinearLayout
                android:id="@+id/widget_layout_green"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#00ff00"
                android:orientation="vertical"/>
            <LinearLayout
                android:id="@+id/widget_layout_blue"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#0000ff"
                android:orientation="vertical"/>
        </FrameLayout>
    </LinearLayout>
</TabHost>
```

请注意上面的布局文件中代码，从上面的布局文件可以发现，TabHost容器内部需要组合两个组件：TabWidget和FrameLayout，其中TabWidget用于定义选项卡的标题条， FrameLayout则用于层叠组合多个选项页面。不仅如此，上面的布局文件中这三个组件的 ID也有要求。

- TabHost 的 ID 应该为@android:id/tabhost。

- TabWidget 的 ID 应该为@android:id/tabs。

- FrameLayout 的 ID 应该为@android:id/tabcontent。


上面这三个ID并不是开发者自己定义的，而是引用了 Android系统已有的ID。

接下来主程序即可加载该布局资源，并将布局文件中的三个Tab页面添加到该TabHost 容器中。新建TabHostTabActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.app.TabActivity;
import android.os.Bundle;
import android.widget.TabHost;
 
public class TabHostTabActivity extends TabActivity {
    private TabHost mTabHost = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.tabhosttab_layout);
 
        // 从TabActivity上面获取放置Tab的TabHost
        mTabHost = getTabHost();
 
        // 添加第一个标签
        TabHost.TabSpec tab1 = mTabHost.newTabSpec("0");
        tab1.setIndicator("红色");
        tab1.setContent(R.id.widget_layout_red);
        mTabHost.addTab(tab1);
 
        // 添加第二个标签
        mTabHost.addTab(mTabHost
                // 创建新标签
                .newTabSpec("1")
                // 设置标签标题
                .setIndicator("绿色")
                // 设置该标签的布局内容
                .setContent(R.id.widget_layout_green));
 
        // 添加第三个标签
        mTabHost.addTab(mTabHost.newTabSpec("2")
                .setIndicator("蓝色")
                .setContent(R.id.widget_layout_blue));
    }
}
```

上面程序中的代码就是为TabHost创建并添加Tab页面的代码。上面的程序一共添加了三个标签页，用了两种方式来实现。

上面的程序调用了 TabHost.TabSpec对象的 setContent(int viewld)方法来设置标签页内容。

除此之外，还可调用setContent(Intent intent)方法来设置标签页内容，Intent还可用于启动其他Activity——这意味着TabHost.TabSpec可直接装载另一个Activity。

#### 继承Activity实现

与继承TabActivity实现TabHost大体步骤差不多，唯一的区别就是没有TabActivity系统封装，必须由开发者自己获取TabHost组件而已。

创建tabhost_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<TabHost xmlns:android="http://schemas.android.com/apk/res/android"
         android:id="@+id/mytabhost"
         android:layout_width="match_parent"
         android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
 
        <TabWidget
            android:id="@android:id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
 
        <FrameLayout
            android:id="@android:id/tabcontent"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1">
            <LinearLayout
                android:id="@+id/widget_layout_red"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#ff0000"
                android:orientation="vertical"/>
            <LinearLayout
                android:id="@+id/widget_layout_green"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#00ff00"
                android:orientation="vertical"/>
            <LinearLayout
                android:id="@+id/widget_layout_blue"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#0000ff"
                android:orientation="vertical"/>
        </FrameLayout>
    </LinearLayout>
</TabHost>
```

从上述代码可以发现，除了TabHost 的id可以自定义外，TabWidget和FrameLayout仍然必须为系统的ID。

界面交互代码稍微有所不同，新建TabHostActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
public class TabHostActivity extends AppCompatActivity {
    private TabHost mTabHost = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.tabhost_layout);
 
        // 获取TabHost组件
        mTabHost = (TabHost) findViewById(R.id.mytabhost);
        //调用 TabHost.setup()
        mTabHost.setup();
 
        // 添加三个标签
        mTabHost.addTab(mTabHost.newTabSpec("0").setIndicator("红色").setContent(R.id.widget_layout_red));;
        mTabHost.addTab(mTabHost.newTabSpec("1").setIndicator("绿色").setContent(R.id.widget_layout_green));
        mTabHost.addTab(mTabHost.newTabSpec("2").setIndicator("蓝色").setContent(R.id.widget_layout_blue));
    }
}
```

 会发现除了查找我们自定义的TabHost组件外，还多了一条语句setup()来加载TabHost，否则程序会报错。

运行修改后的程序，最终效果同继承TabActivity一样。

有木有发现这个界面很不美观，所以在实际开发中经常会借用RadioButton来定制TabHost。

其实TabHost组件在安卓4.0之后已经被废弃了，建议使用Fragment组件来代替它。由于其设计违反了Activity单一窗口原则，它可以同时加载多个Activity，然后再它们之间进行来回切换；另外有个很致命的问题就是当点击别的选项时，按下Back后退键，它会使整个应用程序都退出，而不是切换到前一个选项卡，虽然可以在主程序里覆写OnKeyDown这个方法，但这样就会导致每一次按下Back后退键都只能回到第一个选项菜单。

### RecyclerView

从前面的学习我们知道，ListView的功能非常强大，几乎绝大部分应用程序都会使用到，虽然也学会一些方法技巧来提升ListView的效率，但其性能还是不是很完美。

另外ListView的可扩展性相对来说比较弱，以前要实现每个列表项的高度不同的界面，或者要完成瀑布流效果，需要非常复杂的自定义处理。

谷歌在Android L中新增了RecyclerView，是一种新的视图组，目标是为任何基于适配器的视图提供相似的渲染方式。它被作为ListView和GridView控件的继承者，在最新的support-V7版本中提供支持。

RecyclerView可以看作是ListView的进化版本，当然RecyclerView并不是继承ListView的，RecyclerView直接继承于ViewGroup父类。RecyclerView与ListView原理是类似的：都是仅仅维护少量的View并且可以展示大量的数据集。

在开发RecyclerView时充分考虑了扩展性，因此用它可以创建想到的任何种类的的布局。但在使用上也稍微有些不便，比如使用步骤更加复杂，特别是一些控制点击、长压事件需要自己完成。使用RecyclerView开发的项目结构大致如下图所示：

![1.jpeg](https://i.loli.net/2019/05/09/5cd43fe368185.jpeg)

 从上图可以看到，要使用RecyclerView，需要先了解清楚LayoutManager和Adapter元素，分别如下：

- ​    LayoutManager：用来确定每一个item如何进行排列摆放，何时展示和隐藏。回收或重用一个View的时候，LayoutManager会向适配器请求新的数据来替换旧的数据，这种机制避免了创建过多的View和频繁的调用findViewById方法。目前RecyclerView库提供了如下三种子Manager：
  - ​    LinearLayoutManager：展示了水平或者垂直的滚动列表，相当于之前学习的ListView，但是没有页眉和页尾。
  - ​    GridLayoutManager：在网格中展示条目，相当于之前学习的GridView。
  - ​    StaggeredGridLayoutManager： 在错落的网格中展示条目，比如常见的瀑布流。
- ​    Adapter：这是一种新型适配器，不同于之前使用的BaseAdapter了。在使用RecyclerView之前，需要自定义一个继承自RecyclerView.Adapter的适配器，将数据与每一个item的界面进行绑定。需要注意的是编写Adapter面向的是ViewHolder而不在是View了，复用的逻辑被封装了起来，实现更加简单。使用时需要重写以下两个主要方法：
  - ​    onCreateViewHolder：用来展现视图和它的持有者。
  - ​    onBindViewHolder：主要用来把数据绑定到视图上。

除了上面两个主要元素，通常还会使用到如下三个类：

- ViewHolder：维持了所有被数据填充的实体的视图的引用。
- ItemDecoration：一个实体的周围的装饰。
- ItemAnimator：条目增加删除时重新排序所产生动画。

添加最新的recyclerview依赖库。

创建recyclerview_layout.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

继续新建一个recyclerview_item.xml的列表项布局文件，其代码如下：

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:padding="6dp" >
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_marginRight="6dp"
        android:contentDescription="null"
        android:src="@mipmap/ic_launcher" />
 
    <TextView
        android:id="@+id/title_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/icon_img"
        android:layout_alignParentTop="true"
        android:layout_marginTop="5dp"
        android:gravity="center_vertical"
        android:text="Title"
        android:textSize="16sp" />
 
    <TextView
        android:id="@+id/content_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/icon_img"
        android:layout_alignParentRight="true"
        android:layout_below="@id/title_tv"
        android:layout_marginTop="5dp"
        android:text="content"
        android:textSize="12sp" />
 
</RelativeLayout>
```

接下来就是创建适配器Adapter，新建RecyclerViewAdapter类，继承RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder>类，完成内部类ViewHolder ，并重写以下3个主要方法，具体代码如下：

```

public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder> {
    private ArrayList<String> mDatas = null;
    private LayoutInflater mInflater = null;
 
    public RecyclerViewAdapter(Context context, ArrayList<String> datas) {
        this.mDatas = datas;
        this.mInflater = LayoutInflater.from(context);
    }
 
    // 创建新View，被LayoutManager所调用
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = mInflater.inflate(R.layout.recyclerview_item, parent, false);
        ViewHolder vewHolder = new ViewHolder(view);
        return vewHolder;
    }
 
    // 将数据与界面进行绑定的操作
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        String name = mDatas.get(position);
        holder.titleTv.setText("Title " + name);
        holder.contenTv.setText("content " + name);
    }
 
    // 获取数据的数量
    @Override
    public int getItemCount() {
        return mDatas == null ? 0 : mDatas.size();
    }
 
    // 自定义的ViewHolder，持有每个Item的的所有界面组件
    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView titleTv = null;
        public TextView contenTv = null;
 
        public ViewHolder(View itemView) {
            super(itemView);
 
            titleTv = (TextView) itemView.findViewById(R.id.title_tv);
            contenTv = (TextView) itemView.findViewById(R.id.content_tv);
        }
    }
}
```

然后使用RecyclerView实现ListView效果，使用自定义的RecyclerViewAdapter决定RecyclerView所要显示的内容，并设置显示的界面样式。新建RecyclerViewActivity.java文件，加载上面新建的布局文件，具体代码如下：

```

public class RecyclerViewActivity extends AppCompatActivity {
    private RecyclerView mRecyclerView = null;
    private RecyclerViewAdapter mAdapter = null;
    private ArrayList<String> mDatas = null;
    
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.recyclerview_layout);
 
        // 获取组件
        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);
 
        // 设置管理器
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(layoutManager);
        // 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
        mRecyclerView.setHasFixedSize(true);
 
        // 初始化列表数据
        initDatas();
 
        // 设置适配器
        mAdapter = new RecyclerViewAdapter(this, mDatas);
        mRecyclerView.setAdapter(mAdapter);
    }
 
    private void initDatas() {
        mDatas = new ArrayList<>();
 
        for (int i = 0; i < 50; i++) {
            mDatas.add(i, i + 1 + "");
        }
    }
}
```

从上面例子可以看出来，RecyclerView的用法并不比ListView复杂，反而更灵活好用，它将数据、排列方式、数据的展示方式都分割开来，因此可定制型，自定义的形式也非常多，非常灵活。

#### RecyclerView扩展

接下来继续使用上面的例子实现水平列表、网格和瀑布流，你就会发现其灵活性到底有多高。

如果想要一个横向的List，只要简单设置LinearLayoutManager就行。只需要在RecyclerViewActivity中添加一行设置方向的代码即可，局部代码如下：

```
        // 设置管理器
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);
        mRecyclerView.setLayoutManager(layoutManager);
        // 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
        mRecyclerView.setHasFixedSize(true);
```

其余代码不变，重新运行程序，可以看到水平列表界面效果

如果想要一个GridView布局的列表，只要将之前的LayoutManager换为GridLayoutManager即可，局部代码如下：

```
        // 设置管理器
        GridLayoutManager layoutManager = new GridLayoutManager(this, 3);
        mRecyclerView.setLayoutManager(layoutManager);
        // 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
        mRecyclerView.setHasFixedSize(true);
```

其余代码不变，重新运行程序，可以看到网格界面效果

需要注意的是，在网格布局中也可以设置列表的Orientation属性，来实现横向和纵向的网格布局。

如果想要实现一个瀑布流，同样只需要将之前的LayoutManager换为StaggeredGridLayoutManager即可，局部代码如下：

```
        // 设置管理器
        StaggeredGridLayoutManager layoutManager = new StaggeredGridLayoutManager(3,
                StaggeredGridLayoutManager.VERTICAL);
        mRecyclerView.setLayoutManager(layoutManager);
        // 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
        mRecyclerView.setHasFixedSize(true);
```

由于之前是等高，直接运行和网格布局效果无差异。简单修改一下自定义的RecyclerViewAdapter类中onBindViewHolder方法，使其产生一个随机的高度，代码如下：

```
    // 将数据与界面进行绑定的操作
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        String name = mDatas.get(position);
        holder.titleTv.setText("Title " + name);
        holder.contenTv.setText("content " + name);
 
        int padding = Math.abs(new Random().nextInt() % 50);
        holder.titleTv.setPadding(0, padding, 0, 0);
        holder.contenTv.setPadding(0, 0, 0, padding);
    }
```

### RecyclerView分割线

由于RecyclerView并没有支持divider这样的属性，需要我们自己想办法来完成。主要有两种实现方式，接下来分别对其进行学习。

#### 背景设置显示间隔

先给RecyclerView添加黑色背景，然后再给每个item添加白色背景并设置间隔1dp，这样自然就用背景空隙当做分割线了。

在上一节的基础上进行简单修改即可，修改后的recyclerview_layout.xml文件代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:background="#5000"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

修改后的recyclerview_item.xml文件代码如下：

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="#fff"
                android:layout_marginBottom="1dp"
                android:padding="6dp" >
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_marginRight="6dp"
        android:contentDescription="null"
        android:src="@mipmap/ic_launcher" />
 
    <TextView
        android:id="@+id/title_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/icon_img"
        android:layout_alignParentTop="true"
        android:layout_marginTop="5dp"
        android:gravity="center_vertical"
        android:text="Title"
        android:textSize="16sp" />
 
    <TextView
        android:id="@+id/content_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/icon_img"
        android:layout_alignParentRight="true"
        android:layout_below="@id/title_tv"
        android:layout_marginTop="5dp"
        android:text="content"
        android:textSize="12sp" />
 
</RelativeLayout>
```

其他地方的代码不变

#### 自定义分割线

上面第一种实现方式非常简单，但有时候还是不足以完成实际需求，这就需要用到自定义分割线了。

RecyclerView类提供了一个addItemDecoration方法，我们可以通过该方法添加分割线。

首先我们自定义一个drawable文件recyclerview_item_divider，具体内容后续会进行学习的，这里不做过多介绍，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle">
    <gradient
        android:centerColor="#ff00ff00"
        android:endColor="#ff0000ff"
        android:startColor="#ffff0000"
        android:type="linear" />
    <size android:height="1dp"/>
</shape>
```

由于RecyclerView.ItemDecoration为抽象类，需要自定义一个实现类，该类很好的实现了为RecyclerView添加分割线。新建RecyclerViewItemDivider类，具体代码如下：

```

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.graphics.drawable.Drawable;
import android.support.v7.widget.RecyclerView;
import android.view.View;

public class RecyclerViewItemDivider extends RecyclerView.ItemDecoration {
    private Drawable mDrawable;
 
    public RecyclerViewItemDivider(Context context, int resId) {
        mDrawable = context.getResources().getDrawable(resId);
    }
 
    @Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        int left = parent.getPaddingLeft();
        int right = parent.getWidth() - parent.getPaddingRight();
        int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            View child = parent.getChildAt(i);
            RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child.getLayoutParams();
            int top = child.getBottom() + params.bottomMargin;
            int bottom = top + mDrawable.getIntrinsicHeight();
            mDrawable.setBounds(left, top, right, bottom);
            mDrawable.draw(c);
        }
    }
 
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent,
                               RecyclerView.State state) {
        outRect.set(0, 0, 0, mDrawable.getIntrinsicWidth());
    }
}
```

然后在将自定义的分割线添加到RecyclerView中，局部代码如下：

```
// 设置管理器
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
mRecyclerView.setLayoutManager(layoutManager);
// 自定义分割线
RecyclerView.ItemDecoration itemDecoration = new RecyclerViewItemDivider(this,R.drawable.recyclerview_item_divider);
mRecyclerView.addItemDecoration(itemDecoration);
// 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
mRecyclerView.setHasFixedSize(true);
```

其余代码不变

### RecyclerView点击事件处理

 在介绍RecyclerView开篇的时候简单提到过，要实现一些控制点击、长压事件需要自己完成，不像之前学的ListView有自带ClickListener和LongClickListener，但其实更加灵活多样，可以对点击方式按照自己的方式来实现。

仍然在上一节的代码基础来进行修改，既然RecyclerView没有提供onClick和onLongClick事件，那我们自己来实现就好了。

首先在RecyclerViewAdapter类中分别定义2个接口OnItemClickListener和OnItemLongClickListener，然后提供2个公开方法便于Activity设置事件监听，并在onBindViewHolder方法中设置监听事件，当有事件发生时，则可以回调到Activity，然后即可完成相应的处理。RecyclerViewAdapter类修改后的代码如下：

```
import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
 
 
import java.util.ArrayList;

public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder> {
    private ArrayList<String> mDatas = null;
    private LayoutInflater mInflater = null;
    private OnItemClickListener mOnItemClickListener = null;
    private OnItemLongClickListener mOnItemLongClickListener = null;
 
    public RecyclerViewAdapter(Context context, ArrayList<String> datas) {
        this.mDatas = datas;
        this.mInflater = LayoutInflater.from(context);
    }
 
    // 创建新View，被LayoutManager所调用
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = mInflater.inflate(R.layout.recyclerview_item, parent, false);
        ViewHolder vewHolder = new ViewHolder(view);
        return vewHolder;
    }
 
    // 将数据与界面进行绑定的操作
    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position) {
        String name = mDatas.get(position);
        holder.titleTv.setText("Title " + name);
        holder.contenTv.setText("content " + name);
 
        // 点击事件注册及分发
        if(null != mOnItemClickListener) {
            holder.titleTv.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mOnItemClickListener.onClick(holder.titleTv, position);
                }
            });
            holder.contenTv.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mOnItemClickListener.onClick(holder.contenTv, position);
                }
            });
        }
        // 长按事件注册及分发
        if(null != mOnItemLongClickListener) {
            holder.titleTv.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    return mOnItemLongClickListener.onLongClick(holder.titleTv, position);
                }
            });
            holder.contenTv.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    return mOnItemLongClickListener.onLongClick(holder.contenTv, position);
                }
            });
        }
    }
 
    // 获取数据的数量
    @Override
    public int getItemCount() {
        return mDatas == null ? 0 : mDatas.size();
    }
 
    // 设置点击事件
    public void setOnItemClickListener(OnItemClickListener l) {
        this.mOnItemClickListener = l;
    }
 
    // 设置长按事件
    public void setOnItemLongClickListener(OnItemLongClickListener l) {
        this.mOnItemLongClickListener = l;
    }
 
    // 点击事件接口
    public interface OnItemClickListener {
        void onClick(View parent, int position);
    }
 
    // 长按事件接口
    public interface OnItemLongClickListener {
        boolean onLongClick(View parent, int position);
    }
 
    // 自定义的ViewHolder，持有每个Item的的所有界面组件
    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView titleTv = null;
        public TextView contenTv = null;
 
        public ViewHolder(View itemView) {
            super(itemView);
 
            titleTv = (TextView) itemView.findViewById(R.id.title_tv);
            contenTv = (TextView) itemView.findViewById(R.id.content_tv);
        }
    }
}
```

紧接着就是在Activity中设置监听事件和响应监听事件，RecyclerViewActivity修改后的代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
 

import java.util.ArrayList;

public class RecyclerViewActivity extends AppCompatActivity implements
        RecyclerViewAdapter.OnItemClickListener,
        RecyclerViewAdapter.OnItemLongClickListener {
    private RecyclerView mRecyclerView = null;
    private RecyclerViewAdapter mAdapter = null;
    private ArrayList<String> mDatas = null;
    
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.recyclerview_layout);
 
        // 获取组件
        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);
 
        // 设置管理器
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(layoutManager);
        // 自定义分割线
        RecyclerView.ItemDecoration itemDecoration = new RecyclerViewItemDivider(this,
                R.drawable.recyclerview_item_divider);
        mRecyclerView.addItemDecoration(itemDecoration);
        // 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
        mRecyclerView.setHasFixedSize(true);
 
        // 初始化列表数据
        initDatas();
 
        // 设置适配器
        mAdapter = new RecyclerViewAdapter(this, mDatas);
        mAdapter.setOnItemClickListener(this);
        mAdapter.setOnItemLongClickListener(this);
        mRecyclerView.setAdapter(mAdapter);
    }
 
    private void initDatas() {
        mDatas = new ArrayList<>();
 
        for (int i = 0; i < 50; i++) {
            mDatas.add(i, i + 1 + "");
        }
    }
 
    @Override
    public void onClick(View parent, int position) {
        Toast.makeText(this, "点击了第" + (position + 1) + "项", Toast.LENGTH_SHORT).show();
    }
 
    @Override
    public boolean onLongClick(View parent, int position) {
        Toast.makeText(this, "长压了第" + (position + 1) + "项", Toast.LENGTH_SHORT).show();
        return true;
    }
}
```

  其余布局文件代码不变

### RecyclerView数据动态更新

之前在学习ListView的时候如果数据改变，需要调用notifyDataSetChanged()方法来刷新数据，而在RecyclerView中当数据改变时分别调用notifyItemChanged、notifyItemInserted和notifyItemRemoved方法来更新页面数据。

接下来通过一个案例来学习如何动态更新数据，当单击某个item时则在其下方插入一个item，如果长压某个item时则删除对应item。

继续使用上期的案例，首先在RecyclerViewAdapter类中新增一个插入和删除处理的公开方法，RecyclerViewAdapter类修改后的代码如下：

```
 import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.ArrayList;

public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder> {
    private ArrayList<String> mDatas = null;
    private LayoutInflater mInflater = null;
    private OnItemClickListener mOnItemClickListener = null;
    private OnItemLongClickListener mOnItemLongClickListener = null;
 
    public RecyclerViewAdapter2(Context context, ArrayList<String> datas) {
        this.mDatas = datas;
        this.mInflater = LayoutInflater.from(context);
    }
 
    // 创建新View，被LayoutManager所调用
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = mInflater.inflate(R.layout.recyclerview_item, parent, false);
        ViewHolder vewHolder = new ViewHolder(view);
        return vewHolder;
    }
 
    // 将数据与界面进行绑定的操作
    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position) {
        String name = mDatas.get(position);
        holder.titleTv.setText("Title " + name);
        holder.contenTv.setText("content " + name);
 
        // 点击事件注册及分发
        if(null != mOnItemClickListener) {
            holder.titleTv.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mOnItemClickListener.onClick(holder.titleTv, position);
                }
            });
            holder.contenTv.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mOnItemClickListener.onClick(holder.contenTv, position);
                }
            });
        }
        // 长按事件注册及分发
        if(null != mOnItemLongClickListener) {
            holder.titleTv.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    return mOnItemLongClickListener.onLongClick(holder.titleTv, position);
                }
            });
            holder.contenTv.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    return mOnItemLongClickListener.onLongClick(holder.contenTv, position);
                }
            });
        }
    }
 
    // 获取数据的数量
    @Override
    public int getItemCount() {
        return mDatas == null ? 0 : mDatas.size();
    }
 
    // 设置点击事件
    public void setOnItemClickListener(OnItemClickListener l) {
        this.mOnItemClickListener = l;
    }
 
    // 设置长按事件
    public void setOnItemLongClickListener(OnItemLongClickListener l) {
        this.mOnItemLongClickListener = l;
    }
 
    // 在对应位置增加一个item
    public void addData(int position) {
        mDatas.add(position, "Insert " + position);
        notifyItemInserted(position);
    }
 
    // 删除对应item
    public void removeData(int position) {
        mDatas.remove(position);
        notifyItemRemoved(position);
    }
 
    // 点击事件接口
    public interface OnItemClickListener {
        void onClick(View parent, int position);
    }
 
    // 长按事件接口
    public interface OnItemLongClickListener {
        boolean onLongClick(View parent, int position);
    }
 
    // 自定义的ViewHolder，持有每个Item的的所有界面组件
    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView titleTv = null;
        public TextView contenTv = null;
 
        public ViewHolder(View itemView) {
            super(itemView);
 
            titleTv = (TextView) itemView.findViewById(R.id.title_tv);
            contenTv = (TextView) itemView.findViewById(R.id.content_tv);
        }
    }
}
```

然后在Activity中事件回调的时候进行插入和删除处理，RecyclerViewActivity修改后的代码如下：

```

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;

import java.util.ArrayList;

public class RecyclerViewActivity extends AppCompatActivity implements
        RecyclerViewAdapter.OnItemClickListener,
        RecyclerViewAdapter.OnItemLongClickListener {
    private RecyclerView mRecyclerView = null;
    private RecyclerViewAdapter mAdapter = null;
    private ArrayList<String> mDatas = null;
    
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.recyclerview_layout);
 
        // 获取组件
        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);
 
        // 设置管理器
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(layoutManager);
        // 自定义分割线
        RecyclerView.ItemDecoration itemDecoration = new RecyclerViewItemDivider(this,
                R.drawable.recyclerview_item_divider);
        mRecyclerView.addItemDecoration(itemDecoration);
        // 如果可以确定每个item的高度是固定的，设置这个选项可以提高性能
        mRecyclerView.setHasFixedSize(true);
 
        // 初始化列表数据
        initDatas();
 
        // 设置适配器
        mAdapter = new RecyclerViewAdapter(this, mDatas);
        mAdapter.setOnItemClickListener(this);
        mAdapter.setOnItemLongClickListener(this);
        mRecyclerView.setAdapter(mAdapter);
 
        mRecyclerView.setItemAnimator(new DefaultItemAnimator());
    }
 
    private void initDatas() {
        mDatas = new ArrayList<>();
 
        for (int i = 0; i < 50; i++) {
            mDatas.add(i, i + 1 + "");
        }
    }
 
    @Override
    public void onClick(View parent, int position) {
        mAdapter.addData(position + 1);
        //Toast.makeText(this, "点击了第" + (position + 1) + "项", Toast.LENGTH_SHORT).show();
    }
 
    @Override
    public boolean onLongClick(View parent, int position) {
        mAdapter.removeData(position);
        //Toast.makeText(this, "长压了第" + (position + 1) + "项", Toast.LENGTH_SHORT).show();
        return true;
    }
}
```

其余布局文件代码不变

### 完善RecyclerView，添加首尾视图

 首先来简单回顾一下ListView是如何添加列表头和列表尾的，先定义好首尾视图，然后通过addHeaderView和addFooterView两个方法来加载即可，相对来说比较简单。然后在RecyclerView中并未发现类似的方法，那么应该如何为其添加首尾视图呢？

可能一些细心的同学已经发了RecyclerView.Adapter中还有几个方法没有被重写过，就先来看看是哪几个方法：

- getItemViewType：判断当前item类型。

- isHeaderView：判断当前item是否是HeadView。

- isBottomView：判断当前item是否是FooterView。


同时可以看到在onCreateViewHolder方法里面带一个viewType参数，实际上onCreateViewHolder方法就是根据viewType来判断具体item是列表项、HeaderView或FooterView，然后来分别加载不同的布局文件。

接下来继续使用再上一期的案例来学习如何给RecyclerView添加首尾视图。

在main/res/layout/目录下创建recyclerview_header.xml文件，在其中填充如下代码片段：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="50dp">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="Header View"
        android:textSize="20sp"
        android:gravity="center"/>
</LinearLayout>
```

继续新建一个recyclerview_footer.xml文件，在其中填充如下代码片段：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="50dp">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="Footer View"
        android:textSize="20sp"
        android:gravity="center"/>
</LinearLayout>
```

然后修改RecyclerViewAdapter文件，在getItemViewType方法里面判断了当前Item的类型，然后在onCreateViewHolder跟据item的类型分别加载不同的布局以实现HeaderView和FooterView。修改后的RecyclerViewAdapter代码如下：

```
import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView; 
import java.util.ArrayList;

public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    //item类型
    public static final int ITEM_TYPE_HEADER = 0;
    public static final int ITEM_TYPE_CONTENT = 1;
    public static final int ITEM_TYPE_BOTTOM = 2;
 
    private ArrayList<String> mDatas = null;
    private LayoutInflater mInflater = null;
    private OnItemClickListener mOnItemClickListener = null;
    private OnItemLongClickListener mOnItemLongClickListener = null;
 
    public RecyclerViewAdapter(Context context, ArrayList<String> datas) {
        this.mDatas = datas;
        this.mInflater = LayoutInflater.from(context);
    }
 
    //内容长度
    public int getContentItemCount(){
        return mDatas == null ? 0 : mDatas.size();
    }
 
    //判断当前item是否是HeadView
    public boolean isHeaderView(int position) {
        return 0 == position;
    }
 
    //判断当前item是否是FooterView
    public boolean isBottomView(int position) {
        return position == (getContentItemCount() + 1);
    }
 
    //判断当前item类型
    @Override
    public int getItemViewType(int position) {
        if (0 == position) {
            //头部View
            return ITEM_TYPE_HEADER;
        } else if (getContentItemCount() + 1 == position) {
            //底部View
            return ITEM_TYPE_BOTTOM;
        } else {
            //内容View
            return ITEM_TYPE_CONTENT;
        }
    }
 
    // 创建新View，被LayoutManager所调用
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        RecyclerView.ViewHolder vewHolder = null;
        View view = null;
        if(ITEM_TYPE_HEADER == viewType) {
            view = mInflater.inflate(R.layout.recyclerview_header, parent, false);
            vewHolder = new HeaderViewHolder(view);
        } else if(ITEM_TYPE_BOTTOM == viewType) {
            view = mInflater.inflate(R.layout.recyclerview_footer, parent, false);
            vewHolder = new FooterViewHolder(view);
        } else {
            view = mInflater.inflate(R.layout.recyclerview_item, parent, false);
            vewHolder = new ViewHolder(view);
        }
 
        return vewHolder;
    }
 
    // 将数据与界面进行绑定的操作
    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, final int position) {
        if(holder instanceof HeaderViewHolder) {
 
        } else if(holder instanceof FooterViewHolder){
 
        }else {
            final ViewHolder viewHolder = (ViewHolder) holder;
            String name = mDatas.get(position - 1);
            viewHolder.titleTv.setText("Title " + name);
            viewHolder.contenTv.setText("content " + name);
 
            // 点击事件注册及分发
            if (null != mOnItemClickListener) {
                viewHolder.titleTv.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mOnItemClickListener.onClick(viewHolder.titleTv, position);
                    }
                });
                viewHolder.contenTv.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mOnItemClickListener.onClick(viewHolder.contenTv, position);
                    }
                });
            }
            // 长按事件注册及分发
            if (null != mOnItemLongClickListener) {
                viewHolder.titleTv.setOnLongClickListener(new View.OnLongClickListener() {
                    @Override
                    public boolean onLongClick(View v) {
                        return mOnItemLongClickListener.onLongClick(viewHolder.titleTv, position);
                    }
                });
                viewHolder.contenTv.setOnLongClickListener(new View.OnLongClickListener() {
                    @Override
                    public boolean onLongClick(View v) {
                        return mOnItemLongClickListener.onLongClick(viewHolder.contenTv, position);
                    }
                });
            }
        }
    }
 
    // 获取数据的数量
    @Override
    public int getItemCount() {
        return getContentItemCount() + 2;
    }
 
    // 设置点击事件
    public void setOnItemClickListener(OnItemClickListener l) {
        this.mOnItemClickListener = l;
    }
 
    // 设置长按事件
    public void setOnItemLongClickListener(OnItemLongClickListener l) {
        this.mOnItemLongClickListener = l;
    }
 
    // 在对应位置增加一个item
    public void addData(int position) {
        mDatas.add(position, "Insert " + position);
        notifyItemInserted(position);
 
        if(position != getItemCount()) {
            notifyItemRangeChanged(position, getItemCount());
        }
    }
 
    // 删除对应item
    public void removeData(int position) {
        mDatas.remove(position);
        notifyItemRemoved(position);
 
        if(position != getItemCount()) {
            notifyItemRangeChanged(position, getItemCount());
        }
    }
 
    // 点击事件接口
    public interface OnItemClickListener {
        void onClick(View parent, int position);
    }
 
    // 长按事件接口
    public interface OnItemLongClickListener {
        boolean onLongClick(View parent, int position);
    }
 
    // 自定义的ViewHolder，持有每个Item的的所有界面组件
    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView titleTv = null;
        public TextView contenTv = null;
 
        public ViewHolder(View itemView) {
            super(itemView);
 
            titleTv = (TextView) itemView.findViewById(R.id.title_tv);
            contenTv = (TextView) itemView.findViewById(R.id.content_tv);
        }
    }
    //头部 ViewHolder
    public class HeaderViewHolder extends RecyclerView.ViewHolder {
 
        public HeaderViewHolder(View itemView) {
            super(itemView);
        }
    }
    //底部 ViewHolder
    public class FooterViewHolder extends RecyclerView.ViewHolder {
 
        public FooterViewHolder(View itemView) {
            super(itemView);
        }
    }
}
```

然后其他代码不变



### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)