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

今天要说的，是上一篇[一起用Docker之搭建开发与测试的依赖环境](http://puras.cn/2020/03/17/%E4%B8%80%E8%B5%B7%E7%94%A8Docker%E4%B9%8B%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E4%B8%8E%E6%B5%8B%E8%AF%95%E7%9A%84%E4%BE%9D%E8%B5%96%E7%8E%AF%E5%A2%83/)中提到的Nexus。

Nexus不仅可以做为Maven的私服使用，也可以做为Docker的私有仓库来使用，这里只做简单的使用介绍，更详细的可以查看[官方介绍](https://help.sonatype.com/repomanager3)。

# 创建私有仓库

登录到Nexus管理界面，通过导航依次选择`Repository->Repositories`，在`Repositories`界面中，选择`Create repository`，可以看到下面的界面：

![](/images/docker-repository-select.png)

在其中选择`docker(hosted)`，可以打开创建页面：

![](/images/docker-repository-create.png)