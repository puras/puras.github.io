title: CodeIgniter中Session的异常与解决办法
date: 2014-12-20 00:02:37
comments: true
categories: 
- 技海拾贝
- PHP
- CodeIgniter
tags:
- PHP
- CodeIgniter
---

网上说 有如下可能引起此问题:
1. 本地时间与服务器时间不一致,导致session过期
2. session 里面存在中文
3. POST请求之后不能使用 302来重定向,必须使用307
4. 人品问题

使用原生的Session替换框架的Session。