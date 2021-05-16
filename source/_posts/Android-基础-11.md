---
title: Android  高级控件（一）
date: 2019-04-30 20:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### 拖动条SeekBar

拖动条和进度条非常相似，只是进度条采用颜色填充来表明进度完成的程度，而拖动条则通过滑块的位置来标识数值——而且拖动条允许用户拖动滑块来改变值，因此拖动条通常用于对系统的某种数值进行调节，比如调节音量等。

<!--more-->

由于拖动条SeekBar继承了 ProgressBar,因此ProgressBar所支持的XML属|性和方法完全适用于SeekBar。

SeekBar允许用户改变拖动条的滑块外观，改变滑块外观通过如下属性来指定。

- android:thumb：指定一个Drawable对象，该对象将作为自定义滑块。


为了让程序能响应拖动条滑块位置的改变，程序可以为SeekBar绑定一个OnSeekBaiChangeListener监听器，其三个回调方法如下：

- onProgressChanged：进度发生改变时会触发。

- onStartTrackingTouch：按住SeekBar时会触发。

- onStopTrackingTouch：放开SeekBar时触发。

seekbar_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
    <SeekBar
        android:id="@+id/seekBar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:max="100"
        android:progress="50" />
    <TextView
        android:id="@+id/prompt_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <TextView
        android:id="@+id/pb_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

界面交互代码非常简单，为拖动条绑定一个监听器。新建SeekBarActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.SeekBar;
import android.widget.TextView;
 
public class SeekBarActivity extends AppCompatActivity implements SeekBar.OnSeekBarChangeListener {
    private SeekBar mSeekBar = null;
    private TextView mPromptTv, mProgressTv;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.seekbar_layout);
 
        // 获取界面组件
        mSeekBar = (SeekBar) findViewById(R.id.seekBar);
        mPromptTv = (TextView) findViewById(R.id.prompt_tv);
        mProgressTv = (TextView) findViewById(R.id.pb_tv);
 
        // 注册事件监听器
        mSeekBar.setOnSeekBarChangeListener(this);
    }
 
    // 数值改变
    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
        mPromptTv.setText("正在拖动");
        mProgressTv.setText("当前数值:" + progress);
    }
 
    // 开始拖动
    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {
        mPromptTv.setText("开始拖动");
    }
 
    // 停止拖动
    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
        mPromptTv.setText("停止拖动");
    }
}
```

同ProgressBar一样，SeekBar也是同样的道理可以自定义出来很多不同种类的效果。

### RatingBar

星级评分条与拖动条有相同的父类：AbsSeekBar，因此它们十分相似。实际上星级评分条与拖动条的用法、功能都十分接近：它们都允许用户通过拖动来改变进度。RatingBar与SeekBar的最大区别在于：RatingBar通过星星来表示进度。

RatingBar所支持的常见XML属性如下：

- android:isIndicator：是否用作指示，用户无法更改，默认false。

- android:numStars：显示多少个星星，必须为整数。

- android:rating：默认评分值，必须为浮点数。

- android:stepSize： 评分每次增加的值，必须为浮点数。


为了让程序能响应星级评分条评分的改变，可以考虑为它绑定一个OnRatingBarChangeListener监听器。

ratingbar_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:orientation="vertical">
 
    <RatingBar
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:numStars="5"
        android:rating="3.5"
        android:stepSize="0.5" />
</LinearLayout>
```

很多时候，默认的RatingBar并不能满足我们的要求，一般都是修改RatingBar的大小、图样、颜色等，也可以同ProgressBar一样自定义。

### 视图切换组件ViewSwitcher

ViewAnimator是一个基类，它继承了 FrameLayout，因此它表现出FrameLayout的特征，可以将多个View组件叠在一起。 ViewAnimator额外增加的功能正如它的名字所暗示的一样，ViewAnimator可以在View切换时表现出动画效果。

ViewAnimator及其子类的继承关系图如下图所示。

![1.jpeg](https://i.loli.net/2019/05/09/5cd434dbaec2c.jpeg)



ViewAnimator及其子类也是一组非常重要的UI组件，这种组件的主要功能是增加动画效果，从而使界面更加炫。使用ViewAnimator 时可以指定如下常见XML属性。

- android:animateFirstView：设置ViewAnimator显示第一个View组件时是否使用动画。

- android:inAnimation：设置ViewAnimator显示组件时所使用的动画。

- android:outAnimation：设置ViewAnimator隐藏组件时所使用的动画。


在实际项目中往往会使用ViewAnimator的几个子类。

ViewSwitcher代表了视图切换组件，它本身继承了 FrameLayout,因此可以将多个View 层叠在一起，每次只显示一个组件。当程序控制从一个View切换到另一个View时， ViewSwitcher支持指定动画效果。

为了给ViewSwitcher添加多个组件，一般通过调用ViewSwitcher的setFactory (ViewSwitcherViewFactory)方法为之设置 ViewFactory，并由该 ViewFactory 为之创建 View 即可。 

接下来通过一个简单的示例程序来学习ViewSwitcher的使用，主要是实现Android 5.0的Launcher界面的分屏、左右滚动效果。

viewswitcher_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
    <!-- 定义一个ViewSwitcher组件 -->
    <ViewSwitcher
        android:id="@+id/viewSwitcher"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
 
    <!-- 定义滚动到上一屏的按钮 -->
    <Button
        android:id="@+id/prev_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:text="<" />
    <!-- 定义滚动到下一屏的按钮 -->
    <Button
        android:id="@+id/next_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
	android:layout_alignParentRight="true" 
	android:text=">" />
</RelativeLayout>
```

slide_gridview.xml

```
<?xml version="1.0" encoding="utf-8"?>
<GridView xmlns:android="http://schemas.android.com/apk/res/android"
          android:layout_width="match_parent"
          android:numColumns="4"
          android:layout_height="match_parent">
</GridView>
```

创建GridView中每个Item的布局文件slide_gridview_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<!-- 定义一个垂直的LinearLayout，该容器中放置一个ImageView和一个TextView -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center"
              android:orientation="vertical"
              android:padding="10dp">
 
    <ImageView
        android:id="@+id/icon_img"
        android:layout_width="48dp"
        android:layout_height="48dp" />
 
    <TextView
        android:id="@+id/name_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center" />
</LinearLayout>
```

 为了方便操作，将GridView内使用的内容定义为实体类ViewSwitcherItemData，代码如下：

```

public class ViewSwitcherItemData {
    private String name; // 应用程序名称
    private int icon;  // 应用程序图标
 
    public ViewSwitcherItemData(String name, int icon) {
        this.name = name;
        this.icon = icon;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getIcon() {
        return icon;
    }
 
    public void setIcon(int icon) {
        this.icon = icon;
    }
}
```

接下来创建GridView的适配器ViewSwitcherBaseAdapter，继承BaseAdapter类，代码如下： 

```

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;
 
import java.util.List;

public class ViewSwitcherBaseAdapter extends BaseAdapter {
    private Context mContext = null;
    private List<ViewSwitcherItemData> mItemDatas = null;
 
    public ViewSwitcherBaseAdapter(Context context, List<ViewSwitcherItemData> itemDatas) {
        this.mContext = context;
        this.mItemDatas = itemDatas;
    }
 
    @Override
    public int getCount() {
        // 如果已经到了最后一屏，且应用程序的数量不能整除NUMBER_PER_SCREEN
        if (ViewSwitcherActivity.screenNo == ViewSwitcherActivity.screenCount - 1
                && mItemDatas.size() % ViewSwitcherActivity.NUMBER_PER_SCREEN != 0) {
            // 最后一屏显示的程序数为应用程序的数量对NUMBER_PER_SCREEN求余
            return mItemDatas.size() % ViewSwitcherActivity.NUMBER_PER_SCREEN;
        }
        // 否则每屏显示的程序数量为NUMBER_PER_SCREEN
        return ViewSwitcherActivity.NUMBER_PER_SCREEN;
    }
 
    @Override
    public ViewSwitcherItemData getItem(int position) {
        // 根据screenNo计算第position个列表项的数据
        return mItemDatas.get(
                ViewSwitcherActivity.screenNo * ViewSwitcherActivity.NUMBER_PER_SCREEN + position);
    }
 
    @Override
    public long getItemId(int position) {
        return position;
    }
 
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        if (null == convertView) {
            // 加载R.layout.item布局文件
            convertView = LayoutInflater.from(mContext).inflate(R.layout.slide_gridview_item, null);
            holder = new ViewHolder();
            holder.iconImg = (ImageView)convertView.findViewById(R.id.icon_img);
            holder.nameTv = (TextView)convertView.findViewById(R.id.name_tv);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
 
        holder.iconImg.setImageResource(getItem(position).getIcon());
        holder.nameTv.setText(getItem(position).getName());
        return convertView;
    }
 
    private class ViewHolder{
        ImageView iconImg;
        TextView nameTv;
    }
}
```

使用扩展BaseAdapter的方式为GridView提供Adapter，关键就是根据用户单击的按钮来动态计算该BaseAdapter应该显示哪些程序列表。

使用Activity类的screenNo保存当前正在显示第几屏的程序列表，BaseAdapter会根据screenNo 动态计算该Adapter总共包含多少个列表项（如getCount()方法所示），会根据screenNo计算每个列表项的数据（如getltem(int position)方法所示）。

新建ViewSwitcherActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.GridView;
import android.widget.ViewSwitcher;
 

import java.util.ArrayList;
import java.util.List;

public class ViewSwitcherActivity extends AppCompatActivity implements View.OnClickListener {
    // 定义一个常量，用于显示每屏显示的应用程序数
    public static final int NUMBER_PER_SCREEN = 12;
    // 记录当前正在显示第几屏的程序
    public static int screenNo = -1;
    // 保存程序所占的总屏数
    public static int screenCount;
 
    private ViewSwitcher mViewSwitcher = null;
    private Button mPrevBtn = null;
    private Button mNextBtn = null;
    private ViewSwitcherBaseAdapter mAdapter = null;
    // 保存系统所有应用程序的List集合
    private List<ViewSwitcherItemData> mItemDatas = new ArrayList<ViewSwitcherItemData>();
 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.viewswitcher_layout);
 
        // 获取界面组件
        mViewSwitcher = (ViewSwitcher) findViewById(R.id.viewSwitcher);
        mPrevBtn = (Button) findViewById(R.id.prev_btn);
        mNextBtn = (Button) findViewById(R.id.next_btn);
 
        mViewSwitcher.setFactory(new ViewSwitcher.ViewFactory() {
            // 实际上就是返回一个GridView组件
            @Override
            public View makeView() {
                // 加载R.layout.slide_gridview组件，实际上就是一个GridView组件
                return ViewSwitcherActivity.this.getLayoutInflater()
                        .inflate(R.layout.slide_gridview, null);
            }
        });
 
        // 创建一个包含40个元素的List集合，用于模拟包含40个应用程序
        for (int i = 0; i < 40; i++) {
            ViewSwitcherItemData item =
                    new ViewSwitcherItemData("item" + i, R.mipmap.ic_launcher);
            mItemDatas.add(item);
        }
 
        // 计算应用程序所占的总屏数
        // 如果应用程序的数量能整除NUMBER_PER_SCREEN，除法的结果就是总屏数
        // 如果不能整除，总屏数应该是除法的结果再加1
        screenCount = mItemDatas.size() % NUMBER_PER_SCREEN == 0 ?
                mItemDatas.size() / NUMBER_PER_SCREEN :
                mItemDatas.size() / NUMBER_PER_SCREEN + 1;
 
        mAdapter = new ViewSwitcherBaseAdapter(this, mItemDatas);
 
        mPrevBtn.setOnClickListener(this);
        mNextBtn.setOnClickListener(this);
 
        // 页面加载时先显示第一屏
        next();
    }
 
    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.prev_btn:
                prev();
                break;
            case R.id.next_btn:
                next();
                break;
            default:
                break;
        }
    }
 
    private void next() {
        if (screenNo < screenCount - 1) {
            screenNo++;
            // 为ViewSwitcher的组件显示过程设置动画
            mViewSwitcher.setInAnimation(this, R.anim.slide_in_right);
            // 为ViewSwitcher的组件隐藏过程设置动画
            mViewSwitcher.setOutAnimation(this, R.anim.slide_out_left);
            // 控制下一屏将要显示的GridView对应的Adapter
            ((GridView) mViewSwitcher.getNextView()).setAdapter(mAdapter);
            // 单击右边按钮，显示下一屏
            // 学习手势检测后，也可通过手势检测实现显示下一屏
            mViewSwitcher.showNext();
        }
    }
 
    private void prev() {
        if (screenNo > 0) {
            screenNo--;
            // 为ViewSwitcher的组件显示过程设置动画
            mViewSwitcher.setInAnimation(this, android.R.anim.slide_in_left);
            // 为ViewSwitcher的组件隐藏过程设置动画
            mViewSwitcher.setOutAnimation(this, android.R.anim.slide_out_right);
            // 控制下一屏将要显示的GridView对应的 Adapter
            ((GridView) mViewSwitcher.getNextView()).setAdapter(mAdapter);
            // 单击左边按钮，显示上一屏，当然可以采用手势
            // 学习手势检测后，也可通过手势检测实现显示上一屏
            mViewSwitcher.showPrevious();
        }
    }
}
```

重点在于为ViewSwitcher设置ViewFactory对象，并且当用户单击“<”和“>”两个按钮时控制ViewSwitcher显示“上一屏”和“下一屏”的应用程序。 

当用户单击按钮时，程序的事件处理方法将会控制ViewSwitcher调用showNext() 方法显示下一屏的程序列表。而且此时screenNo被加1，因而Adapter将会动态计算下一屏的程序列表，再将该Adapter传给ViewSwitcher接下来要显示的GridView。

为了实现ViewSwitcher切换View时的动画效果，程序的事件处理方法中调用了 ViewSwitcher的setInAnimation()、setOutAnimation()方法来设置动画效果。本程序由于 Android系统只提供的两个slide_in_left、slide_out_right动画资源，需要自行提供slide_in_right、slide_out_left动画资源，详细会在Android高级部分进行讲解。

其中R.anim.slide_in_right动画资源对应的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 设置从右边拖进来的动画
    android:duration指定动画持续时间  -->
    <translate
        android:fromXDelta="100%p"
        android:toXDelta="0"
        android:duration="@android:integer/config_mediumAnimTime" />
</set>
```

R.anim.slide_out_left动画资源对应的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 设置从左边拖出去的动画
    android:duration指定动画持续时间 -->
    <translate
        android:fromXDelta="0"
        android:toXDelta="-100%p"
        android:duration="@android:integer/config_mediumAnimTime" />
</set>
```

### ImageSwitcher

  ImageSwitcher和ImageSwitcher继承了 ViewSwitcher，因此它具有与ViewSwitcher相同的特征：可以在切换View组件时使用动画效果。ImageSwitcher继承了 ViewSwitcher，并重写了 ViewSwitcher 的 showNext()、showPrevious()方法，因此 ImageSwitcher 使用起来更加简单。

使用 ImageSwitcher 只要如下两步即可。

- 为 ImageSwitcher 提供一个 ViewFactory，该 ViewFactory 生成的 View 组件必须是 ImageView。

- 需要切换图片时，只要调用 ImageSwitcher 的 setImageDrawable(Drawable drawable)、 setImageResource(int resid)和 setImageURI(Uri uri)方法更换图片即可。

创建imageswitcher_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
 
    <ImageSwitcher
        android:id="@+id/switcher"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:inAnimation="@android:anim/slide_in_left"
        android:outAnimation="@android:anim/slide_out_right"/>
</RelativeLayout>
```

上面界面布局文件中的粗体字代码定义了一个ImageSwitcher，并通过android:inAnimation 和android:outAnimation指定了图片切换时的动画效果。

ImageSwitcher的使用一个最重要的地方就是需要为它指定一个ViewFactory，也就是定义它是如何把内容显示出来的，一般做法为在使用ImageSwitcher的该类中实现ViewFactory接口并覆盖对应的makeView方法。新建ImageSwitcherActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.view.ViewGroup;
import android.view.animation.AnimationUtils;
import android.widget.ImageSwitcher;
import android.widget.ImageView;
import android.widget.ViewSwitcher;
 

public class ImageSwitcherActivity extends AppCompatActivity implements ViewSwitcher.ViewFactory {
    private ImageSwitcher mImageSwitcher = null;
    private int[] mImageIds = {
            R.drawable.image_01, R.drawable.image_02, R.drawable.image_03,
            R.drawable.image_04, R.drawable.image_05
    };
    private int mIndex = 0;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.imageswitcher_layout);
 
        // 获取界面组件
        mImageSwitcher = (ImageSwitcher) findViewById(R.id.switcher);
 
        // 为他它指定一个ViewFactory，也就是定义它是如何把内容显示出来的，
        // 实现ViewFactory接口并覆盖对应的makeView方法。
        mImageSwitcher.setFactory(this);
        // 添加动画效果
        mImageSwitcher.setInAnimation(AnimationUtils.loadAnimation(this,
                android.R.anim.fade_in));
        mImageSwitcher.setOutAnimation(AnimationUtils.loadAnimation(this,
                android.R.anim.fade_out));
 
        // 为ImageSwitcher绑定监听事件
        mImageSwitcher.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                mIndex ++;
                if(mIndex >= mImageIds.length){
                    mIndex = 0;
                }
                mImageSwitcher.setImageResource(mImageIds[mIndex]);
            }
        });
 
        mImageSwitcher.setImageResource(mImageIds[0]);
    }
 
    @Override
    public View makeView() {
        ImageView imageView = new ImageView(this);
        imageView.setBackgroundColor(0xFF000000);
        // 设置填充方式
        imageView.setScaleType(ImageView.ScaleType.FIT_XY);
        imageView.setLayoutParams(new ImageSwitcher.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
        return imageView;
    }
}
```

### TextSwitcher

TextSwitcher继承了 ViewSwitcher，因此它具有与ViewSwitcher相同的特征：可以在切换 View组件时使用动画效果。与ImageSwitcher相似的是，使用TextSwitcher也需要设置一个 ViewFactory。与 ImageSwitcher 不同的是，TextSwitcher 所需的 ViewFactory 的 makeView()方法必须返回一个TextView组件。

TextSwitcher与TextView的功能有点相似，它们都可用于显示文本内容，区别在于TextSwitcher的效果更炫，它可以指定文本切换时的动画效果。

textwitcher_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8" ?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent" >
    <!-- 定义一个TextSwitcher，并指定了文本切换时的动画效果 -->
    <TextSwitcher
        android:id="@+id/textSwitcher"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inAnimation="@android:anim/slide_in_left"
        android:outAnimation="@android:anim/slide_out_right" />
</LinearLayout>
```

接下来Activity只要为TextSwitcher设置ViewFactory，该TextSwitcher即可正常工作。新建TextSwitcherActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.graphics.Color;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.TextSwitcher;
import android.widget.TextView;
import android.widget.ViewSwitcher;

public class TextSwitcherActivity extends AppCompatActivity {
    private TextSwitcher mTextSwitcher = null;
    private String[] mContents = {
            "你好", "HelloWorld", "Good!!!", "TextSwitcher", "你会了吗？"
    };
    private int mIndex = 0;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.textwitcher_layout);
 
        mTextSwitcher = (TextSwitcher) findViewById(R.id.textSwitcher);
        mTextSwitcher.setFactory(new ViewSwitcher.ViewFactory() {
            @Override
            public View makeView() {
                TextView tv = new TextView(TextSwitcherActivity.this);
                tv.setTextSize(40);
                tv.setTextColor(Color.MAGENTA);
                return tv;
            }
        });
 
        mTextSwitcher.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                mTextSwitcher.setText(mContents[mIndex++ % mContents.length]);
            }
        });
 
        mTextSwitcher.setText(mContents[0]);
    }
}
```

上面的粗体字代码重写了 ViewFactory的makeView() 方法，该方法返回了一个TextView，这样即可让 
TextSwitcher正常工作。当程序要切换TextSwitcher显示文本时，调用TextSwitcher的setText()方法修改文本即可。

### 翻转视图ViewFlipper打造引导页和轮播图

 ViewFlipper组件继承了 ViewAnimator，它可调用addView(View v)添加多个组件，一旦向 ViewFlipper中添加了多个组件之后，ViewFlipper就可使用动画控制多个组件之间的切换效果。

ViewFlipper与前面介绍的AdapterViewFlipper有较大的相似性，它们可以控制组件切换的动画效果。它们的区别是：ViewFlipper需要开发者通过addView(View v)添加多个View，而 AdapterViewFlipper则只要传入一个Adapter，Adapter将会负责提供多个View。因此ViewFlipper 可以指定与 AdapterViewFlipper 相同的 XML 属性。

ViewFlipper组件的一些常用方法如下：

- setInAnimation：设置View进入屏幕时使用的动画。

- setOutAnimation：设置View退出屏幕时使用的动画。

- showNext：调用该方法来显示ViewFlipper里的下一个View。

- showPrevious：调用该方法来显示ViewFlipper的上一个View。

- setFilpInterval：设置View之间切换的时间间隔。

- setFlipping：使用上面设置的时间间隔来开始切换所有的View，切换会循环进行。

- stopFlipping：停止View切换。

创建viewflipper_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ViewFlipper
        android:id="@+id/details"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:flipInterval="1000">
        <ImageView
            android:src="@drawable/image_01"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:scaleType="fitXY">
        </ImageView>
        <ImageView
            android:src="@drawable/image_02"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:scaleType="fitXY">
        </ImageView>
        <ImageView
            android:src="@drawable/image_03"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:scaleType="fitXY">
        </ImageView>
    </ViewFlipper>
    <Button
        android:text="<"
        android:onClick="prev"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerInParent="true"
        android:onClick="auto"
        android:text="自动播放"/>
    <Button
        android:text=">"
        android:onClick="next"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"/>
</RelativeLayout>
```

上面的界面布局文件中定义了一个ViewFlipper，并在该ViewFlipper中定义了三个 ImageView，这意味着该ViewFlipper包含了三个子组件。

接下来在Activity代码中即可调用 ViewFlipper 的 showPrevious()、showNext()等方法控制 ViewFlipper 显示上一个、下一个子组件。为了控制组件切换时的动画效果，还需要调用ViewFlipper的setlnAnimation()、setOutAnimation() 方法设置动画效果。新建ViewFlipperActivity.java文件，加载上面新建的布局文件，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.ViewFlipper;

public class ViewFlipperActivity extends AppCompatActivity {
    private ViewFlipper mViewFlipper = null;
 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.viewflipper_layout);
        mViewFlipper = (ViewFlipper) findViewById(R.id.details);
    }
 
    // 上一个
    public void prev(View source) {
        mViewFlipper.setInAnimation(this, R.anim.slide_in_right);
        mViewFlipper.setOutAnimation(this, R.anim.slide_out_left);
        // 显示上一个组件
        mViewFlipper.showPrevious();
        // 停止自动播放
        mViewFlipper.stopFlipping();
    }
 
    // 下一个
    public void next(View source) {
        mViewFlipper.setInAnimation(this, android.R.anim.slide_in_left);
        mViewFlipper.setOutAnimation(this, android.R.anim.slide_out_right);
        // 显示下一个组件
        mViewFlipper.showNext();
        // 停止自动播放
        mViewFlipper.stopFlipping();
    }
 
    // 自动播放
    public void auto(View source) {
        mViewFlipper.setInAnimation(this, android.R.anim.slide_in_left);
        mViewFlipper.setOutAnimation(this, android.R.anim.slide_out_right);
        // 开始自动播放
        mViewFlipper.startFlipping();
    }
}
```

上面程序中的代码就是控制ViewFlipper切换组件的动画效果，以及控制ViewFlipper切换组件的关键代码。

在该例子中使用了静态导入ViewFlipper组件页面，实际开发中也可以通过addView动态添加。如果加入手势左右滑动操作，就打造出了应用程序启动的时候经常用到的引导页面；如果同该例子一样使用自动播放，那么就非常容易实现如图片轮播等行为。







### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)