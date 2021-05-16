---
title: Android 高级控件（三）
date: 2019-05-02 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### ViewPager快速实现引导页

ViewPager是android扩展包v4包中的类，这个类可以让用户左右滑动切换当前的view。ViewPager继承自ViewGroup，也就是ViewPager是一个容器类，可以包含其他的View类。

<!--more-->

ViewPager的主要方法有以下几个：

- setAdapter(PagerAdapter adapter) ：为ViewPager设置适配器，ViewPager有三种适配器，包括PagerAdapter、FragmentPagerAdapter、FragmentStatePagerAdapter，它们分别有不同的特性，本期会先来学习PagerAdapter。

- setCurrentItem(int item) ：设置显示item位置的界面。

- setOffscreenPageLimit(int limit) ：用来设置当前显示页面左右两边各缓存的页面数。

- addOnPageChangeListener(OnPageChangeListener listener) ：为ViewPager添加页面切换时的监听。

 关于界面监听的内容，接下来对OnPageChangeListener中的方法进行说明：

- onPageScrollStateChanged(int state) ：该方法在手指操作屏幕的时候发生变化。有三个值：0(END)、1(PRESS) 、2(UP) 。当用手指滑动翻页时，手指按下去的时候会触发这个方法，state值为1，手指抬起时，如果发生了滑动（即使很小），这个值会变为2，然后最后变为0 。总共执行这个方法三次。一种特殊情况是手指按下去以后一点滑动也没有发生，这个时候只会调用这个方法两次，state值分别是1、0 。当setCurrentItem翻页时，会执行这个方法两次，state值分别为2 、0 。

- onPageScrolled(int position, float positionOffset, int positionOffsetPixels) ：该方法在滑动过程中将一直被调用，该方法的参数说明如下： 
  - ​    position：当用手指滑动时，如果手指按在页面上不动，position和当前页面index是一致的；如果手指向左拖动（相应页面向右翻动），这时候position大部分时间和当前页面是一致的，只有翻页成功的情况下最后一次调用才会变为目标页面；如果手指向右拖动（相应页面向左翻动），这时候position大部分时间和目标页面是一致的，只有翻页不成功的情况下最后一次调用才会变为原页面。当直接设置setCurrentItem翻页时，如果是相邻的情况（比如现在是第二个页面，跳到第一或者第三个页面），如果页面向右翻动，大部分时间是和当前页面是一致的，只有最后才变成目标页面；如果向左翻动，position和目标页面是一致的。这和用手指拖动页面翻动是基本一致的。如果不是相邻的情况，比如我从第一个页面跳到第三个页面，position先是0，然后逐步变成1，然后逐步变成2；我从第三个页面跳到第一个页面，position先是1，然后逐步变成0，并没有出现为2的情况。 
  - ​    positionOffset：当前页面滑动比例，如果页面向右翻动，这个值不断变大，最后在趋近1的情况后突变为0。如果页面向左翻动，这个值不断变小，最后变为0。 
  - ​    positionOffsetPixels：当前页面滑动像素，变化情况和positionOffset一致。
- onPageSelected(int position) ：position是被选中页面的索引，该方法在页面被选中或页面滑动足够距离切换到该页手指抬起时调用。

上面三个方法的执行顺序：用手指拖动翻页时，最先执行一遍onPageScrollStateChanged(1)，然后不断执行onPageScrolled，放手指的时候，直接立即执行一次onPageScrollStateChanged(2)，然后立即执行一次onPageSelected，然后再不断执行onPageScrollStateChanged，最后执行一次onPageScrollStateChanged(0)。

在大多数使用适配器的控件里，适配器相对于数据源和视图来说都更加复杂，同时也决定了这个控件主要的功能，ViewPager也不例外。实现一个PagerAdapter时，至少需要重写下面的4个方法：

- getCount()：返回有效视图的数量。

- isViewFromObject(View, Object)：决定一个页面view是否与instantiateItem(ViewGroup, int)方法返回的具体key对象相关联。 

- instantiateItem(ViewGroup, int)：创建指定位置的页面视图。适配器有责任增加即将创建的View视图到给定的container中，确保在finishUpdate(viewGroup)返回时，增加视图的事情已经完成。

- destroyItem(ViewGroup, int, Object)：移除给定位置的view，适配器有责任将该view从container中移除，确保在finishUpdate(viewGroup)返回时，移除视图的事情已经完成。


除了上述4个方法，还有以下一些方法，可根据实际需要进行重写。

- getItemPosition (Object object)：当宿主视图尝试判断一项的位置是否改变时调用。如果给定项的位置没有改变则返回POSITION_UNCHANGED，如果该项不再存在于适配器中则返回POSITION_NONE。

- startUpdate (ViewGroup container) ：在展示的界面中有改变将要发生时调用。

- finishUpdate (ViewGroup container)：展示界面中的改变完成时调用。在这个时间点上，你必须确保所有的页面已被合适的从container中添加或移除。

- notifyDataSetChanged ()：该方法由应用程序在适配器数据改变时主动调用。

- registerDataSetObserver (DataSetObserver observer)：注册一个观察者去接收关联到适配器数据变化的回调。

- unregisterDataSetObserver (DataSetObserver observer)：反注册去接收关联到适配器数据变化的回调的观察者。

- setPrimaryItem (ViewGroup container, int position, Object object)：调用该方法去通知当前适配器的哪一项被考虑为“primary”，它是当前展示给用户的页面。

- getPageTitle (int position)：该方法由ViewPager在获取描述页面的标题时调用，默认返回null。

- getPageWidth (int position)：该方法返回给定页面的比例宽度，范围(0.f-1.f]。

- saveState ()：保存与适配器关联的实例状态，当当前UI状态需要重建时恢复。

- restoreState (Parcelable state, ClassLoader loader)：恢复之前由saveState ()保存的与适配器关联的实例状态。


ViewPager的具体使用类似于之前学习的列表类组件，首先构造适配器，然后提供数据源，最后加载适配器。

创建viewpager_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </android.support.v4.view.ViewPager>
</LinearLayout>
```

然后新建几个页面文件，第一个页面命名为viewpager_pager1.xml

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
 
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:src="@drawable/image_01"/>
</RelativeLayout>
```

另外再新建3个，为了简单布局和上面一样，将其中的代码加载的图片换一下即可

 新建ViewPagerAdapter类，继承PagerAdapter，并重写其方法

```
import android.support.v4.view.PagerAdapter;
import android.view.View;
import android.view.ViewGroup;
 
import java.util.ArrayList;

public class ViewPagerAdapter extends PagerAdapter {
    private ArrayList<View> mPageList = null;
 
    public ViewPagerAdapter(ArrayList<View> pageList) {
        this.mPageList = pageList;
    }
 
    @Override
    public int getCount() {
        return mPageList.size();
    }
 
    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }
 
    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        View pageView = mPageList.get(position);
        container.addView(pageView);
        return pageView;
    }
 
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        // 将当前位置的View移除
        container.removeView(mPageList.get(position));
    }
}
```

新建ViewPagerActivity.java文件，加载上面新建的布局文件，具体代码如下：

```

import android.os.Bundle;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Toast;

import java.util.ArrayList;

public class ViewPagerActivity extends AppCompatActivity implements ViewPager.OnPageChangeListener {
    private ViewPager mViewPager = null;
    private ViewPagerAdapter mAdapter = null;
    private ArrayList<View> mPageList = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.viewpager_layout);
 
        mViewPager = (ViewPager) findViewById(R.id.view_pager);
 
        mPageList = new ArrayList<>();
        LayoutInflater inflater = getLayoutInflater();
        mPageList.add(inflater.inflate(R.layout.viewpager_page1, null, false));
        mPageList.add(inflater.inflate(R.layout.viewpager_page2, null, false));
        mPageList.add(inflater.inflate(R.layout.viewpager_page3, null, false));
        mPageList.add(inflater.inflate(R.layout.viewpager_page4, null, false));
 
        mAdapter = new ViewPagerAdapter(mPageList);
        mViewPager.setAdapter(mAdapter);
 
        mViewPager.addOnPageChangeListener(this);
    }
 
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
 
    }
 
    @Override
    public void onPageSelected(int position) {
        Toast.makeText(this, "第" + (position + 1) + "页", Toast.LENGTH_SHORT).show();
    }
 
    @Override
    public void onPageScrollStateChanged(int state) {
 
    }
}
```

### ViewPager轻松完成TabHost效果

在实际运用中，很多时候只有页面滑动是不够的，还需要有标题栏才够友好。首先来学习一下官方自带的，在android.support.v4包中的两个控件PagerTabStrip与PagerTitleStrip。

上面提到的2个控件，其中PagerTitleStrip是普通文字，PagerTabStrip带有下划线。PagerTabStrip在效果上包含了PagerTitleStrip。

如果只添加PagerTabStrip可以看到只有线，但是它占的布局是有一定高度的，而且默认是不显示标题的，如果要显示出来，需在适配器里重写getPageTitle(int position)方法。关于标题及这条线的颜色，和整个标识View的背景，都可以在代码里设置。

还有一个区别就是，PagerTabStrip可以点击切换View，而PagerTitleStrip不能点击。

 继续再上一期的案例基础上来进行修改，首先修改viewpager_layout.xml文件，修改后的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
 
        <android.support.v4.view.PagerTabStrip
            android:id="@+id/view_pager_tabstrip"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    </android.support.v4.view.ViewPager>
</LinearLayout>
```

然后几个页面布局文件不变，既然需要在顶部显示Tab和标题，那就需要我们通过适配器来设置，修改后的ViewPagerAdapter类代码如下：

```
import android.support.v4.view.PagerAdapter;
import android.view.View;
import android.view.ViewGroup;
 
import java.util.ArrayList;

public class ViewPagerAdapter extends PagerAdapter {
    private ArrayList<View> mPageLists = null;
    private ArrayList<String> mTitleLists = null;
 
    public ViewPagerAdapter(ArrayList<View> pageLists, ArrayList<String> titleLists) {
        this.mPageLists = pageLists;
        this.mTitleLists = titleLists;
    }
 
    @Override
    public int getCount() {
        return (null == mPageLists) ? 0 :mPageLists.size();
    }
 
    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }
 
    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        View pageView = mPageLists.get(position);
        container.addView(pageView);
        return pageView;
    }
 
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        // 将当前位置的View移除
        container.removeView(mPageLists.get(position));
    }
 
    @Override
    public CharSequence getPageTitle(int position) {
        return (null == mTitleLists && position < mTitleLists.size())
                ? null : mTitleLists.get(position);
    }
}
```

接着修改ViewPagerActivity，修改的代码如下：

```
import android.graphics.Color;
import android.os.Bundle;
import android.support.v4.view.PagerTabStrip;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Toast; 
import java.util.ArrayList;

public class ViewPagerActivity extends AppCompatActivity implements ViewPager.OnPageChangeListener {
    private ViewPager mViewPager = null;
    private PagerTabStrip mPagerTabStrip = null;
    private ViewPagerAdapter mAdapter = null;
    private ArrayList<View> mPageLists = null;
    private ArrayList<String> mTitleLists = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.viewpager_layout);
 
        mViewPager = (ViewPager) findViewById(R.id.view_pager);
        mPagerTabStrip = (PagerTabStrip) findViewById(R.id.view_pager_tabstrip);
 
        // 装入分页显示hi的View视图
        mPageLists = new ArrayList<>();
        LayoutInflater inflater = getLayoutInflater();
        mPageLists.add(inflater.inflate(R.layout.viewpager_page1, null, false));
        mPageLists.add(inflater.inflate(R.layout.viewpager_page2, null, false));
        mPageLists.add(inflater.inflate(R.layout.viewpager_page3, null, false));
        mPageLists.add(inflater.inflate(R.layout.viewpager_page4, null, false));
        // 装入每个页面的Title
        mTitleLists = new ArrayList<>();
        mTitleLists.add("第一页");
        mTitleLists.add("第二页");
        mTitleLists.add("第三页");
        mTitleLists.add("第四页");
 
        mAdapter = new ViewPagerAdapter(mPageLists, mTitleLists);
        mViewPager.setAdapter(mAdapter);
 
        // 更改下划线颜色
        mPagerTabStrip.setTabIndicatorColor(Color.BLUE);
        mViewPager.addOnPageChangeListener(this);
    }
 
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
 
    }
 
    @Override
    public void onPageSelected(int position) {
        Toast.makeText(this, "第" + (position + 1) + "页", Toast.LENGTH_SHORT).show();
    }
 
    @Override
    public void onPageScrollStateChanged(int state) {
 
    }
}
```

#### 自定义实现

上面我们使用了系统自带的控件来完成Tab显示，可能有的同学已经发现其与TabHost还是有一定的差别的，上面的Tab只显示3个，而且也不能完全满足实际需求，就需要我们自定义来实现了。

创建viewpager_custom_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:background="#FFFFFF">
 
        <TextView
            android:id="@+id/viewpager_tv_one"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1.0"
            android:gravity="center"
            android:text="第一页"
            android:textColor="#000000" />
 
        <TextView
            android:id="@+id/viewpager_tv_two"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1.0"
            android:gravity="center"
            android:text="第二页"
            android:textColor="#000000" />
 
        <TextView
            android:id="@+id/viewpager_tv_three"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1.0"
            android:gravity="center"
            android:text="第三页"
            android:textColor="#000000" />
    </LinearLayout>
 
    <ImageView
        android:id="@+id/cursor_img"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:scaleType="matrix"
        android:src="@drawable/line" />
 
    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:flipInterval="30"
        android:persistentDrawingCache="animation" />
 
</LinearLayout>
```

在页面顶部有一个线性布局，里面包含3个TextView，也就是ViewPager顶部的3个Tab标签。然后下面紧跟一个ImageView，主要用于指示当前是哪一个Tab标签对应的页面，也就是常说的滑块。最后在最底下是一个ViewPager，其中android:flipInterval属性设置了动画的时间间隔，android:persistentDrawingCache属性指控件的绘制缓存策略，一共有4个可选值，或者选择2个进行组合，分别如下：

- none：不在内存中保存绘图缓存。

- animation:只保存动画绘图缓存。

- scrolling：只保存滚动效果绘图缓存。

- all：所有的绘图缓存都应该保存在内存中。


然后新建几个页面文件，这里继续使用上一期ViewPager快速实现引导页里面的页面文件，同样使用相同的适配器ViewPagerAdapter。

最后新建ViewPagerCustomActivity.java文件，加载上面新建的布局文件，具体代码如下：

```

import android.graphics.BitmapFactory;
import android.graphics.Matrix;
import android.os.Bundle;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.util.DisplayMetrics;
import android.view.LayoutInflater;
import android.view.View;
import android.view.animation.Animation;
import android.view.animation.TranslateAnimation;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.ArrayList;

public class ViewPagerCustomActivity extends AppCompatActivity
        implements ViewPager.OnPageChangeListener, View.OnClickListener {
    private ViewPager mViewPager = null;
    private ImageView mCursorImg = null;
    private TextView mOneTv = null;
    private TextView mTwoTv = null;
    private TextView mThreeTv = null;
 
    private ViewPagerAdapter mAdapter = null;
    private ArrayList<View> mPageList = null;
 
    private int mOffset = 0;// 移动条图片的偏移量
    private int mCurrIndex = 0; // 当前页面的编号
    private int mOneDis = 0; // 移动条滑动一页的距离
    private int mTwoDis = 0; // 滑动条移动两页的距离
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.viewpager_custom_layout);
 
        // 获取界面组件
        mViewPager = (ViewPager) findViewById(R.id.view_pager);
        mCursorImg = (ImageView) findViewById(R.id.cursor_img);
        mOneTv = (TextView) findViewById(R.id.viewpager_tv_one);
        mTwoTv = (TextView) findViewById(R.id.viewpager_tv_two);
        mThreeTv = (TextView) findViewById(R.id.viewpager_tv_three);
 
        // 初始化指示器位置
        initCursorPosition();
 
        mPageList = new ArrayList<>();
        LayoutInflater inflater = getLayoutInflater();
        mPageList.add(inflater.inflate(R.layout.viewpager_page1, null, false));
        mPageList.add(inflater.inflate(R.layout.viewpager_page2, null, false));
        mPageList.add(inflater.inflate(R.layout.viewpager_page3, null, false));
 
        // 设置适配器
        mAdapter = new ViewPagerAdapter(mPageList);
        mViewPager.setAdapter(mAdapter);
 
        // 文本框点击监听器
        mOneTv.setOnClickListener(this);
        mTwoTv.setOnClickListener(this);
        mThreeTv.setOnClickListener(this);
 
        // 页面改变监听器
        mViewPager.addOnPageChangeListener(this);
        // 初始默认第一页
        mViewPager.setCurrentItem(0);
    }
 
    private void initCursorPosition() {
        // 获取指示器图片宽度
        int cursorWidth = BitmapFactory.decodeResource(getResources(), R.drawable.line).getWidth();
 
        // 获取分辨率宽度
        DisplayMetrics dm = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(dm);
        int screenWidth = dm.widthPixels;
 
        // 计算偏移量
        mOffset = (screenWidth / 3 - cursorWidth) / 2;
 
        // 设置动画初始位置
        Matrix matrix = new Matrix();
        matrix.postTranslate(mOffset, 0);
        mCursorImg.setImageMatrix(matrix);
 
        // 计算指示器图片的移动距离
        mOneDis = mOffset * 2 + cursorWidth;// 页卡1 -> 页卡2 偏移量
        mTwoDis = mOneDis * 2;// 页卡1 -> 页卡3 偏移量
    }
 
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.viewpager_tv_one:
                mViewPager.setCurrentItem(0);
                break;
            case R.id.viewpager_tv_two:
                mViewPager.setCurrentItem(1);
                break;
            case R.id.viewpager_tv_three:
                mViewPager.setCurrentItem(2);
                break;
        }
    }
 
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
 
    }
 
    @Override
    public void onPageSelected(int position) {
        // 指示器图片动画设置
        Animation animation = null;
        switch (position) {
            case 0:
                if (1 == mCurrIndex) {
                    animation = new TranslateAnimation(mOneDis, 0, 0, 0);
                } else if (2 == mCurrIndex) {
                    animation = new TranslateAnimation(mTwoDis, 0, 0, 0);
                }
                break;
            case 1:
                if (0 == mCurrIndex) {
                    animation = new TranslateAnimation(mOffset, mOneDis, 0, 0);
                } else if (2 == mCurrIndex) {
                    animation = new TranslateAnimation(mTwoDis, mOneDis, 0, 0);
                }
                break;
            case 2:
                if (0 == mCurrIndex) {
                    animation = new TranslateAnimation(mOffset, mTwoDis, 0, 0);
                } else if (1 == mCurrIndex) {
                    animation = new TranslateAnimation(mOneDis, mTwoDis, 0, 0);
                }
                break;
            default:
                break;
        }
        mCurrIndex = position;
        animation.setFillAfter(true); // True:图片停在动画结束位置
        animation.setDuration(300);
        mCursorImg.startAnimation(animation);
    }
 
    @Override
    public void onPageScrollStateChanged(int state) {
 
    }
}
```

可以发现这里的代码和上期大致相同，只是在其中增加了滑块的位置及动画设置，为3个Tab标签监听了点击事件。其中initCursorPosition()方法主要初始化指示器图标的位置，需要根据屏幕宽度来计算游标显示位置。然后同样设置了页面监听器，主要根据滑动到的页面把游标滑动找指定位置。

### CardView简单实现卡片式布局

CardView是Android 5.0系统引入的控件，相当于FragmentLayout布局控件然后添加圆角及阴影的效果。

CardView继承自Framelayout，所以FrameLayout所有属性CardView均可以直接拿来用，不过CardView还有自己独有的属性，常用属性如下：

- app:cardElevation：设置阴影的大小。

- app:cardMaxElevation：设置阴影最大高度。

- app:cardBackgroundColor：设置卡片的背景色。

- app:cardCornerRadius：设置卡片的圆角大小。

- app:contentPadding：设置内容的padding。

- app:contentPaddingTop：设置内容的上padding。

- app:contentPaddingLeft：设置内容的左padding。

- app:contentPaddingRight：设置内容的右padding。

- app:contentPaddingBottom：设置内容的底padding。

- app:cardUseCompatPadding：是否使用CompatPadding。

- app:cardPreventConrerOverlap：是否使用PreventCornerOverlap。


这里有一点需要值得注意，之前学习到的控件属性都是android:开头的，而这里所列的属性是app:开头的，如果继续使用默认的会提示找不见对应属性，需要我们定义一个app命名空间，在布局文件中需要加入`xmlns:app="http://schemas.android.com/apk/res-auto"`语句，具体见后续案例，这里不作过多介绍，后续再详细学习。

#### 示例1

创建cardview_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:app="http://schemas.android.com/apk/res-auto"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 
    <android.support.v7.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp">
        <TextView
            android:layout_width="match_parent"
            android:layout_height="70dp"
            android:text="正常使用效果"
            android:gravity="center_horizontal|center_vertical"
            android:textSize="20sp"
            android:padding="10dp"
            android:layout_margin="10dp"/>
    </android.support.v7.widget.CardView>
 
    <android.support.v7.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        app:cardBackgroundColor="#669900"
        app:cardCornerRadius="10dp">
    <TextView
            android:layout_width="match_parent"
            android:layout_height="70dp"
            android:text="设置背景和标签"
            android:gravity="center_horizontal|center_vertical"
            android:textSize="20sp"
            android:padding="20dp"
            android:layout_margin="10dp"/>
    </android.support.v7.widget.CardView>
 
    <android.support.v7.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        app:cardBackgroundColor="#874528"
        app:cardElevation="20dp"
        app:cardCornerRadius="10dp">
        <TextView
            android:layout_width="match_parent"
            android:layout_height="70dp"
            android:text="设置阴影"
            android:gravity="center_horizontal|center_vertical"
            android:textSize="20sp"
            android:padding="10dp"
            android:layout_margin="10dp"/>
    </android.support.v7.widget.CardView>
</LinearLayout>
```

然后新建CardViewActivity.java文件，加载上面的布局文件，填充的代码如下：

```
public class CardViewActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.cardview_layout);
    }
}
```

#### 示例2

CardView被包装为一种布局，并且经常在ListView和RecyclerView的Item布局中，作为一种容器使用。CardView应该被使用在显示层次性的内容时；在显示列表或网格时更应该被选择，因为这些边缘可以使得用户更容易去区分这些内容。

继续再上一个案例的基础上进行修改，修改后的cardview_layout.xml文件代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/cardview"
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:layout_margin="15dp">
 
    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerInParent="true"
        android:scaleType="centerCrop"
        android:src="@drawable/image_01"/>
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginBottom="10dp"
        android:layout_marginRight="10dp"
        android:clickable="true"
        android:gravity="right|bottom"
        android:text="CardView作为item使用"
        android:textColor="@android:color/white"
        android:textSize="24sp"/>
</android.support.v7.widget.CardView>
```

继续修改CardViewActivity.java文件，获得CardView组件并动态修改其属性，修改后的代码如下：

```

import android.graphics.Color;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.CardView;

public class CardViewActivity extends AppCompatActivity {
    private CardView mCardView = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.cardview_layout);
 
        mCardView = (CardView) findViewById(R.id.cardview);
 
        // 设置卡片圆角的半径大小
        mCardView.setRadius(20);
        // 设置卡片背景的颜色
        mCardView.setCardBackgroundColor(Color.RED);
        // 设置阴影部分大小
        mCardView.setCardElevation(10);
        // 设置卡片距离阴影大小
        mCardView.setContentPadding(10, 10, 10, 10);
    }
}
```

### SwipeRefreshLayout下拉刷新

SwipeRefrshLayout是Google官方更新的一个控件，可以实现下拉刷新的效果，该控件集成自ViewGroup在support-v4兼容包下。

SwipeRefrshLayout常用的几个方法如下：

- isRefreshing()：判断当前的状态是否是刷新状态。

- setColorSchemeResources(int... colorResIds)：设置下拉进度条的颜色主题，参数为可变参数，并且是资源id，可以设置多种不同的颜色，每转一圈就显示一种颜色。

- setOnRefreshListener(SwipeRefreshLayout.OnRefreshListener listener)：设置监听，需要重写onRefresh()方法，顶部下拉时会调用这个方法，在里面实现请求数据的逻辑，设置下拉进度条消失等等。

- setProgressBackgroundColorSchemeResource(int colorRes)：设置下拉进度条的背景颜色，默认白色。

- setRefreshing(boolean refreshing)：设置刷新状态，true表示正在刷新，false表示取消刷新。


使用SwipeRefrshLayout要想达到刷新的目的，首先需要在这个布局里包裹可以滑动的子控件，如ScrollView、ListView、RecyclerView等，并且只能有一个子控件。然后在代码里设置OnRefreshListener设置监听，最后在监听里设置刷新时的数据获取就可以了。

创建swiperefreshlayout_layout.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container_swipe"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
 
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:id="@+id/content_tv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:paddingTop="10dp"
            android:text="SwipeRefreshLayout下拉刷新控件"
            android:textSize="20sp"
            android:textStyle="bold"/>
    </ScrollView>
 
</android.support.v4.widget.SwipeRefreshLayout>
```

上面的代码中SwipeRefreshLayout只有一个为ScrollView的子元素，其中是一个文本框，通过下拉刷新来更新文本框里面的内容。 

 然后新建SwipeRefreshLayoutActivity.java文件，加载上面的布局文件，填充的代码如下：

```

import android.os.Bundle;
import android.os.Handler;
import android.support.v4.widget.SwipeRefreshLayout;
import android.support.v7.app.AppCompatActivity;

public class SwipeRefreshLayoutActivity extends AppCompatActivity
        implements SwipeRefreshLayout.OnRefreshListener {
    private SwipeRefreshLayout mSwipeRefreshLayout = null;
    private TextView mContentTv = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.swiperefreshlayout_layout);
 
        mSwipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.container_swipe);
        mContentTv = (TextView) findViewById(R.id.content_tv);
 
        // 设置刷新时动画的颜色，可以设置4个
        mSwipeRefreshLayout.setColorSchemeResources(
                android.R.color.holo_blue_light,
                android.R.color.holo_red_light,
                android.R.color.holo_orange_light,
                android.R.color.holo_green_light);
        // 设置下拉监听事件
        mSwipeRefreshLayout.setOnRefreshListener(this);
    }
 
    @Override
    public void onRefresh() {
        mContentTv.setText("正在刷新，请稍后。。。");
 
        // 模拟耗时更新操作
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                mContentTv.setText("刷新完毕！");
                mSwipeRefreshLayout.setRefreshing(false);
            }
        }, 5000);
    }
}
```

 上面的代码很简单，先给SwipeRefreshLayout设置了刷新时的动画颜色，然后给SwipeRefreshLayout添加一个下拉的Listener，在onRefresh()回调方法中来改变文本框里面的内容。这里使用到了一个Handler对象模拟耗时操作，操作完毕后再更新文本框里面的内容。

#### 综合示例

上面的示例将SwipeRefreshLayout和ScrollView结合起来使用，一般开发里面结合ListView和RecyclerView较多，接下来再分享一个简单结合RecyclerView的案例。

在RecyclerView数据动态更新案例的基础上来修改，首先修改布局文件，在RecyclerView的外层LinearLayout替换为SwipeRefreshLayout，修改后的recyclerview_layout.xml文件代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swiperefreshlayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
 
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</android.support.v4.widget.SwipeRefreshLayout>
```

关于RecyclerView的item布局和适配器代码不变， 为了不影响原来的代码，这里新建一个SwipeRecyclerViewActivity文件，代码如下：

```
 
import android.os.Bundle;
import android.os.Handler;
import android.support.v4.widget.SwipeRefreshLayout;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
public class SwipeRecyclerViewActivity extends AppCompatActivity
        implements SwipeRefreshLayout.OnRefreshListener {
    private SwipeRefreshLayout mSwipeView = null;
    private RecyclerView mRecyclerView = null;
    private RecyclerViewAdapter2 mAdapter = null;
    private ArrayList<String> mDatas = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.swip_recycler_view_layout);
 
        // 获取界面组件
        mSwipeView = (SwipeRefreshLayout) findViewById(R.id.swiperefreshlayout);
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
        mAdapter = new RecyclerViewAdapter2(this, mDatas);
        mRecyclerView.setAdapter(mAdapter);
        // 设置默认动画
        mRecyclerView.setItemAnimator(new DefaultItemAnimator());
 
        // 设置颜色属性的时候一定要注意是引用了资源文件还是直接设置16进制的颜色，都是int值容易搞混
        // 设置下拉进度的背景颜色，默认就是白色的
        mSwipeView.setProgressBackgroundColorSchemeResource(android.R.color.white);
        // 设置下拉进度的主题颜色
        mSwipeView.setColorSchemeResources(
                R.color.colorPrimaryDark,
                R.color.colorAccent,
                R.color.colorPrimary);
        // 下拉时触发SwipeRefreshLayout的下拉动画，动画完毕之后就会回调这个方法
        mSwipeView.setOnRefreshListener(this);
    }
 
    private void initDatas() {
        mDatas = new ArrayList<>();
 
        for (int i = 0; i < 50; i++) {
            mDatas.add(i, i + 1 + "");
        }
    }
 
    @Override
    public void onRefresh() {
        // 模拟一些比较耗时的操作，比如联网获取数据，需要放到子线程去执行
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                mAdapter.addData(0);
                mAdapter.notifyDataSetChanged();
 
                mSwipeView.setRefreshing(false);
            }
        }, 2000);
    }
}
```

上述代码首先获取布局控件，先设置RecyclerView显示的管理器和适配器，然后再设置SwipeRefreshLayout。



### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)