---
title: Android 常用控件（一）
date: 2019-04-29 13:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### 前言

Widget是为构建用户交互界面提供服务的视图对象。Widget类是View类的子类。Android的可视控件都在android.widget包内。

<!--more-->

**控件**

- TextView 显示文字，相当于Panel
- ImageView 显示图片
- EditText 输入框，可编辑,可设置软键盘方式
- Button 按钮，可附带图片
- CheckBox 复选框
- RadioButton 单选按钮（和 RadioGroup 配合使用）

**按用途分类：**

文本控件
- TextView
- EditText

按钮控件
- Button
- ImageButton

状态开关按钮
- ToggleButton

单选与复选按钮
- CheckBox和RadioButton

图片控件
- ImageView

时钟控件
- AnalogClock
- DigitalClock

日期与时间选择控件
- DatePicker
- TimePicker


设置控件的属性有两种方法,一种是在布局文件中设置参数,另一种是在代码中调用对应方法实现，以下描述的都只是在布局文件中设置参数的方法。

介绍这些控件之前先介绍下所有控件都有的4个属性id、layout_width以及layout_height，以及android:visibility。
```
android:id = "@+id/xxx"  @+id/xxx表示新增控件命名为xxx
android:layout_width = "xxx"
android:layout_height = "xxx"
//下面这个属性默认可以省略
android:visibility = "visible"
```
其中layout_width以及layout_height属性可选值有两种 match_parent和wrap_content:

- match_parent表示让当前控件的大小和父布局的大小一样,也就是由父布局来决定当前控件的大小；
- wrap_content表示让当前控件的大小能够刚好包含住里面的内容,也就是由控件内容决定当前控件的大小。

android:visibility表示控件的可见属性,所有的Android控件都具有这个属性,可以通过 android:visibility 进行指定,可选值有三种,visible、invisible 和 gone。visible 表示控件是可见的,这个值是 默认值,不指定 android:visibility 时,控件都是可见的。invisible 表示控件不可见,但是它仍 然占据着原来的位置和大小,可以理解成控件变成透明状态了。gone 则表示控件不仅不可见, 而且不再占用任何屏幕空间。一般用在Activity中通过setVisibility方法来指定呈现与否。

下面介绍的控件属性了解即可，不用记忆。

### 文本类控件TextView
TextView是 Android 程序开发中最常用的控件之一,主要功能是向用户展示文本的内容，它是不可编辑的 ,只能通过初始化设置或在程序中修改。

**属性：**

1. android:autoLink设置是否当文本为URL链接/email/电话号码/map时，文本显示为可点击的链接。可选值(none/web/email/phone/map/all)

2. android:autoText如果设置，将自动执行输入值的拼写纠正。此处无效果，在显示输入法并输入的时候起作用。
   android:bufferType指定getText()方式取得的文本类别。选项editable 类似于StringBuilder可追加字符，

3. 也就是说getText后可调用append方法设置文本内容。spannable 则可在给定的字符区域使用样式，参见这里1、这里2。

4. android:capitalize设置英文字母大写类型。此处无效果，需要弹出输入法才能看得到，参见EditView此属性说明。

5. android:cursorVisible设定光标为显示/隐藏，默认显示。

6. android:digits设置允许输入哪些字符。如“1234567890.+-*/% ()”

7. android:drawableBottom在text的下方输出一个drawable，如图片。如果指定一个颜色的话会把text的背景设为该颜色，并且同时和background使用时覆盖后者。

8. android:drawableLeft在text的左边输出一个drawable，如图片。

9. android:drawablePadding设置text与drawable(图片)的间隔，与drawableLeft、drawableRight、drawableTop、drawableBottom一起使用，可设置为负数，单独使用没有效果。

10. android:drawableRight在text的右边输出一个drawable。

11. android:drawableTop在text的正上方输出一个drawable。

12. android:editable设置是否可编辑。

13. android:editorExtras设置文本的额外的输入数据。

14. android:ellipsize设置当文字过长时,该控件该如何显示。有如下值设置：”start”—?省略号显示在开头;”end”——省略号显示在结尾;”middle”—-省略号显示在中间;

15. ”marquee” ——以跑马灯的方式显示(动画横向移动)

16. android:freezesText设置保存文本的内容以及光标的位置。

17. android:gravity设置文本位置，如设置成“center”，文本将居中显示。

18. android:hintText为空时显示的文字提示信息，可通过textColorHint设置提示信息的颜色。此属性在EditView中使用，但是这里也可以用。

19. android:imeOptions附加功能，设置右下角IME动作与编辑框相关的动作，如actionDone右下角将显示一个“完成”，而不设置默认是一个回车符号。这个在EditView中再详细

20. 说明，此处无用。

21. android:imeActionId设置IME动作ID。

22. android:imeActionLabel设置IME动作标签。

23. android:includeFontPadding设置文本是否包含顶部和底部额外空白，默认为true。

24. android:inputMethod为文本指定输入法，需要完全限定名(完整的包名)。例如：com.google.android.inputmethod.pinyin，但是这里报错找不到。

25. android:inputType设置文本的类型，用于帮助输入法显示合适的键盘类型。在EditView中再详细说明，这里无效果。

26. android:linksClickable设置链接是否点击连接，即使设置了autoLink。

27. android:marqueeRepeatLimit在ellipsize指定marquee的情况下，设置重复滚动的次数，当设置为marquee_forever时表示无限次。

28. android:ems设置TextView的宽度为N个字符的宽度。这里测试为一个汉字字符宽度

29. android:maxEms设置TextView的宽度为最长为N个字符的宽度。与ems同时使用时覆盖ems选项。

30. android:minEms设置TextView的宽度为最短为N个字符的宽度。与ems同时使用时覆盖ems选项。

31. android:maxLength限制显示的文本长度，超出部分不显示。

32. android:lines设置文本的行数，设置两行就显示两行，即使第二行没有数据。

33. android:maxLines设置文本的最大显示行数，与width或者layout_width结合使用，超出部分自动换行，超出行数将不显示。

34. android:minLines设置文本的最小行数，与lines类似。

35. android:lineSpacingExtra设置行间距。

36. android:lineSpacingMultiplier设置行间距的倍数。如”1.2”

37. android:numeric如果被设置，该TextView有一个数字输入法。此处无用，设置后唯一效果是TextView有点击效果，此属性在EdtiView将详细说明。

38. android:password以小点”.”显示文本

39. android:phoneNumber设置为电话号码的输入方式。

40. android:privateImeOptions设置输入法选项，此处无用，在EditText将进一步讨论。

41. android:scrollHorizontally设置文本超出TextView的宽度的情况下，是否出现横拉条。

42. android:selectAllOnFocus如果文本是可选择的，让他获取焦点而不是将光标移动为文本的开始位置或者末尾位置。TextView中设置后无效果。

43. android:shadowColor指定文本阴影的颜色，需要与shadowRadius一起使用。

44. 
    android:shadowDx设置阴影横向坐标开始位置。

45. android:shadowDy设置阴影纵向坐标开始位置。

46. android:shadowRadius设置阴影的半径。设置为0.1就变成字体的颜色了，一般设置为3.0的效果比较好。

47. android:singleLine设置单行显示。如果和layout_width一起使用，当文本不能全部显示时，后面用“…”来表示。如android:text="test_ singleLine "    

48. android:singleLine="true" android:layout_width="20dp"将只显示“t…”。如果不设置singleLine或者设置为false，文本将自动换行

49. android:text设置显示文本.

50. android:textAppearance设置文字外观。如“?android:attr/textAppearanceLargeInverse”这里引用的是系统自带的一个外观，?表示系统是否有这种外观，否则使用默认的外观。可设置的值如下：textAppearanceButton/textAppearanceInverse/textAppearanceLarge/textAppearanceLargeInverse/textAppearanceMedium/textAppearanceMediumInverse/textAppearanceSmall/textAppearanceSmallInverse

51. android:textColor设置文本颜色

52. android:textColorHighlight被选中文字的底色，默认为蓝色

53. android:textColorHint设置提示信息文字的颜色，默认为灰色。与hint一起使用。

54. android:textColorLink文字链接的颜色.

55. android:textScaleX设置文字之间间隔，默认为1.0f。

56. android:textSize设置文字大小，推荐度量单位”sp”，如”15sp”

57. android:textStyle设置字形[bold(粗体) 0, italic(斜体) 1, bolditalic(又粗又斜) 2] 可以设置一个或多个，用“|”隔开

58. android:typeface设置文本字体，必须是以下常量值之一：normal 0, sans 1, serif 2, monospace(等宽字体) 3]

59. android:height设置文本区域的高度，支持度量单位：px(像素)/dp/sp/in/mm(毫米)

60. android:maxHeight设置文本区域的最大高度

61. android:minHeight设置文本区域的最小高度

62. android:width设置文本区域的宽度，支持度量单位：px(像素)/dp/sp/in/mm(毫米)，与layout_width的区别看这里。

63. android:maxWidth设置文本区域的最大宽度

64. android:minWidth设置文本区域的最小宽度

一般不对TextView文本进行修改，所以在Activity中的使用此处略。

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <!-- 设置文字颜色、大小、样式 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="红色粗体倾斜的TextView"
        android:textColor="#EA5246"
        android:textStyle="bold|italic"
        android:textSize="18sp" />
 
    <!-- 使用阴影 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:shadowColor="#F9F900"
        android:shadowDx="10.0"
        android:shadowDy="10.0"
        android:shadowRadius="3.0"
        android:text="带阴影的TextView"
        android:textColor="#4A4AFF"
        android:textSize="30sp" />
 
    <!-- 带图片 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:drawableTop="@mipmap/ic_launcher"
        android:drawableLeft="@mipmap/ic_launcher"
        android:drawableRight="@mipmap/ic_launcher"
        android:drawableBottom="@mipmap/ic_launcher"
        android:text="带图片的TextView" />
 
    <!-- 对邮件、电话、网址增加链接 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="邮件：123465@163.com\n电话：1222228888\n博客：https://hanyunpeng0521.github.io/"
        android:autoLink="email|phone|web"/>
 
    <!-- 实现跑马灯效果 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:singleLine="true"
        android:ellipsize="marquee"
        android:marqueeRepeatLimit="marquee_forever"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:text="实现跑马灯效果的TextView，WE ARE A TEAM"/>
</LinearLayout>
```

<img src="https://i.loli.net/2019/05/08/5cd26e3988102.jpg" height=600 >

### 文本类控件EditText

EditText继承关系：View-->TextView-->EditText,所以EditText可以使用TextView的属性

相比TextView, EditText是可以编辑的，可以用来与用户进行交互，其用法和TextView也是类似的，同样，下面介绍一些常见的参数

EditText支持的XML属性及相关方法见TextView表中介绍的与输入有关的属性和方法，其中比较重要的一个属性是inputType，用于为EditText设置输入类型，其属性值主要有以下一些。

```
android:inputType=none：普通字符。
android:inputType=text：普通字符。
android:inputType=textCapCharacters：字母大写。
android:inputType=textCapWords：首字母大写。
android:inputType=textCapSentences：仅第一个字母大写。
android:inputType=textAutoCorrect：自动完成。
android:inputType=textAutoComplete：自动完成。
android:inputType=textMultiLine：多行输入。
android:inputType=textImeMultiLine：输入法多行（如果支持）。
android:inputType=textNoSuggestions：不提示。
android:inputType=textUri：网址。
android:inputType=textEmailAddress：电子邮件地址。
android:inputType=textEmailSubject：邮件主题。
android:inputType=textShortMessage：短讯。
android:inputType=textLongMessage：长信息。
android:inputType=textPersonName：人名。
android:inputType=textPostalAddress：地址。
android:inputType=textPassword：密码。
android:inputType=textVisiblePassword：可见密码。
android:inputType=textWebEditText：作为网页表单的文本。
android:inputType=textFilter：文本筛选过滤。
android:inputType=textPhonetic：拼音输入。
android:inputType=number：数字。
android:inputType=numberSigned：带符号数字格式。
android:inputType=numberDecimal：带小数点的浮点格式。
android:inputType=phone：拨号键盘。
android:inputType=datetime：时间日期。
android:inputType=date：日期键盘。
android:inputType=time：时间键盘。
```


EditText还派生了如下两个子类。

- AutoCompleteTextView：带有自动完成功能的EditText。由于该类通常需要与 Adapter结合使用，因此将会在下一章进行学习。

- ExtractEditText：并不是UI组件，而是EditText组件的底层服务类，负责提供全屏输入法支

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="用户名:"
        android:textSize="16sp"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入用户名" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="密码:"
        android:textSize="16sp"/>
    <!-- android:inputType="numberPassword"表明只能接受数字密码 -->
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入密码"
        android:inputType="numberPassword"/>
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="年龄："
        android:textSize="16sp"/>
    <!-- inputType="number"表明是数值输入框 -->
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入年龄"
        android:inputType="number"/>
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="生日："
        android:textSize="16sp"/>
    <!-- inputType="date"表明是日期输入框 -->
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入生日"
        android:inputType="date"/>
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="电话号码:"
        android:textSize="16sp"/>
    <!-- inputType="phone"表明是输入电话号码的输入框 -->
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入电话号码"
        android:inputType="phone"/>
 
</LinearLayout>
```



### 按钮类控件Button

Button控件也是使用过程中用的最多的控件之一，所以需要好好掌握。用户可以通过单击 Button 来触发一系列事件,然后为 Button 注册监听器,来实现 Button 的监听事件。

首先来看下Button的配置属性，其实和TextView差不多设置更简单点，主要是显示到Button上面的文字提示：

```
<Button

//控件id
android:id = "@+id/xxx"  @+id/xxx表示新增控件命名为xxx

//宽度与高度
android:layout_width="wrap_content"  //wrap_content或者match_parent
android:layout_height="wrap_content"  //wrap_content或者match_parent

//按钮上显示的文字 
android:text="theButton" //两种方式，直接具体文本或者引用values下面的string.xml里面的元素@string/button

//按钮字体大小
android:textSize="24sp"  //以sp为单位

//字体颜色
android:textColor="#0000FF"  //RGB颜色

//字体格式
android:textStyle="normal"  //normal,bold,italic分别为正常，加粗以及斜体，默认为normal
//图片
android:background="@drawable/button"
//是否只在一行内显示全部内容
android:singleLine="true"  //true或者false，默认为false
```

然后我们需要在Activity中为Button的点击事件注册一个监听器。具体参看事件处理。



### 按钮类控件RadioButton与RadioGroup

RadioButton(单选按钮)在 Android  平台上也比较常用,比如一些选择项会用到单选按钮。它是一种单个圆形单选框双状态的按钮,可以选择或不选择。在 RadioButton 没有  被选中时,用户通过单击来选中它。但是,在选中后,无法通过单击取消选中。

RadioGroup 是单选组合框,用于 将 RadioButton 框起来。在多个 RadioButton被 RadioGroup  包含的情况下,同一时刻只可以选择一个 RadioButton,并用 setOnCheckedChangeListener 来对  RadioGroup 进行监听。

下面介绍RadioGroup的常用的属性，因为其中包含有RadioButton:

```
 <?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="请选择性别"/>
 
    <RadioGroup
        android:id="@+id/sex_rg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" >
 
        <RadioButton
            android:id="@+id/male_rb"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="男"
            android:checked="true"/>
 
        <RadioButton
            android:id="@+id/female_rb"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="女"/>
    </RadioGroup>
</LinearLayout>
```

下面给出在Activity中用 setOnCheckedChangeListener 来对 RadioGroup 进行监听的代码, 注意RadioGroup中的RadioButton也都是需要声明和通过控件的id来得到代表控件的对象。

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.RadioButton;
import android.widget.RadioGroup;
import android.widget.Toast;
 
public class MainActivity extends AppCompatActivity {
    private RadioButton mMaleRb = null; // 性别男单选按钮
    private RadioButton mFemaleRb = null; // 性别女单选按钮
    private RadioGroup mSexRg = null; // 性别单选按钮组
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.radiobutton_layout);
 
        // 获取界面组件
        mMaleRb = (RadioButton) findViewById(R.id.male_rb);
        mFemaleRb = (RadioButton) findViewById(R.id.female_rb);
        mSexRg = (RadioGroup) findViewById(R.id.sex_rg);
 
        // 为单选按钮组绑定OnCheckedChangeListener监听器
        mSexRg.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup radioGroup, int checkedId) {
                // 获取用户选中的性别
                String sex = "";
                switch (checkedId) {
                    case R.id.male_rb:
                        sex = mMaleRb.getText().toString();
                        break;
                    case R.id.female_rb:
                        sex = mFemaleRb.getText().toString();
                        break;
                    default:break;
                }
 
                // 消息提示
                Toast.makeText(MainActivity.this,
                        "选择的性别是：" + sex, Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

### 按钮类控件CheckBox

CheckBox(复选按钮),顾名思义是一种可以进行多选的按钮,默认以矩形表示。与 RadioButton  相同,它也有选中或者不选中双状态。我们可以先在布局文件中定义多选按钮, 然后对每一个多选按钮进行事件监听  setOnCheckedChangeListener,通过 isChecked 来判断 选项是否被选中,做出相应的事件响应。

下面给出CheckBox在布局文件中的常用的属性以及用法：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="选择喜欢的城市"/>
 
    <CheckBox
        android:id="@+id/shanghai_cb"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="上海"
        android:checked="true"/>
 
    <CheckBox
        android:id="@+id/beijing_cb"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="北京"/>
 
    <CheckBox
        android:id="@+id/chongqing_cb"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="重庆"/>
</LinearLayout>
```

在Activity中调用的代码如下：

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.CheckBox;
import android.widget.CompoundButton;
import android.widget.Toast;
 
public class MainActivity extends AppCompatActivity {
    private CheckBox mShanghaiCb = null; // 上海复选框
    private CheckBox mBeijingCb = null; // 北京复选框
    private CheckBox mChongqingCb = null; // 重庆复选框
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.checkbox_layout);
 
       // 获取界面组件
        mShanghaiCb = (CheckBox) findViewById(R.id.shanghai_cb);
        mBeijingCb = (CheckBox) findViewById(R.id.beijing_cb);
        mChongqingCb = (CheckBox) findViewById(R.id.chongqing_cb);
 
        // 为上海复选框绑定OnCheckedChangeListener监听器
        mShanghaiCb.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
                // 提示用户选择的城市
                showSelectCity(compoundButton);
            }
        });// 为北京复选框绑定OnCheckedChangeListener监听器
        mBeijingCb.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
                // 提示用户选择的城市
                showSelectCity(compoundButton);
            }
        });// 为重庆复选框绑定OnCheckedChangeListener监听器
        mChongqingCb.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
                // 提示用户选择的城市
                showSelectCity(compoundButton);
            }
        });
    }
 
    /**
     * 提示用户选择的城市
     * @param compoundButton
     */
    private void showSelectCity(CompoundButton compoundButton){
        // 获取复选框的文字提示
        String city = compoundButton.getText().toString();
 
        // 根据复选框的选中状态进行相应提示
        if(compoundButton.isChecked()) {
            Toast.makeText(MainActivity.this, "选中" + city, Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(MainActivity.this, "取消选中" + city, Toast.LENGTH_SHORT).show();
        }
    }
}
```

从上面的Java代码可以看到，有很大一部分代码都是冗余的，大家可以思考一下是否可以有其他办法来处理这个问题呢？

### 图片控件ImageView

ImageView 是一个图片控件,负责显示图片，图片的来源可以是系统提供的资源文件,也可以是 Drawable 对象，相对来说，图片空间还是比较好掌握的，因为前面有讲过ImageButton, 很多属性都是相同的。 
下面直接给出在布局中的属性：

```
<ImageView
//控件id
android:id = "@+id/xxx"  @+id/xxx表示新增控件命名为xxx

//宽度与高度
android:layout_width="wrap_content"   //wrap_content或者match_parent
android:layout_height="wrap_content"  //wrap_content或者match_parent

//此外，可以具体设置高度和宽度显示的像素，不过这样设置如果图片尺寸大于设置的显示的尺寸，则图片是显示不全的,这是可以配合android:scaleType属性。
android:layout_width="200dp"
android:layout_height="200dp" 

ImageView所支持的android:scaleType属性可指定如下属性值。

matrix ( ImageView.ScaleType.MATRIX)：使用 matrix 方式进行缩放。
fitXY ( lmageView.ScaleType.FIT_XY)：对图片横向、纵向独立缩放，使得该图片完全适应于该ImageView,图片的纵横比可能会改变。
fitStart (ImageView.ScaleType.FIT_START )：保持纵横比缩放图片，直到该图片能完全显示在ImageView中（图片较长的边长与ImageView相应的边长相等）,缩放完成后将该图片放在ImageView的左上角。
fitCenter (ImageView.ScaleType.FIT_CENTER )：保持纵横比缩放图片，直到该图片能完全显示在ImageView中（图片较长的边长与ImageView相应的边长相等）， 缩放完成后将该图片放在ImageView的中央。
fitEnd (ImageView.ScaleType.FIT_END )：保持纵横比缩放图片，直到该图片能完全显示在ImageView中（图片较长的边长与ImageView相应的边长相等），缩放完成后将该图片放在ImageView的右下角。
center ( ImageView.ScaleType.CENTER)：把图片放在 ImageView 的中间，但不进行任何缩放。
centerCrop ( ImageView.ScaleType.CENTER_CROP)：保持纵横比缩放图片，以使得图片能完全覆盖ImageView。只要图片的最短边能显示出来即可。
centerlnside (ImageView.ScaleType.CENTER_INSIDE )：保持纵横比缩放图片，以使得ImageView能完全显示该图片。


//图片来源，需要将图片复制放到res/drawable文件夹里面，引用的时候不需要写图片的后缀
android:src ="@drawable/beautiful"/>  
```

附上android:scaleType不同参数对应效果图: 

![1.png](https://i.loli.net/2019/05/08/5cd26184af9a2.png)

为了控制ImageView显示的图片，ImageView提供了如下方法。

- setlmageBitmap(Bitmap bm)：使用 Bitmap 位图设置该 ImageView 显示的图片。
- setlmageDrawable(Drawable drawable)：使用 Drawable 对象设置该 ImageView 显示的图片。
- setlmageResource(int resld)：使用图片资源ID设置该ImageView显示的图片。

- setlmageURI(Uri uri)：使用图片的URI设置该ImageView显示的图片

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/girl"/>
 
    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:scaleType="fitXY"
        android:src="@drawable/girl"/>
 
    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:scaleType="center"
        android:src="@drawable/girl"/>
</LinearLayout>
```

java文件中设置布局文件即可。

### 按钮类控件ImageButton

在Android开发中除了使用Button按钮，还可以使用自带图标的按钮，即ImageButton。Button与ImageButton的区别在于，Button生成的按钮上显示文字，而ImageButton上则显示图片。

需要指出的是，为ImageButton按钮指定android:text属性没用，由于ImageButton的本质是ImageView，即使指定了该属性，图片按钮上也不会显示任何文字。

使用ImageButton图片按钮可以指定android:src属性，该属性既可使用静止的图片，也可使用自定义的Drawable对象，这样即可开发出随用户动作改变图片的按钮。

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <ImageButton
        android:id="@+id/control_ib"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:scaleType="fitXY"
        android:src="@drawable/fast"/>
</LinearLayout>
```

java文件

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.ImageButton;
import android.widget.Toast;
 
public class MainActivity extends AppCompatActivity {
    private ImageButton mControlIb = null; // 播放控制按钮
    private boolean mFlag = false; // 播放控制标记符,默认暂停状态
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.imagebutton_layout);
 
        // 获取界面组件
        mControlIb = (ImageButton) findViewById(R.id.control_ib);
 
        // 为图标按钮绑定OnClickListener监听器
        mControlIb.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 根据记录的控制状态进行图标切换
                if(mFlag) {
                    Toast.makeText(MainActivity.this, "暂停", Toast.LENGTH_SHORT).show();
                    mControlIb.setImageResource(R.drawable.fast);
                    mFlag = false;
                }else {
                    Toast.makeText(MainActivity.this, "快进", Toast.LENGTH_SHORT).show();
                    mControlIb.setImageResource(R.drawable.pause);
                    mFlag = true;
                }
            }
        });
    }
}
```

### ZoomButton

ImageButton派生了一个ZoomButton，ZoomButton可以代表“放大”、“缩小”两个按钮。 ZoomButton 的行为基本类似于 ImageButton，只是 Android 默认提供了 btn_minus、btn_plus 两个 Drawable 资源，只要为 ZoomButton 的 android:src 属性分别指定 btn_minus、btn_plus，即可实现“缩小”、“放大”按钮。当然也可以自己指定图片资源。

实际上Android还提供了一个ZoomControls组件，该组件相当于同时组合了 “放大”、“缩 小”两个按钮，并允许分别为两个按钮绑定不同的事件监听器。

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
 
        <ZoomButton
            android:id="@+id/minus_zb"
            android:layout_width="80dp"
            android:layout_height="80dp"
            android:src="@android:drawable/btn_minus"/>
 
        <ZoomButton
            android:id="@+id/plus_zb"
            android:layout_width="80dp"
            android:layout_height="80dp"
            android:src="@android:drawable/btn_plus"/>
    </LinearLayout>
 
    <ZoomControls
        android:id="@+id/control_zc"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

java文件

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Toast;
import android.widget.ZoomButton;
import android.widget.ZoomControls;
 
public class MainActivity extends AppCompatActivity {
    private ZoomButton mMinusZb = null; // 缩小按钮
    private ZoomButton mPlusZb = null; // 放大按钮
    private ZoomControls mControlZc = null; //缩放组件
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.zoombutton_layout);
 
        // 获取界面组件
        mMinusZb = (ZoomButton) findViewById(R.id.minus_zb);
        mPlusZb = (ZoomButton) findViewById(R.id.plus_zb);
        mControlZc = (ZoomControls) findViewById(R.id.control_zc);
 
        // 为缩小按钮绑定OnClickListener监听器
        mMinusZb.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(MainActivity.this, "缩小", Toast.LENGTH_SHORT).show();
            }
        });
        // 为放大按钮绑定OnClickListener监听器
        mPlusZb.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(MainActivity.this, "放大", Toast.LENGTH_SHORT).show();
            }
        });
 
        // 为缩放组件绑定OnZoomInClickListener监听器
        mControlZc.setOnZoomInClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(MainActivity.this, "放大", Toast.LENGTH_SHORT).show();
            }
        });
        // 为缩放组件绑定OnZoomOutClickListener监听器
        mControlZc.setOnZoomOutClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(MainActivity.this, "缩小", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

发现使用ZoomControls也能轻松实现需要达到的目的。



### ProgressBar

ProgressBar 用于在界面上显示一个进度条,表示我们的程序正在加载一些数据，运行程序，会看到屏幕中有一个圆形进度条正在旋转。 

ProgressBar继承于View类，直接子类有AbsSeekBar和ContentLoadingProgressBar， 其中AbsSeekBar的子类有SeekBar和RatingBar，可见这二者是基于ProgressBar实现的。

进度条也是UI界面中一种非常实用的组件，通常用于向用户显示某个耗时操作完成的百分比。

进度条可以动态地显示进度，因此避免长时间地执行某个耗时操作时，让用户感觉程序失去了响应，从而更好地提高用户界面的友好性。

Android支持多种风格的进度条，通过style属性可以为ProgressBar指定风格。该属性可支持如下几个属性值：

- @android:style/Widget.ProgressBar.Horizontal:水平进度条。

- @android:style/Widget.ProgressBar.Inverse：普通大小的环形进度条。

- @android:style/Widget.ProgressBar.Large：大环形进度条。

- @android:style/Widget.ProgressBar.Large.Inverse：大环形进度条。

- @android:style/Widget.ProgressBar.Small：小环形进度条。

- @android:style/Widget.ProgressBar.Small.Inverse：小环形进度条。


其实在Android开发中，ProgressBar的样式设定有两种方式，除了上面这种，还有一种可以通过如下方式使用：

- ?android:attr/progressBarStyle

- ?android:attr/progressBarStyleHorizontal

- ?android:attr/progressBarStyleInverse

- ?android:attr/progressBarStyleLarge

- ?android:attr/progressBarStyleLargeInverse

- ?android:attr/progressBarStyleSmall

- ?android:attr/progressBarStyleSmallInverse

- ?android:attr/progressBarStyleSmallTitle


除此之外，ProgressBar还支持如下常用XML属性：

- android:max：进度条的最大值。

- android:progress：进度条已完成进度值。

- android:progressDrawable：设置轨道对应的Drawable对象。

- android:indeterminate：如果设置成true，则进度条不精确显示进度。

- android:indeterminateDrawable：设置不显示进度的进度条的Drawable对象。

- android:indeterminateDuration：设置不精确显示进度的持续时间。

- android:secondaryProgress：二级进度条，类似于视频播放的一条是当前播放进度，一条是缓冲进度。


其中android:progressDrawable用于指定进度条的轨道的绘制形式，该属性可指 定为一个LayerDrawable对象的引用。

- ProgressBar提供了如下方法来操作进度：

- getMax()：返回这个进度条的范围的上限。

- getProgress()：返回进度。

- getSecondaryProgress()：返回次要进度。

- incrementProgressBy(int diff)：指定增加的进度。为正数时进度增加；为负数时进度减少。

- isIndeterminate()：指示进度条是否在不确定模式下。

- setIndeterminate(boolean indeterminate)：设置是否为不确定模式。

在布局xml文件中的用法非常简单：

```
 <ProgressBar 
    android:id="@+id/pb"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    //默认是圆形进度条，可以知道样式设置为水平进度条
    style="?android:attr/progressBarStyleHorizontal"/>
    //指定成水平进度条后,我们还可以通过 android:max属性给进度条设置一个最大值,然后在代码中动态地更改进度条的进度
    android:max="100"
    />
```

那么如何才能让进度条在数据加载完成时消失呢，这里我们就需要用一开始所讲的Android 控件的可见属性。 
可以通过代码来设置控件的可见性,使用的是 setVisibility()方法,可以传入 View.VISIBLE、View.INVISIBLE 和 View.GONE 三种值。

下面实现点击一下按钮让进度条消失,再点击一下按钮让进度条出现的这种效果,这里只给出按钮监听的代码：

```
button.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                //通过 getVisibility()方法来判断 ProgressBar 是否可见
                if (progressBar.getVisibility() == View.GONE) {
                    progressBar.setVisibility(View.VISIBLE);
                } else {
                    progressBar.setVisibility(View.GONE);
                }
            }
        });
```

至此，关于Android的常用控件都已经讲了一遍，此处再对将关于TextView中文字单独在一行显示的时候实现跑马灯的方法补充下，这里直接给出代码

```

    <?xml version="1.0" encoding="utf-8"?>  
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
        android:layout_width="fill_parent"  
        android:layout_height="fill_parent"  
        android:orientation="vertical"   
        >  
        <LinearLayout android:layout_width="fill_parent"  
            android:layout_height="100dip"  
            android:clickable="true"  
            android:focusable="true"  
            android:focusableInTouchMode="true"  
            />  
        <TextView  
            android:layout_width="100dip"  
            android:layout_height="wrap_content"  
            android:layout_gravity="center"  
            android:text="走马灯效果的演示"   
            android:singleLine="true"  
            android:ellipsize="marquee"  
            android:clickable="true"  
            android:focusable="true"  
            android:focusableInTouchMode="true"  
            ></TextView>  
      
    </LinearLayout>  
```

这段代码运行之后，首先走马灯不出现，代码中的第二个LinearLayout获得焦点，然后我们点击第二个TextView，走马灯出现，然后我们点击TextView上方的区域，走马灯效果消失，说明焦点转移到代码中的第二个LinearLayout。

好的，总结一下也就是说，在触摸模式下android:clickable="true"是（android:focusable="true"，android:focusableInTouchMode="true"）能获得焦点的必要条件，就是说一个控件想要在触摸模式下获得焦点就一定要可点击，上面三个属性都要有。

### ToggleButton

ToggleButton有两种状态：选中状态和没选中状态（类似一个开关），并且需要为不同的状态设置不同的显示文本。

```
　　android:checked = "true"  ——按钮的状态（true为选中（textOn），false为没有选中（textOff））
　　android:textOff = "关"
　　android:textOn = "开"  
```

示例：

先将下面两个图片放入到资源文件中（分别命名为off和on）



![1.png](https://i.loli.net/2019/05/08/5cd268481152e.png)

![2.png](https://i.loli.net/2019/05/08/5cd2684749df1.png)

xml文件

```
//xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >

    <ToggleButton
        android:id="@+id/toggleButton1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textOn="开"
        android:textOff="关" />

    <ImageView
        android:id="@+id/imageView1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:src="@drawable/off" />
    
</LinearLayout>
```

java文件

```
//java
import android.os.Bundle;
import android.widget.CompoundButton;
import android.widget.CompoundButton.OnCheckedChangeListener;
import android.widget.ImageView;
import android.widget.ToggleButton;
import android.app.Activity;


public class MainActivity extends Activity implements OnCheckedChangeListener{
    private ToggleButton tb;
    private ImageView img;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //第一步：初始化控件（找到需要操作的控件）
        tb = (ToggleButton) findViewById(R.id.toggleButton1);
        img = (ImageView) findViewById(R.id.imageView1);
        
        //给当前的tb控件设置监听器
        tb.setOnCheckedChangeListener(this);
        
    }
    
    @Override
    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
        //当tb被点击的时候，当前的方法会执行
        //buttonView——代表被点击控件的本身
        //isChecked——代表被点击的控件的状态
        //当点击tb这个控件的时候更好img的src
        img.setImageResource(isChecked?R.drawable.on:R.drawable.off);//在java代码中设置ImageView控件的src路径（这里使用三元运算来设置图片的路径）
        
    }
}

```

### Switch

Switch是一个可以在两种状态切换之间切换的开关控件。用户可以拖动来选择，也可以像选择复选框一样点击切换Switch的状态。状态改变时，会触发一个OnCheckedChange事件。

Switch所支持的XML属性和相关方法如下表所示。

![1.jpeg](https://i.loli.net/2019/05/08/5cd272c8839a2.jpeg)

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="打开/关闭蓝牙"
        android:textSize="22sp"/>
 
    <Switch
        android:id="@+id/bluetooth_switch"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

java代码

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.CompoundButton;
import android.widget.Switch;
import android.widget.Toast;
 
public class MainActivity extends AppCompatActivity {
    private Switch mBluetoothSwitch = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.switch_layout);
 
        // 获取界面组件
        mBluetoothSwitch = (Switch) findViewById(R.id.bluetooth_switch);
 
        // 为开关按钮绑定OnCheckedChangeListener监听器
        mBluetoothSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
                if(compoundButton.isChecked()) {
                    Toast.makeText(MainActivity.this, "打开蓝牙", Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(MainActivity.this, "关闭蓝牙", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }
}
```





### 时钟组件TextClock和AnalogClock

1. 时钟UI组件是两个非常简单的组件，DigitalClock本身就集成了TextView—也就是说它本身就是文本框，只是它里面显示的内容是当前时间；AnalogClock则继承了View组件，它重写了View的OnDraw方法，它会在View上显示模拟时钟。

2. DigitalClock和AnalogClock都会显示当前时间。不同的是，DigitalClock显示数字时钟，可以显示当前秒数，AnalogClock显示模拟时钟，不会显示当前秒数。

3. 关于时间的文本显示，Android提供了DigitalClock和TextClock。DigitalClock是Android第1版本发布，功能很简单，只显示时间；在Android4.2(对应API Level 17)中，Android新增了TextClock。TextClock的功能更加强大，它不仅能显示时间，还能显示日期；而且支持自定义格式。因此，推荐在Android4.2之后都使用TextClock。

**AnalogClock**

AnalogClock的XML有3个属性，分别如下：

- android:dial：模拟时钟的表背景。
- android:hand_hour：模拟时钟的表时针。
- android:hand_minute：模拟时钟的表分针

可以使用系统默认的，也可以自定义图片资源。

TextClock提供了两种不同的格式，一种是在24进制中显示时间和日期，另一种是在12进制中显示时间和日期。

- TextClock主要有以下几个XML属性：

- android:format12Hour：设置12时制的格式

- android:format24Hour：设置24时制的格式

- android:timeZone：设置时区。

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
    <AnalogClock
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:dial="@drawable/dial"
        android:layout_centerHorizontal="true"/>
</RelativeLayout>
```



**TextClock**

DigitalClock从API 17开始就已经过时了，建议使用新组件TextClock。TextClock的功能更加强大，它不仅能显示时间，还能显示日期；而且支持自定义格式。

TextClock提供了两种不同的格式，一种是在24进制中显示时间和日期，另一种是在12进制中显示时间和日期。

TextClock的主要方法有：

- getFormat12Hour()：在12进制模式中返回时间模式。

- getFormat24Hour()：在24进制模式中返回时间模式。

- getTimeZone()：返回正在使用的时区。

- is24HourModeEnabled()：检测系统当前是否使用24进制。

- setFormat24Hour(CharSequence format)：设置24时制的格式。

- setFormat12Hour(CharSequence format)：设置12时制的格式。

- setTimeZone(String timeZone)：设置时区

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
    <TextClock
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="5dp"
        android:layout_centerHorizontal="true"
        android:gravity="center_horizontal"
        android:format24Hour="yyyy年MM月dd EE\naa HH:mm:ss"
        android:format12Hour="yyyy年MM月dd EE\naa hh:mm:ss"
        android:textStyle="normal"
        android:fontFamily="sans-serif-light"
        android:textSize="26sp"
        tools:targetApi="jelly_bean_mr1"/>
</RelativeLayout>
```

其中format同Java里面学习的SimpleDateFormat类一样。android:fontFamily=”sans-serif-light”用于设置安卓字体。

### DatePicker

 DatePicker是一个比较简单的组件，从FrameLayout派生而来，供用户选择日期。其在FrameLayout的基础上提供了一些方法来获取当前用户所选择的日期，如果程序需要获取用户选择的日期则可通过为DatePicker添加 OnDateChangedListener 进行监听来实现。

使用DatePicker的常用XML属性如下：

- android:calendarViewShown：设置该日期选择是否显示CalendarView组件。

- android:endYear：设置日期选择器允许选择的最后一年。

- android:maxDate：设置该日期选择器的最大日期。以mm/dd/yyyy格式指定最大日期。

- android:minDate：设置该日期选择器的最小日期。以mm/dd/yyyy格式指定最小日期。

- android:spinnersShown：设置该日期选择器是否显示Spinner日期选择组件。

- android:startYear：设置日期选择器允许选择的第一年。

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
 
    <DatePicker
        android:id="@+id/datePicker"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:startYear="2015"
        android:endYear="2020"
        android:calendarViewShown="true"
        android:spinnersShown="true" />
</RelativeLayout>
```

java文件

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.DatePicker;
import android.widget.Toast;
 
import java.util.Calendar;

public class DatePickerActivity extends AppCompatActivity {
    private DatePicker mDatePicker = null; // 日期选择器
    private Calendar mCalendar = null; // 日历
    private int mYear; // 年
    private int mMonth; // 月
    private int mDay; // 日
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.datepicker_layout);
 
        // 获取日历对象
        mCalendar = Calendar.getInstance();
        // 获取当前对应的年、月、日的信息
        mYear = mCalendar.get(Calendar.YEAR);
        mMonth = mCalendar.get(Calendar.MONTH);
        mDay = mCalendar.get(Calendar.DAY_OF_MONTH);
 
        // 获取DatePicker组件
        mDatePicker = (DatePicker) findViewById(R.id.datePicker);
        // DatePicker初始化
        mDatePicker.init(mYear, mMonth, mDay, new DatePicker.OnDateChangedListener() {
            @Override
            public void onDateChanged(DatePicker view, int year, int monthOfYear, int dayOfMonth) {
                Toast.makeText(DatePickerActivity.this,
                        year + "年" + (monthOfYear + 1) + "月" + dayOfMonth + "日",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```





### TimePicker

TimePicker与DatePicker非常相似，主要是供用户选择时间。也是在FrameLayout的基础上提供了一些方法来获取当前用户所选择的时间，如果程序需要获取用户选择的时间则可通过为TimePicker添加
OnTimeChangedListener 进行监听来实现。

xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
 
    <TimePicker
        android:id="@+id/timePicker"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal" />
</RelativeLayout>
```

java文件

```
 import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.TimePicker;
import android.widget.Toast;
 
import java.util.Calendar;

public class TimePickerActivity extends AppCompatActivity {
    private TimePicker mTimePicker = null; // 时间选择器
    private Calendar mCalendar = null; // 日历
    private int mHour; // 时
    private int mMinute; // 分
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.timepicker_layout);
 
        // 获取日历对象
        mCalendar = Calendar.getInstance();
        // 获取对应的时、分的信息
        mHour = mCalendar.get(Calendar.HOUR_OF_DAY);
        mMinute = mCalendar.get(Calendar.MINUTE);
 
        // 获取TimePicker组件
        mTimePicker = (TimePicker) findViewById(R.id.timePicker);
        // 为TimePicker指定监听器
        mTimePicker.setOnTimeChangedListener(new TimePicker.OnTimeChangedListener() {
            @Override
            public void onTimeChanged(TimePicker view, int hourOfDay, int minute) {
                Toast.makeText(TimePickerActivity.this, hourOfDay + ":" + minute,
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

上面程序代码为TimePicker绑定事件监听器的代码，当用户通过这该组件来选择时间时，监听器就会被触发。

### 数值选择器NumberPicker

 NumberPicker 是用于选择一组预定义好数字的组件，用户既可以通过键盘输入数值，也可以通过滚动来选择数值。

NumberPicker的常用方法如下：

- setMinValue(int minVal)：设置该组件支持的最小值。

- setMaxValue(int maxVal)：设置该组件支持的最大值。

- setValue(int value)：设置该组件的当前值。

- getMaxValue()：获得该组件设置的最大值。

- getMinValue()：获得该组件设置的最小值。

- getValue()：获得当前组件显示的值。

- setValue(int value)：设置当前组件显示的值。


使用NumberPicker一共有2个监听器和一个Formatter格式化处理器，

- NumberPicker.OnValueChangeListener ：用于监听当前value的变化。

- NumberPicker.OnScrollListener：用于监听该控件的scroll状态。主要包括以下三种状态：
  - ​    SCROLL_STATE_TOUCH_SCROLL：用户按下去然后滑动。
  - ​    SCROLL_STATE_FLING： 相当于是SCROLL_STATE_TOUCH_SCROLL的后续滑动操作。
  - ​    SCROLL_STATE_IDLE： NumberPicker不在滚动。
- NumberPicker.Formatter： 用于格式化显示该组件中的value，如0—23格式化为00 — 23。

创建numberpicker_layout.xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >
 
    <NumberPicker
        android:id="@+id/numberPicker"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"/>
</RelativeLayout>
```

新建NumberPickerActivity.java文件，加载上面新建的布局文件，初始化NumberPicker并获取用户的选择，具体代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.NumberPicker;
import android.widget.Toast;

public class NumberPickerActivity extends AppCompatActivity {
    private NumberPicker mNumberPicker = null; // 数值选择器
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.numberpicker_layout);
 
        // 获取NumberPicker组件
        mNumberPicker = (NumberPicker) findViewById(R.id.numberPicker);
        // 设置NumberPicker属性
        mNumberPicker.setMinValue(1);
        mNumberPicker.setMaxValue(20);
        mNumberPicker.setValue(5);
 
        // 监听数值改变事件
        mNumberPicker.setOnValueChangedListener(new NumberPicker.OnValueChangeListener() {
            @Override
            public void onValueChange(NumberPicker picker, int oldVal, int newVal) {
                Toast.makeText(NumberPickerActivity.this, "选择的是：" + newVal,
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

上面程序代码为NumberPicker绑定事件监听器的代码，当用户通过这该组件来选择时间时，监听器就会被触发。

 除了Android系统定义的DatePicker、TimePicker和NumberPicker，在实际开发中往往不能满足，会经常自定义一些Picker组件，比如城市选择器、性别选择器、图片选择器、颜色选择器等。

一般可以借用第三方组件来快速完成，当然也可以自定义实现，此处不做过多介绍！



### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)