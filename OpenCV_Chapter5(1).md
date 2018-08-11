# 人脸检测和识别
介绍人脸检测函数，定义了具体可跟踪对象类型的数据文件
## Haar级联的概念
提取出图像的细节对产生稳定分类结果和跟踪结果很有用。这些提取结果被称为特征，专业的表述为：从图像数据中提取特征。虽然任一像素都可能影响多个特征，但特征应该比像素数少得多。**两个图像的相似程度可以通过它们对应特征的欧式距离来测量**。   
例如，距离可能以空间坐标或颜色坐标来定义。类Haar特征是一种用于实现实时人脸跟踪的特征。每个类Haar特征都描述了相邻图像区域的对比模式。   
对给定的图像，特征可能因为区域大小而有所不同，**区域大小也可被称为窗口大小**。即使窗口大小不一样，尽在阿尺度上不同的两幅图像也应该有相似的特征。因此能为不同大小的窗口生成特征相当有用。**这些特征集合称为级联**。Haar级联具有**尺度不变性**，并可将其保存成指定的文件格式。不具有**旋转不变性**。
## 获取Haar级联数据
除了OpenCV3附带的外，有了强大的耐心以及强大的计算机，就可以创建自己的级联，并训练这些级联来检测各种对象。
## 使用OpenCV进行人脸检测
### 静态图像中的人脸检测
    
    import cv2

    filename = "timg.jpg"

    def detect(filename):
        face_cascade = cv2.CascadeClassifier('./cascades/haarcascade_frontalface_default.xml')

        img = cv2.imread(filename)
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        faces = face_cascade.detectMultiScale(gray, 1.3, 3)
        for (x, y, w, h) in faces:
            img = cv2.rectangle(img, (x, y), (x + w, y + h), (255, 127, 63), 2)
        cv2.namedWindow('sb')
        cv2.imshow('sb', img)
        cv2.waitKey(0)


    detect(filename)
face_cascade变量为CascadeClassifier对象，负责人脸检测，加载文件后转换为灰度图像，因为人脸检测需要这样的色彩空间。接下来是人脸检测函数
    
    faces = face_cascade.datectMultiScale(gray, scaleFactor, minNeighbors)
其中scaleFactor表示人脸检测过程中每次迭代时图像的压缩率，minNeighbors表示每个人脸矩形保留近邻数目的最小值。   
然后通过画矩形圈出人脸。
### 视频中的人脸检测
分析帧即可。
### 人脸识别
人脸识别其实就是一个程序能识别给定图像或视频中的人脸。实现这一目标的方法之一是用一系列分好类的图像来训练程序，并基于这些图像来进行识别。   
人脸识别模块的另一个重要特征是：每个识别都具有转置信评分，因此可在实际应用中通过对其设置阈值来进行筛选。