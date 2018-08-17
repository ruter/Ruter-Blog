---
layout: post
title: 「Odoo 基础教程系列」第三篇——从 Todo 应用开始（2）
categories: [Odoo, Tutorial]
description: Odoo 基础教程系列之从 Todo 应用开始
keywords: Odoo, Tutorial, 基础教程
---

![Powered by @rawpixel](/images/Odoo/hero0.jpeg)

在这篇教程里我们将会了解到 Odoo 模型里的一些其他类型的字段和特殊机制，而我依然会继续带领大家一起完善我们的 Todo 应用，不断地往里面添加一些新的功能特性，让它看起来更丰满也更实用一些🤓

# 选择字段

在上一篇教程中，我们已经创建好了待办事项的模型，但是只是添加了「描述」和「已完成？」两个字段，这肯定是不能满足我们的需求的。现在我们来给待办事项增加一个「紧急程度」的字段，用来表示当前任务的优先级。

    # models.py
    class TodoTask(models.Model):
        _name = 'todo.task'
        _description = '待办事项'
    
        name = fields.Char('描述', required=True)
        is_done = fields.Boolean('已完成？')
        priority = fields.Selection([
            ('todo', '待办'),
            ('normal', '普通'),
            ('urgency', '紧急')
        ], default='todo', string='紧急程度')

我们添加了一个 `Selection` 类型的字段 `priority`，并且指定了三个可供选择的程度类型，一般情况下，**如果一个字段只有固定的几种可选值，通常都会选择使用 `Selection` 字段**，它接受一个元组列表作为参数，其中元组的组成为 `(value, string)`，左边的是数据库中存储的值，右边的是一个用于界面显示的描述。

此处我们还给这个字段添加了默认值 `todo`，表示当一个待办事项被创建后，如果没有指定紧急程度，将默认是待办状态。**我们可以为任意类型的字段添加默认值**。

在上一篇教程中我们提到过，在对模型进行改动之后，需要对模块进行升级才能看到变更后的样子，除了从应用列表中找到模块进行升级外，我们还可以在命令行中给 Odoo 的启动命令加上参数 `-u todo` 指定升级 todo 模块。

    ./odoo-bin --addons-path=addons,../mymodules --db-filter=^demo$ -d demo -u todo

![紧急程度](/images/Odoo/form-urgency.png)

升级后创建或打开任意一条待办事项进入到表单页面，就可以看到已经多了「紧急程度」这个字段了，并且默认是选择了「待办」这一状态的。

# 日期字段

我们已经给待办事项加上紧急程度了，可是光有这个还不够，我们还要给它加上截止时间，毕竟 deadline 是第一生产力呀！

    # models.py
    deadline = fields.Datetime(u'截止时间')

我们把截止日期也放到 TreeView 中，方便查看各个任务的 deadline

    <!-- views.xml -->
    <field name="arch" type="xml">
        <tree string="Todo">
            <field name="name"/>
            <field name="deadline"/>
            <field name="is_done"/>
        </tree>
    </field>

![截止日期](/images/Odoo/form-deadline.png)

# 计算字段与视图装饰器

很多时候我们会需要用不同的颜色对待办事项进行标记，例如我们会希望已经过期的任务以红色标记来提醒我们，这个任务过期了。任务是否已经过期，我们要先知道任务的截止时间（上面一小节已经加上了）和当前时间，然后进行比较判断任务的截止时间是否小于当前时间，如果是则表示任务已经过期了，我们需要在视图上用红色将对应的任务标记起来。那将这个需求转化成代码应该怎么做呢？

这个需求跟时间有关，并且时间是流动（一直在变化）的，所以我们应该要有一个方法在用户每次打开待办事项之前，把这个结果计算好，并且反馈给用户，还好 Odoo 的 ORM 已经为我们实现了相关的机制——计算字段（`Computed fields`）

    # models.py
    is_expired = fields.Boolean(u'已过期', compute='_compute_is_expired')
    
    @api.depends('deadline')
    @api.multi
    def _compute_is_expired(self):
        for record in self:
            record.is_expired = record.deadline < fields.Datetime.now()

计算字段其实和其他字段一样，只不过多了一个 `compute` 属性，它的值是计算这个字段值的方法名。我们来看一下对应的方法 `_compute_is_expired` 头顶上的 `@api.depends` 这个装饰器，它接受了一个参数 `deadline`，表示的是 `is_expired` 这个字段的计算会用到 `deadline` 这个字段的值（我们需要用它的值和当前时间进行比较），如果一个计算字段会用到多个其他字段的值，这里就需要以逗号分隔，将用到的值的字段名依次传入装饰器中。

而 `@api.multi` 则表示该方法中的 `self` 是一个记录集（多个实例的集合），如果不理解，可以暂时不深究，到后面自然会知道这里的实际用法。

再来看看实际的计算逻辑部分，只有一个循环以及一条赋值语句，刚刚已经提到过这里的 `self` 表示一个记录集，我们需要对这个记录集里的每一条记录进行计算，判断这个待办事项是否已经过期，这里的 `record` 就是每一条记录的实例对象，我们用这条记录的 `deadline` 的值和当前时间 `fields.Datetime.now()` 进行比较，然后将结果赋值给字段 `is_expired`，就是这么简单。

其中大家可能会有疑问的应该是当前时间的获取，为什么不是用 `datetime.now()` 吧？实际上获取当前时间用的也是这个方法，只不过 Odoo 的 ORM 替我们封装了一层，`fields.Datetime.now()` 是类 `Datetime` 的静态方法：

    # fields.py
    class Datetime(Field):
        type = 'datetime'
        column_type = ('timestamp', 'timestamp')
        column_cast_from = ('date',)
    
        @staticmethod
        def now(*args):
            """ Return the current day and time in the format expected by the ORM.
                This function may be used to compute default values.
            """
            return datetime.now().strftime(DATETIME_FORMAT)

好的，这里先不过多纠结细节问题，现在我们已经可以计算出来每个待办事项是否已经过期了，那要怎么去用这个计算字段呢？我们打开视图文件来加点东西上去：

    # views.xml
    <field name="arch" type="xml">
        <tree string="Todo" decoration-danger="is_expired">
            <field name="name"/>
            <field name="deadline"/>
            <field name="is_done"/>
            <field name="is_expired" invisible="True"/>
        </tree>
    </field>

在视图中我们把 `is_expired` 字段加了进去，并且还加上了属性 `invisible`，这个属性的作用是将当前字段隐藏起来，因为这里我们不希望用户看到这个字段的值，而是将结果反映在颜色上。然后我们再看到 `<tree />` 标签多了一个属性 `decoration-danger`，这个属性可以接受表达式或字段名作为值，当结果为真时，这个属性就会生效，将 TreeView 中满足表达式的行以红色标记，实际的效果如下：

![截止日期](/images/Odoo/tree-deadline.png)

今天这篇教程的内容就先到这里了，下一篇再继续带大家深入更多的内容。这篇教程中的代码同样会更新在我的 `GitHub` 仓库中。

仓库地址：[Odoo-Tutorial-Demo](https://github.com/ruter/Odoo-Tutorial-Demo)

# 写在最后

距离上一次更新，已经过了好几个月了，这段时间除了忙公司的事情，还额外在做一些别的东西，然后最近在开发一个小程序。一直很想抽空出来更新这个系列的教程，一边又有很多事情在忙，拖更了实在是抱歉了！

如果你有任何的疑问，欢迎留言，我将会尽快给出答复~

期待下一篇教程可以继续和你们见面。

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/08/17/Odoo-basic-tutorial-03/)。

本文永久地址：https://ruterly.com/2018/08/17/Odoo-basic-tutorial-03/