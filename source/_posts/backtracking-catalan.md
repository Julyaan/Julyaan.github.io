---
title: 回溯法打印卡特兰数问题
date: 2017-05-19 11:10:23
tags:
- 卡特兰数
- 算法
- 回溯
- 面试
- BAT
categories: 算法
---

今天在牛客上做到一个题，用到了卡特兰数的知识。(之前专门写了一篇文章讲解卡特兰数，**[传送门](/2017/04/07/catalan/)**)
那道题我会单独写一篇博客，这里把其中一个子问题抽象出来，给出代码方案：
给了长度为2n的顺序数列，先将数列分成两排，要求第一排的每一列小于等于对应的第二排的数字，每排顺序排列，打印出所有排序方案。

<!--more-->
这里我用回溯方法解决，对所有方案进行深度优先遍历，如果‘0的数量大于1’（为什么？还是刚才那个传送门： [4.3.4 卡特兰数排队问题](/2017/04/07/catalan/) ）,则返回上层。
这里我构造了两个空数列：firstLine和secondLine，从头到尾遍历长度为2n的原数组arr，每次都有两个选择方案：将arr[i]放入到firstLine或secondLine。怎么放呢？没关系，我们先进行遍历，按顺序来，如果放入后firstLine的长度大于等于secondLine，则进入下层循环。另外，为了保证回溯，一定要记得及时清理状态，在每次循环后将两个数组的状态返回为上一层的样子。

代码如下：
```javascript
function catalanSort(arr,firstLine,secondLine,i){
    var n=arr.length/2;
    if(firstLine.length==n) {  //搜索到叶节点，输出一个结果
        console.log(firstLine);
    }
    else {
        for (var j = 0; j < 2; j++) { //枚举所有可能的路径
            if (j == 0) {
                firstLine.push(arr[i]);
            } else {
                secondLine.push(arr[i]);
            }
            //判断是否满足约束条件
            if (firstLine.length >= secondLine.length) {
	            //满足条件，扩展搜索空间
                catalanSort(arr,firstLine,secondLine,i+1);
            }
            //回溯前的清理工作：清除所占的状态资源
            if(j==0){
                firstLine.pop();
            }else{
                secondLine.pop();
            }
        }
    }
}
```
调用方式就是：
```javacript
catalanSort(arr,[],[],0)
```