---
layout:     post
title:      "sparse rcnn"
subtitle:   " \"阅读笔记之sparse rcnn \""
date:       2020-12-08 18:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - object detection
    - end2end
    - sparse
---

> [Sparse R-CNN: End-to-End Object Detection with Learnable Proposals](https://arxiv.org/pdf/2011.12450)

> [code](https://github.com/PeizeSun/SparseR-CNN),implemented by detectron2 and DETR

##### abstract

抛弃dense object candidates方式，采用完全sparse的集合预测方式实现目标检测任务，没有many to one的对应关系，不需要采用nms的后处理过程。相比于DETR，收敛速度更快，精度更高

##### introduction

![avatar](../img/in-post/post-sparse-rcnn-comparision.png)
dense目标检测虽然取得很大的成功但是具有明显的三个限制：1)这样的密集预测通常会产生大量冗余甚至重复的目标，从而像NMS这样的后处理过程必不可少。2)many to one 的分配问题会在训练过程中让网络对这种启发式的分配规则非常敏感。3)最终性能受预定义的proposals影响较大，例如anchors的size、aspect、number, 参考点的密度和proposals的生成算法。

那么，是否可能设计一个sparse的检测器呢？

detr是一个利用sparse的集合预测的方法，它根据输入的100个可以学习的目标序列，直接在此基础上产生预测结果，不需要后处理过程。尽管detr简单且非常棒，但是detr每个目标查询元素都需要和image全局进行交互，这样的处理不仅让它训练收敛变慢，也让它无法彻底变成一个sparse的检测框架。

sparse的属性主要体现在两个方面，sparse boxes 和 sparse features。sparse boxes 意味着少量boxes就足够预测图像中的所有目标（e.g. 100）。 sparse features意味着每个的特征不需要和image中的所有features做运算。detr并不是一个完全稀疏的检测方法，它是dense-feat的

本文提出一个完全的sparse方法，设置一个固定的可学习的候选集小集合。这里提出proposal feature（高维向量e.g. 256）用于编码实例特性以及和粗boxes比较。特别的，proposal feat会为它专用的目标识别head产生一系列的自定义参数，称这个操作为Dynamic Instance Interactive Head.与两个权值共享的fc相比，这个head更加灵活和精度更高。

sparse RCNN最特别的地方在于它一直保持sparse-in和sparse-out。最初的输入是一个稀疏的proposal boxes和proposal feats集合，既没有dense-candidates也没有dense feat。比detr收敛更快，再coco上的精度更高。

##### Related work

**dense method** :主流的one stage框架，例如YOLO、SSD、RetinaNet等。

**dense-to-sparse method** :主流的two-stage框架，例如RCNN系列、最近基于transformer的detr（one-stage）等.

**sparse** :从G-CNN发展出来的算法、deformable-DETR等

##### Sparse RCNN

![avatar](../img/in-post/post-sparse-rcnn-overview.png)
一共三个输入：an image、proposal boxes集合、proposal feats。后两个是可学习的并且可以一起使用网络参数优化。

**backbone** 采用带FPN的ResNet。构建p2-p5多尺度特征，所有levels特征都是256 channels。

**learnable proposal box** 设置一个可学习的proposal boxes集合（Nx4）而不是从RPN产生，每个box使用4个范围0~1的参数表示（归一化的中心点坐标cx/cy+w+h）。训练时可以通过反向传播更新box参数，初始化的设置对实验影响很小。

从概念上讲，这些可以学习的proposal boxes是训练集中潜在的目标位置的统计信息，可以看作是最有可能是图像中包含目标区域的初始猜想，不管输入如何。

**learnable proposal feat** 虽然具有4个参数的box可以清晰提供目标的位置信息，但是它会丢失目标姿态和形状等信息，所以引入proposal feature(Nxd), 它是一个高维向量（e.g. d=256），用来表达实例的特征信息

**Dynamic instance interactive head** 根据给定的N个proposal boxes, 使用RoIAlign提取每个box的特征，并以此来生成最后的预测结果。
![avatar](../img/in-post/post-sparse-rcnn-head.png)
每个RoI feat输入到对应的head来预测目标类别和位置。proposal feats和 proposal boxes是严格一一对应的。每个RoI特征feati(SxSxC)和应对的proposal feat P(C)去滤除无效的bins最后输出目标特征feature(C),最后的回归预测结果是通过具有C隐藏层维度带ReLU激活函数的三层感知网络得到，类别预测使用单个线性感知层得到。

proposal feature能被看作是一种attention机制的实现，用于关注SxS的RoI。proposal feat产生卷积的核参数，然后RoI feat 被产生的卷积处理获取最终的feat,这样，大部分前景信息就能作用与目标的分类和定位。（感觉proposal feat只是用来跟新参数的必要输入，所以初始化影响不打的样子...）

与detr的区别：object query 有些类似于proposal feat，但是oq是被学习的位置编码。而且它在于oq互动的时候采用的是一维编码，这样会造成空间信息的大量损失，本文方法不需要位置编码

**set prediction loss** 由于sparseRCNN是集合预测，基于集合的loss会在gts和preds之间产生一个最优的二元匹配。matching loss：Lcls+L1+Lgiou

RCNN家族被many-to-one的分配方式困惑已久...

