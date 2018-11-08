---
layout: post
title: 「Odoo 基础教程系列」第七篇——从 Todo 应用开始（6）
categories: [Odoo, Tutorial]
description: Odoo 基础教程系列之从 Todo 应用开始
keywords: Odoo, Tutorial, 基础教程
---

在今年的情人节（2018.02.14）那天，我写了一篇博客说即将要开一个坑，也就是大家在看的这个系列的教程。今天这个系列教程即将迎来它的最后一篇内容了，我们将要来学习 Odoo 中权限相关的内容。

# 用户组

在多用户系统中，并不是所有数据都是可以开放给全部用户的，所以在多用户系统中，权限管理是非常重要的一环。通常我们会根据业务和组织架构对用户进行分组管理，也就是我们常说的用户组或角色组，然后再根据用户组对用户所能接触到的数据内容进行控制。

在 Odoo 的权限管理体系中，同样也有用户组这一概念的存在，和其他框架（如 Django）可以说大同小异。回到我们的 Todo 应用里来，从一开始我们就是用的超级管理员账户 `admin` 登录和操作的，这在任何生产环境和测试环境里都是不允许出现的，但是在前面的教程里，我们都没有提到过账户和权限相关的内容，是为了让教程能一路走下来，所以我们今天就来纠正这个错误。

在我们的 Todo 应用中，假设有「用户」和「管理员」两个用户组，普通用户可以创建、修改和删除自己创建的待办事项，而管理员除了拥有普通用户的操作权限外，还可以创建、修改和删除分类数据。

现在我们就来看看应该如何定义这两个用户组，和定义其他数据一样，我们同样需要在 xml 文件中定义用户组数据，找到模块下的 `security` 目录，然后创建一个 `todo_security.xml` 文件，用于定义我们的用户组数据：

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">

        <record id="module_category_todo" model="ir.module.category">
            <field name="name">待办事项</field>
        </record>

        <record id="group_todo_user" model="res.groups">
            <field name="name">用户</field>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            <field name="category_id" ref="todo.module_category_todo"/>
            <field name="users" eval="[(4, ref('base.user_root'))]"/>
        </record>

        <record id="group_todo_manager" model="res.groups">
            <field name="name">管理员</field>
            <field name="implied_ids"
                    eval="[(4, ref('base.group_user')), (4, ref('todo.group_todo_user'))]"/>
            <field name="category_id" ref="todo.module_category_todo"/>
            <field name="users" eval="[(4, ref('base.user_root'))]"/>
        </record>

    </data>
</odoo>
```

这里总共有 3 条记录，分别是 Todo 模块分类（`ir.module.category`）和「用户」及「管理员」两个用户组（`res.groups`），重点来看看用户组里各个字段所代表的含义：

- `category_id`：该用户组所属的模块分类
- `implied_ids`：在当前用户组下的用户，同时加入该字段所指定的用户组中
- `users`：该字段所指定的用户默认被加入到当前用户组中

在上面的记录中，还有两个引用（`ref`），分别是 `base.group_user` 基础用户组和 `base.user_root` 管理员账户。

别忘了把 `'security/todo_security.xml'` 加入到 `__manifest__.py` 的 `data` 列表中，将它放在 `# 'security/ir.model.access.csv'` 这一行的上面，然后升级更新模块之后打开设置菜单下的「用户」菜单，打开管理员账户 `Administrator` 的表单视图。

这时我们可以看到在「Application Accesses」下出现了我们刚刚创建的「待办事项」模块分类，进入编辑模式后，可以看到有「用户」和「管理员」两个选项

![用户组](/images/Odoo/Untitled-3c15d090-2c08-4dfc-b99e-317bdf7911dd.png)

可以发现超级管理员账户已经默认被分配到了待办事项分类下的「管理员」组中了，这也和我们前面说的 `users` 字段所表现的行为是一致的，再返回列表，打开 `Demo User` 这个账户看看，是不是并没有加入我们创建的用户组中的任何一个呢 :)

# 用户组权限

用户组创建好了之后，事情远远还没有结束，因为我没并没有为这两个用户组分配相关数据的操作权限。那如果我们现在创建一个新的用户，并且加入到任意的用户组中，然后登录访问系统，会发生什么呢？实践出真知，我们来创建一个 `test` 用户并且分配到「用户」组中，然后登录看看会发生什么

![创建测试用户](/images/Odoo/Untitled-773bc602-1f64-437c-8b18-0325daad60db.png)

**注：修改新创建的账户密码，打开 Action 下拉菜单，点击 Change Password 即可修改**

打开隐身窗口或者在另一个浏览器上登录，你会看到页面上什么都没有，为什么会这样？

![空白的页面](/images/Odoo/Untitled-160cde06-f1a8-40e1-8f1a-da580058ad70.png)

不要感到迷惑，这确实是因为我们没有给用户组分配任何数据的操作权限导致的，用户没有对数据的读写权限，自然就无法查看到相关的内容了。在 Odoo 中给用户组分配权限，是以模型为单位的，也就是说只有某个用户组被分配了对应模型的权限之后，该用户组内的用户才能操作对应模型下的数据。用户组权限的定义是以 `.csv` 格式文件存储的，在创建模块时默认已经生成了一个权限记录文件 `security/ir.model.access.csv`，打开它我们可以看到里面有一条记录，这是在创建模块时自动生成的，可以看到里面有 8 个字段，分别代表的含义如下

- `id`：这条权限记录的 id，可以类比为 `xml_id`
- `name`：权限记录的名称
- `model_id:id`：要配置权限的模型的外部 ID (以 `model_` 开头)
- `group_id:id`：应用此条权限配置的用户组的 id，若为空则默认对所有用户组生效
- `perm_read`：读取记录的权限，1 为拥有该权限，0 为不分配该权限
- `perm_write`：编辑更新记录的权限，取值同上
- `perm_create`：创建新记录的权限，取值同上
- `perm_unlink`：删除记录的权限，取值同上

如果不理解也没关系，我们先来给「用户」组添加对模型 `todo.task` 的操作权限，通过实例会更容易理解：

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_todo_task_user,todo.task.user,model_todo_task,group_todo_user,1,1,1,1
```

我们在这里为「用户」组（`group_todo_user`）添加了对 `todo.task` 待办事项模型的读写创建和删除四个操作的权限，然后打开 `__manifest__.py` 并将 `# 'security/ir.model.access.csv'` 这行的注释取消掉，然后升级更新模块，然后刷新刚刚登录的 `test` 账户的界面，是不是可以看到有数据了呢

![测试账户访问界面](/images/Odoo/Untitled-3bd676bc-016c-4e44-9f98-23ab224ef7ae.png)

终于不是空空如也的页面了，可是有没有觉得哪里不对呢？是的，这个账户是新的账户，但是却看到了超级管理员创建的数据，抛开这个问题不管先，这个我们后面会再来解决。现在我们要做的是创建一条新的记录，看看能不能成功创建一条新的记录

![弹窗警告](/images/Odoo/Untitled-6e0a24c6-db4c-42c4-9b97-f81997ed711a.png)

Oops! 在选择分类的时候突然弹出了个错误，这是怎么回事呢，来仔细看看错误提示的内容

> 对不起，你没有权限访问该文档。如果你认为这是一个错误，请联系系统管理员。
（文档模型：todo.category）

看起来问题并不在我们的待办事项（`todo.task`）模型上，而是在分类模型上，我们刚刚分配用户组权限时只分配了待办事项模型的权限，所以当用户视图读取分类模型的数据时，就会提示没有权限访问。解决这个问题同样很简单，只需要加上分类模型的权限即可

```
access_todo_category_user,todo.category.user,model_todo_category,group_todo_user,1,0,0,0
```

将上面的权限定义加到 `ir.model.access.csv` 文件的行末，然后升级更新模块后再次创建，这次终于成功创建起来了。

对权限分配这部分内容的认识有没有比刚开始更开始要清晰一些了呢？那我们这里就留一个小作业吧：

> 为用户组「管理员」添加权限，允许该用户组下的用户对分类模型进行管理，包括创建、删除、更新和读取

如果不知道写得是否正确，可以参考源码中的我的写法。

# 记录集权限

用户组权限是粒度较粗的权限控制方式，它只能针对于某一个模型的全部记录应用相关权限配置，但是某些时候，我们会需要粒度更细的权限控制方式。一个现成的例子就是上一节中，我们用新建的测试账户登录后，可以看到超级管理员所创建的待办事项的记录，这是不合理的，一个普通的用户，应该只能看到和操作他所创建的记录。

值得庆幸的是，Odoo 为我们提供了基于记录集的权限控制方式，我们可以很轻松地根据业务需求编写相应的规则，将权限控制的粒度精细到具体的记录上。为了体现记录集权限的用法，我们将假设在我们的 Todo 应用中，普通用户只能读取和操作由自己创建的待办事项，而管理员则可以查看和删除（但不能修改）所有用户创建的待办事项。

通常和权限相关的内容，我们都会在模块的 `security` 目录下进行定义，记录集规则的定义自然也不例外。我们创建一个 `ir_rule.xml` 的文件，然后添加记录集规则的定义

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="rule_todo_task_user" model="ir.rule">
            <field name="name">待办事项规则 - 用户</field>
            <field name="model_id" ref="model_todo_task"/>
            <field name="domain_force">[('create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('todo.group_todo_user'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
        </record>

        <record id="rule_todo_task_manager" model="ir.rule">
            <field name="name">待办事项规则 - 管理员</field>
            <field name="model_id" ref="model_todo_task"/>
            <field name="domain_force">[('create_uid', '!=', user.id)]</field>
            <field name="groups" eval="[(4, ref('todo.group_todo_manager'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
        </record>
    </data>
</odoo>
```

之后要做的事当然就是把 `'security/ir_rule.xml'` 添加到 `__manifest__.py` 的 `data` 中啦，再来就是升级更新模块，现在刷新浏览器，应该看不到其他用户创建的待办事项记录了，完美达成目的~现在让我们回到管理员账户所在的浏览器，然后创建一个新的账户，并添加到「管理员」组内，接着再新开一个隐身窗口登录这个新创建的管理员账户，我们可以看到和预期的结果一样，管理员可以看到全部用户创建的待办事项记录，随便点击一条记录并编辑，修改任意内容后保存，这时候你会发现弹出了一个错误警告

![错误警告](/images/Odoo/Untitled-5091cc5e-f730-4259-ad7e-4fefc1d2f791.png)

这个错误提示我们当前用户没有权限修改这一条记录，就如我们前面的假设一样，管理员不能修改其他用户创建的记录，剩下的创建和删除，大家可以随意测试。

看完了效果之后，又到了解释说明的时间了，我们先来看到定义的规则里最后四个属性字段，是不是很熟悉呢？没错，这就是我们在定义用户组权限时所看到的表示读写创建和删除这四个操作的属性，在记录集权限的定义里，他们的含义是一样的，只不过属性值从 0 和 1 变成了 True 和 False，然后我们再来看看另外几个字段：

- `model_id`：要应用该规则的模型的外部 ID
- `domain_force`：过滤条件，符合该条件的记录都将按照所定义权限进行检查，其中变量 `user` 表示当前用户的实例对象，可以直接使用
- `groups`：应用该规则的用户组，如果不指定则默认对全部用户应用该规则

# 菜单隐藏

目前为止，一切看起来都很不错，不过还有点美中不足。我们再次打开测试账户所在的浏览器窗口，我们前面留了一个小作业，分类只允许管理员进行管理，那么我们自然不希望普通用户看到多余的菜单内容了，所以某些时候用户虽然对某些数据具有可读的权限，但是未必会想要开放对应的菜单入口给这些用户。而我们要根据用户组来对菜单的可见性进行处理，其实也很简单，只需要在对应的菜单项上添加一个 `groups` 属性即可，里面的值可以是逗号分隔的多个用户组的外部 ID，以「分类」菜单为例

```xml
<!-- views/menus.xml -->
<menuitem action="action_todo_category" id="submenu_todo_category" name="分类"
          parent="menu_todo_submenu" sequence="8" groups="todo.group_todo_manager"/>
```

就这么简单，更新模块之后刷新浏览器看看，「分类」菜单是不是已经被隐藏起来了呢，而管理员账户依然可以看到该菜单。除了菜单外，在视图的字段上也可以通过添加属性 `groups` 达到针对不同的用户组隐藏相关字段的目的。

# 写在最后

虽说这是「Odoo 基础教程系列」的最后一篇内容了，但是这不意味着之后不会再写 Odoo 相关的内容了，感谢大家一直以来的支持，虽然很小众，但是也有不少小伙伴给予了一些正面的鼓励，虽然更新速度很慢很慢，但是这个系列终于还是完成啦！自我感觉还是有很多写得不好的地方，希望以后可以写得更好~

如果大家有什么问题想要得到解答，可以加入群里，和其他小伙伴一起交流或者提问，添加下面这个微信，备注 Odoo 加群，通过后将会拉入群里的 :)

![WeChat QRCode](/images/Odoo/Untitled-70148a95-ffd4-47e9-a5f8-08bfdac86ff6.png)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/11/08/Odoo-basic-tutorial-07/)。

本文永久地址：https://ruterly.com/2018/11/08/Odoo-basic-tutorial-07/