---
title: Hexo博客建站手记--CSS全屏背景图片加载优化方案
date: 2017-09-04 13:51:36
tags:
- hexo
- CSS
- 性能优化
- 渐进式
categories: 前端
---
# 问题描述

在建站之初，我用的随机背景图片作为整个网站的背景，再加上调用的api:unsplash.com（ [一个免费高清素材网站](https://zhuanlan.zhihu.com/p/20383705) ）逼格极高，所以显得整个网站即文艺又酷炫，但是由于图片尺寸较大，在网络环境一般的情况下，首屏用户体验不是很好。具体是什么样的呢？

<img src='/images/xiamen.jpg' align=center>

<!--more-->

我从segmentfault上找了个图，情况类似：
<img src='https://sfault-image.b0.upaiyun.com/403/232/4032326478-598f2e6002876_articlex' alt='IMG' align=center>

再进行改造之后，期望是这样的：

<img src='https://sfault-image.b0.upaiyun.com/752/451/752451209-598fe52c0b089_articlex' alt='IMG' align=center>

# 可行性方案

web图片加载优化的思路很多，这里不是讨论的重点，贴一个比较好的总结贴吧：

> - [web前端图片极限优化策略](http://jixianqianduan.com/frontend-weboptimize/2015/11/17/front-end-image-optmize.html)

这里可以采取的解决方法主要有两个

1. 采用渐进式jpeg图像
2. 在图片onload之后设为背景

# 使用渐进式JPEG来提升体验

JPEG文件有两种保存方式：Baseline JPEG（标准型）和Progressive JPEG（渐进式）。两种格式有相同尺寸以及图像数据，他们的扩展名也是相同的，唯一的区别是二者显示的方式不同。可以通过Photoshop进行修改

- Baseline JPEG

    这种类型的JPEG文件存储方式是按从上到下的扫描方式，把每一行顺序的保存在JPEG文件中。打开这个文件显示它的内容时，数据将按照存储时的顺序从上到下一行一行的被显示出来，直到所有的数据都被读完，就完成了整张图片的显示。

    <img src='https://www.biaodianfu.com/wp-content/uploads/2013/07/baseline.gif' alt='IMG' align=center>

- Progressive JPEG

    和Baseline一遍扫描不同，Progressive JPEG文件包含多次扫描，这些扫描顺寻的存储在JPEG文件中。打开文件过程中，会先显示整个图片的模糊轮廓，随着扫描次数的增加，图片变得越来越清晰。

    <img src='https://www.biaodianfu.com/wp-content/uploads/2013/07/progressive.gif' alt='IMG' align=center>

由于这里我请求的图片来自网络API所以这种方法及暂时不可取了，下面重点讨论下第二种方法。

# 图像onload之后插入背景

## 随机背景图设置方法

先贴一下本站设置随机背景图的方法，很简单，为body加一个这样的css即可
```css
body {
  background: url(https://source.unsplash.com/random/1600x900);
  background-repeat: no-repeat;
  background-attachment:fixed;
  background-size: cover;
  background-position:50% 50%;
}
```

## onload显示背景图

原理也很简单，三步走起：
1. 设置一个默认的背景色
2. 把背景图放置在沙盒中加载
3. 图片加载完后把背景图拿出来设置为dom中节点的backgroundImage

重点就在如何把背景图和dom节点联结起来。这里也有两种方案，一种比较优雅，但是实施起来会繁琐些，另一种很接地气，用js就可以搞定

### 优雅的方式：IOING

使用IOING(什么是IOING? [一种渐进式Web App 前端开发引擎](http://ioing.com/#docs-started-ioing/) )：IOING 的 DOM 引擎 和 CSS 引擎可以轻松的实现上述功能
```css
.bg {
    background: #fff onload url(./bg.png) center no-repeat;
}
```
这种方法有个很大的缺点就是不能用在body标签上
### 简洁的方式: Image元素构造器

上面那种方法虽然优雅，但是成本未免高了些。因此我自己倒腾了下，算式实现了这个功能，在index的body最后加段script：
```JavaScript
(function(){
    var imageBg = new Image();
      imageBg.onload = function () {
          document.body.style.backgroundImage= 'url(https://source.unsplash.com/random/1600x900)'
        }
      imageBg.src = 'https://source.unsplash.com/random/1600x900'
  })()
```
这里的'沙盒'就是有js创建的Image元素，然后在onload事件中设置背景，由于两次请求的url完全相同，因此浏览器只会发出一次请求，即不会增加新的网络开销，也完成了整图加载的需求。

