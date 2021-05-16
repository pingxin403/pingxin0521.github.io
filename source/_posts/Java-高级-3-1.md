---
title: Java  NIO 基础 (一)
date: 2019-04-21 14:18:59
tags:
 - Java
 - IO
categories:
 - Java
 - 高级
---
### 前言

Java NIO(New IO)是一个可以替代标准 Java IO API 的 IO API(从 Java 1.4 开始),Java NIO 提供了与标准 IO 不同的 IO 工作方式。
<!--more-->
#### 比较

**面向流与面向缓冲**

Java NIO和IO之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。

 Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

**阻塞与非阻塞IO**

Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。

非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

**选择器（Selectors）**
Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。

#### NIO和IO如何影响应用程序的设计

无论您选择IO或NIO工具箱，可能会影响您应用程序设计的以下几个方面：
```
对NIO或IO类的API调用。
数据处理。
用来处理数据的线程数。
```

**API调用**

当然，使用NIO的API调用时看起来与使用IO时有所不同，但这并不意外，因为并不是仅从一个InputStream逐字节读取，而是数据必须先读入缓冲区再处理。

**数据处理**

使用纯粹的NIO设计相较IO设计，数据处理也受到影响。

在IO设计中，我们从InputStream或 Reader逐字节读取数据。假设你正在处理一基于行的文本数据流，
```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```
该文本行的流可以这样处理：
```
InputStream input = … ; // get the InputStream from the client socket
BufferedReader reader = new BufferedReader(new InputStreamReader(input));
String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```


请注意处理状态由程序执行多久决定。换句话说，一旦reader.readLine()方法返回，你就知道肯定文本行就已读完， readline()阻塞直到整行读完，这就是原因。你也知道此行包含名称；同样，第二个readline()调用返回的时候，你知道这行包含年龄等。

 正如你可以看到，该处理程序仅在有新数据读入时运行，并知道每步的数据是什么。一旦正在运行的线程已处理过读入的某些数据，该线程不会再回退数据（大多如此）。下图也说明了这条原则：

 ![](https://i.loli.net/2019/04/24/5cc06065cfbc1.png)

而一个NIO的实现会有所不同，下面是一个简单的例子：
```
ByteBuffer buffer = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buffer);
```
注意第二行，从通道读取字节到ByteBuffer。当这个方法调用返回时，你不知道你所需的所有数据是否在缓冲区内。你所知道的是，该缓冲区包含一些字节，这使得处理有点困难。

假设第一次 read(buffer)调用后，读入缓冲区的数据只有半行，例如，“Name:An”，你能处理数据吗？显然不能，需要等待，直到整行数据读入缓存，在此之前，对数据的任何处理毫无意义。

所以，你怎么知道是否该缓冲区包含足够的数据可以处理呢？好了，你不知道。发现的方法只能查看缓冲区中的数据。其结果是，在你知道所有数据都在缓冲区里之前，你必须检查几次缓冲区的数据。这不仅效率低下，而且可以使程序设计方案杂乱不堪。例如：

```
ByteBuffer buffer = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buffer);
while(! bufferFull(bytesRead) ) {
bytesRead = inChannel.read(buffer);
}
```

bufferFull()方法必须跟踪有多少数据读入缓冲区，并返回真或假，这取决于缓冲区是否已满。换句话说，如果缓冲区准备好被处理，那么表示缓冲区满了。

bufferFull()方法扫描缓冲区，但必须保持在bufferFull（）方法被调用之前状态相同。如果没有，下一个读入缓冲区的数据可能无法读到正确的位置。这是不可能的，但却是需要注意的又一问题。

如果缓冲区已满，它可以被处理。如果它不满，并且在你的实际案例中有意义，你或许能处理其中的部分数据。但是许多情况下并非如此。

![](https://i.loli.net/2019/04/24/5cc06065b6757.png)

**用来处理数据的线程数**

NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。

如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。同样，如果你需要维持许多打开的连接到其他计算机上，如P2P网络中，使用一个单独的线程来管理你所有出站连接，可能是一个优势。

![](https://i.loli.net/2019/04/21/5cbc32edeaaec.jpg)

如果你有少量的连接使用非常高的带宽，一次发送大量的数据，也许典型的IO服务器实现可能非常契合。


### NIO 基础类
虽然 Java NIO 中除此之外还有很多类和组件,但是 Channel, Buffer 和Selector 构成了核心的 API。其它组件,如 Pipe 和 FileLock,只不过是与三个核心组件共同使用的工具类。我们在后面会具体讲到。

#### Path

Java Path 接口是 Java NIO 2 更新的一部分,同 Java NIO 一起已经包括在 Java6和 Java7 中。Java Path 接口是在 Java7 中添加到 Java NIO 的。Path 接口位于java.nio.file 包中,所以 Path 接口的完全限定名称为 java.nio.file.Path。

Java Path 实例表示文件系统中的路径。一个路径可以指向一个文件或一个目录。路径可以是绝对路径,也可以是相对路径。绝对路径包含从文件系统的根目录到它指向的文件或目录的完整路径。相对路径包含相对于其他路径的文件或目录的路径。

不要将文件系统路径与某些操作系统中的 path 环境变量混淆。java.nio.file.Path 接口与 path 环境变量没有任何关系。

在许多方面,java.nio.file.Path 接口类似于 java.io.File 类,但是有一些细微的差别。不过,在许多情况下,您可以使用 Path 接口来替换 File 类的使用。

```
//为了使用 java.nio.file.Path 实例必须创建一个 Path 实例。
//您可以使用Paths 类(java.nio.file.Paths)中的静态方法来创建路径实例,名为Paths.get()。
//要使用 Path 接口和 Paths 类,我们必须首先导入它们。
Path path = Paths.get("/data/myfile.txt");

//创建绝对路径是通过调用 Paths.get()工厂方法,给定绝对路径文件作为参数来完成的。
Path path = Paths.get("/data/myfile.txt");

//相对路径是指从一条路径(基本路径)指向一个目录或文件的路径。
//一条相对路径的完整路径(绝对路径)是通过将基本路径与相对路径相结合而得到的。
//Java NIO Path 类也可以用于处理相对路径。
//可以使用 Paths.get(basePath,relativePath)方法创建一个相对路径。

Path projects = Paths.get("/data", "projects");
Path file= Paths.get("/data","projects/a-project/myfile.txt");
//.表示―当前目录,..表示―父目录或者―上一级目录

//Path 接口的 normalize()方法可以使路径标准化。
//标准化意味着它将移除所有在路径字符串的中间的.和..代码,并解析路径字符串所引用的路径。
String originalPath ="/data/projects/a-project/../another-project";
Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);//path1 =/data/projects/a-project/../another-project
Path path2 = path1.normalize();
System.out.println("path2 = " + path2);//path2 =/data/projects/another-project

//获取本地文件系统或远程文件系统的路径
Path p1 = Paths.get(new URI("路径"));


```

#### Files

Java NIO Files 类(java.nio.file.Files)提供了几种操作文件系统中的文件的方法。这个 Java NIO Files 教程将介绍最常用的这些方法。Files 类包含许多方法,所以如果您需要一个在这里没有描述的方法,那么请检查 JavaDoc。

Files 类可能还会有一个方法来实现它。java.nio.file.Files 类与 java.nio.file.Path 实例一起工作,因此在处理Files 类之前,您需要了解 Path 类。

```
//Files.exists()方法检查给定的 Path 在文件系统中是否存在。
Path path = Paths.get("data/logging.properties");
boolean pathExists =Files.exists(path,new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
//注意 Files.exists()方法的第二个参数。
//这个参数是一个选项数组,它影响Files.exists()如何确定路径是否存在。
//在上面的例子中的数组包含LinkOption.NOFOLLOW_LINKS,这意味着 Files.exists()方法不应该在文件系统中跟踪符号链接,以确定文件是否存在。

//Files.createDirectory()方法,用于根据 Path 实例创建一个新目录

Path path = Paths.get("data/subdir");
try {
Path newDir = Files.createDirectory(path);
} catch(FileAlreadyExistsException e){
// 目录已经存在
} catch (IOException e) {
// 其他发生的异常
e.printStackTrace();
}

//Files.copy()方法从一个路径拷贝一个文件到另外一个目录
Path sourcePath= Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");
try {
Files.copy(sourcePath, destinationPath);
} catch(FileAlreadyExistsException e) {
// 目录已经存在
} catch (IOException e) {
// 其他发生的异常
e.printStackTrace();
}

//可以强制 Files.copy()覆盖现有的文件。
Files.copy(sourcePath, destinationPath,StandardCopyOption.REPLACE_EXISTING);

//Java NIO Files 还包含一个函数,用于将文件从一个路径移动到另一个路径。
//移动文件与重命名相同,但是移动文件既可以移动到不同的目录,也可以在相同的操作中更改它的名称。
//是的,java.io.File 类也可以使用它的 renameTo()方法来完成这个操作,但是现在已经在 java.nio.file.Files 中有了文件移动功能。

Path sourcePath= Paths.get("data/logging-copy.properties");
Path destinationPath =Paths.get("data/subdir/logging-moved.properties");
try {
Files.move(sourcePath, destinationPath,StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
  //移动文件失败
e.printStackTrace();
}

//Files.delete()方法可以删除一个文件或者目录。
Path path = Paths.get("data/subdir/logging-moved.properties");
try {
Files.delete(path);
} catch (IOException e) {
// 删除文件失败
e.printStackTrace();
}

//Files.walkFileTree()方法包含递归遍历目录树的功能。
//walkFileTree()方法将 Path 实例和 FileVisitor 作为参数。
//Path 实例指向您想要遍历的目录。FileVisitor 在遍历期间被调用。

//必须自己实现 FileVisitor 接口,并将实现的实例传递给 walkFileTree()方法。
Files.walkFileTree(path, new FileVisitor<Path>() {
@Override
public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
  System.out.println("pre visit dir:" + dir);
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
System.out.println("visit file: " + file);
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
System.out.println("visit file failed: " + file);
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult postVisitDirectory(Path dir, IOException
exc) throws IOException {
System.out.println("post visit directory: " + dir);
return FileVisitResult.CONTINUE;
}
});

//这四个方法中的每个都返回一个 FileVisitResult 枚举实例。
//CONTINUE 继续意味着文件的执行应该像正常一样继续
// TERMINATE 终止意味着文件遍历现在应该终止
//SKIP_SIBLING 跳过同级意味着文件遍历应该继续,但不需要访问该文件或目录的任何同级
// SKIP_SUBTREE 跳过子级意味着文件遍历应该继续,但是不需要访问这个目录中的子目录
//这个值只有从 preVisitDirectory()返回时才是一个函数。如果从任何其他方法返回,它将被解释为一个 CONTINUE 继续。

//这里是一个用于扩展 SimpleFileVisitor 的 walkFileTree(),以查找一个名为 README.txt 的文件:

Path rootPath = Paths.get("data");
String fileToFind = File.separator + "README.txt";
try {
Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
  @Override
public FileVisitResult visitFile(Path file,
BasicFileAttributes attrs) throws IOException {
String fileString = file.toAbsolutePath().toString();
//System.out.println("pathString = " + fileString);
if(fileString.endsWith(fileToFind)){
System.out.println("file found at path: " +
file.toAbsolutePath());
return FileVisitResult.TERMINATE;
}
return FileVisitResult.CONTINUE;
}
});
} catch(IOException e){
e.printStackTrace();
}

//Files.walkFileTree()也可以用来删除包含所有文件和子目录的目录。
//Files.delete()方法只会删除一个目录,如果它是空的。
//通过遍历所有目录并删除每个目录中的所有文件(在 visitFile())中,
//然后删除目录本身(在postVisitDirectory()中),您可以删除包含所有子目录和文件的目录。

Path rootPath = Paths.get("data/to-delete");
try {
Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
@Override
public FileVisitResult visitFile(Path file,
BasicFileAttributes attrs) throws IOException {
System.out.println("delete file: " + file.toString());
Files.delete(file);
return FileVisitResult.CONTINUE;
}
@Override
public FileVisitResult postVisitDirectory(Path dir,
IOException exc) throws IOException {
Files.delete(dir);
System.out.println("delete dir: " + dir.toString());
return FileVisitResult.CONTINUE;
}
});
} catch(IOException e){
e.printStackTrace();
}

```




参考:
1. [Java NIO类概述](https://www.jianshu.com/p/b215877441ba)
2. [Java NIO](https://pan.baidu.com/s/1uzTW4)
3. [示例代码](http://www.javanio.info/filearea/bookexamples/)
