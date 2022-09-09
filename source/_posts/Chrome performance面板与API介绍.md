---
title: Chrome performance面板与API介绍
date: 2022-08-18 20:00:00
---

性能监测是前端性能优化的重要一环。监测的目的是为了确定性能瓶颈，从而更准确地开展具体的优化工作。平时我们比较推崇的性能监测方案主要有两种：**可视化方案、可编程方案**。这两种方案下都有非常优秀、且触手可及的相关工具，其中可视化检测的代表是performance：

## 可视化监测：Performance 面板

Performance 是 Chrome 提供给我们的开发者工具，用于记录和分析我们的应用在运行时的所有活动。它呈现的数据具有实时性、多维度的特点，可以帮助我们很好地定位性能问题。

### 开始记录

右键打开开发者工具，选中我们的 Performance 面板，当我们选中图中所标示的实心圆按钮，Performance 会开始帮我们记录我们后续的交互操作；当我们选中圆箭头按钮，Performance 会将页面重新加载，计算加载过程中的性能表现。
tips：使用 Performance 工具时，为了规避其它 Chrome 插件对页面的性能影响，我们最好在无痕模式下打开页面：

![示例](1.JPG)

当我们选中图中所标示的实心圆按钮，Performance 会开始帮我们记录我们后续的交互操作；当我们选中圆箭头按钮，Performance 会将页面重新加载，计算加载过程中的性能表现。
tips：使用 Performance 工具时，为了规避其它 Chrome 插件对页面的性能影响，我们最好在无痕模式下打开页面

### 简要分析

打开京东首页，选中 Performance 面板中的圆箭头，来看一下页面加载过程中的性能表现：

![示例](2.JPG)

从上到下，依次为概述面板、详情面板。下我们先来观察一下概述面板，了解页面的基本表现：



我们看右上角的三个栏目：FPS、CPU 和 NET。

**FPS**：这是一个和动画性能密切相关的指标，它表示每一秒的帧数。图中绿色柱状越高表示帧率越高，体验就越流畅。若出现红色块，则代表长时间帧，很可能会出现卡顿。图中以绿色为主，偶尔出现红块，说明网页性能并不糟糕，但仍有可优化的空间。

**CPU**：表示CPU的使用情况，不同的颜色片段代表着消耗CPU资源的不同事件类型。这部分的图像和下文详情面板中的Summary内容有对应关系，我们可以结合这两者挖掘性能瓶颈。

**NET**：粗略的展示了各请求的耗时与前后顺序。这个指标一般来说帮助不大。

门神相关：

**FCP**：首次内容绘制， 指标测量页面从开始加载到页面内容的任何部分在屏幕上完成渲染的时间。对于该指标，"内容"指的是文本、图像（包括背景图像）、`<svg>`元素或非白色的`<canvas>`元素。

**LCP**：最大内容绘制 ，指标会根据页面首次开始加载的时间点来报告可视区域内可见的最大图像或文本块完成渲染的相对时间。最大内容绘制 (LCP) 是测量的一个以用户为中心的重要指标，因为该项指标会在页面的主要内容基本加载完成时，在页面加载时间轴中标记出相应的点，迅捷的 LCP 有助于让用户确信页面是有效的，

​	根据当前[最大内容绘制 API](https://wicg.github.io/largest-contentful-paint/)中的规定，最大内容绘制考量的元素类型为：

- `<img>`元素
- 内嵌在`<svg>`元素内的`<image>`元素
- `<video>`元素（使用封面图像）
- 通过url()函数（而非使用CSS渐变）加载的带有背景图像的元素
- 包含文本节点或其他行内级文本元素子元素的块级元素

**CLS**：累积布局偏移 ，核心Web 指标中的一项指标，通过计算未在用户输入500 毫秒内发生的布局偏移的偏移分数总和来测量内容的不稳定性。 该项指标查看可视区域中可见内容的位移量以及受影响元素的位移距离

​	CLS 较差的最常见原因为：

- 无尺寸的图像
- 无尺寸的广告、嵌入和 iframe
- 动态注入的内容
- 导致不可见文本闪烁 (FOIT)/无样式文本闪烁 (FOUT) 的网络字体
- 在更新 DOM 之前等待网络响应的操作

**TTl**：可交互时间，测量页面从开始加载到主要子资源完成渲染，并能够快速、可靠地响应用户输入所需的时间。

​	如何改进TTL？

* 缩小 JavaScript 文件可以减少有效负载大小和脚本解析时间。借助一些压缩工具：[Terser](https://github.com/terser-js/terser)是一种流行的 JavaScript 压缩工具。webpack v4 默认包含这个库的插件来创建缩小的构建文件。

* 通过预链接提高页面加载速度，添加 `preconnect` 或 `dns-prefetch` 资源提示来建立与重要第三方源的早期连接。

* 声明预加载链接：举个例子

  ```javascript
  index.html
  |--app.js
     |--styles.css
     |--ui.js
  ```

  `index.html`文件声明`<script src="app.js">`. 运行时`app.js`，它会调用`fetch()`以下载`styles.css`和`ui.js`. 在下载、解析和执行最后 2 个资源之前，该页面不会显示为完整。

  在 HTML 中声明预加载链接，以指示浏览器尽快下载关键资源。

  <head>
    ...
    <link rel="preload" href="styles.css" as="style">
    <link rel="preload" href="ui.js" as="script">
    ...
  </head>

* 减少第三方代码的影响，将广告网络、社交媒体按钮、A/B 测试或分析服务添加到页面，通常需要将第三方脚本添加到 HTML中。这些第三方脚本会显着影响页面加载性能。

* 避免链接关键请求：关键请求链是一系列对页面呈现很重要的依赖网络请求。链的长度越长，下载量越大，对页面加载性能的影响就越大。

  使用关键请求链审计结果首先定位对页面加载影响最大的资源：

  - 最小化关键资源的数量：消除它们，推迟下载，将它们标记为 `async` 等等。
  - 优化关键字节数以减少下载时间（往返次数）。
  - 优化剩余关键资源的加载顺序：尽早下载所有关键资产，缩短关键路径长度。

* 减少JavaScript执行时间：当 JavaScript 需要很长时间来执行时，它会以多种方式降低页面性能：

  - 网络成本，更多的字节等于更长的下载时间。
  - 解析和编译成本，JavaScript 在主线程上被解析和编译。当主线程繁忙时，页面无法响应用户输入。
  - 执行成本，JavaScript 也在主线程上执行。如果页面在真正需要之前运行了大量代码，这也会延迟交互时间，这是与用户如何看待页面速度相关的关键指标之一。
  - 内存成本，如果 JavaScript 持有大量引用，则可能会消耗大量内存。页面在消耗大量内存时会出现卡顿或缓慢。内存泄漏会导致页面完全冻结。

* 最小化主线程工作：浏览器的渲染器进程将代码转换为用户可以与之交互的网页。默认情况下，渲染器进程的主线程通常处理大部分代码：它解析 HTML 并构建 DOM，解析 CSS 并应用指定的样式，以及解析、评估和执行 JavaScript。除此之外，主线程还处理用户事件。因此，每当主线程忙于做其他事情时，网页可能无法响应用户交互，从而导致糟糕的体验。

  **TBT**：总阻塞时间，指标测量First Contentful Paint 首次内容绘制 (FCP)与Time to Interactive 可交互时间 (TTI)之间的总时间，这期间，主线程被阻塞的时间过长，无法作出输入响应。每当出现长任务（在主线程上运行超过 50 毫秒的任务）时，主线程都被视作"阻塞状态"。我们说主线程处于"阻塞状态"是因为浏览器无法中断正在进行的任务。因此，如果用户在某个长任务运行期间与页面*进行*交互，那么浏览器必须等到任务完成后才能作出响应。

  

### 挖掘性能瓶颈

详情面板中的内容有很多。但一般来说，我们会主要去看 Main 栏目下的火焰图和 Summary 提供给我们的饼图——这两者和概述面板中的 CPU 一栏结合，可以帮我们迅速定位性能瓶颈。

![示例](3.JPG)

先看 CPU 图表和 Summary 饼图。CPU 图表中，我们可以根据颜色填充的饱满程度，确定 CPU 的忙闲，进而了解该页面的总的任务量。而 Summary 饼图则以一种直观的方式告诉了我们，哪个类型的任务最耗时。这样我们在优化的时候，就可以抓到“主要矛盾”，进而开展后续的工作了。

再看 Main 提供给我们的火焰图。这个火焰图非常关键，它展示了整个运行时主进程所做的每一件事情（包括加载、脚本运行、渲染、布局、绘制等）。x 轴表示随时间的记录。每个长条就代表一个活动。更宽的条形意味着事件需要更长时间。y 轴表示调用堆栈，我们可以看到事件是相互堆叠的，上层的事件触发了下层的事件。

CPU 图标和 Summary 图都是按照“类型”给我们提供性能信息，而 Main 火焰图则将粒度细化到了每一个函数的调用。到底是从哪个过程开始出问题、是哪个函数拖了后腿、又是哪个事件触发了这个函数，这些具体的、细致的问题都将在 Main 火焰图中得到解答。

## 可编程的性能上报方案： W3C 性能 API

W3C 规范为我们提供了 Performance 相关的接口。它允许我们获取到用户访问一个页面的每个阶段的精确时间，从而对性能进行分析。我们可以将其理解为 Performance 面板的进一步细化与可编程化。

当下的前端世界里，数据可视化的概念已经被炒得非常热了，Performance 面板就是数据可视化的典范。那么为什么要把已经可视化的数据再掏出来处理一遍呢？这是因为，需要这些数据的人不止我们前端——很多情况下，后端也需要我们提供性能信息的上报。此外，Performance 提供的可视化结果并不一定能够满足我们实际的业务需求，只有拿到了真实的数据，我们才可以对它进行二次处理，去做一个更加深层次的可视化。

在这种需求背景下，就需要用到Performance API了，Performance API用于精确度量、控制、增强浏览器的性能表现，为测量网站性能，提供以前没有办法做到的精度。在浏览器控制台输入 window.performance，可以看到有哪些属性

![示例](4.JPG)

### performance.timing对象

比如，为了得到脚本运行的准确耗时，需要一个高精度时间戳。传统的做法是使用Date对象的getTime方法。

```javascript
var start = new Date().getTime();

var now = new Date().getTime();

var latency = now - start;

console.log("任务运行时间：" + latency);
```

上面这种做法有两个不足之处。首先，getTime方法（以及Date对象的其他方法）都只能精确到毫秒级别（一秒的千分之一），想要得到更小的时间差别就无能为力了；其次，这种写法只能获取代码运行过程中的时间进度，无法知道一些后台事件的时间进度，比如浏览器用了多少时间从服务器加载网页。

为了解决这两个不足之处，ECMAScript 5引入“高精度时间戳”这个API，部署在performance对象上。它的精度可以达到1毫秒的千分之一（1秒的百万分之一），这对于衡量的程序的细微差别，提高程序运行速度很有好处，而且还可以获取后台事件的时间进度。

目前，所有主要浏览器都已经支持performance对象，包括Chrome 20+、Firefox 15+、IE 10+、Opera 15+。

#### performance.timing对象

performance对象的timing属性指向一个对象，它包含了各种与浏览器性能有关的时间数据，提供浏览器处理网页各个阶段的耗时。

performance.timing对象包含以下属性（全部为只读）：

- **navigationStart**：当前浏览器窗口的前一个网页关闭，发生unload事件时的Unix毫秒时间戳。如果没有前一个网页，则等于fetchStart属性。

- **unloadEventStart**：如果前一个网页与当前网页属于同一个域名，则返回前一个网页的unload事件发生时的Unix毫秒时间戳。如果没有前一个网页，或者之前的网页跳转不是在同一个域名内，则返回值为0。

- **unloadEventEnd**：如果前一个网页与当前网页属于同一个域名，则返回前一个网页unload事件的回调函数结束时的Unix毫秒时间戳。如果没有前一个网页，或者之前的网页跳转不是在同一个域名内，则返回值为0。

- **redirectStart**：返回第一个HTTP跳转开始时的Unix毫秒时间戳。如果没有跳转，或者不是同一个域名内部的跳转，则返回值为0。

- **redirectEnd**：返回最后一个HTTP跳转结束时（即跳转回应的最后一个字节接受完成时）的Unix毫秒时间戳。如果没有跳转，或者不是同一个域名内部的跳转，则返回值为0。MDN官方文档：因为 [Navigation Timing 规范](https://w3c.github.io/navigation-timing/#obsolete)已被弃用，此特性不再有望成为标准。

- **fetchStart**：返回浏览器准备使用HTTP请求读取文档时的Unix毫秒时间戳。该事件在网页查询本地缓存之前发生。

- **domainLookupStart**：返回域名查询开始时的Unix毫秒时间戳。如果使用持久连接，或者信息是从本地缓存获取的，则返回值等同于fetchStart属性的值。

- **domainLookupEnd**：返回域名查询结束时的Unix毫秒时间戳。如果使用持久连接，或者信息是从本地缓存获取的，则返回值等同于fetchStart属性的值。

- **connectStart**：返回HTTP请求开始向服务器发送时的Unix毫秒时间戳。如果使用持久连接（persistent connection），则返回值等同于fetchStart属性的值。

- **connectEnd**：返回浏览器与服务器之间的连接建立时的Unix毫秒时间戳。如果建立的是持久连接，则返回值等同于fetchStart属性的值。连接建立指的是所有握手和认证过程全部结束。

  

注意:这里握手结束，包括安全连接建立完成、SOCKS 授权通过



- **secureConnectionStart**：返回浏览器与服务器开始安全链接的握手时的Unix毫秒时间戳。如果当前网页不要求安全连接，则返回0。
- **requestStart**：返回浏览器向服务器发出HTTP请求时（或开始读取本地缓存时）的Unix毫秒时间戳。
- **responseStart**：返回浏览器从服务器收到（或从本地缓存读取）第一个字节时的Unix毫秒时间戳。
- **responseEnd**：返回浏览器从服务器收到（或从本地缓存读取）最后一个字节时（如果在此之前HTTP连接已经关闭，则返回关闭时）的Unix毫秒时间戳。
- **domLoading**：返回当前网页DOM结构开始解析时（即Document.readyState属性变为“loading”、相应的readystatechange事件触发时）的Unix毫秒时间戳。
- **domInteractive**：返回当前网页DOM结构结束解析、开始加载内嵌资源时（即Document.readyState属性变为“interactive”、相应的readystatechange事件触发时）的Unix毫秒时间戳。

注意:只是 DOM 树解析完成，这时候并没有开始加载网页内的资源

- **domContentLoadedEventStart**：DOM 解析完成后，网页内资源加载开始的时间,文档发生 DOMContentLoaded事件的时间。
- **domContentLoadedEventEnd**：DOM 解析完成后，网页内资源加载完成的时间（如 JS 脚本加载执行完毕），文档的DOMContentLoaded 事件的结束时间。
- **domComplete**：DOM 树解析完成，且资源也准备就绪的时间，Document.readyState 变为 complete，并将抛出 readystatechange 相关事件。
- **loadEventStart**：返回当前网页load事件的回调函数开始时的Unix毫秒时间戳。如果该事件还没有发生，返回0。
- **loadEventEnd**：返回当前网页load事件的回调函数运行结束时的Unix毫秒时间戳。如果该事件还没有发生，返回0。

常用计算：

DNS查询耗时 ：`domainLookupEnd - domainLookupStart`
TCP链接耗时 ：`connectEnd - connectStart`
request请求耗时 ：`responseEnd - responseStart`
解析dom树耗时 ： `domComplete - domInteractive`
白屏时间 ：`responseStart - navigationStart`
domready时间（用户可操作时间节点） ：`domContentLoadedEventEnd - navigationStart`
onload时间（总下载时间）：`loadEventEnd - navigationStart`

这些API的罗列有些枯燥且抽象，来看一张图，浏览器输入URL到页面加载的过程，在看之前，抛出一道经典题目：从输入URL到页面加载发生了什么？

打开浏览器，输入URL，到页面展示出来，这个中间大致经历了这些过程：

1. 输入URL
2. DNS解析
3. TCP握手
4. HTTP请求
5. HTTP响应返回数据
6. 浏览器解析并渲染页面

上面粗劣的介绍了输入URL到页面加载的大致过程，但是缺少更加详细的过程，事实上w3c给我们提供了一个接口performance.timing更加详细地介绍了每个过程，并且可以通过这个过程获取页面性能数据。如下图所示：

![示例](5.JPG)

上图的过程大致可以分为三个大的阶段：

1. 缓存相关：主要包括Prompt for unload，redirect和App cache3个过程
2. 网络相关：主要包括DNS，TCP和HTTP(Request，Response)3个过程
3. 浏览器相关：主要包括Processing和onload两个过程

通过将整个过程细分为3个大的阶段，然后再每个阶段的介绍

##### 缓存相关

* 卸载已有的页面（Prompt for unload）

  我们在页面中输入URL时，首先会卸载掉原来的页面。这是为了释放页面占据的内存，否则没请求一次URL都占据一份内存，会导致浏览器占据内存越来越大。

* 重定向（redirect）

  所谓的重定向实际上就是先从本地缓存中去查找请求的内容，如果本地缓存中有则直接使用，如果没有则向服务器进行请求(这只是简单的理解，实际上如何获取数据是存在缓存策略的)。事实上，每次从服务器获取到文件，文件会被暂时存放到一个指定区域，当我们下次再次请求这个文件时，浏览器会首先从这个区域查看是否已经存在过这个文件，如果已经存在，则不需要再次进行请求数据。

* App cache

  全称Application Cache，HTML5的新特性，允许浏览器在本地存储页面所需要的资源，使得页面离线也可以访问

##### 网络相关

* DNS

  DNS(Domain Name System)域名系统，顾名思义是用来解析域名系统的。在网络中，我们人适合于记忆文本，因此我们输入的都是 www.google.com 这种字符串，但是计算机适合于处理数字，每一台计算机对应的是一个IP地址。因此，如果我们要访问一个指定的资源，必须先找到对应的服务器，而找到服务器需要先将域名转换为对应的IP地址。而DNS就是帮助我们实现这个过程。

  ###### 域名级别：

  顶级域名在开头有一个点(.com .cn .net)
  一级域名就是在"com cn net"前加一级 (google.com)
  二级域名就是在一级域名前再加一级(www.google.com)
  二级域名及以上级别的域名，统称为子域名，不在注册域名的范畴中

  ###### 域名服务器：

  ![示例](6.JPG)

  域名的解析需要用到一系列的服务器，而不是简单的一个服务器。比如：用户想要解析www.google.com:

  1. 在本机上输入www.google.com
  2. 2号服务器是用户在自己电脑上填写的DNS地址，由于域名和ip地址的对照表非常庞大，因此2号服务器会进行分层管理。2号服务器进行域名解析是会先从缓存中进行查找，如果一个域名被频繁访问，通常会被保存到缓存中。如果DNS这没有对应的域名-IP缓存，那么就需要向根服务器(Root Server)发起请求。
  3. 根服务器负责维护全球的域名-IP地址解析。根服务器会检查域名后缀(比如.com)，根据不同的后缀，交给不同的TLD服务器处理。获取到后缀后，返回对应的TLD服务器的ip地址(com = 1.1.1.1)。
  4. DNS拿到TLD服务器的IP地址后，继续向TLD服务器进行询问。TLD服务器只返回顶级域名对应的IP(google.com = 2222)，交给顶级域名对应的Name Server处理。
  5. DNS服务器获取到顶级域名的IP后，继续向Name Server进行询问。Name Server返回具体的域名对应的IP地址。
  6. DNS服务器获取到具体的域名对应的IP后，会先进行缓存，避免下次请求时继续多次询问。

* TCP

  TCP是HTTP的下层协议，我们想要通过HTTP进行请求，必须先通过TCP进行连接，也就是说HTTP是依赖于TCP的。TCP的作用就是连接指定IP地址的服务器(通过DNS已经获取到对应的服务器IP地址)。

  每次连接的时候，TCP都会经历三次握手，每次断开连接时TCP都会经历四次挥手。这些过程就是可以优化的地方，这里不做阐释。

* HTTP请求（Resquest）和相应（Response）

  在 HTTP/1.x 中，如果客户端要想发起多个并行请求以提升性能，则必须使用多个 TCP 连接。 这是 HTTP/1.x 交付模型的直接结果，该模型可以保证每个连接每次只交付一个响应（响应排队）。 更糟糕的是，这种模型也会导致队首阻塞，从而造成底层 TCP 连接的效率低下。 也就是说在目前的HTTP1.X的协议下，浏览器对资源的并发请求个数是有限制的。 等到HTTP2到来的时候，通过二进制分帧层进行优化。 HTTP/2 中新的二进制分帧层突破了这些限制，实现了完整的请求和响应复用：客户端和服务器可以将 HTTP 消息分解为互不依赖的帧，然后交错发送，最后再在另一端把它们重新组装起来。

##### 浏览器相关

* 文档解析和DOM的加载（Processing）

  HTTP请求后返回的是一个文本，我们需要将文本转换成DOM树，然后加载DOM

* 触发Onload时间（onload）

  DOM加载完成之后，触发onload事件。

### performance.memory

描述内存多少，是在Chrome中添加的一个非标准属性。单位字节
`jsHeapSizeLimit`：内存大小限制
`totalJSHeapSize`：可使用的内存
`usedJSHeapSize`：JS对象（包括V8引擎内部对象）占用的内存，不能大于totalJSHeapSize，如果大于，有可能出现了内存泄漏

### performance.navigation对象

除了时间信息，performance还可以提供一些用户行为信息，主要都存放在performance.navigation对象上面。

它有两个属性：

**（1）performance.navigation.type**

该属性返回一个整数值，表示网页的加载来源，可能有以下4种情况：

- **0**：网页通过点击链接、地址栏输入、表单提交、脚本操作等方式加载，相当于常数performance.navigation.TYPE_NAVIGATENEXT。
- **1**：网页通过“重新加载”按钮或者location.reload()方法加载，相当于常数performance.navigation.TYPE_RELOAD。
- **2**：网页通过“前进”或“后退”按钮加载，相当于常数performance.navigation.TYPE_BACK_FORWARD。
- **255**：任何其他来源的加载，相当于常数performance.navigation.TYPE_UNDEFINED。

**（2）performance.navigation.redirectCount**

该属性表示当前网页经过了多少次重定向跳转。

### performance.now()

返回当前网页自从`performance.timing.navigationStart`到当前时间之间的毫秒数。

```javascript
performance.now()

Date.now() - (performance.timing.navigationStart + performance.now())
```

上面代码表示，`performance.timing.navigationStart`加上`performance.now()`，近似等于`Date.now()`，也就是说，`Date.now()`可以替代`performance.now()`。但是，由于`performance.now()`带有小数，因此精度更高。

通过两次调用`performance.now()`方法，可以得到间隔的准确时间，用来衡量某种操作的耗时，伪代码如下：

```javascript
var start = performance.now();
doTasks();
var end = performance.now();

console.log('耗时：' + (end - start) + '毫秒。');
```

### performance.mark()

在浏览器中，根据名称生成高精度时间戳。也就是常说的“打点”。

标记 的 performance entry将具有以下属性值:

- `entryType` - 设置为 "mark".
- `name` - 设置为mark被创建时给出的 "name"
- `startTime` - 设置`为 mark()` 方法被调用时的 timestamp 。
- `duration` - 设置为 "0" (标记没有持续时间)

### performance.measure()

是指定两个`mark`点之间的时间戳。如果说`mark`可以理解为"打点"的话，`measure`就可以理解为"连线"。

计算两个mark之间的时长，创建一个`DOMHighResTimeStamp`保存在资源缓存数据中，可通过`performance.getEntries()`等相关接口获取。

- `entryType` 为字符串 `measure`
- `name` 为创建时设置的值
- `startTime`为调用 measure 时的时间
- `duration`为两个 mark 之间的时长

如何计算两个标记之间的时间差？

```javascript
performance.mark('start1');

setTimeout(()=>{
   performance.mark('end1')
 performance.measure('d1', 'start1', 'end1')
 console.log(performance.getEntriesByName('d1')[0]);
},1000)
```

### performance.getEntries()

浏览器获取网页时，会对网页中每一个对象（脚本文件、样式表、图片文件等等）发出一个HTTP请求。performance.getEntries方法以数组形式，返回这些请求的时间统计信息，有多少个请求，返回数组就会有多少个成员。

由于该方法与浏览器处理网页的过程相关，所以只能在浏览器中使用。
