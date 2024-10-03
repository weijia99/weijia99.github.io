---
title: alist网盘聚合
date: 2023-02-04 18:59:27
tags: [linux,网盘]

---

# alist简介



**AList** 是一个支持在本地加载 18 款常见网盘文件列表的工具，它可以让用户直接在浏览器中预览文件、上传文件、下载文件、播放视频、浏览照片，也可以直接在播放器中打开视频文件，还可以配合 aria2 实现文件下载功能。跨平台简单易用。@[Appinn](https://www.appinn.com/alist-file-list-program/)

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频](https://www.appinn.com/wp-content/uploads/2022/09/Appinn-2022-09-16T150310.322.jpg)

感谢 @**imsoff** 推荐。

## AList – 文件列表程序

AList 自己的定位是一个支持多存储的文件列表程序，使用 Gin 和 Solidjs。它可以将以下网盘聚合在一起：

- 本地存储
- 阿里云盘
- OneDrive / Sharepoint（国际版, 世纪互联,de,us）
- 天翼云盘 (个人云, 家庭云)
- GoogleDrive
- 123云盘
- FTP / SFTP
- PikPak
- S3
- 又拍云对象存储
- WebDav(支持无API的OneDrive/SharePoint)
- Teambition（中国，国际）
- 分秒帧
- 和彩云 (个人云,家庭云)
- Yandex.Disk
- 百度网盘
- 夸克网盘
- 迅雷网盘

然后只需要在浏览器中打开，即可预览文件、上传文件、下载文件、直接播放视频、音频。

## 在线播放视频效果

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频 1](https://www.appinn.com/wp-content/uploads/2022/09/Screen-Appinn2022-09-16-15.59.14.jpg)

并且提供了几个播放器按钮，直接点击就能跳转。

不过青小蛙在测试的时候没搞定云字幕，似乎需要下载字幕之后才能加载。

## 安装 & 配置

AList 的安装比较简单，对于 Windows 用户，直接下载 .exe 文件之后，使用命令提示符运行，就行了。对于其他平台用户，也差不多，另外也支持 docker 部署，即开机用非常简单。

也算是0配置，只需要输入命令 alist.exe admin 获取管理员密码，打开浏览器 127.0.0.1:5244 就行了。

## 添加存储

在管理界面，可以添加很多的网盘：

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频 2](https://www.appinn.com/wp-content/uploads/2022/09/Screen-Appinn2022-09-16-16.30.03.jpg)

这里就需要[阅读文档](https://alist.nn.ci/zh/guide/drivers/common.html)了，但也很简单，最重要的是获取 Token，也提供了自动化获取按钮。不放心的同学可以根据文档说明手动获取。

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频 3](https://www.appinn.com/wp-content/uploads/2022/09/Screen-Appinn2022-09-16-16.31.09.jpg)

## 公网访问

虽然青小蛙主要用来自己在家用，但的确看到了不少人拿来在公网使用。AList 可以设置公告、用户权限、

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频 4](https://www.appinn.com/wp-content/uploads/2022/09/Screen-Appinn2022-09-16-16.39.19.jpg)

嗯，比如[这种](https://kutt.appinn.net/ftlxSu)。

## WebDAV

除了直接将 WebDAV 资源添加进 AList，还支持反向使用，即使用 WebDAV 客户端连接 Alist，然后浏览绑定的内容，这样就极大的扩展了客户端，包括 iPhone、Android 都可以轻松访问你的网盘资源了：

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频 5](https://www.appinn.com/wp-content/uploads/2022/09/Screen-Appinn2022-09-16-16.47.23.jpg)

![AList - 聚合阿里云盘、百度网盘、PikPak、WebDav 等 18 款网盘：文件预览、上传/下载，直接播放视频 6](https://www.appinn.com/wp-content/uploads/2022/09/Screen-Appinn2022-09-16-16.47.28.jpg)

怎么说呢，反正省了台 [NAS](https://www.appinn.com/tag/nas/) 的感觉。



# 2.为什么搭建

笔者手里有多个网盘，百度云（网课视频），阿里云（电视视频），pikpak（下torrent），还有谷歌云（自用，备份手机的）。

主要是笔者中午使用pikpak在线客户端很卡，播放视频非常卡顿，不流畅，所以笔者花了一点时间来进行搭建。搭建完成之后的效果就是可以只要输入一个网站，就可以访问多个网盘，而且播放效果，比官网好多 了。

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16755087318811675508731713.png)

搭建效果如图



# 3.搭建教程

主要参考下面官网文档，本来想使用docker的，但是垃圾服务器，内存根本跑不了docker，所以直接使用的一件脚本

```
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install

```

https://alist.nn.ci/zh/guide/install/script.html 官网教程



搭建完成之后，会出现账号密码，这个自己记住。



之后就是使用如何进行安装网盘。最简单的是pikpak，这个只要输入账号密码就行。具体安装步骤在官网上面。



百度网盘安装教程：

1. https://openapi.baidu.com/oauth/2.0/authorize?response_type=code&client_id=iYCeC9g08h5vuP9UqvPHKKSVrKFXGa1v&redirect_uri=https://tool.nn.ci/baidu/callback&scope=basic,netdisk&qrcode=1
2. 使用上面网站获取令牌还有secret还有id
3. 下面看图，不然没有办法正常播放视频
4. 下载选择非官方就行

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16755090388811675509038218.png)





谷歌网盘：

这是最复杂的，需要自己申请api接口，然后再把这个接口开放给自己一个人使用，得到接口之后才能进行生成id还有token，具体参考下面这个YouTube博主视频

https://youtu.be/Hktdyg7L0Bg



1. 创建项目
2. 然后启用api还有服务，主要使用云盘服务，找到云盘
3. 之后就是创建姘居，创建OA客户端id，填写外部，然后只要输入自己的电子邮件
4. 之后就是添加用户，吧自己添加进去
5. 之后再次回到品聚，创建oa的id，选择应用是桌面客户端之后秘钥还有id就出来了
6. 之后输入到那个网站，点击code，弹出网页，一直继续
7. 就会有token出现
8. 之后就按照放入到alist的后台





# 4.反向代理

下午使用github学生包，白嫖了一个域名，然后，为了懒得记住ip，直接用域名来使用。

问题是alist使用的5244端口，但是直接输入域名+端口非常不好看，所以使用反向代理，这里使用的是nginx，

直接下载nginx安装，笔者之前忘了开80端口，一直打不开，以为是自己配置问题。换了apache也是一样的。于是再换回nginx，才发现80没有打开，大坑。打开之后，一直是配置502，一直以为是自己的配置有问题（实际上没有，只是linux内核问题），谷歌发现要设置一个内核关闭，代码如下

```shell
setsebool -P httpd_can_network_connect 1   
```

之后就可以正常访问了，主要的是配置，参考官网，直接在http模块里面添加一个新的server模块，就可以，配置文档etc、nginx、nginxconf里面，修改如下

```c++
location / {
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_set_header Host $http_host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header Range $http_range;
	proxy_set_header If-Range $http_if_range;
  proxy_redirect off;
  proxy_pass http://127.0.0.1:5244;
  # the max size of file to upload
  client_max_body_size 20000m;
}

```

