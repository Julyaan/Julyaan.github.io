---
title: nodeJS读写文件中文乱码问题整理及计算机文件编码方式科普
date: 2017-04-19 16:30:46
tags:
- node
- 前端
- mongoDB
categories: node
---
最近在用node 做一个读取excel文件的项目，后台用mongoDB存储。由于文件默认编码方式是anti，汉字总是无法显示。
网上都说要转成无BOM的utf-8格式，我用notepad++转了，还是没用。后来读文件时我也把编码方式选为‘utf-8’了也不行（后来明白，是我对这里所有的‘设置编码方式’的理解有误，并不是把编码方式‘改’为utf-8的意思）。

<!--more-->
--------------

最后参考了这三遍文章，问题得到解决。

-  [ANSI、Unicode、UTF-8 这三种编码模式区别](https://www.zhihu.com/question/20650946)
- [node fs模块--文件操作-接口说明](http://www.cnblogs.com/mangoxin/p/5664615.html)
- [node iconv-lite 模块的使用](http://blog.csdn.net/youbl/article/details/29812669)

-------------------
为什么要使用最后那个模块？

由于Node.js仅支持如下编码：utf8, ucs2, ascii, binary, base64, hex，并不支持中文GBK或GB2312之类的编码，因此如果要读写GBK或GB2312格式的文件的中文内容，必须要用额外的模块：iconv-lite。

------

ANSI和GBK什么关系？

ANSI包含GBK！阅读ANSI[百度百科](http://baike.baidu.com/link?url=twZ2lLhhdl5cfDIeiSWRJIZW3VE0S7JeYmBa-ff_lPr6ICP88xi-4ZuJNvLLT1qQFaohzndTQRKq0Rn-FxZKZmpDfUN4mcd6EcjTUKj5bJq)

