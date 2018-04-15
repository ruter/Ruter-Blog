---
layout: post
title: 「玩物志」来自 Jigsaw 的 Outline
categories: [玩物志, Tutorial]
description: 字母表公司的孵化器 Jigsaw 做的代理软件，不来体验一下吗？
keywords: 玩物志, Outline
---

前段时间，来自谷歌（Google）母公司 `Alphabet` 旗下的科技孵化器 [Jigsaw](https://jigsaw.google.com/) 开发了一款名为「Outline」的代理软件，我们今天就来体验一下谷歌同门开发的这款应用，看看它有没有给我们带来什么惊喜🎉

# 关于 Outline

其实 Outline 并不是一个应用软件，而是由服务端（Outline Manager）和客户端（Outline clients）配套组合而成，不过通常 Outline 指的就是客户端，只不过客户端需要配合服务端一起才能使用。事实上如果你只有服务端，其实不需要 Outline 的客户端你也一样可以使用相关的功能，这是后话了，我们先看看官方对这两个项目的介绍吧。

什么是 Outline Manager?

> Outline Manager, developed by Jigsaw. The Outline Manager application creates and manages Outline servers, powered by Shadowsocks. It uses the Electron framework to offer support for Windows, macOS and Linux.

什么是 Outline?

> Outline clients, developed by Jigsaw. The Outline clients use the popular Shadowsocks protocol, and lean on the Cordova and Electron frameworks to support Windows, Android / ChromeOS, iOS and macOS.

一句话说完，Outline Manager 是由 Jigsaw 开发的用于创建和管理 Outline 服务的跨平台应用，而 Outline 则是一款使用目前流行的 SS 协议的跨平台代理应用。

在 Outline 的官方页面上有说到，Outline 是为新闻机构而生的，它易于创建和管理，并且具有很强的私密性和安全性，用于保护新闻机构工作者们的隐私。

# 体验 Outline

在开始体验和创建你的个人 Outline 服务之前，请确保你已经有一个可用的，并且支持 Docker 的服务器，如果没有，那没关系，你可以去购买一个新的 VPS 主机，Outline 官方也提到了，现在的 VPS 主机基本只需要 5 美元一个月，你可以选择使用 [DigitalOcean](https://www.digitalocean.com/), [Linode](https://www.linode.com/) 或者 [Vultr](https://www.vultr.com/?ref=6963885). 我个人目前在使用的是 [Vultr](https://www.vultr.com/?ref=6963885) 家 5 美元每月的计划，其余两家我没有使用过，个人推荐的话我也会推荐你使用 [Vultr](https://www.vultr.com/?ref=6963885) 家的，不仅可以使用支付宝支付，而且迈阿密（Miami）机房的机子还有 2.5 美金每月的计划。

注：如果你是新注册 Vultr，可以使用 `NGINX20` 这个优惠码，注册后会赠送20美元余额。

OK，如果你已经准备好了，接下来要做的事情就显得十分简单了。首先要去 Outline 的官网或者 GitHub 仓库，将 Outline Manager 和 Outline 分别下载好，传送门在下面：

- [Outline 官网](https://getoutline.org/en/home)
- [Outline Manager GitHub 仓库下载地址](https://github.com/Jigsaw-Code/outline-server/releases)
- [Outline GitHub 仓库下载地址](https://github.com/Jigsaw-Code/outline-client/releases)

注：如果是在官方的 Github 仓库下载的话，请选择带有绿色的 `Latest release` 标签的版本，如果你使用的是 `Pre-release` 版本的话，可能会出现一些意外的 Bug.

在下载完毕并且安装好后，我们先打开 Outline Manager，可以看到在界面上让我们跟随指引进行 Outline 服务安装，只有两个步骤，十分简单。

![Outline Manager](/images/Outline/outline-manager.png)

 第一步，登录你的服务器，然后在终端粘贴输入 Outline Manager 的框框里那一长串命令

```shell
wget -qO- https://raw.githubusercontent.com/Jigsaw-Code/outline-server/master/src/server_manager/install_scripts/install_server.sh | bash
```

如果你的服务器已经安装并且启动了 Docker 的服务，那么执行完这条命令之后就已经大功告成了，但是如果你还没有安装 Docker 或者说安装了还没启动，你会发现执行上面的命令之后会出现一到两个错误，不要慌，我们一个个来解决。

如果你发现错误中有一条显示 `Docker CE must be installed...` 这样的错误，说明你的服务器还没安装 Docker，解决方法就是直接运行错误中的那条命令 `curl -sS [https://get.docker.com/](https://get.docker.com/) | sh` 或者根据官方的「[安装指南](https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1)」手动进行安装。

如果你已经安装了 Docker，但是依然有错误，那应该是说你还没将 Docker 服务跑起来，很简单只需要执行 `sudo systemctl start docker` 将服务运行起来就 OK 了。

注：以上操作都是基于服务器环境 CentOS 7.x 进行，如果你使用的是其他系统，请具体参考 Docker 官方的文档进行实施。

完成 Docker 的安装并启动之后，再次执行最开始的那一条长长的命令，这一次估计没有任何的错误了，安装成功之后，会发现终端输出了一段 JSON 格式的内容，也就是类似 Outline Manager 界面中第二个框框所显示的那样，我们将它复制之后粘贴到 Outline Manager 中，然后点击 DONE 即可完成服务器的安装配置，超方便快捷有没有🎉

如果依然发现有错误，Outline Manager 已经给出了明确的提示，是因为连接不到服务器，你需要前往服务器开启相应的端口，允许对应端口的 TCP 和 UDP 连接，因为服务器所使用的系统不一样，开启方法也各有不同，这里就不进行说明了，Google 一下就有答案啦😉

![Outline Manager](/images/Outline/outline-manager2.png)

完成之后我们可以看到 Outline Manager 进入了管理界面，点小齿轮可以进行一些简单的设置，例如修改当前 Outline 服务的名称，删除该服务等。我们把重点放在默认已经创建好的「My access key」上，点击「GET CONNECTED」会询问你是本机连接还是其他设备进行连接，我们选择「CONNECT THIS DEVICE」连接当前设备，然后就会看到一个「access code」可以复制，聪明的小伙伴已经猜到了，这个 `ss://.....` 开头的链接，其实就是 SS 的配置链接，直接将它粘贴到支持 SS 的应用上就可以开始使用了，这也就是为什么一开始我就提到了其实不需要 Outline 的客户端你也一样可以使用相关的功能的缘由。既然我们要体验 Outline，那当然要打开 Outline 客户端了，

![Outline](/images/Outline/outline.png)

Outline 的 Mac 版打开之后会出现在顶部的状态栏中，打开之后点中间的图标，会弹出框框让你填写 access key，只需要把刚刚复制出来的内容粘贴到这里，然后点击「ADD SERVER」即可。接下来的连接和测试网络，我想不用我继续说下去大家也都知道了吧~

如果想和其他人一起使用的话，点击「ADD KEY」创建一个 access key，然后点击「SHARE」复制里面的内容分享给小伙伴吧🙊

# 体验过后

不得不说，Jigsaw 果然是和谷歌同一个妈生的，应用体验十分好，简洁好用，服务器端的配置极大的简化了步骤，基本上只需要复制粘贴然后按回车就好了，全程没有什么特别的技能要求。

使用过程中发现唯一美中不足的是，代理默认是全局的，所有的流量都会通过 Outline 进行访问，没有办法根据需要定制访问规则，在「[issue#8](https://github.com/Jigsaw-Code/outline-client/issues/8)」中可以看到相关的讨论，看起来在后续的版本中会将这个特性加上。

Outline 还只是个新生的项目，即使是谷歌的同门，也不可能一开始就把一切都做得尽善尽美，不如一起期待 Outline 的成长吧👏

# 相关链接

- [Vultr 官网](https://www.vultr.com/?ref=6963885)
- [Linode 官网](https://www.linode.com/)
- [DigitalOcean 官网](https://www.digitalocean.com/)
- [Docker 安装指南](https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1)
- [Outline 官网](https://getoutline.org/en/home)
- [Outline GitHub 仓库](https://github.com/Jigsaw-Code/outline-client)
- [Outline Manager GitHub 仓库](https://github.com/Jigsaw-Code/outline-server)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](https://ruterly.com/)」原创。转载务必注明[出处](https://ruterly.com/2018/04/12/Outline-developed-by-Jigsaw/)。

本文永久地址：https://ruterly.com/2018/04/12/Outline-developed-by-Jigsaw/
