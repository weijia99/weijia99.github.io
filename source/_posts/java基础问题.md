---
title: java基础问题
date: '2024-08-23 09:35:34'
updated: '2024-08-23 17:18:03'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1724377233457-46c62664-1864-4508-8b04-f6019de8c647.png'
description: '重写的范围综上：重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变。方法的重写要遵循两同两小一大”（以下内容摘录自《疯狂]va讲义》）：·“两同”即方法名相同、形参列表相同；·“两小"指的是子类方法返回值类型应比父类方法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方...'
---




## 重写的范围
综上：重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变。

方法的重写要遵循两同两小一大”（以下内容摘录自《疯狂]va讲义》）：

·“两同”**即方法名相同、形参列表相同；**

·“两小"指的是子类方法返回值类型应比父类方**法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等：**

**<font style="background-color:#FBDE28;">相当于就是返回子类是可以的，继承返回孩子类型</font>**

·“一大"指的是子类方法的访问权限应比父类方法的访问权限更大或相等。



## 可变参数
这个和go里面差不多也是使用string.... args

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/5d9c4c62482fa0cb67dbb52cfdda7f23.png?token=AGECE2PEIAXV5CZP2R5OF4TG72WSO)

## string 问题
stringbuffer和string都是线程安全的，因为有锁

builder知识构造者，不是线程安全的



string是private final数组，不可以变，无人更改，没法继承，jdk9是byte，为了支持拉丁文



st<font style="background-color:#FBDE28;">ring的+=重载符号是调用stringbuilder的append来进行构建的，这样会创建多个builder导致出现缓冲问题</font>

<font style="background-color:#FBDE28;"></font>

<font style="background-color:#FBDE28;">intern</font>：是为了将string放入常量池

