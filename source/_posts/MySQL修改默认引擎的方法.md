---
title: MySQL修改默认引擎的方法
comments: true
toc: true
date: 2018-08-15 15:06:51
categories:
- DataBase
tags:
- MySQL 数据库引擎
---
MySQL数据库自5.5版本开始，InnoDB是MySQL数据库的默认引擎（之前是MyISAM）<!--more-->

### 查看引擎show engines
![](https://pic1.zhimg.com/80/v2-459f4aa9d09d1fade4a2424ab939ef55_hd.jpg)


### 主要区别
1. InnoDB支持外键和事务，MyISAM不支持外键和事务
对于InnoDB每一条SQL语句都默认封装成事务，自带提交。
2. InnoDB是聚焦索引，MyISAM是非聚焦索引。
InnoDB是聚集索引，数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而MyISAM是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
3. InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快；
4. InnoDB不支持全文索引，而MyISAM支持全文索引，查询效率上MyISAM要高

### 修改默认引擎方法
那么就需要根据具体的情况进行选择使用哪个引擎，那么就看看如何修改默认引擎：
#### 切换路径：

```
cd /etc/mysql/
```

#### 修改my.cnf文件：

1. 在my.cnf文件的头部添加"[mysqld]"
2. 添加 "defaulr-storage-engine = 数据库引擎名（INNODB/MYISAM）"

![](https://pic2.zhimg.com/80/v2-0c7840a01f8528f637016beb71524467_hd.jpg)

#### 重启MySQL服务
```
sudo service mysql restart
```
默认引擎修改完毕

![](https://pic2.zhimg.com/80/v2-0a65567f86cb6ea008f475d221ec4822_hd.jpg)
