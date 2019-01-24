---
layout: post
title: Odoo 模块推荐——Audit Trail
categories: Odoo
description: 根据规则追踪指定模型和用户增删改的历史记录
keywords: odoo, 模块
---

今天推荐的是审计模块，可以根据规则追踪指定模型和用户组的创建、修改或删除操作，以及相关操作变化前后的历史数据，并且可以提供历史数据查看功能。

# 模块信息

|          |                                                              | 备注                  |
| -------- | ------------------------------------------------------------ | --------------------- |
| 名称     | Audit Trail                                                  |                       |
| 功能     | 可以通过审计规则对用户在系统中的操作进行追踪记录，当前支持对增删改操作进行记录 |                       |
| 商店地址 | [点击前往](https://www.odoo.com/apps/modules/12.0/smile_audit/) | 支持 Odoo 8.0 到 12.0 |
| 仓库地址 | [前往 GitHub](https://github.com/Smile-SA/odoo_addons/tree/12.0/smile_audit/) | 支持 Odoo 8.0 到 12.0 |

# 使用体验

这个审计模块在使用前需要先配置好规则，通过「Settings -> Audit -> Rules」菜单打开规则列表，规则的添加很简单，每一列按顺序分别为名称、模型、是否记录增改删操作、跟踪的用户组以及是否启用规则。除了规则名称需要手动输入外，其他字段只需要用鼠标点选就行了：

![审计规则](/images/Odoo/Untitled-1aa160da-233c-43ef-bf92-284586636086.png)

这里我创建了两条记录，分别对应于用户（`res.users`）模型和员工（`hr.employee`）模型，其中规则 User 没有设置用户组，而 Employee 规则设置了员工模块里的「经理」用户组，接下来就看看这两条规则各自的区别。

为了演示，我先用管理员账户创建了一个新用户 Test User 1 作为测试，并且对该用户进行多次修改（添加管理员权限以及员工模块的经理权限组）操作，现在通过 Action 下的「View audit logs」打开这条用户记录的操作记录列表：

![新用户表单视图](/images/Odoo/Untitled-d2c173e0-0b00-4fa4-8ccd-769e4a6d888f.png)

如果你没有打开开发者模式，那么你看到的列表视图（以及对应记录的表单视图）将会缺少一些关键信息，如操作的记录 ID、记录对应的模型以及操作的类型：

![审计记录列表](/images/Odoo/Untitled-c53c0e72-62e8-4444-9d2e-919219e450d9.png)

这里可以看到记录了当前的管理员账户 Mitchell Admin 所做的操作，另外还有 OdooBot 的操作记录，可以不用理会。打开新的隐身窗口，登录 Test User 1 这个账户，然后对修改该账户的任意字段后打开记录列表查看：

![Test User 1 的操作记录](/images/Odoo/Untitled-50a52226-4a6d-4e30-bcd8-33cf746cafbf.png)

接下来打开员工模块，选择 Marc Demo 这个员工，编辑他的用户名和其他的一些字段：

![修改后的员工表单视图](/images/Odoo/Untitled-e54932e6-90bd-4527-8ecc-d9cae4dfb877.png)

上图是修改之后的结果，现在打开审计记录的列表，选择刚刚产生的变更记录打开，可以看到表单下面列出了发生变更的字段以及原本的值和变更后的值：

![审计记录表单视图](/images/Odoo/Untitled-c6563ee0-1955-4c0c-9614-a1f095b0960d.png)

我们之前做的所有变更这里都一目了然，这还没完，发现右上角有个「History revision」按钮了吗，打开它，我们会发现新世界：

![员工记录变更前的历史内容](/images/Odoo/Untitled-570f6303-100e-4924-b535-e584cf5c76ae.png)

变更前的表单视图，完整地呈现在我们面前了👏对比前面的截图，可以发现高亮的位置就是我之前做过修改的地方，这个功能在某些业务场景下，简直不要太好用！

接下来是最后一个演示，修改 Test User 1 账户的员工权限组为 Officer，然后再对刚刚的 Demo 员工进行修改操作：

![仅有一条审计记录](/images/Odoo/Untitled-ade6b5ab-bb91-41c7-b9fe-bb0819e31c8e.png)

这里我将这个员工的用户名修改回去了（通过面包屑可以看出），但是我们在审计记录里却没有看到任何的操作记录，这是因为前面添加的员工模型的审计规则，指定的用户组为 Manager，而当前用户所在的用户组为 Officer，所以并没有记录相关的操作。

换句话说就是，规则中若指定了用户组，则只会跟踪该组用户的操作记录，若留空则表示记录所有用户的操作记录。

# 适用场景

会使用到这个模块的地方，显而易见的，都是在一些重要业务数据对应的模型上的，审计记录可以很清楚地记录下符合规则的所有操作，同时还能回看历史数据。这些操作记录，在发生了某些不可预测的操作所导致的数据变更或删除之后，是十分有效的追溯手段，同时也能为数据复原提供一定的参照。

建议仅在关键业务添加审计规则，要避免产生大量无意义的审计记录，这会对系统的性能产生一定的影响。

# 实现浅析

这个模块没有涉及到任何前端的内容，所以我们这次一点 JS 代码都不用看了🙊下面这段来自 `models/audit_rule.py` 的代码就是这个审计模块最核心的部分了：

```python
@api.model_cr
def _register_hook(self, ids=None):
    self = self.sudo()
    updated = False
    if ids:
        rules = self.browse(ids)
    else:
        rules = self.search([])
    for rule in rules:
        if rule.model_id.model not in self.env.registry.models or \
                not rule.active:
            continue
        RecordModel = self.env[rule.model_id.model]
        for method in self._methods:
            func = getattr(RecordModel, method)
            while hasattr(func, 'origin'):
                if func.__name__.startswith('audit_'):
                    break
            else:
                RecordModel._patch_method(method, audit_decorator(method))
        updated = bool(ids)
    if updated:
        self.clear_caches()
    return updated
```

这是一个钩子方法，每个模型的这个方法都会在模块加载时被执行。审计规则的钩子方法主要做了两件事：

1. 遍历审计规则，跳过未启用的以及所关联模型未被注册的规则
2. 为符合条件的审计规则所关联的模型执行方法 `_patch_method()`

其中 2 所执行的方法 `_patch_method()` 通过猴子补丁（Monkey Patch）将所关联模型的 `_create`, `_write` 和 `unlink` 三个方法在运行时动态增加了一些功能，在 `tools/decorator.py` 中可以看到 `audit_decorator()` 里以 `audit_` 开头的函数就是对应上述三个方法的修改内容：

```python
@api.model
def audit_create(self, vals):
    result = audit_create.origin(self, vals)
    record = self.browse(result) if isinstance(result, (int, long)) \
        else result
    rule = get_audit_rule(self, 'create')
    if rule:
        new_values = record.read(load='_classic_write')
        rule.log('create', new_values=new_values)
    return result

@api.multi
def audit_write(self, vals):
    rule = get_audit_rule(self, 'write')
    if rule:
        old_values = self.read(load='_classic_write')
    result = audit_write.origin(self, vals)
    if rule:
        new_values = self.read(load='_classic_write')
        rule.log('write', old_values, new_values)
    return result

@api.multi
def audit_unlink(self):
    rule = get_audit_rule(self, 'unlink')
    if rule:
        old_values = self.read(load='_classic_write')
        rule.log('unlink', old_values)
    return audit_unlink.origin(self)
```

这三个方法做的事情都一样，就是在规则所关联的模型执行相关操作前后，分别记录下数据的值，然后创建审计记录。

![TNK-Studio 公众号](/images/mp_qrcode.jpg)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2019/01/24/Odoo-Module-Recommend-Audit-Trail/)。

本文永久地址：https://ruterly.com/2019/01/24/Odoo-Module-Recommend-Audit-Trail/