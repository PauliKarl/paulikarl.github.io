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

**dense-to-sparse method**:  主流的two-stage框架，例如RCNN系列、最近基于transformer的detr（one-stage）等