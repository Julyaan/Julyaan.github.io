---
title: 用NodeJS打造静态文件服务器
date: 2017-04-28 18:44:18
tags:
- node
- 服务器
categories: node
---

在《The Node Beginner Book》的 [中文版](http://nodebeginner.org/index-zh-cn.html) 发布之后，获得国内的好评。也有同学觉得这本书略薄，没有包含进阶式的例子。@otakustay同学说：“确实，我的想法是在这之上补一个简单的MVC框架和一个StaticFile+Mimetype+CacheControl机制，可以成为一个更全面的教程”。正巧的是目前我手里的V5项目有一些特殊性：

- 项目大多数的文件都是属于静态文件，只有数据部分存在动态请求
- 数据部分的请求都呈现为RESTful的特性

<!--more-->

本文放在原 [CSDN博客](http://blog.csdn.net/u010582082/article/details/70922730) ，暂不做迁移