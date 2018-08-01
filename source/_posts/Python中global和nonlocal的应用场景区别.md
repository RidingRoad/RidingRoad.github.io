---
title: Python中global和nonlocal的应用场景区别
comments: true
date: 2018-08-01 09:54:07
tags:
---
在Python中，global和nonlocal的作用都是可以实现代码块内变量使用外部的同名变量，但其中是有很明显的区别的。<!--more-->
### global
global很明显就是声明代码块中的变量使用外部**全局**的同名变量
```
a = 1
def change():
        global a
        a += 1
        print("函数内部的a的值：", a)   # 2
change()
print("调用change函数后， 函数外部的a的值：", a)  # 2
```
### nolocal
nolocal 的使用场景就比较单一，它是使用在闭包中的，使变量使用**外层**的同名变量
```
def foo(func):
        a = 1
        print("外层函数a的值", a)
        def wrapper():
                func()
                nonlocal a
                a += 1
                print("经过改变后，里外层函数a的值：", a)
        return wrapper

@foo
def change():
      print("nolocal的使用")

change()
----------------输出结果----------------
外层函数a的值 1
nolocal的使用
经过改变后，里外层函数a的值： 2
```
### 总结
global的作用对象是全局变量，nonlocal的作用对象是外层变量（很显然就是闭包这种情况）
