---
title: k8s
date: '2024-07-02 14:35:24'
updated: '2024-07-02 17:13:51'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1719911333939-37ad56b8-7e41-40a6-8f95-933a2b4b0e56.png'
description: 问题引出当你部署的博客访问量太大，结果挂了，需要重启，内存大于多少g才可以，这个时候又是自己手动来进行部署，肥肠的耗时间。介绍k8s通过yaml，来进行自动重启，自动扩容架构原理也是控制节点还有工作节点组成控制节点api server：手动实现的api来进行操作scheduler：查看哪一个工...
---
# 问题引出
当你部署的博客访问量太大，结果挂了，需要重启，内存大于多少g才可以，这个时候又是自己手动来进行部署，肥肠的耗时间。



# 介绍
k8s通过yaml，来进行自动重启，自动扩容



# 架构原理
也是控制节点还有工作节点组成

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/6437bc447da6354fd96577bb1c7295e1.png?token=AGECE2NFA4FWB3ZFEHRHMG3G72WZ4)

## 控制节点
api server：手动实现的api来进行操作

scheduler：查看哪一个工作node足够，然后才能部署

controller ：进行创建，关闭服务

etcd：来保存一些数据

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/9a6dccbb62160debee5cc985a8d99cf3.png?token=AGECE2JZF2ZVHZ7YUNGAPMTG72W2A)

## pod
多个container组成一个pod

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ed035cfa1b5fd40e0e6430ecc0bdca20.png?token=AGECE2LQECDUJJSKX6ZL6PTG72W2E)

pod运行在node，

k8s可以将pod移动到其他node上面



kublet这个是接受上面controller的命令的

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/0b1a4f79bebb8056500dd82bd0bd1c57.png?token=AGECE2PVNANQPKP3GFY2SIDG72W2I)



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/40eb8cd7fa6c1bb1c2fd15d80350cfe8.png?token=AGECE2N7JNWUMGXBRVA7CA3G72W2M)

