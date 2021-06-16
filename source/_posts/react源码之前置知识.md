---
title: react源码之前置知识
date: 2021-05-26 20:07:22
tags:
- react
categories: react
author: 马金坤
keywords: react, 源码
description: 在react-native项目中实现沉浸式标题栏
cover: https://img12.360buyimg.com/imagetools/jfs/t1/192410/37/8360/23491/60c9b074Ea765d291/3e52890e0c12a36e.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/171645/8/14914/73312/60c9b074Efb29a81e/efd6e5e2b4a4eb99.jpg
---
react源码版本是17.0.2
## 0. react前置知识
react的基本理念是实现快速响应，实现上，是将同步的更新变为异步可中断、带优先级的更新。
### 0.1 源码架构
react17源码可以分为以下模块：
* Scheduler（调度器）： 排序优先级，让优先级高的任务先进行Reconciler
* Reconciler（协调器）： 找出哪些节点发生了改变，并打上不同的Tag，发生在render(更新渲染)阶段
* Renderer（渲染器）： 将Reconciler中打好标签的节点渲染到视图上，发生在commit(提交)阶段

#### Scheduler（调度器）
js是单线程，浏览器在同一时间内只能执行一个事件。若js代码执行时间过长，可能会导致以下两种情况：1）阻塞用户交互事件(比如点击事件)；2）阻塞浏览器绘制和渲染dom，造成卡帧、丢帧的现象。
> GUI渲染线程与JS引擎是互斥的，当JS引擎执行时，GUI线程会被挂起，GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行

Scheduler存在的主要作用就是为了解决上述问题:
1) 调度优先级，高优先级的任务可以打断低优先级的任务；
2) 时间切片，在事件循环中，分配一个时间片(5ms)给js执行，在这个时间片内，若是还没执行完，那就暂停js，把主线程交还给浏览器进行渲染工作，等下一轮事件继续执行js.

**浏览器一帧的工作内容：**  
![frame](frame.png)
js脚本 => requestAnimationFrame => 重排/重绘 => requestIdleCallback 
> 1）浏览器的刷新频率大多是60次/秒（帧率fps），即每一帧的时长大概是16.6ms，每隔16.6ms刷新一次屏幕  
> 2）requestAnimationFrame会在浏览器重新渲染之前执行，requestAnimationFrame执行间隔是由浏览器的刷新频率决定的
> 3）requestIdleCallback并不是每次都执行，而是在浏览器渲染之后，如果还有空闲时间，浏览器处于空闲状态(无更新渲染工作，无js任务)，才会执行requestIdleCallback。  


**可能会造成页面卡顿的原因**：（渲染间隔大于16ms，上一帧到下一帧的切换不自然）
* js 引擎线程耗时 → js 计算任务过大，执行时间过长，阻塞了 GUI 渲染线程

> 解决方案：降低 js 引擎线程耗时，使用算法进行优化(react的diff算法)，js任务异步可中断，使用web worker多线程
* GUI 渲染线程回流、重绘耗时 → js 修改的样式过多，布局、重绘耗时久

> 解决方案：尽量减少重排，重排一定会引起重绘，重绘不一定引起重排

**事件循环：**
![eventLoop](eventLoop.png)
宏任务(先执行同步代码) => 全部微任务 => 重排/重绘 => 执行下一轮宏任务 => 全部微任务 => 重排/重绘 => ...
1）所有同步任务都在主线程上执行，创建执行环境栈，用来临时保存正在执行函数的执行环境；  
2）js引擎遇到一个异步事件后，不会一直等待其返回结果，而会将这个事件挂起，继续执行执行栈中的其他任务；  
3）当一个异步事件返回结果后，js将这个事件回调加入到事件队列中（根据异步事件的类型，分为宏任务队列或者微任务队列）；  
4）被放入事件队列的异步事件不会立刻执行其回调函数，而是等待当前执行栈中的所有任务都执行完毕；  
5）当主线程执行栈处于闲置状态时，从微任务队列中取出任务，推入栈中执行；
6）微任务队列中的所有任务执行完毕之后，执行栈为空时，判断浏览器是否需要重新渲染，若是需要，由GUI线程接管渲染（不一定渲染）；  
7）**浏览器更新渲染完毕后，才会进行下一轮事件循环**，JS线程继续接管，先取宏任务队列中排在第一位的宏任务，完毕后，紧接着执行完所有的微任务，本轮结束之后，开始检查渲染；  
8）如此反复，这样就形成了一个无限的循环，这个过程称为“事件循环（Event Loop）”  
> 每次事件循环，宏任务之间，浏览器不一定都会重新渲染，可能取决于以下因素：
1）同步代码、宏任务、微任务是否有对页面样式进行修改
2）本次循环js脚本执行时间是否超过16ms；上次渲染时间是否超过16ms
3）宏任务之间的时间间隔短；在一帧的时间内，多次修改dom，浏览器可能会将其合并到一起更新渲染

**微任务（micro task）和宏任务（macro task）：**
* 宏任务（macro task）：
    主代码块、DOM操作、MessageChannel、setTimeout、setInterval
* 微任务（micro task）：可以理解是在当前task执行结束后立即执行的任务，比如Promise
   
请看下面代码：
```js
setTimeout(() => {
    console.log('宏任务：setTimeout1')
    new Promise(resolve => {
        resolve()
    }).then(() => {
        console.log('微任务：Promise.then3')
    })
    new Promise(resolve => {
        resolve()
    }).then(() => {
        console.log('微任务：Promise.then4')
    })
}, 0)
setTimeout(() => {
    console.log('宏任务：setTimeout2')
}, 0)
new Promise(resolve => {
    resolve()
}).then(() => {
    console.log('微任务：Promise.then1')
})
new Promise(resolve => {
    resolve()
}).then(() => {
    console.log('微任务：Promise.then2')
})
```
执行结果是
```
微任务：Promise.then1
微任务：Promise.then2
宏任务：setTimeout1
微任务：Promise.then3
微任务：Promise.then4
宏任务：setTimeout2
```

react通过宏任务来实现时间切片的：
* 宏任务是在下次事件循环中执行，不会阻塞浏览器的更新渲染；
* 浏览器更新渲染完成之后，才会执行下一轮宏任务；
* 下一轮宏任务执行时，预留5ms的时间片执行react的更新任务
宏任务中，MessageChannel的优先级大于setTimeout，支持MessageChannel的浏览器环境采用MessageChannel，不支持的话，采用setTimeout。

**MessageChannel**的实现如下：
```js
var channel = new MessageChannel();
var port = channel.port2;
requestHostCallback = function (callback) {
    port.postMessage(null); // 产生宏任务
};
// 宏任务的回调会在下一轮事件循环执行（浏览器未渲染完成之前，不会执行下一轮事件循环）
channel.port1.onmessage = performWorkUntilDeadline;
```

#### Reconciler（协调器，render阶段）
fiber（Virtual dom）是内存中用来描述dom结构的对象，保存DOM节点的属性、类型和dom信息；Fiber通过child、sibling、return（指向父节点）来形成Fiber树。
在render阶段，Reconciler会创建或者更新Fiber节点。初次渲染时，react会根据jsx生成的元素构建fiber对象；更新时，根据最新的jsx生成的元素和当前的current Fiber树做对比，构建workInProgress Fiber树，这个对比的过程就是diff算法。对比的过程中，react会给发生变化的fiber打上Tag标签，会形成一条effectList，标记更新的节点，在commit阶段把这些标签应用到真实dom上.

![fiber树](fiber树.png)
图中， fiberRootNode是整个项目的根节点，包含应用挂载的目标节点，记录整个应用更新过程的各种信息；rootFiber是当前应用挂载的节点，即ReactDOM.render调用后的根节点

### Renderer（渲染器）
Renderer是在commit阶段工作的，Renderer会遍历render阶段形成的effectList，根据Tag标签，执行真实的DOM操作。
```js
var Placement = 2;
var Update = 4;
var PlacementAndUpdate = 6;
var Deletion = 8;
```

### 0.2 JSX
jsx通过[babel编译器](https://babeljs.io/)转化为可执行的代码。在react中，jsx被编译为React.createElement 方法。
**jsx:**
```jsx
function Counter(props) {
  return (
    <div>
      <span>子节点</span>
    </div>
  )
}
function App(props) {
  return <Counter count="12" key="12" />;
}
ReactDOM.render(<App />, document.getElementById('app'))
```
**通过babel编译后的代码：**
```js
// 转义后的代码
function Counter(props) {
  return React.createElement("div", null, React.createElement("span", null, "\u5B50\u8282\u70B9"));
}
function App(props) {
  return React.createElement(Counter, {
    count: "12",
    key: "12"
  });
}
```
**createElement函数会生成element元素：**
```js
{
    $$typeof: REACT_ELEMENT_TYPE,
    key: null,
    props: {},
    ref: null,
    type: ƒ App(props),
}
```

**createElement**
ReactElement是通过createElement创建的，调用该方法需要传入三个参数：type、config 和 children。
1）type: ReactElement的类型
* 字符串，代表原生DOM，称为HostComponent，比如div，p
* Class类型，继承自Component或者PureComponent的组件，称为ClassComponent
* 方法，就是functional Component
* react原生组件，比如React.Fragment

2）config：props属性对象
3）children：子节点集合
```js
export function createElement(type, config, children) {
  // 处理参数
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props, // props 包含 children 信息，还包含除key、ref、self 和 source 以外的 config 信息
  )
}
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 用于确定是否属于ReactElement
    $$typeof: REACT_ELEMENT_TYPE,
    // ReactElement的类型
    type: type,
    key: key,
    ref: ref,
    props: props,
    // Record the component responsible for creating this element.
    _owner: owner,
  };

  return element
}
```
### 0.3 react优先级
react优先级：
* 事件优先级：按照用户事件的交互紧急程度，划分的优先级
* 更新优先级：事件导致React产生的更新对象（update）的优先级（update.lane）
* 任务优先级：产生更新对象之后，React去执行一个更新任务，这个任务所持有的优先级
* 调度优先级：Scheduler依据React更新任务生成一个调度任务，这个调度任务所持有的优先级

不同事件产生更新的优先级是不一样的，一个更新的产生导致react生成一个更新任务，不同优先级的更新会产生不同优先级的更新任务，最后这个更新任务被Scheduler调度执行。
交互事件 => 触发更新 => 更新任务 => Scheduler调度

#### 事件优先级
分为三种：
* 离散事件（DiscreteEvent）：click、keydown、focusin等，这些事件的触发不是连续的，优先级为0。
* 用户阻塞事件（UserBlockingEvent）：drag、scroll、mouseover等，特点是连续触发，阻塞渲染，优先级为1。
* 连续事件（ContinuousEvent）：video、audio标签的timeupdate(播放位置发生改变时触发)和canplay(音频/视频可以播放时触发)，优先级最高，为2。

#### 更新优先级lanes
```js
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const SyncBatchedLane: Lane = /*                 */ 0b0000000000000000000000000000010;

export const InputDiscreteHydrationLane: Lane = /*      */ 0b0000000000000000000000000000100;
const InputDiscreteLanes: Lanes = /*                    */ 0b0000000000000000000000000011000;

const InputContinuousHydrationLane: Lane = /*           */ 0b0000000000000000000000000100000;
const InputContinuousLanes: Lanes = /*                  */ 0b0000000000000000000000011000000;

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000100000000;
export const DefaultLanes: Lanes = /*                   */ 0b0000000000000000000111000000000;

const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000001000000000000;
const TransitionLanes: Lanes = /*                       */ 0b0000000001111111110000000000000;

const RetryLanes: Lanes = /*                            */ 0b0000011110000000000000000000000;

export const SomeRetryLane: Lanes = /*                  */ 0b0000010000000000000000000000000;

export const SelectiveHydrationLane: Lane = /*          */ 0b0000100000000000000000000000000;

const NonIdleLanes = /*                                 */ 0b0000111111111111111111111111111;

export const IdleHydrationLane: Lane = /*               */ 0b0001000000000000000000000000000;
const IdleLanes: Lanes = /*                             */ 0b0110000000000000000000000000000;

export const OffscreenLane: Lane = /*                   */ 0b1000000000000000000000000000000;
```
#### 更新任务优先级
```js
// ImmediatePriority$1
var SyncLanePriority = 15; // => SyncLane
var SyncBatchedLanePriority = 14; // => SyncBatchedLane
// UserBlockingPriority$2
var InputDiscreteHydrationLanePriority = 13;
var InputDiscreteLanePriority = 12;
var InputContinuousHydrationLanePriority = 11;
var InputContinuousLanePriority = 10;
// NormalPriority$1
var DefaultHydrationLanePriority = 9;
var DefaultLanePriority = 8;
var TransitionHydrationPriority = 7;
var TransitionPriority = 6;
var RetryLanePriority = 5;
var SelectiveHydrationLanePriority = 4;
// IdlePriority$1
var IdleHydrationLanePriority = 3;
var IdleLanePriority = 2;
var OffscreenLanePriority = 1;
// NoPriority$1
var NoLanePriority = 0;
```
#### sheduler调度优先级
* Immediate 立即执行优先级，需要同步执行的任务
* UserBlocking 用户阻塞型优先级（250 ms 后过期），需要作为用户交互结果运行的任务（例如，按钮点击）
* Normal 普通优先级（5 s 后过期），不必让用户立即感受到的更新
* Low 低优先级（10 s 后过期），可以推迟但最终仍然需要完成的任务（例如，分析通知）
* Idle 空闲优先级（永不过期），不必运行的任务（例如，隐藏界面以外的内容）

__sheduler中的调度优先级__
```js
var NoPriority = 0;
var ImmediatePriority = 1;
var UserBlockingPriority = 2;
var NormalPriority = 3;
var LowPriority = 4;
var IdlePriority = 5;
```
__sheduler调度优先级对应的过期时间__
```js
var IMMEDIATE_PRIORITY_TIMEOUT = -1; // Eventually times out
var USER_BLOCKING_PRIORITY_TIMEOUT = 250;
var NORMAL_PRIORITY_TIMEOUT = 5000;
var LOW_PRIORITY_TIMEOUT = 10000; // Never times out
var IDLE_PRIORITY_TIMEOUT = maxSigned31BitInt;
```
__sheduler在react-dom中调度优先级__
```js
var ImmediatePriority = 99;
var UserBlockingPriority = 98;
var NormalPriority = 97;
var LowPriority = 96;
var IdlePriority = 95;
var NoPriority = 90;
```
> 上面两个一一对应，例如NormalPriority=3对应NormalPriority$1=97


事件产生的优先级会记录到Scheduler，由Scheduler保存起来(currentPriorityLevel)；等到react创建更新时，计算更新的优先级直接从Scheduler中拿取


