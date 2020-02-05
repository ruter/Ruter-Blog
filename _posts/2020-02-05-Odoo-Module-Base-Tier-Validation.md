---
layout: post
title: Odoo 模块推荐——Base Tier Validation
categories: Odoo
description: 给模型添加审批功能的实用模块
keywords: Odoo, 模块
---

在各种企业信息系统中，我们经常会碰到需要使用审批流的情况，而在 Odoo 却没有一套完整的解决方案，很多时候都靠开发人员根据需求进行审批流的硬编码，这对系统代码的可维护性和审批流的灵活性是非常不利的。这次推荐的模块可以在一定程度上解决审批流的问题，让系统可以用更灵活的方式去创建和管理审批流及审批节点。

# 模块信息

|          |                                                              | 备注                  |
| -------- | ------------------------------------------------------------ | --------------------- |
| 名称     | Base Tier Validation                                         |                       |
| 功能     | 模块提供了抽象的方法和模型供需要审批流的模型使用             |                       |
| 商店地址 | [点击前往](https://www.odoo.com/apps/modules/12.0/base_tier_validation/) | 商店中最高版本为 12.0 |
| 仓库地址 | [前往 GitHub](https://github.com/OCA/server-ux/tree/12.0/base_tier_validation) | 仓库中最高版本为 12.0 |

# 使用体验

这个模块本身仅提供了抽象模型和方法，并不能直接使用，言外之意就是需要适当的开发才可以使用模块提供的审批功能。为了可以直接体验，省去开发的步骤，我同时还安装了一个使用了该模块的社区模块 [Purchase Tier Validation](https://github.com/OCA/purchase-workflow/tree/12.0/purchase_tier_validation)，在 Base Tier Validation 的 Readme 中也有提到它。

首先要做的当然是安装模块了，然后需要在菜单「设置 → 技术 → 层级审批 → 层级定义」中创建审批规则，这里的审批规则相当于我们常说的审批流中的审批节点，每一个层级都是一个审批节点。

![Odoo%20Base%20Tier%20Validation/Untitled.png](/images/Odoo/base_tier_validation/Untitled.png)

如果找不到菜单，请不要忘记激活 Odoo 的开发者模式。主要的规则配置项都已经在上图中进行了标注说明，另外需要说明的是配置最后的定义和定义域，这里会根据填写的 domain 过滤出适用于当前审批规则的记录集，换言之，只要可以通过定义的域匹配到的记录，都需要根据当前审批规则进行审批。

为了进行演示，这里创建了两条采购订单的审批规则，这里需要特意说明一点，按序列顺序审批时，序列值越大，审批顺序越靠前。

![Odoo%20Base%20Tier%20Validation/Untitled%201.png](/images/Odoo/base_tier_validation/Untitled 1.png)

接着打开菜单「采购 → 询价单」列表，并点开任意一条示例数据，点击「请求验证」按钮提交审批，下图为提交审批后的页面

![Odoo%20Base%20Tier%20Validation/Untitled%202.png](/images/Odoo/base_tier_validation/Untitled 2.png)

此时会看到表单顶部出现了「这个采购订单需要 验证。」的提示信息，因为当前登录的用户是审批规则中的「经理」，属于后审批的用户，所以无法对当前记录进行审批操作。这里将审批规则中「经理」的审批序列值调整一下，然后打开另一条询价单提交审批

![Odoo%20Base%20Tier%20Validation/Untitled%203.png](/images/Odoo/base_tier_validation/Untitled 3.png)

可以看到审批列表中「经理」的审批排在了前面，然后顶部也多出了两个审批操作按钮，点击「验证」或「拒绝」时，会根据审批规则里是否勾选了选项「Comment」决定是否弹出备注的输入框

![Odoo%20Base%20Tier%20Validation/Untitled%204.png](/images/Odoo/base_tier_validation/Untitled 4.png)

当前用户审批通过后，审批列表会发生变化，记录审批状态和备注等信息

![Odoo%20Base%20Tier%20Validation/Untitled%205.png](/images/Odoo/base_tier_validation/Untitled 5.png)

如果需要审批的记录，在未完全审批通过时强行提交，会发生什么？

![Odoo%20Base%20Tier%20Validation/Untitled%206.png](/images/Odoo/base_tier_validation/Untitled 6.png)

你会发现如上图的提示，所以还是好好地等待审批流程走完吧。如果你有留意到右上角系统托盘位置的话，你会发现所有需要当前用户审批的记录数都会在这里出现，点击打开后可以看见对应需要审批的记录类型和各自等待审批的数量

![Odoo%20Base%20Tier%20Validation/Untitled%207.png](/images/Odoo/base_tier_validation/Untitled 7.png)

# 适用场景

通过直接的体验可以确定，这个审批模块已经可以覆盖大部分流程单一的审批流了，配合审批规则中定义域的使用，也能实现不同记录的审批走不同的审批流，例如最常见的，根据采购金额所在区间走不同的审批流，由不同级别的管理人员进行审批，通过定义域的配置就可以简单实现。

但是对于更复杂一些的审批流程，例如需要动态指定审批人这种，只靠这个模块是不够的，还好社区里已经有了一个更高级的模块 [Base Tier Validation Formula](https://github.com/OCA/server-ux/tree/12.0/base_tier_validation_formula) 可以作为补充，让你可以通过 Python 代码来动态指定审批人，除此之外，规则对应的审批记录也可以通过代码来进行指定

![Odoo%20Base%20Tier%20Validation/Untitled%208.png](/images/Odoo/base_tier_validation/Untitled 8.png)

通过以上两个模块的组合使用，基本上常见的审批流程都能覆盖了，如果还有更复杂的审批流呢？那很抱歉，只能自己动手了，这两个模块是非常好的参考，如果有更复杂的需求，可以基于它们进行扩展开发。

# 实现浅析

由于这个模块本身的设计和实现比较复杂，抽象的方法也比较多，因篇幅所限这里就不展开细说，推荐大家直接边体验边阅读源码。

最后根据模块 [Purchase Tier Validation](https://github.com/OCA/purchase-workflow/tree/12.0/purchase_tier_validation) 简单说下如何使用这次推荐的审批模块。首先是继承模型 `tier.definition` 并在方法 `_get_tier_validation_model_names` 添加可选的参考模型

```python
class TierDefinition(models.Model):
    _inherit = "tier.definition"

    @api.model
    def _get_tier_validation_model_names(self):
        res = super(TierDefinition, self)._get_tier_validation_model_names()
        res.append("purchase.order")
        return res
```

然后在需要添加审批功能的模型中继承 `tier.validation` 模型，并设置 `_state_field`, `_state_from`, `_state_to` 及 `_cancel_state` 这四个属性

```python
class PurchaseOrder(models.Model):
    _name = "purchase.order"
    _inherit = ['purchase.order', 'tier.validation']
		_state_field = 'state'
    _state_from = ['draft', 'sent', 'to approve']
    _state_to = ['purchase', 'approved']
		_cancel_state = 'cancel'
```

上述四个属性分别表示对应模型的状态字段相关的值：

- _state_field: 状态字段名，默认值为 `state`
- _state_from: 审批前记录的状态字段值
- _state_to: 审批后记录的状态字段值
- _cancel_state: 记录为取消状态时的值，默认值为 `cancel`

剩下的就是添加和修改视图中的动作按钮了，这里就不赘述了。

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2020/02/05/Odoo-Module-Base-Tier-Validation/)。

本文永久地址：https://ruterly.com/2020/02/05/Odoo-Module-Base-Tier-Validation/
