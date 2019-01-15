---
date: 2018-11-28
title: "Install OpenCV and Compile cpp file for MAC"
tags:
    - OpenCV
    - cmake
categories:
    - OpenCV
comment: true
---

# Install OpenCV and Compile cpp file for Mac

####安装OpenCV2

```bash
brew install opencv@2
echo 'export PATH="/usr/local/opt/opencv@2/bin:$PATH"' >> ~/.zshrc
export LDFLAGS="-L/usr/local/opt/opencv@2/lib"
export CPPFLAGS="-I/usr/local/opt/opencv@2/include"
export PKG_CONFIG_PATH="/usr/local/opt/opencv@2/lib/pkgconfig"
# 不知道这些环境变量是不是必要的，但还是按照提示执行了
```

#### 编译C++程序

安装cmake

```bash
brew install cmake
```

demo程序

```c++
#include "opencv/cv.h"
#include "opencv/highgui.h"

using namespace cv;

int main(int, char**)
{
    VideoCapture cap(0);
    if(!cap.isOpened()) return -1;

    Mat frame, edges;
    namedWindow("edges",1);
    for(;;)
    {
        cap >> frame;
        cvtColor(frame, edges, CV_BGR2GRAY);
        GaussianBlur(edges, edges, Size(7,7), 1.5, 1.5);
        Canny(edges, edges, 0, 30, 3);
        imshow("edges", edges);
        if(waitKey(30) >= 0) break;
    }
    return 0;
}
```

CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 2.8)
project( demo )
find_package( OpenCV REQUIRED )
add_executable( demo demo.cpp )
target_link_libraries( demo ${OpenCV_LIBS} )
```

编译demo.cpp

```bash
cmake .
make
./demo
```

OpenCV程序中\^C终止，\^Z会挂起，停止响应。Ubuntu中是\^Z终止。

#### 运行Python程序

下载`numpy-1.15.4`

```bash
cd numpy-1.15.4
sudo python setup.py build
sudo python setup.py install
```



```python
import sys
#sys.path.append('/Library/Python/2.7/site-packages/')
#import numpy
sys.path.append('/usr/local/Cellar/opencv@2/2.4.13.7/lib/python2.7/site-packages')
import cv2
```
可以设置`PYTHONPATH`环境变量
```bash
export PYTHONPATH=/Library/Python/2.7/site-packages:/usr/local/Cellar/opencv@2/2.4.13.7/lib/python2.7/site-packages:$PYTHONPATH
```

