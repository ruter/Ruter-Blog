---
title: 在 Odoo 中生成唯一不重复的序列号
categories: Odoo
keywords: Odoo, ir.sequence
toc: false
date: 2017-12-08 15:06:23
description: 用 Odoo 的自带模块生成序列
---

最近在做的项目中有一个需求是要让某个字段值根据记录产生的日期和一定的组合规则按顺序生成一个序列号，这个序列号不可重复，这原本是一个很常见的需求，没有多想就写好了。到后面测试的时候才发现一个比较严重的问题，如果用户同时操作产生的记录，生成的序列号会有重复的情况。

经过讨论和思考后有几种解决方案，一是在数据库表层加锁，一是采用类似 redis 的消息队列，还有就是通过文件锁达到数据库排他锁的目的，鉴于时间和项目当前的情况，最后采用了通过文件锁实现这个需求。

其实除了以上几种方式，Odoo本身就有一个模型（`ir.sequence`）是用于生成序列的，可以很方便地实现这个需求，因为之前一直没有接触过这个模块，还是在项目之后的阶段同事使用到了并且告诉我之后才知道原来有这么个好东西的存在。在这里我将会把我原本通过文件锁实现的方式和通过 Odoo 自带的`ir.sequence`实现的方式都记录下来。

## 给文件加锁 - fcntl

`fcntl`是 Python 标准库里的一个模块，用来对文件进行加锁的操作。在实现中主要用到的是下面这个函数：

```python
def flock(fd, operation):
  """
  flock(fd, operation)

  Perform the lock operation op on file descriptor fd. See the Unix 
  manual page for flock(2) for details. (On some systems, this function is
  emulated using fcntl().)
  """
  pass
```

其中`fd`是文件描述符，`operation`为锁的操作，总共有4种：

- fcntl.LOCK_EX - 排他锁
- fcntl.LOCK_NB - 非阻塞锁
- fcntl.LOCK_SH - 共享锁
- fcntl.LOCK_UN - 解锁

关于`fcntl`的其他具体内容请查看 [官方标准库文档](https://docs.python.org/2/library/fcntl.html) 。

下面来看一下具体的实现，在给出代码之前，先描述一下需求，假设模型中有一个字段`sn`用于存储按一定规则生成的序列号，序列号的组成规则如下：

1. 固定的前缀`SN`
2. 取记录生成的日期组成的6位数字`%y%m%d`，如2017年12月8日取值为171208
3. 最后是3位的流水号，从001开始递增
4. 生成的序列号不能有重复
5. 最后的3位流水号每天自动重置，从001开始递增（这个需求涉及到一些扩展，故此文将不实现这一需求）

需求很简单，也很清楚了，下面就上代码开始具体的实现。首先创建一个模块`demo_sequence`：

```bash
./odoo-bin scaffold demo_sequence
```

然后在模块的目录下创建数据文件目录`data/`，在目录下创建一个`data.xml`文件，在后面会用到；继续在模块目录下创建静态文件目录`static/`，在目录下创建一个空文件`SN.LOCK`用作加锁的文件对象。完成之后的目录结构如下：

```bash
demo_sequence
├── __init__.py
├── __manifest__.py
├── controllers
│   ├── __init__.py
│   └── controllers.py
├── data
│   └── data.xml
├── demo
│   └── demo.xml
├── models
│   ├── __init__.py
│   └── models.py
├── security
│   └── ir.model.access.csv
├── static
│   └── SN.LOCK
└── views
├── templates.xml
└── views.xml
```

模型的创建和视图的编写，这里将跳过不说，具体的代码将在后面给出。先创建一个给文件加锁的函数：

```python
def _file_lock(flag=fcntl.LOCK_EX):
  FILE_PATH = get_module_resource('demo_sequence', 'static/SN.LOCK')
  file = open(FILE_PATH)
  fcntl.flock(file.fileno(), flag)
  _logger.info('Acquire Lock')
  return file
```

然后重写模型的`create()`方法：

```python
@api.model
def create(self, vals):
  file = _file_lock()
  sn_prefix = 'SN' + datetime.date.today().strftime("%y%m%d")
  obj = self.env['demo_sequence.fcntl'].search_read([('sn', '=like', sn_prefix + '%')], limit=1, order='sn DESC')
  # 今天已经有序列号，在最新的序列号上递增
  if obj and obj[0]['sn'].startswith(sn_prefix):
  sn_suffix = int(obj[0]['sn'][-3:]) + 1
  vals['sn'] = sn_prefix + str(sn_suffix).zfill(3) # 补0
  else:
  vals['sn'] = sn_prefix + '001'

  res = super(DemoSequence, self).create(vals)
  # 关闭文件将自动解锁
  file.close()
  return res
```

利用`fcntl`给文件加锁后再生成序列号写入数据库，以达到序列号不重复的目的，就这么点代码就搞定了，不过还有更简单的方式，就是利用 Odoo 自带的`ir.sequence`模型产生序列号。

## 生成唯一标识 - ir.sequence

在模型`ir.sequence`中是这样描述的：

> The sequence model allows to define and use so-called sequence objects. Such objects are used to generate unique identifiers in a transaction-safeway.

我们可以利用它生成唯一的标识，下面就看一下怎么用`ir.sequence`实现前面所说的需求。

打开`data/data.xml`并添加以下代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <data noupdate="1">
    <record id="seq_demo_sequence_sn" model="ir.sequence">
      <field name="name">Demo Sequence SN</field>
      <field name="code">demo_sequence.sequence</field>
      <field name="prefix">SN%(y)s%(month)s%(day)s</field>
      <field name="padding">3</field>
    </record>
  </data>
</odoo>
```

这里的参数实际上不止这几个，在实现需求的前提下这几个就够用了，分别说明一下各个参数的作用：

- name - 名字，随便叫什么都行
- code - 调用生成编码的 Key，需保证唯一性
- prefix - 前缀，可以是固定的字面量也可以是组合参数
- padding - 序列递增的位数

注：记得将`data/data.xml`加入到`__manifest__.py`的`data`列表中

接下来就是调用得到按规则生成的序列号，同样重写模型的`create()`方法：

```python
@api.model
def create(self, vals):
  vals['sn'] = self.env['ir.sequence'].next_by_code('demo_sequence.sequence')
  return super(DemoSequence2, self).create(vals)
```

可以看到只需要一行代码就可以得到一个唯一的序列号，比前面用`fcntl`给文件加锁的方式简单了几个级别。这里的调用就用到了前面定义中所写的`code`。

在实际项目中所使用到的两种方式都已经在这里记录下来了，官方的东西确实是个好东西，回头看看自己写的东西，毕竟 too young，可惜官方的文档好像并没有相关的记录（抑或是我没找到？），多翻翻官方实现的功能模块源码，才是精进 Odoo 之道。

## 源码下载

以上出现的所有代码均可在仓库 [ruter/TNK-Odoo-Demo](https://github.com/ruter/TNK-Odoo-Demo) 中 [查看并下载](https://github.com/ruter/TNK-Odoo-Demo/tree/10.0/demo_sequence) 。

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** http://ruterly.com/2017/12/08/Generate-Unique-Sequence-NO-in-Odoo/
