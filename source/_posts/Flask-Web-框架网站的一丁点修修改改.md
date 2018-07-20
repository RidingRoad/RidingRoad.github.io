---
title: Flask Web 框架网站的一丁点修修改改
comments: true
toc: true
date: 2018-07-20 20:22:19
tags:
- Flask
categories:
- Web框架网站
---
基于已开发好了的Web项目网站，发现一些不太合理的地方，就自己动手修修补补了一下下。
<!--more-->

## 添加的功能

### 添加分类删除
#### 分类现状
目前的后台管理的分类管理只有添加功能，却没有删除功能。
每次测试添加后，分类一多，前端页面显示的分类布局就很不美观。
那么，我们需要方便的删除这些多余的分类，而不是跑去数据删记录。
#### 实现删除分类
路由：/admin/delete_category

##### 前端页面的处理

* 基于“添加”的前端样式和布局，在上面添加“删除”的样式和布局就好
```
<table class="common_table">
				<tr>
					<th width="10%">id</th>
					<th width="80%">类别名称</th>
					<th width="10%">管理操作</th>
				</tr>
				{% for category in data.categories %}
                    <tr>
                        <td>{{ category.id }}</td>
                        <td>{{ category.name }}</td>
                        <td>
                            <a href="javascript:;" class="edit" >编辑</a>
                            <br>
                            <a href="javascript:;" class="delete" style="background:#25c192;display:block;width:40px;line-height:20px;margin:-5px auto;border-radius:4px;color:#fff;">删除</a>
                        </td>
                    </tr>
                {% endfor %}
				<tr>
					<td colspan="3"><a href="javascript:;" class="addtype">增加分类</a>
                    </td>
				</tr>
			</table>
```
* 前端根据点击事件并向后端提供分类的ID以删除
字段        说明

delete_id  所删除的分类id
```
//删除分类
   $delete.click(function () {
       delete_id = $(this).parent().siblings().eq(0).html()
       console.log(delete_id)
       if(delete_id==""){
           $error.html('获取分类id失败').show();
           return;

       }else{
           params={"id":delete_id}
       }

       $.ajax({
           url:"/admin/delete_category",
           method: "post",
           headers: {
               "X-CSRFToken": getCookie("csrf_token")
           },
           data: JSON.stringify(params),
           contentType: "application/json",
           success: function (resp) {
               if (resp.errno == "0") {
                   // 刷新当前界面
                   location.reload();
               }else {
                   $error.html(resp.errmsg).show();
               }
           }
       })

   })
```
##### 后端实现
后端获取到前段提供的id就可以进行数据库的删除操作了
```
@admin_blue.route("/delete_category", methods=["POST"])
def delete_category():
    category_id = request.json.get("id")
    category = None
    try:
        category = Category.query.get(category_id)
    except Exception as e:
        current_app.logger.error(e)
        return jsonify(errno=RET.DBERR, errmsg="查询数据失败")
    try:
        db.session.delete(category)
        db.session.commit()
    except Exception as e:
        current_app.logger.error(e)
        db.session.rollback()
        return jsonify(errno=RET.DBERR, errmsg="删除分类失败")
    return jsonify(errno=RET.OK, errmsg="删除分类成功")
```

### 添加审核状态筛选
#### 审核现状
目前的审核的视图只是返回未审核的数据，但是有很多合情合理的场合需要我们把那些已审核通过的文章给撤回来。
所以我们需要实现审核页面处实现可以提供已通过和未通过的数据，以方便我们一站式处理。
不用跑去数据库去修改新闻的审核状态码，还要记状态码，想想多可怕。
##### 前端页面的处理
###### 前端展示和事件出发
* 在审核状态的地方插入下拉框：未审核；以通过；未通过

```
<th width="5%">

      <label>审核状态</label>
      <select class="status_opt"  name="status_id">

          <option value="wait" {% if data.status == 1 %}selected {% endif %} style="font-size: 12px">未审核</option>
          <option value="pass" {% if data.status == 0 %}selected {% endif %} style="font-size: 12px">已通过</option>
          <option value="not_pass" {% if data.status == -1 %}selected {% endif %} style="font-size: 12px">未通过</option>
      </select>
</th>
```
* 事件触发通过监听js的option 的change改变属性而进行ajax局部刷新
```
$(".status_opt").change(function () {
                    option = $(".status_opt option:selected").val();
                    console.log('在筛选状态下的option:'+option);
                    window.location = '/admin/news_review?p=' + "" + '&status=' + option

                })

```

##### 后端实现

通过获取前端传过来的status进行返回过滤后的数据
```
@admin_blue.route("/news_review", methods=["GET", "POST"])
def news_review():
    page = request.args.get("p", 1)
    keywords = request.args.get("keywords")
    status = request.args.get("status", "wait")
    if status:
        if status == "wait":
            status = 1
        elif status == "pass":
            status = 0
        else:
            status = -1
    try:
        page = int(page)
    except Exception as e:
        current_app.logger.error(e)
        page = 1

    filters = [News.status == status]

```

## 改进的小地方
### 分页点击数据错乱
发现目前的后台搜索返回的分页的数据，一旦点击下一页或者指定页数的时候，返回来的数据就是基于整个数据库的分页的数据，
而不是预期的关键字下的分页的数据。
#### 实现逻辑
这里主要是因为之前的代码是通过点击页面触发了关键字事件查询，但此时传过去的关键字是空了，所以只要我们在搜索前和搜索返回结果后，搜索栏里都有关键字就可以了。
具体实现：后端把前端传过来的关键字放到response的数据里，前端取出关键字放到搜索栏里，当下一次点击第几页的时候，前端同时把关键字传输到后端。
##### 前端实现
```
$("#pagination").pagination({
                    currentPage: {{ data.current_page }},
                    totalPage: {{ data.total_page }},
                    callback: function(current) {
                        window.location.href = '/admin/news_review?p=' + current + '&keywords=' + input_text + "&status=" + option
                    }

                });
```
##### 后端实现
```
data = {
       "news_list": news_list,
       "current_page": current_page,
       "total_page": total_page,
       "status": status,
       "keywords": keywords
   }
```
### 阻止自恋
目前用户关注这块，发现可以自己玩自己，咦，还可以这样玩？
其实，**就是一个bug**，好的，就修复一下吧，这个很简单。
#### 实现逻辑
只要判断用户的id和被关注的id是否相同即可
```
if action == "follow":
    if int(user_id) == g.user.id:
        return jsonify(errno=RET.DATAEXIST, errmsg="自恋,在这是不被许可的")
```
