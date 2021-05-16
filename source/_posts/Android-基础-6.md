---
title: Android 基本布局
date: 2019-04-28 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---
### 前言
Android六大基本布局分别是：
1. 线性布局（LinearLayout）：按照垂直或者水平方向布局的组件
2. 帧布局（FrameLayout）：组件从屏幕左上方布局组件
3. 表格布局（TableLayout）：按照行列方式布局组件
4. 绝对布局（AbsoluteLayout）：按照绝对坐标来布局组件
5. 相对布局（RelativeLayout）：相对其它组件的布局方式
6. 约束布局 （ConstraintLayout）：按照约束布局组件
7. 网格布局（GridLayout）

<!--more-->

其中，表格布局是线性布局的子类。网格布局是android 4.0后新增的布局。

在手机程序设计中，绝对布局基本上不用，用得相对较多的是线性布局和相对布局。

学习基本布局要理解两个比较基础概念的图：

![1.png](https://i.loli.net/2019/05/07/5cd14f2c75947.png)

上面这个类图只是说了六大基本布局的关系，其实ViewGroup还有其他一些布局管理器。
这里要理解一点就是布局也是布局管理器，因为布局里面还可以添加布局。

Android布局的XML关系图

![2.png](https://i.loli.net/2019/05/07/5cd14f2c5346a.png)

布局管理器里面既可以添加多个布局管理器又可以添加多个控件，而控件里面不能在添加布局或控件了。


### 线性布局
![1.png](https://i.loli.net/2019/05/07/5cd15121009ed.png)

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"    
    android:layout_height="match_parent"    
    android:orientation="vertical">    

<LinearLayout        
    android:layout_width="match_parent"        
    android:layout_height="250dp"        
    android:orientation="horizontal">        
    <TextView           
        android:layout_width="96dp"      
        android:layout_height="match_parent"
        android:background="#b2dfdb" />        
    <TextView            
        android:layout_width="96dp"               
        android:layout_height="match_parent"             
        android:background="#80cbc4" />       
     <TextView            
        android:layout_width="96dp"                   
        android:layout_height="match_parent"            
        android:background="#4db6ac" />        
    <TextView            
        android:layout_width="96dp"            
        android:layout_height="match_parent"            
        android:background="#26a69a" />    
</LinearLayout>

<LinearLayout        
    android:layout_width="match_parent"            
    android:layout_height="match_parent"        
    android:orientation="vertical">        
    <TextView           
        android:layout_width="match_parent"            
        android:layout_height="68dp"            
        android:background="#b2dfdb" />        
    <TextView            
        android:layout_width="match_parent"                
        android:layout_height="68dp"            
        android:background="#80cbc4" />        
    <TextView            
        android:layout_width="match_parent"                
        android:layout_height="68dp"            
        android:background="#4db6ac" />        
    <TextView            
        android:layout_width="match_parent"                    
        android:layout_height="68dp"            
        android:background="#26a69a" />    
</LinearLayout>
</LinearLayout>
```

线性布局特点，同一级组件之间没有互相依赖，按照添加顺序依次排列，其中线性布局最重要的属性是android:orientation，通过该属性可以指定水平方向排列还是纵向排列

xml属性：

![1.jpeg](https://i.loli.net/2019/05/08/5cd27b7856bd2.jpeg)

LinearLayout 包含的所有子元素都受 LinearLayout.LayoutParams 控制，因此 LinearLayout包含的子元素可以额外指定如如下属性。

- android:layout_gravity：指定该子元素在LinearLayout中的对齐方式。

- android:layout_weight：指定该子元素在LinearLayout中所占的权重。

#### gravity和layout_gravity

gravity控制组件的重心，也叫对齐方式，表示view横向和纵向的停靠位置。主要通过以下两个属性来控制。

- android:gravity：是对view组件本身来说的，是用来设置组件本身的内容应该显示在组件的什么位置，默认值是左侧。

- android:layout_gravity：是相对于包含该元素的父元素来说的，设置该元素在父元素的什么位置。

其属性值主要有以下几种：

- top：将对象放在其容器的顶部，不改变其大小。

- bottom：将对象放在其容器的底部，不改变其大小。

- left：将对象放在其容器的左侧，不改变其大小。

- right：将对象放在其容器的右侧，不改变其大小。

- center_vertical：将对象纵向居中，不改变其大小。垂直对齐方式：垂直方向上居中对齐。

- fill_vertical：必要的时候增加对象的纵向大小，以完全充满其容器。垂直方向填充。

- center_horizontal：将对象横向居中，不改变其大小。水平对齐方式：水平方向上居中对齐。

- fill_horizontal：必要的时候增加对象的横向大小，以完全充满其容器。水平方向填充。

- center：将对象横纵居中，不改变其大小。

- fill：必要的时候增加对象的横纵向大小，以完全充满其容器。

- clip_vertical：附加选项，用于按照容器的边来剪切对象的顶部和/或底部的内容。剪切基于其纵向对齐设置：顶部对齐时剪切底部；底部对齐时剪切顶部；除此之外剪切顶部和底部。垂直方向裁剪。

- clip_horizontal：附加选项，用于按照容器的边来剪切对象的左侧和/或右侧的内容。剪切基于其横向对齐设置：左侧对齐时剪切右侧；右侧对齐时剪切左侧；除此之外剪切左侧和右侧。水平方向裁剪。

示例：

android:gravity

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="top"
        android:gravity="top"
        android:textColor="#ffffff"
        android:background="#ff0000" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="bottom"
        android:gravity="bottom"
        android:textColor="#ffffff"
        android:background="#0000ff" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="left"
        android:gravity="left"
        android:textColor="#ffffff"
        android:background="#ff0000" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="right"
        android:gravity="right"
        android:textColor="#ffffff"
        android:background="#0000ff" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="center_vertical"
        android:gravity="center_vertical"
        android:textColor="#ffffff"
        android:background="#ff0000" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="fill_vertical"
        android:gravity="fill_vertical"
        android:textColor="#ffffff"
        android:background="#0000ff" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="center_horizontal"
        android:gravity="center_horizontal"
        android:textColor="#ffffff"
        android:background="#ff0000" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="fill_horizontal"
        android:gravity="fill_horizontal"
        android:textColor="#ffffff"
        android:background="#0000ff" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="center"
        android:gravity="center"
        android:textColor="#ffffff"
        android:background="#ff0000" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="fill"
        android:gravity="fill"
        android:textColor="#ffffff"
        android:background="#0000ff" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="clip_vertical"
        android:gravity="clip_vertical"
        android:textColor="#ffffff"
        android:background="#ff0000" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:text="clip_horizontal"
        android:gravity="clip_horizontal"
        android:textColor="#ffffff"
        android:background="#0000ff" />
</LinearLayout>
```

android:layout_gravity

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <!-- 水平左右对齐 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="#ff0000">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="left"
            android:background="#ffffff"
            android:layout_gravity="left"  />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="center_horizontal"
            android:background="#ffffff"
            android:layout_gravity="center_horizontal"  />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="right"
            android:background="#ffffff"
            android:layout_gravity="right" />
    </LinearLayout>
 
    <!-- 垂直上下对齐 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:orientation="horizontal"
        android:background="#0000ff">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="top"
            android:background="#ffffff"
            android:layout_gravity="top" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="center_vertical"
            android:background="#ffffff"
            android:layout_gravity="center_vertical" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="bottom"
            android:background="#ffffff"
            android:layout_gravity="bottom"  />
    </LinearLayout>
 
    <!-- 整体居中对齐 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:orientation="horizontal"
        android:background="#ff0000">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="center"
            android:background="#ffffff"
            android:layout_gravity="center"  />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:orientation="vertical"
        android:background="#0000ff">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="center"
            android:background="#ffffff"
            android:layout_gravity="center"  />
    </LinearLayout>
</LinearLayout>
```

从上面两个示例可以发现android:layout_gravity和android:gravity两个属性的差别，一定要理解透彻。

#### padding和margin

**内边距padding**

默认情况下，组件相互之间是紧紧靠在一起的。但是有时候需要组件各边之间有一定的内边距，那就可以通过以下几个属性来设置，内边距的值是具体的尺寸，如5dp。

- android:padding：为组件的四边设置相同的内边距。

- android:paddingLeft：为组件的左边设置内边距。

- android:paddingRight：为组件的右边设置内边距。

- android:paddingTop：为组件的上边设置内边距。

- android:paddingBottom：为组件的下边设置内边距。

![1.jpeg](https://i.loli.net/2019/05/08/5cd27c3ec0ce6.jpeg)

**外边距margin**

通过设置内边距，只能设置内容相对于组件之间的距离，而组件之间仍然是相邻挨着的。在实际开发中，有时候需要组件之间有一定的间隔距离，那么就需要用到外边距了，可以通过以下几个属性来设置。

- android:layout_margin：本组件离上下左右各组件的外边距。

- android:layout_marginStart：本组件离开始的位置的外边距。

- android:layout_marginEnd：本组件离结束位置的外边距。

- android:layout_marginBottom：本组件离下部组件的外边距。

- android:layout_marginTop：本组件离上部组件的外边距。

- android:layout_marginLeft：本组件离左部组件的外边距。

- android:layout_marginRight：本组件离右部组件的外边距。

![2.jpeg](https://i.loli.net/2019/05/08/5cd27c6e6e6d6.jpeg)

如果把布局的内边距和外边距放在一张图中比较会更加直观，如下图所示：

![3.jpeg](https://i.loli.net/2019/05/08/5cd28a6c189c8.jpeg)

### FrameLayout帧布局

![2.png](https://i.loli.net/2019/05/07/5cd15120f057b.png)

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/FrameLayout1"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center"
        android:background="#FF6143" />

    <TextView
        android:layout_width="150dp"
        android:layout_height="150dp"
        android:layout_gravity="center"
        android:background="#7BFE00" />

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:background="#FFFF00" />

</FrameLayout>
```
帧布局会按照添加顺序层叠在一起，默认层叠在左上角位置，本例子因为设置了layout_gravity="center"，所以层叠在视图中心位置

帧布局为每个加入其中的控件创建一个空白区域（称为一帧，每个控件占据一 帧）。釆用帧布局方式设计界面时，只能在屏幕左上角显示一个控件，如果添加多个控件，这些控件会按照顺序在屏幕的左上角重叠显示。



### 表格布局
![3.png](https://i.loli.net/2019/05/07/5cd15121056cd.png)
```
<?xml version="1.0" encoding="utf-8"?>
<TableLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="#ff0000"
    android:shrinkColumns="0,2"
    >

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="我占据一行" />

    <TableRow>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="0" >
        </Button>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="111111111111111111111111" >
        </Button>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="222222222222222222222222" >
        </Button>
    </TableRow>

    <TableRow>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="000000000000000000000000" >
        </Button>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="11" >
        </Button>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="222222222222222222222222" >
        </Button>
    </TableRow>

    <TableRow>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="000000000000000000000000" >
        </Button>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_span="2"
            android:text="我占据2列" >
        </Button>
    </TableRow>
</TableLayout>
```
TableLayout继承了 LinearLayout，因此它的本质依然是线性布局管理器。每次向TableLayout中添加一个TableRow，该TableRow就是一个表格行，TableRow也是容器，因此它也可以不断地添加其他组件，每添加一个子组件该表格就增加一列。如果直接向TableLayout中添加组件，那么这个组件将直接占用一行。

TableLayout的一些属性：

```
android:shrinkColumns 设置可收缩的列
android:stretchColumns 设置可伸展的列
android:collapseColumns 设置要隐藏的列
默认所有的列为stretchColumns，如只有一个控件则独占一行
每列宽度的计算原则：首先根据可伸展列根据其内容最长的一行占据一行的百分比，再根据可收缩的列等分剩余的部分
例如图中，优先计算“ 111111111111111111111111”的宽度，再由剩下的列等分剩余的空间
如果遇到空间已经不足了，则剩余的列均不显示
控件的一些属性：
android:layout_column表示当前控件在第几列
android:layout_span表示合并单元格个数
```

### 绝对布局
难以实现多分辨率适配，不建议使用

绝对布局需要通过指定x、y坐标来控制每一个控件的位置，放入该布局的控件需要通过android:layout_x和android:layout_y 两个属性指定其准确的坐标值，并显示在屏幕上。

需要注意的是当使用AbsoluteLayout作为布局容器时，布局容器不再管理子组件的位置和大小，都需要开发人员自己控制。使用绝对布局时，每个子组件都可指定如下两个XML属性。

- layout_x：指定该子组件的X坐标。

- layout_y：指定该子组件的Y坐标。

```
<?xml version="1.0" encoding="utf-8"?>
<AbsoluteLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
 
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_x="0dp"
        android:layout_y="0dp"
        android:text="按钮1" />
 
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_x="40dp"
        android:layout_y="40dp"
        android:text="按钮2" />
 
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_x="80dp"
        android:layout_y="80dp"
        android:text="按钮3" />
 
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_x="120dp"
        android:layout_y="120dp"
        android:text="按钮4" />
</AbsoluteLayout>
```

 需要注意的是，理论上绝对布局可以完成任何的布局设计，但是实际的工程应用中不提倡使用这种布局。因为使用这种布局不但需要精确计算每个组件的大小，而且当应用程序运行在不同屏幕的手机上产生的效果也不相同，因此，一般不推荐使用绝对布局。一般可以用LinearLayout的weight权重+ RelativeLayout来构建我们的界面。



### 相对布局

已使用约束布局替代

 RelativeLayout，又叫相对布局，使用RelativeLayout标签。相对布局通常有两种形式，一种是相对于容器而言的，一种是相对于控件而言的。

 下表显示了RelativeLayout支持的常用XML属性及相关方法的说明。

![1.png](https://i.loli.net/2019/05/08/5cd28b8358e8e.png)

为了控制该布局容器中各子组件的布局分布，RelativeLayout提供了一个内部类： RelativeLayout.LayoutParams，该类提供了大量的XML属性来控制RelativeLayout布局容器中子组件的布局分布。

在相对于容器定位的属性主要有以下几个，属性值为true或false。

- android:layout_centerHorizontal：控制该组件是否和布局容器的水平居中。

- android:layout_centerVertical：控制该组件是否和布局容器的垂直居中。

- android:layout_centerInparent：控制该组件是否和布局容器的中央位置。

- android:layout_alignParentTop：控制该组件是否和布局容器的顶部对齐。

- android:layout_alignParentBottom：控制该组件是否和布局容器的底端对齐。

- android:layout_alignParentLeft：控制该组件是否和布局容器的左边对齐。

- android:layout_alignParentRight：控制该组件是否和布局容器的右边对齐。

- android:layout_alignParentStart：控制该组件是否和布局容器的开始对齐。

- android:layout_alignParentEnd：控制该组件是否和布局容器的末端对齐。

- android:layout_alignWithParentIfMissing：如果对应的兄弟组件找不到的话就以父容器做参照物。

在相对于其他组件定位的属性主要有以下几个，属性值为其他组件的id。

- android:layout_toLeftOf：本组件在某组件的左边。

- android:layout_toRightOf：本组件在某组件的右边。

- android:layout_toStartOf：本组件在某组件开始端。

- android:layout_toEndOf：本组件在某组件末端。

- android:layout_above：本组件在某组件的上方。

- android:layout_below：本组件在某组件的下方。

- android:layout_alignBaseline：本组件和某组件的基线对齐。

- android:layout_alignTop：本组件的顶部和某组件的的顶部对齐。

- android:layout_alignBottom：本组件的下边缘和某组件的的下边缘对齐。

- android:layout_alignRight：本组件的右边缘和某组件的的右边缘对齐。

- android:layout_alignLeft：本组件左边缘和某组件左边缘对齐。

- android:layout_alignStart：本组件的开始端和某组件开始端对齐。

- android:layout_alignEnd：本组件的末端和某组件末端对齐。

除此之外，RelativeLayout.LayoutParams 还继承了 android view. ViewGroup.MarginLayoutParams，因此 RelativeLayout 布局容器中每个子组件也可指定 android.view.ViewGroiip.MarginLayoutParams所支持的各XML属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
    <!-- 定义该组件位于父容器左上侧 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器左上侧"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"/>
    <!-- 定义该组件位于父容器上侧水平居中 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器上侧水平居中"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"/>
    <!-- 定义该组件位于父容器右上侧 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器右上侧"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"/>
    <!-- 定义该组件位于父容器左侧垂直居中 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="左中"
        android:layout_alignParentLeft="true"
        android:layout_centerVertical="true"/>
    <!-- 定义该组件位于父容器右侧垂直居中 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="右中"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"/>
    <!-- 定义该组件位于父容器左下侧 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器左下侧"
        android:layout_alignParentLeft="true"
        android:layout_alignParentBottom="true"/>
    <!-- 定义该组件位于父容器下侧水平居中 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器下侧水平居中"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"/>
    <!-- 定义该组件位于父容器右下侧 -->
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器右下侧"
        android:layout_alignParentRight="true"
        android:layout_alignParentBottom="true"/>
 
 
    <!-- 定义该组件位于父容器中间 -->
    <Button
        android:id="@+id/center_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="容器中央"
        android:layout_centerInParent="true"/>
    <!-- 定义该组件位于center_btn组件的上方 -->
    <Button
        android:id="@+id/center_top_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="中上"
        android:layout_above="@id/center_btn"
        android:layout_alignLeft="@id/center_btn"/>
    <!-- 定义该组件位于center_btn组件的下方 -->
    <Button
        android:id="@+id/center_bottom_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="中下"
        android:layout_below="@id/center_btn"
        android:layout_alignLeft="@id/center_btn"/>
    <!-- 定义该组件位于center_btn组件的左边 -->
    <Button
        android:id="@+id/center_bottom_left_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="中下左"
        android:layout_toLeftOf="@id/center_btn"
        android:layout_alignTop="@id/center_bottom_btn"/>
    <!-- 定义该组件位于center_btn组件的右边 -->
    <Button
        android:id="@+id/center_top_right_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="中上右"
        android:layout_toRightOf="@id/center_btn"
        android:layout_alignTop="@id/center_top_btn"/>
</RelativeLayout>
```



### 约束布局

![4.png](https://i.loli.net/2019/05/07/5cd15120dc641.png)
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView2"
        android:layout_width="wrap_content"
        android:layout_height="40dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:gravity="center"
        android:text="验证码"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView3" />

    <TextView
        android:id="@+id/textView3"
        android:layout_width="wrap_content"
        android:layout_height="40dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:gravity="center"
        android:text="手机号"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/editText2"
        android:layout_width="197dp"
        android:layout_height="39dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:ems="10"
        android:hint="请输入验证码"
        android:inputType="textPersonName"
        app:layout_constraintEnd_toStartOf="@+id/button2"
        app:layout_constraintStart_toEndOf="@+id/textView2"
        app:layout_constraintTop_toBottomOf="@+id/editText3" />

    <EditText
        android:id="@+id/editText3"
        android:layout_width="298dp"
        android:layout_height="40dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:ems="10"
        android:hint="请输入手机号"
        android:inputType="textPersonName"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/textView3"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="40dp"
        android:layout_marginEnd="8dp"
        android:layout_marginTop="8dp"
        android:text="获取验证码"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/editText3" />

    <Button
        android:id="@+id/button3"
        android:layout_width="358dp"
        android:layout_height="40dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:text="开始"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/editText2" />
</android.support.constraint.ConstraintLayout>
```
约束控件最低支持的版本是 Android 2.3 (Gingerbread)

### GridLayout

  网格布局实现了控件的交错显示，能够避免因布局嵌套对设备性能的影响，更利于自由布局的开发。网格布局用一组无限细的直线将绘图区域分成行、列和单元，并指定控件的显示区域和控件在该区域的显示方式

 下表显示了 GridLayout常用的XML属性及相关方法说明。

![1.jpeg](https://i.loli.net/2019/05/08/5cd28e5855d2e.jpeg)

为了控制GridLayout布局容器中各子组件的布局分布，GridLayout提供了一个内部类： GridLayout.LayoutParams，该类提供了大量的XML属性来控制GridLayout布局容器中子组件的布局分布。

下表显示了 GridLayout.LayoutParams常用的XML属性及相关方法。

![2.jpeg](https://i.loli.net/2019/05/08/5cd28ef01c49d.jpeg)



```
<?xml version="1.0" encoding="utf-8"?>
<GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:columnCount="4"
            android:rowCount="7">
 
    <TextView
        android:id="@+id/result_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_columnSpan="4"
        android:background="#eee"
        android:text="0"
        android:textColor="#000"
        android:textSize="50sp" />
 
    <Button
        android:id="@+id/clear_btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_columnSpan="4"
        android:text="Clear" />
    <Button
        android:id="@+id/one_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="1" />
    <Button
        android:id="@+id/two_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="2" />
    <Button
        android:id="@+id/three_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="3" />
    <Button
        android:id="@+id/devide_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="/" />
    <Button
        android:id="@+id/four_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="4" />
    <Button
        android:id="@+id/five_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="5" />
    <Button
        android:id="@+id/six_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="6" />
    <Button
        android:id="@+id/multiply_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="×" />
    <Button
        android:id="@+id/seven_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="7" />
    <Button
        android:id="@+id/eight_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="8" />
    <Button
        android:id="@+id/nine_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="9" />
    <Button
        android:id="@+id/minus_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="-" />
    <Button
        android:id="@+id/zero_btn"
        android:layout_columnSpan="2"
        android:layout_columnWeight="1"
        android:layout_gravity="fill"
        android:layout_rowWeight="1"
        android:text="0" />
    <Button
        android:id="@+id/point_btn"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="." />
    <Button
        android:id="@+id/plus_btn"
        android:layout_columnWeight="1"
        android:layout_rowSpan="2"
        android:layout_rowWeight="1"
        android:text="+" />
    <Button
        android:id="@+id/equal_btn"
        android:layout_columnSpan="3"
        android:layout_columnWeight="1"
        android:layout_rowWeight="1"
        android:text="=" />
</GridLayout>
```



### 参考：

1. [Android从入门到精通](https://blog.csdn.net/cqkxzsxy/article/category/7017381)