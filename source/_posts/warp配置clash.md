---
title: warp配置clash
date: 2023-07-09 11:11:38
tags: [warp, cf, clash]
---

# 什么是WARP

WARP是cloudflare公司推出的可以用来保护使用者隐私的一款服务，对于经常使用WARP来解锁vps流媒体的人再熟悉不过了。

WARP基于wireguard协议，使用UDP来传输数据，也就意味着在公网中的高QOS，但是WARP的ip相对比较干净，对外访问网络的出口 IP 被很多网站视为真实用户，可以用来解锁流媒体，谷歌学术等。

本文是基于作者的粗略了解成文，故很多地方可能会出现些许错误，若出现错误，还望各位海涵，并发邮件通知我，我会尽快修改。



# 如何使用

直接去官网1.1.1.1进行下载，有安卓还有windows客户端。



# 获取24PB流量

打开Telegram机器人，按要求输入命令进行获取warp+ unlimited license key Wrap+ Bot：https://t.me/generatewarpplusbot

![96080-fuatm3pq65e.png](https://lykqq.com/usr/uploads/2023/04/518483876.png)

更换密钥到客户端

![91479-g9o0s6b1mdl.png](https://lykqq.com/usr/uploads/2023/04/3541976739.png)

![77465-masdybx7iq.png](https://lykqq.com/usr/uploads/2023/04/2513971098.png)





# windows连接

windows连接需要使用clash进行前置连接登录，设置clash为tun模式，然后打开warp。



# 提取warp配置

使用github上的wgcf这个来进行获取配置。warp本质上是基于wireguard协议来进行生成的。因此我们完全可以不需要使用客户端，直接使用支持wireguard协议的第三方客户端来进行使用。



1.下载wgcf

```shell
wget -O wgcf https://github.com/ViRb3/wgcf/releases/download/v2.1.4/wgcf_2.1.4_linux_amd64
chmod +x wgcf
```

2、注册 WARP 账户

```bash
./wgcf register
```

在root文件夹下就会出现/root/wgcf-account.toml文件，里面是wgcf申请的免费账户信息，**你可以将license_key替换成你自己的warp+ key，获取方法打开App右上角菜单 三 --> 账户 --> 按键，复制替换即可。**

```bash
./wgcf update
```

在这一步。我们可以把我们24pb流量的key进行更换，然后再进行升级

3生成Wire­Guard配置文件

```bash
./wgcf generate
```

在root文件夹下就会出现/root/wgcf-profile.conf文件，这就是wgcf生成的Wire­Guard配置文件，可以导入到任何一个支持Wire­Guard协议的软件中使用。





（可选）4.优选ip

```shell
wget -N https://gitlab.com/Misaka-blog/warp-script/-/raw/main/files/warp-yxip/warp-yxip.sh && bash warp-yxip.sh
```

获取优选ip，得到更好的连接体验，更换endpoint为刚刚获取的优选ip还有端口，在wgcf的配置文件里面

![20230328001632](https://github.com/getsomecat/GetSomeCats/raw/Surge/%E4%BC%98%E9%80%89WARP%E7%9A%84EndPoint%20IP%EF%BC%8C%E6%8F%90%E9%AB%98%E6%9C%AC%E5%9C%B0WARP%E8%8A%82%E7%82%B9%E8%AE%BF%E9%97%AE%E6%80%A7%E3%80%81%E4%BF%AE%E6%94%B9%E5%AE%98%E6%96%B9%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84EndPoint%20IP%E4%BB%A5%E5%8F%8A%E8%A7%A3%E9%94%81ChatGPT.assets/20230328001632.png)

![20230328001706](https://github.com/getsomecat/GetSomeCats/raw/Surge/%E4%BC%98%E9%80%89WARP%E7%9A%84EndPoint%20IP%EF%BC%8C%E6%8F%90%E9%AB%98%E6%9C%AC%E5%9C%B0WARP%E8%8A%82%E7%82%B9%E8%AE%BF%E9%97%AE%E6%80%A7%E3%80%81%E4%BF%AE%E6%94%B9%E5%AE%98%E6%96%B9%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84EndPoint%20IP%E4%BB%A5%E5%8F%8A%E8%A7%A3%E9%94%81ChatGPT.assets/20230328001706.png)

![20230312172834](https://github.com/getsomecat/GetSomeCats/raw/Surge/%E4%BC%98%E9%80%89WARP%E7%9A%84EndPoint%20IP%EF%BC%8C%E6%8F%90%E9%AB%98%E6%9C%AC%E5%9C%B0WARP%E8%8A%82%E7%82%B9%E8%AE%BF%E9%97%AE%E6%80%A7%E3%80%81%E4%BF%AE%E6%94%B9%E5%AE%98%E6%96%B9%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84EndPoint%20IP%E4%BB%A5%E5%8F%8A%E8%A7%A3%E9%94%81ChatGPT.assets/20230312172834.png)





# clash配置wireguard

参照官方的clash配置文件写法

```yaml
proxies:
  - name: "wg"
    type: wireguard
    server: myserver.dyn.org #aka peer endpoint
    port: 51820 #aka peer endpoint port
    ip: 10.90.0.2 #aka self ip address or address
    # ipv6: your_ipv6 #aka self ipv6 address or address
    private-key: ANtlGLhcIXthucHavUPT0q8AOmHu+6+smP4Vfh3M2W8=
    public-key: RPoEJdhT+kGH07EnIrwmKrbAdPlRqRwsrzgx4VAYGy0=
    # preshared-key: 66l8JUDdpHBBSKezlDVwNKyaNaEm+xZ+3/1JzXNJ39k=
    # dns: [1.1.1.1, 8.8.8.8]
    # mtu: 1420
    udp: true
    remote-dns-resolve: true #start support from Premium 2023.02.16
```

我们需要设置的只有公钥私钥，还有ip地址

这些文件，直接按照提取的参考config来进行配置，server可以设置成之前得到的优选ip



# （可选）clash设置relay，进行链式配置（得到稳定台湾ip）

1.设置代理组，参照官方clash文件

```yaml
Proxy Groups:
- name: "relay"
  type: relay
  proxies:
    - select
    - vmess1
    - ss1
    - ss2
 #disable-udp: true
```

类型是relay。

原理：clash首先会通过垃圾机场，访问warp，warp得到ip是基于访问ip的来进行生成附近的ip。所以我们可以用垃圾机场的ip得到稳定优质台湾ip。



接下来，我们还需要进行配置策略组，让访问网站走relay代理组

![img](https://docs.cfw.lbyczf.com/assets/img/ui-profiles-rules1.d499c4e2.png)

只需要在这里进行添加match，选择relay，就可以走链式代理

- MATCH：全匹配

![16888739339471688873933188.png](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16888739339471688873933188.png)



这样所有的匹配就会走clash的链式代理组。





安卓使用sagernet，手动设置代理链就可以使用
