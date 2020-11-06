---
layout:     post
title:      "ship from xView"
subtitle:   " \"ship detection, convert2coco\""
date:       2020-11-06 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - detection
    - ship
    - dataset
---

> 参考师兄的工具集[wwtool](https://github.com/jwwangchn/wwtool.git)制作自己的转换代码

本文的目的在于记录从[xView](http://xviewdataset.org/)数据集中提取出舰船目标，并转换成coco数据格式用于mmdetection训练。

### 正文

首先，从60类标签见[xview_class_labels.txt](https://github.com/PauliKarl/pktool/blob/main/pktool/datasets/xview/xview_class_labels.txt)中挑选出舰船目标

` Maritime_label = {'Maritime Vessel', 'Motorboat', 'Sailboat', 'Tugboat', 'Barge', 'Fishing Vessel', 'Ferry', 'Yacht', 'Container Ship','Oil Tanker'} `


##### 提取的bbox标注格式转化成txt文件格式，见[geojson2txt.py](https://github.com/PauliKarl/pktool/blob/main/pktool/datasets/xview/geojson2txt.py)

bbox = [xmin, ymin, xmax, ymax]
存储的txt文件格式 *xmin,ymin,xmax,ymax,label*

##### 将xview图像切割到固定尺寸，例如800x800，代码[split_image.py](https://github.com/PauliKarl/pktool/blob/main/pktool/datasets/xview/split_image.py)

*设置原图路径和从geojson文件转的txt标注文件路径*

##### 按比例设置数据集训练集和验证集[generate_dataset.py](https://github.com/PauliKarl/pktool/blob/main/pktool/datasets/xview/generate_dataset.py)
```
if __name__=="__main__":
    origin_dataset_dir = '/data/pd/xview/origin'
    trainval_dir = '/data/pd/xview/v1/trainval'
    test_dir = '/data/pd/xview/v1/test'
    shuffle_dataset(origin_dataset_dir,trainval_dir,test_dir)
    #shuffle_dataset(origin_dataset_dir, trainval_dir, test_dir, trainval_rate=0.8, image_format='.png', label_format='.txt', seed=0)
``` 

##### txt转化为coco中json的格式即可[xview2coco.py](https://github.com/PauliKarl/pktool/blob/main/pktool/datasets/xview/xview2coco.py)
```
coco_annotation['segmentation']=[object_struct['segmentation']] #=pointobb,[polygon]格式，用于跑mask RCNN系列
coco_annotation['iscrowd'] = 0
```

至此，数据集制作完毕
```
--xview
    --origin
        --images
            1036_0_1800.png
            ...
        --labels
            1036_0_1800.txt
            ...
    --shiptxt
        1036.txt
        ...
    --v1
        --coco
            --annotations
                xview_trainval_v1.json
                xview_test_v1.json
        --test
            --images
                ...
            --labels
               ...
        --trainval
            --images
                ...
            --labels
                ...
```
