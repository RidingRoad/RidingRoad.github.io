---
title: 修改Redis数据库个数的方法
comments: true
toc: true
categories:
- DataBase
date: 2018-08-15 16:04:57
tags:
Redis
---
Redis的数据库个数默认是16个（0-15），既然是默认，那么肯定可以修改<!--more-->
修改方法：
#### 备份配置文件
```
cd /etc/redis/
sudo cp redis.conf redis.conf.backup
```

#### 修改数据库个数
打开配置文件，修改databases 参数后面的个数即可

![](https://pic3.zhimg.com/80/v2-491be7313f144f15b4ca67e5da8213e7_hd.jpg)

#### 重启Redis数据库
重启数据库并查看数据库数量
```
sudo service redis restart
redis-cli
config get databases
```
![](https://pic4.zhimg.com/80/v2-8a8b12e2f44d6eb58bdf61f3205c9b60_hd.jpg)

[小知识点]Redis的hash槽的个数是2**14=16384个
