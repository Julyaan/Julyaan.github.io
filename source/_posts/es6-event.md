---
title: 用es6语法实现event类
date: 2017-08-28 15:18:42
tags:
- es6
- event
- 设计模式
categories: 前端
---

公司转正面试做的一道题，由于实习期间也一直用的es6语法，所以就用es6实现了下。写法很多，整理出来，供大家参考。
> 题目，实现一个Event类，继承自此类的对象都会拥有四个方法on,off,once和emit,可以实现链式操作

题干描述很简单，考察的点却有一定深度。由这道题的解题思路一定程度上也能反映出解题者对一些设计模式的理解，如发布-订阅者模式，单例模式。
<!--more-->
-------------------------------------------------------------------

代码：
```javascript
class EventEmitter{
    constructor(){
        this._events={}
    }
    on(event,callback){
        let callbacks = this._events[event] || []
        callbacks.push(callback)
        this._events[event] = callbacks
        return this
    }
    off(event,callback){
        let callbacks = this._events[event]
        this._events[event] = callbacks && callbacks.filter(fn => fn !== callback)
        return this
    }
    emit(...args){
        const event = args[0]
        const params = [].slice.call(args,1)
        const callbacks = this._events[event]
        callbacks.forEach(fn => fn.apply(this, params))
        return this
    }
    once(event,callback){
        let wrapFunc = (...args) => {
            callback.apply(this,args)
            this.off(event,wrapFunc)
        }
        this.on(event,wrapFunc)
        return this
    }
}

// 测试
let ee = new EventEmitter();
function a() {
    console.log('a')
}
function b() {
    console.log('b')
}
function c() {
    console.log('c')
}
function d(...a) {
    console.log('d',...a)
}
ee.on('TEST1', a).on('TEST2', b).once('TEST2', c).on('TEST2',d);

ee.emit('TEST1');
console.log('....')
ee.emit('TEST2');
// In test2
// In test2 again
console.log('....')
ee.emit('TEST2');

```

更多写法可以参见张容铭的那版《JavaScript设计模式》第17章：通信卫星-观察者模式