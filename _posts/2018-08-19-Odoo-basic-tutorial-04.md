---
layout: post
title: 「Odoo 基础教程系列」第四篇——从 Todo 应用开始（3）
categories: [Odoo, Tutorial]
description: Odoo 基础教程系列之从 Todo 应用开始
keywords: Odoo, Tutorial, 基础教程
---

![Powered by @rawpixel](/images/Odoo/hero1.jpeg)

在这一篇教程中，将会涉及到外键字段，可以将两个模型关联起来，然后很方便地获取到对应的数据。

# 关联字段

这一小节里，我们会给待办事项加上分类，并且这个分类可以让用户自己创建维护。我们需要先创建一个新的模型 `TodoCategory`，然后将它和待办事项关联起来：

```python
# models.py
class TodoCategory(models.Model):
    _name = 'todo.category'
    _description = '分类'

    name = fields.Char(u'名称')
    task_ids = fields.One2many('todo.task', 'category_id', string=u'待办事项')
    count = fields.Integer(u'任务数量', compute='_compute_task_count')

    @api.depends('task_ids')
    @api.multi
    def _compute_task_count(self):
        pass


class TodoTask(models.Model):
    _name = 'todo.task'
    _description = '待办事项'
    # ...
    category_id = fields.Many2one('todo.category', string=u'分类')
```

在上面的代码中，我们定义了一个 `todo.category` 模型，包含三个字段，然后添加了一个 `category_id` 到待办事项模型中，我们重点来看看 `category_id` 和 `task_ids` 这两个字段。

这两个字段都是关联字段，一个是 `Many2one`，另一个是 `One2many`，还有一种我们暂时不会讲到的 `Many2many` 多对多的关联字段。`Many2one` 有一个必填的属性 `comodel_name` 表示要关联的模型的 `_name`，这个字段的值可能是 0 个或 1 个所关联对象的记录集，我们可以通过这个字段直接获取到所关联的数据对象，而不需要自己去查找对应的实例。另一个关联字段 `One2many` 同样有必填的属性 `comodel_name`，同时还有一个 `inverse_name` 属性，表示的是与当前模型所关联的模型（comodel_name 所指的模型）的 `Many2one` 字段的字段名，在此例中即 `category_id`，通过 `One2many` 字段我们可以直接获取到所有关联了当前记录的数据集。在这个例子中，假设我们有一个分类是「工作」，也就是说我们可以通过工作这个分类的 `task_ids` 这个字段获取到所有待办事项中 `category_id` 所关联的分类是「工作」的所有待办事项。

回到我们的代码中，我们看到分类模型中还有一个计算字段 `count`，我们希望可以看到在一个分类下有多少待办事项，所以需要用到上一篇教程中所讲到的计算字段，这里就当作是复习，一起来完成这个字段的计算逻辑：

```python
# models.py
@api.depends('task_ids')
@api.multi
def _compute_task_count(self):
    for record in self:
        record.count = len(record.task_ids)
```

这里的逻辑也十分简单，我们只需要通过记录集的实例对象 `record` 获取到对应的待办事项，然后用 `len()` 获取 `task_ids` 的长度即可。

好的，模型已经有了，还差了点什么呢？没错，还少了菜单和视图，这里我们直接给出代码，如果还有不理解怎么创建菜单和视图的小伙伴，记得翻看一下之前的教程内容。

```xml
<!-- menus.xml -->
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- 主菜单定义 -->
        <menuitem id="menu_todo" name="Todo"/>
        <menuitem id="menu_todo_submenu" parent="menu_todo" name="待办事项"/>
        <!-- 菜单动作定义 -->
        <record id="action_todo_task" model="ir.actions.act_window">
            <field name="name">待办事项</field>
            <field name="res_model">todo.task</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="target">current</field>
        </record>
    
        <record id="action_todo_category" model="ir.actions.act_window">
            <field name="name">分类</field>
            <field name="res_model">todo.category</field>
            <field name="view_type">form</field>
            <field name="view_mode">tree,form</field>
            <field name="target">current</field>
        </record>
        <!-- 子菜单定义 -->
        <menuitem action="action_todo_category" id="submenu_todo_category" name="分类" parent="menu_todo_submenu" sequence="8"/>
        <menuitem action="action_todo_task" id="submenu_todo_task" name="待办事项" parent="menu_todo_submenu" sequence="10"/>
    </data>
</odoo>
```

菜单这里我们多增加了一层，聪明的你们应该能够一眼看出来哪里不同了，如果还是没找到，实际运行之后再观察一下菜单的结构吧~

```xml
<!-- views.xml -->
<odoo>
    <data>
        <!-- ... -->
        <record id="todo_category_view_tree" model="ir.ui.view">
            <field name="name">todo.category.view_tree</field>
            <field name="model">todo.category</field>
            <field name="type">tree</field>
            <field name="arch" type="xml">
                <tree string="Todo Category">
                    <field name="name"/>
                    <field name="count"/>
                </tree>
            </field>
        </record>
    </data>
</odoo>
```

OK，来更新一下模块，然后打开看看效果吧，再尝试创建几个分类，并且给待办事项关联上分类。

# 视图

一切看起来还不错，但是有没有觉得，创建待办事项的表单视图（Form View），以及分类的表单视图，显得有些凌乱了？虽然并不是不能用，但是，我们还是决定要改造一下！

我们先从分类的视图开始，首先可以看到分类中主要的信息就两个——分类的名称和分类下的任务数量。待办事项我们其实不需要从分类中直接去查看，所以我们大可不必把待办事项的记录显示出来，那我们的目标已经很明确了，隐藏分类表单视图中的待办事项记录，和 Tree View 一样，我们把视图先写好：

```xml
<!-- views.xml -->
<record id="todo_category_view_form" model="ir.ui.view">
    <field name="name">todo.category.view_form</field>
    <field name="model">todo.category</field>
    <field name="type">form</field>
    <field name="arch" type="xml">
        <form string="Todo Category">
            <sheet>
                <group>
                    <group>
                        <field name="name"/>
                    </group>
                    <group>
                        <field name="count" readonly="True"/>
                    </group>
                </group>
            </sheet>
        </form>
    </field>
</record>
```

![分类表单视图](/images/Odoo/cate_form1.png)

怎么样，看起来是不是舒服多了~再仔细一想，创建分类其实也只需要填一个名称，能不能不需要跳转到专门的表单视图里去创建咧？那当然是没问题的啦，我们可以让分类直接就在 Tree View 中创建而不需要专门到 Form View 中去：

```xml
<!-- views.xml -->
<record id="todo_category_view_tree" model="ir.ui.view">
    <field name="name">todo.category.view_tree</field>
    <field name="model">todo.category</field>
    <field name="type">tree</field>
    <field name="arch" type="xml">
        <tree string="Todo Category" editable="bottom">
            <field name="name"/>
            <field name="count"/>
        </tree>
    </field>
</record>
```

![分类列表视图](/images/Odoo/cate_tree1.png)

其实很简单，我们只需要在分类的 Tree View 中的 `<tree />` 标签中加上 `editable="bottom"` 这个属性即可，超简单有没有！分类的视图的处理，我们就先到这里，再来看看待办事项的表单视图应该怎么弄比较好。

待办事项的表单视图，我们只需要简单地排一下版就好啦，没有复杂的处理：

```xml
<!-- views.xml -->
<record id="todo_task_view_form" model="ir.ui.view">
    <field name="name">todo.task.view_form</field>
    <field name="model">todo.task</field>
    <field name="type">form</field>
    <field name="arch" type="xml">
        <form string="Todo">
            <sheet>
                <group>
                    <group>
                        <field name="name"/>
                        <field name="category_id"/>
                        <field name="is_done"/>
                    </group>
                    <group>
                        <field name="priority"/>
                        <field name="deadline"/>
                        <field name="is_expired" readonly="True"/>
                    </group>
                </group>
            </sheet>
        </form>
    </field>
</record>
```

![待办事项表单视图](/images/Odoo/task_form1.png)

视图部分，就先到这里，这还只是很基础的一小部分内容，后面还会有更多关于视图部分的特性，在我们用到的时候将会给大家讲解。

今天这篇教程的内容就先到这里了，教程中的代码会更新在我的 `GitHub` 仓库中。

仓库地址：[Odoo-Tutorial-Demo](https://github.com/ruter/Odoo-Tutorial-Demo)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2018/08/19/Odoo-basic-tutorial-04/