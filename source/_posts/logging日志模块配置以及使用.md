---
title: logging日志模块配置以及使用
comments: true
toc: true
categories:
- Python
date: 2018-08-02 14:02:03
tags:
- logging 日志
---
在项目开发中，调试是必不可少的，logging模块为我们调试提供了极大的便利。<!--more-->
### logging模块的简单配置
#### 日志等级
logging提供了5个日志等级，利用不同的日志函数，消息可以按某个等级记入日志

|级别|日志函数|描述|
|-----|----|----|
|DEBUG|logging.debug()|最低级别。用于小细节。通常只有在诊断问题时，你才会关心这些消息|
|INFO|logging.info()|用于记录程序中一般事件的信息，或确认一切工作正常|
|WARNNING|logging.warning()|用于表示可能的问题，它不会阻止程序的工作，但将来可能会|
|ERROR|logging.error()|用于记录错误，它导致程序做某事失败|
|CRITICAL|logging.critical()|最高级别。用于表示致命的错误，它导致或将要导致程序完全停止工作|


```
import logging
# logging初始化
logging.basicConfig()
```
#### 创建logging实例对象
```
logger = logging.getLogger()
```
#### 设置级别
```
logger.setLevel(logging.DEBUG)
```
#### 指定logger实例对象的handler
在日志文件达到指定的大小后将清空原来的日志文件，比如log.1设置了1M，满1M后将新建log.2进行记录日志，如此循环
```
from logging import handlers
# 创建handler对象，指定日志文件位置以及大小，日志份数
handler = handlers.RotatingFileHandler("logs/log", maxBytes=1024*50, backupCount=5)
# 指定handler对象的日志输出格式
handler.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(lineno)s - %(message)s"))
```
#### logger实例对象添加handler
```
logger.addHandler(handler)
```
#### 完整代码
```
import logging
from logging import handlers
# logging初始化
logging.basicConfig()
# 创建logger实例对象
logger = logging.getLogger()
# 设置logger对象调试级别
logger.setLevel(logging.DEBUG)
# 为logger对象创建handler对象，指定日志文件位置以及大小，日志份数
handler = handlers.RotatingFileHandler("logs/log", maxBytes=1024*50, backupCount=5)
# 指定handler对象的日志输出格式
handler.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(lineno)s - %(message)s"))
# 把handler绑定到logger对象中
logger.addHandler(handler)
```
### logger日志使用
现在我们就可以把需要的日志信息通过调用logging提供的函数写入到日志文件中
```
n = 0
try:
    print(10 / n)
except Exception as e:
    print('e:',e)
    logging.error(e)
logger.error("发生错误")

```
查看日志文件
```
2018-08-02 13:54:30,576 - root - ERROR - 24 - division by zero
2018-08-02 13:54:30,576 - root - ERROR - 25 - 发生错误

```
