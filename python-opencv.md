# python-opencv-----01

## 1、安装python-opencv

### 临时使用

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple python-opencv
```

### 设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip install python-opencv
```

## 2、初识opencv

### 2.1 目标

* 在这里你将学会怎样读入一幅图像，怎样显示一幅图像，以及如何保存一幅图像
*  你将要学习如下函数：`cv2.imread()，cv2.imshow()，cv2.imwrite()`
* 如果你愿意的话，我会叫你如何使用 Matplotlib 显示一幅图片

### 2.2 读入图像

使用函数 **cv2.imread()** 读入图像，这幅图像应该在此程序的工作路径，或者给函数提供完整路径，第二个参数是要告诉函数应该如何读取这幅图片。

*  **cv2.IMREAD_COLOR**：读入一副彩色图像。图像的透明度会被忽略，这是默认参数。
* **cv2.IMREAD_GRAYSCALE**：以灰度模式读入图像
* **cv2.IMREAD_UNCHANGED**：读入一幅图像，并且包括图像的 alpha 通道

```python
# -*- coding: utf-8 -*-
import cv2 as cv

img = cv.imread("C:\Users\Ryan\Pictures\1.jpg")
```

### 2.3显示图像

使用函数 **cv2.imshow()** 显示图像。窗口会自动调整为图像大小。**第一个参数是窗口的名字，其次才是我们的图像**。你可以创建多个窗口，只要你喜欢，但是必须给他们不同的名字

```
# 显示图像
cv.imshow('image', img)
cv.waitKey(0)
cv.destroyAllWindows()
```

截图如下：

![](http://img.vim-cn.com/2d/a7df115ab172826e692a62452c5a10a6085cc1.png)

显示了名称：image 以及图像

**cv2.waitKey()** 是一个键盘绑定函数。需要指出的是它的时间尺度是毫秒级。函数等待特定的几毫秒，看是否有键盘输入。特定的几毫秒之内，如果按下任意键，这个函数会返回按键的 ASCII 码值，程序将会继续运行。如果没有键盘输入，返回值为 -1，如果我们设置这个函数的参数为 0，那它将会无限期的等待键盘输入。它也可以被用来检测特定键是否被按下，例如按键 a 是否被按下，这个后面我们会接着讨论。

**cv2.destroyAllWindows()** 可以轻易删除任何我们建立的窗口。如果你想删除特定的窗口可以使用 cv2.destroyWindow()，在括号内输入你想删除的窗口名。



>  建 议：一 种 特 殊 的 情 况 是， **你 也 可 以 先 创 建 一 个 窗 口， 之 后再 加 载 图 像。 这 种 情 况 下， 你 可 以 决 定 窗 口 是 否 可 以 调 整大 小**。 使 用 到 的 函 数 是 **cv2.namedWindow()**。 初 始 设 定 函 数标 签 是 **cv2.WINDOW_AUTOSIZE**。 但 是 如 果 你 把 标 签 改 成**cv2.WINDOW_NORMAL**，你就可以调整窗口大小了。当图像维度太大，或者要添加轨迹条时，调整窗口大小将会很有用

```
cv.namedWindow('image',cv.WINDOW_AUTOSIZE)
```

代码如下：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/9/30 23:05
# @Author  : Juntech
# @File    : open_cv_demo1.py
# @Software: PyCharm

import cv2 as cv

# 读取图像
img = cv.imread("C:/Users/Ryan/Pictures/1.jpg")
cv.namedWindow('image',cv.WINDOW_AUTOSIZE)
# 显示图像
cv.imshow('image', img)
cv.waitKey(0)
cv.destroyAllWindows()
```

### 2.4保存图像

使用函数 **cv2.imwrite()** 来保存一个图像。首先需要一个文件名，之后才是你要保存的图像。

`cv2.imwrite('messigray.png',img)`

### 2.5总结一下

下面的程序将会加载一个灰度图，显示图片，按下’s’键保存后退出，或者按下 ESC 键退出不保存。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/9/30 23:05
# @Author  : Juntech
# @File    : open_cv_demo1.py
# @Software: PyCharm

import cv2 as cv

# 读取图像
img = cv.imread("C:/Users/Ryan/Pictures/1.jpg")
# cv.namedWindow('image',cv.WINDOW_AUTOSIZE)
# 显示图像
cv.imshow('image', img)
k = cv.waitKey(0)
# 判断 键盘输入值  27 --->esc
if k==27:
    cv.destroyAllWindows()
elif k == ord('s'):
    '''保存img文件为messigray.png'''
    cv.imwrite('messigray.png',img)
    '''关闭窗口'''
    cv.destroyAllWindows()
```

按‘s'，保存了文件，不按’s'就什么操作也没有就只是显示一下图像,就关闭窗口

## 3、视频

### 3.1 目标

* 学会读取视频文件，显示视频，保存视频文件
* 学会从摄像头获取并显示视频
* 你将会学习到这些函数：**cv2.VideoCapture()，cv2.VideoWrite()**

### 3.2 用摄像头捕获视频

我们经常需要使用摄像头捕获实时图像。OpenCV 为这中应用提供了一个非常简单的接口。让我们使用摄像头来捕获一段视频，并把它转换成灰度视频显示出来。从这个简单的任务开始吧。为了获取视频，你应该创建一个 VideoCapture 对象。他的参数可以是设备的索引号，或者是一个视频文件。设备索引号就是在指定要使用的摄像头。一般的笔记本电脑都有内置摄像头。所以参数就是 0。你可以通过设置成 1 或者其他的来选择别的摄像头。之后，你就可以一帧一帧的捕获视频了。但是最后，别忘了停止捕获视频。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/9/30 23:33
# @Author  : Juntech
# @File    : open_cv_video.py
# @Software: PyCharm

import cv2 as cv

capture = cv.VideoCapture(0)
while(True):
    ret, frame = capture.read()
    gray = cv.cvtColor(frame,cv.COLOR_BAYER_BG2GRAY)
    cv.imshow('image',gray)
    if cv.waitKey(1) == ord('q'):
        break

capture.release()
cv.destroyAllWindows()
```

**cap.read()** 返回一个布尔值（True/False）。如果帧读取的是正确的，就是 True

> 注意：当你的程序报错时，你首先应该检查的是你的摄像头是否能够在其他程序中正常工作（比如 linux 下的 Cheese）。

### 3.3 从文件中播放视频

与从摄像头中捕获一样，你只需要把设备索引号改成视频文件的名字。在播放每一帧时，使用 **cv2.waiKey() 设置适当的持续时间**。如果设置的太低视频就会播放的非常快，如果设置的太高就会播放的很慢（你可以使用这种方法控制视频的播放速度）。通常情况下 25 毫秒就可以了。

```
• CV_CAP_PROP_POS_MSEC Current position of the video file
in milliseconds.
• CV_CAP_PROP_POS_FRAMES 0-based index of the frame to
be decoded/captured next.
• CV_CAP_PROP_POS_AVI_RATIO Relative position of the
video file: 0 - start of the film, 1 - end of the film.
• CV_CAP_PROP_FRAME_WIDTH Width of the frames in the
video stream.
• CV_CAP_PROP_FRAME_HEIGHT Height of the frames in the
video stream.
• CV_CAP_PROP_FPS Frame rate.
• CV_CAP_PROP_FOURCC 4-character code of codec.
• CV_CAP_PROP_FRAME_COUNT Number of frames in the
video file.
• CV_CAP_PROP_FORMAT Format of the Mat objects returned
by retrieve() .
• CV_CAP_PROP_MODE Backend-specific value indicating the
current capture mode.
• CV_CAP_PROP_BRIGHTNESS Brightness of the image (only
for cameras).
• CV_CAP_PROP_CONTRAST Contrast of the image (only for
cameras).
• CV_CAP_PROP_SATURATION Saturation of the image (only
for cameras).
• CV_CAP_PROP_HUE Hue of the image (only for cameras).
• CV_CAP_PROP_GAIN Gain of the image (only for cameras).
• CV_CAP_PROP_EXPOSURE Exposure (only for cameras).
• CV_CAP_PROP_CONVERT_RGB Boolean flags indicating
whether images should be converted to RGB.
• CV_CAP_PROP_WHITE_BALANCE Currently unsupported
• CV_CAP_PROP_RECTIFICATION Rectification flag for stereo
cameras (note: only supported by DC1394 v 2.x backend currently)
```

## 4.opencv中的绘图函数

### 4.1目标

* 学习使用 OpenCV 绘制不同几何图形
* 你将会学习到这些函数：**cv2.line()，cv2.circle()，cv2.rectangle()，cv2.ellipse()，cv2.putText()** 等。

### 4.2 代码

上面所有的这些绘图函数需要设置下面这些参数：

* img：你想要绘制图形的那幅图像。
* color：形状的颜色。以 RGB 为例，需要传入一个元组，例如：（255,0,0）代表蓝色。对于灰度图只需要传入灰度值。


* thickness：线条的粗细。如果给一个闭合图形设置为 -1，那么这个图形就会被填充。默认值是 1.
* linetype：线条的类型，8 连接，抗锯齿等。默认情况是 8 连接。cv2.LINE_AA为抗锯齿，这样看起来会非常平滑。

### 4.3画线

要画一条线，你只需要告诉函数这条线的起点和终点。我们下面会画一条从左上方到右下角的蓝色线段。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/10/1 7:56
# @Author  : Juntech
# @File    : open_cv_line.py
# @Software: PyCharm

import cv2 as cv
import numpy as np
# 创建一个蓝色的图像
img = np.zeros((512,512,3),np.uint8)
# 画一条5px粗的蓝线
cv.line(img,(0,0),(511,511),(255,0,0),5)
```

### 4.4画矩形

要画一个矩形，你需要告诉函数的左上角顶点和右下角顶点的坐标。这次我们会在图像的右上角话一个绿色的矩形

```
#3px边框厚的顶点是0,0 下定点是511,511矩形
cv.rectangle(img,(0,0),(511,511),(0,255,0),3)

```

### 4.5画圆

要画圆的话，只需要指定圆形的中心点坐标和半径大小。我们在上面的矩形中画一个圆。

```
cv.circle(img,(447,63), 63, (0,0,255), -1)
```

### 4.6画椭圆

画椭圆比较复杂，我们要多输入几个参数。一个参数是中心点的位置坐标。下一个参数是长轴和短轴的长度。椭圆沿逆时针方向旋转的角度。椭圆弧演顺时针方向起始的角度和结束角度，如果是 0 很 360，就是整个椭圆。查看cv2.ellipse() 可以得到更多信息。下面的例子是在图片的中心绘制半个椭圆。

```
cv.ellipse(img,(256,256),(100,50),0,0,180,255,-1)
```

### 4.7画多边形

画多边形，需要指点每个顶点的坐标。用这些点的坐标构建一个大小等于行数 X1X2 的数组，行数就是点的数目。这个数组的数据类型必须为 int32。这里画一个黄色的具有四个顶点的多边形。

```
 pts=np.array([[10,5],[20,30],[70,20],[50,10]], np.int32)
 pts=pts.reshape((-1,1,2))
```

### 4.8 在图片上添加文字

要在图片上绘制文字，你需要设置下列参数：

• 你要绘制的文字

• 你要绘制的位置

• 字体类型（通过查看 cv2.putText() 的文档找到支持的字体）

• 字体的大小

• 文字的一般属性如颜色，粗细，线条的类型等。为了更好看一点推荐使用

**linetype=cv2.LINE_AA。**

在图像上绘制白色的 OpenCV。 

```
font=cv.FONT_HERSHEY_SIMPLEX
cv.putText(img,'OpenCV',(10,500), font, 4,(255,255,255),2)
```

**注意**：所 有 的 绘 图 函 数 的 返 回 值 都 是 None， 所 以 不 能 使 用 img =cv2.line(img,(0,0),(511,511),(255,0,0),5)。

### 结果

下面就是最终结果了，通过你前面几节学到的知识把他显示出来吧。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/10/1 7:56
# @Author  : Juntech
# @File    : open_cv_line.py
# @Software: PyCharm

import cv2 as cv
import numpy as np
# 创建一个蓝色的图像
img = np.zeros((512,512,3),np.uint8)
# 画一条5px粗的蓝线
cv.line(img,(0,0),(511,511),(255,0,0),5)
cv.rectangle(img,(0,0),(511,511),(0,255,0),3)
pts=np.array([[10,5],[20,30],[70,20],[50,10]], np.int32)
pts=pts.reshape((-1,1,2))
cv.circle(img,(447,63), 63, (0,0,255), -1)
cv.ellipse(img,(256,256),(100,50),0,0,180,255,-1)
winname = 'example'
cv.namedWindow(winname)
cv.imshow(winname,img)
cv.waitKey(0)
cv.destroyAllWindows()
```

图像:

![](http://img.vim-cn.com/11/6c2442f792e216e03ed3f8cc1f783cf437dbb1.png)