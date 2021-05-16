---
title: Apache Commons IO
date: 2019-04-22 20:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

### Commons IO

Apache Commons IO是Apache基金会创建并维护的Java IO库。它提供了许多类使得开发者的常见任务变得简单，同时减少重复代码，这些代码可能遍布于每个独立的项目中，你却不得不重复的编写。这些类由经验丰富的开发者维护，对各种问题的边界条件考虑周到，并持续修复相关bug。

<!--more-->

下载地址：http://commons.apache.org/proper/commons-io/download_io.cgi

也可以使用Maven或Grande导入。

```xml
 <dependency>
     <groupId>commons-io</groupId>
     <artifactId>commons-io</artifactId>
     <version>2.6</version>
 </dependency>
```

工具类包括FileUtils、IOUtils、FilenameUtils和FileSystemUtils，前三者的方法并没有多大的区别，只是操作的对象不同，故名思议：FileUtils主要操作File类，IOUtils主要操作IO流，FilenameUtils则是操作文件名，FileSystemUtils包含了一些JDK没有提供的用于访问文件系统的实用方法。当前，只有一个用于读取硬盘空余空间的方法可用。

不同的计算机体系结构使用不同约定的字节排序。在所谓的“低位优先”体系结构中（如Intel），低位字节处于内存中最低位置，而其后的字节，则处于更高的位置。在“高位优先”的体系结构中（如Motorola），这种情况恰恰相反。

这个类库上有两个相关类：

```
EndianUtils包含用于交换java原对象和流之间的字节序列。
SwappedDataInputStream类是DataInput接口的一个实例。使用它，可以读取非本地的字节序列。
```

org.apache.commons.io.LineIterator类提供了一个灵活的方式与基于行的文件交互。可以直接创建一个实例，或者使用FileUtils或IOUtils的工厂方法来创建。

org.apache.commons.io.filefilter包定义了一个合并了java.io.FileFilter以及java.io.FilenameFilter的接口(IOFileFilter)。除此之外，这个包还提供了一系列直接可用的IOFileFilter的实现类，可以通过他们合并其它的文件过滤器。比如，这些文件过滤器可以在列出文件时使用或者在使用文件对话框时使用。

org.apache.commons.io.comparator包为java.io.File提供了一些java.util.Comparator接口的实现。例如，可以使用这些比较器对文件集合或数组进行排序。

org.apache.commons.io.input和org.apache.commons.io.output包中包含的针对数据流的各种各样的的实现。包括：

```
空输出流－默默吸收发送给它的所有数据
T型输出流－全用两个输出流替换一个进行发送
字节数组输出流－这是一个更快版本的JDK类
计数流－计算通过的字节数
代理流－使用正确的方法委拖
可锁写入－使用上锁文件提供同步写入
等等
```

#### FileUtils 文件操作工具类

复制文件夹

```
//复制文件夹（文件夹里面的文件内容也会复制），file1和file2平级。
//参数1：文件夹； 参数2：文件夹
void copyDirectory( file1 , file2 );
//复制文件夹到另一个文件夹。 file1是file2的子文件夹.
//参数1：文件夹； 参数2：文件夹
void copyDirectoryToDirectory( file1 , file2 );
//复制文件夹，带有文件过滤功能
void copyDirectory(File srcDir, File destDir, FileFilter filter)
```

复制文件

```
void copyFile(final File srcFile, final File destFile) //复制文件到另外一个文件
void long copyFile(final File input, final OutputStream output) //复制文件到输出流
void copyFileToDirectory( file1 , file2)  //复制文件到一个指定的目录
//把输入流里面的内容复制到指定文件
void copyInputStreamToFile( InputStream source, File destination)
//把URL 里面内容复制到文件。可以下载文件。
//参数1：URL资源 ； 参数2：目标文件
void copyURLToFile(final URL source, final File destination)
```

把字符串写入文件

```
//把URL 里面内容复制到文件。可以下载文件。
//参数1：URL资源 ； 参数2：目标文件；参数3：http连接超时时间 ； 参数4：读取超时时间
void copyURLToFile(final URL source, final File destination,final int connectionTimeout, final int readTimeout)
void writeStringToFile(final File file, final String data, final String encoding)
//参数1：需要写入的文件，如果文件不存在，将自动创建。  参数2：需要写入的内容
//参数3：编码格式     参数4：是否为追加模式（ ture: 追加模式，把字符串追加到原内容后面）
void writeStringToFile(final File file, final String data, final Charset encoding, final boolean append)
```

把字节数组写入文件

```
//File:目标文件
//byte[]： 字节数组
//boolean append ： 是否为追加模式
//final int off: 数组开始写入的位置 ; final int len ：写入的长度

void writeByteArrayToFile(final File file, final byte[] data)

void writeByteArrayToFile(final File file, final byte[] data, final boolean append)

void writeByteArrayToFile(final File file, final byte[] data, final int off, final int len)

void writeByteArrayToFile(final File file, final byte[] data, final int off, final int len,
    final boolean append)
```

把集合里面的内容写入文件

```
//File file: 目标文件
//Collection<?> lines: 内容集合
//boolean append : 是否为追加模式
//String encoding : 编码方式，比如"UTF-8"
//String lineEnding : 每一行以什么结尾
void writeLines(final File file, final Collection<?> lines)

void writeLines(final File file, final Collection<?> lines, final boolean append)

void writeLines(final File file, final String encoding, final Collection<?> lines)

void writeLines(final File file, final String encoding, final Collection<?> lines,
final boolean append)

void writeLines(final File file, final String encoding, final Collection<?> lines,
final String lineEnding)

void writeLines(final File file, final String encoding, final Collection<?> lines,
final String lineEnding, final boolean append)

void writeLines(final File file, final Collection<?> lines, final String lineEnding)

void writeLines(final File file, final Collection<?> lines, final String lineEnding,
final boolean append)
```

往文件里面写内容

```
/**
* 参数解释
* File file：目标文件
* CharSequence data ： 要写入的内容
* Charset encoding；String encoding ： 编码格式
* boolean append：是否为追加模式
*/

void write(final File file, final CharSequence data, final Charset encoding)

void write(final File file, final CharSequence data, final String encoding)

void write(final File file, final CharSequence data, final Charset encoding, final boolean append)

void write(final File file, final CharSequence data, final String encoding, final boolean append)
```

文件移动

```
//文件夹移动，文件夹在内的所有文件都将移动
void moveDirectory(final File srcDir, final File destDir)

//文件夹移动到另外一个文件内部。boolean createDestDir：如果destDir文件夹不存在，是否要创建一个
void moveDirectoryToDirectory(final File src, final File destDir, final boolean createDestDir)

//移动文件
void moveFile(final File srcFile, final File destFile)

//把文件移动到另外一个文件内部。boolean createDestDir：如果destDir文件夹不存在，是否要创建一个
void moveFileToDirectory(final File srcFile, final File destDir, final boolean createDestDir)

//移动文件或者目录到指定的文件夹内。
//boolean createDestDir：如果destDir文件夹不存在，是否要创建一个
void moveToDirectory(final File src, final File destDir, final boolean createDestDir)
```

清空和删除文件夹

```
//删除一个文件夹，包括文件夹和文件夹里面所有的文件
void deleteDirectory(final File directory)

//清空一个文件夹里面的所有的内容
void cleanDirectory(final File directory)

//删除一个文件，会抛出异常
//如果file是文件夹，就删除文件夹及文件夹里面所有的内容。如果file是文件，就删除。
//如果某个文件/文件夹由于某些原因无法被删除，会抛出异常
void forceDelete(final File file) 

//删除一个文件，没有任何异常抛出
//如果file是文件夹，就删除文件夹及文件夹里面所有的内容。如果file是文件，就删除。
//如果某个文件/文件夹由于某些原因无法被删除，不会抛出任何异常
boolean deleteQuietly(final File file)
```

创建文件夹

```
//创建一个文件夹，如果由于某些原因导致不能创建，则抛出异常
//一次可以创建单级或者多级目录
void forceMkdir(final File directory)

//创建文件的父级目录
void forceMkdirParent(final File file)
```

文件获取输入/输出流

```
//获取输入流
FileInputStream openInputStream(final File file)

//获取输出流
FileOutputStream openOutputStream(final File file)
```

读取文件

```
//把文件读取到字节数组里面
byte[] readFileToByteArray(final File file)

//把文件读取成字符串 ；Charset encoding：编码格式
String readFileToString(final File file, final Charset encoding)

//把文件读取成字符串 ；String encoding：编码格式
String readFileToString(final File file, final String encoding)

//把文件读取成字符串集合 ；Charset encoding：编码格式
List<String> readLines(final File file, final Charset encoding)

//把文件读取成字符串集合 ；String encoding：编码格式
List<String> readLines(final File file, final String encoding)
```

测试两个文件的修改时间那个比较新/老

```
//判断file文件的修改是否比reference文件新
boolean isFileNewer(final File file, final File reference)

//判断file文件的修改是否比 date日期新
boolean isFileNewer(final File file, final Date date)

//判断file文件的修改是否比 timeMillis 毫秒值新
boolean isFileNewer(final File file, final long timeMillis)

//判断file文件的修改是否比reference文件老
boolean isFileOlder(final File file, final File reference)

//判断file文件的修改是否比 date日期老
boolean isFileOlder(final File file, final Date date)

//判断file文件的修改是否比 timeMillis 毫秒值老
boolean isFileOlder(final File file, final long timeMillis)
```

其他

```
//判断文件夹内是否包含某个文件或者文件夹
boolean directoryContains(final File directory, final File child)

//获取文件或者文件夹的大小
long sizeOf(final File file)

//获取临时目录文件
File getTempDirectory()

//获取临时目录路径
String getTempDirectoryPath()

//获取用户目录文件 
File getUserDirectory()

//获取用户目录路径 
static String getUserDirectoryPath()

//如果不存在,新建文件或者创建单级目录或者多级目录
//如果存在,修改文件修改时间 
void touch(final File file)

//比较两个文件内容是否相同
boolean contentEquals(final File file1, final File file2)
```

#### IOUtils

copy：这个方法可以拷贝流，算是这个工具类中使用最多的方法了。支持多种数据间的拷贝。copy内部使用的其实还是copyLarge方法。因为copy能拷贝Integer.MAX_VALUE的字节数据，即2^31-1。

```
copy(inputstream,outputstream)
copy(inputstream,writer)
copy(inputstream,writer,encoding)
copy(reader,outputstream)
copy(reader,writer)
copy(reader,writer,encoding)
```

copyLarge：这个方法适合拷贝较大的数据流，比如2G以上。

```
copyLarge(reader,writer) 默认会用1024*4的buffer来读取
copyLarge(reader,writer,buffer)
```

获取输入流

```
//通过文本获取输入流 ， 可以指定编码格式
InputStream toInputStream(final String input, final Charset encoding)

InputStream toInputStream(final String input, final String encoding)

//获取一个缓冲输入流，默认缓冲大小 1KB
InputStream toBufferedInputStream(final InputStream input) 

//获取一个指定缓冲流的大小的输入流
InputStream toBufferedInputStream(final InputStream input, int size)

//把流的全部内容放在另一个流中
BufferedReader toBufferedReader(final Reader reader)

//把流的全部内容放在另一个流中
BufferedReader toBufferedReader(final Reader reader, int size)
```

获取输入流里面的内容

```
// 输入流 --》 字符串
String toString(final InputStream input, final Charset encoding)

// 输入流 --》 字符串
String toString(final InputStream input, final String encoding)

// 字符输入流 --》 字符串
String toString(final Reader input)

// 字符数组 --》 字符串
String toString(final byte[] input, final String encoding)

//输入流 --》 字符数组
byte[] toByteArray(final InputStream input)

//输入流 --》 字符数组
byte[] toByteArray(final Reader input, final Charset encoding)

//输入流 --》 字符数组
byte[] toByteArray(final Reader input, final String encoding)

//URL   --》 字符数组
byte[] toByteArray(final URI uri)

// URL  --》 字符串
String toString(final URL url, final Charset encoding)

// URL  --》 字符串
String toString(final URL url, final String encoding)

// URLConnection --》 字符串
byte[] toByteArray(final URLConnection urlConn)
```

字符串读写

```
List<String> readLines(InputStream input)

List<String> readLines(InputStream input, final Charset encoding)

List<String> readLines(InputStream input, final String encoding)

List<String> readLines(Reader input)

void writeLines(Collection<?> lines, String lineEnding, OutputStream output)

void writeLines(Collection<?> lines, String lineEnding, OutputStream output, Charset encoding)

void writeLines(Collection<?> lines, String lineEnding, OutputStream output, final encoding)

void writeLines(Collection<?> lines, String lineEnding,Writer writer)
```

write：这个方法可以把数据写入到输出流中

```
write(byte[] data, OutputStream output)
write(byte[] data, Writer output)
write(byte[] data, Writer output, Charset encoding)
write(byte[] data, Writer output, String encoding)

write(char[] data, OutputStream output)
write(char[] data, OutputStream output, Charset encoding)
write(char[] data, OutputStream output, String encoding)
write(char[] data, Writer output)

write(CharSequence data, OutputStream output)
write(CharSequence data, OutputStream output, Charset encoding)
write(CharSequence data, OutputStream output, String encoding)
write(CharSequence data, Writer output)

write(StringBuffer data, OutputStream output)
write(StringBuffer data, OutputStream output, String encoding)
write(StringBuffer data, Writer output)

write(String data, OutputStream output)
write(String data, OutputStream output, Charset encoding)
write(String data, OutputStream output, String encoding)
write(String data, Writer output)
```

read：从一个流中读取内容

```
read(inputstream,byte[])
read(inputstream,byte[],offset,length)
//offset是buffer的偏移值，length是读取的长度

read(reader,char[])
read(reader,char[],offset,length)
```

readFully：这个方法会读取指定长度的流，如果读取的长度不够，就会抛出异常

```
readFully(inputstream,byte[])
readFully(inputstream,byte[],offset,length)
readFully(reader,charp[])
readFully(reader,char[],offset,length)
```

contentEquals：比较两个流是否相等

```
contentEquals(InputStream input1, InputStream input2)
contentEquals(Reader input1, Reader input2)
```

contentEqualsIgnoreEOL：比较两个流，忽略换行符

```
contentEqualsIgnoreEOL(Reader input1, Reader input2)
```

skip：这个方法用于跳过指定长度的流

```
long skip(inputstream,skip_length)
long skip(ReadableByteChannel,skip_length)
long skip(reader,skip_length)
```

skipFully：这个方法类似skip，只是如果忽略的长度大于现有的长度，就会抛出异常。

```
skipFully(inputstream,toSkip)
skipFully(readableByteChannel,toSkip)
skipFully(inputstream,toSkip)
```

ineIterator：读取流，返回迭代器

```
LineIterator lineIterator(InputStream input, Charset encoding)
LineIterator lineIterator(InputStream input, String encoding)
LineIterator lineIterator(Reader reader)
```

close：关闭流

```
//关闭 URLConnection
void close(final URLConnection conn)

//closeQuietly 忽略nulls和异常，关闭某个流
void closeQuietly(final Reader input)

void closeQuietly(final Writer output)

void closeQuietly(final InputStream input)

void closeQuietly(final OutputStream output)

void closeQuietly(final Socket sock)

void closeQuietly(final ServerSocket sock)
```

更多使用方法参考[文档](http://commons.apache.org/proper/commons-io/javadocs/api-2.4/overview-summary.html)

#### 文件监控

使用Commons-io的monitor下的相关类可以处理对文件进行监控，它采用的是观察者模式来实现的

1. 可以监控文件夹的创建、删除和修改
2. 可以监控文件的创建、删除和修改
3. 采用的是**观察者模式**来实现的
4. 采用线程去定时去刷现检测文件的变化情况

**代码分析**

1. FileAlterationListener提供了检测文件夹和文件的变化回调函数的接口，观察模式回调的接口

   - 提供了文件夹的创建、删除和修改的接口

   - 提供了文件的创建、删除和修改的接口

2. FileAlterationListenerAdaptor实现了FileAlterationListener的接口，只是空的实现，可以根据用户的使用情况来处理文件的变化

3. FileAlterationObserver重点的观察者模式的类

   - 提供对某路径下文件监控

   - 使用FileFilter来控制对那些文件进行监控，在实际的使用情况是使用FileFilterUtils来控制，他设置了添加一系列的FileFilter

   - IOCase可以用来对系统的判断，使用是Unix和Windows进行不同的处理，Unix支持文件名的大小写，Windows不区分文件的大小写

4. FileAlterationMonitor类

   - 它继承了Runnable接口

   - 它检测文件的过程是采用一个线程去不停的进行文件的检测

   - 精髓之处，文件的内容的改变处理过程；对于文件的变化有点不太准确，只是判断文件名、文件大小、文件的修改日期来判断

5. FileEntry类

   - 提供了文件夹和文件夹下文件的层级结构

   - 提供了文件是否改变了，采用了备忘录模式(形式上有点，没有很严格的控制)，将上一次的状态进行保存，在比较的时候重新读取新的状态，比较完后备忘录重新将新的状态进行保存。

6. Common-io中提供一序列的文件的FileFilter类，使用是可以看情况查看源代码

   - 提供了FileFilterUtils来提供添加一些列的FileFilter

   - 添加一序列的FileFilter的实现是使用AndFileFilter来实现的

   - 提供相识的OrFileFilter和NotFileFilter

```java
import org.apache.commons.io.monitor.FileAlterationListener;
import org.apache.commons.io.monitor.FileAlterationMonitor;
import org.apache.commons.io.monitor.FileAlterationObserver;

import java.io.File;
/**
 * 文件监听类
 * @author hyp
 * Project name is javalearn
 * Include in com.hyp.learn.base.Demo05
 * hyp create at 2019/6/25
 **/
class FileListener implements FileAlterationListener {

    FileMonitor monitor = null;

    @Override
    public void onStart(FileAlterationObserver observer) {
        System.out.println("onStart");
    }

    @Override
    public void onDirectoryCreate(File directory) {
        System.out.println("onDirectoryCreate:" + directory.getName());
    }

    @Override
    public void onDirectoryChange(File directory) {
        System.out.println("onDirectoryChange:" + directory.getName());
    }

    @Override
    public void onDirectoryDelete(File directory) {
        System.out.println("onDirectoryDelete:" + directory.getName());
    }

    @Override
    public void onFileCreate(File file) {
        System.out.println("onFileCreate:" + file.getName());
    }

    @Override
    public void onFileChange(File file) {
        System.out.println("onFileChange : " + file.getName());
    }

    @Override
    public void onFileDelete(File file) {
        System.out.println("onFileDelete :" + file.getName());
    }

    @Override
    public void onStop(FileAlterationObserver observer) {
        System.out.println("onStop");
    }


}


/**
 * 测试类
 * @author hyp
 * Project name is javalearn
 * Include in com.hyp.learn.base.Demo05
 * hyp create at 2019/6/25
 **/
public class FileMonitor {

    private FileAlterationMonitor monitor = null;

    public FileMonitor(long interval) throws Exception {
        monitor = new FileAlterationMonitor(interval);
    }

    /**
     * 给文件添加监听
     *
     * @param path
     * @param listener
     */
    public void monitor(String path, FileAlterationListener listener) {
        FileAlterationObserver observer = new FileAlterationObserver(new File(path));
        monitor.addObserver(observer);
        observer.addListener(listener);
    }

    public void stop() throws Exception {
        monitor.stop();
    }

    public void start() throws Exception {
        monitor.start();
    }

    public static void main(String[] args) throws Exception {
        //初始化监听
        FileMonitor m = new FileMonitor(1000);//设置监控的间隔时间
        //指定文件夹，添加监听
        m.monitor("/home/hyp/Downloads/", new FileListener());
        //开启监听
        m.start();
    }
}

```

#### Tailer用来监控文件

在Linux下有使用tail命令，在Commons-io中也提供这种方法，他采用的是线程方式来监控文件内容的变化

1. Tailer类（采用线程的方式进行文件的内容变法）

2. TailerListener类

3. TailerListenerAdapter类，该类是集成了TailerListener的实现空的接口方式

```java

import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;
import org.apache.commons.io.input.Tailer;
import org.apache.commons.io.input.TailerListenerAdapter;
 
import java.io.File;
public class TailerTest {
 
    public static void main(String []args) throws Exception{
        TailerTest tailerTest = new TailerTest();
        tailerTest.test();
        boolean flag = true;
        File file = new File("/home/hyp/Downloads/test.java");
 
        while(flag){
            Thread.sleep(1000);
            FileUtils.write(file,""+System.currentTimeMillis()+ IOUtils.LINE_SEPARATOR,true);
        }
 
    }
 
    public void test() throws Exception{
        File file = new File("C:/Users/yezi/Desktop/test/1.txt");
        FileUtils.touch(file);
 
        Tailer tailer = new Tailer(file,new TailerListenerAdapter(){
 
            @Override
            public void fileNotFound() {  //文件没有找到
                System.out.println("文件没有找到");
                super.fileNotFound();
            }
 
            @Override
            public void fileRotated() {  //文件被外部的输入流改变
                System.out.println("文件rotated");
                super.fileRotated();
            }
 
            @Override
            public void handle(String line) { //增加的文件的内容
                System.out.println("文件line:"+line);
                super.handle(line);
            }
 
            @Override
            public void handle(Exception ex) {
                ex.printStackTrace();
                super.handle(ex);
            }
 
        },4000,true);
        new Thread(tailer).start();
    }
}
```

### 参考

1. [Commons IO - 教程](https://iowiki.com/commons_io/commons_io_index.html)

