---
title: CentOS升级Python2.6到Python2.7并安装pip
date: 2015-12-03 20:19:00
categories: Server
keywords: Python, Server
description: 在 CentOS 6.x 中升级 Python 版本
---

貌似CentOS 6.X系统默认安装的Python都是2.6版本的？平时使用以及很多的库都是要求用到2.7版本或以上，所以新系统要做的第一件事必不可少就是升级Python啦！在这里做个简单的升级操作记录 :)

# 0. 依赖安装
```
yum -y update
yum install epel-release
yum install sqlite-devel
yum install -y zlib-devel.x86_64
yum install -y openssl-devel.x86_64
```

# 1. 升级Python
系统默认安装的Python是2.6.6的，我们需要升级到Python2.7，用`wget`命令从官方下载源文件，然后解压进行编译
```
wget http://www.python.org/ftp/python/2.7.10/Python-2.7.10.tar.xz
unxz Python-2.7.10.tar.xz
tar -vxf Python-2.7.10.tar
```
执行完以上命令会解压得到`Python-2.7.10`这个文件夹，进入该目录并执行以下命令进行配置
```
./configure --enable-shared --enable-loadable-sqlite-extensions --with-zlib
```
其中`--enable-loadable-sqlite-extensions`是sqlite的扩展，如果需要使用的话则带上这个选项。

之后执行
```
vi ./Modules/Setup
```
找到`#zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz`去掉注释并保存，然后进行编译和安装
```
make && make install
```

安装好`Python2.7`之后我们需要先把`Python2.6`备份起来，然后再对`yum`的配置进行修改，如果不进行这一步操作的话，执行`yum`命令将会提示你Python的版本不对。

执行以下命令，对`Python2.6`进行备份，然后为`Python2.7`创建软链接
```
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/bin/python2.7 /usr/bin/python
```
然后编辑`/usr/bin/yum`，将第一行的`#!/usr/bin/python`修改成`#!/usr/bin/python2.6.6`
现在执行`yum`命令已经不会出现之前的错误信息了。

我们执行`python -V`查看版本信息，如果出现错误
> error while loading shared libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory

编辑配置文件
```
vi /etc/ld.so.conf
```
添加新的一行内容`/usr/local/lib`，保存退出，然后
```
/sbin/ldconfig  
/sbin/ldconfig -v
```

# 2. 安装pip
下载最新版的pip，然后安装
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```
查找pip的位置
```
whereis pip
```
找到`pip2.7`的路径，为其创建软链作为系统默认的启动版本
```
ln -s /usr/local/bin/pip2.7 /usr/bin/pip
```

pip安装完毕，现在可以用它下载安装各种包了 :)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** http://ruterly.com/2015/12/03/Update-python/
