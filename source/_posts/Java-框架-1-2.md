---
title: Apache Commons FileUpload、Apache Tika
date: 2019-08-22 22:18:59
tags:
 - Java
 - 框架
categories:
 - Java
 - 框架
---

网站:<http://commons.apache.org/proper/commons-fileupload/>

<!--more-->

文件上传在 WEB 开发中应用的十分广泛，是我们每个 web 开发者应知应会的技术

简单说一下什么是**文件上传**？所谓文件上传就是将我们本地的资源文件通过网络传输发送到服务器上。

**文件上传**的原理其实是IO流实现的，通过二进制流的方式向服务器传输数据，服务器再通过流读取到数据，然后解析成文件，最终保存到服务器上。

**commons-fileupload** 工具包主要是我们用来操作文件上传的小助手，里面封装了对流操作的全过程，大大简化了我们实现文件上传的代码复杂度，只需合理的运用类中的方法就可以达到文件上传的效果。

**细节**

可以成功将文件上传到服务器上面的指定目录当中，但是文件上传功能有许多需要注意的小细节问题，以下列出的几点需要特别注意的

1. 为保证服务器安全，上传文件应该放在外界无法直接访问的目录下，比如放于WEB-INF目录下。
2. 为防止文件覆盖的现象发生，要为上传文件产生一个唯一的文件名。
3. 为防止一个目录下面出现太多文件，要使用hash算法打散存储。
4. 要限制上传文件的最大值。
5. 要限制上传文件的类型，在收到上传文件名时，判断后缀名是否合法。

配置：

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
<!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

准备一个 jsp页面，此页面中要有一个 form表单，此表单的特征如下：

```xml
表单的 method 属性值必须是 post；
表单的 enctype 属性值必须是 multipart/form-data；
表单中必须提供至少一个 <input type="file"> 这样的文件上传项。


<form method="POST" enctype="multipart/form-data" action="upload">
  File to upload: <input type="file" name="upfile"><br/>
  Notes about the file: <input type="text" name="note"><br/>
  <br/>
  <input type="submit" value="Press"> to upload the file!
</form>
```

我们需要建立一个 Servlet 用来处理 表单中的数据，在表单中将 `type属性值为file` 的数据在以下代码中称为“文件数据项”，其他的称为“普通数据项”。在此 Servlet 中需要涉及到三个类，分别是 **DiskFileItemFactory类**、**ServletFileUpload类**和 **FileItem类**。

**DiskFileitemFactory类**负责管理磁盘文件，**ServletFileUpload类**负责上传和解析文件，**FileItem类**负责保存每个表单数据项的信息。

1. Servlet 具体代码的实现

   ```java
   @Override
   protected void doPost(HttpServletRequest request,
                             HttpServletResponse response) throws ServletException, IOException {
   
           // 判断是普通表单，还是文件上传表单
           if (!ServletFileUpload.isMultipartContent(request)) {
               throw new RuntimeException("不是文件上传表单！");
           }
           List<String> fileNames = FileUploadUtils.handleRequest(request);
           request.setAttribute("fileNames", fileNames);
   
           request.getRequestDispatcher("download.jsp").forward(request, response);
   
       }
   ```
   
2. 两个处理“数据项”信息的方法

   一个用来处理“普通数据项”信息的，另一个是用来处理“文件数据项”信息的。

   ```java
   import org.apache.commons.fileupload.FileItem;
   import org.apache.commons.fileupload.FileUploadException;
   import org.apache.commons.fileupload.ProgressListener;
   import org.apache.commons.fileupload.disk.DiskFileItemFactory;
   import org.apache.commons.fileupload.servlet.ServletFileUpload;
   
   import javax.servlet.http.HttpServletRequest;
   import java.io.File;
   import java.io.UnsupportedEncodingException;
   import java.util.ArrayList;
   import java.util.List;
   import java.util.UUID;
   
   /**
    * @author hyp
    * Project name is javaLearn
    * Include in com.hyp.learn.file
    * hyp create at 19-12-14
    **/
   public class FileUploadUtils {
   
       public static final Integer SIZETHRESHOLD = 1024 * 100;
       public static final Integer FILESIZEMAX = 1024 * 1024;
   
       public static final Integer SIZEMAX = 1024 * 1024 * 10;
   
   
       /**
        * 处理上传文件请求
        *
        * @param request
        * @return
        */
       public static List<String> handleRequest(HttpServletRequest request) {
   
           //得到上传文件的保存目录，将上传的文件存放于WEB-INF目录下，不允许外界直接访问，保证上传文件的安全
           String savePath = request.getServletContext().getRealPath("/WEB-INF/upload");
   
           // 判断是普通表单，还是文件上传表单
           if (!ServletFileUpload.isMultipartContent(request)) {
               //按照传统方式获取数据
               return null;
           }
           //上传时生成的临时文件保存目录
           String tempPath = request.getServletContext().getRealPath("/WEB-INF/temp");
   
           File tmpFile = new File(tempPath);
           if (!tmpFile.exists()) {
               //创建临时目录
               tmpFile.mkdir();
           }
   
           //消息提示
           String message = "";
           List<String> fileNames = new ArrayList<String>();
   
   
           try {
   
               // 创建上传所需要的两个对象
               // 1.磁盘文件对象
               DiskFileItemFactory fileFactory = new DiskFileItemFactory();
               //设置工厂的缓冲区的大小，当上传的文件大小超过缓冲区的大小时，就会生成一个临时文件存放到指定的临时目录当中。
               fileFactory.setSizeThreshold(SIZETHRESHOLD);//设置缓冲区的大小为100KB，如果不指定，那么缓冲区的大小默认是10KB
               //设置上传时生成的临时文件的保存目录
               fileFactory.setRepository(tmpFile);
   
               //2. 文件上传对象
               ServletFileUpload fu = new ServletFileUpload(fileFactory);
               //监听文件上传进度
               fu.setProgressListener(new ProgressListener() {
                   @Override
                   public void update(long pBytesRead, long pContentLength, int arg2) {
                       System.out.println("文件大小为：" + pContentLength + ",当前已处理：" + pBytesRead);
                       /**
                        * 文件大小为：14608,当前已处理：4096
                        文件大小为：14608,当前已处理：7367
                        文件大小为：14608,当前已处理：11419
                        文件大小为：14608,当前已处理：14608
                        */
                   }
               });
               // 设置解析文件上传中的文件名的编码格式
               fu.setHeaderEncoding("utf-8");
   
               //设置上传单个文件的大小的最大值，目前是设置为1024*1024字节，也就是1MB
               fu.setFileSizeMax(FILESIZEMAX);
               //设置上传文件总量的最大值，最大值=同时上传的多个文件的大小的最大值的和，目前设置为10MB
               fu.setSizeMax(SIZEMAX);
   
               //4. 创建 list容器用来保存 表单中的所有数据信息
               List<FileItem> items = new ArrayList<FileItem>();
   
   
               // 将表单中的所有数据信息放入 list容器中
               items = fu.parseRequest(request);
   
               System.out.println(items);
   
   
               // 遍历 list容器，处理 每个数据项 中的信息
               for (FileItem item : items) {
                   // 判断是否是普通项
                   if (item.isFormField()) {
                       // 处理 普通数据项 信息
                       FileUploadUtils.handleFormField(item);
                   } else {
                       // 处理 文件数据项 信息
                       FileUploadUtils.handleFileField(item, savePath, fileNames);
                   }
               }
   
   
           } catch (FileUploadException e) {
               System.out.println("单个文件大小不能超过1MB，总文件大小不能超过10MB");
               e.printStackTrace();
           }
   
           return fileNames;
       }
   
       /**
        * @param filename 文件的原始名称
        * @return uuid+"_"+文件的原始名称
        * @Method: makeFileName
        * @Description: 生成上传文件的文件名，文件名以：uuid+"_"+文件的原始名称
        */
       private static String makeFileName(String filename) {  //2.jpg
           //为防止文件覆盖的现象发生，要为上传文件产生一个唯一的文件名
   
           int delimiter = filename.lastIndexOf("/");
           if (delimiter == -1) {
               return UUID.randomUUID() + "_" + filename.substring(delimiter + 1);
           } else {
               return UUID.randomUUID() + "_" + filename;
           }
       }
   
       /**
        * 为防止一个目录下面出现太多文件，要使用hash算法打散存储
        *
        * @param filename 文件名，要根据文件名生成存储目录
        * @param savePath 文件存储路径
        * @return 新的存储目录
        * @Method: makePath
        */
       private static String makePath(String filename, String savePath) {
           //得到文件名的hashCode的值，得到的就是filename这个字符串对象在内存中的地址
           int hashcode = filename.hashCode();
           int dir1 = hashcode & 0xf;  //0--15
           int dir2 = (hashcode & 0xf0) >> 4;  //0-15
           //构造新的保存目录
           String dir = savePath + "/" + dir1 + "/" + dir2;  //upload\2\3  upload\3\5
           //File既可以代表文件也可以代表目录
           File file = new File(dir);
           //如果目录不存在
           if (!file.exists()) {
               //创建目录
               file.mkdirs();
           }
           return dir;
       }
   
   
       //处理“文件数据项”信息的方法
   
       /**
        * 处理 文件数据项
        */
       public static void handleFileField(FileItem item, String savePath, List<String> fileNames) {
           // 获取 文件数据项中的 文件名
           String fileName = item.getName();
           String newFileName = null;
   
   
           // 判断 此文件的文件名是否合法
           if (fileName == null || "".equals(fileName)) {
               return;
           }
   
           // 控制只能上传图片
           if (!item.getContentType().startsWith("image")) {
               return;
           }
   
           // 将文件信息 输出到控制台
           System.out.println("fileName:" + fileName);         // 文件名
           System.out.println("fileSize:" + item.getSize());   // 文件大小
   
   
           int delimiter = fileName.lastIndexOf("/");
           if (delimiter == -1) {
               newFileName = UUID.randomUUID() + "_" + fileName.substring(delimiter + 1);
           } else {
               newFileName = UUID.randomUUID() + "_" + fileName;
           }
   
           //得到文件保存的名称
           String saveFilename = makeFileName(fileName);
           //得到文件的保存目录,使用UUID的前几位创建文件夹
           String realSavePath = makePath(saveFilename, savePath);
   
           //加入list
           fileNames.add(realSavePath + newFileName);
   
           // 将文件保存到服务器上（UUID是通用唯一标识码，不用担心会有重复的名字出现）
           try {
               item.write(new File(realSavePath, newFileName));
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   
       //处理“普通数据项”信息的方法
   
       /**
        * 处理 普通数据项
        *
        * @param item
        */
       public static void handleFormField(FileItem item) {
           // 获取 普通数据项中的 name值
           String fieldName = item.getFieldName();
   
           // 获取 普通数据项中的 value值
           String value = "";
           try {
               value = item.getString("utf-8");  // 以 utf-8的编码格式来解析 value值
           } catch (UnsupportedEncodingException e) {
               e.printStackTrace();
           }
   
           // 输出到控制台
           System.out.println("fieldName:" + fieldName + "--value:" + value);
       }
   
   }
   ```

#### 扩展延伸

**文件上传的常见限制要求**

1. 限制上传文件类别。如：只能上传图片

   ```java
   // 控制只能上传图片
   if (!item.getContentType().startsWith("image")) {
       return;
   }
   ```

   

2. 限制上传文件大小。如：单个文件大小不能超过 10KB，总文件大小不能超过 100KB

   ```java
   // 创建上传所需要的两个对象
   DiskFileItemFactory factory = new DiskFileItemFactory();  // 磁盘文件对象
   ServletFileUpload sfu = new ServletFileUpload(factory);   // 文件上传对象
   
   // 限制单个文件的大小
   sfu.setFileSizeMax(1024*10);
   // 限制上传的总文件大小
   sfu.setSizeMax(1024*100);
   
   // 创建 list容器用来保存 表单中的所有数据信息
   List<FileItem> items = new ArrayList<FileItem>();
   
   // 将表单中的所有数据信息放入 list容器中
   try {
       items = sfu.parseRequest(req);
   } catch (FileUploadException e) {
       System.out.println("单个文件大小不能超过10KB，总文件大小不能超过100KB");
       e.printStackTrace();
   }
   ```

**common-fileupload 包中三大核心类**

1. DiskFileItemFactory类

   **作用：**磁盘文件工厂类，它还可以设置缓冲区大小以及设置临时保存位置。

   ```java
   setSizeThreshold( int sizeThreshold ) 
       // 设置缓冲区大小，默认sizeThreshold的大小为：10240（10KB）。 
   setRepository( File repository ); 
       // 设置临时文件的保存位置，如果不设置，repository为系统的临时目录。 
   DiskFileItemFactory(); 
       // 构造方法，缓冲区大小为默认sizeThreshold和临时文件为目录为默认repository的磁盘文件工厂类。 
   DiskFileItemFactory( int SizeThreshold , File repository ); 
       // 构造方法，指定缓冲区大小和指定临时文件的磁盘文件工厂类。
   ```

2. ServletFileUpload类

   **作用：**文件上传类，实现上传的一些实用方法的集合。

   ```java
   public List<FileItem> parseRequest( HttpSevletRequest request ); 
       // 解析request对象，获取表单中的所有数据信息，并返回一个List<FileItem>集合，
       // 其中每个FileItem就是一个表单数据项信息。
   boolean isMultipartContext( HttpServletRequest request ); 
       // 根据请求对象来判断是普通表单，还是文件上传表单，如果是文件上传表单，则返回true，否则false。
   setFileSizeMax( long fileSizeMax ); 
       // 设置单个文件的上传的大小上限。 
   setSizeMax( long sizeMax ); 
       // 设置总文件上传的大小上限。 
   setHeaderEncoding( Charset charset); 
       // 解决上传文件中的文件名中文乱码的问题。 
   ServletFileUpload( DiskFileItemFactory factory ); 
       // 构造方法，指定磁盘文件工厂类，并使用factory指定的缓冲区大小和临时文件。 
   ```

3. FileItem类

   **作用：**表单数据项类，保存了表单数据项的所有信息，以下是完整的 FileItem类对象信息。

   **普通数据项：** name=null, StoreLocation=XXX.tmp, size=5 bytes, isFormField=true, FieldName=username

   **文件数据项：** name=要要.jpg, StoreLocation=XXX.tmp, size=8739 bytes, isFormField=false, FieldName=file2

   ```java
   isFormField(); 
       // 判断是否为普通数据项。 
   String getFieldName(); 
       // 获取该表单数据项的名称。即：<input>标签中的name属性。 
   String getString( String encoding ); 
       // 以指定的编码格式解析该表单数据项的值。即：<input>标签中的value属性。 
   String getName();
       // 获取上传文件中的文件名。 
   getInputStream()； 
       // 获取上传文件的内容的输入流，使用文件复制就能完成文件的上传。 
   delete();
       // 删除临时文件。
   ```

**解决中文乱码问题**

1. 上传的文件名乱码

   使用 ServletFileUpload类 中的 setHeaderEncoding( String encoding ) 方法。

2. 普通表单数据项的内容乱码

   使用 FileItem 中的 getString( String encoding ) 方法。

#### Apache Tika

Apache Tika是基于java的内容检测和分析的工具包，可检测并提取来自上千种不同文件类型（如PPT，XLS和PDF）中的元数据和结构化文本。 它提供了命令行界面、GUI界面和一个java库。Tika可帮助搜索引擎抓取内容后的数据处理。

上述上传使用commons upload判断文件类型，但是会有攻击者伪造ContentType，因此Tika可以帮助对文件元数据进行操作

**处理过程**

内置解析器会在后台通过外部程序提供的API与之交互，并进行相应的文档内容信息和文档相关信息的解析处理，具体过程如下：

![UTOOLS1576556650318.png](https://i.loli.net/2019/12/17/HOUKslGQIWj21dP.png)

正如上图所展示的，Tika对文档内容信息的解析处理的流程如下：Tika通过MimeType（MIME是MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来打开的方式类型）来实现对一个文档的具体识别工作，通过Language identifier来识别语言。根据MimeType和Language identifier的识别结果，选择调用具体的Parser来解析文档。而处理则由ContentHandler接口来完成。其中parser负责解析具体的文档，当解析到需要进行处理的时候，调用具体的信息处理类中的contentHandler进行解析内容的处理。解析、处理后得到的结果作为返回的值。

另外，关于文档的元信息会在处理的过程中被解析，并保存在Metadata对象中。比如一个文档的最后编辑时间，最后的保存时间，标题，作者以及contentType等。这些信息对于用一些关键信息进行文档检索非常有用。

**重要特点**:

- 统一解析器接口：Tika封装在一个单一的解析器接口的第三方解析器库。由于这个特征，用户逸出从选择合适的解析器库的负担，并使用它，根据所遇到的文件类型。
- 低内存占用：Tika因此消耗更少的内存资源也很容易嵌入Java应用程序。也可以用Tika平台像移动那样PDA资源少，运行该应用程序。
- 快速处理：从应用连结内容检测和提取可以预期的。
- 灵活元数据：Tika理解所有这些都用来描述文件的元数据模型。
- 解析器集成：Tika可以使用可在单一应用程序中每个文件类型的各种解析器库。
- MIME类型检测： Tika可以检测并从所有包括在MIME标准的媒体类型中提取内容。
- 语言检测： Tika包括语言识别功能，因此可以在一个多语种网站基于语言类型的文档中使用。

![UTOOLS1576557217458.png](https://i.loli.net/2019/12/17/MRPeOmLTgE4JF1l.png)



**使用**

```xml
<!-- https://mvnrepository.com/artifact/org.apache.tika/tika-parsers -->
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>1.23</version>
</dependency>
```

解析API是Tika的核心功能，抽象解析操作的复杂性，仅依赖单个方法：

```java
void parse(
  InputStream stream, 
  ContentHandler handler, 
  Metadata metadata, 
  ParseContext context) 
  throws IOException, SAXException, TikaException
```

我们简要解释方法参数：

- stream，从需要被解析文档创建的InputStream实例
- handler，接收从输入文档解析XHTML SAX事件序列的ContentHandler对象，负责处理事件并以特定的形式导出结果。
- metadata,元数据对象，它在解析器中传递元数据属性
- context，带有上下文相关信息的ParseContext实例，用于自定义解析过程。

如果从输入流读取失败，则parse方法抛出IOException异常，从流中获取的文档不能被解析抛TikaException异常，处理器不能处理事件则抛SAXException异常。

当解析文档时，Tika尽量重用已经存在的解析库，如Apache POI或PDFBox。因此，大多数解析器实现类仅适配这些外部类库。下面，我们将了解如何使用处理程序和元数据参数来提取文档的内容和元数据。为了方便，我们能使用Tika的门面类调用解析器Api。

Apache Tika可以根据文档本身而无需额外信息则可以自动检测文档的类型和语言。

**检测文档类型**

可以通过Detector接口的实现类检测文档类型，其仅有一个方法：

```
MediaType detect(java.io.InputStream input, Metadata metadata) throws IOException
```

方法带有文档流和其元数据参数，返回MediaType对象用于描述文档最佳可能相关类型。

元数据并不是探测器所依赖的唯一信息来源，还可以使用对应在文件开头的特殊模式的魔术字节，或者将检测过程委托给更合适的检测器。

事实上，探测器算法依赖实现。举例说明，默认探测器首先使用魔术字节，然后是元数据属性。如果还不能获得文档类型，则使用service loader去发现所有有效的探测器，然后依次尝试。

**语言检测**

除了文档类型之外，Tika还可以在没有元数据信息的情况下识别它的语言。
Tika早期版本，使用LanguageIdentifier 实例检测文档语言。然而，LanguageIdentifier对于web服务的支持已经被弃用。

语言检测服务现在通过抽象类LanguageDetector的子类提供。使用web服务，您还可以访问完全成熟的在线翻译服务，如谷歌翻译或Microsoft Translator。为了简洁起见，我们不详细讨论这些服务。

**示例**

```java
//下面代码可以检测InputStream对应文档类型：
public static String detectDocTypeUsingDetector(InputStream stream) 
  throws IOException {
    Detector detector = new DefaultDetector();
    Metadata metadata = new Metadata();

    MediaType mediaType = detector.detect(stream, metadata);
    return mediaType.toString();
}

//测试代码
//假设我们有一个pdf文件在类路径下，扩展名我们故意修改为txt去误导tika，但实际真实类型仍然能被检测：
  @Test
    public void whenUsingDetector_thenDocumentTypeIsReturned() throws IOException {
        InputStream stream = this.getClass().getClassLoader()
                .getResourceAsStream("tika.txt");
        String mediaType = TikaAnalysis.detectDocTypeUsingDetector(stream);

        assertEquals("application/pdf", mediaType);
        stream.close();
    }


//很明显，错误的文件扩展名没能让tika犯错，因为pdf的魔术字节在pdf文件的开头。为了简洁，我们可以使用Tika的门面类重写之前的方法:
public static String detectDocTypeUsingFacade(InputStream stream) 
  throws IOException {

    Tika tika = new Tika();
    String mediaType = tika.detect(stream);
    return mediaType;
}
//抽取文档内容
public static String extractContentUsingParser(InputStream stream) 
  throws IOException, TikaException, SAXException {

    Parser parser = new AutoDetectParser();
    ContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    ParseContext context = new ParseContext();

    parser.parse(stream, handler, metadata, context);
    return handler.toString();
}
//类路径下有word文档，内容如下：
//Apache Tika - a content analysis toolkit
//The Apache Tika™ toolkit detects and extracts metadata and text ...
//测试代码
@Test
public void whenUsingParser_thenContentIsReturned() 
  throws IOException, TikaException, SAXException {
    InputStream stream = this.getClass().getClassLoader()
      .getResourceAsStream("tika.docx");
    String content = TikaAnalysis.extractContentUsingParser(stream);

    assertThat(content, 
      containsString("Apache Tika - a content analysis toolkit"));
    assertThat(content, 
      containsString("detects and extracts metadata and text"));

    stream.close();
}

//当然，使用Tika类会更方便：
public static String extractContentUsingFacade(InputStream stream) 
  throws IOException, TikaException {

    Tika tika = new Tika();
    String content = tika.parseToString(stream);
    return content;
}

//抽取元数据
public static Metadata extractMetadatatUsingParser(InputStream stream) 
  throws IOException, SAXException, TikaException {

    Parser parser = new AutoDetectParser();
    ContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    ParseContext context = new ParseContext();

    parser.parse(stream, handler, metadata, context);
    return metadata;
}
//在类路径下放一个tika.xlsx文件进行单元测试：
@Test
public void whenUsingParser_thenMetadataIsReturned() 
  throws IOException, TikaException, SAXException {
    InputStream stream = this.getClass().getClassLoader()
      .getResourceAsStream("tika.xlsx");
    Metadata metadata = TikaAnalysis.extractMetadatatUsingParser(stream);

    assertEquals("org.apache.tika.parser.DefaultParser", 
      metadata.get("X-Parsed-By"));
    assertEquals("Microsoft Office User", metadata.get("creator"));

    stream.close();
}
//当然，另一版本可以通过Tika类实现：
public static Metadata extractMetadatatUsingFacade(InputStream stream) 
  throws IOException, TikaException {
    Tika tika = new Tika();
    Metadata metadata = new Metadata();

    tika.parse(stream, metadata);
    return metadata;
}

```



### 参考

1. https://www.cnblogs.com/qlqwjy/p/9801364.html

