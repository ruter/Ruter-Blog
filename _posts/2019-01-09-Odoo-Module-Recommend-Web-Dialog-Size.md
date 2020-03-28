---
layout: post
title: Odoo 模块推荐——Web Dialog Size
categories: Odoo
description: 一个让用户可以放大弹窗到全屏宽，并且让弹窗可以拖动的模块
keywords: odoo, 模块
---

好久不见，我又回来了，带着新的内容板块再次和大家见面啦 😆 这是你没有见过的全新板块（自动播放喳喳辉发音）

在新的内容板块里，我将会为大家推荐一些实用的 Odoo 模块，并且通过简单的实例来说明这些模块的使用方法和适用场景（可能会在某些时候虚构一些业务场景），除此之外我还将对部分模块的实现和源码进行简单的讲解和分析，毕竟会用不是我们的目的，知晓其核心才更有意义。

今天就先推荐一个小模块吧，虽然这个模块很简单，但是却能带来很实在的用户体验上的提升。

# 模块信息

|          |                                                              | 备注                  |
| -------- | ------------------------------------------------------------ | --------------------- |
| 名称     | Web Dialog Size                                              |                       |
| 功能     | 让用户可以放大弹窗到全屏宽，并且让弹窗可以拖动               |                       |
| 商店地址 | [点击前往](https://apps.odoo.com/apps/modules/11.0/web_dialog_size/) | 商店中最高版本为 11.0 |
| 仓库地址 | [前往 GitHub](https://github.com/OCA/web/tree/12.0/web_dialog_size) | 仓库中最高版本为 12.0 |

# 使用体验

本次体验使用的 Odoo 版本是 12.0，不同版本之间可能会有细微差异，请以实际使用情况为准。

模块安装之后不需要任何配置，这个模块对 Odoo 中的弹窗做了一些优化，为了看到效果我们需要找一个弹窗出来。

在打开「开发者模式」后点击菜单「Update Apps List」就有一个弹窗出现：

![Odoo 弹窗默认大小](/images/Odoo/odoo_dialog_01.png)

把注意力放在上图弹窗右上角的高亮处，在安装该模块前，高亮位置是没有这样的扩展图标的。在点击该图标后，弹窗的宽度发生了变化：

![Odoo 弹窗放大后](/images/Odoo/odoo_dialog_02.png)

同时扩展图标也变成了收缩图标，除了缩放图标和弹窗宽度的变化外，这个模块还让原本不能拖动的弹窗变得可以随意拖动了：

![可以随意拖动的弹窗](/images/Odoo/odoo_drag_window.gif)

# 适用场景

上面只是简单的试用，并不能体现出这个模块的优点，那我们打开另一个弹窗再看一下效果吧：

![Odoo 个人选项弹窗](/images/Odoo/odoo_dialog_03.png)

这是个人选项（在右上角下拉菜单里的「Preferences」）的弹窗，就签名（Signature）来说，在纵向高度不是很够的时候，如果横向宽度也不够宽的话，我们在富文本框内输入时，很容易就会换行，然后需要滚动文本去查看上面的内容，而把窗口放大到全屏宽之后，文本框内每行就能多显示一些字符了。

又或者说在弹窗内有列表需要显示，而且字段相对较多，弹窗宽度不够就需要横向滚动列表，稍微有些麻烦，这时候将弹窗放大到全屏宽就能显示更多列的内容了。

其实我觉得最有用的还是窗口拖动这个功能。不妨设想一个场景，你正在编辑一个表单页面上的 x2many 字段，你创建了一些这个字段的记录，在你又一次创建新记录并且在填写了一半的内容之后，想要看一下现在正在创建的这条记录是不是在前面已经创建过了，可是列表前面的几条数据刚好被弹窗遮挡住了，这时为了避免创建重复的数据，你不得不关闭弹窗去查看列表前面的记录，如果并没有创建过，前面填写的这么多内容就都白费了😤

而窗口可以拖动的话，就可以轻松地查看到被挡住的内容，然后你可以继续填写剩下的字段，舒服呀！对于我这种金鱼记忆的人来说，这个功能再好用不过了🤣

# 实现浅析

上面看完了怎么用，现在我们来看看怎么写。因为这是个对前端展现内容做修改的模块，核心部分基本上都在其 JS 代码中，所以我们直奔主题，打开 `static/src/js/` 目录下的 JS 文件，一般情况下这种类型的模块其 JS 命名是和模块名一样的，在这个模块里就是 `web_dialog_size.js` 了。

先来看看 `willStart()` 这个方法：

```javascript
willStart: function () {
    var self = this;
    return this._super.apply(this, arguments).then(function () {
        self.$modal.find('.dialog_button_extend').on('click', self.proxy('_extending'));
        self.$modal.find('.dialog_button_restore').on('click', self.proxy('_restore'));
        return config.done(function(r) {
            if (r.default_maximize) {
                self._extending();
            } else {
                self._restore();
            }
        });
    });
},
```

在执行完 `_super()` 之后，这里只是简单地做了三件事：

1. 为扩展按钮添加点击事件，绑定方法 `_extending()`
2. 为收缩按钮添加点击事件，绑定方法 `_restore()`
3. 查询系统参数，判断弹窗是否默认为放大状态，然后执行相应的方法

其中第 3 点中用到了 RPC 请求去调用模型的方法查询数据：

```javascript
var config = rpc.query({
    model: 'ir.config_parameter',
    method: 'get_web_dialog_size_config',
});
```

打开模块的 `models/ir_config_parameter.py` 可以看到只有一个模型方法：

```python
@api.model
def get_web_dialog_size_config(self):
    get_param = self.sudo().get_param
    return {
        "default_maximize": const_eval(
            get_param("web_dialog_size.default_maximize", "False"))
    }
```

这个方法所做的就是查找系统参数中弹窗默认是否最大化的值，默认值为 `False`。

划重点！当我们编写的前端内容需要从后台获取返回值，例如上面的获取配置信息，就可以像这个模块这样做，最后返回得到的是一个 Promise，可以使用 `.then()` 和 `.done()` 等方法链。

下面再看看 `opened()` 这个方法：

```javascript
opened: function(handler) {
    return this._super.apply(this, arguments).then(function(){
        if (this.$modal) {
            this.$modal.draggable({
                handle: '.modal-header',
                helper: false
            });
        }
    }.bind(this));
},
```

这个方法将会在弹窗打开（`open()`）之后执行，这里只做了一件事，让窗口可以拖动。这里是利用 jQuery UI 的 [Draggable]([http://api.jqueryui.com/draggable/](http://api.jqueryui.com/draggable/)) 实现的。

重点来了，Odoo 的前端用到的一个基础框架就是 [jQuery UI]([http://jqueryui.com](http://jqueryui.com/))，也就是说里面的 Widgets 和各种效果以及工具方法我们都可以在自定义 Widget 或编写扩展的时候用上，在准备实现某些相关的功能前，不妨先看看官方文档，说不定就省了很多的功夫呢🤓

![TNK-Studio 公众号](/images/mp_qrcode.jpg)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2019/01/09/Odoo-Module-Recommend-Web-Dialog-Size/