---
layout: post
title: Odoo 电商下单用户上传自定义图片
categories: Odoo
description: 电商用户上传自定义 Odoo 商品图片属性
keywords: Odoo, Research
---

在 Odoo 的属性值中，我们可以设定自定义值的属性，用于某些商品的自定义属性。但是在一些场景下，客户会需要在下单时上传图纸或者样式图等，以图片进行辅助定制化产品，此时 Odoo 默认的自定义属性就不能满足需求了。

可以通过对 `product.attribute` 模型进行扩展，添加图片类型的属性

```python
class ProductAttribute(models.Model):
    _inherit = "product.attribute"

    display_type = fields.Selection(selection_add=[('image', 'Image')])
```

效果如下图所示

![创建图片类型的自定义属性](/images/Odoo/odoo_product_attribute_value.png)

仅仅添加新的属性类型还不够，在对前端组件进行扩展后，才可达到预期的效果

![自定义图片属性演示](/images/Odoo/odoo_custom_img_shop.gif)

在购物车中提供自定义图片的查看功能，方便下单前的最后确认

![可查看自定义的图片](/images/Odoo/odoo_shop_cart_img.png)

在后台的订单明细中，打开明细行的表单后即可看见客户提交的自定义图片

![订单明细行表单](/images/Odoo/odoo_order_line_img.png)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2020/03/13/Odoo-eCommerce-customize-img-attr/