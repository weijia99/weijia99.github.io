---
title: git控制
date: 2022-10-25 10:26:53
tags: 
---





# git操作



## 1.模型



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666653059061666665304945.png)

模型树。对于文件夹，叫做tree，对于文件叫做blob，根目录是root





git工作流

使用分支branch，还有merge合并

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666656019051666665601612.png)

元数据：作者，message





**数据模型**

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666657919001666665791145.png)

定义的文件为数组，tree是hash隐射，然后commit是要提交的stack



一个obejcect是一个版本，维护，使用hash进行映射

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666659669051666665966713.png)

 





## 2git demo

暂存区





> 为什么add和commit分开

因为add可能已经完成了一个新的feature，还有一些没有完成，只提交完成的就可以进行发布

也可能不想上传日志文件







git checkout 进行版本切换回退 





git diff进行代码比较

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666813658981666681365480.png)

可以比较不同时期某个文件的的区别





## 3.分支

git branch cat，新建一个cat分支

 ![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666817818981666681781631.png)

可以之间checkout -b创新建新分支，然后进去





回到什么功能都没有的master分支，进行合并cat和dog分支

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666821539001666682152951.png)



首先git merge cat





然后出现合并不兼容的情况（conflic）

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666822748981666682274329.png)

 

因为那个if判断不对







最终结果

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16666824468971666682446494.png)
