---
title: 段子一键取并存为Markdown文档
comments: true
date: 2018-07-29 09:36:01
toc: true
categories:
- Python
tags:
- 爬虫
---

一键运行获取内涵吧的所有内涵段子并保存为Markdown格式文档。<!---more-->
源码获取：公众号***Python孙行者***后台回复***‘内涵’***即可

#### 本文意愿

本文的意愿为通过对某网站的爬取，掌握基本的爬虫技能并非以爬取其段子内容为目的。在爬取过程中，请务必设置好爬取的时间间隔，以免影响网站的正常运行。

#### 实战目标

1. 分析网页的规律
2. 使用requests库爬取
3. 运用正则进行数据清洗和构造
4. 本地文件保存方法
#### 分析网页规律

通过观察网址的变化，我们只需要更改url最后一个数字即可，并且这个数字即为当前页数。

```
# 第一页
https://www.neihan8.com/article/list_5_1.html
# 第二页
https://www.neihan8.com/article/list_5_2.html
# 第三页
https://www.neihan8.com/article/list_5_3.html
```
#### 使用requests库爬取

这里主要是使用get请求方式获取网页内容，需要注意的是需要带上User-Agent和Referer连个请求头信息。因为通过F12发现浏览器发送的请求头携带了这两个参数，为避免对方以这两个字段进行反爬而获取失败，我们就带上这两个字段
![](http://paresur4s.bkt.clouddn.com/FlBMB5wSuKl5VhINJzyLmDldX2Cw)

```
 # 请求某一具体页面数据
    def request(self, url, index):
        print('url:', url)
        # 设置时间间隔， 以免对目标网站造成太大压力
        time.sleep(0.5)
        # 为了规避反爬，爬取的不是第一页需带上Referer请求头信息
        if index != 1:
            # 动态生成url
            self.headers['Referer'] = 'https: // www.neihan8.com / article / list_5_{}.html'.format(index - 1)
        # 获取响应数据
        response = requests.get(url, headers=self.headers)
        # 此网站使用GBK编码，对返回的数据进行相应解码
        return response.content.decode('gbk')
```
#### 数据清洗

在这里使用正则进行对我们需要的数据进行清洗和处理。
```
# 对数据进行提取清洗
    def parse_data_list(self, index):
        # result_list存放已解析清洗好的段子数据
        self.result_list = []
        # 获取到的段子列表数据放到duanzi
        duanzi = self.get_data(index)
        # 对段子列表数据duanzi每一个段子去掉空格和其他字符，保留换行
        for item in duanzi:
            result = re.sub(r'[<b>|</b>|<br />|<br>|<p>|</p>|\\u3000|\\r|\s|t|div|sa]', "", str(item))
            result = result.replace('&ldqo;', '“').replace('&dqo;', '”').replace('&helli;', '...').replace('&lqo;',
                                                                                                           '“').replace(
                '&qo;', '”').replace('&hell;', '...').replace('n', '\\n').replace('&zwj;', '').replace('&qot;',
                                                                                                       '"').replace(
                '&mh;', '').strip(' ')
            result = re.sub(r'yle.*?(?:x;|\)|;|\n|x)"', '', result)
            result = re.sub(r'hef.*?ml"', '', result)
            result = re.sub(r"cl=.*?(?:we'|\d+\")", '', result)
            result = re.sub(r'c=.*?\.jg"', '', result)

            # 把清洗好的数据添加到result_list
            self.result_list.append(eval(result))
        return self.result_list
```
#### 保存为Markdown文档

主要构造为三号标题和正文即可
```
# 每一页数据保存为一个单独的Markdowm文档
    def save_md(self, index):
        duanzi_list = self.parse_data_list(index)
        file = open('内涵段子{}.md'.format(index), 'w')
        for item in duanzi_list:
            # 拼接成Markdown格式
            file.write('### ' + item[0] + item[1])
        # 关闭文件
        file.close()
```
#### 一键运行效果

妥妥的506页，506个Markdown文档，5057个段子。
源码获取：公众号***Python孙行者***后台回复***‘内涵’***即可
![](http://paresur4s.bkt.clouddn.com/Frxbv1iTlDzM5XCthk19PXTfgn_E)
![](http://paresur4s.bkt.clouddn.com/FmYHVEpvsfFYV0lu4Zgrkb7pRoK9)
