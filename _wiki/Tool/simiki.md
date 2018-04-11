---
layout: wiki
title: "Simiki"
date: 2016-11-01 10:12
update: 2017-02-25 10:10
keywords: 模板, wiki
categories: Tool
---

# 页面头信息

wiki页面的元数据使用的是YAML Front Matter格式，常用以下头作为模板

```
---
title: "页面头信息"
date: 2016-11-1 00:00
updated: 2016-11-1 10:00
collection: Document
tag: wiki, template
---
```

# 生成目录
在YAML Front Matter下面, wiki内容的顶部，添加一行

```
---
title: "生成目录"
date: 2016-11-1 00:00
updated: 2016-11-1 10:00
collection: Document
tag: wiki, template
---

[TOC]

<content>
```

# Favicon
将文件名为`favicon.ico`的icon文件放在wiki根目录下，在主题模板添加

```
<link rel="shortcut icon" href="{{ site.root }}/favicon.ico" type="image/x-icon">
<link rel="icon" href="{{ site.root }}/favicon.ico" type="image/x-icon">
```
# 更换电脑后使用

在Github的[issue#23](https://github.com/tankywoo/simiki/issues/23)中有相关讨论，下面引用作者的其中一条[回复](https://github.com/tankywoo/simiki/issues/23#issuecomment-62398867):

> 以github pages with domain 为例，master分支保存源文件、gh-pages分支保存生成的output内容。换一台电脑clone:
>
> ```bash
> $ git clone -b master git@github.com:username/projectname.git
> $ cd projectname/
> $ git clone -b gh-pages git@github.com:username/projectname.git output
> ```
>
> 就可以了。