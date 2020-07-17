---
layout: post
title: 在 Odoo 13 列表视图添加按钮
categories: [Odoo, Tutorial]
description: 在 Odoo 13 列表视图上添加按钮
keywords: Odoo, Tutorial
---

最近不时会看到群里有小伙伴问怎么在 Tree 视图上添加自定义按钮执行指定的方法，也有不少热心群友给出了解决方案，那我也来写一个在 Odoo 13 列表视图添加按钮的方法吧。

在 Odoo 13 中，视图标签如 `<tree />` 和 `<form />` 都可以添加一个 `js_class` 的特殊属性，然后把自己新增的视图和这个属性值关联后，即可在对应的视图上应用你创建的视图。

光靠文字描述显得有些苍白无力，下面就用一个最简单的例子来演示吧。

首先创建好一个空模块，然后创建一个列表视图，为了免去权限配置等步骤，我们就直接继承用户列表的视图 `base.view_users_tree` 来作为演示吧:

```xml
<record id="customize_user_tree_view" model="ir.ui.view">
    <field name="name">res.users.customize.tree.view</field>
    <field name="model">res.users</field>
    <field name="inherit_id" ref="base.view_users_tree"/>
    <field name="arch" type="xml">
        <xpath expr="//tree" position="attributes">
            <attribute name="js_class">customize_user_tree</attribute>
        </xpath>
    </field>
</record>
```

接下来要做的就是创建一个 JS 文件了，我们需要添加一个 `Controller` 和 `View` 以实现自定义视图，请注意查看注释部分:

```javascript
odoo.define('user.customize.tree', function (require) {
	"use strict";
    var core = require('web.core');
    var ListController = require('web.ListController');
    var ListView = require('web.ListView');
    var viewRegistry = require('web.view_registry');

	// 扩展 ListController
    var UsersListController = ListController.extend({
		// 重载列表按钮渲染的方法
        renderButtons: function ($node) {
            this._super.apply(this, arguments);
            // 新增两个按钮
            var $preButton = $('<button type="button" class="btn btn-primary">pre BTN</button>');
            var $aftButton = $('<button type="button" class="btn btn-primary">aft BTN</button>');
            // 绑定按钮的点击事件
            $preButton.on('click', function(e){alert('pre BTN clicked.')});
            $aftButton.on('click', function(e){alert('aft BTN clicked.')});
            // 在列表按钮的前面添加按钮
            $preButton.prependTo($node.find('.o_list_buttons'));
            // 在列表按钮的后面添加按钮
            $aftButton.appendTo($node.find('.o_list_buttons'));
        },
    });

	// 扩展 ListView
    var UsersListView = ListView.extend({
        config: _.extend({}, ListView.prototype.config, {
			// 配置 Controller 为上面定义的 UsersListController
            Controller: UsersListController,
        }),
    });

	// 注册视图，视图名为之前所添加的 js_class 的值
    viewRegistry.add('customize_user_tree', UsersListView);
});
```

添加后请不要忘记引入资源文件，具体操作这里略过。下图是安装或更新好模块后的效果:

![添加自定义按钮后的用户列表视图](/images/Odoo/add-btn-tree-view.png)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2020/07/17/Odoo-Add-TreeView-Buttons/
