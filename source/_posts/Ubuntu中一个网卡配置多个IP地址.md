---
title: Ubuntu中一个网卡配置多个IP地址
comments: true
date: 2018-08-16 17:36:56
categories:
- Linux
tags:
- IP
toc: true

---
一个网卡配置多个IP地址,也是分临时和永久<!--more-->
### 临时配置
重启后会失效
```
sudo ifconfig ens39:0 192.168.29.155 netmask 255.255.255.0 broadcast 192.168.29.255 up
sudo ifconfig ens39:1 192.168.29.165 netmask 255.255.255.0 broadcast 192.168.29.255 up
```
![](https://pic3.zhimg.com/80/v2-a9fa7715ed9b203fbeb9087cf6ae39d6_hd.jpg)

### 永久的配置
把配置信息写到/etc/network/interfaces 文件里，可以实现永久修改
重启服务或者重启电脑后都不会失效
```

auto ens39:0
iface ens39:0 inet static
address 192.168.29.155
netmask 255.255.255.0
gateway 192.168.43.1


auto ens39:1
iface ens39:1 inet static
address 192.168.29.165
netmask 255.255.255.0
gateway 192.168.43.1
```

![](https://pic1.zhimg.com/80/v2-3d512ec7f0e4137dcb61a828025287bc_hd.jpg)
