---
title: Python函数传参既不是传值也不是传引用
comments: true
date: 2019-05-28 09:27:47
tags:
- 函数传参
categories:
- Python
---
在进入Python函数传参是传值还是传引用的问题前，先介绍一下下面的4个基本概念：
1、形参；
2、实参；
3、传值；
4、传引用。
<!--more-->
### 形参和实参
**形参**：在函数定义中的参数的名字被叫做形式参数(形参)
**实参**：调用函数时所赋值给形式参数(形参)的变量是实际参数(实参)
```Python
pen_name = 'RidingRoad'


def my_name(name):
    # name 是形参
    print("My name is {}".format(name))

# pen_name是实参
my_name(pen_name)
```
### 传值
传值：
实际参数的值赋值给形式参数后，在函数内对形式参数做任何修改都不会影响到实际参数。
简言之就是值拷贝，就是你在我这沾了”敬业福“，你沾到”敬业福“后，你对你拥有的“敬业福”是送给别人还是合成，都不会影响到我的“敬业福”的状态。
### 传引用
传引用：
传引用的意思就是形式参数和实际参数都指向同一块内存， 你即是我，我即是你，在函数内对形式参数的操作就是对实际参数的操作。

### Python函数传参
很多人对Python函数传参是传值还是传引用这个问题的理解，会认为可变对象传引用，不可变对象传值。但实际情况真的是这样吗？
常见的可变对象：列表；字典；集合
常见的不可变对象：数值型；字符串；元组
```
outer_list = [1, 2, 3]


def pass_parameter(inner_list=None):
    if not isinstance(inner_list, list):
        raise Exception('{} is not list.'.format(inner_list))
    print('inner_list id is {}'.format(id(inner_list)))
    # (1)按照传可变对象传引用的说法，修改inner_list即等同修改outer_list
    inner_list.append(4)
    print('inner_list.append(4)后值为{}, id为{}'.format(inner_list, id(inner_list)))
    # (2)按照传可变对象传引用的说法，修改inner_list即等同修改outer_list
    inner_list = [5, 6, 7]
    print('inner_list = [5, 6, 7]后的id为{}'.format(id(inner_list)))


print('outer_list id is {}'.format(id(outer_list)))
pass_parameter(inner_list=outer_list)
print("修改后outer_list的值是{}, id是{}".format(outer_list, id(outer_list)))
```
在(1)处修改列表的值，内部和外部的列表都同时被修改成功；
但在(2)处修改内部列表的值却没有成功修改外部列表的值；

上面的例子说明了，可变对象传引用，不可变对象传值的说法是不正确的。
在这里需要指出的是， 在Python中万物皆对象， **Python函数传参是传递整个对象**。
![Python函数传参是传对象](https://i.loli.net/2019/05/27/5cec07813cbc156152.png)
例子中的变量outer_list 的值[1, 2, 3]是一个对象，变量outer_list只是贴在[1, 2, 3]这个对象上一个标签而已。

当执行pass_parameter(inner_list=outer_list)时，相当于执行pass_parameter(inner_list=[1, 2, 3])，此时对象[1, 2, 3]已经贴有了两个标签(outer_list, inner_list)。

当执行inner_list.append(4)时，由于inner_list所指向的对象[1, 2, 3]是可变对象，Python会在其后添加4后返回原来的id，即返回原来的对象， 由于inner_list和outer_list都是指向[1, 2, 3]，所以对对象[1, 2, 3]的修改，会同时在inner_list和outer_list上得到体现。

当执行inner_list = [5, 6, 7]时， Python会新生成对象[5, 6, 7]， 并把原来贴在[1, 2, 3, 4]的标签inner_list贴在了对象[5, 6, 7]， 但outer_list仍贴在对象[1, 2, 3, 4]。


想了解更多关于Python的知识，看下面
![](https://i.loli.net/2019/05/25/5ce9014d84a0d55925.png)
