---
title: 换电脑更新hexo
date: 2022-10-18 18:02:26
tags: [hexo,git,npm]
---





## 换电脑更新hexo



> 因为更换电脑之后，原始数据在之前的电脑，所以博客一直没有更新。最近为了，监督自己复盘，花费两天重新折腾 了一下博客。本来想着把博客迁移到hugo，结果hugo，更新的教程还没有hexo教程多。而且我这个github.io的域名已经使用了。如果更新hugo。就得把这个仓库的数据清空。（主要是折腾两天了，githubpage使用action自动更新还比较繁琐）。所以才换回hexo。下面我来复盘一下这两天的折腾过程。



## 1.更换hugo



主要参考的是这个up的教程。

[Hugo + GitHub Action，搭建你的博客自动发布系统 · Pseudoyu](https://www.pseudoyu.com/zh/2022/05/29/deploy_your_blog_using_hugo_and_github_action/)



### 1.1hugo本地搭建

本地搭建参照这个up没有任何问题，主要是要把下载的主题里面的example文件复制下来，然后更新**config。toml**文件，注意这里的坑点（需要解析自己baseurl为自己github的地址）不然搭建之后直接跳转到example的网站，让我以为是自己搭建问题，折腾了几个小时。然后就没有什么坑点了



### 1.2域名购买

为了贯彻讲白嫖进行到底的思想，我本来想着是使用freenom进行搭建一个 域名。但是一直购买失败。然后我又突然想起来，我之前认证了github学生包了的，所以直接去github学生包找到了一个提供免费域名的公司。顶级域名是.tech.一年的白嫖时间。博主使用的cloudflare，这个也不错，之前有过部署服务器的经验。所以我吧购买的域名，直接把dns改到cloudflare里的域名了。（可能等了一个小时才有结果）



域名设置到cloudflare后，需要进行cname设置（cname的意思就是重新跳转，例如输入世界一流大学.com直接跳转到sdu.edu.cn）



上述过程还好，没有花费多少时间。主要下面的设置自定义域名



### 1.3设置自定义域名

因为我的原始github.io的域名已经设置成为hexo的网站了，不能再次使用。所以我就参考了别人的方法，不能够有多个github.io的网站，但是可以在其他仓库里面开启一个仓库，然后使用github .io/projectname 来进行构成一个新的网站。所以哦去setting里面打开了一个新的page，然后此时还有一个坑点（**仓库默认的是main分支，我之后按照这个up的都是master分支，所以会出现几次配置失败的情况）**我删除重建了几个仓库才解决这个问题。



在setting进行悬着page，然后悬着page，在page里选择master分支，使用githubbot进行更新网页。



下面是设置自定义域名的坑，因为设置自定义域名，会**新建一个cname文件**，造成本地与远程版本内容不一样，所以我之前使用push操作都是push失败，后来强制push，造成cname丢失，无法通过自定义域名访问到网页。这个卡了一晚上。





### 1.4自定义action的坑点

因为使用自定义action的话，是需要设置github workflows的，我直接新建一个仓库，进行设置了workflows，结果

就是push的时候本地雨远程还是有 差异。而且他这个是需要把本地hugo的文件夹全部上传。（原文没些清楚，我以为只需要一个workflows工作流），结果也是看了一上午，触发任务失败。然后痛定思痛还是换成hexo了c





### 1.5总结错误

目前犯了下列几个错误

1. git操作不熟悉（remote branch，checkout，push -f）不知道怎么查看分支，如何删除，还有切换分支，分支管理
2. 自定义域名，这个大坑
3. 主要是设置config.toml文件没有进行设置，一直跳转到example网页，心态崩了几次
4. 之后就是主题设置问题，theme
5. 还有就是两个仓库，一个仓库上传public内容，一个进行上传所有的文件
6. 觉得没有时间然后换回来hexo了



## 2.hexo进行重装

[hexo博客迁移到另一台电脑 | wj'blog (wungjyan.github.io)](https://wungjyan.github.io/2018/08/17/move-hexo/)

复制下面的文件，就相当于在hexo分支里面的文件

```
_config.yml
package.json
scaffolds/
source/
themes/
```



```
// mac环境
sudo npm install -g hexo
npm install
npm install hexo-deployer-git --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
hexo g
hexo s

```

说起来也是比较简单，就是直接git下来所有的文件，但是也有小坑

git版本问题，因为，我是最新的npm，但是hexo的版本还是3.6的，结果就造成版本不同。按照上述的直接进行重装，就会造成安装版本不兼容，直接造成无法推送到服务端。但是本地没有问题。



所以接下来就是进行更新npm的不兼容的hexo版本





主要参考的是下面的

[Hexo版本升级指南 | novnan's notes](https://novnan.github.io/Hexo/update_hexo/)



```shell
//以下指令均在Hexo目录下操作，先定位到Hexo目录
//查看当前版本，判断是否需要升级
> hexo version

//全局升级hexo-cli
> npm i hexo-cli -g

//再次查看版本，看hexo-cli是否升级成功
> hexo version

//安装npm-check，若已安装可以跳过
> npm install -g npm-check

//检查系统插件是否需要升级
> npm-check

//安装npm-upgrade，若已安装可以跳过
> npm install -g npm-upgrade

//更新package.json
> npm-upgrade

//更新全局插件
> npm update -g

//更新系统插件
> npm update --save

//再次查看版本，判断是否升级成功
> hexo version

```

需要我们自己安装npm-check工具，然后就是网络原因，不提了（直接使用clash打开cmd）



然后更新也有可能会有问题，于是就进行重启了。



之后hexo就成功的到hexo



## 3.更换hexo主题

由于之前一直是使用yiliya主题，这次换的是butterfly主题，然后新的主题也有坑的，需要在原始的config里面进行更换主题轻微butterfly。同事还得继续复制自带的config文件，到跟目录，进行重命名，修改文件的信息，才可以更换主题细节。具体还有其他细节还在摸索，是参考下面这个的。有时间再继续折腾一下。

[Butterfly主题美化日记 | Akilarの糖果屋](https://akilar.top/posts/f99b208/)





## 4.复盘



失败原因，还有一段时间尝试使用gitee，结果gitee使用网页，需要认证身份证。弄了半天，我就跑路了，耽误时间。



1. 不想丢失以前写的hexo数据
2. 然后使用自定义域名不熟悉
3. 不熟悉使用git操作
4. 好久没使用相关前段工具
5. 折腾太少了，应该及时止损
6. b站有非常详细的教程
