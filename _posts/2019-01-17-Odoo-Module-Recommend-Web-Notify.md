---
layout: post
title: Odoo 模块推荐——Web Notify
categories: Odoo
description: 通过总线发送提示信息给用户，并显示在系统的右上角
keywords: odoo, 模块
---

又到了模块推荐时间，这次要推荐的模块可以通过 Python 代码发送提示信息给当前回话中的用户，并将提示内容显示在右上角，很简单却很实用的一个模块。

# 模块信息

|          |                                                              | 备注                  |
| -------- | ------------------------------------------------------------ | --------------------- |
| 名称     | Web Notify                                                   |                       |
| 功能     | 通过 Python 代码发送提示信息给当前回话中的用户               |                       |
| 商店地址 | [点击前往](https://apps.odoo.com/apps/modules/11.0/web_notify/) | 商店中最高版本为 11.0 |
| 仓库地址 | [前往 GitHub](https://github.com/OCA/web/tree/12.0/web_notify/) | 仓库中最高版本为 12.0 |

# 使用体验

模块安装完毕之后，我们随意打开一个用户的表单页面，然后找到选项卡的第二项「Test web notify」并打开，可以看到模块自带了用于测试效果的 Demo，先分别点击该选项卡下的两个按钮看看效果：

![提示信息](/images/Odoo/odoo_notify_1.png)

我们可以看到右上角分别出现了一个黄色（info）的和一个红色（warning）的提示信息，若干秒之后会消失，这个模块就做这一件事。

提示信息的图标旁边以黑色粗体显示的是标题，分割线下面的是提示信息内容，此处因为 Demo 的按钮直接调用相关方法导致参数 `message` 的值变成了传入的 `context`，所以信息内容变成了 [object Object]，我们先做一点小修改，然后再继续往下。

打开 `models/res_users.py` 找到如下方法并修改：

```python
# 修改前
def notify_info(self, message="Default message", title=None, sticky=False):
    title = title or _('Information')
    self._notify_channel('notify_info_channel_name', message, title, sticky)

def notify_warning(self, message="Default message", title=None, sticky=False):
    title = title or _('Warning')
    self._notify_channel('notify_warning_channel_name', message, title, sticky)

# 修改后
def notify_info(self, ctx, message="Default message", title=None, sticky=True):
    title = title or _('Information')
    self._notify_channel('notify_info_channel_name', message, title, sticky)

def notify_warning(self, ctx, message="Default message", title=None, sticky=False):
    title = title or _('Warning')
    self._notify_channel('notify_warning_channel_name', message, title, sticky)
```

我们为上面的两个方法都添加了一个参数 `ctx` 表示从视图中传入的 `context` 上下文内容，然后将 `notify_info()` 的 `sticky` 默认值改成了 `True`，重新运行后我们再来看看效果：

![修改后的提示信息](/images/Odoo/odoo_notify_2.png)

好了，这回提示信息的内容终于正常显示了，然后你会发现警告提示自动消失了（和之前的表现一样），而普通提示则一直保留在页面上，并且在它的右上角还多了一个关闭按钮，这是因为刚刚修改代码的时候把 `notify_info()` 的 `sticky` 改成了 `True`，这个参数决定了提示信息会不会被固定在页面上，当它被固定后，只有手动点击关闭按钮才会消失。

# 适用场景

这种消息提示在很多场景中我们都会用到，主要目的是给用户一个反馈和提醒，它的作用应该是弱于弹窗提示的。弹窗提示一般会中断用户当前的操作，并且让用户在弹窗中可以进行操作，而出现在右上角的消息提示一般是在用户操作结束后给出提示，提供结果的反馈。

假设现在有一张采购订单，用户可以在这张单上执行一键催单的操作，当用户执行完之后，如果此时系统并没有任何的反馈告知用户催单邮件已经发送给供应商的话，这个用户会怎么办呢？

他很可能会多次执行该操作，从而导致多次发出催单邮件给供应商，因为他从直觉上会认为系统出问题了，刚刚的操作没有产生任何的效果。如果我们通过这个模块发送提示信息给用户，让他可以接收到操作成功的反馈，他一定不会多次执行催单操作的，除非这个供应商半个月了还没发货😡

# 实现浅析

Web Notify 中用于消息通知的核心部分来自于 `Bus` 模块，它是通过长连接（longpolling）来实现即时通知的。

打开 `static/src/js/web_client.js` 看看模块的实现：

```javascript
start_polling: function() {
    this.channel_warning = 'notify_warning_' + session.uid;
    this.channel_info = 'notify_info_' + session.uid;
    this.call('bus_service', 'addChannel', this.channel_warning);
    this.call('bus_service', 'addChannel', this.channel_info);
    this.call('bus_service', 'on', 'notification', this, this.bus_notification);
    this.call('bus_service', 'startPolling');
}
```

在 `start_polling()` 中，前两行根据当前用户的 id 定义了两个频道标识符（`channel identifier`），分别是警告和普通提示；其后两行则是调用 `addChannel()` 将刚定义的频道标识符加入监听列表；接下来是绑定通知触发事件以及启动长连接。

这个流程也是我们利用总线消息通知监听和触发事件的标准流程：

1. 添加监听频道
2. 绑定事件
3. 开始长连接
4. 发送通知到指定频道触发事件

然后我们再看看发送通知的实现：

```python
def notify_info(self, message="Default message", title=None, sticky=False):
    title = title or _('Information')
    self._notify_channel('notify_info_channel_name', message, title, sticky)

def notify_warning(self, message="Default message", title=None, sticky=False):
    title = title or _('Warning')
    self._notify_channel('notify_warning_channel_name', message, title, sticky)
```

这两个方法是我们的用于发送通知时直接调用的方法：

```python
# 普通提示
self.env.user.notify_info(message='My information message')
# 警告提示
self.env.user.notify_warning(message='My marning message')
```

这两个方法都执行了同一个私有方法 `_notify_channel()`，只是参数值不同：

```python
def _notify_channel(self, channel_name_field, message, title, sticky):
    if (not self.env.user._is_admin() and any(user.id != self.env.uid for user in self)):
        raise exceptions.UserError(
            _('Sending a notification to another user is forbidden.')
        )
    bus_message = {
        'message': message,
        'title': title,
        'sticky': sticky
    }
    notifications = [(record[channel_name_field], bus_message) for record in self]
    self.env['bus.bus'].sendmany(notifications)
```

最终真正发出通知的是 `self.env['bus.bus'].sendmany(notifications)` 这一句，除此之外还有发送单个通知的方法：

```python
# 发送多个通知
self.env['bus.bus'].sendmany([(channel1, message1), (channel2, message2), ...])
# 发送单个通知
self.env['bus.bus'].sendone(channel, message)
```

这里有一份 `bus.bus` 的参考文档：[bus.bus](https://odoo-development.readthedocs.io/en/latest/odoo/models/bus.bus.html)

这份文档里有更详细的说明，虽然不一定适用于当前的 Odoo 版本，但是可以很大程度上帮助理解相关的概念。

**注意：在正式环境使用前，务必要将前面做的修改还原，并且删除 `__manifest__.py` 中的 `'views/res_users_demo.xml'` 这一行内容。**

![TNK-Studio 公众号](/images/mp_qrcode.jpg)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2019/01/17/Odoo-Module-Recommend-Web-Notify/