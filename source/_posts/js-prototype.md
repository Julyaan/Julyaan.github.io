---
title: 由一道js题探讨构造函数、prototype和__proto__之间的关系
date: 2017-05-02 21:58:07
tags:
- JavaScript
- 原型
- 前端
categories: 前端
---

javascript原型世界还是挺有意思的，这篇文字主要是在借助chrome浏览器'剖跟溯源'，大家可以直接跳到文末看结论。
# 缘起
今天在牛客上看到这样一个题：
```javascript
var F = function(){};
Object.prototype.a = function(){};
Function.prototype.b = function(){};
var f = new F();
```
问：能否通过f取到方法a，方法b?

<!--more-->
---

# 刨根

在git上看到有人画了这么一个图：
![这里写图片描述](http://img.blog.csdn.net/20170502203216424?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

你如果理不清prototype，__proto_ _，construcor这些概念和关系，会觉得这副图很杂乱。这里，我将根据自己的理解和实验，重新画一幅图。画之前，先总结下自己对这些概念的理解：

prototype是每个函数默认会拥有的一个属性，该属性是一个指针，指向一个原型对象，这个对象包含一个不可枚举的属性constructor，而constructor的值则是一个函数对象。当调用构造函数创建一个实例后，该实例也会包含一个内部属性，也是一个指针，指向构造函数的原型对象，es5中将该指针定义为[[prototype]]，__ proto__ 是FireFox,Safari和Chrome提供的该指针的非标准的访问方法，另外要说一句的是，[[prototype]]是一个内置属性，js通过该属性来寻找原型链，这个属性并不只是在实例对象中才存在的，每个对象中都会有（除了 Object.prototype . __ proto __ ， 一般地，认为：Object.prototype .  __ proto __ =NULL）。

做一个测试：
```javascript
var F=function(){};
var p=F.prototype;
var c=p.constructor;
console.log(c===F)  // ==> true
```

-----------------------

回到题目中，现在chrome调试器中敲入题目中的代码，由于f是F的一个实例，因此，__ proto __ 指向F.prototype，另外由于js中所有引用类型都默认继承了Object，因此F.protoype中仍然包含 一个指针，为了弄清楚这个继承关系，我在控制台输入console.log(f.__ proto __ ) ：
![这里写图片描述](http://img.blog.csdn.net/20170502210055954?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

果然，F.prototype里包含了两个默认属性，construcor和 __ proto __ 。可以看到construcor指向一个函数对象，而这个函数对象正是F，而F的prototype属性指向的Object正是F.prototype(哈哈，这大概不是废话)。根据继承的概念，那么这个__ proto __ 应该指向父类的原型，也就是Object.prototype，这里验证一下：
![这里写图片描述](http://img.blog.csdn.net/20170502211429868?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
属性里包含a方法，因此可以肯定就是Object.prototype了，然后在观察它的constructor：
![这里写图片描述](http://img.blog.csdn.net/20170502211716744?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
正式Object的构造函数，里面也看到了很多熟悉的方法。

------

## 一个问题
看到这里不免产生疑问 为什么Object的构造函数比原型多了这么多属性？
![这里写图片描述](http://img.blog.csdn.net/20170502212841516?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
getOwnPropertyNames()方法返回对象的所有属性（包括不可枚举类型）。测试发现Object.prototype中所有属性有13个（包括后来添加的a），而Object中包含25个 （如create）,那么这另外12个属性是哪来的？

-------------------

## 解决

这时我注意到，function Object 中除了prototype这个指针属性外，还有一个指针：__ proto  __。 原来，秘密都在这里。

![这里写图片描述](http://img.blog.csdn.net/20170502214110346?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

该函数对象中包含b方法，由此 可以证明，该函数对象正是Function.prototype.

------------------

Function终于出现了！Function跟Object一样，是js中原生的构造函数,，每个函数都是Function的实例。回忆下Function的一种用法：
```javascript
var sum=new Function('num1','num2','return num1+num2');
```
通过构造函数定义函数，这正式js中三种定义函数的方法之一，同时也是不太推荐的一种定义方式。

回到题目中，继续观察Function.prototype的结构：
![这里写图片描述](http://img.blog.csdn.net/20170502215142839?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
通过观察constructor属性和__ proto __ 属性，验证了Function.protptype.constructor = Function，另外也得出一个结论，Function继承至Object：Function.protptype.__ proto __ = Object.prototype。


顺着Function.protptype.constructor这个"瓜"，再来观察Function，发现Function的两个指针prototype和__ proto __  均指向 Function.protptype （没毛病，哈哈）。

现在我根据已有的实验结果，画出这样的原型链图来：
![这里写图片描述](http://img.blog.csdn.net/20170502224811954?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

----------------------

回到最开始我们输出的f.__ proto __  中，刚才只讨论的f.__ proto __ 的__ proto __ 属性，还有一个constructor属性指向的F函数还没有研究，另外图中还缺少Function的构造函数，即function Function，我们可以猜想出，F的[[prototype]]应该是指向Function.prototype，下面验证一下：
![这里写图片描述](http://img.blog.csdn.net/20170502225325417?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

果然如此，而且我们还看到 F的[[prototype]]的constructor指向function Function(), 那么查看function Function() ：
![这里写图片描述](http://img.blog.csdn.net/20170502225534512?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
内置两个指针prototype 和 __ proto __ 均指向Function.prototype。

------------

# 见底

至此，整个原型链结构图我们可以画出来了：

![这里写图片描述](http://img.blog.csdn.net/20170502230632110?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到，所有的对象最终都继承于Object.prototype，而所有的构造函数，包括function Object在内，又都可以看做是function Function的实例。
那么这道题的答案在此时是显而易见的：
通过f能过调用a方法，但不能调用b方法；
通过F是能取到b的，可以通过f.constructor.b 调用；
同样，通过F也能取到a，可以同过f.constructor.a 调用。

-----

最近在知乎专栏上看到一种更形象的表示方法（跟上面我总结的那张图是一致的）：
把 Object.prototype 必做JavaScript事件中对象的祖先，Function.prototpe必做机器的祖先，可以画出这样的图来（用[p]指代 \__ proto \__)

<img src='http://7xwnfc.com1.z0.glb.clouddn.com/proto.jpeg'>

可以认为，你的prototype所指的是你的模板对象，[p]所指的是你的原型对象，所以String,Number,包括作为构造函数的function这些机器也需要一个模板对象，表明自己继承自Object的关系，因此，再进行补充：

<img src='http://7xwnfc.com1.z0.glb.clouddn.com/more.jpeg'>

参考文章 [《JavaScript 世界万物诞生记》](zhuanlan.zhihu.com/p/22989691)