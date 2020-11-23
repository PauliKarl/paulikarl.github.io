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

记录舰船目标检测数据集生成的细节

###### 数据集中舰船目标的属性
ratios表示width/height的范围<----1/5----1/3----1/1----3/1----5/1----<

small_32表示目标长边小于32pixels

big_512表示目标长边大于512pixels

| Type | images | instances | small32 | big512 |                  ratios                |image size |
|------|--------|-----------|---------|--------|----------------------------------------|-----------|
|Airbus|42556   |81723      |36986    |0       |[203, 1622, 32805 , 42019 , 4527 , 547 ]|768x768----|
|DIOR  |2702    |62400      |18504    |352     |[207, 4506, 28419 , 25478 , 3638 , 152 ]|800x(800,788,802,801,813,816,787,783)|
|DOTA  |4708    |100383     |20231    |110     |[40 , 1408, 52827 , 45013 , 1040 , 55  ]|800x800    |
|HRSC  |2000    |5217       |0        |559     |[310, 499 , 1974  , 1679  , 554  , 201 ]|900x900    |
|LEVIR |1494    |3025       |537      |4       |[0  , 0   , 3025  , 0     , 0    , 0   ]|800x600    |
|MASATI|2368    |4054       |3564     |0       |[0  , 25  , 2552  , 1469  , 8    , 0   ]|512x512    |
|RS    |1162    |5752       |2        |0       |[217, 404 , 2116  , 2032  , 504  , 479 ]|900x900    |
|xView |429     |3943       |1833     |45      |[0  , 51  , 1900  , 1877  , 103  , 12  ]|900x900    |
|------|--------|-----------|---------|--------|----------------------------------------|-----------|
|total |57419   |266497     |81657    |1070    |[977, 8515, 125618, 119567, 10374, 1446]|all above  |


###### convert2coco
图像的尺寸目前不需要resize到统一尺寸，mmdet中选择keep_ratio=True，val_ratio=0.4


##### baseline

##### faster rcnn

|    backbone   |  FPN  | num_GPUs | per_img_gpu | learning_rate | epochs | mAP |
|---------------|-------|----------|-------------|---------------|--------|-----|
|   resnet50    |  yes  |    2     |      2      |      0.02     |   20   |-----|


