---
title: whistle使用实践（实例篇）
date: 2020-12-24 10:00:00
tags:
- whistle
cover: https://raw.githubusercontent.com/avwo/whistle/master/biz/webui/htdocs/img/whistle.png
top_img: https://raw.githubusercontent.com/avwo/whistle/master/docs/assets/whistle-en_US.png
---

# 抓包

作为一名前端开发，利用代理工具抓包是最基础的技能。通过抓包，我们可以获取的信息有下面这些：

1. 具体的url
2. 请求的Method，Status等信息
3. 请求接口携带的参数
4. 请求头信息（cookie、UA、请求参数等）
5. 返回头信息（返回结果、是否支持跨域等）

这部分和其他的代理工具没什么区别，都在Network模块里，不熟悉的话多点点就找到了。需要啰嗦几句的是，如果页面出问题，我们抓包的排查步骤是什么：首先确认控制台是否报错（见下节），确认页面地址及url携带的参数是否正确，还要确认要加载的静态资源或者接口的请求是否有非200的情况，当然，如果确切知道问题出在哪个接口上，直接查看接口的请求头和返回值信息就行了。

# 查看控制台报错

远程调试页面或者线上页面出现问题时，我们希望能够看到控制台的报错。这里有三种实现方式：

- 使用log规则
- 在页面注入vConsole
- 使用Weinre

如果只是简单查看控制台报错的话，经常用的是前两种方式，Weinre我会在下文中的Weinre模块单独说明。这里以查看京东首页的控制台内容来举例。

## 使用log规则

使用log规则很简单，在Rules里增加以下规则：

``` js
https://m.jd.com/ log://
```
规则添加完成后，切换到Network模块，刷新京东首页，在左侧的请求中找到https://m.jd.com/并选中，在右侧选中Tools标签，就可以看到控制台打印的信息了。

![控制台信息](https://img14.360buyimg.com/imagetools/jfs/t1/152818/39/11356/1022862/5fe45986Eba613fd6/c7bc33721f017f2d.jpg)

## 在页面注入vConsole

在页面注入cConsole其实就是使用了whsitle的js规则，该规则在响应中追加脚本，如果响应是html文档，则自动用`<script></script>`包装后插入

> vConsole.js 文件地址：http://wechatfe.github.io/vconsole/lib/vconsole.min.js?v=3.3.0

1. 在Values中增加vConsole.js，并把从上面链接获取到的vConsole.js的内容粘贴到新增的vConsole.js中
2. 在vConsole.js中追加初始化逻辑：
``` js
new window.VConsole();
``` 
3. 在Rules中配置如下规则：

``` js
https://m.jd.com/ js://{vConsole.js}
```
4. 刷新京东主页，就可以在页面右下角看到vConsole了，点击vConsole，就可以看到控制台的信息了
![vConsole](https://img11.360buyimg.com/imagetools/jfs/t1/154434/11/11513/852204/5fe45fb5Ec33d1d66/3b1229d5e9d38030.jpg)

就我个人而言，我比较习惯使用注入vConsle的方式，因为可以同时使用多台设备，输出的内容互不影响，而且使用vConsole可以查看localStorage和sessionStorage等缓存信息。

# 配Host

配置方式和我们使用SwitchHosts的软件配置host的方式一样。比如开发时需要使用jd.com的域名才能使用登录态，可以这样配置：

``` js
// dev.jd.com是我经常用的，表示开发环境，可以随便叫 XXX.jd.com
127.0.0.1 dev.jd.com
```

如果一个IP对应多个域名，可以这样写：
``` js
127.0.0.1 dev.jd.com dev1.jd.com
```

# 代理域名 

比如预发环境接口域名为xxx-api.m.jd.com，线上域名是api.m.jd.com，如果遇到预发环境在发布或者预发环境有问题，而我们在做的任务只涉及到前端的改动，这时候我们可以使用whistle暂时将域名的接口先代理为线上，预发好了再关掉代理就行了。这样子我们就可以不必在我们代码上来回做更改，也不必频繁启动项目了。

``` js
// 将域名为xxx-api.m.jd.com的请求代理到api.m.jd.com
xxx-api.m.jd.com api.m.jd.com
```

# 代理端口

比如我启动的项目端口是3000，我用80又不想重启项目（懒的有点过分哈），可以这样代理：

``` js
dev.jd.com dev.jd.com:3000
```

这样子即使启动的是3000，我们依然可以使用dev.jd.com。

# 代理https

有时候因为HSTS，请求被转成https，或者某些app只能打开https的请求。这时候可以使用whislte将https请求代理成http：

``` js
https://dev.jd.com http://dev.jd.com
```
这样子即使是用https访问，页面也可以正常打开。

# mock数据

在项目开发中，经常遇到的情况是前端静态页和交互好了，后端接口没完成或者没数据，或者说有些数据的状态不易获得，这时候，使用mock数据来开发是比较高效的手段。还有一种情况是线上遇到问题，我们需要拿不同状态的数据去复现。（将mock的数据写在代码里也是一种方式，但这种会涉及代码的改动，不建议这样做，首先是因为我们要保证代码逻辑和mock数据的分离，另一个原因是如果不是开发环境，这种方式也不适用）

拿获取用户是否是新用户的接口举例，这个接口支持JSONP和非JSONP两种形式

## mock非JSONP请求

> https://api.m.jd.com/api?appid=paimai&functionId=queryNewComerInfo&body={}

我们需要先在Files或者Values添加名为queryNewComerInfo.json的文件，并在这个文件里根据接口文档要求的数据格式创建假数据：

``` json
{
    code:0,
    data:
        {
            newComerFlag:false,
            pin:"XXXXXXX"
        },
    datas:[],
    message:"操作成功",
    statusCode:200
}
```
再在Rules里添加规则

``` js
// 使用正则匹配queryNewComerInfo的方法，返回queryNewComerInfo.json里的内容
/functionId=queryNewComerInfo/ file://{queryNewComerInfo.json}
```

## mock JSONP 请求

> https://api.m.jd.com/api?appid=paimai&functionId=queryNewComerInfo&body={}&_=1608811151553&jsonp=Zepto1608810637767

 如果访问上面的地址，JSONP请求返回的数据是这样的：
 ``` json
    Zepto1608810637767(
        {
        code: 0,
        data: {
            newComerFlag: false,
            pin: "XXXXXXX"
        },
            datas: [ ],
            message: "操作成功",
            statusCode: 200
        }
    )
 ```

 熟悉JSONP的应该都知道，返回内容的回调函数名称Zepto1608810637767和url中jsonp（默认为callback）的参数值保持一致的。因为jsonp的参数值是随机的，我们就不能使用json格式的数据直接返回。这里我们要用到whislte的tpl：

 > tpl基本功能跟file一样可以做本地替换，但tpl内置了一个简单的模板引擎，可以把文件内容里面{name}替换请求参数对应的字段(如果不存在对应的自动则不会进行替换)，一般可用于mock jsonp的请求。

所以，如果请求是JSONP，我们就需要在返回结果的json中使用{jsonp}来替换url中jsonp的参数值：

 ``` json
    {jsonp}(
        {
        code: 0,
        data: {
            newComerFlag: false,
            pin: "XXXXXXX"
        },
            datas: [ ],
            message: "操作成功",
            statusCode: 200
        }
    )
 ```

whsitle规则配置如下：

``` js
/functionId=queryNewComerInfo/ tpl://{queryNewComerInfo.json}
```

如果以上还不能满足mock数据的要求，想mock的数据有更大的灵活性或者数据项中间有逻辑关系，可以使用[whistle.vase](https://github.com/whistle-plugins/whistle.vase)

> vase插件内置default、doT、dust、ejs、handlebars、jade、mock、mustache、nunjucks、swig、vm及用于解析自定义脚本的script等渲染引擎，通过该whistle插件，可以通过模板结合相应的引擎mock开发过程中需要的json、html、图片等数据，也可以通过script来自定义脚本更加灵活的获取模板及数据，控制输出等（如果只是静态数据不需要借助模板引擎批量生成，直接利用whistle的 file 或 xfile 即可实现）

工作中我基本上用的都是whistle的file或者tpl，如果数据项有逻辑关系，我觉得手动修改json文件就可以满足需要了，这里不再对vase多做说明，[whistle.vase](https://github.com/whistle-plugins/whistle.vase)官网也说的很清楚了，有兴趣的可以自己去了解。

# 使用Weinre

whistle集成了weinre的功能，用于调试远程页面特别是移动端的页面。
配置方式是：```pattern weinre://key```

比如我要调试京东主页。在whistle中的配置如下，这里的key就是honepage：

``` js
https://m.jd.com/ weinre://homepage
```

配置完之后，点击页面顶部的Weinre，就可以看到一条homepage的数据（配置规则中不指定key，默认为anonymous）：
![weinre](https://img11.360buyimg.com/imagetools/jfs/t1/155708/16/3330/237012/5fec1465Ecfb6517a/d399c319383ef160.jpg)

点击homepage，显示如下：
![weinre](https://img10.360buyimg.com/imagetools/jfs/t1/157577/18/688/562109/5fec14e3E05672fea/55603df3d5f7323d.jpg)

从上图我们看到，weinre中有我们在浏览器中常用的Elements、Resources、Network、TimeLine、Console等模块。
Targets表示当前符合过滤规则的页面，显示为none表示没有。我们使用配置好代理的手机打开https://m.jd.com/，就会发现Targets中多了条记录：
![weinre](https://img12.360buyimg.com/imagetools/jfs/t1/159465/6/668/607563/5fec16acEad4410fc/64a1619cf4cab11a.jpg)
这时候我们点击页面上的Elements、Console等模块，就可以和看到对应的页面的内容了：
![weinre](https://img14.360buyimg.com/imagetools/jfs/t1/156879/24/3306/2128832/5fec170dE67da5de7/8e75d2736994cac4.jpg)

在Elements中选中对应的DOM节点，对应的手机端就会和浏览器一样标注出DOM的位置，这对我们查看DOM有没有正确渲染是很有用的：
![weinre](https://img14.360buyimg.com/imagetools/jfs/t1/159371/15/644/1970789/5fec189dE5b74bea6/9c2e9cc7d43353a6.jpg)
![weinre](https://img11.360buyimg.com/imagetools/jfs/t1/160851/39/83/1764028/5fec18c6Eee4af1f7/9c35ed315b9e8807.png)


















 





























