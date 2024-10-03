---
title: 多源bfs
date: 2022-11-12 21:26:33
tags:
---

# 多源bfs&最小树

## 0.证明



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682604660491668260465791.png)

归纳法,开始为0,不用证明

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682605600491668260559129.png)

去除对头的第一个元素x,可以加入3个x+1的元素,最多有两端





两个特性,一般是队列,前面是x,后面是x+1

默认开始的元素是最小值,喝dij的使用优先队列的最小值一样.

入队就是最小值的

## 1.bfs

### 1.1矩阵距离

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682596642021668259663290.png)

> 大致意识就是求每个位置到1的最短距离

这个就是求最短路,求每个点到一堆起点的距离,建立一个虚拟起点,让1 作为起点开始寻找,然后使用虚拟起点,连接所有的 1

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682598242021668259823715.png)

思路:先把所有是1的位置加入到queue里面,距离是0

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682603282041668260328053.png)

重点是找到放入所有的值,还有一个就是进行更新,tt=-1



______



总体思路如下

1. 使用1作为开始的点,把所有的1进行插入到队列
2. 之后就是常规bfs,进行pop
3. 然后第二阶段就是搜索周围的元素,找到符合的元素,二姐没有被使用(没有被使用就是距离为-1),使用的直接continue
4. 然后进行更新,更新之后在把他插入到队列里面,





## 2.魔棒

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682614080511668261407501.png)

 



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682616770571668261677035.png)

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682616960541668261695181.png)

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16682617290551668261728112.png)
