---
title: Data Distillation
mathjax: true
type: categories
categories: paper-note
---

### Data Distillation: Towards Omni-Supervised Learning

[论文链接](http://cn.arxiv.org/pdf/1712.04440)

#### 整体结构

![data distillation](https://raw.githubusercontent.com/Joker1994k/pic/master/data%20distillation.PNG)

#### 关键思想

​	将多种未标注图片的变换（数据增广等）结合，使用单个模型，自动地生成标签标注。然后将生成标签的图片与人工标注的图片混合训练，达到提升网络性能的目的。Data Distillation总的来说是一个通用的半监督学习方法，目标网络在不需要训练大量的模型的前提下，可以从未标注的数据中学习。

#### 主要实现

​	主要分为四步：

+ 在人工标注的数据集上训练一个模型

+ 用模型去预测没有标注信息的数据

  ​	包括了使用随机裁剪等方法提升网络性能的一般方法。最终作者使用多重变换的方法处理大量的未标注图像数据。

+ 聚合未标注数据的预测结果，并将结果转化为标签

  ​	将不同变换的未标注图像预测结果聚合，往往会在一个固定的变换下得到一个比这个模型的其他预测都要好的结果。作者观察到的是聚合不同的预测结果将时网络学习到新的特征，并且通过生成标签，原则上能使网络从自身学习到新的特征等。

  ​	考虑到可能产生的““soft label”（概率向量，不能直接在联合训练中使用）和输出空间结构（在一些特定任务中例如人体姿态检测中没有意义）等问题，作者在生成标签的时候通过“ a small amount of task specific logic that addresses the structure of the problem”（例如非极大值抑制）生成困难标签（结构、种类和人工标注的数据类似）。**作者发现当自动生成的数据标签数量与人工标注的标签数量相似的时候网络的鲁棒性更好，能使生成的图像数据更加接近人工实际标注的图像质量**。

+ 将人工标注数据与自动生成标签的数据混合训练

  ​	主要考虑两点：1）确保每个minibatch都包含了人工标注数据和自动生成标签的数据，能保证每个minibatch中都有一定真实标签的比例，使得最终的梯度预测更好。2）随着训练数据的增加，需要充分利用这些数据。（？）

#### 总结

​	作者在人体关键点检测和目标检测上分别做了实验。实验的效果上是非常好的，在不同的backbone上基本都有一个点以上的提升。文章展示了使用全部的监督数据和大量的未标注数据，能使omni-supervised训练具有超越监督训练的可能性。

​	但是这篇文章总体给人的感觉是，创新点比较少。最后在数据大量增加的前提下（比如35K到35K+80K的训练数据）表现出来的实验效果得到了巨大的提升。比较厉害的是实验完善，做了人体关键点和目标检测的实验，同时在试验中还做了backbone网络的对比，实验量很大，并在一些细节和实验安排上比较熟练，使用了不同的能用一般的方法达到很好的实验结果。