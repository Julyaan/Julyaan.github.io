---
title: 用原生js实现addClass,removeClass,hasClass方法
date: 2017-04-18 13:52:28
tags:
- JavaScript
- dom
- 前端
- 面试
categories: 前端
---

其实html5已经扩展了class操作的相关API，其中classList属性就以及实现了class的增删和判断。
classList属性的方法有：
> - add(value)  添加类名，如果有则不添加
> - contains(value) 判断是否存在类名，返回Boolean值
> - remove(value) 从列表中删除类名
> - toggle(value)  切换类名：如果列表中存在则删除，否则添加


<!--more-->

----

根据红宝书的介绍，目前支持classList属性的浏览器有FireFox 3.6+和Chrome。因此为了更好的兼容性，我们可以自己手动实现这几个方法。
这里利用了DOM属性 className，我们始终是在操作这个对象。
```javascript
function hasClass( elements,cName ){
  return !!elements.className.match( new RegExp( "(\\s|^)" + cName + "(\\s|$)") );
};
function addClass( elements,cName ){
  if( !hasClass( elements,cName ) ){
    elements.className += " " + cName;
  };
};
function removeClass( elements,cName ){
  if( hasClass( elements,cName ) ){
    elements.className = elements.className.replace( new RegExp( "(\\s|^)" + cName + "(\\s|$)" ), " " );
  };
};
```