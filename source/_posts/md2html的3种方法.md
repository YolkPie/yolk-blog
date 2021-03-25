---
title: MD To Html 的前端实现
date: 2021-03-25 21:28:00
author: zjk537
---

.md文件是markdown的一种标记语言，和html比较起来，更简单快捷，主要体现在：标记符的数量和书写上。
> 标记符的数量：html文档要用N多个标记符加上css来控制样式； 而markdown文档只用到4个基本的标记符号。
标记符的书写：HTML文档需要<开始>和<结束>标识一个网页，而markdown文档只要在开始位置标记即可# 这是一个md文档。

下面看看前端是如何实现.md文件转换成.html文件。

## 方式一：使用i5ting_toc插件
需要先安装npm（安装node.js后会自带npm），然后才能安装i5ting插件：
```bash
npm install i5ting_toc -g
```
执行命令行生成html文件，在输入前要进入到对应根目录下：
```bash
i5ting_toc -f **.md
```
需要注意的是：写md文档的特殊符号时记得添加空格。

## 方式二：使用gitbook
同样先需要安装node，然后运行
```bash
npm i gitbook gitbook-cli -g
```
生成md文件,这个命令会生成相应的md的文件，然后在相应的文件里写你的内容即可:
```bash
gitbook init
```
md转html,生成一个_doc目录，打开就可以看到你html文件了。
```bash
gitbook build
```
## 方式三：利用前端代码
实现原理是采用node.js搭建服务器，读取md文件转化为html片断。浏览器发送ajax请求获取片段后再渲染生成html网页。  

`node代码`
```js
var express = require('express');
var http = require('http');
var fs = require('fs');
var bodyParser = require('body-parser');
var marked = require('marked');    // 将md转化为html的js包
var app = express();

app.use(express.static('src'));  //加载静态文件
var urlencodedParser = bodyParser.urlencoded({ extended: false });

app.get('/getMdFile',urlencodedParser, function(req, res) {
    var data = fs.readFileSync('src/test.md', 'utf-8');    //读取本地的md文件
    res.end(JSON.stringify({
        body : marked(data)
    }));
} );

//启动端口监听
var server = app.listen(8088, function () {
    var host = server.address().address;
    var port = server.address().port;
    console.log("应用实例，访问地址为 http://%s:%s", host, port)
});
```
前端
```html
<div id="content">
    <h1 class="title">md-to-HTML web app</h1>
    <div id="article">
    </div>
</div>
<script type="text/JavaScript" src="js/jquery-1.11.3.min.js"></script>
<script>
    var article = document.getElementById('article');
    $.ajax({
        url: "/getMdFile", success: function(result) {  
            console.log('数据获取成功');
            article.innerHTML = JSON.parse(result).body;
        }, error: function (err) {
            console.log(err);
            article.innerHTML = '<p>获取数据失败</p>';
        }
    });
</script>
```