---
title: 你不知道的scroll事件绑定
date: 2021-12-30 17:44:45
tags:
  - H5
categories: scroll事件
keywords: scroll事件
description: scroll事件
cover: 
top_img: 
---

### 前言
在Web开发中，Scroll事件是一个很重要的事件。通过Scroll事件，我们可以通过一些方式获知视口在页面的位置。知道这个信息可以帮助我们判断很多东西，如用户即将浏览到页面底部，是不是该调用API加载一些新的内容等等。

在实际运用中，scroll事件常常带来很多迷惑。最近在开发某个需要监听滚动时间的项目时就遇到了一个问题，在react中的组件绑定scroll事件，事件处理函数似乎不会触发。这个问题好像困扰过很多人， 滚动事件失效能发现很多相关内容。比如说这一个问题，可以一个个去试他们给出的解决方案，总会发现几个有用的。但是作为开发者，知其然还要知其所以然。那么，本文就来探究一下这个scroll事件的背后到底有什么不为人知的东西。



### JS事件模型

大家都知道，JavaScript事件有两个阶段：（1）捕获阶段(Capture Phase)（2）冒泡阶段(Bubble Phase)。捕获阶段是事件从document到传递到目标元素的过程，而冒泡阶段是事件从目标元素传递到document的过程。


在平时，大家一般是监听事件的冒泡阶段，即elem.addEventListener('scroll', handler)。Vue中也提供了v-on指令监听某个事件，默认用的也是事件的冒泡阶段。那么这里会有什么坑呢？

根据MDN对scroll事件的描述，我们发现了惊人的事实：
- 冒泡: element的scroll事件不冒泡, 但是document的defaultView的scroll事件冒泡

这句话的意思就是说，如果scroll的目标元素是一个元素的话，比如说是一个div元素。那么此时事件只有从document到div的捕获阶段以及div的冒泡阶段。如果尝试在父级监视scroll的冒泡阶段监视这一事件是无效的。如果scroll是由document.defaultView（目前document关联的window对象）产生的有冒泡阶段。但是由于其本身就是DOM树里最顶级的对象，因此只能在window里监视scroll的捕获阶段以及冒泡阶段。

注意到在元素为目标元素时，也在目标元素上监视scroll事件的捕获阶段以及冒泡阶段。两种情况其实内在是一致的。接下来我们会通过两个Demo证明这一点。


#### 当scroll的目标元素为document.defaultView时
- 这是最常见的情形，比如可以在console里执行window.addEventListener('scroll', e => console.log(e.target), true)，然后滑动滚动条就可以看见效果。

接下来，我们分别在window与div#main上监听scroll事件的捕获阶段以及冒泡阶段；

  ```html
    <body>
      <div id="main">
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
        <div>111111</div>
      </div>
    <script src="scroll-in-window.js"></script>
  </body>
    
```

  

```js
//scroll-in-window.js
window.addEventListener('scroll', e => console.log('scroll--capture---capture-------'), true)
window.addEventListener('scroll', e => console.log('scroll--bubble---bubble-------'))

const root = document.getElementById('main')
root.addEventListener('scroll', e => console.log('scroll--capture---capture-------'), true)
root.addEventListener('scroll', e => console.log('scroll--bubble---bubble-------'))

```
可以看到在全程只有window上监听scroll的捕获阶段以及冒泡阶段的回调函数执行了。这验证之前的结论：
- 如果scroll是由document.defaultView（目前document关联的window对象）产生的有冒泡阶段。但是由于其本身就是DOM树里最顶级的对象，因此只能在window里监视scroll的捕获阶段以及冒泡阶段

### 当scroll的目标元素是元素时

这种情况也常见，我给出一种示例：一个双栏布局，左侧为导航栏，右侧为内容，父容器使用Flex布局，要求导航栏不随内容的滑动而滑动。HTML文档如下：


```html


<!DOCTYPE html>
<html>

<head>
  <title>Scroll In Element</title>
  <style>
    body {
      margin: 0;
    }

    #wrapper {
      width: 100%;
      height: 100vh;
      display: flex;
      overflow: hidden;
    }

    #left-col {
      overflow: hidden;
      flex: 0 0 300px;
      background-color: aqua;
    }

    #right-col {
      flex: 1;
      overflow: auto;
    }
  </style>
</head>

<body>
  <div id="wrapper">
    <div id="left-col">
      <nav>
        barabara
      </nav>
    </div>
    <div id="right-col">
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
      <div>111111</div>
    </div>
  </div>
  <script src="scroll-in-element.js"></script>
</body>

</html>
    
```
此时scroll事件的目标元素是div#right-col。接下来我们，分别在window,div#wrapper,div#right-col上监听scroll的捕获阶段以及冒泡阶段

```js
//scroll-in-element.js
const log = (elem, phase) => {
  console.log(`scroll handler is trigged in ${elem} during ${phase}`)
}

const bindEvent = (elem, elemName)  => {
  elem.addEventListener('scroll', log.bind(null, elemName, 'capture'), true)
  elem.addEventListener('scroll', log.bind(null, elemName, 'bubble'))
}

bindEvent(window, 'window')
const wrapper = document.querySelector('#wrapper')
bindEvent(wrapper, 'wrapper')
const rightCol = document.querySelector('#right-col')
bindEvent(rightCol, 'rightCol')

```
这里的运行结果很清楚的展示了scroll事件的传递轨迹，也证明了展示了我们之前说的：

- 如果scroll的目标元素是一个元素的话，比如说是一个div元素。那么此时事件只有从document到div的捕获阶段以及div的冒泡阶段。


### 浏览器兼容

- 我们知道关于事件绑定是有兼容性的，在IE6以下是不兼容addEventListener,所以我们使用Scroll事件绑定的时候，也是需要有兼容的，最好是维护一个公共的方法，调用公用的事件绑定。
下面直接上代码

    ```js
     /**
     * @description 绑定事件 on(element, event, handler)
     */
      export const on = (() => {
        if (document.addEventListener) {
          return (element: any, event: any, handler: any) => {
            if (element && event && handler) {
              element.addEventListener(event, handler, false)
            }
          }
        }

        return (element: any, event: any, handler: any) => {
          if (element && event && handler) {
            element.attachEvent(`on${event}`, handler)
          }
        }
      })()

      /**
       * @description 解绑事件 off(element, event, handler)
       */
      export const off = (() => {
        if (document.removeEventListener) {
          return (element: any, event: any, handler: any) => {
            if (element && event) {
              element.removeEventListener(event, handler, false)
            }
          }
        }

        return (element: any, event: any, handler: any) => {
          if (element && event) {
            element.detachEvent(`on${event}`, handler)
          }
        }
      })()
    ```
    

### 建议

在M端列表数据很长的时候，不管是单独开启倒计时还是页面只开启一个倒计时，都会有性能问题，所以当数据超过1000条时，一般我们需要获取当前屏幕数据做处理，而不是操作整个列表数据。
获取当前屏幕的优点显而易见，它大大减少了数据操作的长度，但是也有缺点，因为你需要不断的去获取当前屏幕的dom，或者说视窗窗口的index下标，这也额外的增加了不必要的非业务逻辑；所以我们建议当已知列表的长度可能超过1000条数据时，再开启获取当前屏方法；

1. scroll事件的目标元素是什么？也可以说谁产生了scroll事件。如果实在不清楚的话，可以用Chrome开发者工具的Performance模块录制一下滚动动作，然后在Event Log里查看scroll事件的目标元素；

2. 监听scroll事件的元素是否在scroll事件的传递路径上，是否监听了正确的阶段？
- 弄清楚这些基本上就能够判定为什么scroll事件的回调触发不了了。

另外有一个万能的方法，就是在window上监听scroll的捕获阶段，即window.addEventListener('scroll', handler, true)。这也是很多类似问题中答主回答的答案。希望在看完本文之后能对其原理有所了解。


### 总结

当触发元素为div时，事件从document（在浏览器的实现上是window）开始向下传递，直到目标元素，此时捕获阶段结束。接下来在目标元素上进行一次冒泡阶段，然后不再冒泡。

对于document.defaultView产生的scroll事件一样，由于其本身就是顶层元素，在其本身上冒泡可以视为冒泡阶段结束。

  

