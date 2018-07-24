---
title: 使用正则校验密码强度以及正则版的Python字符串strip处理函数
comments: true
date: 2018-07-24 08:51:12
toc: true
categories:
- Python
- Fun
tags:
- 正则
- 密码强度校验
---


# 在Python中正则这么玩
## 正则,就是用来匹配字符串的.但总是显得深不可测,那么通过下面两个例子,希望你会喜欢上正则.
### strip字符串处理函数正则版
#### Python自带的字符串处理函数strip()效果
总的来说,就是当strip()不带参数时,去掉两端空格;带参数时,将把字符串中与参数一样的字符删除但两端的空格不做处理.
```python
print('  12334 536 738  '.strip())
print('  12334 536 738  '.strip('3'))
# 输出结果
12334 536 738
  12334 536 738  
```
#### 那么,我们用正则怎么实现呢?
> 解题思路
a.如果没有参数,那么删除两端的空白
     * 通过分组的形式,分离两端空白部分和需要保留的部分
     * 正则部分:r'(\s\*)(.+\w)(\s*)'
b. 如果有两个参数,那么删除指定字符,使用re.sub进行替换即可

```python
def strip_regex_version(string_data,delete=None):
    if delete == None:
        # 删除两侧空白
        string_data_regex = re.compile(r'(\s*)(.+\w)(\s*)') # 分组提取需要保留的部分
        result = string_data_regex.match(string_data).group(2) # 第二个括号匹配出来的值是我们需要的
    else:
        # 删除指定的字符,直接用空字符替换就好
        result = re.sub(delete,'',string_data)
    return result
```

### 校验强口令正则版
> 强口令:长度不少于8个字符,同时包含大小写字母,至少有一位数字

> 解决思路:
    1.使用len() 检测密码的长度
    2.使用一个正则检测是否有至少一位数字
    3.使用一个正则检测是否有大写字母
    4.使用一个正则检测是否有小写字母
    5.上面四个条件都为真的话,就返回True,否则返回False
```python
def strong_password_detection(password):
    length_flag = False # 长度标志
    digit_flag = False  # 数字标志
    upper_flag = False  # 大写标志
    lower_flag = False  # 小写标志

    # 匹配数字的正则
    digit_regex = re.compile(r'\d')
    # 匹配大写字母的正则
    upper_regex = re.compile(r'[A-Z]')
    # 匹配小写字母的正则
    lower_regex = re.compile(r'[a-z]')

    if len(password) >= 8: # 判断长度
        length_flag = True

    if len(digit_regex.findall(password)) >= 1: # 判断是否包含至少一位数字
        digit_flag = True

    if len(upper_regex.findall(password)) > 0: # 判断是否包含大写字母
        upper_flag = True

    if len(lower_regex.findall(password)) > 0: # 判断是否包含小写字母
        lower_flag = True

    if length_flag and digit_flag and upper_flag and lower_flag: # 判断是否同时满足4个条件
        return True
    else:
        return False
```
### 正则,可以实现的东西超出你我的想象
公众号Python孙行者回复"strip"获取完整代码
