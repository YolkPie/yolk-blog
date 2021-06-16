---
title: react源码之调度开始
date: 2021-06-16 16:12:15
tags:
- react
categories: react
author: 马金坤
keywords: react, 源码
description: 在react-native项目中实现沉浸式标题栏
cover: https://img12.360buyimg.com/imagetools/jfs/t1/192410/37/8360/23491/60c9b074Ea765d291/3e52890e0c12a36e.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/171645/8/14914/73312/60c9b074Efb29a81e/efd6e5e2b4a4eb99.jpg
---
## 3.调度开始
上一节讲到，react根据更新任务的优先级lanePriority来执行不同的调度任务.
* lanePriority=SyncLanePriority，执行同步调度，scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root))
* lanePriority!=SyncLanePriority&&!=SyncBatchedLanePriority，执行异步调度，scheduleCallback(schedulerPriorityLevel, performConcurrentWorkOnRoot.bind(null, root))

同步调度：
```js
function scheduleSyncCallback(callback) {
  // Push this callback into an internal queue. We'll flush these either in
  // the next tick, or earlier if something calls `flushSyncCallbackQueue`.
  // 将同步更新任务，推入同步队列中
  if (syncQueue === null) { // 当前同步任务队列为空，可以立即执行当前更新任务
    syncQueue = [callback]; // Flush the queue in the next tick, at the earliest.

    immediateQueueCallbackNode = Scheduler_scheduleCallback(Scheduler_ImmediatePriority, flushSyncCallbackQueueImpl);
  } else { // 当前同步任务队列不为空，说明当前已经有调度任务在执行
    // Push onto existing queue. Don't need to schedule a callback because
    // we already scheduled one when we created the queue.
    syncQueue.push(callback);
  }
  return fakeCallbackNode;
}
```
异步调度：
```js
function scheduleCallback(reactPriorityLevel, callback, options) {
  // 将react优先级转为调度优先级
  var priorityLevel = reactPriorityToSchedulerPriority(reactPriorityLevel);
  return Scheduler_scheduleCallback(priorityLevel, callback, options);
}
function reactPriorityToSchedulerPriority(reactPriorityLevel) {
  switch (reactPriorityLevel) {
    case ImmediatePriority$1:
      return Scheduler_ImmediatePriority;
    case UserBlockingPriority$2:
      return Scheduler_UserBlockingPriority;
    case NormalPriority$1:
      return Scheduler_NormalPriority;
    case LowPriority$1:
      return Scheduler_LowPriority;
    case IdlePriority$1:
      return Scheduler_IdlePriority;
    default:
  }
}
```
由上可知，无论是同步任务调度还是异步任务调度，最终都会执行Scheduler_scheduleCallback。
### 3.1 前置说明
#### 存和取任务
* 更新任务进入调度程序时，会根据任务是否是延时任务，存入taskQueue或timerQueue任务队列中。其中，taskQueue存放需要立即执行的任务；timerQueue存放延时执行的任务。
* 任务插入(push)任务队列时，会根据sortIndex、id 属性进行优先级排序(siftUp)，优先级最高的任务排在队列首位。
Scheduler会优先取出(peek)taskQueue中的任务去执行，任务执行完成之后，从taskQueue移除(pop)，移除之后对任务队列重新排序(siftDown)。
* 若taskQueue为空，timerQueue中的任务会开始定时器任务，到达任务开始执行时间后，从timerQueue中取出(peek、pop)首个任务，存入(push)taskQueue中.

任务队列采用的是二叉堆(最小堆)，即父节点的键值总是小于或等于任何一个子节点的键值。添加元素时，在数组的最末尾插入新节点，然后自下而上调整子节点和父节点的位置(siftUp)；删除元素时，将数组的末尾元素放入到根节点，然后自上而下调整子节点和父节点的位置(siftDown).
 
二叉堆添加和删除元素代码：
```js
// 将node插入到heap中，并根据sortIndex、id 属性进行优先级排序，值越小优先级越高
function push(heap, node) {
  var index = heap.length;
  heap.push(node);
  siftUp(heap, node, index);
}
// 二叉堆(最小堆)-父结点的键值总是小于或等于任何一个子节点的键值
function siftUp(heap, node, i) {
  var index = i;
  while (true) {
    var parentIndex = index - 1 >>> 1;
    var parent = heap[parentIndex];
    // 当前node的优先级更高
    if (parent !== undefined && compare(parent, node) > 0) {
      heap[parentIndex] = node;
      heap[index] = parent;
      index = parentIndex;
    } else {
      return;
    }
  }
}
// 先比较sortIndex，再比较index，值越小，优先级越高
function compare(a, b) {
  // Compare sort index first, then task id.
  var diff = a.sortIndex - b.sortIndex;
  return diff !== 0 ? diff : a.id - b.id;
}
// 从heap堆里取出第一个元素
function peek(heap) {
  var first = heap[0];
  return first === undefined ? null : first;
}
// 从heap堆里取出第一个元素，并删除该元素
function pop(heap) {
  var first = heap[0];

  if (first !== undefined) {
    var last = heap.pop();

    if (last !== first) {
      heap[0] = last;
      siftDown(heap, last, 0);
    }

    return first;
  } else {
    return null;
  }
}
// 从二叉堆中取出优先级最高的元素，即根节点
function siftDown(heap, node, i) {
  var index = i;
  var length = heap.length;

  while (index < length) {
    var leftIndex = (index + 1) * 2 - 1;
    var left = heap[leftIndex];
    var rightIndex = leftIndex + 1; // index+1 <<< 1
    var right = heap[rightIndex]; // If the left or right node is smaller, swap with the smaller of those.
    if (left !== undefined && compare(left, node) < 0) {
      if (right !== undefined && compare(right, left) < 0) {
        heap[index] = right;
        heap[rightIndex] = node;
        index = rightIndex; 
      } else {
        heap[index] = left;
        heap[leftIndex] = node;
        index = leftIndex;
      }
    } else if (right !== undefined && compare(right, node) < 0) {
      heap[index] = right;
      heap[rightIndex] = node;
      index = rightIndex;
    } else {
      // Neither child is smaller. Exit.
      return;
    }
  }
}
```
peek、pop都是用于从二叉堆任务队列里取出首个任务，pop会从二叉堆中移除首个任务，然后进行堆排序；peek只是单纯的取出二叉堆的首个任务，不会移除元素。
#### 调度过程
**1）调度分为两个阶段**：
* 第一阶段：新任务进来后，根据任务过期时间和插入顺序，对任务进行二叉堆的堆排序，优先级最高的任务放在队列的首位。此时并不会立即执行任务队列里的任务，而是port.postMessage(null)，等到下一个宏任务再执行；此时设置isHostCallbackScheduled=true，标志着回调正在进行，任务调度中；
* 第二阶段：浏览器渲染完成，下一轮宏任务开始执行(通过channel.port1.onmessage回调)。执行回调函数flushWork，在时间片的范围内循环执行taskQueue中的任务，此时设置isHostCallbackScheduled=false，标志着第一阶段的任务调度结束，开始执行调度的更新任务了

**2）宏任务触发机制**

浏览器一帧的执行顺序：  
一个宏任务 => 队列中全部微任务 => requestAnimationFrame => 重排/重绘 => requestIdleCallback  
其中，requestIdleCallback并不是每次都执行，而是在重绘重排之后，如果还有空闲时间，才会执行requestIdleCallback。

若js执行时间过长，会导致浏览器没时间绘制dom，造成丢帧和卡顿的现象。不影响浏览器重排/重绘最好的方式，是等浏览器渲染完之后，再执行js，即在requestIdleCallback中执行。由于requestIdleCallback存在兼容性和触发时机的问题，react并未采用requestIdleCallback，而是在每一帧分配一个时间片(5ms)给js执行，在这个时间片内，若是还没执行完，那就暂停js，把主线程交个浏览器去绘制，等下一帧继续执行js.

Scheduler的时间切片功能是通过task（宏任务）实现的，浏览器渲染完成之后，才会执行下一轮的宏任务。Scheduler会在宏任务开始时，执行react的更新任务。

浏览器中宏任务优先级排序：主代码块 > setImmediate（node） > MessageChannel > setTimeout / setInterval。可以看出，宏任务中的MessageChannel优先级高于setTimeout / setInterval。

若当前宿主环境支持MessageChannel(浏览器环境)，则采用MessageChannel；不支持MessageChannel(非浏览器环境)，则采用setTimeout。

* 支持MessageChannel

```js
var channel = new MessageChannel();
var port = channel.port2;
// 会在下一轮宏任务中执行（浏览器未渲染完成之前，不会执行下一轮的宏任务）
channel.port1.onmessage = performWorkUntilDeadline;
// callback为flushWork
requestHostCallback = function (callback) {
    scheduledHostCallback = callback;
    port.postMessage(null);
};
```
* 不支持MessageChannel

```js
requestHostCallback = function (cb) {
    if (_callback !== null) {
      // Protect against re-entrancy.
      setTimeout(requestHostCallback, 0, cb);
    } else {
      _callback = cb;
      setTimeout(_flushCallback, 0);
    }
}
```
#### Sheduler调度优先级
* Immediate 立即执行优先级，需要同步执行的任务
* UserBlocking 用户阻塞型优先级（250 ms 后过期），需要作为用户交互结果运行的任务（例如，按钮点击）
* Normal 普通优先级（5 s 后过期），不必让用户立即感受到的更新
* Low 低优先级（10 s 后过期），可以推迟但最终仍然需要完成的任务（例如，分析通知）
* Idle 空闲优先级（永不过期），不必运行的任务（例如，隐藏界面以外的内容）

**sheduler中的调度优先级**
```js
var NoPriority = 0;
var ImmediatePriority = 1;
var UserBlockingPriority = 2;
var NormalPriority = 3;
var LowPriority = 4;
var IdlePriority = 5;
```
**sheduler调度优先级对应的过期时间**
```js
var IMMEDIATE_PRIORITY_TIMEOUT = -1; // Eventually times out
var USER_BLOCKING_PRIORITY_TIMEOUT = 250;
var NORMAL_PRIORITY_TIMEOUT = 5000;
var LOW_PRIORITY_TIMEOUT = 10000; // Never times out
var IDLE_PRIORITY_TIMEOUT = maxSigned31BitInt;
```
### 3.2 代码
* isHostCallbackScheduled：表示当前是否有调度任务(第二阶段中宏任务里要执行的回调函数是否被执行了)。若是当前有下一轮宏任务需要执行的回调函数(flushWork)，设置isHostCallbackScheduled=true；若回调函数任务开始执行了，设置为isHostCallbackScheduled=false;（针对taskQueue任务的第一阶段）
* isHostTimeoutScheduled: 表示当前是否有定时器任务。timerQueue中的任务设置定时器延时执行，此时isHostTimeoutScheduled=true；若是延时任务到了开始执行的时间，设置isHostTimeoutScheduled=false（针对timerQueue中的任务）
* isPerformingWork：当前是否有正在执行的更新任务（针对taskQueue任务的第二阶段）

#### 第一阶段
* 计算任务的开始时间
* 根据任务优先级priorityLevel计算任务的过期时间
* 创建调度任务
* 任务还没到开始执行时间，存入timerQueue；否则，存入taskQueue
* taskQueue不为空，调用requestHostCallback，port.postMessage(null)，等待下一个宏任务
* taskQueue为空，timerQueue中首个任务开始定时器任务，直到任务开始时间，存入taskQueue中，调用requestHostCallback

```js
function unstable_scheduleCallback(priorityLevel, callback, options) {
  // localPerformance.now(),获取当前时间
  var currentTime = exports.unstable_now();
  var startTime;
	// 第一次更新，options为null（好像代码里没有options有值的情况）
	// 获取任务预期执行时间startTime，若是有延迟时间，当前时间需要加上延迟时间
  if (typeof options === 'object' && options !== null) {
    var delay = options.delay;

    if (typeof delay === 'number' && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }
  // 根据调度优先级，获取任务多长时间过期(timeout)
  var timeout;
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }
  // 获取当前任务的过期时间
  var expirationTime = startTime + timeout;
  // 创建调度任务
  var newTask = {
    id: taskIdCounter++, // 任务节点的序号，创建任务时自增 1
    callback: callback, // 任务本体，即需要执行的更新任务
    priorityLevel: priorityLevel, // 任务的优先级。优先级按 ImmediatePriority、UserBlockingPriority、NormalPriority、LowPriority、IdlePriority 顺序依次越低
    startTime: startTime, // 任务预期执行时间，默认为当前时间，即同步任务。可通过 options.delay 设为异步延时任务
    expirationTime: expirationTime, // 过期时间
    sortIndex: -1 // 默认值为 -1。对于异步延时任务，该值将赋为startTime，否则为expirationTime
  };
  // taskQueue 存放需要立即执行的任务；timerQueue 存放延时执行的任务
  // 说明当前任务是异步延时任务，将其放入timerQueue
  if (startTime > currentTime) {
    // This is a delayed task.
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    // taskQueue中没有可执行的任务，newTask在timerQueue中优先级最高
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // 说明之前有等待从timerQueue取出任务的定时器，需要取消先前的调度任务的定时器
      if (isHostTimeoutScheduled) {
        // 清除定时器，clearTimeout
        cancelHostTimeout();
      } else { // 说明调度任务刚开始，设置为true
        isHostTimeoutScheduled = true;
      }
      // 延时执行timerQueue中的任务
	  // 通过setTimeout(fn，startTime - currentTime)延时执行
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else { // 准备就绪的任务，存放至taskQueue
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);
    // isHostCallbackScheduled表示是否有任务被安排进去了
    // isPerformingWork有任务安排了，并且在持续循环安排中
    // 前者负责一开始的调度，后者负责调度开始了，在允许的时间范围内是否持续调度
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      // flushWork，真正执行任务的地方
      requestHostCallback(flushWork);
    }
  }
  return newTask; // root.callbackNode=newTask
}
```

**taskQueue中的任务操作：**
```js
// callback为flushWork
requestHostCallback = function (callback) {
    scheduledHostCallback = callback;
    // isMessageLoopRunning=false，说明没有在循环处理的任务
	// 通过MessageChannel的port消息通知，进入到下一个宏任务中处理flushWork
    if (!isMessageLoopRunning) {
      isMessageLoopRunning = true; // 标志当前有正在循环处理的任务
      port.postMessage(null);
    }
}
```
**timerQueue中的任务操作：**
```js
function handleTimeout(currentTime) {
  isHostTimeoutScheduled = false;
  // 将timerQueue中已经到开始执行时间的任务放入taskQueue
  advanceTimers(currentTime);
  if (!isHostCallbackScheduled) {
    // taskQueue有任务，执行任务
    if (peek(taskQueue) !== null) {
      // 在调用requestHostCallback的回调函数flushWork之前，设置isHostCallbackScheduled=true
      // 标志有任务被调度了
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    } else { // taskQueue为null，即没有待执行的任务，那就等待timerQueue中的任务
      var firstTimer = peek(timerQueue);

      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}
// 从timerQueue中取出已经到开始执行时间的任务，取出来放入taskQueue，并设置sortedIndex为当前任务的expirationTime
function advanceTimers(currentTime) {
  // Check for tasks that are no longer delayed and add them to the queue.
  var timer = peek(timerQueue);
  // 循环遍历timerQueue，找出已经到开始执行时间的任务，存入taskQueue
  while (timer !== null) {
    if (timer.callback === null) {
      // Timer was cancelled.
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {
      // Timer fired. Transfer to the task queue.
      pop(timerQueue);
      timer.sortIndex = timer.expirationTime;
      push(taskQueue, timer);
    } else {
      // Remaining timers are pending.
      return;
    }

    timer = peek(timerQueue);
  }
}
requestHostTimeout = function (callback, ms) {
    taskTimeoutID = _setTimeout(function () {
      callback(exports.unstable_now());
    }, ms);
};
cancelHostTimeout = function () {
    _clearTimeout(taskTimeoutID);
    taskTimeoutID = -1;
};
```

#### 第二阶段
在下一轮宏任务中执行更新任务。真正执行更新任务的入口：**performWorkUntilDeadline**
* 计算任务的暂停时间，当前时间加上时间片(5ms);
* 在时间片的范围内，循环执行taskQueue中的任务，并发模式是performConcurrentWorkOnRoot；block模式是performSyncWorkOnRoot；
* 时间片内，调度的任务taskQueue执行完毕，重置isMessageLoopRunning以及scheduledHostCallback，停止调度；从timerQueue找到优先级最高的任务，开启定时器任务，直到任务的开始时间
* 时间片结束，还有任务没有执行完成，继续port.postMessage(null)，通过宏任务进入下一轮performWorkUntilDeadlined，继续执行workLoop

```js
var channel = new MessageChannel();
var port = channel.port2;
// 会在下一轮宏任务中执行（浏览器未渲染完成之前，不会执行下一轮的宏任务）
channel.port1.onmessage = performWorkUntilDeadline;
```
```js
/* 任务的最大工作时长
   react此版并未精确的计算一个帧的剩余时间
   若是需要一个精确的帧剩余时间，需要通过requestAnimationFrame
*/
var yieldInterval = 5; 
var performWorkUntilDeadline = function () {
    if (scheduledHostCallback !== null) { // scheduledHostCallback，当前任务的回调函数
      var currentTime = exports.unstable_now(); 
      // 计算任务的暂停时间，一旦 workLoop 执行到 deadline 时间后，scheduler 会让出主线程以执行其他任务
      deadline = currentTime + yieldInterval;
      var hasTimeRemaining = true;
      try {
        // scheduledHostCallback=flushWork
        var hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
        if (!hasMoreWork) { // 没有其他任务了，停止调度
          isMessageLoopRunning = false;
          scheduledHostCallback = null;
        } else { // 说明时间片用完，任务被打断，还有任务未执行完成，继续port.postMessage(null)通过宏任务进入下一轮performWorkUntilDeadlined
          port.postMessage(null);
        }
      } catch (error) {
        port.postMessage(null);
        throw error;
      }
    } else {
      isMessageLoopRunning = false;
    }
  };
// 设置 isHostCallbackScheduled=false，代表当前调度的回调函数任务开始执行了
// 保存当前任务优先级currentPriorityLevel
// 批量循环执行taskQueue中的任务 ，函数为workLoop
function flushWork(hasTimeRemaining, initialTime) {
  isHostCallbackScheduled = false; // 代表当前调度的回调函数任务开始执行了
  if (isHostTimeoutScheduled) { // 当前有定时器任务，取消它
    isHostTimeoutScheduled = false;
    cancelHostTimeout();
  }
  isPerformingWork = true; // 表示调度的任务正在执行
  var previousPriorityLevel = currentPriorityLevel; // 保存当前任务优先级currentPriorityLevel
  try {
    return workLoop(hasTimeRemaining, initialTime)
  } finally {
    currentTask = null;
    currentPriorityLevel = previousPriorityLevel;
    isPerformingWork = false;
  }
}
/*
  循环执行taskQueue中的更新任务
  hasTimeRemaining：每次执行workLoop都为true；initialTime：执行任务的开始时间
*/
function workLoop(hasTimeRemaining, initialTime) {
  var currentTime = initialTime;
  advanceTimers(currentTime); // 从timerQueue中取出已经到开始执行时间的任务，取出来放入taskQueue
  currentTask = peek(taskQueue); // 先取出优先级最高的任务
  // 循环执行taskQueue中的任务
  while (currentTask !== null && !(enableSchedulerDebugging )) {
    // 当前任务未过期，但已经到了时间片的末尾，需要中断循环
    if (currentTask.expirationTime > currentTime && (!hasTimeRemaining || shouldYieldToHost())) { // shouldYieldToHost => return getCurrentTime() >= deadline
      // This currentTask hasn't expired, and we've reached the deadline.
      break;
    }
    // 取出当前任务的回调函数，并发模式是performConcurrentWorkOnRoot；block模式是performSyncWorkOnRoot
    var callback = currentTask.callback;
    if (typeof callback === 'function') {
      currentTask.callback = null; // 置空当前任务的回调
      currentPriorityLevel = currentTask.priorityLevel;
      var didUserCallbackTimeout = currentTask.expirationTime <= currentTime; // 当前任务是否已过期
      var continuationCallback = callback(didUserCallbackTimeout);
      currentTime = getCurrentTime();
      if (typeof continuationCallback === 'function') { // 若返回函数类型，说明任务尚未执行完成；作为新的任务回调函数放入当前的task上
        currentTask.callback = continuationCallback;
      } else {
        if (currentTask === peek(taskQueue)) { // 当前任务回调函数执行完成，从taskQueue出堆
          pop(taskQueue);
        }
      }
      advanceTimers(currentTime);
    } else {
      pop(taskQueue); // 当前任务从taskQueue出栈
    }
    currentTask = peek(taskQueue);
  } // Return whether there's additional work
  if (currentTask !== null) { // 时间片结束，还有任务没有执行完成，继续port.postMessage(null)，通过宏任务进入下一轮performWorkUntilDeadlined，继续执行workLoop
    return true;
  } else { // taskQueue中的任务已执行完成
    // 从timerQueue找到优先级最高的任务，开启定时器任务，直到任务的开始时间
    var firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    return false;
  }
}
```
