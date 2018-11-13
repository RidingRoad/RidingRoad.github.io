---
title: Python实现UTC时间戳和格式化字符串互换
comments: true
date: 2018-11-13 21:32:25
tags:
- 时间戳
categories:
- Python
---
Python实现互转：时间戳和字符串
<!--more-->
后台开发经常需要对时间进行记录和返回时间戳给前端。
为了通用性，我们一般都会存储UTC的时间戳。
但是，一般对数据库进行时间戳字段的插入，往往是以**格式化的时间字符串进行插入**的，例如：
```
insert into table_name(user_id, login_time) values ("RidingRoad", "2018-11-13 12:19:39");
```
### 业务场景
假如前端传来了用户的某个UTC时间戳，后台需要录入数据库，当前端请求时候，返回这个UTC时间戳。
### 业务分析
这个UTC时间戳的状态变化：
1. 前端传过来的：double型的时间戳
2. 插入数据库时：格式化的时间字符串
3. 返回给前端的：double型的时间戳
所以我们需要一个对**时间和格式化的时间字符串进行转换**的函数
### 转换实现
```
import time

# UTC 转换为 格式化的时间字符串
def u2s(utc_timestamp):
    return time.strftime("%Y-%m-%d %H:%M:%S", time.gmtime(utc_timestamp))

# 格式化的时间字符串 转换为 UTC
def s2u(utc_time_string):
    return time.mktime(time.strptime(utc_time_string, "%Y-%m-%d %H:%M:%S"))\
           - time.mktime(time.gmtime(0))
```
### 效果展示
```
if __name__ == "__main__":
    # 取得当前的UTC时间戳
    timestamp = time.time()
    print(timestamp)

    # 把UTC时间戳转换为日期格式字符串
    utc_time_string = u2s(timestamp)
    print(utc_time_string)

    # 把日期字符串转换回UTC时间戳
    utc_timestamp = s2u(utc_time_string)
    print(utc_timestamp)

    # 把转换后的UTC时间戳转换成日期字符串
    # 看看误差，秒数也一样，一般不影响
    after_conver = u2s(utc_timestamp)
    print(after_conver)
```
### 结果输出
两个时间戳的误差在零点毫秒级，前后的日期格式字符串一样，一般的业务可以接受这样的误差。**有更精准的方法，下面评论告诉我吧。**
```
1542112115.95
2018-11-13 12:28:35
1542112115.0
2018-11-13 12:28:35
```
### Python全面学习资料
公众号“Python孙行者”后台回复“电子书“即可
![](https://mmbiz.qpic.cn/mmbiz_png/1ndlcPm7Ab4p0zUxJ9N2icqVOPm4KaibT1XzumWCK636mibdwmUZFMEMNNiaYnxYlZCibdeKdiaCRIpCmicEiadNticPgtQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

