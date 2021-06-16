---
title: react源码之react入口函数
date: 2021-05-31 10:31:03
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
## 1. react 入口
react入口函数有三种模式：
* legacy(传统)模式： `ReactDOM.render(, rootNode)`；**LegacyRoot = 0**。
> 这是当前 React app 使用的方式。构建dom的过程是同步的，所以在render阶段，如果diff特别耗时，会导致js一直阻塞高优先级的任务(例如用户的点击事件)，表现为页面的卡顿，无法响应。
* blocking(阻塞)模式： `ReactDOM.createBlockingRoot(rootNode).render()`；**BlockingRoot = 1**。目前正在实验中。作为迁移到 concurrent 模式的第一个步骤。
> 最新的react文档指出，下面所有关于 “blocking 模式” 和 createBlockingRoot 的说法都已过时，应忽略。
* concurrent(并发)模式： `ReactDOM.createRoot(rootNode).render()`；**ConcurrentRoot = 2**。
> 目前在实验中，未来稳定之后，打算作为 React 的默认开发模式。将长任务分成一个个小任务，每一帧都预留出时间片(5ms)供react执行js代码，通过调度实现异步可中断、带优先级的更新任务.高优先级的任务可以打断低优先级任务。当前时间片时间用完，就暂停react的任务，将主线程控制权交还给浏览器使其有时间渲染 UI。react等待下一帧的时间片，再继续被打断的任务。

下图为入口函数的执行过程：
![入口函数](入口函数.png)
### 1.1 react 入口函数
#### legacy模式入口
`ReactDOM.render(<App />, rootElement)`
```js
function render(element, container, callback) {
  return legacyRenderSubtreeIntoContainer(null, element, container, false, callback);
}
function legacyRenderSubtreeIntoContainer(parentComponent, children, container, forceHydrate, callback) {
  var root = container._reactRootContainer;
  var fiberRoot;
  // 第一次执行ReactDOM.render时，root为null
  if (!root) {
    // 创建根DOM实例，里面有_internalRoot属性，存放fiberRoot
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      var originalCallback = callback;
      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // 首次渲染不进行批量更新
    unbatchedUpdates(function () {
      // children 是传入ReactDOM.render方法的第一参数，react元素<App/>
       children = element = {
          $$typeof: Symbol(react.element),
          key: null,
          props: {},
          ref: null,
          type: App
        }
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      var _originalCallback = callback;
      callback = function () {
        var instance = getPublicRootInstance(fiberRoot);
        _originalCallback.call(instance);
      };
    } // Update
    updateContainer(children, fiberRoot, parentComponent, callback);
  }

  return getPublicRootInstance(fiberRoot);
}
function legacyCreateRootFromDOMContainer(container, forceHydrate) {
  // 判断是否是服务器端渲染	
  // 是否需要复用原先的dom节点，再合并（为true表示需要有子节点且是服务器端渲染
  var shouldHydrate = forceHydrate || shouldHydrateDueToLegacyHeuristic(container); 
  // 如果不是服务端渲染
  if (!shouldHydrate) {
    var rootSibling;
    // 把container（即传入div#app）的子节点全部删除
    while (rootSibling = container.lastChild) {
      container.removeChild(rootSibling);
    }
  }
  return createLegacyRoot(container, shouldHydrate ? {
    hydrate: true
  } : undefined);
}
function createLegacyRoot(container, options) {
  // LegacyRoot = 0
  return new ReactDOMBlockingRoot(container, LegacyRoot, options);
}
function ReactDOMBlockingRoot(container, tag, options) {
  this._internalRoot = createRootImpl(container, tag, options);
}
```
#### concurrent模式入口
`ReactDOM.createRoot(rootNode).render(<App/>)`
```js
function createRoot(container, options) {
  return new ReactDOMRoot(container, options);
}
// Concurrent 模式，ConcurrentRoot = 2
function ReactDOMRoot(container, options) {
  this._internalRoot = createRootImpl(container, ConcurrentRoot, options);
}
/*
  ReactDOM.unstable_createRoot(document.getElementById('root')).render(<App />)
  ReactDOM.unstable_createRoot(document.getElementById('root')) 返回ReactDOMRoot实例
*/
ReactDOMRoot.prototype.render = ReactDOMBlockingRoot.prototype.render = function (children) {
  var root = this._internalRoot;
  updateContainer(children, root, null, null);
}
```
#### 创建fiberRoot 和 rootFiber的公共逻辑
* 调用createRootImpl，会调用到createFiberRoot创建fiberRootNode，然后调用createHostRootFiber创建rootFiber
> 其中fiberRootNode是整个项目的根节点，包含应用挂载的目标节点，记录整个应用更新过程的各种信息
> rootFiber是当前应用挂载的节点，即ReactDOM.render调用后的根节点
* fiberRoot和rootFiber的关系
> fiberRoot.current = rootFiber
  rootFiber.stateNode = fiberRoot
  rootFiber.child = fiber

```js
function createRootImpl(container, tag, options) {
  // 是否是服务器端渲染
  var hydrate = options != null && options.hydrate === true;
  var hydrationCallbacks = options != null && options.hydrationOptions || null;
  // Legacy模式下，tag=0；Concurrent 模式下，tag=2
  var root = createContainer(container, tag, hydrate);
  // 事件挂载
  {
    var rootContainerElement = container.nodeType === COMMENT_NODE ? container.parentNode : container;
    // 监听所有原生事件，暂不细讲
    listenToAllSupportedEvents(rootContainerElement);
  }
  return root;
}
// Legacy模式下，tag=0；Concurrent 模式下，tag=2
function createContainer(containerInfo, tag, hydrate, hydrationCallbacks) {
  return createFiberRoot(containerInfo, tag, hydrate);
}
// Legacy模式下，tag=0；Concurrent 模式下，tag=2
function createFiberRoot(containerInfo, tag, hydrate, hydrationCallbacks) {
  // 获取fiberRoot
  var root = new FiberRootNode(containerInfo, tag, hydrate);
  // 获取rootFiber，Legacy模式下，tag=0；Concurrent 模式下，tag=2
  var uninitializedFiber = createHostRootFiber(tag);
  // fiberRoot和rootFiber相关联
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;
  /*初始化rootFiber的更新队列updateQueue
  uninitializedFiber.updateQueue = {
      baseState: fiber.memoizedState,
      baseQueue: null,
      shared: {
        pending: null
      },
      effects: null
  }*/
  initializeUpdateQueue(uninitializedFiber);
  return root;
}
// 初始化更新队列
function initializeUpdateQueue(fiber) {
  var queue = {
    baseState: fiber.memoizedState,
    firstBaseUpdate: null,
    lastBaseUpdate: null,
    shared: {
      pending: null
    },
    effects: null
  };
  fiber.updateQueue = queue;
}
// Legacy模式下，tag=0；Concurrent 模式下，tag=2
function FiberRootNode(containerInfo, tag, hydrate) {
    this.tag = tag; // 标记不同组件类型，如函数组件、类组件、文本、原生组件...
    this.current = null; // FiberRoot.current = RootFiber
    this.containerInfo = containerInfo; // 与fiberRoot关联的DOM容器信息,render方法接收的第二个参数
    this.callbackNode = null;
    this.callbackPriority = NoLanePriority;
    this.eventTimes = createLaneMap(NoLanes); // 更新任务开始时间，是长度为31的数组，初始值是[0,0,...]
    this.expirationTimes = createLaneMap(NoTimestamp); // 更新任务过期时间，是长度为31的数组，初始值是[-1,-1,...]
    this.pendingLanes = NoLanes; // 未执行任务的lanes
    this.suspendedLanes = NoLanes; // 挂起任务的lanes
    this.pingedLanes = NoLanes; // 挂起后恢复的任务lanes
    this.expiredLanes = NoLanes; // 过期任务的lanes
    this.finishedLanes = NoLanes; // 已完成任务的lanes
}
// Legacy模式下，tag=0；Concurrent 模式下，tag=2
function createHostRootFiber(tag) {
/*
var NoMode = 0;
var StrictMode = 1; 
var BlockingMode = 2;
var ConcurrentMode = 4;
var ProfileMode = 8;*/
  var mode;
  if (tag === ConcurrentRoot) {
    mode = ConcurrentMode | BlockingMode | StrictMode;
  } else if (tag === BlockingRoot) {
    mode = BlockingMode | StrictMode;
  } else {
    mode = NoMode;
  }
  if ( isDevToolsPresent) { // 处于开发者工具模式，mode = ProfileMode = 8
    mode |= ProfileMode;
  }
  // HostRoot = 3，根组件的tag值
  return createFiber(HostRoot, null, null, mode);
}
var createFiber = function (tag, pendingProps, key, mode) {
  return new FiberNode(tag, pendingProps, key, mode);
};
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag; // 标记不同组件类型，如函数组件、类组件、文本、原生组件...
  this.key = key; // react 元素上的 key
  this.elementType = null; // createElement的第一个参数，ReactElement 上的 type
  this.type = null;  // 表示fiber的真实类型 ，跟 elementType 基本一样，在使用了懒加载之类的功能时可能会不一样
  this.stateNode = null; // 实例对象，比如 class 组件 new 完后就挂载在这个属性上面，如果是RootFiber，那么它上面挂的是 FiberRoot,如果是原生节点就是 dom 对象
  // Fiber
  this.return = null; // 父节点，指向上一个 fiber
  this.child = null; // 子节点，指向自身下面的第一个子 fiber
  this.sibling = null; // 兄弟组件, 指向一个兄弟节点
  this.index = 0; //  一般如果没有兄弟节点的话是0；当某个父节点下的子节点是数组类型的时候会给每个子节点一个 index，index 和 key 要一起做 diff
  this.ref = null; // reactElement 上的 ref 属性
  this.pendingProps = pendingProps; // 新的 props
  this.memoizedProps = null; // 旧的 props
  this.updateQueue = null; // fiber 上的更新队列执行一次 setState，就会往这个属性上挂一个新的更新, 每条更新最终会形成一个链表结构，最后做批量更新
  this.memoizedState = null;  // 对应  memoizedProps，上次渲染的 state，相当于当前的 state
  this.mode = mode; // 表示当前组件下的子组件的渲染方式
  // Effects
  this.flags = NoFlags; // 表示当前 fiber 要进行何种更新（更新、删除等）
  this.nextEffect = null;  // 指向下个需要更新的fiber
  this.firstEffect = null;  // 指向所有子节点里，需要更新的 fiber 里的第一个
  this.lastEffect = null;  // 指向所有子节点中需要更新的 fiber 的最后一个
  this.lanes = NoLanes; // 任务优先级
  this.childLanes = NoLanes; // child 任务优先级
  this.alternate = null; // current 树和 workInprogress 树之间的相互引用
}
function initializeUpdateQueue(fiber) {
  var queue = {
    baseState: fiber.memoizedState,
    baseQueue: null,
    shared: {
      pending: null
    },
    effects: null
  };
  fiber.updateQueue = queue;
}
```
#### 补充
**执行非批量更新的操作 unbatchedUpdates** 
* 保存初始上下文，增加非批量上下文，删除批量更新上下文， 执行函数， 重置上下文，重置时间和执行同步队列  
* 非批量更新下，会走同步更新逻辑，因为是第一次更新，需要用户尽快看到页面内容，不走异步更新的逻辑  
* 执行完fn，需要恢复之前的执行环境 prevExecutionContext

```js
// react transaction 事务机制
function unbatchedUpdates(fn, a) {
    // 初始 executionContext = NoContext = 0
    // NoContext = 0b000000 = 0
    // BatchedContext = 0b000001 = 1
    // LegacyUnbatchedContext = 0b001000 = 8
    var prevExecutionContext = executionContext;
    // executionContext &= ~BatchedContext的效果其实就是还原了上次executionContext |= BatchedContext;
    // 去除批量更新的标记 假设executionContext=0b000001，~BatchedContext=11111110，0b000001&11111110=0
    executionContext &= ~BatchedContext; // ~00000001=11111110；00000000&11111110=00000000；带符号的二进制转换成十进制，保留符号位取反加1，11111110取反10000001，加1得10000010=-2
    // 更新executionContext为非批量 LegacyUnbatchedContext
    executionContext |= LegacyUnbatchedContext; // 00000000&00001000=00001000=8
    try {
      return fn(a);
    } finally {
      // 在执行完回调之后，会恢复上下文
      executionContext = prevExecutionContext;
    }
}
```

### 1.2 创建更新 updateContainer
* 获取rootFiber
* 获取当前时间
* 获取更新优先级lane 
* 创建更新
* 将更新加入更新队列
* 更新任务调度

```js
// container为fiberRoot，第一次，parentComponent=null
function updateContainer(element, container, parentComponent, callback) {
  var current$1 = container.current; // 获取rootFiber
  var eventTime = requestEventTime(); // 获取当前时间
  var lane = requestUpdateLane(current$1);
  // 创建更新
  var update = createUpdate(eventTime, lane);
  /* rootFiber的update.payload存放的不是更新的state
     而是存放的ReactDOM.render中传入的第一个参数，即通过jsx的createElement生成的元素
     element = {
        $$typeof: REACT_ELEMENT_TYPE,
        key: null,
        props: {},
        ref: null,
        type: ƒ App(props)
     }
    update.payload = {element}
  */
  update.payload = {
    element: element
  };
  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    update.callback = callback;
  }
  enqueueUpdate(current$1, update);
  scheduleUpdateOnFiber(current$1, lane, eventTime);
  return lane;
}
```
1）获取当前时间  
```js
function requestEventTime() {
  // 执行完unbatchedUpdates之后，executionContext = LegacyUnbatchedContext = 00001000 = 8，非批量更新
  // RenderContext = 00010000 = 16
  // CommitContext = 00100000 = 32
  // 在render和commit阶段，直接获取当前真实时间
  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    return now();
  }
  // 其他阶段，对所有更新使用相同的开始时间，除非重新进入react
  // 有currentEventTime，返回currentEventTime
  if (currentEventTime !== NoTimestamp) {
    return currentEventTime;
  } 
  // 没有currentEventTime，初次更新，没有任务，计算一个新的当前时间并赋给全局变量。
  currentEventTime = now();
  return currentEventTime;
}
// performance.now(), 从 page load 的时候从零开始计数,单位是ms
var initialTimeMs = performance.now()
// 如果初始时间相当小，小于10s，则now为当前时间；否则，当前时间减去初始时间 initialTimeMs
var now = initialTimeMs < 10000 ? performance.now : function () {
  return performance.now() - initialTimeMs;
};
```
2）获取更新优先级lane，requestUpdateLane
```js
function requestUpdateLane(fiber) {
  /*
    入口为ConcurrentRoot， mode = ConcurrentMode | BlockingMode | StrictMode
    入口为BlockingRoot， mode = BlockingMode | StrictMode
    其他情况，mode = NoMode
  */
  var mode = fiber.mode; // Legacy模式下是0，开发者模式下是8
  // BlockingMode = 8，ConcurrentMode = 16
  if ((mode & BlockingMode) === NoMode) { // 不是BlockingMode 模式，说明是Legacy模式，同步更新
    // 同步的，旧模式中lane只为SyncLane = 1
    return SyncLane;
  // BlockingMode 模式
  } else if ((mode & ConcurrentMode) === NoMode) { // 非ConcurrentMode模式
  	// 通过获取当前优先级getCurrentPriorityLevel 判断，是同步还是批量同步更新
    return getCurrentPriorityLevel() === ImmediatePriority$1 ? SyncLane : SyncBatchedLane; // 同步或同步批量
  } 
  // 以下是Concurrent模式的代码逻辑
  // 获取当前fiberRoot所拥有的lanes（已占用的赛道）；刚加载时，workInProgressRootIncludedLanes=0
  if (currentEventWipLanes === NoLanes) {
    currentEventWipLanes = workInProgressRootIncludedLanes;
  }
  // 获取调度优先级
  // getCurrentPriorityLevel() => return currentPriorityLevel
  // 刚开始，currentPriorityLevel = NormalPriority(3) <=> NormalPriority$1(97)
  var schedulerPriority = getCurrentPriorityLevel(); 
  var lane;
  // 若是事件产生的更新，且是离散事件的执行环境(click、keydown、focus等不连续事件)，且当前的调度优先级是用户阻塞型优先级
  // InputDiscreteLanePriority = 12
  if ( 
  (executionContext & DiscreteEventContext) !== NoContext && schedulerPriority === UserBlockingPriority$2) {
    lane = findUpdateLane(InputDiscreteLanePriority, currentEventWipLanes);
  } else {
    // 调度优先级转换为更新任务优先级
    var schedulerLanePriority = schedulerPriorityToLanePriority(schedulerPriority);
    lane = findUpdateLane(schedulerLanePriority, currentEventWipLanes);
  }

  return lane;
}
/*
根据更新任务优先级lanePriority获取优先级lane（赛车的初始赛道）
从lanePriority对应高优先级lane开始，从高到低，递归获取未被占用的lane
*/
function findUpdateLane(lanePriority, wipLanes) {
  switch (lanePriority) {
    case NoLanePriority:
      break;
    case SyncLanePriority:
      return SyncLane;
    case SyncBatchedLanePriority:
      return SyncBatchedLane;
    case InputDiscreteLanePriority:
      {
        var _lane = pickArbitraryLane(InputDiscreteLanes & ~wipLanes); // lanes & -lanes
        if (_lane === NoLane) {
          // 上一个优先级的InputDiscreteLanes被占满，下移到到相邻的较低优先级范围内寻找空闲的lane
		  // 从高到低，循环遍历没被占用的赛道
          return findUpdateLane(InputContinuousLanePriority, wipLanes);
        }
        // _lane不等于0，说明InputDiscreteLanes没有被占满
        return _lane;
      }
    case InputContinuousLanePriority:
      {
        var _lane2 = pickArbitraryLane(InputContinuousLanes & ~wipLanes);
        if (_lane2 === NoLane) {
          return findUpdateLane(DefaultLanePriority, wipLanes);
        }
        return _lane2;
      }
    case DefaultLanePriority:
      {
        /* 
           DefaultLanes & ~wipLanes，从DefaultLanes中去除已被占用的lanes
           pickArbitraryLane(lanes): 找到lanes中最靠左的那个1所对应的lanes值，找到lanes中优先级最高lane
           lanes 为0时，pickArbitraryLane(lanes)才会等于0
        */
        var _lane3 = pickArbitraryLane(DefaultLanes & ~wipLanes);
        if (_lane3 === NoLane) {  
          // 若DefaultLanes以前的lanes都被占满，则到TransitionLanes寻找空闲的lane，TransitionLanes的可用位数更多
          _lane3 = pickArbitraryLane(TransitionLanes & ~wipLanes);
          // 若连TransitionLanes都被占用了，那么这时候赋值一个默认优先级级别的lane，可能会打断正在进行更新任务
          if (_lane3 === NoLane) {
            _lane3 = pickArbitraryLane(DefaultLanes);
          }
        }
        return _lane3;
      }
    case TransitionPriority: 
    case RetryLanePriority:
      break;
    case IdleLanePriority:
      var lane = pickArbitraryLane(IdleLanes & ~wipLanes);

      if (lane === NoLane) {
        lane = pickArbitraryLane(IdleLanes);
      }
      return lane;
  }
} 

/*
  找到lanes中最靠左的那个1所对应的lanes值，lanes中，1越靠左，个数越少，优先级越高
  ~a: 取反；-a：取反+1
  例如：0110 & -0110 = 0010
*/
function pickArbitraryLane(lanes) {
  /*
    getHighestPriorityLane(lanes) => lanes & -lanes
  */
  return getHighestPriorityLane(lanes);
}
```
**getCurrentPriorityLevel：Scheduler调度优先级转换为react-dom中Scheduler调度优先级**
```js
// react-dom.development.js
function getCurrentPriorityLevel() {
  switch (Scheduler_getCurrentPriorityLevel()) {
    case Scheduler_ImmediatePriority:
      return ImmediatePriority$1;

    case Scheduler_UserBlockingPriority:
      return UserBlockingPriority$2;

    case Scheduler_NormalPriority:
      return NormalPriority$1;

    case Scheduler_LowPriority:
      return LowPriority$1;

    case Scheduler_IdlePriority:
      return IdlePriority$1;

    default:
      // 报错
  }
}
// scheduler.development.js
// 初始，currentPriorityLevel = NormalPriority = 3，普通
function unstable_getCurrentPriorityLevel() {
  return currentPriorityLevel;
}
```
**schedulerPriorityToLanePriority：调度优先级转换为更新任务优先级**
```js
function schedulerPriorityToLanePriority(schedulerPriorityLevel) {
  switch (schedulerPriorityLevel) {
    case ImmediatePriority:
      return SyncLanePriority;
    case UserBlockingPriority:
      return InputContinuousLanePriority;
    case NormalPriority:
    case LowPriority:
      return DefaultLanePriority; // 普通优先级和低优先级具有相同的更新任务优先级
    case IdlePriority:
      return IdleLanePriority;
    default:
      return NoLanePriority;
  }
}
```
3）创建更新
```js
// 创建更新
function createUpdate(eventTime, lane) {
  var update = {
    eventTime: eventTime, // 更新的开始时间
    lane: lane, // 更新优先级
    /*var UpdateState = 0;
    var ReplaceState = 1;
    var ForceUpdate = 2;
    var CaptureUpdate = 3;*/
    tag: UpdateState, // 更新的类型，0：更新；1: 替换；2：强制更新；3：捕获性的更新
    payload: null, // 更新的内容，比如setState接收的第一个参数
    callback: null, // 对应更新后的回调，比如setState({}, callback )
    next: null // 指向下一个更新
  };
  return update;
}
```
4）将更新插入到更新链表
```js
function enqueueUpdate(fiber, update) {
  /*fiber.updateQueue的初始值为：
  {
      baseState: fiber.memoizedState,
      baseQueue: null,
      shared: {
        pending: null
      },
      effects: null
  }*/
  var updateQueue = fiber.updateQueue;
  // 若updateQueue为null，说明fiber已被卸载
  if (updateQueue === null) {
    // Only occurs if the fiber has been unmounted.
    return;
  }

  var sharedQueue = updateQueue.shared;
  var pending = sharedQueue.pending;
  // update1指向updtae2，update2执行update3...update3指向update1
  if (pending === null) { // 第一次更新，创建循环单链表
    // This is the first update. Create a circular list.
    /*
      update1.next = update1
      sharedQueue.pending = update1
    */
    update.next = update;
  } else { // 将新的update插入到pending之后
    // update1.next=update2,update2.next=update3,update3.next=update1
    // sharedQueue.pending = update3
    update.next = pending.next;
    pending.next = update;
  }
  // 等待更新的update
  sharedQueue.pending = update;
}
```
