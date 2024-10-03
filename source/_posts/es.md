---
title: es
date: '2024-07-02 14:50:36'
updated: '2024-07-02 16:54:05'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1719907987230-3347aacd-3f8c-4c03-8f73-156a0a8403af.png'
description: 问题引入倒排序，就是通过搜索引擎输入关键词，插入出文章，改怎么处理使用倒排索引，就是进行切词，然后根据词语，设置哪些文章出现了这个词语但是这样查询就是ON的复杂度，一个个查询单词，因此可以使用字典树单词+出现的列表就构成了倒排索引因为数据太大，只能放入到disk里面term index数据压缩...
---
# 问题引入
> 倒排序，就是通过搜索引擎输入关键词，插入出文章，改怎么处理
>

使用倒排索引，就是进行切词，然后根据词语，设置哪些文章出现了这个词语

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/5b6409dde38978d12a588510c261ed7d.png?token=AGECE2J5U5GZFIOQIQWWWHTG72WTK)

但是这样查询就是ON的复杂度，一个个查询单词，因此可以使用字典树

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/b93397f56428fcbe1b339015745c1f58.png?token=AGECE2PKWCRLWN5VNXWXB4DG72WTO)

单词+出现的列表就构成了倒排索引

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/63df2f28aaabecd69889113e40bf0d16.png?token=AGECE2LEGBC7VOSES4DJOCLG72WTS)

因为数据太大，只能放入到disk里面

## 
## term index
数据压缩，有共同的前缀，不需要一个个单词出现

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/4fd228683c1daa4fbf43a51d95a93bb4.png?token=AGECE2NH2M4RCLU7NMPFHSLG72WTW)

## stored fields
之前查询的是文档id，需要id查到内容，存放完整的内容就是stored

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ce6b24e2d9c52c9d6ec7e00297ba2426.png?token=AGECE2OS3O3LGEX2T7BWW3LG72WT2)



## doc value
是将文档id映射到对应的字段，例如文档是一个手机的介绍，他就映射到时间是什么时候产的，价格是多少

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/c644bde6b616e9d197ca6c29d39ac5a6.png?token=AGECE2MRKGUC4B34I2P4ZADG72WT6)



## segment就是上面的合并
![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/46c57ffecac39cb317a78b7933b2547c.png?token=AGECE2PPRQBNCNIJVZNLGT3G72WUE)



## lucene
![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ae11fce5573a5d738a49bf3b92c2553a.png?token=AGECE2PQRT3M5XVAEWWTSOLG72WUI)并发读取segment，小的segment定期进行合并



# 高性能
也是按照之前kafka的思路优化，切分为不同的topic，这里的就是index name1

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ebfca7788229aef230572aaecd3cad77.png?token=AGECE2MAN5OYCZEBFZJWQALG72WUO)

之后再次按照kafka更新的思路，切换为不同分区，这里是叫做shard

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/97d88b3d08285948bb6de3c7fa31932f.png?token=AGECE2MXYBXJO7ATDCFDWS3G72WUS)



# 高扩展性
照样参考kafka，这个是broker编程node

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/8bdeea91361d0b850ec0cb9d119f0526.png?token=AGECE2OMHDRE233H6QBW5WDG72WUW)

# 高可用性
参考kafka的leader

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/2a23ed93f2ccd481658f70cba1613194.png?token=AGECE2PYJEM3XEV6DLQZPOLG72WU2)



# node角色化
这个是参考gfs，分为master，数据节点，协调节点

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/cf7a814bdeaa7cf9321b6eb31cd3b586.png?token=AGECE2PO6MMXDUD3SRM3GEDG72WVE)<font style="color:rgb(63, 63, 63);">叫</font>**<font style="color:rgb(15, 76, 129);">主节点(Master Node)， 负责存储管理数据的，叫数据节点(Data Node)， 负责接受客户端搜索查询请求的叫协调节点(Coordinate Node)。集群规模小的时候，一个 Node 可以同时</font>**<font style="color:rgb(63, 63, 63);">充当多个角色，随着集群规模变大，可以让一个 Node 一个角色。</font>

<font style="color:rgb(63, 63, 63);"></font>

<font style="color:rgb(63, 63, 63);">使用raft来进行去中心化</font>

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/3843516bb33ed0e16c85b9cdedcd4327.png?token=AGECE2LSXIIJDKUDINDSGA3G72WVI)



# 写入流程
使用协调node，找到对应的data node

写到底层的segment，之后进行同步shard

然后发送ack代表写入ok

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/f810441a87eddd5c9e4677f776bcc01d.png?token=AGECE2OCZOUKYPNIFPMVBYDG72WVM)



# 查询阶段
和之前写入阶段，差不多，但是是并发查询segment，然后协调节点来进行排序聚合。

之后根据doc id再次获取完整的内容

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/5d0a7933f6fbd8378251559191881e6f.png?token=AGECE2NL62YKHNHWGMN6ZODG72WVQ)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/08d6e33ba9e5428d97fbd5d25b53dab5.png?token=AGECE2NKDT3F3WO7JNMIBHLG72WVW)

