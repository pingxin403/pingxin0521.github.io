---
title: Android 常用基本控件
date: 2019-04-28 13:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### View

在Android中，什么是View？View是Android中所有控件的基类，不管是简单的TextView，Button还是复杂的LinearLayout和ListView，它们的共同基类都是View；

View是一种界面层的控件的一种抽象，它代表了一个控件，除了View还有ViewGroup，从名字来看ViewGroup可以翻译为控件组，即一组View；
<!--more-->
在Android中，ViewGroup也继承了View，这就意味着View可以是单个控件，也可以是由多个控件组成的一组控件；

Android应用的所有UI组件都是继承了View类，View组件非常类似于Swing编程的JPanel，它表示一个空白的矩形区域。View类还有一个很重要的子类：ViewGroup，但是ViewGroup通常作为其他组件的容器使用。Android的所有UI组件都是建立在View、ViewGroup基础之上的，Android采用了“组合器”模式来设计View和ViewGroup：ViewGroup是View的子类，因此ViewGroup也可以被当做View使用。对于一个Android应用的图形用户界面来说，ViewGroup作为容器来盛装其他组件，而ViewGroup里除了可以包含普通View组件之外，还可以再次包含ViewGroup组件。

![1.jpeg](https://i.loli.net/2019/05/07/5cd149d2acf45.jpeg)

 View和ViewGroup中有许多方法都值得我们去研究，不管是处理view的滑动冲突还是明白view的工作原理以至于我们要自定义view，都起到至关重要的作用。这里只是表明了它们之间的关系，具体到一些重要的方法函数，我会逐个介绍。



#### View的位置参数

View和位置主要由它的四个顶点来决定，分别对应View的四个属性：top、left、right、bottom，top是左上角纵坐标，left是左上角横坐标，right是右下角横坐标，bottom是右下角纵坐标；对应如图所示：

![1.png](https://i.loli.net/2019/05/07/5cd14302d98e5.png)

根据上图我们可以得到View的宽高和坐标的关系；
```
width = right - left;
hight = bottom - top;
```
如何得到View的这四个参数呢？
```
left = getLeft();
right = getRight();
top = getTop();
bottom = getBottom();
```
注：从Android 3.0开始，View增加了几个额外的参数：x，y，translationX和translationY，其中xy是View左上角的坐标，而translationX和translationY是View左上角相对于父容器的偏移量。这几个参数也是相对于父容器的坐标；这几个参数的换算关系如下：
```
x = left + translationX;
y = top + translationY;
```
**静态坐标系**

![](https://i.loli.net/2019/05/07/5cd14c12504ec.png)

MationEvent 触摸事件坐标：
![](https://i.loli.net/2019/05/07/5cd14c125c184.png)

**滑动坐标系**

![3.png](https://i.loli.net/2019/05/07/5cd14c126e814.png)

用一张表格具体看看相关参数的含义：

![](https://i.loli.net/2019/05/07/5cd14c127c318.png)

#### MotionEvent

当手指触摸屏幕是（View或ViewGroup派生的控件），将产生Touch事件。Touch事件的相关细节（触摸发生的位置、事件及怎么的触摸）被封装成MotionEvent对象。
在手指触摸屏幕后所产生的一系列Touch事件中，典型的事件类型如下几种：

|事件类型 |	具体动作|
|--- | --- |
|MotionEvent.ACTION_DOWN| 	手指刚接触屏幕|
|MotionEvent.ACTION_UP |	手指从屏幕上松开的一瞬间|
|MotionEvent.ACTION_MOVE| 	手指在屏幕上移动|
|MotionEvent.ACTION_CANCEL| 	动作取消（非人为）|

正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，一般会有两种情况：
（1）点击屏幕后离开松开，事件序列为 DOWN -> UP;
（2）点击屏幕滑动一会再松开，事件序列为 DOWN -> MOVE ->….-> MOVE -> UP;
在上述典型的三种Tocuh事件，我们可以通过TotionEvent对象得到点击事件发生的x和y的坐标。为此，系统提供了两组方法：getX/getY和getRawX/getRawY。区别如下：
（1）getX/getY：返回的是相对于当前View左上角的 x 和 y 坐标
（2）getRawX/getRawY ： 返回的是相对于手机屏幕左上角的 x 和 y 坐标

**TouchSlop**

TouchSlop是系统所能识别出的被认为是滑动的最小距离，即当手指在屏幕上滑动时，如果两次滑动之间的距离小于这个常量，那么系统就不认为你是在进行滑动操作。原因是：滑动的距离太短，系统不认为它不是滑动。这是个常量，和设备有关，在不同的设备上这个值是不同的，可通过：ViewConfiguration.get(getContext()).getScaledTouchSlop()来获取这个常量。
这个常量的意义在于：当我们在处理滑动时，可以利用这个常量来做一些过滤，比如当两次滑动事件的滑动距离小于这个值，我们就可以认为未达到滑动距离的临界值，因此就可以认为它们不是滑动，这样做可以有更好的用户体验。

#### View 的滑动


**使用scrollTo/scrollBy**

首先我们创建一个布局，有一个TextView和一个Button，当我们点击Button时，移动TextView；
当我们调用scrollTo方法时
```
mTv.scrollTo(-100,-100);
```
这里我们可以看到TextView中的text值 Hello World！移动到（-100,-100）的位置，TextView本身并没有移动，所以scrollTo方法移动的只是View中的内容，而非View本身；

这里说一下scrollTo和scrollBy的区别，scrollTo是基于所传递参数的绝对移动，而scrollBy是基于当前位置的相对移动；就是To是我就移动到这个位置就不动了，By是基于我当前的位置继续偏移；


上面我们知道了什么是View以及View坐标的关系和如何获取坐标值，以及View的移动，接下来我们利用上面的知识来做一个小功能来巩固以下，需求是创建一个自定义的TextView，然后在屏幕上随我们的手指滑动而滑动；具体实现思路如下：
注：（这里需要用到View的触摸事件处理，要重写View的onTouchEvent方法，不理解的同学可以自行查阅相关资料）
1. 手指按下时我们获取到当前手指的坐标点；
2. 手指滑动时我们计算手指的偏移量并根据偏移量移动View
自定义的TextView完全代码如下：

```
@SuppressLint("AppCompatCustomView")
public class MyTextView extends TextView {
    private int mX;
    private int mY;
    float moveX;
    float moveY;

    public MyTextView(Context context) {
        super(context);
    }

    public MyTextView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public MyTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN://手指按下
                //按下的时候获取手指触摸的坐标
                mX = (int) event.getRawX();
                mY = (int) event.getRawY();
                break;
            case MotionEvent.ACTION_MOVE://手指滑动
                //滑动时计算偏移量
                moveX = event.getRawX() - mX;
                moveY = event.getRawY() - mY;
                //随手指移动
                setTranslationX(moveX);
                setTranslationY(moveY);
                break;
            case MotionEvent.ACTION_UP://手指松开
                break;
        }
        return true;
    }
}
```

**使用动画**

还是上面的初始布局，这次我们来用动画移动View，首先我们创建一个位移动画xml文件
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fillAfter="true">

    <translate
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="100"
        android:toYDelta="100"></translate>
</set>
```
延x轴和y轴平移100的距离
```
Animation animation = AnimationUtils.loadAnimation(this, R.anim.move);
mTv.startAnimation(animation);
```
可以看到动画实现了View的位移，移动的是View本身，而非View中的内容；
另外Android3.0版本之后我们也可以通过View本身的setTranslationX和setTranslationY方法来实现View的滑动，例如实现上面的滑动效果我们也可以用以下代码：
```
mTv.setTranslationX(100);
 mTv.setTranslationY(100);
```
即View移动到(100,100)的位置，注意该移动方式为绝对移动，即类似于scrollTo;

**Scroller**

使用scrollTo()和scrollBy()来实现 View 的滑动的时候并不多，因为这两个方法产生的滑动时不连贯的，跳跃的，最终的效果也不够平滑。所以，Android提供了Scroller这个类来实现平滑的滑动。下面的也是让自定义的 View 跟随手指移动的核心代码为：

```
//初始化Scroller
Scroller scroller = new Scroller(context);

// 分别记录上次滑动的坐标
 private int mLastX = 0;
 private int mLastY = 0;

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        int rawX = (int)event.getRawX();
        int rawY = (int)event.getRawY();

        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                Log.i("MotionEvent","MotionEvent.ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i("MotionEvent","MotionEvent.ACTION_MOVE");
                int offsetX = rawX - mLastX;
                int offsetY = rawY - mLastY;

                smoothScrollTo(-offsetX,-offsetY);
                break;
            case MotionEvent.ACTION_UP:
                Log.i("MotionEvent","MotionEvent.ACTION_UP");
                break;
        }
        mLastX = rawX;
        mLastY = rawY;
       return super.onTouchEvent(event);
    }

 private void smoothScrollTo(int destX,int  destY){
        scroller.startScroll(0,0,destX,destY,1000);
        invalidate();
 }

 @Override
 public void computeScroll() {
        // 判断Scroller是否执行完毕
        if(scroller.computeScrollOffset()){
             //使整个控件都滑动
            ((View)getParent()).scrollBy(scroller.getCurrX(),scroller.getCurrY());
            // 通过重绘来不断调用computeScroll
            invalidate();
        }
 }
```
startScroll()来开启平滑移动过程，前两个参数的意思是起始坐标，接下来的两个坐标是偏移量，最后一个是显示的时长。Scroller类提供了 computeScrollOffset()方法来判断是否完成了滑动，同时也提供了 getCurrX()和getCurrY()来获取当前滑动坐标。需要注意的是，invalidate()方法，因为只能在computeScroll()方法中获取滑动过程中的scrollX和scrollY的坐标。

但是computeScroll()方法是不会自动调用的，只能通过 invalidate() –>draw() –> computeScroll() 来间接调用computeScroll()方法，所以，需要在上述代码中调用 invalidate(),实现循环获取 scrollX和scrollY的目的。当滑动结束后，scroller.computeScrollOffset()方法会返回false，从而实现整个平滑移动的过程。

**layout**

在View进行绘制时，会调用 onLayout()方法来设置显示的位置。当热，可以通过修改View的 left，top，right，bottom 四个属性来控制View 的坐标。在onTouchEvent()方法时调用onLayout()方法。下面的也是让自定义的 View 跟随手指移动的核心代码为：
```

// 分别记录上次滑动的坐标
 private int mLastX = 0;
 private int mLastY = 0;

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        int rawX = (int)event.getRawX();
        int rawY = (int)event.getRawY();

        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                Log.i("MotionEvent","MotionEvent.ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i("MotionEvent","MotionEvent.ACTION_MOVE");
                int offsetX = rawX - mLastX;
                int offsetY = rawY - mLastY;

                layout(getLeft()+offsetX,
                        getTop()+offsetY,
                        getRight()+offsetX,
                        getBottom()+offsetY);
                break;
            case MotionEvent.ACTION_UP:
                Log.i("MotionEvent","MotionEvent.ACTION_UP");
                break;
        }
        mLastX = rawX;
        mLastY = rawY;
       return super.onTouchEvent(event);
    }
```
**offsetLeftAndRight 和 offsetTopAndBottom**

这个方法相当于系统提供的一个对上下、左右移动的封装。当计算出偏移量后，只要使用如下的自定义View 也可以跟随手指移动：
```

// 分别记录上次滑动的坐标
 private int mLastX = 0;
 private int mLastY = 0;

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        int rawX = (int)event.getRawX();
        int rawY = (int)event.getRawY();

        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                Log.i("MotionEvent","MotionEvent.ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i("MotionEvent","MotionEvent.ACTION_MOVE");
                int offsetX = rawX - mLastX;
                int offsetY = rawY - mLastY;

                // 同时对left和right进行偏移
                offsetLeftAndRight(offsetX);
                // 同时对top和bottom进行偏移
                offsetTopAndBottom(offsetY);
                break;
            case MotionEvent.ACTION_UP:
                Log.i("MotionEvent","MotionEvent.ACTION_UP");
                break;
        }
        mLastX = rawX;
        mLastY = rawY;
       return super.onTouchEvent(event);
    }
```

**LayoutParams**

LayoutParams保存了一个View的布局参数。因此可以在程序中，通过改变LayoutParams来动态地修改一个布局的位置参数，从而达到改变View位置的效果。我们可以很方便地在程序中使用getLayoutParams()来获取一个View的LayouParams。当然，计算偏移量的方法与在Layout方法中计算offset也是一样。当获取到偏移量之后，就可以通过setLayoutParams来改变其LayoutParams，代码如下所示。

```
// 分别记录上次滑动的坐标
 private int mLastX = 0;
 private int mLastY = 0;

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        int rawX = (int)event.getRawX();
        int rawY = (int)event.getRawY();

        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                Log.i("MotionEvent","MotionEvent.ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i("MotionEvent","MotionEvent.ACTION_MOVE");
                int offsetX = rawX - mLastX;
                int offsetY = rawY - mLastY;

//ConstraintLayout.LayoutParams layoutParams = (ConstraintLayout.LayoutParams) getLayoutParams();①
//ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();②

                LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
                layoutParams.leftMargin = getLeft() + offsetX;
                layoutParams.topMargin = getTop() + offsetY;
                Log.i("MotionEvent","MotionEvent.ACTION_MOVE leftMargin:"+(getLeft() + offsetX));
                Log.i("MotionEvent","MotionEvent.ACTION_MOVE topMargin:"+(getTop() + offsetY));
                setLayoutParams(layoutParams);
                break;
            case MotionEvent.ACTION_UP:
                Log.i("MotionEvent","MotionEvent.ACTION_UP");
                break;
        }
        mLastX = rawX;
        mLastY = rawY;
       return super.onTouchEvent(event);
    }

```
通过getLayoutParams()获取LayoutParams时，需要根据View所在父布局的类型来设置不同的类型，比如这里将View放在LinearLayout中，那么就可以使用LinearLayout.LayoutParams。类似地，如果在RelativeLayout中，就要使用RelativeLayout.LayoutParams。当然，这一切的前提是你必须要有一个父布局，不然系统不法获取LayoutParams。不过这里需要注意的是，①和②两种方式的布局不能移动自定义View。

Android提供的布局类都是LayoutParams的子类，LayoutParams的子类主要有：

-  AbsListView.LayoutParams

-  AbsoluteLayout.LayoutParams

-  FrameLayout.LayoutParams

- LinearLayout.LayoutParams

-  RadioGroup.LayoutParams

-  RelativeLayout.LayoutParams

-  TableLayout.LayoutParams

-  TableRow.LayoutParams

-  ViewGroup.MarginLayoutParams

- WindowManager.LayoutParams


其中常用的是RelativeLayout.LayoutParams、LinearLayout.LayoutParams、ViewGroup.MarginLayoutParams。将会在后续内容中陆续学习，此处不在赘述。

### 自定义View

很多初入Android开发的程序员，对于Android自定义View可能比较恐惧，但这又是高手进阶的必经之路，这里先不做过多学习，只是简单了解。

如果说要按类型来划分的话，自定义View的实现方式大概可以分为三种：自绘控件、组合控件、以及继承控件。

- 自绘控件：内容都是开发者自己绘制出来的，一般在View的onDraw方法中完成绘制。

- 组合控件：就是将一些小的控件组合起来形成一个新的控件，这些小的控件多是系统自带的控件。比如很多应用中普遍使用的标题栏控件，其实用的就是组合控件。

- 继承控件：继承已有的控件，创建新控件，保留继承的父控件的特性，并且还可以引入新特性。



介于目前掌握的Android基础知识较为薄弱，本节先简单学习一下自绘控件。首先定义一个继承View基类的子类，然后重写View 类的一个或多个方法。通常可以被用户重写的方法如下。

- 构造器：重写构造器是定制View的最基本方式，当Java代码创建一个View实例，或根据XML布局文件加载并构建界面时将需要调用该构造器。

- onFinishInflate()：这是一个回调方法，当应用从XML布局文件加载该组件并利用它 来构建界面之后，该方法将会被回调。

- onMeasure(int, int)：调用该方法来检测View组件及其所包含的所有子组件的大小。

- onLayout(boolean, int, int, int, int)：当该组件需要分配其子组件的位置、大小时，该方法就会被回调。

- onSizeChanged(int, int, int, int)：当该组件的大小被改变时回调该方法。

- onDraw(Canvas)：当该组件将要绘制它的内容时回调该方法进行绘制。

- onKeyDown(int, KeyEvent)：当某个键被按下时触发该方法。

- onKeyUp(int, KeyEvent)：当松开某个键时触发该方法。

- onTrackballEvent(MotionEvent):当发生轨迹球事件时触发该方法。

- onTouchEvent(MotionEvent)：当发生触摸屏事件时触发该方法。

- onFocusChanged (boolean gainFocus, int direction, Rect previouslyFocusedRect)：当该组件焦点发生改变时触发该方法。

- onWindowFocusChanged(boolean)：当包含该组件的窗口失去或得到焦点时触发该方法。

- onAttachedToWindow()：当把该组件放入某个窗口时触发该方法。

- onDetachedFromWindow()：当把该组件从某个窗口上分离时触发该方法。

- onWindowVisibilityChanged(int)：当包含该组件的窗口的可见性发生改变时触发该方法。

当需要开发自定义View时，开发者并不需要重写上面列出的所有方法，而是可以根据业务需要重写其中部分方法。

下面就实现一个简单的计数器，每点击它一次，计数值就加1并显示出来。

```
import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.util.AttributeSet;
import android.view.View;

public class CounterView extends View {
    // 定义画笔
    private Paint mPaint;
    // 用于获取文字的宽和高
    private Rect mBounds;
    // 计数值，每点击一次本控件，其值增加1
    private int mCount = 0;
 
    public CounterView(Context context, AttributeSet attrs) {
        super(context, attrs);
 
        // 初始化画笔、Rect
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mBounds = new Rect();
        // 本控件的点击事件
        setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                mCount ++;
 
                // 重绘
                invalidate();
            }
        });
    }
 
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setColor(Color.BLUE);
        // 绘制一个填充色为蓝色的矩形
        canvas.drawRect(0, 0, getWidth(), getHeight(), mPaint);
 
        mPaint.setColor(Color.YELLOW);
        mPaint.setTextSize(50);
        String text = String.valueOf(mCount);
        // 获取文字的宽和高
        mPaint.getTextBounds(text, 0, text.length(), mBounds);
        float textWidth = mBounds.width();
        float textHeight = mBounds.height();
 
        // 绘制字符串
        canvas.drawText(text, getWidth() / 2 - textWidth / 2, getHeight() / 2
                + textHeight / 2, mPaint);
    }
}
```

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <CounterView
        android:id="@+id/counter_view"
        android:layout_width="100dp"
        android:layout_height="100dp"/>
</LinearLayout>
```






### 参考：
1. [Android View的绘制流程](https://www.jianshu.com/p/5a71014e7b1b)
2. [Android中View的知识体系之基础知识](https://blog.csdn.net/wanliguodu/article/details/81412951)
3. [安卓自定义View进阶-手势检测(GestureDetector)](https://www.gcssloop.com/customview/gestruedector)

4. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)