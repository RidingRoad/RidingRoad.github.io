---
title: Linux中su/su -/sudo的区别
comments: true
date: 2018-08-01 11:12:35
categories:
- Linux
tags:
- su
---
在linux中会经常使用到su/su -/sudo这三个命令，总结一下这三个命令的区别。<!--more-->
### su
使用root密码，切换到root用户，但是并没有转到root用户家目录下。
![](http://paresur4s.bkt.clouddn.com/FsQytdVVK0BiT0qRS7XeQdifOzKh)
### su -
使用root密码，切换到root用户，并转到root用户的家目录下。
![](http://paresur4s.bkt.clouddn.com/FsQytdVVK0BiT0qRS7XeQdifOzKh)
### sudo
使用**当前用户密码**，把某些超级权限有针对性的下放，sudo 也可以称为受限制的su。
![](http://paresur4s.bkt.clouddn.com/FtkIjNHA8lCrQtzYtbaXxbrT7j6b)
### 总结
* su 使用的是root密码，但不会切换到root家目录
* su - 使用的也是root密码，切换到root家目录
* sudo 使用的是当前用户的密码
