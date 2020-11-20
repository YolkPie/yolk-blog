---
title: WebSocket(一)
date: 2020-11-20 20:00:09
cover: "https://m.360buyimg.com/img/jfs/t1/150391/7/7970/9687/5fb7ae0dEe8c94509/0363e802babbf257.png"
---


## WebSocket(-)

### 简介
WebSocket protocol是HTML5一种新的协议。它实现了浏览器与服务器全双工通信(full-duplex)。一开始的握手需要借助HTTP请求完成。

### 原理&机制
#### 原理
网站上的即时通讯是很常见的，比如网页的QQ，微信等。按照以往的技术能力通常是采用轮询等技术解决。
HTTP协议是非持久化的，单向的网络协议，在建立连接后只允许浏览器向服务器发出请求后，服务器才能返回相应的数据。
当需要即时通讯时，通过轮询在特定的时间间隔（如1秒），由浏览器向服务器发送Request请求，然后将最新的数据返回给浏览器。
缺点：会导致过多不必要的请求，浪费流量和服务器资源，每一次请求、应答，都浪费了一定流量在相同的头部信息上，而在WebSocket中，只需要服务器和浏览器通过HTTP协议进行一个握手的动作，然后单独建立一条TCP的通信通道进行数据的传送。
WebSocket同HTTP一样也是应用层的协议，但是它是一种双向通信协议，是建立在TCP之上的。
广泛被用来做即时通讯，以替代轮询。

#### 机制
WebSocket是HTML5的新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯，它建立在TCP之上，同HTTP一样通过 TCP 来传输数据，但是它和 HTTP 最大不同在于：

- WebSocket 是一种双向通信协议，在建立连接后，WebSocket 服务器和 
Browser/Client Agent 都能主动的向对方发送或接收数据，就像 Socket 一样。
- WebSocket 需要类似TCP的客户端和服务器端通过握手连接，连接成功后
才能相互通信。

WebSocket是类似Socket的TCP长连接的通讯模式，一旦 WebSocket 连接建立后，后续数据都以帧序列的形式传输。
在客户端断开 WebSocket 连接或 Server 端断掉连接前，不需要客户端和服务端重新发起连接请求。
在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。

### 连接过程&特点
#### 连接过程
- 浏览器、服务器建立TCP连接，三次握手。这是通信的基础，传输控制层，若失败后
续都不执行。
- TCP连接成功后，浏览器通过HTTP协议向服务器传送WebSocket支持的版本号等信息。
（开始前的HTTP握手）
- 服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据。
- 当收到了连接成功的消息后，通过TCP通道进行传输通信。
#### 特点
- 建立在 TCP 协议之上，服务器端的实现比较容易。
- 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 
协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送文本，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL

### 主要Api
#### 构造函数
WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。
```
const ws = new WebSocket('ws://localhost:80');
```

#### webSocket.readyState
readyState属性返回实例对象的当前状态，例如
- CONNECTING：值为0，表示正在连接。
- OPEN：值为1，表示连接成功，可以通信了。
- CLOSING：值为2，表示连接正在关闭。
- CLOSED：值为3，表示连接已经关闭，或者打开连接失败。

```
switch (ws.readyState) {
  case WebSocket.CONNECTING:
    // do something
    break;
  case WebSocket.OPEN:
    // do something
    break;
  case WebSocket.CLOSING:
    // do something
    break;
  case WebSocket.CLOSED:
    // do something
    break;
  default:
    // this never happens
    break;
}
```

#### webSocket.onopen
实例对象的onopen属性，用于指定连接成功后的回调函数。

```
ws.onopen = function () {
  ws.send('Hello WebSocket!');
}
```
#### webSocket.onclose
实例对象的onclose属性，用于指定连接关闭后的回调函数。
```
ws.onclose = function(event) {
  const {code, reason, wasClean} = event;
};
```
#### webSocket.onmessage
实例对象的onmessage属性，用于指定收到服务器数据后的回调函数。
```
ws.onmessage = function(event){
  if(typeof event.data === String) {
    //
  }else{
    //
  }
}
```
#### webSocket.send
实例对象的send()方法用于向服务器发送数据。
```
// 发送文本的例子。(也可以发送图片等其他文件对象)
ws.send('your message');
```
#### webSocket.onerror
实例对象的onerror属性，用于指定报错时的回调函数。
```
socket.onerror = function(event) {
  console.log(event)
  //处理错误异常
};
```

#### 示例代码
```
const ws = new WebSocket("wss://localhost:80");  
ws.onopen = function(evt) {   
  console.log("Connection open ...");   
  ws.send("Hello WebSockets!"); 
};  
ws.onmessage = function(evt) {  
  console.log( "Received Message: " + evt.data);
  //Received Message: Hello WebSockets!
  ws.close(); 
};  
ws.onclose = function(evt) {  
  console.log("Connection closed."); 
}; 
```

### WebSocket与HTTP的不同 （图解）
![](https://m.360buyimg.com/img/jfs/t1/142749/34/13761/207298/5fa93d0dE977b31a4/b2458a2ffda633bb.jpg)


