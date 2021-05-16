---
title: Java  File
date: 2019-04-06 11:19:59
tags:
 - Java
categories:
 - Java
 - 基础
---

#### 非流式文件类--File类

从定义看，File类是Object的直接子类，同时它继承了Comparable接口可以进行数组的排序。File类的操作包括文件的创建、删除、重命名、得到路径、创建时间等。

<!--more-->

File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法。

**RandomAccessFile类**

该对象并不是流体系中的一员，其封装了字节流，同时还封装了一个缓冲区（字符数组），通过内部的指针来操作字符数组中的数据。

1.    该对象只能操作文件，所以构造函数接收两种类型的参数：a.字符串文件路径；b.File对象。

2.    该对象既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模式(r,rw)

注意：该对象在实例化时，如果要操作的文件不存在，会自动创建；如果文件存在，写数据未指定位置，会从头开始写，即覆盖原有的内容。 可以用于多线程下载或多个线程同时写数据到文件。

常用文件属性方法：

![](https://i.loli.net/2019/04/12/5cb021a60ca2a.png)

用法：

```java
//1.创建文件夹   
//import java.io.*;   
File myFolderPath = new File(str1);   
try {   
  if (!myFolderPath.exists()) {
             myFolderPath.mkdir();
             System.out.println("创建目录" + str1);
         }else {
             System.out.println("该目录已存在");
         }
       }   
catch (Exception e) {   
    System.out.println("新建目录操作出错");   
    e.printStackTrace();   
}    

//2.创建文件   
//import java.io.*;   
File myFilePath = new File(str1);   
try {   
    if (!myFilePath.exists()) {   
        myFilePath.createNewFile();   
    }   
    FileWriter resultFile = new FileWriter(myFilePath);   
    PrintWriter myFile = new PrintWriter(resultFile);   
    myFile.println(str2);   
    resultFile.close();   
}   
catch (Exception e) {   
    System.out.println("新建文件操作出错");   
    e.printStackTrace();   
}    

//3.删除文件   
//import java.io.*;   
File myDelFile = new File(str1);   
try {   
    myDelFile.delete();   
}   
catch (Exception e) {   
    System.out.println("删除文件操作出错");   
    e.printStackTrace();   
}    

//4.删除文件夹   
//import java.io.*;   
File delFolderPath = new File(str1);   
try {   
    delFolderPath.delete(); //删除空文件夹   
}   
catch (Exception e) {   
    System.out.println("删除文件夹操作出错");   
    e.printStackTrace();   
}    

//5.删除一个文件下夹所有的空文件夹   
//import java.io.*;   
File delfile=new File(str1);   
File[] files=delfile.listFiles();   
for(int i=0;i<files.length;i++){   
    if(files[i].isDirectory()){   
        files[i].delete();   
    }   
}     

//6.清空文件夹   
//import java.io.*;   
File delfilefolder=new File(str1);   
try {   
    if (!delfilefolder.exists()) {   
        delfilefolder.delete();   
    }   
    delfilefolder.mkdir();   
}   
catch (Exception e) {   
    System.out.println("清空目录操作出错");   
    e.printStackTrace();   
}    

public static void deleteFiles(File file)
   {
       File[] files = file.listFiles();
       if (null==files)
       {
           return;
       }
       for (File f : files) {
           if (f.isDirectory())
           {
               deleteFiles(f);
           }
           else {
               f.delete();
           }
       }
       file.delete();
   }


//7.读取文件   
//import java.io.*;   
// 逐行读取数据   
FileReader fr = new FileReader(str1);   
BufferedReader br = new BufferedReader(fr);   
String str2 = br.readLine();   
while (str2 != null) {   
    str3   
    str2 = br.readLine();   
}   
br.close();   
fr.close();    

//8.写入文件   
//import java.io.*;   
// 将数据写入文件   
try {   
    FileWriter fw = new FileWriter(str1);   
    fw.write(str2);   
    fw.flush();   
    fw.close();    
} catch (IOException e) {   
    e.printStackTrace();   
}   

//9.写入随机文件   
//import java.io.*;   
try {   
    RandomAccessFile logFile=new RandomAccessFile(str1,"rw");   
    long lg=logFile.length();   
    logFile.seek(str2);   
    logFile.writeByte(str3);   
}catch(IOException ioe){   
    System.out.println("无法写入文件："+ioe.getMessage());   
}    

//10.读取文件属性   
//import java.io.*;   
// 文件属性的取得   
File f = new File(str1);   
if (af.exists()) {   
    System.out.println(f.getName() + "的属性如下： 文件长度为：" + f.length());   
    System.out.println(f.isFile() ? "是文件" : "不是文件");   
    System.out.println(f.isDirectory() ? "是目录" : "不是目录");   
    System.out.println(f.canRead() ? "可读取" : "不");   
    System.out.println(f.canWrite() ? "是隐藏文件" : "");   
    System.out.println("文件夹的最后修改日期为：" + new Date(f.lastModified()));   
    } else {   
    System.out.println(f.getName() + "的属性如下：");   
    System.out.println(f.isFile() ? "是文件" : "不是文件");   
    System.out.println(f.isDirectory() ? "是目录" : "不是目录");   
    System.out.println(f.canRead() ? "可读取" : "不");   
    System.out.println(f.canWrite() ? "是隐藏文件" : "");   
    System.out.println("文件的最后修改日期为：" + new Date(f.lastModified()));   
}   
if(f.canRead()){
    FileReader fr=new FileReader(f);
    BufferedReader br=new BufferedReader(fr);
    String s="";
    do {
        s=br.readLine();
        if (null==s)
        {
            break;
        }
        System.out.println(s);
    }while (true);
    br.close();
    fr.close();
}
if(f.canWrite()){
//                FileWriter fw=new FileWriter(f);
//                BufferedWriter bw=new BufferedWriter(fw);
//                bw.write(str2);
//                bw.close();
//                fw.close();
    RandomAccessFile rf=new RandomAccessFile(f,"rw");
    rf.seek(rf.length());
    rf.writeUTF(str2);
    rf.close();

    System.out.println("插入" + str2);
}

//11.写入属性   
//import java.io.*;   
File filereadonly=new File(str1);   
try {   
    boolean b=filereadonly.setReadOnly();   
}   
catch (Exception e) {   
    System.out.println("拒绝写访问："+e.printStackTrace());   
}    

//12.枚举一个文件夹中的所有文件   
//import java.io.*;   
//import java.util.*;   
LinkedList<String> folderList = new LinkedList<String>();   
folderList.add(str1);   
while (folderList.size() > 0) {   
    File file = new File(folderList.peek());   
    folderList.remove();   
    File[] files = file.listFiles();   
    ArrayList<File> fileList = new ArrayList<File>();   
    for (int i = 0; i < files.length; i++) {   
        if (files[i].isDirectory()) {   
            folderList.add(files[i].getPath());   
        } else {   
            fileList.add(files[i]);   
        }   
    }   
    for (File f : fileList) {   
      String s = f.getAbsolutePath();
      System.out.println(s);
    }   
}   

//13.复制文件夹    

    public static void main(String [] args)
    {
        String str1="/tmp/cmospwd-5.0/";
        String str2="/tmp/cmospwd";
        try {
            LinkedList<String> folderList1 = new LinkedList<String>();
            folderList1.add(str1);
            LinkedList<String> folderList2 = new LinkedList<String>();
            folderList2.add(str2+ str1.substring(str1.lastIndexOf("/")));

            LinkedList<String> fl1=new LinkedList<>(),fl2=new LinkedList<>();

            while (folderList1.size()>0)
            {
                String s2=folderList2.peek();
                new File(s2).mkdir();
                folderList2.remove();

               File f1 =new File(folderList1.peek());
               folderList1.remove();

                File[] files = f1.listFiles();
                if(null!=files){
                for (File file : files) {
                    if (file.isDirectory())
                    {
                        folderList1.add(file.getAbsolutePath());
                        folderList2.add(s2+File.separator+file.getName());
                    }else {
                        fl1.add(file.getAbsolutePath());
                        fl2.add(s2+File.separator+file.getName());
                    }
                }
              }
                for (;;) {
                    if (null==fl1.peek())
                    {
                        break;
                    }
                    File file1=new File(fl1.peek());
                    File file2=new File(fl2.peek());
                    fl1.remove();
                    fl2.remove();
                    copyToFile(file1,file2);
                }
            }
        }
        catch (Exception e) {
            System.out.println("操作出错");
            e.printStackTrace();
        }
    }

    private static int copyToFile(final File src,final File dest) throws IOException {
        if (src.exists() ) {
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream(src));
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(dest));
            int bytesum = 0;
            int byteread = 0;
            byte[] bytes = new byte[1024];
            while ( (byteread = bis.read(bytes)) != -1) {
                bytesum += byteread; //字节数 文件大小
                bos.write(bytes, 0, byteread);
            }

            bis.close();
            bos.close();
            final long srcLen = src.length(); // TODO See IO-386
            final long dstLen = dest.length(); // TODO See IO-386
            if (srcLen != dstLen) {
                throw new IOException("Failed to copy full contents from '" +
                        src + "' to '" + dest + "' Expected length: " + srcLen + " Actual: " + dstLen);
            }
            return 1;
        }
        return 0;
    }


//14.复制一个文件夹下所有的文件夹到另一个文件夹下   
//import java.io.*;   
//import java.util.*;   

public static void main(String [] args)
{

    String str1="/tmp/cmospwd-5.0/";
    String str2="/tmp/cmospwd";

    try {
        File f1=new File(str1);
        File f2=new File(str2);
        if (!f2.exists())
        {
            f2.mkdir();
        }

        if (f1.exists())
        {
            File[] files = f1.listFiles();
            if (null!=files)
            {
                for (File file : files) {
                    if (file.isFile())
                    {
                        File f3=new File(str2+File.separator+file.getName());
                        f3.createNewFile();
                        copyToFile(file,f3);
                    }
                }
            }
        }
    }catch (Exception e)
    {
        e.printStackTrace();
    }
}



//15.移动文件夹   (复制文件夹后删除)
//import java.io.*;   
//import java.util.*;   
LinkedList<String> folderList = new LinkedList<String>();   
folderList.add(str1);   
LinkedList<String> folderList2 = new LinkedList<String>();   
folderList2.add(str2 + str1.substring(str1.lastIndexOf("\\")));   
while (folderList.size() > 0) {   
    (new File(folderList2.peek())).mkdirs(); // 如果文件夹不存在 则建立新文件夹   
    File folders = new File(folderList.peek());   
    String[] file = folders.list();   
    File temp = null;   
    try {   
        for (int i = 0; i < file.length; i++) {   
            if (folderList.peek().endsWith(File.separator)) {   
                temp = new File(folderList.peek() + File.separator + file[i]);   
            } else {   
                temp = new File(folderList.peek() + File.separator + file[i]);   
            }   
            if (temp.isFile()) {   
                FileInputStream input = new FileInputStream(temp);   
                FileOutputStream output = new FileOutputStream(   
                folderList2.peek() + File.separator + (temp.getName()).toString());   
                byte[] b = new byte[5120];   
                int len;   
                while ((len = input.read(b)) != -1) {   
                    output.write(b, 0, len);   
                }   
                output.flush();   
                output.close();   
                input.close();   
                if (!temp.delete())   
                System.out.println("删除单个文件操作出错!");   
            }   
            if (temp.isDirectory()) {// 如果是子文件夹   
                for (File f : temp.listFiles()) {   
                    if (f.isDirectory()) {   
                        folderList.add(f.getPath());   
                        folderList2.add(folderList2.peek() + File.separator + f.getName());   
                    }   
                }   
            }   
        }   
    } catch (Exception e) {   
        // System.out.println("复制整个文件夹内容操作出错");   
        e.printStackTrace();   
    }   
    folderList.removeFirst();   
    folderList2.removeFirst();   
}   
File f = new File(str1);   
if (!f.delete()) {   
    for (File file : f.listFiles()) {   
        if (file.list().length == 0) {   
            System.out.println(file.getPath());   
            file.delete();   
        }   
    }   
}   
//16.移动一个文件夹下所有的文件夹到另一个目录下   
//import java.io.*;   
//import java.util.*;   
File movefolders=new File(str1);   
File[] movefoldersList=movefolders.listFiles();   
for(int k=0;k<movefoldersList.length;k++){   
    if(movefoldersList[k].isDirectory()){   
        ArrayList<String>folderList=new ArrayList<String>();   
        folderList.add(movefoldersList[k].getPath());   
        ArrayList<String>folderList2=new ArrayList<String>();   
        folderList2.add(str2+"/"+movefoldersList[k].getName());   
        for(int j=0;j<folderList.length;j++){   
             (new File(folderList2.get(j))).mkdirs(); //如果文件夹不存在 则建立新文件夹   
             File folders=new File(folderList.get(j));   
             String[] file=folders.list();   
             File temp=null;   
             try {   
                 for (int i = 0; i < file.length; i++) {   
                     if(folderList.get(j).endsWith(File.separator)){   
                         temp=new File(folderList.get(j)+"/"+file[i]);   
                     }   
                     else{   
                         temp=new File(folderList.get(j)+"/"+File.separator+file[i]);   
                     }   
                     FileInputStream input = new FileInputStream(temp);   
                     if(temp.isFile()){   
                         FileInputStream input = new FileInputStream(temp);   
                         FileOutputStream output = new FileOutputStream(folderList2.get(j) + "/" + (temp.getName()).toString());   
                         byte[] b = new byte[5120];   
                         int len;   
                         while ( (len = input.read(b)) != -1) {   
                             output.write(b, 0, len);   
                         }   
                         output.flush();   
                         output.close();   
                         input.close();   
                         temp.delete();   
                     }   
                     if(temp.isDirectory()){//如果是子文件夹   
                         folderList.add(folderList.get(j)+"/"+file[i]);   
                         folderList2.add(folderList2.get(j)+"/"+file[i]);   
                     }   
                 }   
             }   
             catch (Exception e) {   
                 System.out.println("复制整个文件夹内容操作出错");   
                 e.printStackTrace();   
             }   
        }   
        movefoldersList[k].delete();   
    }   
}   

//17.以一个文件夹的框架在另一个目录创建文件夹和空文件   
//import java.io.*;   
//import java.util.*;   
boolean b=false;//不创建空文件   
ArrayList<String>folderList=new ArrayList<String>();   
folderList.add(str1);   
ArrayList<String>folderList2=new ArrayList<String>();   
folderList2.add(str2);   
for(int j=0;j<folderList.length;j++){   
    (new File(folderList2.get(j))).mkdirs(); //如果文件夹不存在 则建立新文件夹   
    File folders=new File(folderList.get(j));   
    String[] file=folders.list();   
    File temp=null;   
    try {   
        for (int i = 0; i < file.length; i++) {   
            if(folderList.get(j).endsWith(File.separator)){   
                temp=new File(folderList.get(j)+"/"+file[i]);   
            }   
            else{   
                temp=new File(folderList.get(j)+"/"+File.separator+file[i]);   
            }   
            FileInputStream input = new FileInputStream(temp);   
            if(temp.isFile()){   
                if (b) temp.createNewFile();   
            }   
            if(temp.isDirectory()){//如果是子文件夹   
                folderList.add(folderList.get(j)+"/"+file[i]);   
                folderList2.add(folderList2.get(j)+"/"+file[i]);   
            }   
        }   
    }   
    catch (Exception e) {   
        System.out.println("复制整个文件夹内容操作出错");   
        e.printStackTrace();   
    }   
}   

//18.复制文件   
//import java.io.*;   
 int bytesum = 0;   
 int byteread = 0;   
 File oldfile = new File(str1);   
 try {   
 if (oldfile.exists()) { //文件存在时   
 FileInputStream inStream = new FileInputStream(oldfile); //读入原文件   
 FileOutputStream fs = new FileOutputStream(new File(str2,oldfile.getName()));   
 byte[] buffer = new byte[5120];   
 int length;   
 while ( (byteread = inStream.read(buffer)) != -1) {   
 bytesum += byteread; //字节数 文件大小   
 System.out.println(bytesum);   
 fs.write(buffer, 0, byteread);   
 }   
 inStream.close();   
 }   
 }   
 catch (Exception e) {   
 System.out.println("复制单个文件操作出错");   
 e.printStackTrace();   
 }    

//复制文件   
 private static int copyToFile(final File src,final File dest) throws IOException {
    if (src.exists() ) {
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream(src));
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(dest));

        byte[] bytes = new byte[1024];

        final long size = bis.available(); // TODO See IO-386
        long pos = 0;
        long count = 0;
        while (pos < size) {
            final long remain = size - pos;
            count = remain > 1024 ? 1024 : remain;
            final long bytesCopied=bis.read(bytes,0,(int)(count));
            bos.write(bytes,0,(int)count);

            if (bytesCopied == 0) { // IO-385 - can happen if file is truncated after caching the size
                break; // ensure we don't loop forever
            }
            pos += bytesCopied;
        }
        bis.close();
        bos.close();
            final long srcLen = src.length(); // TODO See IO-386
            final long dstLen = dest.length(); // TODO See IO-386
            if (srcLen != dstLen) {
                throw new IOException("Failed to copy full contents from '" +
                src + "' to '" + dest + "' Expected length: " + srcLen + " Actual: " + dstLen);
                }
        return 1;
    }
    return 0;

}


//19.复制一个文件夹下所有的文件到另一个目录 
//import java.io.*; 
File copyfiles=new File(str1);
File[] files=copyfiles.listFiles();
for(int i=0;i<files.length;i++){
    if(!files[i].isDirectory()){ 
        int bytesum = 0;   
        int byteread = 0;   
        try {   
            InputStream inStream = new FileInputStream(files[i]); //读入原文件   
            FileOutputStream fs = new FileOutputStream(new File(str2,files[i].getName());   
            byte[] buffer = new byte[5120];   
            int length;   
            while ( (byteread = inStream.read(buffer)) != -1) {   
                bytesum += byteread; //字节数 文件大小   
                System.out.println(bytesum);   
                fs.write(buffer, 0, byteread);   
            }   
            inStream.close();   
        } catch (Exception e) {   
            System.out.println("复制单个文件操作出错");   
            e.printStackTrace();   
        }   
    }
} 

//提取扩展名 
String str2=str1.substring(str1.lastIndexOf(".")+1);
```

一般使用Apache的[Commons IO](http://commons.apache.org/proper/commons-io/)来封装函数，节省IO代码的编写。

### 标准IO

System类对IO的支持，针对一些频繁的设备交互，Java语言系统预定了3个可以直接使用的流对象，分别是：

 - System.in（标准输入），通常代表键盘输入。

 - System.out（标准输出）：通常写往显示器。

 - System.err（标准错误输出）：通常写往显示器。

 Java程序可通过命令行参数与外界进行简短的信息交换，同时，也规定了与标准输入、输出设备，如键盘、显示器进行信息交换的方式。而通过文件可以与外界进行任意数据形式的信息交换。

 注意：
（1）System类不能创建对象，只能直接使用它的三个静态成员。
（2）每当main方法被执行时,就自动生成上述三个对象


1) 标准输出流 System.out

System.out向标准输出设备输出数据，其数据类型为PrintStream。方法：
```
Void print(参数)
Void println(参数)
Void printf("格式",参数...)
```
2)标准输入流 System.in

System.in读取标准输入设备数据（从标准输入获取数据，一般是键盘），其数 据类型为InputStream。方法：
```
int read()  //返回ASCII码。若,返回值=-1，说明没有读取到任何字节读取工作结束。
int read(byte[] b)//读入多个字节到缓冲区b中返回值是读入的字节数
```
3)标准错误流

System.err输出标准错误，其数据类型为PrintStream。可查阅API获得详细说明。

标准输出通过System.out调用println方法输出参数并换行，而print方法输出参数但不换行。println或print方法都通 过重载实现了输出基本数据类型的多个方法，包括输出参数类型为boolean、char、int、long、float和double。同时，也重载实现 了输出参数类型为char[]、String和Object的方法。其中，print（Object）和println（Object）方法在运行时将调 用参数Object的toString方法。

**IOException异常类的子类**

1. public class  EOFException ：   非正常到达文件尾或输入流尾时，抛出这种类型的异常。    

2. public class FileNotFoundException：   当文件找不到时，抛出的异常。

3. public class InterruptedIOException： 当I/O操作被中断时，抛出这种类型的异常。

### 对象的序列化与反序列化

什么是序列化和反序列化呢？这是针对对象来说的，因为我们在写入文件的时候，常常因为要保存的是一个对象，也就是一个obj，但是里面的变量又很多，我们不可能挨个申明，一个个写入，这时候，我们就可以使用对象的序列化与反序列化。

**序列化就是对象到保存文件的过程。**

**反序列化就是从保存的文件，转换为对象的过程。**

#### Serializable

Java的对象序列化是指将那些实现了Serializable接口的对象转换成一个字符序列，并能够在以后将这个字节序列完全恢复为原来的对象。这一过程甚至可通过网络进行，这意味着序列化机制能自动弥补不同操作系统之间的差异。

只要对象实现了Serializable接口（记住，这个接口只是一个标记接口，不包含任何的方法）。

如果我们想要序列化一个对象，首先要创建某些OutputStream(如FileOutputStream、ByteArrayOutputStream等)，然后将这些OutputStream封装在一个ObjectOutputStream中。这时候，只需要调用writeObject()方法就可以将对象序列化，并将其发送给OutputStream（记住：对象的序列化是基于字节的，不能使用Reader和Writer等基于字符的层次结构）。而饭序列的过程（即将一个序列还原成为一个对象），需要将一个InputStream(如FileInputstream、ByteArrayInputStream等)封装在ObjectInputStream内，然后调用readObject()即可。

对象序列化过程不仅仅保存单个对象，还能追踪对象内所包含的所有引用，并保存那些对象（这些对象也需实现了Serializable接口）。下面这段代码演示了此过程：

```java
import java.io.Serializable;
import java.util.Random;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

class Data implements Serializable {
    private static final long serialVersionUID = 7247714666080613254L;
    public int n;
    public Data(int n) {
        this.n = n;
    }
    public String toString(){
        return Integer.toString(n);
    }
}
class Worm implements Serializable {
    private static final long serialVersionUID = 5468335797443850679L;
    private Data[] d = {
            new Data(random.nextInt(10)),
            new Data(random.nextInt(10)),
            new Data(random.nextInt(10))
    };
    private static Random random = new Random(47);
    private Worm next;
    private char c;
    
    public Worm(int i , char x) {
        System.out.println("Worm constructor:" +i);
        c = x;
        if(--i > 0) {
            next = new Worm(i , (char)(x+1));
        }
    }
    public Worm() {
        System.out.println("Default constructor!");
    }
    
    public String toString() {
        StringBuilder sb = new StringBuilder(":");
        sb.append(c);
        sb.append("(");
        for(Data data : d) {
            sb.append(data);
        }
        sb.append(")");
        if(next!=null) {
            sb.append(next);
        }
        return sb.toString();
    }
}
public class SerializableTest {
    
    public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
        Worm w = new Worm(6 ,'a');
        System.out.println("序列化操纵之前");
        System.out.println("w="+w);
        
        //序列化操作1--FileOutputStream
        ObjectOutputStream oos1 = new ObjectOutputStream(new FileOutputStream("worm.out"));
        oos1.writeObject("Worm storage By FileOutputStream ");
        oos1.writeObject(w);//必须所有引用的对象都实现序列化（本例终究是Data这个类），否则抛出有java.io.NotSerializableException:这个异常
        oos1.close();
        
        //反序列化操作1---FileInputStream
        ObjectInputStream ois1 = new ObjectInputStream(new FileInputStream("worm.out"));
        String s1 = (String)ois1.readObject();
        Worm w1 = (Worm)ois1.readObject();
        ois1.close();
        System.out.println("反序列化操作1之后");
        System.out.println(s1);
        System.out.println("w1:"+w1);
        
        //序列化操作2--ByteArrayOutputStream
        ByteArrayOutputStream byteOutStream = new ByteArrayOutputStream();
        ObjectOutputStream oos2 = new ObjectOutputStream(byteOutStream);
        oos2.writeObject("Worm storage By ByteOutputStream ");
        oos2.writeObject(w);
        oos2.flush();
        
        //反序列操作2--ByteArrayInputStream
        ByteArrayInputStream byteInStream = new ByteArrayInputStream(byteOutStream.toByteArray());
        ObjectInputStream ois2 = new ObjectInputStream(byteInStream);
        String s2 = (String)ois2.readObject();
        Worm w2 = (Worm)ois2.readObject();
        ois2.close();
        System.out.println("反序列化操作2之后");
        System.out.println(s2);
        System.out.println("w2:"+w2);
    }
    
    
}
```

运行结果：

```
Worm constructor:6
Worm constructor:5
Worm constructor:4
Worm constructor:3
Worm constructor:2
Worm constructor:1
序列化操纵之前
w=:a(853):b(119):c(802):d(788):e(199):f(881)
反序列化操作1之后
Worm storage By FileOutputStream 
w1::a(853):b(119):c(802):d(788):e(199):f(881)
反序列化操作2之后
Worm storage By ByteOutputStream 
w2::a(853):b(119):c(802):d(788):e(199):f(881)
```

 思考：

1. 反序列化后的对象，需要调用构造函数重新构造吗？

   答案：不需要。对于Serializable对象，对象完全以它存储的二进制位作为基础来构造，而不调用构造器。

   ```java
   
   class House implements Serializable {
       private static final long serialVersionUID = -6091530420906090649L;
       
       private Date date = new Date(); //记录当前的时间
       
       public String toString() {
           return "House:" + super.toString() + ".Create Time is:" + date;
       }
   
   }
   class Animal implements Serializable {
       private static final long serialVersionUID = -213221189192962074L;
       
       private String name;
       
       private House house;
       
       public Animal(String name , House house) {
           this.name = name;
           this.house = house;
           System.out.println("调用了构造器");
       }
       
       public String toString() {
           return  name + "[" +super.toString() + "']" + house;
       }
   
   }
   
   public class Myworld {
   
       public static void main(String[] args) throws IOException, ClassNotFoundException {
           House house = new House();
           System.out.println("序列化前");
           Animal animal = new Animal("test",house);
           ByteArrayOutputStream out = new ByteArrayOutputStream();
           ObjectOutputStream oos = new ObjectOutputStream(out);
           oos.writeObject(animal);
           oos.flush();
           oos.close();
   
           System.out.println("反序列化后");
           ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());
           ObjectInputStream ois = new ObjectInputStream(in);
           Animal animal1 = (Animal)ois.readObject();
           ois.close();    
       }
   
   }
   ```

   运行结果

   ```
   序列化前
   调用了构造器
   反序列化后
   ```

   从上面的结果中可以看到，在序列化前，当我们使用

   ```java
   Animal animal = new Animal("test",house);
   ```
    时，调用了Animal的构造器（打印了输出语句），但是反序列后并没有再打印任何语句，说明并没有调用构造器。

2. 序列前的对象与序列化后的对象是什么关系？是("=="还是equal？是浅复制还是深复制？)

    答案：深复制，反序列化还原后的对象地址与原来的的地址不同。 我们还是看上面思考1）中给出的代码，前两个类不变化，修改第三个类（MyWorld.java）的部分代码，修改后的代码如下：

   ```java
   public class Myworld {
   
       public static void main(String[] args) throws IOException, ClassNotFoundException {
           House house = new House();
           System.out.println("序列化前");
           Animal animal = new Animal("test",house);
           System.out.println(animal);
           ByteArrayOutputStream out = new ByteArrayOutputStream();
           ObjectOutputStream oos = new ObjectOutputStream(out);
           oos.writeObject(animal);
           oos.writeObject(animal);//在写一次，看对象是否是一样，
           oos.flush();
           oos.close();
           
           ByteArrayOutputStream out2 = new ByteArrayOutputStream();//换一个输出流
           ObjectOutputStream oos2 = new ObjectOutputStream(out2);
           oos2.writeObject(animal);
           oos2.flush();
           oos2.close();
   
           System.out.println("反序列化后");
           ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());
           ObjectInputStream ois = new ObjectInputStream(in);
           Animal animal1 = (Animal)ois.readObject();
           Animal animal2 = (Animal)ois.readObject();
           ois.close();
           
           ByteArrayInputStream in2 = new ByteArrayInputStream(out2.toByteArray());
           ObjectInputStream ois2 = new ObjectInputStream(in2);
           Animal animal3 = (Animal)ois2.readObject();
           ois2.close();
           
           System.out.println("out流：" +animal1);
           System.out.println("out流：" +animal2);
           System.out.println("out2流：" +animal3);
           
           
           System.out.println("测试序列化前后的对象 == ："+ (animal==animal1));
           System.out.println("测试序列化后同一流的对象："+ (animal1 == animal2));
           System.out.println("测试序列化后不同流的对象==:" + (animal1==animal3));
           
       }
   
   }
   ```

   运行结果：

   ```
   序列化前
   调用了构造器
   test[com.hyp.learn.base.Demo05.Animal@3b6eb2ec']House:com.hyp.learn.base.Demo05.House@1e643faf.Create Time is:Thu Jun 27 11:47:50 CST 2019
   反序列化后
   out流：test[com.hyp.learn.base.Demo05.Animal@d2cc05a']House:com.hyp.learn.base.Demo05.House@4f933fd1.Create Time is:Thu Jun 27 11:47:50 CST 2019
   out流：test[com.hyp.learn.base.Demo05.Animal@d2cc05a']House:com.hyp.learn.base.Demo05.House@4f933fd1.Create Time is:Thu Jun 27 11:47:50 CST 2019
   out2流：test[com.hyp.learn.base.Demo05.Animal@548a9f61']House:com.hyp.learn.base.Demo05.House@1753acfe.Create Time is:Thu Jun 27 11:47:50 CST 2019
   测试序列化前后的对象 == ：false
   测试序列化后同一流的对象：true
   测试序列化后不同流的对象==:false
   ```

   从结果可以看到

   序列化前后对象的地址不同了，但是内容是一样的，而且对象中包含的引用也相同。换句话说，通过序列化操作，我们可以实现对任何可Serializable对象的”深度复制（deep copy）"——这意味着我们复制的是整个对象网，而不仅仅是基本对象及其引用。对于同一流的对象，他们的地址是相同，说明他们是同一个对象，但是与其他流的对象地址却不相同。也就说，只要将对象序列化到单一流中，就可以恢复出与我们写出时一样的对象网，而且只要在同一流中，对象都是同一个。

3.  serialVersionUID 的作用

   在Java中，软件的兼容性是一个大问题，尤其在使用到对象串行性的时候，那么在某一个对象已经被串行化了，可是这个对象又被修改后重新部署了，那么在这种情况下， 用老软件来读取新文件格式虽然不是什么难事，但是有可能丢失一些信息。 serialVersionUID来解决这些问题，新增的serialVersionUID必须定义成下面这种形式：`static final long serialVersionUID=-2805284943658356093L;`。其中数字后面加上的L表示这是一个long值。 通过这种方式来解决不同的版本之间的串行话问题。

   Java串行化机制定义的文件格式似乎很脆弱，只要稍微改动一下类的定义，原来保存的对象就可能无法读取。

   - 类的名字。 

   - 域的名字。 

   - 方法的名字。 

   - 已实现的接口。 

   改动上述任意一项内容（无论是增加或删除），都会引起编码值变化，从而引起类似的异常警报。这个数字序列称为“串行化版本统一标识符”（serial version universal identifier），简称UID。解决这个问题的办法是在类里面新增一个域serialVersionUID，强制类仍旧使用原来的UID。新增的域必须是： 

   - static：该域定义的属性作用于整个类，而非特定的对象。 

   - final：保证代码运行期间该域不会被修改。 

   - long：它是一个64位的数值

   也就是说，新增的serialVersionUID必须定义成下面这种形式：`static final long serialVersionUID=-2805284943658356093L;`。其中数字后面加上的L表示这是一个long值。 

   当然，改动之后的类不一定能够和原来的对象兼容。例如，如果把一个域的定义从String改成了int，执行逆-串行化操作时系统就不知道如何处理该值，显示出错误信息：java.io.InvalidClassException:
   Save; incompatible types for field name。

#### Externalizable

Java默认的序列化机制非常简单，而且序列化后的对象不需要再次调用构造器重新生成，但是在实际中，我们可以会希望对象的某一部分不需要被序列化，或者说一个对象被还原之后，其内部的某些子对象需要重新创建，从而不必将该子对象序列化。 在这些情况下，我们可以考虑实现Externalizable接口从而代替Serializable接口来对序列化过程进行控制（后面我们会讲到一个更简单的方式，通过transient的方式）。

Externalizable接口extends Serializable接口，而且在其基础上增加了两个方法：writeExternal()和readExternal()。这两个方法会在序列化和反序列化还原的过程中被自动调用，以便执行一些特殊的操作。

```java
package java.io;

import java.io.ObjectOutput;
import java.io.ObjectInput;


public interface Externalizable extends java.io.Serializable {
   
    void writeExternal(ObjectOutput out) throws IOException;

    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

下面这段代码示范了如何完整的保存和恢复一个Externalizable对象

```java
class Blip implements Externalizable {
    
    private int i ;
    
    private String s;//没有初始化
    
    public Blip() {
         //默认构造函数必须有，而且必须是public
        System.out.println("Blip默认构造函数");
    }
    
    public Blip(String s ,int i) {
        //s,i只是在带参数的构造函数中进行初始化。
        System.out.println("Blip带参数构造函数");
        this.s = s;
        this.i = i;
    }
    
    public String toString() {
        return s  + i ;
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException,
            ClassNotFoundException {
        System.out.println("调用readExternal（）方法");
        s = (String)in.readObject();//在反序列化时，需要初始化s和i，否则只是调用默认构造函数，得不到s和i的值
        i = in.readInt();
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        System.out.println("调用writeExternal（）方法");
        out.writeObject(s); //如果我们不将s和i的值写入的话，那么在反序列化的时候，就不会得到这些值。
        out.writeInt(i);
    }

}
public class ExternalizableTest {

public static void main(String[] args) throws IOException, ClassNotFoundException {
        System.out.println("序列化之前");
        Blip b = new Blip("This String is " , 47);
        System.out.println(b);
        
        System.out.println("序列化操作，writeObject");
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(out);
        oos.writeObject(b);
        System.out.println("反序列化之后,readObject");
        ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(in);
        Blip bb = (Blip)ois.readObject();
        System.out.println(bb);     
    }
}
```

运行结果：

```
序列化之前
Blip带参数构造函数
This String is 47
序列化操作，writeObject
调用writeExternal（）方法
反序列化之后,readObject
Blip默认构造函数
调用readExternal（）方法
This String is 47
```

分析结果：

在Blip类中，字段s和i只在第二个构造器中初始化，而不是在默认的构造器其中初始化的，每次writeObject时，都会调用WriteExtenal()方法，而在WriteExtenal（）方法中我们需要将当前对象的值写入到流中；而每次readObject()时，调用的是默认的构造函数，如果我们不在 readExternal（）方法中初始化s和i,那么s就会为null,而i就会为0。

与Serizable对象不同，使用Externalizabled，就意味着没有任何东西可以自动序列化， 为了正常的运行，我们需要在writeExtenal()方法中将自对象的重要信息写入，从而手动的完成序列化。对于一个Externalizabled对象，对象的默认构造函数都会被调用（包括哪些在定义时已经初始化的字段），然后调用readExternal()，在此方法中必须手动的恢复数据。

#### transient(瞬时)关键字

当我们队序列化进行控制时，可能某个特定子对象不想让Java的序列化机制自动保存与恢复。如果子对象表示的是我们不希望将其序列化的敏感信息（如密码），通常就会面临这种情况。即使对象中的这些信息是private属性，一经序列化处理，人们就可以通过读取文件或者拦截网络传输的方式来访问它。

由于Externalizable对象在默认情况下不保存它的任何字段（即任何字段都不进行序列化处理），所以transient关键字只能和Serializable对象一起使用。

**Externalizable的替代方法**

如果不是特别坚持实现Externalizable接口，那么我们可以通过实现Serializable接口，并**添加**名为writeObject()和readObject()方法（注意：这里用的是添加而不是“覆盖”或者“实现”，因为这两个方法不是基类Object也不是接口Serializable中的方法）。这样一旦对象被序列化或者反序列还原，就会自动地分别调用者两个方法。也就是说，只要我们提供了这两个方法，就会使用它们而不是默认的序列化机制。

但是一定要注意，这两个方法的必须具有准确的方法特征签名，必须严格按照如下形式

```java
private void writeObject(ObjectOutputStream stream) throws IOException{
         //TODO
    }

private void readObject(ObjectInputStream stream) throws IOException , ClassNotFoundException{
        //TODO
    }
```

这两个方法看似简单，但是其实包含了一些比较有意思的处理。首先，这个方式不是Serializable接口或者基类中的一部分，所以必须在类内部自己实现。其次，注意到这个方式其实是private类型。也就是说这两个方法仅能被这个类的其他成员调用，但是我们看到，其实我们没有在这个类的其他的方法中调用这两个方法。那么到底是谁调用这两个方法呢？其实，实际上我们并没有从这个类的其他方法中调用它们，而是ObjectOutputStream和ObjectInputStream对象的writeObject和readObject()方法分别调用者两个方法（通过过反射机制来访问类的私有方法）。

在调用ObjectOutputStream.writeObject()时，会检查所传递的Serializable对象，利用反射来搜索是否有writeObject()方法。如果有，就会跳过正常的序列化过程，转而调用这个它的writeObject()方法，readObject方法处理方式也一样。

这里面有一个技巧，在类的writeObject()内部，可以通过ObjectOutputStream.defaultWriteObject()来执行默认的writeObject()（非transient字段由这个方法保存），同样的，在类readObject内部，可以通过ObjectInputStream.defalutReadObject()来执行默认的readObject()方法。

下面这段代码就显示上述的技巧

```java
public class SeriCtrol implements Serializable {
    private static final long serialVersionUID = -4994939941552821559L;
    
    private String a;
    
    private String b;
    
    private transient String c;
    
    public SeriCtrol(String a,String b,String c) {
        this.a = "非瞬时默认实现：" + a;
        this.b = "非瞬时非默认实现："+ b;
        this.c = "瞬时实现：" + c;
    }
    
    private void writeObject(ObjectOutputStream stream) throws IOException{
        stream.defaultWriteObject();
        stream.writeObject(b);
        stream.writeObject(c);
    }
    
    private void readObject(ObjectInputStream stream) throws IOException , ClassNotFoundException{
        stream.defaultReadObject();
        stream.readObject();
        b= "null";
        c = (String)stream.readObject();
    }
    
    public String toString() {
        return a + b + c;
    }
    
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        SeriCtrol sCtrol = new SeriCtrol("test1","test2","test3");
        System.out.println("序列化之前");
        System.out.println(sCtrol);
        
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ObjectOutputStream oos  = new ObjectOutputStream(out);
        oos.writeObject(sCtrol);
        
        System.out.println("反序列化操作之后");
        ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(in);
        SeriCtrol src = (SeriCtrol) ois.readObject();
        System.out.println(src);
    }

}
```

运行结果

```
序列化之前
非瞬时默认实现：test1非瞬时非默认实现：test2瞬时实现：test3
反序列化操作之后
非瞬时默认实现：test1null瞬时实现：test3
```

从结果中可以看到，字段a没做什么处理，正常序列化，b在readObject()时，将其设置为null,相当于没有序列化了，而瞬时的c，我们可以手动的将其序列化。

注意，个人在操作的过程发现，writeObject()和readObject()方法不需要成对出现，但是这时候readObject()要小心，如果没有writeObject()方法，在readObject（）方法就不能再调用ObjectInputStream的readObject()方法。

在readObject（）方法中，将废弃的属性置为null。

在“Thinking in Java”中提到如果想序列化static静态字段，必须自己手动去实现。

### 面试

1. 字节流和字符流，你更喜欢使用拿一个？
    个人来说，更喜欢使用字符流，因为他们更新一些。许多在字符流中存在的特性，字节流中不存在。比如使用BufferedReader而不是BufferedInputStreams或DataInputStream，使用newLine()方法来读取下一行，但是在字节流中我们需要做额外的操作。

2. `System.out.println()`是什么？
    `println`是PrintStream的一个方法。`out`是一个静态PrintStream类型的成员变量，`System`是一个java.lang包中的类，用于和底层的操作系统进行交互。

3. 什么是Filter流？
    Filter Stream是一种IO流主要作用是用来对存在的流增加一些额外的功能，像给目标文件增加源文件中不存在的行数，或者增加拷贝的性能。

4. 有哪些可用的Filter流？
   在java.io包中主要由4个可用的filter Stream。两个字节filter stream，两个字符filter stream.  分别是FilterInputStream, FilterOutputStream, FilterReader and  FilterWriter.这些类是抽象类，不能被实例化的。

   有些Filter流的子类。

   - LineNumberInputStream 给目标文件增加行号
      \- DataInputStream 有些特殊的方法如`readInt()`, `readDouble()`和`readLine()` 等可以读取一个  int,  double和一个string一次性的,
   - BufferedInputStream 增加性能
   - PushbackInputStream 推送要求的字节到系统中 

5. SequenceInputStream的作用？
   在拷贝多个文件到一个目标文件的时候是非常有用的。可用使用很少的代码实现

   ```java
   public class TwoFiles {
       public static void main(String args[]) throws IOException
       {
           FileInputStream fistream1 = new FileInputStream("/Users/aihe/Desktop/Songshu/code/java8source/src/main/resources/A.txt");  // first source file
           FileInputStream fistream2 = new FileInputStream("/Users/aihe/Desktop/Songshu/code/java8source/src/main/resources/B.txt");  //second source file
   
           SequenceInputStream sistream = new SequenceInputStream(fistream1, fistream2);
           FileOutputStream fostream = new FileOutputStream("C.txt");        // destination file
   
           int temp;
           while( ( temp = sistream.read() ) != -1)
           {
               System.out.print( (char) temp ); // to print at DOS prompt
               fostream.write(temp);   // to write to file
           }
           fostream.close();
           sistream.close();
           fistream1.close();
           fistream2.close();
       }
   }
   
   ```

6. 说说PrintStream和PrintWriter
    他们两个的功能相同，但是属于不同的分类。字节流和字符流。他们都有println()方法。

7. 在文件拷贝的时候，那一种流可用提升更多的性能？
    在字节流的时候，使用BufferedInputStream和BufferedOutputStream。
    在字符流的时候，使用BufferedReader 和 BufferedWriter

8. 说说管道流(Piped Stream)
    有四种管道流， PipedInputStream, PipedOutputStream, PipedReader 和 PipedWriter.在多个线程或进程中传递数据的时候管道流非常有用。

9. 说说File类
    它不属于 IO流，也不是用于文件操作的，它主要用于知道一个文件的属性，读写权限，大小等信息。

10. 说说RandomAccessFile?
     它在java.io包中是一个特殊的类，既不是输入流也不是输出流，它两者都可以做到。他是Object的直接子类。通常来说，一个流只有一个功能，要么读，要么写。但是RandomAccessFile既可以读文件，也可以写文件。  DataInputStream 和 DataOutStream有的方法，在RandomAccessFile中都存在。

### 参考

1. [java IO流面试总结](https://blog.csdn.net/baidu_37107022/article/details/76890019)
2. [Java IO面试题](https://www.imooc.com/article/24305)
3. [commons-io 实现监控本地文件夹](https://blog.csdn.net/xjiuge/article/details/81128109)