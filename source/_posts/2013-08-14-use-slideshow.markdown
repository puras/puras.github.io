---
layout: post
title: "使用SlideShow"
date: 2013-08-14 16:29
comments: true
categories: 
- 技海拾贝
- Other
---

一直以来  
都觉得做PPT是件痛苦的事  
更痛苦的是做完之后  
还要到处的复制  

<!-- more -->

以前也用过一些在线PPT  
像Impress.js这样的  
不过制作起来比较麻烦  
昨天在邮件列表中  
看到有人发的一个在线的Go-Lang的在线PPT  
是用SlideShow-S9制作的  
以下简称S9
觉得很不错  
赶紧试用一下  
觉得就是自己要找的东西  

S9可以直接使用Markdown、textile等直接编写PPT  
再编译成HTML代码  
这样就不需要到处的复制  
不需要担心PowerPoint的版本不同而效果不同  
只需要将HTML代码上传到服务器  
就可以一处编写  
到处查看啦  

要使用S9也非常的简单  
首先你要有Ruby环境  
之后直接通过GEM进行安装便可以了  

    gem install slideshow

安装完成后便可直接使用了  

如果遇到提示没有slideshow这个命令  
可以将slideshow加到PATH中  
这样就可以正常使用了  

写好Markdown文档  
通过命令生成HTML代码  

    slideshow build xxx

也可以下载一些现有的模板  
来美化你的PPT  

比如下载并使用我喜欢的HTML5的模板  

    slideshow install g5
    slideshow build xxx -t g5

初次体验是不错的  
以后也要多多使用  

S9官网迁移到了Github  
地址是[SlideShow](http://slideshow-s9.github.io/)