---
layout:     post
title:      "create ship dataset from others"
subtitle:   " \"generate dataset, convert2coco\""
date:       2020-11-23 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - detection
    - ship
    - dataset
---

> 包含舰船目标的光学遥感数据集，[DOTA](https://captain-whu.github.io/DOTA/index.html)、[HRSC2016](http://www.escience.cn/people/liuzikun/DataSet.html)、[LEVIR](http://levir.buaa.edu.cn/Code.htm)、[NWPU VHR-10](https://pan.baidu.com/s/1hqwzXeG#list/path=%2F)、[DIOR](http://www.escience.cn/people/gongcheng/DIOR.html)、Airbus、[MASATI](https://www.iuii.ua.es/datasets/masati/)、[xView](http://xviewdataset.org/)

记录舰船目标检测数据集生成和baseline

###### 数据集中舰船目标的属性

coco中的目标大小分类

|  Type  | min rectangle area | max rectangle area |
|--------|--------------------|--------------------|
| small  |        0x0         |       32x32        | 
| medium |       32x32        |       96x96        |
| large  |       96x96        |      infxinf       |

目标类别，正框/单类

ratios表示width/height的范围<----1/5----1/3----1/1----3/1----5/1----<

| Type | images | instances |  small  | large  |                  ratios                |image size |
|------|--------|-----------|---------|--------|----------------------------------------|-----------|
|Airbus|42556   |81723      |42756    |9908    |[203, 1622, 32805 , 42019 , 4527 , 547 ]|768x768----|
|DIOR  |2702    |62400      |39166    |1810    |[207, 4506, 28419 , 25478 , 3638 , 152 ]|800x(800,788,802,801,813,816,787,783)|
|DOTA  |4708    |100383     |47424    |2622    |[40 , 1408, 52827 , 45013 , 1040 , 55  ]|800x800    |
|HRSC  |2000    |5217       |0        |4734    |[310, 499 , 1974  , 1679  , 554  , 201 ]|900x900    |
|LEVIR |1494    |3025       |537      |1067    |[0  , 0   , 3025  , 0     , 0    , 0   ]|800x600    |
|MASATI|2368    |4054       |3797     |1       |[0  , 25  , 2552  , 1469  , 8    , 0   ]|512x512    |
|RS    |1162    |5752       |24       |3447    |[217, 404 , 2116  , 2032  , 504  , 479 ]|900x900    |
|xView |429     |3943       |2248     |465     |[0  , 51  , 1900  , 1877  , 103  , 12  ]|900x900    |
|------|--------|-----------|---------|--------|----------------------------------------|-----------|
|total |57419   |266497     |135952   |24054   |[977, 8515, 125618, 119567, 10374, 1446]|all above  |
|------|--------|-----------|---------|--------|----------------------------------------|-----------|
|val   |23113   |105793     |55300    |9509    |medium_40984

###### convert2coco
图像的尺寸不resize到统一尺寸，val_ratio=0.4


##### baseline

TITAN XP 12GB, linux



##### two-stage
###### tricks：softnms、rFPN、albu、multi-scale test、

batch_size=num_GPUs x per_img_gpu

|           Type            | backbone |  FPN  | batch_size | lr | epochs | mAP50 | mAP75 |  mAP  | mAP_s | mAP_m | mAP_L |
|---------------------------|----------|-------|------------|----|--------|-------|-------|-------|-------|-------|-------|
|faster rcnn                |   R50    |  yes  |    2x2     |0.02|   20   | 0.763 | 0.545 | 0.487 | 0.340 | 0.608 | 0.727 |
|faster rcnn +soft-nms      |   R50    |  yes  |    2x2     |0.02|   20   | 0.767 |-------| 0.495 | 0.347 | 0.621 | 0.741 |
|faster rcnn                |   R101   |  yes  |    2x2     |0.02|   20   | 0.763 |-------| 0.487 | 0.337 | 0.612 | 0.737 |
|faster rcnn +soft-nms      |   R101   |  yes  |    2x2     |0.02|   20   | 0.767 |-------| 0.497 | 0.343 | 0.625 | 0.752 |
|faster rcnn                |   X101   |  yes  |    2x2     |0.02|   12   | 0.765 |-------| 0.490 | 0.340 | 0.615 | 0.735 |
|faster rcnn +soft-nms      |   X101   |  yes  |    2x2     |0.02|   12   | 0.770 |-------| 0.500 | 0.347 | 0.629 | 0.750 |
|cascade rcnn               |   R50    |  yes  |    2x2     |0.02|   12   | 0.772 | 0.575 | 0.511 | 0.355 | 0.643 | 0.761 |
|cascade rcnn +soft-nms     |   R50    |  yes  |    2x2     |0.02|   12   | 0.777 |-------| 0.517 | 0.360 | 0.648 | 0.773 |
|cascade rcnn               |   R101   |  yes  |    2x2     |0.02|   12   | 0.772 |-------| 0.509 | 0.352 | 0.641 | 0.771 |
|cascade rcnn +soft-nms     |   R101   |  yes  |    2x2     |0.02|   12   | 0.776 |-------| 0.515 | 0.357 | 0.646 | 0.779 |
|cascade rcnn               |   X101   |  yes  |    2x2     |0.02|   12   | 0.775 |-------| 0.517 | 0.357 | 0.647 | 0.775 |
|cascade rcnn +soft-nms     |   x101   |  yes  |    2x2     |0.02|   12   | 0.781 |-------| 0.523 | 0.363 | 0.654 | 0.783 |
|Dynamic rcnn               |   R50    |  yes  |    2x2     |0.02|   12   | 0.765 | 0.547 | 0.489 | 0.343 | 0.607 | 0.721 |
|Dynamic rcnn +soft-nms     |   R50    |  yes  |    2x2     |0.02|   12   | 0.764 |-------| 0.495 | 0.348 | 0.615 | 0.734 |
|libra rcnn                 |   R50    |  yes  |    2x2     |0.02|   12   | 0.755 | 0.538 | 0.478 | 0.334 | 0.601 | 0.721 |

##### single-stage

|           Type            | backbone |  FPN  | batch_size | lr | epochs | mAP50 | mAP75 |  mAP  | mAP_s | mAP_m | mAP_L |
|---------------------------|----------|-------|------------|----|--------|-------|-------|-------|-------|-------|-------|
|fcos                       |   R50    |  yes  |    4x2     |5e-3|   12   | 0.782 | 0.524 | 0.482 | 0.333 | 0.608 | 0.716 |
|fcos                       |   R101   |  yes  |    4x1     |5e-3|   12   | 0.776 | 0.516 | 0.477 | 0.325 | 0.601 | 0.731 |


##### detr


