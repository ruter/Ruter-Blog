---
layout: wiki
title: "使用dd指令烧录ISO镜像到U盘"
date: 2017-01-02 00:19
keywords: 烧录, 镜像, dd指令
categories: Mac
---

# 查看位置
打开`终端`，使用`diskutil list`命令查看所有磁盘，记住所要写入的U盘的位置，如`/dev/disk1`

注：`Ubuntu`使用`sudo fdisk -l`查看

# 取消挂载
用`diskutil unmount /dev/disk1`命令取消挂载U盘

# 写入镜像
使用如下命令将镜像写入U盘，其中`if`表示镜像所在路径:

```bash
sudo dd if=xxx.iso of=/dev/disk1 bs=1m; sync

# Ubuntu
sudo dd if=xxx.iso of=/dev/sdx bs=4M && sync
```

# 弹出U盘
写入完毕后，即可弹出U盘:

```
diskutil eject /dev/disk1
```
