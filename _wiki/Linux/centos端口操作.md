---
layout: wiki
title: "CentOS端口操作"
date: 2016-11-01 15:55
Keywords: centos, 端口
categories: Linux
---

# CentOS 6
查看当前所有的`iptables`配置

```
iptables -L -n
```

添加允许`INPUT`访问规则，如果需要拒绝访问，将`ACCEPT`改为`DROP`即可

```
# HTTP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# 打开一个范围的端口
iptables -A INPUT -p tcp --dport 9000:9009 -j ACCEPT
```

保存`iptables`的设置

```
/etc/rc.d/init.d/iptables save
```

开启、停止、状态、重启

```
/etc/init.d/iptables start
/etc/init.d/iptables stop
/etc/init.d/iptables status
/etc/init.d/iptables restart # 或者 service iptables restart
```


# CentOS 7
CentOS 7使用`firewalld`代替了原来的`iptables`，相关端口命令如下

开启端口

```
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 打开多个端口
firewall-cmd --zone=public --add-port={10010/tcp,10080/tcp,10090/tcp} --permanent
```

含义

```
--zone #作用域

--add-port=80/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效
```

重启防火墙

```
firewall-cmd --reload
```