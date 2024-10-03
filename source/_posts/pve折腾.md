---
title: pve折腾
date: 2023-04-15 21:31:59
tags: [pve，虚拟机]
---



# pve折腾

自己的老电脑用了5年了，cpu还是4核心的，显卡是1050.硬盘还是1t的机械，都没多少剩余空间了。随着最近的硬盘内存大降价，我就凑巨资300买了一个1t的SATA固态，谁能想到，现在的nvme比SATA还便宜（我当初可是花费500元买的500g的nvme），然后硬盘前几天就到了。于是我就直接安装虚拟机了，想着在虚拟机你运行多个系统，目前已经完成的有openwrt，kali，deepin，安卓x86还有win11.但是win11没有进行显卡直通，其余都接近完美了。下面就对这几天的行为进行一次复盘。





## 1.pve安装

直接去官网进行下载，连接是选择https://pve.proxmox.com/wiki/Downloads，选择最新的版本就可以进行下载，然后安装的话使用ventoy，只需要把pve放入到u盘就可以。

https://zhuanlan.zhihu.com/p/510216630

可以参考上面的教程，只不过需要注意的是，要填写的ip,是需要和你的局域网在一个网段里面。我自己定义的网络是192.168.100.2，gate：192.168.100.1。



## 2.openwrt安装

固件是在这个大神的网站下载的，自动编译，还可以进行自定义插件。https://supes.top/

1. 首先把img上传到pve上面
2. 然后就是新建一个虚拟机，但是不要创建iso
3. 之后使用img2iso，记住我们的磁盘id，使用qm importdisk 100 /openwrt.img local-lvm
4. 然后再启动项，把这个磁盘进行加入。
5. 之后进行启动，设置网络ip vim etc

https://post.smzdm.com/p/a7nqp3r9/

参考上面的安装流程





**2.1op作为主路由**

在openwrt里面进行PPPoE拨号，那么op就是要进行桥接2个电脑的网口，一个作为lan，一个作为lan，wan口连接宽带，然后去进行pppoe拨号。



**2.2op作为胖路由**

只需要桥接一个网络口，然后连接主路由的时候，需要进行设置。进入接口里面的lan设置ip为静态ip，然后gate设置为192.168.100.1（主路由的ip），然后dns也是主路由的ip，同时关闭DHCP，这样就不会干扰DHCP。



主要的痛点就是看你要做旁路由还是主路由，主路由就是没有





## 3.deepin安装

deepin安装就是普通的linux安装流程，但是有一点就是安装pcie直通（把wifi）给deepin，无限网卡教程参考如下

https://www.orcy.net.cn/185.html

1. 修改grup，允许iommu
2. 更新grub
3. 重启
4. 之后增加模块在module里面
5. 然后就直接在虚拟机上添加pcie设备就行



## 4.primeos安装

目前测试了几个x86的系统在虚拟机上，都无法进行安装，只有三哥的这个才可以，选择的系统是classical版本，安卓7.0，然后也是正常的安装流程，要注意的是在安装的使用，你需要新建一个分区，然后再次写入。就可以了。然后功能都有，play商店也有。



## 5.win10/11的坑

主要是为了远程玩游戏，但是看知乎这篇文章说，原神，不让用虚拟机，然后显卡直通弄了半天也没有成功，可能是win11的问题吧，等有时间了换成win10 ltsc试试。主要参考如下的教程

https://zhuanlan.zhihu.com/p/571224296

这个大佬的目前还没有成功



https://www.youtube.com/watch?v=00GxKDGUhxA&ab_channel=VedioTalk

有关这个大佬得到是核显进行驱动



https://blog.csdn.net/Qwertyuiop2016/article/details/127940349

这个大佬是直接进行ltsc驱动的，不知道是不是我的版本原因，下次再试试吧
