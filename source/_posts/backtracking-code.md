---
title: 运动员分组问题：如何分组使得中位数之和最大
date: 2017-05-19 20:43:36
tags:
- 卡特兰数
- 算法
- 回溯
- 面试
categories: 算法
---

题目来自牛客网上某大厂实习生在线笔试做的一道编程题。上一篇博文（ [《回溯法打印卡特兰数问题》](/2017/05/19/backtracking-catalan/) ）中给出了子问题的解决算法。这里给出完整题目和代码。
题目就贴截图好了：
*更多卡特兰数算法题请戳这篇： [《二叉搜索树形态问题--从一道算法题探讨神奇的Catalan数》](/2017/04/07/catalan/)*

<!--more-->
![这里写图片描述](http://img.blog.csdn.net/20170519232931413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170519232952039?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```javascript
var readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
    terminal:false
});

var n = -1;// 初始状态为负数，表示还没开始读取
var res = 0;
rl.on('line', function(line){ // javascript每行数据的回调接口
    if (n < 0) { // 测试用例第一行读取n
        n = parseInt(line.trim())
    } else {
        var tokens = line.split(' ').map(function (x) {
            return parseInt(x);
        });
        var arr=tokens.sort(sortNumber); //将数列从小到大排序
        var hm=arr.slice(n); //获取[n+1,3n]顺序区间内的数列
        catalanSort(hm,[],[],0);
        console.log(res);
    }
});

function sortNumber(a,b) {
    return a-b
}
function catalanSort(arr,firstLine,secondLine,i){ //firstLine存储各分组的中位数
    var n=arr.length/2;
    if(firstLine.length==n) {   //搜索到叶子节点，得到一组符合条件的中位数
        res=getSum(firstLine)>res?getSum(firstLine):res;  //判断该组中位数之和是否最大
    }
    else {
        for (var j = 0; j < 2; j++) {
            if (j == 0) {
                firstLine.push(arr[i]);
            } else {
                secondLine.push(arr[i]);
            }
            if (firstLine.length >= secondLine.length) { //约束条件判断
                catalanSort(arr,firstLine,secondLine,i+1);  //符合条件，扩展搜索空间
            }
            //回溯前，清除上一步占用的空间状态
            if(j==0){
                firstLine.pop();
            }else{
                secondLine.pop();
            }
        }
    }
}

function getSum(arr){  //计算数组中所有项之和
    return arr.reduce(function(x,y){
        return parseInt(x)+parseInt(y);
    });
}
```
