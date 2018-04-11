---
layout: wiki
title: "Jinja2高亮active状态菜单项"
date: 2017-02-23 21:51
keywords: Jinja2
---

通常我们网站的导航条选项都会有`active`属性以明显提示我们所处在哪个菜单项之下，在`Jinja2`中要实现这样的效果并不难做到。

一般情况下我们会将导航栏写入到`base.html`中，作为基础模板使用。我们把导航栏的选项和相应的信息定义成一个列表，然后通过`active_page`设置处于active状态的菜单项：

{% raw %}
```jinja2
{% set navigation_bar = [
    ('/', 'index', 'Index'),
    ('/blog/', 'blog', 'Blog'),
    ('/about/', 'about', 'About')
] -%}
{% set active_page = active_page|default('index') -%}
...
<ul id="navigation">
{% for href, id, caption in navigation_bar %}
  <li{% if id == active_page %} class="active"{% endif
  %}><a href="{{ href|e }}">{{ caption|e }}</a></li>
{% endfor %}
</ul>
...
```
{% endraw %}

这里通过遍历`navigation_bar`来为导航栏的选项加上链接以及标题，并且通过遍历出来的`id`和`active_page`的值相比较以确定当前项是否是active状态。

其中`active_page`是在我们的子模板中进行设置的，假设当前页面是`index.html`：

{% raw %}
```jinja2
{% extends "base.html" %}
{% set active_page = "index" %}
...
```
{% endraw %}

当我访问`index.html`这个页面时，`active_page`和`id`为`index`的那一项相匹配，即`Index`为active状态。