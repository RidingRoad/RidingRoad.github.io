---
title: Django中序列化器read_only和write_only
comments: true
toc: true
categories:
- Python
date: 2018-08-09 21:50:42
tags:
- Django
---

在Django RESTFramework 的序列化器中，有一些需要注意的通用参数。<!--more-->
首先，我们需要知道的一点就是，序列化器不仅仅使用于模型类，这很重要。
序列化器的主要作用是进行数据的**校验**（还有进行对数据对象进行转换），并不是仅能对models进行校验，了解到这一点，后面的注意点就容易理解了。
### read_only 仅序列化输出
这个参数的作用是告诉序列化器：
1. 这个字段只需要进行校验格式或数据类型是否正确，无需入库
2. 这个字段需要序列化返回给请求
其实read_only字段的作用主要在项目中是为了添加模型类中没有的字段信息，比如用户注册后返回用户信息给前端的同时签发Json Web Token，通过作为user的一个token属性进行返回给前端
```
from rest_framework_jwt.settings import api_settings

class CreateUserSerializer(serializers.ModelSerializer):
    """
    创建用户序列化器
    """
    ...
    token = serializers.CharField(label='登录状态token', read_only=True)  # 增加token字段

    class Meta：
        ...
        fields = ('id', 'username', 'password', 'password2', 'sms_code', 'mobile', 'allow', 'token')  # 增加token
        ...

    def create(self, validated_data):
        """
        创建用户
        """
        # 移除数据库模型类中不存在的属性
        del validated_data['password2']
        del validated_data['sms_code']
        del validated_data['allow']
        user = super().create(validated_data)

        # 调用django的认证系统加密密码
        user.set_password(validated_data['password'])
        user.save()

        # 补充生成记录登录状态的token
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
        payload = jwt_payload_handler(user)
        token = jwt_encode_handler(payload)
        user.token = token

        return user

```
### write_only  仅反序列化输入
write_only 和read_only 刚好相反，write_only表示校验输入数据并且入库，但不能进行序列化输出，比如为了安全，密码我们一般会设置为write_only，不会把密码也返回出去到前端。
