---
title: 将博客部署到GitHub Pages上
date: 2016-01-19 13:20:35
toc: true
keywords: GitHub Pages, Hexo, Blog
categories: Tutorial
description: 自己服务器里搭建和管理博客太麻烦，不如用 GitHub Pages 吧
---

咳咳！之前在[用Hexo创建博客](http://ruter.github.io/2015/12/13/Create-blog-with-hexo/)这篇博文末尾说过要写一篇部署教程的，结果还是由于种种原因（什么拖延啊、期末啦还有各种blahblahblah的）拖到今天才开始填坑~那就开始进入正题吧！

# 准备篇
要把创建的博客部署到网上，首先需要有一个`Github`账户，然后还需要...好像不需要什么了XD好吧，你还需要这篇教程！

打开[Github](https://github.com/)，我们需要注册一个名字高大上的账户，强调一点就是，这个用户名就是你博客的三级域名的名字噢，所以记得注册个有特色的容易记住的让人一看就知道是你的ID！

![Github首页](/images/github-pages/reg.PNG)

看到首页就有注册的选项了，填写完毕点击「Sign up for Github」提交就OK了，完了之后你可以先去个人页面设置头像和别的详细内容，这里就从略了。

# 创建仓库
现在开始这篇教程的核心部分——创建仓库。

![创建仓库](/images/github-pages/new.PNG)

打开[Create a new repository](https://github.com/new)页面或者点击网页右上角的「+」下拉菜单 ->「New repository」进行创建。

![创建新仓库页面](/images/github-pages/repository.PNG)

看到图上的画面，其中「Owner」是这个仓库的所有者，我们默认是自己的，不用改。然后看到「Repository name」，这个是仓库的名字，因为我们要创建的是`Github Pages`用于部署博客，所以这里的填写有特殊的技巧，稍后再讲。再接下来就是「Description」，关于这个仓库的一些描述，这里是可选的，你可以填上对你要部署的博客的一些简短的描述。还有要注意的一点就是「Public」和「Private」，默认保持「Public」即可，至于「Private」，大可点一下试试 :)这里只作博客部署的教学，对于`Github`的更详细的内容如果想深入了解，请不要怕麻烦，动动手自行搜索学习，总会有收获的w

关于「Repository name」的填写，格式是`username.github.io`，其中`username`是你的用户名，即「Owner」默认的那个账户的名字，如图示我应该填写的应该是`ruter.github.io`这样的。填写好了之后会提示没有错误的话，在输入框的旁边会有个绿色的勾勾，然后就可以点击「Create repository」提交啦！

# 提交部署
创建好仓库之后，剩下的就是部署文件了。打开`Git bash`，然后输入下面的命令，记得替换`username`为你的用户名，下同

```
git clone https://github.com/username/username.github.io
```

这会在你的本地建立一个仓库，这个仓库是用于存放博客文件的。打开之前创建好的博客，将`public/`目录下的文件拷贝到`username.github.io`这个目录下，然后执行以下命令

```
cd username.github.io

git add -A

git commit -m "Add blog file."

git push -u origin master
```

然后就OK了！是不是超简单！有木有学会！现在只需要访问`http://username.github.io`就可以看到博客啦！

# 博客更新
更新博客需要配合`Hexo`产生的静态文件，然后拷贝`public/`目录下的文件到`username.github.io`，之后的操作和上面一样，`push`可以去掉`-u`选项，即

```
git push origin master
```

# 进阶
其实`Github pages`还有更高级的玩法噢，不过这里就交给各位去自行探索吧！例如自定义域名还有404页面什么的~可以参考下官方的文档自行实践 :)

- [About custom domains for GitHub Pages sites](https://help.github.com/articles/about-custom-domains-for-github-pages-sites/)
- [Setting up a custom domain with GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/)
- [Further reading on GitHub Pages](https://help.github.com/articles/further-reading-on-github-pages/)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://ruterly.com/)」原创。转载务必注明[出处](http://ruterly.com/2016/01/19/GitHub-Pages-Site/)。

本文永久地址：http://ruterly.com/2016/01/19/GitHub-Pages-Site/
