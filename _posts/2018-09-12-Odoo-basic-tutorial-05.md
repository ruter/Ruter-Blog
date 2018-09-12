---
layout: post
title: 「Odoo 基础教程系列」第五篇——从 Todo 应用开始（4）
categories: [Odoo, Tutorial]
description: Odoo 基础教程系列之从 Todo 应用开始
keywords: Odoo, Tutorial, 基础教程
---

![Powered by @atgranberry](/images/Odoo/hero-code.jpeg)

在前面教程中，我们使用了两种类型的视图——TreeView 和 FormView. 今天我们将学习使用另一种类型的视图——SearchView, 搜索视图。

# 搜索字段

通过之前的实践，我们应该已经知道 Odoo 会为我们的模型定义默认的 TreeView 和 FormView 了吧，那对于我们今天要说的主角 SearchView 其实也是一样的，我们打开任意一个列表视图，都可以在列表的右上方看到有搜索框，除了能在搜索框里输入文字进行检索外，点一下搜索框右侧的放大镜还能展开更多的搜索功能，包括过滤器（`Filters`），分组（`Group By`）以及收藏（`Favorites`），其中最常用的就是过滤器了，我们稍候将会讲到。

默认的搜索视图只会对模型的 `name` 字段（或 `_rec_name` 指定的字段）进行检索，例如在我们的待办事项列表中搜索，会看到搜索框下方自动出现了搜索项，这里的「描述」也就是我们的待办事项模型 `todo.task` 的 `name` 字段了：

![默认搜索](/images/Odoo/search-keyword.png)

那我们如果想让「分类」也能用于待办事项的搜索，该怎么做呢？我们接下来就打开视图文件，先定义好搜索视图的基本结构：

```xml
    <!-- views.xml -->
    <record id="todo_task_view_filter" model="ir.ui.view">
        <field name="name">todo.task.view_filter</field>
        <field name="model">todo.task</field>
        <field name="arch" type="xml">
            <search string="Todo">
                ...
            </search>
        </field>
    </record>
```

搜索视图和列表视图以及表单视图一样，都有相似的结构，唯一不同的就是在 `arch` 内的部分。接下来我们要在 `<search />` 中加上我们想要搜索的字段：

```xml
    <search string="Todo">
        <field name="name"/>
        <field name="category_id"/>
    </search>
```

超简单有没有！只需要把想要被用于搜索的字段放到 `<search />` 内就可以了，这里需要注意的是，我们自己定义了搜索视图，相当于覆盖掉了默认的视图，所以我们还需要将字段 `name` 也加入到里面，不然我们就没办法根据待办事项的描述进行搜索啦。

升级一下模块，在输入框随便输入搜索内容看看发生了什么变化：

![自定义搜索字段](/images/Odoo/search-field.png)

还记得我们之前创建过「工作」这个分类吗，在下面列出的「分类」的搜索结果中，点击小三角符号，就会打开匹配到的相关结果。

# 过滤器

我们再来看看 Odoo 的搜索视图自带的过滤器，点开之后我们只看到一个添加自定义过滤器的选项，再点一下看看，出现了一些选项：

![过滤器](/images/Odoo/filter.png)

我们试试随意添加一些规则，然后点击应用，看看会有什么发生（我分别选择了「紧急程度」「is」「普通」）：

![添加自定义过滤器](/images/Odoo/custom-filter.png)

我们可以看到在搜索框中，应用了这个过滤器，并且这个过滤器被添加到了过滤器的选项中，我们还可以再添加几个过滤器试试看。

通过以上的操作，我们不难看出，这里的过滤器其实就是一个个过滤规则，而且我们可以同时应用多个不同的规则。如果我们有大量的数据，而我们想要快速筛选出特定的数据的话，就可以使用过滤器达到这个目的。

如果我们切换到其他菜单或者刷新页面，就会发现之前应用的过滤器，又消失了，如果我们在某些数据上需要高频次进行过滤操作的话，每次都要手动添加过滤规则的话，就显得十分不方便了，所以我们来看看怎么在搜索视图中定义过滤器吧：

```xml
    <search string="Todo">
        <field name="name"/>
        <field name="category_id"/>
        <separator/>
        <filter string="未完成" name="undone" domain="[('is_done', '=', False)]"/>
        <filter string="已完成" name="done" domain="[('is_done', '=', True)]"/>
        <separator/>
        <filter string="待办" name="todo" domain="[('priority', '=', 'todo')]"/>
        <filter string="普通" name="normal" domain="[('priority', '=', 'normal')]"/>
        <filter string="紧急" name="urgency" domain="[('priority', '=', 'urgency')]"/>
    </search>
```

我们在 `<search />` 里添加了五个过滤器 `<filter />`，其中 `string` 是显示的名称，`name` 是数据库中存储的名字，我们可以通过它来获取到对应的过滤器，最关键的一个属性就是 `domain` 了。在 Odoo 中的各种过滤都是通过`domain`来实现的，它是一个由逻辑前缀和表达式元组组成的列表，关于 `domain` 的更多细节，这里先不展开来说，后面应该会专门写一篇来说说它。

我们以「未完成」这个过滤器的 domain 为例，先不管条件是什么，先写下一个空的列表 `[]`，这里要记住了，只要是用到 domain 的地方，都先写下一对中括号 `[]`，因为刚接触常常会因为少写了括号而导致程序出现错误。

然后我们先不考虑逻辑前缀的事情，我们这里也暂时没有用到，所以不要让这个名词困扰到了。在刚刚写下的空列表中，添加一对圆括号 `()`，也就是一个元组，最基本的结构已经有了，现在我们来写表达式。我们用来标记一个待办事项是否完成，用的是字段 `is_done`，如果它的值为 `False` 则表示这个待办事项未完成，反之则表示已完成，根据这个规则，我们可以在元组中添加我们的表达式了，先写下字段名 `'is_done'`（注意这里是字符串），然后写下表达式的操作符 `'='`（这里也是一个字符串），最后写下未完成对应的值 `False`（这里是一个布尔值，非字符串）。

怎么样，是不是很简单呢，可能有同学要问了，为什么这里是用等于号 `=` 而不是 `==` 呢？这里就需要大家注意并且习惯了，不同于我们平时写代码判断相等时会用 `==` 双等号，在 domain 的表达式中判断相等是使用的 `=` 单个等号，千万记住啦！

其他的过滤器我们也能使用同样的方式写出来，大家再好好理解一下吧。我们再来看到上面出现的 `<separator />` 这个标签，它有两个作用，一个是将搜索字段和过滤器之间进行分组隔离，另一个作用是以逻辑与（Python 中的 `and` ）对搜索字段和过滤器进行连接，说再多不如实践来得清楚，我们先升级一下代码，然后刷新页面看看实际的效果吧：

![过滤器](/images/Odoo/filters.png)

心细的你一定发现了，是否完成的过滤器和紧急程度的过滤器被一条分割线分隔开了，从上面的代码中我们可以看到这根分割线正是 `<separator />` 这个标签在界面上显示的效果。我们如果同时勾选同一组内的过滤器，可以发现搜索框内的过滤器被 `or` 连接起来了，这表明了同一组内的过滤器会使用逻辑或（Python 中的 `or` ）进行连接，而如果同时选中两个不同组的过滤器，搜索框内的过滤器标签则是分开的，也就是我们前面说的，被用逻辑与连接起来了。

试试看在搜索框中输入内容并搜索，然后加上过滤器组合看看效果吧，有没有发现什么呢？再看看搜索视图的代码，在 `<field />` 和 `<filter />` 之间还有一个 `<separator />` 呢。

# 分组

使用字段搜索和过滤器，已经可以满足大部分数据的搜索需求了，有时候我们除了需要按规则过滤出所需的数据外，还会需要对数据进行分组查看，这种时候就可以用上分组（`group by`）功能了。

和过滤器一样，分组功能我们同样可以自己添加自定义的分组，但是要比过滤器简单得多，只需要选择一个字段然后应用即可。

我们来看看在搜索视图中如何定义分组项：

```xml
    <search string="Todo">
        ...
        <filter string="紧急" name="urgency" domain="[('priority', '=', 'urgency')]"/>
        <group expand="0" string="分组">
            <filter string="分类" domain="[]" context="{'group_by':'category_id'}"/>
            <filter string="紧急程度" domain="[]" context="{'group_by':'priority'}"/>
        </group>
    </search>
```

我们添加了一个 `<group />` 标签，并且在里面添加了两个 `<filter />`，眼尖的你一定发现了，这里和过滤器长得一模一样，只是多了一个 `context` 属性，是的没有错，分组的关键也就在这个 `context` 属性里，里面是一个字典，对应的键值对表示按指定字段进行分组。

另外我们再看到这里留空的 `domain`，大家可以尝试在里面添加一些表达式，看看会有什么效果，这里我就不细说了，快升级模块然后去页面中刷新看看效果吧：

![分组](/images/Odoo/groupby.png)

我们可以看到数据按照我们所选的分组项进行了分组，并且默认是折叠起来的，点击展开之后我们可以看到对应分组下的数据，默认折叠这个行为是由 `<group />` 标签的 `expand` 属性决定的，如果设置为 1 的话就会是默认展开。

大家可以试试看同时应用多个分组，或者将分组和过滤器组合使用，仔细观察和思考产生的结果。

# 瞎说几句

以上就是这次教程的内容啦，SearchView 中还有一个收藏功能我们没有讲到，这个功能我们目前或者很长的一段时间内都不会用到，所以也不打算在之后的教程中讲解，如果感兴趣的话可以自己尝试着去使用，安装一些官方的模块，又或者看看源代码，去学习相关的知识。

还是老规矩，教程中的代码会更新在 `GitHub` 仓库「[Odoo-Tutorial-Demo](https://github.com/ruter/Odoo-Tutorial-Demo)」中。

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/09/12/Odoo-basic-tutorial-05/)。

本文永久地址：https://ruterly.com/2018/09/12/Odoo-basic-tutorial-05/