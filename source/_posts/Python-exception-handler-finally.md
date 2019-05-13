---
title: Python异常处理机制下finally的陷阱
comments: true
date: 2019-05-12 20:54:15
tags:
- finally
category:
- Python
---

整理异常处理中finally的两个陷阱
<!--more-->
### Python异常处理机制
![](https://i.loli.net/2019/05/12/5cd81ac89751d.gif)

**如果try异常处理中存在finally，finally中的代码总会得到执行**下面例子只是作为演示，不用去纠结业务逻辑
### 挖坑1
看下面的代码，写出输出结果
```
def dig_dig1():
    while True:
        print("I'm in while loop")
        try:
            print("I'm in try")
            raise EOFError
        except IOError:
            print("IOEoor")
        finally:
            print("I'm in finally")
            break


dig_dig1()
```
原以为会输出的结果：
```
I'm in while loop
I'm in try
I'm in finally
EOFError

Process finished with exit code 1
```
运行结果
```
I'm in while loop
I'm in try
I'm in finally

Process finished with exit code 0
```
**如果异常处理中存在finally， finally总会被执行**。那么在执行finally之前，try中的产生的异常将会被临时保存起来，当finally的代码执行完成后，再抛出异常。但当finally中存在raise或return或break时， try中的异常将会被抛弃。
### 挖坑2
看下面的代码，写出输出结果
```
def dig_dig2(index):
    try:
        print("I'm in try")
        if index < 0:
            raise IndexError
        else:
            return index
    except IndexError:
        print("I'm in except")
        return "except"
    finally:
        print("I'm in finally")
        return "finally"


print(dig_dig2(12))
```
原以为会输出的结果：
```
I'm in try
12
```
运行结果
```
I'm in try
I'm in finally
finally

Process finished with exit code 0

```
**如果异常处理中存在finally， finally总会被执行**。
1. 如果在try块语句中存在return的同时又存在finally块语句，那么将会在执行try块语句中return语句之前去执行finally语句块，然后再回来执行try块语句中return语句。
2. 但是例子中在finally块语句中存在return语句，整个函数已结束，所以try块语句中return语句将永远得不到执行。
