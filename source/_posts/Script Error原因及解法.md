
---
title: Script Error原因及解法
date: 2021-09-09 14:49:25
tags:
- 微前端
categories: JS
author: zjk537
keywords: script error
description: script error
cover: 
top_img: 
---

`Script error` 可能是你遇到的最神秘的错误之一, 最让人抓狂的是这种错误没有提供完整的报错信息(错误堆栈), 让排查无从下手.

### 产生Script Error的原因
`Script error` 有时也被称为跨域错误. 当网站请求并执行一个托管在第三方域名下的脚本，就可能抛出 `Script error` 最常见的情况是采用CDN托管JS资源.为了更好地理解`Script error`, 假设有如下HTML页面，部署在 test.com 域名下
```html
<!doctype html>
<html>
<head>
  <title>Test page in http://test.com</title>
</head>
<body>
  <script src="http://another-domain.com/app.js"></script>
  <script>
  window.onerror = function (message, url, line, column, error) {
    console.log(message, url, line, column, error);
  }
  foo(); // 调用app.js中定义的foo方法
  </script>
</body>
</html>
```
假定foo方法中的内容如下, 调用了一个未被定义的bar方法
```js
// another-domain.com/app.js
function foo() {
  bar(); // ReferenceError: bar is not a function
}
```
页面运行之后，捕获到的异常信息如下：
```js
=>  Script error, "", 0, 0, undefined
```
其实这并不是一个`JavaScript bug`, 基于安全考虑浏览器有意隐藏其它域JS文件抛出的具体错误信息。这样可以有效避免敏感信息无意中被第三方(不受控制的)脚本捕获到，因此，浏览器只允许同域下的脚本捕获具体的错误信息。其它脚本只知道发生了一个错误，而不知具体发生了什么错误。且看 Webkit源码：
```js
bool ScriptExecutionContext::sanitizeScriptError(String& errorMessage, int& lineNumber, String& sourceURL)
	{
	    KURL targetURL = completeURL(sourceURL);
	    if (securityOrigin()->canRequest(targetURL))
	        return false;
	    errorMessage = `Script error`;
	    sourceURL = String();
	    lineNumber = 0;
	    return true;
	}
```
了解了`Script error`产生的原因, 接下来看看如何解决这类问题。
### 解法1: 开启CORS跨域资源共享

为了跨域捕获`javaScript`异常，分两步走
#### 第一步: 添加 crossorigin=”anonymous”属性
```html
<script src="http://another-domain.com/app.js" crossorigin="anonymous"></script>
```

这一步告诉浏览器，目标脚本通过匿名方式获取。这意味着请求脚本时没有潜在的用户身份信息(如cookies、HTTP 证书等)发送到服务端
#### 第二步: 添加跨域HTTP响应头
```js
Access-Control-Allow-Origin: *
```
或者
```js
Access-Control-Allow-Origin: http://test.com
```
注：大部分主流CDN默认添加了`Access-Control-Allow-Origi`属性, 如下是CDN的一个示例
```bash
$ curl --head https://retcode.xxxcdn.com/retcode/bl.js | grep -i "access-control-allow-origin"

=> access-control-allow-origin: *

```
完成上述两步之后，跨域脚本的报错就可以通过`window.onerror`捕获到，回到之前的案例，重新运行之后，捕获到的结果是
```js
=> "ReferenceError: bar is not defined", "http://another-domain.com/app.js", 2, 1, [Object Error]
```
#### 解法2: try catch
有时候，不容易往HTTP请求响应头里面添加跨域属性，这时还可以考虑`try catch`这个候选方案回到之前的案例
```html
<!doctype html>
<html>
<head>
  <title>Test page in http://test.com</title>
</head>
<body>
  <script src="http://another-domain.com/app.js"></script>
  <script>
  window.onerror = function (message, url, line, column, error) {
    console.log(message, url, line, column, error);
  }
  try {
    foo(); // 调用app.js中定义的foo方法
  } catch (e) {
    console.log(e);
    throw e; 
  }
  </script>
</body>
</html>
```
再次运行，输出结果如下:
```js
=> ReferenceError: bar is not defined
  at foo (http://another-domain.com/app.js:2:3)
  at http://test.com/:15:3

=> "Script error.", "", 0, 0, undefined
```
可以看出来， `try catch`中的`console`语句输出了完整的信息, 但`window.onerror`中只能捕获`Script error` 基于这个特点，可以在`catch`语句中，将捕获的异常手动上报。
