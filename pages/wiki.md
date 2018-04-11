---
layout: page
title: Wiki
description: 人越学越觉得自己无知
keywords: 维基, Wiki
comments: false
menu: Wiki
permalink: /wiki/
---

> 好记性不如烂笔头，总是记不住不如写下来

{% assign sorted_wiki = site.wiki | sort: "categories" %}

<ul class="listing">
{% for wiki in sorted_wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
