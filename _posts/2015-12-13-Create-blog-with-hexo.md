---
title: 用Hexo创建博客
date: 2015-12-13 16:04:26
toc: true
categories: Tutorial
keywords: Blog, Hexo
description: 静态博客生成器中的优等生——Hexo
---

啊~有个好点子呢，想要记下来分享给别人看呢，不如做个博客吧！看看要什么...服务器、域名、博客程序blahblahblah...一点都不想弄了好吗！就没有简单的方法吗，不过是想好好地写点东西就那么难咩[怨念脸]~看我发现了什么，一起来用`Hexo`建一个博客吧！
# 安装必要程序
我们这里用Windows机器进行演示，其他系统操作类似，可以查看官方文档～
## Git
我们很经常会要用到`Git`以及`Github`，首先来安装Git：

- [下载Git](https://git-scm.com/downloads)

选择合适的版本下载然后安装，我们稍后将会用上。

## Node.js
因为Hexo是一款基于`Node.js`的静态博客框架，所以我们还需要安装好它：

- [下载Node.js](https://nodejs.org/en/download/)

同样选择对应自己电脑的版本下载安装。

## Hexo
安装好Git和Node.js之后，轮到我们今天的主角登场了~先打开`Git Bash`，然后输入下面这条命令来安装Hexo：
```
npm install hexo-cli -g
```

咦？是不是发现没反应了呢？不要急，稍微等等就会出现下面这些安装信息了：
```
npm WARN optional dep failed, continuing fsevents@1.0.6
C:\Users\Administrator\AppData\Roaming\npm\hexo -> C:\Users\Administrator\AppData\Roaming\npm\node_modules\hexo-cli\bin\hexo
hexo-cli@0.1.9 C:\Users\Administrator\AppData\Roaming\npm\node_modules\hexo-cli
├── abbrev@1.0.7
├── minimist@1.2.0
├── bluebird@3.0.6
├── tildify@1.1.2 (os-homedir@1.0.1)
├── chalk@1.1.1 (supports-color@2.0.0, ansi-styles@2.1.0, escape-string-regexp@1.0.3, strip-ansi@3.0.0, has-ansi@2.0.0)
└── hexo-fs@0.1.5 (escape-string-regexp@1.0.3, graceful-fs@4.1.2, chokidar@1.4.1)
```

来试试安装成功没有，输入以下命令查看Hexo的版本信息：
```
hexo -v
```

如果出现类似内容说明安装成功啦！
```
hexo-cli: 0.1.9
os: Windows_NT 6.1.7601 win32 ia32
http_parser: 2.3
node: 0.12.2
v8: 3.28.73
uv: 1.4.2-node1
zlib: 1.2.8
modules: 14
openssl: 1.0.1m
```


# Hexo的使用
非常好！现在我们可以开始创建博客写文章了！是不是很快~下面会介绍几个Hexo的常用命令，掌握之后就能愉悦地开始写作之旅了XD
## 新建博客
假设我在`E`盘有个叫`blog`的文件夹，现在我们用Git Bash进去这个文件夹里，然后将它初始化为我们的博客目录：
```
cd e:blog

hexo init
```

安装成功的提示信息：
```
INFO  Copying data to E:\blog
INFO  You are almost done! Don't forget to run 'npm install' before you start blogging with Hexo!
```

注意看到第二条！提示我们在开始使用前要先执行
```
npm install
```

这条命令是用来安装依赖包的，具体安装内容可以在`package.json`文件里找到。安装好了之后会看到一大串的信息，这里就不贴出来了。现在我们可以看到blog目录下的文件结构是这样的：
```
.
├── _config.yml
├── node_modules
├── package.json
├── scaffolds
├── source
|   └── _posts
└── themes
```

## 写新文章
弄了那么久什么时候才能开始写东西嘛！好好好~现在就开始写第一篇文章吧 :)
执行下面这条命令来创建一篇新文章「Hello」：
```
hexo new "Hello"
```

然后会提示我们新建的文章所在路径：
```
INFO  Created: E:\blog\source\_posts\Hello.md
```

我们找到这个文件之后用Markdown编辑器或其他文本编辑器打开，我们会看到里面已经有内容了：
```
title: Hello
date: 2015-12-13 15:04:33
tags:
---
```

这是由模板生成的`Front-Matter`，在`---`前面的是文章的一些基本信息例如标题、日期及标签，还能添加其他一些选项如分类，我们后面有机会再来说说。

我们的文章内容要写在`---`下面，随便写点什么来看看效果吧：
```
# 你好
这是我的第一篇文章:)
```

保存之后就可以进入下一步啦~

## 生成页面
文章写好了怎么看效果呢？只需要下面这一条命令就可以将我们写的Markdown文件 (`.md`后缀) 转换成`.html`静态页面了：
```
hexo generate
```

然后就会看到一串类似下面这样的信息：
```
INFO  Files loaded in 2.37 s
INFO  Generated: js/script.js
INFO  Generated: fancybox/jquery.fancybox.pack.js
INFO  Generated: fancybox/jquery.fancybox.js
INFO  Generated: fancybox/jquery.fancybox.css
INFO  Generated: fancybox/helpers/jquery.fancybox-thumbs.js
INFO  Generated: fancybox/helpers/jquery.fancybox-thumbs.css
INFO  Generated: fancybox/helpers/jquery.fancybox-media.js
INFO  Generated: fancybox/helpers/jquery.fancybox-buttons.js
INFO  Generated: fancybox/helpers/jquery.fancybox-buttons.css
INFO  Generated: fancybox/helpers/fancybox_buttons.png
INFO  Generated: fancybox/fancybox_sprite@2x.png
INFO  Generated: fancybox/fancybox_sprite.png
INFO  Generated: fancybox/fancybox_overlay.png
INFO  Generated: fancybox/fancybox_loading@2x.gif
INFO  Generated: fancybox/fancybox_loading.gif
INFO  Generated: fancybox/blank.gif
INFO  Generated: css/style.css
INFO  Generated: css/images/banner.jpg
INFO  Generated: css/fonts/fontawesome-webfont.woff
INFO  Generated: css/fonts/fontawesome-webfont.ttf
INFO  Generated: css/fonts/fontawesome-webfont.svg
INFO  Generated: css/fonts/fontawesome-webfont.eot
INFO  Generated: css/fonts/FontAwesome.otf
INFO  Generated: 2015/12/13/Hello/index.html
INFO  Generated: 2015/12/13/hello-world/index.html
INFO  Generated: archives/index.html
INFO  Generated: archives/2015/index.html
INFO  Generated: archives/2015/12/index.html
INFO  Generated: index.html
INFO  29 files generated in 2.19 s
```

怎么样，我们的博客已经生成完毕啦，so easy！

## 页面预览
是不是迫不及待想要看到实际效果了，现在还差一步就可以看到庐山真面目了~我们需要在本地生成一个小型服务器，把博客挂载上去，才能访问。什么？！那要多麻烦！不不不~Hexo已经帮我们都做了这些工作了，我们要做的只是执行下面这条命令而已：
```
hexo server
```

我们会看到这样一条提示：
```
INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
```

现在打开浏览器访问[http://localhost:4000/](http://localhost:4000/)试试吧！

PS: 可能会弹出防火墙的提示，放行就OK了~

## 配置信息
是不是很激动！成功了有木有~不过等等，博客名写着Hexo，可不是我想的那样噢，还有还有，页面底部的版权信息里作者名字也不是我啊喂！来，我们来改掉它们！

找到博客根目录下的配置文件`_config.yml`，用自己喜欢的文本编辑器编辑它。看到`# Site`的那一部分，里面的`title`就是博客的名字，`subtitle`就是副标题，`author`对应的那个就是作者名字啦，把自己的大名写上去吧！

还有很多配置项可以修改，这里就不详细讲了，可以查看[Hexo官方文档](https://hexo.io/docs/configuration.html)对照着修改配置。

## 更换主题
在这个看脸的世界，我们的博客怎么可以没有颜值呢，折腾完上面的那些之后，是时候给博客挑上件漂亮的“衣服”了。

Hexo官方网站上展示了25套主题，可以按自己的口味挑选，或者动手能力强的，可以试着自己写一套独一无二的主题噢XD

这里以[Apollo](https://github.com/pinggod/hexo-theme-apollo)这个主题为例，进去github页面之后可以看到很详细的安装和启动方法，这里就不展开来讲了。


# 部署
我们创建好博客之后，要怎么放到互联网上给世界友人访问到呢？

由于篇幅限制，关于部署我会单独写一篇博客详细地说说如何利用`Github Pages`部署我们的博客(拖延症什么的才不是呢！)，蟹蟹阅读XD

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://blog.ruterly.com/)」原创。转载务必注明[出处](http://blog.ruterly.com/2015/12/13/Create-blog-with-hexo/)。

本文永久地址：http://blog.ruterly.com/2015/12/13/Create-blog-with-hexo/
