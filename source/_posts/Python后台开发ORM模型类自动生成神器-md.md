---
title: Python后台开发ORM模型类自动生成神器
comments: true
date: 2018-11-20 20:54:59
tags:
- 自动生成Model模型类
properties:
- Python
---
今天介绍一个后台开发神器，很适合当我们数据库中已存在了这些表，然后你想得到它们的model类使用ORM技术进行CRUD操作(或者我根本就不知道怎么写modle类，但我会写create这个表的sql的时候)，手写100张表的model类？这是。。。。。。。。。
是不可能的，这辈子都不可能的。
因为我们有**sqlacodegen神器**，
**一行命令获取数据库所有表的模型类**。
<!--more-->
### 应用场景
1、后台开发中，需要经常对数据库进行CRUD操作；
2、这个过程中，我们就经常借助ORM技术进行便利的CURD，比如成熟的SQLAlchemy；
3、**但是，进行ORM操作前需要提供和table对应的模型类；**
4、并且，很多历史table已经存在于数据库中；
5、如果有**几百张table**呢？还自己一个个去写吗？
6、我相信你心中会有个念头。。。
### 福音
还是那句话，Python大法好。
这里就提供了一个根据已有数据库(表)结构生成对应SQLAlchemy模型类的神器：
**sqlacodegen**
> This is a tool that reads the structure of an existing database and generates the appropriate SQLAlchemy model code, using the declarative style if possible.

安装方法：
```
pip install sqlacodegen
```
### 快快使用
使用方法也很简单，只需要在终端(命令行窗口)运行一行命令即可：
常用数据库的使用方法：
将会获取到整个数据库的model
```
sqlacodegen postgresql:///some_local_db
sqlacodegen mysql+oursql://user:password@localhost/dbname
sqlacodegen sqlite:///database.db
```
查看具体参数可以输入：
```
sqlacodegen --help
```
参数含义：
```
optional arguments:
  -h, --help         show this help message and exit
  --version          print the version number and exit
  --schema SCHEMA    load tables from an alternate schema
  --tables TABLES    tables to process (comma-separated, default: all)
  --noviews          ignore views
  --noindexes        ignore indexes
  --noconstraints    ignore constraints
  --nojoined         don't autodetect joined table inheritance
  --noinflect        don't try to convert tables names to singular form
  --noclasses        don't generate classes, only tables
  --outfile OUTFILE  file to write output to (default: stdout)
```
目前我在postgresql的默认的postgres数据库中有个这样的表：
```
create table friends
(
  id   varchar(3) primary key ,
  address  varchar(50) not null ,
  name varchar(10) not null
);

create unique index name_address
on friends (name, address);
```
为了使用ORM进行操作，我需要获取它的modle类
**但唯一索引的model类怎么写呢？**
我们**借助sqlacodegen来自动生成**就好了
```
sqlacodegen postgresql://ridingroad:ridingroad@127.0.0.1:5432/postgres --outfile=models.py  --tables friends
```
### 模型类效果
查看输出到models.py的内容
```
# coding: utf-8
from sqlalchemy import Column, Index, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
metadata = Base.metadata


class Friend(Base):
    __tablename__ = 'friends'
    __table_args__ = (
        Index('name_address', 'name', 'address', unique=True),
    )

    id = Column(String(3), primary_key=True)
    address = Column(String(50), nullable=False)
    name = Column(String(10), nullable=False)

```
如果你有很多表，就直接指定数据库呗(这是会生成整个数据库的ORM模型类哦),不具体到每张表就好了，
后面就可以愉快的CRUD了，耶
### 注意事项
Why does it sometimes generate classes and sometimes Tables?
>Unless the --noclasses option is used, sqlacodegen tries to generate declarative model classes from each table. There are two circumstances in which a Table is generated instead:
1、the table has no primary key constraint (which is required by SQLAlchemy for every model class)
2、the table is an association table between two other tables

当你的表的字段缺少primary key或这张表是有两个外键约束的时候，会生成table而不是模型类了。比如，我那张表是这样的结构：
```
create table friends
(
  id   varchar(3) ,
  address  varchar(50) not null ,
  name varchar(10) not null
);

create unique index name_address
  on friends (name, address);
```
再执行同一个命令：
```
sqlacodegen postgresql://ridingroad:ridingroad@127.0.0.1:5432/postgres --outfile=models.py  --tables friends
```
获取到的是Table：
```
# coding: utf-8
from sqlalchemy import Column, Index, MetaData, String, Table

metadata = MetaData()


t_friends = Table(
    'friends', metadata,
    Column('id', String(3)),
    Column('address', String(50), nullable=False),
    Column('name', String(10), nullable=False),
    Index('name_address', 'name', 'address', unique=True)
)
```
其实和模型类差不多嘛，但是还是尽量带上primary key吧，免得手动修改成模型类
### Python全面学习资料
公众号"Python孙行者"后台回复”电子书“即可。如果你有什么好东西好神器，欢迎后台私信，以后总结随文分享，耶
![](https://mmbiz.qpic.cn/mmbiz_png/1ndlcPm7Ab4p0zUxJ9N2icqVOPm4KaibT1XzumWCK636mibdwmUZFMEMNNiaYnxYlZCibdeKdiaCRIpCmicEiadNticPgtQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
