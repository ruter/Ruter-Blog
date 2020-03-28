---
title: 配置Nginx和uWSGI搭建Django运行环境
date: 2015-12-05 12:39:32
toc: true
categories: Server
keywords: Server, Nginx, uWSGI
description: 在自己的服务器上用 Nginx 和 uWSGI 搭建 Django 的运行环境
---

不久前试用了阿里云的ECS，用来试着部署之前用`Django`写的一个博客，遇到了不少问题啊TvT在Google上搜出来的方法都是旧的没法解决问题呢，所以就摸索着弄，最后不得不说，官方文档才是人类的好基友啊(缺胳膊少腿的我们暂时忽略掉吧)！

# 更新和安装需要的包
我使用的系统是64位的CentOS 6.5

```
yum -y update
yum install -y epel-release sqlite-devel zlib-devel.x86_64 openssl-devel.x86_64 python-devel
```

## 安装PCRE
下载并解压
```
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
tar zxvf pcre-8.37.tar.gz
```
编译安装
```
cd pcre-8.37
./configure
make && make install
```
安装完成后可以查看版本号`pcre-config --version`

## 安装Nginx
我这里用的是1.8.0 stable 版本，先下载然后解压进目录进行配置编译安装，这里直接给出命令不再详述
```
wget http://nginx.org/download/nginx-1.8.0.tar.gz
tar nginx-1.8.0.tar.gz
cd nginx-1.8.0
./configure --prefix=/usr/local/nginx
make && make install
```

## 安装uWSGI
在上一篇升级Python的博客里已经顺便安装好了pip，现在派上用场了
```
pip install uwsgi
```

## 安装Django
```
pip install django
```

到这里我们已经把需要安装的东西都准备好了，现在开始进行配置

# 配置
## 配置uWSGI
假设我们已经有一个Django的项目叫`blog`，路径是`/var/www/blog/`，现在进入这个项目的目录下，新建一个`blog.ini`文件，添加如下内容
```
[uwsgi]
uid = www
gid = www

chdir = /var/www/blog
module = blog.wsgi

master = true
processes = 10

socket = /tmp/blog.sock
chmod-socket = 664

vacuum = true

daemonize = /var/www/blog/blog.log
```

## 配置Nginx
创建 Nginx 运行使用的用户 www：
```
/usr/sbin/groupadd www
/usr/sbin/useradd -g www www
```

编辑Nginx的配置文件`nginx.conf`
```
vi /usr/local/nginx/conf/nginx.conf
```
将第一行的`#user  nobody;` 改成 `user	www www;`
然后找到下面这两行，去掉注释`#` 
```
#error_log  logs/error.log;
#pid        logs/nginx.pid;
```
然后在`http {}`块内的最下面添加以下内容
```
upstream blog {
  server unix:///tmp/blog.sock;
}

server {
  listen 8000;
  server_name .example.com;

  charset utf-8;

  client_max_body_size 75M;

  location /media {
  	alias /var/www/blog/media;
  }

  location /static {
  	alias /var/www/blog/static;
  }

  location / {
    uwsgi_pass blog;
    include uwsgi_params;
  }
```

噔噔！我们已经完成了Nginx的基础配置了，想要了解更多Nginx的具体配置请参考官方提供的[完整配置示例](https://www.nginx.com/resources/wiki/start/topics/examples/full/)

# 启动测试
## 开启Nginx
在启动Nginx之前先对配置文件的语法进行检查
```
/usr/local/nginx/sbin/nginx -t
```

确认无误后启动Nginx
```
/usr/local/nginx/sbin/nginx
```

这时可能会出现如下的错误
```
nginx: [error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
```

只需要执行以下这条命令就可以解决问题了
```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

Nginx的常用命令有
```
/usr/local/nginx/sbin/nginx -s reload|reopen|stop
```

分别是重新读取配置文件，重新启动以及停止

## 开启uWSGI
启动Nginx（请参考前面）之后，我们再来启动uWSGI 
```
uwsgi --ini /var/www/blog/blog.ini
```

开启成功之后就可以访问Nginx配置里`server_name`所对应的ip或域名进行访问了，如这里的配置示例，我们访问[http://example.com:8000](http://example.com:8000)就可以看到创建好的blog啦！

# 更多参考文档
[Another nginx.conf Full Example](https://www.nginx.com/resources/wiki/start/topics/examples/fullexample2/)
[Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.org/en/latest/tutorials/Django_and_nginx.html#configure-nginx-for-your-site)
[How To Serve Django Applications with uWSGI and Nginx on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-uwsgi-and-nginx-on-ubuntu-14-04)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** http://ruterly.com/2015/12/05/Configurate-nginx-and-uwsgi/
