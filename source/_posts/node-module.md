---
title: node中javascript模块的编译原理
date: 2017-05-15 10:49:04
tags:
- node
- 模块化
- JavaScript
categories: node
---

>众说周知，Node中的模块是根据CommonJS规范来实现的，但是，Node是实现过程中也对规范进行了一定的取舍。尽管规范中exports,require和module听起来十分简单，但是Node在实现过程中经历了什么，还是值得进一步思考和探索的。

本篇博客是本人阅读朴灵的《深入浅出理解NodeJS》一书的一篇读书笔记。主要记录下自己在阅读JS模块编译部分章节的一点心得。

Node中，每个文件模块都是一个对象，理解这一点就好了。具体的模块编译，这本书的第2章讲解的特别棒，推荐有兴趣的人去阅读下，这里就不再总结了。

<!--more-->
---

之前我一直有个疑惑，每个模块文件中都存在 require,exports,module这些变量，但是并没有看到它们在哪定义的。估计大部分人都产生过这样的疑惑，而这本书给了我们详尽的解答。

事实上，编译过程中，Node会对获取的Javascript文件进行头尾包装：

```javascript
(function(exports,require,module,_filename,_dirname){
\\----------头部-------------
.
.
文件中的js代码
.
.
\\----------尾部--------------
});
```
对了，_filename和 _ dirname 也是‘自带’的两个属性。

通过这种方法，每个模块文件之间都进行了作用域隔离。接下来，包装之后的代码会通过VM原生模块的runInThisContext()方法执行，该方法类似eval()，只是具有明确的上下文，无污染全局。

该方法会返回一个具体的function对象。最后，将该当前模块对象的exports属性，require()方法，module（即模块对象自身）以及文件定位中得到的完整的文件路径和目录作为参数传递给这个function（)执行。

执行之后，模块的exports属性被返回给了调用方，因此exports属性上的任何方法和属性都可以被外部调用到，但是模块中的其他属性或方法则不可直接调用。

以上就是Node中js模块化的实现原理。

---------------------------

另外，还应注意一个细节：
> 尽管exports也是直接被返回的属性，但是一定要通过module.exports来添加属性或方法，直接用exports通常会得到失败的结果。

为什么呢？这里贴一下node官方文档：
![这里写图片描述](http://img.blog.csdn.net/20170515174401386?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到，最终返回的是exports对象是通过形参传入的，直接赋值形参，只会改变形参的引用，Module.exports才是真正的接口，exports只不过是它的一个辅助工具。最终返回给调用的是Module.exports而不是exports。如果module.exports本身不具备任何属性和方法，exports收集到的属性和方法，会赋值给了Module.exports。如果，Module.exports已经具备一些属性和方法，那么exports收集来的信息将被忽略。

比如：
foo.js
```javascript
 module.exports = {a: 2}
 exports.a = 1
```
test.js
```javascript
var f=require(./foo);
console.log(f.a);  // 2
```
可见，exports在module.exports 被改变后，失效。

总结一下exports和module.exports的区别：
1.module.exports 初始值为一个空对象 {}
2.exports 是指向的 module.exports 的引用
3.require() 返回的是 module.exports 而不是 exports

所以在实际使用中，我们最好按照官方的说明：ignore **exports** and only use **module.exports**
