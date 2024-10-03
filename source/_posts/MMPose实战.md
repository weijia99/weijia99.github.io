---
title: MMPose实战
date: 2023-06-03 19:33:33
tags: [mmlab]

---

# 1.mmpose实战

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/1685794454339MMPose%E5%AE%9E%E6%88%98.png)

主要包括安装，mmdet实战，mmpose实战等三部分

## 1.安装

安装方法，可以直接去看官网教程，主要是是首先下载pytorch，之后下载mmcv，使用mim来进行安装。然后在github下载源码，使用源码进行安装。难点包括网速原因（ladder或者是搜索加速网站）

建议新装一个环境，pytorch和mmcv要一一对应。我之前是2.0.安装之后无法训练，是版本原因。所以重装了一个conda环境。



## 2.mmdet

mmlab代码的整体训练流程

1. 数据集同意处理程coco的格式，使用lableme
2. 设置cfg配置文件，这个歌文件包括模型，优化器，超参数，数据集还有pipeline，和权重
3. 使用train来训练上述cfg或者是分布式训练使用bash脚本bash dist——train模型cfg
4. 之后就是使用test来进行验证文件 test 模型 +pth进行验证
5. 之后就是预测，一般使用命令行得到结果



## 3.mmpose

与mmdet同理，还是多个步骤



最后一步是集合mmdet的权重+mmpose的权重来指定向下来进行预测



## 4.作业

关于中医耳朵

1. 首先是数据集下载，使用bypy这个python包进行下载
2. 使用mmdet的训练流程来进行训练
3. 使用mmposexunlliucheng进行训练
4. 使用test来进行计算准确率
5. 集合上述两个一起进行结果预测



