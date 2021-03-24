---
title: reactHook之useState和useEffect的原理与简单实现
date: 2021-02-24 20:12:50
tags:
- react
- react hook
categories: react
author: 马金坤
keywords: react hook, useState, useEffect
description: react-native在Ios端提示用户App应用版本更新
cover: https://img13.360buyimg.com/imagetools/jfs/t1/165706/12/8850/11857/60364392E830fa141/8cc507f9b004a4a2.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/130200/34/17431/262954/5fc092d7E0b54491c/bb832c9742a8f536.png
---

### useState
#### 实现一个state
useState接收状态初始值，返回一个数组，第一个是状态的当前值，第二个是函数，用来更新状态.

以`const [count, setCount] = useState(0)`为例：  
组件第一渲染时，从useState拿到初始值0。当调用setCount(count+1)时，调用dispatch，state更新为1，之后执行render，组件重新渲染，此时拿到新的count（每一次都拿到独立的count状态，但是这个状态在一次渲染过程中是常量）
```js
let _state
function useState(initialState) {
	_state = _state || (typeof initialState === 'function' ? initialState() : initialState)
	const dispatch = action => {
		const curState = _state
		const newState = basicStateReducer(curState, action)
		if (curState === newState) return
		_state = newState
		render()
	}
	return [_state, dispatch]
}
function basicStateReducer(state, action) {
	return typeof action === 'function' ? action(state) : action
}
```
#### 实现多个state
若是多个state，该如何实现？  

通过数组，把多个state放在数组memoizedState中，根据索引index，更修改相应的state
* useState会读取memoizedState[index]
* index由useState调用的顺序决定
* setState会修改state，并触发组件更新
```js
let memoizedState = [] // hooks 存放在这个数组
let index = 0 // 当前 memoizedState 下标

function useState(initialState) {
	const currentIndex = index
	index++
	memoizedState[currentIndex] = memoizedState[currentIndex] || (typeof initialState === 'function' ? initialState() : initialState)
	const dispatch = action => {
		const curState = memoizedState[currentIndex]
		const newState = basicStateReducer(curState, action)
		if (curState === newState) return
		memoizedState[currentIndex] = newState
		render()
	}
	return [memoizedState[currentIndex], dispatch]
}
function basicStateReducer(state, action) {
	return typeof action === 'function' ? action(state) : action
}

const render = () => {
	index = 0
	ReactDOM.render(<App/>, rootElement)
}
```
从上面可以看出，useState要按顺序执行，不能使用条件语句，因为可能会打乱顺序。
注：源码中，useState是通过链表结构来实现的
```js
hook1 => Fiber.memoizedState 
state1 === hook1.memoizedState

hook1.next => hook2 
state2 === hook2.memoizedState
```
### 实现useEffect
useEffect接收两个参数。第一个是函数，放所需执行的代码；第二个参数是一个数组，里面是Effect的依赖项，数组发生变化，useEffect就会执行。第二个参数可以省略，每次渲染就会执行useEffect中的函数。

setState时，组件会重新渲染，此时会重新触发useEffect函数。（并不是count的值在“不变”的effect中发生了改变，而是effect函数本身在每一次渲染中都不相同）
```js
let effectHOOKS = []
let effectIndex = 0 // 当前effect的下标
function useEffect(callback, depArray) {
	const effect = effectHOOKS[effectIndex]
	const deps = effect && effect.deps
	const hasNoDeps = !depArray
	const hasChangedDeps = deps
		? !depArray.every((el, i) => el === deps[i])
		: true

	if (hasNoDeps || hasChangedDeps) {
		const destroy = effect && effect.destroy
		// 上一次的effect会在重新渲染后被清除掉
		destroy && typeof destroy === 'function' && destroy()
		// 执行本次的回调函数
		const callbackRes = callback()
		effectHOOKS[effectIndex] = {
			...effectHOOKS[effectIndex],
			deps: depArray, destroy: callbackRes
		}
	}
	effectIndex++
}
```
从上述代码可知，当depArray=[]时，useEffect中的callback回调函数只执行一次。上一次的effect只会在重新渲染后被清除掉。
有问题的定时器组件写法，count永远是0。
```js
const [count, setCount] = useState(0)
useEffect(() => {
    const id = setInterval(() => {
        // count永远是0
        setCount(count + 1)
    }, 1000)
    return () => clearInterval(id)
}, [])
```
优化：
```js
useEffect(() => {
    const id = setInterval(() => {
        // count永远是0
        setCount(count + 1)
    }, 1000)
    return () => clearInterval(id)
}, [count])
=> 
useEffect(() => {
    const id = setInterval(() => {
        // count传入的是当前state的值
        setCount(count => count + 1)
    }, 1000)
    return () => {
        clearInterval(id)
        console.log('clearInterval')
    }
}, [])
```
