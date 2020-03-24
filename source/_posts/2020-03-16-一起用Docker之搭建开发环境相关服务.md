---
title: 一起用Docker之搭建开发环境相关服务
date: 2020-03-16 22:10:10
categories: 
- 技海拾贝
- 一起用Docker
tags:
- Docker
- 开发环境
- MySQL
- Redis
- Nginx
---

如果你现在拿到一台新的工作电脑，第一步要做的是什么？是不是要安装各种软件，进行开发环境的搭建？

在日常的工作中，你是否会遇到下面的场景：

    测试：你的SQL脚本，在测试环境中执行不了。
    开发：不可能的啊，在我机器上是好用的啊，你的MySQL是什么版本
    测试：测试环境的MySQL版本是5.7
    开发：啊，我本地的版本是5.6……

那么有什么好的办法能快速的解决开发环境的问题？且能与测试环境保持同样的版本？或是在同测试环境版本不同的时候，能快速的更新自己的开发环境呢？

<!-- more -->

最直接的办法，一个个的安装嘛，版本不同，把老的版本卸载掉，重新安装匹配的版版就好了。

但是这种办法，效率很低，而且也不能解决多个版本并存的情况。

最近看了一些Docker，想到用Docker来解决开发环境的安装的问题，几个命令，就可以安装好所需要的本地服务，而且可以安装多个不同版本的服务，是不是很爽呢？

本次介绍如何在开发电脑上安装`MySQL`、`Redis`和`Nginx`：

准备工作：
====

- 安装Docker：Docker的安装，比较容易，直接查看官方的文档便可； 
- 拉取相关的镜像，当然，也可以直接在使用`Docker run`命令进行执行，在本地找不到的时候，同样会进行下载，但提前准备好，可以在多次调整容器的配置时快速使用：

拉取MySQL镜像
----

MySQL当前使用5.7版本的较多，不计较小版本的话，可以使用下面的命令进行拉取：

    docker pull mysql:5.7

当然，你也可以拉取当前比较新的8.0版本，或者拉取最新的版本：

    docker pull mysql:8.0
    or 
    docker pull mysql
    or 
    docker pull mysql:latest

拉取Redis镜像
----

Redis同样，拉取需要的版本便可：

    docker pull redis:5

拉取Nginx镜像
----

Nginx同上，拉取需要的版本：

    docker pull nginx:1.16

启动容器
====

提取好相关的镜像，便可以启动容器啦。启动容器比较简单，但为了以后能复用，可以通过将本地目录挂载到容器中，这样的好处，一是可以复用，二是有问题修改也方便。

启动MySQL
----

启动容器：

    docker run \
        -p 3306:3306 \
        --name mysql5.7 \
        -v $PWD/conf:/etc/mysql/conf.d \
        -v $PWD/logs:/logs \
        -v $PWD/data:/mysql_data \
        -v $PWD/lib/mysql:/var/lib/mysql \
        -e MYSQL_ROOT_PASSWORD=123456 \
        -d \
        mysql:5.7

参数说明：

1. -p 将容器的3306端口映射到本机的3306端口上；
1. --name 创建新的容器名为`mysql5.7`；
1. -v 挂载目录；
1. -e 设置参数，比如数据库的`root`账号的密码；
1. -d 后台执行；
1. 最后一行则是使用的镜像名称；

命令执行成功后，会返回一个容器ID，通过`docker ps`可以查看当前正在运行的容器。

可以通过MySQL命令访问一下，进行验证：

    mysql -uroot -p123456 -h127.0.0.1

如果能成功的看到MySQL命令行提示符，则代表安装成功啦。

启动Redis
----

启动容器：

    docker run \
        -p 6679:6679 \
        -v $PWD/conf/redis.conf:/usr/local/etc/redis/redis.conf \
        -v $PWD/data:/data \
        --name redis \
        -d \
        redis:5 \
        redis-server /usr/local/etc/redis/redis.conf

 参数说明：

1. -p 将容器的6679端口映射到本机的6679端口上；
1. -v 挂载目录；
1. --name 创建新的容器名为`redis`；
1. -d 后台执行；
1. 是使用的镜像名称；
1. 通过指定的配置文件启动Redis； 

启动Nginx
----

启动容器：

    docker run \
    -p 80:80 \
    -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v $PWD/logs:/var/log/nginx \
    -v $PWD/html:/usr/share/nginx/html \
    -d\
    --name nginx \
    nginx:1.16

参数说明：

1. -p 将容器的80端口映射到本机的80端口上；
1. -v 挂载目录及文件；
1. -d 后台执行；
1. --name 创建新的容器名为`nginx`；
1. 是使用的镜像名称；

启动成功后，打开浏览器访问`http://localhost`，看到Nginx的默认欢迎界面，则代表成功啦。


推荐的理由
====

1. 可以快速的安装开发环境；
1. 比较容易的解决多版本并存的问题；
1. 测试环境使用同样的方式，可以保证开发与测试环境相同；

***

一点小小的想法，希望能对大家有用。

Over~~~
