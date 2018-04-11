---
title: 「玩物志」Shadowrocket的使用及配置
date: 2016-10-07 17:05:59
toc: true
keywords: Shadowsocks, Shadowrocket
categories: [Tutorial, 玩物志]
description: 用小火箭带你跨越高墙
---

这篇Blog是应朋友要求写的，之前安利他买了`Shadowrocket`，然后一直没有教他怎么用 :)没错，我就是这么坑XD

# 一点说明
什么是`Shadowrocket`? 用它可以干些什么？

> Shadowrocket is Shadowsocks client for iOS

是的你没看错，作者的博客就是这么介绍`Shadowrocket`的，简单粗暴明了。那么问题来了，什么是`Shadowsocks`呢？

> Shadowsocks is a high-performance cross-platform secured socks5 proxy. It will help you surf the internet privately and securely

简单地说，`Shadowsocks`是一种代理，而要使用这个代理你需要一个客户端，`Shadowrocket`就是众多客户端之一。

那`Shadowsocks`具体是怎么工作的呢？这里配上一张原理图进行简单的说明

![Shadowsocks原理图](/images/Shadowrocket/what-is-shadowsocks.png)

1. PC客户端（即你的电脑）发出请求和SS Local端进行通讯
2. SS Local发出加密的请求和SS Server进行通讯，经过GFW时将会被放行（GFW无法对没有明显特征码的通讯数据进行解密）
3. SS Server将收到的数据解密还原请求后发送到用户需要访问的网站，然后再将响应加密通过原路返回

# 准备工作
前面说明里已经提到了`Shadowrocket`和`Shadowsocks`了，这两样东西当然是必不可少的了，除此之外还需要一台「iOS 9」以上的iPhone或iPad

- iPhone或iPad（iOS 9+）
- Shadowrocket（App Store 购买 ¥6）
- Shadowsocks 账号一枚（自己搭建或购买）

# 基本配置
准备好之后我们来进行配置，我们可以通过扫描二维码添加SS的配置信息，也可以通过手动填写来添加

![生成二维码](/images/Shadowrocket/SS_QRcode.png)

在电脑SS客户端点击「生成二维码...」就会出现包含当前使用的配置信息的二维码，只需通过小火箭左上角的扫描二维码按钮打开扫码功能进行扫描即可将配置信息成功添加到小火箭里

![手动添加](/images/Shadowrocket/mannual.png)

手动添加配置只需要点击右上角的「+」，再根据不同的选项填写好你SS服务的配置信息即可

其中「类型」目前一般都是用`Shadowsocks`或`ShadowsocksR`，具体用哪个请根据服务器搭建的类型而定

「服务器」、「端口」、「密码」、「算法」这几项都是根据你的SS账号配置来填写，「备注」则可以任意填写，如日本服务器可以填写「🇯🇵JP」以方便识别

如果没有特殊需求，配置到这里就已经算结束了，选中你的配置然后连接上即可畅游网络了 :)

![请求允许](/images/Shadowrocket/allow.png)

第一次连接会询问你是否要添加配置，点击「Allow」后输入密码同意即可正常连接了，一个配置只会在初次连接时出现询问操作

![访问Youtube](/images/Shadowrocket/youtube.PNG)

# 进阶配置
如果想要对不同的连接进行区分，对于是否要通过SS进行访问拥有一种可控的有效措施，那就要进行「规则」配置了

点击小火箭的「设置」选项，然后点击「配置」

![规则配置](/images/Shadowrocket/config.png)

我们可以从二维码、iCloud Drive和Wi-Fi导入配置，也可以从远程文件下载导入，网上有很多规则配置文件可供下载使用，如[Abclite](http://www.abclite.cn/Abclite_SR.conf)的配置文件

这里将演示通过远程文件来添加配置，点击「添加配置」，然后输入Abclite提供的配置文件的地址

```
http://www.abclite.cn/Abclite_SR.conf
```

![添加远程配置文件](/images/Shadowrocket/down.png)

点击「下载」即可，配置现在还是没有导入的状态，需要点击「远程文件」下面的链接，选择「使用配置」才会正式导入并切换使用该配置

![使用配置](/images/Shadowrocket/use.png)

说了那么多，也只是用别人的配置而已，那要是想自己定制规则要怎么办呢？如果想要自己添加规则，只需要点击当前使用规则旁边的提示符号，然后点击「添加规则」就可以按照自己的需求来添加了

![规则](/images/Shadowrocket/rule.png)

规则里各种类型的说明以及修改方法如下

```
// 基于域名判断并屏蔽（REJECT）请求
DOMAIN,pingma.qq.com,REJECT

// 基于域名后缀判断屏蔽（REJECT）请求
DOMAIN-SUFFIX,flurry.com,REJECT

// 基于关键词后缀判断走代理（Proxy），强制不尊重系统代理的请求走
Packet-Tunnel-Provider DOMAIN-KEYWORD,google,Proxy,force-remote-dns

// 基于域名后缀判断请求走直连（DIRECT）
DOMAIN-SUFFIX,126.net,DIRECT

// Telegram.app 指定“no-resolve”Surge 忽略这个规则与域的请求。
IP-CIDR,91.108.56.0/22,Proxy,no-resolve

// 判断是否是局域网，如果是，走直连
IP-CIDR,192.168.0.0/16,DIRECT

// 判断服务器所在地，如果是国内，走直连
GEOIP,CN,DIRECT

// 其他的全部走代理
FINAL,Proxy

// 其他的全部不走代理
FINAL,DIRECT
```

因为`Shadowrocket`的规则是兼容`Surge`的，所以如果想深入学习规则的配置的话，可以查看本文「参考资料」部分提供的链接 :)

# 参考资料
- [ShadowSocks的翻墙原理](https://tumutanzi.com/archives/13005)
- [Surge -定制自己的规则配置](https://medium.com/@scomper/surge-%E5%AE%9A%E5%88%B6%E8%87%AA%E5%B7%B1%E7%9A%84%E8%A7%84%E5%88%99%E9%85%8D%E7%BD%AE-34a6d74b0434#.rf4vk0tfg)
- [iOS Surge/Shadowrocket配置文件](http://www.abclite.cn/1995.html)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://blog.ruterly.com/)」原创。转载务必注明[出处](http://blog.ruterly.com/2016/10/07/How-to-use-Shadowrocket/)。

本文永久地址：http://blog.ruterly.com/2016/10/07/How-to-use-Shadowrocket/
