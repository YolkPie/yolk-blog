---
title: Fetch 和 Ajax 有什么区别
date: 2021-09-27 11:30:23
tags:
- Fetch、Ajax、XMLHttpRequest、Axios
categories: javascript
author: zjk537
keywords: Fetch、Ajax、XMLHttpRequest、Axios
description: Fetch 和 Ajax 有什么区别
cover: 
top_img: 
---
## 概念和特性
首先，我们来了解一下 Ajax、Axios 和 Fetch 它们各自的概念。
### Ajax
英文全称为 `Asynchronous JavaScript + XML` ，翻译过来就是异步`JavaScript`和`XML`。
它是用来描述一种使用现有技术集合的“新”方法的，这里的“新”方法主要涉及到:  `HTML` 或 `XHTML`、`CSS`、 `JavaScript`、DOM、XML、XSLT，以及最重要的 XMLHttpRequest。
当使用结合了这些技术的 AJAX 模型以后， 网页应用能够快速地将增量更新呈现在用户界面上，而不需要重载（刷新）整个页面。这使得程序能够更快地回应用户的操作。
> Ajax 是一个概念模型，是一个囊括了众多现有技术的集合，并不具体代指某项技术。

Ajax 最重要的特性就是可以**局部刷新页面**。

### Axios
Axios 是一个基于 Promise 网络请求库，作用于 Node.js 和浏览器中。 它是 isomorphic 的(即同一套代码可以运行在浏览器和 Node.js中)。在服务端它使用原生 Node.js http 模块，而在客户端则使用 XMLHttpRequest。

这里我们只关注客户端的 Axios，它是基于 XHR 进行二次封装形成的工具库。
客户端 Axios 的主要特性有：
> 1、从浏览器创建 XMLHttpRequests
2、支持 Promise API
3、拦截请求和响应
4、转换请求和响应数据
5、取消请求
6、自动转换JSON数据
7、客户端支持防御XSRF

### Fetch
Fetch 提供了一个获取资源的接口（包括跨域请求）。
`Fetch` 是一个现代的概念, 等同于 XMLHttpRequest。它提供了许多与 XMLHttpRequest 相同的功能，但被设计成更具可扩展性和高效性。
Fetch 的核心在于对 HTTP 接口的抽象，包括 Request、Response、Headers 和 Body，以及用于初始化异步请求的 global fetch。得益于 JavaScript 实现的这些抽象好的 HTTP 模块，其他接口能够很方便的使用这些功能。
除此之外，Fetch 还利用到了请求的异步特性——它是基于 Promise 的。
fetch()  方法必须接受一个参数——资源的路径。无论请求成功与否，它都返回一个 Promise 对象，resolve 对应请求的 Response。

## Fetch 和 Axios/Ajax 的关系
直接上图，一览无余
![Fetch&Axios&Ajax关系图](Fetch&Axios&Ajax关系图.jpg)
稍做解释：
- Ajax 是一种代表异步 JavaScript + XML 的模型（技术合集），所以 Fetch 也是 Ajax 的一个子集
- 在之前，我们常说的 Ajax 默认是指以 XHR 为核心的技术合集，而在有了 Fetch 之后，Ajax 不再单单指 XHR 了，我们将以 XHR 为核心的 Ajax 技术称作传统 Ajax。
- Axios 属于传统 Ajax（XHR）的子集，因为它是基于 XHR 进行的封装。

## Fetch 真的会取代 Ajax 吗？
其实这个问题更准确的问法应该是：Fetch 真的会取代传统 Ajax ( XHR ) 吗？
要回答这个问题，我们需要清楚以下几点：

- 异步编程是 JavaScript 发展的大趋势，且绝大多数浏览器都已支持标准 Promise。
- Fetch API 是浏览器自带的 API，且它是基于标准 Promise 的。
- 传统 Ajax 原生写法结构比较混乱，不符合关注分离的原则，写过远程 XHR 的同学应该深有体会。
- Axios 是基于 XHR 封装的 Promise 请求库，用起来确实方便。

虽然目前来看，传统 Ajax （比如 Axios 之类的）在使用规模上远远超过 Fetch，但要知道，这是 XHR 十来年累积下来的效果。
封装得到的 Axios 在易用性上甩了原生 XHR 十万八千里，但毕竟是封装的，和原生的 Fetch 相比较，Axios 在出身上就已略输一筹，且原生的 API 天然上会支持更多的功能，使用上会更加灵活。

## Fetch 工具库推荐
其他社区上推荐的Fetch 工具库，名为 [Mande](https://github.com/posva/mande)

参考文档
FetchAPI: https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch
https://github.com/posva/mande