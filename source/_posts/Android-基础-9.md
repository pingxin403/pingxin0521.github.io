---
title: Android 事件处理
date: 2019-04-30 13:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### 事件处理

Android提供了两种方式的事件处理：基于回调的事件处理和基于监听的事件处理。

- 基于监听的事件处理：主要做法就是为Android界面组件绑定特定的事件监听器，前面小节已经见到大量这种事件处理的示例。

- 基于回调的事件处理：主要做法就是重写Android组件特定的回调方法， 或者重写Activity的回调方法。Android为绝大部分界面组件都提供了事件响应的回调方法，开发者只要重写它们即可。

<!--more-->

一般来说，基于回调的事件处理可用于处理一些具有通用性的事件，基于回调的事件处理代码会显得比较简洁。但对于某些特定的事件，无法使用基于回调的事件处理，只能采用基于监听的事件处理。

 基于监听的事件处理是一种更“面向对象”的事件处理，在事件监听的处理模型中主要涉及如下三类对象。

- Event Source （事件源）：事件发生的场所，通常就是各个组件，例如按钮、窗口、菜单等。

- Event （事件）：事件封装了界面组件上发生的特定事情（通常就是一次用户操作）。如果程序需要获得界面组件上所发生事件的相关信息，一般通过Event对象来取得。

- Event Listener （事件监听器）：负责监听事件源所发生的事件，并对各种事件做出相应的响应。

当用户按下一个按钮或者单击某个菜单项时，这些动作就会激发一个相应的事件，该事件就会触发事件源上注册的事件监听器（特殊的Java对象），事件监听器调用对应的事件处理器 （事件监听器里的实例方法）来做出相应的响应。

每个组件均可以针对特定的事件指定一个事件监听器，每个事件监听器也可监听一个或多个事件源。因为同一个事件源上可能发生多种事件，委派式事件处理方式可以把事件源上所有可能发生的事件分别授权给不同的事件监听器来处理；同时也可以让一类事件都使用同一个事件监听器来处理。

 Android事件处理流程如下图所示：

![1.jpeg](https://i.loli.net/2019/05/08/5cd290221e01e.jpeg)

  从上图可以知道，基于监听的事件处理模型的流程如下：

为某个事件源（界面组件）设置一个监听器，用于监听用户操作。

- 当用户操作时，会触发事件源的监听器。

- 生成了对应的事件对象。

- 将这个事件源对象作为参数传给事件监听器。

- 事件监听器对事件对象进行判断，执行对应的事件处理器（对应事件的处理方法）。

Android中基于监听的事件处理模型的开发步骤如下。

- 获取普通界面组件（事件源），也就是被监听的对象。

- 实现事件监听器类，该监听器类是一个特殊的Java类，必须实现一个XxxListener接口。

- 调用事件源的setXxxListener方法将事件监听器对象注册给普通组件（事件源）。


对于这三件事情，事件源可以是任何界面组件，不太需要开发者参与；注册监听器也只要一行代码即可，因此事件编程的关键就是实现事件监听器类。

 在基于监听的事件处理模型中，事件监听器必须实现事件监听器接口，Android为不同的界面组件提供了不同的监听器接口，这些接口通常以内部类的形式存在。以View类为例，它包含了如下几个内部接口。

- View.OnClickListener：单击事件的事件监听器必须实现的接口。

- View.OnCreateContextMenu Listener ：创建上下文菜单事件的事件监听器必须实现的接口。

- View.OnFocusChangeListener：焦点改变事件的事件监听器必须实现的接口。

- View.OnKeyListener：按键事件的事件监听器必须实现的接口。

- View.OnLongClickListener：长按事件的事件监听器必须实现的接口。

- View.OnTouchListener：触摸事件的事件监听器必须实现的接口。


通过前面的学习，知道事件监听器就是实现了特定接口的Java类的实例。在程序中实现事件监听器，通常有如下几种形式。

- 匿名内部类形式：使用匿名内部类创建事件监听器对象。

- 内部类形式：将事件监听器类定义成当前类的内部类。

- 外部类形式：将事件监听器类定义成一个外部类。

- Activity本身作为事件监听器类：让Activity本身实现监听器接口，并实现事件处理方法。

- 直接绑定到标签形式：直接在xml布局文件对应的Activity中定义一个事件处理方法，然后在布局文件中引用要触发的事件。

### 监听

1. 通过匿名内部类作为事件监听器类

   ```
   public class MainActivity extends Activity {
       private EditText edittext;
       private Button button;
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
           edittext=(EditText) findViewById(R.id.edit_text);
           button = (Button) findViewById(R.id.button);
           //为button按钮注册监听器，并通过匿名内部类实现
           button.setOnClickListener(new OnClickListener() {
               @Override
               public void onClick(View v) {
               //点击Button会改变edittext的文字为"点击了Button"
               edittext.setText("点击了Button");
               }
           });
       }
   }
   ```

   大部分时候，事件处理器都没有什么利用价值（可利用代码通常都被抽象成了业务逻辑方法），因此大部分事件监听器只是临时使用一次，所以使用匿名内部类形式的事件监听器更合适，实际上，这种形式是目前是最广泛的事件监听器形式。上面的程序代码就是匿名内部类来创建事件监听器的！！！

   对于使用匿名内部类作为监听器的形式来说，唯一的缺点就是匿名内部类的语法有点不易掌握，如果读者java基础扎实，匿名内部类的语法掌握较好，通常建议使用匿名内部类作为监听器。

2. 内部类作为监听器

   将事件监听器类定义成当前类的内部类。

   (1)使用内部类可以在当前类中复用监听器类，因为监听器类是外部类的内部类

   (2)所以可以自由访问外部类的所有界面组件。

3. 使用实现接口的方式来进行注册，让Activity类实现了OnClickListener事件监听接口，从而可以在该Activity类中直接定义事件处理器方法：onClick(view v)，当为某个组件添加该事件监听器对象时，直接使用this作为事件监听器对象即可：

   ```
   public class MainActivity extends Activity implements OnClickListener {
       private EditText edittext;
       private Button button;
       private Button button2;

       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
           edittext=(EditText) findViewById(R.id.edit_text);
           button = (Button) findViewById(R.id.button);
           button2 = (Button) findViewById(R.id.button2);
           button.setOnClickListener(this);
           button2.setOnClickListener(this);
       }

       @Override
       //用switch区分是哪个id
       public void onClick(View v) {
           switch (v.getId()){
           case R.id.button:
               edittext.setText("点击了Button");
               break;
           case R.id.button2:
               edittext.setText("点击了Button2");
               break;
           }
       }
   }
   ```

   这种形式使用activity本身作为监听器类，可以直接在activity类中定义事件处理器方法，这种形式非常简洁。但这种做法有两个缺点：

   （1）这种形式可能造成程序结构混乱。Activity的主要职责应该是完成界面初始化；但此时还需包含事件处理器方法，从而引起混乱。

   （2）如果activity界面类需要实现监听器接口，让人感觉比较怪异。
   上面的程序让Activity类实现了OnClickListener事件监听接口，从而可以在该Activity类中直接定义事件处理器方法：onClick(view v)，当为某个组件添加该事件监听器对象时，直接使用this作为事件监听器对象即可。

4. 外部类作为监听器

   使用顶级类定义事件监听器类的形式比较少见，主要因为如下两个原因：
   1、事件监听器通常属于特定的gui界面，定义成外部类不篮球提高程序的内聚性。
   2、外部类形式的事件监听器不能自由访问创建gui界面的类中的组件，编程不够简洁。
   但如果某个事件监听器确实需要被多个gui界面所共享，而且主要是完成某种业务逻辑的实现，则可以考虑使用外部类的形式来定义事件监听器类。

5. 直接绑定到标签

   Android还有一种更简单的绑定事件监听器的的方式，直接在界面布局文件中为指定标签绑定事件处理方法。
   对于很多Android标签而言，它们都支持如onClick、onLongClick等属性，这种属性的属性值就是一个形xxx(View source)的方法的方法名。

   为Button按钮绑定一个事件处理方法：clickHanlder，这意味着开发者需要在该界面布局对应的Activity中定义一个void clickHanler(View source)方法，该方法将会负责处理该按钮上的单击事件。

### 回调

基于监听的事件处理机制，简单说就是为事件源（组件）添加一个监听器，然后当用户触发了事件后交给监听器去处理，根据不同的事件执行不同的操作。那么基于回调的事件处理机制又是什么样的原理呢？

对于基于回调的事件处理模型来说，事件源与事件监听器是统一的，或者说事件监听器完全消失了。当用户在GUI组件上激发某个事件时，组件自己特定的方法将会负责处理该事件。

为了实现回调机制的事件处理，Android为所有GUI组件都提供了一些事件处理的回调方法，以View为例，该类包含如下方法。

- boolean onKeyDown(int keyCode, KeyEvent event)：当用户在该组件上按下某个按键时触发该方法。

- boolean onKeyLongPress(int keyCode, KeyEvent event)：当用户在该组件上长按某个按键时触发该方法。

- boolean onKeyShortcut(int keyCode, KeyEvent event)：当一个键盘快捷键事件发生时触发该方法。

- boolean onKeyUp(int keyCode, KeyEvent event)：当用户在该组件上松开某个按键时触发该方法。

- boolean onTouchEvent(MotionEvent event)：当用户在该组件上触发触摸屏事件时触发该方法。

- boolean onTrackballEvent(MotionEvent event)：当用户在该组件上触发轨迹球事件时触发该方法。

- void onFocusChanged(boolean gainFocus, int direction, Rect previously FocusedRect)：当组件的焦点发生改变时触发该方法。和前面的6个方法不同，该方法只能够在View中重写。

对于基于监听的事件处理模型来说，事件源和事件监听器是分离的，当事件源上发生特定事件时，该事件交给事件监听器负责处理；对于基于回调的事件处理模型来说，事件源和事件监听器是统一的，当事件源发生特定事件时，该事件还是由事件源本身负责处理。

几乎所有基于回调的事件处理方法都有一个boolean类型的返回值，该返回值用于标识该处理方法是否能完全处理该事件。

- 如果处理事件的回调方法返回true，表明该处理方法己完全处理该事件，该事件不会传播出去。

- 如果处理事件的回调方法返回false，表明该处理方法并未完全处理该事件，该事件会传播出去。


对于基于回调的事件传播而言，某组件上所发生的事件不仅会激发该组件上的回调方法， 也会触发该组件所在Activity的回调方法——只要事件能传播到该Activity。

```
//MyButton.java
import android.content.Context;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.widget.Button;
import android.widget.Toast;


public class MyButton extends Button {
    // 构造方法
    public MyButton(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    // 重写 onTouchEvent触碰事件的回调方法
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        super.onTouchEvent(event);
        // 消息提示
        Toast.makeText(getContext(), "MyButton回调onTouchEvent方法", Toast.LENGTH_SHORT).show();

        // 返回false，表明该事件会向外扩散
        return false;
    }
}

//EventCallbackActivity
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.MotionEvent;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class EventCallbackActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.mybutton_layout);

        // 获取自定义按钮的实例对象
        Button myButton = (Button) findViewById(R.id.mybutton);

        // 为自定义按钮绑定事件监听器
        myButton.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                // 消息提示
                Toast.makeText(EventCallbackActivity.this,
                        "Activity收到onTouch事件监听", Toast.LENGTH_SHORT).show();
                // 返回false，表明该事件会继续向外扩散
                return false;
            }
        });
    }

    // 重写onTouchEvent触碰事件的回调方法
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        super.onTouchEvent(event);
        // 消息提示
        Toast.makeText(EventCallbackActivity.this,
                "Activity回调onTouchEvent方法", Toast.LENGTH_SHORT).show();
        // 返回false，表明未完成处理该事件，该事件会继续向外扩散
        return false;
    }
}
```

当点击按钮时，Android系统最先触发的应该是该按钮上绑定的事件监听器，然后才触发该按钮提供的事件回调方法，最后还会传播到该按钮所在的Activity。

如果我们让任何一个事件处理方法返回了 true，那么该事件将不会继续向外传播。如将上述代码中按钮绑定的事件监听器中返回true，运行程序发现只能收到onTouch事件监听。

对比Android提供的两种事件处理模型，可发现基于监听的事件处理模型具有更大的优势。

- 基于监听的事件处理模型分工更明确，事件源、事件监听器由两个类分幵实现，具有更好的可维护性。

- Android的事件处理机制保证基于监听的事件监听器会被优先触发。

### 系统事件

#### Configuration类

Configuration类专门用于描述手机设备上的配置信息，这些配置信息既包括用户特定的配置项，也包括系统的动态设备配置。程序可调用Activity的如下方法来获取系统的Configuration对象：

```
   Configurationcfg=getResources().getConfiguration();
```

一旦获得了系统的Configuration对象，就可以使用该对象提供的如下常用属性来获取系统的配置信息。

- densityDpi：屏幕密度。

- fontScale：当前用户设置的字体的缩放因子。

- hardKeyboardHidden：判断硬键盘是否可见，有两个可选值：

- HARDKEYBOARDHIDDEN_NO，值为十六进制的0。

- HARDKEYBOARDHIDDEN_YES，值为十六进制的1。

- keyboard：获取当前关联额键盘类型：该属性的返回值：

- KEYBOARD_12KEY：只有12个键的小键盘。

- KEYBOARD_NOKEYS：无键盘。

- KEYBOARD_QWERTY：普通键盘。

- keyboardHidden：该属性返回一个boolean值用于标识当前键盘是否可用。该属性不仅会判断系统的硬件键盘，也会判断系统的软键盘（位于屏幕）。

- locale：获取用户当前的语言环境。

- mcc：获取移动信号的国家码。

- mnc：获取移动信号的网络码。

- ps:国家代码和网络代码共同确定当前手机网络运营商。

- navigation：判断系统上方向导航设备的类型。该属性的返回值：

- NAVIGATION_NONAV：无导航。

- NAVIGATION_DPAD：DPAD导航。

- NAVIGATION_TRACKBALL：轨迹球导航。

- NAVIGATION_WHEEL：滚轮导航。

- orientation：获取系统屏幕的方向。该属性的返回值：

- ORIENTATION_LANDSCAPE：横向屏幕。

- ORIENTATION_PORTRAIT：竖向屏幕。

- screenHeightDp，screenWidthDp：屏幕可用高和宽，用dp表示。

- touchscreen：获取系统触摸屏的触摸方式。该属性的返回值：

- TOUCHSCREEN_NOTOUCH：无触摸屏。

- TOUCHSCREEN_STYLUS：触摸笔式触摸屏。

- TOUCHSCREEN_FINGER：接收手指的触摸屏。


如果程序需要监听系统设置的更改，则可以考虑重写Activity的onConfigurationChanged (Configuration newConfig)方法，该方法是一个基于回调的事件处理方法：当系统设置发生更改时，该方法会被自动触发。

当然，为了让Activity能监听系统配置更改的事件，需要在配置Activity时指定 androidiconfigChanges 属性，该属性可以支持 mcc、mnc、locale、touchscreen、keyboard、keyboardHidden、navigation、orientation、screenLayout、uiMode、screenSize、smallestScreenSize、fontScale等属性值。

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <TextView
        android:id="@+id/configuration_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

java文件

```
import android.content.res.Configuration;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.TextView;
import android.widget.Toast;

public class SystemEventActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.system_event_layout);
        // 获取界面组件
        TextView configurationTv = (TextView) findViewById(R.id.configuration_tv);

        // 获取系统的Configuration对象
        Configuration cfg = getResources().getConfiguration();

        // 获取系统的配置信息
        StringBuffer status = new StringBuffer();
        status.append("屏幕密度：" + cfg.densityDpi + "\n");
        status.append("字体缩放因子：" + cfg.fontScale + "\n");
        status.append("硬键盘是否可见：" + cfg.hardKeyboardHidden + "\n");
        status.append("键盘类型：" + cfg.keyboard + "\n");
        status.append("当前键盘是否可用：" + cfg.keyboardHidden + "\n");
        status.append("语言环境：" + cfg.locale + "\n");
        status.append("移动信号的国家码：" + cfg.mcc + "\n");
        status.append("移动信号的网络码：" + cfg.mnc + "\n");
        status.append("方向导航设备的类型：" + cfg.navigation + "\n");
        status.append("方向导航设备是否可用：" + cfg.navigationHidden + "\n");
        status.append("系统屏幕的方向：" + cfg.orientation + "\n");
        status.append("屏幕可用高：" + cfg.screenHeightDp + "\n");
        status.append("屏幕可用宽：" + cfg.screenWidthDp + "\n");
        status.append("系统触摸屏的触摸方式：" + cfg.touchscreen + "\n");
        status.append("screenLayout：" + cfg.screenLayout + "\n");
        status.append("smallestScreenWidthDp：" + cfg.smallestScreenWidthDp + "\n");
        status.append("uiMode：" + cfg.uiMode + "\n");

        // 配置信息输出
        configurationTv.setText(status.toString());
    }

    // 重写onConfigurationChanged方法，用于监听系统设置的更改
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);

        // 获取更改后的屏幕方向
        int screen = newConfig.orientation;
        String ori = (Configuration.ORIENTATION_LANDSCAPE == screen) ? "横屏" : "竖屏";

        // 消息提示
        Toast.makeText(this, "系统的屏幕方向改变为：" + ori, Toast.LENGTH_SHORT).show();
    }
}
```

为了让Activity能监听到屏幕方向的更改事件，需要在配置该Activity时指定 androidiconfigChanges 属性，应用的AndroidManifest.xml文件改为如下形式。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.jinyu.cqkxzsxy.android.widgetsample">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".SystemEventActivity"
            android:configChanges="orientation|screenSize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

上面的配置代码指定了该Activity可以监听屏幕方向改变的事件，这样当程序改变手机屏幕方向时，Activity的onConfigurationChanged()方法就会被回调。

### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)