---
layout: post
title: "MacOS10.9上执行Eclipse3.7报错"
date: 2014-01-12 00:13
comments: true
categories: 
- 技海拾贝
- 工具
tags:
- Eclipse
- Mac OS
---

在Mac OS升级到10.9之后  
使用Eclipse3.7时会报错误

    Failed to create the Java Virtual Machine

<!-- more -->

原因为是:

    /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home/bundle/Libraries/libserver.dylib

    JavaVM FATAL: Failed to load the jvm library.

只需在Eclipse.app/Contents/Info.plist中增加下面的内容后，
重启Eclipse便可以正常了：

    <string>-vm</string>
    <string>/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home/jre/lib/server/libjvm.dylib</string>

如果没有解决问题，可进行如下操作 
重置一下Eclipse.app：

    cp -r Eclipse.app Eclipse.app.bak
    rm -r Eclipse.app
    mv Eclipse.app.bak Eclipse.app

以上！