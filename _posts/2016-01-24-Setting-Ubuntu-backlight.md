---
title: 笔记本安装Ubuntu后背光亮度的调节
date: 2016-01-24 08:15:51
keywords: Ubuntu
categories: Tutorial
description: 笔记本安装 Ubuntu 后无法调节亮度，这一篇帮你解决
---

有一台淘汰下来的渣配置本本，游戏也跑不起，二手似乎也不值几个钱，扔了又可惜，本着发扬中华民族优良传统的主旨，把本本换上了`Ubuntu`作为学习机用，可是用得久了之后就发现了一个问题——这屏幕是要亮瞎我的狗眼啊！在`Linux`系统上的显卡驱动好像并不是那么多那么全啊，导致没法通过`Fn`加快捷键进行屏幕亮度的调节，用久了眼镜真的很累啊有木有！所以只能想办法手动解决啦~

Google了很多方法都未果，大多是通过什么brightness文件修改数值什么的，然并卵，或许是我这机器太傲娇了不成？！既然软的不行，那就来硬的了呗！废话不多说，动手实操~

P.S. 以下操作均是在`Ubuntu`环境下，其他`Linux`分支系统仅作为参考（我觉得差别应该不大吧？），具体哪个版本的`Ubuntu`其实都是没问题的，我在几个版本下都用同样的方法成功调节了屏幕的亮度 :)

执行`lspci`命令列出所有的硬件信息

```
00:00.0 Host bridge: Intel Corporation Mobile 4 Series Chipset Memory Controller Hub (rev 07)
00:02.0 VGA compatible controller: Intel Corporation Mobile 4 Series Chipset Integrated Graphics Controller (rev 07)
00:02.1 Display controller: Intel Corporation Mobile 4 Series Chipset Integrated Graphics Controller (rev 07)
00:1a.0 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #4 (rev 03)
00:1a.1 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #5 (rev 03)
00:1a.2 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #6 (rev 03)
00:1a.7 USB controller: Intel Corporation 82801I (ICH9 Family) USB2 EHCI Controller #2 (rev 03)
00:1b.0 Audio device: Intel Corporation 82801I (ICH9 Family) HD Audio Controller (rev 03)
00:1c.0 PCI bridge: Intel Corporation 82801I (ICH9 Family) PCI Express Port 1 (rev 03)
00:1c.1 PCI bridge: Intel Corporation 82801I (ICH9 Family) PCI Express Port 2 (rev 03)
00:1d.0 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #1 (rev 03)
00:1d.1 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #2 (rev 03)
00:1d.2 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #3 (rev 03)
00:1d.7 USB controller: Intel Corporation 82801I (ICH9 Family) USB2 EHCI Controller #1 (rev 03)
00:1e.0 PCI bridge: Intel Corporation 82801 Mobile PCI Bridge (rev 93)
00:1f.0 ISA bridge: Intel Corporation ICH9M LPC Interface Controller (rev 03)
00:1f.2 SATA controller: Intel Corporation 82801IBM/IEM (ICH9M/ICH9M-E) 4 port SATA Controller [AHCI mode] (rev 03)
00:1f.3 SMBus: Intel Corporation 82801I (ICH9 Family) SMBus Controller (rev 03)
01:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 06)
02:00.0 Network controller: Qualcomm Atheros AR9285 Wireless Network Adapter (PCI-Express) (rev 01)
```

找到`VGA`那一行，记下前面的编号`00:02.0`，稍后会用到，可能在你的机器上看到的是其他编号，到时候以你自己机器的为准。

在终端输入下面这条命令，将XX替换为00~FF的任意数值

```
sudo setpci -s 00:02.0 F4.B=XX
```

其中`00:02.0`就是我们前面记下来的`VGA`的编号，`F4.B`是我们要改变的属性，这里指的是亮度，取值是两位十六进制数。

**注意！**千万不要把XX设置得太低！不然屏幕会一片漆黑！什么都看不见！想要重新设置回去就只能对着漆黑一片的屏幕，靠着直觉用命令自个儿调吧，不要问我为什么知道，我以前不懂事的时候因为这事儿重装过一次系统！建议从10开始微调，调节至不刺眼为止。

搞定之后是不是~~眼前一黑~~屏幕一黑，眼睛舒服多啦！可是每次都要打这么一串东西烦不烦啊，多浪费时间多没效率对吧，既然如此，那我们就把它做成脚本呗~

新建一个叫做`backlight.sh`的脚本，输入这两行内容，同样记得把XX改成00~FF中的一个数值

```
#!/bin/bash
sudo setpci -s 00:02.0 F4.B=XX
```

又或者想根据心情来手动调节一下亮度，不把数值写死了，那就按下边这样写

```
#!/bin/bash
echo "Enter a value:"
read s
sudo setpci -s 00:02.0 F4.B=$s
```

搞定之后打开终端，输入`sh backlight.sh`调用脚本试试有没有效果，调用的时候会需要输入密码，完事立马就能看到屏幕亮度有了变化啦！

```
sh backlight.sh

Input a value:
20
[sudo] password for ruter: 
```

懒得自己写脚本的，也可以下载了直接用嗷~

- [固定数值为20的脚本](/Scripts/backlight.sh)
- [手动输入数值的脚本](/Scripts/backlight2.sh)

感觉每次都要手动调用还是好麻烦呀~能不能有什么开机自启动的方法啊？回答是肯定的！不过，把脚本加入开机启动项好像不是很难嘛~既然如此为什么不自己动手试试？如果不会，记得善用搜索引擎XD

---

![知识共享许可协议](https://i.creativecommons.org/l/by-nc-nd/3.0/cn/88x31.png)

**声明：**本站的所有文章，都采用[知识共享署名-非商业性使用-禁止演绎 3.0 中国大陆许可协议](http://creativecommons.org/licenses/by-nc-nd/3.0/cn/)进行许可。

**注意：**若未作说明，则本文为「[**TNK**](http://ruterly.com/)」原创。转载务必注明[出处](http://ruterly.com/2016/01/24/Setting-Ubuntu-backlight/)。

本文永久地址：http://ruterly.com/2016/01/24/Setting-Ubuntu-backlight/
