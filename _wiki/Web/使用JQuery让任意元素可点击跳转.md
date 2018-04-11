---
layout: wiki
title: "使用JQuery让任意元素可点击跳转"
date: 2017-02-25 10:21
keywords: Javascript, JQuery, HTML, clickable
categories: Web
---

在做页面的时候，可能会遇到要让某些非`a`标签的元素可以被点击并实现类似`a`标签的链接跳转功能，例如要让一个`table`的行可以点击并跳转，我们可以利用JS进行实现。

# 给需要点击跳转的元素加上标识

例如给`table`的`tr`加上一个`clickable`类，然后加上一个`data-href`属性，属性值为需要跳转的目的地址：

```html
<tbody>
  <tr class="clickable" data-href="http://ruterly.com">
    <td></td>
    <td></td>
    <td></td>
  </tr>
</tbody>
```

# 通过JS进行跳转

使用`JQuery`给带有类`clickable`的元素加上点击事件，跳转目的地址是`data-href`属性的值：

```javascript
$(document).ready(function() {
  $(".clickable").click(function() {
    window.document.location = $(this).data("href");
  });
});
```

# 添加点击指针样式

为了让用户可以清楚地知道这个元素是可点击的，我们将鼠标的悬停样式修改为`pointer`：

```css
.clickable {
  cursor: pointer;
}
```

完成！之后要让任意元素可以点击跳转，只需要加上类`clickable`以及`data-href`属性就OK了。
