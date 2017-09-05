---
title: 通过chrome调试器测试了解浏览器解析和渲染HTML的过程
date: 2017-04-25 16:34:09
tags:
categories: 前端
---

# 1.基础知识：了解chrome的Timeline工具

仅仅是通过理论知识，很难记住和理解浏览器解析html的原则，因此我动手做了些小实验。而做这个实验，不得不用到一个工具：chrome的Timeline工具。

这个工具真的很强大，Timeline工具栏提供了对于在装载Web应用的过程中，时间花费情况的概览，这些应用包括处理DOM事件, 页面布局渲染或者向屏幕绘制元素。Timeline可以通过事件，框架，和实时内存用量3个方面的数据来监测网页，通过这些数据，我们可以方便的找出页面中存在问题的地方。
![这里写图片描述](http://img.blog.csdn.net/20170425160933286?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

具体的使用方式，这篇[博客](http://www.ghugo.com/chrome-timeline/)里有详尽的说明。
<!--more-->
------

# 2.主要过程

主要渲染过程可以这么归纳（参考文献：http://www.cnblogs.com/dojo-lzz/p/3983335.html）

- 解析HTML
- 构建DOM树
- DOM树与CSS样式进行附着构造呈现树
- 布局
- 绘制

# 3.解析与构建DOM树

浏览器有专门的html解析器，并且是边解析边构建do'm树的，因此将前两部分放在一块讲。
总体的解析原则是：1.自上而下顺序解析。2.解析过程中遇到外部样式（link,style）和外部脚本(script),会阻塞浏览器的解析。3.外部样式和外部脚本（在没有async、defer属性下）会并行加载，但是外部样式会阻塞外部脚本的执行。

 即：html解析->外部样式、脚本加载->外部样式执行->外部脚本执行->html继续解析
  情况一：如果是动态脚本（即内联脚本）则不受样式影响，在解析到它时会执行。
  情况二：如果是动态创建的样式文件，则不会阻塞后续任何类型脚本的执行。
  情况三：外部样式**后续**外部脚本含有async属性（IE下为defer），外部样式不会阻塞该脚本的加载与执行

-------------------

## 3.1外部样式、脚本并行加载，外部样式会阻塞后续脚本执行，直到外部样式加载并解析完毕。
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>JS Bin</title>
  <script>var start = +new Date;</script>
  <link href="http://udacity-crp.herokuapp.com/style.css?rtt=2" rel="stylesheet">
   <link href="http://udacity-crp.herokuapp.com/style.css?rtt=1" rel="stylesheet">
</head>
<body>
  <p>I am here!</p>
  <span id="result"></span>
  <p>I am here  two!</p>
  <script>
    var end = +new Date;
    document.getElementById('result').innerHTML = (end-start);
  </script>
</body>
</html>
```
代码运行结果：
![这里写图片描述](http://img.blog.csdn.net/20170425162903749?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
并行加载：
![这里写图片描述](http://img.blog.csdn.net/20170425162958048?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
阻塞执行：
![这里写图片描述](http://img.blog.csdn.net/20170425163541617?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，外部css加载并解析（parse）完毕之后才执行后面的js代码。

-------------

**注意**
dom树构建完毕会触发DOMContentLoaded事件，但此时外部文件不一定加载完毕，比如一些图片。当页面加载完毕后才出发load事件，这也是 jquery的\$ (document).ready(function(){})和原生代码window.onload=function(){}的区别。（其他区别：    window.onload不能同时编写多个，如果有多个window.onload方法，只会执行一个; $(document).ready()可以同时编写多个，并且都可以得到执行 ）

-------

## 3.2 外部样式不会阻塞后续脚本的加载，但会阻塞后续脚本的执行
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>JS Bin</title>
  <script>var start = +new Date;</script>
  <link href="http://udacity-crp.herokuapp.com/style.css?rtt=2" rel="stylesheet">

</head>

<body>
  test
  <script src="http://udacity-crp.herokuapp.com/time.js?rtt=1&a"></script>
  <div id="result"></div>
  <script>var end = +new Date;document.getElementById("result").innerHTML = end-start;</script>

</body>
</html>
```
外部脚本代码：
```javascript
var loadTime = document.createElement('div');
loadTime.innerText = document.currentScript.src + ' executed @ ' + window.performance.now();
loadTime.style.color = 'blue';
document.body.appendChild(loadTime);
```
执行结果：
![这里写图片描述](http://img.blog.csdn.net/20170425164957548?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
在瀑布流中查看外部文件加载顺序：
![这里写图片描述](http://img.blog.csdn.net/20170425165140578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，确实是按先后顺序发起请求，但是并行加载。
![这里写图片描述](http://img.blog.csdn.net/20170425165616662?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看你到，在外部样式加载和解析完毕，外部脚本才开始执行

------------

## 3.3 如果后续外部脚本含有async属性，则该脚本不会被样式文件阻塞
这里我直接在3.2 中外部脚本标签中加入async ,代码就不贴了，看下执行结果：
![这里写图片描述](http://img.blog.csdn.net/20170425170250437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170425170334230?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170425170447622?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到 外部引入的time.js文件在不没有被css阻塞。

--------------------

## 3.4 动态创建的样式文件不会阻塞其后的script的执行，不管script标签是否具有async属性

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>JS Bin</title>
  <script>var start = +new Date;</script>

</head>

<body>
  test
  <script>
    var link = document.createElement('link');
    link.href = "http://udacity-crp.herokuapp.com/style.css?rtt=2";
    link.rel = "stylesheet";
    document.head.appendChild(link);
  </script>
  <div id="result"></div>
  <script>var end = +new Date;document.getElementById("result").innerHTML = end-start;</script>

</body>
</html>
```
运行结果：
![这里写图片描述](http://img.blog.csdn.net/20170425171538160?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
动态创建的link加载了3秒左右，而第一个script脚本（line:6）在67ms时执行，第三个script脚本（line:19）在75ms左右执行。
![这里写图片描述](http://img.blog.csdn.net/20170425171426454?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170425171406344?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

------

## 3.5动态创建的样式文件不会阻塞其后动态创建的script的执行

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>JS Bin</title>
  <script>var start = +new Date;</script>

</head>

<body>
  test
  <script>
    var link = document.createElement('link');
    link.href = "http://udacity-crp.herokuapp.com/style.css?rtt=2";
    link.rel = "stylesheet";
    document.head.appendChild(link);
    var script = document.createElement('script');
    script.src = "http://udacity-crp.herokuapp.com/time.js?rtt=1&a";
    document.head.appendChild(script);
  </script>
  <div id="result"></div>
  <script>var end = +new Date;document.getElementById("result").innerHTML = end-start;</script>

</body>
</html>
```
![这里写图片描述](http://img.blog.csdn.net/20170425174715092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，动态脚本在3秒左右才加载完毕，后其后的脚本在1.8秒左右就以及加载并执行了。另外第三个script标签在2ms左右就执行了，可见，js脚本无论是动态创建还是直接引入在页面的，顺序加载，但并行执行。这个在3.6节进行进一步证明。

------------

## 3.6动态创建的脚本的加载和执行均会被之前的样式文件阻塞,但不会阻塞后续脚本的执行

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>JS Bin</title>
  <script>var start = +new Date;</script>
  <link href="http://udacity-crp.herokuapp.com/style.css?rtt=2" rel="stylesheet">
</head>

<body>
  test
  <script>
	var script = document.createElement('script');
    script.src = "http://udacity-crp.herokuapp.com/time.js?rtt=1&a";
    document.head.appendChild(script);
  </script>
  </script>
  <div id="result"></div>
  <script>var end = +new Date;document.getElementById("result").innerHTML = end-start;</script>

</body>
</html>
```
![这里写图片描述](http://img.blog.csdn.net/20170425173537548?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170425173613423?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到，动态创建的脚本并没有阻塞后面脚本的执行，而且它的执行方式和其他脚本一样，均会被之前的CSS阻塞

---------

# 4 构建渲染树，绘制与布局

 在外部样式执行完毕后，css附着于DOM，创建了一个渲染树（渲染树是一些被渲染对象的集），也就是说渲染树与dom树是同时构建的。每个渲染对象都包含了与之对应的计算过样式的DOM对象，对于每个渲染元素来说，位置都经过计算，所以这里被叫做“布局”。然后将“布局”显示在浏览器窗口，称之为“绘制”。
同脚本文件一样，CSS样式表会阻塞图片的加载，而脚本文件不会。接着脚本的执行完毕后，DOM树构建完成。

渲染树的节点（渲染器），在Gecko中称为frame，而在webkit中称为renderer。渲染器是在文档解析和创建DOM节点后创建的，会计算DOM节点的样式信息。

在webkit中，renderer是由DOM节点调用attach()方法创建的。attach()方法计算了DOM节点的样式信息。attach()是自上而下的递归操作。也就是说，父节点总是比子节点先创建自己的renderer。销毁的时候，则是自下而上的递归操作，也就是说，子节点总是比父节点先销毁。

 如果元素的display属性被设置成了none，或者如果元素的子孙继承了display:none，renderer不会被创建。节点的子类和display属性一起决定为该节点创建什么样的渲染器。但是visibility：hidden的元素会被创建

具体如何构建渲染树，可参见：

- [【浏览器渲染原理】渲染树构建之渲染树和DOM树的关系](http://blog.csdn.net/greenqingqingws/article/details/19822139)

- [HTML渲染过程详解](http://www.cnblogs.com/dojo-lzz/p/3983335.html)