---
title: mmpretrain实战
date: 2023-06-07 16:04:01
tags: [image classification,mmlab]
---

# mmpretrain实战

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/1686128130571mmpretrain.png)

主要讲解了安装,还有使用教程.安装教程直接参考官网.下面讲解一下mmpretrain使用



## 实战教程

### 2.1简单使用

我们可以直接从定义好的模型来进行推理,首先list_model可以列出所有的分类,然后通过关键字可以识别出来resnet所有的模型,然后我们通过get_model,输入关键字就可以得到模型,之后,我们通过使用inference来进行传入模型,还有ckp,还有图形就可以直接来进行推理.



### 2.2自定义使用

首先整个mmlab都是通过使用cfg来进行配置的,所以我们如果要进行自己的resnet50配置,我们可以从官网的cfg来进行参考.

首先是模型,模型分为backbone骨干网络,head就是输出头,使用neck来进行连接网络.然后最后的loss,实在模型里就定义号了,使用的是topk

```python
# model settings
model = dict(
    type='ImageClassifier',
    backbone=dict(
        type='ResNet',
        depth=50,
        num_stages=4,
        out_indices=(3, ),
        style='pytorch'),
    neck=dict(type='GlobalAveragePooling'),
    head=dict(
        type='LinearClsHead',
        num_classes=33,
        in_channels=2048,
        loss=dict(type='CrossEntropyLoss', loss_weight=1.0),
        topk=(1, 5),
    ),
    init_cfg = dict(type='Pretrained',checkpoint = 'https://download.openmmlab.com/mmclassification/v0/resnet/resnet50_8xb32_in1k_20210831-ea4938fc.pth')
    )
```

之后就是dataset的配置,我们使用的type是自定义的type,设置输入的train,还有val路径,之后设置val的评估指标,使用top1.



下面就是训练时候的配置,循环次数,还有优化器



最后就是训练时候的配置,自动保存权重最高的,还有值保留最近5个文件





剩下的地方可以设置args参数 例如load_file还有work-dir

```python
work_dir = './exp'
```

```python
  checkpoint=dict(type='CheckpointHook', interval=1,max_keep_ckpts=5,save_best='auto'),
```
