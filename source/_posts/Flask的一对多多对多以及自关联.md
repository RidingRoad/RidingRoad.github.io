---
title: Flask的一对多多对多以及自关联
comments: true
toc: true
date: 2018-08-06 21:49:36
categories:
- Python
tags:
- Flask
---
Flask的一对多多对多以及自关联的总结<!--more-->
在Flask中创建数据模型类，需要继续自flask_sqlalchemy.SQLAchemy().models.Model
```
from flask_sqlalchemy import SQLAchemy
db = SQLAchemy(flask_app).models.Model
```

### 一对多
#### 一对多模型的实现
一对多的模型类实现可以通过db.relationship（在一的一方）和db.ForeignKey（在多的一方）.比如一个作者可以有多篇文章
```
class Author(db.Model):
    __tablename__ = 'info_author'
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(20), nullable=False)
    # 一的一方，relationship为Author 添加article属性，Author_obj.article内容是以Author_obj.id == Article.author_id的一组Article对象
    # backref（反向引用） 则为Article添加author属性，Article_obj.author内容是以Article.author_id == Author_obj.id 的Author_obj
    article = db.relationship("Article",backref='author',lazy='dynamic')
    """
    lazy: 指定sqlalchemy数据库什么时候加载数据
        select: 就是访问到属性的时候，就会全部加载该属性的数据
        joined: 对关联的两个表使用联接
        subquery: 与joined类似，但使用子子查询
        dynamic: 不加载记录，但提供加载记录的查询，也就是生成query对象
    """


class Article(db.Model):
    __tablename__ = "info_article"
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(20),nullable=False)
    # 多的一方，author_id 的取值范围只能在info_author.id的范围内
    author_id = db.Column(db.Integer,db.ForeignKey('info_author.id'))

```
#### 一对多和多对一的关联查询使用
在flask_sqlalchemy中，插入、修改、删除操作，均由数据库会话管理。
会话用 db.session 表示。在准备把数据写入数据库前，要先将数据添加到会话中然后调用 commit() 方法提交会话。
在 flask_sqlalchemy中，查询操作是通过 query 对象操作数据。
最基本的查询是返回表中所有数据，可以通过过滤器进行更精确的数据库查询。
#### 数据库迁移
```
from flask_migrate import Migrate, MigrateCommand
from flask_script import Manager
app = Flask(__name__)
manager = Manager(app) # 脚本命令
migrate = Migrate(app,db) # 数据库迁移绑定
manager.add_command('mysql',MigrateCommand) # 添加数据库迁移的脚本命令
```
迁移三部曲：
* pyhton manage.py **migrate_command** **init** # 这里的migrate_command即是上面的‘mysql’
* python manage.py **migrate_command** **migrate** -m "database_migrate"
* python manage.py **migrate_command** **upgrade**
#### 查找一个作者的所有文章
```
Author.query.filter(Author.id==1).first().article.all()
# 返回一个查询集列表
```
#### 查找某篇文章的作者
```
author = Article.query.filter(Article.title=='Python孙行者').first().author
# 返回的是一个作者查询集对象，可以author.name进行查询具体的名字
```
### 多对多
多对多一般使用一张中间表和两个一对多进而降低难度和更容易理解。下面以用户收藏新闻为例，一个用户可以收藏多条新闻，一条新闻可以被多个用户收藏，那么这个就是一个多对多的关系。
我们可以通过一张用户表（用户id，姓名，用户收藏的新闻的一个关联属性），一张新闻表（新闻id，新闻标题）以及一张中间表（用户id，新闻id）。
用户表：
|id|name|
|-----|----|
|1|张三|
|2|李四|
|3|王五|
新闻表：
|id|title|
|-----|----|
|1|六一|
|2|七一|
|3|十一|

中间表：
|user_id|news_id|
|-----|----|
|1|1|
|1|2|
|3|1|
比如说，需要查找张三收藏了哪些新闻，那么我们就可以根据张三的用户id到中间表查询user_id为1的所有的news_id的数据，然后再回到新闻表，去根据news_id就可以查到对应的标题了。
代码：
```
# 中间表
tb_user_collections = db.Table("info_user_collections",
                                                 db.Cloumn("user_id", db.Integer, db.ForeignKey("info_user.id")),
                                                 db.Cloumn("news_id", db.Integer, db.ForgignKey("info_news.id")))
# 用户模型
class User(db.Model):
    __tablename__ = "info_user"

    id = db.Column(db.Integer, primary_key=True)  # 用户编号
    name = db.Column(db.String(32), unique=True, nullable=False)  # 用户昵称
    collection_news = db.relationship("News", secondary=tb_user_collections, backref="user", lazy="dynamic")   
# 新闻模型
class News(db.Model):
        __tablename__ = "info_news"       
    id = db.Column(db.Integer, primary_key=True)  # 新闻id   
    title = db.Column(db.String(32), nullable=False) # 新闻标题
```

### 自关联
自关联最常见就是评论盖楼和地址的三级联动了，下面一评论盖楼为例：
一个评论下面有很多追加的评论，这是可以通过记录父评论的id就可以一环扣一环形成完整了评论楼层
|id|content|parent_id|
|-----|----|----|
|1|哈哈|NULL|
|2|6666|1|
|3|88888|1|

```
class Comment(db.Model):
    """评论"""
    __tablename__ = "info_comment"
    id = db.Column(db.Integer, primary_key=True)  # 评论编号
    content = db.Column(db.Text, nullable=False)  # 评论内容
    parent_id = db.Column(db.Integer, db.ForeignKey("info_comment.id"))  # 父评论id
    parent = db.relationship("Comment", remote_side=[id])  # 自关联
```
