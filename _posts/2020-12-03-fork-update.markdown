---
layout:     post
title:      "github上fork后同步官方的更新"
subtitle:   " \"fork后保持官方的更新 \""
date:       2020-12-03 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - git
    - fork
---

> 拉取官方的更新同时保留自己的修改，采用bash操作

##### 先查看远程状态
> $ git remote -v

```
origin  https://github.com/PauliKarl/mmdetection.git (fetch)
origin  https://github.com/PauliKarl/mmdetection.git (push)
```
##### 添加fork上游仓库
> $ git remote add upstream https://github.com/open-mmlab/mmdetection.git

备注，删除命令：git remote remove upstream

##### 确认fork上游仓库添加成功
> $ git remote -v

```
origin  https://github.com/PauliKarl/mmdetection.git (fetch)
origin  https://github.com/PauliKarl/mmdetection.git (push)
upstream        https://github.com/open-mmlab/mmdetection.git (fetch)
upstream        https://github.com/open-mmlab/mmdetection.git (push)
```

##### 从上游仓库 fetch 分支和提交点，提交给本地 master，并会被存储在一个本地分支 upstream/master...

> $ git fetch upstream

```
remote: Enumerating objects: 301, done.
remote: Counting objects: 100% (301/301), done.
remote: Total 523 (delta 301), reused 301 (delta 301), pack-reused 222
Receiving objects: 100% (523/523), 186.98 KiB | 0 bytes/s, done.
Resolving deltas: 100% (381/381), completed with 210 local objects.
From https://github.com/open-mmlab/mmdetection
 * [new branch]      detr       -> upstream/detr
 * [new branch]      master     -> upstream/master
 * [new branch]      refactor_doc -> upstream/refactor_doc
 * [new branch]      yolov4     -> upstream/yolov4
 * [new tag]         v2.7.0     -> v2.7.0
```

##### 合并到本地仓库
> $ git merge upstream/master

如果报错就根据错误提示修改

##### 提交到个人的远程仓库

> $ git push

everything is ok!
