---
title: 使用redux-thunk实现异步操作的中止
date: 2017-08-30 17:10:13
tags:
- react
- redux
- es6
categories: 前端
---

前端开发中常常会有这样的需求：设计一个select组件，每做一次选择就用ajax异步加载数据到指定页面。

实现这个需求并不困难，但是仅仅是这样做，难免会出现一些不太理想的体验：如果用户迅速的切换选项，那么返回的结果不一定是用户最后选择的那个结果。

因为请求是异步的，而请求发出到获得响应的过程时间是不可估的，用户很可能临时改变主意或等不及了而去发出一个新的请求。而页面最终呈现的结果是以最后到达的请求为准，因此具有一定的随机性。
<img src='http://7xwnfc.com1.z0.glb.clouddn.com/redux-doors.jpg'>
<!--more-->
这里使用redux的中间件thunk来改进action构造函数，达到‘中断‘用户最后一次之前的所有请求，只保留最后一次的效果。
> 关于中断和截流，月影有一篇关于函数式编程的PPT将的特别好，推荐大家去看一下，PPT中有嵌入JSBIN，大家可以直接在PPT中调试，[《函数式编程，你必须知道的那些事》](http://ppt.baomitu.com/display?slide_id=0bda92b8#/)

而这里要提到的方法跟上述PPT中的都不一样，而且同样巧妙。
这里预先定义三个action创造函数:

-	fetchDataStarted
-	fetchDataSuccess
-	fetchDataFailed

顾名思义，分别返回数据发起和数据返回成功／失败的action，展示组件将根据传入的props的特定的状态渲染不同的内容（即‘数据加载中···‘，‘数据加载成功即其内容‘，‘数据加载失败及其原因‘）。那么现在就是要对每点击一次要派发的那个action对应的构造函数进行改写：
```javascript
let nextSeqId = 0
export const fetchDataAction = (selectQuery) => {
	return (dispatch) {
		const apiUrl = 'data/select/${selectQuery}'
		const SeqId = ++nextSeqId
		const dispatchIfValid = action => {
			if（SeqId === nextSeqId）{
				return dispatch(action)
			}
		}
		// 将状态改为开始发起请求
		dispatchIfValid(fetchDataStarted)
		//  发起请求
		fetch(apiUrl).then((response) => {
			if(response.status !== 200) {
				throw new Error('Fail to get Data with status'+response.status)
			}
			response.json().then((responseJson) => {
				dispatchIfValid(fetchDataSuccess(responseJson.data))
			}).catch( error => {
				dispatchIfValid(fetchDataFailed(error))
			})
		}).catch( error => {
			dispatchIfValid(fetchDataFailed(error))
		})
	}
}
```
这里定义了一个文件模块级的变量nextSeqId, 给所有的dispatch函数加了个包装函数，在执行前先判断请求发起时定义的seqId是否和当前的nextSeqId相等，如果相等才执行。虽然不能真正的‘中止‘请求，但是可以通过这种方法让一个API请求被忽略，达到同样的效果。