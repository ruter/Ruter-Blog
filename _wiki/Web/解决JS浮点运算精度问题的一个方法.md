---
layout: wiki
title: "解决JS浮点运算精度问题的一个方法"
date: 2017-03-14 17:31
keywords: JavaScript, JS
categories: Web
---

在 JS 中进行浮点运算会出现精度有误差的问题，例如：`1 - 0.9`会得出`0.09999999999999998`这样的结果。

可是对整数进行运算就不会出现这种情况。根据这个特点我们可以对浮点数先进行处理后再运算，将浮点数乘以 10 的 n 次方，运算完成后再将结果除以 10 的 n 次方，其中 n 为对应浮点数小数位的位数：

```javascript
var formatFloat = function (num, dig) {
  var pow = Math.pow(10, Math.abs(dig));
  if (dig < 0) {
    return num / pow;
  } else if (dig > 0) {
    return num * pow;
  } else {
    return num;
  }
}
```

> 注：`num`为要处理的数值，`dig`为小数点的位数

对前面的例子进行计算：`formatFloat(formatFloat(1, 1) - formatFloat(0.9, 1), -1)`可以得出正确的结果 `0.1`。