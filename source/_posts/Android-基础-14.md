---
title: Android 对话框
date: 2019-05-02 20:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

Android提供了丰富的Dialog函数，本文介绍最常用的8种对话框的使用方法，包括普通（包含提示消息和按钮）、列表、单选、多选、等待、进度条、编辑、自定义等多种形式

有时，我们希望在对话框创建或关闭时完成一些特定的功能，这需要复写Dialog的create()、show()、dismiss()等方法。

<!--more-->

dialog就是一个在屏幕上弹出一个可以让用户做出一个选择，或者输入额外的信息的对话框，一个对话框并不会沾满我们整个的屏幕，并且通常用于模型事件当中需要用户做出一个决定后才会继续执行。

Dialog虽然可以显示到屏幕上，但是Dialog并非继承自View，而是继承自Object。Dialog的生命周期由Activity来控制，所以当Activity被销毁后，如果再有对Dialog的操作会导致异常：java.lang.IllegalArgumentException: View not attached to window manager。

当Dialog显示的时候，下面的Activity会失去焦点，用户的注意力将全部集中在Dialog上，进行操作；当Dialog消失后，Activity将重新获得焦点。

Dialog类是dialog对话框的基类，但是我们应该避免直接使用这个类来实例化一个dialog对话框，我们应当使用其子类来得到一个对话框：

```
java.lang.Object
   ↳     android.app.Dialog
 
Known Direct Subclasses
AlertDialog, CharacterPickerDialog, MediaRouteChooserDialog, MediaRouteControllerDialog, Presentation
 
Known Indirect Subclasses
DatePickerDialog, ProgressDialog, TimePickerDialog 
```

我们看到，Dialog有很多的子类实现，所以我们要定义一个对话框，使用其子类来实例化一个即可，而不要直接使用Dialog这个父类来构造。

### AlertDialog

AlertDialog是功能最丰富、实践应用最广的对话框，它可以生成各种内容的对话框。但实际上AlertDialog生成的对话框总体可分为以下4个区域：图标区、标题区、内容区、按钮区。

![1.png](https://i.loli.net/2019/06/27/5d1499ecb819914001.png)

- 区域1那里就是定义弹出框的头部信息，包括标题名或者是一个图标。

- 区域2那里是AlertDialog对话框的content部分，在这里我们可以设置一些message信息，或者是定义一组选择框，还可以定义我们自己的布局弹出框。

- 区域3那里使我们的Action Buttons部分，这里我们可以定义我们的操作按钮。

在AlertDialog中，定义按钮都是通过 setXXXButton 方法来完成，其中一共有3种不同的Action Buttons供我们选择：

1. `setPositiveButton(CharSequence text, DialogInterface.OnClickListener listener)`
   这是一个相当于OK、确定操作的按钮，

2. `setNegativeButton (CharSequence text, DialogInterface.OnClickListener listener)`
   这是一个相当于取消操作的按钮。

3. `setNeutralButton (CharSequence text, DialogInterface.OnClickListener listener)`
这个是相当于一个忽略操作的按钮。

我们每一种action buttons最多只能出现一个，即弹出对话框最多只能出现一个PositiveButton。

一般创建一个对话框需要经过以下几步：

1. 创建AlertDialog.Builder对象。
2. 调用AlertDialog.Builder的setTitle()或者setCustomTitle()方法设置标题。
3. 调用AlertDialog.Builder的setIcon()方法设置标题logo。
4. 调用AlertDialog.Builder的相关方法设置对话框内容。
5. 调用AlertDialog.Builder的setPositiveButton()、setNegativeButton()或setNeutralButton()方法添加多个按钮。
6. 调用AlertDialog.Builder的create()方法创建AlertDialog对象，再调用AlertDialog对象的show()方法将该对话框显示出来。

其中，第4步设置对话框的内容，这里有6种方法来指定：

- setMessage():设置对话框内容为简单文本内容。

- setItems():设置对话框内容为简单列表项。

- setSingleChoiceItems():设置对话框内容为单选列表项。

- setMultiChoiceItems():设置对话框内容为多选列表项。

- setAdapter():设置对话框内容为自定义列表项。

- setView():设置对话框内容为自定义View。

**bug**

![2.png](https://i.loli.net/2019/06/27/5d149ab7e8b5885064.png)

解决方法有两种：

- 将DialogActivity 改为直接继承Activity

```
import android.app.AlertDialog;
```

- 根据提示来使用AppCompat的theme

```
<activity android:name=".CommonDialogActivity"
android:theme="@style/Theme.AppCompat.Light.NoActionBar" />
```



#### 常见几种AlertDialog对话框

![](https://i.loli.net/2019/06/27/5d1488dfab15916751.jpg)

##### 普通Dialog(图1与图2)

**2个按钮**

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button buttonNormal = (Button) findViewById(R.id.button_normal);
        buttonNormal.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showNormalDialog();
            }
        });
    }
    
    private void showNormalDialog(){
       final AlertDialog.Builder dialog = new AlertDialog.Builder(MainActivity.this) {
    @Override
    public AlertDialog create() {
        d("对话框create，创建时调用");
        return super.create();
    }

    @Override
    public AlertDialog show() {
        d("对话框show，显示时调用");
        return super.show();
    }
};

dialog.setOnCancelListener(new DialogInterface.OnCancelListener() {
    public void onCancel(DialogInterface dialog) {
        d("对话框取消");
    }
});

dialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
    public void onDismiss(DialogInterface dialog) {
        d("对话框销毁");
    }
});

dialog.setIcon(R.mipmap.ic_launcher)
        .setTitle("我是标题")
        .setMessage("我是要显示的消息")
        .setCancelable(true)
        .setPositiveButton("确定",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        d("点击确定");
                    }
                })
        .setNegativeButton("取消",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        d("点击取消");
                    }
                });
dialog.show();
    }
}
```

创建对话框采用了建造者模式，通过构建AlertDialog.Builder来进行参数设置，最后通过show函数显示对话框。
 初始化AlertDialog.Builder的时候，可以重写create和show函数，在创建和显示的时候做一些额外工作。

```
setOnCancelListener：设置对话框取消时的回调函数
 setOnDismissListener：设置对话框消失时的回调函数
 setIcon：设置对话框的图标
 setTitle：设置对话框标题
 setMessage：设置对话框消息
 setCancelable：设置对话框是否可以被取消，如果是true，则点击非对话框区域，对话框消失；false则刚好相反
 setPositiveButton：设置确定按钮的文字和回调函数
 setNegativeButton：设置取消按钮的文字和回调函数
 show：显示对话框
```

这里要注意一点，对话框的cancel和dismiss两个函数的区别
 查看源码可以发现

```java
public void cancel() {
    if (mCancelMessage != null) {
        // Obtain a new message so this dialog can be re-used
        Message.obtain(mCancelMessage).sendToTarget();
    }
    dismiss();
}

public void setOnCancelListener(final OnCancelListener listener) {
    if (listener != null) {
        mCancelMessage = mListenersHandler.obtainMessage(CANCEL, listener);
    } else {
        mCancelMessage = null;
    }
}
```

如果没有调用setOnCancelListener，那么cancel和dismiss两个函数功能是一样的；当调用了setOnCancelListener时，会在执行cancel函数时向观察者发出一个消息，仅此而已。

**3个按钮**

```java
/* @setNeutralButton 设置中间的按钮
 * 若只需一个按钮，仅设置 setPositiveButton 即可
 */
private void showMultiBtnDialog(){
 final AlertDialog.Builder dialog = new AlertDialog.Builder(MainActivity.this);
dialog.setIcon(R.mipmap.ic_launcher)
        .setTitle("我是标题")
        .setMessage("我是要显示的消息")
        .setPositiveButton("确定",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        d("确定");
                    }
                })
        .setNeutralButton("说明",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        d("说明");
                    }
                })
        .setNegativeButton("取消",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        d("取消");
                    }
                });
dialog.show();
}
```

与双按钮对话框相比，三按钮对话框多了作为说明操作的按钮，当对话框的message无法显示全部内容是，通过第三个按钮将用户引导到新的界面，显示长篇幅说明文字。

##### 列表Dialog（图3）

```java
private void showListDialog() {
    final String[] items = { "我是1","我是2","我是3","我是4" };
    AlertDialog.Builder listDialog = 
        new AlertDialog.Builder(MainActivity.this);
    listDialog.setTitle("我是一个列表Dialog");
    listDialog.setItems(items, new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            // which 下标从0开始
            // ...To-do
            Toast.makeText(MainActivity.this, 
                "你点击了" + items[which], 
                Toast.LENGTH_SHORT).show();
        }
    });
    listDialog.show();
}
```

列表对话框，用于显示一个列表并进行单选。
这里主要通过setItems函数为对话框配置要显示的列表和选择时的回调函数，选择后对话框自动消失。
请注意回调函数的参数which从0开始。

##### 单选Dialog（图4）

```java
int yourChoice;
private void showSingleChoiceDialog(){
    final String[] items = { "我是1","我是2","我是3","我是4" };
    yourChoice = -1;
    AlertDialog.Builder singleChoiceDialog = 
        new AlertDialog.Builder(MainActivity.this);
    singleChoiceDialog.setTitle("我是一个单选Dialog");
    // 第二个参数是默认选项，此处设置为0
    singleChoiceDialog.setSingleChoiceItems(items, 0, 
        new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            yourChoice = which;
        }
    });
    singleChoiceDialog.setPositiveButton("确定", 
        new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            if (yourChoice != -1) {
                Toast.makeText(MainActivity.this, 
                "你选择了" + items[yourChoice], 
                Toast.LENGTH_SHORT).show();
            }
        }
    });
    singleChoiceDialog.show();
}
```

单选对话框，与列表对话框功能相似，主要区别是选择后对话框是否消失。单选对话框可以反复选择，直到用户点击确定按钮。
setSingleChoiceItems函数的第一个参数是要显示的列表文本，第二个参数表示默认选中哪一项，下标从0开始。

##### 多选Dialog（图5）

```java
ArrayList<Integer> yourChoices = new ArrayList<>();
private void showMultiChoiceDialog() {
    final String[] items = { "我是1","我是2","我是3","我是4" };
    // 设置默认选中的选项，全为false默认均未选中
    final boolean initChoiceSets[]={false,false,false,false};
    yourChoices.clear();
    AlertDialog.Builder multiChoiceDialog = 
        new AlertDialog.Builder(MainActivity.this);
    multiChoiceDialog.setTitle("我是一个多选Dialog");
    multiChoiceDialog.setMultiChoiceItems(items, initChoiceSets,
        new DialogInterface.OnMultiChoiceClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which,
            boolean isChecked) {
            if (isChecked) {
                yourChoices.add(which);
            } else {
                yourChoices.remove(which);
            }
        }
    });
    multiChoiceDialog.setPositiveButton("确定", 
        new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            int size = yourChoices.size();
            String str = "";
            for (int i = 0; i < size; i++) {
                str += items[yourChoices.get(i)] + " ";
            }
            Toast.makeText(MainActivity.this, 
                "你选中了" + str, 
                Toast.LENGTH_SHORT).show();
        }
    });
    multiChoiceDialog.show();
}
```

setMultiChoiceItems函数

- 参数1：设置要显示的文本列表items
- 参数2：和文本数据个数一致的boolean数组selected，selected[i]表示items[i]是否是选中状态
- 参数3：选项被点击时的回调函数，isChecked为true表示选中；false表示取消

##### 等待Dialog（图6）

```java
private void showWaitingDialog() {
    /* 等待Dialog具有屏蔽其他控件的交互能力
     * @setCancelable 为使屏幕不可点击，设置为不可取消(false)
     * 下载等事件完成后，主动调用函数关闭该Dialog
     */
    ProgressDialog waitingDialog= 
        new ProgressDialog(MainActivity.this);
    waitingDialog.setTitle("我是一个等待Dialog");
    waitingDialog.setMessage("等待中...");
    waitingDialog.setIndeterminate(true);
    waitingDialog.setCancelable(false);
    waitingDialog.show();
}
```

这里使用了ProgressDialog，由于setCancelable设置为false，对话框无法取消，所以此时只能杀死app！

setIndeterminate参数为true表示该进度条不能明确等待进度，仅仅告知用户需要等待，没有预期。

##### 进度条Dialog（图7）

```java
private void showProgressDialog() {
    /* @setProgress 设置初始进度
     * @setProgressStyle 设置样式（水平进度条）
     * @setMax 设置进度最大值
     */
    final int MAX_PROGRESS = 100;
    final ProgressDialog progressDialog = 
        new ProgressDialog(MainActivity.this);
    progressDialog.setProgress(0);
    progressDialog.setTitle("我是一个进度条Dialog");
    progressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
    progressDialog.setMax(MAX_PROGRESS);
    progressDialog.show();
    /* 模拟进度增加的过程
     * 新开一个线程，每个100ms，进度增加1
     */
    new Thread(new Runnable() {
        @Override
        public void run() {
            int progress= 0;
            while (progress < MAX_PROGRESS){
                try {
                    Thread.sleep(100);
                    progress++;
                    progressDialog.setProgress(progress);
                } catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            // 进度达到最大值后，窗口消失
            progressDialog.cancel();
        }
    }).start();
}
```

同样使用ProgressDialog表示一个有进度概念的等待对话框，给用户一个心理预期。

- setProgress函数：设置当前进度值
- setMax函数：设置最大进度值
- setProgressStyle函数：设置进度条类型

##### 编辑Dialog（图8）

```java
private void showInputDialog() {
    /*@setView 装入一个EditView
     */
    final EditText editText = new EditText(MainActivity.this);
    AlertDialog.Builder inputDialog = 
        new AlertDialog.Builder(MainActivity.this);
    inputDialog.setTitle("我是一个输入Dialog").setView(editText);
    inputDialog.setPositiveButton("确定", 
        new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            Toast.makeText(MainActivity.this,
            editText.getText().toString(), 
            Toast.LENGTH_SHORT).show();
        }
    }).show();
}
```

##### 自定义Dialog（图9）

```xml
<!-- res/layout/dialog_customize.xml-->
<!-- 自定义View -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <EditText
        android:id="@+id/edit_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" 
        />
</LinearLayout>
```

方法：

```java
private void showCustomizeDialog() {
    /* @setView 装入自定义View ==> R.layout.dialog_customize
     * 由于dialog_customize.xml只放置了一个EditView，因此和图8一样
     * dialog_customize.xml可自定义更复杂的View
     */
    AlertDialog.Builder customizeDialog = 
        new AlertDialog.Builder(MainActivity.this);
    final View dialogView = LayoutInflater.from(MainActivity.this)
        .inflate(R.layout.dialog_customize,null);
    customizeDialog.setTitle("我是一个自定义Dialog");
    customizeDialog.setView(dialogView);
    customizeDialog.setPositiveButton("确定",
        new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            // 获取EditView中的输入内容
            EditText edit_text = 
                (EditText) dialogView.findViewById(R.id.edit_text);
            Toast.makeText(MainActivity.this,
                edit_text.getText().toString(),
                Toast.LENGTH_SHORT).show();
        }
    });
    customizeDialog.show();
}
```

#### 复写回调函数

```java
/* 复写Builder的create和show函数，可以在Dialog显示前实现必要设置
 * 例如初始化列表、默认选项等
 * @create 第一次创建时调用
 * @show 每次显示时调用
 */
private void showListDialog() {
    final String[] items = { "我是1","我是2","我是3","我是4" };
    AlertDialog.Builder listDialog = 
        new AlertDialog.Builder(MainActivity.this){
        
        @Override
        public AlertDialog create() {
            items[0] = "我是No.1";
            return super.create();
        }

        @Override
        public AlertDialog show() {
            items[1] = "我是No.2";
            return super.show();
        }
    };
    listDialog.setTitle("我是一个列表Dialog");
    listDialog.setItems(items, new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            // ...To-do
        }
    });
    /* @setOnDismissListener Dialog销毁时调用
     * @setOnCancelListener Dialog关闭时调用
     */
    listDialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
        public void onDismiss(DialogInterface dialog) {
            Toast.makeText(getApplicationContext(),
                "Dialog被销毁了", 
                Toast.LENGTH_SHORT).show();
        }
    });
    listDialog.show();
}
```

### 其他对话框

#### 日期选择对话框

![1.png](https://i.loli.net/2019/06/27/5d149dd54e48f56889.png)

```java
Calendar c = Calendar.getInstance();
DatePickerDialog dialog = new  DatePickerDialog(MainActivity.this,
        new DatePickerDialog.OnDateSetListener() {
            @Override
            public void onDateSet(DatePicker view, int year, int monthOfYear, int dayOfMonth) {
                d("选择日期：" + year + "年" + (monthOfYear+1) + "月" + dayOfMonth + "日");
            }
        }, c.get(Calendar.YEAR), c.get(Calendar.MONTH), c.get(Calendar.DAY_OF_MONTH));
dialog.show();
```

注意，当选择成功回调的时候，月份是从0开始的，所以要注意加一。

#### 时间选择对话框

![2.png](https://i.loli.net/2019/06/27/5d149dd56542142306.png)

```java
Calendar c = Calendar.getInstance();
TimePickerDialog dialog = new TimePickerDialog(MainActivity.this,
        new TimePickerDialog.OnTimeSetListener() {
            @Override
            public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
                d("选择时间：" + hourOfDay + "时" + minute + "分");
            }
        }, c.get(Calendar.HOUR_OF_DAY), c.get(Calendar.MINUTE), true);
dialog.show();
```

构造参数最后一个参数true表示采用24小时制

#### 自定义UI对话框

![3.png](https://i.loli.net/2019/06/27/5d149dd57e27f80666.png)

```java
public class CustomDialog1 extends Dialog {

    private Context context;

    public CustomDialog1(Context context) {
        super(context);
        init(context);
    }

    private void init(Context context) {
        this.context = context;
        build();
    }

    private void build() {
        LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        View view = inflater.inflate(R.layout.custom_dialog1_layout, null);

        view.findViewById(R.id.like).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(context, "喜欢", Toast.LENGTH_SHORT).show();
                dismiss();
            }
        });

        view.findViewById(R.id.dislike).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(context, "一般", Toast.LENGTH_SHORT).show();
                dismiss();
            }
        });

        setContentView(view);
    }
}
```

先通过inflate函数解析布局文件，然后通过setContentView为对话框设置视图

#### 自定义尺寸位置对话框

![4.png](https://i.loli.net/2019/06/27/5d149dd58ba0348546.png)

```java
public class CustomDialog2 extends Dialog {

    private Context context;

    public CustomDialog2(Context context) {
        super(context);
//        super(context, R.style.custom_dialog2_style);
        init(context);
    }

    public void init(Context context) {
        this.context = context;
        setContentView(R.layout.custom_dialog2_layout);

        Window window = getWindow();
        window.setBackgroundDrawableResource(R.drawable.custom_dialog1_bg);//设置window的背景色
        WindowManager.LayoutParams lp = window.getAttributes();
        window.setGravity(Gravity.BOTTOM);

        lp.x = 250;
        lp.y = 250;
        lp.width = 400;
        lp.height = 400;
        lp.alpha = 0.8f;
        window.setAttributes(lp);

        setCanceledOnTouchOutside(true);//设置点击Dialog外部任意区域关闭Dialog
    }
}
```

custom_dialog2_layout布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="80dp"
    android:layout_height="80dp"
    android:background="@android:color/holo_red_dark"
    >

</FrameLayout>
```

对话框的gravity是通过window的setGravity来实现贴边功能的。

对话框窗口大小是通过WindowManager.LayoutParams来进行调整的：

```
x：负数表示向左移动，正数表示向右移动
y：负数表示向下移动，正数表示向上移动
width：表示对话框的宽
height：表示对话框的高
alpha：表示对话框的透明度
```

上面显示了这样一个对话框：

 1.对话框尺寸`400*400`

 2.对话框居中贴底边 

3.右移250，上移250 

4.内部有一个`80dp*80dp`的红色视图

#### 动画对话框

![](https://i.loli.net/2019/06/27/5d149e71a35ea58108.gif)

```java
public class AnimDialog1 extends Dialog {

    private Context context;

    public AnimDialog1(Context context) {
        super(context);
        init(context);
    }

    private void init(Context context) {
        this.context = context;
        build();
    }

    private void build() {
        LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        View view = inflater.inflate(R.layout.custom_dialog1_layout, null);

        view.findViewById(R.id.like).setVisibility(View.GONE);
        view.findViewById(R.id.dislike).setVisibility(View.GONE);

        setContentView(view);

        Window window = getWindow();
        window.setWindowAnimations(R.style.dialog1_window_anim);
    }
}
```

对话框的出现消失动画是通过window的setWindowAnimations函数来设置的，这里并不是设置一个anim，而是设置了一个

```xml
<style name="dialog1_window_anim" parent="android:Animation" >
    <item name="android:windowEnterAnimation">@anim/dialog1_enter_anim</item>
    <item name="android:windowExitAnimation">@anim/dialog1_exit_anim</item>
</style>
```

这里可以清楚的看到，可以为window设置进入和退出动画

部分手机如果没有动画效果，请在Manifest.xml的application下的android:theme的style定义当中加入

```xml
 <item name="android:windowNoTitle">true</item>
```

#### 采用自定义布局和功能方式的对话框

自定义对话框布局： high_opinion_dialog_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingTop="@dimen/dp_20"
    android:paddingBottom="@dimen/dp_10"
    android:paddingLeft="@dimen/dp_15"
    android:paddingRight="@dimen/dp_15"
    android:orientation="vertical">

        <TextView
            android:text="Rate US"
            android:gravity="center"
            android:textSize="@dimen/sp_18"
            android:textColor="@color/black"
            android:layout_gravity="center_horizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <TextView
            android:text="We're glad you're enjoying using our app! Would you mind giving us a review?"
            android:gravity="center"
            android:textSize="@dimen/sp_12"
            android:layout_marginTop="@dimen/dp_5"
            android:textColor="@color/black"
            android:layout_gravity="center_horizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_marginTop="@dimen/dp_15"
            android:layout_height="@dimen/dp_37">

            <Button
                android:id="@+id/btn_cancel_high_opion"
                android:layout_width="0dp"
                android:layout_height="match_parent"
                android:text="Maybe later"
                android:background="@drawable/btn_cancer_high_opion_shape"
                android:textColor="@color/white"
                android:layout_weight="1"/>

            <View
                android:layout_width="@dimen/dp_20"
                android:layout_height="match_parent"
                />
            <Button
                android:id="@+id/btn_agree_high_opion"
                android:layout_width="0dp"
                android:text="Sure"
                android:textColor="@color/white"
                android:background="@drawable/btn_agree_high_opinion_shape"
                android:layout_height="match_parent"
                android:layout_weight="1"/>
        </LinearLayout>
</LinearLayout>
```

然后在activity或者fragment中想要加点击弹出对话框的控件的监听事件中调用初始化下面方法

```java
public class HomeActivity extends AppCompatActivity{
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);   
        setContentView(R.layout.activity_home);
        
        Button btn= findViewById(R.id.btn)
        //点击按钮弹出对话框
        btn.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
				       showDialog();
                    }
                });

   }
   
//初始化并弹出对话框方法
private void showDialog(){
 View view = LayoutInflater.from(this).inflate(R.layout.high_opinion_dialog_layout,null,false);
        final AlertDialog dialog = new AlertDialog.Builder(this).setView(view).create();

        Button btn_cancel_high_opion = view.findViewById(R.id.btn_cancel_high_opion);
        Button btn_agree_high_opion = view.findViewById(R.id.btn_agree_high_opion);

        btn_cancel_high_opion.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SharedPreferencesUnitls.setParam(getApplicationContext(),"HighOpinion","false");
               //... To-do
                dialog.dismiss();
            }
        });

        btn_agree_high_opion.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //... To-do
                dialog.dismiss();
            }
        });

                dialog.show();
            //此处设置位置窗体大小，我这里设置为了手机屏幕宽度的3/4  注意一定要在show方法调用后再写设置窗口大小的代码，否则不起效果会
            dialog.getWindow().setLayout((ScreenUtils.getScreenWidth(this)/4*3),LinearLayout.LayoutParams.WRAP_CONTENT);
}

}
```

此处附上ScreenUtils工具类代码

```java
public class ScreenUtils {

    /**
     * 获取屏幕高度(px)
     */
    public static int getScreenHeight(Context context) {
        return context.getResources().getDisplayMetrics().heightPixels;
    }
    /**
     * 获取屏幕宽度(px)
     */
    public static int getScreenWidth(Context context) {
        return context.getResources().getDisplayMetrics().widthPixels;
    }

}


```

需要注意的问题总结：系统的dialog的宽度默认是固定的，即使你自定义布局的怎么修改宽高也不起作用，如果想修改弹出窗体大小，可以使用下面这段代码在调用dialog.show()方法之后来实现改变对话框的宽高的需求

```java
		//此处设置位置窗体大小，
        dialog.getWindow().setLayout(width,height);
```

Android4.0的Alertdialog对话框，设置点击其他位置不消失

```
Android4.0以上AlertDialog，包括其他自定义的dialog，在触摸对话框边缘外部，对话框消失。

可以设置这么一条属性，当然必须先AlertDialog.Builder.create()之后才能调用这两个方法

方法一：

setCanceledOnTouchOutside(false);调用这个方法时，按对话框以外的地方不起作用。按返回键还起作用

方法二：

setCancelable(false);调用这个方法时，按对话框以外的地方不起作用。按返回键也不起作用

```

改变Android Dialog弹出后的Activity背景亮度：
在代码中修改.lp.alpha大小随自己要求设置

```java
// 设置屏幕背景变暗
private void setScreenBgDarken() {
    WindowManager.LayoutParams lp = getWindow().getAttributes();
    lp.alpha = 0.5f;
    lp.dimAmount = 0.5f;
    getWindow().setAttributes(lp);
}
// 设置屏幕背景变亮
private void setScreenBgLight() {
        WindowManager.LayoutParams lp = getWindow().getAttributes();
    lp.alpha = 1.0f;
    lp.dimAmount = 1.0f;
    getWindow().setAttributes(lp);
}

```

**如何控制弹窗弹出的位置：**
一般都是在屏幕正中间弹出默认，但也可以控制从别的地方弹出，比如从底部弹出，可以这样写

```java

        View view = LayoutInflater.from(this).inflate(R.layout.send_dialog, null);
        final Dialog dialog = new AlertDialog.Builder(this, R.style.home_dialog)
                .setView(view)
                .setCancelable(true)
                .create();
        dialog.show();

        Window win = dialog.getWindow();
        win.setGravity(Gravity.BOTTOM);   // 这里控制弹出的位置
        win.getDecorView().setPadding(0, 0, 0, 0);
        WindowManager.LayoutParams lp = win.getAttributes();
        lp.width = WindowManager.LayoutParams.MATCH_PARENT;
        lp.height = WindowManager.LayoutParams.WRAP_CONTENT;
        dialog.getWindow().setBackgroundDrawable(null);
        win.setAttributes(lp);

        view.findViewById(R.id.cancle_dialog).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dialog.dismiss();
            }
        });


```

#### 对话框常用样式

```xml
<item name="android:windowFrame">@null</item><!--是否有边框-->
<item name="android:windowIsFloating">true</item><!--是否浮现在activity之上-->
<item name="android:windowIsTranslucent">false</item><!--是佛半透明-->
<item name="android:windowNoTitle">true</item><!--有无标题-->
<item name="android:windowBackground">@android:color/holo_green_light</item><!--背景颜色-->
<item name="android:backgroundDimEnabled">false</item><!--是否充许对话框的背景变暗-->
```

### [PhoneUtil（手机信息相关）](https://www.cnblogs.com/xqxacm/p/5853850.html)

```java
package com.ujs.learn;

import android.annotation.SuppressLint;
import android.content.Context;
import android.content.Intent;
import android.content.pm.ApplicationInfo;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.pm.PackageManager.NameNotFoundException;
import android.content.res.Configuration;
import android.net.Uri;
import android.os.Environment;
import android.os.StatFs;
import android.telephony.TelephonyManager;
import android.view.WindowManager;

import java.io.File;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;


/**
 * 1、获取手机系统版本号
 * 2、获取手机型号 
 * 3、获取手机宽度 
 * 4、获取手机高度 
 * 5、获取手机imei串号 ,GSM手机的 IMEI 和 CDMA手机的 MEID.
 * 6、获取手机sim卡号 
 * 7、获取手机号 
 * 8、判断sd卡是否挂载 
 * 9、获取sd卡剩余空间的大小
 * 10、获取sd卡空间的总大小
 * 11、判断是否是平板 
 * 12、判断一个apk是否安装
 * 13、拨打电话 
 * 14、打开网页
 * 15、获取应用权限 名称列表 
 * 16、获取手机内安装的应用
 * 17、获取手机安装非系统应用 
 * 18、获取安装应用的信息
 * 19、打开指定包名的应用
 * 20、卸载指定包名的应用
 * 21、手机号判断
 */
public class PhoneUtil {
    private static PhoneUtil phoneUtil;

    public static PhoneUtil getInstance() {
        if (phoneUtil == null) {
            synchronized (PhoneUtil.class) {
                if (phoneUtil == null) {
                    phoneUtil = new PhoneUtil();
                }
            }
        }
        return phoneUtil;
    }

    /**
     * 手机号判断
     */
    public static boolean isMobileNO(String mobile) {
        Pattern p = Pattern
                .compile("^((145|147)|(15[^4])|(17[0-9])|((13|18)[0-9]))\\d{8}$");
        Matcher m = p.matcher(mobile);
        return m.matches();
    }

    /**
     * 获取手机系统版本号
     *
     * @return
     */
    public int getSDKVersionNumber() {
        int sdkVersion;
        try {
            sdkVersion = Integer.valueOf(android.os.Build.VERSION.SDK_INT);
        } catch (NumberFormatException e) {
            e.printStackTrace();
            sdkVersion = 0;
        }
        return sdkVersion;
    }

    /**
     * 获取手机型号
     */
    public String getPhoneModel() {
        return android.os.Build.MODEL;
    }

    /**
     * 获取手机宽度
     */
    @SuppressWarnings("deprecation")
    public int getPhoneWidth(Context context) {
        WindowManager wm = (WindowManager) context
                .getSystemService(Context.WINDOW_SERVICE);
        return wm.getDefaultDisplay().getWidth();
    }

    /**
     * 获取手机高度
     *
     * @param context
     */
    @SuppressWarnings("deprecation")
    public int getPhoneHeight(Context context) {
        WindowManager wm = (WindowManager) context
                .getSystemService(Context.WINDOW_SERVICE);
        return wm.getDefaultDisplay().getHeight();
    }

    /**
     * 获取手机imei串号 ,GSM手机的 IMEI 和 CDMA手机的 MEID.
     *
     * @param context
     */
    @SuppressLint("MissingPermission")
    public String getPhoneImei(Context context) {
        TelephonyManager tm = (TelephonyManager) context
                .getSystemService(Context.TELEPHONY_SERVICE);
        if (tm == null)
            return null;
        return tm.getDeviceId();
    }

    /**
     * 获取手机sim卡号
     *
     * @param context
     */
    @SuppressLint("MissingPermission")
    public String getPhoneSim(Context context) {
        TelephonyManager tm = (TelephonyManager) context
                .getSystemService(Context.TELEPHONY_SERVICE);
        if (tm == null)
            return null;
        return tm.getSubscriberId();
    }

    /**
     * 获取手机号
     *
     * @param context
     */
    @SuppressLint("MissingPermission")
    public String getPhoneNum(Context context) {
        TelephonyManager tm = (TelephonyManager) context
                .getSystemService(Context.TELEPHONY_SERVICE);
        if (tm == null)
            return null;
        return tm.getLine1Number();
    }

    /**
     * 判断sd卡是否挂载
     */
    public boolean isSDCardMount() {
        return Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED);
    }

    /**
     * 获取sd卡剩余空间的大小
     */
    @SuppressWarnings("deprecation")
    public long getSDFreeSize() {
        File path = Environment.getExternalStorageDirectory(); // 取得SD卡文件路径
        StatFs sf = new StatFs(path.getPath());
        long blockSize = sf.getBlockSize(); // 获取单个数据块的大小(Byte)
        long freeBlocks = sf.getAvailableBlocks();// 空闲的数据块的数量
        // 返回SD卡空闲大小
        return (freeBlocks * blockSize) / 1024 / 1024; // 单位MB
    }

    /**
     * 获取sd卡空间的总大小
     */
    @SuppressWarnings("deprecation")
    public long getSDAllSize() {
        File path = Environment.getExternalStorageDirectory(); // 取得SD卡文件路径
        StatFs sf = new StatFs(path.getPath());
        long blockSize = sf.getBlockSize(); // 获取单个数据块的大小(Byte)
        long allBlocks = sf.getBlockCount(); // 获取所有数据块数
        // 返回SD卡大小
        return (allBlocks * blockSize) / 1024 / 1024; // 单位MB
    }

    /**
     * 判断是否是平板
     */
    public boolean isTablet(Context context) {
        return (context.getResources().getConfiguration().screenLayout & Configuration.SCREENLAYOUT_SIZE_MASK) >= Configuration.SCREENLAYOUT_SIZE_LARGE;
    }

    /**
     * 判断一个apk是否安装
     *
     * @param context
     * @param packageName
     */
    public boolean isApkInstalled(Context context, String packageName) {
        PackageManager pm = context.getPackageManager();
        try {
            pm.getPackageInfo(packageName, 0);
        } catch (PackageManager.NameNotFoundException e) {
            return false;
        }
        return true;
    }

    /**
     * 拨打电话
     *
     * @param context
     * @param phoneNum
     */
    public void call(Context context, String phoneNum) throws Exception {
        if (phoneNum != null && !phoneNum.equals("")) {
            Uri uri = Uri.parse("tel:" + phoneNum);
            Intent intent = new Intent(Intent.ACTION_DIAL, uri);
            context.startActivity(intent);
        }
    }

    /**
     * 打开网页
     */
    public void openWeb(Context context, String url) {
        try {
            Uri uri = Uri.parse(url);
            context.startActivity(new Intent(Intent.ACTION_VIEW, uri));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取应用权限 名称列表
     */
    public String[] getAppPermissions(Context context)
            throws NameNotFoundException {
        PackageManager pm = context.getPackageManager();
        PackageInfo packageInfo = pm.getPackageInfo(context.getPackageName(),
                PackageManager.GET_PERMISSIONS);
        return getAppPermissions(packageInfo);
    }

    public String[] getAppPermissions(PackageInfo packageInfo)
            throws NameNotFoundException {
        return packageInfo.requestedPermissions;
    }

    /**
     * 获取手机内安装的应用
     */
    public List<PackageInfo> getInstalledApp(Context context) {
        PackageManager pm = context.getPackageManager();
        return pm.getInstalledPackages(0);
    }

    /**
     * 获取手机安装非系统应用
     */
    @SuppressWarnings("static-access")
    public List<PackageInfo> getUserInstalledApp(Context context) {
        List<PackageInfo> infos = getInstalledApp(context);
        List<PackageInfo> apps = new ArrayList<PackageInfo>();
        for (PackageInfo info : infos) {
            if ((info.applicationInfo.flags & info.applicationInfo.FLAG_SYSTEM) <= 0) {
                apps.add(info);
            }
        }
        infos.clear();
        infos = null;
        return apps;
    }

    /**
     * 获取安装应用的信息
     */
    public Map<String, Object> getInstalledAppInfo(Context context,
                                                   PackageInfo info) {
        Map<String, Object> appInfos = new HashMap<String, Object>();
        PackageManager pm = context.getPackageManager();
        ApplicationInfo aif = info.applicationInfo;
        appInfos.put("icon", pm.getApplicationIcon(aif));
        appInfos.put("lable", pm.getApplicationLabel(aif));
        appInfos.put("packageName", aif.packageName);
        return appInfos;
    }

    /**
     * 打开指定包名的应用
     */
    public void startAppPkg(Context context, String pkg) {
        Intent startIntent = context.getPackageManager()
                .getLaunchIntentForPackage(pkg);
        startIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(startIntent);
    }

    /**
     * 卸载指定包名的应用
     */
    public void unInstallApk(Context context, String packageName) {
        Uri uri = Uri.parse("package:" + packageName);
        Intent intent = new Intent(Intent.ACTION_DELETE);
        intent.setData(uri);
        context.startActivity(intent);
    }

}
```

#### [ZipUtils(压缩/解压缩文件相关)](https://www.cnblogs.com/xqxacm/p/9999309.html)

```java
package com.ujs.learn;

import android.content.Context;
import android.util.Log;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;
import java.util.zip.ZipInputStream;
import java.util.zip.ZipOutputStream;


public class ZipUtils {
    public static final String TAG = "ZIP";

    public ZipUtils() {
    }

    /**
     * 解压zip到指定的路径
     *
     * @param zipFileString ZIP的名称
     * @param outPathString 要解压缩路径
     * @throws Exception
     */
    public static void UnZipFolder(String zipFileString, String outPathString, Context context) throws Exception {
        ZipInputStream inZip = new ZipInputStream(new FileInputStream(zipFileString));
        ZipEntry zipEntry;

        String szName = "";
        while ((zipEntry = inZip.getNextEntry()) != null) {
            szName = zipEntry.getName();
            if (zipEntry.isDirectory()) {
                //获取部件的文件夹名
                szName = szName.substring(0, szName.length() - 1);
                File folder = new File(outPathString + File.separator + szName);
                folder.mkdirs();
            } else {
                Log.e(TAG, outPathString + File.separator + szName);
                File file = new File(outPathString + File.separator + szName);
                if (!file.exists()) {
                    Log.e(TAG, "Create the file:" + outPathString + File.separator + szName);
                    file.getParentFile().mkdirs();
                    file.createNewFile();
                }
                // 获取文件的输出流
                FileOutputStream out = new FileOutputStream(file);
                int len;
                byte[] buffer = new byte[1024];
                // 读取（字节）字节到缓冲区
                while ((len = inZip.read(buffer)) != -1) {
                    // 从缓冲区（0）位置写入（字节）字节
                    out.write(buffer, 0, len);
                    out.flush();
                }
                out.close();
            }
        }
        inZip.close();
    }

    public static void UnZipFolder(String zipFileString, String outPathString, String szName) throws Exception {
        ZipInputStream inZip = new ZipInputStream(new FileInputStream(zipFileString));
        ZipEntry zipEntry;
        while ((zipEntry = inZip.getNextEntry()) != null) {
            //szName = zipEntry.getName();
            if (zipEntry.isDirectory()) {
                //获取部件的文件夹名
                szName = szName.substring(0, szName.length() - 1);
                File folder = new File(outPathString + File.separator + szName);
                folder.mkdirs();
            } else {
                Log.e(TAG, outPathString + File.separator + szName);
                File file = new File(outPathString + File.separator + szName);
                if (!file.exists()) {
                    Log.e(TAG, "Create the file:" + outPathString + File.separator + szName);
                    file.getParentFile().mkdirs();
                    file.createNewFile();
                }
                // 获取文件的输出流
                FileOutputStream out = new FileOutputStream(file);
                int len;
                byte[] buffer = new byte[1024];
                // 读取（字节）字节到缓冲区
                while ((len = inZip.read(buffer)) != -1) {
                    // 从缓冲区（0）位置写入（字节）字节
                    out.write(buffer, 0, len);
                    out.flush();
                }
                out.close();
            }
        }
        inZip.close();
    }

    /**
     * 压缩文件和文件夹
     *
     * @param srcFileString 要压缩的文件或文件夹
     * @param zipFileString 解压完成的Zip路径
     * @throws Exception
     */
    public static void ZipFolder(String srcFileString, String zipFileString) throws Exception {
        //创建ZIP
        ZipOutputStream outZip = new ZipOutputStream(new FileOutputStream(zipFileString));
        //创建文件
        File file = new File(srcFileString);
        //压缩
        ZipFiles(file.getParent() + File.separator, file.getName(), outZip);
        //完成和关闭
        outZip.finish();
        outZip.close();
    }

    /**
     * 压缩文件
     *
     * @param folderString
     * @param fileString
     * @param zipOutputSteam
     * @throws Exception
     */
    private static void ZipFiles(String folderString, String fileString, ZipOutputStream zipOutputSteam) throws Exception {
        if (zipOutputSteam == null)
            return;
        File file = new File(folderString + fileString);
        if (file.isFile()) {
            ZipEntry zipEntry = new ZipEntry(fileString);
            FileInputStream inputStream = new FileInputStream(file);
            zipOutputSteam.putNextEntry(zipEntry);
            int len;
            byte[] buffer = new byte[4096];
            while ((len = inputStream.read(buffer)) != -1) {
                zipOutputSteam.write(buffer, 0, len);
            }
            zipOutputSteam.closeEntry();
        } else {
            //文件夹
            String[] fileList = file.list();
            //没有子文件和压缩
            if (fileList.length <= 0) {
                ZipEntry zipEntry = new ZipEntry(fileString + File.separator);
                zipOutputSteam.putNextEntry(zipEntry);
                zipOutputSteam.closeEntry();
            }
            //子文件和递归
            for (int i = 0; i < fileList.length; i++) {
                ZipFiles(folderString, fileString + File.separator + fileList[i], zipOutputSteam);
            }
        }
    }

    /**
     * 返回zip的文件输入流
     *
     * @param zipFileString zip的名称
     * @param fileString    ZIP的文件名
     * @return InputStream
     * @throws Exception
     */
    public static InputStream UpZip(String zipFileString, String fileString) throws Exception {
        ZipFile zipFile = new ZipFile(zipFileString);
        ZipEntry zipEntry = zipFile.getEntry(fileString);
        return zipFile.getInputStream(zipEntry);
    }

    /**
     * 返回ZIP中的文件列表（文件和文件夹）
     *
     * @param zipFileString  ZIP的名称
     * @param bContainFolder 是否包含文件夹
     * @param bContainFile   是否包含文件
     * @return
     * @throws Exception
     */
    public static List<File> GetFileList(String zipFileString, boolean bContainFolder, boolean bContainFile) throws Exception {
        List<File> fileList = new ArrayList<File>();
        ZipInputStream inZip = new ZipInputStream(new FileInputStream(zipFileString));
        ZipEntry zipEntry;
        String szName = "";
        while ((zipEntry = inZip.getNextEntry()) != null) {
            szName = zipEntry.getName();
            if (zipEntry.isDirectory()) {
                // 获取部件的文件夹名
                szName = szName.substring(0, szName.length() - 1);
                File folder = new File(szName);
                if (bContainFolder) {
                    fileList.add(folder);
                }
            } else {
                File file = new File(szName);
                if (bContainFile) {
                    fileList.add(file);
                }
            }
        }
        inZip.close();
        return fileList;
    }

}
```

使用

```java
// 需要解压的文件
String zipFileName = Environment.getExternalStorageDirectory().getPath() + "/xxxx/resource/"+fileName;
// 解压到的目录位置
String zipToFilePath = Environment.getExternalStorageDirectory().getPath() + "/xxxx/"+fileName;
// 解压指定文件到指定目录 ，成功后删除
try {
ZipUtils.UnZipFolder(zipFileName, zipToFilePath,getActivity());
hasDowmResourceNum++; // 已经下载完成的资源个数+1
} catch (Exception e) {
e.printStackTrace();
Log.i("xqxinfo","e:"+e.getMessage().toString());
hasDowmErrorResourceNum++; // 下载失败的资源个数+1
}
```



### 参考

1. [自定义样式日期时间选择对话框控件（精简版）](https://blog.csdn.net/wjj1996825/article/details/80573507)
2. [Android自定义Dialog对话框的几种方法（精简版](https://blog.csdn.net/wjj1996825/article/details/80522104)