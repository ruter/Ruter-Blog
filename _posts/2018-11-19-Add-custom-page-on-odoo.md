---
layout: post
title: template page
categories: [Odoo, Tutorial]
description: 如何在 Odoo 中创建自定义页面并引用第三方库
keywords: odoo, tutorial
---

前些天群里的小伙伴问了些关于在 Odoo 管理后台自定义页面和 Widget 的问题，那我就来写一篇简短的内容，教大家如何创建自定义页面并引用第三方库。如果大家有看我之前写的基础教程的话，应该知道我们从一开始就是用的是 `Odoo11` 吧，从今天开始，我们的教程全都将基于 `Odoo12` 进行编写，所以在开始前，请确保你现在使用的是 `Odoo12`，或者跟随官方的[安装指南](https://www.odoo.com/documentation/12.0/setup/install.html)安装好环境再开始今天的内容。

# 创建模块

话不多说，先用 `scaffold` 命令创建一个新的模块 `custom_page`，结构如下：

```
myaddons
└── custom_page
	├── __init__.py
	├── __manifest__.py
	├── controllers
	├── demo
	├── models
	├── security
	├── static
	└── views
```

我将会通过使用 [wired-elements](https://wiredjs.com/) 这个 `Web Components` 组件库创建一个简单的自定义页面，从而演示如何引入并使用第三方库。所以我们先准备好相关库，先[下载](https://unpkg.com/wired-elements@latest/dist/wired-elements.bundled.min.js)好这个库，然后将它放到 `static` 目录下：

```
static
└── lib
   └── wired-elements
      └── wired-elements.bundled.min.js
```

以上的目录层级结构是约定俗成的，第三方 JS 库推荐都放在 `static/lib` 下。

# 创建页面

接下来是创建自定义页面，同样的在 `static` 下创建：

```
static
└── src
   └── xml
      └── base.xml
```

自定义的页面、JS 和 CSS 等资源文件，都按约定放置在 `static/src` 下，下面是自定义页面 `base.xml` 文件的内容：

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <templates id="template" xml:space="preserve">
    
        <t t-name="DemoPage">
            <div class="container-fluid py-2 o_web_settings_dashboard">
                <div class="form-row">
                    <div class="col-12 col-lg-6 o_web_settings_dashboard_col">
                        <wired-card elevation="3">
                            <div class="mt8">
                                <wired-input placeholder="Enter name"></wired-input>
                            </div>
                            <div>
                                <wired-radio-group selected="two">
                                    <wired-radio name="one" text="Radio One"></wired-radio>
                                    <wired-radio name="two" text="Radio Two"></wired-radio>
                                    <wired-radio name="three" text="Radio Three"></wired-radio>
                                </wired-radio-group>
                            </div>
                            <div class="mt8 mr8 mb8 text-right">
                                <wired-button>Cancel</wired-button>
                                <wired-button elevation="3" class="demo-submit">Submit</wired-button>
                            </div>
                        </wired-card>
                    </div>
                </div>
            </div>
        </t>
    
    </templates>
```

页面中使用到了 `wired-elements` 的组件，具体使用方式请参考[官方文档](https://github.com/wiredjs/wired-elements/tree/master/packages)。上面的定义中，我们看到有一个属性 `t-name`，这个是页面的名称，可在 JS 中调用，这里暂不展开说明。

# 定义动作

自定义页面实际上是一个 `client action`，也就是客户端动作，通过对 `AbstractAction` 这个抽象类进行扩展，从而指定自定义页面的模板和页面的事件等，下面的代码定义了一个最简单的客户端动作，它包含了一个点击事件：

```javascript
    odoo.define('custom_page.demo', function (require) {
    "use strict";
    
    var AbstractAction = require('web.AbstractAction');
    var core = require('web.core');
    
    var CustomPageDemo = AbstractAction.extend({
        template: 'DemoPage',
        events: { 'click .demo-submit': '_onSubmitClick' },
    
        _onSubmitClick: function (e) {
            e.stopPropagation();
            alert('Submit clicked!');
        }
    });
    
    core.action_registry.add('custom_page.demo', CustomPageDemo);
    
    return CustomPageDemo;
    
    });
```

上面这段代码的核心部分在 `AbstractAction.extend(...)` 中，其中 `template` 指定了自定义页面的模板名称，也就是我们前面所说的 `t-name` 这个属性，而 `events` 则定义了页面的一些事件，如该例中，定义的是当点击（`click`）带有类 `.demo-submit` 的元素时执行 `_onSubmitClick` 这个方法。

需要注意的是，我们需要通过 `core.action_registry.add()` 这个方法对动作进行注册，第一个参数表示注册的动作名（`tag`），第二个参数是要注册的动作对象。

# 创建菜单

既然自定义的页面是一个动作，那么我们就需要创建菜单和它进行关联，在 `views/views.xml` 定义菜单：

```xml
    <odoo>
        <data>
            <record id="action_custom_page" model="ir.actions.client">
                <field name="name">Custom Page</field>
                <field name="tag">custom_page.demo</field>
            </record>
    
            <menuitem
                    id="menu_root_custom_page"
                    name="Custom Page"
                    action="action_custom_page"
                    groups="base.group_user"
                    sequence="1"/>
        </data>
    </odoo>
```

# 加载资源

和普通页面一样，页面中所引用的 JS 或者 CSS 等资源文件，都需要在页面上引入才能使用，在 Odoo 中也不例外，不过有一点不同的是，Odoo 中有专门的集中引入资源文件的地方，这样做是为了将资源进行打包压缩，以减少请求的包体大小。

一般情况下都是在 `views/templates.xml` 中继承 `web.ssets_backend` 后在其末尾添加相关资源路径：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <data>
            <template id="assets_backend" name="custom page assets" inherit_id="web.assets_backend">
                <xpath expr="//script[last()]" position="after">
                    <script type="text/javascript" src="/custom_page/static/lib/wired-elements/wired-elements.bundled.min.js"></script>
                    <script type="text/javascript" src="/custom_page/static/src/js/demo.js"></script>
                </xpath>
            </template>
        </data>
    </odoo>
```

除了 JS 资源需要引入外，我们编写的页面模板也许要引入，打开 `__manifest__.py` 并在底部添加我们的自定义页面文件：

```json
'qweb': ["static/src/xml/base.xml"]
```

OK! 大功告成，一个最简单的自定义页面已经完成了，安装模块然后运行看看效果吧。

# 开发者模式

很不幸的是，打开之后好像只有几个小点在页面，然后并没有什么东西了。先不慌，我们先打开设置页面，然后在页面最右侧那一列的底部，可以看到两个链接，分别是『激活开发者模式』和『激活开发者模式 (带 assets)』，我们点一下第二个带 assets 的链接，然后再次访问我们的自定义页面，结果如何？

![Custom Page](/images/Odoo/Untitled-c2d08ea0-ca4f-4f99-8796-a9ce9c53445d.png)

那两个链接顾名思义，就是进入开发者模式，其中带 assets 的开发者模式会跳过执行资源打包的动作，在打开这个模式前，我们看到页面上的组件挤成一团了，是因为资源打包后导致 `wired-elements` 的部分属性无法找到。

**注：以上操作只是为了演示说明相关功能，在生产环境中，强烈不建议任何非开发或运维人员进入开发模式进行操作。**

源码都在仓库 [Odoo-Tutorial-Demo](https://github.com/ruter/Odoo-Tutorial-Demo/tree/master/custom_page) 中，请先自己动手实操，遇到问题再下载源码安装查看，如果有其他难题可以提出来，我会抽空解答的~

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/11/19/Add-custom-page-on-odoo/)。

本文永久地址：https://ruterly.com/2018/11/19/Add-custom-page-on-odoo/

