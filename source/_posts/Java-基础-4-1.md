---
title: Java Swing程序设计（一）
date: 2019-04-05 21:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
早期的AWT（抽象窗口工具包）组件开发的图形用户界面，要依赖本地系统，当把AWT组件开发的应用程序移植到其他平台的系统上运行时，不能保证其外观风格，因此AWT是依赖于本地系统平台的。而使用Swing开发的Java应用程序，其界面是不受本地系统平台限制的，也就是说Swing开发的Java应用程序移植到其他系统平台上时，其界面外观是不会改变的。但要注意的是，虽然Swing提供的组件可以方便开发Java应用程序，但是Swing并不能取代AWT，在开发Swing程序时通常要借助与AWT的一些对象来共同完成应用程序的设计。

>本章使用IDEA

<!--more-->

常用Swing组件：

![](https://i.loli.net/2019/04/11/5caeeab9bc5e9.png)

### 常用窗体

Swing窗体是Swing的一个组件，同时也是创建图形化用户界面的容器，可以将其它组件放置在窗体容器中。

#### JFrame框架窗体
JFrame窗体是一个容器，在Swing开发中我们经常要用到，它是Swing程序中各个组件的载体。语法格式如下：
```
    JFrame()：构造一个初始时不可见的新窗体。
    JFrame(String title)：创建一个具有 title 指定标题的不可见新窗体。
```
当然，在开发中更常用的方式是通过继承java.swing.JFrame类创建一个窗体，可通过this关键字调用其方法。

而在JFrame对象创建完成后，需要调用getContentPane()方法将窗体转换为容器，然后在容器中添加组件或设置布局管理器，通常这个容器用来包含和显示组件。如果需要将组件添加至容器，可以使用来自Container类的add()方法进行设置。至于JPanel容器会在后面提到。

Swing窗体通常与组件和容器相关，所以在创建JFrame对象之后，需要调用getContentPane（）将窗体转换成容器，然后在容器中增删组件，通常这个容器用来包含和显示组件。

![](https://i.loli.net/2019/04/11/5caef0f16d28d.png)

显示有“大家好”的 Swing 组件需要放到内容窗格的上面，内容窗格再放到 JFrame 顶层容器的上面。菜单栏可以直接放到顶层容器 JFrame 上，而不通过内容窗格。内容窗格是一个透明的没有边框的中间容器。

JFrame 类中的常用方法

![](https://i.loli.net/2019/04/11/5caef131bf471.png)

下面举一个JFrame窗体的例子。
```（java）
import javax.swing.JFrame;
import javax.swing.WindowConstants;

public class JFrameDemo {

    public void CreateJFrame() {
        JFrame jf = new JFrame("这是一个无所不包的JFrame窗体");        // 实例化一个JFrame对象
        jf.setVisible(true);        // 设置窗体可视
        jf.setSize(500, 350);        // 设置窗体大小
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);        // 设置窗体关闭方式
    }

    public static void main(String[] args) {
        new JFrameDemo().CreateJFrame();        // 调用CreateJFrame()方法
    }

}
```
运行结果如下：

![](https://i.loli.net/2019/04/11/5caeec1725adb.png)

这就是一个500*350的窗体，用的是setSize()方法；标题为“这是一个JFrame窗体”，在实例化对象时就可以定义；窗体关闭方式见窗体右上角为“EXIT_ON_CLOSE”；窗体可视setVisible()方法中的参数为“false”或不写setVisible()方法时，此窗体不可见。

常用的窗体关闭方式有四种，分别为“DO_NOTHING_ON_CLOSE”、“DISPOSE_ON_CLOSE”、“HIDE_ON_CLOSE”、“EXIT_ON_CLOSE”。第一种表示什么也不做就将窗体关闭；第二种表示任何注册监听程序对象后会自动隐藏并释放窗体；第三种表示隐藏窗口的默认窗口关闭；第四种表示退出应用程序默认窗口关闭。

下面再举一个用继承JFrame的方式编写的代码，并加入Container容器及JLabel标签（后面会提到），来看一下具体的流程。

```（java）
import java.awt.Color;
import java.awt.Container;

import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.SwingConstants;
import javax.swing.WindowConstants;

public class JFrameDemo2 extends JFrame{

    public void init() {
        this.setVisible(true);        // 可视化
        this.setSize(500, 350);        // 大小
        this.setTitle("我很黄，不信你看我的窗体");        // 标题
        this.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);        // 关闭方式

        JLabel jl = new JLabel("我可是一个能显示你的idea的JLael标签");        // 创建一个JLabel标签
        jl.setHorizontalAlignment(SwingConstants.CENTER);        // 使标签文字居中

        Container container = this.getContentPane();        // 获取一个容器
        container.add(jl);        // 将标签添加至容器
        container.setBackground(Color.YELLOW);        // 设置容器背景颜色
    }

    public static void main(String[] args) {
        new JFrameDemo2().init();
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caeec172754c.png)

这里继承了JFrame类，所以方法中实现时用this关键字即可（或直接实现，不加this）。12~15行为创建JFrame框体；17、18行为JLabel标签，用与添加一个标签；20~22行为Container容器，用getContentPane()方法为JFrame窗体获取容器，并用add()方法将JLabel标签添加到容器上。

####  JPanel 面板

JPanel 是一种中间层容器，它能容纳组件并将组件组合在一起，但它本身必须添加到其他容器中使用。JPanel 类的构造方法如：
```
JPanel()：使用默认的布局管理器创建新面板，默认的布局管理器为 FlowLayout。
JPanel(LayoutManagerLayout layout)：创建指定布局管理器的 JPanel 对象。
```
JPanel 类的常用方法：

![](https://i.loli.net/2019/04/11/5caef1a64dc3f.png)

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class JPanelDemo {
    public static void main(String[] agrs)
    {
        JFrame jf=new JFrame("Java第二个GUI程序");    //创建一个JFrame对象
        jf.setBounds(300, 100, 400, 200);    //设置窗口大小和位置
        JPanel jp=new JPanel();    //创建一个JPanel对象
        JLabel jl=new JLabel("这是放在JPanel上的标签");    //创建一个标签
        jp.setBackground(Color.white);    //设置背景色
        jp.add(jl);    //将标签添加到面板
        jf.add(jp);    //将面板添加到窗口
        jf.setVisible(true);    //设置窗口可见
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caef28768abd.png)

如上述代码，首先创建了一个 JFrame 对象，并设置其大小和位置，然后创建了一个 JPanel对象表示面板，调用 setBackground() 方法设置面板的背景色为白色，调用 add() 方法将标签添加到此面板。JFrame 类的 add() 方法将 JPanel 面板添加到 JFmme 窗口中。最后调用 setVisible() 方法将窗口设置为可见。


#### JDialog窗体

JDialog窗体是Swing组件中的对话框，继承了AWT组件中的java.awt.Dialog类。功能是从一个窗体中弹出另一个窗体。

下面来看一个实例：

```（java）
import java.awt.Container;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

import javax.swing.JButton;
import javax.swing.JDialog;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.WindowConstants;

public class JDialogDemo extends JDialog {
    public JDialogDemo(JFrame father) {
        super(father, "这是一个提示的JDialog窗体", true); // 实例化一个JDialog类对象，指定其父窗体、窗口标题和类型
        Container container = this.getContentPane();
        container.add(new JLabel("我可是很有用的"));
        this.setSize(500, 350);
    }


    public static void main(String[] args) {
        new MyJFrame();
    }

}

// 下面这部分内容包含监听器，可自行查阅资料
class MyJFrame extends JFrame {
    public MyJFrame() {
        this.setVisible(true);
        this.setSize(700, 500);
        this.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);

        Container container = this.getContentPane();
        container.setLayout(null);

        JButton jb = new JButton("点击弹出对话框");        // 创建按钮
        jb.setBounds(30, 30, 200, 50);        // 按钮位置及大小
        jb.addActionListener(new ActionListener() {        // 监听器，用于监听点击事件
            @Override
            public void actionPerformed(ActionEvent e) {
                new JDialogDemo(MyJFrame.this).setVisible(true);
            }
        });
        container.add(jb);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caeef422edfd.png)

当我们点击按钮时，触发点击事件，这时执行第42行的语句，创建一个JDialog的实例化对象，弹出一个窗口。这里出现了许多我们之前学过的知识，比如super关键字，在之前提到过，这里相当于使用了JDialog(Frame f, String title, boolean model)形式的构造方法；监听器的实现就是一个匿名内部类，之前也提到过。

#### 对话框组件

对话框通常用作从用户处接收附加信息，或者提供发生了某种事件的通知。Java 提供了 JOptionPane 类，用来创建标准对话框，也可以通过扩展 JDialog 类创建自定义的对话框。JOptionPane 类可以用来创建 4 种类型的标准对话框：确认对话框、消息对话框、输入对话框和选项对话框。

**确认对话框**

确认对话框显示消息，并等待用户单击“确定”按钮来取消对话框，该对话框不返回任何值。而确认对话框询问一个问题，需要用户单击合适的按钮做出响应。确认对话框返回对应被选按钮的值。

创建确认对话框的方法如下：

```(java)
    public static int showConfirmDialog(Component parentComponent,Object message,String title,int optionType,int messageType,Icon icon)
```

参数 parentComponent、message、title、messageType 和 icon 与 showMessageDialog() 方法中的参数的含义相同。其中，只有 parentComponent 和 message 参数是必需的，title 的默认值为“选择一个选项”。messageType 的默认值是 QUESTION_MESSAGE。optionType 参数用于控制在对话框上显示的按钮，可选值如下：
```
0 或 JOptionPane.YES_NO_OPTIION。
1 或 JOptionPane.YES_NO_CANCEL_0PTII0N。
2 或 JOptionPane.OK_CANCEL_OPTIION。
```
例如，使用 showCon&mDialog() 方法创建 3 个确认对话框，该方法中指定的参数个数和参数值都是不同的，语句如下：

```(java)
import javax.swing.*;
import java.awt.*;

public class JDialogDemo2 {
  public  static void  main(String args []){
       JFrame jFrame=new JFrame("不同的确认对话框,确认三次删除本窗口");
       jFrame.setSize(500,500);
       jFrame.setVisible(true);
       jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
       Container p = jFrame.getContentPane();
       if (0==JOptionPane.showConfirmDialog(p,"确定要删除吗？","删除提示",0))
       {
           if (0==JOptionPane.showConfirmDialog(p,"确定要删除吗？","删除提示",1,2))
           {
               ImageIcon icon=new ImageIcon("/home/hyp/Picture/favicon-32x32.png");
               if(0==JOptionPane.showConfirmDialog(p,"确定要删除吗？","删除提示",2,1,icon))
               {
                   jFrame.dispose();
               }
           }
       }
   }
}
```
showConfirmDialog() 方法返回所选选项对应的值，这些值可以是整数或常量值，如下：

```
0 或 JOptionPane.YES_OPTIION。
1 或 JOptionPane.NO_OPTIION。
2 或 JOptionPane.CANCEL_OPTIION。
0 或 JOptionPane.OK_OPTIION。
-1 或 JOptionPane.CLOSED_OPTIION。
```
提示：除了 CLOSED_OPTIION 外，其他常量值都对应于激活的按钮。CLOSED_OPTIION 表示对话框在没有任何按钮激活的情况下关闭，例如单击对话框上的关闭图标按钮。

**消息对话框**

消息对话框显示一条提示或警告用户的信息，并等待用户单击 OK 或“确定”按钮以关闭对话框。创建消息对话框的方法如下：

```
    public static void showMessageDialog(Component parentComponent,Object message,String title,int messageType,Icon icon)
```
其中，只有 parentComponent 参数和 message 参数是必须指定的。parentComponent 可以是任意组件或者为空；message 用来定义提示信息，它是一个对象，但是通常使用字符串表示；title 是设置对话框标题的字符串；messageType 是以下整型或常量中的一个。
```
0 或 JOptionPane.ERROR_MESSAGE。
1 或 JOptionPane.INFORMATION_MESSAGE。
JOptionPane.PLAIN_MESSAGE。
2 或 JOptionPane.WARNING_MESSAGE。
3 或 JOptionPane.QUESTION_MESSAGE。
```
默认情况下，messageType 的值是 JOptionPane.INFORMATION_MESSAGE。除类型 PLAIN_MESSAGE外，每种类型都有相应的图标，也可以通过 icon 参数提供自己的图标。

 例如，下面的代码演示了不同的 messageType 取值实现的效果。

 ```(java)
 import javax.swing.*;
import java.awt.*;

public class JDialogDemo3 {
    public static void main(String args [])
    {
        JFrame jFrame=new JFrame("不同的消息对话框");
        jFrame.setSize(500,500);
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        Container p = jFrame.getContentPane();
        JOptionPane.showMessageDialog(p,"用户名或密码错误！","错误 ",0);
        JOptionPane.showMessageDialog(p,"请注册或登录...","提示",1);
        JOptionPane.showMessageDialog(p,"普通会员无权执行删除操作！","警告",2);
        JOptionPane.showMessageDialog(p,"你是哪一位？请输入用户名","问题",3);
        JOptionPane.showMessageDialog(p,"扫描完毕，没有发现病毒！","提示",JOptionPane.PLAIN_MESSAGE);
        jFrame.dispose();
    }
}
 ```
 第一行语句表示创建一个错误对话框。第二行语句表示创建一个提示对话框。第三行语句表示创建一个警告对话框。第四行语句表示创建一个问题对话框。第五行语句表示创建一个无图标对话框。

 **输入对话框**

输入对话框用于接收用户的输入。输入组件可以由文本框、下拉列表或者列表框进行实现。如果没有指定可选值，那么就使用文本框接收输入；如果指定了一组可选值，可选值的个数小于 20，那么将使用下拉列表显示；如果可选值的个数大于或等于 20，那么这些可选值将通过列表框显示。

创建输入对话框的方法如下：

```
public static String showInputDialog(Component parentComponent,Object message,String title,int messageType)

public static Object showInputDSalog(Component parentComponent,Object message,String title,int messageType,Icon icon,Object[] selectionValue,Object initValue)
```

其中，第一个 showInputDialog() 方法用于使用文本框输入，第二个 showInputDialog() 方法用于下拉列表或列表框的显示方式。参数 parentComponent 是必需的，message 默认为空，title 默认值为“输入”，messageType 的值默认为 3 或 JOptionPane.QUESTION_MESSAGE。

例如，使用 showInputDialog() 方法创建两个输入文本框，语句如下：

```(java)
import javax.swing.*;
import java.awt.*;

public class JDialogDemo4 {
    public static void main(String args [])
    {
        JFrame jFrame=new JFrame("不同的输入对话框");
        jFrame.setSize(500,500);
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        Container p = jFrame.getContentPane();
        JOptionPane.showInputDialog(p,"请输入用户名","输入用户名",1);
        String[] str={"admin","maxianglin","calcl23456","adminl23"};
        JOptionPane.showInputDialog(p,"请选择用户名","选择用户名",1,null,str,str[0]);
        jFrame.dispose();
    }
}
```
 第一个对话框没有指定列表值，那么将显示文本框；第二个对话框值显示为下拉列表的形式。

**选项对话框**

 选项对话框允许用户自己定制按钮内容。创建选项对话框的方法如下：
 ```
     public static int showOptionDialog(Component parentComponent,Object message,String title,int optionType,int messageType,icon icon,Object[] options,Object initValue)
 ```

其中，使用 options 参数指定按钮，initValue 参数用于指定默认获得焦点的按钮。该方法返回表明激活的按钮的一个整型值。

例如，创建一个 JButton 按钮数组，然后使用 showOptionDialog() 方法创建一个选项对话框，根据这个 JButton 数组来显示对话框的按钮，如下：

```(java)
import javax.swing.*;
import java.awt.*;

public class JDialogDemo5 {
    public static void main(String args [])
    {
        JFrame jFrame=new JFrame("不同的选项对话框");
        jFrame.setSize(500,500);
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        Container p = jFrame.getContentPane();

        JButton[] bs={new JButton("确定"),new JButton("取消"),new JButton("重置")};
        JOptionPane.showOptionDialog(p,"请选择其中的一项：","选择",1,3,null,bs,bs[0]);

        jFrame.dispose();
    }
}
```

### 布局管理器

在使用 Swing 向容器添加组件时，需要考虑组件的位置和大小。如果不使用布局管理器，则需要先在纸上画好各个组件的位置并计算组件间的距离，再向容器中添加。这样虽然能够灵活控制组件的位置，实现却非常麻烦。

为了加快开发速度，Java 提供了一些布局管理器，它们可以将组件进行统一管理，这样开发人员就不需要考虑组件是否会重叠等问题。本节介绍 Swing 提供的 6 种布局类型，所有布局都实现 LayoutManager 接口。

#### 边框布局管理器

BorderLayout（边框布局管理器）是 Window、JFrame 和 JDialog 的默认布局管理器。边框布局管理器将窗口分为 5 个区域：North、South、East、West 和 Center。其中，North 表示北，将占据面板的上方；Soufe 表示南，将占据面板的下方；East表示东，将占据面板的右侧；West 表示西，将占据面板的左侧；中间区域 Center 是在东、南、西、北都填满后剩下的区域。

![](https://i.loli.net/2019/04/11/5caf13038e9b1.png)

提示：边框布局管理器并不要求所有区域都必须有组件，如果四周的区域（North、South、East 和 West 区域）没有组件，则由 Center 区域去补充。如果单个区域中添加的不只一个组件，那么后来添加的组件将覆盖原来的组件，所以，区域中只显示最后添加的一个组件。

BorderLayout 布局管理器的构造方法如下所示。

```
    BorderLayout()：创建一个 Border 布局，组件之间没有间隙。
    BorderLayout(int hgap,int vgap)：创建一个 Border 布局，其中 hgap 表示组件之间的横向间隔；vgap 表示组件之间的纵向间隔，单位是像素。
```

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class BorderLayoutDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java第三个GUI程序");    //创建Frame窗口
        frame.setSize(400,200);
        frame.setLayout(new BorderLayout());    //为Frame窗口设置布局为BorderLayout
        JButton button1=new JButton ("上");
        JButton button2=new JButton("左");
        JButton button3=new JButton("中");
        JButton button4=new JButton("右");
        JButton button5=new JButton("下");
        frame.add(button1,BorderLayout.NORTH);
        frame.add(button2,BorderLayout.WEST);
        frame.add(button3,BorderLayout.CENTER);
        frame.add(button4,BorderLayout.EAST);
        frame.add(button5,BorderLayout.SOUTH);
        frame.setBounds(300,200,600,300);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}

```

运行结果：

![](https://i.loli.net/2019/04/11/5caf137258c00.png)

在该程序中分别指定了 BorderLayout 布局的东、南、西、北、中间区域中要填充的按钮。如果未指定布局管理器的 NORTH 区域，即将“frame.add(button1,BorderLayout.NORTH);”注释掉，则 WEST、CENTER 和 EAST 3 个区域将会填充 NORTH 区域。

同理，如果未指定布局管理器的 WEST 区域，即将“frame.add(button2,BorderLayout.WEST);”注释掉，则 CENTER 区域将会自动拉伸填充 WEST 区域。


#### 流式布局管理器

FlowLayout（流式布局管理器）是 JPanel 和 JApplet 的默认布局管理器。FlowLayout 会将组件按照从上到下、从左到右的放置规律逐行进行定位。与其他布局管理器不同的是，FlowLayout 布局管理器不限制它所管理组件的大小，而是允许它们有自己的最佳大小。

FlowLayout 布局管理器的构造方法如下。
```
    FlowLayout()：创建一个布局管理器，使用默认的居中对齐方式和默认 5 像素的水平和垂直间隔。
    FlowLayout(int align)：创建一个布局管理器，使用默认 5 像素的水平和垂直间隔。其中，align 表示组件的对齐方式，对齐的值必须是 FlowLayoutLEFT、FlowLayout.RIGHT 和 FlowLayout.CENTER，指定组件在这一行的位置是居左对齐、居右对齐或居中对齐。
    FlowLayout(int align, int hgap,int vgap)：创建一个布局管理器，其中 align 表示组件的对齐方式；hgap 表示组件之间的横向间隔；vgap 表示组件之间的纵向间隔，单位是像素。
```

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class FlowLayoutDemo {
    public static void main(String[] agrs)
    {
        JFrame jFrame=new JFrame("Java第四个GUI程序");    //创建Frame窗口
        JPanel jPanel=new JPanel();    //创建面板
        JButton btn1=new JButton("1");    //创建按钮
        JButton btn2=new JButton("2");
        JButton btn3=new JButton("3");
        JButton btn4=new JButton("4");
        JButton btn5=new JButton("5");
        JButton btn6=new JButton("6");
        JButton btn7=new JButton("7");
        JButton btn8=new JButton("8");
        JButton btn9=new JButton("9");
        jPanel.add(btn1);    //面板中添加按钮
        jPanel.add(btn2);
        jPanel.add(btn3);
        jPanel.add(btn4);
        jPanel.add(btn5);
        jPanel.add(btn6);
        jPanel.add(btn7);
        jPanel.add(btn8);
        jPanel.add(btn9);
        //向JPanel添加FlowLayout布局管理器，将组件间的横向和纵向间隙都设置为20像素
        jPanel.setLayout(new FlowLayout(FlowLayout.LEADING,20,20));
        jPanel.setBackground(Color.gray);    //设置背景色
        jFrame.add(jPanel);    //添加面板到容器
        jFrame.setBounds(300,200,300,400);    //设置容器的大小
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caf1473e7202.png)

上述程序向 JPanel 面板中添加了 9 个按钮，并使用 HowLayout 布局管理器使 9 个按钮间的横向和纵向间隙都为 20 像素。此时这些按钮将在容器上按照从上到下、从左到右的顺序排列，如果一行剩余空间不足容纳组件将会换行显示，

#### 卡片布局管理器

CardLayout（卡片布局管理器）能够帮助用户实现多个成员共享同一个显不空间，并且一次只显示一个容器组件的内容。

CardLayout 布局管理器将容器分成许多层，每层的显示空间占据整个容器的大小，但是每层只允许放置一个组件。CardLayout 的构造方法如下。
```
    CardLayout()：构造一个新布局，默认间隔为 0。
    CardLayout(int hgap, int vgap)：创建布局管理器，并指定组件间的水平间隔（hgap）和垂直间隔（vgap）。
```

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class CardLayoutDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java第五个程序");    //创建Frame窗口
        JPanel p1=new JPanel();    //面板1
        JPanel p2=new JPanel();    //面板2
        JPanel cards=new JPanel(new CardLayout());    //卡片式布局的面板
        p1.add(new JButton("登录按钮"));
        p1.add(new JButton("注册按钮"));
        p1.add(new JButton("找回密码按钮"));
        p2.add(new JTextField("用户名文本框",20));
        p2.add(new JTextField("密码文本框",20));
        p2.add(new JTextField("验证码文本框",20));
        cards.add(p1,"card1");    //向卡片式布局面板中添加面板1
        cards.add(p2,"card2");    //向卡片式布局面板中添加面板2
        CardLayout cl=(CardLayout)(cards.getLayout());
        cl.show(cards,"card1");    //调用show()方法显示面板2
        frame.add(cards);
        frame.setBounds(300,200,400,200);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```
运行结果:

![](https://i.loli.net/2019/04/11/5caf1599ceec0.png)

上述代码创建了一个卡片式布局的面板 cards，该面板包含两个大小相同的子面板 p1 和 p2。需要注意的是，在将 p1 和 p2 添加到 cards 面板中时使用了含有两个参数的 add() 方法，该方法的第二个参数用来标识子面板。当需要显示某一个面板时，只需要调用卡片式布局管理器的 show() 方法，并在参数中指定子面板所对应的字符串即可，这里显示的是 p1 面板。


#### 网格布局管理器

GridLayout（网格布局管理器）为组件的放置位置提供了更大的灵活性。它将区域分割成行数（rows）和列数（columns）的网格状布局，组件按照由左至右、由上而下的次序排列填充到各个单元格中。

GridLayout 的构造方法如下。
```
    GridLayout(int rows,int cols)：创建一个指定行（rows）和列（cols）的网格布局。布局中所有组件的大小一样，组件之间没有间隔。
    GridLayout(int rows,int cols,int hgap,int vgap)：创建一个指定行（rows）和列（cols）的网格布局，并且可以指定组件之间横向（hgap）和纵向（vgap）的间隔，单位是像素。
```

提示：GridLayout 布局管理器总是忽略组件的最佳大小，而是根据提供的行和列进行平分。该布局管理的所有单元格的宽度和高度都是一样的。

```（java）

```
运行结果

![](https://i.loli.net/2019/04/11/5caf164ec3c29.png)

上述程序设置面板为 4 行 4 列、间隙都为 5 像素的网格布局，在该面板上包含 16 个按钮，其横向和纵向的间隙都为 5。

#### 网格包布局管理器

GridBagLayout（网格包布局管理器）是在网格基础上提供复杂的布局，是最灵活、 最复杂的布局管理器。GridBagLayout 不需要组件的尺寸一致，允许组件扩展到多行多列。每个 GridBagLayout 对象都维护了一组动态的矩形网格单元，每个组件占一个或多个单元，所占有的网格单元称为组件的显示区域。

GridBagLayout 所管理的每个组件都与一个 GridBagConstraints 约束类的对象相关。这个约束类对象指定了组件的显示区域在网格中的位置，以及在其显示区域中应该如何摆放组件。除了组件的约束对象，GridBagLayout 还要考虑每个组件的最小和首选尺寸，以确定组件的大小。

为了有效地利用网格包布局管理器，在向容器中添加组件时，必须定制某些组件的相关约束对象。GridBagConstraints 对象的定制是通过下列变量实现的。

1. gridx 和 gridy
用来指定组件左上角在网格中的行和列。容器中最左边列的 gridx 为 0，最上边行的 gridy 为 0。这两个变量的默认值是 GridBagConstraints.RELATIVE，表示对应的组件将放在前一个组件的右边或下面。

2. gridwidth 和 gridheight
用来指定组件显示区域所占的列数和行数，以网格单元而不是像素为单位，默认值为 1。

3. fill
指定组件填充网格的方式，可以是如下值：GridBagConstraints.NONE（默认值）、GridBagConstraints.HORIZONTAL（组件横向充满显示区域，但是不改变组件高度）、GridBagConstraints.VERTICAL（组件纵向充满显示区域，但是不改变组件宽度）以及 GridBagConstraints.BOTH（组件横向、纵向充满其显示区域）。

4. ipadx 和 ipady
指定组件显示区域的内部填充，即在组件最小尺寸之外需要附加的像素数，默认值为 0。

5. insets
指定组件显示区域的外部填充，即组件与其显示区域边缘之间的空间，默认组件没有外部填充。

6. anchor
指定组件在显示区域中的摆放位置。可选值有 GridBagConstraints.CENTER（默认值）、GridBagConstraints.NORTH、GridBagConstraints.
NORTHEAST、GridBagConstraints.EAST、GridBagConstraints.SOUTH、GridBagConstraints.SOUTHEAST、GridBagConstraints.WEST、GridBagConstraints.SOUTHWEST 以及 GridBagConstraints.NORTHWEST。

7. weightx 和 weighty
用来指定在容器大小改变时，增加或减少的空间如何在组件间分配，默认值为 0，即所有的组件将聚拢在容器的中心，多余的空间将放在容器边缘与网格单元之间。weightx 和 weighty 的取值一般在 0.0 与 1.0 之间，数值大表明组件所在的行或者列将获得更多的空间。

示例：
```（java）
import javax.swing.*;
import java.awt.*;

public class GridBagLayoutDemo {
    //向JFrame中添加JButton按钮
    public static void makeButton(String title,JFrame frame,GridBagLayout gridBagLayout,GridBagConstraints constraints)
    {
        JButton button=new JButton(title);    //创建Button对象
        gridBagLayout.setConstraints(button,constraints);
        frame.add(button);
    }
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("拨号盘");
        GridBagLayout gbaglayout=new GridBagLayout();    //创建GridBagLayout布局管理器
        GridBagConstraints constraints=new GridBagConstraints();
        frame.setLayout(gbaglayout);    //使用GridBagLayout布局管理器
        constraints.fill=GridBagConstraints.BOTH;    //组件填充显示区域
        constraints.weightx=0.0;    //恢复默认值
        constraints.gridwidth = GridBagConstraints.REMAINDER;    //结束行
        JTextField tf=new JTextField("13612345678");
        gbaglayout.setConstraints(tf, constraints);
        frame.add(tf);
        constraints.weightx=0.5;    // 指定组件的分配区域
        constraints.weighty=0.2;
        constraints.gridwidth=1;
        makeButton("7",frame,gbaglayout,constraints);    //调用方法，添加按钮组件
        makeButton("8",frame,gbaglayout,constraints);
        constraints.gridwidth=GridBagConstraints.REMAINDER;    //结束行
        makeButton("9",frame,gbaglayout,constraints);
        constraints.gridwidth=1;    //重新设置gridwidth的值

        makeButton("4",frame,gbaglayout,constraints);
        makeButton("5",frame,gbaglayout,constraints);
        constraints.gridwidth=GridBagConstraints.REMAINDER;
        makeButton("6",frame,gbaglayout,constraints);
        constraints.gridwidth=1;

        makeButton("1",frame,gbaglayout,constraints);
        makeButton("2",frame,gbaglayout,constraints);
        constraints.gridwidth=GridBagConstraints.REMAINDER;
        makeButton("3",frame,gbaglayout,constraints);
        constraints.gridwidth=1;

        makeButton("返回",frame,gbaglayout,constraints);
        constraints.gridwidth=GridBagConstraints.REMAINDER;
        makeButton("拨号",frame,gbaglayout,constraints);
        constraints.gridwidth=1;
        frame.setBounds(400,400,400,400);    //设置容器大小
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```

运行结果：

![](https://i.loli.net/2019/04/11/5caf170ee35ec.png)

在上述程序中创建了一个 makeButton() 方法，用来将 JButton 组件添加到 JFrame 窗口中。在 main() 方法中分别创建了 GridBagLayout 对象和 GridBagConstraints 对象，然后设置 JFrame 窗口的布局为 GridBagLayout，并设置了 GridBagConstraints 的一些属性。接着将 JTextField 组件添加至窗口中，并通知布局管理器的 GridBagConstraints 信息。

####  盒布局管理器

BoxLayout（盒布局管理器）通常和 Box 容器联合使用，Box 类有以下两个静态方法。
```
    createHorizontalBox()：返回一个 Box 对象，它采用水平 BoxLayout，即 BoxLayout 沿着水平方向放置组件，让组件在容器内从左到右排列。
    createVerticalBox()：返回一个 Box 对象，它采用垂直 BoxLayout，即 BoxLayout 沿着垂直方向放置组件，让组件在容器内从上到下进行排列。
```
Box 还提供了用于决定组件之间间隔的静态方法

![](https://i.loli.net/2019/04/11/5caf176d64e5c.png)

BoxLayout 类只有一个构造方法，如下所示。
```
    BoxLayout(Container c,int axis)
```
其中，参数 Container 是一个容器对象，即该布局管理器在哪个容器中使用；第二个参数为 int 型，用来决定容器上的组件水平（X_AXIS）或垂直（Y_AXIS）放置，可以使用 BoxLayout 类访问这两个属性。

示例：
```（java）
import javax.swing.*;
import java.awt.*;

public class BoxLayoutDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java示例程序");
        Box b1=Box.createHorizontalBox();    //创建横向Box容器
        Box b2=Box.createVerticalBox();    //创建纵向Box容器
        frame.add(b1);    //将外层横向Box添加进窗体
        b1.add(Box.createVerticalStrut(200));    //添加高度为200的垂直框架
        b1.add(new JButton("西"));    //添加按钮1
        b1.add(Box.createHorizontalStrut(140));    //添加长度为140的水平框架
        b1.add(new JButton("东"));    //添加按钮2
        b1.add(Box.createHorizontalGlue());    //添加水平胶水
        b1.add(b2);    //添加嵌套的纵向Box容器
        //添加宽度为100，高度为20的固定区域
        b2.add(Box.createRigidArea(new Dimension(100,20)));
        b2.add(new JButton("北"));    //添加按钮3
        b2.add(Box.createVerticalGlue());    //添加垂直组件
        b2.add(new JButton("南"));    //添加按钮4
        b2.add(Box.createVerticalStrut(40));    //添加长度为40的垂直框架
        //设置窗口的关闭动作、标题、大小位置以及可见性等
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setBounds(100,100,400,200);
        frame.setVisible(true);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caf17f634eed.png)

在程序中创建了 4 个 JButton 按钮和两个 Box 容器（横向 Box 容器和纵向 Box 容器），并将前两个 JButton 按钮添加到横向 Box 容器中，将后两个 JButton 容器添加到纵向 Box 容器中。

提示：使用盒式布局可以像使用流式布局一样简单地将组件安排在一个容器内。包含盒式布局的容器可以嵌套使用，最终达到类似于无序网格布局那样的效果，却不像使用无序网格布局那样麻烦。



### 组件

#### 标签
在Swing中显示文本或提示信息的方法是使用标签，它支持文本字符串和图标。上面我们提到的JLabel就是这里的内容。

标签由JLabel类定义，可以显示一行只读文本、一个图像或带图像的文本。

JLabel类提供了许多构造方法，可查看API选择需要的使用，如显示只有文本的标签、只有图标的标签或包含文本与图标的标签等。因为上面已经出现过了，这里就不再举例了。 标签是一种可以包含文本和图片的非交互组件，其文本可以是单行文本，也可以是 HTML 文本。对于只包含文本的标签可以使用 JLabel 类，该类的主要构造方法有如下几种形式。

```
JLabel()：创建无图像并且标题为空字符串的 JLabel。
JLabel(Icon image)：创建具有指定图像的 JLabel。
JLabel(String text)：创建具有指定文本的 JLabel。
JLabel(String textjcon image,int horizontalAlignment)：创建具有指定文本、图像和水平对齐方式的 JLabel，horizontalAlignment 的取值有 3 个，即 JLabel.LEFT、JLabel.RIGHT 和 JLabel.CENTER。
```
常用方法：

![](https://i.loli.net/2019/04/11/5caeffebc8e5e.png)

示例：
```（java）

import javax.swing.*;

public class JFrameDemo3 {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java标签组件示例");    //创建Frame窗口
        JPanel jp=new JPanel();    //创建面板
        JLabel label1=new JLabel("普通标签");    //创建标签
        JLabel label2=new JLabel();
        label2.setText("调用setText()方法");
        ImageIcon img=new ImageIcon("/home/hyp/Pictures/favicon-32x32.png");    //创建一个图标
        //创建既含有文本又含有图标的JLabel对象
        JLabel label3=new JLabel("开始理财",img,JLabel.CENTER);
        jp.add(label1);    //添加标签到面板
        jp.add(label2);
        jp.add(label3);
        frame.add(jp);
        frame.setBounds(300,200,400,400);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

    }        // 调用CreateJFrame()方法
}
```

运行结果：

![](https://i.loli.net/2019/04/11/5caf017fac8c9.png)

#### 图标
Swing中的图标可以放置在按钮、标签等组件上，用于描述组件的用途。图标可以用Java支持的图片文件类型进行创建，也可以使用java.awt.Graphics类提供的功能方法来创建。

在Swing中通过Icon接口来创建图标，可以在创建时给定图标的大小、颜色等特性。注意，Icon是接口，在使用Icon接口的时候，必须实现Icon接口的三个方法：

```
public int getIconHeight()
public int getIconWidth()
public void paintIcon(Component arg0, Graphics arg1, int arg2, int arg3)
```
前两个方法用于获取图片的长宽，paintIcon()方法用于实现在指定坐标位置画图。

下面看一个用Icon接口创建图标的实例：
```（java）
import javax.swing.*;
import java.awt.*;

public class IconDemo  extends JFrame implements Icon{

    private int width;        // 声明图标的宽
    private int height;        // 声明图标的长

    public IconDemo() {
    }        // 定义无参构造方法

    public IconDemo(int width, int height) {        // 定义有参构造方法
        this.width = width;
        this.height = height;
    }

    @Override
    public int getIconHeight() {        // 实现getIconHeight()方法
        return this.height;
    }

    @Override
    public int getIconWidth() {            // 实现getIconWidth()方法
        return this.width;
    }

    @Override
    public void paintIcon(Component arg0, Graphics arg1, int arg2, int arg3) {        // 实现paintIcon()方法
        arg1.fillOval(arg2, arg3, width, height);        // 绘制一个圆形
    }

    public void init() {    // 定义一个方法用于实现界面
        IconDemo iconDemo = new IconDemo(15, 15);        // 定义图标的长和宽
        JLabel jb = new JLabel("icon测试", iconDemo, SwingConstants.CENTER);    // 设置标签上的文字在标签正中间

        Container container = getContentPane();
        container.add(jb);

        this.setVisible(true);
        this.setSize(500, 350);
        this.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }

    public static void main(String[] args) {
        new IconDemo().init();
    }
}
```
运行结果：
![](https://i.loli.net/2019/04/11/5caf0285826f3.png)

这样如果需要在窗体中使用图标，就可以用如下代码创建图标：
```
IconDemo iconDemo = new IconDemo(15, 15);
```
#### 按钮

按钮是图形界面上常见的元素，在前面已经多次使用过它。在 Swing 中按钮是 JButton 类的对象，JButton 类的常用构造方法如下。
```
JButton()：创建一个无标签文本、无图标的按钮。
JButton(Icon icon)：创建一个无标签文本、有图标的按钮。
JButton(String text)：创建一个有标签文本、无图标的按钮。
JButton(String text,Icon icon)：创建一个有标签文本、有图标的按钮。
```
常用方法：

![](https://i.loli.net/2019/04/11/5caf0430746df.png)

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class JButtonDemo {
    public static void main(String[] args)
    {
        JFrame frame=new JFrame("Java按钮组件示例");    //创建Frame窗口
        frame.setSize(400, 200);
        JPanel jp=new JPanel();    //创建JPanel对象
        JButton btn1=new JButton("我是普通按钮");    //创建JButton对象
        JButton btn2=new JButton("我是带背景颜色按钮");
        JButton btn3=new JButton("我是不可用按钮");
        JButton btn4=new JButton("我是底部对齐按钮");
        jp.add(btn1);
        btn2.setBackground(Color.YELLOW);    //设置按钮背景色
        jp.add(btn2);
        btn3.setEnabled(false);    //设置按钮不可用
        jp.add(btn3);
        Dimension preferredSize=new Dimension(160, 60);    //设置尺寸
        btn4.setPreferredSize(preferredSize);    //设置按钮大小
        btn4.setVerticalAlignment(SwingConstants.BOTTOM);    //设置按钮垂直对齐方式
        jp.add(btn4);
        frame.add(jp);
        frame.setBounds(300, 200, 600, 300);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caf04ecbda37.png)

上述代码创建了 1 个 JFrame 窗口对象、1 个 JPanel 面板对象和 4 个 JButton 按钮，然后调用 JButton 类的 setBackground() 方法、setEnabled() 方法、setPreferredSize() 方法和 setVerticalAlignment() 方法设置按钮的显示外观。

#### 单行文本框
Swing 中使用 JTextField 类实现一个单行文本框，它允许用户输入单行的文本信息。该类的常用构造方法如下。
```
JTextField()：创建一个默认的文本框。
JTextField(String text)：创建一个指定初始化文本信息的文本框。
JTextField(int columns)：创建一个指定列数的文本框。
JTextField(String text,int columns)：创建一个既指定初始化文本信息，又指定列数的文本框。
```

常用方法：

![](https://i.loli.net/2019/04/11/5caf0573c1536.png)

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class JTextFieldDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java文本框组件示例");    //创建Frame窗口
        JPanel jp=new JPanel();    //创建面板
        JTextField txtfield1=new JTextField();    //创建文本框
        txtfield1.setText("普通文本框");    //设置文本框的内容
        JTextField txtfield2=new JTextField(28);
        txtfield2.setFont(new Font("楷体",Font.BOLD,16));    //修改字体样式
        txtfield2.setText("指定长度和字体的文本框");
        JTextField txtfield3=new JTextField(30);
        txtfield3.setText("居中对齐");
        txtfield3.setHorizontalAlignment(JTextField.CENTER);    //居中对齐
        jp.add(txtfield1);
        jp.add(txtfield2);
        jp.add(txtfield3);
        frame.add(jp);
        frame.setBounds(300,200,400,400);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caf0643bebf7.png)

上述程序中，第一个文本框 txtfield1 使用 JTextField 的默认构造方法创建；第二个文本框 txtfield2 在创建时指定了文本框的长度，同时还修改了文本的字体样式；第三个文本框 txtfield3 设置文本为居中对齐。

#### 文本域组件

文本域与文本框的最大区别就是文本域允许用户输入多行文本信息。在 Swing 中使用 JTextArea 类实现一个文本域，其常用构造方法如下。
```
JTextArea()：创建一个默认的文本域。
JTextArea(int rows,int columns)：创建一个具有指定行数和列数的文本域。
JTextArea(String text)：创建一个包含指定文本的文本域。
JTextArea(String text,int rows,int columns)：创建一个既包含指定文本，又包含指定行数和列数的多行文本域。
```
常用方法：

![](https://i.loli.net/2019/04/11/5caf06fc03cf8.png)

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class JTextAreaDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java文本域组件示例");    //创建Frame窗口
        JPanel jp=new JPanel();    //创建一个JPanel对象
        JTextArea jta=new JTextArea("请输入内容",7,30);
        jta.setLineWrap(true);    //设置文本域中的文本为自动换行
        jta.setForeground(Color.BLACK);    //设置组件的背景色
        jta.setFont(new Font("楷体",Font.ROMAN_BASELINE,14));    //修改字体样式
        jta.setBackground(Color.YELLOW);    //设置按钮背景色
        JScrollPane jsp=new JScrollPane(jta);    //将文本域放入滚动窗口
        Dimension size=jta.getPreferredSize();    //获得文本域的首选大小
        jsp.setBounds(110,90,size.width,size.height);
        jp.add(jsp);    //将JScrollPane添加到JPanel容器中
        frame.add(jp);    //将JPanel容器添加到JFrame容器中
        frame.setBackground(Color.LIGHT_GRAY);
        frame.setSize(400,400);    //设置JFrame容器的大小
        frame.setVisible(true);
        frame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```
运行结果：

![](https://i.loli.net/2019/04/11/5caf07ea429a3.png)

在上述代码中将 JTextArea 文本域放入滚动窗口中，并通过 getPreferredSize() 方法获得文本域的显示大小。将滚动窗口的大小设置成与文本域大小相同，再将滚动窗口添加到 JPanel 面板中。

#### 复选框组件

一个复选框有选中和未选中两种状态，并且可以同时选定多个复选框。Swing 中使用 JCheckBox 类实现复选框，该类的常用构造方法如下。
```
JCheckBox()：创建一个默认的复选框，在默认情况下既未指定文本，也未指定图像，并且未被选择。
JCheckBox(String text)：创建一个指定文本的复选框。
JCheckBox(String text,boolean selected)：创建一个指定文本和选择状态的复选框。
```

示例：
```（java）

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;


public class JCheckBoxDemo {
    public static void main(String[] agrs) {
        JFrame frame = new JFrame("Java复选组件示例");    //创建Frame窗口
        JPanel jp = new JPanel();    //创建面板
        JLabel label = new JLabel("流行编程语言有：");
        label.setFont(new Font("楷体", Font.BOLD, 14));    //修改字体样式
        JCheckBox chkbox1 = new JCheckBox("C#", true);    //创建指定文本和状态的复选框
        JCheckBox chkbox2 = new JCheckBox("C++");    //创建指定文本的复选框
        JCheckBox chkbox3 = new JCheckBox("Java");    //创建指定文本的复选框
        JCheckBox chkbox4 = new JCheckBox("Python");    //创建指定文本的复选框
        JCheckBox chkbox5 = new JCheckBox("PHP");    //创建指定文本的复选框
        JCheckBox chkbox6 = new JCheckBox("Perl");    //创建指定文本的复选框
        jp.add(label);
        jp.add(chkbox1);
        jp.add(chkbox2);
        jp.add(chkbox3);
        jp.add(chkbox4);
        jp.add(chkbox5);
        jp.add(chkbox6);
        frame.add(jp);

        JButton button=new JButton("确认");
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                frame.dispose();
            }
        });
        jp.add(button);

        frame.setBounds(300, 200, 400, 400);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caf0b9b59117.png)

上述程序一共创建了 6 个复选框，其中第一个调用 JCheckBox 构造方法时指定了复选框为选中状态。

#### 单选按钮组件

单选按钮与复选框类似都有两种状态，不同的是一组单选按钮中只能有一个处于选中状态。Swing 中 JRadioButton 类实现单选按钮，它与 JCheckBox 一样都是从 JToggleButton 类派生出来的。JRadioButton 通常位于一个 ButtonGroup 按钮组中，不在按钮组中的 JRadioButton 也就失去了单选按钮的意义。

在同一个 ButtonGroup 按钮组中的单选按钮，只能有一个单选按钮被选中。因此，如果创建的多个单选按钮其初始状态都是选中状态，则最先加入 ButtonGroup 按钮组的单选按钮的选中状态被保留，其后加入到 ButtonGroup 按钮组中的其他单选按钮的选中状态被取消。

JRadioButton 类的常用构造方法如下。
```
JRadioButton()：创建一个初始化为未选择的单选按钮，其文本未设定。
JRadioButton(Icon icon)：创建一个初始化为未选择的单选按钮，其具有指定的图像但无文本。
JRadioButton(Icon icon,boolean selected)：创建一个具有指定图像和选择状态的单选按钮，但无文本。
JRadioButton(String text)：创建一个具有指定文本但未选择的单选按钮。
JRadioButton(String text,boolean selected)：创建一个具有指定文本和选择状态的单选按钮。
JRadioButton(String text,Icon icon)：创建一个具有指定的文本和图像并初始化为未选择的单选按钮。
JRadioButton(String text,Icon icon,boolean selected)：创建一个具有指定的文本、图像和选择状态的单选按钮。
```

示例：
```（java）

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class JRadioButtonDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("Java单选组件示例");    //创建Frame窗口
        JPanel panel=new JPanel();    //创建面板
        JLabel label1=new JLabel("现在是哪个季节：");
        JRadioButton rb1=new JRadioButton("春天");    //创建JRadioButton对象
        JRadioButton rb2=new JRadioButton("夏天");    //创建JRadioButton对象
        JRadioButton rb3=new JRadioButton("秋天",true);    //创建JRadioButton对象
        JRadioButton rb4=new JRadioButton("冬天");    //创建JRadioButton对象

        label1.setFont(new Font("楷体",Font.BOLD,16));    //修改字体样式

        ButtonGroup group=new ButtonGroup();
        //添加JRadioButton到ButtonGroup中
        group.add(rb1);
        group.add(rb2);
        group.add(rb3);
        group.add(rb4);
        panel.add(label1);
        panel.add(rb1);
        panel.add(rb2);
        panel.add(rb3);
        panel.add(rb4);
        frame.add(panel);

        JButton button=new JButton("确认");
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                frame.dispose();
            }
        });
        panel.add(button);

        frame.setBounds(300, 200, 400, 400);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```
运行结果：

![](https://i.loli.net/2019/04/11/5caf0d3869fcc.png)

在上述程序中创建了 4 个 JRadioButton 单选按钮，并将这 4 个单选按钮添加到 ButtonGroup 组件中。

#### 下拉列表

下拉列表的特点是将多个选项折叠在一起，只显示最前面的或被选中的一个。选择时需要单击下拉列表右边的下三角按钮，这时候会弹出包含所有选项的列表。用户可以在列表中进行选择，也可以根据需要直接输入所要的选项，还可以输入选项中没有的内容。

下拉列表由 JComboBox 类实现，常用构造方法如下。
```
JComboBox()：创建一个空的 JComboBox 对象。
JComboBox(ComboBoxModel aModel)：创建一个 JComboBox，其选项取自现有的 ComboBoxModel。
JComboBox(Object[] items)：创建包含指定数组中元素的 JComboBox。
```
常用方法：

![](https://i.loli.net/2019/04/11/5caf0db492271.png)

JComboBox 能够响应 ItemEvent 事件和 ActionEvent 事件，其中 ItemEvent 触发的时机是当下拉列表框中的所选项更改时，ActionEvent 触发的时机是当用户在 JComboBox 上直接输入选择项并回车时。要处理这两个事件，需要创建相应的事件类并实现 ItemListener 接口和 ActionListener 接口。

示例：

```（java）
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class JComboBoxDemo {
    public static void main(String[] args)
    {
        JFrame frame=new JFrame("Java下拉列表组件示例");
        JPanel jp=new JPanel();    //创建面板
        JLabel label1=new JLabel("证件类型：");    //创建标签
        JComboBox cmb=new JComboBox();    //创建JComboBox
        cmb.addItem("--请选择--");    //向下拉列表中添加一项
        cmb.addItem("身份证");
        cmb.addItem("驾驶证");
        cmb.addItem("军官证");
        jp.add(label1);
        jp.add(cmb);
        frame.add(jp);
        JButton button=new JButton("确认");
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                frame.dispose();
            }
        });
        jp.add(button);
        frame.setBounds(300,200,400,100);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}
```

运行结果：

![](https://i.loli.net/2019/04/11/5caf0e6d8956b.png)

上述代码创建了一个下拉列表组件 cmb，然后调用 addItem() 方法向下拉列表中添加 4 个选项。

#### 列表框

列表框与下拉列表的区别不仅仅表现在外观上，当激活下拉列表时，会出现下拉列表框中的内容。但列表框只是在窗体系上占据固定的大小，如果需要列表框具有滚动效果，可以将列表框放到滚动面板中。当用户选择列表框中的某一项时，按住 Shift 键并选择列表框中的其他项目，可以连续选择两个选项之间的所有项目，也可以按住 Ctrl 键选择多个项目。

Swing 中使用 JList 类来表示列表框，该类的常用构造方法如下。
```
JList()：构造一个空的只读模型的列表框。
JList(ListModel dataModel)：根据指定的非 null 模型对象构造一个显示元素的列表框。
JList(Object[] listData)：使用 listData 指定的元素构造—个列表框。
JList(Vector<?> listData)：使用 listData 指定的元素构造一个列表框。
```

上述的第一个构造方法没有参数，使用此方法创建列表框后可以使用 setListData() 方法对列表框的元素进行填充，也可以调用其他形式的构造方法在初始化时对列表框的元素进行填充。常用的元素类型有 3 种，分别是数组、Vector 对象和 ListModel 模型。

示例：

```（java）

import javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;

public class JListDemo extends  JFrame{
    public JListDemo()
    {
        setTitle("Java列表框组件示例");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);    //设置窗体退出时操作
        setBounds(100,100,300,200);    //设置窗体位置和大小
        JPanel contentPane=new JPanel();    //创建内容面板
        contentPane.setBorder(new EmptyBorder(5, 5, 5, 5));    //设置面板的边框
        contentPane.setLayout(new BorderLayout(0, 0));    //设置内容面板为边界布局
        setContentPane(contentPane);    //应用内容面板
        JScrollPane scrollPane=new JScrollPane();    //创建滚动面板
        contentPane.add(scrollPane,BorderLayout.CENTER);    //将面板增加到边界布局中央
        JList list=new JList();
        //限制只能选择一个元素
        list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
        scrollPane.setViewportView(list);    //在滚动面板中显示列表
        String[] listData=new String[12];    //创建一个含有12个元素的数组
        for (int i=0;i<listData.length;i++)
        {
            listData[i]="这是列表框的第"+(i+1)+"个元素~";    //为数组中各个元素赋值
        }
        list.setListData(listData);    //为列表填充数据
    }
    public static void main(String[] args)
    {
        JListDemo frame=new JListDemo();
        frame.setVisible(true);
    }
}

```
运行结果：

![](https://i.loli.net/2019/04/11/5caf0fe700e88.png)

上述代码调用了 setSelectionMode() 方法，并指定 ListSelectionModel.SINGLE_SELECTION 常量来限制列表框一次只能选择一项。该方法还支持如下两个常量。
```
    ListSelectionModel.SINGLE_INTERVAL_SELECTION：允许选择一个或多个连续的元素。
    ListSelectionModel.MULTIPLE_INTERVAL_SELECTION：允许选择一个连续的元素。
```

### 实现计算机界面

计算器界面可以分成两部分，即显示区和键盘区。显示区可以使用文本框组件，键盘区则是由很多按钮组成，可以使用网格布局管理器。详细的实现过程如下。

(1) 新建一个继承自 JFrame 的 CalculatorDemo 类。

(2) 为类添加构造方法和 main() 方法。

(3) 在构造方法中设置窗口的标题和大小等属性，然后使用边界面板向北部添加一个 JTextField 组件。

(4) 接下来使用网格布局管理器添加多个按钮作为计算器的键盘区。

(5) 最终程序的运行效果如下：

![](https://i.loli.net/2019/04/11/5caf1127770a5.png)

代码如下：

```（java）
import javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;

public class CalculatorDemo extends JFrame {
    private JPanel contentPane;    //内容面板
    private JTextField textField;    //文本框
    public CalculatorDemo(){
        setTitle("计算器");    //设置窗体的标题
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);    //设置窗体退出时操作
        setBounds(100, 100, 250, 250);    //设置窗体位置和大小
        contentPane=new JPanel();    //创建内容面板
        contentPane.setBorder(new EmptyBorder(5,5,5,5));    //设置面板的边框
        contentPane.setLayout(new BorderLayout(0,0));    //设置内容面板为边界布局
        setContentPane(contentPane);    //应用内容面板
        JPanel panel1=new JPanel();    //新建面板用于保存文本框
        contentPane.add(panel1,BorderLayout.NORTH);    //将面板放置在边界布局的北部
        textField=new JTextField();    //新建文本框
        textField.setHorizontalAlignment(SwingConstants.RIGHT);    //文本框中的文本使用右对齐
        panel1.add(textField);    //将文本框增加到面板中
        textField.setColumns(18);    //设置文本框的列数是18

        JPanel panel2=new JPanel();    //新建面板用于保存按钮
        contentPane.add(panel2, BorderLayout.CENTER);    //将面板放置在边界布局的中央
        panel2.setLayout(new GridLayout(4,4,5,5));    //面板使用网格4X4布局
        JButton button01=new JButton("7");    //新建按钮
        panel2.add(button01);    //应用按钮
        JButton button02=new JButton("8");    //新建按钮
        panel2.add(button02);    //应用按钮
        JButton button03=new JButton("9");    //新建按钮
        panel2.add(button03);    //应用按钮
        JButton button04=new JButton("+");    //新建按钮
        panel2.add(button04);    //应用按钮
        JButton button05=new JButton("4");    //新建按钮
        panel2.add(button05);    //应用按钮
        JButton button06=new JButton("5");    //新建按钮
        panel2.add(button06);    //应用按钮
        JButton button07=new JButton("6");    //新建按钮
        panel2.add(button07);    //应用按钮
        JButton button08=new JButton("-");    //新建按钮
        panel2.add(button08);    //应用按钮
        JButton button09=new JButton("3");    //新建按钮
        panel2.add(button09);    //应用按钮
        JButton button10=new JButton("2");    //新建按钮
        panel2.add(button10);    //应用按钮
        JButton button11=new JButton("1");    //新建按钮
        panel2.add(button11);    //应用按钮
        JButton button12=new JButton("*");    //新建按钮
        panel2.add(button12);    //应用按钮
        JButton button13=new JButton("0");    //新建按钮
        panel2.add(button13);    //应用按钮
        JButton button14=new JButton(".");    //新建按钮
        panel2.add(button14);    //应用按钮
        JButton button15=new JButton("=");    //新建按钮
        panel2.add(button15);    //应用按钮
        JButton button16=new JButton("/");    //新建按钮
        panel2.add(button16);    //应用按钮
    };    //构造方法
    public static void main(String[] args)
    {
        CalculatorDemo frame=new CalculatorDemo();
        frame.setVisible(true);
    }
}
```
