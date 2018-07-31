---
title: Python中else其他用法
comments: true
toc: true
date: 2018-07-31 09:19:55
categories:
- Python
tags:
- else其他用法
---
在Python中else最常见的用法就是用在判断语句中，其实还可以用在循环语句和异常处理中。<!--more-->
下面来总结一下else的用法：
### 判断语句
这是最常见的用法，当if语句中的条件不满足时，将执行else语句中的代码。
```
a = False
if a:
    print("a为真")
else:
    print("a为假")
```

### 循环语句
如果else子句紧接在循环语句的后面，那么在以下两种情况将会执行else子句的代码：
* 当循环体没有执行break的时候，即循环体正常结束
```
print("两次输入机会")
for i in range(2):
    num = int(input("请输入一个数字："))
    if 10 == num:
        print("10 == num,触发break，不会执行else子句")
        break
else:
    print("循环体没有执行break语句，执行else子句")
print("程序结束")
```
执行代码：
当触发break时,不会执行else子句：
```
两次输入机会
请输入一个数字：1
请输入一个数字：10
10 == num,触发break，不会执行else子句
程序结束
```
当没有触发break时,执行else子句：
```
两次输入机会
请输入一个数字：2
请输入一个数字：3
循环体没有执行break语句，执行else子句
程序结束
```
* 当while循环体完全不执行时也会执行紧跟在后面的else子句
```
while False:
    pass
else:
    print("循环体不执行，我也会执行")
# 执行后的输出结果：
# 循环体不执行，我也会执行
```
### 异常处理
当没有发生异常的时候会执行紧跟在异常处理代码后面的else子句
```
num1 = int(input("输入一个整数："))
num2 = int(input("输入另外一个整数："))
print('-'*20)
try:
    print("{}/{}=".format(num1,num2),num1//num2)
except ZeroDivisionError:
    print("输入非法，ZeroDivisionError")
else:
    print("输入合法")
print("程序结束")   
```
代码执行：
当没发生异常时：
```
输入一个整数：2
输入另外一个整数：1
----------------------------------------
2/1= 2
输入合法
程序结束
```
发生异常时：
```
输入一个整数：2
输入另外一个整数：0
----------------------------------------
输入非法，ZeroDivisionError
程序结束
```

### 总结
else子句的触发条件：
1. 在判断语句中，当if语句条件不满足时会就执行else子句的代码
2. 在循环语句中，当循环体没有执行或者循环体里执行了break语句
3. 在异常处理中，当没有发生异常时会执行else子句
