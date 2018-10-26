---
title: MegDet
mathjax: true
type: categories
categories: paper-note
---

### MegDet: A Large Mini-Batch Object Detector

[论文链接](http://cn.arxiv.org/pdf/1711.07240.pdf)

#### 关键思想

​	作者发现最近的检测网络，主要是从网络框架，loss的设计上进行改进，但是在mini-batch size上的学习研究非常少，所以他们就提出了一个Large Mini-batch Object Detector(MegDet)网络，通过学习率策略和新提出的Cross-GPU Batch Normalization，能够显著的提高batch size 的大小（文中提到能从16提升到256），缩短训练时间，同时在检测精度上也能得到一定的提升。

#### 主要实现

​	目前的目标检测算法，mini-batch size都设的比较小，例如R-CNN的mini-batch size 就只有2。作者认为，过小的mini-batch 不能为BN层提供精确的统计数据，可能会导致正负样本不均衡的问题，从而影响到最后的检测精度。而如果单纯的加大mini-batch size 就需要相应的提高学习率，但是大的学习率往往容易引起网络的不收敛，如果继续使用小的学习率训练，最终的学习效果就会不好。为了解决这个矛盾，作者提出了Cross-GPU Batch Normalization，将不同GPU中BN的统计信息结合以得到更加精确的结果，结构如下所示：

![MegDet](https://raw.githubusercontent.com/Joker1994k/pic/master/MegDet.PNG)

​	输入为x，在不同的设备(GPU)上得到的mini-batch $B=\bigcup_{i=1}^{n}{B_i}$ ,$B_i=\{x_{i_1 \cdots i_n}\}$ 。就是在每次运算结束后将结果统计得到期望方差，再根据公式$y=\gamma \frac{x-\mu B}{\sqrt{\sigma^2_B+\epsilon}}+\beta$ 计算其输出。

​	而最终的实验结果也证明了在一定的范围内batch-size越大，得到的实验结果也会相应的得到一定的提升：

![MegDet_table](https://raw.githubusercontent.com/Joker1994k/pic/master/MegDet_table.PNG)

#### 总结

​	这是旷世在2017年的COCO目标检测比赛后衍生的论文，他们在基于MegDet 的基础上，同时也集成了OHEM，atrous convolution，更强的base models，large kernel,segmentation supervision,diverse network structure,contextual modules,ROIAlign,multi-scale training and testing等各种其他方法，最终取得到了目标检测上的第一名。

​	最开始是以为有特殊的优化方法，可以在单个gpu上将batch-size提高，最终发现还是在多个gpu上做的。目前没有开源，听过这个论文作者的讲解直播，貌似说过因为是公司的东西，所以开源的可能性很低。总的来说，有一定的启发价值，但是基于现在的设备限制，对于后续工作的参考价值不是很高。