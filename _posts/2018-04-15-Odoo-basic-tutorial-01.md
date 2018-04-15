---
layout: post
title: 「Odoo 基础教程系列」第一篇——环境准备
categories: [Odoo, Tutorial]
description: Odoo 基础教程系列之开发环境的准备
keywords: Odoo, Tutorial, 基础教程
---

这是「Odoo 基础教程系列」的第一篇，在开始教程的正文之前，这里要先跟大家知会一声，在这篇教程以及后面的教程中，都将默认开发环境是 `macOS` 或 `Ubuntu`，因为在 `Windows` 中开发总是能遇到未知的坑，处理起来往往很是耗费时间精力，这里不去争论系统的优劣，如果没有 `Windows` 外的其他系统的小伙伴，但是又想学习本教程的内容，推荐你们使用虚拟机安装一个 `Ubuntu` 作为开发环境 :)

OK，下面正式开始本系列教程的第一篇内容。

# 基本准备

在开始干活前，首先要把工具给准备好，那我们进行 `Odoo` 开发，需要有哪些工具呢？下面就把最最基础的罗列出来：

- `Python 3.5+`
- `PostgreSQL`
- `Node.js`
- `LESS`
- `Git`

## 安装 Python3

首先当然少不了 Python 啦，在 Odoo 10.0 版本之前，使用的都是 Python 2，在 11.0 开始，官方就推荐使用 Python 3.5+，当然使用 Python 3 也是大势所趋，大胆地切换到 3 吧！

如果你当前使用的系统没有安装 Python 3.5+，你可以按照下面的步骤轻松完成安装：

```shell
# 如果你使用的是 Mac
brew install python3
# Ubuntu 用户
sudo apt-get update
sudo apt-get install python3
```

## 安装 PostgreSQL

然后是强大的数据库 `PostgreSQL`，这个数据库是 Odoo 官方钦定使用的，不能像 `Django` 那样可以根据喜好用各种库。Mac 用户可以选择通过[官网](http://postgresapp.com/)下载 APP 并跟随步骤进行安装或者使用 `Homebrew` 进行安装并启动：

```shell
brew install postgresql
brew services start postgresql
```

如果你是 Ubuntu 用户，则可以通过以下命令进行安装：

```shell
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

然后执行 `psql -V` 查看版本号，如果没有错误并且输出了版本号，则说明已经成功安装并启动，如果出现以下错误信息：

```
psql: could not connect to server: No such file or directory
Is the server running locally and accepting
connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

可以参考 stackoverflow 上的[这个回答](https://stackoverflow.com/a/30841396)进行除错。

安装完毕之后，我们先创建一个数据库的超级用户，在之后运行 Odoo 初始化数据库的时候将会用上：

```shell
sudo createuser --superuser username
```

## 安装 Node.js 和 LESS

Odoo 中有很多的样式文件是使用 `LESS` 写的，在运行的时候 Odoo 会将这些样式文件转换成 CSS 文件，具体细节我们不必深究。我们需要先安装好 `Node.js` 后才能安装 `LESS`，继续跟着步骤把它们安装好。

Ubuntu 13.10 及之前的版本需要手动安装 Node.js：

```shell
wget -qO- https://deb.nodesource.com/setup | bash -
sudo apt-get install -y nodejs
```

Mac用户直接使用 Homebrew 进行安装：

```shell
brew install node
```

安装好 Node.js 之后接着安装 LESS：

```shell
sudo npm install -g less
```

就是这么简单，我们已经把 Node.js 和 LESS 都安装好了，如果感兴趣的话，可以去官网了解一下：

- [Node.js 官网](https://nodejs.org/zh-cn/)
- [LESS 官网](http://lesscss.org/)

## 安装 Git

`Git` 是个非常好用的版本管理工具，日常工作已经离不开它了 :) 在大部分系统中都已经预装好了，如果不知道当前系统中有没有预装 git，不妨在终端里输入 `git --version` 查看一下，如果提示未找到命令，则需要手动安装，过程同样十分简单：

```shell
# Ubuntu
sudo apt-get update
sudo apt-get install git
# Mac
brew install git
```

完美！到这里我们已经把目前所需要的工具都准备好了，这才只是个开始，不过别担心，接下来也不会很难。

# 依赖管理器

熟悉 `Python` 开发的小伙伴应该都了解或者使用过各种虚拟环境，如 `virtualenv` 和 `virtualenvwrapper` 等，在这系列的教程中，我们将用 `Pipenv` 来创建和管理项目的虚拟环境和依赖包。

如果你还没了解过 `Pipenv`，那下面就来简单了解一下。`Pipenv` 是 Python 项目的依赖管理器，类似于 `Node.js` 的 `npm` 和 `yarn` 等。它会自动为项目创建和管理一个虚拟环境，当你添加或删除包时，它也会自动地在 `Pipfile` 中添加或删除对应的包信息。

在终端执行以下命令进行安装：

```shell
pip install --user pipenv
```

选项 `--user` 表示安装在用户目录下，这样做可以防止破坏任何系统范围的包。安装好后，在 Shell 中执行 `pipenv --version` 查看版本信息，应该会看到输出如下信息（版本号可能会不同）：

```shell
pipenv, version 10.0.0
```

如果提示未找到 pipenv，则需要手动将用户基础目录下的 `/bin` 添加到 `PATH` 中。使用 `python -m site --user-base` 找到用户基础目录的路径，然后执行：

```shell
export PATH="/Users/name/Library/Python/3.6/bin:$PATH"
```

记得将 `/bin:$PATH` 前的路径替换为你找到的路径。

注：如果你使用的是 `Zsh`，编辑 `~/.zshrc` 并将 `export PATH=/Users/name/Library/Python/3.6/bin:$PATH` 添加到文件底部，不要忘了替换路径 :)

再次执行 `pipenv --version` ，此时应该可以成功查看到版本信息了。

# 获取 Odoo 源码

`Odoo` 有两个版本，一个是社区版，一个是企业版，社区版是开源免费的，企业版当然是相应需要收费的，两个版本会有一些差异，但是这些问题不大，我们拉取官方仓库的社区版源码：

git clone [https://github.com/odoo/odoo.git](https://github.com/odoo/odoo.git) -b 11.0 --depth=1

因为众所周知的网络问题可能速度会比较慢，耐心等待拉取完成后，进入源码目录 `cd odoo`，可以看到第一层目录结构如下：

```shell
$ tree -L 1
.
├── CONTRIBUTING.md
├── COPYRIGHT
├── LICENSE
├── MANIFEST.in
├── Makefile
├── README.md
├── addons
├── debian
├── doc
├── odoo
├── odoo-bin
├── requirements.txt
├── setup
├── setup.cfg
└── setup.py
```

很好，已经万事俱备了，还差一点我们就可以成功运行 Odoo 了，事不宜迟，继续我们的教程。

# 安装依赖

编辑文件 `requirements.txt`，删除最后一行的 `pypiwin32 ; sys_platform == 'win32'` 并保存（如果你的开发环境是 Windows，你应该删除的是带有 `sys_platform != 'win32'` 的行），然后创建虚拟环境（使用 Python 3）并安装相关依赖：

```shell
pipenv install --three
```

选项 `--three` 用于指定初始化环境所使用的 `Python` 的版本为 Python3，如果你在其他项目中需要使用 Python2 初始化环境，可以将该选项指定为 `--two`。

因为目录下存在文件 `requirements.txt`，故 `pipenv` 会将其转换成 `Pipfile`，并且会自动安装里面列出的所有依赖。在经过可能有一点点漫长的等待之后，终于安装完毕啦！此时应该会看到这样的提示信息：

```
To activate this project's virtualenv, run the following:
$ pipenv shell
```

来试试看执行 `pipenv shell` 激活虚拟环境，然后运行 `python`，尝试导入几个包看看依赖包是否已经成功安装：

```python
>>> import jinja2
>>> import lxml
>>> import qrcode
```

如果没有报错，就说明我们的环境已经准备好了，接下来我们要尝试运行 Odoo 的服务了，紧不紧张，刺不刺激，继续下一步吧！

# 运行 Odoo

先创建一个数据库，用于 Odoo 初始化数据使用：

```shell
createdb demo -U username
```

还记得前面我们创建了一个超级用户吗？请不要忘记把命令里的 `username` 替换成你创建的那个用户名哦 XD

在 Odoo 根目录下执行以下命令运行 Odoo 服务：

```shell
./odoo-bin --addons-path=addons --db-filter=^demo$ -d demo
```

这里解释一下上面这条命令里各个选项的意义：

- `--addons-path` 指定要加载的模块目录
- `--db-filter` 用于强制指定使用的数据库，支持正则表达式，这个选项的具体用途我们在后面涉及到的话会再次讲解，现在不理解也没关系
- `-d` 用于指定使用的数据库

命令里的 `demo` 就是我们前面创建的数据库的名称，如果你不是用的这个名称，记得替换成你创建的数据库名。

在一个新的数据库中初次运行 Odoo，它将会进行初始化，这里需要稍微等待一小会儿，当你看到以下字样时，就说明 Odoo 的服务已经成功运行并可以访问了：

```
2018-02-28 12:34:51,398 1561 INFO demo odoo.modules.loading: Modules loaded.
```

现在可以打开浏览器，访问 [`http://localhost:8069/`](http://localhost:8069/) 这个地址，如果可以看到登录界面，恭喜你，你已经成功搭建好了运行环境了！

![Odoo Login Page](/images/Odoo/Odoo-Login.png)

输入帐号和密码（默认是 `admin`）登录，就会跳转到后台界面，现在暂时什么都没有，默认打开的是 Apps 应用安装模块的页面，这里看到的这些都是 Odoo 自带的一些基础应用，你可以随便装几个应用，如 `Blogs` 和 `Live Chat` 等试试看 :)

![Odoo Apps](/images/Odoo/Odoo-Apps.png)

# 瞎说几句

这个系列第一篇正式的教程，就到这里了，如果遇到什么问题尽管在评论中提出来，我会尽量协助大家解决的。碍于 996 的上班时间，接下来的教程更新可能会有点慢，希望各位小伙伴可以耐心等待 :)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/04/15/Odoo-basic-tutorial-01/)。

本文永久地址：https://ruterly.com/2018/04/15/Odoo-basic-tutorial-01/
