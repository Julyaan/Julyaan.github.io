---
title: 阿里前端算法面试题两道
date: 2017-08-15 15:55:25
tags:
- 深度优先搜索
- 面试
- BAT
- 算法
categories: 算法
---

前两天去参加阿里的提前批面试，也是巧了，两位不同的面试官出的算法题解题思路都是异曲同工。都可以用递归的方式进行深度优先搜索

更巧的是阿里今年秋招的前端笔试也出了一道类似的题, 学生信息组合的全排列填充表格，跟我一面的那道题基本一样。话不多说，上代码。

<!--more-->

# 二维数组的全排列组合。

> 如输入[[1,2],[3,4],[5,6]]
> 输出：
> [ 1, 3, 5 ]
[ 1, 3, 6 ]
[ 1, 4, 5 ]
[ 1, 4, 6 ]
[ 2, 3, 5 ]
[ 2, 3, 6 ]
[ 2, 4, 5 ]
[ 2, 4, 6 ]

代码实现：
```javascript
function printArr(arr,n,res){
    for(var i = 0; i<arr[i].length;i++){
        if(n == 0){
            res = []
        }
        if(n<arr.length){
            var _res = res.slice()
            _res.push(arr[n][i])
            if(n == arr.length-1){
                console.log(_res)
            }else{
                printArr(arr,n+1,_res)
            }
        }
    }
}
// 测试：
var arr = [[1,2],[3,4],[5,6]]
printArr(arr,0)
```

-----

# 打印青蛙跳台阶的所有方式

> 注意 不是求方式的个数，而是打印每种情况
> 台阶数为10，每次跳1次或两次

代码实现：
```javascript
function step(n,res){
    if(n==0){
        res=[]
    }
    var i=1
    while(i<3){
        if(n+i<=10){
            var _res = res.slice()
            _res.push(i)
            if(n+i == 10) {
                console.log(_res)
            }else{
                step(n+i, _res)
            }
        }
        i++
    }
}
step(0)
```