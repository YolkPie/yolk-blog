---
title: react源码之调度入口
date: 2021-06-16 16:07:54
tags:
- react
categories: react
author: 马金坤
keywords: react, 源码
description: 在react-native项目中实现沉浸式标题栏
cover: https://img12.360buyimg.com/imagetools/jfs/t1/192410/37/8360/23491/60c9b074Ea765d291/3e52890e0c12a36e.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/171645/8/14914/73312/60c9b074Efb29a81e/efd6e5e2b4a4eb99.jpg
---
## 2. 调度入口
* 获取rootFiber；更新fiber的lanes；从当前fiber往上遍历至fiberRoot，更新父fiber的childLanes
* 更新rootFiber的pendingLanes和eventTimes
* 若是同步非批量更新，调用performSyncWorkOnRoot(root)；同步非批量更新和异步更新，调用ensureRootIsScheduled(root, eventTime)

```js
function scheduleUpdateOnFiber(fiber, lane, eventTime) {
  // 得到fiberRoot节点
  // 更新fiber的lanes；从当前fiber往上遍历至fiberRoot，更新父fiber的childLanes
  var root = markUpdateLaneFromFiberToRoot(fiber, lane);
  // 更新rootFiber的pendingLanes和eventTimes
  markRootUpdated(root, lane, eventTime);
  if (root === workInProgressRoot) {
    {
      workInProgressRootUpdatedLanes = mergeLanes(workInProgressRootUpdatedLanes, lane);
    }
    // 当前任务被暂停，即workInProgressRootRenderLanes都是被暂停的lanes
	// 从suspendedLanes中移除已经恢复任务的lanes和已经更新完成任务的lanes
	// 更新expirationTimes数组，对suspendedLanes对应index的过期时间设置为-1
    if (workInProgressRootExitStatus === RootSuspendedWithDelay) {
      markRootSuspended$1(root, workInProgressRootRenderLanes);
    }
  }
  if (lane === SyncLane) { // 同步更新
    if ( // 当前是同步非批量更新，且非render或非commit阶段
    (executionContext & LegacyUnbatchedContext) !== NoContext && // Check if we're not already rendering
    (executionContext & (RenderContext | CommitContext)) === NoContext) {
      // 执行同步更新
      performSyncWorkOnRoot(root);
    } else { // 批量更新
      ensureRootIsScheduled(root, eventTime);
      // ???
      if (executionContext === NoContext && (fiber.mode & ConcurrentMode) === NoMode) {
        resetRenderTimer();
        flushSyncCallbacksOnlyInLegacyMode();
      }
    }
  } else { // 异步更新
    // Schedule other updates after in case the callback is sync.
    ensureRootIsScheduled(root, eventTime);
  }

  return root;
}
```

### 2.1 markUpdateLaneFromFiberToRoot: 更新fiber的lanes,返回fiberRoot
从当前fiber开始往上到rootFiber更新所有fiber的childLanes
* 更新当前fiber的lanes和alternate的lanes，
* 从下往上遍历父fiber，更新父fiber的childLanes和父fiber.alternate的childLanes，直到rootFiber 
* 返回fiberRoot

```js  
function markUpdateLaneFromFiberToRoot(sourceFiber, lane) {
  // 更新当前fiber的lanes，将lane加入到sourceFiber.lanes中
  // mergeLanes(a, b) => a | b
  sourceFiber.lanes = mergeLanes(sourceFiber.lanes, lane);
  // 第一次更新，sourceFiber.alternate = null
  var alternate = sourceFiber.alternate;
  // alternate非空，也更新alternate的lanes
  if (alternate !== null) {
    alternate.lanes = mergeLanes(alternate.lanes, lane);
  }
  var node = sourceFiber;
  // 获取当前fiber的父fiber，若当前fiber为rootFiber，其父fiber为null
  var parent = sourceFiber.return;
  // 从当前fiber开始，从下往上，遍历父fiber，更新父fiber的childLanes和父fiber.alternate的childLanes，直到rootFiber
  while (parent !== null) {
    parent.childLanes = mergeLanes(parent.childLanes, lane);
    alternate = parent.alternate;
    if (alternate !== null) {
      alternate.childLanes = mergeLanes(alternate.childLanes, lane);
    }
    node = parent;
    parent = parent.return;
  }
  // 返回fiberRoot（rootFiber.tag=3=HostRoot）
  if (node.tag === HostRoot) {
    var root = node.stateNode;
    return root;
  } else {
    return null;
  }
} 
```

### 2.2 markRootUpdated：更新rootFiber的pendingLanes和eventTimes
* 将当前更新的lane和更新开始时间分别保存到rootFiber.pendingLanes和rootFiber.eventTimes

```js
// 更新rootFiber的pendingLanes和eventTimes（将当前更新的lane和更新开始时间分别保存到rootFiber.pendingLanes和rootFiber.eventTimes）
function markRootUpdated(root, updateLane, eventTime) {
  // 将当前更新的lane(updateLane)放入到rootFiber的pendingLanes中
  root.pendingLanes |= updateLane;
  // 获取比当前updateLane优先级更高的lanes
  var higherPriorityLanes = updateLane - 1; // Turns 0b1000 into 0b0111
 // ??? 目的是剔除掉suspendedLanes 和 pingedLanes中优先级低于本次更新优先级（updateLane）的lane
  // 实现上方注释中的 “取消同等或较低优先级的更新。”
  root.suspendedLanes &= higherPriorityLanes;
  root.pingedLanes &= higherPriorityLanes;
  // 获取所有的更新开始时间数组eventTimes，31位
  var eventTimes = root.eventTimes;
  // 找到lanes中最靠左的那个1在lanes中的index
  var index = laneToIndex(updateLane); 
  // 将当前更新的开始时间保存到root的更新时间数组中
  eventTimes[index] = eventTime;
}
```

### 2.3 ensureRootIsScheduled: 定义了高优先级任务插队逻辑（赛车抢赛道）
获取优先级最高的lane，高优先级任务打断低优先级任务
* 1）循环遍历root.pendingLanes，没有过期时间，计算过期时间；有过期时间，判断是否过期，若过期，加入到root.expiredLanes中
* 2）从root.expiredLanes和root.pendingLanes未执行任务中找出优先级最高的lane和对应的优先级，获取lanes集合的优先级顺序是，过期任务的expiredLanes>非空闲且非挂起任务的lanes > 非空闲且挂起又恢复任务的lanes > 空闲且非挂起任务的lanes > 空闲且挂起又恢复任务的lanes
* 3）比较新旧任务的优先级，两者相等，无需调度；前者大于后者，取消旧任务
* 4）新任务的优先级高于旧任务，根据新任务的优先级来进行不同的调度任务：
> 同步优先级：调用scheduleSyncCallback去同步执行任务。
> 同步批量执行：调用scheduleCallback，将任务以立即执行的优先级去加入调度。
> 属于concurrent模式的优先级：调用scheduleCallback，将任务通过lanePriorityToSchedulerPriority获取到的新任务优先级去加入调度。

```js
function ensureRootIsScheduled(root, currentTime) {
	// 获取旧任务
  var existingCallbackNode = root.callbackNode; 
	// 从root.pendingLanes最左边的第一个1开始，循环遍历，没有过期时间，计算过期时间；有过期时间，判断是否过期，若过期，加入到root.expiredLanes中
  markStarvedLanesAsExpired(root, currentTime); // Determine the next lanes to work on, and their priority.
	// 从root.pendingLanes未执行任务中和root.expiredLanes中找出优先级最高的lane（过期任务的优先级最高）
  var nextLanes = getNextLanes(root, root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes); 
	// 获取新任务的优先级，即nextLanes对应的优先级，return_highestLanePriority
  var newCallbackPriority = returnNextLanesPriority();
	// nextLanes为未执行任务中优先级最高的lanes，若它为空，则说明不需要调度，取消回调，return
  if (nextLanes === NoLanes) {
    if (existingCallbackNode !== null) {
      cancelCallback(existingCallbackNode);
      root.callbackNode = null;
      root.callbackPriority = NoLanePriority;
    }
    return;
  }
  // 若存在旧任务
  // 新任务的优先级高于或等于旧任务优先级
  if (existingCallbackNode !== null) {
    // 获取旧任务的优先级
    var existingCallbackPriority = root.callbackPriority;
    // 若新旧任务的优先级一样，进入batchedUpdate的逻辑
    // 无需再次发起一次调度，直接复用旧任务即可，让旧任务在处理更新的时候顺便把新任务给做了
    if (existingCallbackPriority === newCallbackPriority) {
      return;
    }
    // 新旧任务的优先级不相等，则说明新任务的优先级一定高于旧任务，这种情况就是高优先级任务插队，需要把旧任务取消掉
    // 每次获取nextLanes时，都是从root.pendingLanes中获取优先级最高的lanes，nextLanes对应的优先级newCallbackPriority也是最高的
    cancelCallback(existingCallbackNode); // existingCallbackNode.callback = null
  }
  var newCallbackNode;
  // 若新任务的优先级为同步优先级，则同步调度，传统的同步渲染和过期任务会走这里
  if (newCallbackPriority === SyncLanePriority) {
    newCallbackNode = scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
  } else if (newCallbackPriority === SyncBatchedLanePriority) { // 同步模式到concurrent模式的过渡模式：blocking模式会走这里
    newCallbackNode = scheduleCallback(ImmediatePriority$1, performSyncWorkOnRoot.bind(null, root));
  } else { // concurrent模式的渲染会走这里
	// 根据任务优先级获取Scheduler的调度优先级
    var schedulerPriorityLevel = lanePriorityToSchedulerPriority(newCallbackPriority);
    newCallbackNode = scheduleCallback(schedulerPriorityLevel, performConcurrentWorkOnRoot.bind(null, root));
  }
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}
```

#### 1）markStarvedLanesAsExpired
* 循环遍历pendingLanes（未执行的任务包含的lane），如果没过期时间就计算一个过期时间，如果过期了就加入root.expiredLanes中
* 在下次调用getNextLane函数的时候会优先返回expiredLanes

```js
function markStarvedLanesAsExpired(root, currentTime) {
  var pendingLanes = root.pendingLanes;
  var suspendedLanes = root.suspendedLanes;
  var pingedLanes = root.pingedLanes;
  var expirationTimes = root.expirationTimes; 
  var lanes = pendingLanes;
  while (lanes > 0) {
  	// 以8位二进制0001100为例
	// clz32(lanes) = 3，7 - 3 = 4，index = 4
	// lane = 1 << 4 = 00010000
    var index = pickArbitraryLaneIndex(lanes); // 31 - clz32(lanes)
    var lane = 1 << index;
    var expirationTime = expirationTimes[index];
		// 当前没有过期时间就计算一个过期时间
    if (expirationTime === NoTimestamp) {
    	// 当前lane不是挂起任务或当前lane是挂起任务被恢复的lanes，计算当前任务的过期时间
      if ((lane & suspendedLanes) === NoLanes || (lane & pingedLanes) !== NoLanes) {
        expirationTimes[index] = computeExpirationTime(lane, currentTime);
      }
    } else if (expirationTime <= currentTime) { // 当前任务过期了
      root.expiredLanes |= lane; // 在expiredLanes加入当前遍历到的lane
    }
    lanes &= ~lane;
  }
}

// 根据优先级，计算lane的过期时间，过期时间分为3种情况：
// 1）用户阻塞型优先级：currentTime + 250（250 ms 后过期），需要用户交互的任务，如按钮点击；
// 2）普通优先级：currentTime + 5000（5 后过期），不必让用户立即感受到的更xin
// 3) 更低优先级或空闲优先级（永不过期）：-1，不必运行的任务（例如，隐藏界面以外的内容）
function computeExpirationTime(lane, currentTime) {
  // 获取lane对应的更新任务优先级
  getHighestPriorityLanes(lane); // lane=512，对应的return_highestLanePriority=DefaultLanePriority=8
  var priority = return_highestLanePriority;
  // InputContinuousLanePriority = 10
  if (priority >= InputContinuousLanePriority) {
    return currentTime + 250;
  } else if (priority >= TransitionPriority) { // TransitionPriority = 6
    return currentTime + 5000;
  } else {
    // 任何空闲或更低的优先级都不应该过期
    return NoTimestamp;
  }
}
```

#### 2）getNextLanes
获取下一个执行任务的lanes，即从root.pendingLanes未执行任务和root.expiredLanes中找出优先级最高的lane
```js
function getNextLanes(root, wipLanes) {
  var pendingLanes = root.pendingLanes;
	// 没有未执行任务的时，return
  if (pendingLanes === NoLanes) {
    return_highestLanePriority = NoLanePriority;
    return NoLanes;
  }
  var nextLanes = NoLanes;
  var nextLanePriority = NoLanePriority;
  var expiredLanes = root.expiredLanes;
  var suspendedLanes = root.suspendedLanes;
  var pingedLanes = root.pingedLanes; 
  /*检查是有任务过期
    1）任务已过期，设置渲染任务为同步优先级，nextLanes=expiredLanes,nextLanePriority=SyncLanePriority，过期任务优先级最高
    2）任务未过期，从未执行任务的lanes(pendingLanes)获取lanes集合的优先级顺序是，非空闲且非挂起任务的lanes > 非空闲且挂起又恢复任务的lanes > 空闲且非挂起任务的lanes > 空闲且挂起任务的lanes

     3）通过getHighestPriorityLanes，从获取到优先级lanes集合中拿到优先级最高lanes
   拿到nextLanes后，通过getHighestPriorityLanes(lanes)，获取lanes对应的优先级nextLanePriority */
  if (expiredLanes !== NoLanes) { // 有任务过期
    nextLanes = expiredLanes;
    nextLanePriority = return_highestLanePriority = SyncLanePriority; // 15，过期任务优先级最高，需要立即执行
  } else { // 无任务过期
	// 获取未执行任务lans中非空闲优先级的任务lans
    var nonIdlePendingLanes = pendingLanes & NonIdleLanes;
	// 从未执行任务lans中，先取非挂起任务的lans，从非挂起任务的lans中获取最高的优先级的lane和lane对应的优先级
	// 若全是非挂起任务的lans，则从非挂起任务的lans中获取最高的优先级的lanes和lanes对应的优先级
    if (nonIdlePendingLanes !== NoLanes) { // 有非空闲任务的lans
      var nonIdleUnblockedLanes = nonIdlePendingLanes & ~suspendedLanes; // 去掉挂起任务的lans
	// nonIdleUnblockedLanes：nonIdlePendingLanes中去掉挂起任务的lans
      if (nonIdleUnblockedLanes !== NoLanes) { // 说明nonIdlePendingLanes中，除了有非挂起任务的lans，还有其他的lans
        nextLanes = getHighestPriorityLanes(nonIdleUnblockedLanes);
        nextLanePriority = return_highestLanePriority;
      } else { // nonIdleUnblockedLanes为null，说明nonIdlePendingLanes中只有挂起任务的lans
        // 从nonIdlePendingLanes中获取挂起任务lanes中被恢复的lanes
        var nonIdlePingedLanes = nonIdlePendingLanes & pingedLanes;

        if (nonIdlePingedLanes !== NoLanes) {
          nextLanes = getHighestPriorityLanes(nonIdlePingedLanes);
          nextLanePriority = return_highestLanePriority;
        }
      }
    } else { // 只有空闲任务的lans
	  // 从未执行任务的lans中去掉挂起任务的lans，即获取非阻塞任务lans
      var unblockedLanes = pendingLanes & ~suspendedLanes;
      // 也是优先取非挂起任务的lans，没有的话，再取挂起任务的lans
	  // 从lanes中，获取最高的优先级的lanes和lanes对应的优先级
      if (unblockedLanes !== NoLanes) {
        nextLanes = getHighestPriorityLanes(unblockedLanes);
        nextLanePriority = return_highestLanePriority;
      } else {
        if (pingedLanes !== NoLanes) {
          nextLanes = getHighestPriorityLanes(pingedLanes);
          nextLanePriority = return_highestLanePriority;
        }
      }
    }
  }
	// 如果nextLanes=0，则说明不需要调度任务，return
  if (nextLanes === NoLanes) {
    return NoLanes;
  }
    // 通过getEqualOrHigherPriorityLanes获取比nextLanes优先级更高的lanes
    // 如果未执行任务的pendingLanes中有比nextLanes优先级更高的lanes，也放入到nextLanes中
    // getEqualOrHigherPriorityLanes(0b001100) = 0b001111
  nextLanes = pendingLanes & getEqualOrHigherPriorityLanes(nextLanes);
  
  // wipLanes不为空，说明正在渲染，若渲染任务的lanes不等于新任务的lanes，且其不为挂起任务
  // 判断新旧任务的优先级，旧任务优先级高，忽略新任务，继续执行旧任务，往下渲染；新任务优先级高，那就打断旧任务，插队
  if (wipLanes !== NoLanes && wipLanes !== nextLanes && 
  (wipLanes & suspendedLanes) === NoLanes) {
    getHighestPriorityLanes(wipLanes);
    var wipLanePriority = return_highestLanePriority;

    if (nextLanePriority <= wipLanePriority) {
      return wipLanes;
    } else {
      return_highestLanePriority = nextLanePriority;
    }
  }
  return nextLanes;
}
```
**getHighestPriorityLanes：从优先级lanes集合中拿到优先级最高lanes和对应的更新任务优先级**
```js
// 根据lanes，获取最高的优先级的lanes和lanes对应的优先级
// 若所有的lanes都未匹配到，则返回lanes和DefaultLanePriority
function getHighestPriorityLanes(lanes) {
  if ((SyncLane & lanes) !== NoLanes) {
    return_highestLanePriority = SyncLanePriority;
    return SyncLane;
  }
  if ((SyncBatchedLane & lanes) !== NoLanes) {
    return_highestLanePriority = SyncBatchedLanePriority;
    return SyncBatchedLane;
  }
  if ((InputDiscreteHydrationLane & lanes) !== NoLanes) {
    return_highestLanePriority = InputDiscreteHydrationLanePriority;
    return InputDiscreteHydrationLane;
  }
  var inputDiscreteLanes = InputDiscreteLanes & lanes;
  if (inputDiscreteLanes !== NoLanes) {
    return_highestLanePriority = InputDiscreteLanePriority;
    return inputDiscreteLanes;
  }
  if ((lanes & InputContinuousHydrationLane) !== NoLanes) {
    return_highestLanePriority = InputContinuousHydrationLanePriority;
    return InputContinuousHydrationLane;
  }
  var inputContinuousLanes = InputContinuousLanes & lanes;
  if (inputContinuousLanes !== NoLanes) {
    return_highestLanePriority = InputContinuousLanePriority;
    return inputContinuousLanes;
  }
  if ((lanes & DefaultHydrationLane) !== NoLanes) {
    return_highestLanePriority = DefaultHydrationLanePriority;
    return DefaultHydrationLane;
  }
  var defaultLanes = DefaultLanes & lanes;
  if (defaultLanes !== NoLanes) {
    return_highestLanePriority = DefaultLanePriority;
    return defaultLanes;
  }
  if ((lanes & TransitionHydrationLane) !== NoLanes) {
    return_highestLanePriority = TransitionHydrationPriority;
    return TransitionHydrationLane;
  }
  var transitionLanes = TransitionLanes & lanes;
  if (transitionLanes !== NoLanes) {
    return_highestLanePriority = TransitionPriority;
    return transitionLanes;
  }
  var retryLanes = RetryLanes & lanes;
  if (retryLanes !== NoLanes) {
    return_highestLanePriority = RetryLanePriority;
    return retryLanes;
  }
  if (lanes & SelectiveHydrationLane) {
    return_highestLanePriority = SelectiveHydrationLanePriority;
    return SelectiveHydrationLane;
  }
  if ((lanes & IdleHydrationLane) !== NoLanes) {
    return_highestLanePriority = IdleHydrationLanePriority;
    return IdleHydrationLane;
  }
  var idleLanes = IdleLanes & lanes;
  if (idleLanes !== NoLanes) {
    return_highestLanePriority = IdleLanePriority;
    return idleLanes;
  }
  if ((OffscreenLane & lanes) !== NoLanes) {
    return_highestLanePriority = OffscreenLanePriority;
    return OffscreenLane;
  }

  return_highestLanePriority = DefaultLanePriority;
  return lanes;
}
```
**getEqualOrHigherPriorityLanes：获取比当前lanes优先级更高或相等的lanes集合**
```js
function getEqualOrHigherPriorityLanes(lanes) {
  // 假设lanes=0b001100,getLowestPriorityLane(0b001100) = 0b001000
  // 0b001000 << 1 = 0b010000
  // 0b010000 - 1  = 0b001111
  return (getLowestPriorityLane(lanes) << 1) - 1;
}

// 从lanes中获取最低优先级的lane，即找到lanes中最靠左的那个1所对应的lane
function getLowestPriorityLane(lanes) {
  // This finds the most significant non-zero bit.
  var index = 31 - clz32(lanes);
  return index < 0 ? NoLanes : 1 << index;
}
// 找到lanes中最靠左的那个1在lanes中的index 
function pickArbitraryLaneIndex(lanes) {
  return 31 - clz32(lanes);
}
function laneToIndex(lane) {
  return pickArbitraryLaneIndex(lane);
}
```
**lanePriorityToSchedulerPriority：更新任务优先级转为调度优先级**
```js
function lanePriorityToSchedulerPriority(lanePriority) {
  switch (lanePriority) {
    case SyncLanePriority:
    case SyncBatchedLanePriority:
      return ImmediatePriority;

    case InputDiscreteHydrationLanePriority:
    case InputDiscreteLanePriority:
    case InputContinuousHydrationLanePriority:
    case InputContinuousLanePriority:
      return UserBlockingPriority;

    case DefaultHydrationLanePriority:
    case DefaultLanePriority:
    case TransitionHydrationPriority:
    case TransitionPriority:
    case SelectiveHydrationLanePriority:
    case RetryLanePriority:
      return NormalPriority;

    case IdleHydrationLanePriority:
    case IdleLanePriority:
    case OffscreenLanePriority:
      return IdlePriority;

    case NoLanePriority:
      return NoPriority;

    default:
      
  }
}
```
