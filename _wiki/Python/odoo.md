---
layout: wiki
title: "Odoo"
date: 2017-01-24 12:32
updated: 2017-02-06 16:20
keywords: Odoo
categories: Python
---

# 注意事项

1. `Odoo`不支持`Python3`
2. 不能使用`root`身份开启odoo服务



# 安装Odoo

## 安装基础依赖

```bash
# 安装系统更新
$ sudo apt-get update && sudo apt-get upgrade
# 安装Git和npm
$ sudo apt-get install git
$ sudo apt-get install npm
# 创建nodejs软链
$ sudo ln -s /usr/bin/nodejs /usr/bin/node
# 安装less
$ sudo npm install -g less less-plugin-clean-css
```

## 从源码安装Odoo

```bash
# 创建工作目录
$ mkdir ~/odoo-dev
$ cd ~/odoo-dev
# 下载Odoo源码
$ git clone https://github.com/odoo/odoo.git -b 10.0 --depth=1
# 安装Odoo的系统依赖
$ ./odoo/setup/setup_dev.py setup_deps
# 安装PostgreSQL
$ ./odoo/setup/setup_dev.py setup_pg
```

## Tip

启动Odoo服务器实例用命令:

```bash
$ ~/odoo-dev/odoo/odoo-bin
```

Odoo默认监听`8069`端口，启动实例后可通过`http://<server-address>:8069`访问。



# 初始化新的Odoo数据库

在创建数据库前需保证当前用户是`PostgreSQL`的超级用户:

```bash
$ sudo createuser --superuser $(whoami)
```

如果出现`createuser: could not connect to database postgres: FATAL: role "root" does not exist`这个错误的话，使用如下命令解决:

```bash
$ sudo -u postgres -i
$ createuser --superuser username
```

创建新数据库用`createdb`命令:

```bash
$ createdb demo
```

将数据库初始化为Odoo的数据模式，需要使用`-d`选项:

```bash
$ ~/odoo-dev/odoo/odoo-bin -d demo
```

**注意**: 初始化数据库默认会添加演示数据，如果不需要，只需添加`--without-demo-data=all`即可。

## 数据库管理

除了可以创建空的数据库外，还能从现有的数据库进行拷贝，以方便测试等使用。在开始之前需要保证关闭了源数据库的所有连接，然后执行以下命令:

```bash
$ createdb --template=demo demo-test
```

列出现有数据库，可以用如下命令:

```bash
$ psql -l
```

删除数据库使用:

```bash
$ dropdb demo-test
```



# Odoo配置选项

## 配置文件

Odoo的大部分配置选项都会存储在配置文件里，默认使用的是`.odoorc`，位于`home`下。

初次安装Odoo，`.odoorc`配置文件是不会自动生成的，如果它不存在，我们应该用`--save`选项创建默认配置文件，保存最近一次运行实例的配置:

```bash
$ ~/odoo-dev/odoo/odoo-bin --save --stop-after-init
```

其中`--stop-after-init`选项会在完成所有动作后自动停止服务。一般用于运行测试或检查模块是否正确安装。

除了使用默认配置，我们还可以用`--conf=<filepath>`选项指定配置文件。

## 改变监听端口

使用`--xmlrpc-port=<port>`命令选项可以更改Odoo服务监听的端口。改变监听端口可用于运行多个不同的Odoo实例，不同的实例可以使用相同的或不同的数据库。

## 数据库过滤选项

使用数据库过滤选项可以保证Odoo服务只请求我们需要的数据库，以防止潜在的错误。

使用`--db-filter`选项以过滤数据库，它接受正则表达式，完全匹配数据库`demo`使用如下命令:

```bash
$ ~/odoo-dev/odoo/odoo-bin --db-filter=^demo$
```

## 管理日志

`--log-level`选项允许设置日志的级别，例如debug级别，可以使用`--log-level=debug`选项。

日志输出默认是标准输出，即输出到控制台，可以使用`--logfile=<filepath>`选项指定输出到文件中。



# 安装第三方模块

## 社区的模块

Odoo有App Store，地址是[apps.odoo.com](apps.odoo.com)。除了官方商店，还有很多社区贡献和维护的模块，可以在GitHub仓库[https://github.com/OCA/](https://github.com/OCA/)获取。

安装模块到Odoo，只需要将其拷贝到`addons`目录下即可，不过对于使用版本管理系统来管理代码而言，这种做法并不推荐。

我们可以为自定义的模块指定不同的目录，以和官方的区分开。

## 配置路径

以一个例子作为说明，先从GitHub下载代码，然后配置Odoo的`addons_path`让Odoo可以查找到自定义的模块:

```bash
$ cd ~/odoo-dev
$ git clone https://github.com/dreispt/todo_app.git -b 10.0
```

现在`/todo_app`和`/odoo`都在同一目录下，前者包含了我们需要的自定义模块。

我们可以提供一个包含模块的目录路径的列表，让Odoo可以从这些路径里找到所需的模块。

按如下命令启动Odoo，指定`addons_path`:

```bash
$ cd ~/odoo-dev/odoo
$ ./odoo-bin -d demo --addons-path="../todo_app,./addons"
```

指定`addons_path`后可以在命令中加入`--save`选项，将命令中的选项保存到配置文件中，下一次开启服务直接运行`./odoo-bin`，将会直接使用最后一次保存的选项。

## 更新应用列表

在新添加的模块可以被安装前，我们还需要让Odoo更新它的模块列表。

首先需要打开开发者模式(developer mode)，接着点击**Update Apps List**菜单项。

在更新模块列表后，我们可以通过应用列表或搜索`todo`查找到我们刚安装的模块。



# 创建模块的基础框架

Odoo的`scaffold`命令可以自动创建一个包含基础结构的目录，输入以下命令查看详细说明:

```bash
$ ~/odoo-dev/odoo/odoo-bin scaffold --help
```

一个Odoo扩展模块就是一个包含`__manifest__.py`描述符文件的目录。

同时为了可以被Python引用，还需要有`__init__.py`文件。

`__manifest__.py`文件中只包含一个Python字典，里面有很多属性，其中只有属性`name`是必需的。

以下是一个`__manifest__.py`文件的内容:

```python
{
  'name': 'To-Do Application',
  'description': 'Manage your personal To-Do tasks.',
  'author': 'Daniel Reis',
  'depends': ['base'],
  'application': True,
}
```

其中`depends`属性可以是一组需要的模块，当这个模块被安装的时候Odoo会自动将列出的依赖模块安装好。这个属性是非强制的，不过建议总是包含它，如果没有所需的依赖，就填入核心模块`base`。

**注意**：请确保所需依赖都分列在`depends`属性中，否则将会导致安装失败或加载错误。

除了上述属性外，还建议加入以下属性:

- `summary`作为模块的副标题显示
- `version`版本号，默认是`1.0`，遵循`semantic versioning rules`
- `license`，默认是`LGPL-3`
- `website`
- `category`模块的功能分类，默认是`Uncategorized`

还有其他可以使用的属性:

- `installable`属性默认为`True`，当为`False`时禁用模块
- `auto_install`如果为`True`，则表示该模块及其依赖将会被自动安装

## 关于license

大部分Odoo模块都是使用`GNU Lesser General Public License (LGPL)`和`Affero General Public License (AGPL)`。前者允许商用并且无需公布相关源码，后者则是更加严格的开源许可，要求公开源码。

## 安装新模块

要安装新增的模块，首先需要开启**开发者模式**，然后选择**Update Apps List**手动更新列表，然后在搜索框输入新增的模块或app的名字即可查找到，点击Install进行安装。

**注意**：如果有多个数据库，请确保使用了正确的那一个，在后台页面的用户名旁的小括号内可以看到当前使用的数据库的名称。手动强制使用对应数据库的一个方法就是加入`--db-filter=^MYDB$`选项。



# 升级模块

当数据文档或代码有变化，需要应用更改时，就需要重启odoo的服务，以`todo_app`为例可以这样对模块进行升级:

```bash
$ ./odoo-bin -d todo -u todo_app
```

其中`-u`选项(或`--update`)需要`-d`选项指定使用的数据库，并且可以接受一组以逗号分隔的模块，进行批量升级。例如`-u todo_app,mail`。

**注意**：升级模块列表以及卸载模块，目前均不支持在命令行中进行操作，它们都只能通过网页在`Apps`菜单进行操作。



# 服务器开发模式

在开启Odoo服务器实例时加上`--dev=all`选项可以开启开发模式。

开发者模式为开发提供了一些便利的特性，最重要的两点是：

- 只要有Python文件被保存，就会自动重新加载Python代码，免去了手动重启服务器的步骤
- 直接从`XML`文件读取视图的定义，避免了手动更新模块

`--dev`选项可以接受一组以逗号分隔的选项，`all`适用于大部分时候。