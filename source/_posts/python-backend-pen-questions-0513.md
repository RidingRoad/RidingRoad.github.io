---
title: Python后台开发笔试题0513
comments: true
date: 2019-05-14 12:54:47
tags:
- pen_questions
categories:
- Python
---
自己尝试做了一些常见的Python后台开发笔试题，其中第3题，希望有更好的解决方案，欢迎评论区留言留下你的方法。
<!--more-->
1. 用Python实现Fibonacci数列
这里使用迭代器进行实现
```
# fabonacci数列：后面的数字等于前面两个数之和，第一个数是0， 第二个数是1


def fabonacci(length):
    a = 0
    b = 1
    counter = 0
    while counter < length:
        counter += 1
        yield a
        a, b = b, a + b


def fixed_length_fabonacci(length):
    return [i for i in fabonacci(length)]


print(fixed_length_fabonacci(10))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```
2. 使用装饰器实现统计函数执行时间
```
import time


def time_counter(func):
    def wrapper():
        start_time = time.time()
        func()
        end_time = time.time()
        running_time = end_time - start_time
        print("func running time is {}".format(running_time))
        return running_time

    return wrapper


@time_counter
def some_func():  # 需要被统计运行时间的函数, some_func = time_counter(some_func)
    # do something
    time.sleep(1.11)


some_func()
```
3. 输入一个ip地址，返回下一个连续的ip地址.例如：输入192.168.1.23，返回192.168.1.24
```
MAX_IP = '255.255.255.255'


def get_next_valid_ip(ip):
    """
    ip字符串 --> 二进制数字 --> 十进制数字
            --> 十进制数字 + 1 --> next_ip字符串
    """
    current_ip = ip.strip()[:]
    if current_ip == MAX_IP:
        return '0.0.0.0'

    ip = ip.split('.')
    # 判断是否存在非法ip数字
    invalid_ip_nums = [i for i in ip if (int(i) > 2 ** 8 - 1) or (int(i) < 0)]
    if invalid_ip_nums:
        # print("{} is invalid ip, please check".format(current_ip))
        raise Exception("{} is invalid ip, please check".format(current_ip))

    # 转为二进制数字
    old_ip = [bin(int(i)) for i in ip[:]]
    for i, value in enumerate(old_ip):  # 补齐八位
        length = len(value)
        if length != 10:
            old_ip[i] = '0b' + ''.join(['0' for _ in range(0, 10-length)]) + old_ip[i][2:]
    # 转为十进制数字
    old_ip_binary = old_ip[0] + ''.join([i[2:] for i in old_ip[1:4]])
    old_ip_decimal = int(old_ip_binary, 2)

    # 十进制数字加1获得下一个ip
    next_ip_decimal = old_ip_decimal + 1
    # 十进制转为二进制
    next_ip_binary = bin(next_ip_decimal)[2:]
    # 以步长为8进行切割
    next_ip_binary_list = ['0b' + next_ip_binary[8*i:8*(i+1)] for i in range(0,4)]
    # 二进制转为十进制
    next_ip_decimal_list = [int(i, 2) for i in next_ip_binary_list]
    next_ip = '.'.join([str(i) for i in next_ip_decimal_list])

    return next_ip
```
4. 使用Python实现单例模式
```
class Singleton(object):
    _instance = None

    def __new__(cls):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls)
        return cls._instance
```
5. 假如有以下函数
```

def return_list(val, list=[]):
    list.append(val)
    return list
```
（1）写出相应的运行结果，并解释为什么
```
print(return_list(1))
print(return_list(2, []))
print(return_list(3))
# [1]
# [2]
# [1, 3]
# 使用可变对象为默认参数时，只会初始化一次
```
(2)如果希望预期输出如下，应该怎么修改？
预期结果
```
[1]
[2]
[3]
```
代码修改：
```
def return_list(val, lists=None):
    if not lists:
        lists = []
    lists.append(val)
    return lists
```

