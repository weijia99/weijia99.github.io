---
layout: git
title: github的action
date: 2022-10-26 17:58:55
tags:  [action]
---

# action 操作

## 1.workflow是什么

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667785330551666778532504.png)

假设你开发的java软件又bug,用户提交问题到issue,代码人员进行修复,修复完成,之后进行pull request,然后就进行合并,合并之后旧的进行测试才能发布,这就是一个流程





![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667787870631666778786133.png)





## 2.名称解释



**event**

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667789010571666778900736.png)

就是我触发的条件(pull request)





workflow就是一系列自动化流程

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667789610561666778960997.png)





常用的cicd

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667791370571666779136700.png)





## 3.为什么使用action

因为你 不想配置环境变量   





## 3.demo





### 3.1yaml教程

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667809480571666780947313.png)

使用tab就是一个对象,使用kv来记录纸





使用过 list记录多个,那就需要-

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667810550641666781054310.png)





### 3.2demo简介

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667812140551666781213509.png)





![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667812860561666781285494.png)

1. name是可选
2. on就是event,单位需要出发的事件







jobs,就是执行的事件

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667813680551666781367823.png)



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667815090571666781508152.png)

到这一步就是进行代码检查,使用github编译好的checkout

每一个-,代表一个list,就代表一次操作

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667816150561666781614964.png)





> uses,时使用别人的action,run是自己执行linux命令

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667816900551666781689860.png)





## 4.cicd在哪里执行

在github的服务器上 





jobs是并行的,如果publish需要build,那就要使用关键字,need





## 5.总结

首先 on 是触发条件(push,pull)



接下来就是正常的jobs



首先第一步就是checkout,第二部就是设置值环境时候用java,然后就是读取使用run,之后就clone文件,并且进入,之后就是build,一般使用gradle,并且权限777



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16667840363551666784036329.png)
