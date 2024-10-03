---
title: mmdet代码实战
date: 2023-06-10 17:01:45
tags: [detection,pytorch]
---





# MMDetection代码实战

这一届主要讲解了，如何构建cfg，如何在目标检测上设置自己的配置文件。主要的思维导图如下



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/1686387759691MMDetection%E4%BB%A3%E7%A0%81.png)

## 1.安装

直接跳过



## 2.设置配置文件

我们首先设置metainfo，因为只有一个猫类，所以我们只写猫，还有类别是1，那么head头输出就是1，然后设置dataset，首先是设置metainfo，然后就是设置json文件，还有iimg的文件，之后就是设置学习率，还有一些保存checkpoint的配置。



为了验证模型的数据对不对，我们使用自定义代买来进行检查，传入到cfg，之后吧dataset进行可视化

```python
# 现在是测试cfg对不对的
from mmdet.registry import DATASETS,VISUALIZERS
from mmengine.config import Config
from mmengine.registry import init_default_scope
import matplotlib.pyplot as plt
import os
cfg = Config.fromfile('./rtmdet_tiny_1xb12-40e_balloon.py')
init_default_scope(cfg.get('default_scope','mmdet'))

dataset = DATASETS.build(cfg.train_dataloader.dataset)
visualizer = VISUALIZERS.build(cfg.visualizer)
visualizer.dataset_meta = dataset.metainfo

plt.figure(figsize=(16,5))

for i in range(8):
    item = dataset[i]
    img = item['inputs'].permute(1,2,0).numpy()
    data_sample = item['data_samples'].numpy()
    gt_instances =data_sample.gt_instances
    img_path = os.path.basename(item['data_samples'].img_path)
    
    gt_bboxes = gt_instances.get('bboxes',None)
    gt_instances.bboxes = gt_bboxes.tensor
    data_sample.gt_instances = gt_instances
    
    visualizer.add_datasample(
        os.path.basename(img_path),
        img,
        data_sample,
        draw_pred=False,
        show = False,
    )
    drawed_image = visualizer.get_image()
    
    plt.subplot(2,4,i+1)
    plt.imshow(drawed_image[...,[2,1,0]])    
    plt.title(img_path)
    plt.xticks([])
    plt.yticks([])
```





之后就是训练阶段，使用train来进行训练



## 3.测试与分析

测试与之前一样，使用test来测，使用--show dir可以可视化。之后就是对图像进行特征化，这个时候需要借助mmyolo的功能，因此需要下载mmyolo





```
python demo/featmap_vis_demo.py 5555705118_3390d70abe_b.jpg ../mmdetection/rtmdet_tiny_1xb12-40e_balloon.py ../mmdetection/work_dirs/rtmdet_tiny_1xb12-40e_balloon/best_coco_bbox_mAP_epoch_80.pth --channel-reduction squeeze_mean --target-layers neck
```



```
python demo/boxam_vis_demo.py 5555705118_3390d70abe_b.jpg ../mmdetection/rtmdet_tiny_1xb12-40e_balloon.py ../mmdetection/work_dirs/rtmdet_tiny_1xb12-40e_balloon/best_coco_bbox_mAP_epoch_80.pth --target-layer neck.out_convs[2]
```

