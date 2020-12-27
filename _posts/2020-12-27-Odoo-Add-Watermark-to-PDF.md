---
layout: post
title: Odoo 下载 PDF 附件添加水印
categories: [Odoo, Tutorial]
description: 给从 Odoo 中下载的 PDF 附件添加水印
keywords: Odoo, Tutorial
---

这篇内容给大家简单讲讲怎么给 Odoo 中的 PDF 附件（ir.attachment）添加水印，本文相关代码均基于 Odoo 14 进行开发，在其他版本中可能会有所差异，请自行调试修改。

# 安装依赖

生成水印时会用到的库 `reportlab` 在 Odoo 14 的依赖中已经存在，如果你正确安装了依赖，则无需再次安装，如果你使用的是其他版本的 Odoo 或当前环境中没有找到该库，可以使用以下命令进行安装：

```bash
pip install reportlab
```

# 功能实现

首先第一步当然是创建一个新的模块，这里把模块命名为 `pdf_watermark`，基本的目录结构如下：

```bash
pdf_watermark
├── __init__.py
├── __manifest__.py
└── models
    ├── __init__.py
    └── models.py
```

Odoo 中附件的下载会经过 `ir.http` 的 `def binary_content()` 方法获取附件内容等必要信息，所以我们需要继承 `ir.http` 模型并重写 `binary_content` 方法，对 PDF 类型的附件添加水印，在 `[models.py](http://models.py)` 中添加继承的代码：

```python
import base64
import logging

from odoo import models

_logger = logging.getLogger(__name__)

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def binary_content(self, *args, **kwargs):
        status, headers, content = super(IrHttp, self).binary_content(*args, **kwargs)

        # 从请求头中获取附件类型，仅对 PDF 类型附件添加水印
        content_type = dict(headers).get('Content-Type')
        if content_type == 'application/pdf':
            content = self.add_watermark(base64.b64decode(content))
            content = base64.b64encode(content)

        return status, headers, content
```

上面的这段代码非常简单，就不多解释了，添加水印的核心在 `self.add_watermark()` 这句代码里，接下来我们就要实现这个方法。

在 `def add_watermark()` 这个方法中，大致会分成三个部分：

1. 获取水印文本
2. 生成水印文件
3. 在 PDF 中添加水印（合并水印文件）

其中前面两部分的实现如下：

```python
from reportlab.pdfbase import pdfmetrics, ttfonts
from reportlab.pdfgen import canvas
from reportlab.lib.units import cm

from odoo import fields, models

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def binary_content(self, *args, **kwargs):
        ...

    def _get_watermark(self):
        """
        获取水印文本
        :return:
        """
        return f'{self.env.company.name} {fields.Date.context_today(self)}'

    def _generate_watermark(self):
        """
        生成水印
        :return:
        """
        # 水印文件临时存储路径
        filename = f'/tmp/watermark.pdf'
        watermark = self._get_watermark()
        # 获取画布并修改原点坐标
        c = canvas.Canvas(filename)
        c.translate(1.5 * cm, -3 * cm)

        try:
            font_name = 'SimSun'
            # 从系统路径中引入中文字体(新宋)
            pdfmetrics.registerFont(ttfonts.TTFont('SimSun', 'SimSun.ttf'))
        except Exception as e:
            # 默认字体，不支持中文
            font_name = 'Helvetica'
            _logger.error(f'Register Font Error: {e}')

        # 设置字体及大小，旋转 -20 度并设置颜色和透明度
        c.setFont(font_name, 14)
        c.rotate(20)
        c.setFillColor('#27334C', 0.15)
        # 平铺写入水印
        for i in range(0, 30, 6):
            for j in range(0, 35, 5):
                c.drawString(i * cm, j * cm, watermark)
        c.save()
        return filename
```

这里取了当前的公司名称及当前日期作为水印文本，你可以根据需要对 `def _get_watermark()` 这个方法进行修改，获取你所需的水印文本。

先看一下 `def _generate_watermark()` 这个方法里的 `try...except...` 语句，这里会尝试注册中文字体新宋，如果失败则会使用默认字体 `Helvetica`。

如果你的运行环境中没有对应的中文字体，但是水印中又出现中文的话，在输出的 PDF 文件里水印的中文会变成一个个方框，所以别忘了给运行环境安装可以使用的中文字体。

reportlab 会从系统路径中查找字体，以下是预定义的一些目录：

```python
TTFSearchPath = (
	'c:/winnt/fonts',
	'c:/windows/fonts',
	'/usr/lib/X11/fonts/TrueType/',
	'/usr/share/fonts/truetype',
	'/usr/share/fonts',             #Linux, Fedora
	'/usr/share/fonts/dejavu',      #Linux, Fedora
	'%(REPORTLAB_DIR)s/fonts',      #special
	'%(REPORTLAB_DIR)s/../fonts',   #special
	'%(REPORTLAB_DIR)s/../../fonts',#special
	'%(CWD)s/fonts',                #special
	'~/fonts',
	'~/.fonts',
	'%(XDG_DATA_HOME)s/fonts',
	'~/.local/share/fonts',
	#mac os X - from
	#http://developer.apple.com/technotes/tn/tn2024.html
	'~/Library/Fonts',
	'/Library/Fonts',
	'/Network/Library/Fonts',
	'/System/Library/Fonts',
)
```

各个系统字体的安装这里就略过不谈了，请自行查找相关内容。

继续回到水印的实现上来，剩下最后一步就是将生成的水印文件和将要下载的 PDF 文件合并在一起，来看看具体实现：

```python
import io
from PyPDF2 import PdfFileWriter, PdfFileReader

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def binary_content(self, *args, **kwargs):
        ...

    def _get_watermark(self):
        ...

    def _generate_watermark(self):
        ...

    def add_watermark(self, content):
        """
        添加水印
        :param content:
        :return:
        """
        watermark = self._generate_watermark()
        pdf_input = PdfFileReader(io.BytesIO(content), strict=False)
        watermark = PdfFileReader(open(watermark, "rb"), strict=False)
        pdf_output = PdfFileWriter()
        page_count = pdf_input.getNumPages()
        for page_number in range(page_count):
            input_page = pdf_input.getPage(page_number)
            input_page.mergePage(watermark.getPage(0))
            pdf_output.addPage(input_page)
        stream = io.BytesIO()
        pdf_output.write(stream)
        data = stream.getvalue()
        return data
```

合并步骤很简单，就是遍历要下载的 PDF 将它的每一页都和水印文件合并起来，然后再输出即可。

上面就是全部的实现了，安装模块之后，可以在开启开发者模式后打开「设置——技术——附件」菜单，创建并上传一个 PDF 文件，然后再下载查看是否成功添加上了水印。

如果你想要直接使用这个模块或者下载完整的源码，可以前往[这里](https://github.com/ruter/Odoo-Tutorial-Demo/tree/14.0/pdf_watermark)查看或下载；如果你想查看完整的 commits 信息，可以查看[这个 PR](https://github.com/ruter/Odoo-Tutorial-Demo/pull/6) 里的内容。

### 扩展阅读

- [How to watermark your PDF files with Python.](https://medium.com/@schedulepython/how-to-watermark-your-pdf-files-with-python-f193fb26656e)

- [How to set any font in reportlab Canvas in python?](https://stackoverflow.com/questions/4899885/how-to-set-any-font-in-reportlab-canvas-in-python)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2020/12/27/Odoo-Add-Watermark-to-PDF/
