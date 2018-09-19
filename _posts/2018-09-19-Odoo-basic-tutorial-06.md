---
layout: post
title: 「Odoo 基础教程系列」第六篇——从 Todo 应用开始（5）
categories: [Odoo, Tutorial]
description: Odoo 基础教程系列之从 Todo 应用开始
keywords: Odoo, Tutorial, 基础教程
---

大家好鸭，我又来更新啦！还记得我们在[第二篇教程](https://ruterly.com/2018/06/26/Odoo-basic-tutorial-02/#%E5%88%9B%E5%BB%BA%E8%8F%9C%E5%8D%95)中提到过的动作（actions）吗，今天我们就来专门讲讲在 Odoo 中的 `action`，学习不同类型的动作对应的应用场景，并且在我们的 Todo 应用中使用上其中一些类型的动作。

# 窗口动作

窗口动作在 Odoo 中是最常见也是最常用的动作类型，在第二篇教程中我们创建菜单的时候就已经用过窗口动作了，我们打开 `menus.xml` 来看看一个窗口动作的组成：

```xml
    <!-- menus.xml -->
    <record id="action_todo_task" model="ir.actions.act_window">
        <field name="name">待办事项</field>
        <field name="res_model">todo.task</field>
        <field name="view_type">form</field>
        <field name="view_mode">tree,form</field>
        <field name="target">current</field>
    </record>
```

这里的 `model` 之前我们也说过，对应的是这个动作的类型（每个类型的动作都对应一个模型），我们重点来看看下面列出的字段以及没有列出来的字段，分别说明一下它们各自的作用。

- `res_model`：要打开的视图（窗口）关联的数据模型
- `view_type`：视图类型，默认值为 `form`，一般情况下我们取默认值就可以了
- `view_mode`：允许打开的视图类型，以逗号分隔，默认值为 `tree,form`
- `target`：打开的窗口类型，常用的有 `current`（当前窗口打开）和 `new` （弹窗打开）这两种，默认为 `current`

除了上述列出的一些字段外，还有一些非必填的字段在某些时候我们是会用上的，这里也分列出来：

- `view_ids`：关联的视图对象 id，需注意区分和 `view_id` 的区别
- `view_id`：关联的视图的 id, 例如在不同时候需要打开同一个数据模型不同的表单视图，就可以通过这个字段指定要打开的视图的 id
- `res_id`：仅在视图类型为 `form` 时有效，表示打开该 id 对应的记录的表单视图，如未指定则打开新建页面
- `context`：传递到上下文中的数据，一个字典
- `domain`：过滤规则，对视图中的记录进行过滤
- `limit`：列表中每页显示的记录数量，默认为 80
- `search_view_id`：指定搜索视图，不指定则按默认规则加载
- `multi`：如果设置为 `True` 且动作绑定了模型（`src_model`）的话，该动作按钮会只出现在所绑定模型列表视图的「动作」下拉列表中（在搜索视图左侧）
- `views`：由 `(view_id, view_type)` 这样的元组对组成的列表，`view_id` 为指定视图的 id 或是 `False`（按默认值取出对应视图），`view_type` 表示视图类型

其中 `views` 是一个计算字段，该字段会根据 `view_ids`, `view_mode` 和 `view_id` 自动计算出来，大致分为三个步骤：

1. 从 `view_ids` 中顺序取出 id 和视图类型，组成 `(view_id, view_type)` 元组对加入到 `views` 列表中
2. 若定义了 `view_id` 且对应的视图类型不在 `view_ids` 中，则加入到 `views` 中
3. 最后对 `view_mode` 中指定的视图类型，但是不存在于 `view_ids` 和 `view_id` 中，则将其 `view_id` 置为 `False` 组成元组对后加入到 `views` 中

这里需要强调一点的是，如果我们在某些时候需要直接通过 Python 代码或 JS 代码打开一个窗口动作，此时我们将需要自己添加 `views` 字段，否则将会抛出错误，具体的实例我们将在之后使用到了再进行讲解分析。

# 服务器动作

Server Action 可以在服务端执行复杂的逻辑代码，是非常强大的一种动作类型。我们先给代办事项添加一个 Server Action 用于快速完成任务的动作，这样我们就不用每次都要点进去一个任务然后进入编辑状态再勾选保存这么多步骤了。

我们来看看定义一个最简单的 Server Action 应该包含哪些内容：

```xml
    <!-- views.xml -->
    <record id="action_mark_todo_task_done" model="ir.actions.server">
        <field name="name">标记完成</field>
        <field name="model_id" ref="model_todo_task"/>
        <field name="binding_model_id" ref="model_todo_task"/>
        <field name="state">code</field>
        <field name="code">records.write({'is_done': True})</field>
    </record>
```

和其他所有在数据文件（XML）中定义的数据一样，首先是一个包含属性 `id` 和 `model` 的 `<record />` 标签，我们要定义的是 Server Action, 所以需要将 `model` 设置为 `ir.actions.server`，然后是对应 Server Action 模型下的一些字段：

- `model_id`：当前的动作是在哪个模型上运行的
- `binding_model_id`：绑定的模型，当前动作将会出现在绑定的模型的视图中
- `state`：服务器动作的类型，总共有 4 种可选的类型，分别是 `code`（执行 Python 代码），`object_create`（创建一条新记录），`object_write`（更新记录），`multi`（执行多个动作）
- `code`：对应 `state` 的类型 `code`，为当前动作运行时所要执行的 Python 代码

大家应该都留意到了在上面的定义中出现了一个属性 `ref`，如果我没有理解错，它应该是 `reference` 的缩写，它的值是一个外部 ID（`external id`），我们上面定义的这个动作的外部 ID 就是定义时添加的 `id` 属性的值 `action_mark_todo_task_done`，它指向一条具体的记录（就像 `Many2one` 字段一样）。

我们定义的所有模型都会在 `ir.model.data` 对应的表中存在相应的记录，这些模型的外部 ID 形如 `model_model_name`，其中 `model_name` 是将模型名 `[model.name](http://model.name)` 的 `.` 替换成 `_`，例如我们的 `todo.task` 模型的外部 ID 就是前缀 `model_` 加上 `todo_task` 组成的 `model_todo_task` 了。

接下来我们再看到字段 `code` 里面的内容，在这里面我们有一些变量是可以直接使用的：

- `env`：Odoo 的运行环境
- `model`：动作触发时对应的 Odoo 模型实例
- `record`：动作触发时对应的单个记录（如在表单视图中运行对应当前表单所指向的记录），可能是空的
- `records`：动作触发时对应的记录集（如在列表视图中勾选多条记录触发，记录集指向这些选中的记录），可能是空的
- Python 库：`time`, `datetime`, `dateutil`, `timezone` 时间相关的 Python 库
- `log`：用来记录日志信息
- `Warning`：通过 `raise Warning('xxxxx')` 抛出警告信息

除了上述可以直接使用的变量外，还有一个 `action` 变量，当我们想让当前动作执行完毕之后，返回一个新的动作继续执行，就可以将返回的动作的定义（一个字典）赋值给变量 `action`，客户端将会自动执行该动作。

OK, 前面说了这么多，我们更新一下代码，然后刷新浏览器看看效果吧：

![Server Action](/images/Odoo/server-action.png)

在列表视图中，当我们勾选了任意的记录后，将会出现一个「动作」菜单，打开之后就可以看到我们刚刚定义的「标记完成」这个 Server Action 了，试试看执行会发生什么，注意观察「已完成?」这一列的变化。然后再创建一条新的记录，在表单页面中，同样有「动作」菜单，我们在这里也可以执行「标记完成」的动作，怎么样，是不是比原本的编辑再勾选再保存要方便得多了，而且我们还能在列表里勾选多条记录进行批量操作，不能更方便了！

受限于篇幅，我们这里只涉及了 `state` 为 `code` 类型的服务器动作，相对其他三种类型，已经足以应付绝大多数场景的需求了，这里就不展开细说，感兴趣的同学可以看看官方模块的相关实现，要学会看源码哈！

# URL 动作

URL Action 用来打开一个网页链接，十分简单的一种动作类型：

```xml
    <record id="action_open_google" model="ir.actions.act_url">
    		<field name="name">打开谷歌</field>
    		<field name="target">new</field>
    		<field name="url">https://google.com</field>
    </record>
```

上面这个动作触发后将会新开浏览器窗口（或新标签）打开谷歌首页，主要的两个字段如下：

- `target`：有两个可选值，分别是新窗口（`new`）打开链接，相当于 `<a target='_blank' />`，以及当前窗口（`self`）打开，相当于 `<a target='_self' />`
- `url`：要打开的目标页面的链接，可以是外部页面也可以是同域下的内部页面

# 客户端动作

触发一个完全由客户端（浏览器）执行的动作，例如某些模块的「仪表板」就属于 Client Action，它的基本组成如下：

```xml
    <record id="backend_dashboard" model="ir.actions.client">
        <field name="name">Dashboard</field>
        <field name="tag">backend_dashboard</field>
    </record>
```

上面这个 Client Action 是在 Odoo 自带模块 `website` 中定义，用来打开仪表板，来看看客户端动作有哪些字段是可用的：

- `tag`：客户端动作的标识，需要是在 `action_registry` 中注册了的动作
- `target`：同窗口动作
- `context`：同窗口动作
- `params`：传递给客户端动作的参数，一个字典

客户端动作的本体实际上是 `tag` 所指向的动作，这个动作是用 JS 编写的一些逻辑（在 Odoo 的 JS 框架下），在之后会有专门的教程教大家相关的内容，这里请先略过。

# 报表动作

这类型的动作用于触发报表打印，例如打印发票等。这里不对该类型作介绍，感兴趣的同学同样可以去看看 Odoo 自带的一些模块如 `account` 等。

# 瞎说几句

在实际开发中，最常接触和使用的动作基本上就窗口动作（`ir.actions.act_window`）和服务器动作（`ir.actions.server`）这两种了，其中 Server Action 最复杂也最强大，可以用它来实现很多的功能。

上面没有展开细说的内容，希望大家可以多多翻看官方模块的实现，学会阅读源码才是最好的学习方式。

教程中的代码会更新在 GitHub 仓库「[Odoo-Tutorial-Demo](https://github.com/ruter/Odoo-Tutorial-Demo)」中，如果遇到什么问题，欢迎提出，我会及时解答 ;-)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/09/19/Odoo-basic-tutorial-06/)。

本文永久地址：https://ruterly.com/2018/09/19/Odoo-basic-tutorial-06/