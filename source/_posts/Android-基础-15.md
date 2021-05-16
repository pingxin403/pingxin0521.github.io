---
title: Android 数据存储
date: 2019-05-02 10:18:59
tags:
 - Android
categories:
 - Android
 - 基础
---

### 前言

学习Android相关知识，数据存储是其中的重点之一，如果不了解数据，那么让你跟一款没有数据的应用玩，你能玩多久呢？答案是这和没有手机几乎是差不多的。我们聊QQ，聊微信，看新闻，刷朋友圈等都是看里面的数据，所以在Android中数据对我们是多么重要。

<!--more-->

数据，如今是数据大时代，谁拥有数据，谁就能掌握未来，这一点很可怕的，现在你用的手机APP中存在着你的大量数据信息，大数据的积累可以掌握出你的作息时间，生活规律等。

对数据的存储有着良好的技术支持，是一个好的开发平台的体现，如果不能长时间保持数据，那么必然会被时代发展所淘汰。那么有长期保持数据的概念，就有瞬时数据这一概念的出现，什么是瞬时数据呢？

见名知意，瞬时代表一瞬间，指当存储的数据因程序关闭或其他原因等导致数据丢失，如果你想发个自拍发个朋友圈，可是一刷新就没了，是不是很恼怒呢？气不气，气不气，是不是想砸手机？

本文介绍Android平台进行数据存储的五大方式,分别如下:   

1. 使用SharedPreferences存储数据

2. 文件存储数据      

3. SQLite数据库存储数据

4. 使用ContentProvider存储数据

5. 网络存储数据

文件存储\SharedPreference存储和SQLite数据库存储数据文件默认存放位置：

![1.jpg](https://i.loli.net/2019/05/14/5cda5d31515f660475.jpg)

### 使用SharedPreferences存储数据

适用范围：保存少量的数据，且这些数据的格式非常简单：字符串型、基本类型的值。比如应用程序的各种配置信息（如是否打开音效、是否使用震动效果、小游戏的玩家积分等），解锁口 令密码等

核心原理：保存基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息。通过DDMS的File Explorer面板，展开文件浏览树,很明显SharedPreferences数据总是存储在`/data/data/<package name>/shared_prefs`目录下。SharedPreferences对象本身只能获取数据而不支持存储和修改,存储修改是通过SharedPreferences.edit()获取的内部接口Editor对象实现。 SharedPreferences本身是一 个接口，程序无法直接创建SharedPreferences实例，只能通过Context提供的getSharedPreferences(String name, int mode)方法来获取SharedPreferences实例，该方法中name表示要操作的xml文件名，第二个参数具体如下：

- ​             Context.MODE_PRIVATE: 指定该SharedPreferences数据只能被本应用程序读、写。

- ​             Context.MODE_WORLD_READABLE:  指定该SharedPreferences数据能被其他应用程序读，但不能写。

- ​             Context.MODE_WORLD_WRITEABLE:  指定该SharedPreferences数据能被其他应用程序读，写

Editor有如下主要重要方法：

- ​             SharedPreferences.Editor clear():清空SharedPreferences里所有数据

- ​             SharedPreferences.Editor putXxx(String key , xxx value): 向SharedPreferences存入指定key对应的数据，其中xxx 可以是boolean,float,int等各种基本类型据

- ​             SharedPreferences.Editor remove(): 删除SharedPreferences中指定key对应的数据项

- ​             boolean commit(): 当Editor编辑完成后，使用该方法提交修改

   **实际案例**：运行界面如下

![1.png](https://i.loli.net/2019/05/14/5cda5c5ba8e0461181.png)

这里只提供了两个按钮和一个输入文本框，布局简单，故在此不给出界面布局文件了,程序核心代码如下：         

```
class ViewOcl implements View.OnClickListener{

        @Override
        public void onClick(View v) {

            switch(v.getId()){
            case R.id.btnSet:
                //步骤1：获取输入值
                String code = txtCode.getText().toString().trim();
                //步骤2-1：创建一个SharedPreferences.Editor接口对象，lock表示要写入的XML文件名，MODE_WORLD_WRITEABLE写操作
                SharedPreferences.Editor editor = getSharedPreferences("lock", MODE_WORLD_WRITEABLE).edit();
                //步骤2-2：将获取过来的值放入文件
                editor.putString("code", code);
                //步骤3：提交
                editor.commit();
                Toast.makeText(getApplicationContext(), "口令设置成功", Toast.LENGTH_LONG).show();
                break;
            case R.id.btnGet:
                //步骤1：创建一个SharedPreferences接口对象
                SharedPreferences read = getSharedPreferences("lock", MODE_WORLD_READABLE);
                //步骤2：获取文件中的值
                String value = read.getString("code", "");
                Toast.makeText(getApplicationContext(), "口令为："+value, Toast.LENGTH_LONG).show();
                
                break;
                
            }
        }
        
    }
```

读写其他应用的SharedPreferences: 步骤如下

1. 在创建SharedPreferences时，指定MODE_WORLD_READABLE模式，表明该SharedPreferences数据可以被其他程序读取

2. 创建其他应用程序对应的Context: Context pvCount =  createPackageContext("com.tony.app",  Context.CONTEXT_IGNORE_SECURITY);这里的com.tony.app就是其他程序的包名

3. 使用其他程序的Context获取对应的SharedPreferences: SharedPreferences read = pvCount**.**getSharedPreferences("lock", Context.MODE_WORLD_READABLE);

4. 如果是写入数据，使用Editor接口即可，所有其他操作均和前面一致。

SharedPreferences对象与SQLite数据库相比，免去了创建数据库，创建表，写SQL语句等诸多操作，相对而言更加方便，简洁。但是SharedPreferences也有其自身缺陷，比如其职能存储boolean，int，float，long和String五种简单的数据类型，比如其无法进行条件查询等。所以不论SharedPreferences的数据存储操作是如何简单，它也只能是存储方式的一种补充，而无法完全替代如SQLite数据库这样的其他数据存储方式。

### 文件存储数据

在介绍文件存储之前我们要先了解内存、外部存储、内部存储三个概念，我们先来考虑一个问题：

打开手机设置，选择应用管理，选择任意一个App，然后你会看到两个按钮，一个是清除缓存，另一个是清除数据，那么当我们点击清除缓存的时候清除的是哪里的数据？当我们点击清除数据的时候又是清除的哪里的数据？读完本文相信你会有答案。

在android开发中我们常常听到这样几个概念，内存，内部存储，外部存储，很多人常常将这三个东西搞混，那么我们今天就先来详细说说这三个东西是怎么回事？

内存，我们在英文中称作memory，内部存储，我们称为InternalStorage，外部存储我们称为ExternalStorage，这在英文中本不会产生歧义，但是当我们翻译为中文之后，前两个都简称为内存，于是，混了。

#### 那么究竟什么是内部存储什么是外部存储呢？

**内部存储**：

data文件夹就是我们常说的内部存储，当我们打开data文件夹之后（没有root的手机不能打开该文件夹）

一个文件夹是app文件夹，还有一个文件夹就是data文件夹，app文件夹里存放着我们所有安装的app的apk文件，其实，当我们调试一个app的时候，可以看到控制台输出的内容，有一项是uploading .....就是上传我们的apk到这个文件夹，上传成功之后才开始安装。另一个重要的文件夹就是data文件夹了，这个文件夹里边都是一些包名，打开这些包名之后我们会看到这样的一些文件：

1. data/data/包名/shared_prefs
2. data/data/包名/databases
3. data/data/包名/files

4. data/data/包名/cach

 如果打开过data文件，应该都知道这些文件夹是干什么用的，我们在使用sharedPreferenced的时候，将数据持久化存储于本地，其实就是存在这个文件中的xml文件里，我们App里边的数据库文件就存储于databases文件夹中，还有我们的普通数据存储在files中，缓存文件存储在cache文件夹中，存储在这里的文件我们都称之为内部存储。

**外部存储**

外部存储才是我们平时操作最多的，外部存储一般就是我们上面看到的storage文件夹，当然也有可能是mnt文件夹，这个不同厂家有可能不一样。

一般来说，在storage文件夹中有一个sdcard文件夹，这个文件夹中的文件又分为两类，一类是公有目录，还有一类是私有目录，其中的公有目录有九大类，比如DCIM、DOWNLOAD等这种系统为我们创建的文件夹，私有目录就是Android这个文件夹，这个文件夹打开之后里边有一个data文件夹，打开这个data文件夹，里边有许多包名组成的文件夹。

参考：http://www.cnblogs.com/jingmo0319/p/5586559.html

#### 文件存储

 **核心原理**: Context提供了两个方法来打开数据文件里的文件IO流 FileInputStream  openFileInput(String name); FileOutputStream(String name , int  mode),这两个方法第一个参数 用于指定文件名，第二个参数指定打开文件的模式。具体有以下值可选：

- ​             ***MODE_PRIVATE***：为默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容，如果想把新写入的内容追加到原文件中。可   以使用Context.MODE_APPEND
- ​             **MODE_APPEND**：模式会检查文件是否存在，存在就往文件追加内容，否则就创建新文件。
- ​             ***MODE_WORLD_READABLE***：表示当前文件可以被其他应用读取；
- ​             **MODE_WORLD_WRITEABLE**：表示当前文件可以被其他应用写入。

 除此之外，Context还提供了如下几个重要的方法：

- ​             **getDir(String name , int mode)**:在应用程序的数据文件夹下获取或者创建name对应的子目录
- ​             **File getFilesDir()**:获取该应用程序的数据文件夹得绝对路径
- ​             **String[] fileList()**:返回该应用数据文件夹的全部文件               

实际案例：界面沿用上图

```
public String read() {
        try {
            FileInputStream inStream = this.openFileInput("message.txt");
            byte[] buffer = new byte[1024];
            int hasRead = 0;
            StringBuilder sb = new StringBuilder();
            while ((hasRead = inStream.read(buffer)) != -1) {
                sb.append(new String(buffer, 0, hasRead));
            }

            inStream.close();
            return sb.toString();
        } catch (Exception e) {
            e.printStackTrace();
        } 
        return null;
    }
    
    public void write(String msg){
        // 步骤1：获取输入值
        if(msg == null) return;
        try {
            // 步骤2:创建一个FileOutputStream对象,MODE_APPEND追加模式
            FileOutputStream fos = openFileOutput("message.txt",
                    MODE_APPEND);
            // 步骤3：将获取过来的值放入文件
            fos.write(msg.getBytes());
            // 步骤4：关闭数据流
            fos.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

openFileOutput()方法的第一参数用于指定文件名称，不能包含路径分隔符“/” ，如果文件不存在，Android 会自动创建它。创建的文件保存在`/data/data/<package name>/files`目录，如：`/data/data/cn.tony.app/files/message.txt`，

#### 读写sdcard上的文件

其中读写步骤按如下进行:

1. 调用Environment的getExternalStorageState()方法判断手机上是否插了sd卡,且应用程序具有读写SD卡的权限，如下代码将返回true

   ```
   1. Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)
   ```

2. 调用Environment.getExternalStorageDirectory()方法来获取外部存储器，也就是SD卡的目录,或者使用"/mnt/sdcard/"目录

3. 使用IO流操作SD卡上的文件 

注意点：手机应该已插入SD卡，对于模拟器而言，可通过mksdcard命令来创建虚拟存储卡

  必须在AndroidManifest.xml上配置读写SD卡的权限

```
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

案例：

```
// 文件写操作函数
    private void write(String content) {
        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) { // 如果sdcard存在
            File file = new File(Environment.getExternalStorageDirectory()
                    .toString()
                    + File.separator
                    + DIR
                    + File.separator
                    + FILENAME); // 定义File类对象
            if (!file.getParentFile().exists()) { // 父文件夹不存在
                file.getParentFile().mkdirs(); // 创建文件夹
            }
            PrintStream out = null; // 打印流对象用于输出
            try {
                out = new PrintStream(new FileOutputStream(file, true)); // 追加文件
                out.println(content);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (out != null) {
                    out.close(); // 关闭打印流
                }
            }
        } else { // SDCard不存在，使用Toast提示用户
            Toast.makeText(this, "保存失败，SD卡不存在！", Toast.LENGTH_LONG).show();
        }
    }

    // 文件读操作函数
    private String read() {

        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) { // 如果sdcard存在
            File file = new File(Environment.getExternalStorageDirectory()
                    .toString()
                    + File.separator
                    + DIR
                    + File.separator
                    + FILENAME); // 定义File类对象
            if (!file.getParentFile().exists()) { // 父文件夹不存在
                file.getParentFile().mkdirs(); // 创建文件夹
            }
            Scanner scan = null; // 扫描输入
            StringBuilder sb = new StringBuilder();
            try {
                scan = new Scanner(new FileInputStream(file)); // 实例化Scanner
                while (scan.hasNext()) { // 循环读取
                    sb.append(scan.next() + "\n"); // 设置文本
                }
                return sb.toString();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (scan != null) {
                    scan.close(); // 关闭打印流
                }
            }
        } else { // SDCard不存在，使用Toast提示用户
            Toast.makeText(this, "读取失败，SD卡不存在！", Toast.LENGTH_LONG).show();
        }
        return null;
    }
```

### SQLite存储数据

SQLite是轻量级嵌入式数据库引擎，它支持 SQL  语言，并且只利用很少的内存就有很好的性能。现在的主流移动设备像Android、iPhone等都使用SQLite作为复杂数据的存储引擎，在我们为移动设备开发应用程序时，也许就要使用到SQLite来存储我们大量的数据，所以我们就需要掌握移动设备上的SQLite开发技巧

SQLiteDatabase类为我们提供了很多种方法，上面的代码中基本上囊括了大部分的数据库操作；对于添加、更新和删除来说，我们都可以使用

```
db.executeSQL(String sql);  
db.executeSQL(String sql, Object[] bindArgs);//sql语句中使用占位符，然后第二个参数是实际的参数集 
```

除了统一的形式之外，他们还有各自的操作方法：

```
db.insert(String table, String nullColumnHack, ContentValues values);  
db.update(String table, Contentvalues values, String whereClause, String whereArgs);  
db.delete(String table, String whereClause, String whereArgs);
```

说明：

- 以上三个方法的第一个参数都是表示要操作的表名；
- insert中的第二个参数表示如果插入的数据每一列都为空的话，需要指定此行中某一列的名称，系统将此列设置为NULL，不至于出现错误；
- insert中的第三个参数是ContentValues类型的变量，是键值对组成的Map，key代表列名，value代表该列要插入的值；
- update的第二个参数也很类似，只不过它是更新该字段key为最新的value值，第三个参数whereClause表示WHERE表达式，比如`age ? and age < ?`等，最后的whereArgs参数是占位符的实际参数值；
- delete方法的参数也是一样

**数据的添加**

```
//使用insert方法
ContentValues cv = new ContentValues();//实例化一个ContentValues用来装载待插入的数据
cv.put("title","you are beautiful");//添加title
cv.put("weather","sun"); //添加weather
cv.put("context","xxxx"); //添加context
String publish = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
                        .format(new Date());
cv.put("publish ",publish); //添加publish
db.insert("diary",null,cv);//执行插入操作

//使用execSQL方式来实现
String sql = "insert into user(username,password) values ('Jack Johnson','iLovePopMuisc');//插入操作的SQL语句
db.execSQL(sql);//执行SQL语句
```

**数据的删除**

```
String whereClause = "username=?";//删除的条件
String[] whereArgs = {"Jack Johnson"};//删除的条件参数
db.delete("user",whereClause,whereArgs);//执行删除

//使用execSQL方式来实现
String sql = "delete from user where username='Jack Johnson'";//删除操作的SQL语句
db.execSQL(sql);//执行删除操作
```

**数据修改**

```
ContentValues cv = new ContentValues();//实例化ContentValues
cv.put("password","iHatePopMusic");//添加要更改的字段及内容
String whereClause = "username=?";//修改条件
String[] whereArgs = {"Jack Johnson"};//修改条件的参数
db.update("user",cv,whereClause,whereArgs);//执行修改
//使用execSQL方式来实现

//使用execSQL方式来实现
String sql = "update user set password = 'iHatePopMusic' where username='Jack Johnson'";//修改的SQL语句
db.execSQL(sql);//执行修改
```

**数据查询**

下面来说说查询操作。查询操作相对于上面的几种操作要复杂些，因为我们经常要面对着各种各样的查询条件，所以系统也考虑到这种复杂性，为我们提供了较为丰富的查询形式：

```
db.rawQuery(String sql, String[] selectionArgs);  
db.query(String table, String[] columns, String selection, String[] selectionArgs, String groupBy, String having, String orderBy);  
db.query(String table, String[] columns, String selection, String[] selectionArgs, String groupBy, String having, String orderBy, String limit);  
db.query(String distinct, String table, String[] columns, String selection, String[] selectionArgs, String groupBy, String having, String orderBy, String limit);
```

上面几种都是常用的查询方法，第一种最为简单，将所有的SQL语句都组织到一个字符串中，使用占位符代替实际参数，selectionArgs就是占位符实际参数集；

各参数说明：

- table：表名称
- colums：表示要查询的列所有名称集
- selection：表示WHERE之后的条件语句，可以使用占位符
- selectionArgs：条件语句的参数数组
- groupBy：指定分组的列名
- having：指定分组条件,配合groupBy使用
- orderBy：y指定排序的列名
- limit：指定分页参数
- distinct：指定“true”或“false”表示要不要过滤重复值
- Cursor：返回值，相当于结果集ResultSet

最后，他们同时返回一个Cursor对象，代表数据集的游标，有点类似于JavaSE中的ResultSet。下面是Cursor对象的常用方法：

```
c.move(int offset); //以当前位置为参考,移动到指定行  
c.moveToFirst();    //移动到第一行  
c.moveToLast();     //移动到最后一行  
c.moveToPosition(int position); //移动到指定行  
c.moveToPrevious(); //移动到前一行  
c.moveToNext();     //移动到下一行  
c.isFirst();        //是否指向第一条  
c.isLast();     //是否指向最后一条  
c.isBeforeFirst();  //是否指向第一条之前  
c.isAfterLast();    //是否指向最后一条之后  
c.isNull(int columnIndex);  //指定列是否为空(列基数为0)  
c.isClosed();       //游标是否已关闭  
c.getCount();       //总数据项数  
c.getPosition();    //返回当前游标所指向的行数  
c.getColumnIndex(String columnName);//返回某列名对应的列索引值  
c.getString(int columnIndex);   //返回当前行指定列的值
```

实现代码

```
String[] params =  {12345,123456};
Cursor cursor = db.query("user",columns,"ID=?",params,null,null,null);//查询并获得游标
if(cursor.moveToFirst()){//判断游标是否为空
    for(int i=0;i<cursor.getCount();i++){
        cursor.move(i);//移动到指定记录
        String username = cursor.getString(cursor.getColumnIndex("username");
        String password = cursor.getString(cursor.getColumnIndex("password"));
    }
}
```

通过rawQuery实现的带参数查询

```
Cursor result=db.rawQuery("SELECT ID, name, inventory FROM mytable");
//Cursor c = db.rawQuery("s name, inventory FROM mytable where ID=?",new Stirng[]{"123456"});     
result.moveToFirst(); 
while (!result.isAfterLast()) { 
    int id=result.getInt(0); 
    String name=result.getString(1); 
    int inventory=result.getInt(2); 
    // do something useful with these 
    result.moveToNext(); 
 } 
 result.close();
```

在上面的代码示例中，已经用到了这几个常用方法中的一些，关于更多的信息，大家可以参考官方文档中的说明。

最后当我们完成了对数据库的操作后，记得调用SQLiteDatabase的close()方法释放数据库连接，否则容易出现SQLiteException。

上面就是SQLite的基本应用，但在实际开发中，为了能够更好的管理和维护数据库，我们会封装一个继承自SQLiteOpenHelper类的数据库操作类，然后以这个类为基础，再封装我们的业务逻辑方法。

这里直接使用案例讲解：下面是案例demo的界面

![2.png](https://i.loli.net/2019/05/14/5cda5f11e83da73553.png)

#### SQLiteOpenHelper类介绍

SQLiteOpenHelper是SQLiteDatabase的一个帮助类，用来管理数据库的创建和版本的更新。一般是建立一个类继承它，并实现它的onCreate和onUpgrade方法。

| 方法名                                                       | 方法描述                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **SQLiteOpenHelper(Context context,String name,SQLiteDatabase.CursorFactory factory,int version)** | 构造方法，其中 context 程序上下文环境 即：XXXActivity.this; name :数据库名字; factory:游标工厂，默认为null,即为使用默认工厂; version 数据库版本号 |
| **onCreate(SQLiteDatabase db)**                              | 创建数据库时调用                                             |
| **onUpgrade(SQLiteDatabase db,int oldVersion , int newVersion)** | 版本更新时调用                                               |
| **getReadableDatabase()**                                    | 创建或打开一个只读数据库                                     |
| **getWritableDatabase()**                                    | 创建或打开一个读写数据库                                     |

首先创建数据库类

```
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteDatabase.CursorFactory;
import android.database.sqlite.SQLiteOpenHelper;

public class SqliteDBHelper extends SQLiteOpenHelper {

    // 步骤1：设置常数参量
    private static final String DATABASE_NAME = "diary_db";
    private static final int VERSION = 1;
    private static final String TABLE_NAME = "diary";

    // 步骤2：重载构造方法
    public SqliteDBHelper(Context context) {
        super(context, DATABASE_NAME, null, VERSION);
    }

    /*
     * 参数介绍：context 程序上下文环境 即：XXXActivity.this 
     * name 数据库名字 
     * factory 接收数据，一般情况为null
     * version 数据库版本号
     */
    public SqliteDBHelper(Context context, String name, CursorFactory factory,
            int version) {
        super(context, name, factory, version);
    }
    //数据库第一次被创建时，onCreate()会被调用
    @Override
    public void onCreate(SQLiteDatabase db) {
        // 步骤3：数据库表的创建
        String strSQL = "create table "
                + TABLE_NAME
                + "(tid integer primary key autoincrement,title varchar(20),weather varchar(10),context text,publish date)";
        //步骤4：使用参数db,创建对象
        db.execSQL(strSQL);
    }
    //数据库版本变化时，会调用onUpgrade()
    @Override
    public void onUpgrade(SQLiteDatabase arg0, int arg1, int arg2) {

    }
}
```

正如上面所述，数据库第一次创建时onCreate方法会被调用，我们可以执行创建表的语句，当系统发现版本变化之后，会调用onUpgrade方法，我们可以执行修改表结构等语句。

 我们需要一个Dao，来封装我们所有的业务方法，代码如下：

```
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

import com.chinasoft.dbhelper.SqliteDBHelper;

public class DiaryDao {

    private SqliteDBHelper sqliteDBHelper;
    private SQLiteDatabase db;

    // 重写构造方法
    public DiaryDao(Context context) {
        this.sqliteDBHelper = new SqliteDBHelper(context);
        db = sqliteDBHelper.getWritableDatabase();
    }

    // 读操作
    public String execQuery(final String strSQL) {
        try {
            System.out.println("strSQL>" + strSQL);
            // Cursor相当于JDBC中的ResultSet
            Cursor cursor = db.rawQuery(strSQL, null);
            // 始终让cursor指向数据库表的第1行记录
            cursor.moveToFirst();
            // 定义一个StringBuffer的对象，用于动态拼接字符串
            StringBuffer sb = new StringBuffer();
            // 循环游标，如果不是最后一项记录
            while (!cursor.isAfterLast()) {
                sb.append(cursor.getInt(0) + "/" + cursor.getString(1) + "/"
                        + cursor.getString(2) + "/" + cursor.getString(3) + "/"
                        + cursor.getString(4)+"#");
                //cursor游标移动
                cursor.moveToNext();
            }
            db.close();
            return sb.deleteCharAt(sb.length()-1).toString();
        } catch (RuntimeException e) {
            e.printStackTrace();
            return null;
        }

    }

    // 写操作
    public boolean execOther(final String strSQL) {
        db.beginTransaction();  //开始事务
        try {
            System.out.println("strSQL" + strSQL);
            db.execSQL(strSQL);
            db.setTransactionSuccessful();  //设置事务成功完成 
            db.close();
            return true;
        } catch (RuntimeException e) {
            e.printStackTrace();
            return false;
        }finally {  
            db.endTransaction();    //结束事务  
        }  

    }
}
```

我们在Dao构造方法中实例化sqliteDBHelper并获取一个SQLiteDatabase对象，作为整个应用的数据库实例；在增删改信息时，我们采用了事务处理，确保数据完整性；最后要注意释放数据库资源db.close()，这一个步骤在我们整个应用关闭时执行，这个环节容易被忘记，所以朋友们要注意。

我们获取数据库实例时使用了getWritableDatabase()方法，也许朋友们会有疑问，在getWritableDatabase()和getReadableDatabase()中，你为什么选择前者作为整个应用的数据库实例呢？在这里我想和大家着重分析一下这一点。

我们来看一下SQLiteOpenHelper中的getReadableDatabase()方法：

```
public synchronized SQLiteDatabase getReadableDatabase() {  
    if (mDatabase != null && mDatabase.isOpen()) {  
        // 如果发现mDatabase不为空并且已经打开则直接返回  
        return mDatabase;  
    }  
  
    if (mIsInitializing) {  
        // 如果正在初始化则抛出异常  
        throw new IllegalStateException("getReadableDatabase called recursively");  
    }  
  
    // 开始实例化数据库mDatabase  
  
    try {  
        // 注意这里是调用了getWritableDatabase()方法  
        return getWritableDatabase();  
    } catch (SQLiteException e) {  
        if (mName == null)  
            throw e; // Can't open a temp database read-only!  
        Log.e(TAG, "Couldn't open " + mName + " for writing (will try read-only):", e);  
    }  
  
    // 如果无法以可读写模式打开数据库 则以只读方式打开  
  
    SQLiteDatabase db = null;  
    try {  
        mIsInitializing = true;  
        String path = mContext.getDatabasePath(mName).getPath();// 获取数据库路径  
        // 以只读方式打开数据库  
        db = SQLiteDatabase.openDatabase(path, mFactory, SQLiteDatabase.OPEN_READONLY);  
        if (db.getVersion() != mNewVersion) {  
            throw new SQLiteException("Can't upgrade read-only database from version " + db.getVersion() + " to "  
                    + mNewVersion + ": " + path);  
        }  
  
        onOpen(db);  
        Log.w(TAG, "Opened " + mName + " in read-only mode");  
        mDatabase = db;// 为mDatabase指定新打开的数据库  
        return mDatabase;// 返回打开的数据库  
    } finally {  
        mIsInitializing = false;  
        if (db != null && db != mDatabase)  
            db.close();  
    }  
}
```

在getReadableDatabase()方法中，首先判断是否已存在数据库实例并且是打开状态，如果是，则直接返回该实例，否则试图获取一个可读写模式的数据库实例，如果遇到磁盘空间已满等情况获取失败的话，再以只读模式打开数据库，获取数据库实例并返回，然后为mDatabase赋值为最新打开的数据库实例。既然有可能调用到getWritableDatabase()方法，我们就要看一下了：

```
public synchronized SQLiteDatabase getWritableDatabase() {  
    if (mDatabase != null && mDatabase.isOpen() && !mDatabase.isReadOnly()) {  
        // 如果mDatabase不为空已打开并且不是只读模式 则返回该实例  
        return mDatabase;  
    }  
  
    if (mIsInitializing) {  
        throw new IllegalStateException("getWritableDatabase called recursively");  
    }  
  
    // If we have a read-only database open, someone could be using it  
    // (though they shouldn't), which would cause a lock to be held on  
    // the file, and our attempts to open the database read-write would  
    // fail waiting for the file lock. To prevent that, we acquire the  
    // lock on the read-only database, which shuts out other users.  
  
    boolean success = false;  
    SQLiteDatabase db = null;  
    // 如果mDatabase不为空则加锁 阻止其他的操作  
    if (mDatabase != null)  
        mDatabase.lock();  
    try {  
        mIsInitializing = true;  
        if (mName == null) {  
            db = SQLiteDatabase.create(null);  
        } else {  
            // 打开或创建数据库  
            db = mContext.openOrCreateDatabase(mName, 0, mFactory);  
        }  
        // 获取数据库版本(如果刚创建的数据库,版本为0)  
        int version = db.getVersion();  
        // 比较版本(我们代码中的版本mNewVersion为1)  
        if (version != mNewVersion) {  
            db.beginTransaction();// 开始事务  
            try {  
                if (version == 0) {  
                    // 执行我们的onCreate方法  
                    onCreate(db);  
                } else {  
                    // 如果我们应用升级了mNewVersion为2,而原版本为1则执行onUpgrade方法  
                    onUpgrade(db, version, mNewVersion);  
                }  
                db.setVersion(mNewVersion);// 设置最新版本  
                db.setTransactionSuccessful();// 设置事务成功  
            } finally {  
                db.endTransaction();// 结束事务  
            }  
        }  
  
        onOpen(db);  
        success = true;  
        return db;// 返回可读写模式的数据库实例  
    } finally {  
        mIsInitializing = false;  
        if (success) {  
            // 打开成功  
            if (mDatabase != null) {  
                // 如果mDatabase有值则先关闭  
                try {  
                    mDatabase.close();  
                } catch (Exception e) {  
                }  
                mDatabase.unlock();// 解锁  
            }  
            mDatabase = db;// 赋值给mDatabase  
        } else {  
            // 打开失败的情况：解锁、关闭  
            if (mDatabase != null)  
                mDatabase.unlock();  
            if (db != null)  
                db.close();  
        }  
    }  
}
```

大家可以看到，几个关键步骤是，首先判断mDatabase如果不为空已打开并不是只读模式则直接返回，否则如果mDatabase不为空则加锁，然后开始打开或创建数据库，比较版本，根据版本号来调用相应的方法，为数据库设置新版本号，最后释放旧的不为空的mDatabase并解锁，把新打开的数据库实例赋予mDatabase，并返回最新实例。

看完上面的过程之后，大家或许就清楚了许多，如果不是在遇到磁盘空间已满等情况，getReadableDatabase()一般都会返回和getWritableDatabase()一样的数据库实例，所以我们在DBManager构造方法中使用getWritableDatabase()获取整个应用所使用的数据库实例是可行的。当然如果你真的担心这种情况会发生，那么你可以先用getWritableDatabase()获取数据实例，如果遇到异常，再试图用getReadableDatabase()获取实例，当然这个时候你获取的实例只能读不能写了

最后，让我们看一下如何使用这些数据操作方法来显示数据,界面核心逻辑代码:

```
public class SQLiteActivity extends Activity {

    public DiaryDao diaryDao;

    //因为getWritableDatabase内部调用了mContext.openOrCreateDatabase(mName, 0, mFactory);  
    //所以要确保context已初始化,我们可以把实例化Dao的步骤放在Activity的onCreate里
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        diaryDao = new DiaryDao(SQLiteActivity.this);
        initDatabase();
    }

    class ViewOcl implements View.OnClickListener {

        @Override
        public void onClick(View v) {

            String strSQL;
            boolean flag;
            String message;
            switch (v.getId()) {
            case R.id.btnAdd:
                String title = txtTitle.getText().toString().trim();
                String weather = txtWeather.getText().toString().trim();;
                String context = txtContext.getText().toString().trim();;
                String publish = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
                        .format(new Date());
                // 动态组件SQL语句
                strSQL = "insert into diary values(null,'" + title + "','"
                        + weather + "','" + context + "','" + publish + "')";
                flag = diaryDao.execOther(strSQL);
                //返回信息
                message = flag?"添加成功":"添加失败";
                Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
                break;
            case R.id.btnDelete:
                strSQL = "delete from diary where tid = 1";
                flag = diaryDao.execOther(strSQL);
                //返回信息
                message = flag?"删除成功":"删除失败";
                Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
                break;
            case R.id.btnQuery:
                strSQL = "select * from diary order by publish desc";
                String data = diaryDao.execQuery(strSQL);
                Toast.makeText(getApplicationContext(), data, Toast.LENGTH_LONG).show();
                break;
            case R.id.btnUpdate:
                strSQL = "update diary set title = '测试标题1-1' where tid = 1";
                flag = diaryDao.execOther(strSQL);
                //返回信息
                message = flag?"更新成功":"更新失败";
                Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
                break;
            }
        }
    }

    private void initDatabase() {
        // 创建数据库对象
        SqliteDBHelper sqliteDBHelper = new SqliteDBHelper(SQLiteActivity.this);
        sqliteDBHelper.getWritableDatabase();
        System.out.println("数据库创建成功");
    }
}
```

###  使用ContentProvider存储数据

ContentProvider（内容提供者）是Android的四大组件之一，管理android以结构化方式存放的数据，以相对安全的方式封装数据（表）并且提供简易的处理机制和统一的访问接口供其他程序调用。　 
　　 
Android的数据存储方式总共有五种，分别是：Shared Preferences、网络存储、文件存储、外储存储、SQLite。但一般这些存储都只是在单独的一个应用程序之中达到一个数据的共享，有时候我们需要操作其他应用程序的一些数据，就会用到ContentProvider。而且Android为常见的一些数据提供了默认的ContentProvider（包括音频、视频、图片和通讯录等）。

但注意ContentProvider它也只是一个中间人，真正操作的数据源可能是数据库，也可以是文件、xml或网络等其他存储方式。

URL（统一资源标识符）代表要操作的数据，可以用来标识每个ContentProvider，这样你就可以通过指定的URI找到想要的ContentProvider,从中获取或修改数据。 

在Android中URI的格式如下图所示：

![3.jpeg](https://i.loli.net/2019/05/14/5cda615018cbc18008.jpeg)

Ａ ：schema，已经由Android所规定为：content://．　 

Ｂ：主机名（Authority），是URI的授权部分，是唯一标识符，用来定位ContentProvider。Ｃ部分和D部分：是每个ContentProvider内部的路径部分

Ｃ：指向一个对象集合，一般用表的名字，如果没有指定D部分，则返回全部记录。

Ｄ：指向特定的记录，这里表示操作user表id为7的记录。如果要操作user表中id为7的记录的name字段， D部分变为 /7/name即可。

URI模式匹配通配符

```
*：匹配的任意长度的任何有效字符的字符串。

＃：匹配的任意长度的数字字符的字符串。
```

如：

```
content://com.example.app.provider/* 
匹配provider的任何内容url

content://com.example.app.provider/table3/# 
匹配table3的所有行
```

MIME是指定某个扩展名的文件用一种应用程序来打开，就像你用浏览器查看PDF格式的文件，浏览器会选择合适的应用来打开一样。Android中的工作方式跟HTTP类似，ContentProvider会根据URI来返回MIME类型，ContentProvider会返回一个包含两部分的字符串。MIME类型一般包含两部分，如：

```
 text/html 
text/css 
text/xml 
application/pdf
```

分为类型和子类型，Android遵循类似的约定来定义MIME类型，每个内容类型的Android MIME类型有两种形式：多条记录（集合）和单条记录。

```
vnd.android.cursor.dir/自定义

vnd.android.cursor.item/自定义
```

Android中类型已经固定好了，不能更改，只能区别是集合还是单条具体记录，子类型可以按照格式自己填写。 
在使用Intent时，会用到MIME，根据Mimetype打开符合条件的活动。

下面分别介绍Android系统提供了两个用于操作Uri的工具类：ContentUris和UriMatcher。

#### ContentUris

ContetnUris包含一个便利的函数withAppendedId()来向URI追加一个id。

```
Uri uri = Uri.parse("content://cn.scu.myprovider/user")
Uri resultUri = ContentUris.withAppendedId(uri, 7); 
 
//生成后的Uri为：content://cn.scu.myprovider/user/7
```

同时提供parseId(uri)方法用于从URL中获取ID:

```
Uri uri = Uri.parse("content://cn.scu.myprovider/user/7")
long personid = ContentUris.parseId(uri);
//获取的结果为:7
```

#### UriMatcher

UriMatcher本质上是一个文本过滤器，用在contentProvider中帮助我们过滤，分辨出查询者想要查询哪个数据表。

 第一步，初始化：

```
UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
//常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码
```

第二步，注册需要的Uri：

```
//USER 和 USER_ID是两个int型数据
matcher.addURI("cn.scu.myprovider", "user", USER);
matcher.addURI("cn.scu.myprovider", "user/#",USER_ID);
//如果match()方法匹配content://cn.scu.myprovider/user路径，返回匹配码为USER
```

第三步，与已经注册的Uri进行匹配:

```
/* 
     * 如果操作集合，则必须以vnd.android.cursor.dir开头 
     * 如果操作非集合，则必须以vnd.android.cursor.item开头 
     * */  
    @Override  
    public String getType(Uri uri) {  
    Uri uri = Uri.parse("content://" + "cn.scu.myprovider" + "/user");  
        switch(matcher.match(uri)){  
        case USER:  
            return "vnd.android.cursor.dir/user";  
        case USER_ID:  
            return "vnd.android.cursor.item/user";  
        }  
    } 
```

#### ContentProvider的主要方法

```
public boolean onCreate()//　　ContentProvider创建后　或　打开系统后其它应用第一次访问该ContentProvider时调用。
 public Uri insert(Uri uri, ContentValues values)//　　外部应用向ContentProvider中添加数据。
 public int delete(Uri uri, String selection, String[] selectionArgs)//外部应用从ContentProvider删除数据。
 public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)//　　外部应用更新ContentProvider中的数据。
 public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)　//供外部应用从ContentProvider中获取数据。
 public String getType(Uri uri)//　　该方法用于返回当前Url所代表数据的MIME类型。
```

#### ContentResolver

ContentResolver通过URI来查询ContentProvider中提供的数据。除了URI以 外，还必须知道需要获取的数据段的名称，以及此数据段的数据类型。如果你需要获取一个特定的记录，你就必须知道当前记录的ID，也就是URI中D部分。

ContentResolver 类提供了与ContentProvider类相同签名的四个方法：

```
 public Uri insert(Uri uri, ContentValues values)　//添加

public int delete(Uri uri, String selection, String[] selectionArgs)　//删除

public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)　//更新

public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)//获取

```

示例：

```
ContentResolver resolver =  getContentResolver();
Uri uri = Uri.parse("content://cn.scu.myprovider/user");
 
//添加一条记录
ContentValues values = new ContentValues();
values.put("name", "fanrunqi");
values.put("age", 24);
resolver.insert(uri, values);  
 
//获取user表中所有记录
Cursor cursor = resolver.query(uri, null, null, null, "userid desc");
while(cursor.moveToNext()){
   //操作
}
 
//把id为1的记录的name字段值更改新为finch
ContentValues updateValues = new ContentValues();
updateValues.put("name", "finch");
Uri updateIdUri = ContentUris.withAppendedId(uri, 1);
resolver.update(updateIdUri, updateValues, null, null);
 
//删除id为2的记录
Uri deleteIdUri = ContentUris.withAppendedId(uri, 2);
resolver.delete(deleteIdUri, null, null);
```

#### ContentObserver

ContentObserver(内容观察者)，目的是观察特定Uri引起的数据库的变化，继而做一些相应的处理，它类似于数据库技术中的触发器(Trigger)，当ContentObserver所观察的Uri发生变化时，便会触发它.

下面是使用内容观察者监听短信的例子：

```
public class MainActivity extends Activity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
//注册观察者Observser    
this.getContentResolver().registerContentObserver(Uri.parse("content://sms"),true,new SMSObserver(new Handler()));
 
    }
 
    private final class SMSObserver extends ContentObserver {
 
        public SMSObserver(Handler handler) {
            super(handler);
 
        }
 
 
        @Override
        public void onChange(boolean selfChange) {
 
 Cursor cursor = MainActivity.this.getContentResolver().query(
Uri.parse("content://sms/inbox"), null, null, null, null);
 
            while (cursor.moveToNext()) {
                StringBuilder sb = new StringBuilder();
 
                sb.append("address=").append(
                        cursor.getString(cursor.getColumnIndex("address")));
 
                sb.append(";subject=").append(
                        cursor.getString(cursor.getColumnIndex("subject")));
 
                sb.append(";body=").append(
                        cursor.getString(cursor.getColumnIndex("body")));
 
                sb.append(";time=").append(
                        cursor.getLong(cursor.getColumnIndex("date")));
 
                System.out.println("--------has Receivered SMS::" + sb.toString());
 
 
            }
 
        }
 
    }
}

```

同时可以在ContentProvider发生数据变化时调用 
getContentResolver().notifyChange(uri, null)来通知注册在此URI上的访问者

```
public class UserContentProvider extends ContentProvider {
   public Uri insert(Uri uri, ContentValues values) {
      db.insert("user", "userid", values);
      getContext().getContentResolver().notifyChange(uri, null);
   }
}

```

#### 实例

数据源是SQLite, 用ContentResolver操作ContentProvider。

![3.png](https://i.loli.net/2019/05/14/5cda6392c7f8590403.png)

Constant.java（储存一些常量）

```
public class Constant {  
 
    public static final String TABLE_NAME = "user";  
 
    public static final String COLUMN_ID = "_id";  
    public static final String COLUMN_NAME = "name";  
 
 
    public static final String AUTOHORITY = "cn.scu.myprovider";  
    public static final int ITEM = 1;  
    public static final int ITEM_ID = 2;  
 
    public static final String CONTENT_TYPE = "vnd.android.cursor.dir/user";  
    public static final String CONTENT_ITEM_TYPE = "vnd.android.cursor.item/user";  
 
    public static final Uri CONTENT_URI = Uri.parse("content://" + AUTOHORITY + "/user");  
}  
```

DBHelper.java(操作数据库)

```
public class DBHelper extends SQLiteOpenHelper {  
 
    private static final String DATABASE_NAME = "finch.db";    
    private static final int DATABASE_VERSION = 1;    
 
    public DBHelper(Context context) {  
        super(context, DATABASE_NAME, null, DATABASE_VERSION);  
    }  
 
    @Override  
    public void onCreate(SQLiteDatabase db)  throws SQLException {  
        //创建表格  
        db.execSQL("CREATE TABLE IF NOT EXISTS "+ Constant.TABLE_NAME + "("+ Constant.COLUMN_ID +" INTEGER PRIMARY KEY AUTOINCREMENT," + Constant.COLUMN_NAME +" VARCHAR NOT NULL);");  
    }  
 
    @Override  
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)  throws SQLException {  
        //删除并创建表格  
        db.execSQL("DROP TABLE IF EXISTS "+ Constant.TABLE_NAME+";");  
        onCreate(db);  
    }  
}  
```

MyProvider.java(自定义的ContentProvider)　

```
public class MyProvider extends ContentProvider {    
 
    DBHelper mDbHelper = null;    
    SQLiteDatabase db = null;    
 
    private static final UriMatcher mMatcher;    
    static{    
        mMatcher = new UriMatcher(UriMatcher.NO_MATCH);    
        mMatcher.addURI(Constant.AUTOHORITY,Constant.TABLE_NAME, Constant.ITEM);    
        mMatcher.addURI(Constant.AUTOHORITY, Constant.TABLE_NAME+"/#", Constant.ITEM_ID);    
    }    
 
 
    @Override    
    public String getType(Uri uri) {    
        switch (mMatcher.match(uri)) {    
        case Constant.ITEM:    
            return Constant.CONTENT_TYPE;    
        case Constant.ITEM_ID:    
            return Constant.CONTENT_ITEM_TYPE;    
        default:    
            throw new IllegalArgumentException("Unknown URI"+uri);    
        }    
    }    
 
    @Override    
    public Uri insert(Uri uri, ContentValues values) {    
        // TODO Auto-generated method stub    
        long rowId;    
        if(mMatcher.match(uri)!=Constant.ITEM){    
            throw new IllegalArgumentException("Unknown URI"+uri);    
        }    
        rowId = db.insert(Constant.TABLE_NAME,null,values);    
        if(rowId>0){    
            Uri noteUri=ContentUris.withAppendedId(Constant.CONTENT_URI, rowId);    
            getContext().getContentResolver().notifyChange(noteUri, null);    
            return noteUri;    
        }    
 
        throw new SQLException("Failed to insert row into " + uri);    
    }    
 
    @Override    
    public boolean onCreate() {    
        // TODO Auto-generated method stub    
        mDbHelper = new DBHelper(getContext());    
 
        db = mDbHelper.getReadableDatabase();    
 
        return true;    
    }    
 
    @Override    
    public Cursor query(Uri uri, String[] projection, String selection,    
            String[] selectionArgs, String sortOrder) {    
        // TODO Auto-generated method stub    
        Cursor c = null;    
        switch (mMatcher.match(uri)) {    
        case Constant.ITEM:    
            c =  db.query(Constant.TABLE_NAME, projection, selection, selectionArgs, null, null, sortOrder);    
            break;    
        case Constant.ITEM_ID:    
            c = db.query(Constant.TABLE_NAME, projection,Constant.COLUMN_ID + "="+uri.getLastPathSegment(), selectionArgs, null, null, sortOrder);    
            break;    
        default:    
            throw new IllegalArgumentException("Unknown URI"+uri);    
        }    
 
        c.setNotificationUri(getContext().getContentResolver(), uri);    
        return c;    
    }    
 
    @Override    
    public int update(Uri uri, ContentValues values, String selection,    
            String[] selectionArgs) {    
        // TODO Auto-generated method stub    
        return 0;    
    }
 
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // TODO Auto-generated method stub
        return 0;
    }    
 
}    
```

MainActivity.java(ContentResolver操作)

```
public class MainActivity extends Activity {
    private ContentResolver mContentResolver = null; 
    private Cursor cursor = null;  
         @Override
        protected void onCreate(Bundle savedInstanceState) {
            // TODO Auto-generated method stub
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
 
               TextView tv = (TextView) findViewById(R.id.tv);
 
                mContentResolver = getContentResolver();  
                tv.setText("添加初始数据 ");
                for (int i = 0; i < 10; i++) {  
                    ContentValues values = new ContentValues();  
                    values.put(Constant.COLUMN_NAME, "fanrunqi"+i);  
                    mContentResolver.insert(Constant.CONTENT_URI, values);  
                } 
 
                tv.setText("查询数据 ");
                cursor = mContentResolver.query(Constant.CONTENT_URI, new String[]{Constant.COLUMN_ID,Constant.COLUMN_NAME}, null, null, null);  
                if (cursor.moveToFirst()) {
                    String s = cursor.getString(cursor.getColumnIndex(Constant.COLUMN_NAME));
                    tv.setText("第一个数据： "+s);
                }
        }
 
}  
```

### 网络存储数据

####  HttpUrlConnection

HttpUrlConnection是Java.net包中提供的API，我们知道Android SDK是基于Java的，所以当然优先考虑HttpUrlConnection这种最原始最基本的API，其实大多数开源的联网框架基本上也是基于JDK的HttpUrlConnection进行的封装罢了，掌握HttpUrlConnection需要以下几个步骤：

1. 将访问的路径转换成URL。

    URL url = new URL(path);

2. 通过URL获取连接。

    HttpURLConnection conn = (HttpURLConnection) url.openConnection();

3. 设置请求方式。

    conn.setRequestMethod(GET);

4. 设置连接超时时间。

    conn.setConnectTimeout(5000);

5. 设置请求头的信息。

    conn.setRequestProperty(User-Agent, Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0));

7. 针对不同的响应码，做不同的操作（请求码200，表明请求成功，获取返回内容的输入流）

工具类：

```
public class StreamTools {
    /**
     * 将输入流转换成字符串
     * 
     * @param is
     *            从网络获取的输入流
     * @return
     */
    public static String streamToString(InputStream is) {
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int len = 0;
            while ((len = is.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
            baos.close();
            is.close();
            byte[] byteArray = baos.toByteArray();
            return new String(byteArray);
        } catch (Exception e) {
            Log.e(tag, e.toString());
            return null;
        }
    }
}
```

 HttpUrlConnection发送GET请求

```
public static String loginByGet(String username, String password) {
        String path = http://192.168.0.107:8080/WebTest/LoginServerlet?username= + username + &password= + password;
        try {
            URL url = new URL(path);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setConnectTimeout(5000);
            conn.setRequestMethod(GET);
            int code = conn.getResponseCode();
            if (code == 200) {
                InputStream is = conn.getInputStream(); // 字节流转换成字符串
                return StreamTools.streamToString(is);
            } else {
                return 网络访问失败;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return 网络访问失败;
        }
    }
```

HttpUrlConnection发送POST请求

```
public static String loginByPost(String username, String password) {
        String path = http://192.168.0.107:8080/WebTest/LoginServerlet;
        try {
            URL url = new URL(path);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setConnectTimeout(5000);
            conn.setRequestMethod(POST);
            conn.setRequestProperty(Content-Type, application/x-www-form-urlencoded);
            String data = username= + username + &password= + password;
            conn.setRequestProperty(Content-Length, data.length() + );
            // POST方式，其实就是浏览器把数据写给服务器
            conn.setDoOutput(true); // 设置可输出流
            OutputStream os = conn.getOutputStream(); // 获取输出流
            os.write(data.getBytes()); // 将数据写给服务器
            int code = conn.getResponseCode();
            if (code == 200) {
                InputStream is = conn.getInputStream();
                return StreamTools.streamToString(is);
            } else {
                return 网络访问失败;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return 网络访问失败;
        }
    }
```

####  HttpClient

HttpClient是开源组织Apache提供的Java请求网络框架，其最早是为了方便Java服务器开发而诞生的，是对JDK中的HttpUrlConnection各API进行了封装和简化，提高了性能并且降低了调用API的繁琐，Android因此也引进了这个联网框架，我们再不需要导入任何jar或者类库就可以直接使用，值得注意的是Android官方已经宣布不建议使用HttpClient了。

**HttpClient发送GET请求**

1. 创建HttpClient对象

2. 创建HttpGet对象，指定请求地址（带参数）

3. 使用HttpClient的execute(),方法执行HttpGet请求，得到HttpResponse对象

4. 调用HttpResponse的getStatusLine().getStatusCode()方法得到响应码

5. 调用的HttpResponse的getEntity().getContent()得到输入流，获取服务端写回的数据

```
public static String loginByHttpClientGet(String username, String password) {
        String path = http://192.168.0.107:8080/WebTest/LoginServerlet?username=
                + username + &password= + password;
        HttpClient client = new DefaultHttpClient(); // 开启网络访问客户端
        HttpGet httpGet = new HttpGet(path); // 包装一个GET请求
        try {
            HttpResponse response = client.execute(httpGet); // 客户端执行请求
            int code = response.getStatusLine().getStatusCode(); // 获取响应码
            if (code == 200) {
                InputStream is = response.getEntity().getContent(); // 获取实体内容
                String result = StreamTools.streamToString(is); // 字节流转字符串
                return result;
            } else {
                return 网络访问失败;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return 网络访问失败;
        }
    }
```

**HttpClient发送POST请求**

1. 创建HttpClient对象

2. 创建HttpPost对象，指定请求地址

3. 创建List，用来装载参数

4. 调用HttpPost对象的setEntity()方法，装入一个UrlEncodedFormEntity对象，携带之前封装好的参数

5. 使用HttpClient的execute()方法执行HttpPost请求，得到HttpResponse对象

6.  调用HttpResponse的getStatusLine().getStatusCode()方法得到响应码

7.  调用的HttpResponse的getEntity().getContent()得到输入流，获取服务端写回的数据

```
public static String loginByHttpClientPOST(String username, String password) {
        String path = http://192.168.0.107:8080/WebTest/LoginServerlet;
        try {
            HttpClient client = new DefaultHttpClient(); // 建立一个客户端
            HttpPost httpPost = new HttpPost(path); // 包装POST请求
            // 设置发送的实体参数
            List parameters = new ArrayList();
            parameters.add(new BasicNameValuePair(username, username));
            parameters.add(new BasicNameValuePair(password, password));
            httpPost.setEntity(new UrlEncodedFormEntity(parameters, UTF-8));
            HttpResponse response = client.execute(httpPost); // 执行POST请求
            int code = response.getStatusLine().getStatusCode();
            if (code == 200) {
                InputStream is = response.getEntity().getContent();
                String result = StreamTools.streamToString(is);
                return result;
            } else {
                return 网络访问失败;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return 访问网络失败;
        }
    }
```

 参考：

 [Android开发请求网络方式详解](http://www.2cto.com/kf/201501/368943.html)

####  Android提供的其他网络访问框架

HttpClient和HttpUrlConnection的两种网络访问方式编写网络代码，需要自己考虑很多，获取数据或许可以，但是如果要将手机本地数据上传至网络，根据不同的web端接口，需要组织不同的数据内容上传，给手机端造成了很大的工作量。 
　　 
目前有几种快捷的网络开发开源框架，给我们提供了非常大的便利。下面是这些项目Github地址，有文档和Api说明。 

-  [android-async-http](https://github.com/loopj/android-async-http)　
-  [http-request](https://github.com/kevinsawicki/http-request)
-  [okhttp](https://github.com/square/okhttp)

