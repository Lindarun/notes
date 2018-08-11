# 目标检测与识别
[TOC]
## 目标检测与识别技术
目标检测是一个程序，它用来确定图像某个区域是否含有要识别的对象，对象识别是程序识别对象的能力。识别通常只处理已检测到对象的区域，人们总是会在有人脸图像的区域去识别人脸。   
有许多目标检测和识别的技术，本章会用到：
- 梯度直方图
- 图像金字塔
- 滑动窗口
这些算法是互补的，比如在梯度直方图中会使用滑动窗口技术。   
### HOG描述符
HOG是一个特征描述符，因此和SURF，SIFT，ORB属于同一类。HOG不是基于颜色值而是基于梯度来计算直方图的。具体原理参照书上108页。   
还有两个主要问题
#### 尺度问题
如果在比较过程中找不到一组相同的梯度，则检测就会失败。
##### 图像金字塔
图像金字塔是图像的多尺度表示，有助于解决不同尺度下的目标检测问题。
#### 位置问题
要检测的目标可额位于图像的任何地方，所以需要扫描图像的各个部分，以确保能找到感兴趣的区域。
##### 滑动窗口
滑动窗口是用于计算机视觉的一种技术，包括了图像中要移动部分（滑动窗口）的检查以及图像金字塔对各部分进行检测。   
通过扫描较大图像的较小区域来解决定位问题，进而在统一图像的不同尺度下重复扫描。这种技术需要将每幅图像分解成多个部分，然后去掉不太可能包含对象的部分并对剩余部分进行分类。
> 这样会有个*区域重叠*的问题：   
区域重叠是指在对图像执行人脸检测时使用滑动窗口。
每个窗口都会丢掉几个像素，意味着一个滑动窗口可以对同一张人脸的四个不同位置进行正匹配，此时对最高评分的图像感兴趣。
#### 菲最大抑制
这是一种与图像同一区域相关的所有结果进行抑制的技术。   
如何确定窗口的评分：需要一个分类系统来确定某一特征是否存在，并且对这种分类会有一个置信度评分，这里采用**支持向量机SVM**分类。
#### 支持向量机
SVM是一种算法，通过一个优化的超平面进行分类。是目标检测的重要检测部分（用来区分哪些是目标）。
### 检测人
HOGDescriptor函数可检测人

    # 确定某矩形是否完全在另一个矩形中
    def is_inside(o, i):
        ox, oy, ow, oh = o
        ix, iy, iw, ih = 0
        return ox > ix and oy > iy and ow + ox < ix + iw and oy + oh < iy + ih
    
    # 绘制方框框出监测到的人
    def draw_person(image, person):
        x, y, w, h = person
        cv2.rectangle(img, (x,y), (x + w, y + h), (0, 255, 255), 2)
        
    img = cv2.imread(filename)
    hog = cv2.HOGDescriptor()  # 指定检测人的默认检测器
    hog.setSVMDetector(cv2.HOGDescriptor_getDefaultPeopleDetector())
    
    # 加载图像（此处不需要转化为灰度）
    found, w = hog.detectMultiScale(img)
    
    # 上述检测方法将返回一个与矩形相关的数组，但如果直接绘制会发现某些矩形会完全包含在其他矩形中。说明检测出现了错误，应该丢弃。
    found_filtered = []
    for ro, r in enumerate(found):
        for qi, q in enumerate(found):
            if ri != qi and is_inside(r, q):
                break
            else:
                found_filtered.append(r)
    
    for person in found_filtered:
        draw_person(img, person)
        
    cv2.imshou("file", img)
### 创建和训练目标检测器
只是现任检测和人脸检测肯定不够。还需要了解如何得到用于人脸检测器的特征，并且还要能改进这些特征，还要搞明白这些概念是否用于其他类型的目标检测。   
如何构建分类器呢：SVM和**词袋技术**
#### 词袋BOW
通常用来在一系列文档中计算每个词出现的次数，然后用其构成的向量来重新表示文档。而在计算机视觉中，BOW实现步骤如下：
- 取一个样本数据集
- 对数据集每幅图像提取描述符
- 将每一个描述符都加入到BOW训练器中
- 将描述符聚类到k簇中（其中心就是视觉单词）
这个过程需要提供视觉单词字典。
   
   
先简单介绍K-means聚类，对充分理解如何创建视觉单词有帮助，更好地理解基于BOW和SVM的目标检测。
##### K-means聚类
用于数据分析的向量量化算法。对于给定的数据集，k表示要分割的数据集簇数。簇的均值其实就是这个簇中点的几何中心。
## 汽车检测
### 代码的功能
真的涉及了好多知识。   
创建两个SIFT实例：一个提取关键点，一个提取特征

    detect = cv2.xfeatures2d.SIFT_create()
    extract = cv2.xfeatures2d.SIFT_create()
    
看到SIFT时，就可断定一定会涉及一些特征匹配算法。基于FLANN的：

    
    flann_params = dict(algorithm - 1, trees = 5)
    flann = cv2.FlannBasedMatcher(flann_params, {})
    
创建BOW训练器，簇数指定为40
    
    bow_kmeans_trainer = cv2.BOWKmeansTrainer(40)
初始化BOW提取器。视觉词汇将作为BOW类的输入，在测试图像中会检测这些视觉词汇：
    
    extract_bow = cv2.BOWImgDescriptorExtractor(extract, flann)
为了从图像中提取SIFT，先获取图像路径，并以灰度格式读取，然后返回描述符：
    
    def extract)sift(fn):
        im = cv2.imread(fn, 0)
        return extract.compute(im, detect.detect(im))[1]