---
title: js创建dom节点之最容易被忽略的createDocumentFragment()方法
date: 2017-04-16 18:36:27
tags:
- JavaScript
- dom
- 前端
categories: 前端
---

js常见的创建dom节点的方法有

- createElement() 创建一个元素节点   => 接收参数为**string**类型的nodename
- createTextNode() 创建一个文本节点  =>   接收参数为**string**类型的text内容
- createAttribute() 创建一个属性节点   =>   接收参数为**string**类型的属性名称
- createComment() 创建一个注释节点   => 接收参数为**string**类型的注释文本

本文要说的createDocumentFragment()方法，则是用了创建一个虚拟的节点对象，或者说，是用来创建文档碎片节点。它可以包含各种类型的节点，在创建之初是空的。
<!--more-->
-----

DocumentFragment节点不属于文档树，继承的parentNode属性总是null。它有一个很实用的特点，当请求把一个DocumentFragment节点插入文档树时，插入的不是DocumentFragment自身，而是它的所有子孙节点。这个特性使得DocumentFragment成了占位符，暂时存放那些一次插入文档的节点。它还有利于实现文档的剪切、复制和粘贴操作。
另外，当需要添加多个dom元素时，如果先将这些元素添加到DocumentFragment中，再统一将DocumentFragment添加到页面，会减少页面渲染dom的次数，效率会明显提升。

-----

还有一个很重要的特性是，如果使用appendChid方法将原dom树中的节点添加到DocumentFragment中时，会删除原来的节点。
为了证明这一点我做了以下测试：
``` html
<body>
	<ul>
		<li>Alice</li>
		<li>Bob</li>
	</ul>
	<button onclick="test()">测试</button>
</body>
```
js代码中test()方法如下：
```javascript
function test(){
	var li = document.getElementByTaName('li')[0];  //ul中的第一个li节点
	alert(document.getElementByTaName('li')[0].innerText) // 显示Alice

	var newFrag = document.createDocumentFragment();
	newFrag.appendChild(li);

	alert(document.getElementByTaName('li')[0].innerText) // 显示Bod
	alert(document.getElementByTaName('ul')[0].innerHTML)} //显示<li>Bob</li>,由此可见，第一个节点确实被删除了

	//现在fragment中的修改节点
	newFrag.childNode[0].childNodes[0].nodeValue='Candy';
	//更改一个孩子节点的文本内容
	// .childNodes[0].nodeValue等同于：.innerText  或.textContent

	 document.getElementByTaName('ul')[0].appendChild(newFrag);
	 alert(document.getElementByTaName('ul')[0].innerHTML)} //显示<li>Bob</li><li>Candy</li>  ,由此可见仅仅是添加了newFrag的子孙节点。
```


