---
title: Application Cache 无法加载其他静态资源的问题
date: 2017-04-28 17:39:56
tags:
- Application Cache
- HTML5
- node
categories: node
---

今天在自己做的项目中加了Application Cache 功能，把一些js文件放在了列表里。然后第一次加载没问题，我原来是这样写的：把NETWORK和FALLBACK都去掉了。以为这个清单的作用就是 如果查到，就存入本地，下次从本地加载，如果未查到就从服务器上下载：
<!--more-->
```
CACHE MANIFEST

#需要缓存的列表 v2017-4-28
\assets\js\demo.js
\assets\js\bootstrap-notify.js
\assets\js\jquery.min.js
```
实践证明，我的想法是错的。第二次加载时我发现其他的所有静态资源都不发加载了，原来 必须指定NETWORK,于是我做了这样的修改：
```
CACHE MANIFEST

#需要缓存的列表  v2017-4-28
\assets\js\demo.js
\assets\js\bootstrap-notify.js
\assets\js\jquery.min.js

NETWORK:
*
```

可以了。

附：[HTML5应用程序缓存Application Cache](http://www.cnblogs.com/yexiaochai/p/4271834.html)