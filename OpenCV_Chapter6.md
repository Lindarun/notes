# 图像检索以及基于图像描述符的搜索
[TOC]
OpenCV可以监测图像的主要特征，然后提取这些形成**图像描述符**，这些图像特征可以作为图像搜索的数据库。人们还可以利用关键点将图像拼接起来，组成一个更大的图像（如全景）。   
将介绍如何通过OpenCV来检测图像特征，并利用这些进行图像匹配和搜索。通过*单应性*（从一个平面到另一个平面的投影映射）来检测这些图像是否存在于另一个图像中。
## 特征检测算法
> 角点：有具体定义的、或者是能够具体检测出来的兴趣点   
  斑点检测：指在数字图像中找出和周围区域特性不同（很大差别）的区域，这些特性包括光照或颜色等。

最常用的特征检测算法和提取算法有：
- Harris：检测角点
- SIFT：检测斑点
- SURF：检测斑点
- FAST：检测角点
- BRIEF：检测斑点
- ORB：代表带方向的FAST与具有旋转不变性的BRIEF   
特征匹配方法：
- 暴力
- 基于FLANN的匹配法
### 特征定义
特征就是有意义的图像区域，该区域具有独特性或易于识别性，因此角点及高密度区域是很好的特征。边缘也可以，斑点也是有意义的特征。   
有一些涉及脊向的概念，可以认为脊向是细长物体的对称轴（识别图像的路）
#### 检测角点的特征
方便实用的检测图像角点的函数
    
    dat = cv2.cornerHarris(gray, 2, 23, 0.04)
- img- 数据类型为 float32 的输入图像。(gray = np.float32(gray))
- blockSize - 角点检测中要考虑的领域大小。
- ksize - Sobel 求导中使用的窗口大小.该参数限定了Sobel算子的中孔。简单来说就是定义了角度检测的敏感度，其取值必须是介于3和31之间的奇数，越高越不灵敏。
- k - Harris 角点检测方程中的自由参数,取值参数为 [0,04,0.06]
再将角点标记为红色
    
    img[dat> 0.01 * dat.max()] = [0, 0, 255]
调整cornerHarris第二个参数值可以改变标记的红点大小，即为角点大小。参数值越小记号越小。
### 使用DoG和SIFT进行特征提取与描述
运用上面的方法不能避免放大缩小图像时对角点检测的影响，尺度不变特征变换（SIFT）可以解决这个问题，SIFT并不检测关键点（关键点可以由DoG检测），但SIFT可以通过一个特征向量来描述关键点周围区域的情况。   
DoG是对同一图像使用不同高斯滤波器所得到的结果，最终结果会得到感兴趣的区域，   
应用：
    
    sift = cv2.xfeatures2d.SIFT_create()
    keypoints, descriptor = sift.detectAndCompute(gray, None)
返回值是关键点信息和描述符
绘制关键点
    
    img = cv2.drawKeypoints(image=img, outImage=img, keypoints = keypoints, flags = cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINT, color = (51, 163, 236))
改代码对图像每个关键点都绘制了圆圈和方向
#### 关键点剖析
关键点有如下属性定义：
- pt（点）属性表示图像中的关键点的x坐标和y坐标
- size表示特征的直径
- angle表示特征的方向
- response表示关键点的强度，有一些特征会通过SIFT分类，因为它得到的特征比其他特征更好，通过这个属性可以评估特征强度
- octave表示特征所在金字塔的层级，表示检测到的关键点所在的层级
- class_id表示关键点的ID
### 使用快速Hessian算法和SURF来提取和检测特征
SURF特征检测算法比SIFT快好几倍，它吸收了SIFT算法思想
    
    surf = cv2.xfeatures2d.SURF_create(img)
所采用的Hessian阈值为8000，阈值越高，能识别的特征就越少，可以使用试探法来得到最优预测。
### 基于ORB的特征检测和特征匹配
#### FAST
FAST算法会在像素周围画一个圈，然后会将每个像素与加上一个阈值的圆心像素进行比较，若有连续、比加上一个阈值的圆心像素还亮或暗的像素，则可认为圆心是角点
#### BRIEF
BRIEF只是一个**描述符**，描述符是图像的一种表示，可以作为特征匹配的一种方法。BRIEF是目前最快的描述符。
#### 暴力匹配
第一个描述符的所有特征都用来和第二个进行比较，OpenCV专门提供了BFMatcher对象来实现暴力匹配。
### ORB特征匹配
上节所学的FAST和BRIEF则是ORB的基础。ORB旨在**优化和加快操作速度**，包括非常重要的一步：**旋转感知**的方式使用BRIEF，这样即使在训练图像与查询图像之间旋转差别很大的情况下也能够提高匹配效果。具体代码如下   
创建ORB特征检测器和描述符
    
    orb = cv2.ORB_create()
    kp1, des1 = orb.detectAndCompute(img1,None)
    kp2, des2 = orb.detectAndCompute(img2,None)
暴力匹配BFMatcher实现了匹配

    bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
    matches = bf.match(des1,des2)
    matches = sorted(matches, key = lambda x:x.distance)
用matplotlib绘制匹配
    
    img3 = cv2.drawMatches(img1,kp1,img2,kp2, matches[:40], img2,flags=2)
    plt.imshow(img3)
    plt.show()
### kNN匹配
KNN是计算成本较低的一种算法，注意match返回最佳匹配，knnMatch返回k个匹配   
上面程序只需要更换一部分
    
    matches = bf.knnMatch(des1,des2, k=2)
    img3 = cv2.drawMatchesKnn(img1,kp1,img2,kp2, matches, img2,flags=2)
### FLANN匹配
近似最近邻的快速库（FLANN），其具有一种内部机制，该机制可以根据数据本身选择最合适的算法来处理数据集。经验证，FLANN比其他的最近邻搜索软件快10倍。具体请看97页。  
    
    FLANN_INDEX_KDTREE = 0
    indexParams = dict(algorithm = FLAMM_INDEX_KDTREE, trees = 5)
    searchParams = dict(checks=50)
    flann = cv2.FlannBasedMatcher(indexParams,searchParams)
    matches = flann.knnMatch(des1,des2, k=2)
FLANN匹配器有两个参数。这两个在python以字典形式进行参数传递，为了计算匹配，FLANN内部会决定如何处理索引和搜索对象。这种情况下可以选择KTreeIndex，这个的配置索引很简单，只需要配置处理核密度树的数量，最理想在1~16之间，并且KTreeIndex非常灵活（可并行处理）。searchParams字典只包含一个字段，用来指定索引树要遍历的次数，越高时间越长精确度越高。   
匹配效果很大程度上取决于输入，5kd-treess和50checks总能取得具有合理精度的效果，而且很短时间内就能完成、
### FLANN的单应性匹配
单应性是一个条件：该条件表明当两幅图像的一个出现投影畸变时，他们还能彼此匹配。   
在前面的实例加入：
    
    # 保存所有按照Lowe‘s比率试验得出的优解
    good = []
    for m, n in matches:
        if m.distance < 0.7*n.distance:
            good.append(m)
            
    # 核心
    if len(good) > MIN_MATCH_COUNT:  # 要确保至少有一定数目的良好匹配（计算单应性至少四个）
        src_pts = np.float32([kp1[m.queryIdx].pt for m in good]).reshape(-1, 1, 2)  # 在原始图像和训练图像中发现关键点
        dst_pts = np.float32([kp2[m.trainIdx].pt for m in good]).reshape(-1, 1, 2)
        M, mask = cv2.findHomography(src_pts, dst_pts, cv2.RANSAC, 5.0)  # 单应性
        matchesMask = mask.ravel().tolist()  # 最后用来绘制图
        #  对第二张图像计算投影畸变，并绘制边框
        h, w = img1.shape
        pts = np.float32([0,0], [0,h-1], [w-1,h-1], [w-1,0]).reshape(-1,1,2)
        dat = cv2.perspectiveTransform(pts,M)
        img2 = cv2.polylines(img2, [np.int32(dst)], True, 255, 3, cv2.LINE_AA)
### 基于纹身取证的应用程序实例
