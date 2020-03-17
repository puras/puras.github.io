title: MacOS下安装LXML
date: 2015-02-11 15:23:29
comments: true
categories: 
- 技海拾贝
- Python
tags:
- Python
- lxml
---

这些天在写微信公众号平台，需要解析XML数据。因为是刚刚着手使用Python来解决问题，所以一般的东西都是在网上查询的。

在网上查了一下，说是LXML解析XML效率比较好，便准备使用它了。没想到它的安装和其他的库不一样，略有波折。

<!-- more -->

没有使用Brew等工具，纯手工安装。

使用了VirtualEnv，在使用下面的命令安装lxml：
    
    pip install lxml

未能安装成功，提示xmlversion.h这个文件没有找到。

但是查找了一下：

    xxx:/$sudo find . -name xmlversion.h
    Password:
    ./Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/usr/include/libxml2/libxml/xmlversion.h
    ./Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/usr/include/libxml2/libxml/xmlversion.h
    ./Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator7.1.sdk/usr/include/libxml2/libxml/xmlversion.h
    ./Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/libxml2/libxml/xmlversion.h
    ./Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.9.sdk/usr/include/libxml2/libxml/xmlversion.h

发现是可以找到该文件的，在网上查找了一下，是因为Mac系统中的该文件未被引用到路径中，所以在使用pip安装的时候，只需要将其引用进来便可以了，于是有发如下的命令：

    C_INCLUDE_PATH=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/libxml2:/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/libxml2/libxml:/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include pip install lxml

可以安装成功。

记录一下，以备后用……