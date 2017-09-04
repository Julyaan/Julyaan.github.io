---
layout: post
title: 动态规划法解析two eggs problem及其延伸问题
date: 2017-03-28 16:23:33
tags:
- 算法
- 动态规划
- 面试

categories: 算法
mathjax: true
---



Two eggs problem可以说是互联网面试中老生常谈的算法题了，经常可以在各大互联网公司的笔试真题中看到它的各类变种（腾讯大厦，球掉落问题，玻璃珠问题等等）。本文将深入探讨此类问题及其延伸问题的通用解法，并给出javascript代码实现。
<img src="https://source.unsplash.com/random/900x600" width = "900" height = "600" alt="git" align=center />
<!--more-->

# 问题描述

## Two-eggs problem

首先看下游戏的定义：

> 有一幢100层的大楼，玩家可以到达大楼的任意楼层。现给玩家两个一模一样的鸡蛋，当玩家拿着鸡蛋从某一楼层扔下时，一定会出现两个结果，鸡蛋碎了或者没碎（完好无损）。该大楼有一个临界楼层，低于它的楼层往下扔鸡蛋时鸡蛋不会碎，等于或高于它时会碎。请为该玩家设计一种丢鸡蛋策略，使得该策略下的最坏情况的次数比其他任何策略最坏的次数都少。输出该策略下的最坏情况次数。

很多人乍一看到这道题会很懵，什么叫丢鸡蛋策略？怎么设计？如果用代码给出设计方案？你可能会去把问题分解成第一次在哪个楼层扔，鸡蛋会不会碎，还剩几个鸡蛋，第二次在哪个楼层扔，鸡蛋会不会碎，还剩几个鸡蛋，第三次……然后你就会觉得这是一个特别复杂/抽象的问题，然后就放弃了。

所以此类问题我们要换个思路去想，这里先抛开那些经典的算法思想，总结下解题思路：

- **分解为若干子问题**
- **化抽象为具体，不妨代入未知数，先进入子问题再说**
- **简化问题求解，比如将2个鸡蛋改成1个鸡蛋，100层楼改成10层楼**
- **根据上面两步，确定最终使用的方法（动态规划？回溯？分治）**

有了以上的思路，我们很容易想到，用**递归**的思想去解决，如何递归呢？这里我们暂时卖下关子，看一下此类问题的通用表示方法：n-eggs problem

-------------------

## 扩展到 n-eggs problem

> 现在有m层楼（m>1）,玩家手里有n个鸡蛋，求设计一种方法，使得在该方法下找到临界楼层的最坏情况次数最少。

这个问题可以抽象为这样一个函数：
<center> throwNumber(floors,eggs)  => numOfWorstCase </center>

函数入口是楼层数和鸡蛋数，返回最优方案的最坏次数（throwNumbers of the worst case of the best strategy）。

在下文中我们不妨用**$\it Tn(f,e)$**来表示f层楼e个鸡蛋的输出结果。

-----
# 解决方案

回到刚才讨论的解决思路，我们不妨分别考察下一个鸡蛋，两个鸡蛋，三个鸡蛋的$\it Tn(f,e)$ 求法。

## one-eggs problem

<center>![one-egg problem](http://img.blog.csdn.net/20170328155444822?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDU4MjA4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)</center>

这类问题是不是很简单？因为你就只有一个鸡蛋，所以为了保证一定能找到临界楼层，我们只能从第一层开始遍历楼层，所以最坏情况的次数是楼层f, 那么可以得到：

> <center>**$\it Tn(f,1) = f $**</center>

------

## two-eggs problem

我们不妨设 $\alpha _i $ 为第i次扔蛋选择的楼层。每次扔蛋有两种结果：

- 鸡蛋碎了，那么临界楼层一定在 $\alpha _i $层及以下楼层范围内，问题转移为1个蛋，$\alpha _i $-1层楼的问题，即**$\it Tn(\alpha _i-1,1)  $**
- 鸡蛋没碎，那么临界楼层一定在 $\alpha _i $层以上，问题转移为2个蛋，100-$\alpha _i $层楼的问题，即**$\it Tn(100-\alpha _i,2)  $**

可以用以下流程图表示：

```flow
st=>start: Tn(100,2)
e=>end: Tn(a-1,1)
op=>operation: 选择在a层扔蛋
cond=>condition: 蛋是否碎了？
ee=>operation: Tn(100-a,2)

st->op->cond
cond(yes)->e
cond(no)->ee
```
同时，我们应注意到$\it Tn(f,1) = f $，因此在第1次扔蛋后，蛋碎了，我们将直接得出确定性结果，如果蛋没碎，则问题转化为100-$\alpha_1$层楼的two-eggs问题，直接递归就可以求解了。说到这里，这道题其实就已经解决了，剩下的问题就是编程了。但是我们的要求要更高一些，通过数学推导，我们可以将时间复杂度降到O(1)，如何做到呢？

试想我们第一次在第n层扔鸡蛋，如果碎了，我们只剩一个蛋，接下来就只能从第一层开始向上遍历到第n-1层，直到鸡蛋摔碎，此时的worstcase次数为n-1，加上第一次扔，总次数为n；

如果没碎，我们应该向上跳跃n-1层取试，即取n+（n-1）层(这样能保证如果鸡蛋碎了,我们的worstcase即从第n+1层向上遍历到n+（n-1）层次数为n-2，加上第二次扔的一次，第一次的一次，总次数仍为n);

如果第2次扔没碎，则应取n+(n-1)+(n-2)层扔，仍没碎则取n+(n-1)+(n-2)+(n-3),……直到达到第n次向上取1层。

为了保证一定能找到临界楼层，需要满足：
<center>n+(n-1)+(n-2)+…+1>=100</center>
对n求解，向上取整，即得到最优方案的worstcase次数。如果你觉得以上论述不够严谨的话，我们可以通过数学公式进行推导论证：
> 首先注意到:<center>$ Tn(\alpha,1)=\alpha$</center>
> 第一次扔蛋后，我们可以得到最坏情况下次数满足不等式：
<center>$ Tn(100,2) = max(Tn(\alpha_1-1,1)+1,Tn(100-\alpha_1,2)+1)\leq\alpha_1 $
$\Downarrow$
$Tn(100-\alpha_i,2)\leq\alpha_1-1$
$\Downarrow$
将不等式(2)带回不等式(1)：
$ Tn(100-\alpha_1,2) = max(Tn(\alpha_2-1,1)+1,Tn(100-\alpha_1-\alpha_2,2)+1)\leq\alpha_2 \leq\alpha_1-1$
$\Downarrow$
…
$\Downarrow$
$Tn(100-\alpha_n,2) = max(Tn(\alpha \_n-1,1)+1,Tn(100-\alpha \_1-\alpha \_2…-\alpha \_n,2)+1)\leq\alpha \_n \leq\alpha \_{n -1}-1$
$\Downarrow$
$\alpha \_n\leq\alpha \_{n -1}-1…\leq\alpha \_1-(n-1)$
</center>
注意到，当$\alpha_n$为1时，达到最终状态，不再继续递归，另外式中还应有约束条件：$100-\alpha_1-\alpha_2…-\alpha_n>0$，即$\alpha_1+\alpha_2…-+\alpha_n$的值应**恰好**大于100。我们直接将$\alpha_n$取值为1，则根据最后一项不等式可推：
<center>100$\leq n+(n-1)+…+2+1\leq \alpha_1+\alpha_2…+\alpha_n$
$\Downarrow$
$1+2+…+n= \frac{n(n+1)}{2}\geq 100$</center>
解出n$\geq$13.651, 取n=14.


可以看到，此题的解题关键就是**分解出子问题，找到子问题之间的重叠关系，推出状态转移方程**。

有了以上方法论，相信n-eggs问题我们也能迎刃而解。

------

## n-eggs problem

### 以3-eggs为例

-   如果鸡蛋破碎，问题转化为k-1层下的2-eggs问题，状态转移方程为Tn(n,3)=1+Tn(k-1,2)

- 如果鸡蛋摔碎 ，问题转化为n-k层下的3-eggs问题，状态转移方程为Tn(n,3)=1+Tn(n-k,3)

 现在我们已经知道:
 >$Tn(n，1)=n$
 >$Tn(n，2)=min (x)    s.t.   \frac{x(x+1)}{2}\geq 100  $

因此我们只要需要递归求解，动态更新最坏情形下的次数即可。但是还有一个很重要的点需要注意，这一点也是我们为什么采用动态规划法的原因，我们在下一部分展开。

### n-eggs

其他将到这里，如果分解子问题，如何递推出状态转移方程已经不用多说了，直接将上部分Tn(f,e)中的3替换成未知量n,2替换成n-1即可。我们可以很容易的写出代码来，但如果仅仅如此，我们的代码很可能会内存溢出。

**内存溢出**的原因其实很好理解，这里不多解释。如何避免呢？这里就是动态规划法的优势所在了，**由于经分解得到子问题并不是互相独立的**。**我们如果保存已解决的子问题的答案，在需要时再找出已求得的答案，这样就可以避免大量的重复计算，节省时间**。这里我们在写代码时创建一个二维数组memory[F][E]来保存每次计算得出的Tn(f,e)的结果，并在函数入口处判断memory[f][e]是否存在，如果存在直接返回结果。

------

# 代码实现

这里我用的javascript实现，运用闭包存储计算结果。

```javascript
function throwNumber() {
    var memory=[];//声明一个存储结果的一维矩阵
    return function(floor,eggs){
        if(typeof(memory[floor])!='object') memory[floor]=new Array();//动态创建二维矩阵
        if( memory[floor][eggs])  {  return memory[floor][eggs];}//如果矩阵中存在结果，直接返回结果
        if(floor<3) {   memory[floor][eggs] = floor;return floor;}
        if(eggs==1) {   memory[floor][eggs] = floor;return floor;}
        if(eggs==2){ //计算2-eggs问题
            var sum=0,i=0;
            while(sum<floor){
                i++;
                sum=i*(i+1)/2;
            }
            memory[floor][eggs]=i;
            return i
        }
        if(eggs>2){ //计算n-eggs问题
            var res=floor;
            for(var i=2;i<floor;i++){
                var isBroken=arguments.callee(i-1,eggs-1)+1;
                var notBroken=arguments.callee(floor-i,eggs)+1;
                var temp=isBroken>notBroken?isBroken:notBroken;
                (res>temp) &&  (res =temp)>0;
            }
            memory[floor][eggs]=res
            return res;
        }
    }
}
//主函数
function Tn(f,e) {
    return throwNumber().call(this,f,e);
}
```
-----


# 附部分计算结果



floor    | 2-eggs|3-eggs|4-eggs|5-eggs
-| -----
50 | 10|7|6|6
100| 14|9|8|7
200 | 20|11|9|8
300 | 24|13|10|9




