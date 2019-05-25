---
title: python装饰器所有常见用法
comments: true
date: 2019-05-25 17:32:49
tags:
- decorator
categories：
- Python
---
此篇文章包含装饰器四部分内容：
1. 简单装饰器
2. 装饰器的传参
3. 装饰器兼容是否传参
4. 多个装饰器的执行顺序
<!--more-->
介绍装饰器之前，先介绍一下闭包。
### 闭包
有以下这个函数：
```Python
def outer(a, b):
    print(a, b)
    def inner():
        print(a + 1, b + 1)
    return inner

outer_result = outer(666, 888)
print(outer_result)
print(type(outer_result))
outer_result()
```
输出结果
```Python
666 888
<function outer.<locals>.inner at 0x100fcc158>
<class 'function'>
667 889
```
![闭包例子](https://i.loli.net/2019/05/25/5ce8c0f5a64c785632.png
)
闭包实现了一个非常重要的功能：
参数缓存。我可以把参数都先放起来到一边，先去干别的，等我准备好了的时候再调用内部函数就可以了
### 最简单的装饰器
装饰器正是闭包的一个实现,装饰器也是闭包。只不过，闭包一般都是传变量进去，但装饰器是传函数名(函数定义所在内存的物理地址位置，类似指针)进去。
看下面这个例子：
比如说，我们需要做一个菜，
```
def cooking():
    print("洗菜")
    print("宽油下锅")
    print("下菜落锅")
    print("加调味料")
    print("耐心翻炒")
    print("尝尝味道")
    print("装碟")

```
现在做菜的函数有了，但是还没有食材啊，巧妇难为无米之炊，然后我们不想在做菜的函数里去写与做菜无关的事情，那么装饰器可以实现在不改变做菜函数的基础上先去买菜。
```Python
def buy_online(func):
    def wrapper():
        print("线上生鲜到家买买买")
        func()
    return wrapper


@buy_online
def cooking():
    print("洗菜")
    print("宽油下锅")
    print("下菜落锅")
    print("加调味料")
    print("耐心翻炒")
    print("尝尝味道")
    print("装碟")


cooking()
```
输出结果
```Python
线上生鲜到家买买买
洗菜
宽油下锅
下菜落锅
加调味料
耐心翻炒
尝尝味道
装碟
```
上面的buy_online就是一个闭包，只不过现在传进去的参数是一个函数而已。
那么传进去给闭包的参数是函数的话，那个这个函数就被叫做装饰器。
使用装饰器很简单，只需要在被装饰的函数的上方加@装饰器函数名就可以了。
#### @的工作机制
@会把下面被装饰的函数名作为一个参数传递给装饰器，然后把装饰器的返回值赋值给被装饰的函数名,最终被装饰的函数名保存的是装饰器的内部函数的地址。
![](https://i.loli.net/2019/05/25/5ce8cc0773fd441495.png
)

### 装饰器的传参
实现不同情况装饰器进行不同的处理，那么需要给装饰器进行传参控制。要实现装饰器参数的缓存，需要再进行一层闭包，即通过两层闭包实现给装饰器传参。
```Python
def buy_online(mall):
    def which_mall_online(func):

        def wrapper(*args, **kwargs):
            print("'{}' 线上生鲜到家 买买买".format(mall))
            func(*args, **kwargs)

        return wrapper

    return which_mall_online


@buy_online('每日优鲜')
def cooking(*args, **kwargs):
    print("洗菜")
    print("宽油下锅")
    print("下菜落锅")
    print("加调味料")
    print("耐心翻炒")
    print("尝尝味道")
    print("装碟")
```
输出结果
```Python
'每日优鲜' 线上生鲜到家 买买买
洗菜
宽油下锅
下菜落锅
加调味料
耐心翻炒
尝尝味道
装碟
```
![装饰器传参](https://i.loli.net/2019/05/25/5ce8e56b1cb5499318.png)

#### 装饰器兼容传参和不传参
如果希望上面的buy_online装饰器同时支持可以传参也可以不传参或者向第一个例子那样调用，如何实现一个装饰器同时支持下面的调用兼容呢？
```
@buy_online
@buy_online('每日优鲜')
@buy_online()
```
兼容方案思路：
1. 有传参和不传参的区别在于，有传参需要再闭包封装一次
2. 如果没有参数的话，我们手动调用最外层函数则返回了内部的第二个函数
```
def buy_online(decorated_func=None, mall=None):
    if decorated_func and mall:
        raise Exception('被装饰器装饰的函数和装饰器的参数不会在同一处出现')

    def which_mall_online(func):

        def wrapper(*args, **kwargs):
            nonlocal mall
            if mall == None:
                mall = ''
            print("{} 线上生鲜到家 买买买".format(mall))
            func(*args, **kwargs)

        return wrapper
    if decorated_func:
        # 如果是传递了被装饰的函数，则直接返回最内层wrapper函数
        return which_mall_online(decorated_func)
    else:
        # 进入这里，说明是装饰器是有参数或使用默认参数的，返回外层函数
        return which_mall_online


@buy_online(mall='饿了么生鲜')
def cooking(*args, **kwargs):
    print("洗菜")
    print("宽油下锅")
    print("下菜落锅")
    print("加调味料")
    print("耐心翻炒")
    print("尝尝味道")
    print("装碟")


@buy_online()
def cooking_default_parameter(*args, **kwargs):
    print("洗菜")
    print("宽油下锅")
    print("下菜落锅")
    print("加调味料")
    print("耐心翻炒")
    print("尝尝味道")
    print("装碟")


@buy_online
def cooking_decorator_no_parameter(*args, **kwargs):
    print("洗菜")
    print("宽油下锅")
    print("下菜落锅")
    print("加调味料")
    print("耐心翻炒")
    print("尝尝味道")
    print("装碟")

# 执行正常
cooking()
cooking_default_parameter()
cooking_decorator_no_parameter()
```

### 多个装饰器的执行顺序
说明多个装饰器的执行顺序，我们以最后达到了人生巅峰为例子。
在达到人生巅峰之前，需要走过很多坎坷或者不同的阶段。
那么我们以”达到人生巅峰?“为被装饰的函数，在它的上面不断的叠加不同的阶端。
```
受精卵
婴儿
幼稚园
小学
初中
高中
大学
社会大学
走向人生巅峰?
```
#### 各个阶段装饰器
```
def zero_stage(func):
    print("进入zero_stage函数定义")

    def zero_inner():
        print("还是个受精卵")
        func()

    return zero_inner


def kindergarten_stage(func):
    print("进入kindergarten_stage函数定义")

    def kindergarten_inner():
        print("懵懵懂懂进入了幼稚园")
        func()

    return kindergarten_inner


def primary_school_stage(func):
    print("进入primary_school_stage函数定义")

    def primary_school_stage_inner():
        print("每天佩戴红领巾的小学生")
        func()

    return primary_school_stage_inner


def middle_school_stage(func):
    print("进入 middle_school_stage函数定义")

    def middle_school_stage_inner():
        print("上晚自习的初中生")
        func()

    return middle_school_stage_inner


def high_school_stage(func):
    print("进入 high_school_stage函数定义")

    def high_school_stage_inner():
        print("一个月只能回一次家的高中生")
        func()

    return high_school_stage_inner


def university_stage(func):
    print("进入university_stage函数定义")

    def university_inner():
        print("从此知道了二进制的CS大学生")
        func()

    return university_inner


def social_stage(func):
    print("进入social_stage函数定义")

    def social_inner():
        print("社会大学，深不可测")
        func()

    return social_inner

```
#### 走向人生巅峰
```
def peak_of_life():
    print("走向人生巅峰")
```
#### 函数定义时的执行顺序
```
@zero_stage
@kindergarten_stage
@primary_school_stage
@middle_school_stage
@high_school_stage
@university_stage
@social_stage
def peak_of_life():
    print("走向人生巅峰?")
```
输出结果
```
进入social_stage函数定义
进入university_stage函数定义
进入 high_school_stage函数定义
进入 middle_school_stage函数定义
进入primary_school_stage函数定义
进入kindergarten_stage函数定义
进入zero_stage函数定义
```
函数定义时的顺序
![](https://i.loli.net/2019/05/25/5ce8fc77374f519409.png)
最终，
peak_of_life指向的是zero_inner,
zero_inner的func又指向了kindergarten_inner,
kindergarten_inner的func又指向了primary_school_stage_inner，
以此类推，....
#### 函数调用时的执行顺序
```
peak_of_life()
```
输出结果
```
还是个受精卵
懵懵懂懂进入了幼稚园
每天佩戴红领巾的小学生
上晚自习的初中生
一个月只能回一次家的高中生
从此知道了二进制的CS大学生
社会大学，深不可测
走向人生巅峰?
```
函数调用时的顺序，因为函数定义时的顺序决定了调用时的顺序。
执行peak_of_life()
即为执行zero_inner(),以此类推

#### 总结多个装饰器时的顺序
1. 被装饰函数定义时，由被装饰函数由下而上
2. 被装饰函数调用时时，从装饰器由上而下的顺序执行

通俗的说：
1. 被装饰器装饰的函数定义时就像穿衣服，先穿最里面的
2. 被装饰器装饰的函数被调用时就像脱衣服，先脱最外面的
![](https://i.loli.net/2019/05/25/5ce900fa274f365549.png)

想进一步学习的话，这里有很多电子书
![](https://i.loli.net/2019/05/25/5ce9014d84a0d55925.png)



