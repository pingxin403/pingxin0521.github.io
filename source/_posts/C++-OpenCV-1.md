---
title: Ubuntu 18.04 安装Opencv 3.4.6
date: 2019-04-20 13:18:59
tags:
 - C++
 - OpenCV
categories:
 - C++
 - OpenCV
---
### 前言
OpenCV是一个基于BSD许可（开源）发行的跨平台计算机视觉库，可以运行在Linux、Windows、Android和Mac OS操作系统上。它轻量级而且高效——由一系列 C 函数和少量 C++ 类构成，同时提供了Python、Ruby、MATLAB等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。

OpenCV用C++语言编写，它的主要接口也是C++语言，但是依然保留了大量的C语言接口。该库也有大量的Python、Java and MATLAB/OCTAVE（版本2.5）的接口。这些语言的API接口函数可以通过在线文档获得。如今也提供对于C#、Ch、Ruby,GO的支持。

所有新的开发和算法都是用C++接口。一个使用CUDA的GPU接口也于2010年9月开始实现。

<!--more-->

### 安装

安装依赖包
```
sudo apt-get install libgtk2.0-dev
sudo apt-get install pkg-config
sudo apt-get install cmake 
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev
sudo apt-get install libavformat-dev libjpeg.dev libtiff4.dev libswscale-dev libjasper-dev 
sudo apt-get install libcanberra-gtk-module 
```

去官网下载opencv，在本教程中选用的时opencv3.4.3，其他版本的配置方法异曲同工。
下载链接 [http://opencv.org/releases.html](http://opencv.org/releases.html)，选择sources版本。

解压zip包
```
unzip opencv-3.4.3.zip
cd opencv-3.4.3
```

执行cmake
```
# 新建编译文件夹
mkdir bld
cd bld
/* 执行cmake */
cmake ..
```

执行make命令
```
sudo make -j 2  #时间比较漫长
```

执行install命令
```
sudo make install
```

这一步执行完毕之后，Opencv的编译过程就结束了，接下来的工作就是配置一些Opencv的编译环境。

### 配置

首先将OpenCV的库添加到路径，从而可以让系统找到
```
sudo gedit /etc/ld.so.conf.d/opencv.conf
```
执行此命令后打开的可能是一个空白的文件，不用管，只需要在文件末尾添加
```
/usr/local/lib 
```
生效配置文件。执行如下命令使得刚才的配置路径生效：
```
sudo ldconfig
```

配置bash
```
sudo gedit /etc/bash.bashrc
```

在末尾追加：
```
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig 
export PKG_CONFIG_PATH
```
保存，执行如下命令使得配置生效：
```
source /etc/bash.bashrc
```
更新：
```
sudo updatedb
```
至此，所有配置都已经完成。

**小程序测试**

找到 opencv-3.4.6/samples/cpp/example_cmake 目录下，官方已经给出了一个cmake的example，我们可以拿来测试下。按顺序执行：
```
cmake .
make
./opencv_example
```

即可看到打开了摄像头，在左上角有一个hello opencv ，即表示配置成功。

### clion编写opencv

以下是opencv的demo程序需要配置的CmakeList.txt内容和main.cpp的内容
可以直接使用cmake编译，也可以通过clion使用cmake编译


```
cmake_minimum_required(VERSION 3.10)

set(PROJECT_NAME  FaceRecognition)

project(${PROJECT_NAME})

set(CMAKE_CXX_STANDARD 11)

add_executable(${PROJECT_NAME} main.cpp)

# find required opencv
find_package(OpenCV REQUIRED)
# directory of opencv headers
include_directories(${OpenCV_INCLUDE_DIRS})

# directory of opencv library
link_directories(${OpenCV_LIBRARY_DIRS})
# opencv libraries
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})

```
PROJECT指令的语法是：
```
PROJECT(${PROJECT_NAME} [CXX] [C] [Java])
```
你可以用这个指令定义工程名称，并可指定工程支持的语言，支持的语言列表是可以忽略的，这个指令隐式的定义了两个cmake变量:`<projectname>_BINARY_DIR以及<projectname>_SOURCE_DIR`。前者指构建路径，后者指工程路径，即CMakeLists.txt所在的路径。

同时cmake系统也帮助我们预定义了PROJECT_BINARY_DIR和PROJECT_SOURCE_DIR变量，他们的值分别跟opencv_test_BINARY_DIR与opencv_test_SOURCE_DIR一致。

为了统一起见，建议以后直接使用PROJECT_BINARY_DIR，PROJECT_SOURCE_DIR，即使修改了工程名称，也不会影响这两个变量。如果使用了`<projectname>_SOURCE_DIR`，修改工程名称后，需要同时修改这些变量。

接下来是设置cmake要求的最低版本号：
```
cmake_minimum_required(VERSION 3.10)
```
SET指令的语法是：
```
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
```
现阶段，你只需要了解SET指令可以用来显式的定义变量即可。这里我们将变量CMAKE_RUNTIME_OUTPUT_DIRECTORY定义为${opencv_test_SOURCE_DIR}/bin也就是工程路径下的bin目录。

下面介绍一个很重要的指令：find_package这个指令以被用来在系统中自动查找配置构建工程所需的程序库。在linux和unix类系统下这个命令尤其有用。CMake自带的模块文件里有大半是对各种常见开源库的find_package支持，支持库的种类非常多。

当它找到OpenCV程序库之后，就会帮助我们预定义几个变量，OpenCV_FOUND、OpenCV_INCLUDE_DIRS、OpenCV_LIBRARY_DIRS、OpenCV_LIBRARIES，它们分别指是否找到OpenCV，OpenCV的头文件目录，OpenCV的库文件目录，OpenCV的所有库文件列表。接着我们就可以使用这些变量来配置了：

```
include_directories(${OpenCV_INCLUDE_DIRS})
```
这个指令用来设置包含的头文件的路径。
```
link_directories(${OpenCV_LIBRARY_DIRS})
```
这个指令用来设置库文件的路径。
```
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})
```
这个指令用来设置需要的库文件，它的语法是：
```
TARGET_LINK_LIBRARIES(target library1<debug | optimized> library2...)
```
其中的target就是前面设置生成的目标文件（可执行文件）：
```
add_executable(opencv_test src/opencv_test.cpp)
```
这个命令很好理解，首先是可执行文件的名字，然后是源码的名字。因此，这个命令一定要在TARGET_LINK_LIBRARIES之前使用。

现在我们的CMakeLists.txt就介绍完了。



#### 实例主程序


```
#include <cv.h>
#include <highgui.h>

using namespace std;
int main()
{
    IplImage * test;
    test = cvLoadImage("图片路径");//图片路径
    cvNamedWindow("test_demo", 1);
    cvShowImage("test_demo", test);
    cvWaitKey(0);
    cvDestroyWindow("test_demo");
    cvReleaseImage(&test);
    return 0;
}
```
如果还有错误，请自行搜索解决方法，或者联系作者。




参考：
1. [环境配置—Ubuntu 16.04 安装Opencv 3.4.3](https://www.jianshu.com/p/f646448da265)
2. [使用CMake构建OpenCV项目](https://blog.csdn.net/github_30605157/article/details/79839177)
