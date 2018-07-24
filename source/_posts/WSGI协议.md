---
title: WSGI协议
comments: true
date: 2018-07-24 21:36:39
toc: true
categories:
- Python
tags:
- WSGI
---
了解WSGI协议对Web框架学习具有很重要的意义。<!--more-->
### WSGI协议

* 实现WSGI协议必须要有wsgi server和application。

* WSGI协议其实是定义了一种server与application解耦的规范，即可以有多个实现WSGI server的服务器，也可以有多个实现WSGI application的框架，那么就可以选择任意的server和application组合实现自己的web应用。例如uWSGI和Gunicorn都是实现了WSGI server协议的服务器，Django，Flask是实现了WSGI application协议的web框架，可以根据项目实际情况搭配使用。

### WSGI server


准备 environ 参数

定义 start_response 函数

调用程序端的可调用对象

### WSGI application

WSGI 规定每个 python 程序（Application）必须是一个可调用的对象（实现了__call__ 函数的方法或者类），接受两个参数 environ（WSGI 的环境信息） 和 start_response（开始响应请求的函数），并且返回 iterable。


```

def application(environ, start_response):

    status = '200 OK'

    response_headers = [('Content-Type', 'text/html')]

    start_response(status, response_headers)

    return str(environ)

```


* environ 和 start_response 由 http server 提供并实现

* environ 变量是包含了环境信息的字典

* Application 内部在返回前调用 start_response

* start_response也是一个 callable，接受两个必须的参数，status（HTTP状态）和 response_headers（响应消息的头）, 数据格式为列表， 列表里面是一个个响应头元组

* 可调用对象要返回一个值，这个值是可迭代的。
