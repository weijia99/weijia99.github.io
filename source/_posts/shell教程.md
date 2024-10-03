---
title: shell教程
date: 2022-10-19 10:44:36
tags: [linux,shell,计算机教程缺失的一课]
---







# 0.shell脚本



## 0.1shell赋值



一般使用双一号,这样可以进行输出

使用方法和python的f差不多,直接使用$

```shell
foo=bar
echo "$foo"
# 打印 bar
echo '$foo'
# 打印 $foo
```

![image-20221019203249378](D:\code\site\weijia99.github.io\source\_posts\shell教程\image-20221019203249378.png)

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661831919761666183191589.png)

## 0.2shell进行函数变换

如何使用sh脚本,直接加载到source,使用source保存,然后直接运行函数

&1-9是保存的函数变量,



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661833769821666183376434.png)





> 经常会遇到权限不够的问题,写入,或者读取,那么只要使用sudo!!,就可以执行上面一个权限不够的命令

 

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661835669731666183566863.png)





grep是查找函数 ,$?代表是不是有错误,正确就是0(没有错误,0个错误)



这里的||的意思是第一个不对,就执行第二个

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661838109721666183810355.png)



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661839759721666183975015.png)

目前看来这里的||还有&& 都是相反的,一个是只有,一个是或者

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661840729731666184072240.png)

**使用()代表的是全局变量,局部自定义的变量不需要括号,linux命令**

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661843409731666184340326.png)











例题讲解

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661844299741666184429787.png)



0代表函数的名称, &#代表参数的数量(#是number的意思) $$代表当前运行的pid 



- `$@` - 所有参数 读取参数

-ne是不相等就代表错误

for循环所有的参数,然后进行grep查找,有就进行写入,没有就有错误,然后进行追加









- > 花括号`{}` - 当你有一系列的指令，其中包含一段公共子串时，可以用花括号来自动展开这些命令。这在批量移动或转换文件时非常方便。



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661858169731666185816168.png)

直接进行扩展,可以建立多个文件,或者是少些几个命令



同时花括号还宽裕使用{a..b},遵循笛卡尔乘积,使用{a...h}表示a到h

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661859079801666185907753.png)

```shell
convert image.{png,jpg}
# 会展开为
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# 会展开为
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# 也可以结合通配使用
mv *{.py,.sh} folder
# 会移动所有 *.py 和 *.sh 文件

mkdir foo bar

# 下面命令会创建foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h这些文件
touch {foo,bar}/{a..h}
touch foo/x bar/y
# 比较文件夹 foo 和 bar 中包含文件的不同
diff <(ls foo) <(ls bar)
# 输出
# < x
# ---
# > y
```

shell最开始是指定运行的文件位置

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16661863736751666186372698.png)





# 1.shell工具

## 1.1查询使用帮助

1. 使用-h
2. 或者直接man rm（man是Manuel





## 1.2查找文件

顾名思义就是使用使用find

```
find . -name src -type d 
#这是查找名称为src的文件夹，type可以分为d，f，f是文件
```



``` 
find . -path */test/*.py -type f
#这是查找路径
```



还有其他的参数 -exec就是执行命令，找到后删除



fd还可以使用





ctrl+r 也是可以进行查找使用的快捷键





# 2.课后练习



## 2.1ls命令



> 1. 阅读 [`man ls`](https://man7.org/linux/man-pages/man1/ls.1.html) ，然后使用`ls` 命令进行如下操作：
>
>    - 所有文件（包括隐藏文件）
>    - 文件打印以人类可以理解的格式输出 (例如，使用454M 而不是 454279954)
>    - 文件以最近访问顺序排序
>    - 以彩色文本显示输出结果
>
>    典型输出如下：
>
>    ```
>     -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
>     drwxr-xr-x   5 user group  160 Jan 14 09:53 .
>     -rw-r--r--   1 user group  514 Jan 14 06:42 bar
>     -rw-r--r--   1 user group 106M Jan 13 12:12 foo
>     drwx------+ 47 user group 1.5K Jan 12 18:08 ..
>    ```



1.直接使用ls -a





2.搜索打印就是-h



![](https://files.catbox.moe/kuxnto.png)





3.直接搜索time

![](https://files.catbox.moe/a0vaqn.png)



4.直接搜索color

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16662495226761666249521717.png)





## 2.2shell函数

> 1. 编写两个bash函数 `marco` 和 `polo` 执行下面的操作。 每当你执行 `marco` 时，当前的工作目录应当以某种形式保存，当执行 `polo` 时，无论现在处在什么目录下，都应当 `cd` 回到当时执行 `marco` 的目录。 为了方便debug，你可以把代码写在单独的文件 `marco.sh` 中，并通过 `source marco.sh`命令，（重新）加载函数。



```shell
i
 #这里忘了加开始解释的地址
 marco(){
     echo "$(pwd)" > $HOME/marco_history.log
     echo "save pwd $(pwd)"
 }
 polo(){
     cd "$(cat "$HOME/marco_history.log")"
 }
```





## 2.3错误检验

> 假设您有一个命令，它很少出错。因此为了在出错时能够对其进行调试，需要花费大量的时间重现错误并捕获输出。 编写一段bash脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。 加分项：报告脚本在失败前共运行了多少次。
>
> ```shell
>  #!/usr/bin/env bash
> 
>  n=$(( RANDOM % 100 ))
> 
>  if [[ n -eq 42 ]]; then
>      echo "Something went wrong"
>      >&2 echo "The error was using magic numbers"
>      exit 1
>  fi
> 
>  echo "Everything went according to plan"
> ```





```shell
count=1

 while true
 do
     ./buggy.sh 2> out.log
     if [[ $? -ne 0 ]]; then
         echo "failed after $count times"
         cat out.log
         break
     fi
     ((count++))

 done
```

