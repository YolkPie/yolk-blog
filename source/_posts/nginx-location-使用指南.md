---
title: nginx location 使用指南
date: 2020-02-04 16:17:49
tags: nginx
categories: 服务器
author: 老四
keywords: nginx,location
description: nginx location 是nginx基于http协议跟外界沟通的桥梁，用来匹配并执行一个规范化的URI，任何http请求都需要经过它的“同意”才能通过nginx的大门。
cover: https://img12.360buyimg.com/imagetools/jfs/t1/95958/2/11624/13392/5e392950Ed685cee4/cf582b9085cb9559.jpg
top_img: https://img11.360buyimg.com/imagetools/jfs/t1/107850/26/5428/10753/5e392934E259812cc/b3dfc28b19f74835.jpg
---
## 什么是location
* 它是nginx基于http协议跟外界沟通的桥梁，用来匹配并执行一个规范化的URI，任何http请求都需要经过它的“同意”才能通过nginx的大门
* 它长什么样子呢，比如我们常见的一种配置
<!-- more -->

```nginx
location /hello {
   return 200 'hello world';
}
```
* /xxxx 除了这种还有没有其他的写法？ 是不是没有？ 要想知道有没有其他的写法是不是要从系统层面（api）去了解一下，才能扩展 [官方](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)

```nginx
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```

语法规则很简单，一个location关键字，后面跟着可选的修饰符，后面是要匹配的字符，花括号中是要执行的操作。

* 修饰符
= 表示精确匹配。只有请求的url路径与后面的字符串完全相等时，才会命中。
~ 表示该规则是使用正则定义的，区分大小写。
~* 表示该规则是使用正则定义的，不区分大小写。
^~ 表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找

## 精确匹配
```nginx
location = /a {
    default_type "text/html";
    return 200 "wo shi /,url=$uri";
}
```

精确匹配一个字都不能错, /a 可以，但是 /a/ 就不行

## 普通匹配    / 和  ^~ 

* /   我们经常使用，是以其后设置的uri作为前缀来匹配请求的
* ^~  不常用的 也是以其后设置的uri作为前缀来匹配请求的
* 从概念上讲他们是一样的，实际使用也是一样的，但是他们有哪些细微差别呢？ 这个稍后讲

```nginx
location /b {
    default_type "text/html";
    return 200 "wo shi /,url=$uri";
}

location ^~ /b {
    default_type "text/html";
    return 200 "wo shi ^~,url=$uri";
}
```

我们访问  /b 或者 /bc 或者 /ba 都是可以访问的

验证一下 / ,^~ 是前缀匹配，我们可以试一下 /x/b 看看能不能访问？

## 正则匹配
* ~  区分大小写  ，在其后的uri就变成了正则表达式
* ~* 不区分大小写，在其后的uri就变成了正则表达式


```nginx
location  ~ /b {
    default_type "text/html";
    return 200 "wo shi /,url=$uri";
}

location ~* /b {
    default_type "text/html";
    return 200 "wo shi /,url=$uri";
}
```

上面说普通匹配的时候 我们使用了location /b ,这里正则也使用了 /b 如果同时存在，他们先使用那个呢？ 为什么？这个呢也稍后讲

还有普通匹配试了 /x/b 返回状态404 ， 那使用正则模式呢，状态是多少？ 正则模式 /b 可以是第一个 也可是是最后一个

正则是不是完事了？咦，不对 好像少了点什么？ 说好的正则 我堂堂 \d \w [a-z] \d{9,11} 哪里去了，是不是感觉少了点东西

```nginx
location  ~ "/(\d{1,11})" {
    default_type "text/html";
    return 200 "hi paimai,url=$uri";
}
```

是不是这样  我们拍卖单品页是不是这样的  paimai.jd.com/123456789 
是不是很完美？ 有没有问题?
前面我们正则部分我们说了  普通模式 /b 和正则模式 /b 区别 ，回顾一下
我之前写的时候就遇到过这样的问题  paimai.jd.com/album/123456789 也被匹配到了

那么问题来了 正则模式里面怎么精确匹配

^/a 的意思就是，匹配以/a开头的字符串
.html$   的意思就是 以.html结尾

再重新写一下单品页的匹配方式
```nginx
location ~* "^/(\d{9,11})$" {
    default_type text/html;
    return 200 "hi paimai,url=$uri";
  }  
```


使用正则定义的location在配置文件中出现的顺序很重要。因为找到第一个匹配的正则后，查找就停止了，后面定义的正则就是再匹配也没有机会了。

到这里正则模式才算完了^_^

## @ 模式
* 这里内部的意思是指外部用户看不到的location

```nginx
location @/d {
   return 200 "paimai";
}
```
我们可以试着访问一下

正确使用
error_page 404 @/d

## 刚刚我们在说  普通匹配 和正则匹配里面 放了两个 这个稍后讲 的问题 
1. / 和  ^~ 普通模式里面，两个同时存在的时候 使用的是那个location,是不是有个优先级的问题

```nginx
location /b {
    default_type "text/html";
    return 200 "wo shi /,url=$uri";
}

location ^~ /b {
    default_type "text/html";
    return 200 "wo shi ^~,url=$uri";
}
```

2. / 和  ~  普通模式 和 匹配模式 同时存在 /b 的时候，优先使用的是那个location 

```nginx
location /b {
    default_type "text/html";
    return 200 "wo shi /,url=$uri";
}

location  ~ /b {
    default_type "text/html";
    return 200 "wo shi ~/,url=$uri";
}
```

3. ^~ 和  ~ 的优先级呢

```nginx
location ^~ /b {
    default_type "text/html";
    return 200 "wo shi ^~,url=$uri";
}

location  ~ /b {
    default_type "text/html";
    return 200 "wo shi ~/,url=$uri";
}

```

得出的结论

普通匹配优先级要低于正则匹配，而^~的匹配优先级要高于正则匹配