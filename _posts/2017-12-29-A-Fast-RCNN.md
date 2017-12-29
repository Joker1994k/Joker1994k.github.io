---
title: A-Fast-RCNN
mathjax: true
type: categories
categories: paper-note
---

### A-Fast-RCNN: Hard Positive Generation via Adversary for Object Detection 

[论文链接](cn.arxiv.org/pdf/1704.03414.pdf)

#### 整体结构

![a-fast-rcnn](https://raw.githubusercontent.com/Joker1994k/pic/master/A-fast-rcnn.PNG)  

+ fast rcnn的检测主干
+ Adversarial Spatial Dropout Netwrok（ASDN）用于对目标特征图做遮挡操作
+ Adversarial Spatial Transformer Network(ASTN)  用于对目标特征图上做变形操作

#### 关键思想

​	当遇到遮挡、角度不同或变形的图像时，目标检测识别的效果就会下降。而为了解决这个问题一般就使用覆盖条件范围更加大的数据，并且希望通过模型训练加强对物体检测识别的鲁棒性。但是数据集往往不能做到覆盖一个目标的各种状态，因此文中使用对抗网络来自我生成遮挡与变形的物体例子，从而让检测网络难以进行分类。随着检测网络的性能逐渐提升，对抗网络产生的图片质量也在提升。通过这种对抗策略，使得目标检测的准确性得到了进一步提升。

#### 主要实现

​	文中所提到的遮挡与变形都是在特征提取后的feature map上做的相关操作，因为作者发现在feature map上进行处理会更加高效同时也会更加有效。各个子网络都有单独训练的过程，然后再联合训练。

+ **Adversarial Spatial Dropout Netwrok**
  ​	初始化ASDN网络：通过滑动窗进行遮挡测试，生成新的特征图进行识别，选择使loss最大的滑动窗，用这个窗口生成二值掩码，并使用二值交叉熵损失训练ASDN，经过学习，网络可以可以学习特征图中的哪些部分对于分类比较重要，学习过程的直观表示如图所示：

  ![ASDN](https://raw.githubusercontent.com/Joker1994k/pic/master/ASDN.PNG)

  ​	联合训练：通过生成的二值掩码将特征图对应部分清零，修改后的特征图继续向前传播计算损失。通过这个过程生成了困难特征，用于训练检测网络，得到更优的结果，训练过程如下所示：

  ![ASDN-joint](https://raw.githubusercontent.com/Joker1994k/pic/master/ASDN-joint.PNG)

+ **Adversarial Spatial Transformer Network**

  ​	STN网络主要包括了定位网络（估计形变参数），网格生成器，采样器。而ASTN主要是为了旋转特征图使其更加难以被识别，训练方法与ASDN 类似，将能使检测网络将前景误判的变换作为最优的变换。它将特征图划分为四个block，每个block估计了四个方向的旋转，大大增加了任务的复杂度。还可以将两个对抗网络结合起来，使其得到更加鲁棒性的结果。

#### 总结

​	将对抗网络用在了目标检测的方向。根据文中的baseline来看，还是非常有效的，与Fast-RCNN比较，在VOC2007上，mAP增加了2.3%，VOC2012上增加了2.6%。与其他图像增广的方法比如图像旋转等做了对比，都会高接近1%个点。

​	这个方法与Online Hard Example Mining(OHEM)类似，都是对困难样本的挖掘利用，mAP上与OHEM相比，在VOC07上比OHEM高了0.5%个点，但在VOC12上少了0.8%个点。后来作者将两个网络进行ensemble之后，精度也得到了提高。总之，这也是目标检测的新的思路，现实的数据集不能覆盖物体的所有状态，检测器也就不能学习到物体的所有状态。那么在数据集有限的情况下就可以通过对抗网络在特征上面模仿遮挡或者形变的情况，从而改善检测网络的鲁棒性。