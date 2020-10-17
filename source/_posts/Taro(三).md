---
title: Taro(三)
date: 2020-10-17 14:00:09
cover: "https://m.360buyimg.com/img/jfs/t1/118880/20/18814/113416/5f6ded26Ec120c939/bd3281048222ce87.png"
---


## 七、小程序运行时

为了使 Taro 组件转换成小程序组件并运行在小程序环境下， Taro 主要做了两个方面的工作：编译以及运行时适配。编译过程会做很多工作，例如：将 JSX 转换成小程序 .wxml 模板，生成小程序的配置文件、页面及组件的代码等等。编译生成好的代码仍然不能直接运行在小程序环境里，那运行时又是如何与之协同工作的呢？…

### 7.1 注册程序、页面以及自定义组件

在小程序中会区分程序、页面以及组件，通过调用对应的函数，并传入包含生命周期回调、事件处理函数等配置内容的 object 参数来进行注册：

```js
Component({
  data: {},
  methods: {
    handleClick() {},
  },
});
```

而在 Taro 里，它们都是一个组件类：

```js
class CustomComponent extends Component {
  state = { }
  handleClick () { }
}...
```

- 那么 Taro 的组件类是如何转换成小程序的程序、页面或组件的呢？
- 例如，有一个组件：customComponent，编译过程会在组件底部添加一行这样的代码（此处代码作示例用，与实际项目生成的代码不尽相同）：

```js
Component(createComponent(customComponent));
```

- createComponent 方法是整个运行时的入口，在运行的时候，会根据传入的组件类，返回一个组件的配置对象
  > 在小程序里，程序的功能及配置与页面和组件差异较大，因此运行时提供了两个方法 createApp 和 createComponent 来分别创建程序和组件（页面）。createApp 的实现非常简单

createComponent 方法主要做了这样几件事情：

- 将组件的 state 转换成小程序组件配置对象的 data
- 将组件的生命周期对应到小程序组件的生命周期
- 将组件的事件处理函数对应到小程序的事件处理函数

### 7.2 组件 state 转换

其实在 Taro（React） 组件里，除了组件的 state，JSX 里还可以访问 props 、render 函数里定义的值、以及任何作用域上的成员。而在小程序中，与模板绑定的数据均来自对应页面（或组件）的 data。因此 JSX 模板里访问到的数据都会对应到小程序组件的 data 上。接下来我们通过列表渲染的例子来说明 state 和 data 是如何对应的…

在 JSX 里访问 state

> 在小程序的组件上使用 wx:for 绑定一个数组，就可以实现循环渲染。例如，在 Taro 里你可能会这么写：

```js
{
  state = {
    list: [1, 2, 3]
  }
  render () {
    return (
      <View>
        {this.state.list.map(item => <View>{item}</View>)}
      </View>
    )
  }
}
```

编译后的小程序组件模板：

```js
<view>
  <view wx:for="{{list}}" wx:for-item="item">
    {{ item }}
  </view>
</view>
```

其中 state.list 只需直接对应到小程序（页面）组件的 data.list 上即可…

在 render 里生成了新的变量

然而事情通常没有那么简单，在 Taro 里也可以这么用

```js
{
  state = {
    list = [1, 2, 3]
  }
  render () {
    return (
      <View>
        {this.state.list.map(item => ++item).map(item => <View>{item}</View>)}
      </View>
    )
  }
}
```

编译后的小程序组件模板是这样的：

```js
<view>
  <view wx:for="{{$anonymousCallee_1}}" wx:for-item="item">{{item}}</view>
</view>...
```

> 在编译时会给 Taro 组件创建一个 \_createData 的方法，里面会生成 $anonymousCallee_1 这个变量， $anonymousCallee**1 是由编译器生成的，对 this.state.list 进行相关操作后的变量。 \$anonymousCallee**1 最终会被放到组件的 data 中给模板调用：

```js
var $anonymousCallee_1 = this.state.list.map(function(item) {
  return ++item;
});
```

> render 里 return 之前的所有定义变量或者对 props、state 计算产生新变量的操作，都会被编译到 \_createData 方法里执行，这一点在前面 JSX 编译成小程序模板的相关文章中已经提到。每当 Taro 调用 this.setState API 来更新数据时，都会调用生成的 \_createData 来获取最新数据…

### 7.3 将组件的生命周期对应到小程序组件的生命周期

> 初始化过程里的生命周期对应很简单，在小程序的生命周期回调函数里调用 Taro 组件里对应的生命周期函数即可，例如：小程序组件 ready 的回调函数里会调用 Taro 组件的 componentDidMount 方法。它们的执行过程和对应关系如下图…

![](https://m.360buyimg.com/img/jfs/t1/88595/18/7355/40049/5dfb4602E7f9b20b7/8a7e4b779f5c9422.jpg)

小程序页面的 componentWillMount 有一点特殊，会有两种初始化方式。由于小程序的页面需要等到 onLoad 之后才可以获取到页面的路由参数，因此如果是启动页面，会等到 onLoad 时才会触发。而对于小程序内部通过 navigateTo 等 API 跳转的页面，Taro 做了一个兼容，调用 navigateTo 时将页面参数存储在一个全局对象中，在页面 attached 的时候从全局对象里取到，这样就不用等到页面 onLoad 即可获取到路由参数，触发 componentWillMount 生命周期…

状态更新

![](https://m.360buyimg.com/img/jfs/t1/106521/3/7426/31329/5dfb463bE58b19784/746931b709b55cfb.jpg)

- Taro 组件的 setState 行为最终会对应到小程序的 setData。Taro 引入了如 nextTick ，编译时识别模板中用到的数据，在 setData 前进行数据差异比较等方式来提高 setState 的性能。
- 如上图，组件调用 setState 方法之后，并不会立刻执行组件更新逻辑，而是会将最新的 state 暂存入一个数组中，等 nextTick 回调时才会计算最新的 state 进行组件更新。这样即使连续多次的调用 setState 并不会触发多次的视图更新。在小程序中 nextTick 是这么实现的…

```js
const nextTick = (fn, ...args) => {
  fn = typeof fn === 'function' ? fn.bind(null, ...args) : fn
  const timerFunc = wx.nextTick ? wx.nextTick : setTimeout
  timerFunc(fn)
}...
```

除了计算出最新的组件 state ，在组件状态更新过程里还会调用前面提到过的 \_createData 方法，得到最终小程序组件的 data，并调用小程序的 setData 方法来进行组件的更新

### 7.4 事件处理函数对应

在小程序的组件里，事件响应函数需要配置在 methods 字段里。而在 JSX 里，事件是这样绑定的：

```js
<View onClick={this.handleClick}></View>
```

编译的过程会将 JSX 转换成小程序模板：

```js
<view bindclick="handleClick"></view>...
```

在 createComponent 方法里，会将事件响应函数 handleClick 添加到 methods 字段中，并且在响应函数里调用真正的 this.handleClick 方法。

在编译过程中，会提取模板中绑定过的方法，并存到组件的 \$events 字段里，这样在运行时就可以只将用到的事件响应函数配置到小程序组件的 methods 字段中。

在运行时通过 processEvent 这个方法来处理事件的对应，省略掉处理过程，就是这样的…

```js
function processEvent(eventHandlerName, obj) {
  obj[eventHandlerName] = function(event) {
    // ...
    scope[eventHandlerName].apply(callScope, realArgs);
  };
}
```

这个方法的核心作用就是解析出事件响应函数执行时真正的作用域 callScope 以及传入的参数。在 JSX 里，我们可以像下面这样通过 bind 传入参数：

```js
<View onClick={this.handleClick.bind(this, arga, argb)}></View>
```

小程序不支持通过 bind 的方式传入参数，但是小程序可以用 data 开头的方式，将数据传递到 event.currentTarget.dataset 中。编译过程会将 bind 方式传递的参数对应到 dataset 中，processEvent 函数会从 dataset 里取到传入的参数传给真正的事件响应函数。

至此，经过编译之后的 Taro 组件终于可以运行在小程序环境里了…

### 7.5 对 API 进行 Promise 化的处理

> Taro 对小程序的所有 API 进行了一个分类整理，将其中的异步 API 做了一层 Promise 化的封装。例如，wx.getStorage 经过下面的处理对应到 Taro.getStorage(此处代码作示例用，与实际源代码不尽相同)

```js
Taro['getStorage'] = options => {
  let obj = Object.assign({}, options)
  const p = new Promise((resolve, reject) => {
	['fail', 'success', 'complete'].forEach((k) => {
	  obj[k] = (res) => {
	    options[k] && options[k](res)
	    if (k === 'success') {
		  resolve(res)
	    } else if (k === 'fail') {
		  reject(res)
	    }
	  }
	})
	wx['getStorage'](obj)
  })
  return p
}...
```

就可以这么调用了：

```js
// 小程序的调用方式
Taro.getStorage({
  key: 'test',
  success() {

  }
})
// 在 Taro 里也可以这样调用
Taro.getStorage({
  key: 'test'
}).then(() => {
  // success
})...
```

## 八、H5 运行时

### 8.1 H5 运行时解析

> 首先，我们选用 Nerv 作为 Web 端的运行时框架。你可能会有问题：同样是类 React 框架，为何我们不直接用 React，而是用 Nerv 呢？
> 为了更快更稳。开发过程中前端框架本身有可能会出现问题。如果是第三方框架，很有可能无法得到及时的修复，导致整个项目的进度受影响。Nerv 就不一样。作为团队自研的产品，出现任何问题我们都可以在团队内部快速得到解决。与此同时，Nerv 也具有与 React 相同的 API，同样使用 Virtual DOM 技术进行优化，正常使用与 React 并没有区别，完全可以满足我们的需要。

使用 Taro 之后，我们书写的是类似于下图的代码…

![](https://m.360buyimg.com/img/jfs/t1/85585/15/7357/213299/5dfb48dbE72b5b45a/30977175349dae04.png)

我们注意到，就算是转换过的代码，也依然存在着 view、button 等在 Web 开发中并不存在的组件。如何在 Web 端正常使用这些组件？这是我们碰到的第一个问题

### 8.1.1 组件实现

![](https://m.360buyimg.com/img/jfs/t1/94514/10/7444/106813/5dfb4900Ecaea3260/acbb909c42c72547.png)

作为开发者，你第一反应或许会尝试在编译阶段下功夫，尝试直接使用效果类似的 Web 组件替代：用 div 替代 view，用 img 替代 image，以此类推。

费劲心机搞定标签转换之后，上面这个差异似乎是解决了。但很快你就会碰到一些更加棘手的问题：hover-start-time、hover-stay-time 等等这些常规 Web 开发中并不存在的属性要如何处理？

回顾一下：在前面讲到多端转换的时候，我们说到了 babel。在 Taro 中，我们使用 babylon 生成 AST，babel-traverse 去修改和移动 AST 中的节点。但 babel 所做的工作远远不止这些。

我们不妨去 babel 的 playground 看一看代码在转译前后的对比：在使用了@babel/preset-env 的 BUILT-INS 之后，简单的一句源码 new Map()，在 babel 编译后却变成了好几行代码…

![](https://m.360buyimg.com/img/jfs/t1/107508/31/1180/104356/5dfb4930Eca3a9a65/17016a525139f5d7.png)

注意看这几个文件：core-js/modules/web.dom.iterable，core-js/modules/es6.array.iterator，core-js/modules/es6.map。我们可以在 core-js 的 Git 仓库找到他们的真身。很明显，这几个模块就是对应的 es 特性运行时的实现。

从某种角度上讲，我们要做的事情和 babel 非常像。babel 把基于新版 ECMAScript 规范的代码转换为基于旧 ECMAScript 规范的代码，而 Taro 希望把基于 React 语法的代码转换为小程序的语法。我们从 babel 受到了启发：既然 babel 可以通过运行时框架来实现新特性，那我们也同样可以通过运行时代码，实现上面这些 Web 开发中不存在的功能。

举个例子。对于 view 组件，首先它是个普通的类 React 组件，它把它的子组件如实展示出来…

```js
import Nerv, { Component } from 'nervjs';

class View extends Component {
  render() {
    return (
      <div>{this.props.children}</div>
    );
  }
}...
```

> 接下来，我们需要对 hover-start-time 做处理。与 Taro 其他地方的命名规范一致，我们这个 View 组件接受的属性名将会是驼峰命名法：hoverStartTime。hoverStartTime 参数决定我们将在 View 组件触发 touch 事件多久后改变组件的样式…

```js
// 示例代码
render() {
  const {
    hoverStartTime = 50,
    onTouchStart
  } = this.props;

  const _onTouchStart = e => {
    setTimeout(() => {
      // @TODO 触发touch样式改变
    }, hoverStartTime);
    onTouchStart && onTouchStart(e);
  }
  return (
    <div onTouchStart={_onTouchStart}>
      {this.props.children}
    </div>
  );
}...
```

再稍加修饰，我们就能得到一个功能完整的 Web 版 View 组件

view 可以说是小程序最简单的组件之一了。text 的实现甚至比上面的代码还要简单得多。但这并不说明组件的实现之路上就没有障碍。复杂如 swiper，scroll-view，tabbar，我们需要花费大量的精力分析小程序原生组件的 API，交互行为，极端值处理，接受的属性等等，再通过 Web 技术实现。…

### 8.2 API 适配

> 除了组件，小程序下有一些 API 也是 Web 开发中所不具备的。比如小程序框架内置的 wx.request/wx.getStorage 等 API；但在 Web 开发中，我们使用的是 fetch/localStorage 等内置的函数或者对象

![](https://m.360buyimg.com/img/jfs/t1/106825/8/7473/124175/5dfb4985E4d2133de/82548ed913b69f37.png)

小程序的 API 实现是个巨大的黑盒，我们仅仅知道如何使用它，使用它会得到什么结果，但对它内部的实现一无所知。

如何让 Web 端也能使用小程序框架中提供的这些功能？既然已经知道这个黑盒的入参出参情况，那我们自己打造一个黑盒就好了。

换句话说，我们依然通过运行时框架来实现这些 Web 端不存在的能力。

具体说来，我们同样需要分析小程序原生 API，最后通过 Web 技术实现。有兴趣可以在 Git 仓库中看到这些原生 API 的实现。下面以 wx.setStorage 为例进行简单解析。

wx.setStorage 是一个异步接口，可以把 key: value 数据存储在本地缓存。很容易联想到，在 Web 开发中也有类似的数据存储概念，这就是 localStorage。到这里，我们的目标已经十分明确：我们需要借助于 localStorage，实现一个与 wx.setStorage 相同的 API。…

> 而在 Web 中，如果我们需要往本地存储写入数据，使用的 API 是 localStorage.setItem(key, value)。我们很容易就可以构思出这个函数的雏形

```js
/* 示例代码 */
function setStorage({ key, value }) {
  localStorage.setItem(key, value);
}
```

我们顺手做点优化，把基于异步回调的 API 都给做了一层 Promise 包装，这可以让代码的流程处理更加方便。所以这段代码看起来会像下面这样：

```js
/* 示例代码 */
function setStorage({ key, value }) {
  localStorage.setItem(key, value);
  return Promise.resolve({ errMsg: 'setStorage:ok' });
}...
```

看起来很完美，但开发的道路不会如此平坦。我们还需要处理其余的入参：success、fail 和 complete。success 回调会在操作成功完成时调用，fail 会在操作失败的时候执行，complete 则无论如何都会执行。setStorage 函数只会在 key 值是 String 类型时有正确的行为，所以我们为这个函数添加了一个简单的类型判断，并在异常情况下执行 fail 回调。经过这轮变动，这段代码看起来会像下面这样…

```js
/* 示例代码 */
function setStorage({ key, value, success, fail, complete }) {
  let res = { errMsg: 'setStorage:ok' }
  if (typeof key === 'string') {
    localStorage.setItem(key, value);
    success && success(res);
  } else {
    fail && fail(res);
    return Promise.reject(res);
  }
  complete && complete(res);
  return Promise.resolve({ errMsg: 'setStorage:ok' });
}...
```

把这个 API 实现挂载到 Taro 模块之后，我们就可以通过 Taro.setStorage 来调用这个 API 了。

当然，也有一些 API 是 Web 端无论如何无法实现的，比如 wx.login，又或者 wx.scanCode。我们维护了一个 API 实现情况的列表，在实际的多端项目开发中应该尽可能避免使用它们…

### 8.3 路由

> 作为小程序的一大能力，小程序框架中以栈的形式维护了当前所有的页面，由框架统一管理。用户只需要调用 wx.navigateTo,wx.navigateBack,wx.redirectTo 等官方 API，就可以实现页面的跳转、回退、重定向，而不需要关心页面栈的细节。但是作为多端项目，当我们…

小程序的路由比较轻量。使用时，我们先通过 app.json 为小程序配置页面列表:

```js
{
  "pages": [
    "pages/index/index",
    "pages/logs/logs"
  ],
  // ...
}
```

> 在运行时，小程序内维护了一个页面栈，始终展示栈顶的页面（Page 对象）。当用户进行跳转、后退等操作时，相应的会使页面栈进行入栈、出栈等操作
> 同时，在页面栈发生路由变化时，还会触发相应页面的生命周期

对于 Web 端单页应用路由，我们则以 react-router 为例进行说明

- 首先，react-router 开始通过 history 工具监听页面路径的变化。
- 在页面路径发生变化时，react-router 会根据新的 location 对象，触发 UI 层的更新。
- 至于 UI 层如何更新，则是取决于我们在 Route 组件中对页面路径和组件的绑定，甚至可以实现嵌套路由。
- 可以说，react-router 的路由方案是组件级别的。
- 具体到 Taro，为了保持跟小程序的行为一致，我们不需要细致到组件级别的路由方案，但需要为每次路由保存完整的页面栈。
- 实现形式上，我们参考 react-router：监听页面路径变化，再触发 UI 更新。这是 React 的精髓之一，单向数据流…

![](https://m.360buyimg.com/img/jfs/t1/85653/15/7454/12059/5dfb4a06Ed6a96cb0/9f223d0aecac549c.png)

> @tarojs/router 包中包含了一个轻量的 history 实现。history 中维护了一个栈，用来记录页面历史的变化。对历史记录的监听，依赖两个事件：hashchange 和 popstate。

```js
/* 示例代码 */
window.addEventListener("hashchange", () => {});
window.addEventListener("popstate", () => {});
```

- 对于使用 Hash 模式的页面路由，每次页面跳转都会依次触发 popstate 和 hashchange 事件。由于在 popstate 的回调中可以取到当前页面的 state，我们选择它作为主要跳转逻辑的容器。
- 作为 UI 层，@tarojs/router 包提供了一个 Router 组件，维护页面栈。与小程序类似，用户不需要手动调用 Router 组件，而是由 Taro 自动处理。
- 对于历史栈来说，无非就是三种操作：push, pop，还有 replace。在历史栈变动时触发 Router 的回调，就可以让 Router 也同步变化。这就是 Taro 中路由的基本原理…

### 8.4 Redux 处理

- 每当提到 React 的数据流，我们就不得不提到 Redux。通过合并 Reducer，Redux 可以让大型应用中的数据流更加规则、可预测。
- 我们在 Taro 中加入了 Redux 的支持，通过导入@tarojs/redux，即可在小程序端使用 Redux 的功能。
- 对于 Web 端，我们尝试直接使用 nerv-redux 包提供支持，但这会带来一些问题…

```js
import Nerv from 'nervjs'
import { connect } from 'nerv-redux'

@connect(() => {})
class Index extends Nerv.Componnet {
  componentDidShow() { console.log('didShow') }
  componentDidMount() { console.log('didMount') }
  render() { return '' }
}...
```

- 回想一下前面讲的 componentDidShow 的实现：我们继承，并且改写 componentDidMount。
- 但是对于使用 Redux 的页面来说，我们继承的类，是经过@connect 修饰过的一个高阶组件。
- 问题就出在这里：这个高阶组件的签名里并没有 componentDidShow 这一个函数。所以我们的 componentDidMount 内，理所当然是取不到 componentDidShow 的。
- 为了解决这个问题，我们对 react-redux 代码进行了一些小改装，这就是@taro/redux-h5 的由来…

## 九、更多参考

Taro 官方文档 https://taro.aotu.io/home/in.html

## 十、官方文档

Taro 小册子 https://git.jd.com/huangli47/taro-ebook/tree/master/ebook
