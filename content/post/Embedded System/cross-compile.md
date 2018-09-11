## 安装交叉编译

#### 克隆编译器
```bash
git clone https://github.com/xupsh/CodeSourcery.git
```
>Sourcery CodeBench contains the complete GNUToolchain including all of the following components:	
>
>* CodeSourcery Debug Sprite for ARM	
>		GNU Binary Utilities (Binutils)	
>		GNU C Compiler (GCC)	
>* GNU C Library (GLIBC)
>		GNU C++ Compiler (G++)	
>		GNU C++ Runtime Library (Libstdc++)	
>		GNU Debug Server (GDBServer)	
>* GNU Debugger (GDB)Visit:  http://www.codesourcery.comto access the Sourcery CodeBench support website.

####设置环境变量
```bash
export CROSS_COMPILE=arm-xilinx-linux-gnueabi-
export PATH=/home/cheio/CrossCompile/CodeSourcery/bin:$PATH
```

####可能出现的错误：
./arm-none-linux-gnueabi-gcc: 没有那个文件或目录
> LSB（Linux Standards Base）是一套核心标准，它保证了LINUX发行版同LINUX应用程序之间的良好结合

```bash
sudo apt-get install lsb-core
```
#### 安装完成，执行
```bash
arm-xilinx-linux-gnueabi-gcc -v
```
可以看到正确的输出
```plain
gcc version 4.7.2 (Sourcery CodeBench Lite 2012.09-104)
```
#### 可以进行交叉编译
```bash
arm-xilinx-linux-gnueabi-gcc -o b testcp.c
# 但是会出现bash识别不了的错误
# 采用静态库编译可以成功运行
arm-xilinx-linux-gnueabi-gcc -static -o c testcp.c
```
#### 安装文件

如果不采用git，也有安装文件

下载地址[百度网盘](https://pan.baidu.com/s/1nvMWAHN)

但是没有试过



## 交叉编译opencv

####make-gui
`Configure`

generator为`Unix Makefiles`

`arm-linux`

`/home/cheio/CrossCompile/CodeSourcery/bin/arm-xilinx-linux-gnueabi-gcc`

`/home/cheio/CrossCompile/CodeSourcery/bin/arm-xilinx-linux-gnueabi-g++`



Target Root`/home/cheio/CrossCompile/CodeSourcery/bin`

####修改cmake的配置

cmake配置修改工作如下：

​    （1）加上BUILD_PNG和BUILD_JPEG

​    （2）去掉WITH_TIFF，WITH_OPENEXR和BUILD_EXAMPLES

​    （3）修改CMAKE_BUILD_TYPE为Release

​    （4）修改CMAKE_INSTALL_PREFIX为/home/cheio/opencv/opencv-arm

重新Configure, Generate，生成makefile文件。

如果编译静态库：取消勾选`BUILD_SHARED_LIBS`

####修改编译配置

```cmake
//Flags used by the linker.
CMAKE_EXE_LINKER_FLAGS:STRING=
```

改为

```cmake
//Flags used by the linker.
CMAKE_EXE_LINKER_FLAGS:STRING= -lpthread -lrt -ldl
```

```bash
make
```

进行编译

#### Could not read symbols错误

```plain
../../3rdparty/lib/libzlib.a: could not read symbols: Bad value
```

进入opencv目录下的3rdparty的对应目录CMakeFiles下的每个`.dir`目录中，打开`flags.make`，在C_FLAGS = 后添加`-O3 -fPIC`用来支持在64位机上编译
如../3rdparty/zlib/CMakeFiles/zlib.dir/flags.make
修改`C_FLAGS =  -W -Wall `为 `C_FLAGS =  -O3 -fPIC  -W -Wall`
和
修改`CXX_FLAGS =  -W -Wall `为 `C_FLAGS =  -O3 -fPIC  -W -Wall`

#### 安装

```
make install
```

拷到开发板/lib

```bash
sudo cp -d libopencv_*  /media/cheio/EXT/lib
```

拷到arm编译器lib

```bash
sudo cp -d libopencv_* /home/cheio/CrossCompile/CodeSourcery/arm-xilinx-linux-gnueabi/lib
```

arm编译test
```bash
arm-xilinx-linux-gnueabi-g++ -Wno-psabi -I /home/cheio/opencv/opencv-arm/include/ -L /home/cheio/opencv/opencv-arm/lib/ -lopencv_core -lopencv_highgui -lpthread -lrt -ldl -statc -o test test.cpp
# 静态库编译
```



```bash
ls /dev | grep ttyUSB
# 用minicom连接 115200
```



```bash
cheio@cuvm:~/test$ arm-xilinx-linux-gnueabi-g++ -Wno-psabi -I /home/cheio/opencv/opencv-arm/include/ -L /home/cheio/opencv/opencv-arm/lib/ -lopencv_core -lopencv_highgui -lpthread -lrt -ldl -static -o test test.cpp


g++ -Wno-psabi -I /usr/include/ -L /usr/lib/ -lopencv_core -lopencv_highgui -lpthread -lrt -ldl -o test-read test-read.cpp


g++ -Wno-psabi -lopencv_core -lopencv_highgui -lpthread -lrt -ldl -o test-read test-read.cpp

/home/cheio/CrossCompile/CodeSourcery/bin/../lib/gcc/arm-xilinx-linux-gnueabi/4.7.2/../../../../arm-xilinx-linux-gnueabi/bin/ld: cannot find -lopencv_core
/home/cheio/CrossCompile/CodeSourcery/bin/../lib/gcc/arm-xilinx-linux-gnueabi/4.7.2/../../../../arm-xilinx-linux-gnueabi/bin/ld: cannot find -lopencv_highgui
collect2: error: ld returned 1 exit status

arm-xilinx-linux-gnueabi-g++ -Wno-psabi -I /home/cheio/opencv/opencv-arm/include/ -L /home/cheio/opencv/opencv-arm/lib/ -lopencv_core -lopencv_highgui -lpthread -lrt -ldl -static -o test test.cpp


arm-xilinx-linux-gnueabi-g++ -Wno-psabi -I /home/cheio/opencv/opencv-arm/include/ -L /home/cheio/opencv/opencv-arm/lib/libopencv_core.a /home/cheio/opencv/opencv-arm/lib/libopencv_highgui.a /home/cheio/opencv/opencv-arm/lib/libopencv_imgproc.a -lpthread -lrt -ldl -static -o test-read test-read.cpp


arm-xilinx-linux-gnueabi-g++ -Wno-psabi -I /home/cheio/opencv/opencv-arm/include/ -L /home/cheio/opencv/opencv-arm/lib/libopencv_core.a /home/cheio/opencv/opencv-arm/lib/libopencv_highgui.a -lpthread -lrt -ldl -static test.cpp -o test


arm-xilinx-linux-gnueabi-g++ -Wno-psabi -I /usr/local/opencv/install_opencv/include/opencv/ -L /usr/local/opencv/install_opencv/lib/ -lopencv_core -lopencv_highgui -lpthread -lrt -o test test.cpp


```

```
# Package Information for pkg-config
prefix=/home/cheio/opencv/opencv-arm
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir_old=${prefix}/include/opencv
includedir_new=${prefix}/include
Name: OpenCV
Description: Open Source Computer Vision Library
Version: 2.4.10
Libs: -L${libdir} -lopencv_core -lopencv_imgproc -lopencv_highgui -lopencv_ml -lopencv_video -lopencv_features2d -lopencv_calib3d -lopencv_objdetect -lopencv_contrib -lopencv_legacy -lopencv_flann
cflags: -I${includedir_old} -I${includedir_new}
```

```bash
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/cheio/opencv/opencv-arm/pkgconfig
export PKG_CONFIG_PATH
arm-xilinx-linux-gnueabi-g++ `pkg-config --cflags opencv-arm` `pkg-config –libs opencv-arm` -lpthread -lrt -static -o test-read test-read.cpp
```

```makefile
CC = /home/cheio/CrossCompile/CodeSourcery/bin/arm-xilinx-linux-gnueabi-g++
#AR = /usr/bin/arm-linux-gnueabihf-ar
CFLAGS += -I./include
LDFLAGS = -lstdc++  -lrt -lpthread
LDFLAGS += -L./libs/opencv -lopencv_core -lopencv_highgui -lopencv_imgproc -lm
LDFLAGS += -L./libs/3rd -lzlib
 
TARGET =  test-read
#TARGET = libXXX.a
(TARGET) : test-read.o
	$(CC) -o $(TARGET) $^ $(LDFLAGS) -static
       #$(AR) r $(TARGET) $^
 
test.o: test-read.cpp
	$(CC) -c $(CFLAGS) $^
 
clean :
	rm -rf test-read.o $(TARGET)
```

```bash
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/cheio/opencv/opencv-arm/pkgconfig
```

