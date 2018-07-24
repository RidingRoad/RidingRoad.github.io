---
title: Python自动化Markdown无序列表
comments: true
date: 2018-07-24 08:44:27
toc: true
categories:
- Python
- Fun
tags:
- 无序列表
- Markdown
---

## Python自动化Markdown无序列表

#### 应用场景
假如我们在编辑Markdown格式的文档，需要显示一个很大的列表，但目前只有每行的文本，那么需要在每一行的前面添加“* ” ，这样在Markdown的文档就可以形成无序列表了。
<!--more-->
Markdown语法
```python
* 哈哈
* 哈哈
```
效果：
* 哈哈
* 哈哈
#### 例如有朱自清的246行《毁灭》长诗需要以无序列表的形式显示
如果要手工在每一行前添加“* ”，那么多么的累啊，我们要自动化
#### 那么就演示这个小程序的使用过程吧，看视频
#### 操作步骤
1. 选中246行长诗，右击复制
2. 运行本程序
3. 在你需要插入的地方右击粘贴，Bingo

#### 我们看看Python的pyperclip模块是怎样实现的
pyperclip模块不是内置模块需要自己安装
**pip3 install pyperclip**
1. pyperclip.paste()
从电脑剪贴板中获取复制的内容
2. pyperclip.copy(text)
把text内容放到电脑的剪贴板，那么粘贴时的数据就变成text的内容了
#### 那么，我们来实现一下自动化在每行前面添加“* ”
```python
#！/usr/bin/env python3
# coding=utf-8
__author__ = "RidingRoad"

import pyperclip

def main():
    """运行前把需要形成无序列表的数据选中右击复制
       运行后右击粘贴即可生成Mark down格式无序列表
    """
    # 获取剪贴板的数据
    text = pyperclip.paste()
    # 对长字符串根据"\n"进行分割到一个列表
    text_split = text.split('\n')
    # 在每一行前添加"* "(*号和一个空格)
    for i in range(len(text_split)):
        text_split[i] = "* " + text_split[i]
    # 合并
    text = "\n".join(text_split)
    # 把处理后的数据放回剪贴板
    pyperclip.copy(text)


if __name__ == "__main__":
    main()
```


#### 运行后剪贴板内容：
```python
* 踯躅在半路里，
* 垂头丧气的，
* 是我，是我！
* 五光吧，
* 十色吧，
* 罗列在咫尺之间：
* 这好看的呀！
* 那好听的呀！
* 闻着的是浓浓的香，
* 尝着的是腻腻的味；
* 况手所触的，
* 身所依的，
* 都是滑泽的，
* 都是松软的！
* 靡靡然！
* 怎奈何这靡靡然？
...............
```
#### 运行后Markdown效果
* 踯躅在半路里，
* 垂头丧气的，
* 是我，是我！
* 五光吧，
* 十色吧，
* 罗列在咫尺之间：
* 这好看的呀！
* 那好听的呀！
* 闻着的是浓浓的香，
* 尝着的是腻腻的味；
* 况手所触的，
* 身所依的，
* 都是滑泽的，
* 都是松软的！
* 靡靡然！
* 怎奈何这靡靡然？——
。。。。。。。。。
#### 好了，到这里，已经自动化解决了
我的个性签名:Focusing on the Python and firmly convincing that nothing can replace hard work.
需要完整的代码关注我的公众号Python孙行者,聊天界面回复"自动化无序列表",即可获取.
