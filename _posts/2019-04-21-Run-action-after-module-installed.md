---
layout: post
title: 在 Odoo 模块安装后执行指定动作
categories: [Odoo, Tutorial]
description: 在 Odoo 模块安装完毕后执行指定的动作
keywords: Odoo, Tutorial
---

在一个模块被安装之后，我们可能会希望它可以执行一些动作，例如打开某个菜单页面，打开某个网址，或者执行一些数据的初始化和处理等，我们可以借助 Odoo 的 `ir.actions.todo` 类型的动作来实现这个需求。

如果安装过 `website` 模块的话，应该会注意到在安装完毕之后页面跳转到了主题选择页面了，选择完主题后（主题也是一个模块，对应的主题模块会被安装）会跳转到网站首页，下面就是主题模块安装后跳转到首页的动作定义：

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_website" model="ir.actions.act_url">
        <field name="name">Website</field>
        <field name="url">/</field>
        <field name="target">self</field>
    </record>
    <record id="base.open_menu" model="ir.actions.todo">
        <field name="action_id" ref="action_website"/>
        <field name="state">open</field>
    </record>
</odoo>
```

首先定义了一个跳转到首页的 `act_url` 动作，然后定义了一个 `todo` 类型的动作，并且在该动作的定义中指定 `action_id`（要执行的动作）为前面定义的动作。

在 `todo` 类型动作中所指定的动作是不限类型的，可以是窗口动作（`act_window`），服务器动作(`server`)，也可以是客户端动作（`client`）以及打开 URL 的动作（`act_url`）。

其中 `state` 为该动作的状态，当一个动作被执行后，会被置为 `done`，之后便不会被触发。

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2019/04/21/Run-action-after-module-installed/
