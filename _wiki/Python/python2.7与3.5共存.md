---
layout: wiki
title: "Python2.7与3.5共存"
date: 2016-11-17 00:19
keywords: Python
categories: Python
---

假定系统已安装`Python2.7`，现要安装3.5版本

## 先安装依赖

```
yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
```

## 下载源码并解压

```
wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz

tar -zxvf Python-3.5.2.tgz
```

## 安装
使用`make altinstall`而不是`make install`安装

```
./configure --prefix=/usr/local
make
make altinstall
```

查看版本`python3.5 -V`

```
Python 3.5.2
```

用`whereis pip`可以看到`pip3.5`也一并被安装好了

```
pip: /usr/bin/pip2.7 /usr/bin/pip /usr/local/bin/pip3.5
```