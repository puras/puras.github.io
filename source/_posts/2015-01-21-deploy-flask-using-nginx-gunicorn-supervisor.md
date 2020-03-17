title: 使用Nginx、Gunicorn和Supervisor部署Flask应用
date: 2015-01-21 23:23:35
comments: true
categories: 
- 技海拾贝
- Python
tags:
- Python
- Flask
- Nginx
- Gunicorn
- Supervisor
---

前两天研究了一下使用uWSGI来部署Flask应用，只是在使用Supervisor时遇到了一些问题，一直没解决掉。看网友说现在使用Gunicorn比较流行，OK，我也换一种方式尝试一下。

<!-- more -->

## 前提条件

更新Ubuntu：

    sudo apt-get update && sudo apt-get upgrade -y

安装Nginx：

    ./configure --prefix=/home/puras/apps/nginx
    make && make install

安装pip和virtualenv:

    sudo apt-get install python-pip
    sudo pip install virtualenv

创建虚拟环境，并安装相应的库：

    cd /path/to/your/app
    virtualenv .env --no-site-packages
    source .env/bin/activate

    pip install flask


## 使用Gunicorn

在虚拟环境中安装Gunicorn:

    pip install gunicorn

通过gunicorn启动app：

    gunicorn -w 4 -b 127.0.0.1:5000 main:app

在工程目录下创建Gunicorn的配置文件gunicorn.conf:

    workers = 2
    bind = '127.0.0.1:8000'
    proc_name = 'moowo-web'
    pidfile = '/tmp/moowo-web.pid'

## 使用Nginx

配置Nginx，在vhosts目录中创建一个带有域名的配置文件，如moowo.com:

    server {
        listen 80;
        server_name www.moowo.com moowo.com;

        root /home/puras/mooko/moowo/moowo-web;
        index index.html;

        location / {
            proxy_pass       http://127.0.0.1:8000/;
            proxy_redirect   off;
            proxy_set_header Host            $host;
            proxy_set_header X-Real-IP       $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        if ($host !~ ^(moowo.com|www.moowo.com)$ ) {
            return 444;
        }
    }

## 使用Supervisor

先安装Supervisor：

    sudo apt-get install supervisor

在/etc/supervisor/conf.d下增加配置文件，名为moowo.conf,文件名可根据需求替换：

    [program:moowo-web]
    command=/home/puras/proj/moowo/.env/bin/gunicorn -c /home/puras/proj/moowo/moowo
    -web/gunicorn.conf main:app
    directory=/home/puras/proj/moowo/moowo-web
    user=puras
    autostart=true
    autorestart=true
    startsecs = 10
    redirect_stderr = true
    stdout_logfile = /home/puras/proj/moowo/moowo-web/moowo.log
    loglevel = info
    environment = MODE="PRODUCTION"

通过supervisorctl更新配置文件，并查看状态：

    sudo supervisorctl
    supervisorctl > update
    supervisorctl > status

启动没有问题，但是进程被杀掉是否能自动重启的效果暂时没有测试，待测。


