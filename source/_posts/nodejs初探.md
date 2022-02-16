---
title: nodejs初探
date: 2022-02-09 18:08:07
---
# node

> nodejs就是基于js语法增加与操作系统之间的交互



**node版本管理**

- n、nvm安装 node



> ```js
> curl -o-
> https://raw.githubusercontent.com/nvm-
> sh/nvm/v0.35.3/install.sh | bash
> ```



**nodejs工作原理**
![](./assets/nodejs.jpg)

- application 编写的程序
- V8  引擎 解析二进制代码
- 调用nodejs
- 调用libuv事件处理库

> 循环等待事件处理，有事件进来后先插入至事件队列



**nodejs事件处理**
![](./assets/nodejs事件处理.jpg)

- 收到请求
- 生成事件插入到事件队列
- libvu通过事件循环，从事件队列中取出事件处理
  - 最终结果-直接执行callback
  - 从线程池取出线程做响应处理后回收线程
  - 中间结果-重新插入事件队列





**两个v8引擎**


![](./assets/两个v8.jpg)



- 请求之前，nodejs通过v8引擎运行服务
- 客户端发送请求
- 客户端收到服务端返回的js
- 客户端的v8进行解析



**最简单的http服务**

1. require 引入 http模块
2. 创建http服务
3. 侦听端口

```javascript
'use strict'

const http = require('http');

const host = 'localhost'
const port = 3333
const app = http.createServer((req, res) => {
    console.log(req)
    res.writeHead(200, {
        'Content-Type': 'text/plain'
    })
    res.end('hello world')
}).listen(port, host, () => {
    console.log(`http://${host}:${port}`)
});
```

启动方式

- node app.js
- nohub node app.js &
- forever start app.js

> forever stop app.js



**创建https服务**

- 生成Https证书

```
mkcert -install

mkcert localhost 127.0.0.1 ::1
```



- 引入Https模块
- 指定证书位置，并创建Https服务



你访问的不是私密链接解决办法

> 控制台输入 sendCommand(SecurityInterstitialCommandId.CMD_PROCEED)



```javascript
'use strict'

const https = require('https');
const fs = require('fs');

const host = 'localhost'
const port = 8080
const options = {
    key: fs.readFileSync('./ca/local.shanyue.tech-key.pem'),
    cert: fs.readFileSync('./ca/local.shanyue.tech.pem'),
}

const app = https.createServer(options, (req, res) => {
    res.writeHead(200, {
        'Content-Type': 'text/plain'
    })
    res.end('hello world')
}).listen(port, host, () => {
    console.log(`https://${host}:${port}`)
})

```



**真正的Web服务**

- 引入 express 模块
- 引入serve-index模块
- 指定发布目录

```javascript
'use strict'

const http = require('http');
const https = require('https');

const fs = require('fs');

const express = require('express');
const serveIndex = require('serve-index');

const app = express()
app.use(serveIndex('./public')) // 浏览目录
app.use(express.static('./public')) // 指定发布路径

const options = {
    key: fs.readFileSync('./ca/local.shanyue.tech-key.pem'),
    cert: fs.readFileSync('./ca/local.shanyue.tech.pem'),
}
const httpServe = http.createServer(app);
const httpsServe = https.createServer(options, app);

httpServe.listen(80, '0.0.0.0', () => {
    console.log('http://0.0.0.0')
})
httpsServe.listen(8080, '0.0.0.0', () => {
    console.log('https://0.0.0.0:8080')
})
```



**node.js的底层依赖**

- v8引擎：主要是js语法的解析，识别js语法
- libuv：C语言实现的一个高性能异步非阻塞IO库，实现node.js的事件循环
- http-parse/||http: 底层处理http请求，处理报文，解析请求包等内容
- openssl：处理加密算法
- zlib: 处理压缩等内容



**node.js常见内置模块**

> node.js 中最主要的内容，就是实现了一套 CommonJS 的模块化规范，以及内置了一些常⻅的模块。

- fs: 文件系统，能够读取写入当前安装系统环境中硬 盘的数据
- path: 路径系统，能够处理路径之间的问题
- crypto: 加密相关模块，能够以标准的加密方式对我 们的内容进行加解密
- dns: 处理 dns 相关内容，例如我们可以设置 dns 服 务器等等
- http: 设置一个 http 服务器，发送 http 请求，监听 响应等等
- readline: 读取 stdin 的一行内容，可以读取、增加、 删除我们命令行中的内容
- os: 操作系统层面的一些 api，例如告诉你当前系统类 型及一些参数
- vm: 一个专⻔处理沙箱的虚拟机模块，底层主要来调 用 v8 相关 api 进行代码解析。



**实现简易commonjs**

1. 读取文件并解析执行 vm.Script 作用是把字符串变为可执行代码

```javascript
const vm = require('vm')
const path = require('path')
const fs = require('fs')

const pathToFile = path.resolve(__dirname, './index1.js');
const content = fs.readFileSync(pathToFile, 'utf-8')


const script = new vm.Script(content, {
    filename: 'index1.js'
})

const result = script.runInThisContext()

```

2. 注入require函数 

```javascript
const vm = require('vm')
const path = require('path')
const fs = require('fs')

const pathToFile = path.resolve(__dirname, './index2.js');
const content = fs.readFileSync(pathToFile, 'utf-8')

const wrapper = [
    '(function(require, module, exports){',
    '})'
]

const wrapperContent = wrapper[0] + content + wrapper[1]

const script = new vm.Script(wrapperContent, {
    filename: 'index1.js'
})

const result = script.runInThisContext()
console.log(result)

```



3. 注入 module exports 并挂载

```javascript
const vm = require('vm')
const path = require('path')
const fs = require('fs')

/* const pathToFile = path.resolve(__dirname, './index2.js');
const content = fs.readFileSync(pathToFile, 'utf-8')

const wrapper = [
    '(function(require, module, exports){',
    '})'
]

const wrapperContent = wrapper[0] + content + wrapper[1]

const script = new vm.Script(wrapperContent, {
    filename: 'index1.js'
})

const result = script.runInThisContext()
console.log(result) */

function r(filename) {
    const pathToFile = path.resolve(__dirname, filename);
    const content = fs.readFileSync(pathToFile, 'utf-8')

    const wrapper = [
        '(function(require, module, exports){',
        '})'
    ]

    const wrapperContent = wrapper[0] + content + wrapper[1]

    const script = new vm.Script(wrapperContent, {
        filename: 'index1.js'
    })

    const result = script.runInThisContext()
    const module = {
        exports: {

        }
    }
    result(r, module, module.exports)
    return module.exports
}

global.r = r
```



**全局对象**

- process

> process 是一个全局变量，即 global 对象的属性。它用于描述当前Node.js 进程状态的对象，提供了一个与操作系统的简单接口。



比较常用的：

process.env  环境变量

process.argv 获取命令行输入参数  实现cli的commander底层就是使用的这个



1. exit

当进程准备退出时触发。

2. beforeExit

当 node 清空事件循环，并且没有其他安排时触发这个事件。通常来说，当没有进程安排时 node 退出，但是 ‘beforeExit’ 的监听器可以异步调用，这样 node 就会继续执行。

3. uncaughtException

当一个异常冒泡回到事件循环，触发这个事件。如果给异常添加了监视器，默认的操作（打印堆栈跟踪信息并退出）就不会发生。

4. Signal 事件

当进程接收到信号时就触发。信号列表详见标准的 POSIX 信号名，如 SIGINT、SIGUSR1 等。



```
process.on('exit', function(code) {
  setTimeout(function() {
    console.log("setTimeout");
  }, 0);

  console.log('退出码为:', code);

});

console.log("程序执行结束");
```



```
/ 输出到终端

process.stdout.write("Hello World!" + "\n");


// 通过参数读取

process.argv.forEach(function(val, index, array) {
   console.log(index + ': ' + val);
});


// 获取执行路局
console.log(process.execPath);


// 平台信息
console.log(process.platform);

```



**看看这段代码输出什么**

```javascript
console.log(this);    
module.exports.foo = 5;
console.log(this);   
```







**事件循环**



```

async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2')
}
console.log('script start')
setTimeout(function () {
    console.log('setTimeout0')
    setTimeout(function () {
        console.log('setTimeout1');
    }, 0);
    setImmediate(() => console.log('setImmediate'));
}, 0)

process.nextTick(() => console.log('nextTick'));
async1();
new Promise(function (resolve) {
    console.log('promise1')
    resolve();
    console.log('promise2')
}).then(function () {
    console.log('promise3')
})
console.log('script end')

```











在 Node 领域，微任务是来自以下对象的回调：

1. process.nextTick()

2. then()



> 优先级 process.nextTick > promise.then





**setImmediate 和 setTimeout 的区别**

setImmediate 和 setTimeout 相似，但是根据调用时间的不同，它们的行为也不同。

- setImmediate 设计为在当前轮询 poll 阶段完成后执行脚本。

- setTimeout 计划在以毫秒为单位的最小阈值过去之后运行脚本。



Tips: 计时器的执行顺序将根据调用它们的上下文而有所不同。 如果两者都是主模块中调用的，则时序将受到进程性能的限制.

来看两个例子：



在主模块中执行

```javascript
  setTimeout(() => {
    console.log('timeout');
  }, 0);


  setImmediate(() => {
   console.log('immediate');
  });
```

>  两者的执行顺序是不固定的, 可能timeout在前, 也可能immediate在前



 在同一个I/O回调里执行

```
const fs = require('fs');
fs.readFile(__filename, () => {
    setTimeout(() => {
       console.log('timeout');
    }, 0);

    setImmediate(() => {
        console.log('immediate');
    });
});
```



> 在同一个I/O回调里执行 setImmediate总是先执行





**nodejs周边生态**

- 另一种解析引擎 quickjs




