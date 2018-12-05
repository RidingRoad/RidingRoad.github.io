---
title: namedtuple的妙用
comments: true
date: 2018-11-27 19:17:22
---

上一次介绍了后台开发神器，**sqlacodegen神器**，**一行命令获取数据库所有表的模型类**。
今天介绍**collections里面的一个好用的小函数: namedtuple函数(不创类而可以拥有类的便利)**，例如可以使用**object.attribute**
<!--more-->
### 先看演示
**像类一样的访问属性**
```
from collections import namedtuple

Friend = namedtuple('Friend', ['name', 'gender', 'address', 'star', 'signature'])

RidingRoad = Friend('RidingRoad', 'male', 'Mars', 'The five-star high praise',
                    'Change the world by Program!\n'
                    'Do what you like!\n'
                    'Live what you want!')

print(RidingRoad.name)
print(RidingRoad.gender)
print(RidingRoad.address)
print(RidingRoad.star)
print(RidingRoad.signature)

RidingRoad
male
Mars
The five-star high praise
Change the world by Program!
Do what you like!
Live what you want!
```
### 类似字典的访问
**像字典一样访问items、keys、values**
```
for key, value in RidingRoad.__dict__.items():
    print(key, value)

print("*" * 30)

for key in RidingRoad.__dict__.keys():
    print('{}: '.format(key), eval('RidingRoad.{}'.format(key)))

print("*" * 30)

for value in RidingRoad.__dict__.values():
    print(value)

('name', 'RidingRoad')
('gender', 'male')
('address', 'Mars')
('star', 'The five-star high praise')
('signature', 'Change the world by Program!\nDo what you like!\nLive what you want!')
******************************
('name: ', 'RidingRoad')
('gender: ', 'male')
('address: ', 'Mars')
('star: ', 'The five-star high praise')
('signature: ', 'Change the world by Program!\nDo what you like!\nLive what you want!')
******************************
RidingRoad
male
Mars
The five-star high praise
Change the world by Program!
Do what you like!
Live what you want!

```
### 为什么可以这样？
到这里，你应该会有两个疑问：
1. **为什么有类的影子？**
2. **为什么有字典的影子？**
### 源码解析
#### 为什么有类的影子？
看源码的_class_template部分，其实函数内部为我们创了一个类了
```
# Fill-in the class template
    class_definition = _class_template.format(
        typename = typename,
        field_names = tuple(field_names),
        num_fields = len(field_names),
        arg_list = repr(tuple(field_names)).replace("'", "")[1:-1],
        repr_fmt = ', '.join(_repr_template.format(name=name)
                             for name in field_names),
        field_defs = '\n'.join(_field_template.format(index=index, name=name)
                               for index, name in enumerate(field_names))
    )
    if verbose:
        print class_definition
```
**然后_class_template干了什么？对类进行定义**
```
_class_template = '''\
class {typename}(tuple):
    '{typename}({arg_list})'

    __slots__ = ()

    _fields = {field_names!r}

    def __new__(_cls, {arg_list}):
        'Create new instance of {typename}({arg_list})'
        return _tuple.__new__(_cls, ({arg_list}))

    @classmethod
    def _make(cls, iterable, new=tuple.__new__, len=len):
        'Make a new {typename} object from a sequence or iterable'
        result = new(cls, iterable)
        if len(result) != {num_fields:d}:
            raise TypeError('Expected {num_fields:d} arguments, got %d' % len(result))
        return result

    def __repr__(self):
        'Return a nicely formatted representation string'
        return '{typename}({repr_fmt})' % self

    def _asdict(self):
        'Return a new OrderedDict which maps field names to their values'
        return OrderedDict(zip(self._fields, self))

    def _replace(_self, **kwds):
        'Return a new {typename} object replacing specified fields with new values'
        result = _self._make(map(kwds.pop, {field_names!r}, _self))
        if kwds:
            raise ValueError('Got unexpected field names: %r' % kwds.keys())
        return result

    def __getnewargs__(self):
        'Return self as a plain tuple.  Used by copy and pickle.'
        return tuple(self)

    __dict__ = _property(_asdict)

    def __getstate__(self):
        'Exclude the OrderedDict from pickling'
        pass

{field_defs}
'''
```
#### 为什么有字典的影子？
看源码的 _asdict部分，这里封装成了有序字典，所以我们可以通过\_\_dict\_\_访问字典的特性了
```
__dict__ = _property(_asdict)
 def _asdict(self):
        'Return a new OrderedDict which maps field names to their values'
        return OrderedDict(zip(self._fields, self))
```
### Python全面学习资料
公众号"Python孙行者"后台回复”电子书“即可。如果你有什么好东西好神器，欢迎后台私信，以后总结随文分享，一个人很渺小，有你的参与==>人从众众众众众众众
![](https://i.loli.net/2018/11/27/5bfd27354b6d3.png)




