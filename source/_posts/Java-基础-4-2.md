---
title: Java Swing程序设计（二）
date: 2019-04-05 22:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 事件监听

事件表示程序和用户之间的交互，例如在文本框中输入，在列表框或组合框中选择，选中复选框和单选框，单击按钮等。事件处理表示程序对事件的响应，对用户的交互或者说对事件的处理是事件处理程序完成的。

当事件发生时，系统会自动捕捉这一事件，创建表示动作的事件对象并把它们分派给程序内的事件处理程序代码。这种代码确定了如何处理此事件以使用户得到相应的回答。

<!--more-->

#### 事件处理模型

前面我们讲解了如何放置各种组件，使图形界面更加丰富多彩，但是还不能响应用户的任何操作。若使图形界面能够接收用户的操作，必须给各个组件加上事件处理机制。在事件处理的过程中，主要涉及三类对象。

```
Event（事件）：用户对组件的一次操作称为一个事件，以类的形式出现。例如，键盘操作对应的事件类是 KeyEvent。
Event Source（事件源）：事件发生的场所，通常就是各个组件，例如按钮 Button。
Event Handler（事件处理者）：接收事件对象并对其进行处理的对象事件处理器，通常就是某个 Java 类中负责处理事件的成员方法。
```

例如，如果鼠标单击了按钮对象 Button，则该按钮 Button 就是事件源，而 Java 运行时系统会生成 ActionEvent 类的对象 ActionEvent，该对象中描述了单击事件发生时的一些信息。之后，事件处理者对象将接收由 Java 运行时系统传递过来的事件对象 ActionEvent，并进行相应的处理。

![](https://i.loli.net/2019/04/11/5caf19263b635.png)

由于同一个事件源上可能发生多种事件，因此，Java 采取了授权模型（Delegation Model），**事件源可以把在其自身上所有可能发生的事件分别授权给不同的事件处理者来处理**。例如，在 Panel 对象上既可能发生鼠标事件，也可能发生键盘事件，该 Panel 对象可以授权给事件处理者 a 来处理鼠标事件，同时授权给事件处理者 b 来处理键盘事件。

有时也将事件处理者称为监听器，主要原因在于 **监听器时刻监听事件源上所有发生的事件类型，一旦该事件类型与自己所负责处理的事件类型一致，就马上进行处理。授权模型把事件的处理委托给外部的处理实体进行处理，实现了将事件源和监听器分开的机制。**

**事件处理者（监听器）通常是一个类，该类如果能够处理某种类型的事件，就必须实现与该事件类型相对的接口。** 例如，一个 ButtonHandler 类之所以能够处理 ActionEvent 事件，原因在于它实现了与 ActionEvent 事件对应的接口 ActionListener。每个事件类都有一个与之相对应的接口。

####  动作事件监听器

动作事件监听器是 Swing 中比较常用的事件监听器，很多组件的动作都会使用它监听，像按钮被里击、列表框中选择一项等。与动作事件监听器有关的信息如下。
```
    事件名称：ActionEvent。
    事件监听接口: ActionListener。
    事件相关方法：addActionListener() 添加监听，removeActionListener() 删除监听。
    涉及事件源：JButton、JList、JTextField 等。
```
示例：

```（java）

import javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class ActionListenerDemo extends JFrame {
    JList list;
    JLabel label;
    JButton button1;
    static  int clicks=1;
    public ActionListenerDemo()
    {
        setTitle("动作事件监听器示例");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setBounds(100,100,400,200);
        JPanel contentPane=new JPanel();
        contentPane.setSize(50,50);
        contentPane.setBorder(new EmptyBorder(5,5,5,5));
        contentPane.setLayout(new BorderLayout(0,0));
        setContentPane(contentPane);
        label=new JLabel(" ");
        label.setFont(new Font("楷体",Font.BOLD,16));    //修改字体样式
        contentPane.add(label, BorderLayout.SOUTH);
        button1=new JButton("我是普通按钮");    //创建JButton对象
        button1.setFont(new Font("黑体",Font.BOLD,16));    //修改字体样式
        button1.addActionListener(new ActionListener()
        {
            public void actionPerformed(ActionEvent e)
            {
                label.setText("按钮被单击了 "+(clicks++)+" 次");
            }
        });
        button1.setSize(50,50);
        contentPane.add(button1);
    }
    //处理按钮单击事件的匿名内部类
    class button1ActionListener implements ActionListener
    {
        @Override
        public void actionPerformed(ActionEvent e)
        {
            label.setText("按钮被单击了 "+(++clicks)+" 次");
        }
    }
    public static void main(String[] args)
    {
        ActionListenerDemo frame=new ActionListenerDemo();
        frame.setVisible(true);
    }
}
```
运行结果:

![](https://i.loli.net/2019/04/11/5caf1d93383ba.png)

####   焦点事件监听器

除了单击事件外，焦点事件监听器在实际项目中应用也比较广泛，例如将光标离开文本框时弹出对话框，或者将焦点返回给文本框等。

与焦点事件监听器有关的信息如下。
```
    事件名称：FocusEvent。
    事件监听接口： FocusListener。
    事件相关方法：addFocusListener() 添加监听，removeFocusListener() 删除监听。
    涉及事件源：Component 以及派生类。
```

FocusEvent 接口定义了两个方法，分别为 focusGained() 方法和 focusLost() 方法，其中 focusGained() 方法是在组件获得焦点时执行，focusLost() 方法是在组件失去焦点时执行。

示例：

```（java）
mport javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;
import java.awt.event.FocusEvent;
import java.awt.event.FocusListener;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class FocusListenerDemo extends JFrame {

    JList list;
    JLabel label;
    JButton button1;
    JTextField txtfield1;
    public FocusListenerDemo()
    {
        setTitle("焦点事件监听器示例");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setBounds(100,100,400,200);
        JPanel contentPane=new JPanel();
        contentPane.setBorder(new EmptyBorder(5,5,5,5));
        contentPane.setLayout(new BorderLayout(0,0));
        setContentPane(contentPane);
        label=new JLabel(" ");
        label.setFont(new Font("楷体",Font.BOLD,16));    //修改字体样式
        contentPane.add(label, BorderLayout.SOUTH);
        txtfield1=new JTextField();    //创建文本框
        txtfield1.setFont(new Font("黑体", Font.BOLD, 16));    //修改字体样式
        txtfield1.addFocusListener(new FocusListener()
        {
            @Override
            public void focusGained(FocusEvent arg0)
            {
                // 获取焦点时执行此方法
                label.setText("文本框获得焦点，正在输入内容");
            }
            @Override
            public void focusLost(FocusEvent arg0)
            {
                // 失去焦点时执行此方法
                label.setText("文本框失去焦点，内容输入完成");
            }
        });
        contentPane.add(txtfield1);
    }
    public static void main(String[] args)
    {
        FocusListenerDemo frame=new FocusListenerDemo();
        frame.setVisible(true);
    }
}

```
运行结果：

![](https://i.loli.net/2019/04/11/5caf1cba7f903.png)

上述代码为 txtfield1 组件调用 addFocusListener() 方法添加了焦点事件监听器，并且监听器使用匿名的实现方式。在实现 FocusListener 接口的代码中编写 focusGained() 方法和 focusLost() 方法的代码。

####  监听列表项选择事件

列表框控件 JList 会显示很多项供用户选择，通常在使用时会根据用户选择的列表项完成不同的操作。

本案例将介绍如何监听列表项的选择事件，以及事件监听器的处理方法，实现过程如下。

(1) 创建一个继承自 JFrame 的 JListDemo2 类。

(2) 在 JListDemo2 类中添加 JList 组件和 JLabel 组件的声明，并创建空的构造方法。

(3) 在构造方法中为列表框填充数据源

(4) 为列表框组件 list 添加选择事件监听。

(5) 创建 do_liSt_ValueChanged() 方法将用户选择的列显示到标签中。

代码示例：
```（java）
import javax.swing.*;
import javax.swing.border.EmptyBorder;
import javax.swing.event.ListSelectionEvent;
import javax.swing.event.ListSelectionListener;
import java.awt.*;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class JListDemo2 extends JFrame {
    JList list;
    JLabel label;
    public JListDemo2(){
        setTitle("监听列表项选择事件");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setBounds(100,100,400,200);
        JPanel contentPane=new JPanel();
        contentPane.setBorder(new EmptyBorder(5,5,5,5));
        contentPane.setLayout(new BorderLayout(0,0));
        setContentPane(contentPane);
        label=new JLabel(" ");
        contentPane.add(label,BorderLayout.SOUTH);
        JScrollPane scrollPane=new JScrollPane();
        contentPane.add(scrollPane,BorderLayout.CENTER);
        list=new JList();
        scrollPane.setViewportView(list);
        String[] listData=new String[7];
        listData[0]="《一点就通学Java》";
        listData[1]="《一点就通学PHP》";
        listData[2]="《一点就通学Visual Basic）》";
        listData[3]="《一点就通学Visual C++）》";
        listData[4]="《Java编程词典》";
        listData[5]="《PHP编程词典》";
        listData[6]="《C++编程词典》";
        list.setListData(listData);
        list.addListSelectionListener(new ListSelectionListener()
        {
            public void valueChanged(ListSelectionEvent e)
            {
                do_list_valueChanged(e);
            }
        });

    };

    protected void do_list_valueChanged(ListSelectionEvent e)
    {
        label.setText("感谢您购买："+list.getSelectedValue());
    }


    public static void main(String[] args)
    {
        JListDemo2 frame=new JListDemo2();
        frame.setVisible(true);
    }
}
```

运行结果：

![](https://i.loli.net/2019/04/11/5caf1c7058475.png)

### 星座选择器界面实现

在了解各种基本组件的使用，以及常见事件的处理之后，本案例将综合文本框、按钮和下拉列表组件，实现一个星座选择器程序。程序允许用户在下拉列表中选择一个自己的星座，如果不在列表中还可以增加星座，也可以删除星座.

代码如下：

```（java）
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;
import javax.swing.event.ListSelectionListener;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Vector;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class SampeDemo extends JFrame {
    Vector<String> strings;
    private JPanel panel1 = new JPanel();
    private JPanel panel2 = new JPanel();
    private JList<String> list = new JList<>();
    private JScrollPane scrollPane = new JScrollPane();    //创建滚动面板
    private JLabel label = new JLabel("添加新星座：");
    private JLabel showInfo = new JLabel();    //用于显示信息
    private JTextField jtf = new JTextField(16);    //用于输入信息
    private JButton buttonAdd = new JButton("新增");
    private JButton buttonDel = new JButton("删除");

    public SampeDemo() {
        JFrame frame = new JFrame("选择你的星座");
        frame.setLayout(new GridLayout(1, 2, 5, 5));

        panel1.setLayout(new BorderLayout(0, 0));
        panel1.add(scrollPane, BorderLayout.CENTER);    //将面板增加到边界布局中央

        list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
        scrollPane.setViewportView(list);    //在滚动面板中显示列表

        strings = new Vector<String>();
        strings.add("金牛座");
        strings.add("巨蟹座");
        strings.add("双鱼座");

        list.setListData(strings);


        panel2.setLayout(new GridLayout(5, 1));
        panel2.add(label);
        panel2.add(jtf);
        panel2.add(buttonAdd);
        panel2.add(buttonDel);

        frame.add(panel1);
        frame.add(panel2);

        buttonAdd.addActionListener(new MyActionListener());    //“添加”按钮的事件
        buttonDel.addActionListener(new MyActionListener());    //“删除”按钮的事件
        list.addListSelectionListener(new MyItemListener());    //下拉列表的事件

        frame.setBounds(300, 200, 600, 200);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }

    public static void main(String[] args) {
        new SampeDemo();    //调用构造方法
    }

    // 监听选中事件
    class MyItemListener implements ListSelectionListener {


        @Override
        public void valueChanged(ListSelectionEvent e) {
            String str = list.getSelectedValue();
            panel2.add(showInfo);
            showInfo.setText("您选择的星座是：" + str);
        }
    }

    // 监听添加和删除按钮事件
    class MyActionListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            String command = e.getActionCommand();
            //添加按钮处理
            if (command.equals("新增")) {
                String text = jtf.getText();
                if (jtf.getText().length() != 0) {
                    for (String str : strings) {
                        if (str.equals(text))

                        {
                            showInfo.setText("已存在" + text);
                            return;
                        }
                    }
                    strings.add(text);
                    list.setListData(strings);
                    panel2.add(showInfo);
                    showInfo.setText("添加成功，新增了：" + jtf.getText());
                } else {
                    panel2.add(showInfo);
                    showInfo.setText("请输入要添加星座");
                }
            }
            //删除按钮处理
            if (command.equals("删除")) {
                if (list.getSelectedIndex() != -1) {
                    //先获得要删除的项的值
                    String strDel = list.getSelectedValue();
                    strings.remove(list.getSelectedIndex());
                    list.setListData(strings);
                    panel2.add(showInfo);
                    showInfo.setText("删除成功，删除了：" + strDel);
                } else {
                    panel2.add(showInfo);
                    showInfo.setText("请选择要删除的星座");
                }
            }
        }
    }
}
```
运行效果：

![](https://i.loli.net/2019/04/11/5caf2d1f5f0fd.png)


### 布局

在前面的章节中，我们介绍了 Swing 设计简单界面所需的窗口、布局组件以及如何响应事件。Swing 还提供了很多高级组件，如菜单栏、工具栏、文件选择器、表格以及树等。使用这些高级组件可以实现更为复杂的布局，也可以使程序界面更加人性化，以提高程序的灵活性。在之后的章节中，我们将开始详细介绍这些高级组件。

在学习其他高级组件之前，我们先来介绍一些布局组件，包括滑块、进度条、计时器、菜单栏和工具栏，本节我们首先来介绍滑块。

#### 滑块组件

滑块（JSlider）是一个允许用户在有限区间内通过移动滑块来选择值的组件。常用构造方法：

![](https://i.loli.net/2019/04/11/5caf2dade9f08.png)

例如，创建一个最小值为 30，最大值为 120，初始值为 55 的水平滑块的语句如下所示。

```
    JSIider slider=new JSIider(30,120,55);
```

滑块可以显示主刻度标记以及主刻度之间的次刻度标记。刻度标记之间的值的个数由 setMajorTickSpacing() 方法和 setMinorTickSpacing() 方法来控制。刻度标记的绘制由 setPaintTicks() 方法控制。

滑块也可以在固定时间间隔（或在任意位置）沿滑块刻度打印文本标签，标签的绘制由 setLabelTable() 方法和 setPaintLabels() 方法控制。

常用方法

![](https://i.loli.net/2019/04/11/5caf2e0ed1ce2.png)

示例：
```
import javax.swing.*;
import java.awt.*;


public class JSliderDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("滑块组件示例");
        frame.setSize(400,200);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        Container contentPane=frame.getContentPane();
        JSlider slider=new JSlider(0,100);
        slider.setMajorTickSpacing(10);
        slider.setMinorTickSpacing(5);
        slider.setPaintLabels(true);
        slider.setPaintTicks(true);
        contentPane.add(slider);
        frame.setVisible(true);
    }
```

上述代码首先创建一个 JFrame 窗口并进行必要属性设置，接着创建一个 JSIider 对象，设置最小值为 0，最大值为 100，然后设置滑块对象的刻度值。

![](https://i.loli.net/2019/04/11/5caf2ea5e0220.png)


#### 进度条组件

进度条（JProgressBar）是一种以可视化形式显示某些任务进度的组件。JProgressBar 类实现了一个用于为长时间的操作提供可视化指示器的 GUI 进度条。在任务的完成进度中，进度条显示该任务完成的百分比。此百分比通常由一个矩形以可视化形式表示，该矩形开始是空的，随着任务的完成逐渐被填充。此外，进度条可显示此百分比的文本表示形式。

JProgressBar 类的常用构造方法和 JSlider 类的常用构造方法一样，这里不再重复。如下示例代码演示了如何创建一个 JProgressBar 类实例。

```
//创建一个最小值是0，最大值是100的进度条
JProgressBar pgbar=new JProgressBar(0,100);
//创建一个最小值是0，最大值是60，当前值是20的进度条
JProgressBar pgbar=new JProgressBar(0,60,20);
```
常用方法：

![](https://i.loli.net/2019/04/11/5caf2f1a52ad5.png)


其中，setOrientation() 方法的参数值必须为 SwingConstants.VERTICAL 或 SwingConstants.HORIZONTAL。JProgressBar 使用 BoundedRangeModel 作为其数据模型，以 value 属性表示该任务的“当前”状态，minimum 和 maximum 属性分别表示开始点和结束点。

技巧：如果要执行一个未知长度的任务，可以调用 setlndeterminate(true) 将进度条设置为不确定模式。不确定模式的进度条将持续地显示动画来表示正进行的操作。一旦可以确定任务长度和进度量，则应该更新进度条的值，将其切换到确定模式。

```（java）
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class JProgressBarDemo extends JFrame {

    //static JProgressBarDemo frame;
    public JProgressBarDemo()
    {
        setTitle("使用进度条");
        JLabel label=new JLabel("欢迎使用在线升级功能！");
        //创建一个进度条
        JProgressBar progressBar=new JProgressBar();
        JButton button=new JButton("完成");
        button.setEnabled(false);
        Container container=getContentPane();
        container.setLayout(new GridLayout(3,1));
        JPanel panel1=new JPanel(new FlowLayout(FlowLayout.LEFT));
        JPanel panel2=new JPanel(new FlowLayout(FlowLayout.CENTER));
        JPanel panel3=new JPanel(new FlowLayout(FlowLayout.RIGHT));
        panel1.add(label);    //添加标签
        panel2.add(progressBar);    //添加进度条
        panel3.add(button);    //添加按钮
        container.add(panel1);
        container.add(panel2);
        container.add(panel3);
        progressBar.setStringPainted(true);
        //如果不需要进度上显示“升级进行中...”，可注释此行
        progressBar.setString("升级进行中...");
        //如果需要使用不确定模式，可使用此行
        //progressBar.setIndeterminate(true);
        //开启一个线程处理进度
        new Progress(progressBar, button).start();
        //单机“完成”按钮结束程序
        button.addActionListener(new ActionListener()
        {
            @Override
            public void actionPerformed(ActionEvent e)
            {
                dispose();
                System.exit(0);
            }
        });
        setSize(200,100);
    }
    /**
     * @param args
     */
    public static void main(String[] args)
    {
        // TODO Auto-generated method stub
        JProgressBarDemo frame=new JProgressBarDemo();
        //frame.setBounds(300,200,300,150);    //设置容器的大小
        frame.setVisible(true);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.pack();
    }

    private class Progress extends Thread
    {
        JProgressBar progressBar;
        JButton button;
        //进度条上的数字
        int[] progressValues={6,18,27,39,51,66,81,100};
        Progress(JProgressBar progressBar,JButton button)
        {
            this.progressBar=progressBar;
            this.button=button;
        }
        public void run()
        {
            for(int i=0;i<progressValues.length;i++)
            {
                try
                {
                    Thread.sleep(3000);
                }
                catch(InterruptedException e)
                {
                    e.printStackTrace();
                }
                //设置进度条的值
                progressBar.setValue(progressValues[i]);
            }
            progressBar.setIndeterminate(false);
            progressBar.setString("升级完成！");
            button.setEnabled(true);
        }
    }
}
```

上述代码定义了一个进度条的进度数组 progressValues。线程每隔 1000 毫秒从数组中取一个数字作为当前进度，并使用 JProgressBar 类的 setValue() 方法更新到进度条。最后使进度条显示“升级完成！”字符串，并使“完成”按钮可用。

#### 计时器

计时器（Timer）组件可以在指定时间间隔触发一个或多个 ActionEvent。设置计时器的过程包括创建一个 Timer 对象，在该对象上注册一个或多个动作侦听器，以及使用 start() 方法启动该计时器。

例如，以下代码创建并启动一个每秒（该时间由 Timer 构造方法的第一个参数指定）触发一次动作事件的计时器。Timer 构造方法的第二个参数指定接收计时器动作事件的监听器。

```
int delay=1000;    //时间间隔，单位为毫秒
ActionListener taskPerformer=new ActionListener()
{
    public void afrfcionPerformed(ActionEvent evt)
    {
        //具体的任务
    }
};
new Timer(delay,taskPerformer).start();
```
创建 Timer 类时要指定一个延迟参数和一个 ActionListener。延迟参数用于设置初始延迟和事件触发之间的延迟（以毫秒为单位）。启动计时器后，它将在向已注册监听器触发第一个 ActionEvent 之前等待初始延迟。第一个事件之后，每次超过事件间延迟时它都继续触发事件，直到被停止。

创建 Timer 类之后，可以单独更改初始延迟和事件间延迟，并且可以添加其他 ActionListener。如果希望计时器只在第一次时触发然后停止，可以对计时器调用 setRepeats(false)。

常用用法:

![](https://i.loli.net/2019/04/11/5caf308b14bac.png)


示例

```（java）

import javax.swing.*;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class JProgressBarDemo1  implements ActionListener,ChangeListener {

    JFrame frame=null;
    JProgressBar progressbar;
    JLabel label;
    Timer timer;
    JButton b;
    public JProgressBarDemo1()
    {
        frame=new JFrame("软件安装");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        Container contentPane=frame.getContentPane();
        label=new JLabel(" ",JLabel.CENTER);    //创建显示进度信息的文本标签
        progressbar=new JProgressBar();    //创建一个进度条
        progressbar.setOrientation(JProgressBar.HORIZONTAL);
        progressbar.setMinimum(0);
        progressbar.setMaximum(100);
        progressbar.setValue(0);
        progressbar.setStringPainted(true);
        progressbar.addChangeListener(this);    //添加事件监听器
        //设置进度条的几何形状
        progressbar.setPreferredSize(new Dimension(300,20));
        progressbar.setBorderPainted(true);
        progressbar.setBackground(Color.pink);
        //添加启动按钮
        JPanel panel=new JPanel();
        b=new JButton("安装");
        b.setForeground(Color.blue);
        //添加事件监听器
        b.addActionListener(this);
        panel.add(b);
        timer=new Timer(100, this);    //创建一个计时器，计时间隔为100毫秒
        //把组件添加到frame中
        contentPane.add(panel,BorderLayout.NORTH);
        contentPane.add(progressbar,BorderLayout.CENTER);
        contentPane.add(label,BorderLayout.SOUTH);
        frame.pack();
        frame.setVisible(true);
    }
    //实现事件监听器接口中的方法
    public void actionPerformed(ActionEvent e)
    {
        if(e.getSource()==b)
            timer.start();
        if(e.getSource()==timer)
        {
            int value=progressbar.getValue();
            if(value<100)
            {
                progressbar.setValue(++value);
            }
            else
            {
                timer.stop();
                frame.dispose();
            }
        }
    }

    public void stateChanged(ChangeEvent e1)    //实现事件监听器接口中的方法
    {
        int value=progressbar.getValue();
        if(e1.getSource()==progressbar)
        {
            label.setText("目前已完成进度："+Integer.toString(value)+" %");
            label.setForeground(Color.blue);
        }
    }


    public static void main(String[] agrs)
    {
        new JProgressBarDemo1();    //创建一个实例化对象
    }
}
```

#### 菜单和弹出式菜单

菜单由 Swing 中的 JMenu 类实现，可以包含多个菜单项和带分隔符的菜单。在菜单中，菜单项由 JMenuItem 类表示，分隔符由 JSeparator 类表示。

菜单本质上是带有关联 JPopupMenu 的按钮。当按下“按钮”时，就会显示 JPopupMenu。如果“按钮”位于 JMenuBar 上，则该菜单为顶层窗口。如果“按钮”是另一个菜单项，则 JPopupMenu 就是“下拉”菜单。


**JMenu 类的常用方法**

创建菜单常用构造方法有两个：JMenu() 和 JMenu(String s)。第一个构造方法创建一个无文本的 JMenu 对象，第二个构造方法创建一个带有指定文本的 JMenu 对象。

常用方法：

![](https://i.loli.net/2019/04/11/5caf508f5933f.png)

示例：

```（java）
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.KeyEvent;

public class JMenuDemo1 extends JMenuBar {
    public JMenuDemo1()
    {
        add(createFileMenu());    //添加“文件”菜单
        add(createEditMenu());    //添加“编辑”菜单
        setVisible(true);
    }
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("菜单栏");
        frame.setSize(300,200);
        frame.setJMenuBar(new JMenuDemo1());
        frame.setVisible(true);
    }
    //定义“文件”菜单
    private JMenu createFileMenu()
    {
        JMenu menu=new JMenu("文件(F)");
        menu.setMnemonic(KeyEvent.VK_F);    //设置快速访问符
        JMenuItem item=new JMenuItem("新建(N)",KeyEvent.VK_N);
        item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_N,ActionEvent.CTRL_MASK));
        menu.add(item);
        item=new JMenuItem("打开(O)",KeyEvent.VK_O);
        item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_O,ActionEvent.CTRL_MASK));
        menu.add(item);
        item=new JMenuItem("保存(S)",KeyEvent.VK_S);
        item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S,ActionEvent.CTRL_MASK));
        menu.add(item);
        menu.addSeparator();
        item=new JMenuItem("退出(E)",KeyEvent.VK_E);
        item.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_E,ActionEvent.CTRL_MASK));
        menu.add(item);
        return menu;
    }
    //定义“编辑”菜单
    private JMenu createEditMenu()
    {
        JMenu menu=new JMenu("编辑(E)");
        menu.setMnemonic(KeyEvent.VK_E);
        JMenuItem item=new JMenuItem("撤销(U)",KeyEvent.VK_U);
        item.setEnabled(false);
        menu.add(item);
        menu.addSeparator();
        item=new JMenuItem("剪贴(T)",KeyEvent.VK_T);
        menu.add(item);
        item=new JMenuItem("复制(C)",KeyEvent.VK_C);
        menu.add(item);
        menu.addSeparator();
        JCheckBoxMenuItem cbMenuItem=new JCheckBoxMenuItem("自动换行");
        menu.add(cbMenuItem);
        return menu;
    }
}
```


上述代码调用 JMenu 对象的 setMnemonic() 方法设置当前菜单的快速访问符。该符号必须对应键盘上的一个键，并且应该使用 java.awt.event.KeyEvent 中定义的 VK—XXX 键代码之一指定。

提示：快速访问符是一种快捷键，通常在按下 Alt 键和某个字母时激活。例如，常用的 Alt+F 是“文件” 菜单的快速访问符。

JMenuItem 类实现的是菜单中的菜单项。菜单项本质上是位于列表中的按钮。当用户单击“按钮”时，则执行与菜单项关联的操作。JMenuItem 的常用构造方法有以下三个。
```
    JMenuItem(String text)：创建带有指定文本的 JMenuItem。
    JMenuItem(String text,Icon icon)：创建带有指定文本和图标的 JMenuItem。
    JMenuItem(String text,int mnemonic)：创建带有指定文本和键盘助记符的 JMenuItem。
```

在该实例中，创建菜单项后调用 JMenuItem 对象的 setAccelerator(KeyStroke) 方法来设置修改键，它能直接调用菜单项的操作监听器而不必显示菜单的层次结构。在本实例中没有实现事件监听机制，所以使用快捷键时将得不到程序的任何响应，但是在菜单项中将出现快捷键。

![](https://i.loli.net/2019/04/11/5caf51a2c2c04.png)

**弹出式菜单 JPopuMenu**

弹出式菜单由 JPopupMenu 类实现，它是一个可弹出并显示一系列选项的小窗口。它还用于当用户选择菜单项并激活它时显示的“右拉式(pull-right)”菜单，可以在想让菜单显示的任何其他位置使用。例如，当用户在指定区域中右击时。

常用用法：

![](https://i.loli.net/2019/04/11/5caf51b6cc5a9.png)


示例：

```（java）
import javax.swing.*;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class JPopupMenuDemo extends JFrame {
    JMenu fileMenu;
    JPopupMenu jPopupMenuOne;
    JMenuItem openFile,closeFile,exit;
    JRadioButtonMenuItem copyFile,pasteFile;
    ButtonGroup buttonGroupOne;
    public JPopupMenuDemo()
    {
        jPopupMenuOne=new JPopupMenu();    //创建jPopupMenuOne对象
        buttonGroupOne=new ButtonGroup();
        //创建文件菜单及子菜单，并将子菜单添加到文件菜单中
        fileMenu=new JMenu("文件");
        openFile=new JMenuItem("打开");
        closeFile=new JMenuItem("关闭");
        fileMenu.add(openFile);
        fileMenu.add(closeFile);
        //将fileMenu菜单添加到弹出式菜单中
        jPopupMenuOne.add(fileMenu);
        //添加分割符
        jPopupMenuOne.addSeparator();
        //创建单选菜单项，并添加到ButtonGroup对象中
        copyFile=new JRadioButtonMenuItem("复制");
        pasteFile=new JRadioButtonMenuItem("粘贴");
        buttonGroupOne.add(copyFile);
        buttonGroupOne.add(pasteFile);
        //将copyFile添加到jPopupMenuOne中
        jPopupMenuOne.add(copyFile);
        //将pasteFile添加到jPopupMenuOne中
        jPopupMenuOne.add(pasteFile);
        jPopupMenuOne.addSeparator();
        exit=new JMenuItem("退出");
        //将exit添加到jPopupMenuOne中
        jPopupMenuOne.add(exit);
        //创建监听器对象
        MouseListener popupListener=new PopupListener(jPopupMenuOne);
        //向主窗口注册监听器
        this.addMouseListener(popupListener);
        this.setTitle("弹出式菜单");
        this.setBounds(100,100,250,150);
        this.setVisible(true);
        this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
    public static void main(String args[])
    {
        new JPopupMenuDemo();
    }
    //添加内部类，其扩展了MouseAdapter类，用来处理鼠标事件
    class PopupListener extends MouseAdapter
    {
        JPopupMenu popupMenu;
        PopupListener(JPopupMenu popupMenu)
        {
            this.popupMenu=popupMenu;
        }
        public void mousePressed(MouseEvent e)
        {
            showPopupMenu(e);
        }
        public void mouseReleased(MouseEvent e)
        {
            showPopupMenu(e);
        }
        private void showPopupMenu(MouseEvent e)
        {
            if(e.isPopupTrigger())
            {
                //如果当前事件与鼠标事件相关，则弹出菜单
                popupMenu.show(e.getComponent(),e.getX(),e.getY());
            }
        }
    }
}
```

在使用弹出菜单时一定要注意层次关系和菜单的位置。程序运行后由于菜单没有被激活所以窗口是空白的。

### 工具栏

工具栏提供了一个用来显示常用按钮和操作的组件。它可以把任意类型的组件附加到工具条上，但是通常是增加按钮。

JToolBar的构造方法：

![](https://i.loli.net/2019/04/11/5caf53ae7f804.png)

与 JMenuBar 不一样，JToolBar 对象可以直接被添加到容器中。

常用方法：

![](https://i.loli.net/2019/04/11/5caf542db53bc.png)

示例：

```（java）
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.net.URL;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class ToolBarDemo extends JPanel implements ActionListener {
    protected JTextArea textArea;
    protected String newline="\n";
    static final private String OPEN="OPEN";
    static final private String SAVE="SAVE";
    static final private String NEW="NEW";
    //事件监听器部分的代码省略，请查阅源文件
    protected void displayResult(String actionDescription)
    {
        textArea.append(actionDescription+newline);
    }
    public static void main(String[] args)
    {
        JFrame.setDefaultLookAndFeelDecorated(true);
        //定义窗体
        JFrame frame=new JFrame("工具栏");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        //定义面板
        ToolBarDemo newContentPane=new ToolBarDemo();
        newContentPane.setOpaque(true);
        frame.setContentPane(newContentPane);
        //显示窗体
        frame.pack();
        frame.setVisible(true);
    }
    @Override
    public void actionPerformed(ActionEvent e)
    {
        // TODO 自动生成的方法存根
    }

    public ToolBarDemo()
    {
        super(new BorderLayout());
        //创建工具栏
        JToolBar toolBar=new JToolBar();
        addButtons(toolBar);
        //创建一个文本域，用来输出一些信息
        textArea=new JTextArea(15, 30);
        textArea.setEditable(false);
        JScrollPane scrollPane=new JScrollPane(textArea);
        //把组件添加到面板中
        setPreferredSize(new Dimension(450, 110));
        add(toolBar,BorderLayout.PAGE_START);
        add(scrollPane,BorderLayout.CENTER);
    }
    protected void addButtons(JToolBar toolBar)
    {
        JButton button=null;
        button=makeNavigationButton("new1",NEW,"新建一个文件","新建");
        toolBar.add(button);
        button=makeNavigationButton("open1",OPEN,"打开一个文件","打开");
        toolBar.add(button);
        button=makeNavigationButton("save1",SAVE,"保存当前文件","保存");
        toolBar.add(button);
    }
    protected JButton makeNavigationButton(String imageName,String actionCommand,String toolTipText,String altText)
    {
        //搜索图片
        String imgLocation=imageName+".jpg";
        URL imageURL=ToolBarDemo.class.getResource(imgLocation);
        //初始化工具按钮
        JButton button=new JButton();
        //设置按钮的命令
        button.setActionCommand(actionCommand);
        //设置提示信息
        button.setToolTipText(toolTipText);
        button.addActionListener(this);
        if(imageURL!=null)
        {
            //找到图像
            button.setIcon(new ImageIcon(imageURL));
        }
        else
        {
            //没有图像
            button.setText(altText);
            System.err.println("Resource not found: "+imgLocation);
        }
        return button;
    }
}
```

### 文件选择器和颜色选择器

在开发应用程序时经常需要选择文件和选择颜色的功能。例如，从选择的文件中导入数据，为窗体选择背景颜色等。

**文件选择器**

文件选择器为用户能够操作系统文件提供了桥梁。swing 中使用 JFileChooser 类实现文件选择器，该类常用的构造方法如下。
```
   JFileChooser()：创建一个指向用户默认目录的 JFileChooser。
   JFileChooser(File currentDirectory)：使用指定 File 作为路径来创建 JFileChooser。
   JFileChooser(String currentDirectoryPath)：创建一个使用指定路径的 JFileChooser。
   JFileChooser(String currentDirectoryPath, FileSystemView fsv)：使用指定的当前目录路径和 FileSystem View 构造一个 JFileChooser。
```

JFileChooser 类的常用方法如下所示。

```
   int showOpenDialog(Component parent)：弹出打开文件对话框。
   int showSaveDialog(Component parent)：弹出保存文件对话框。
```
示例：

```（java）
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class JFileChooserDemo {
    private JLabel label=new JLabel("所选文件路径：");
    private JTextField jtf=new JTextField(25);
    private JButton button=new JButton("浏览");
    public JFileChooserDemo()
    {
        JFrame jf=new JFrame("文件选择器");
        JPanel panel=new JPanel();
        panel.add(label);
        panel.add(jtf);
        panel.add(button);
        jf.add(panel);
        jf.pack();    //自动调整大小
        jf.setVisible(true);
        jf.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        button.addActionListener(new MyActionListener());    //监听按钮事件
    }
    //Action事件处理
    class MyActionListener implements ActionListener
    {
        @Override
        public void actionPerformed(ActionEvent arg0)
        {
            JFileChooser fc=new JFileChooser("F:\\");
            int val=fc.showOpenDialog(null);    //文件打开对话框
            if(val==fc.APPROVE_OPTION)
            {
                //正常选择文件
                jtf.setText(fc.getSelectedFile().toString());
            }
            else
            {
                //未正常选择文件，如选择取消按钮
                jtf.setText("未选择文件");
            }
        }
    }
    public static void main(String[] args)
    {
        new JFileChooserDemo();
    }
}
```

在上述程序中使用内部类的形式创建了一个名称为 MyActionListener 的类，该类实现了 ActionListener 接口。其中 showOpenDialog() 方法将返回一个整数，可能取值情况有 3 种：JFileChooser.CANCEL—OPTION、JFileChooser.APPROVE_OPTION 和 JFileChooser.ERROR_OPTION，分别用于表示单击“取消”按钮退出对话框，无文件选取、正常选取文件和发生错误或者对话框已被解除而退出对话框。因此在文本选择器交互结束后，应进行判断是否从对话框中选择了文件，然后根据返回值情况进行处理。

使用 JFileChooser 对象调用 showSaveDialog() 方法会显示保存文件对话框，即将“int val=fc.showOpenDialog(null);”语句换成“int val=fc.showSaveDialog(null);”。在保存文件对话框中“保存”按钮对应的常量值是 JFileChooser.APPROVE_OPTION，“取消”按钮对应的常量值是JFileChooser.CANCEL_ OPTION


**颜色选择器**

JColorChooser 类提供一个用于允许用户操作和选择颜色的控制器窗格。该类提供三个级别的 API：
```
   显示有模式颜色选取器对话框并返回用户所选颜色的静态便捷方法。
   创建颜色选取器对话框的静态方法，可以指定当用户单击其中一个对话框按钮时要调用的 ActionListener。
   能直接创建 JColorChooser 窗格的实例（在任何容器中），可以添加 PropertyChange 作为监听器以检测当前“颜色”属性的更改。
```

颜色选择器的常用构造方法如下。
```
   JColorChooser()：创建初始颜色为白色的颜色选取器窗格。
   JColorChooser(Color initialColor)：创建具有指定初始颜色的颜色选取器窗格。
   JColorChooser(ColorSelectionModel model)：创建具有指定 ColorSelectionModel 颜色选取器窗格。
```

一般使用 JColorChooser 类的静态方法 showDialog(Component component,String title,Color initialColor) 创建一个颜色对话框，在隐藏对话框之前一直堵塞进程。其中 component 参数指定对话框所依赖的组件，title 参数指定对话框的标题，initialColor 参数指定对话框返回的初始颜色，即对话框消失后返回的默认值。

用法：

![](https://i.loli.net/2019/04/11/5caf56178cf8c.png)

示例：

```（java）
import javax.swing.*;
import java.awt.*;

public class JColorChooserDemo {
    public static void main(String[] args)
    {
        JFrame frame=new JFrame("颜色选择器");
        JColorChooser cc=new JColorChooser();
        cc.showDialog(frame,"颜色选择器",Color.white);
        //JColorChooser.showDialog(frame,"颜色选择器",Color.white);
        //设置窗口的关闭动作、标题、大小位置以及可见性等
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setBounds(100,100,400,200);
        frame.setVisible(true);
        frame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}
```

使用 JFrame 作为容器，在该容器内显示一个标题是“颜色选择器”、使用白色作为默认色的颜色选择器。

 也可以不创建 JColorChooser 类实例，直接调用其 showDialog() 方法显示颜色选择器。即将如下代码
```
JFrame frame=new JFrame("颜色选择器");
JColorChooser cc=new JColorChooser();
cc.showDialog(frame,"颜色选择器",Color.white);
```
换成：
```
JColorChooser.showDialog(frame,"颜色选择器",Color.white);
```

### 表格

表格是 Swing 新增加的组件，主要功能是把数据以二维表格的形式显示出来，并且允许用户对表格中的数据进行编辑。表格组件是最复杂的组件之一，它的表格模型功能非常强大、灵活而易于执行。

Swing 使用 JTable 类实现表格，常用构造方法如下所示。
```
    JTable()：构造一个默认的 JTable，使用默认的数据模型、默认的列模型和默认的选择模型对其进行初始化。
    JTable(int numRows,int numColumns)：使用 DefaultTableModel 构造具有 numRows 行和 numColumns 列个空单元格的 JTable。
    JTable(Object[][] rowData,Object[] columnNames)：构造一个 JTable 来显示二 维数组 rowData 中的值，其列名称为 columnNames。
```
创建一个带有滚动条的 JTable 对象非常简单，如下所示。
```
JTable table=new JTable(5,6);
JScrollPane pane=new JScrollPane(table);
```
第一条语句创建了一个 JTable 对象。第二条语句创建了一个存放 JTable 对象的 JScrollPane 对象，该对象是一个视图对象。JScrollPane是一个垂直和水平滚动条，以及可设置行和列标题的容器。

表格有很多的选项设置，因此 JTable 类常用方法也很多。

![](https://i.loli.net/2019/04/11/5caf57704107d.png)

示例:

```(java)
import javax.swing.*;
import java.awt.*;

public class JTableDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("学生成绩表");
        frame.setSize(500,200);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        Container contentPane=frame.getContentPane();
        Object[][] tableDate=new Object[5][8];
        for(int i=0;i<5;i++)
        {
            tableDate[i][0]="1000"+i;
            for(int j=1;j<8;j++)
            {
                tableDate[i][j]=0;
            }
        }
        String[] name={"学号","软件工程","Java","网络","数据结构","数据库","总成绩","平均成绩"};
        JTable table=new JTable(tableDate,name);
        contentPane.add(new JScrollPane(table));
        frame.setVisible(true);
    }
}

```
如上述代码所示，表格组件和其他组件类似，可以方便地创建一个 JTable 对象。 如果 JTbale 对象直接添加到 JFrame 中，则表头显示不出来，需要把表格对象放入 JScrollPane 对象中，之后把 JScrollPane 对象添加到 JFrame 中。


Swing 中表格的数据可以根据需求动态变化，比如对于表格中不需要的数据，可以将其删除。本实例将演示如何从表格中删除用户选择的行，实现过程如下。

示例：

```（java）
import javax.swing.*;
import javax.swing.border.EmptyBorder;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class RowDeleteDemo extends JFrame {
    private JPanel contentPane;
    private JTable table;
    /**
     * Launch the application.
     */
    public static void main(String[] args)
    {
        RowDeleteDemo frame = new RowDeleteDemo();
        frame.setVisible(true);
    }

    public RowDeleteDemo()
    {
        addWindowListener(new WindowAdapter()
        {
            @Override
            public void windowActivated(WindowEvent e)
            {
                do_this_windowActivated(e);
            }
        });
        setTitle("图书信息表");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setBounds(100,100,450,200);
        contentPane=new JPanel();
        contentPane.setBorder(new EmptyBorder(5,5,5,5));
        contentPane.setLayout(new BorderLayout(0,0));
        setContentPane(contentPane);
        JPanel panel=new JPanel();
        contentPane.add(panel,BorderLayout.SOUTH);
        JButton button=new JButton("删除");
        button.addActionListener(new ActionListener()
        {
            public void actionPerformed(ActionEvent e)
            {
                do_button_actionPerformed(e);
            }
        });
        panel.add(button);
        JScrollPane scrollPane=new JScrollPane();
        contentPane.add(scrollPane,BorderLayout.CENTER);
        table=new JTable();
        table.setSelectionMode(ListSelectionModel.SINGLE_INTERVAL_SELECTION);
        scrollPane.setViewportView(table);
    }

    protected void do_this_windowActivated(WindowEvent e)
    {
        DefaultTableModel tableModel=(DefaultTableModel) table.getModel();    //获得表格模型
        tableModel.setRowCount(0);    //清空表格中的数据
        tableModel.setColumnIdentifiers(new Object[]{"书名","出版社","出版时间","丛书类别","定价"});    //设置表头
        tableModel.addRow(new Object[]{"Java从入门到精通（第2版）","清华大学出版社","2010-07-01","软件工程师入门丛书","59.8元"});    //增加列
        tableModel.addRow(new Object[]{"PHP从入门到精通（第2版）","清华大学出版社","2010-07-01","软件工程师入门丛书","69.8元"});
        tableModel.addRow(new Object[]{"Visual Basic从入门到精通（第2版）","清华大学出版社","2010-07-01","软件工程师入门丛书","69.8元"});
        tableModel.addRow(new Object[]{"Visual C++从入门到精通（第2版）","清华大学出版社","2010-07-01","软件工程师入门丛书","69.8元" });
        table.setRowHeight(30);
        table.setModel(tableModel);    //应用表格模型
    }

    protected void do_button_actionPerformed(ActionEvent e)
    {
        DefaultTableModel model=(DefaultTableModel) table.getModel();    //获得表格模型
        int[] selectedRows=table.getSelectedRows();
        for(int i=selectedRows[0];i<selectedRows.length;i++)
        {
            model.removeRow(selectedRows[0]);
        }
        table.setModel(model);
    }
}
```

### 树

如果要显示一个层次关系分明的一组数据，用树结构是最合适的。树如同 Windows 资源管理器的左半部，可通过单击文件夹展开或者收缩内容。

Swing 使用 JTree 类实现树，它的主要功能是把数据按照树状进行显示，其数据来源于其他对象。JTree 树中最基本的对象叫作节点，表示在给定层次结构中的数据项。树以垂直方式显示数据，每行显示一个节点。树中只有一个根节点，所有其他节点从这里引出。除根节点外，其他节点分为两类：一类是代子节点的分支节点，另一类是不带子节点的叶节点。


![](https://i.loli.net/2019/04/11/5caf58f26f122.png)

树节点由 javax.swing.tree 包中的接口 TreeNode 定义，该接口被 DefaultMutableTreeNode 类实现。

为了创建一个树，使用 DefaultMutableTreeNode 类为树创建节点，它的两个常用的构造方法如下。

```
    DefaultMutableTreeNode(Object userObject)：创建没有父节点和子节点，但允许有子节点的树节点，并使用指定的用户对象对它进行初始化。
    DefaultMutableTreeNode(Object userObject,boolean allowsChildren)：创建没有父节点和子节点的树节点，使用指定的用户对象对它进行初始化，仅在指定时才允许有子节点。
```

示例：

```（java）
import javax.swing.*;
import javax.swing.tree.DefaultMutableTreeNode;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class JTreeDemo {
    public static void main(String[] agrs)
    {
        JFrame frame=new JFrame("教师学历信息");
        frame.setSize(400,300);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.getContentPane().add(new JTreeDemo().createComponent());
        frame.pack();
        frame.setVisible(true);
    }
    private JPanel createComponent()
    {
        JPanel panel=new JPanel();
        DefaultMutableTreeNode root=new DefaultMutableTreeNode("教师学历信息");
        String Teachers[][]=new String[3][];
        Teachers[0]=new String[]{"王鹏","李曼","韩小国","穆保龄","尚凌云","范超峰"};
        Teachers[1]=new String[]{"胡会强","张春辉","宋芳","阳芳","朱山根","张茜","宋媛媛"};
        Teachers[2]=new String[]{"刘丹","张小芳","刘华亮","聂来","吴琼"};
        String gradeNames[]={"硕士学历","博士学历","博士后学历"};
        DefaultMutableTreeNode node=null;
        DefaultMutableTreeNode childNode=null;
        int length=0;
        for(int i=0;i<3;i++)
        {
            length=Teachers[i].length;
            node=new DefaultMutableTreeNode(gradeNames[i]);
            for (int j=0;j<length;j++)
            {
                childNode=new DefaultMutableTreeNode(Teachers[i][j]);
                node.add(childNode);
            }
            root.add(node);
        }
        JTree tree=new JTree(root);
        panel.add(tree);
        panel.setVisible(true);
        return panel;
    }
}

```

### 选项卡

使用选项卡可以在有限的布局空间内展示更多的内容。Swing 使用 JTabbedPane 类实现选项卡。

JTabbedPane 类创建的选项卡可以通过单击标题或者图标在选项卡之间进行切换。JTabbedPane 类的常用构造方法如下所示。
```
    JTabbedPane()：创建一个具有默认 JTabbedPane.TOP 布局的空 TabbedPane。
    JTabbedPane(int tabPlacement)：创建一个空的 TabbedPane，使其具有以下指定选项卡布局中的一种：JTabbedPane.TOP、JTabbedPane.BOTTOM、JTabbedPane.LEFT 或 JTabbedPane.RIGHT。
```

创建了 JTabbedPane 实例之后，可使用 addTab() 方法和 insertTab() 方法将选项卡/组件添加到 TabbedPane 对象中。选项卡通过对应于添加位置的索引来表示，其中第一个选项卡的索引为 0，最后一个选项卡的索引为选项卡数量减 1。

TabbedPane 使用 SingleSelectionModel 属性来表示选项卡索引集和当前所选择的索引。如果选项卡数量大于 0，则总会有一个被选定的索引，此索引默认被初始化为第一个选项卡；如果选项卡数量为 0，则所选择的索引为 -1。

![](https://i.loli.net/2019/04/11/5caf59c81deb8.png)

选项卡面板和卡片布局不同的是，选项卡面板可以有标签。下面的示例代码创建了一个选项卡面板，并在选项卡面板中添加了一个 JPand 面板。

```
JTabbedPane tabbedPane=new JTabbedPane();
ImageIcon icon=new ImageIcon("temp.gif");
JComponent panel1=makeTextPanel("Panel#1");    //创建一个jPanel容器，容纳其他组件
tabbedPane.addTab("Tab 1",icon,panel1,"Does nothing");
tabbedPane.setMnemonicAt(0,KeyEvent.VK_1);    //设置快捷键
```
该代码段中的第三条语句向选项卡面板 tabbedPane 中添加了一个 panel1 组件（该组件是一个 JPanel 对象），该方法中的第一个参数是选项卡标签文本；第二个参数是 Icon 对象，它作为选项卡标签上的图标；第三个参数是添加到选项卡上的组件；第四个参数是当鼠标指针放在选项卡标签上时出现的提示信息。

示例：

```（java）
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class TabbedPaneDemo extends JPanel{

    public static void main(String[] args)
    {
        JFrame frame=new JFrame("我的电脑 - 属性");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.add(new TabbedPaneDemo(),BorderLayout.CENTER);
        frame.pack();
        frame.setVisible(true);
    }
    public TabbedPaneDemo()
    {
        super(new GridLayout(1,1));
        JTabbedPane tabbedPane=new JTabbedPane();
        ImageIcon icon=createImageIcon("tab.jp1g");
        JComponent panel1=makeTextPanel("计算机名");
        tabbedPane.addTab("计算机名",icon, panel1,"Does nothing");
        tabbedPane.setMnemonicAt(0,KeyEvent.VK_1);
        JComponent panel2=makeTextPanel("硬件");
        tabbedPane.addTab("硬件",icon,panel2,"Does twice as much nothing");
        tabbedPane.setMnemonicAt(1,KeyEvent.VK_2);
        JComponent panel3=makeTextPanel("高级");
        tabbedPane.addTab("高级",icon,panel3,"Still does nothing");
        tabbedPane.setMnemonicAt(2,KeyEvent.VK_3);
        JComponent panel4=makeTextPanel("系统保护");
        panel4.setPreferredSize(new Dimension(410,50));
        tabbedPane.addTab("系统保护",icon,panel4,"Does nothing at all");
        tabbedPane.setMnemonicAt(3,KeyEvent.VK_4);
        add(tabbedPane);
    }
    protected JComponent makeTextPanel(String text)
    {
        JPanel panel=new JPanel(false);
        JLabel filler=new JLabel(text);
        filler.setHorizontalAlignment(JLabel.CENTER);
        panel.setLayout(new GridLayout(1,1));
        panel.add(filler);
        return panel;
    }
    protected static ImageIcon createImageIcon(String path)
    {
        java.net.URL imgURL=TabbedPaneDemo.class.getResource(path);
        if(imgURL!=null)
        {
            return new ImageIcon(imgURL);
        }
        else
        {
            System.err.println("Couldn't find file: "+path);
            return null;
        }
    }
}
```


### 文本编辑器

源代码：

```（java）
import javax.swing.*;
import javax.swing.filechooser.FileNameExtensionFilter;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;

/**
 * @author hyp
 * Project name is hello-world
 * Include in com.hyp.learn.swing
 * hyp create at 19-4-11
 **/
public class TextFileOpener extends JFrame {
    private static final long serialVersionUID=-9077023825514749548L;
    private JTextArea ta_showText;    //定义显示文件属性的文本域
    private JTextArea ta_showProperty;    //定义显示文件内容的文本域
    //Launch the application.
    public static void main(String[] args)
    {
        TextFileOpener frame=new TextFileOpener();
        frame.setVisible(true);
    }

    public TextFileOpener()
    {
        setTitle("文本编辑器");    //设置窗体标题
        setBounds(100,100,400,250);    //设置窗体位置和大小
        setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);    //设置窗体默认关闭方式
        final JMenuBar menuBar=new JMenuBar();    //创建菜单栏
        setJMenuBar(menuBar);    //把菜单栏放到窗体上
        final JMenu mn_file=new JMenu();    //创建文件菜单
        mn_file.setText("文件");    //为文件菜单设置标题
        menuBar.add(mn_file);    //把文件菜单添加到菜单栏上
        final JMenuItem mi_open=new JMenuItem();    //创建打开菜单项
        mi_open.addActionListener(new ActionListener()
        {
            //为打开菜单项添加监听器
            public void actionPerformed(final ActionEvent arg0)
            {
                openTextFile();    //调用方法，操作文件
            }
        });
        mi_open.setText("打开");    //设置打开菜单项的标题
        mn_file.add(mi_open);    //把打开菜单项添加到文件菜单
        mn_file.addSeparator();    //添加菜单分隔符
        final JMenuItem mi_exit=new JMenuItem();    //创建退出菜单项
        mi_exit.addActionListener(new ActionListener()
        {
            //为退出菜单项添加监听器
            public void actionPerformed(final ActionEvent arg0)
            {
                System.exit(0);    //退出系统
            }
        });
        mi_exit.setText("退出");    //设置退出菜单项的标题
        mn_file.add(mi_exit);    //把退出菜单项添加到文件菜单
        final JMenu mn_edit=new JMenu();    //创建编辑菜单
        mn_edit.setText("编辑");    //为编辑菜单设置标题
        menuBar.add(mn_edit);    //把编辑菜单添加到菜单栏上
        final JMenuItem mi_copy=new JMenuItem();    //创建复制菜单项
        mi_copy.setText("复制");    //设置复制菜单项的标题
        mn_edit.add(mi_copy);    //把复制菜单项添加到编辑菜单
        final JMenuItem mi_cut=new JMenuItem();    //创建剪切菜单项
        mi_cut.setText("剪切");    //设置剪切菜单项的标题
        mn_edit.add(mi_cut);    //把剪切菜单项添加到编辑菜单
        final JMenuItem mi_paste=new JMenuItem();    //创建粘贴菜单项
        mi_paste.setText("粘贴");    //设置粘贴菜单项的标题
        mn_edit.add(mi_paste);    //把粘贴菜单项添加到编辑菜单
        final JToolBar toolBar=new JToolBar();    //创建工具栏
        getContentPane().add(toolBar,BorderLayout.NORTH);    //把工具栏放到窗体上方
        final JButton btn_open=new JButton();    //创建工具按钮
        btn_open.addActionListener(new ActionListener()
        {
            //添加动作监听器
            public void actionPerformed(final ActionEvent arg0)
            {
                openTextFile();    //调用方法，操作文件
            }
        });
        btn_open.setText("  打  开  ");    //设置工具按钮的标题
        toolBar.add(btn_open);    //把工具按钮添加到工具栏上
        final JButton btn_exit=new JButton();    //创建工具按钮
        btn_exit.addActionListener(new ActionListener()
        {
            //添加动作监听器
            public void actionPerformed(final ActionEvent arg0)
            {
                System.exit(0);    //退出系统
            }
        });
        btn_exit.setText("  退  出  ");    //设置工具按钮的标题
        toolBar.add(btn_exit);    //把工具按钮添加到工具栏上
        final JTabbedPane tabbedPane=new JTabbedPane();    //创建选项卡面板
        getContentPane().add(tabbedPane,BorderLayout.CENTER);    //把选项卡面板放到窗体中央
        final JScrollPane scrollPane1=new JScrollPane();    //创建滚动面板
        //把滚动面板放到选项卡的第一个选项页
        tabbedPane.addTab("文件的属性",null,scrollPane1,null);
        ta_showProperty=new JTextArea();    //创建文本域
        //把文本域添加到滚动面板的视图中
        scrollPane1.setViewportView(ta_showProperty);
        final JScrollPane scrollPane2=new JScrollPane();    //创建滚动面板
        //把滚动面板放到选项卡的第二个选项页
        tabbedPane.addTab("文件的内容",null,scrollPane2,null);
        ta_showText=new JTextArea();    //创建文本域
        //把文本域添加到滚动面板的视图中
        scrollPane2.setViewportView(ta_showText);
    }

    //用于打开文件并获得文件信息的方法
    public void openTextFile()
    {
        JFileChooser fileChooser=new JFileChooser();    //创建文件选择对话框
        fileChooser.setFileFilter(new FileNameExtensionFilter("文本文件","txt"));
        int returnValue=fileChooser.showOpenDialog(getContentPane());    //打开文件选择对话框
        if(returnValue==JFileChooser.APPROVE_OPTION)
        {
            //判断用户是否选择了文件
            File file=fileChooser.getSelectedFile();    //获得文件对象
            //获得文件的绝对路径
            ta_showProperty.append("文件的绝对路径是："+file.getAbsolutePath()+"\n");
            //是否为隐藏文件
            ta_showProperty.append("该文件是隐藏文件吗？"+file.isHidden()+"\n");
            FileReader reader;    //声明字符流
            BufferedReader in;    //声明字符缓冲流
            try
            {
                reader=new FileReader(file);    //创建字符流
                in=new BufferedReader(reader);    //创建字符缓冲流
                String info=in.readLine();    //从文件中读取一行信息
                while(info!=null)
                {
                    //判断是否读到内容
                    ta_showText.append(info+"\n");    //把读到的信息追加到文本域中
                    info=in.readLine();    //继续读下一行信息
                }
                in.close();    //关闭字符缓冲流
                reader.close();    //关闭字符流
            }
            catch(Exception ex)
            {
                ex.printStackTrace();    //输出栈踪迹
            }
        }
    }
}
```

运行效果

![](https://i.loli.net/2019/04/11/5caf5bb90157b.gif)
