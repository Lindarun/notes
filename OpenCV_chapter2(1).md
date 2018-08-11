[TOC]
## 处理文件，摄像头和图形界面
### 基本I/O脚本
#### 读写图像文件
**imread**函数和**imwrite**函数能支持各种静态图像文件格式。不同系统支持的文件格式不一样，但都支持**BMP**，通常还支持**PNG，JPEG，TIFF**。   
每个像素都会有一个值（但不同格式表示像素的值有所不同）。   
这里通过二维numpy数组简单创建一个黑色正方形图像
    
    img = numpy.zeros((3, 3), dtype=numpy.uint8)
可得到一个二维array数组，每个数记为一个像素，数的范围即为像素的范围，这里是0~255（八位）。
利用**cvtColor函数转换成BGR格式
    
    img = cv2.cvtColor(img, cv2.COLOR_GRAY2BGR)
可得到一个三元数组，每个表示一个通道（分别为B，G，R）。   
查看图像的结构**shape**，返回行和列，若有颜色还返回通道数
    
    >>>img.shape
    (3, 3, 3)
将PNG格式转化为JPG格式
    
    image = cv2.imread('filename.png')
    cv2.imwrite('filename.jpg', img)
加载PNG文件作为灰度图像（在这过程会丢失所有颜色信信）
    
    grayimage = cv2.imread('filename.png', cv2.IMREAD_GRAYSCALE)
>注意：无论采用哪种模式，imread()函数都会删除所有alpha通道（透明度）的信息。imwrite函数要求图像为BGR或灰度格式，并且每个通道都有一定的位，输出格式要支持这些通道。   
#### 图像与原始字节之间的转换
一个像素通常有每个通道的一个字节表示。   
可用表达式访问像素的值，
    
    img[0, 0]  # 灰度图像的y(0表示顶部) x(0表示左边)坐标
    img[0, 0, 0]  # 彩色图像 第三个值表示通道名
>像素值为255表现为白色。0为黑色   

若图像每个通道为8位，则可显示转换为标准一维python bytearray格式
    
    byte_array = bytearray(image)
>**bytearray**函数是python内置的，返回一个新的字节数组。
bytearray类是range 0 < = x < 256的一个可变序列。
>- 如果它是一个字符串，那么您还必须给出编码(以及可选的错误)参数;bytearray()然后使用str.encode()将字符串转换为字节。
>- 如果它是一个整数，那么数组将具有这个大小，并将用null字节初始化。
>- 如果它是符合缓冲区接口的对象，则将使用对象的只读缓冲区来初始化字节数组。
>- 如果它是可迭代的，那么它必须是range 0 < = x < 256的整数的迭代，它被用作数组的初始内容   

#### 使用numpy.array访问图像数据
加载OpenCV图像最简单最常见的方式就是使用imread函数，该函数返回一幅图像，是一个数组。   
假设改变一个特定元素的颜色，numpy.array提供的**item**方法非常方便
    
    img.item(x, y, channel)
它将返回一个像素的值。   
使用**itemset**方法设定元素在指定通道的值
    
    img.itemset((x, y, channel), pixel)
上述方法只能处理一个像素，应与切片结合使用来处理一块区域。   
应用：将图片一部分拷贝至另一部分
    
    my_roi = img[0:100, 0:100]  # ROI：感兴趣区域
    img[300:400, 300:400] = my_roi
>要确保两个区域大小一样，否则立马报错。

获取图像属性
    
    >>>img.shape
    >>>img.size
    >>>img.dtype
分别返回图像的宽高度通道数，像素的大小，数据类型。   
#### 视频文件的读写
OpenCV提供了**VideoCapture**类和**VideoWriter**类来支持各种格式的视频文件。都应该支持AVI。在到达视频末尾之前VideoCapture可通过**read**函数获取新的帧，每一帧都是基于BGR格式的图像。可将一幅图像传递给VideoWriter的**write**函数，该函数会将这图像加到其所指向的文件中。   
实例：读取avi文件的帧，并采用YUV颜色编码写入另一帧中。
    
    videoCapture = cv2.VideoCapture('filename.avi')
    fps = videoCapture.get(cv2.CAP_PROP_FPS)
    size = (int(videoCapture.get(cv2.CAP_PROP_FRAME_WIDTH)),
         int(videoCapture.get(cv2.CAP_PROP_FRAME_HEIGHT)))
    videoWriter = cv2.VideoWriter(
        'output.avi', cv2.VideoWriter_fourcc('I', '4', '2', '0'), fps, size)
    success, frame = videoCapture.read()
    while success:
      videoWriter.write(frame)
      success, frame = videoCapture.read()
>特别注意：必须要为VideoWriter类构造函数指定视频文件名，若原来存在则会覆盖，也必须为其指定视频编码器（相关常用选项看书23页）   
帧速率fps和帧大小也必须要指定，这些属性可以通过VideoCapture的get函数得到
#### 捕获摄像头的帧
VideoCapture类可以获得摄像头的帧流，不使用视频的文件名来构造而用摄像头的设备索引来构造类。   
下面捕获摄像头10s的视频信息，并将其入avi中
    
    cameraCapture = cv2.VideoCapture(0)
    fps = 30  # 假定
    size = (int(cameraCapture.get(cv2.CAP_PROP_FRAME_WIDTH)),
            int(cameraCapture.get(cv2.CAP_PROP_FRAME_HEIGHT)))
    VideoWriter = cv2.VideoWriter(
     'Output.avi', cv2.VideoWriter_fourcc('I', '4', '2', '0'), fps, size
    )

    success, frame = cameraCapture.read()
    numFramesRemaining = 10 * fps - 1
    while success and numFramesRemaining > 0:
      VideoWriter.write(frame)
      success, frame = cameraCapture.read()
      numFramesRemaining -= 1
    cameraCapture.release()
>然而VideoCapture类get方法不能返回摄像头帧速率的准确值，为了针对摄像头创建合适的类，要么假设，要么使用计时器来测量(better)。   
摄像头数量和顺序由系统决定。为了不让read函数从没有正确打开的类中获取数据，可在执行该函数之后使用**VideoCapture.isOpened**方法做一个判断，该方法返回一个布尔值

#### 在窗口显示图像
**imshow**便可实现该操作
    
    CV2.imshow("Window's name", img)
    cv2.waitKey()
    cv2.destroyAllWindows()  # 释放所有

#### 在窗口显示摄像头帧
OpenCV的**namedWindow(), imshow()和DestroyWindow()**函数允许指定窗口名来创建，显示和销毁窗口。此外任意窗口下都可以通过**waitKey**函数来获取键盘输入，**setMouseCallback()**函数来获取鼠标输入。   
以下代码可以实时显示摄像头帧
    
    clicked = False


    def on_mouse(event, x, y, flags, param):
      global clicked
      if event == cv2.EVENT_LBUTTONUP:
          clicked = True


    cameraCapture = cv2.VideoCapture(0)
    cv2.namedWindow("Window's name")
    cv2.setMouseCallback("Window's name", on_mouse)
    
    print('点击窗口或按任意键停止')
    success, frame = cameraCapture.read()
    while success and cv2.waitKey(1) == -1 and not clicked:
      cv2.imshow("Window's name", frame)
     success, frame = cameraCapture.read()

    cv2.destroyWindow("Window's name")
    cameraCapture.release()
waitKey参数为等待键盘触发出发的时间，时间为毫秒，返回值为-1（无键输入）或者ASCII码
>python提供了一个标准函数ord()，可将字符转换为ASCII码

>在一些系统中，waitKey的返回值可能比其ASCII码大。在所有系统中，可以通过读取返回值的最后一个字节来保证只提取ASCII码
    
    keycode = cv2.waitKey(1)
    if keycode != -1:
        keycode &= 0xFF

OpenCV窗口函数和waitKey函数相互依赖。鼠标回调函数setMouseCallback有五个参数.回调时间参数可以取很多值。详情参考书26页。
### Cameo项目

