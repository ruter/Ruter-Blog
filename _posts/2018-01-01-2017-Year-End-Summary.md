---
title: 2017 年终小总结
categories: Summary
tags: 2017, summary
toc: false
date: 2018-01-01 15:39:04
description: 是时候对过去的一年做个小小的总结了
---

2017 年这就过去了，距离大学毕业就过去半年了，距离入职当前的这家公司也已经 10 个月之久了，过去这年除了忙着毕业，忙着写论文做毕设和准备毕业答辩，剩下的时间都基本花在了工作上了。以前一直没有做总结的习惯，不如现在就趁着元旦假期，针对技术做一个小小的总结吧。

<!--more-->

## 关注的技术和框架

这里就列出 6 个技术或框架，然后再分别说一说：

- [GraphQL](http://graphql.org/)
- [Odoo](https://www.odoo.com/zh_CN/)
- [React](https://reactjs.org/)
- [Vue](https://vuejs.org/)
- [V2ray](https://www.v2ray.com/)
- [微信公众号](https://mp.weixin.qq.com/)

先说说 `GraphQL` 这个查询语言，实际并没有详细地深入学习，初次接触的时候可以用一头雾水来形容了。我的个人 [Wiki](http://wiki.ruterly.com) 是用 `Simiki` 生成之后托管在 `Github` 上的，后来查了一波 Github 的 [GraphQL API](https://developer.github.com/v4/) ，就想用它配合 `Vue.js` 重构自己的个人 Wiki，于是就有了这个项目—— [Vue-wiki](https://github.com/ruter/Vue-wiki) .

然后是 `Vue` 和 `React` 这两个前端届的产物，说起 React 大概都知道之前 Facebook 因为修改了它的许可协议而引起的风波。据同事反馈，这个 React 是越写越上手，行云流水，实际我并没有在任何项目里使用过它，只是稍微了解学习过一些。再来说说 Vue，出自国人之手，实际使用之后我只想说，以后谁再让我用 `jQuery` 写页面我就跟谁急！Vue 跟 React 不同的一点是它可以直接在页面里引入使用，如果你不想去学习怎么用它，也请去官方文档看看它是怎么工作的，实在不想碰什么 `webpack` 之流的话，你还可以把它当高配版的 jQuery 来用啊！

微信公众号开发是一时兴起，做了个图片搜索的功能，用的是 `Pixabay` 的 API，项目同样托管在 Github 上—— [WeChat-img-Search](https://github.com/ruter/WeChat-img-Search) .

关于 `V2ray`，其实也不用过多介绍，要用 Google 什么的，就得用上它了，之前一直用的 SS(R) 等已经被我无情抛弃了。

最后来说说 `Odoo` 吧，这是个非常庞大，并且一直在进化的企业级应用框架，Github 上每天都有大量的 issue 和 PR，社区可以说是相当活跃了。因为它的庞大，以及历史遗留的代码，导致社区在短时间内，仍不能完整地迁移到 `Python3` 上。我是在进现在这家公司前，才知道这个框架的，因为看到 JD 里有写到，才去了解了一下，现在每天的工作就是在跟这个家伙打交道了，Odoo 的很多特性都能极大地方便扩展和开发，是一个非常优秀的框架。

## 关于博客

一年的时间，只写了少得可怜的 6 篇博客，其中有 3 篇是在工作中使用 `Odoo` 遇到的问题和解决方法，1 篇是新开坑的 API 文档翻译系列，1 篇是应用推荐系列的博客，还有一篇是一个简短的教程（没有发布在个人博客里），过去一年里留下的文字实在是太少太少了。从大学开始就想着要多写写博客，养成记录的习惯，但是一直都没有很好的去践行，罪过罪过。

## 这一年参与的项目

过去一年里，从入职当前公司开始，总共参与了 3 个公司的项目，都是企业级应用，现在回过头去看，真是觉得当初第一个项目写得那叫一个惨不忍睹。慢慢地第二、第三个项目，随着对所使用的技术框架的了解，可以写出更好的代码来，但是过一段时间回去看，也是会觉得当时写得都是些什么玩意儿。

![码云上的贡献度](/images/Summary/gitee-contr.png)

除了公司的项目，个人也做了一些小的 Side Project，2017 年创建并维护了 8 个 Github 仓库，基本上是使用一些新的技术或框架做出来的小项目，总共拿到了 27 个 stars，个人比较满意的大概就是为 Odoo 写的一些扩展了，都发布到了官方应用商店，可以在这里看到—— [Odoo Apps](https://apps.odoo.com/apps/browse?repo_maintainer_id=196663) .

![Github commits trend](/images/Summary/Commits.png)

虽然一直想为开源社区做点贡献，但是一直都没有付诸行动，这一年赶在年末，为社区做了一点小贡献，为 Odoo 这个大项目提了一个 PR 并且被合并到了 10.0 版本分支中，以及为另一个项目提了若干的 issue，虽然没有作出特别大的贡献，但是总算为社区做了一点点事情了。

![Github contributions in the last year](/images/Summary/github-contr.png)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** http://ruterly.com/2018/01/01/2017-Year-End-Summary/
