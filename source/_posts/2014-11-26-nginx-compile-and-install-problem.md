title: Ubuntu下Nginx编译与安装过程中的问题及解决
date: 2014-11-26 09:14:58
categories: 
- 技海拾贝
- Nginx
tags:
- Ubuntu
- Nginx
---

安装过多次了，但每次遇到这些问题都需要去Google，之后便记不得了。虽然每次都可以解决，但还是记下来的比较好，以后再安装，也可以回来查一下。

本文主要记录PCRE和zlib库的安装问题。

<!-- more -->

## PCRE

在执行configure命令时，如果未安装PCRE库，会遇到如下问题：

    ``` [shell] [PCRE错误]
    ./configure: error: the HTTP rewrite module requires the PCRE library.
    You can either disable the module by using --without-http_rewrite_module
    option, or install the PCRE library into the system, or build the PCRE library
    statically from the source with nginx by using --with-pcre=<path> option.
    ```

可以通过下面的命令安装PCRE库，来解决问题：

    ``` [shell] [PCRE错误解决办法]
    sudo apt-get install libpcre3 libpcre3-dev
    ```

## zlib

在执行configure命令时，如果未安装zlib库，会遇到如下问题：

    ``` [shell] [zlib错误]
    ./configure: error: the HTTP gzip module requires the zlib library.
    You can either disable the module by using --without-http_gzip_module
    option, or install the zlib library into the system, or build the zlib library statically from the source with nginx by using --with-zlib=<path> option.
    ```

可以通过下面的命令安装zlib的相关库，来解决问题：

    ``` [shell] [zlib错误解决办法]
    sudo apt-get install zlibc zlib1g zlib1g-dev
    ```

## 安装路径

如果不指定安装路径，默认会将nginx安装到/usr/local/nginx处。如果想修改nginx的安装位置，可以通过--prefix参数指定，如：

    ``` [shell] [指定安装目录]
    ./configure --prefix=/home/puras/apps/nginx
    ```

注意不要使用~/apps这种形式，会将位置指定到编译目录内。