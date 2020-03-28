---
layout: post
title: 使用 Docker 实现 Odoo 自动化配置
categories: [Odoo, Docker]
description: Use docker with Odoo
keywords: Odoo, Docker
---

本文将会着重讲解如何配置和使用 Odoo Docker 镜像，以及如何实现一定程度的 Odoo 自动化配置。在开始阅读这篇文章之前，希望你已经具备以下条件：

- Odoo 的基本知识
- 对 Docker 和 Docker Compose 有基本的认知且知晓其常用命令
- 一台已经安装了 Docker 和 Docker Compose 的电脑

即使你没有任何基础知识，只要跟随本文的步骤进行，最终你也可以顺利地在你的电脑上创建并启动一个 Odoo 容器。

电脑上还没有安装 Docker 和 Docker Compose？希望下面的文档可以给你帮助：

- [Install Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
- [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
- [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Install Docker Compose](https://docs.docker.com/compose/install/)

# PostgreSQL

我们先要配置好 PostgreSQL 的容器，并将数据目录映射到宿主机（电脑）的本地目录中进行持久化存储，如果没有进行映射，当容器被移除后，数据将会跟随容器一起消失。

首先创建工作目录 `docker`，我们将会使用 Docker Compose 对 Docker 容器应用进行集中管理，所以需要在工作目录下创建一个 `docker-compose.yml` 文件：

```shell
$ mkdir docker
$ cd docker
$ touch docker-compose.yml
```

接下来要做的很简单，把下面的内容复制到 docker-compose.yml 中，并按需设置环境变量 `environment` 下的值，然后在终端执行 `echo $UID` 查看当前用户的 UID，并将 `$UID` 替换为该值即可：

```yaml
version: '3.3'
services:

  postgres:
    image: postgres:9.5
    ports:
      - 127.0.0.1:22700:5432
    volumes:
      - ./volumes/postgres:/var/lib/postgresql/data
      - /etc/passwd:/etc/passwd:ro
    environment:
      # psql 用户
      - POSTGRES_USER=postgres
      # psql 密码
      - POSTGRES_PASSWORD=6a6bf96d9d81792e8cdcabd232e39e27
    user: $UID:$UID
    restart: always
```

容器内的端口使用 psql 的默认端口 `5432`，映射到宿主机的端口可以随意分配未被占用的端口。而数据目录的映射这里直接选择放在当前工作目录下的 `vloumes/postgres` 目录下，在容器启动时如果该目录不存在，则会自动被创建，你也可以先创建好该目录：

```shell
$ mkdir -p vloumes/postgres
```

然后启动容器，查看状态和日志：

```shell
$ docker-compose up -d
Creating network "docker_default" with the default driver
Creating docker_postgres_1 ... done

$ docker-compose ps
Name                           Command              State             Ports
---------------------------------------------------------------------------------------
docker_postgres_1   docker-entrypoint.sh postgres   Up      127.0.0.1:22700->5432/tcp

$ docker-compose logs -f
```

日志会输入类似的内容：

```shell
Attaching to docker_postgres_1
postgres_1  | The files belonging to this database system will be owned by user "postgres".
postgres_1  | This user must also own the server process.
postgres_1  |
postgres_1  | The database cluster will be initialized with locale "en_US.utf8".
postgres_1  | The default database encoding has accordingly been set to "UTF8".
postgres_1  | The default text search configuration will be set to "english".
postgres_1  |
postgres_1  | Data page checksums are disabled.
postgres_1  |
postgres_1  | fixing permissions on existing directory /var/lib/postgresql/data ... ok
postgres_1  | creating subdirectories ... ok
postgres_1  | selecting default max_connections ... 100
postgres_1  | selecting default shared_buffers ... 128MB
postgres_1  | selecting dynamic shared memory implementation ... posix
postgres_1  | creating configuration files ... ok
postgres_1  | creating template1 database in /var/lib/postgresql/data/base/1 ... ok
postgres_1  | initializing pg_authid ... ok
postgres_1  | setting password ... ok
postgres_1  | initializing dependencies ... ok
postgres_1  | creating system views ... ok
postgres_1  | loading system objects' descriptions ... ok
postgres_1  | creating collations ... ok
postgres_1  | creating conversions ... ok
postgres_1  | creating dictionaries ... ok
postgres_1  | setting privileges on built-in objects ... ok
postgres_1  | creating information schema ... ok
postgres_1  | loading PL/pgSQL server-side language ... ok
postgres_1  | vacuuming database template1 ... ok
postgres_1  | copying template1 to template0 ... ok
postgres_1  | copying template1 to postgres ... ok
postgres_1  | syncing data to disk ... ok
postgres_1  |
postgres_1  | Success. You can now start the database server using:
postgres_1  |
postgres_1  |
postgres_1  | WARNING: enabling "trust" authentication for local connections
postgres_1  | You can change this by editing pg_hba.conf or using the option -A, or
postgres_1  | --auth-local and --auth-host, the next time you run initdb.
postgres_1  |     pg_ctl -D /var/lib/postgresql/data -l logfile start
postgres_1  |
postgres_1  | waiting for server to start....LOG:  database system was shut down at 2019-04-27 01:39:11 UTC
postgres_1  | LOG:  MultiXact member wraparound protections are now enabled
postgres_1  | LOG:  autovacuum launcher started
postgres_1  | LOG:  database system is ready to accept connections
postgres_1  |  done
postgres_1  | server started
postgres_1  |
postgres_1  | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
postgres_1  |
postgres_1  | LOG:  received fast shutdown request
postgres_1  | waiting for server to shut down...LOG:  aborting any active transactions
postgres_1  | LOG:  autovacuum launcher shutting down
postgres_1  | LOG:  shutting down
postgres_1  | .LOG:  database system is shut down
postgres_1  |  done
postgres_1  | server stopped
postgres_1  |
postgres_1  | PostgreSQL init process complete; ready for start up.
postgres_1  |
postgres_1  | LOG:  database system was shut down at 2019-04-27 01:39:13 UTC
postgres_1  | LOG:  MultiXact member wraparound protections are now enabled
postgres_1  | LOG:  database system is ready to accept connections
postgres_1  | LOG:  autovacuum launcher started
```

当容器启动成功后，我们可以运行以下命令打开 PostgreSQL，之后创建一个新的数据库用户：

```shell
$ docker-compose exec postgres psql -U postgres
psql (9.5.16)
Type "help" for help.

postgres=# CREATE user odoo WITH password 'a_strong_psql_pwd';
CREATE ROLE
postgres=# ALTER user odoo WITH createdb;
ALTER ROLE
postgres=#
```

数据库已经准备就绪了，接下来我们开始配置 Odoo 容器。

# Odoo

打开 `docker-compose.yml` 继续编辑，以添加一个 Odoo 12 的容器配置为例，可以按如下方式进行配置，将下面的内容添加到 `services` 下：

```yaml
  odoo12:
    build: ./build/odoo12
    command: start
    ports:
      - 127.0.0.1:22869:8069
      - 127.0.0.1:22872:8072
    links:
      - postgres:db
    volumes:
      - ./volumes/odoo12/addons:/opt/odoo/additional_addons
      - ./volumes/odoo12/filestore:/opt/odoo/data/filestore
      - ./volumes/odoo12/sessions:/opt/odoo/data/sessions
      - ./volumes/odoo12/ssh:/opt/odoo/ssh:ro
    env_file:
      - ./env/odoo12.env
    restart: always
```

现在的 docker-compose.yml 文件看起来应该是这样的：

```yaml
version: '3.3'
services:

  postgres:
    image: postgres:9.5
    ports:
      - 127.0.0.1:22700:5432
    volumes:
      - ./volumes/postgres:/var/lib/postgresql/data
      - /etc/passwd:/etc/passwd:ro
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=6a6bf96d9d81792e8cdcabd232e39e27
    user: 501:501
    restart: always

  odoo12:
    build: ./build/odoo12
    command: start
    ports:
      - 127.0.0.1:22869:8069
      - 127.0.0.1:22872:8072
    links:
      - postgres:db
    volumes:
      - ./volumes/odoo12/addons:/opt/odoo/additional_addons
      - ./volumes/odoo12/filestore:/opt/odoo/data/filestore
      - ./volumes/odoo12/sessions:/opt/odoo/data/sessions
      - ./volumes/odoo12/ssh:/opt/odoo/ssh:ro
    env_file:
      - ./env/odoo12.env
    restart: always
```

**注：`501` 是我当前用户的 UID，请按上一节所述将其替换为你当前用户的 UID**

我们为 odoo12 指定了 `build` 的路径，这是因为我们在开发时，可能会在原始的镜像上添加新的库和依赖等，需要通过 `Dockerfile` 构建适合项目的运行环境，最简单的 Dockerfile 看起来是这样的：

```dockerfile
FROM elicocorp/odoo:12.0
MAINTAINER Elico Corp <webmaster@elico-corp.com>
```

端口映射使用 Odoo 默认端口 `8069` 和 `8072`，宿主机端口则可以任意分配。接下来我们看看挂载的各个目录：

- addons: 放置 Odoo 模块的目录
- filestore: Odoo 的附件等数据存储目录
- sessions: 存储 Odoo 实例 session 的目录
- ssh: 用于拉取私密仓库的密钥存放目录

而环境变量则用单独的一个文件（`odoo12.env`）进行配置，我们将其放置在工作目录下的 `env` 中，一般会有如下内容：

```shell
# 默认要拉取的模块仓库地址
ADDONS_REPO=
# 是否自动拉取关联模块，通过 oca_dependencies.txt 定义
FETCH_OCA_DEPENDENCIES=True
# 宿主机的 UID
TARGET_UID=$UID
# ODOO_ 开头的都是 Odoo 的配置变量，可以自由添加更多的 Odoo 配置变量
ODOO_ADMIN_PASSWD=bde463de157d607dc4d090580ca36170
ODOO_DB_USER=odoo
ODOO_DB_PASSWORD=6a6bf96d9d81792e8cdcabd232e39e27
ODOO_DBFILTER=odoo_.*
```

到目前为止的目录结构应该是这样的：

```shell
.
├── build
│   └── odoo12
│       └── Dockerfile
├── docker-compose.yml
├── env
│   └── odoo12.env
└── volumes
    ├── odoo12
    │   ├── addons
    │   ├── filestore
    │   ├── sessions
    │   └── ssh
    └── postgres
```

在准备好以上配置之后，我们就可以启动 Odoo 容器了：

```shell
$ docker-compose up -d odoo12
```

如果是第一次启动，将会自动下载并编译镜像，你会看到类似的输出：

```shell
Building odoo12
Step 1/2 : FROM elicocorp/odoo:12.0
12.0: Pulling from elicocorp/odoo
7e6591854262: Pull complete
089d60cb4e0a: Pull complete
9c461696bc09: Pull complete
45085432511a: Pull complete
f2681a053cdd: Pull complete
d12dfe0ebf40: Pull complete
15bb4efbf0eb: Pull complete
c53edc8a2e95: Pull complete
b08d48167b84: Pull complete
f00cbf24b3d4: Pull complete
a2e74d4c0033: Pull complete
4a1f843de7d7: Pull complete
704d33171476: Pull complete
83319c077539: Pull complete
6cbcbabe8479: Pull complete
2d65cb1e0ca5: Pull complete
cfaa2f4f9377: Pull complete
30ba6bcb44bf: Pull complete
5595ce086bd2: Pull complete
2b2dd52243fd: Pull complete
ac93402cbef7: Pull complete
4be0786abc0a: Pull complete
a43c62a3a15b: Pull complete
1f21b2b73084: Pull complete
ce6bc9f04f89: Pull complete
2ce939a5e4cf: Pull complete
35a3637dbc19: Pull complete
1d9de7e06844: Pull complete
Digest: sha256:6a59505c36ac642d64172065e95b165174e8330476cd74a35790940ffad055e5
Status: Downloaded newer image for elicocorp/odoo:12.0
 ---> d2abf6dfbb02
Step 2/2 : MAINTAINER Elico Corp <webmaster@elico-corp.com>
 ---> Running in 3bbcadb7a459
Removing intermediate container 3bbcadb7a459
 ---> 98faa23e0e3a

Successfully built 98faa23e0e3a
Successfully tagged docker-compose-odoo_odoo12:latest
WARNING: Image for service odoo12 was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating docker-compose-odoo_postgres_1 ... done
Creating docker-compose-odoo_odoo12_1   ... done
```

启动完毕后我们就可以通过 `localhost:端口号` 访问 Odoo 服务了：

![Odoo](/images/Odoo/Odoo_docker_sc.png)

如果出现以上画面，那说明你前面的所有配置和成果都生效了，给自己一点掌声👏

数据库的创建和初始化，这里就不多赘述，相信各位一定轻车熟路了，接下来就进入下一个小节的内容吧。

# 获取依赖模块

如果你有仔细查看 Dockerfile 里面的内容的话，你应该发现了我们使用的并不是 Odoo 官方提供的镜像，而是来自 [Elico-Corp/odoo-docker](https://github.com/Elico-Corp/odoo-docker) 的镜像，这个镜像添加了一些实用的脚本，可以自动处理一些依赖和配置，下面我们就来看看都有哪些实用的功能。

在前面的配置中，我们有几个地方都用到了 `$UID`，这是为了解决用户的权限问题，在这个 Odoo 镜像中只需要将 UID 作为环境变量传入即可，详细的描述请查看 [Host user mapping and volumes](https://github.com/Elico-Corp/odoo-docker#host_user_mapping) 这一节的内容。

在 `odoo12.env` 中，有一个变量 `ADDONS_REPO`，它的值为模块的仓库地址，只要填写了该变量，Odoo 容器就会在启动的时候自动拉取对应的仓库。除此之外，还有更方便的特性，现在以一个简单的例子进行说明：

- 假设你有一个模块 Sale 存放在仓库 sale_repo 中
- Sale 模块需要一个叫做 Product 的模块，它存放在 product_repo 仓库中

当你部署的时候，必须找到这两个模块的仓库并分别拉取下来，然后将他们添加到 `addons_path` 中，而现在你有更简单方法来处理这种情况。

你只需要在 Sale 模块的仓库根目录下添加一个 `oca_dependencies.txt` 文件，并将其所依赖的模块按照如下格式每行一个添加进去即可：

```shell
# 模块名 模块仓库地址 模块的分支
product https://github.com/<Username>/product_repo.git 12.0
```

如果 Product 也有依赖其他仓库的模块呢？只需要像上面那样，添加 `oca_dependencies.txt` 即可，这个镜像中的脚本会自动遍历所有仓库下的依赖并自动将它们拉取下来。而在拉取完成之后，脚本还会自动将这些模块添加到 `addons_path` 中，你完全不必自己手动拉取和配置模块的路径。

有时候你并不想在每次启动容器时都自动拉取这些模块，这个时候 `FETCH_OCA_DEPENDENCIES` 变量就可以帮到你了，只要将它设置为 `False`，在启动容器的时候脚本将会跳过拉取依赖这一步，通常建议你在第一次拉取完之后将它关闭，这样可以提高 Odoo 服务的启动速度。

⚠️ **注：使用这种方式自动拉取和添加模块目录到 addons_path 中，要求仓库的目录结构应当遵循 OCA 的规范，模块应该放置在仓库目录下与 oca_dependencies.txt 同级**

该镜像还能在你提供了正确的私钥后，拉取私有仓库中的内容。在拉取私有仓库前，必须生成一个不带密码的 SSH key，并将公钥添加到你的 GitHub 账户中，然后将其添加到对应项目下的 volumes/$module/ssh/ 中，请确保其命名为 `id_rsa`，在容器启动时将会自动扫描该密钥并将其用于私有仓库的拉取。

# 运行更多的 Odoo 实例

当你有更多的项目要运行在不同的 Odoo 实例中时，你可以直接在 `docker-compose.yml` 中添加新的配置，就像前面所描述的一样。例如我要新增一个 Odoo 10 的实例：

```yaml
version: '3.3'
services:

  postgres:
    image: postgres:9.5
    ports:
      - 127.0.0.1:22700:5432
    volumes:
      - ./volumes/postgres:/var/lib/postgresql/data
      - /etc/passwd:/etc/passwd:ro
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=6a6bf96d9d81792e8cdcabd232e39e27
    user: 501:501
    restart: always

  odoo12:
    build: ./build/odoo12
    command: start
    ports:
      - 127.0.0.1:22869:8069
      - 127.0.0.1:22872:8072
    links:
      - postgres:db
    volumes:
      - ./volumes/odoo12/addons:/opt/odoo/additional_addons
      - ./volumes/odoo12/filestore:/opt/odoo/data/filestore
      - ./volumes/odoo12/sessions:/opt/odoo/data/sessions
      - ./volumes/odoo12/ssh:/opt/odoo/ssh:ro
    env_file:
      - ./env/odoo12.env
    restart: always

  odoo10:
    build: ./build/odoo10
    command: start
    ports:
      - 127.0.0.1:23869:8069
      - 127.0.0.1:23872:8072
    links:
      - postgres:db
    volumes:
      - ./volumes/odoo10/addons:/opt/odoo/additional_addons
      - ./volumes/odoo10/filestore:/opt/odoo/data/filestore
      - ./volumes/odoo10/sessions:/opt/odoo/data/sessions
      - ./volumes/odoo10/ssh:/opt/odoo/ssh:ro
    env_file:
      - ./env/odoo10.env
    restart: always
```

目录结构应该是这样的：

```shell
.
├── build
│   ├── odoo10
│   │   └── Dockerfile
│   └── odoo12
│       └── Dockerfile
├── docker-compose.yml
├── env
│   ├── odoo10.env
│   └── odoo12.env
└── volumes
    ├── odoo10
    │   ├── addons
    │   ├── filestore
    │   ├── sessions
    │   └── ssh
    ├── odoo12
    │   ├── addons
    │   ├── filestore
    │   ├── sessions
    │   └── ssh
    └── postgres
```

在其他的 Odoo 实例中，我们都共用同一个 PostgreSQL 服务，只需要为不同的 Odoo 服务创建不同的数据库即可，就像在本地安装使用时一样，你并不会为每一个 Odoo 实例单独安装一个 PostgreSQL.

上面出现的配置你可以在仓库 [ruter/docker-compose-odoo](https://github.com/ruter/docker-compose-odoo) 中找到，克隆到本地后直接修改里面的部分参数即可直接使用，别忘了替换 `$UID` 为你本地用户的 UID 哦😯

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** https://ruterly.com/2019/05/01/Odoo-with-Docker/
