---
title: Ubuntu临时和永久修改ip地址掩码网关
comments: true
toc: true
categories:
- Linux
date: 2018-08-15 16:45:23
tags:
- IP掩码网关修改
---
在终端修改ip有临时和永久修改两种方式<!--more-->
## 临时修改网卡ip地址
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
原来的网关信息
![原来网卡信息](https://pic1.zhimg.com/80/v2-a49d6130299ffb6d671291861e5cd12a_hd.jpg)

使用route命令修改网卡的网关信息
```
sudo route add default gw 网关信息 网卡名
route -n # 查看网关修改是否生效
```
![](https://pic3.zhimg.com/80/v2-f3903eb8dad876abd3f472a5d3489b87_hd.jpg)

## 永久行修改ip地址
### 修改配置文件
常用网络配置的文件有以下两个：
1. /etc/network/interfaces 设置ip等信息相关的配置文件
2. /etc/resolv.conf 设置DNS域名服务器的配置文件
我们只需要修改第一个文件即可，设置静态IP常用于桥接模式下的虚拟机和主机进行通讯，其他见下图：
![](https://pic3.zhimg.com/80/v2-3f33ec56f6516032c45ade4d0027ae52_hd.jpg)

### 重启网络服务
```
sudo /etc/init.d/networking restart
或
sudo service networking restart
```
设置ip地址信息就可以永久修改生效了。
