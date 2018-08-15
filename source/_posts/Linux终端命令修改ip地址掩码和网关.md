---
title: Linux终端命令修改ip地址掩码和网关
comments: true
toc: true
categories:
- Linux
date: 2018-08-15 16:45:23
tags:
- IP掩码网关修改
---
在终端修改指定网卡的ip地址、掩码和网关<!--more-->
### 查看网卡
终端输入 ifcofig 并回车，查看需要修改的网卡名称

![](https://pic3.zhimg.com/80/v2-c5f733c334f371ed7eb56f1d2d971728_hd.jpg)

### 修改ip和子网掩码
```
sudo ifconfig 网卡名 ip信息 netmask 掩码
ifconfig  #查看是否生效
```

![](https://pic2.zhimg.com/80/v2-c4226f2ee33033bd4cb9e81c70778c26_hd.jpg)


### 修改网关的方法
原来的网关

![](https://pic1.zhimg.com/80/v2-a49d6130299ffb6d671291861e5cd12a_hd.jpg)

使用route命令修改网卡的网关信息
```
sudo route add default gw 网关信息 网卡名
route -n # 查看网关修改是否生效
```
![](https://pic3.zhimg.com/80/v2-f3903eb8dad876abd3f472a5d3489b87_hd.jpg)
