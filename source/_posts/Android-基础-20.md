---
title: Android  Activity数据传递
date: 2019-05-07 18:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### Activity数据传递

在Android开发中，经常要在Activity之间传递数据。前面也学习了Activity和Intent相关基础，接下来一起来学习Activity的数据传递。

<!--more-->

通过前面的学习知道，Intent可以用来开启Activity，同样它也可以用来在Activity之间传递数据。Intent提供了多个重载的方法来携带额外的数据，如下所示。

- putExtra(String name,  xxx value)：向 Intent 中按 key-value 对的形式存入数据。

- getXxxExtra(String name)：从Intent中按key取出指定类型的数据。 

- putExtras(Bundle data)：向Intent中放入需要携带的数据包。

- Bundle getExtras()：取出Intent中所携带的数据包。

使用Intent传递数据只需调用putExtra()方法将想要存储的数据存在Intent中即可。当启动了另一个Activity后，再把这些数据从Intent中取出即可。其核心示例代码如下：

```
// 从MainActivity传递数据到SecondActivity
 
Intent intent=new Intent(MainActivity.this,SecondActivity.class);
String name="admin ";
intent.putExtra("extra_data_name",name);
startActivity(intent);

// 取出MainActivity传递过来的数据 
Intent intent=getIntent();
String name=intent.getStringExtra("extra_data_name");
```

还有另外一种方式，就是传递Bundle对象。Bundle对象包含了多个方法来存入数据和取出数据，如下所示。

- putXxx(String key , Xxx data)：向 Bundle 中放入 int、long 等各种类型的数据。

- putSerializable(String key，Serializable data)：向 Bundle 中放入一个可序列化的对象。

- getXxx(String key)：从Bundle中取出int、long等各种类型的数据。

- getSerializable(String key, Serializable data)：从 Bundle 中取出一个可序列化的对象。


使用Bundle对象传递数据的核心代码如下：

```
// 从MainActivity传递数据到SecondActivity
Bundle bundle=new Bundle();
bundle.putString("name","Linda ");
bundle.putInt("age",20); 
Intent intent=new Intent(MainActivity.this,SecondActivity.class);
intent.putExtras(bundle); 
startActivity(intent);

// 取出MainActivity传递过来的数据
Intent intent=getIntent();
Bundle bundle=intent.getExtras();
String stuName=bundle.getString("name");
int stuAge=bundle.getString("age");
```

在上述代码中，在接收Bundle对象封装的数据时，需要先创建对应的Bundle对象，然后再根据存入的key值取出value。其实用Intent传递数据以及对象时，它的内部也是调用了Bundle对象相应的put()方法，也就是说Intent内部也是用Bundle来实现数据传递的，只是封装了一层而已。 

#### 示例

接下来通过一个示例来学习两个Activity之间如何通过Bundle交换数据。

创建一个示例程序，非常简单，一共有两个界面，其中第一个界面有用户名、密码和性别等信息，然后有一个注册按钮，第二个界面包含多个文本框。让用户将信息填写完整后点击注册，将所有信息传入到第二个页面去模拟注册，这里就简单显示出来即可。

第一个Activity对应的布局文件（activity_main）的代码如下所示

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    <LinearLayout
        android:id="@+id/regist_username_ll"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="22dp"
        android:orientation="horizontal" >
        <TextView
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:gravity="right"
            android:paddingRight="5dp"
            android:text="用户名 :" />
        <EditText
            android:id="@+id/name_et"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="请输入您的用户名"
            android:textSize="14dp" />
    </LinearLayout>
    <LinearLayout
        android:id="@+id/regist_password_ll"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/regist_username_ll"
        android:layout_centerHorizontal="true"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="5dp"
        android:orientation="horizontal" >
        <TextView
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:gravity="right"
            android:paddingRight="5dp"
            android:text="密    码 :" />
        <EditText
            android:id="@+id/password_et"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="请输入您的密码"
            android:inputType="textPassword"
            android:textSize="14dp" />
    </LinearLayout>
    <RadioGroup
        android:id="@+id/sex_rg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/regist_password_ll"
        android:layout_marginLeft="30dp"
        android:contentDescription="性别"
        android:orientation="horizontal" >
        <RadioButton
            android:id="@+id/male_rb"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:checked="true"
            android:text="男" >
        </RadioButton>
        <RadioButton
            android:id="@+id/female_rb"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="女" />
    </RadioGroup>
 
    <Button
        android:id="@+id/register_btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/sex_rg"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="24dp"
        android:text="注册" />
 
</RelativeLayout>
```

在上述代码中，定义了一个相对布局RelativeLayout，该布局中创建了一个EditText和一个Button按钮，分别用于输入内容和单击“注册”按钮进行数据传递。 

接下来创建一个用于数据接收的界面activity_second.xml，该界面的布局比较简单，只添加了三个TextView用来展示用户信息，因此不展示界面效果。activity_second.xml界面代码如下所示：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
 
    <TextView
        android:id="@+id/name_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:layout_marginTop="10dp"
        android:textSize="20dp" />
    <TextView
        android:id="@+id/password_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:layout_marginTop="10dp"
        android:textSize="20dp" />
    <TextView
        android:id="@+id/sex_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:layout_marginTop="10dp"
        android:textSize="20dp" />
 
</LinearLayout>
```

当界面创建好之后，需要在MainActivity中编写与页面交互的代码，用于实现数据传递具体代码如下所示：

```
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioButton;
 
public class MainActivity extends AppCompatActivity {
    private Button mRegisterBtn = null;
    private EditText mNameEt = null;
    private RadioButton mMaleRb = null;
    private RadioButton mFemaleRb = null;
    private EditText mPasswordEt = null;
 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        mNameEt = (EditText) findViewById(R.id.name_et);
        mPasswordEt = (EditText) findViewById(R.id.password_et);
        mRegisterBtn = (Button) findViewById(R.id.register_btn);
        mMaleRb = (RadioButton) findViewById(R.id.male_rb);
        mFemaleRb = (RadioButton) findViewById(R.id.female_rb);
 
        //点击发送按钮进行数据传递
        mRegisterBtn.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                register();
            }
        });
    }
 
    /**
     * 注册
     */
    public void register() {
        //创建Intent对象，启动SecondActivity
        Intent intent = new Intent(this, SecondActivity.class);
        //将数据存入Intent对象
        intent.putExtra("name", mNameEt.getText().toString().trim());
        intent.putExtra("password", mPasswordEt.getText().toString().trim());
        String str = "";
        if(mMaleRb.isChecked()){
            str = "男";
        }else if(mFemaleRb.isChecked()){
            str = "女";
        }
        intent.putExtra("sex", str);
        startActivity(intent);
    }
}
```

在上述代码中，register()方法实现了获取用户输入数据，并且将Intent作为载体进行数据传递。接下来再创建一个SecondActivity，用于接收数据并展示，具体代码如下所示：

```
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;
 
public class SecondActivity extends AppCompatActivity {
    private TextView mNameTv = null;
    private TextView mPasswordTv = null;
    private TextView mSexTv = null;
 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
 
        Intent intent=getIntent();
        String name = intent.getStringExtra("name");
        String password = intent.getStringExtra("password");
        String sex = intent.getStringExtra("sex");
 
        mNameTv =(TextView) findViewById(R.id.name_tv);
        mPasswordTv = (TextView) findViewById(R.id.password_tv);
        mSexTv = (TextView) findViewById(R.id.sex_tv);
        
        mNameTv.setText("用户名：" + name);
        mPasswordTv.setText("密    码：" + password);
        mSexTv.setText("性    别：" + sex);
    }
}
```

  在上述代码中，通过getIntent()方法获取到Intent对象，然后通过该对象的getStringExtra()方法获取输人的用户名，并将得到的用户名绑定在TextView控件中进行显示。需要注意的是，getStringExtra(String str)方法传人的参数必须是MainActivity中intent.putExtra()方法中传人的key，否则会返回null。



### Activity数据回传

前面己经提到，Activity 还提供了一个 startActivityForResult(Intent intent, int requestCode) 方法来启动其他Activity。该方法用于启动指定Activity，而且期望获取指定Activity返回的结果。这种请求对于实际应用也是很常见的，例如应用程序第一个界面需要用户进行选择——但需要选择的列表数据比较复杂，必须启动另一个Activity让用户选择。当用户在第二个Activity 中选择完成后，程序返回第一个Activity，第一个Activity必须能获取并显示用户在第二个 Activity中选择的结果。在这种应用场景下，也是通过Bundle进行数据交换的。

为了获取被启动的Activity所返回的结果，需要从以下三方面着手。

- 使用startActivityForResult(Intent intent, int requestCode) 方法启动指定Activity。

- 当前 Activity 需要重写 onActivityResult(int requestCode，int resultCode，Intent intent), 当被启动的Activity返回结果时，该方法将会被触发，其中requestCode代表请求码， 而resultCode代表Activity返回的结果码，这个结果码也是由开发者根据业务自行设定的。

- 被启动的Activity需要调用setResult()方法设置处理结果

关于启动Activity并回传数据的核心代码如下所示：

```
// 启动SecondActivity
Intent intent=new Intent(MainActivity.this, SecondActivity.class);
startActivityForResult(intent,1);
 
 
// 从SecondActivity返回数据给MainActivity
Intent intent=new Intent();
intent.putExtra("extra_data_name","admin");
setResult(1,intent);
SecondActivity.this.finish();
```

上述本例代码中，startActivityForResult()方法接收两个参数，第一个参数是Intent，第二个参数是请求码，用于在判断数据的来源。setResult()方法也接收两个参数，第一个参数resultCode结果码，第二个参数则是把带有数据的Intent传递回去。

然后是在启动SecondActivity的MainActivity中重写onActivityResult()方法，实现获取返回的数据，核心代码如下：

```
//  处理SecondActivity返回来的数据
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
  super.onActivityResult(requestCode, resultCode, data);
  if(1==resultCode){
    String name=data.getStringExtra("extra_data_name");
  }
}
```

上述代码中，onActivityResult()方法有三个参数，第一个参数requestCode表示在启动Activity时传递的请求码；第二个参数resultCode表示在返回数据时传入结果码；第三个参数data表示携带返回数据的Intent。

需要注意的是，在一个Activity中很可能调用startActivityForResult()方法启动多个 Activity，每一个Activity返回的数据都会回调到onActivityResult()这个方法中，因此，首先要做的就是通过检查requestCode的值来判断数据来源，确定数据是从SecondActivity返回的，然后再通过resultCode的值来判断数据处理结果是否成功，最后从data中取出数据，这样就完成了 Activity中数据返回的功能。

#### 示例

接下来通过一个示例来学习Activity如何通过Bundle回传数据。

创建一个示例程序，也非常简单，模拟游戏里面的装备升级，主要包括两个界面，其中第一个界面为角色当前装备的实力，可以通过按钮跳转到商店页面购买所需的装备，然后返回升级原来的装备等级。

程序对应的主布局文件（activity_main.xml )如下所示：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"  >
    <ImageView
        android:id="@+id/pet_img"
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginBottom="5dp"
        android:src="@drawable/pet" />
    <TextView
        android:id="@+id/pet_dialog_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginBottom="25dp"
        android:gravity="center"
        android:text="主人，快给小宝宝购买装备吧" />
    <TableLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_marginBottom="20dp"
        android:padding="20dp">
        <TableRow
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:padding="10dp">
            <TextView
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="生命值:"
                android:textColor="@android:color/black"
                android:textSize="14sp" />
            <ProgressBar
                android:id="@+id/life_pb"
                style="?android:attr/progressBarStyleHorizontal"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:layout_weight="3" />
 
            <TextView
                android:id="@+id/life_progress_tv"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="0"
                android:gravity="center"
                android:textColor="#000000" />
        </TableRow>
        <TableRow
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:padding="10dp">
 
            <TextView
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="攻击力:"
                android:textColor="@android:color/black"
                android:textSize="14sp" />
 
            <ProgressBar
                android:id="@+id/attack_pb"
                style="?android:attr/progressBarStyleHorizontal"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="3" />
 
            <TextView
                android:id="@+id/attack_progress_tv"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="0"
                android:gravity="center"
                android:textColor="#000000" />
        </TableRow>
        <TableRow
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:padding="10dp">
 
            <TextView
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="敏捷:"
                android:textColor="@android:color/black"
                android:textSize="14sp" />
 
            <ProgressBar
                android:id="@+id/speed_pb"
                style="?android:attr/progressBarStyleHorizontal"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="3" />
 
            <TextView
                android:id="@+id/speed_progress_tv"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="0"
                android:gravity="center"
                android:textColor="#000000" />
        </TableRow>
    </TableLayout>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal">
        <Button
            android:id="@+id/buy_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentTop="true"
            android:drawableRight="@android:drawable/ic_menu_add"
            android:drawablePadding="3dp"
            android:text="主人购买装备"
            android:textSize="14sp" />
    </RelativeLayout>
</LinearLayout>
```

上述布局代码使用到了控件ProgressBar （进度条），它是用来显示小宝宝的生命值，攻击力和敏捷度的。 

 创建装备选择界面activity_shop.xml，该界面是来选择装备的，代码如下所示。

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/buy_rl"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
 
    <View
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:layout_alignParentLeft="true"
        android:layout_centerVertical="true"
        android:background="@android:drawable/ic_menu_info_details" />
 
    <TextView
        android:id="@+id/name_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_marginLeft="60dp"
        android:text="商品名称" />
 
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:orientation="vertical">
 
        <TextView
            android:id="@+id/life_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="生命值"
            android:textSize="13sp" />
 
        <TextView
            android:id="@+id/attack_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="攻击力"
            android:textSize="13sp" />
 
        <TextView
            android:id="@+id/speed_tv"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="速度"
            android:textSize="13sp" />
    </LinearLayout>
</RelativeLayout>
```

ShopActivity是用来展示装备信息的，当单击ShopActivity的装备时，会调回MainActivity并将装备信息回传给MainActivity。ShopActivity的具体代码如下所示：

```
 
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;
 
public class ShopActivity extends AppCompatActivity implements View.OnClickListener {
    public static final String DATA_KEY_NAME = "name";
    public static final String DATA_KEY_LIFE = "life";
    public static final String DATA_KEY_SPEED = "speed";
    public static final String DATA_KEY_ATTACK = "attack";
 
    private static final String NAME = "小宝";
    private static final int LIFE_VALUE = 20;
    private static final int SPEED_VALUE = 30;
    private static final int ATTACK_VALUE = 10;
 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_shop);
 
        TextView mLifeTV = (TextView) findViewById(R.id.life_tv);
        TextView mNameTV = (TextView) findViewById(R.id.name_tv);
        TextView mSpeedTV = (TextView) findViewById(R.id.speed_tv);
        TextView mAttackTV = (TextView) findViewById(R.id.attack_tv);
 
        mNameTV.setText(NAME);
        mLifeTV.setText("生命值+" + LIFE_VALUE);
        mSpeedTV.setText("敏捷度+" + SPEED_VALUE);
        mAttackTV.setText("攻击力+" + ATTACK_VALUE);
 
        findViewById(R.id.buy_rl).setOnClickListener(this);
    }
 
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.buy_rl:
                Intent intent = new Intent();
 
                Bundle data = new Bundle();
                data.putString(DATA_KEY_NAME, NAME);
                data.putInt(DATA_KEY_LIFE, LIFE_VALUE);
                data.putInt(DATA_KEY_SPEED, SPEED_VALUE);
                data.putInt(DATA_KEY_ATTACK, ATTACK_VALUE);
                intent.putExtras(data);
 
                setResult(1, intent);
                finish();
                break;
        }
    }
}
```

从上述代码中可以看出，setReult()方法的作用是让当前Activity返回到它的调用者，在这里可以理解为让ShopActivity返回到MainActivity。

接下来编写MainActivity。MainActivity主要用于响应按钮的点击事件，并将返回的装备信息显示到指定的控件中，具体代码如下所示：

```
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.ProgressBar;
import android.widget.TextView;
 
public class MainActivity extends AppCompatActivity {
    private Button mBuyBtn = null;
    private ProgressBar mLifePb = null;
    private ProgressBar mAttackPb = null;
    private ProgressBar mSpeedPb = null;
    private TextView mLifeTV = null;
    private TextView mAttackTV = null;
    private TextView mSpeedTV = null;
 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        mBuyBtn = (Button) findViewById(R.id.buy_btn);
        mLifeTV = (TextView) findViewById(R.id.life_progress_tv);
        mAttackTV = (TextView) findViewById(R.id.attack_progress_tv);
        mSpeedTV = (TextView) findViewById(R.id.speed_progress_tv);
        mLifePb = (ProgressBar) findViewById(R.id.life_pb);
        mAttackPb = (ProgressBar) findViewById(R.id.attack_pb);
        mSpeedPb = (ProgressBar) findViewById(R.id.speed_pb);
 
        mLifePb.setMax(100);
        mAttackPb.setMax(100);
        mSpeedPb.setMax(100);
 
        mBuyBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(MainActivity.this, ShopActivity.class);
                startActivityForResult(intent, 1); // 返回请求结果,请求码为1
            }
        });
    }
 
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (data != null) {
            if (resultCode == 1) {
                if (requestCode == 1) {
                    Bundle equipment = data.getExtras();
                    //更新ProgressBar的值
                    updateProgress(equipment);
                }
            }
        }
    }
 
    //更新ProgressBar的值
    private void updateProgress(Bundle info) {
        int lifeProgress = mLifePb.getProgress();
        int attackProgress = mAttackPb.getProgress();
        int speedProgress = mSpeedPb.getProgress();
 
        mLifePb.setProgress(lifeProgress + info.getInt(ShopActivity.DATA_KEY_LIFE));
        mAttackPb.setProgress(attackProgress + info.getInt(ShopActivity.DATA_KEY_ATTACK));
        mSpeedPb.setProgress(speedProgress + info.getInt(ShopActivity.DATA_KEY_SPEED));
        mLifeTV.setText(mLifePb.getProgress() + "");
        mAttackTV.setText(mAttackPb.getProgress() + "");
        mSpeedTV.setText(mSpeedPb.getProgress() + "");
    }
}
```



### Activity间数据传递方法汇总

#### 常用数据类型

 在前面几节我们只学习了一些常用类型的数据传递，主要是以下这些重载方法：

- putExtra(String name, boolean value)

- putExtra(String name, byte value)

- putExtra(String name, char value)

- putExtra(String name, short value)

- putExtra(String name, int value)

- putExtra(String name, long value)

- putExtra(String name, float value)

- putExtra(String name, double value)

- putExtra(String name, String value)

- putExtra(String name, CharSequence value)

- putExtras(Intent src)

- putExtras(Bundle extras)

- putExtra(String name, Bundle value)

- getBooleanExtra(String name, boolean defaultValue)

- getByteExtra(String name, byte defaultValue)

- getCharExtra(String name, char defaultValue)

- getShortExtra(String name, short defaultValue)

- getIntExtra(String name, int defaultValue)

- getLongExtra(String name, long defaultValue)

- getFloatExtra(String name, float defaultValue)

- getDoubleExtra(String name, double defaultValue)

- getStringExtra(String name)

- getCharSequenceExtra(String name)

- getExtras()

- getBundleExtra(String name)


可以发现主要包括boolean、byte、char、short、int、long、float、double、String、CharSequence几个，当然也可以先将数据打包为Bundle或Intent对象再传递

#### 数组、列表类型数据

 然而在实际开发中经常会遇见以上常用类型的数组或列表的组合型数据，其实也非常简单。
1、数组

认真的同学可能已经发现了，每一个基本数据类型都有对应数组数据的重载方法，分别如下：

- putExtra(String name, boolean[] value)

- putExtra(String name, byte[] value)

- putExtra(String name, short[] value)

- putExtra(String name, char[] value)

- putExtra(String name, int[] value)

- putExtra(String name, long[] value)

- putExtra(String name, float[] value)

- putExtra(String name, double[] value)

- putExtra(String name, String[] value)

- putExtra(String name, CharSequence[] value)

- getBooleanArrayExtra(String name)

- getByteArrayExtra(String name)

- getShortArrayExtra(String name)

- getCharArrayExtra(String name)

- getIntArrayExtra(String name)

- getLongArrayExtra(String name)

- getFloatArrayExtra(String name)

- getDoubleArrayExtra(String name)

- getStringArrayExtra(String name)

- getCharSequenceArrayExtra(String name)

   putExtra()方法的参数简单替换为数组类型的即可，然后使用数组的专用方法获取，使用起来也非常简单。

2、列表 

在传递列表型数据的时候稍微有一些不同了，Intent还提供了以下这几个重载方法：

- putIntegerArrayListExtra(String name, ArrayList<Integer\> value)

- putStringArrayListExtra(String name, ArrayList<String\> value)

- putCharSequenceArrayListExtra(String name, ArrayList<CharSequence\> value)

- getIntegerArrayListExtra(String name)

- getStringArrayListExtra(String name)

- getCharSequenceArrayListExtra(String name)

从以上几个方法可以知道，Intent自带传递Integer、String、CharSequence三种类型的列表数据，如果需要传递到额数据是这几种类型，或能够转换为这几种类型，那么数据的传递也变得很顺利了。

#### 对象

 前面学习的几个方法使用起来还是比较简单的，但是会发现一个问题，以上学习的方法无法传输对象（如图片）、对象的数组或集合，那就需要用到以下这些方法了。

intent还有以下这些重载方法：

- putExtra(String name, Serializable value)

- putExtra(String name, Parcelable value)

- putExtra(String name, Parcelable[] value)

- putParcelableArrayListExtra(String name, ArrayList<? extends Parcelable> value)

- getSerializableExtra(String name)

- getParcelableExtra(String name)

- getParcelableArrayExtra(String name)

- getParcelableArrayListExtra(String name)


可能你已经发现了，这里提到的Serializable类型和Parcelable类型数据到底是什么呢？接下来分别来学习。

**序列化对象Serializable**

Serializable接口是启用其序列化功能的接口，实现java.io.Serializable 接口的类是可序列化的，没有实现此接口的类将不能使它们的任一状态被序列化或逆序列化（如果不懂序列化，建议复习巩固Java部分的序列化知识模块）。

Serializable实现序列化的方法也很简单，将需要序列化的类实现Serializable接口，Serializable接口中没有任何方法，只需在类中指定serialVersionUID的值，该值可以任意指定一个值。可以理解为一个标记，即表明这个类可以序列化。

假如需要使用Intent传递一个Person对象，就先要将其序列化，如下示例代码：

```
import java.io.Serializable;

public class Person implements Serializable {
    private static final long serialVersionUID = 1L; // 序列化ID
 
    private String name; // 姓名
    private int age; // 年龄
 
    public Person() {
        this.name = "未知";
        this.age = 18;
    }
 
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
}
```

然后即可调用前面的put和get方法来传递复杂对象数据了

**序列化对象Parcelable** 

由于Serializable在序列化的时候会产生大量的临时变量，从而引起频繁的GC，会影响持续性能。在使用内存的时候，Parcelable比Serializable性能高，所以推荐使用Parcelable。

实现Parcelable接口稍微复杂一些，但效率更高，推荐用这种方法提高性能。实现步骤如下：

将需要序列化的类实现Parcelable接口。

重写writeToParcel方法，将对象序列化为一个Parcel对象。

重写describeContents方法，描述内容接口，默认返回0。实例化静态内部对象CREATOR实现接口Parcelable.Creator。其中public static final一个都不能少，内部对象CREATOR的名称也不能改变，必须全部大写。需重写本接口中的两个方法：

createFromParcel(Parcel in) 实现从Parcel容器中读取传递数据值，封装成Parcelable对象返回逻辑层。

newArray(int size) 创建一个类型为T，长度为size的数组，仅一句话即可（return new T[size]），供外部类反序列化本类数组使用。

接下来将上面的Person类进行改造，代码如下：

```
import android.os.Parcel;

public class Person implements Parcelable {
    private String name; // 姓名
    private int age; // 年
 
    protected Person(Parcel in) {
        // 在读取Parcel容器里的数据时，必须按成员变量声明的顺序读取数据，不然会出现获取数据出错
        name = in.readString();
        age = in.readInt();
    }
 
    public static final Creator<Person> CREATOR = new Creator<Person>() {
        // 再通过createFromParcel将Parcel对象映射成原对象
        @Override
        public Person createFromParcel(Parcel in) {
            return new Person(in);
        }
 
        // 供外部类反序列化本类数组使用
        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };
 
    // 内容接口描述，默认返回0即可
    @Override
    public int describeContents() {
        return 0;
    }
 
    // 按照声明顺序打包数据到Parcel对象中，既将数据打包到Parcel容器中
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(age);
    }
 
    public Person() {
        this.name = "未知";
        this.age = 18;
    }
 
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
}
```

然后即可调用前面的put和get方法来传递复杂对象数据了，当然也可以是对象的数组或列表型数据。

在使用中需要注意的是，Parcelable不能使用在要将数据存储在磁盘上的情况，因为Parcelable不能很好的保证数据的持续性在外界有变化的情况下。尽管Serializable效率低点，但此时还是建议使用Serializable。

### 全局Application

如果需要将一个对象在多个Activity之间传递，或者要连续传递好几层，这种情况下如果使用以上方法就需要重复多次，使用起来就特别别扭，这种情况就可以考虑使用全局Application。

Android系统在每个程序运行的时候都会创建一个Application对象，而且只会创建一个，所以Application 是单例(singleton)模式的一个类，而且Application对象的生命周期是整个程序中最长的，他的生命周期等于这个程序的生命周期。如果想存储一些值，使用 Application就需要自定义类实现Application类，然后在AndroidManifest.xml中使用我们自定义的Application 而非系统默认的。

这里简单使用一个示例来学习，这里简化为全局保存一个状态值，可以方便在各Activity中进行传递。首先自定义Application类，代码如下：

```
 
import android.app.Application;

public class MyApplication extends Application {
    private int state;
 
    public int getState() {
        return state;
    }
 
    public void setState(int state) {
        this.state = state;
    }
}
```

然后在AndroidManifest.xml中声明，为application标签添加android:name属性.

最后在需要使用定义的全局变量的地方即可调用，核心代码如下：

```
 
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
 
public class TestActivity extends AppCompatActivity {
 
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_shop);
 
        // ...
        MyApplication app = (MyApplication) getApplicationContext();
        // 保存数据
        app.setState(1);
        // ...
        // 读取数据
        int state = app.getState();
        // ...
    }
}
```

这样就非常方便的在各Activity之间进行数据传递了。如果想要在整个应用程序中任何位置都能使用，可以对MyApplication类进行适当的改造，这里不做过多说明。

但是需要注意的是，当由于某些原因（比如系统内存不足），我们的app会被系统强制杀死，此时再次点击进入应用时，系统会直接进入被杀死前的那个界面，制造一种从来没有被杀死的假象。那么问题来了，系统强制停止了应用，进程死了，那么再次启动时Application自然新的，那里边的数据自然木有啦，如果直接使用很可能报空指针或者其他错误。

所以在使用时一定要做好非空判断，如果数据为空可以考虑逻辑上让应用直接返回到最初的Activity。

#### 单例模式

 上面的Application就是基于单例的，单例模式的特点就是可以保证系统中一个类有且只有一个实例。

  这里做一个简单的示例，如定义一个数据持有者类，代码如下：

```

public class DataHolder {
    private String data;
 
    public String getData() {
        return data;
    }
 
    public void setData(String data) {
        this.data = data;
    }
 
    private static final DataHolder holder = new DataHolder();
 
    public static DataHolder getInstance() {
        return holder;
    }
}
```

然后在使用的地方即可直接调用，如下所示：

```
// 更新数据
DataHolder.getInstance().setData(data);
// 获取数据
Stringdata=DataHolder.getInstance().getData();
```

 这样使用起来也非常简单

#### 静态变量

这个可以直接在Activity中完成单独一个数据结构，和单例差不多。

```

public class DataHolder {
    private static String data;
 
    public static String getData() {
        return data;
    }
 
    public static void setData(String strData) {
        data = strData;
    }
}
```

这样就可以在启动Activity之前设置数据，新的Activity中获取数据。

```
// 更新数据

DataHolder.setData(data);

// 获取数据

Stringdata=DataHolder.getData();

```

需要注意的是，如果数据很大很多（如bitmap），处理不当是很容易导致内存泄露或者内存溢出的，可以考虑使用WeakReferences 将数据包装起来。

 除了以上介绍的几种方式，还可以使用持久化数据等方法，这里先不做过多介绍，在后续的学习中会陆续接触到。

### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)