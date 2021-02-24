---
title: reactHook之useState的源码解析
date: 2021-02-24 20:18:24
tags:
- react
- react hook
categories: react
author: 马金坤
keywords: react hook, useState
description: react-native在Ios端提示用户App应用版本更新
cover: https://img13.360buyimg.com/imagetools/jfs/t1/165706/12/8850/11857/60364392E830fa141/8cc507f9b004a4a2.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/130200/34/17431/262954/5fc092d7E0b54491c/bb832c9742a8f536.png
---
下面分析16.14.2版本react中useState的源码实现
### 介绍
#### 数据结构
当前fiber节点的数据结构是：
```js
const fiberNode = {
	tag: xxx, // 标记不同的组件类型
	key: xxx,
	memoizedState: { // hooks
		baseState: xxx, // 初始化 initialState， 每次 dispatch 之后的newState
		memoizedState: xxx, // 上次更新完之后的最终值
		queue: { // 缓存的更新队列，
			lastRenderedReducer: basicStateReducer(state, action),
			lastRenderedState: xxx, // 上次state的值
			pending: null // 存放即将更新的newState信息
		},
		next: null, // link到下一个hooks，通过next串联每一个hooks
	}
}
```
React 的 Hooks 是一个单向链表，Hook.next 指向下一个 Hook。
```js
Fiber.memoizedState => hook1  state1 === hook1.memoizedState 
hook1.next => hook2           state2 === hook2.memoizedState
hook2.next => hook3           state3 === hook2.memoizedState
```
#### 入口文件
react/cjs/react.development.js
```js
function useState(initialState) {
  var dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
```
resolveDispatcher返回的是ReactCurrentDispatcher.current，所以`useState = ReactCurrentDispatcher.current.useState`
### 首次渲染
`ReactDOM.render(<App/>, rootElement)`，react从render开始执行
```
render => ... => beginWork => updateFunctionComponent =>  renderWithHooks
```
hooks的核心渲染逻辑入口是renderWithHooks。
#### renderWithHooks
```js
// 14762
renderWithHooks()
    // 14787 核心逻辑
    if (current !== null && current.memoizedState !== null) {
      ReactCurrentDispatcher.current = HooksDispatcherOnUpdateInDEV;
    } else { // 组件首次挂载时
      ReactCurrentDispatcher.current = HooksDispatcherOnMountInDEV;
    }
    =>
    15694 HooksDispatcherOnMountInDEV.useState = {
      return mountState(initialState) // 15766
    }
    15881 HooksDispatcherOnUpdateInDEV.useState = {
      return updateState(initialState) // 15939
    }
    // 14851
    currentlyRenderingFiber$1 = null;
    currentHook = null;
    workInProgressHook = null;
}
```
从上述代码可知，首次挂载时，`useState = HooksDispatcherOnMountInDEV.useState = mountState`;state更新时，`useState = HooksDispatcherOnUpdateInDEV.useState = mountState`
#### mountState
```js
// 15213 第一次调用组件的 useState 时实际调用的方法
function mountState(initialState) {
  var hook = mountWorkInProgressHook(); // / 创建一个新的 Hook，并返回当前 workInProgressHook

  if (typeof initialState === 'function') {
    initialState = initialState();
  }

  hook.memoizedState = hook.baseState = initialState;
  var queue = hook.queue = {
    pending: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: initialState
  };
  // 绑定当前 fiber 和 queue 到 dispatchAction 上
  // currentlyRenderingFiber$1是一个全局变量，表示当前正在渲染的Fiber节点
  var dispatch = queue.dispatch = dispatchAction.bind(null, currentlyRenderingFiber$1, queue);
  // const [name, setName] = useState('king')
  // king 赋值给了 hook.memoizedState， setName表示dispatch，当调用setName('queen')会执行dispatchAction('queen')
  return [hook.memoizedState, dispatch];
}
```
（1）执行mountWorkInProgressHook()，更新hook链表，并返回当前 workInProgressHook
```js
// 14921 创建一个新的 hook，并返回当前 workInProgressHook
function mountWorkInProgressHook() {
  var hook = {
    memoizedState: null,
    baseState: null,
    baseQueue: null,
    queue: null,
    next: null
  };
  if (workInProgressHook === null) {
		// 只有在第一次打开页面的时候，workInProgressHook 为空
    currentlyRenderingFiber$1.memoizedState = workInProgressHook = hook;
  } else {
		// 已经存在 workInProgressHook 就将新创建的这个 Hook 接在 workInProgressHook 的尾部
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```
（2）获取当前的workInProgressHook后，初始化了hook.memoizedState、hook.baseState、hook.queue和queue.dispatch

（3）basicStateReducer的源码如下：
```js
// 15007
function basicStateReducer(state, action) {
  return typeof action === 'function' ? action(state) : action;
}
```
（4）最终返回一个数组 `[hook.memoizedState, dispatch]`
执行`setName('queen')`，其实就是执行`dispatch('queen')`

第一次`const [name, setName] = useState('king')`，mountWorkInProgressHook()后，hook的值为
```js
hook = {
	baseQueue: null,
	baseState: 'king',
	memoizedState: 'king',
	next: null,
	queue: {
            dispatch: fucntion,
            lastRenderedReducer: basicStateReducer(state, action),
            lastRenderedState: 'king',
            pending: null
	}
}
```
第二次`const [count, setCount] = useState(0)`，mountWorkInProgressHook()后，hook的值为
```js
hook = {
	baseQueue: null,
	baseState: 'king',
	memoizedState: 'king',
	queue: {
        dispatch: fucntion,
		lastRenderedReducer: basicStateReducer(state, action),
		lastRenderedState: 'king',
		pending: null
	},
	next: {
		baseQueue: null,
		baseState: 0,
		memoizedState: 0,
		queue: {
            dispatch: fucntion,
			lastRenderedReducer: basicStateReducer(state, action),
			lastRenderedState: 0,
			pending: null
		},
	},
}
```
#### dispatchAction
以`const [name, setName] = useState('king') setName('queen')`为例：

执行`setName('queen')`，其实就是执行`dispatchAction(fiber, queue, 'queen')`

* dispatchAction() 会创建 update 对象({action:'queen'})
* 将 update 加至 hook.queue 的末尾：hook.queue.pending = update
* 执行 scheduleWork()，走 updateFunctionComponent() 流程
核心代码：
```js
// 15564
function dispatchAction(fiber, queue, action) {
  // action 就是传进来要更新的 state->'queen'
  var update = {
    expirationTime: expirationTime,
    suspenseConfig: suspenseConfig,
    action: action,
    eagerReducer: null,
    eagerState: null,
    next: null
  };
  // 这里的queue，是之前传入的hook对象中的queue，这里保留了一个引用！！（即queue发生变化，当前fiber节点的hook数据也是同步变更的）
  var pending = queue.pending;
  if (pending === null) {
    // This is the first update. Create a circular list.
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  //  将update对象加至hook.queue的末尾pending中
  queue.pending = update;

  var currentState = queue.lastRenderedState;
  // currentState: king，action: queen
  // 将新的state queen 赋值到 update.eagerState
  var eagerState = lastRenderedReducer(currentState, action); 
  update.eagerReducer = lastRenderedReducer;
  update.eagerState = eagerState;
  // ...
  scheduleWork(fiber, expirationTime)
}
```
`setName('queen')`，即执行dispatchAction()后，hook变为
```js
hook = {
	baseQueue: null,
	baseState: 'king',
	memoizedState: 'king',
	queue: {
		lastRenderedReducer: basicStateReducer(state, action),
		lastRenderedState: 'king',
		pending: {
			action: 'queen',
			eagerState: 'queen',
			next: pending // pedding对象，单链表
		}
	},
	next: {
		baseQueue: null,
		baseState: 0,
		memoizedState: 0,
		queue: {
			lastRenderedReducer: basicStateReducer(state, action),
			lastRenderedState: 0,
			pending: null
		},
	},
}
```
最后会执行scheduleWork(fiber, expirationTime)，经过React的调度，会带上action（setName的传参），再次进入hook组件核心渲染逻辑：renderWithHooks。
```
dispatchAction => scheduleUpdateOnFiber => ensureRootIsScheduled => performConcurrentWorkOnRoot => performUnitOfWork => beginWork => updateFunctionComponent 
=> renderWithHooks
```
此时，由于并非首次渲染组件，React会使用HooksDispatcherOnUpdateInDEV对象上的useState，`useState = HooksDispatcherOnUpdateInDEV.useState = mountState`。在这个useState中，会使用一个叫做updateState的函数来更新最新的state值。
```js
// 14762
renderWithHooks()
    // 14787 核心逻辑
    if (current !== null && current.memoizedState !== null) {
      ReactCurrentDispatcher.current = HooksDispatcherOnUpdateInDEV;
    } else { // 组件首次挂载时
      ReactCurrentDispatcher.current = HooksDispatcherOnMountInDEV;
    }
    =>
    15694 HooksDispatcherOnMountInDEV.useState = {
      return mountState(initialState) // 15766
    }
    15881 HooksDispatcherOnUpdateInDEV.useState = {
      return updateState(initialState) // 15939
    }
    // ...
    var children = Component(props, secondArg)
}
```
renderWithHooks()中有一个Component()方法，用来执行App()，此时又会执行 `const [name, setName] = useState('king')`，最终调用的是`HooksDispatcherOnUpdateInDEV.useState(king') => updateState('king')`。
### 更新
#### updateState()
由updateState可以看出，实际调用的是 updateReducer
```js
// 15233
function updateState(initialState) {
  return updateReducer(basicStateReducer);
}
function basicStateReducer(state, action) {
  return typeof action === 'function' ? action(state) : action;
}
```
#### updateReducer()
updateReducer的核心代码：
```js
// 15033
funtion updateReducer() {
    var hook = updateWorkInProgressHook();
    var queue = hook.queue;
    var pendingQueue = queue.pending
    current.baseQueue = baseQueue = pendingQueue;
    queue.pending = null;
    var first = baseQueue.next;
    var newState = current.baseState;
    var update = first;
    if (update.eagerReducer === reducer) {
      newState = update.eagerState;
    } 
    hook.memoizedState = newState;
    hook.baseState = newBaseState;
    var dispatch = queue.dispatch;
    return [hook.memoizedState, dispatch]; // 返回['queen', dispatch]
}
```
（1）第一步执行updateWorkInProgressHook()，获取当前正在工作中的 hook 
```js
// 14941
function updateWorkInProgressHook() {
  // 此时currentlyRenderingFiber$1.memoizedState，但是为fiber的副本，保留着fiber.memoizedState的内容
  // 核心代码
   var nextCurrentHook;
  if (currentHook === null) { // 第一次useState
    var current = currentlyRenderingFiber$1.alternate;
    if (current !== null) {
      nextCurrentHook = current.memoizedState;
    } else {
      nextCurrentHook = null;
    }
  } else {
    nextCurrentHook = currentHook.next;
  }
  var nextWorkInProgressHook;
  if (workInProgressHook === null) { // 第一次useState
    nextWorkInProgressHook = currentlyRenderingFiber$1.memoizedState;
  } else {
    nextWorkInProgressHook = workInProgressHook.next;
  }
  currentHook = nextCurrentHook;
  var newHook = {
    memoizedState: currentHook.memoizedState,
    baseState: currentHook.baseState,
    baseQueue: currentHook.baseQueue,
    queue: currentHook.queue,
    next: null
  }
  if (workInProgressHook === null) {
    currentlyRenderingFiber$1.memoizedState = workInProgressHook = newHook;
  } else {
    workInProgressHook = workInProgressHook.next = newHook;
  }
  return workInProgressHook
}
```
第一次`const [name, setName] = useState('king')`，workInProgressHook的值为
```js
workInProgressHook={
	memoizedState: "king",
	baseState: "king",
	queue:{
		pending:{
			action: "queen",
			eagerState: "queen",
		}
	},
	next: null
}
```
第二次`const [count, setCount] = useState(0)`，workInProgressHook的值为
```js
workInProgressHook={
	memoizedState: 0,
	baseState: 0,
	queue:{
		pending:{
			action: 0,
			eagerState: 0,
		}
	},
	next: null
}
```
（2）从workInProgressHook的数据结构可以看出，我们需要更新的值就在queue.pending.eagerState/action中
```js
if (update.eagerReducer === reducer) {
   newState = update.eagerState;
} 
```
（3）更新state
```js
hook.memoizedState = newState;
hook.baseState = newBaseState;
var dispatch = queue.dispatch;
return [hook.memoizedState, dispatch]; // 返回['queen', dispatch]
```
将`'queen'`赋值给`hook.memoizedState`，返回`['queen', diapatch]`，此时的`name`已经更新为`'queen'`

由上述源码可知，useState按顺序执行的原因是，useState是通过next来查找下一个hooks
