---
title: 「玩物志」Syncthing的安装与使用
date: 2016-09-06 12:56:43
toc: true
keywords: Syncthing, Cloud
categories: [Tutorial, Server, 玩物志]
description: 「玩物志」系列之创建自己的私密网盘
---

现在的网盘，一言不合就被「脱裤」，又或者「根据相关法律法规」要整改，要么干脆就关闭了，你们这样让我非常angry！这样乱搞，还让不让人好好地备份文件啦？本着「自己动手丰衣足食」的理念，今天老司机我带大家用`Syncthing`来创建属于自己的同步网盘。

# 什么是Syncthing
按照惯例应该是要先介绍今天的主角的，下面是摘自[Syncthing](https://syncthing.net/ "访问Syncthing官网")官网首页的一段介绍
> Syncthing replaces proprietary sync and cloud services with something open, trustworthy and decentralized. Your data is your data alone and you deserve to choose where it is stored, if it is shared with some third party and how it's transmitted over the Internet.

一句话说完就是「我们这个东西跟那些云服务不一样，是非常安全可靠值得信赖的，你的数据由你来作主」。

# 准备
既然要同步文件，当然要有至少两台机器了，我这里用的是一台笔记本电脑和一个CentOS的VPS，笔记本作为本地设备，VPS作为远程设备。这里要说明一点，用作文件同步的设备，可以是任何系统任何设备，并不是限定于必须要有一台服务器，在局域网内的两台电脑都可以建立你自己的同步网盘(网盘这个说法其实并不准确)。除了两台机器外，还需要机器系统对应的Syncthing的二进制文件，具体可以从[Syncthing首页](https://syncthing.net/ "访问Syncthing首页")的「Syncthing Core (CLI & Web UI)」里找到对应版本的下载地址。

现在需要的东西都已经准备好了：
- Windows 7 32bit
- CentOS 6 64bit
- [Syncthing Core Windows 32 bit](https://github.com/syncthing/syncthing/releases/download/v0.14.5/syncthing-windows-386-v0.14.5.zip "Syncthing Core Windows 32 bit")
- [Syncthing Core Linux 64 bit](https://github.com/syncthing/syncthing/releases/download/v0.14.5/syncthing-linux-amd64-v0.14.5.tar.gz "Syncthing Core Linux 64 bit")

# 安装
先从官网下载好Windows 32位版(我本本对应的系统版本)的Syncthing，解压后可以看到如下文件结构
![Syncthing文件结构](/images/Syncthing/1.PNG)

直接运行`syncthing.exe`会弹出一个黑框框，里面会有一大堆信息，可以不用管
![syncthing.exe](/images/Syncthing/2.PNG)

同时浏览器还会打开`http://127.0.0.1:8384/`这个网址，可以看到默认已经创建了一个默认文件夹`yct7k-lrebo`，所在路径为`C:\Users\Administrator\Sync`
![Syncthing管理页面](/images/Syncthing/3.PNG)

本地的机器Windows版本就这么简单搞定啦！接下来给VPS也装上，用`Xshell`连上服务器，然后用`wget`命令下载`Syncthing`的Linux 64位版，版本号对应官网上的最新版，请自行选择:
```
cd ~
wget https://github.com/syncthing/syncthing/releases/download/v0.14.5/syncthing-linux-amd64-v0.14.5.tar.gz
```

现在可以把下载到的文件解压，然后进入解压后的目录:
```
tar xzvf syncthing-linux-amd64-v0.14.5.tar.gz
cd syncthing-linux-amd64-v0.14.5
```

有个可执行文件`syncthing`，我们要把它放到我们的`PATH`中，以便直接执行:
```
cp syncthing /usr/local/bin
```

之前下载和解压出来的文件可以全部删掉了:
```
cd ~
rm -rf syncthing*
```

到这里我们在VPS上的Syncthing已经安装好了，可是直接运行的话，并不能通过外网访问到管理页面，因为Syncthing的管理页面默认是只有本机可以访问的，所以接下来还要进行一点修改，先运行Syncthing:
```
syncthing
```

随后就会看到有很多信息，和之前在Windows运行一样，看到类似以下内容的时候就可以按`CTRL-C`退出程序了:
```
[OH4IP] 13:32:15 INFO: Completed initial scan (rw) of folder edatb-zzc5f
[OH4IP] 13:32:15 INFO: Device OH4IPQD-QDCDAZB-YMMZE4F-BAK4BLQ-3EZLPTD-V73J37V-LTW44V6-YSM6JQ7 is "ruter.ga" at [dynamic]
[OH4IP] 13:32:15 INFO: Loading HTTPS certificate: open /root/.config/syncthing/https-cert.pem: no such file or directory
[OH4IP] 13:32:15 INFO: Creating new HTTPS certificate
[OH4IP] 13:32:15 INFO: GUI and API listening on 127.0.0.1:8384
[OH4IP] 13:32:15 INFO: Access the GUI via the following URL: http://127.0.0.1:8384/
[OH4IP] 13:32:16 INFO: Detected 0 NAT devices
```

我们第一次运行是为了让它创建配置文件，然后我们再进行修改。用以下命令对配置文件进行编辑:
```
vim ~/.config/syncthing/config.xml
```

一瞬间是不是懵逼了？不要慌，先找到下面这几行:
```
<gui enabled="true" tls="false" debugging="false">
    <address>127.0.0.1:8384</address>
    <apikey>2GeGJK9z6tXKP3nHJYU56ZHoYSYnqQ9S</apikey>
    <theme>default</theme>
</gui>
```

然后把IP`127.0.0.1`修改成`0.0.0.0`即可保存退出:
```
<gui enabled="true" tls="false" debugging="false">
    <address>0.0.0.0:8384</address>
    <apikey>2GeGJK9z6tXKP3nHJYU56ZHoYSYnqQ9S</apikey>
    <theme>default</theme>
</gui>
```

设置好之后执行`syncthing`运行，就可以通过`http://your_ip_addr:8384`来进行访问管理了，如果直接通过外网ip:端口访问还是无法打开管理页面，那就需要进行防火墙的设置开启8384端口了:
```
iptables -I INPUT -p tcp --dport 8384 -j ACCEPT
service iptables save
service iptables restart
syncthing
```

再次打开`http://your_ip_addr:8384`就能看见管理页面了
![VPS上的管理页面](/images/Syncthing/4.PNG)

可以很明显地看到一条警告信息，提醒我们设置管理用户及密码，点击「设置」，然后把「用户名」和「密码」填写好，「使用加密连接到图形管理页面」这个是开启`HTTPS`，按需勾选
![设置页面](/images/Syncthing/5.PNG)

# 同步
打开本地管理页面`http://127.0.0.1:8384/`，然后点击「添加远程设备」将VPS添加到同步列表里，其中「设备ID」需要在VPS的管理页面打开「操作」--「显示ID」查看，将ID复制到「设备ID」一栏中，「地址列表」默认使用`dynamic`即可，其他按需修改
![添加设备](/images/Syncthing/6.PNG)

保存之后我们可以在VPS端的管理页面上看见连接请求
![连接请求](/images/Syncthing/7.PNG)

添加成功后会有共享文件夹的提示
![分享文件夹](/images/Syncthing/8.PNG)

为了测试文件同步是否成功，我在本地同步路径`C:\Users\Administrator\Sync`添加了一个文件`ROR.txt`，自动同步完成后可以在VPS端管理页面看到「最后接收的文件」显示「已更新 ROR.txt」
![同步成功](/images/Syncthing/9.PNG)

# 进阶
在服务器上使用Syncthing可以修改配置文件后使用外网进行访问管理，本地端也可以如法炮制，如果没有外网IP则需要使用`花生壳`之类的进行映射，具体操作请移步Google :)

Syncthing有一些高级的功能前面没有提及，例如每个共享的文件夹都可以在「选项」内打开「高级设置」，进行一些设置，如开启「版本控制」。

通过Syncthing共享的文件夹，被取消共享后，本地已经同步的文件也依然会存在。

除了自己使用，在小圈子内也是很有利用价值，例如共享资源什么的，再也不用忍受各种网盘的龟速上传下载以及删资源啦！

# 扩展
以上只是简单的安装和设置步骤，还有很多内容没有涉及到，例如开机启动、忽略同步内容、命令行操作等等，具体请查看官方文档，里面有非常详尽的教程。
- [官方文档](https://docs.syncthing.net/index.html)
- [开机自启动](https://docs.syncthing.net/users/autostart.html)
- [命令行操作](https://docs.syncthing.net/users/syncthing.html)

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://blog.ruterly.com/)」原创。转载务必注明[出处](http://blog.ruterly.com/2016/09/06/How-to-Use-Syncthing/)。

本文永久地址：http://blog.ruterly.com/2016/09/06/How-to-Use-Syncthing/
