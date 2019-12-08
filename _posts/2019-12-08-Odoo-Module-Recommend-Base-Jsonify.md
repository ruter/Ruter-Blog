---
layout: post
title: Odoo 模块推荐——Base Jsonify
categories: Odoo
description: 在 ORM 中添加 jsonify 方法用于解析记录集
keywords: Odoo, 模块
---

如果你平时需要用 Odoo 处理一些接口，并且总是有很多的关系字段和日期字段需要处理，那你可能不会想要错过这个模块的。

# 模块信息

|          |                                                              | 备注                  |
| -------- | ------------------------------------------------------------ | --------------------- |
| 名称     | Base Jsonify                                                 |                       |
| 功能     | 这个模块在 ORM 中添加了 `jsonify()` 方法，它接受一个参数 `parser` 用于决定如何解析以及解析哪些字段 |                       |
| 商店地址 | [点击前往](https://www.odoo.com/apps/modules/8.0/base_jsonify/) | 商店中最高版本为 8.0  |
| 仓库地址 | [前往 GitHub](https://github.com/OCA/server-tools/tree/12.0/base_jsonify) | 仓库中最高版本为 13.0 |

# 使用体验

安装好模块之后，我们来打开 `Odoo Shell` 试试看它的功能，使用以下命令进入 Odoo Shell，请替换尖括号的内容为你当前所使用环境对应的值

```shell
cd </path/to/odoo/source>
python3 odoo-bin shell -c <odoo.conf> -d <db_name>
```

正确执行后将会进入到初始化后可交互的 Shell 环境中

```shell
2019-12-08 09:10:18,049 220 INFO db_name odoo.modules.loading: 11 modules loaded in 0.43s, 0 queries 
2019-12-08 09:10:18,220 220 INFO db_name odoo.modules.loading: Modules loaded. 
env: <odoo.api.Environment object at 0x7f69cfaadf28>
odoo: <module 'odoo' from '/opt/odoo/sources/odoo/odoo/__init__.py'>
openerp: <module 'odoo' from '/opt/odoo/sources/odoo/odoo/__init__.py'>
self: res.users(1,)
Python 3.5.2 (default, Oct  8 2019, 13:06:37) 
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
(Console)
>>>
```

接下来我们直接用当前环境中的用户数据作为演示，看看这个模块的效果

```shell
>>> self
res.users(1,)
>>> self.name
'System'
>>> self.read(['name', 'lang', 'create_date', 'company_id', 'groups_id'])
[{'id': 1, 'lang': 'zh_CN', 'name': 'System', 'create_date': datetime.datetime(2019, 11, 4, 15, 3, 17, 459424), 'company_id': (1, 'My Company'), 'groups_id': [1, 6, 7, 3, 2]}]
>>> parser = [
...   'name',
...   'lang',
...   'create_date:creationDate',
...   ('company_id:company', ['id', 'name']),
...   ('groups_id:groups', ['id', 'name'])
... ]
>>> self.jsonify(parser)
[{'lang': 'zh_CN', 'creationDate': '2019-11-04T15:03:17.459424+00:00', 'name': 'System', 'company': {'id': 1, 'name': 'My Company'}, 'groups': [{'id': 1, 'name': '\u5185\u90e8\u7528\u6237'}, {'id': 6, 'name': '\u6280\u672f\u7279\u6027'}, {'id': 7, 'name': '\u8054\u7cfb\u4eba\u521b\u5efa'}, {'id': 3, 'name': '\u8bbe\u7f6e'}, {'id': 2, 'name': '\u8bbf\u95ee\u6743\u9650'}]}]
```

我使用 ORM 自带的方法 `read()` 和模块新增的方法 `jsonify()` 读取了相同的字段，经过简单的对比我们可以清楚地看到两者之间的差别：

- 前者会默认将记录的 `id` 获取，即使你并没有在参数中指定
- 前者获取的时间和日期是一个 Python 对象，而后者获取的是 `ISO` 格式的字符串
- 前者获取的 m2o 类型字段的记录是一个元组，而后者获取的是一个字典
- 前者获取的 x2m 类型字段的记录是一个由 id 组成的列表，而后者获取的是一个由对应键值对的字段组成的列表
- 前者不能对字段重命名，而后者可以根据需求重命名字段

根据上面的内容可以总结出 `jsonify()` 方法的一些优点：

- 可以重命名字段
- 时间对象会转换为 `ISO` 格式的字符串
- m2o 和 x2m 字段可以指定要获取的字段并转换成键值对形式

# 适用场景

根据前面的使用结果来看，如果是要读取普通字段，如 `Integer`, `Boolean`, `Float`, `Char`, `Text` 这些，不论是用 `read()` 还是 `jsonify()` 效果都是一样的。

而对于关系型字段（`Many2one`, `One2many`, `Many2many`）和时间字段来说，这两者的区别就很大了。如果你有很多的关系型字段需要处理，或者需要用 JavaScript 处理时间，那么选择 `jsonify()` 会大大提高你的效率，毕竟你不需要再写很多的逻辑去根据关联对象的 ID 实例化，然后处理成自己想要的字典形式。

这个模块很适合在需要开发一些不太复杂的接口的情况下使用。

# 实现浅析

实现方式相对简单，这里不作详细说明，请自行查阅源码以及注释。

这个模块还提供了 `get_json_parser()` 方法，感兴趣的可以自行前往仓库查看说明和源码。

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2019/12/08/Odoo-Module-Recommend-Base-Jsonify/)。

本文永久地址：https://ruterly.com/2019/12/08/Odoo-Module-Recommend-Base-Jsonify/
