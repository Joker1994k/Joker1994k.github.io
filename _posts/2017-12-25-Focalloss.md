---
title: Focal loss
mathjax: true
type: categories
categories: paper-note
---

### Focal Loss for Dense Object Detection 

[论文链接](http://cn.arxiv.org/pdf/1708.02002.pdf) 

#### 关键思想 

​	作者认为，one-stage (yolo，ssd等)目标检测网络的结果不如two-stage（R-CNN系列）网络的原因主要是由于极度不平衡的正负样本比例，同时大量的负样本大部分都是简单易分类的背景框。虽然这些简单样本的loss很低，但是由于数量大，对于loss的计算最终还是会有很明显的影响，使得模型的优化方向不是我们所希望的方向，从而导致收敛不到一个很好的结果。所以作者按照loss减少那些简单样本的权重，使训练倾向于更加有意义的样本中。

#### 主要实现 

##### 公式原理 

​	$FL(p_t)=-α_t(1-p_t)^γ log(p_t)$  

​	在这个公式中，$α_t$ 是为了应对数据不均衡的加权值。$p_t$ 是预选框对应正确类别的置信度， $γ$ 为大于等于0的参数，控制着难易样本对loss的贡献。**而当样本被正确分类的时候$p_t$ 是趋近于1的，也就是$p_t$越大说明越容易分类, 对应的$(1-p_t)^γ$越小, 对loss的贡献也就越小。**这种机制降低了大量的简单样本对于loss的最终影响，使得loss对于难以分类的样本更加专注。

​	文中$γ$ 的取值范围是在$[0,5]$ 之间进行的实验，不同取值的效果如图所示

​	![图1](https://github.com/Joker1994k/pic/blob/master/focalloss_params.PNG?raw=true)  

##### 实验框架

​	作者在实验的时候设计了一个叫做RetinaNet的one-stage网络，实际上是一个基于resnet的FPN网络。证明通过focalloss，one-stage的网络结构在能达到two-stage网络准确率的同时也能保持原本的检测速度。实验表明不同$γ$ 取值对loss的影响如下图所示，y轴是归一化的loss累计结果，x轴是样本比例。可以看到当$γ$ =2时，背景样本对loss的贡献是相当有限的，抑制效果明显，让模型在后期尽量学习那些困难样本，以得到更好检测结果。

 	![图2](https://raw.githubusercontent.com/Joker1994k/pic/master/focalloss_test_data.PNG)



#### 总结  

​	总的来说，这篇文章解决了SSD，yolo这类one-stage检测网络的精度落后于R-CNN系列的two-stage网络，并指出一步检测的效果差是因为在训练过程中负样本过多导致。最近做图片分类的比赛也加上了focalloss，使用论文中默认的$γ$ =2，相对于cross entropy loss来说，收敛速度更快，loss的值更低，在分类的准确度上最终会有0.2%左右的提高。看网上别人在其他网络上的测试，$γ$ 针对不同的数据集和网络调试之后，得到的结果可能会更好点，有时间可以测试下。

