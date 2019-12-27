---
layout: post
title: Odoo 模块推荐——Multi-Steps Wizards
categories: Odoo
description: 一个可以让弹窗分步骤执行的模块
keywords: Odoo, 模块
---

Odoo 的 wizard 弹窗点击任意按钮后就会关闭，如果要实现多步输入的弹窗，就要自己实现很多相关的逻辑，如果你有这样的需求，不妨试试这次推荐的这个模块吧。

# 模块信息

|          |                                                              | 备注                  |
| -------- | ------------------------------------------------------------ | --------------------- |
| 名称     | Multi-Steps Wizards                                          |                       |
| 功能     | 这个模块提供了创建多步骤弹窗的基础功能                       |                       |
| 商店地址 | [点击前往](https://apps.odoo.com/apps/modules/12.0/multi_step_wizard/) | 商店中最高版本为 12.0 |
| 仓库地址 | [前往 GitHub](https://github.com/OCA/server-ux/tree/12.0/multi_step_wizard) | 仓库中最高版本为 12.0 |

# 使用体验

写了一个没有任何用处的 demo 来体验了一下这个模块，下面是演示的动图

![Multi Steps Wizard Demo](/images/Odoo/multi_step.gif)

说是一个 Demo，其实啥事没干，就 3 个文本字段，然后每一步显示一个字段，最后显示一句话提示完成。不如直接看看 Python 代码和视图定义：

```python
class DemoMultiStepWizard(models.TransientModel):
    _name = 'demo.multi.step.wizard'
    _inherit = ['multi.step.wizard.mixin']
    _description = 'Multi Step Wizard Demo'

    step1 = fields.Char('Step 1')
    step2 = fields.Char('Step 2')
    step3 = fields.Char('Step 3')

    @api.model
    def _selection_state(self):
        return [
            ('start', 'Start'),
            ('step2', 'Step 2'),
            ('step3', 'Step 3'),
            ('final', 'Final'),
        ]

    def state_exit_start(self):
        self.state = 'step2'

    def state_exit_step2(self):
        self.state = 'step3'

    def state_exit_step3(self):
        self.state = 'final'
```

上面的代码中 `_selection_state` 和 `state_exit_` 开头的几个方法，具体作用会在后文提到，这里只有一点需要注意，就是希望使用 Multi Step 的弹窗，都需要继承一个 mixin 模型 `_inherit = ['multi.step.wizard.mixin']` 才能实现，所以这不是一个开箱即用的模块，是需要开发的，但是相当简单，所以和开箱即用也没差太多了。

接下来是视图的定义：

```xml
<odoo>
  <data>
    <record model="ir.ui.view" id="multi_step_wizard_demo_form">
      <field name="name">demo.multi.step.wizard.form</field>
      <field name="model">demo.multi.step.wizard</field>
      <field name="mode">primary</field>
      <field name="inherit_id" ref="multi_step_wizard.multi_step_wizard_form"/>
      <field name="arch" type="xml">
        <xpath expr="//footer" position="before">
          <group name="Start" attrs="{'invisible': [('state', '!=', 'start')]}">
            <group>
              <field name="step1"/>
            </group>
          </group>
          <group name="Step 2" attrs="{'invisible': [('state', '!=', 'step2')]}">
            <group>
              <field name="step2"/>
            </group>
          </group>
          <group name="Step 3" attrs="{'invisible': [('state', '!=', 'step3')]}">
            <group>
              <field name="step3"/>
            </group>
          </group>
          <div name="Final" attrs="{'invisible': [('state', '!=', 'final')]}">
            <p>Done.</p>
          </div>
        </xpath>
      </field>
    </record>

    <record model="ir.actions.act_window" id="action_multi_wizard_demo">
      <field name="name">Multi Step Wizard Demo</field>
      <field name="res_model">demo.multi.step.wizard</field>
      <field name="view_mode">form</field>
      <field name="target">new</field>
    </record>

    <record model="ir.ui.view" id="multi_step_wizard_demo_user_form">
      <field name="name">demo.multi.step.wizard.user.form</field>
      <field name="model">res.users</field>
      <field name="inherit_id" ref="base.view_users_form"/>
      <field name="arch" type="xml">
        <header position="inside">
          <button type="action" name="%(action_multi_wizard_demo)d" string="Multi Wizard" class="oe_highlight"/>
        </header>
      </field>
    </record>
  </data>
</odoo>
```

这个 wizard 视图继承了模块的 `multi_step_wizard_form` 视图（后文同样有说明），然后在 `<footer />` 标签前新增了一些字段内容，根据状态 `state` 判断在什么时候该显示什么内容。

为了演示，我在用户表单上添加了一个按钮，用于触发这个 Demo 的多步弹窗。整体的实现难度其实可以说是 0 了，因为在模块的 Readme 中已经给出了示例代码，使用的时候只需要复制粘贴然后根据需求修改就好了，超简单有没有？

模块已经封装了一些方法，让你可以直接在继承之后使用，而不需要费心去处理不同状态需要执行的方法和窗口的重新打开，只需要实现每个状态对应的方法即可。

# 适用场景

在一些需要分步骤进行，并且可能会根据前面步骤所得到的值进行一些处理和分支判断的场景，会使用到这个模块提供的能力。例如一个简单的报销流程可能是这样的：

1. 选择报销类型
2. 填写报销金额
3. 根据报销类型和金额判断是否超过一定数额，是则下一步，否则跳到 5
4. 填写报销超额说明
5. 提交

上述流程如果使用这个模块提供的能力就可以轻松地实现相应的需求。

# 实现浅析

在文件 `/models/multi_step_wizard.py` 中可以看到一段注释，这段内容已经把这个模块的功能和使用方式描述清楚了：

```
Mixin to ease the creation of multisteps wizards

_selection_state must return all possible step of
the wizard.

For each state but final, there must be a method named
"state_exit_X" where X is the name of the state. Each
of these method must set the next state in self.state.

The final state has no related method because the view
should only display a button to close the wizard.

open_next and _reopen_self should not need to be
overidden, but _selection_state and state_exit_start
likely will need to.
```

接下来具体看看代码上的实现。首先看一下 `state` 字段：

```python
state = fields.Selection(
    selection='_selection_state',
    default='start',
    required=True,
)

@api.model
def _selection_state(self):
    return [
        ('start', 'Start'),
        ('final', 'Final'),
    ]
```

方法 `_selection_state` 需要在具体的实现中被重写，返回一个表示当前 wizard 所有状态（步骤）的元组列表，需要注意一点，返回的状态中（默认情况下）需要有一个值是 `final` 以表示结束状态，默认状态使用的是 `start`，若不重写状态字段的 `default` 值，则返回的状态中还应有值 `start` 表示初始状态。

接下来再看两个不需要重写的方法：

```python
def open_next(self):
    state_method = getattr(self, 'state_exit_%s' % (self.state,), None)
    if state_method is None:
        raise NotImplementedError(
            'No method defined for state %s' % (self.state,)
        )
    state_method()
    return self._reopen_self()

def _reopen_self(self):
    return {
        'type': 'ir.actions.act_window',
        'res_model': self._name,
        'res_id': self.id,
        'view_mode': 'form',
        'target': 'new',
    }
```

方法 `open_next` 会在点击弹窗中的 `Next` 按钮后被调用，目的是执行每个状态变化时对应的方法，这些方法以 `state_exit_X` 规则命名，其中 `X` 表示当前的状态值。这就意味着你需要实现 N 个这样规则命名的方法，N 是你的总状态数 -1（排除结束状态），如当前状态是默认状态 `start`，你就需要有一个方法 `state_exit_start`，在点击按钮 `Next` 后就会被调用。在你写的所有这些状态相关的方法中，每一个方法都需要将状态 `state` 进行变更。

在 `open_next` 的最后会执行 `_reopen_self` 方法重新打开一个窗口，这一步操作是因为 wizard 的弹窗，会在点击任何按钮之后关闭，所以需要调用该方法再次打开窗口。

来看看文件 `/views/multi_step_wizard_views.xml` 里的内容：

```xml
<record id="multi_step_wizard_form" model="ir.ui.view">
  <field name="name">multi.step.wizard.form</field>
  <field name="model">multi.step.wizard.mixin</field>
  <field name="arch" type="xml">
    <form>
      <field name="state" invisible="1"/>
      <footer>
        <div name="states_buttons" attrs="{'invisible': [('state', '=', 'final')]}">
          <button name="open_next" string="Next" type="object" class="btn-primary"/>
          <button string="Cancel" class="btn btn-default" special="cancel" />
        </div>
        <div name="final_buttons" attrs="{'invisible': [('state', '!=', 'final')]}">
          <button string="Close" class="btn btn-primary" special="cancel" />
        </div>
      </footer>
    </form>
  </field>
</record>
```

这个视图中已经将状态字段 `state` 添加进去了，然后按钮 `Next` 也已经包含在里 `footer` 中，可以看到它默认是以状态 `final` 作为结束状态进行判断该显示哪一组按钮的，这就是为什么上面会特意提醒 `_selection_state` 需要返回的状态中需要包含 `final` 这个状态。

一般情况下，只要是使用了这个模块，我们定义 wizard 的视图都会继承这个模块已经写好的视图，然后在标签 `<footer />` 前面添加我们的视图内容：

```xml
<field name="mode">primary</field>
<field name="inherit_id" ref="multi_step_wizard.multi_step_wizard_form"/>
<field name="arch" type="xml">
  <xpath expr="//footer" position="before">
    <field name="..."/>
  </xpath>
</field>
```

这里还有一点需要注意，继承这个视图，需要设置字段 `mode` 的值为 `primary`，具体原因请看源码中的这段帮助文本：

```
Only applies if this view inherits from an other one (inherit_id is not False/Null).

* if extension (default), if this view is requested the closest primary view
is looked up (via inherit_id), then all views inheriting from it with this
view's model are applied

* if primary, the closest primary view is fully resolved (even if it uses a
different model than this one), then this view's inheritance specs
(<xpath/>) are applied, and the result is used as if it were this view's
actual arch.
```



---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2019/12/28/Odoo-Module-Recommend-Multi-Step-Wizards/)。

本文永久地址：https://ruterly.com/2019/12/28/Odoo-Module-Recommend-Multi-Step-Wizards/
