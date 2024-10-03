---
title: sovits训练教程
date: 2023-07-10 11:06:38
tags: [语音识别, deep learning]
---

# 简介

[so-vits-svc/README_zh_CN.md at 4.1-Stable · svc-develop-team/so-vits-svc · GitHub](https://github.com/svc-develop-team/so-vits-svc/blob/4.1-Stable/README_zh_CN.md)

歌声音色转换模型，通过SoftVC内容编码器提取源音频语音特征，与F0同时输入VITS替换原本的文本输入达到歌声转换的效果。同时，更换声码器为 [NSF HiFiGAN](https://github.com/openvpi/DiffSinger/tree/refactor/modules/nsf_hifigan)解决断音问题。





# 训练流程

## 1.数据集处理

### 1.1收集数据集

以b站直播为例：

1. 打开up主的直播回放列表[乃琳Queen的个人空间_哔哩哔哩_bilibili](https://space.bilibili.com/672342685/channel/seriesdetail?sid=222754)
2. 选择单播一个人的
3. 使用downkyi，[Releases · leiurayer/downkyi (github.com)](https://github.com/leiurayer/downkyi/releases)进行视频下载



### 1.2数据集预处理

首先进行人声提取，我们主要训练的是人声

![img](https://www.freedidi.com/wp-content/uploads/2023/06/2023-06-09-163727.png)

使用uvr5，[Release v5.5 - UVR GUI · Anjok07/ultimatevocalremovergui (github.com)](https://github.com/Anjok07/ultimatevocalremovergui/releases/tag/v5.5.0)，选择模型demucs（这个是facebook开元的去噪模型），进行数据集预处理





接下来是数据集进行分片，使用[flutydeer/audio-slicer: A simple GUI application that slices audio with silence detection (github.com)](https://github.com/flutydeer/audio-slicer)，来把谷歌得到人声进行切片，保证每一个最长不超过15s





## 2.训练过程



参照官网文件

[so-vits-svc/README_zh_CN.md at 4.1-Stable · svc-develop-team/so-vits-svc · GitHub](https://github.com/svc-develop-team/so-vits-svc/blob/4.1-Stable/README_zh_CN.md)

### 2.1下载预训练权重

![16889594099901688959409276.png](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16889594099901688959409276.png)

[Box](https://ibm.ent.box.com/s/z1wgl1stco8ffooyatzdwsqn2psd9lrr)，获取模型权重，放入到pretrain文件夹里面。作为编码器（把声音转化为vector）



### 2.2数据预处理

把上面处理好的数据，放入到data_raw文件夹里面，我们首先，进行重采样到44k

```shell
python resample.py
```



接下来进行划分训练集还有测试集

```
python preprocess_flist_config.py --speech_encoder vec768l12
```



接下来修改训练参数，现在工业化的的代码，都是写入到从config文件里面，我们修改config的batchsize还有port，其他的按需修改。



最后就是生成hubert

```
python preprocess_hubert_f0.py --f0_predictor dio
```



### 2.3开始训练

和mmtools一样，也是使用config文件进行训练，，-m 代表，训练保存的名字，在logs目录下面

我们为了加快训练（可以放入g—0，还有d-0来加快训练流程）

![16889598089881688959808436.png](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16889598089881688959808436.png)

```
python train.py -c configs/config.json -m 44k
```



### （可选）2.4hpc训练流程

使用sbatch来进行训练，主要的是用于下面这个脚本来执行gpu.slurm

```shell
#!/bin/bash       
#SBATCH --job-name=test-gpu
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --gres=gpu:1
#SBATCH --error=%j.err    
#SBATCH --output=%j.out    

CURDIR=`pwd` 
rm -rf $CURDIR/nodelist.$SLURM_JOB_ID
NODES=`scontrol show hostnames $SLURM_JOB_NODELIST`
for i in $NODES
do
echo "$i:$SLURM_NTASKS_PER_NODE" >> $CURDIR/nodelist.$SLURM_JOB_ID
done

echo "process will start at : "
date
echo "++++++++++++++++++++++++++++++++++++++++"


nvidia-smi
source /home/software/anaconda/anaconda3-2023.03/bin/activate pytorch
#wget https://mirrors.tuna.tsinghua.edu.cn/julia-releases/bin/winnt/x86/1.9/julia-1.9.1-win32.tar.gz
python train.py -c configs/config.json -m as
#ifconfig
#ping baidu.com
#jupyter notebook
# conda list | grep mmeng
# which python
# python demo/image_demo.py demo/demo.jpg rtmdet_tiny_8xb32-300e_coco.py --weights rtmdet_tiny_8xb32-300e_coco_20220902_112414-78e30dcc.pth --device cuda:0


mpirun -np $SLURM_NPROCS hostname

echo "++++++++++++++++++++++++++++++++++++++++"
echo "processs will sleep 30s"
sleep 30
echo "process end at : "
date
rm -rf $CURDIR/nodelist.$SLURM_JOB_ID

```

和上面没有什么差别，但是我们只需要激活自己的环境，使用source，之后就是和之前一样，使用python进行训练，上面的参数，重点关注#SBATCH --gres=gpu:1  这个是分配gpu的数量。

然后使用训练代码sbatch gpu.slurm进行训练

之后使用cat 进程号.out来查看训练流程

# 推理流程

从服务器下载权重文件，只需要g-20000.pth来进行生成，因为是生成阶段，所以我们只要进行生成器，同事还需要，这个的config文件。



下载到本地使用本地的webui来进行执行。或者直接在服务器使用

```shell
# 例
python inference_main.py -m "logs/44k/G_30400.pth" -c "configs/config.json" -n "君の知らない物語-src.wav" -t 0 -s "nen"
```





注意推理也是只要人声阶段。





# 合成音频

使用剪映，把谷歌生成得到的人声，还有使用uvr5提取的伴奏，都拖到剪映的音轨上面，进行对齐，这样，我们就可以直接使用剪映导出生成的视频。
