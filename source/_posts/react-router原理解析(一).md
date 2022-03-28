---
title: react-router原理解析 (一)
date: 2022-03-28 16:01:13
tags: react react-router
categories: react
---

## 概要

react-router 作为react项目最基本的的导航框架,  基本配置并不复杂, 但是它也提供了非常灵活的api, 和丰富的组件, 本文将从源码角度深入介绍react-router提供的导航组件, 以及其设计原理

react-router 对外提供3个包

```json-doc
react-router  // react-router 导航的核心包
react-router-dom // 对react-router进行了封装, 是react-router的拓展, 提供一些react组件
react-router-native // 同样是对react-router的封装, 区别是它是适用于react-native的库
``` 


对于react项目, 我们通常不会直接依赖react-router, 因为react-router-dom拥有其所有的功能, 并额外提供了BrowserRouter, HashRouter, Link等多个react组件 

```shell
yarn add react-router-dom  
```

## Router

Router是react-router管理路由的核心组件,  是导航命令传递者，若没有引入Router，那么任何跳转都不会生效。  需要路由导航的页面都需要放在Router内被Router管理, Router的源码并不复杂,  因为其push, replace, back等导航功能的实现, 依赖另一个库 <code>history</code> 来实现, history库在React Router中扮演着导航执行者与监听者的重要角色。对于React Router，所有的“副作用”都由history库完成,   而Router源码中, 并没有直接持有history, history 需要从props从传入, 下面是Router的核心代码:

```javascript
// 监听history的变化
props.history.listen(location => {    
  if (this._isMounted) {   
      this.setState({ location });   
  } else {   
      this._pendingLocation = location;   
  }   
});   
```


```jsx
<RouterContext.Provider >
  <HistoryContext.Provider
    children={this.props.children || null}
    value={this.props.history}
  />
</RouterContext.Provider>
```


可以看到 Router的工作就是监听history的变化, 通过创建的RouterContext. 将history的状态透传给子组件, 这也解释了为什么Router必须放在所有导航的组件的最外层, 因为只有这样子组件才能接受到context的变化.

通常情况下,  我么并不会直接使用Router组件, react-router-dom对Router进行了封装, 并提供了, BrowserHistory, HashHistory等组件,  其根据接收外部history对象的不同能提供不同的功能：如果接收browserHistory，则得到BrowserRouter，称为浏览器路由；如果接收hashHistory，则得到HashRouter，称为哈希路由；如果接收memoryHistory，则得到MemoryRouter，称为内存路由.



我们常用的BrowserRouter 就是 对 router 的封装, 不需要传入history 对象, 默认使用了  history 的 createBrrowserHistory  对象, BrowserRouter支持命名式路由跳转, 传递state等功能, 如果如果没有特殊的需求, 项目一般使用BrowserRouter, 作为路由管理对象

> 需要注意的是, BrowserRouter需要服务端提供一些配置支持, 因为在用户强制刷新的场景, 如果当前路径为根路径, 如果仅给用户提供单纯的CDN静态文件，那么考虑当使用BrowserRouter后如导航到/foo/baz路径，这时用户强制刷新了页面，如果仅有单纯CDN静态文件的支持，由于找不到/foo/baz路径下的资源，页面就会返回“404无法找到”或者视具体情况产生其他错误.  解决这个问题的方式是在nginx的配置中, location字段添加 try_files $uri /index.html 这个配置




其他常用router有HashRouter, StaticRouter等, 由于HashRouter不支持state持久化存储，其目的是支持在旧式浏览器上运行路由

StaticRouter一般称为静态路由。StaticRouter与其他类型Router的最大区别在于其不改变路径地址，且不记录历史栈，为无状态路由，大多在服务端渲染场景中使用, 其history并不由history第三方库提供，而是直接内化在其源码实现中。对于StaticRouter，可从react-router包中引入，无须传入history：主要用户服务端渲染

## Route

Route组件的职责为接收路径信息并执行渲染。Route也称为路由端口，用于接收Router的命令,  当Route从Router接收到的location匹配的path参数时, Route就会渲染component参数中的组件, 同时Route也提供了render, children方式用于自定义渲染组件



```jsx
<RouterContext.Consumer>
  {context => {
    invariant(context, "You should not use <Route> outside a <Router>");
    const location = this.props.location || context.location;
    const match = this.props.computedMatch
      ? this.props.computedMatch 
      : this.props.path
      ? matchPath(location.pathname, this.props)
      : context.match;

    const props = { ...context, location, match };

    let { children, component, render } = this.props;
    if (Array.isArray(children) && children.length === 0) {
      children = null;
    }

    return (
      <RouterContext.Provider value={props}>
        {props.match
          ? children
            ? typeof children === "function"
              ? __DEV__
                ? evalChildrenDev(children, props, this.props.path)
                : children(props)
              : children
            : component
            ? React.createElement(component, props)
            : render
            ? render(props)
            : null
          : typeof children === "function"
          ? __DEV__
            ? evalChildrenDev(children, props, this.props.path)
            : children(props)
          : null}
      </RouterContext.Provider>
    );
  }}
</RouterContext.Consumer>
```


默认情况下, Route的匹配是模糊匹配  /cart  既会匹配 '/' 也会匹配 /cart  , 所以 / 路径会同时渲染两个页面, 解决的方式是使用exact参数, 只要加上 exact. 表示会精准匹配 /       

还有一种方式是将/放在最下面, 并包裹Switch组件, 因为switch是从上到下匹配的, /cart匹配到了/cart 下面的 /就不会被匹配到了, 后面会详细介绍Switch的原理


Route允许自行传入location进行匹配，而不是使用上下文中的location。传入的location的pathname可以与当前的pathname不相同，这可在某些场景中发挥作用，如路由动画等. 



## Switch

Switch拥有挑选Route的能力，会挑选并渲染第一个匹配路由路径的Route。当某Route匹配命中时，其余未匹配命中或者即便匹配路径的Route，都会返回null，Switch只渲染第一个匹配命中的Route

Switch的核心代码如下, 利用React.children.forEach遍历其子组件(也就是Route) , 然后, 匹配Route的path属性, 渲染出第一个匹配到的Route

```jsx
<RouterContext.Consumer>
  {context => {
    invariant(context, "You should not use <Switch> outside a <Router>");

    const location = this.props.location || context.location;

    let element, match;
    React.Children.forEach(this.props.children, child => {
      if (match == null && React.isValidElement(child)) {
        element = child;

        const path = child.props.path || child.props.from;

        match = path
          ? matchPath(location.pathname, { ...child.props, path })
          : context.match;
      }
    });

    return match
      ? React.cloneElement(element, { location, computedMatch: match })
      : null;
  }}
</RouterContext.Consumer>
```


为什么源码中不使用React的React.Children.toArray方法转换children，而直接使用forEach?这里要考虑同一个组件渲染在不同URL中的情况，如下所示：

```jsx
<Switch>
  <Route path="/a" component={A} />  
  <Route path="/b" component={A} />
</Switch>
```


在路径/a、/b同时渲染同一个A组件的情况下，若当前的路径为/a，并从该路径导航到/b路径，原先/a路径命中并渲染过A组件，且导航到/b路径也同样渲染A组件。由于对Switch的子组件来说，将同样渲染Route，Route也没有key的变化，Route的渲染也没有发生变化（都渲染A组件），因此A组件并不会触发componentWillUnmount，而是会进入A组件更新的生命周期。如果源码使用React.Children.toArray方法，由于该方法会为组件增加key标志，所以这时Route会因为key的不同，使旧key对应的Route被销毁，新key对应的Route被挂载。这样的销毁和挂载过程会导致同一个A组件也被销毁与重新挂载。

如果希望每次命中路由都能销毁旧组件，并重新渲染进而执行componentDidMount生命周期方法，则可以为渲染相同组件的各Route加入唯一的key值，如下所示：

```jsx
// 当导航时，A组件会先销毁，再重新渲染并执行componentDidMount生命周期方法
<Switch>  
  <Route path="/a" key="a" component={A} />  
  <Route path="/b" key="b" component={A} />
 </Switch>
```


这时，由于key值不同，当导航从/a到/b时，key值为b对应的Route将得到渲染，但是由于原先的Route的key值为a，key值不一致，所以按照React的diff机制，key值为a对应的Route将会被销毁，key值为b对应的Route将会被挂载。对应的A组件也会执行相同的操作，即A组件会被销毁，并重新渲染，会执行componentDidMount生命周期方法。

## WithRouter

withRouter是React Router提供的高阶组件。在一些处于很深层级的组件中，如果希望获得props.history、props.location等对象，又不希望从上层逐级传入，则可使用withRouter高阶组件注入相关RouteComponentProps属性

```jsx
<RouterContext.Consumer>
  {context => {
    invariant(
      context,
      `You should not use <${displayName} /> outside a <Router>`
    );
    return (
      <Component
        {...remainingProps}
        {...context}
        ref={wrappedComponentRef}
      />
    );
  }}
</RouterContext.Consumer>
```


从源码中可以看到, withRouter这个高阶组件, 所做的就是给子组件产地一个Router的Context, 是其获得路由状态和导航能力



## Hooks API

react-router提供了一些hook方法用于获取路由参数和history对象, 他们的原理也十分简单, 仅仅是对context做了一层封装

```js
export function useHistory() {
  return useContext(HistoryContext);
}
export function useLocation() {
  return useContext(Context).location;
}

export function useParams() {
  const match = useContext(Context).match;
  return match ? match.params : {};
}

export function useRouteMatch(path) {
  const location = useLocation();
  const match = useContext(Context).match;

  return path ? matchPath(location.pathname, path) : match;
}

```


由于location, match, history等对象, 在RouterContext中, 这些hook所做的仅仅就是返回对应的属性


