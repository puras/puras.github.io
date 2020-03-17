title: 使用Virtualenv,UWSGI和Nginx来搭建Flask
date: 2015-01-19 22:51:35
categories: 
- 技海拾贝
- Python
tags:
- Python
- Flask
- Nginx
- uWSGI
---

想在新的项目中应用Flask，跟以往的Java的环境不太一样，所以调研了一下服务器的搭建过程。以下是详细步骤的记录。

<!-- more -->

## 更新系统

先更新Ubuntu系统：

    sudo apt-get update
    sudo apt-get upgrade

## 安装Nginx

下载源代码进行编译安装：

    ./configure --prefix=/home/puras/apps/nginx
    make && make install

注：新的Ubuntu系统，编译安装可能会遇到一些问题，可查询之前的文章，里面有解决办法。

## 安装PIP和Virtualenv

    sudo apt-get python-pip
    sudo pip install virtualenv

## 创建Virutalenv

进入工程目录，通过下面的命令来创建虚拟环境

    virtualenv .env --no-site-packages
    source .env/bin/activate

在虚拟环境中安装Flask：

    pip install flask

## 安装uwsgi

    sudo apt-get install uwsgi uwsgi-plugin-python

## 配置Nginx

先配置Nginx成多域名，需要对默认的配置文件进行一些修改，将默认配置文件中的Server修改成以下内容：

    server {
        listen       80;
        server_name  _;
        server_name_in_redirect off;
 
        location / {
        root /home/puras/mooko/php;
        index index.html;
        }
    }
    include /home/puras/apps/nginx/conf/vhosts/*; 

在nginx/conf下创建一下vhosts目录，存放新的配置文件。

在vhosts目录中创建一个带有域名的配置文件，如moowo.com:

    server {
        listen 80;
        server_name www.moowo.com moowo.com;

        root /home/puras/mooko/moowo/moowo-web;
        index index.html;

        location / {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:5000;
        }

        if ($host !~ ^(moowo.com|www.moowo.com)$ ) {
            return 444;
        }
    }

指定域名和根目录，并将请求转发至指定的目录。

## uWSGI配置文件

在工程目录下创建uwsgi_config.xml文件，内容如下：

    <uwsgi>
        <virtualenv>/home/puras/mooko/moowo/moowo-web/.env</virtualenv>
        <pythonpath>/home/puras/mooko/moowo/moowo-web</pythonpath>
        <module>main</module>
        <callable>app</callable>
        <socket>127.0.0.1:5000</socket>
        <master/>
        <processes>5</processes>
        <memory-report/>
        <buffer-size>32768</buffer-size>
        <plugins>python</plugins>
    </uwsgi>

其中：

* virtualenv：Virtualenv的目录
* pythonpath：Flask工程目录
* module：要执行的文件
* callable：Flask的App名称
* processes：要启动的线程

其他默认便可。之后启动uWSGI：

    uwsgi uwsgi -x uwsgi_config.xml -d uwsgi.log

浏览器里访问：http://moowo.com，正常情况下便可以看到Flask应用的页面了。

注：moowo.com不是我的域名，我是通过修改hosts做的映射，^_^