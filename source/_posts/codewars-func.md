---
title: codewars上的一道训练函数式编程思维的编程题
date: 2017-08-25 15:14:10
tags:
- 函数式编程
- es6
categories: 技术杂谈

---

分享codewars上的一道训练函数式编程思维的编程题。
> https://www.codewars.com/kata/church-numbers-add-multiply-exponents/train/javascript
这个题目用了大量的篇幅进行思路引导,最后让实现加法，乘法和指数乘法三个函数，这里我就不翻译了，只给出题目给出的条件和所有目标

<!--more-->

# 条件
整理一下题目给出的条件：

- 函数churchify：
使用方式：churchify(n:Number)(f:function)(x:number)，返回结果为`f performed on x n times ` ，即嵌套执行n次f(x)。该函数的实现方式很多，题目直接给出了一种实现：

 ```javascript
var churchify = (n) => (f) => (x) => {
      for(var i = 0; i < n; i++) x = f(x);
      return x;
    };
 ```
- 函数numerify:
 接收一个函数作为参数，这个函数是一个辅助函数，定义是这样的：

 ```javascript
 var numerify = (c) => c((i) => i + 1)(0);
 ```
- 函数tesChurch:
 这个函数很重要：

 ```javascript
 var testChurch = (fn, x, y) => numerify( fn( churchify(x) )( churchify(y) ) )
 ```

# 问题

要求设计三个函数Add,Mul,Pow。使得：

- testChurch(Add,x,y)返回的结果为 x+y的结果
- testChurch(Mul,x,y)返回的结果为 x*y的结果
- testChurch(Pow,x,y)返回的结果为 x^y(x的y次方)的结果

另外题目还要求要考虑算法的复杂度，否则会超时，不要直接调用Math.pow方法。

这道题挺有意思的，大家不妨思考下怎么解，给出我的答案吧：

# 解答

```javascript
var Add = (c1) => (c2) => (f) => (x) => c1(f)(c2(f)(x));
var Mul = (c1) => (c2) => (f) => (x) => c1((i)=>i+c2(f)(x))(x);
var Pow = (c1) => (c2) => (f) => (x) => c2((i)=>i*c1(f)(x))(1);
```