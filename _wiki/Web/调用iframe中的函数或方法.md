---
layout: wiki
title: "调用iframe中的函数或方法"
date: 2017-03-21 19:43
keywords: iframe, JavaScript
categories: Web
---

假设 A 页面有一个`iframe`的 ID 为`mainfr`，链接的是 B 页面。

其中 B 页面有一个函数`callSub()`，如果要在 A 页面中对它进行调用，可以这样操作：

```javascript
$("#mainfr")[0].contentWindow.callSub();
```