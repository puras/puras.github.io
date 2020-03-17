---
layout: post
title: "Mac OS 安装与使用Virtualenv和Virtualenvwrapper"
date: 2013-11-14 22:16
comments: true
categories: 
- 技海拾贝
- 工具
tags:
- Python
- Mac OS
---

## 背景

在开发Python应用的时候，经常需要定装一些依赖库 

但有时会遇到不同的应用，所需要的依赖库版本不一致这样的情况 

这时要如何解决呢 

可以借助VirtualEnv 

<!-- more -->


_VirtualEnv用于在一台机器上创建多个独立的python运行环境_

## 环境

* Mac OS 10.9
* Python 2.7.3

## 安装

在Mac OS上安装VirtualEnv很容易，直接使用pip来安装便可。
如果没有安装pip，可以通过eash_install来安装pip。

    sudo easy_install pip

在这里使用pip来安装它。 

    sudo pip install virtualenv

## 使用

### 创建虚拟环境
安装完virtualenv之后，则可以使用： 

    virtualenv [虚拟环境名称] 

命令来创建虚拟环境了

如例

    virtualenv moobo

创建一个名为moobo的虚拟环境

### 启动与退出虚拟环境

    cd moobo
    source ./bin/activate

此时，命令行上为多一个moobo提示，如：

    (moobo)puras@puras-pc:~$


退出虚拟环境则用命令：

    deactivate

安装完VirtualEnv后，便可以直接使用pip来安装依赖包了，

但要注意的是，如果未启动虚拟环境，而且系统也安装了pip，此时会安装到系统环境中

为了避免类似的情况发生，可以在~/.bashrc中添加行：

    export PIP_REQUIRE_VIRTUALENV=true

来强制pip使用虚拟环境

另外在~/.bashrc中添加行：

    export PIP_DOWNLOAD_CACHE=$HOME/.pip/cache

来设置pip的缓存

## VirtualEnvWrapper

但是这样使用虚拟环境有点麻烦，可以使用VirtualEnvWrapper来简化操作 

_VirtualEnvWrapper为前者提供了一些便利的命令行上的封装_

同样使用pip来安装VirtualEnvWrapper：

    pip virtualenvwrapper


### 配置

安装完之后先进行设置后再使用：

* 创建目录来存放虚拟环境

        mkdir $HOME/.virtualenvs

* 在~/.bashrc中添加行：

        export WORKON_HOME=$HOME/.virtualenvs

* 在.bashrc中添加行：

        source /usr/local/bin/virtualenvwrapper.sh

* 运行

        source ~/.bashrc

之后便可以使用Virtualenvwrapper了。

### 使用
* 列出虚拟环境列表

        workon

* 也可以使用

        lsvirtualenv

* 新建虚拟环境

        mkvirtualenv [虚拟环境名称]

* 启动/切换虚拟环境

        workon [虚拟环境名称]

* 关闭虚拟环境

        deactivate

* 删除虚拟环境

        rmvirtualenv [虚拟环境名称]
