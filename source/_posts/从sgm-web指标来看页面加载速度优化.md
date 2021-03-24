---
title: 从sgm-web指标来看页面加载速度优化
date: 2021-03-23 10:20:00
tags:
- 异常
cover: https://img14.360buyimg.com/imagetools/jfs/t1/158535/22/7215/96620/602f2a91Eb6020785/997b23c6f02a6baa.jpg
top_img: https://img11.360buyimg.com/imagetools/jfs/t1/161861/15/7640/328340/6034dbb6Eb76a6896/512483da73d8865c.png
---

# 前言

页面加载速度优化是提升用户体验、提高用户访问率和留存率的关键，因此，加载速度优化是前端开发必须掌握的技能。通常，我们会使用监控平台提供的 js 进行各种数据的上报，并依靠这些监控平台提供的各项数据来评估我们的页面，但对于这些数据的上报节点以及哪些资源或者操作会影响这些数据，我们却常常是不求甚解的。所以经常会出现这种情况，随着需求的累积，页面加载越来越慢。  
这篇文章的目的很简单，就是弄清楚我们使用的监控平台的性能数据对应的时间节点是什么，在这些节点上，页面的加载状态是什么样子的。

# sgm-web 的统计指标

目前，我们的页面使用的是 sgm-web 平台来进行页面性能的监控，这个平台提供了白屏时间、页面加载完成时间和 HTML 加载完成时间这三个数据项作为衡量页面加载速度的依据。为了弄清楚这些数据项是按照什么方式统计的，我新建了一个 sgm 的应用，并且通过在 sgm.js 中打断点的方式，获取到了上报时的数据，如下：

![sgm.js断点](https://img11.360buyimg.com/imagetools/jfs/t1/165220/28/6892/627382/602f829cEcadc06b2/21d67416578abccf.jpg)

```
cct: 0
cot: 0
dnt: 0
dot: 492
fbe: 15
fpt: 16
ldt: 1522
rdc: 0
rdt: 0
ret: 1014
rte: 508
slt: 0
tbt: 7
tst: 1
tte: 508
typ: "reload"
```

此时，sgm-web 后台统计出来的数据如下：

![sgm-web统计结果](https://img14.360buyimg.com/imagetools/jfs/t1/158535/22/7215/96620/602f2a91Eb6020785/997b23c6f02a6baa.jpg)

因为只有一次上报，我们可以很清楚的找到统计口径和上报数据的对应关系，而且通过 sgm.js，我们也可以知道各个上报数据是怎么计算出来的：

```
白屏时间 -> fpt -> responseEnd - startTime
页面加载完成时间 -> ldt -> loadEventEnd - startTime
HTML加载完成时间 -> rte -> domContentLoadedEventEnd - startTime
```

对 Performance 稍微有了解的大概都会猜到，responseEnd/loadEventEnd/domContentLoadedEventEnd/startTime 等都是 Performance 中的属性（responseEnd/loadEventEnd/domContentLoadedEventEnd/navigationStart 为PerformanceTiming 中的属性， startTime 为 PerformanceNavigationTiming 中的属性， 这部分我们下节会讲）。下一步，我们要弄清楚这些属性的含义是什么。

# Performance API

注意： Performance API 这节主要为了说明 Performance 的使用及兼容情况，如果你觉得枯燥，可以跳过这一节。

还是先从 sgm.js 中的一段代码说起（如下），这部分代码用于获取和性能相关的数据：

``` js
    // b 代表 window， e 为最终获取的性能的数据
    var e, t = b.performance, r = t.timing, n = t.navigation, i = void 0 === n ? {} : n;
    if (b.PerformanceNavigationTiming) {
        var a = b.performance.getEntriesByType("navigation") || [];
        e = a[0] || {}
    } else {
        var o = i.type
            , s = i.redirectCount;
        e = r || {},
        e.type = Mo(o),
        e.redirectCount = s,
        e.startTime = r.navigationStart
    }
```

从上面的代码我可以看到，是先判断了 window 下是否有 PerformanceNavigationTiming 对象， 如果有的话使用 window.performance.getEntriesByType("navigation") 方法获取，没有的话就使用 window.performance.timing 对象，并且追加window.performance.timing.navigationStart 、window.performance.navigation.type 、window.performance.navigation.redirectCount 属性。凭借经验，我们知道这可能和Performance 下不同类型的对象的兼容性有关系，那我们就从Performance 开始展开吧。

## Performance

通过 window.performance 可以获取到 Performance 的对象。Performance 的定义如下：

``` java
interface Performance extends EventTarget {
    /** @deprecated */
    readonly navigation: PerformanceNavigation;
    onresourcetimingbufferfull: ((this: Performance, ev: Event) => any) | null;
    readonly timeOrigin: number;
    /** @deprecated */
    readonly timing: PerformanceTiming;
    clearMarks(markName?: string): void;
    clearMeasures(measureName?: string): void;
    clearResourceTimings(): void;
    getEntries(): PerformanceEntryList;
    getEntriesByName(name: string, type?: string): PerformanceEntryList;
    getEntriesByType(type: string): PerformanceEntryList;
    mark(markName: string): void;
    measure(measureName: string, startMark?: string, endMark?: string): void;
    now(): number;
    setResourceTimingBufferSize(maxSize: number): void;
    toJSON(): any;
    addEventListener<K extends keyof PerformanceEventMap>(type: K, listener: (this: Performance, ev: PerformanceEventMap[K]) => any, options?: boolean | AddEventListenerOptions): void;
    addEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | AddEventListenerOptions): void;
    removeEventListener<K extends keyof PerformanceEventMap>(type: K, listener: (this: Performance, ev: PerformanceEventMap[K]) => any, options?: boolean | EventListenerOptions): void;
    removeEventListener(type: string, listener: EventListenerOrEventListenerObject, options?: boolean | EventListenerOptions): void;
}
```

> Performance 接口可以获取到当前页面中与性能相关的信息。它是 High Resolution Time API 的一部分，同时也融合了 Performance Timeline API、Navigation Timing API、 User Timing API 和 Resource Timing API。

在这里，我们需要关注 Performance 中的 navigation 和 timing 属性，它们分别是 PerformanceNavigation 和 PerformanceTiming 的对象。
另外，在最新的 W3C 标准中（[点这里](https://www.w3.org/TR/navigation-timing-2/#the-performancetiming-interface)），PerformanceTiming 和 PerformanceNavigation 被弃用了，推荐使用 PerformanceNavigationTiming。我们可以通过调用 Performance 中的 getEntriesByType("navigation")[0] 获取 PerformanceNavigationTiming 实例（PerformanceNavigationTiming 继承自PerformanceEntryList）。

![PerformanceTiming 和 PerformanceNavigation 被弃用](https://img10.360buyimg.com/imagetools/jfs/t1/157877/34/8369/127459/6034d531E017ca82c/e27ff52a52190a11.jpg)

## PerformanceNavigation

PerformanceNavigation 的定义如下：

``` java
interface PerformanceNavigation {
  const unsigned short TYPE_NAVIGATE = 0;
  const unsigned short TYPE_RELOAD = 1;
  const unsigned short TYPE_BACK_FORWARD = 2;
  const unsigned short TYPE_RESERVED = 255;
  readonly attribute unsigned short type;
  readonly attribute unsigned short redirectCount;
  [Default] object toJSON();
};
```
PerformanceNavigation 中只有两个只读属性：type 和 redirectCount。

type 表示是如何导航到当前页面的，如下：
- TYPE_NAVIGATE (0)： 当前页面是通过点击链接，书签和表单提交，或者脚本操作，或者在url中直接输入地址，type值为0
- TYPE_RELOAD (1)：点击刷新页面按钮或者通过Location.reload()方法显示的页面，type值为1
- TYPE_BACK_FORWARD (2)：页面通过历史记录和前进后退访问时。type值为2
- TYPE_RESERVED (255)：任何其他方式，type值为255

redirectCount 表示在到达这个页面之前重定向了多少次。

## PerformanceTiming

PerformanceTiming 的定义如下：

``` java
interface PerformanceTiming {
  readonly attribute unsigned long long navigationStart;
  readonly attribute unsigned long long unloadEventStart;
  readonly attribute unsigned long long unloadEventEnd;
  readonly attribute unsigned long long redirectStart;
  readonly attribute unsigned long long redirectEnd;
  readonly attribute unsigned long long fetchStart;
  readonly attribute unsigned long long domainLookupStart;
  readonly attribute unsigned long long domainLookupEnd;
  readonly attribute unsigned long long connectStart;
  readonly attribute unsigned long long connectEnd;
  readonly attribute unsigned long long secureConnectionStart;
  readonly attribute unsigned long long requestStart;
  readonly attribute unsigned long long responseStart;
  readonly attribute unsigned long long responseEnd;
  readonly attribute unsigned long long domLoading;
  readonly attribute unsigned long long domInteractive;
  readonly attribute unsigned long long domContentLoadedEventStart;
  readonly attribute unsigned long long domContentLoadedEventEnd;
  readonly attribute unsigned long long domComplete;
  readonly attribute unsigned long long loadEventStart;
  readonly attribute unsigned long long loadEventEnd;
  [Default] object toJSON();
};
```

各个属性的定义如下：
- navigationStart：如果有前一个页面，则返回前一个页面卸载完的时间戳，如果没有上一个页面，返回当前页面创建时的时间
- unloadEventStart：如果当前 url 与上一个 url 是同源，则返回的值是指上一个页面卸载开始的时间，如果与上一个不同域或者没有上一个 url，则返回 0
- unloadEventEnd：如果当前 url 与上一个 url 是同源，则返回的值是指上一个页面卸载完成的时间，如果与上一个不同域或者没有上一个 url，则返回 0
- redirectStart：如果来源于同源的 url 重定向，则该值返回的是开始重定向的时间，如果不同源或者无重定向，返回为 0
- redirectEnd：如果来源于同源的 url 重定向，则该值返回的是重定向完成的时间，如果不同源或者无重定向，返回为 0
- fetchStart：如果要使用“GET”请求方法获取新资源，fetchStart 返回的是浏览器发起请求到检测缓存前时间，否则直接返回浏览器请求时间
- domainLookupStart：返回查询 DNS 开始时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 fetchStart
- domainLookupEnd：返回查询 DNS 结束时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 fetchStart
- connectStart：返回与服务端建立连接开始时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 domainLookupEnd
- connectEnd：返回与服务端建立连接完成时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 domainLookupEnd
- secureConnectionStart：如果是 https 请求并且这个属性是可获取的，则返回握手连接开始时间
- requestStart：返回向服务器发送请求时（或者从本地缓存读取时）开始时间
- responseStart：返回从服务器端（或者从本地缓存中）接收到第一个字节的时间
- responseEnd：返回从服务器端（或者从本地缓存中）接收到最后一个字节的时间
- domLoading：返回当前网页DOM结构开始解析时（即 Document.readyState属性变为“loading”、相应的 readystatechange事件触发时）的时间
- domInteractive：返回当前网页DOM结构结束解析、开始加载内嵌资源时（即Document.readyState属性变为 interactive”、相应的readystatechange事件触发时）的时间，domInteractive并非DOMReady，它早于DOMReady触发，表示DOM文档解析完毕（即 DOM tree创建完成），但内嵌资源（比如css、js等）还未加载的时间
- domContentLoadedEventStart：触发 DOMContentLoaded 事件开始时间
- domContentLoadedEventEnd：DOMContentLoaded 事件结束时间
- domComplete：返回当前文档解析完成，即 Document.readyState 变为 'complete' 且相对应的readystatechange 被触发时的时间
- loadEventStart：返回触发 onload 开始时间，当load事件尚未触发时，它返回零
- loadEventEnd：返回 onload 完成时间，当load事件尚未触发时，它返回零

这些属性对应的节点如下图：
![PerformanceTiming](https://img11.360buyimg.com/imagetools/jfs/t1/161348/21/7710/62981/6034d887Ef700f9a8/93bc74c1fe77d248.png)


## PerformanceNavigationTiming

PerformanceNavigationTiming 的定义如下：

``` java
interface PerformanceNavigationTiming : PerformanceResourceTiming {
    readonly        attribute DOMHighResTimeStamp unloadEventStart;
    readonly        attribute DOMHighResTimeStamp unloadEventEnd;
    readonly        attribute DOMHighResTimeStamp domInteractive;
    readonly        attribute DOMHighResTimeStamp domContentLoadedEventStart;
    readonly        attribute DOMHighResTimeStamp domContentLoadedEventEnd;
    readonly        attribute DOMHighResTimeStamp domComplete;
    readonly        attribute DOMHighResTimeStamp loadEventStart;
    readonly        attribute DOMHighResTimeStamp loadEventEnd;
    readonly        attribute NavigationType      type;
    readonly        attribute unsigned short      redirectCount;
    [Default] object toJSON();
};
```
这些属性对应的节点如下图：

![PerformanceNavigationTiming](https://img11.360buyimg.com/imagetools/jfs/t1/161861/15/7640/328340/6034dbb6Eb76a6896/512483da73d8865c.png)

上图中黄色区域是 PerformanceResourceTiming 的属性，我们计算时使用的 respondEnd 就是PerformanceResourceTiming 中的属性。

## PerformanceResourceTiming

PerformanceResourceTiming 的定义如下：

``` java
interface PerformanceResourceTiming : PerformanceEntry {
    readonly        attribute DOMString           initiatorType;
    readonly        attribute DOMString           nextHopProtocol;
    readonly        attribute DOMHighResTimeStamp workerStart;
    readonly        attribute DOMHighResTimeStamp redirectStart;
    readonly        attribute DOMHighResTimeStamp redirectEnd;
    readonly        attribute DOMHighResTimeStamp fetchStart;
    readonly        attribute DOMHighResTimeStamp domainLookupStart;
    readonly        attribute DOMHighResTimeStamp domainLookupEnd;
    readonly        attribute DOMHighResTimeStamp connectStart;
    readonly        attribute DOMHighResTimeStamp connectEnd;
    readonly        attribute DOMHighResTimeStamp secureConnectionStart;
    readonly        attribute DOMHighResTimeStamp requestStart;
    readonly        attribute DOMHighResTimeStamp responseStart;
    readonly        attribute DOMHighResTimeStamp responseEnd;
    readonly        attribute unsigned long long  transferSize;
    readonly        attribute unsigned long long  encodedBodySize;
    readonly        attribute unsigned long long  decodedBodySize;
    [Default] object toJSON();
};
```

各个属性的定义如下：

- initiatorType：返回值可能是 "css"、"xmlhttprequest"、"fetch"、"beacon"、"other" 等，initiator 类型
- nextHopProtocol：返回网路资源协议，可能返回为 ""
- workerStart：如果有 active worker，则返回 worker fetch 时间
- redirectStart：如果来源于同源的 url 重定向，则该值返回的是初始化重定向开始时间，否则返回 0
- redirectEnd：如果来源于同源的 url 重定向，则该值返回的是最后一个重定向接收最后一个字节时的时间，否则返回 0
- fetchStart：返回请求这个资源开始时间
- domainLookupStart：返回查询 DNS 开始时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 fetchStart，如果请求这个资源失败，返回 0
- domainLookupEnd：返回查询 DNS 结束时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 fetchStart，如果请求这个资源失败，返回 0
- connectStart：返回与服务端建立连接开始时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 fetchStart，如果请求这个资源失败，返回 0
- connectEnd：返回与服务端建立连接完成时间，如果是持久连接或者是从缓存中获取资源，则这个值等于 fetchStart，如果请求这个资源失败，返回 0
- secureConnectionStart：如果没有使用 https 或者资源加载失败，返回 0，如果持久连接或者从缓存或本地读取，则与 - fetchStart 相同，否则指握手连接时间 说明
- requestStart：返回向服务器发送请求时（或者从本地缓存读取时）开始时间
- responseStart：返回从服务器端（或者从本地缓存中）接收到第一个字节的时间
- responseEnd：返回从服务器端（或者从本地缓存中）接收到最后一个字节的时间
- transferSize：响应头和响应体大小，当从缓存中读取时，该值为 0，与 chrome 中 devtool Network里的size一致，该值还可以用来判断是否从缓存中读取的，如果为 0，表示从缓存中读取的
- encodedBodySize：返回响应体编码压缩后字节大小，该值减去 transferSize 等于响应头大小
- decodedBodySize：返回响应体编码压缩前的字节大小，该值比 encodedBodySize 大很多
- serverTiming：返回列表 PerformanceServerTiming，只有在 Web Workers 中有效

上面罗列了 Performance 的各种 API，有两个目的，最直接的目的是为了说明Performance我们用到的几个属性在页面加载时处在哪个节点上，另外一个目的就是了解 Performance 的兼容性，如果我们要自己写性能监控的功能，可以作为参考。

我们知道如何获取各个节点上的数值之后，接下来要考虑的就是在什么时候获取这些值了。

# 触发节点

我们再来看下【sgm-web 的统计指标】指标计算方式：

```
白屏时间 -> fpt -> responseEnd - startTime
页面加载完成时间 -> ldt -> loadEventEnd - startTime
HTML加载完成时间 -> rte -> domContentLoadedEventEnd - startTime
```

再回顾下这张图：
![PerformanceNavigationTiming](https://img11.360buyimg.com/imagetools/jfs/t1/161861/15/7640/328340/6034dbb6Eb76a6896/512483da73d8865c.png)

## 白屏时间

白屏时间指首次渲染时间，指页面出现第一个文字或图像所花费的时间。

从上图中我们可以知道，在 respondEnd 之前，还有页面 unload、重定向、DNS、TCP 连接、发送/获取请求等阶段，这些阶段都是 Resource Timing 中的节点。

## HTML 加载完成时间

HTML 加载完成时间对应 DOMContentLoaded 事件。我们来看下 MDN 对于 DOMContentLoaded 的解释：

> 当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完全加载。

DOMContentLoaded 需要使用 addEventListener 方法来捕获：

``` js
document.addEventListener('DOMContentLoaded', () => {
    console.log('DOMContentLoaded event');
});
```

从上面 MDN 的解释，我们可以知道，DOMContentLoaded 触发的条件是 DOM 树解析完成。同样的，MDN 也提供了优化的建议：

![DOMContentLoaded 优化](https://img10.360buyimg.com/imagetools/jfs/t1/157430/32/14734/152518/60580decE8b0c37a7/f48d8d86afd6c2d1.jpg)

1. JavaScript 脚本：
浏览器解析DOM时，如果在文档中遇到 script 标签，因为脚本可能会修改 DOM，所以浏览器在这时就会停止构建 DOM，直到文件加载并执行完成。

``` js
<script>
  document.addEventListener('DOMContentLoaded', () => {
    console.log('DOMContentLoaded');
  });
</script>

<script src="http://wechatfe.github.io/lib/vconsole/3.4.0/vconsole.min.js"></script>

<script>
  console.log('script loaded and executed');
</script>
```
因为 DOMContentLoaded 是在 script 加载并执行完成后才会触发，因此我们先看到 script loaded and executed, 等 vConsole.js 加载完成后才能看到 DOMContentLoaded 被触发。

如果有些不重要或者不影响页面渲染的脚本（比如分享脚本或者非首屏脚本），我们可以采用不阻塞 DOMContentLoaded 事件执行的方式加载：

- 增加async 属性：具有 async 特性（attribute）的脚本不会阻塞 DOMContentLoaded，如下图：

![async属性的script](https://pic4.zhimg.com/80/v2-59d63189e9fe4c165370d81512b4fe73_1440w.jpg)

- 动态添加：使用 document.createElement('script') 动态生成并添加到网页的脚本也不会阻塞 DOMContentLoaded。

2. 样式表：

因为 DOM 和 CSS 的解析是并行的，CSS 的加载不会影响 DOM 的解析。但是，由于脚本可能会操作之前的 DOM 节点和 CSS 样式，样式表会在后面的 js 执行前先加载，因此 css 会阻塞后面 js 的执行。所以，当 DOMContentLoaded 等待脚本时，它也在等待脚本前面的样式。
如果我们的 js 都在 css 的前面， css 的加载自然不会影响 DOMContentLoaded 的触发时间，但是使用这种方式的话， js 和 css 加载的过程中，页面都是没有样式的。

## 页面加载完成时间

页面加载完成时间指页面完全加载完所用的时间，这时候触发完成了 onload 事件。onload 事件触发时，浏览器不仅加载完成了 HTML，还加载完成了所有外部资源，如图片、样式等。这时候，外部资源已加载完成，样式已被应用，图片大小也已知了。

load 事件可以直接使用 window.onload 方法：

``` js
    window.onload = () => log('onload');
```

只有当所有资源被加载完了，load 事件才会被触发，所以除了 Resource Timing 中的网络要素外，所有的静态资源的加载、页面的渲染等都是影响页面加载完成性能数据的因素。

# 总结

总的来说，这篇文章只是大致理清了各指标数据的统计口径及上报节点，以及这些节点与哪些资源或者事件相关。如果要详细了解 Performance 各阶段优化的细节，之后我会出一个系列慢慢讲。 





