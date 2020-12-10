---
layout:     post
title:      "Crowded Scenes"
subtitle:   " \"阅读笔记之Crowded detection \""
date:       2020-11-27 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - object detection
    - Crowded Scenes
---



#### [Detection in Crowded Scenes: One Proposal, Multiple Predictions](https://arxiv.org/abs/2003.09163)

> [code](https://github.com/megvii-model/CrowdDetection)

##### Abstract

为了解决密集场景下具有高重叠区域的目标检测问题，旷世在cpvr2020 oral中展示了该工作，即用单个proposal产生多个预测候选替换了之前的一一对应规则，同时增加相应的EMD Loss + set NMS来提升网络性能，使之在密集场景检测任务中达到SOTA水平。

##### Introduction

proposal-based 方法已经被广泛的使用在一阶段/二阶段等现代目标检测器中，这个机制主要包括两个部分：一是手动设计完全包含目标的proposals或者像RPNs一样采用可学习的策略；二是每个proposal都预测唯一一个实例并且给出置信度分数和目标再定位。采取这种方式必须使用NMS的后处理来删去重叠目标。

虽然proposal-based的方法在coco和voc上都取得了很好的效果，但是它们在密集场景的检测效果却不尽如人意，尤其是具有高重叠区域的目标。造成这种困难的因素有两个：第一，由于高重叠区域（proposals也是如此）的实例特征非常相似使得检测器很难区分；第二，NMS可能会错误抑制具有高重叠区域的预测结果。

之前是如何解决这个问题的呢？ sophisticated NMS/re-scoring/part-based detectors/new loss,（不能从根本上解决问题/效果一般？）

因此，本文提出解决该问题的新框架：对于每个proposal，产生一个可能是高重叠区域的预测集合，同时希望把靠近的proposals预测为同一个set而不是单独区分开，才很容易就能办到。另外，在新框架中还应用了几种其他技术，EMD用来监督实例预测集合的学习过程；使用set NMS而不是NMS来抑制来自不同proposals的重复预测；最后，一个可选的refinement module(RM)用来处理潜在的虚警。

该方法十分简单且几乎不增加复杂度，可以用在所有proposal-based的框架中（例如retina、fasterRCNN、MaskRCNN、FPN）

##### Background

正如前文所说，proposal-based方法包括proposals生成和实例预测两个部分，本文主要研究和改进第二个部分。在做实例预测时，流行的检测框架使用一个目标函数来分配proposals（判定每个proposal是否属于某一个GT实例），如果是则进一步预测类别标签和位置修正。这个机制确定了每个proposal只能匹配一个gt（与proposal重叠区域最大的gt），因此，为了确保每个实例都有机会被检测到，proposals必须能够完全覆盖gt，预测必定出现很多重复，NMS就是用于出去重复的方法。

但是，NMS会造成密集目标检测的漏检，也有研究者试图克服这个困难

Advanced NMS：soft-NMS, softer-NMS

Loss functions for crowded detection: Aggregation Loss, Repulsion Loss

Re-scoring: 在一些框架中如fasterRCNN，proposals和gts之间存在many to one的关系，只要proposal和gt的iou高于阈值，最后用nms出去重复目标。相反，如果我们重新设计一个鼓励one to one的loss，那么就可以不用nms处理以达到减少漏检（由nms造成的漏检）。YOLO和YOLO9000在训练中gt和proposal就是严格遵守one to one，但是由于他们中缺少proposals之间的联系，proposal不能确定其所预测的目标是否已经被其他proposal预测，预测结果往往很模糊，事实上，NMS在YOLO和YOLO9000中依然被使用，RelationNet通过直接对proposals的关系建模来克服YOLO的限制，使用re-scoring的RelationNet在coco上取得很好的性能甚至不需要NMS，但是在CrowdHuman数据集上表现很差。。

##### Multiple Instance Predictionj

Instance set prediction: 以前的预测对（label,coordinates）,扩展为集合{（label_i,coordinates_i）...}, 集合大小预定义K

EMD loss: 不足K个补dummy bbox (label为背景，没有回归损失)

set NMS: 重复的实例来自不同的proposal预测集合，因此nms处理之前检查被抑制和保留的是否来自同一集合，如果是则放弃NMS处理

Refinement module： 每个proposal会产生一个预测集合，这样会造成虚警，可以对预测结果再次经过RM修正

![avatar](../img/in-post/post-crowded-det-architecture.png)

##### discuss

多实例预测并不是新方法，一些不是proposal-based，或者像YOLOv1/v2在某一个位置预测所有的实例中心；特别是loss(这部分看起来是被审稿人怼了才加的~_~)

![avatar](../img/in-post/post-crowded-det-discuss.png)


#### [NMS by Representative Region: Towards Crowded Pedestrian Detection by Proposal Pairing](https://arxiv.org/pdf/2003.12729)

> no code

