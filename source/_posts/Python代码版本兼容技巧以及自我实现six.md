---
title: Python代码版本兼容技巧以及自我实现six
comments: true
date: 2018-08-10 20:58:04
toc: true
categories:
- Python
tags:
- 兼容
---

开发中经常会遇到有些包在Python2和Python3中的名字或者导包路径已经不一样，如果Python2版本里的代码拿到3版本的环境去运行就会发生错误。<!--more-->
例如：爬虫常用的urllib的查询字符串转化处理函数urlencode的导包路径就不一样
### urlencode不同版本使用

Python2:
```
import urllib
query_string = {
  'name': 'Python',
  'version': 2
}
query_string = urllib.urlencode(query_string)  
# version=2&name=Python
```

Python3:
```
import urllib

query_string = {
   'name': 'Python',
   'version': 2
}
query_string = urllib.parse.urlencode(query_string)
# version=2&name=Python
```

那么为了实现代码的兼容，我们可以使用一个非常好用的模块去判断当前环境所使用的Python版本：six  就是非常666的意思了（其实是2和3的最小公倍数）

### 使用six实现Python版本兼容
```
import six

if six.PY2:
   import urllib
else:
   import urllib.parse as urllib

query_string = {
   'name': 'Python',
   'version': 2
}
query_string = urllib.urlencode(query_string)
# version=2&name=Python
```

### 下面看看six到底是怎么实现的

查看six在Python2中的源码：

![](http://paresur4s.bkt.clouddn.com/python2.png)
查看six在Python3中的源码：
![](http://paresur4s.bkt.clouddn.com/python3.png)


看到这里，我。。。，原来是这样啊：

1. Python2版本six源码直接把PY2写为True

2. Python3版本six源码直接把PY3写为True

所以看到这里，一点都不玄乎

### 自我实现six

那么我们自己实现一个和six一样功能的mysix模块动态获取Python版本：

#### 思路：
sys模块有一个属性可以获取到Python的版本信息：version，那么我们也可以根据这个属性来判断当前的Python版本决定使用哪种导包方式



mysix.py
```
# coding=utf-8
import sys


PY2 = False
PY3 = False

def python_version():
   version = sys.version[0]
   # sys.version 返回版本信息字符串 3.7.0......
   if version == '2':
       global PY2
       PY2 = True
   else:
       global PY3
       PY3 = True
   return

# 导包时直接执行获取到版本信息
python_version()
```

和six一样直接导入使用即可：
```
import mysix

if mysix.PY2:
   import urllib
else:
   import urllib.parse as urllib

query_string = {
   'name': 'Python',
   'version': 2
}
query_string = urllib.urlencode(query_string)
# version=2&name=Python
```

至此，我们就自己实现了six中判断Python版本的功能进而实现代码版本兼容。

有时候，看一下源码，会豁然开朗很多，其实自己也可以实现。

学习要知其所以然，那样也会有动力很多，清晰很多。
