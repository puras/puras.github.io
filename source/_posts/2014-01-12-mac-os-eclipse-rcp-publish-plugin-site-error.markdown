---
layout: post
title: "MacOS 10.9下EclipseRCP发布PluginSite时错误解决"
date: 2014-01-12 00:08
comments: true
categories: 
- 技海拾贝
- 工具
tags:
- Eclipse
- RCP
- Mac OS
---

### 背景
MacOS 10.9下开发Eclipse的插件 
在发布Site时，Eclipse报错 
导致无法正常生成。

<!-- more -->

### 错误
下面的是错误信息：


    /Users/puras/workspace/eclipse-rcp/.metadata/.plugins/org.eclipse.pde.core/temp/org.eclipse.pde.container.feature/compile.org.eclipse.pde.container.feature.xml:4: The following error occurred while executing this line:

    /Users/puras/workspace/eclipse-rcp/mk-plugin/net.mooko.eclipse.plugin/build.xml:31: /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home/Classes does not exist.

    The following error occurred while executing this line:

    /Users/puras/workspace/eclipse-rcp/mk-plugin/net.mooko.eclipse.plugin/build.xml:31: /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home/Classes does not exist.

### 解决办法

根据错误描述，是找不jdk下的Contents/Home/Classes这个目录 
那在相应的目录下，创建该目录，便可解决该问题 
如在下面的目录中创建Classes：

    /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home

经测试，创建目录后，可正常发布Site了。