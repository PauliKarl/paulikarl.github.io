---
layout:     post
title:      "DETR"
subtitle:   " \"阅读笔记之DETR \""
date:       2020-11-25 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - object detection
    - end2end
    - transformer
---

> "[End-to-End Object Detection with Transformers](https://arxiv.org/abs/2005.12872)"

> [code](https://github.com/facebookresearch/detr)、[mmdetection/config/detr/](https://github.com/open-mmlab/mmdetection/tree/master/configs/detr)

##### Abstract

摒弃需要手工设置的anchor generation和NMS处理组分，采用全局 set loss + 二元匹配 + transformer encoder-decoder框架

##### Introducetion

目标检测任务本质是从图像中寻找一个包含所有目标表达（例如bbox）的集合，现在大部分的目标检测框架都是通过在预定义的集合（proposals、anchors、centers）上完成目标的定位回归和分类预测任务,他们的性能受制于一些后处理步骤（例如，去除重叠框、anchor集合设置、启发式的将GT
分配给anchor）...

为了简化检测框架，facebook AI提出一个崭新的基于transformer的DETR，由三个部分组成：backbone + transformer + FFN。

![avatar](../img/in-post/post-detr-framework.png)

detr采用标准的encoder-decoder结构，训练时把目标检测任务转化成所有目标的集合预测问题，而transformer中的self-attention机制能有有效解决重复目标（其他大部分通过NMS完成）；端到端训练使用 **二元匹配(bipartite matching)** 来计算预测集合和GT集合的loss, 预测和GT一一对应。

优点：新的目标检测框架 + 精度堪比faster rcnn

缺点：训练时间太长，小目标检测效果差/大目标检测效果好

##### Set Prediction

set prediction 中最重要的两个问题：a)使用一个set loss迫使预测和gt完成one to one的对应；b)能够生成预测集合并且建模集合间元素的关系；

set中elements之间的关系用于解决重复目标的问题，属于postprocessing-free，并且采用匈牙利算法（Hungarian algorithm）的完成一一对应匹配同时无关element在set中的位置，pred和gt配对

首先detr中固定预测集合的大小N（超参）选择大于一般图像中的目标个数即可；主要的训练困难是预测和GT相对应的分数（类别、位置、尺寸）。本文的loss能在预测集合和gt集合之间产生一个最优的二元匹配并且能优化bbox loss.

如果GT集合元素的个数不足，则用dummy补齐，并标记为no object

这里的loss选择GIoU+L1+Lcls

#####  Transformers and Parallel Decoding

transformer的self-attention机制于non-local net类似，拿出每个element同时根据所有elements的信息来自我更新

##### DETR model

detr的网络框架图如下所示
![avatar](../img/in-post/post-detr-overall.png)

CNN-based backbone + encoder/decoder transformer + feed forward network(FFN)

backbone可以采用其他检测器的特征提取网络

transformer 的encoder只能接受一维序列输入，所以先使用1x1卷积把backbone的输出features(CxHxW)降维到(dxHxW),然后再将二维特征拉成一维即(dxHW),(这里就是为了使用transformer强行降维，甚至会损失一定信息spatial information, 不知道能不能专门设计一个二维的transformer结构). 每个encoder都由一个self-attentio和FFN组成。

##### results

(没卡趁早退坑-_|) 本文使用16 V100训练300epochs，花了3天时间，imgs_per_gpu=4(batch_size=64).
