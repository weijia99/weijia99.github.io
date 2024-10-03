---
title: jvm小结
date: 2023-03-03 17:11:40
tags: [面试,jvm]
---



# 1.类文件结构详解

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/bg/desktop%E7%B1%BB%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84%E6%A6%82%E8%A7%88.png)

JVM只认识class文件所有的虚拟机都只支持字节码。在 Java 中，JVM 可以理解的代码就叫做`字节码`（即扩展名为 `.class` 的文件），它不面向任何特定的处理器，只面向虚拟机。而且JVM虚拟机

可以说`.class`文件是不同的语言在 Java 虚拟机之间的重要桥梁，同时也是支持 Java 跨平台很重要的一个原因。

所以 jvm是跨语言的平台。



## 1.2class类文件的结构

主要包括魔数，class的版本号，常量池，访问标志，当前值，字段表，方法表，睡醒吧IAO

```
ClassFile {
    u4             magic; //Class 文件的标志
    u2             minor_version;//Class 的小版本号
    u2             major_version;//Class 的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;//Class 的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//Class 文件的字段属性
    field_info     fields[fields_count];//一个类可以有多个字段
    u2             methods_count;//Class 文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}

```



### 1.2.1魔数

魔数就是进行判断验证，这个文件是不是能被jvm接受的



### 1.2.2版本号

首先是小版本号，接着就是大版本号



### 1.2.3常量池

常量池从1开始，到n-1。

常量池主要存放两大常量：字面量和符号引用。字面量比较接近于 Java 语言层面的的常量概念，如文本字符串、声明为 final 的常量值等。而符号引用则属于编译原理方面的概念。包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16780000496321678000048909.png)

1个word是tag，然后看不同的类型。不同的属性，有name还有length。以utf8位例子，首先是tage，接着2个word是length之后就是这些长度的byte。



###  1.2.4访问标志(Access Flags)

这个就是来进行判断方法是不是抽象类还是公开类，或者是函数是不是抽象的，这个只需要把正确的表示，进行想或

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/%E6%9F%A5%E7%9C%8B%E7%B1%BB%E7%9A%84%E8%AE%BF%E9%97%AE%E6%A0%87%E5%BF%97.png)



### 1.2.5 当前类（This Class）、父类（Super Class）、接口（Interfaces）索引集合

顺序的

```
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口

```

类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，由于 Java 语言的单继承，所以父类索引只有一个，除了 `java.lang.Object` 之外，所有的 java 类都有父类，因此除了 `java.lang.Object` 外，所有 Java 类的父类索引都不为 0。

接口索引集合用来描述这个类实现了那些接口，这些被实现的接口将按 `implements` (如果这个类本身是接口的话则是`extends`) 后的接口顺序从左到右排列在接口索引集合中

------





### 1.2.6字段表

描述的接口生命的变量。然后看他是不静态变量，类变量还是final修饰的

**access_flags:** 字段的作用域（`public` ,`private`,`protected`修饰符），是实例变量还是类变量（`static`修饰符）,可否被序列化（transient 修饰符）,可变性（final）,可见性（volatile 修饰符，是否强制从主内存读写）。

**name_index:** 对常量池的引用，表示的字段的名称；（ 从常量池检查到）

**descriptor_index:** 对常量池的引用，表示字段和方法的描述符；

**attributes_count:** 一个字段还会拥有一些额外的属性，attributes_count 存放属性的个数；

**attributes[attributes_count]:** 存放具体属性具体内容。





### 1.2.7方法表

和字段表一样 ，不过这个就是在方法上面了，还是访问表示，名称索引，描述

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/%E6%96%B9%E6%B3%95%E8%A1%A8%E7%9A%84%E7%BB%93%E6%9E%84.png)



### 1.2.8属性表

在 Class 文件，字段表，方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。与 Class 文件中其它的数据项目要求的顺序、长度和内容不同，属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写 入自己定义的属性信息，Java 虚拟机运行时会忽略掉它不认识的属性。

------



## 1.3虚拟机类加载机制

首先记住加载流程

加载-linking-初始化-使用

### 1.3.1加载

加载就是把磁盘上的内容存到内存上去

主动加载（包括new，invoke，静态变量，使用静态方法）



被动加载：初次之外都是被动



### 1.3.2加载过程

通过类的路径来获得类文件class，讲这个二进制转化俄日运行内存里面的数据结构，最后在内存生成一个对象，作为入口

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口





### 1.3.3链接-验证

验证代码文件是不是满足class文件的标准，还有语法验证，语义验证（本来是int类型，却是用long的方法），最后一个是符号验证（通常来讲，就是是不是访问了被进制的类，protect对象子类的）

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/%E9%AA%8C%E8%AF%81%E9%98%B6%E6%AE%B5.png)



### 1.3.4准备

准备阶段就是进行填入初始值，质包括被static修饰的变量，初始化为全0

**比如我们定义了`public static int value=111` ，那么 value 变量在准备阶段的初始值就是 0 而不是 111（初始化阶段才会赋值）。（初始化为0，之后才是进行赋值）**

**敞常量就是原始状态**

**特殊情况：比如给 value 变量加上了 final 关键字`public static final int value=111` ，那么准备阶段 value 的值就被赋值为 111。**





### 1.3.5解析

进行减符号引用，转化为内存地址

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作**主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符** 7 类符号引用进行。



符号引用就是一组符号来描述目标，可以是任何字面量。**直接引用**就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。在程序实际运行时，只有符号引用是不够的，举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用。

综上，解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，也就是得到类或者字段、方法在内存中的指针或者偏移量。



**符号引用（如类的全限定名）解析为直接引用（类在实际内存中的地址） **

------





### 1.3.6类初始化

就是执行clinit方法

在这个阶段把之前的static变量进行赋值，这个时候才会UI变成113，不是一开始就是113



有静态方法就是由clint







### 1.3.7常用的类加载器

一个最开始的bootstrap，一个ext，一个app，最低那个曾就是开始的bootstrap

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1. **BootstrapClassLoader(启动类加载器)** ：最顶层的加载类，由 C++实现，负责加载 `%JAVA_HOME%/lib`目录下的 jar 包和类或者被 `-Xbootclasspath`参数指定的路径中的所有类。
2. **ExtensionClassLoader(扩展类加载器)** ：主要负责加载 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的 jar 包。
3. **AppClassLoader(应用程序类加载器)** ：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。

自定义只需要实现loadclass，找到这个全限定的地址的二进制





### 1.3.8双亲委派

就是递归寻找，首先智商往下到达最顶层bootstrap，之后才是ext，最后才是app，看他的父类能不能处理，不能就自己进行加载

除了 `BootstrapClassLoader` 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`。如果我们要自定义自己的类加载器，很明显需要继承 `ClassLoader`。

------

