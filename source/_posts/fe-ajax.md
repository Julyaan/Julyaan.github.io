---
title: 前端面试ajax考点汇总
date: 2017-05-08 17:05:10
tags:
- Ajax
- 面试
- 前端
categories: 前端
---
Ajax(Asynchronous JavaScript and XML)是一系列web开发技术的集合，使用很多的web技术在客户端开发异步web应用。利用Ajax，web应用可以异步的发送数据获取数据，而不干扰现有页面的显示和行为。通过解耦数据接口层和展现层，Ajax允许web页面或者其他扩展的web应用动态的改变数据而不用重新加载整个页面。实现通常选择JSON代替XML，因为更接近JavaScript。

Ajax不是一种技术，是一组技术。这里我总结了一些偏基础的ajax面试考点,其中一部分也是自己在面试中遇到过的。

<!--more-->


# 手写一个ajax get方法

```javascript
var xhr=new XMLHttpRequest();
xhr.onreadystatechange=function(){
	if(xhr.readyState == 4){
		if(xhr.status>=200 &&xhr.status<300 || xhr.status == 304){
			console.log(xhr.responseText);
		}
	}
}
xhr.open('get','https://www.baidu.com',true);
xhr.send(Null);
```
考点：

1. 收到响应后，响应的数据会自动填充XHR对象的属性，因此readState、status和responseText属性要通过xhr去拿，而不是 回调函数的参数！ 这是很多人容易 犯的一个错误。回调函数传回来的是event对象，该对象的target属性指向的次啊是XMLHttpRequest实例。最好也不要通过this去获取，因为作用域问题，有的浏览器中会报错，因此为了可靠，应使用xhr对象来获取这些属性。

2. readyState属性都代表什么？
0：未初始化。尚未调用open()方法；
1：启动。已经调用open()方法，但是未调用send()方法；
2：发送。已经调用send()方法，但尚未接收到响应；
3：接收。已经接收到部分响应数据；
4：完成。已经接收到全部响应数据。

3. 只要readyState属性的值由一个变成另一个，就会触发一个readystatechange事件，因此最好在open()之前制定onreadystatechange事件处理程序。

-----------------

# 设置头部信息

使用setRequestHeader()方法设置自定义的请求头部信息。如：
xhr.setRequestHeader('MyHeader','MyValue');
必须在open()方法之后，send()方法之前调用。

----------

# post请求，发生表单数据

方法一：
首先设置Content-Type头部信息为application/x-www-form-urlencoded;然后对表单进行序列化处理，创建一个字符串，最后发送字符串：

```javascript
	xhr.open('post','/posturl',true);
	xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
	var form=document.getElementById('form');
	xhr.send(serialize(form));
```
方法二：
使用FormData对象，该方法不必设置请求头部。
```javascript
var data= new FormData();
data.append('name','value');
//也可以通过项构造函数传表单元素
var data=new FormDate(document.forms[0])
```
append()方法接收两个参数：键和值。

-------

# XMLHttpRequest 2级

1. formData
2. 超时设定
  xhr.timeout = 1000
  xhr.ontimeout= function(){}
3. load事件
  xhr.onload=function(){}
  该方法在接收到完整的响应数据时触发
4. abort事件
  调用abort()方法触发
5. progress事件
  该事件在接收响应期间持续触发
  ```javascript
  xhr.onload=function(event){

  }
  ```
  event包含三个额外的属性：lengthComputable、position和totalSize，分别表示进度信息是否可用（boolean），已经接收的字节数和总字节数（根据响应头部中的Content-Length获取）


