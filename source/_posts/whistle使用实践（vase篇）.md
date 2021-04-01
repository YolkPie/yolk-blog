---
title: whistle使用实践（vase篇）
date: 2021-03-27 11:30:00
tags:
- whistle
cover: https://raw.githubusercontent.com/avwo/whistle/master/biz/webui/htdocs/img/whistle.png
top_img: https://raw.githubusercontent.com/avwo/whistle/master/docs/assets/whistle-en_US.png
---

# 碎碎念

之前我在[whistle使用实践（实例篇）](https://yolkpie.net/2020/12/24/whistle%E4%BD%BF%E7%94%A8%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%AE%9E%E4%BE%8B%E7%AF%87%EF%BC%89/)中的 mock 数据一节中说过使用 file 或者 tpl 就能满足大部分的工作需要，但是想 mock 的数据有更大的灵活性或者数据项中间有逻辑关系，可以使用whistle.vase。因为 vase 官网已经说的很详细了，而且我在工作中一般都手动改 json 文件，也没用上过 vase，本来不打算写这篇的。但是最近部门的测试同学遇到了在页面回退时需要变更 mock 数据的情况，来找我问解决方案，这让我想起了自己之前手动改 json 文件的繁琐，因此就以此为契机，记录下 whistle.vase 来 mock 数据的方式，仅供大家参考。

这篇文章依然从实用性出发，只是介绍了对于前端人员最易上手的方式，如果想要了解更多，依然推荐去 [whistle.vase 官网](https://github.com/whistle-plugins/whistle.vase)。 

# whistle.vase

whistle.vase 是 whsitle 的一个插件，内置了 default、doT、dust、ejs、handlebars、jade、mock、mustache、nunjucks、swig、vm 以及用于解析自定义脚本的 script 渲染引擎，通过该插件，我们可以更灵活的控制要 mock 的数据。

## 安装

whistle.vase 不是 whistle 内置的插件（whistle 没有内置任何插件），我们还需要手动安装一下:

``` 
npm install -g whistle.vase

```

安装成功后，可以在 whistle 控制台的 Plugins 下面看到。如果没有的话，可以重启下 whistle。

![whistle Plugins](https://img14.360buyimg.com/imagetools/jfs/t1/157098/20/15542/552298/605e9e6dE8ded985b/75e0d92dda2c8ba8.jpg)

有两种方式可以打开 vase 的配置页面，一种是在上图中找到 vase 点进去，另一种是使用浏览器访问http://local.whistlejs.com/whistle.vase/。因为我习惯在一个页面配所有，采用的是第一种方式。

![vase 配置界面](https://img14.360buyimg.com/imagetools/jfs/t1/156343/34/18186/548396/605e9e6cE9da59800/e3a05e272c9ee81e.jpg)

从上图可以看到，vase 的配置页面和 whistle 长的很像，如果使用 vase 的渲染引擎 mock 数据，就要在 vase 的配置页添加。

## 添加模板

点 Create 按钮， 选择对应的渲染引擎，这里我们先选择 default。default 是 vase 的默认引擎，即直接输出设置的文本，不做任何加工。（感觉和 Values 的功能一样呢，存疑，这部分以后再了解。）

![添加 default 模板](https://img13.360buyimg.com/imagetools/jfs/t1/172683/28/616/656218/605e9e6bEb28c681a/b86f96870f45953d.jpg)

## 配置

vase 的配置规则很简单，左侧是对应的文件，右侧是要匹配的规则：

```
vase://vase-file rule1 rule2 rule3
```

比如，我要 mock 京东首页的数据：

``` 
vase://hello.html https://m.jd.com/
```

访问首页，结果如下：

![mock 京东首页](https://img12.360buyimg.com/imagetools/jfs/t1/163843/40/14672/1121348/605e9e69E2a1c705c/506874b21621653f.jpg)

# 强大的 script

上面提到的 default 渲染引擎因为是静态的，依然解决不了灵活输出的问题，这时候就要用到我们前端人员最为熟悉的 script 了。

> script 是 vase 系统自定义的脚本解析器，保留了 JavaScript 的一些基本特性，如：基本类型、条件语句、循环体、方法等，剔除了 JavaScript 内置的一些 api，如：process、setTimeout、setInterval 等，并内置了一些方法用于读取及处理 vase 的模板、本地文件、线上文件等，且所有调用都是同步的方式。

也就是说，除了 setTimeout 这些异步的方法之外，script 还提供了读取 vase 模板、本地和线上文件的功能，使我们能够更加灵活的获取数据源、控制输出。另外，script 还可以控制速度、分段输出、设置响应头、响应状态码、响应内容编码等，是不是很强大呢？

script 模板的创建很简单，在创建的时候文件使用 .js 后缀，选择 script 引擎就可以了。

![script 创建](https://img12.360buyimg.com/imagetools/jfs/t1/162308/22/15178/687759/605efacaEbba851a7/f8caa826f75da9bf.jpg)

本着一招鲜吃遍天的原则，script 是我认为的 vase 中最创新的（况且渲染时还可以使用所有上面列出来的渲染引擎，见script API 中render 方法）且是前端最容易掌握的，如果不想学其他的渲染引擎的话，我觉得这个就够了。我们先逐个看下 script 提供的方法吧。

## script API

1. out(data, delay, speed)  

所有的数据都要通过该方法才能输出到响应中，也可以用 write。

- data：要输出的对象，可以是 json、文本，也可以是 vase 加载的资源。

``` js
// 输出文本
out('hello world');

// 输出 json
out({
    content: 'hello world'
});

// 输出 vase 加载的资源（略，详见 14:render）
```

- delay: 设置延迟多少毫秒输出。
- speed: 设置输出的速度(单位kbs)。

2. status  
设置输出的http状态码，默认为200，也可以写成 statusCode(code);

``` 
out(status(500));
```

3. header(name, value)

设置响应头。

``` js
// 设置允许跨域的相应头
out(header('Access-Control-Allow-Origin', '*'));

```

4. headers(obj)
和 header 方法一样，用于设置相应头，不同的是 headers 可以同时设置多个。

``` js
// 同时设置多个请求头
out(headers({
    'Access-Control-Allow-Origin': '*',
    'content-type': 'text/plain; charset=utf8'
}));

```
5. file(path)  
读取本地文件。

``` js
// mac
out(file('/User/xxx/hello.html'));

// windows
out(file('D:/xxx/hello.html')); 
```

6. get(url|options)  

通过get方式获取线上文件，支持https及http协议。

``` js
// 参数为url
out(get('https://m.jd.com/'));

// 参数为options时，可以自定义请求头
out(get({
	url: 'https://www.taobao.com/',
	headers: {
		'User-Agent': 'vase/x.y.z'
	}
}));
```

7. post(url|options)  

通过post方式获取线上文件，支持https及http协议。

``` js
// 和 get 不同之处：还可以自定义表单数据
out(post({
	url: 'https://m.jd.com/',
	headers: {
		'User-Agent': 'vase/x.y.z'
	}, 
	form: {key:'value'}
}));
```

8. request(options)  

通过自定义方式获取线上文件，支持https及http协议。

``` js
out(request({
	url: 'https://m.jd.com/',
	method: 'get',
	headers: {
		'User-Agent': 'vase/x.y.z'
	}
}));
```

9. json(data)  

将文本转换成json 对象。

``` js
// 转换字符串
out(json('{"test": 123}'));

// 转换本地加载的文件内容
out(json(file('/User/xxx/hello.txt')));

//  转换线上获取到的内容
out(json(get('https://m.jd.com/xxx/hello.txt'));
```

10. merge(json, ..., jsonN)

合并 json 对象。

``` js
// 合并不同来源的 json 数据
out(merge(json('{"test": 123}'), json(file('/User/xxx/hello.txt'))));

// vase >= v1.3.1 时，支持深度合并 merge(deepMerge, json, ..., jsonN)
out(true, merge(json('{"test": 123}'), json(file('/User/xxx/hello.txt'))));
```

11. random(arg1, ..., argN)

随机获取参数列表中的一个。

``` js
// 随机取数
out(random(1, 5, 8, 10));
```

12. join(arr, seperator)

与数组的 join 方法一样用于拼接数组，seperator默认为''。

``` js
// 返回拼接好的地址
out(join(['province', 'city', 'county', 'town'], '-'));

// 如果使用默认的seperator， 可以写成join(arg1, ..., argN)
out(join('province', 'city', 'county', 'town'));
```

13. req

请求对象，包含：headers、method、body、query、url、locals(=merge(req.query, req.body))。

``` js
// 判断请求对象的 method 是否为 OPTION，是的话放过
if (req.method === 'OPTION') {
    out(status(200));
}
```

14. render(tpl[, locals[, engineType]])

渲染模板。（这里是重点！）

- tpl：vase 的模板名称或模板字符串。

- locals：可选，用于渲染的json对象。

- engineType：可选，渲染引擎名称，包含 default、doT、dust、doT、ejs、jade、mock、mustache、nunjucks、swig、vm。

14.1 如果tpl是字符串或数字，且 vase 里面有对应名称的模板，则会自动加载 vase 的模板内容。

``` js
// 使用 req.locals(=merge(req.query, req.body)) 的值作为模板的数据
out(render('test-tpl'), req.locals);

// 使用获取到的 json 数据作为模板的数据

out(render('test-tpl'), json(get('https://m.jd.com/xxx/test.txt'));

// 如果模板使用的是其他渲染引擎（如handlebars），还需要在第三个参数指定
out(render('test-tpl'), req.locals, 'handlebars');

```

14.2 如果tpl是字符串或数字，且没有对应的vase模板，则这些字符串作为模板内容。

``` js
// 字符串作为模板内容
out(render('Hello {{name}}', {name: 'World'}, 'handlebars'));

```

14.3 可以渲染线上或本地文件模板

``` js
// 渲染线上模板
out(render(get('http://m.jd.com/'), json(get('https://m.jd.com/xxx/test.txt'), 'handlebars'));

// 渲染本地文件模板
out(render(file('/User/xxx/test-tpl.txt'), json(get('https://m.jd.com/xxx/test.txt'), 'handlebars'));
```

从上面的 API 可以看出，vase 获取数据（本地文件、vase 模板、线上、字符串等数据内容，以及请求头内容req）、渲染数据（render 使用的模板和要渲染的数据可以有多种来源，可以使用多种渲染引擎）的方式都很灵活，还可以灵活的设置请求头和延迟时间，只要我们面对具体情况的时候善于组合，就妥妥的了。

## 实例

回到测试同学遇到的问题，简单来说，测试需要的是同一个字段可以使用不同的值（0 和 1），看了上面的 API 之后，你应该就会觉得太简单了，随机下 0 和 1 就行了，杀鸡用牛刀的感觉：

``` js
out(random(1, 5, 8, 10));
```

再稍微复杂点，比如在拍卖业务中，不同业务线返回的对象是不同的。那我们可以根据请求参数，渲染不同的模板或者返回不一样的值。

``` js
// 使用不同模板
if (req.locals) {
    switch (req.locals.publishSource === 'OPTION') {
        case 'sifa':
            // render 方法可以同样根据 req 中拿到的数进行逻辑上的处理
            out(render('sifa-tpl')）;
            break;
        case 'haiguan':
            out(render('sifa-tpl')）;
            break;
        ...
    }
    out(status(200));
}
```

如果你需要造很多条数据，也可以使用 vase 来写（当然，你也可以选一个模板来写循环）：

``` js
// 造100条数据
let arr = [];
for (let i = 0; i < 100; i++) {
  arr.push({
    id: i,
    name: `user${i}`
  })
}
out(arr);
```

总之，灵活运用很重要。

# 追加的碎碎念

whistle 真是个宝藏，所以我应该不会说什么不再写的 flag 了，如果再遇到具体的问题，这个系列就又要更新了。


