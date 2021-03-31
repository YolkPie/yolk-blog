---
title: SameSiteForCookie
date: 2021-02-03 15:20:23
tags:
- cookie

author: YYY
---
# Cookie、Session、Token到底是什么

## 背景
1. 通天塔页面分享至京喜小程序中交互丢失
2. 定位问题为iOS14以上，跨站请求默认不再携带cookie
3. 2月24号 正式宣布将在 iOS、iPad OS 13.4 和 macOS 上的 Safari 13.1 里默认完全屏蔽第三方 Cookie。
4. 随后chrom80版本也做了同样操作


[Full Third-Party Cookie Blocking and More](https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/)

### HTTP是无状态的Web服务器
> 什么是无状态呢？一次对话完成后下一次对话完全不知道上一次对话发生了什么。


```
洛：大爷，楼上322住的是马冬梅家吧？
大爷：马都什么？
夏洛：马冬梅。
大爷：什么都没啊？
夏洛：马冬梅啊。
大爷：马什么没？
夏洛：行，大爷你先凉快着吧。
```

### 让无状态的服务器记住一些事情
> 当然Web服务器是记不住东西的，我们需要一些外部的办法。

#### cookie的产生过程

1. 浏览器第一次访问服务端时，服务器此时肯定不知道他的身份，所以创建一个独特的身份标识数据，格式为key=value，放入到Set-Cookie字段里，随着响应报文发给浏览器。
2. 浏览器看到有Set-Cookie字段以后就知道这是服务器给的身份标识，于是就保存起来，下次请求时会自动将此key=value值放入到Cookie字段中发给服务端。
3. 服务端收到请求报文后，发现Cookie字段中有值，就能根据此值识别用户的身份然后提供个性化的服务。

![cookie](./image/http.png)

### 搭建一个自己的后台服务器

> npm install express cors body-parser http-server -S

``` javascript
const express = require('express')
const app = express()
const cors = require('cors')
const bodyParser = require('body-parser') // 解析参数
const corsOptions = {
    origin: ['http://10.0.49.168:8081', 'http://localhost:8081'],
    credentials: true,
    optionsSuccessStatus: 200 // some legacy browsers (IE11, various SmartTVs) choke on 204
}
app.use(cors(corsOptions)) // 解决跨域
app.use(bodyParser.json()) // json请求
app.use(bodyParser.urlencoded({
    extended: false
})) // 表单请求
app.listen(8080, () => console.log('Server is running at http://127.0.0.1:8080/'))
```

### 生成cookie
``` javascript
app.get('/', (req, res) => {
    res.cookie('cookieName', 'cookieValue');
    res.send('<p>Hello World</p>')
})
```
1. 访问http://127.0.0.1:8080/ 发现Set-Cookie字段
2. 再次访问，cookie已经被携带进去了
3. 访问http://127.0.0.1:8080/test/ 发现cookie被携带过去了

### cookie的属性
> 我们知道Cookie就是服务器委托浏览器存储在客户端里的一些数据，而这些数据通常都会记录用户的关键识别信息,cookie会有一些属性用来保护，防止外泄或者窃取。
![](./image/cookie.png)

1. Max-Age/Expires  设置cookie的过期时间，单位为秒
2. Domain 指定了Cookie所属的域名
3. Path 指定了Cookie所属的路径
4. HttpOnly 告诉浏览器此Cookie只能靠浏览器Http协议传输,禁止其他方式访问
5. Secure 告诉浏览器此Cookie只能在Https安全协议中传输,如果是Http则禁止传输
6. value 用 JavaScript 操作 Cookie 的时候注意对 Value 进行编码处理。

#### Expires
``` javascript
res.cookie('cookieexpires', 'cookieexpires', {expires: new Date(Date.now() + 100)})
```
1. 当 Expires 属性缺省时，表示是会话性 Cookie,一般值为Session，当为会话性 Cookie 的时候，值保存在客户端内存中，并在用户关闭浏览器时失效。
2. 持久性 Cookies 会保存在用户的硬盘中，直至过期或者清除 Cookie。
3. 设定的日期和时间只与客户端相关，而不是服务端。

#### Max-Age
``` javascript
res.cookie('cookiemaxAge', 'cookiemaxAge', {maxAge: 300})
```
1. Max-Age 可以为正数、负数、甚至是 0
2.  max-Age 属性为正数时，浏览器会将其持久化，即写到对应的 Cookie 文件中
3. max-Age 属性为负数，则表示该 Cookie 只是一个会话性 Cookie
4. max-Age 为 0 时，则会立即删除这个 Cookie
5. Expires 和 Max-Age 都存在，Max-Age 优先级更高


#### Domain
``` javascript
res.cookie('cookieDomain', 'cookieDomain', {domain: '127.0.0.1'})
```
1. 访问http://127.0.0.1:8080/ 发现Set-Cookie字段
2. 再次访问，cookie被携带进去了
3. 访问http://localhost:8080 发现cookie没有被携带过去
4. Domain 指定了 Cookie 可以送达的主机名
5. 假如没有指定，那么默认值为当前文档访问地址中的主机部分
6. 不能跨域设置 Cookie: 比如京东域名下的页面把 Domain 设置成百度是无效的


#### path

``` javascript
res.cookie('cookiePath', 'cookiePath', {path: '/admin'})
```
1. 访问http://127.0.0.1:8080/ 发现Set-Cookie字段
2. 再次访问，cookie没有被携带进去了
3. 访问http://127.0.0.1:8080/admin 发现cookie被携带过去了
4. Path 指定了一个 URL 路径
5. Domain 和 Path 标识共同定义了 Cookie 的作用域：即 Cookie 应该发送给哪些 URL。

#### Secure
1. 标记为 Secure 的 Cookie 只应通过被HTTPS协议加密过的请求发送给服务端

#### HTTPOnly
1. 设置 HTTPOnly 属性可以防止客户端脚本通过 document.cookie 等方式访问 Cookie，有助于避免 XSS 攻击
2. 登录态中的pin就是httponly

#### SameSite

##### 作用
SameSite 属性可以让 Cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）。

##### 属性值
1. Strict 仅允许一方请求携带 Cookie，即浏览器将只发送相同站点请求的 Cookie，即当前网页 URL 与请求目标 URL 完全一致。
2. Lax 允许部分第三方请求携带 Cookie
3. None 无论是否跨站都会发送 Cookie

###### 跨域和跨站
1. 同站(same-site)/跨站(cross-site)
2. 同源(same-origin)/跨域(cross-origin)
3. 同源策略的同源是指两个 URL 的协议/主机名/端口一致 判断是比较严格的
4. Cookie中的「同站」判断就比较宽松：只要两个 URL 的 eTLD+1 相同即可，不需要考虑协议和端口
5. eTLD 表示有效顶级域名
6. eTLD+1 则表示，有效顶级域名+二级域名
7. www.jd.com 和 www.baidu.com 是跨站
8. www.a.jd.com 和 www.b.jd.com 是同站

###### 改变
![](./image/SameSite.png)

[测试链接](https://pro.jingxi.com/mini/active/2xk84tfaKUqLvStvZuExqFh4APPq/index.html?cookie=%7B%22visitkey%22%3A%2224741336555031603799422594%22%2C%22__wga%22%3A%221608022089230.1608022089230.1607414308064.1603799422965.1.37%22%2C%22PPRD_P%22%3A%22EA.17078.1.1-LOGID.1608022089346.405882235-CT.138631.36.4%22%2C%22__jda%22%3A%22122270672.abe16a807e8ba6679efd909f6eb9587a.1608022089155.1608022089155.1608022089155.1%22%2C%22__jdv%22%3A%22122270672%7Cdirect%7Ct_1000578828_xcx_1001_fxrk%7Cxcx%7C-%7C1608022438768%22%2C%22unpl%22%3A%22%22%2C%22wxapp_type%22%3A2%2C%22cd_eid%22%3A%22%22%2C%22pinStatus%22%3A4%2C%22wxapp_openid%22%3A%22o33sZ0a-42w18t9nPxfE40MSeWUE%22%2C%22wxapp_version%22%3A%226.12.100%22%2C%22buildtime%22%3A20201210%7D&wxAppName=jx)


###### 解决

> 解决方案就是设置 SameSite 为 none。

注意点
1. HTTP 接口不支持 SameSite=none
> 如果你想加 SameSite=none 属性，那么该 Cookie 就必须同时加上 Secure 属性，表示只有在 HTTPS 协议下该 Cookie 才会被发送。
2. 需要 UA 检测，部分浏览器不能加 SameSite=none
> IOS 12 的 Safari 以及老版本的一些 Chrome 会把 SameSite=none 识别成 SameSite=Strict，所以服务端必须在下发 Set-Cookie 响应头时进行 User-Agent 检测，对这些浏览器不下发 SameSite=none 属性


### Session

> Cookie是存储在客户端方，Session是存储在服务端方，客户端只存储SessionId

> 既然浏览器已经通过Cookie实现了有状态这一需求，那么为什么又来了一个Session呢？这里我们想象一下，如果将账户的一些信息都存入Cookie中的话，一旦信息被拦截，那么我们所有的账户信息都会丢失掉。所以就出现了Session，在一次会话中将重要信息保存在Session中，浏览器只记录SessionId一个SessionId对应一次会话请求。

![](./image/session.png)


### 创建Session

> npm install --save express-session session-file-store

``` javascript
app.get('/session', (req, res) => {
    const { name, password } = req.query
    req.session.name = name
    req.session.password = password
    res.json(req.session)
})
```
1. 访问http://127.0.0.1:8080/session?name=yyy&password=123456 发现Set-Cookie字段
2. 再次访问，cookie被携带进去了

``` javascript
app.get('/getsession', (req, res) => {
    res.json(req.session.name)
})
```
1. 访问http://127.0.0.1:8080/getsession 返回我们之前存储的name



### Token
> Session是将要验证的信息存储在服务端，并以SessionId和数据进行对应，SessionId由客户端存储，在请求时将SessionId也带过去，因此实现了状态的对应。而Token是在服务端将用户信息经过Base64Url编码过后传给在客户端，每次用户请求的时候都会带上这一段信息，因此服务端拿到此信息进行解密后就知道此用户是谁了，这个方法叫做JWT(Json Web Token)。

![](./image/token.png)


> Token相比较于Session的优点在于，当后端系统有多台时，由于是客户端访问时直接带着数据，因此无需做共享数据的操作。


#### Token的优点
1. 简洁：可以通过URL,POST参数或者是在HTTP头参数发送，因为数据量小，传输速度也很快
2. 自包含：由于串包含了用户所需要的信息，避免了多次查询数据库
3. 因为Token是以Json的形式保存在客户端的，所以JWT是跨语言的
4. 不需要在服务端保存会话信息，特别适用于分布式微服务


#### 实现token
> npm install express-access-token cookie-parser -S

``` javascript
const accessTokens = [
    "6d7f3f6e-269c-4e1b-abf8-9a0add479511",
    "110546ae-627f-48d4-9cf8-fd8850e0ac7f",
    "04b90260-3cb3-4553-a1c1-ecca1f83a381"
];
const firewall = (req, res, next) => {
    console.log(req.accessToken, '====')
    const {accessToken} = req.query
    const authorized = accessTokens.includes(accessToken || req.accessToken);
    if (!authorized) return res.status(403).send('Forbidden');
    next();
};

// attaching to route group
app.use('/api',
    expressAccessToken, // attaching accessToken to request
    firewall, // firewall middleware that handles uses req.accessToken
    (req, res) => res.status(200).send({
        message: 'api route'
    }));


// attaching to dedicated method, route
app.get('/restricted-route',
    expressAccessToken, // attaching accessToken to request
    firewall, // firewall middleware that handles uses req.accessToken
    (req, res) => res.send('Welcome to restricted page'));
```

1. http://127.0.0.1:8080/restricted-route?accessToken=6d7f3f6e-269c-4e1b-abf8-9a0add479511 










[HTTPS数字证书和数字证书链](https://github.com/wuchangming/https-mitm-proxy-handbook/blob/master/doc/Chapter3.md)

[给 localhost 颁发一份证书](https://blog.swwind.me/post/certificate)

[一文彻底搞懂Cookie、Session、Token到底是什么](https://zhuanlan.zhihu.com/p/101819794)
[浏览器系列之 Cookie 和 SameSite 属性](https://github.com/mqyqingfeng/Blog/issues/157)
