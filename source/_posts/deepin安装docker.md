---
title: deepin安装docker和pytorch
date: 2023-06-01 17:28:58
tags: [linux, torch,docker]
---



# deepin安装docker和pytorch

总体的流程图大致如下，首先是安装linux，这个直接跳过，接下来就是安装docker，之后，安装docker之后，安装pytorch image，然后使用vscode来进行深度学习开发。这样。不需要每次都要进行配置环境，直接使用这个镜像，构建多个容器，可以互不影响。



![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/1685611681945deepin%E5%AE%89%E8%A3%85docker.png)



## 1.docker相关设置

### 1.docker安装

> 前情提示，由于dockerhub，最近几天已经被gfw给404了，建议首先安装好了就来进行更换镜像。

由于deepin是Debian的分支，因此我们选择Debian的安装命令，详细教程参考[Debian Docker 安装 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/debian-docker-install.html)

主要命令就是

```shell
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
```

**注意，安装完成之后，请务必换源**

[Docker 镜像加速 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-mirror-acceleration.html)

主要是在下面进行在  /etc/docker/daemon.json 下面增加源，这个地方也可以修改，image默认下载的地方





```shell
{"registry-mirrors":["https://reg-mirror.qiniu.com/"]}
```

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

之后执行重启。







### 1.2加入到docker用户组

之后，因为使用docker默认是需要特权docker组里的成员才能使用，所以我们需要把当前用户加入到docker组里面



主要参考的是这个博文，首先查看是不是已经有了docker用户组，已经有了我们直接进行加入当前用户，然后进行重启。

```
sudo cat /etc/group | grep docker

sudo usermod -aG docker 要添加的user

sudo chmod a+rw /var/run/docker.sock

sudo systemctl restart docker

```



### 1.3附加内容（配置自己的images）

每次换到一个新电脑的时候，最麻烦的事情就是进行环境配置，下载所需要的安装软件，一个个的执行。现在有了docker，我们可以直接去dockerhub你输入命令来得到所需要的环境。例如java开发的环境（mysql，MongoDB和redis等）。

参考下面的教程：

[Docker搭建Java开发环境 (chennn.com)](https://blog.chennn.com/docker-build-java-development-environment/)

现在我要构建一个属于自己的开发环境，例如只需要java和maven，那么我们改如何得到呢。现在，这里我们需要使用的是dockerfile这个文件，通过这个文件看我们可以构建出来哦自己的images。

。dockerfile语法如下 ：[Docker Dockerfile | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-dockerfile.html)

首先需要定义一个from，这个就相当于原始文件，我们首先定义from openjdk，来作为自己的原始image，之后根据自己的需要来进行修改。之后，我们需要使用maven，这里我们选择编译安装。分为下面几步。

1. 下载maven源码
2. 进行解压
3. 然后进行移动到local目录里面
4. 设置环境变量

我们使用run 来执行每一个shell 命令，



之后就是设置环境变量，因为gfw的原因，有时候，我们需要使用代理，还有吧上面的源码放入到env上面，所以需要使用或者其他端口隐射到envv来进行设置。

```shell
FROM openjdk:8

RUN wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.zip
RUN unzip apache-maven-3.8.6-bin.zip
RUN mv apache-maven-3.8.6 /usr/local/maven
RUN rm -rf apache-maven-3.8.6-bin.zip

ENV MAVEN_HOME=/usr/local/maven
ENV PATH=$MAVEN_HOME/bin:$PATH
```

这样，我们就构建好了自己的开发包。

上面的代码还可以精简，我们只需要使用一个run ，剩下的全部用&&连接shell命令就可以





然后 我们进行构建，使用这个dockerfile

```shell
docker build -t java-env:8 .
```

最后我们就是启动容器

```shell
docker run -itd --name java-base-env java-env:8
```

如果需要进入到容器内部，我们可以使用docker exec -it 容器名称

------

> 扩展：使用gateway来进行远程开发。我们可以在images里面安装openssherver，然后修改、etc/ssh/sshd_config里面的允许root登录还有密码登录，之后，设置这个为自动启动。



参考下面这个连接：[VSCode+Docker: 打造最舒适的深度学习环境 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/80099904)



之后，我们在docker启动的时候设置端口映射这个容器，吧9001或者其他端口映射到22，然后使用gateway进行远程连接，输入端口9001，账户密码，就可以进行开发了。







## 2.deepin配置pytorch镜像

### 2.1deepin的设置

首先，我们需要对deepin进行设置，默认没有安装cuda，我们直接使用deepin官方的命令，安装cuda-toolkit，还有nvidia-smi进行apt安装。之后就完成了。有些版本是需屏蔽自己显卡，然后才能安装，笔者目前没有与到，安装比较简单。命令都在开始的思维导图上面。

### 2.2（可选）配置proxychains

来进行国内访问加速，直接apt 进行安装就行，然后修改配置文件 /etc/proxychains.conf。

参考这篇文章 [linux命令行代理神器-proxychains - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/166375631)



在.bashrc里面设置alias pr =proxychains，进行缩短单词



### 2.3安装pytorch

直接根据沐神的github推荐的image来使用

[mli/transformers-benchmarks: real Transformer TeraFLOPS on various GPUs (github.com)](https://github.com/mli/transformers-benchmarks)

```
sudo docker run --gpus all -it --rm -p 8888:8888 -v ~:/workspace \
	--ipc=host --ulimit memlock=-1 --ulimit stack=67108864 \
	nvcr.io/nvidia/pytorch:22.07-py3
```

这样我们直接进入到容器里面了。我们使用screen 吧这个容器放入到后台执行。

这样就安装完成。同事还把我们的home目录隐射到workdir里面，我们可以访问到workdir的文件来找到服务器的文件



## 3.vscode/idea/pycharm的使用

### 3.1vscode进行配置

我们首先进去插件商店，安装dev dontainer。之后使用远程连接到docker宿主机上 

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16856142119451685614211207.png)

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16856142399431685614239438.png)

在宿主机上面选择开发容器，我们就可以进入到刚刚的pytorch容器里面

### 3.2idea的设置

这里直接使用gateway来进行设置，选择新建一个ssh，我们输入之前映射到22的端口，还有账号密码（详情加1.3），然后进行连接，之后选择后端版本的idea，这样idea后端就部署到容器里面。之后，我们就可以进行远程及开发





### 3.2pycharm设置

对于pycharm，不需要上述那么复杂，我们直接进行远程开发，ssh到宿主机器的22端口，然后我们选择解释器interpret为docker里面的容器，就可以直接执行python代码，来进行调试开发了。
