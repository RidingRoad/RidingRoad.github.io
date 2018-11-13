---
title: Python中实现Linux管道
comments: true
date: 2018-11-13 21:14:11
categories:
- Python
tags:
- 管道
---
### 需求描述
假如需要实现一个随机生成三餐的食谱的需求，提供全部随机生成或自定义同时随机生成两种选项，尽量Pythonic。例如，这样调用即可：
```
breakfast >> lunch >> dinner
```
<!--more-->
### 需求分析
1.>\> 的语法实现需要使用\__rshift__和\__rrshift__。
2. 自定义一个三餐的列表作为随机选取的对象
3. 如果是全部随机生成，则无需传递参数，进行4和5的步骤
4. breakfast的结果传递给lunch，在breakfast的结果基础插入lunch
5. lunch的结果传递给dinner，在lunch的结果基础插入dinner
6. 如果是自定义同时随机生成，则需要一个函数进记录参数，然后重复45步骤
### 实现>>管道语法
语法例子建议参考前两期推文
```
# -*- coding=utf-8 -*-
import functools
import random
import pprint

class Pipe:

    def __init__(self, function):
        self.function = function
        functools.update_wrapper(self, function)

    # 当 >> 两边其中一个没有实现__rshift__时调用
    def __rrshift__(self, other):
        return self.function(other)

    # 实现携带自定义参数
    def curry(self, *args, **kwargs):
        return Pipe(lambda x: self.function(x, *args, **kwargs))
```
### 菜单候取对象
```
breakfast_menu = ['bacon', 'egg', 'milk', 'porridge', 'bread', 'corn']
lunch_menu = ['fish', 'beef', 'chicken', 'radish', 'spinach', 'watercress']
dinner_menu = ['lean meat', 'mutton', 'celery', 'tofu', 'cucumber']
```
### 菜单随机实现
```
@Pipe
def breakfast(menu, *args, **kwargs):
    menu['breakfast'] = random.sample(breakfast_menu, 3 if args == ('',) else 3 - len(args))
    if not args == ('',):
        for custom in args:
            menu['breakfast'].append(custom)
    return menu


@Pipe
def lunch(menu, *args, **kwargs):
    menu['lunch'] = random.sample(lunch_menu, 3 if args == ('',) else 3 - len(args))
    if not args == ('',):
        for custom in args:
            menu['lunch'].append(custom)
    return menu


@Pipe
def dinner(menu, *args, **kwargs):
    menu['dinner'] = random.sample(dinner_menu, 3 if args == ('',) else 3 - len(args))
    if not args == ('',):
        for custom in args:
            menu['dinner'].append(custom)
    return menu
```
### 整体控制及管道实现
```
if __name__ == "__main__":
    # menu存放菜单
    menu = dict()

    random_choice = int(input("菜单是否自动生成？是：1 否：0\n"))
    # 自动随机生成
    if random_choice:
        pprint.print(menu >> breakfast >> lunch >> dinner)
    # 自定义菜单，传递参数
    else:
        breakfast_custom_list = input("早餐菜单(最多3样),不够3样，剩下随机生成：").split(' ')
        lunch_custom_list = input("午餐菜单(最多3样),不够3样，剩下随机生成：").split(' ')
        dinner_custom_list = input("晚餐菜单(最多3样),不够3样，剩下随机生成：").split(' ')

        # 携带参数, *的作用是解包，[1,2,3] => 1,2,3
        breakfast = breakfast.curry(*(breakfast_custom_list))
        lunch = lunch.curry(*(lunch_custom_list))
        dinner = dinner.curry(*(dinner_custom_list))

        # 输出的漂亮格式就不搞了，直接打印啦
        pprint.print(menu >> breakfast >> lunch >> dinner)
```
### 随机生成效果
```
菜单是否自动生成？是：1 否：0 
1
{'breakfast': ['bread', 'egg', 'corn'],
 'dinner': ['tofu', 'cucumber', 'lean meat'],
 'lunch': ['spinach', 'chicken', 'fish']}
```
### 自定义菜单生成效果
```
菜单是否自动生成？是：1 否：0
0
早餐菜单(最多3样),不够3样，剩下随机生成：地瓜 玉米 白粥
午餐菜单(最多3样),不够3样，剩下随机生成：
晚餐菜单(最多3样),不够3样，剩下随机生成：丝瓜汤
{'breakfast': ['地瓜', '玉米', '白粥'],
 'dinner': ['tofu', 'cucumber', '丝瓜汤'],
 'lunch': ['beef', 'fish', 'radish']}
```
### 携带参数的实现
```
# 实现携带自定义参数
    def curry(self, *args, **kwargs):
        return Pipe(lambda x: self.function(x, *args, **kwargs))
```
原理： 匿名函数的\*args和\**kwargs保存了需要携带的参数。
### 其他
其他语法，未提到的，欢迎留言沟通

