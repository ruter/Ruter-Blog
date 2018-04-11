---
layout: wiki
title: "Flask中文编码"
date: 2016-11-08 13:51
keywords: flask, 编码
categories: Python
---

使用`Flask`编写API时，返回的`json`里如果有中文会遇到编码的问题，可以用如下方法解决:

```python
from flask import json, make_response
return make_response(json.dumps({'result': get_entry_list()}, ensure_ascii=False).decode('utf-8'))
```

将`ensure_ascii`设置成`False`，并且将数据用`make_response()`包裹起来