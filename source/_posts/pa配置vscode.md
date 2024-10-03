---
title: pa配置vscode
date: 2022-11-18 20:59:12
tags: [pa,nemu,vscode]
---

# pa配置vscode

## 1.为什么会写这一个

目前正在写nju pa。pa是用makefile进行构建的，然后使用了多个自定义宏，导致，无法进行，直接的和之前一样，按一下run就可以运行。所以得进行手动配置。

还有就是，没有使用ide进行编码，导致一堆{}对齐的问题，然后找bug比较难受

## 2.配置方法

根据jyy[的课程](https://www.bilibili.com/video/BV1qa4y1j7xk?p=4&vd_source=d043013c6aa637931d747eae4c52b842)在1小时16分钟左右的时候，只需要进行vscode配置一会儿就可以正常使用。把defines里面进行添加宏就可以，这个宏如何查找，在第三届讲makefile的时候介绍过。我们使用make -nB(可以进行强制编译所有的)然后我们使用vim

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16687772080711668777207848.png)

```shell
make -nB \
 | grep -ve '^\(\#\|echo\|mkdir\)' \
 | vim -
```

然后进行搜索 -D ，这个参数就是在编译里面加入自定义宏，然后我们把这些宏进行拷贝过来，使用管道连接，去除echo 还有mkidr，传递给vim，vim用搜索按钮查找就可以



## 3.vscode配置教程

参考教程：https://zhuanlan.zhihu.com/p/87864677

1. 安装c++插件
2. 安装远程开发（我是用的虚拟机，进行ssh开发）
3. 之后就是通过ssh连接虚拟机，然后插件安装后，进行设置
4. 进行配置c_cpp_properties.json，在defines进行添加宏（无人其他改动）
5. **接下来就是配置gdb**



```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "__GUESR_ISA__=riscv64",
                "_GNU_SOURCE",
                "__STDC_CONSTANT_MACROS",
                "__STDC_FORMAT_MACROS",
                "__STDC_LIMIT_MACROS"
            ],
            "compilerPath": "/usr/bin/clang",
            "cStandard": "c17",
            "cppStandard": "c++14",
            "intelliSenseMode": "linux-clang-x64"
        }
    ],
    "version": 4
}
```

gdb配置有坑：

参考下面 ： https://www.youtube.com/watch?v=9VpiGwp8Vos&ab_channel=SavvyNik

我们要进行配置gdb，就得要在编译过程加入- g ，这个可以在makefile发现是在build.mk里面的，我们加入-g选项。然后还得设置launch.json

参考如下：https://blog.csdn.net/zztiger123/article/details/105544640

修改program为build下面那个输出的，然后cwd，修改为nemu的目录



```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {"name": "(gdb) Launch",
        "type": "cppdbg",
        "request": "launch",
        "program": "${workspaceRoot}/nemu/build/riscv64-nemu-interpreter",
        "args": ["-b"],
        "stopAtConnect": false,
        "cwd": "${workspaceRoot}/nemu",
        "environment": [],
        "externalConsole": false,
        "MIMode": "gdb",
        "setupCommands": [
            {
                "description": "为 gdb 启用整齐打印",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            },
            {
                "description":  "将反汇编风格设置为 Intel",
                "text": "-gdb-set disassembly-flavor intel",
                "ignoreFailures": true
            }
        ],
        "miDebuggerPath": "/usr/bin/gdb", 
    }
    ]
}
```

这个结束之后，有个坑，jyy在is_mode_batch这个函数设置为true，我用gdb就追踪不了，我输入p命令的结果，把那个函数值设置为false就可以走正常流程，进行追踪。



```json
{
    "tasks": [
        {
            "label": "make",
            "type": "shell",
            "command": "make -j8",
            "problemMatcher": [],
            "group": "build"
        },
        {
            "type": "cppbuild",
            "label": "C/C++: gcc 生成活动文件",
            "command": "/usr/bin/gcc",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```



## 4.clion应该也一样

明天有空在进行配置clion，修改自定义make目标，然后进行添加，可以参考下面的连接

https://www.jetbrains.com/help/clion/custom-build-targets.html#custom-rundebug



## 5.总结

pa确实难，对于我这个非科班跨考的。不过感觉也学到挺多知识的，好的大学，课程真的可以和没过高校的课程差不多，现在才做完1.2，还得接着写。
