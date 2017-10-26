---
title: 同态加密开源库Helib使用方式
date: 2017-03-08 18:18:57
tags:
- 密码学
- 同态加密
- 信息安全
categories: 信息安全
---


全同态算法的编译和测试（BGV方案）

# 系统：
windows7及以上

# 配置软件包：
- 安装dev-C++，下载地址：http://yunpan.cn/cy3G9PJbJwBtn提取码 513a
- 下载HElib文件包，下载地址：http://yunpan.cn/cy3GDLkzik6r7提取码 a7de
- 下载WinNTL-6_2_1包，下载地址：http://yunpan.cn/cy3GzxTG69AtT提取码 fb6b
- 下载HElib.a文件，下载地址：http://yunpan.cn/cy3GpK43iScwf提取码 87b0
- 下载NTL-6_2_1.a文件，下载地址： http://yunpan.cn/cy3GpiNMKgHpH提取码 77ce

<!--more-->
# 配置方法：

1、 下载安装dev-C++；

2、 在D盘或者其他位置建立文件夹bgv；

3、 将刚才下载的四个文件包拷入bgv文件夹中，并将WinNTL-6_2_1和HElib解压；

4、 文件——项目——新建项目——ConsoleApplicaion，输入项目名称test，点击确定，选取bgv文件件，点击保存，继而会同时出现一个CPP文件，命名为main.cpp；

5、 项目——项目属性——参数——加入库或者对象，将HElib.a和NTL-6_2_1.a加入；

6、 继而点击 文件/目录——包含文件目录，将HElib\src和WinNTL-6_2_1\include加入目录中，然后点击确定；

7、之后便可以测试BGV全同态算法库了

Linux下安装环境比较简单，首先安装gmp，然后安装NTL，之后直接make便可以了，gmp和NTL的Linux安装可以百度，很容易解决！