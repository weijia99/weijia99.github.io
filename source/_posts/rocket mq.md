---
title: rocket mq
date: '2024-07-02 14:35:44'
updated: '2024-07-02 16:06:45'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1719906718228-0e60bcab-2c37-4e90-9c53-f3b2a17cc754.png'
description: 问题引入上一节介绍了kafka是一个消息队列，但是如果我想某个消息过了半小时之后再次进行消费，这就是延时消息，这个使用需要rocket mq与kafka对比都是消息队列，那么就可以进行对比kafka是使用zookeper来进行，rocket mq使用的nameserver分区被修改程队列使用的...
---


# 问题引入
上一节介绍了kafka是一个消息队列，但是如果我想某个消息过了半小时之后再次进行消费，这就是延时消息，这个使用需要rocket mq



# 与kafka对比
都是消息队列，那么就可以进行对比



kafka是使用zookeper来进行，rocket mq使用的nameserver

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/10067c03abf3a7dccd4a8c8662368479.png?token=AGECE2IATYPCCGZD2J4Q5UTG72WWC)

分区被修改程队列

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/a1a19687b4902c96239db43e6efb62e1.png?token=AGECE2MPQROFEXNUVUHHALDG72WWG)

使用的mysql回表查询方式，首先找到offset，然后去commitlog找到body

kafka直接查询offset就可以使用![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ad795c75ee1a2d7ecbf85d915ef4273f.png?token=AGECE2JD3K2DSHTUOMV6PVLG72WWK)



rocketmq在单个broker下降所有的topic的数据写入到一个broker里面，然后直接查询这个commit loig

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/d789bc14c972fc98d96385588d600ad7.png?token=AGECE2M6ZTZBYBFKICKLZFLG72WWO)



备份机制，备份的是commitlog ，不再是分区

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/00642f778f42b58f5dd6150580a10548.png?token=AGECE2NXGDJHHD3X33HJ3H3G72WWW)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/c92408ad85004d8172418f2e0d80ef68.png?token=AGECE2JB3ECMBFPTMXJ2Z33G72WW2)



# 独有的功能
### 消息过滤
vip用户的消息还有普通用户的消息，可以打入tag来作为标记来进行区分

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/6a18785b3f4b653f8bab7059aa488e28.png?token=AGECE2LSJ3TXLEQEUJ2COBLG72WXA)



## 延时队列
延时查询，是否缴费



## 死信队列
消息重试次数太多，放入死信队列



## 实现分布式事务
![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/9ee0101f2f9a5f74f3d5439d9a8edd31.png?token=AGECE2PRRX66U2EGF3Y7CLTG72WXE)

