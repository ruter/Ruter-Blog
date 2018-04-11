---
title: 接替Odoo导航栏菜单实现自定义
categories: Odoo
keywords: Odoo, Web
toc: false
date: 2017-09-20 17:08:41
description: Odoo自带的菜单很好用，但是我们想要更强的自定义菜单
---

这篇文章将会教大家如何在保留系统导航栏菜单自带功能的基础上，将其接管以实现自定义菜单。

# 背景

在使用 `Odoo` 的一些系统模块时，我们往往只需要使用到其中的一部分功能，一些对我们所开发的模块不重要或者对于客户来说是多余的内容，必要时我们会需要将其隐藏起来。

这次要被动刀子的就是 `website` 模块的菜单导航栏了，在安装了这个模块后，访问首页可以看到导航栏有一个 `首页` 和 `联系我们` 的菜单。在一些情况下我们并不需要系统自带的这些模块生成的菜单，我们可以在顶部紫色的菜单栏点击 `Content —> Edit Menu` 对现有的菜单进行编辑，而这个方式也仅仅是只能修改或删除现有的菜单以及新增菜单入口，并且每当安装一个带有前端导航栏菜单的模块，这里就会新增一个菜单项，我们就要手动去修改或删除，显得就有点繁琐了。此外，如果我们想要这里的导航栏菜单拥有和系统后台的菜单一样可以根据权限来显示或隐藏的话，那要怎么办？

# 实施方案

在正式开始之前，初步拟定的修改方向是利用 `Odoo` 提供的「视图继承」机制，对 `website` 模块的模板进行修改，将其生成菜单相关的节点替换( `replace` )掉。当我们需要自定义导航栏菜单的时候，只需要在相应的节点里添加一个菜单项即可。这样做会有一个小问题，但是不太影响整体的使用，就是我们这样添加的菜单项，会失去动态添加 `active` 类这一特性（其实我们可以做一些事情去实现这个功能），而这一特性是系统原本的菜单渲染已经做好了的。

方向确定，现在开始搞事情。先找到模板 `website.layout` ，然后分析一下节点的构成，以便我们进行替换。其中和导航栏菜单相关的节点在 `<header/>` 标签里，很容易就可以发现很多 `nav` 相关的类，以下这一段就是它的主体了：

```xml
<ul class="nav navbar-nav navbar-right" id="top_menu">
  <t t-foreach="website.menu_id.child_id" t-as="submenu">
    <t t-call="website.submenu"/>
  </t>
  <li class="divider" t-ignore="true" t-if="website.user_id != user_id"/>
  <li class="dropdown" t-ignore="true" t-if="website.user_id != user_id">
    <a href="#" class="dropdown-toggle" data-toggle="dropdown">
      <b>
        <span t-esc="(len(user_id.name)&gt;25) and (user_id.name[:23]+'...') or user_id.name"/>
        <span class="caret"/>
      </b>
    </a>
    <ul class="dropdown-menu js_usermenu" role="menu">
      <li id="o_logout"><a t-attf-href="/web/session/logout?redirect=/" role="menuitem">Logout</a></li>
    </ul>
  </li>
</ul>
```

再回到首页看一下导航栏，可以发现在用户名和导航栏菜单项之间有一条分割线，通过查看该元素，我们可以发现就是以上代码中的 `<li class="divider"/>` ，在它前面的都是导航栏的菜单项，即我们要移除的目标对象，而这个分割线元素本身可以作为我们插入菜单的基准节点，后面马上会用到。

到这里我们已经确定了相关节点，马上就可以着手实现了，但是先不急，再花点时间研究一下分割线前面的菜单项相关内容：

```xml
<t t-foreach="website.menu_id.child_id" t-as="submenu">
  <t t-call="website.submenu"/>
</t>
```

这段内容相当简单，没有什么特殊之处，就是遍历了菜单模型的数据，然后调用 `website.submenu` 模板对每一个菜单项进行渲染，而关键就在这个渲染方式上。前面所说的动态添加 `active` 类，就是在这个模板中渲染完成的，既然已经有现成的模板可以调用，为什么不用呢，官方的实现总是比我们自己写的要美丽得太多了，所以不要犹豫就用它了。

至于要怎么用，其实很简单，再看回上面那段，模板 `website.submenu` 只负责渲染单个菜单项，再上一层就是遍历出所有的菜单项给这个模板渲染，所以我们就可以在这里做文章了，如果我想控制只显示指定模块的指定菜单，只需要找出指定的数据，扔给 `website.submenu` 渲染就好了:)

# 实例

话不投机半句多，没有代码说个蛇。

先创建一个模板，继承 `website.layout` ，然后将生成导航栏菜单的那一段移除掉：

```xml
<template id="home_submenu_remove" name="Remove submenu layout : default menu entries" inherit_id="website.layout">
  <xpath expr="//ul[@id='top_menu']/t" position="replace">
  </xpath>
</template>
```

好了，把默认的给干掉了，现在可以为所欲为了，先给模块增加两条菜单的记录（ `/data/data.xml` ）：

```xml
<!-- 博客入口菜单 -->
<record id="menu_my_blog" model="website.menu">
  <field name="name">Blog</field>
  <field name="url">/blog</field>
  <field name="parent_id" ref="website.main_menu"/>
  <field name="sequence" type="int">1</field>
</record>
<!-- 博客后台入口菜单 -->
<record id="menu_my_blog_admin" model="website.menu">
  <field name="name">Blog Admin</field>
  <field name="url">/blog/admin</field>
  <field name="parent_id" ref="website.main_menu"/>
  <field name="sequence" type="int">2</field>
</record>
```

我们这里作为示例，添加了两条记录，接下来是给页面导航栏添加菜单入口：

```xml
<template id="my_blog_submenu" name="Submenu layout : blog menu entries" inherit_id="website.layout">
  <xpath expr="//ul[@id='top_menu']/li[@class='divider']" position="before">
    <t t-if="request.session.uid and not request.env.user.share">
      <t t-foreach="website.menu_id.child_id.filtered(lambda r: r.url in ['/blog', '/blog/admin'])" t-as="submenu">
        <t t-call="website.submenu" t-if="submenu.url == '/blog' or request.env.user.has_group('my_blog.group_blog_admin')"/>
      </t>
    </t>
  </xpath>
</template>
```

粗略描述一下这段内容所做的事情，就是在分割线前面添加渲染我们前面新增的两个菜单项，其中可以看到有很多的条件判断语句 `t-if` ，这就是一开始所说的根据不同的情况对菜单进行显示和隐藏，不过这里实际所做的是根据这些指定的条件来决定是否渲染指定的菜单项。

完整地说明一下渲染规则：

1. 用户已经登录系统，并且登录的用户不是外部用户( `user.share == False` )
2. 通过匹配 `url` 的值[注1]查找我们新增的两条菜单记录( `url in ['/blog', '/blog/admin']` )
3. 如果用户在管理员用户组 `group_blog_admin` 中则渲染后台菜单项和博客入口菜单，否则只渲染博客入口菜单

# 最后

简而言之，要对所显示的菜单做各种骚操作，只需要在找出你想要渲染的菜单记录之后，在调用模板 `website.submenu` 进行渲染之前，完成你想要做的各种操作即可。

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://blog.ruterly.com/)」原创。转载务必注明[出处](http://blog.ruterly.com/2017/09/20/Replace-Odoo-navbar-menu/)。 

本文永久地址：http://blog.ruterly.com/2017/09/20/Replace-Odoo-navbar-menu/