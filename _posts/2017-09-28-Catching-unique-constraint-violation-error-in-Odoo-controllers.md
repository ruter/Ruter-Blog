---
title: 在 Odoo 控制器中捕获数据唯一性约束错误
categories: Odoo
keywords: Odoo
toc: false
date: 2017-09-28 11:21:15
description: 捕获 Odoo 中的约束错误
---

开发中经常会有的一个需求就是对数据字段做唯一性约束，在 `Odoo` 中为模型添加约束进行数据校验有两种方法：在 Python 程序中用`@api.constrains` 装饰器方法定义需要校验的字段；使用 `_sql_constraints` 属性添加 SQL 约束。一般来说，如果只是做唯一性约束，直接用后者即可，方便快速。

当违反约束时，程序会抛出一个 `psycopg2.IntegrityError` 错误，在这种情况我们要对这个错误进行捕获并处理，这篇文章将以一个小例子来教你如何在 controllers 中对这个错误进行捕获，别担心，一切看起来都非常简单 :)

# 创建模型添加约束

首先创建一个简单的用户名模型，并为其 `username` 字段添加唯一性约束：

```python
from odoo import models, fields, api


class MyUsername(models.Model):
  _name = 'my.username'
  _description = 'A simple username model.'
  
  username = fields.Char('Username')
  
  # Add SQL Constraints
  _sql_constraints = [
      ('username_uniq', 'unique (username)', 'The username already exists!')
  ]
```

# 创建控制器

这里判断请求类型是不是 `POST` ，如果是则创建一条数据，否则显示提交表单的页面：

```python
from odoo import http
from psycopg2 import IntegrityError


class MyUsernameController(http.Controller):
  @http.route('/myusername/create', type='http', auth='public', website=True)
  def create(self, **kwargs):
    if http.request.httprequest.method == 'POST':
      username = kwargs.get('username', None)
      try:
        http.request.env['my.username'].create({
            'username': username
        })
        return http.request.render('my.ok', {
            'username': username
        })
      except IntegrityError:
        # IntegrityError handler
        http.request._cr.rollback()
        return http.request.render('my.error')
    return http.request.render('my.home')
```

# 创建模板

这里只是做简单的演示，只要创建一个简单的用于提交数据的表单即可，然后另外成功后我们显示提交的用户名，抛出约束错误后显示错误：

```xml
<template id="home">
    <form class="oe_login_form" role="form" action="/myusername/create" method="post">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <input type="text" name="username"/>
        <button type="submit" class="btn btn-default">Submit</button>
    </form>
</template>

<template id="ok">
    <span>hello, <t t-esc="username"/></span>
</template>

<template id="error">
    <span>IntegrityError!</span>
</template>
```

到这里就已经 OK 了，安装之后打开 `http://localhost:8069/myusername/create` 可以看到一个输入框，输入一个名字然后提交，会显示 `hello, <username>` ，其中 `<username>` 是你输入的名字，然后再重新打开同一个链接，再次输入相同的名字提交，这次会发现抛出了错误提示 `IntegrityError!` 。

我们在示例里捕获错误之后，简单地进行了回滚 `rollback()` 操作，而你在捕获错误之后应该根据需求进行相应的处理。这里要注意的是不能使用 `http.request.env.cr` ( 或者其他任何可能涉及数据库操作的方法和属性 ) 进行操作，因为 `env` ( 这些操作 ) 会对数据库进行查询操作，而在抛出错误后这一事务还没结束，是不能对数据库进行操作的。

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://ruterly.com/)」原创。转载务必注明[出处](http://ruterly.com/2017/09/28/Catching-unique-constraint-violation-error-in-Odoo-controllers/)。

本文永久地址：http://ruterly.com/2017/09/28/Catching-unique-constraint-violation-error-in-Odoo-controllers/
