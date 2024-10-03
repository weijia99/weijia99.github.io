---
title: 第一周，go语言拾遗
date: '2024-10-03 19:56:17'
updated: '2024-10-03 21:03:42'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1727956986927-b9d68bdf-c839-4ebc-abf1-e35003ecff6e.png'
description: 本节主要讲解的是go语言的基础语法，主要参照的是自己的遗留问题。文件路径package和java里面的包一样，也是需要先定义好文件夹路径package的名字可以与文件夹的名字不一样，同一个文件夹下面的包名药一直，test文件除外基础数据和java差不多也是int，uint，float这些数字的...
---
本节主要讲解的是go语言的基础语法，主要参照的是自己的遗留问题。





# 文件路径
## package
和java里面的包一样，也是需要先定义好文件夹路径

package的名字可以与文件夹的名字不一样，同一个文件夹下面的包名药一直，test文件除外

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/3a093da88d6ce2171247f5e3a29f60a6.png?token=AGECE2KKSNTZTRBWVMEVVALG72WHE)



## 基础数据
和java差不多也是int，uint，float这些

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/650d6d11a76e0771b81c9a8fb140c502.png?token=AGECE2JYNMOACWXFRDGUD43G72WHK)

数字的极限，只能通过使用math包来进行获取

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/b8056e8ba4db9c3e14680882bf2b174b.png?token=AGECE2PA5B4HU6NISN2A5TLG72WHO)





### string
与python类似，使用反引号这是个特点`进行作为字符串

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/e37d05e1672f313baefef246512e8323.png?token=AGECE2LAY47C5XJXSZA7KQ3G72WHS)



# 常量与变量
使用var来进行定义，这个与js差不多，自动类型推断，先写变量名称在写类型，与rust差不多。

。通过使用大小写来作为pubilc与private的进行区别。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/d12d12908ddefd5965378077c6833049.png?token=AGECE2PUFNMXWDJOBUYAXVTG72WHW)



使用：=来进行类型推断。只有局部变量才可以

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/d089224da19e1a497b7cfba9551b16fd.png?token=AGECE2LWRKUVSAAHVRBYI6TG72WH4)



常量，使用const来进行修饰。这个字不可以进行修改的

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/55d6436c3449dca1fcfe7c7f819ee4ab.png?token=AGECE2M2QYTKJS7HSWVQWFDG72WIC)



iota进行自增加，主要是自我增加。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/a798ab7f2a1a0082b64f0153c32369dd.png?token=AGECE2NMADWJNQ3IBMF6JR3G72WIG)



# 方法声明
常见的代码，首先 定义是函数，然后设置名称和类型。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/b1c04c4efe91f85974c7b089a09bd339.png?token=AGECE2JSACDVS2FJ5FULN33G72WIK)

可以返回多个类型与python类似



返回值的接受与变量差不多，直接使用：=来进行接受返回值。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/27bc303fddc3c6c8d44164fda30ca690.png?token=AGECE2LJG4S6OHJXCESUC73G72WIO)



## 函数式编程
> 主要思想就是把函数作为参数，然后如果要使用函数，直接加上（）进行调用，没有括号就是普通的变量
>

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/40f64782bb62257bb6b10eb3d06cf5c7.png?token=AGECE2IOKZ5YGX2PS62ZY63G72WIY)



## 闭包
这个概念与rust的概念差不多，就是函数的自带参数，可以传入到她返回的函数里面。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/29d3ea64fd7cfce26ea2f8a90687bef6.png?token=AGECE2PQX2WZOLODPGG6WOLG72WI2)



## 不定参数
这个与java类似，差不多就是。。。表示

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/38e32681df6a36654d7fe4fbc0a8c248.png?token=AGECE2J2DN4J4NIVV7FWLQ3G72WJA)



## defer
经典的关键字，相当于java的finally，但是我可以提前写，这是最后要执行的语句

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/afcc91f99008e8c59f6a88e4c68fbc86.png?token=AGECE2PRXLYEZNJBSRFTXF3G72WJE)

顶用顺序与栈类似，filo，第一个最后一个调用





# 内置数据结构
## 切片
相当于arraylist，使用切片可以作为扩容，

主要的操作，使用make进行构建，append初始化，make可以坐初始化和capacity的初始化

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ca220cabfb464f745941f4de640c824a.png?token=AGECE2NA6GOUYPMJJLJQ4ULG72WJU)

> 只要进行扩容了，数组就不是同一个数组的啦，那就不相等了
>

范围获取，参考python，铅笔后开



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/d6ddf70a8941b69bc5300cbb2691d0d2.png?token=AGECE2P3E62ESWSDERXBHHTG72WJ2)





## map
hashmap的视线，也是通过构造make来实现

map[string]int这种返回类型，



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/f9584b9dd796bee0f3c0ab0ab38d8546.png?token=AGECE2LLH4E6JP562ITY7KDG72WJ6)

赋值就和pyton差不多了



读取的时候，要自己进行处理，不存在的逻辑。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/0c87a0c73b4d7ca0e149ca3684e2baf2.png?token=AGECE2NYBTNJZNSUWDUSXOTG72WKC)

返回err

，删除元素使用delete进行

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/85dd6169c0496ac559483795fa7fa5e0.png?token=AGECE2KGXMAKCTO632TQW7LG72WKE)



# 接口
与java的接口差不多

语法就是tppe a interface，进行定义类型行为

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/f3b80543db4d78a98e62d974571756d7.png?token=AGECE2KKLCUGWJE4O7MOVLLG72WKI)



## 结构体
与class类似，只要实现上面接口的所有方法就是实现，无须关键字



前面的犯法，加上限定范围，与rust类似，表明是class的函数

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/daced2fe4a8f337034d522241ab2a841.png?token=AGECE2M57P52G6JM7VTLWXDG72WKM)



初始化，这个参考的是c++的，直接使用{}就可以进行初始化，加入&，表明取到地址了。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/dd8037ad287980a8ee6016e77e6f07e5.png?token=AGECE2KQBD2AM3YUIMGHG7TG72WKU)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/0d31cf0400997aa471a955f04d2356c8.png?token=AGECE2IRQD6RXO3BCH6MS7TG72WK6)



*是具体操作对象，&是取地址，参照c++的语法。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/b70262dcbb6bbecfa654417e63976f57.png?token=AGECE2NZN5V63XM6QY6CT23G72WLA)



一般方法，使用指针在前面，



class实现interface接口，使用ide自动搜索interface，限定参数都是地址。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/2859c00d503cbf5c0259cd38b75605b2.png?token=AGECE2P4W6PNEJPBJDBBKBTG72WLG)



# 泛型
与java类似，也是使用T来作为



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/a83bd47918753998b7ac2bcdb09eef42.png?token=AGECE2LASCNOIL3UYBMFWTTG72WLK)



![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/d85576cf2cd618052afde864167163d1.png?token=AGECE2M4UZRTUGSWHB4Z27DG72WLO)

参照，这些语法来进行，number是作为约束，与java的？这种限定差不多

