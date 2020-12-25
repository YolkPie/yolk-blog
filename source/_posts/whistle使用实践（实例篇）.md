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
- 使用Wenire

如果只是简单查看控制台报错的话，经常用的是前两种方式，Wenire我会在本文单独说明。这里以查看京东首页的控制台内容来举例。

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


# mock数据

在项目开发中，经常遇到的情况是前端静态页和交互好了，后端接口没完成或者没数据，或者说有些数据的状态不易获得，这时候，使用mock数据来开发是比较高效的手段。当然，mock数据的前提是后端提供了接口的数据结构。

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

 熟悉JSONP的应该都知道，返回内容的回调函数名称和url中jsonp的参数值（默认为callback）保持一致的，因为jsonp的参数值是随机的，我们就不能使用json格式的数据直接返回。这里我们要用到whislte的tpl：

 > tpl基本功能跟file一样可以做本地替换，但tpl内置了一个简单的模板引擎，可以把文件内容里面{name}替换请求参数对应的字段(如果不存在对应的自动则不会进行替换)，一般可用于mock jsonp的请求。

所以，如果请求是JSONP，我们就需要在返回结果的json中使用{jsonp}来替换url中jsonp的参数值：

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

# 使用Weinre












 





























