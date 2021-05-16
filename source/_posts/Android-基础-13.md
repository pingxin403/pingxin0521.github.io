---
title: Android  菜单
date: 2019-04-30 23:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### 前言

Google在开发者网站上关于[Menu的API指南](https://developer.android.google.cn/guide/topics/ui/menus.html)中为开发者推荐了三种基本的菜单：选项菜单（OptionsMenu）、上下文菜单（ContextMenu）和弹出菜单（PopupMenu）。下面分别给出相应的基本实现步骤。

<!--more-->

从Android3.0（API level 11）开始，Android设备不再要求提供一个专门的菜单按钮，转而推荐使用ActionBar。所以现在市面上很多新设备使用三个虚拟按键，并不再额外提供菜单按钮。

因为Android版本的发展，对于菜单的支持各个版本有很大的区别，而Android3.0是个分水岭，大概可以分为下面三类：

- OptionMenu和ActionBar：一些操作的集合，如果开发的平台在Android3.0之上，推荐使用ActionBar，如果开发的平台在Android2.3或之下，还是可以使用OptionMenu的。
- ContextMenu和ActionMode：ContextMenu是一个浮动的窗口形式展现一个选项列表，ActionMode是一个显示在屏幕顶部的操作栏，允许用户选择多个选项，ActionMode在Android3.0之后才有支持。
- Popup Menu：PopupMenu是固定在View上的模态菜单，以弹出的方式显示，在Android3.0之后才有支持。

#### 常用方法

- onCreateOptionsMenu(Menu menu)
  每次Activity一创建就会执行，一般只执行一次，创建并**保留**Menu的实例；

  ```
  //获取MenuInflater
      MenuInflater inflater = getMenuInflater();
  //加载Menu资源
      inflater.inflate(R.menu.option_menu_normal,menu);
      
  //此方法必须返回true，否则不予显示
      return true；
  ```

- onPrepareOptionsMenu(Menu menu)
   每次menu被打开时，该方法就会执行一次，可用于对传入的旧Menu对象进行修改操作；

- onOptionsItemSelected(MenuItem item)
   每次menu菜单项被点击时，该方法就会执行一次；

  ```
  @Override
  public boolean onOptionsItemSelected(MenuItem item) {
      switch (item.getItemId()){
          case R.id.option_normal_1:
              return true; //返回true，将结束点击事件，不会进入下一级菜单
          case R.id.option_normal_2:
              return true;
          case R.id.option_normal_3:
              return true;
          case R.id.option_normal_4:
              return true;
          default:
              return super.onOptionsItemSelected(item); //不返回true，避免截断菜单项的点击事件
      }
  }
  ```

- invalidateOptionsMenu()
  刷新menu里的选项里内容，它会调用onCreateOptionsMenu(Menu menu)方法
- onCreateContextMenu()
  创建控件绑定的上下文菜单menu，根据方法里的View参数识别是哪个控件绑定
- onContextItemSelected(MenuItem item)
  点击控件绑定的上下菜单menu的内容项

- invalidateOptionsMenu()
  通知系统刷新Menu，之后，onPrepareOptionsMenu会被调用

#### 使用XML定义Menu

理论上而言，使用XML和Java代码都可以创建Menu。但是在实际开发中，往往通过XML文件定义Menu，这样做有以下几个好处：

- 使用XML可以获得更清晰的菜单结构
- 将菜单内容与应用的逻辑代码分离
- 可以使用应用资源框架，为不同的平台版本、屏幕尺寸创建最合适的菜单（如对drawable、string等系统资源的使用）

### OptionsMenu

OptionsMenu在页面的标题栏的右侧创建一个菜单按钮，前提是菜单栏可以显示的时候才有，否则会被隐藏。点击按钮弹出OptionsMenu菜单。

1. 创建选项菜单：重写Activity的onCreateOptionMenu（Menu menu）方法。

   - 设置菜单项可用代码动态设置menu.add();

   - 还可以通过xml设置MenuInflater.inflate();

2. 设置菜单点击事件：onOptionsItemSelected();

3. 菜单关闭后发生的动作：onOptionMenuClosed(Menu menu);

4. 选项菜单显示之前会调用，可以在这里根据需要调整菜单：onPrepareOptionsMenu(Menu menu);

5. 打开后发生的动作： onMenuOpened(int featureId,Menu menu);

#### 方法一：

通过xml设置菜单（res/menu/menu.xml）

```xml
    <menu xmlns:android="http://schemas.android.com/apk/res/android"  
        xmlns:tools="http://schemas.android.com/tools"  
        tools:context="com.example.menudemo.MenuActivity" >  
       <group android:id="@+id/group1">  
         <item  
            android:id="@+id/action_menu1"  
            android:orderInCategory="300"  
            android:menuCategory="container"  
            android:showAsAction="never"  
            android:title="menu1"/>  
        <item  
            android:id="@+id/action_menu2"  
            android:orderInCategory="200"  
            android:menuCategory="system"  
            android:showAsAction="never"  
            android:title="menu2"/>  
       </group>  
       <group android:id="@+id/group2">  
           <item  
            android:id="@+id/action_menu3"  
            android:orderInCategory="100"  
            android:menuCategory="secondary"  
            android:showAsAction="never"  
            android:title="menu3"/>  
        <item  
            android:id="@+id/action_menu4"  
            android:orderInCategory="400"  
            android:menuCategory="alternative"  
            android:showAsAction="never"  
            android:title="menu4"/>  
       </group>    
    </menu>  
```

 **<item\>标签的属性含义解释：**

| 属性名             | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| menuCategory       | 设置菜单项的种类。有四个可选值：[Container](http://lib.csdn.net/base/docker)、system、secondary、alternative。通过menuCategory属性可以控制菜单项的位置。 |
| orderInCategory    | 同类菜单的排列顺序，为整数值，值越大显示越靠前。             |
| titleCondensed     | 菜单项的短标题。当菜单文字太长时显示这个                     |
| alphabeticShortcut | 菜单项的字母快捷键。                                         |
| showAsAction       | **Never**：总是显示在移除菜单中。  **Always**：显示在ActionBar上。  **ifRoom**：如果actionBar空间足够就显示在ActionBar上。  **withText**：默认格式如果是含有文字和图表的话，只显示图标，使用ifRoom\|withText可以显示图标和文字。 android:showAsAction属性也可包含“collapseActionView”属性值，这个值是可选的，并且声明了这个操作视窗应该被折叠到一个按钮中，当用户选择这个按钮时，这个操作视窗展开。否则，这个操作视窗在默认的情况下是可见的，并且即便在用于不适用的时候，也要占据操作栏的有效空间。  参考链接：http://blog.csdn[.NET](http://lib.csdn.net/base/dotnet)/think_soft/article/details/7370686 |

**<group\>标签**的作用是可以进行整组操作，把一些具有相同操作的菜单放到一个组内。

Fragment的onCreate()方法中调用 setHasOptionsMenu(true) 方法使之显示出来。

在Activity里加载菜单

```java
@Override  
public boolean onCreateOptionsMenu(Menu menu) {  
// Inflate the menu; this adds items to the action bar if it is present.  
getMenuInflater().inflate(R.menu.menu, menu);  
return true;  
}       
```

#### 方法二：

通过代码添加menu

```java
    @Override  
        public boolean onCreateOptionsMenu(Menu menu) {  
            
            /* 
             * add()方法的四个参数，依次是： 
             * 1、组别，如果不分组的话就写Menu.NONE, 
             * 2、Id，这个很重要，Android根据这个Id来确定不同的菜单 
             * 3、顺序，那个菜单现在在前面由这个参数的大小决定，参数越小，显示的越前 
             * 4、文本，菜单的显示文本 
             */  
            menu.add(Menu.NONE, Menu.FIRST + 1, 5, "删除").setIcon(  
            android.R.drawable.ic_menu_delete);  
            // setIcon()方法设置菜单图标  
            menu.add(Menu.NONE, Menu.FIRST + 2, 2, "保存").setIcon(  
            android.R.drawable.ic_menu_save);  
            menu.add(Menu.NONE, Menu.FIRST + 3, 6, "帮助").setIcon(  
            android.R.drawable.ic_menu_help);  
            menu.add(Menu.NONE, Menu.FIRST + 4, 1, "添加").setIcon(  
            android.R.drawable.ic_menu_add);  
            menu.add(Menu.NONE, Menu.FIRST + 5, 4, "详细").setIcon(  
            android.R.drawable.ic_menu_info_details);  
            menu.add(Menu.NONE, Menu.FIRST + 6, 3, "发送").setIcon(  
            android.R.drawable.ic_menu_send);  
            return true;  
    }  
    /**
     * 通过反射，设置menu显示icon
     *
     * @param view
     * @param menu
     * @return
     */
    @SuppressLint("RestrictedApi")
    @Override
    protected boolean onPrepareOptionsPanel(View view, Menu menu) {
        if (menu != null) {
            if (menu.getClass() == MenuBuilder.class) {
                try {
                    @SuppressLint("PrivateApi")
                    Method m = menu.getClass().
                            getDeclaredMethod("setOptionalIconsVisible",
                                    Boolean.TYPE);
                    m.setAccessible(true);
                    m.invoke(menu, true);
                } catch (Exception e) {
                }
            }
        }
        return super.onPrepareOptionsPanel(view, menu);
    }

```

![1.png](https://i.loli.net/2019/05/14/5cda152a8a9d355073.png)

**选项菜单设置点击监听**

```java

    @Override  
        public boolean onOptionsItemSelected(MenuItem item) {  
            Log.i(TAG, "onOptionsItemSelected");  
            int id = item.getItemId();  
            switch (id) {  
            case Menu.FIRST+1:  
                Toast.makeText(MenuActivity.this, "点击了删除按钮", Toast.LENGTH_SHORT).show();  
                break;  
            default:  
                break;  
            }  
            return super.onOptionsItemSelected(item);  
        }  
```



### ContextMenu

当用户长时间按键不放时，弹出来的菜单称为上下文菜单。Windows中用鼠标右键弹出的菜单就是上下文菜单。

上下文菜单是用户长按某一元素时出现的浮动菜单。它提供的操作将影响所选内容，主要应用于列表中的每一项元素（如长按列表项弹出删除对话框）。上下文操作模式将在屏幕顶部栏（菜单栏）显示影响所选内容的操作选项，并允许用户选择多项，一般用于对列表类型的数据进行批量操作。

要提供浮动上下文菜单，可以参照以下步骤：

- 在Activity或Fragment中调用 registerForContextMenu(View v) 方法，注册需要和上下文菜单关联的View。如果将ListView或GridView作为参数传入，那么每个列表项将会有相同的浮动上下文菜单。
- 在Activity或Fragment中重写onCreateContextMenu方法，加载Menu资源。
- 在Activity或Fragment中重写onContextItemSelected方法，实现菜单项的点击逻辑。

 创建上下文菜单的步骤：

1. 调用registerForContextMenu（）方法，为视图注册上下文菜单。如`textView tv`

   ```
   
       @Override  
           protected void onCreate(Bundle savedInstanceState) {  
               super.onCreate(savedInstanceState);  
               setContentView(R.layout.activity_menu);  
               tv = (TextView)findViewById(R.id.tv);  
               registerForContextMenu(tv);  
           }  
   ```

2. 覆盖Activity的onCreateContextMenu（）方法，调用Menu的add方法添加菜单项（MenuItem）

   ```
   
       @Override  
           public void onCreateContextMenu(ContextMenu menu, View v,  
                   ContextMenuInfo menuInfo) {  
               setIconVisible(menu);  
               // TODO Auto-generated method stub  
                menu.add(Menu.NONE, Menu.FIRST + 1, 5, "删除").setIcon(  
                           android.R.drawable.ic_menu_delete);  
               // setIcon()方法设置菜单图标  
               menu.add(Menu.NONE, Menu.FIRST + 2, 2, "保存").setIcon(  
                           android.R.drawable.ic_menu_save);  
               menu.add(Menu.NONE, Menu.FIRST + 3, 6, "帮助").setIcon(  
                           android.R.drawable.ic_menu_help);  
               menu.add(Menu.NONE, Menu.FIRST + 4, 1, "添加").setIcon(  
                           android.R.drawable.ic_menu_add);  
               menu.add(Menu.NONE, Menu.FIRST + 5, 4, "详细").setIcon(  
                           android.R.drawable.ic_menu_info_details);  
               menu.add(Menu.NONE, Menu.FIRST + 6, 3, "发送").setIcon(  
                           android.R.drawable.ic_menu_send);  
               super.onCreateContextMenu(menu, v, menuInfo);  
           }  
   ```

3. 覆盖onContextItemSelected（）方法，响应菜单单击事件。

   ```
   
       @Override  
           public boolean onContextItemSelected(MenuItem item) {  
               // TODO Auto-generated method stub  
               int id = item.getItemId();  
               switch (id) {  
               case Menu.FIRST+1:  
                   Toast.makeText(MenuActivity.this, "点击了删除按钮", Toast.LENGTH_SHORT).show();  
                   break;  
               default:  
                   break;  
               }  
               return super.onContextItemSelected(item);  
           }  
   ```

   ![1.png](https://i.loli.net/2019/05/14/5cda163bda93484721.png)

####  创建子菜单步骤：

1. 覆盖Activity的onCreateOptionsMenu（）方法，调用Menu的addSubMenu（）方法；添加子菜单项（subMenu）

   ```
   
       @Override  
           public boolean onCreateOptionsMenu(Menu menu) {  
               //调用这个方法设置图标的可见性  
               setIconVisible(menu);  
               /* 
                * add()方法的四个参数，依次是： 
                * 1、组别，如果不分组的话就写Menu.NONE, 
                * 2、Id，这个很重要，Android根据这个Id来确定不同的菜单 
                * 3、顺序，那个菜单现在在前面由这个参数的大小决定，参数越小，显示的越前 
                * 4、文本，菜单的显示文本 
                */  
                
             //添加子菜单    
               SubMenu subMenu = menu.addSubMenu(0,2,Menu.NONE, "难度星级->").setIcon(android.R.drawable.ic_menu_directions);    
                 //添加子菜单项    
                 subMenu.add(2, 100, 1, "☆☆☆☆☆");    
                 subMenu.add(2, 101, 2, "☆☆☆");    
                 subMenu.add(2, 102, 3, "☆");  
       }  
   ```

2. 覆盖onOptionsItemSelected()方法响应点击事件。上下文菜单处理方式和此一致。

   ```
   
   
       @Override  
           public boolean onOptionsItemSelected(MenuItem item) {  
               Log.i(TAG, "onOptionsItemSelected");  
               int id = item.getItemId();  
               switch (id) {  
               case 100:  
                   Toast.makeText(MenuActivity.this, "点击了五颗星", Toast.LENGTH_SHORT).show();  
                   break;  
               default:  
                   break;  
               }  
               return super.onOptionsItemSelected(item);  
           }
   ```

   



#### 示例

ContextMenu针对在Activity的View中的所有控件，在长按View控件达到2秒钟弹出一个菜单项。

效果图：

![1.png](https://i.loli.net/2019/05/11/5cd6c3efec080.png)


实现方式：

1. 在Activity的OnCreate()方法中对Button按钮或者整个活动页面添加菜单响应监听：

   ```
   //长按弹出上下文菜单的Button
   Button contextMenu = (Button) findViewById(R.id.contextMenu);
   //长按页面弹出上下文菜单
   View view = findViewById(R.id.theLayout);
   contextMenu.setOnCreateContextMenuListener(this);//为Button单独实现ContextMenu
   this.registerForContextMenu(view);//长按页面空白地方会弹出Menu
   ```

   因为Activity已经实现了OnCreateContextMenuListener，所以要在Activity中重写该类的一些方法：

2. 创建菜单

   ```
       //方法2，因为activity已经实现了OnCreateContextMenuListener，所以只需要重写onCreateContextMenu()方法
       @Override
       public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {//这里是ContextMenu参数
           setContextMenuIconEnable(menu, true);//设置菜单的Icon图标可见，这个方法后面详解,针对Android高版本的菜单图标显示的方法
           menu.add(1, 2, 1, "添加").setIcon(android.R.drawable.ic_menu_add);
           menu.add(1, 3, 2, "删除").setIcon(android.R.drawable.ic_menu_delete);
           SubMenu m = menu.addSubMenu(0, 4, 3, "子菜单").setIcon(android.R.drawable.ic_media_next);
           m.add(0, 41, 0, "子菜单项1");
           m.add(0, 42, 0, "子菜单项2");
           m.add(0, 43, 0, "子菜单项3");
           super.onCreateContextMenu(menu, v, menuInfo);
      }
   ```

3. 菜单关闭

   ```
       @Override
       public void onContextMenuClosed(Menu menu) {
           Toast.makeText(this,"关闭",Toast.LENGTH_SHORT).show();
           super.onContextMenuClosed(menu);
       }
   ```

4. 菜单被点击

   ```
    @Override
       public boolean onContextItemSelected(MenuItem item) {
           switch (item.getItemId()) {
               case 2:
                   Toast.makeText(this, "添加", Toast.LENGTH_SHORT).show();
                   break;
               case 3:
                   Toast.makeText(this, "删除", Toast.LENGTH_LONG).show();
                   break;
               default:
                   break;
           }
           return super.onContextItemSelected(item);
       }
   ```



### PopupMenu

弹出菜单以垂直列表形式显示一系列操作选项，一般由某一控件触发，弹出菜单将显示在对应控件的上方或下方。它适用于提供与特定内容相关的大量操作

可以将弹出菜单的使用拆分为以下四个步骤：

- 实例化PopupMenu，它的构造方法需要两个参数，分别为Context以及PopupMenu依赖的View对象。
- 使用MenuInflater将Menu资源加载到PopupMenu.getMenu()返回的Menu对象中。
- 调用setOnMenuItemClickListener方法为PopupMenu设置点击监听器。
- 调用==PopupMenu.show()==将弹出菜单显示出来。

```
//XML文件代码

<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/popup_add"
        android:title="添加"/>
    <item android:id="@+id/popup_delete"
        android:title="删除"/>
    <item android:id="@+id/popup_more"
        android:title="更多"/>
</menu>

//Java文件代码

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_popup_menu);

    findViewById(R.id.popup_menu_view).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            PopupMenu popupMenu=new PopupMenu(PopupMenuActivity.this,view);//1.实例化PopupMenu
            getMenuInflater().inflate(R.menu.popup_menu,popupMenu.getMenu());//2.加载Menu资源

            //3.为弹出菜单设置点击监听
            popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
                @Override
                public boolean onMenuItemClick(MenuItem item) {
                    switch (item.getItemId()){
                        case R.id.popup_add:
                            Toast.makeText(PopupMenuActivity.this,"添加",Toast.LENGTH_SHORT).show();
                            return true;
                        case R.id.popup_delete:
                            Toast.makeText(PopupMenuActivity.this,"删除",Toast.LENGTH_SHORT).show();
                            return true;
                        case R.id.popup_more:
                            Toast.makeText(PopupMenuActivity.this,"更多",Toast.LENGTH_SHORT).show();
                            return true;
                        default:
                            return false;
                    }
                }
            });
            popupMenu.show();//4.显示弹出菜单
        }
    });
}

```

#### 示例

效果图：

![3.png](https://i.loli.net/2019/05/11/5cd6c3f00b537.png)

实现方式：

1. 为控件添加属性如下:

```html
android:onClick="popUpMenu"
```

 2. 在Activity中添加对应函数

    ```
        public void popUpMenu(View view) {
            /**
             * 第一个参数上下文对象
             * 第二个参数为锚点，何谓锚点？就是你这个弹出式菜单以哪个view为标准弹出，
             * 如果在这个view的下面有空间就在这个view下面的空间弹出，如果没有空间就
             * 在此view的上面弹出
             */
            PopupMenu popupMenu = new PopupMenu(this, view);
            setOptionsAndPopupMenuIconEnable(popupMenu.getMenu(), true);//显示Menu的图标方法，稍后详解
            /**
             * 第一个参数为菜单布局文件
             * 第二个参数为Menu对象，通过getMenu()获得。
             */
            getMenuInflater().inflate(R.menu.menu, popupMenu.getMenu());
            /**
             * 要想显示必须调用.show()方法
             */
            //弹出菜单的点击事件也没有回调事件，需要自己设置监听事件onMenuItemCilckListener()，
            //并在其中实现逻辑，代码如下：
            //值得注意的是最后的返回值，这个涉及到view的事件分发。简单的意思就是，返回true代码消耗这次点击事件，
            //事件不会继续传播，如果返回false则事件会继续传播的。
            //比如这个弹出菜单的又设置了setOnDismissListener这个事件，
            //那么如果返回true，则这个事件不起作用，返回false则点击事件处理完后，这个事件还会起作用。
            //但是亲测true好像没有用
            popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
                @Override
                public boolean onMenuItemClick(MenuItem menuItem) {
                    Toast.makeText(MainActivity.this, menuItem.getTitle().toString(), Toast.LENGTH_SHORT).show();
                    return false;
                }
            });
            popupMenu.setOnDismissListener(new PopupMenu.OnDismissListener() {
                @Override
                public void onDismiss(PopupMenu popupMenu) {
                    Toast.makeText(MainActivity.this, "PopupMenu的Dismiss", Toast.LENGTH_SHORT).show();
                }
            });
            popupMenu.setGravity(Gravity.BOTTOM);//没有用
            popupMenu.show();
        }
    ```

上面可以看出关于PopupMenu菜单的所有的实现以及点击按钮事件都在这个方法内完成。

### 关于菜单的创建说明

1. add方法的参数解析

   ```
   //        add()方法的四个参数，依次是：
   //        1、组别，如果不分组的话就写Menu.NONE,
   //        2、Id，这个很重要，Android根据这个Id来确定不同的菜单
   //        3、顺序，哪个菜单项在前面由这个参数的大小决定
   //        4、文本，菜单项的显示文本
           menu.add(0, 5, 1, "添加1").setIcon(android.R.drawable.ic_menu_add);
   ```

2. 通过布局文件创建菜单

   新建布局文件，在res资源目录下新建一个menu资源文件夹，选择类型就是menu。然后在menu文件下面新建一个menu.xml文件，这个名字可以自己取。下面包含了二级菜单的创建。

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <menu xmlns:android="http://schemas.android.com/apk/res/android">
       <item
           android:id="@+id/add_item"
           android:icon="@android:drawable/ic_menu_add"
           android:title="Add" />
       <item
           android:id="@+id/remove_item"
           android:icon="@android:drawable/ic_popup_reminder"
           android:title="Remove" />
       <item
           android:id="@+id/item_china"
           android:orderInCategory="100"
           android:title="中国">
           <menu>
               <item
                   android:id="@+id/item_beijing"
                   android:orderInCategory="200"
                   android:title="北京"/>
               <item
                   android:id="@+id/item_shanghai"
                   android:orderInCategory="200"
                   android:title="上海"/>
           </menu>
       </item>
   </menu>
   ```

   通过资源文件新建菜单

   ```java
   getMenuInflater().inflate(R.menu.menu, Menu);
   ```

   通过该方法直接将meun.xml的菜单解析到Menu实例中。

#### 关于菜单图片的显示问题

Android的4.0之前，设置图标是默认显示的，但是4.0之后，就不在默认了。现在针对三种菜单Menu的图标显示方法如下

1. 针对ContextMenu菜单

   ```
       // 经过试验ContextMenu有效，对OptionMenu无效
       private void setContextMenuIconEnable(Menu menu, boolean enable) {
           if (menu != null) {
               //com.android.internal.view.menu.ContextMenuBuilder，这是菜单menu的实体类
               Log.d("AAA", menu.getClass().getName().toString());
               try {
                   //这是ContextMenu加载菜单的Builder，com.android.internal.view.menu.MenuBuilder
                   Class<?> clazz = Class.forName("com.android.internal.view.menu.MenuBuilder");
                   Method m = clazz.getDeclaredMethod("setOptionalIconsVisible", Boolean.TYPE);
                   Log.d("aaa", m.toString());
                   m.setAccessible(true);
                   //MenuBuilder实现Menu接口，创建菜单时，传进来的menu其实就是MenuBuilder对象(java的多态特征)
                   m.invoke(menu, enable);
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }
       }
   ```

   

2. 针对PopupMenu和OptionsMenu菜单

   ```
      //这是针对OptionsMenu和PopupMenu菜单的设置Icon图标可见的方法
       private void setOptionsAndPopupMenuIconEnable(Menu menu, boolean enable) {
           if (menu != null) {
               Log.d("BBB", menu.getClass().getName().toString());
               if (menu.getClass().getSimpleName().equals("MenuBuilder")) {
                   try {
                       //这是OptionMenu加载菜单的Builder，android.support.v7.view.menu.MenuBuilder
                       //Class clazz = Class.forName("android.support.v7.view.menu.MenuBuilder");
                       Method m = menu.getClass().getDeclaredMethod(
                               "setOptionalIconsVisible", Boolean.TYPE);
                       m.setAccessible(true);
                       m.invoke(menu, enable);
                   } catch (Exception e) {
                   }
               }
           }
       }
   ```

   