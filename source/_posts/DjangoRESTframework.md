---
title: DjangoRESTframework
comments: true
date: 2018-07-24 21:49:20
toc: true
categories:
- Django
tags:
- DRF
---


Django REST Framework是基于Django进行的二次封装，提供快速序列化以及丰富的视图，实现快速的符合RESTful 风格的API开发。
<!---more-->

### RESTful设计方法

* 域名

* 版本

* 路径

(1) 资源作为网址，只能有名词，不能有动词，而且所用的名词往往与数据库的表名对应。

(2) API中的名词应该使用复数。无论子资源或者所有资源。

* HTTP动词


常用的HTTP动词有下面四个（括号里是对应的SQL命令）。

GET（SELECT）：从服务器取出资源（一项或多项）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
DELETE（DELETE）：从服务器删除资源。
还有三个不常用的HTTP动词。

PATCH（UPDATE）：在服务器更新(更新)资源（客户端提供改变的属性）。
HEAD：获取资源的元数据。
OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
* 过滤信息

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果


?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&per_page=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?animal_type_id=1：指定筛选条件
* 状态码

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

200 OK - [GET]：服务器成功返回用户请求的数据
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
* 错误处理

如果状态码是4xx，服务器就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。

```
{
    error: "Invalid API key"
}
```
* 返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范。

GET /collection：返回资源对象的列表（数组）
GET /collection/resource：返回单个资源对象
POST /collection：返回新生成的资源对象
PUT /collection/resource：返回完整的资源对象
PATCH /collection/resource：返回完整的资源对象
DELETE /collection/resource：返回一个空文档
* 超媒体

* 数据格式

一般返回json格式数据

#### RESTful开发核心任务

将请求的数据（如JSON格式）转换为模型类对象  
操作数据库
将模型类对象转换为响应的数据（如JSON格式）

简而言之，我们可以将序列化理解为：

将程序中的一个数据结构类型转换为其他格式（字典、JSON、XML等），例如将Django中的模型类对象装换为JSON字符串，这个转换过程我们称为序列化。

反之，将其他格式（字典、JSON、XML等）转换为程序中的数据，例如将JSON字符串转换为Django中的模型类对象，这个过程我们称为反序列化。

### 工程搭建


1.安装DRF

```
pip install djangorestframework
```
2. 注册rest_framework应用

我们利用在Django框架学习中创建的demo工程，在settings.py的INSTALLED_APPS中添加'rest_framework'

```
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```


### 序列化

#### rest_framework.serializers.Serializer

* 定义序列化器

继承 rest_framework.serializers.Serializer

```


class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    id = serializers.IntegerField(label='ID', read_only=True)
    btitle = serializers.CharField(label='名称', max_length=20)
    bpub_date = serializers.DateField(label='发布日期', required=False)
    bread = serializers.IntegerField(label='阅读量', required=False)
    bcomment = serializers.IntegerField(label='评论量', required=False)
    image = serializers.ImageField(label='图片', required=False)
```

* 构造序列化器对象


```
from booktest.serializers import BookInfoSerializer
book_qs = BookInfo.objects.all()


serializer = BookInfoSerializer(book_qs, many=True)

```
* 查看序列化器对象的数据

serializer.data

##### 关联对象序列化

* PrimaryKeyRelatedField

```
serializers.PrimaryKeyRelatedField(label='图书', queryset=BookInfo.objects.all()[,read_only=True])
包含read_only=True参数时，该字段将不能用作反序列化使用
包含queryset参数时，将被用作反序列化时参数校验使用
```
* StringRelatedField

此字段将被序列化为关联对象的字符串表示方式（即__str__方法的返回值）


```
hbook = serializers.StringRelatedField(label='图书')
```
* 使用关联对象的序列化器

```
hbook = BookInfoSerializer()
```

* HyperlinkedRelatedField

此字段将被序列化为获取关联对象数据的接口链接


```
hbook = serializers.HyperlinkedRelatedField(label='图书', read_only=True, view_name='books-detail')
```
* SlugRelatedField

此字段将被序列化为关联对象的指定字段数据


```
hbook = serializers.SlugRelatedField(label='图书', read_only=True, slug_field='bpub_date')
```
### 反序列化

使用序列化器进行反序列化时，需要对数据进行验证后，才能获取验证成功的数据或保存成模型类对象。

在获取反序列化的数据前，必须调用is_valid()方法进行验证，验证成功返回True，否则返回False。

验证失败，可以通过序列化器对象的errors属性获取错误信息，返回字典，包含了字段和字段的错误。如果是非字段错误，可以通过修改REST framework配置中的NON_FIELD_ERRORS_KEY来控制错误字典中的键名。

验证成功，可以通过序列化器对象的validated_data属性获取数据。

在定义序列化器时，指明每个字段的序列化类型和选项参数，本身就是一种验证行为。

#### 验证

##### 序列化器字段验证

通过构造序列化器对象，并将要反序列化的数据传递给data构造参数，进而进行验证


data = {'btitle': 'python'}
serializer = BookInfoSerializer(data=data)
serializer.is_valid(raise_exception=True)  # True
serializer.errors  # {}
serializer.validated_data  #  OrderedDict([('btitle', 'python')])
is_valid()方法还可以在验证失败时抛出异常serializers.ValidationError，可以通过传递raise_exception=True参数开启，REST framework接收到此异常，会向前端返回HTTP 400 Bad Request响应。

##### 自定义验证行为

###### validate_<field_name>  单个字段验证


对<field_name>字段进行验证，如

```
class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    ...

    def validate_btitle(self, value):
        if 'django' not in value.lower():
            raise serializers.ValidationError("图书不是关于Django的")
        return value

```
###### validate  多个字段进行验证


在序列化器中需要同时对多个字段进行比较验证时，可以定义validate方法来验证，如

```
class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    ...

    def validate(self, attrs):
        bread = attrs['bread']
        bcomment = attrs['bcomment']
        if bread < bcomment:
            raise serializers.ValidationError('阅读量小于评论量')
        return attrs

```
###### validators  在序列化器的字段添加验证

在字段中添加validators选项参数，也可以补充验证行为

validators = [func]

```


def about_django(value):
    if 'django' not in value.lower():
        raise serializers.ValidationError("图书不是关于Django的")

class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    id = serializers.IntegerField(label='ID', read_only=True)
    btitle = serializers.CharField(label='名称', max_length=20, validators=[about_django])
    bpub_date = serializers.DateField(label='发布日期', required=False)
    bread = serializers.IntegerField(label='阅读量', required=False)
    bcomment = serializers.IntegerField(label='评论量', required=False)
    image = serializers.ImageField(label='图片', required=False)

```
#### 保存

如果在验证成功后，想要基于validated_data完成数据对象的创建，可以通过在序列化器实现create()和update()两个方法来实现。


```
class BookInfoSerializer(serializers.Serializer):
    """图书数据序列化器"""
    ...

    def create(self, validated_data):
        """新建"""
        return BookInfo.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """更新，instance为要更新的对象实例"""
        instance.btitle = validated_data.get('btitle', instance.btitle)
        instance.bpub_date = validated_data.get('bpub_date', instance.bpub_date)
        instance.bread = validated_data.get('bread', instance.bread)
        instance.bcomment = validated_data.get('bcomment', instance.bcomment)
        instance.save()
        return instance
```


实现了上述两个方法后，在反序列化数据的时候，就可以通过save()方法返回一个数据对象实例了

```
book = serializer.save()
```
如果创建序列化器对象的时候，没有传递instance实例，则调用save()方法的时候，create()被调用，相反，如果传递了instance实例，则调用save()方法的时候，update()被调用。

```
from db.serializers import BookInfoSerializer
data = {'btitle': '封神演义'}
serializer = BookInfoSerializer(data=data)
serializer.is_valid()  # True
serializer.save()  # <BookInfo: 封神演义>

from db.models import BookInfo
book = BookInfo.objects.get(id=2)
data = {'btitle': '倚天剑'}
serializer = BookInfoSerializer(book, data=data)
serializer.is_valid()  # True
serializer.save()  # <BookInfo: 倚天剑>
book.btitle  # '倚天剑'
```

两点说明：

1） 在对序列化器进行save()保存时，可以额外传递数据，这些数据可以在create()和update()中的validated_data参数获取到

```
serializer.save(owner=request.user)
```
2）默认序列化器必须传递所有required的字段，否则会抛出验证异常。但是我们可以使用**partial参数来允许部分字段更新**

```
Update `comment` with partial data
serializer = CommentSerializer(comment, data={'content': u'foo bar'}, partial=True)

```
#### 模型类序列化器ModelSerializer


如果我们想要使用序列化器对应的是**Django的模型类**，DRF为我们提供了ModelSerializer模型类序列化器来帮助我们快速创建一个Serializer类。

ModelSerializer与常规的Serializer相同，但提供了：

基于模型类自动生成一系列字段
基于模型类自动为Serializer生成validators，比如unique_together
包含默认的create()和update()的实现
##### 定义模型类序列化器


比如我们创建一个BookInfoSerializer

```
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = '__all__'   或者使用元组  ('btitle', 'bpub_date')
```
model 指明参照哪个模型类
fields 指明为**模型类的哪些字段生成**

##### 指定字段

1) 使用fields来明确字段，__all__表名包含所有字段，也可以写明具体哪些字段，如

```
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = ('id', 'btitle', 'bpub_date')  元组类型

```
2) 使用exclude可以明确排除掉哪些字段

```
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        exclude = ('image',)  元组类型

```
3) 默认ModelSerializer使用主键作为关联字段，但是我们可以使用depth来简单的生成嵌套表示，depth应该是整数，表明嵌套的层级数量。如：

```
class HeroInfoSerializer2(serializers.ModelSerializer):
    class Meta:
        model = HeroInfo
        fields = '__all__'
        depth = 1
```
形成的序列化器如下：

```
HeroInfoSerializer():
    id = IntegerField(label='ID', read_only=True)
    hname = CharField(label='名称', max_length=20)
    hgender = ChoiceField(choices=((0, 'male'), (1, 'female')), label='性别', required=False, validators=[<django.core.valators.MinValueValidator object>, <django.core.validators.MaxValueValidator object>])
    hcomment = CharField(allow_null=True, label='描述信息', max_length=200, required=False)
    hbook = NestedSerializer(read_only=True):
        id = IntegerField(label='ID', read_only=True)
        btitle = CharField(label='名称', max_length=20)
        bpub_date = DateField(allow_null=True, label='发布日期', required=False)
        bread = IntegerField(label='阅读量', max_value=2147483647, min_value=-2147483648, required=False)
        bcomment = IntegerField(label='评论量', max_value=2147483647, min_value=-2147483648, required=False)
        image = ImageField(allow_null=True, label='图片', max_length=100, required=False)
```
4) 指明显示字段，如：

```
class HeroInfoSerializer(serializers.ModelSerializer):
    hbook = BookInfoSerializer()

    class Meta:
        model = HeroInfo
        fields = ('id', 'hname', 'hgender', 'hcomment', 'hbook')

```
5) 指明只读字段

可以通过read_only_fields指明只读字段，即仅用于序列化输出的字段

```
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = ('id', 'btitle', 'bpub_date'， 'bread', 'bcomment')
        read_only_fields = ('id', 'bread', 'bcomment')

```
#####  添加额外参数

我们可以使用extra_kwargs参数为ModelSerializer添加或修改原有的选项参数

```
class BookInfoSerializer(serializers.ModelSerializer):
    """图书数据序列化器"""
    class Meta:
        model = BookInfo
        fields = ('id', 'btitle', 'bpub_date', 'bread', 'bcomment')
        extra_kwargs = {
            'bread': {'min_value': 0, 'required': True},
            'bcomment': {'min_value': 0, 'required': True},
        }
BookInfoSerializer():
id = IntegerField(label='ID', read_only=True)
btitle = CharField(label='名称', max_length=20)
bpub_date = DateField(allow_null=True, label='发布日期', required=False)
bread = IntegerField(label='阅读量', max_value=2147483647, min_value=0, required=True)
bcomment = IntegerField(label='评论量', max_value=2147483647, min_value=0, required=True)
```



### 视图

#### DRF的Request对象



1）REST framework 提供了Parser解析器，在接收到请求后会自动根据Content-Type指明的请求数据类型（如JSON、表单等）将请求数据进行parse解析，解析为类字典对象保存到Request对象中。)

2）Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果。

3）无论前端发送的哪种格式的数据，我们都可以以统一的方式读取数据。



##### 常用属性

1）.data  请求体

request.data 返回解析之后的请求体数据。类似于Django中标准的request.POST和request.FILES属性，但提供如下特性：

包含了解析之后的文件和非文件数据
包含了对POST、PUT、PATCH请求方式解析后的数据
利用了REST framework的parsers解析器，不仅支持表单类型数据，也支持JSON数据
2）.query_params  查询字符串

request.query_params与Django标准的request.GET相同，只是更换了更正确的名称而已。



#### DRF的Response对象



```

rest_framework.response.Response

```

REST framework提供了一个响应类Response，使用该类构造响应对象时，响应的具体数据内容会被转换（render渲染）成符合前端需求的类型。

REST framework提供了Renderer 渲染器，用来根据请求头中的Accept（接收数据类型声明）来自动转换响应数据到对应格式。如果前端请求中未进行Accept声明，则会采用默认方式处理响应数据，我们可以通过配置来修改默认响应格式。

```
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (  # 默认响应渲染类
        'rest_framework.renderers.JSONRenderer',  # json渲染器
        'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览API渲染器
    )
}
```


##### 构造方式

```
Response(data, status=None, template_name=None, headers=None, content_type=None)
```


参数说明：

data: 为响应准备的序列化处理后的数据；
status: 状态码，默认200；
template_name: 模板名称，如果使用HTMLRenderer 时需指明；
headers: 用于存放响应头信息的字典；
content_type: 响应数据的Content-Type，通常此参数无需传递，REST framework会根据前端所需类型数据来设置该参数。


##### 常用属性

1）.data

传给response对象的序列化后，但尚未render处理的数据

2）.status_code

状态码的数字

3）.content

经过render处理后的响应数据

#### RDF状态码

为了方便设置状态码，REST framewrok在rest_framework.status模块中提供了常用状态码常量。


#### 两个基类视图

##### APIView

1）支持定义的属性：

authentication_classes 列表或元组，身份认证类
permissoin_classes 列表或元组，权限检查类
throttle_classes 列表或元组，流量控制类
##### GenericAPIViewe

继承自APIVIew，增加了对于列表视图和详情视图可能用到的通用支持方法。通常使用时，可搭配一个或多个Mixin扩展类。


1）支持定义的属性：

* 列表视图与详情视图通用：
queryset 列表视图的查询集
serializer_class 视图使用的序列化器
* 列表视图使用：
pagination_class 分页控制类
filter_backends 过滤控制后端
* 详情页视图使用：
lookup_field **查询单一数据库对象时使用的条件字段，默认为'pk'**
lookup_url_kwarg 查询单一数据时URL中的参数关键字名称，默认与look_field相同)



2）提供的方法：

* 列表视图与详情视图通用：

get_queryset(self)

返回视图使用的查询集，是列表视图与详情视图获取数据的基础，默认返回queryset属性，可以重写，例如：

```
def get_queryset(self):
    user = self.request.user
    return user.accounts.all()

```
get_serializer_class(self)

返回序列化器类，默认返回serializer_class，可以重写，例如：

```
def get_serializer_class(self):
    if self.request.user.is_staff:
        return FullAccountSerializer
    return BasicAccountSerializer

```
get_serializer(self, args, *kwargs)

返回序列化器对象，被其他视图或扩展类使用，如果我们在视图中想要获取序列化器对象，可以直接调用此方法。

注意，在提供序列化器对象的时候，REST framework会向对象的context属性补充三个数据：request、format、view，这三个数据对象可以在定义序列化器时使用。

* 详情视图使用：

get_object(self) 返回详情视图所需的模型类数据对象，默认使用**lookup_field参数**来过滤queryset。 在试图中可以调用该方法获取详情信息的模型类对象。

若详情访问的模型类对象不存在，会返回404。

该方法会默认使用APIView提供的check_object_permissions方法检查当前对象是否有权限被访问。




#### 5个扩展类视图,一般结合GenericAPIView

1) ListModelMixin

列表视图扩展类，提供list(request, \*args, \*\*kwargs)方法快速实现列表视图，返回200状态码。

该Mixin的list方法会对数据进行过滤和分页


2) CreateModelMixin


创建视图扩展类，提供create(request, \*args, \*\*kwargs)方法快速实现创建资源的视图，成功返回201状态码。

如果序列化器对前端发送的数据验证失败，返回400错误


3） RetrieveModelMixin

详情视图扩展类，提供retrieve(request, \*args, \*\*kwargs)方法，可以快速实现返回一个存在的数据对象。

如果存在，返回200， 否则返回404。

源代码：

```
class RetrieveModelMixin(object):
    """
    Retrieve a model instance.
    """
    def retrieve(self, request, *args, **kwargs):
        # 获取对象，会检查对象的权限
        instance = self.get_object()
        # 序列化
        serializer = self.get_serializer(instance)
        return Response(serializer.data)

```
举例：

```
class BookDetailView(RetrieveModelMixin, GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    def get(self, request, pk):
        return self.retrieve(request)

```
4）UpdateModelMixin

更新视图扩展类，提供update(request, \*args, \*\*kwargs)方法，可以快速实现更新一个存在的数据对象。

同时也提供partial_update(request, \*args, \*\*kwargs)方法，可以实现局部更新。

成功返回200，序列化器校验数据失败时，返回400错误。

源代码：

```
class UpdateModelMixin(object):
    """
    Update a model instance.
    """
    def update(self, request, *args, **kwargs):
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        self.perform_update(serializer)

        if getattr(instance, '_prefetched_objects_cache', None):
            # If 'prefetch_related' has been applied to a queryset, we need to
            # forcibly invalidate the prefetch cache on the instance.
            instance._prefetched_objects_cache = {}

        return Response(serializer.data)

    def perform_update(self, serializer):
        serializer.save()

    def partial_update(self, request, *args, **kwargs):
        kwargs['partial'] = True
        return self.update(request, *args, **kwargs)

```
5）DestroyModelMixin

删除视图扩展类，提供destroy(request, \*args, \*\*kwargs)方法，可以快速实现删除一个存在的数据对象。

成功返回204，不存在返回404。

源代码：

```
class DestroyModelMixin(object):
    """
    Destroy a model instance.
    """
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        self.perform_destroy(instance)
        return Response(status=status.HTTP_204_NO_CONTENT)

    def perform_destroy(self, instance):
        instance.delete()

```
#### 7个可用子类视图

1） CreateAPIView

提供 post 方法

继承自： GenericAPIView、CreateModelMixin

2）ListAPIView

提供 get 方法

继承自：GenericAPIView、ListModelMixin

3）RetireveAPIView

提供 get 方法

继承自: GenericAPIView、RetrieveModelMixin

4）DestoryAPIView

提供 delete 方法

继承自：GenericAPIView、DestoryModelMixin

5）UpdateAPIView

提供 put 和 patch 方法

继承自：GenericAPIView、UpdateModelMixin

6）RetrieveUpdateAPIView

提供 get、put、patch方法

继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin

7）RetrieveUpdateDestoryAPIView

提供 get、put、patch、delete方法

继承自：GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin
#### 视图集

视图集只在使用as_view()方法的时候，才会将action动作与具体请求方式对应上。如：

```
class BookInfoViewSet(viewsets.ViewSet):

    def list(self, request):
        ...

    def retrieve(self, request, pk=None):
        ...

```
在设置路由时，我们可以如下操作

```
urlpatterns = [
    url(r'^books/$', BookInfoViewSet.as_view({'get':'list'}),
    url(r'^books/(?P<pk>\d+)/$', BookInfoViewSet.as_view({'get': 'retrieve'})
]
```
##### ViewSetMixin

```

class ViewSetMixin(object):
    """
    This is the magic.

    Overrides `.as_view()` so that it takes an `actions` keyword that performs
    the binding of HTTP methods to actions on the Resource.

    For example, to create a concrete view binding the 'GET' and 'POST' methods
    to the 'list' and 'create' actions...

    view = MyViewSet.as_view({'get': 'list', 'post': 'create'})
    """
```

#####  rest_framework.viewsets.ViewSet

继承自 rest_framework.views.APIView 和rest_framework.viewsets.ViewSetMixin

##### rest_framework.viewsets.GenericViewSet

继承自rest_framework.generic.GenericAPIView和rest_framework.viewsets.ViewSetMixin

##### rest_framework.viewsets.ModelViewSet

继承自GenericAPIVIew，同时包括了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin。

##### rest_framework.viewsets.ReadOnlyModelViewSet

继承自GenericAPIVIew，同时包括了ListModelMixin、RetrieveModelMixin。

##### 视图集中自定义action动作


在视图集中，除了上述默认的方法动作外，还可以添加自定义动作。

添加自定义动作需要使用rest_framework.decorators.action装饰器。

以action装饰器装饰的方法名会作为action动作名，与list、retrieve等同。

action装饰器可以接收两个参数：

methods: 该action支持的请求方式，列表传递
detail: 表示是action中要处理的是否是视图资源的对象（即是否通过url路径获取主键）
True 表示使用通过URL获取的主键对应的数据对象
False 表示不使用URL获取主键
举例：

```
from rest_framework import mixins
from rest_framework.viewsets import GenericViewSet
from rest_framework.decorators import action

class BookInfoViewSet(mixins.ListModelMixin, mixins.RetrieveModelMixin, GenericViewSet):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    detail为False 表示不需要处理具体的BookInfo对象
    @action(methods=['get'], detail=False)
    def latest(self, request):
        """
        返回最新的图书信息
        """
        book = BookInfo.objects.latest('id')
        serializer = self.get_serializer(book)
        return Response(serializer.data)

     detail为True，表示要处理具体与pk主键对应的BookInfo对象
    @action(methods=['put'], detail=True)
    def read(self, request, pk):
        """
        修改图书的阅读量数据
        """
        book = self.get_object()
        book.bread = request.data.get('read')
        book.save()
        serializer = self.get_serializer(book)
        return Response(serializer.data)

```
url的定义

```
urlpatterns = [
    url(r'^books/$', views.BookInfoViewSet.as_view({'get': 'list'})),
    url(r'^books/latest/$', views.BookInfoViewSet.as_view({'get': 'latest'})),
    url(r'^books/(?P<pk>\d+)/$', views.BookInfoViewSet.as_view({'get': 'retrieve'})),
    url(r'^books/(?P<pk>\d+)/read/$', views.BookInfoViewSet.as_view({'put': 'read'})),
]
```
### 路由

对于视图集ViewSet，我们除了可以自己手动指明请求方式与动作action之间的对应关系外，还可以使用Routers来帮助我们快速实现路由信息。

REST framework提供了两个router

SimpleRouter
DefaultRouter


####  使用方法

1） 创建router对象，并注册视图集，例如

```
from rest_framework import routers

router = routers.SimpleRouter()
router.register(r'books', BookInfoViewSet, base_name='book')

```
register(prefix, viewset, base_name)

prefix 该视图集的路由前缀
viewset 视图集
base_name 路由名称的前缀
如上述代码会形成的路由如下：

```
^books/$    name: book-list
^books/{pk}/$   name: book-detail
```
2）添加路由数据

可以有两种方式：

```
urlpatterns = [
    ...
]
urlpatterns += router.urls
```
或

```
urlpatterns = [
    ...
    url(r'^', include(router.urls))
]
```
#### 视图集中包含附加action的

```
class BookInfoViewSet(mixins.ListModelMixin, mixins.RetrieveModelMixin, GenericViewSet):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    @action(methods=['get'], detail=False)
    def latest(self, request):
        ...

    @action(methods=['put'], detail=True)
    def read(self, request, pk):
        ...

```
此视图集会形成的路由：

```
^books/latest/$    name: book-latest
^books/{pk}/read/$  name: book-read
```
