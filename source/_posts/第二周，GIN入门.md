---
title: 第二周，GIN入门
date: '2024-10-03 21:03:19'
updated: '2024-10-03 21:45:28'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1727960975879-69b8efa7-7210-44e9-aebc-9ba048065f69.png'
description: 1.入门首先最简单的直接抄袭官网的快速开始，常见的三步。1.定义server的断开。2.设置路由对应handle函数。3.设置handle函数设置server设置对应的处理函数，匿名函数，传入的是ctx是关键开始run路由匹配参照spring的request param和正则匹配通过param...
---


# 1.入门
首先最简单的直接抄袭官网的快速开始，常见的三步。1.定义server的断开。2.设置路由对应handle函数。3.设置handle函数



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ccb9d52bdd48f22e00244b608e64ede1.png?token=AGECE2OZ66JW2LSJYPS3NQDG72WDY)



1. 设置server
2. 设置对应的处理函数，匿名函数，传入的是ctx是关键
3. 开始run





## 路由匹配
参照spring的request param和正则匹配

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/f0411e25f3e7e25a5aa5b95d2a2de0e0.png?token=AGECE2NBVZSV74BCWCVUISLG72WD6)

通过param来获取参数，这个与java<font style="background-color:#FBDE28;">@request param</font>取值差不多的思路



还有一种<font style="background-color:#FBDE28;">是在？这种参数，</font>查询参数，使用query进行获取

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/9de678930dfcb14e293b05b4329c2934.png?token=AGECE2NYQ2E7DBXLMVXLLPLG72WEC)





# 设置register函数
> 当需要注册的路由太多，我们需要进行调用的server.get次数就会很多次。可以进行抽象出来一个方法，放到专属的类，每一个路由都在类里面就有处理方法。
>



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/c501efd1a7253d328030d649cfc23418.png?token=AGECE2P3WBJHIRFOAS44AG3G72WEG)





下面就是优雅之后的代码。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/9497128b0e43301c60bf291612ef876e.png?token=AGECE2K3HGWGHCGTDS54U4DG72WEK)





## 分组路由
相当于spring的requestmapping放入到类最前面的路径



通过ug来接着子路径

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/aad681de2fd34d93819c5778d07c748e.png?token=AGECE2IGNKYVUHQX5KN2A73G72WEO)



# 数据接受
## vo传输
与java类似，传输数据前后端，使用的需要vo，pojo来进行json传输数据。不同的是java可以自动封装传输的数据位class对象，这边需要把前后端的进行映射才能组装位vo，使用的是反引号·

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/1cd06b1ef0d3bbbcf3bb61edac21efa8.png?token=AGECE2K6CP5DMAOU7R5KKATG72WEU)

<font style="color:#ED740C;">这个也与email小写就是私有变量有关，所以需要进行编码</font>



## 绑定接受
主要是在ctx数据，调用ctx来进行绑定数据

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/62baf837038d6b89187cff06e59773e1.png?token=AGECE2LLUAAQOFLK6VUB6GDG72WEY)



## 业务处理
使用正则进行校验密码是否符合规则



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/0d15d8c52ffa4ec42cdfc4df863bed4d.png?token=AGECE2LGDE22PECR5KIN4MDG72WE4)

之后直接调用这些

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/19268f4cb075fcbeb81bcbcbe9363a62.png?token=AGECE2LSNWLTUMCXZ6R773DG72WFA)



# 跨域问题
后端的是8080端口，前端的是3000端口，这就设计到跨域问题

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/1a84704adda37b138603dd1799f1c5c1.png?token=AGECE2P3RQVZHOLGDL7KFMDG72WFQ)

csrf，这个时候需要引入中间件，设置cors可以进行访问对应的路径





解决的方案使用preflight，就是发给server后，server返回我接受3000所有发过来的方法。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/34c72da4a2d0160c09a324c700201eb6.png?token=AGECE2JSZ4OOIVVGHQZYVZ3G72WFU)



通过这里middleware进行注册，授权访问。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/a0527c2e021bcd46d7fdcfbf8b0895f0.png?token=AGECE2ODCDZL6HKMGNLTYP3G72WF2)

相当于spring的filter，这里面来进行处理，在这个allow里面进行允许，3000的端口





# gorm入门
最终的数据操作还是要落到数据库的crud，所以需要gorm



## 0、docker安装mysql
使用docker-compose来进行安装，这里基本上，就是构建操作的image，映射的端口还有volume。这个相当于节省了自己写docker命令，

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/b57dbe40157c0df10f99b667c9012d67.png?token=AGECE2KILXGVTA6CD3TXIKTG72WF6)

## 快速入门
1. 定义连接
2. 定义对应的class
3. 操作数据对象

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/6e70d6c203a3fceb36489b897027740e.png?token=AGECE2O6YXPS3NIYD57VHYLG72WGQ)





## class对象定义
这里用到<font style="background-color:#FBDE28;">组合的语法，</font>传入class，不加变量就是组合，之后自定义自己需要的变量

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/a57b3ea83cefe6e4b8e59c88701c0c7b.png?token=AGECE2JGPTDDOYNVQEYOZYDG72WGW)





# service-repository-dao
相当于mvc的三层操作方法，repository表示database和redis，都是可以进行返回数据的地方，dao才是最终去数据的操作。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/a40eefd96b46e7c02297056ef7ace7ef.png?token=AGECE2KEJ4MQ6R2DZUWJP73G72WG2)

web文件夹是相当于视图层了，service代表业务的完整流程，class变成其他的class。





