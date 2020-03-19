---
title: 一起用Docker之用Nexus搭建本地Docker私有仓库
date: 2020-03-18 16:29:52
categories: 
- 技海拾贝
- 一起用Docker
tags:
- Docker
- Nexus
- 私服
---

使用Docker的过程中，我们不停的在拉取各种类型的镜像，我们自己制作的镜像，发布到`hub.docker.io`上，也可以供他们下载了。但是一些内部的镜像，比如说我们开发的Spring Boot的程序，想交给测试同学进行验证，这种情况怎么办呢？也发布到`hub.docker.io`上么？先不说是否适合，只是`hub.docker.io`的速度，就够咱们喝一壶的了。

<!-- more -->

今天要说的，是上一篇[一起用Docker之搭建开发与测试的依赖环境](http://puras.cn/2020/03/17/%E4%B8%80%E8%B5%B7%E7%94%A8Docker%E4%B9%8B%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E4%B8%8E%E6%B5%8B%E8%AF%95%E7%9A%84%E4%BE%9D%E8%B5%96%E7%8E%AF%E5%A2%83/)中提到的Nexus。

Nexus不仅可以做为Maven的私服使用，也可以做为Docker的私有仓库来使用，这里只做简单的使用介绍，更详细的可以查看[官方介绍](https://help.sonatype.com/repomanager3)。

# 创建私有仓库

登录到Nexus管理界面，通过导航依次选择`Repository->Repositories`，在`Repositories`界面中，选择`Create repository`，可以看到下面的界面：

![](/images/docker-repository-select.png)

在其中选择`docker(hosted)`，可以打开创建页面：

![](/images/docker-repository-create.png)

在这个界面中设置如下内容：

- 设置HTTP端口，比如8082，还记得之前运行Nexus容器时，映射的端口么，8082就用在这了；
- `Allow anonymous docker pull`是否允许匿名Pull；
- `Allow clients to use the V1 API to interact with this repository`允许V1版本的API与该库交互；

设置完，点保存就可以啦。

# 设置Docker

为了让我们能访问Docker的私服，需要对Docker进行一些设置，修改`/etc/docker/daemon.json`文件，当然，这个是Linux系统下的，其他的系统，可参考官网给出的位置，增加如下内容：

     "insecure-registries": [
        "10.211.55.6:8082"
      ]

因为我们搞的私服，是不可靠的，所以需要将服务器的信息，加入到配置文件中，修改完，重启一下服务就好了：

    sudo systemctl daemon-reload
    sudo systemctl restart docker

# 登录私有仓库

之前我们执行`docker login`命令，是登录的`hub.docker.io`，但现在我们不登录啦，换成咱们自己的私服：

    docker login ip:port
    输入用户名
    输入密码
    显示登录结果信息

下面是登录刚搭建的Nexus私服的信息，其中IP是10.211.55.6，端口是8802：

    puras@ubuntu-server:~$ docker login 10.211.55.6:8082
    Username: puras
    Password: 
    Error response from daemon: login attempt to http://10.211.55.6:8082/v2/ failed with status: 401 Unauthorized
    puras@ubuntu-server:~$ docker login 10.211.55.6:8082
    Username: puras
    Password: 
    WARNING! Your password will be stored unencrypted in /home/puras/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store

    Login Succeeded

# 发布镜像

登录到私服之后，就可以进行镜像的Push啦，拿我们之后要讲到的SpringBoot应用来做示例，先查查私服上有没有它：

    docker search 10.211.55.6:8082/spring-boot
    // 应该是啥也没有，因为还没有上传过呢

再看看本地的镜像：

    puras@puras-macbook:~$ docker image list
    REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
    puras/docker-spring-boot   0.1.0               ad0a319215d0        23 hours ago        122MB
    redis                      5                   f0453552d7f2        5 days ago          98.2MB
    mysql                      5.7                 84164b03fa2e        2 weeks ago         456MB
    nginx                      1.16                8c5ec390a315        3 weeks ago         127MB
    openjdk                    8-jdk-alpine        a3562aa0b991        10 months ago       105MB

嗯，其中的puras/docker-spring-boot就是我们今天要Push的对象啦，先给它做一个Tag，需要将私服的地址加上的：

    docker tag puras/docker-spring-boot:0.1.0 10.211.55.6:8082/docker-spring-boot:0.1.0

再查看本地的镜像

    puras@puras-macbook:~$ docker image list
    REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
    10.211.55.6:8082/docker-spring-boot   0.1.0               ad0a319215d0        23 hours ago        122MB
    puras/docker-spring-boot              0.1.0               ad0a319215d0        23 hours ago        122MB
    redis                                 5                   f0453552d7f2        5 days ago          98.2MB
    mysql                                 5.7                 84164b03fa2e        2 weeks ago         456MB
    nginx                                 1.16                8c5ec390a315        3 weeks ago         127MB
    openjdk                               8-jdk-alpine        a3562aa0b991        10 months ago       105MB

新的Tag的镜像已经出现了，细心的朋友一定会注意到，两个ImageId是相同的，删除的时候，千万不要使用ImageId，直接使用ImageName就好了，不然同ImageId的镜像会都被删除掉的哦。

好了，现在将镜像Push到私服上吧：

    docker push 10.211.55.6:8082/docker-spring-boot:0.1.0
    The push refers to repository [10.211.55.6:8082/docker-spring-boot]
    f763aecd8e8d: Pushed
    ceaf9e1ebef5: Pushed
    9b9b7f3d56a0: Pushed
    f1b5933fe4b5: Pushed
    0.1.0: digest: sha256:ec6041935fde6a4b32295af56fc72a8e3c12cf0118e08081db3797fda6a42a59 size: 1159

再来查看私服里，是否已经有了新的镜像呢：

    puras@puras-macbook:~$ docker search 10.211.55.6:8082/docker-spring-boot:0.1.0
    NAME                                        DESCRIPTION         STARS               OFFICIAL            AUTOMATED
    10.211.55.6:8082/docker-spring-boot:0.1.0

嗯，已经有了，现在将本地的镜像删除，直接按ImageId删除吧，一个也不要了：

    docker rmi -f ad0a319215d0

再查看一下：

    puras@puras-macbook:~$ docker image list
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    redis               5                   f0453552d7f2        5 days ago          98.2MB
    mysql               5.7                 84164b03fa2e        2 weeks ago         456MB
    nginx               1.16                8c5ec390a315        3 weeks ago         127MB
    openjdk             8-jdk-alpine        a3562aa0b991        10 months ago       105MB


没有了，现在从私服同步一个刚刚上传的镜像：

    docker pull 10.211.55.6:8082/docker-spring-boot:0.1.0


运行新同步的镜像：

    docker run -p 8080:8080 10.211.55.6:8082/docker-spring-boot:0.1.0


然后打开浏览器，看看执行的效果：

![](/images/docker-run.png)

OK，至此私服已经可用了。

推荐的理由：

- 有了自己的私服，发布和拉取，保证隐私的情况下还会很方便
- 开发将完整的应用做成镜像，测试直接拉取运行便可，只需要有一个Docker环境就好了，不需要其他的环境

PS：一步步走向开发测试使用Docker环境大统一的方向~~~

***

一点小小的想法，希望能对大家有用。

Over~~~