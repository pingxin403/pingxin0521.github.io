---
title: Android  Fragment
date: 2019-05-03 20:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### 概述

Fragment 是一种可以嵌人在Activity中的UI片段，它能让程序更加合理地利用大屏幕空间，因而Fragment在平板上应用非常广泛。Fragment与Activity十分相似，它包含布局，同时也具有自己的生命周期。

<!--more-->

一个Fragment代表着Activity中一种行为或者Activity用户界面中的一部分。我们可以将多个Fragment组合在一个Activity中，组成一个多窗格布局；同样我们也可以在多个Activity中重复使用某个Fragment。我们可以将Fragment当作一个Activity中的小模块（它有它自己的生命周期，自己的事件处理机制），在Activity运行过程中，我们可以动态地添加或者移除这个模块。

Android 3.0引入Fragment的初衷是为了适应大屏幕的平板电脑，由于平板电脑的屏幕比手机屏幕更大，因此可以容纳更多的UI组件，且这些UI组件之间存在交互关系。Fragment简化了大屏幕UI的设计，它不需要开发者管理组件包含关系的复杂变化，开发者使用Fragment对 UI组件进行分组、模块化管理，就可以更方便地在运行过程中动态更新 Activity的用户界面。

比如说：我们的应用中有一个文章列表和文章详情页面，由于平板设备空间大，列表Fragment和详情Fragment可以放在同一个页面中，而在手持设备上，则分为两个Activity作展示。如下图：

![4.jpeg](https://i.loli.net/2019/05/11/5cd6da1d271fe.jpeg)

如上图所示的新闻浏览界面，该界面需要在屏幕左边显示新闻列表，并在屏幕右边显示新闻内容，此时就可以在Activity中显示两个并排的Fragment：左边的Fragment显示新闻列表，右边的Fragment显示新闻内容。由于每个Fragment都拥有自己的生命周期，并可响应用户输入事件，因此可以非常方便地实现：当用户单击左边列表中的指定新闻时，右边的Fragment就会显示相应的新闻内容。上图所示左边的“平板电脑”部分显示了这种UI界面。

通过使用上面的Fragment设计机制，可以取代传统的让一个Activity显示新闻列表，另—个Activity显示新闻内容的设计。由于Fragment是可复用的组件，因此如果需要在正常尺寸的手机屏幕上运行该应用，则可以改为使用两个 Activity: Activity A 包含 Fragment A、Activity B 包含 FragmentB。其中 ActivityA仅包含显示文章列表的Fragment A，而当用户选择一篇文章时，它会启动包含新闻内容的Activity B，如上图所示右边的手机部分。

### 优势

从上面的例子知道，Fragment在开发中非常重要。概括起来，使用Fragment有以下一些好处：

- Fragment可以将Activity分离成多个可重用的组件，每个都有它自己的生命周期和UI。

- Fragment可以轻松创建动态灵活的UI设计，可以适应于不同的屏幕尺寸。

- Fragment是一个独立的模块，紧紧地与Activity绑定在一起。可以运行中动态地移除、加入、交换等。

- Fragment提供一个新的方式让我们在不同的安卓设备上统一UI。

- Fragment 可以解决Activity间的切换不流畅，轻量切换问题。

- Fragment 可以替代TabActivity做导航，性能更好。

- Fragment 在4.2.版本中新增了嵌套Fragmeng的使用方法，能够生成更好的界面效果。

- Fragment做局部内容更新更方便，原来为了达到这一点要把多个布局放到一个Activity里面，现在可以用多Fragment来代替，只有在需要的时候才加载Fragment，大大提高了性能

### 使用

 与创建Activity类似，要创建一个Fragment必须创建一个类继承自Fragment。很多时候我们都是直接重写Fragment，其实Fragment类是一个基类，还有几个派生子类，

  ● DialogFragment

显示一个浮动的对话框。使用这个类创建对话框是替代Activity创建对话框的最佳选择。因为可以把fragmentdialog放入到Activity的返回栈中，使用户能再返回到这个对话框。

  ● ListFragment

显示一个列表控件，就像ListActivity类，它提供了很多管理列表的方法，比如onListItemClick()方法响应click事件。

  ● PreferenceFragment

显示一个由Preference对象组成的列表，与PreferenceActivity相同。它用于为程序创建“设置”Activity。

  ● WebViewFragment

WebViewFragment封装了WebView，随着WebViewFragment的暂停或恢复，WebView也进入暂停或恢复状态。

  创建时可以根据需要使用Fragment基类或它的任意子类。为了控制Fragment显示的组件，通常需要重写onCreateView()方法，该方法返回的View 将作为该Fragment显示的View组件，当Fragment绘制界面组件时将会回调该方法。

需要注意的是，Android系统提供了两个Fragment类，分别是android.app.Fragment和 android.support.v4.app.Fragment。其中继承 android.app.Fragment 类则程序只能兼容 Android 4.0 以上的系统，继承android.support.v4.app.Fragment类可以兼容低版本的Android系统。

接下来通过一段示例代码来演示如何创建Fragment，首先创建一个布局文件，里面只有一个文本框，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:background="#1fc186"
              android:orientation="vertical">
 
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是我的第一个Fragment"
        android:textColor="#0b0faf"
        android:textSize="18sp"/>
 
</LinearLayout>
```

接下来创建我们的第一个Fragment，并让其继承Fragment，代码如下：

```

import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
 
public class FirstFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_first, container, false);
        return view;
    }
}
```

上述代码重写了 Fragment的onCreateView()方法，并在该方法中调用了 Layoutlnflater的 inflate()方法加载了布局文件，并返回该布局文件对应的View组件。

Fragment创建完成后并不能单独使用，还需要将Fragment加载到Activity中

#### 静态加载

静态加载Fragment非常简单，直接把Fragment当成普通的控件写在Activity的布局文件中。使用<fragment \></fragment\>标签，该标签与其他控件的标签类似，但必须要指定android:name属性或class属性，其属性值为Fragment的全路径名称。

接下来在上一期的案例基础上继续学习，首先修改activity_main.xml文件，修改后的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <fragment
        android:id="@+id/fragment_one"
        android:name="com.jinyu.cqkxzsxy.android.fragmentsample.FirstFragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

当然，也可以这样使用：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <fragment
        android:id="@+id/fragment_one"
        class="com.jinyu.cqkxzsxy.android.fragmentsample.FirstFragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

上面两种方法选择其中一种即可，然后在Activity在onCreate( )方法中调用setContentView()加载布局文件即可。默认已经加载activity_main布局，这里就不做修改。

#### 动态加载

已经学会了在布局文件中添加Fragment的方法，非常简单，但是有一个缺点，那就是一旦添加就不能在运行时将其删除。所以我们还可以通过动态加载的方式来完成，这种方式是在运行时添加，相对来说比较灵活。

首先Activity需要有一个ViewGroup容器存放Fragment，一般使用FrameLayout。

继续在Activity对应的activity_main布局文件中加入FrameLayout，再次修改后的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

 然后在Activity中，通过代码将Fragment添加进Activity中。动态添加Fragment主要分为4步：

1. 获取到FragmentManager对象，在V4包中通过getSupportFragmentManager方法获取，在系统中原生的Fragment是通过getFragmentManager获得的。

2. 开启一个事务，通过调用beginTransaction方法开启。

3. 向容器内加入Fragment，一般使用add或者replace方法实现，需要传入容器的id和Fragment的实例。

4. 提交事务，调用commit方法提交。


接下来修改MainActivity的代码，如下所示：

```
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
 
public class MainActivity extends AppCompatActivity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        // 1. 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 2. 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        // 3. 向容器内加入Fragment
        FirstFragment  firstFragment = new FirstFragment ();
        fragmentTransaction.add(R.id.fragment_container, firstFragment);
        // 4. 提交事务
        fragmentTransaction.commit();
    }
}
```

 以上几步是不是也是非常简单的，如果对FragmentManager和FragmentTransaction对象不理解没关系，这里先记住这4步即可，后续会专门来学习这2个类的使用。

这样就完成了在Activity中动态加载Fragment的功能，重新运行程序，可以看到和上面静态添加Fragment相同的界面。

### Fragment生命周期

一个Activity可以同时组合多个Fragment，一个Fragment也可被多个Activity 复用。Fragment可以响应自己的输入事件，并拥有自己的生命周期，但它们的生命周期直接被其所属的Activity的生命周期控制。

![5.jpeg](https://i.loli.net/2019/05/11/5cd6df6f460ee.jpeg)

#### Fragment状态

  与Activity类似的是，Fragment也存在如下4种状态：

- 运行状态：当前Fmgment位于前台，用户可见，可以获得焦点。

- 暂停状态：其他Activity位于前台，该Fragment依然可见，只是不能获得焦点。

- 停止状态：该Fragment不可见，失去焦点。

- 销毁状态：该Fragment被完全删除，或该Fragment所在的Activity被结束。


结合之前学习Activity的状态，理解Fragment的状态非常简单。

很多地方都在说明 Fragment有三个状态，包括官方文档没有提到Fragment的 销毁状态。这也是合理的，因为处于销毁状态的Fragment基本不可用了，只能等着被回收。

#### Fragment生命周期

Fragment的生命周期与Activity的生命周期十分相似，如下图所示：

![6.jpeg](https://i.loli.net/2019/05/11/5cd6e1be92136.jpeg)

 从上图可以看出，Activity中的生命周期方法，Fragment中基本都有，但是Fragment比Activity多几个方法。各生命周期方法的含义如下：

- onAttach()：当该Fragment被添加到Activity时被回调。该方法只会被调用一次。

- onCreate(Bundle savedStatus)：创建Fragment时被回调。该方法只会被调用一次。

- onCreateView()：每次创建、绘制该Fragment的View组件时回调该方法，Fragment将会显示该方法返回的View 组件。

- onActivityCreated()：当 Fragment 所在的Activity被启动完成后回调该方法。

- onStart()：启动 Fragment 时被回调。

- onResume():恢复 Fragment 时被回调，在onStart()方法后一定会回调 onResume()方法。

- onPause()：暂停 Fragment 时被回调。

- onStop()：停止 Fragment 时被回调。

- onDestroyView()：销毁该 Fragment 所包含的View组件时调用。

- onDestroy()：销毁 Fragment 时被回调。 该方法只会被调用一次。

- onDetach()：将该 Fragment 从Activity中删除、替换完成时回调该方法，在onDestroy()方法后一定会回调 onDetach()方法。该方法只会被调用一次。

 正如开发Activity时可以根据需要有选择性地覆盖指定方法一样，开发Fragment时也可根据需要有选择性地覆盖指定方法。其中最常见的就是覆盖onCreateView()方法——该方法返回的View将由Fragment显示出来。

#### 示例

首先创建一个布局文件fragment_lifecycle.xml，在其中简单添加一个文本框，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="Fragment生命周期"
        android:textColor="#1418e6"
        android:textSize="18sp"/>
 
</LinearLayout>
```

新建一个LifeCycleFragment类，继承Fragment基类，并重写其全部生命周期方法，并在每一个生命周期方法里面将对应的方法名输出到Logcat，代码如下：

```
import android.app.Fragment;
import android.content.Context;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
 
public class LifeCycleFragment extends Fragment {
    private static final String TAG = "LifeCycleFragment";
 
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        Log.d(TAG, "onAttach");
    }
 
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG, "onCreate");
    }
 
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             Bundle savedInstanceState) {
        Log.d(TAG, "onCreateView");
 
        View view = inflater.inflate(R.layout.fragment_lifecycle, container, false);
        return view;
    }
 
    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        Log.d(TAG, "onActivityCreated");
    }
 
    @Override
    public void onStart() {
        super.onStart();
        Log.d(TAG, "onStart");
    }
 
    @Override
    public void onResume() {
        super.onResume();
        Log.d(TAG, "onResume");
    }
 
    @Override
    public void onPause() {
        super.onPause();
        Log.d(TAG, "onPause");
    }
 
    @Override
    public void onStop() {
        super.onStop();
        Log.d(TAG, "onStop");
    }
 
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.d(TAG, "onDestroyView");
    }
 
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");
    }
 
    @Override
    public void onDetach() {
        super.onDetach();
        Log.d(TAG, "onDetach");
    }
}
```

修改Activity的布局文件activity_main.xml，这里简单使用静态加载的方式，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">
 
    <fragment
        android:id="@+id/fragment_one"
        android:name="com.jinyu.cqkxzsxy.android.fragmentlifecycle.LifeCycleFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
 
</LinearLayout>
```

修改MainActivity文件，同样也重写其全部生命周期方法，并在每一个生命周期方法里面将对应的方法名输出到Logcat，代码如下：

```
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
 
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG, "onCreate");
 
        setContentView(R.layout.activity_main);
    }
 
    @Override
    public void onStart() {
        super.onStart();
        Log.d(TAG, "onStart");
    }
 
    @Override
    protected void onRestart() {
        super.onRestart();
        Log.d(TAG, "onRestart");
    }
 
    @Override
    public void onResume() {
        super.onResume();
        Log.d(TAG, "onResume");
    }
 
    @Override
    public void onPause() {
        super.onPause();
        Log.d(TAG, "onPause");
    }
 
    @Override
    public void onStop() {
        super.onStop();
        Log.d(TAG, "onStop");
    }
 
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");
    }
}
```

Activity第一次加载Fragment并显示出来获得焦点时，会依次执行onAttach -> onCreate -> 
onCreateView -> onActivityCreated -> onStart ->onResume。

当按下Home键后，会依次执行onPause -> onStop。

当回到程序界面恢复时，会依次执行onStart -> onResume。

当按下Back键，退出程序会依次执行onPause -> onStop -> onDestoryView -> onDestory -> onDetach。

通过上述操作，可以看到Fragment的生命周期方法执行顺序和前面的生命周期图完全吻合。

 同时可以看到，当显示Fragment的时候都先执行Activity方法，当销毁的时候都是先执行 Fragment的方法，这样更好理解Fragment是嵌套在Activity中 

![7.jpeg](https://i.loli.net/2019/05/11/5cd6e2aa1e7a9.jpeg)

至此，Fragment的生命周期体验结束，当和Activity对比起来会发现非常简单，仅仅只是多个几个方法而已。

### Fragment添加、删除、替换

在前面的学习中，特别是动态加载的时候，有提到FragmentManager和FragmentTransaction类，这里先来详细了解一下其到底为何物。

1. FragmentManager 

要管理Activity中的Fragments，就需要使用FragmentManager类。通过getFragmentManager()或getSupportFragmentManager()获得 。

FragmentManager类常用的方法有以下几个：

- findFragmentById(int id)：根据ID来找到对应的Fragment实例，主要用在静态添加Fragment的布局中，因为静态添加的Fragment才会有ID 。

- findFragmentByTag(String tag)：根据TAG找到对应的Fragment实例，主要用于在动态添加的Fragment中，根据TAG来找到Fragment实例 。

- getFragments()：获取所有被add进Activity中的Fragment 实例。

- benginTransatcion()：开启一个事物。

2. FragmentTransaction 

如果需要添加、删除、替换Fragment，则需要借助于FragmentTransaction对象，FragmentTransaction 代表 Activity 对 Fragment 执行的多个改变。

FragmentTransaction类常用的方法有以下几个：

- add(int containerViewId, Fragment fragment, String tag)：将一个Fragment实例添加到Activity的最上层 。

- remove(Fragment fragment)：将一个Fragment实例从Activity的Fragment队列中删除。

- replace(int containerViewId, Fragment fragment)：替换containerViewId中的Fragment实例。注意，它首先把containerViewId中所有Fagment删除，然后再add进去当前的Fragment 实例。

- hide(Fragment fragment)：隐藏当前的Fragment，仅仅是设为不可见，并不会销毁。

- show(Fragment fragment)：显示之前隐藏的Fragment。

- detach(Fragment fragment)：会将view从UI中移除，和remove()不同，此时Fragment的状态依然由FragmentManager维护。

- attach(Fragment fragment)：重建view视图，附加到UI上并显示。

- commit()：提交一个事务。

#### 示例

前面了解了Fragment管理和事物，为了更好的理解和掌握，用一个示例来进一步深度学习。

该示例非常简单，界面包含3个按钮和3个Fragment容器，其中第一个容器静态加载FirstFragment，后面两个通过动态加载的方式来完成，也是本示例主要需要学习的地方。

首先创建第一个容器的FirstFragment对应布局文件fragment_first.xml，布局代码如下

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:background="#af520b"
              android:orientation="vertical">
 
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是我的第一个Fragment"
        android:textColor="#0c1ff1"
        android:textSize="18sp"/>
 
</LinearLayout>
```

接着创建第一个容器的FirstFragment文件，代码如下：

```
import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
 
public class FirstFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_first, container, false);
        return view;
    }
}
```

同理创建fragment_second.xml和fragment_third.xml文件，唯一的区别就是显示的提示文字和颜色不同，创建SecondFragment和ThirdFragment加载对应的布局文件，这里不在给出代码。

然后是修改activity_main.xml文件，修改后的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
 
        <Button
            android:id="@+id/add_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="add" />
 
        <Button
            android:id="@+id/remove_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="remove" />
 
        <Button
            android:id="@+id/replace_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="replace" />
    </LinearLayout>
 
    <fragment
        android:id="@+id/fragment_one"
        android:name="com.cqkxzsxy.jinyu.android.fragmentbasemanager.FirstFragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
    <FrameLayout
        android:id="@+id/fragment_container1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
    <FrameLayout
        android:id="@+id/fragment_container2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
</LinearLayout>
```

最后是修改MainActivity的代码，如下所示：

```
import android.app.Fragment;
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
 
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private Button mAddBtn = null;
    private Button mRemoveBtn = null;
    private Button mReplaceBtn = null;
    private Fragment mSecondFragment = null;
    private Fragment mThirdFragment = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        mAddBtn = (Button) findViewById(R.id.add_btn);
        mRemoveBtn = (Button) findViewById(R.id.remove_btn);
        mReplaceBtn = (Button) findViewById(R.id.replace_btn);
 
        // 创建和获取Fragment实例
        mSecondFragment = new SecondFragment();
        mThirdFragment = new ThirdFragment();
 
        // 设置监听事件
        mAddBtn.setOnClickListener(this);
        mRemoveBtn.setOnClickListener(this);
        mReplaceBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View v) {
        // 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
 
        // Fragment操作
        switch (v.getId()) {
            case R.id.add_btn:
                // 向容器内加入Fragment
                if (!mSecondFragment.isAdded()) {
                    fragmentTransaction.add(R.id.fragment_container1, mSecondFragment);
                }
                if (!mThirdFragment.isAdded()) {
                    fragmentTransaction.add(R.id.fragment_container2, mThirdFragment);
                }
                break;
            case R.id.remove_btn:
                // 从容器类移除Fragment
                fragmentTransaction.remove(mSecondFragment);
                break;
            case R.id.replace_btn:
                if (!mSecondFragment.isAdded()) {
                    fragmentTransaction.replace(R.id.fragment_container2, mSecondFragment);
                }
                break;
            default:
                break;
        }
 
        // 提交事务
        fragmentTransaction.commit();
    }
}
```

主要就是为3个按钮设置监听事件，其中第一个按钮为后2个容器添加Fragment，第二个按钮移除第一个容器的Fragment，第三个按钮将容器2里面的Fragment替换。

#### Fragment显示和隐藏

由于上一节有简单介绍过对应的api，这里直接通过案例来进行学习。

创建一个Fragment对应的布局文件fragment_demo.xml，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:background="#f1d60d"
              android:orientation="vertical">
 
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是我的第一个Fragment"
        android:textColor="#0c1ff1"
        android:textSize="18sp"/>
 
</LinearLayout>
```

紧接着创建一个Fragment文件，命名为DemoFragment，代码非常简单，如下：

```
 
import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
 
public class DemoFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_demo, container, false);
        return view;
    }
}
```

然后就是我们要操作的界面设计了，这里一共包括2个按钮，分别表示隐藏Fragment和显示Fragment，主布局acticity_main文件的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
 
        <Button
            android:id="@+id/show_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="show" />
 
        <Button
            android:id="@+id/hide_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="hide" />
 
    </LinearLayout>
 
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
</LinearLayout>
```

然后就是修改主界面MainActivity的代码，获取按钮并设置监听事件，对应不同的事件做出不同的操作，代码如下：

```
import android.app.Fragment;
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
 
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private Button mHideBtn = null;
    private Button mShowBtn = null;
    private Fragment mDemoFragment = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        mHideBtn = (Button) findViewById(R.id.hide_btn);
        mShowBtn = (Button) findViewById(R.id.show_btn);
 
        // 创建和获取Fragment实例
        mDemoFragment = new DemoFragment();
        // 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        // 添加Fragment
        fragmentTransaction.add(R.id.fragment_container, mDemoFragment);
        // 提交事务
        fragmentTransaction.commit();
 
        // 设置监听事件
        mHideBtn.setOnClickListener(this);
        mShowBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View v) {
        // 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
 
        // Fragment操作
        switch (v.getId()) {
            case R.id.hide_btn:
                if (!mDemoFragment.isHidden()) {
                    fragmentTransaction.hide(mDemoFragment);
                }
                break;
            case R.id.show_btn:
                if (mDemoFragment.isHidden()) {
                    fragmentTransaction.show(mDemoFragment);
                }
                break;
            default:
                break;
        }
 
        // 提交事务
        fragmentTransaction.commit();
    }
}
```

 到这里有的同学就会有疑问了：将Fragment隐藏的时候是否将其销毁了，然后再显示的时候重新新建的？那么接下来通过Logcat来进行验证。

将DemoFragment的生命周期方法补全，并每一个生命周期方法中加一句Logcat代码，然后重新运行程序。可以发现，无论我们是隐藏还是显示Fragment，没有任何生命周期方法被调用。说明hide操作只是将Fragment变得不可见而已，其本身仍然是存在的，在具体使用的时候需要注意。

### Fragment绑定和解绑

这里同样是直接跳过案例来进行学习，新建一个新的module名为fragmentattachdetach，然后创建一个Fragment对应的布局文件fragment_demo.xml，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:background="#f1d60d"
              android:orientation="vertical">
 
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是我的第一个Fragment"
        android:textColor="#0c1ff1"
        android:textSize="18sp"/>
 
</LinearLayout>
```

紧接着创建一个Fragment文件，命名为DemoFragment，代码非常简单，如下：

```
import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
 
public class DemoFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_demo, container, false);
        return view;
    }
}
```

然后就是我们要操作的界面设计了，这里一共包括2个按钮，分别表示绑定Fragment和解绑Fragment，主布局acticity_main文件的代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
 
        <Button
            android:id="@+id/detach_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="detach" />
 
        <Button
            android:id="@+id/attach_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="attach" />
 
    </LinearLayout>
 
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
</LinearLayout>
```

然后就是修改主界面MainActivity的代码，获取按钮并设置监听事件，对应不同的事件做出不同的操作，代码如下：

```
import android.app.Fragment;
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
 
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private Button   mDetachBtn      = null;
    private Button   mAttachBtn      = null;
    private Fragment mDemoFragment = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        mAttachBtn = (Button) findViewById(R.id.attach_btn);
        mDetachBtn = (Button) findViewById(R.id.detach_btn);
 
        // 创建和获取Fragment实例
        mDemoFragment = new DemoFragment();
        // 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        // 添加Fragment
        fragmentTransaction.add(R.id.fragment_container, mDemoFragment);
        // 提交事务
        fragmentTransaction.commit();
 
        // 设置监听事件
        mAttachBtn.setOnClickListener(this);
        mDetachBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View v) {
        // 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
 
        // Fragment操作
        switch (v.getId()) {
            case R.id.attach_btn:
                if (mDemoFragment.isDetached()) {
                    fragmentTransaction.attach(mDemoFragment);
                }
                break;
            case R.id.detach_btn:
                if (!mDemoFragment.isDetached()) {
                    fragmentTransaction.detach(mDemoFragment);
                }
                break;
            default:
                break;
        }
 
        // 提交事务
        fragmentTransaction.commit();
    }
}
```

有的同学又会问了，这里的操作和前面的显示隐藏效果一样，背后的原理是否也一样呢？同样将DemoFragment的生命周期方法补全，并每一个生命周期方法中加一句Logcat代码，然后重新运行程序。

可以发现，当我们detach操作的时候，首先将Fragment从UI中移除，但是仍然保存在FragmentManager对象中进行维护；attach操作就是重建Fragment视图，然后附加到UI并显示出来。

在实际操作中，假如我不小心隐藏或解绑了Fragment，应该如何回到之前的状态呢？

#### Fragment回退栈

当我们完成一些操作后，如果想要回到操作之前的状态，一般我们都会按返回键，然而发现并没有按照我们想要的那样进行，反而退出了程序，那应该怎么得到想要的效果呢？

我们知道Activity有任务栈，用户通过startActivity将Activity加入栈，点击返回按钮将Activity出栈。Fragment也有类似的栈，称为回退栈（Back Stack），回退栈是由FragmentManager管理的

![0IAQuaBCWUi.jpeg](https://i.loli.net/2019/05/11/5cd6e60318ad8.jpeg)

如果没有加入回退栈，则用户点击返回按钮会直接将Activity出栈；如果加入了回退栈，则用户点击返回按钮会回滚Fragment事务。

默认情况下，Fragment事务是不会加入回退栈的，如果想将Fragment加入回退栈并实现事物回滚，首先需要在commit()方法之前调用事务的以下方法将其添加到回退栈中：

- addToBackStack(String tag)：标记本次的回滚操作。


这里在Fragment添加、删除、替换案例的基础来进行学习，布局代码和Fragment代码不变，只需要在MainActivity的onClick方法中增加一行代码即可，代码如下：

```
 
import android.app.Fragment;
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
 
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private Button mAddBtn = null;
    private Button mRemoveBtn = null;
    private Button mReplaceBtn = null;
    private Fragment mSecondFragment = null;
    private Fragment mThirdFragment = null;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        mAddBtn = (Button) findViewById(R.id.add_btn);
        mRemoveBtn = (Button) findViewById(R.id.remove_btn);
        mReplaceBtn = (Button) findViewById(R.id.replace_btn);
 
        // 创建和获取Fragment实例
        mSecondFragment = new SecondFragment();
        mThirdFragment = new ThirdFragment();
 
        // 设置监听事件
        mAddBtn.setOnClickListener(this);
        mRemoveBtn.setOnClickListener(this);
        mReplaceBtn.setOnClickListener(this);
    }
 
    @Override
    public void onClick(View v) {
        // 获取到FragmentManager对象
        FragmentManager fragmentManager = getFragmentManager();
        // 开启一个事务
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
 
        // Fragment操作
        switch (v.getId()) {
            case R.id.add_btn:
                // 向容器内加入Fragment
                if (!mSecondFragment.isAdded()) {
                    fragmentTransaction.add(R.id.fragment_container1, mSecondFragment);
                }
                if (!mThirdFragment.isAdded()) {
                    fragmentTransaction.add(R.id.fragment_container2, mThirdFragment);
                }
                break;
            case R.id.remove_btn:
                // 从容器类移除Fragment
                fragmentTransaction.remove(mSecondFragment);
                break;
            case R.id.replace_btn:
                if (!mSecondFragment.isAdded()) {
                    fragmentTransaction.replace(R.id.fragment_container2, mSecondFragment);
                }
                break;
            default:
                break;
        }
 
        // 加入回退栈
        fragmentTransaction.addToBackStack(null);
        // 提交事务
        fragmentTransaction.commit();
    }
}
```

可以看到这次只是简单的回退到了之前的状态，并没有退出Activity。如果要退出Activity，需要再按一次返回键。

### 弹出回退栈

 Fragment的回退非常简单，然而这里又会出现一个新的问题，就是在修改后的案例每次只能回退到上一步操作，而并不能一次性回退到我们想要的位置，这样才更满足实际开发需要。

这就需要我们来多了解事物回滚的相关原理，其实在Fragment回退时，默认调用FragmentManager的popBackStack()方法将最上层的操作弹出回退栈。当栈中有多层时，我们可以根据id或TAG标识来指定弹出到的操作所在层。

- popBackStack(int id, int flags)：其中id表示提交变更时commit()的返回值。

- popBackStack(String name, int flags)：其中name是addToBackStack(String tag)中的tag值。


在上面2个方法里面，都用到了flags，其实flags有两个取值：0或FragmentManager.POP_BACK_STACK_INCLUSIVE。当取值0时，表示除了参数指定这一层之上的所有层都退出栈，指定的这一层为栈顶层；当取值POP_BACK_STACK_INCLUSIVE时，表示连着参数指定的这一层一起退出栈。

如果想要了解回退栈中Fragment的情况，可以通过以下2个方法来实现：

- getBackStackEntryCount()：获取回退栈中Fragment的个数。

- getBackStackEntryAt(int index)：获取回退栈中该索引值下的Fragment。


使用popBackStack()来弹出栈内容的话，调用该方法后会将事物操作插入到FragmentManager的操作队列，只有当轮询到该事物时才能执行。如果想立即执行事物的话，可以使用下面这几个方法：

- popBackStackImmediate()

- popBackStackImmediate(String tag)

- popBackStackImmediate(String tag, int flag)

- popBackStackImmediate(int id, int flag)


这几个方法的参数同popBackStack()，由于比较简单这里就不在单独举例了，建议大家自行练习。

Fragment的基本操作已经学习完毕，产生了一些新的问题：一个Activity中可能会有多个Fragment，在这些Fragment中如何相互通信呢？又如何与所在的Activity相互通信？

### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)