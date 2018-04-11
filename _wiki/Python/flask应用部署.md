---
layout: wiki
title: "Flask应用部署"
date: 2017-01-17 13:23
keywords: Flask, 部署
categories: Python
---

# 部署工具

## 应用容器

在开发应用时，本地运行的那个服务器并不能处理真实的请求。当你真的需要向公众发布你的应用，你需要在应用容器，例如`Gunicorn`，上运行它。Gunicorn接待请求，并处理诸如线程的复杂事务。

要想使用Gunicorn，需要通过pip安装`gunicorn`到虚拟环境中。

为了在后台运行这个服务器（也即使它变成守护进程），我们可以传递`-D`选项给Gunicorn。这下它会持续运行，即使你关闭了当前的终端会话。

如果我们这么做了，当我们想要关闭服务器时就会困惑于到底应该关闭哪个进程。我们可以让Gunicorn把进程ID储存到文件中，这样如果想要停止或者重启服务器时，我们可以不用在一大串运行中的进程中搜索它。我们使用`-p <file> `选项来这么做。现在，我们的Gunicorn部署命令是这样：

```bash
(ourapp)$ gunicorn app:app -p app.pid -D
(ourapp)$ cat app.pid
63101
```

要想重新启动或者关闭服务器，我们可以运行对应的命令：

```bash
(ourapp)$ kill -HUP `cat app.pid` # 发送一个SIGHUP信号，终止进程
(ourapp)$ kill `cat app.pid`
```

默认下Gunicorn会运行在`8000`端口。如果这已经被另外的应用占用了，你可以通过添加`-b`选项来指定端口。

```bash
(ourapp)$ gunicorn app:app -p app.pid -b 127.0.0.1:7999 -D
```

## Nginx反向代理

反向代理处理公共的HTTP请求，发送给Gunicorn并将响应带回给发送请求的客户端。

要想配置Nginx作为运行在127.0.0.1:8000的Gunicorn的反向代理，我们可以在*/etc/nginx/sites-available*下给应用创建一个文件。不如称之为*exploreflask.com*吧。

*/etc/nginx/sites-available/exploreflask.com*

```
# Redirect www.exploreflask.com to exploreflask.com
server {
        server_name www.exploreflask.com;
        rewrite ^ http://exploreflask.com/ permanent;
}

# Handle requests to exploreflask.com on port 80
server {
        listen 80;
        server_name exploreflask.com;

        # Handle all locations
        location / {
                # Pass the request to Gunicorn
                proxy_pass http://127.0.0.1:8000;

                # Set some HTTP headers so that our app knows where the request really came from
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

现在在*/etc/nginx/sites-enabled*下创建该文件的符号链接，接着重启Nginx。

```
$ sudo ln -s \
/etc/nginx/sites-available/exploreflask.com \
/etc/nginx/sites-enabled/exploreflask.com
```

## ProxyFix

有时，你会遇到Flask不能恰当处理转发的请求的情况。这也许是因为在Nginx中设置的某些HTTP报文头部造成的。我们可以使用Werkzeug的ProxyFix来fix转发请求。

*app.py*

```
from flask import Flask

# Import the fixer
from werkzeug.contrib.fixers import ProxyFix

app = Flask(__name__)

# Use the fixer
app.wsgi_app = ProxyFix(app.wsgi_app)

@app.route('/')
def index():
    return "Hello World!"
```

## 总结

- 你可以把Flask应用托管到AWS EC2， Heroku和Digital Ocean。
- Flask应用的基本部署依赖包括一个应用容器（比如Gunicorn）和一个反向代理（比如Nginx）。
- Gunicorn应该退居Nginx幕后并监听127.0.0.1（内部请求）而非0.0.0.0（外部请求）
- 使用Werkzeug的ProxyFix来处理Flask应用遇到的特定的转发报文头部。