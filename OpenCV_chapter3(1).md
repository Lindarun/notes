[TOC]
# 处理图像
## 不同色彩空间的转换   
计算机视觉中有三种常有的色彩空间：灰度，BGR以及HSV(Hue, Saturation, Value)   
- 灰度色彩空间是通过去除彩色信息来将其转换成灰阶，对中间处理特别有效，如人脸检测
- BGR，蓝绿红色彩空间，注意计算机软件所用的色彩模型是**加色模型**
- HSV，Hue是色调， Saturation是饱和度， Value是黑暗度(明亮度)   

## 傅里叶变换
利用傅里叶变换，可以区分图像哪些区域的信号（例如图像像素）变化特别强哪些不那么强，从而**标记噪声区域，感兴趣区域，前景和背景等**。   
图像的幅度谱：呈现了原始图像在变化方面的一种表示：把一幅图像中最明亮的像素放到图像中央，然后逐渐变暗，在边缘上的像素最暗。这样可以发现**图像中有多少亮的像素和暗的像素，以及它们分布的百分比**。
### 高通滤波器
高通滤波器(HPF)使监测图像某个区域，然后根据像素与周围像素的亮度差值提升该像素亮度的滤波器。   
以如下的核(kernel)(滤波器矩阵)为例：
    
    [[0, -0.25, 0],
     [-0.25, 1, -0.25],
     [0, -0.25, 0]]
>注意：核是指一组权重的集合，它会应用在原图像的一个区域，并由此生成目标图像的一个像素。比如，大小为7的核意味着每49个原图像的像素会产生目标图像的一个像素。可把核看做一块覆盖在原图像上可移动的毛玻璃片，玻璃片覆盖区域的光线会按某种方式进行扩散混合后透过去。

如果一个像素比他周围的像素更突出，就会提升他的亮度。这在**边缘检测**上尤其有效，它会采用一种称为高频提升滤波器的高通滤波器。高通滤波器和低通滤波器都有个称为**半径**的属性，它决定了多大面积的临近像素参与滤波运算。
>注意：滤波器中所有值加起来为0。   

高斯滤波函数（低通滤波器）：高斯滤波是一种线性平滑滤波，对于除去高斯噪声有很好的效果。高斯算法在官方文档给出的解释是高斯滤波是通过对输入数组的每个点与输入的高斯滤波模板执行卷积计算然后将这些结果一块组成了滤波后的输出数组，通俗的讲就是高斯滤波是对整幅图像进行加权平均的过程，每一个像素点的值都由其本身和邻域内的其他像素值经过加权平均后得到。高斯滤波的具体操作是：用一个模板（或称卷积、掩模）扫描图像中的每一个像素，用模板确定的邻域内像素的加权平均灰度值去替代模板中心像素点的值。
    
    blurred = cv2.GaussianBlur(img, (x,y), std)
img为源图像，（x，y）为卷积核大小，为奇数，std为标准差，为0时计算机会自己计算
>书上代码：出错的原因可能是因为用scipy的’ndimage.convolve’缘故。ndimage提供的卷积，如果想让卷积工作，图像和内核必须有相同的**维数**。其中任何一个尺寸不正确都会导致错误。
### 低通滤波器
低通滤波器(LPF)则是在像素与周围像素的亮度差值小于一个特定值时，平滑该像素的亮度。主要用于**去噪和模糊化**，高斯模糊是最常用的模糊滤波器（平哈滤波器）之一，他是一个削弱高频信号强度的低通滤波器。
## 边缘检测
边缘在人类视觉和计算机视觉中均起着重要的作用。OpenCV提供了许多边缘检测率波函数，包括**Laplacian(),Sobel(),Scharr()**。这些滤波函数都会将非边缘区域转为黑色，将边缘区域转为白色或其他饱和的颜色。但是都很容易将噪声错误的识别为边缘，缓解这个问题的方法就是模糊处理，包括**blur()(简单的算术平均)，medianBlur()(中值平均)，高斯模糊**。都有一个**ksize**参数，它是一个奇数，表示滤波核的宽和高(像素为单位)。
## 用定制内核做卷积
核（卷积矩阵）的格式：卷积矩阵是一个二维数组，有**奇数行，奇数列**，中心原色对应与感兴趣像素，每个元素都有一个整数或浮点数的值，这些值就是权重。例如：
    
    kernel = numpy.array([-1, -1, -1], 
                         [-1, 9, -1],
                         [-1, -1, -1])
上面感兴趣元素权重为9，邻近元素权重为-1.对于感兴趣元素，新元素值就是当前元素值*9，再减去8个邻近元素值。如果感兴趣元素已经与邻近元素有一点差别，那么这个差别会增加，图像锐化。   
OpenCV提供了一个非常通用的filter2D()函数，它运用用户指定的任意核。  
    
    cv2.filter2D(src, -1, kernel, dat)
第二个参数制定了目标图像每个通道的位深度，如果为负值，则表示目标图像和源图像有同样的位深度。
>这里会对每个通道都是用相同的核
## Canny边缘检测
算法非常复杂，有五个步骤：高斯滤波去噪，计算梯度，边缘上使用非最大抑制(NMS)边缘上双阈值去除假阳性，分析所有边缘之间的连接，消除不明显的边缘
    
    cv2.Canny(img, threshold1, threshold2)
其中较大的阈值2用于检测图像中明显的边缘，但一般情况下检测的效果不会那么完美，边缘检测出来是断断续续的。所以这时候用较小的第一个阈值用于将这些间断的边缘连接起来。
## 轮廓检测
不单是检测图像或者视频帧中物体的轮廓，还与计算多边形边界，形状逼近和计算感兴趣区域有关。
    
    img = np.zeros((200, 200), dtype=np.uint8)
    img[50:150, 50:150] = 255  # 切片赋值

    ret, thresh = cv2.threshold(img, 127, 255, 0)  # 二值化
    image, contours, hierarchy = cv2.findContours(thresh, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
    color = cv2.cvtColor(img, cv2.COLOR_GRAY2BGR)
    img = cv2.drawContours(color, contours, -1, (0, 255, 0), 2)
    cv2.imshow("contours", color)
    cv2.waitKey()
    cv2.destroyAllWindows()
这段代码首先创建了一个200x200大小的黑色空白图像个，接着在图像中央放置一个白色方块。接下来对图像进行二值化操作(cv2.threshold),而后调用**findContours()**，该函数有三个参数：输入图像，层次类型和轮廓逼近方法。
- 这个函数会修改输入图像，故注意备份(通过img.copy()作为输入图像)
- 由函数返回的层次树相当重要：cv2.RETR_TREE参数会得到图像中轮廓的整体层次结构，以此建立轮廓之间的“关系”。如果只想得到最外面的轮廓，可使用cv2.RETR_EXTERNAL。这对消除包含在其他轮廓中的轮廓很有用。

其函数有三个返回值：修改后的图像，图像轮廓以及他们的层次。
## 边界框，最小矩形区域和最小闭圆的轮廓
## 凸轮廓和Douglas算法
凸形状：内部任意两点连线都在形状里面。   
**cv2.approxPloyDP**用来计算近似的多边形框。有三个参数：
- “轮廓“
- 原轮廓与近似多边形的最大差值
- 布尔标记，表示多边形是否闭合

获取轮廓的周长信息：L = cv2.arcLength(countour, True)
    
    epsilon = 0.01 * cv2.arcLength(cnt, True)
    approx = cv2.approxPolyDP(cnt, epsilon, True)
为了计算凸形状，可用cv2.convexHull(cnt)函数获取处理过的轮廓信息 
## 直线和圆检测
### 直线检测
可通过**HoughLines(标准)和HoughLinesP(概率)**完成，后者是前者的优化版本，有几个参数：
- 需要处理的图像
- 线段的几何表示rho和theta，一般分别取1和np.pi/180
- 阈值。低于阈值的直线会被忽略。
- 最小直线长度
- 最大线段间隙

### 圆检测
**HoughCircles**函数可用来检测圆，与直线一样，有一个最小圆心距和圆的最小及最大半径。