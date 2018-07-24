---
title: Django框架
comments: true
date: 2018-07-24 21:43:07
toc: true
categories:
- Django
tags:
- Django
---

Django框架，一个重量级的Python Web框架。<!--more-->

### 创建工程

```

django-admin startproject  工程名称

```

### 配置文件

1. DEBUG

2. 修改时区与语言

### 创建子应用

1. python manage.py startapp 子应用名称

2. 注册子应用　到配置文件中INSTALLED_APPS的列表中添加注册

### 创建模型

* 修改配置文件

```
DATABASES = {
           'default': {
              'ENGINE': 'django.db.backends.mysql',
              'NAME': 'books'  # 数据库名字
              'HOST': '127.0.0.1',  # 数据库主机
              'PORT': 3306,  # 数据库端口
              'USER': 'root',  # 数据库用户名
              'PASSWORD': 'mysql',  # 数据库用户密码
           }
       }
```
* 安装包

 pip install PyMySQL
*  在Django的工程同名子目录的__init__.py文件中添加如下语句


```
from pymysql import install_as_MySQLdb
install_as_MySQLdb()

```
*  到数据库软件里去创建数据库

```
create database 数据库名称 default charset=utf8;
```
*  创建模型类


模型类被定义在"应用/models.py"文件中。
模型类必须继承自Model类，位于包django.db.models中。



```

from django.db import models


class BookInfo(models.Model):
    btitle = models.CharField(max_length=20, verbose_name='名称')
    bpub_date = models.DateField(verbose_name='发布日期')
    bread = models.IntegerField(default=0, verbose_name='阅读量')
    bcomment = models.IntegerField(default=0, verbose_name='评论量')
    is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')

    class Meta:
        db_table = 'tb_books'  # 指明数据库表名
        verbose_name = '图书'  # 在admin站点中显示的名称
        verbose_name_plural = verbose_name  # 显示的复数名称

    def __str__(self):
        """定义每个数据对象的显示信息"""
        return self.btitle

```

*  数据库迁移

```

python manage.py makemigrations [某个子应用]

python manage.py migrate [某个应用]

```

### 站点管理

1. 创建超级管理员

```

python manage.py createsuperuser  按提示输入用户名密码等信息即可

```

2. 注册模型类到站点

打开　子应用/admin.py文件，编写如下代码：

```
from django.contrib import admin
from users.models import BookInfo

admin.site.register(BookInfo)
```
### 创建视图和路由

1. 在子应用的urls.py 文件添加路由和对应的视图

urlpatterns =  [url(正则, 对应的视图函数)]

2. 在工程总路由urls.py文件中通过include添加子应用的路由数据

urlpatterns = [url(正则, include('子应用.urls'))]

### 数据库

#### 定义模型类

* 数据表名　db_name = 数据表名

* 属性命名　字段名　＝　models.字段类型(选项)

#### 数据库操作 增删改查

#####  增加两种方式 create 和 save

create :  模型类.objects.create()

save:   分两步

 a.创建模型类对象；

 b.执行对象的save()方法保存到数据库中

##### 查询

###### 基本查询


get 查询单一结果，如果不存在会抛出模型类.DoesNotExist异常。

all 查询多个结果。

count 查询结果数量。

###### 过滤查询


filter 过滤出多个结果

exclude 排除掉符合条件剩下的结果

过滤条件的表达语法如下：

```
属性名称__比较运算符=值
属性名称和比较运算符间使用两个下划线，所以属性名不能包括多个下划线
```
1） 相等 exact

2） 模糊查询 contains  startswith endswith

3）空查询 isnull

4）范围查询 User.objects.filter(id__in=[1,2,3,4])

5）比较查询  lt  gt lte gte

6）日期查询 year、month、day、week_day、hour、minute、second：对日期时间类型的属性进行运算。

7）F对象 两个属性比较    F('属性名')

8)  Q对象  Q对象可以使用&、|连接，&表示逻辑与，|表示逻辑或。～表示取反

###### 聚合函数

使用aggregate()过滤器调用聚合函数。聚合函数包括：Avg 平均，Count 数量，Max 最大，Min 最小，Sum 求和，被定义在django.db.models中。aggregate的返回值是一个字典类型

###### 排序


```
BookInfo.objects.all().order_by('bread')  # 升序
BookInfo.objects.all().order_by('-bread')  # 降序
```
###### 关联查询

1由一到多的访问语法：

一对应的模型类对象.多对应的模型类名小写_set 例：

```
b = BookInfo.objects.get(id=1)
b.heroinfo_set.all()
```
2）由多到一的访问语法:

多对应的模型类对象.多对应的模型类中的关系类属性名 例：

```
h = HeroInfo.objects.get(id=1)
h.hbook
```
3）访问一对应的模型类关联对象的id语法:

多对应的模型类对象.关联类属性_id

例：

```
h = HeroInfo.objects.get(id=1)
h.hbook_id
```
###### 关联过滤查询


1由多模型类条件查询一模型类数据:

语法如下：

关联模型类名小写__属性名__条件运算符=值
注意：如果没有"__运算符"部分，表示等于。

例：

查询图书，要求图书英雄为"孙悟空"

```
BookInfo.objects.filter(heroinfo__hname='孙悟空')
```
查询图书，要求图书中英雄的描述包含"八"

```
BookInfo.objects.filter(heroinfo__hcomment__contains='八')
```
2）由一模型类条件查询多模型类数据:

语法如下：

```
一模型类关联属性名__一模型类属性名__条件运算符=值
```
注意：如果没有"__运算符"部分，表示等于。

例：

查询书名为“天龙八部”的所有英雄。

```
HeroInfo.objects.filter(hbook__btitle='天龙八部')
```
查询图书阅读量大于30的所有英雄

```
HeroInfo.objects.filter(hbook__bread__gt=30)
```
##### 修改


修改更新有两种方法

1）save

修改模型类对象的属性，然后执行save()方法

```
hero = HeroInfo.objects.get(hname='猪八戒')
hero.hname = '猪悟能'
hero.save()
```
2）update

使用模型类.objects.filter().update()，会返回受影响的行数

```
HeroInfo.objects.filter(hname='沙悟净').update(hname='沙僧')
```
##### 删除


删除有两种方法

1）模型类对象.delete()

```
hero = HeroInfo.objects.get(id=13)
hero.delete()
```
2）模型类.objects.filter(条件).delete()

```
HeroInfo.objects.filter(id=14).delete()
```
#### 查询集


当调用如下过滤器方法时，Django会返回查询集（而不是简单的列表）：

all()：返回所有数据。
filter()：返回满足条件的数据。
exclude()：返回满足条件之外的数据。
order_by()：对结果进行排序。

exists()：判断查询集中是否有数据，如果有则返回True，没有则返回False。

##### 两大特性

1）惰性执行

2）缓存

##### 限制查询集

查询集进行取**下标或切片**操作

#### 管理器 models.Manager类的对象

 1）默认为objects )

2）在模型类中自定义管理器

```

books = models.Manager()

```

### Request

* 提取URL部分 ： 正则

* 查询字符串(不区分请求方式)： request.GET.get/getlist(key, default_value)

* 请求体：

1）表单类型：request.POST.get/getlist(key)

2） 非表单类型json等：request.body  返回的是bytes类型，需要**解码(Python3.6以上版本可以忽略解码)以及使用json.loads() **转换为字典类型

* 请求头： request.META  字典类型

### Response

#### HttpResponse(content=相应体, content_type=xxxxx, status=xxxx)

#### JsonResponse(字典类型数据)

#### redirect(reverse(namespace:name))

### Cookies

#### 设置Cookies

HttpResponse.set_cookies(key, value=xxxx,max_age=xxxx)

单位为秒

#### 获取cookies

request.COOKIES.get(key)

### Sessions

#### 启用Sessions

Django项目默认启用Session。

#### 存储方式

##### 数据库


存储在数据库中，如下设置可以写，也可以不写，这是默认存储方式。

SESSION_ENGINE='django.contrib.sessions.backends.db'
如果存储在数据库中，需要在项INSTALLED_APPS中安装Session应用。

##### 本地缓存


存储在本机内存中，如果丢失则不能找回，比数据库的方式读写更快。

SESSION_ENGINE='django.contrib.sessions.backends.cache'
#####  混合缓存


优先从本机内存中存取，如果没有则从数据库中存取。

SESSION_ENGINE='django.contrib.sessions.backends.cached_db'
##### Redis


在redis中保存session，需要引入第三方扩展，我们可以使用django-redis来解决。

1） 安装扩展

```
pip install django-redis
```
2）配置

在settings.py文件中做如下设置

```
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```
#### Session操作


通过HttpRequest对象request的session属性进行会话的读写操作。

1） 以键值对的格式写session。

```
request.session['键']=值
```
2）根据键读取值。

```
request.session.get('键',默认值)
```
3）清除所有session，在存储中删除值部分。

```
request.session.clear()
```
4）清除session数据，在存储中删除session的整条数据。

```
request.session.flush()
```
5）删除session中的指定键及值，在存储中只删除某个键及对应的值。

```
del request.session['键']
```
6）设置session的有效期

```
request.session.set_expiry(value)
```
如果value是一个整数，session将在value秒没有活动后过期。
如果value为0，那么用户session的Cookie将在用户的浏览器关闭时过期。
如果value为None，那么session有效期将采用系统默认值，**默认为两周**，可以通过在settings.py中设置SESSION_COOKIE_AGE来设置全局默认值。
### 类视图

使用类视图可以将视图对应的不同请求方式以类中的不同方法来区别定义，配置路由时，使用类视图的as_view()方法来添加。


```
from django.views.generic import View
View提供了as_view()方法,可以自动根据请求方式的不同返回对应的视图函数

```
#### 类视图使用装饰器

##### 在URL中装饰

```
urlpatterns = [
    url(r'^demo/$', my_decorate(DemoView.as_view()))
]
```
##### 在类视图中装饰

* 在类视图的上方装饰


在类视图中使用为函数视图准备的装饰器时，不能直接添加装饰器，需要使用method_decorator将其转换为适用于类视图方法的装饰器。

method_decorator装饰器支持使用name参数指明被装饰的方法


```
# 为全部请求方法添加装饰器
from django.utils.decorators import method_decorator

@method_decorator(my_decorator, name='dispatch')
class DemoView(View):
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')


# 为特定请求方法添加装饰器
@method_decorator(my_decorator, name='get')
class DemoView(View):
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')

```

如果要给类视图多个方法添加装饰器，但不是全部或者某一个方法，可以直接在需要添加装饰器的方法上直接使用method_decorator

```
class DemoView(View):
    @method_decorator(my_decorator)  # 给get添加装饰器
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    @method_decorator(my_decorator) # 给post添加装饰器
    def post(self, request):
        print('post方法')
        return HttpResponse('ok')

    def put(self, request):
        print('put 方法')
        return HttpResponse('ok') # 没有给put增加装饰器
```
### 中间件

在Django处理视图的不同阶段对输入或输出进行干预。

1）中间件的定义方法


定义一个中间件工厂函数，然后返回一个可以别调用的中间件。

中间件工厂函数需要接收一个可以调用的get_response对象。

返回的中间件也是一个可以被调用的对象，并且像视图一样需要接收一个request对象参数，返回一个response对象。

```


def simple_middleware(get_response):
    # 此处编写的代码仅在Django第一次配置和初始化的时候执行一次。

    def middleware(request):
        # 此处编写的代码会在每个请求处理视图前被调用。

        response = get_response(request)

        # 此处编写的代码会在每个请求处理视图之后被调用。

        return response

    return middleware

```


2）中间件的注册

定义好中间件后，需要在settings.py 文件中添加注册中间件

```

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'users.middleware.my_middleware',  # 添加中间件
]
```



3）多个中间件的执行顺序


在请求视图被处理前，中间件由上至下依次执行
在请求视图被处理后，中间件由下至上依次执行
