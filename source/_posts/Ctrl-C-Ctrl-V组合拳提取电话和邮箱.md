---
title: 'Ctrl+C,Ctrl+V组合拳提取电话和邮箱'
comments: true
toc: true
date: 2018-07-24 08:39:33
categories:
- Python
- Fun
tags:
- 提取电话邮箱
---
# Ctrl+C,Ctrl+V组合拳提取电话和邮箱
> 目前假设有这么一个需求: 你的一份文件里面有许多电话、邮箱以及其他文本内容,领导要求你从文本中把电话和邮箱提取处理整理到一个文档中,你想可不可以我只需要Ctrl + C ,Ctrl + V 就可以了呢.那么本条推文就可以帮你实现.
<!--more-->
### 提取思路
1. 打开文件,Ctrl + C
2. 从剪贴板获取你内容
3. 正则匹配,保存到一个列表里
4. 把结果返回剪贴板
5. 你Ctrl + V 到一个新文档,美滋滋的完成,Bingo
### 匹配电话
```python
 # 提取固话即分机号码的正则
    phoneRegex = re.compile(r'''(
        (\d{3}|\(\d{3}\))?          # 匹配区号,'?'表示区号可有可无
        (\s|-|\.)?                  # 分隔符 ,号码数字之间的分隔符 比如 020-333-3333,'?'表示分隔符可有可无
        (\d{3})                     # 号码中前面三个数字
        (\s|-|\.)                   # 分隔符
        (\d{4})                     # 最后的4个数字
        (\s*(ext|x|ext.)\s*(\d{2,5}))? # 获取分机号码
        )''', re.VERBOSE)  # re.VERBOSE 表示可以在正则表达式中写注释,正则中注释也是以'#'开头
```

### 匹配邮箱

```
    # 提取邮箱的正则
    emailRegex = re.compile(r'''(
        [a-zA-Z0-9.%+-]+        # 邮箱的用户名
        @                       # @标志
        [a-zA-Z0-9.-]+          # 域名
        (\.[a-zA-Z]{2,4})       # 顶级域名
        )''', re.VERBOSE)
```

### 获取剪贴板信息

```python
text = str(pyperclip.paste())  # 获取剪贴板的信息存到变量text中
```

### 处理过程
```
matches = []  # 匹配成功的邮箱和电话存放到matches列表中

    for groups in phoneRegex.findall(text):  # 遍历匹配到的电话列表
        phoneNum = '-'.join([groups[1], groups[3], groups[5]])  # 统一格式 020-333-3333
        if groups[8] != '':  # 如果有分机,则把分机号码也提取出来
            phoneNum += 'x' + groups[8]
        matches.append(phoneNum)

    for groups in emailRegex.findall(text):  # 遍历匹配到的邮箱
        matches.append(groups[0])

    if len(matches) > 0:  # 判断文本中是否存在电话邮箱地址
        pyperclip.copy("\n".join(matches))  # 把匹配结果存放到剪贴板中
        # print('Copied to clipboard:')
        # print('\n'.join(matches))
    else:
        print("文本中不存在邮箱或电话")
```
### 到这里,可以Ctrl + V了
到了这里，CV组合实现提取电话和邮箱了。需要完整的代码关注我的公众号Python孙行者,聊天界面回复"CV",即可获取.
