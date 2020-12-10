---
layout:     post
title:      "Dynamic RCNN"
subtitle:   " \"阅读笔记之Dynamic RCNN \""
date:       2020-12-01 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - object detection
    - rcnn
---

##### [Dynamic R-CNN: Towards High Quality Object Detection via Dynamic Training](https://arxiv.org/pdf/2004.06002)

> mask-rcnn benchmark: https://github.com/hkzhang95/DynamicRCNN 

> mmdetection: https://github.com/open-mmlab/mmdetection/tree/master/configs/dynamic_rcnn

##### Abstract

这篇文章首先指出固定的网络参数和动态训练过程之间的不一致性大大影响了网络性能，例如固定的标签分配策略（label assignment strategy）以及回归损失函数并不能够有效适应proposals的分布变化从而阻碍训练出高质量的检测器。作者提出一种新的动态标签分配标准（DLA）和动态smoothloss(DSL)使高质量的proposals能够充分训练检测器。

##### introduction

近年来，随着CNNs的快速发展，目标检测已经取得非凡的成就，现阶段流行的检测器主要分为单阶段和双阶段。而这些框架基本都是通过训练一个分类器和一个回归器来完成分类和定位任务。因此，一个有效的训练过程对发展高质量的目标检测器有着至关重要的作用。

与图像分类不同，目标检测中的类别标签是bbox的标注，因此在分类训练过程中，正负样本的界限相当模糊，所有很难清晰的将正负样本标签和proposals对应起来。目前，被广泛认可的方案是设置一个proposal和对应gt间IOU的阈值来进行判定。但正如CascadeRCNN中讨论的一样，单一的IOU阈值训练出来的分类器不能很好的适应其他IOUs，同时也不能直接提高IOU阈值，因为训练前期proposal质量总体偏低，导致缺少正样本，正负样本不平衡很容易引起过拟合问题，cascadeRCNN利用多级方式逐级提升proposalS的质量，效果的真的赞

对于回归器，随着训练的进行，proposals的质量逐渐提高，固定IOU分配出来的正样本越来越多，会使得其中高质量的proposal在计算loss的时候具有很小的贡献值（sample时基数中混入一些接近IOU阈值的低质量proposals），导致高质量的proposals没有得到充分训练

##### Dynamic Quality in the Training Procedure


目标检测是一个包含目标识别和定位的复杂任务。识别需要区分前景和背景以及确定目标的类别信息，另外定位任务就是精确找到目标bbox的位置。为了实现高质量的目标检测器，本文在训练过程进行了探索...

在fasterrcnn中，通过比较proposals和GT之间IOU和预定义的IOU阈值来判定proposals的类别，第一阶段通常正样本阈值0.7/负样本阈值0.3，第二阶段默认值都是0.5。由于分类器的目标是区分正负样本，所以IOU阈值与分类器质量直接相关。

经过实验验证，在训练过程中，proposals的质量是不断提高的，这启发式的产生一种策略，在刚开始训练的时候，使用较低的IoU阈值（这里的Iou只针对第二阶段，不涉及RPN中的判定），随着训练进行，不断提高IoU阈值，从而使得检测器的性能有所提升。、

这里采用动态更行IoU，确保充分训练高质量的proposals，自动调整smoothL1Loss的beta参数实现高质量proposals的loss权重自动提升

![avatar](../img/in-post/post-dynamic-rcnn-algorithm.png)

这里的跟新是每个C个迭代步长进行的，每次选取第topk大的IoU值加入保留集合Si，每过C个迭代，更新IoU_new = mean(Si); 每次计算预测值和GT的回归误差，选取第topk小的加入保留集合Sb,每过C个迭代，更新beta_new = median(Sb)

简直就是cascade的简化版，精度比不过cascade，只有一个预测头当然比cascade的速度快，topk这两个参数比较难选，特别对于不同的数据集

> ps还是cascade香









