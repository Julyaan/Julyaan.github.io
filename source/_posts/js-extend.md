---
title: 理解JavaScript中所谓的‘寄生组合式继承’
date: 2017-05-11 17:24:13
tags:
- JavaScript
- 继承
categories: 前端
---

最近又反复阅读了《js高级程序设计》上对js继承的介绍，然后在掘金上也看到篇比较深刻的介绍js类与继承的博文，所以来谈下自己的理解。
先贴下那篇博文:[JavaScript 的继承与多态](https://juejin.im/post/5912753ba22b9d005817524e)
再贴两篇能够更深刻地帮助理解js对象模型的文章：
1.[js运算符instanceof的工作原理](http://blog.csdn.net/kevinwon1985/article/details/7719906)
2.[JavaScript对象模型-执行模型](http://www.cnblogs.com/RicCC/archive/2008/02/15/JavaScript-Object-Model-Execution-Model.html)

<!--more-->

看完以上文章你应该能easy地解释这两行代码为什么会这样输出：
```javascript
Function instanceof Object  //true
Object instanceof Function  //true
```

-----------------

基本的方法就不说了，原型链啊，借用构造函数啊，组合式啊，寄生（分离）式啊，以及各自的优缺点啊，内部constructor,prototype,[[prototype]]这些指针的指向以及意义啊，这些都是必须掌握的。

在这篇文章，我想讨论目前为止，最好的继承实现方法——这里暂且不讨论es6、7中的类与继承——寄生组合式继承。

-------------------

其实最迷惑初学者的是，同样的原理，实现同样的功能，却有五花八门的实现方法和叫法，所以会误以为是有好多种方法。我最开始学习时，就进入了这样一种误区，然后回去记忆这些不同的方法，导致在用的时候总会混淆。现在看来，真的好蠢，为什么不花点时间好好琢磨下原理呢？这样没准你自己也能写一套独有的‘最优方法’呢。

-----------

闲话说多了，进入正题。
首先，为什么要用寄生组合式呢？组合是指什么？
弄清第一个问题很容易，自己去翻看《js高级程序设计》第6章就好了。那组合指的是什么？我的理解就是这两部分的组合：构造函数属性的继承和建立子类和父类原型的链接。

1.构造函数属性的继承
在子类中调用超类的构造函数：
```javacript
SuperType.apply(this,arguments)
```
2.建立子类和父类原型的链接
最简单的方法就是用Object.create()方法对父类的原型进行浅复制，赋给子类原型：
```javacript
SubType.prototype=Object.create(SuperType.prototype);
```
如果仅仅是这样，SubType.prototype.constructor指向的还是Supertype(但并不影响instanceof，为什么？戳[《js运算符instanceof的工作原理》](http://blog.csdn.net/kevinwon1985/article/details/7719906)),最好，在进行增强原型：
```javacript
SubType.prototype.constructor = SubType;
```
那么使用Object.create的原型链是什么样的呢？其实很简单：
```javacript
SubType.prototype.__ proto __ = SuperType.protype
```
也就是说，子类的原型相当于是父类原型的一个实例，这不就是实现了两者的链接了吗？相比之前的原型链继承方法：SubType.prototype=new SuperType()；这里没有创建构造函数；相比之前的经典组合式继承，不必在调用子类构造函数时再重写超类的实例属性（因为压根就没有继承过），也就是相当于，去掉了无用的、会被覆盖掉的一部分代码，因此这种方法被认为是继承的最佳实现。

另外，红宝书上在介绍寄生组合继承时引入了一个函数：
```javascript
function inheritPrototype(subType,superType){
	var prototype=object(superType.prototype) //object()方法是ES5前Object.create()的非规范化实现，后面会给出代码以及二者的比较
	prototype.constructor = subType; //增强对象
	subType.prototype = prototype; //指定对象
}
```
其实就是把上面的两个语句整合在了一个函数里面而已，换汤不换药。下面给出object()方法：
```javascript
function object(o){
	function F(){}
	F.prototype = o;
	return new F();
}
```
返回的是F（）的一个实例，那这个实例会有一个__ proto __ 指针，这个指针指向F.prototype, 也就是o, 所以 跟刚才Object.creat()的用法是相同的功能。

说了这里，不得不再来聊一下历史。这个object（）方法是道格拉斯·克罗克福德在2006年发表的一篇题为Prototypal Inheritance in JavaScript中提出的（大牛啊，再瞧瞧我们现在水的那些所谓的“学术型”论文，汗颜啊）。后来ECMAScript5 通过新增Object.create()方法规范化了这种继承方式。在传入一个参数的情况下，Object.create()方法和object()方法的行为相同，不同的是Object.create()可以接收可选参数[propertiesObject],该参数对象是一组属性与值，该对象的属性名称将是新创建的对象的属性名称，值是属性描述符（这些属性描述符的结构与Object.defineProperties()的第二个参数一样）,具体请戳[MDN-Object.creat()方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

因此，这里我们可以将inheritPrototype（）中第一条语句改为
```javascript
var prototype=Object.create(SuperType.prototype)
```
那么最终的实现方法是：
```javascript
function SuperType(name){
	this.name=name;
}
SuperType.prototype.sayName=function(){console.log(this.name)}

function SubType(name,age){
	SuperType.call(this,name);
	this.age=age;
}
inherit(SubType,SuperType);
```