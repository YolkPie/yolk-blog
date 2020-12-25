---
title: whistle使用实践（配置与基础篇）
date: 2020-12-22 10:00:00
tags:
- whistle
cover: https://raw.githubusercontent.com/avwo/whistle/master/biz/webui/htdocs/img/whistle.png
top_img: https://raw.githubusercontent.com/avwo/whistle/master/docs/assets/whistle-en_US.png
---

# 简介

whistle是基于Node实现的跨平台web调试代理工具，同类型的工具有Fiddler和Charles，主要用于查看、修改HTTP、HTTPS、Websocket的请求、响应，也可以作为HTTP代理服务器使用。

在使用了Fiddler、Charles以及whistle这三款代理工具之后，总结出来的whistle的优势有以下几点：
1. 配置简单：whistle的配置类似于系统hosts的配置，一切操作都可以通过配置实现，支持域名、路径、正则表达式、通配符、通配路径等多种匹配方式。
2. 支持扩展：whistle提供了插件扩展能力，通过插件可以新增whistle的协议实现更复杂的操作、也可以用来存储或监控指定请求、集成业务本地开发调试环境等等，基本上可以做任何你想做的事情，且开发、发布及安装whistle插件也都很简单。
3. 内置weinre：通过weinre可以修改调试移动端DOM结构、捕获页面异常等。
4. 界面简单易懂：从界面来看，whistle的功能划分为了network（网络）、rules（规则）、values（数据）、pulgins（插件）四大模块，通过tab页签进行切换。
5. 文档全面：whistle官网提供了详细的说明文档，工作中遇到的情况只要查阅文档都能解决。

下图是whistle支持的功能:
![whistle功能](https://raw.githubusercontent.com/avwo/whistle/master/docs/assets/whistle-en_US.png)

本文将结合本人在使用whistle过程中遇到的问题，对whisle的安装启动、配置及基础知识做下梳理。在介绍安装、配置等步骤时，有时会杂糅一些遇到的问题，没有遇到的话可以直接跳过。另外需要说明的是，由于是结合使用体验做的梳理，许多平时用不到或使用频率较低的内容，会写的很简略，如果本文解决不了你的问题的话，还请到[whsitle官网](https://wproxy.org/whistle/)去查看文档。

# 安装启动

## 安装Node

由于whistle是基于Node的，自然需要先安装Node环境，这里不再多做说明。下面是whistle官网对Node版本的建议：

> whistle支持v0.10.0以上版本的Node，为获取更好的性能，推荐安装最新版本的Node。

## 安装whistle

Node安装完成后，执行如下命令安装whistle：

``` js
$ npm install -g whistle
```

查看版本：
``` js 
whisle -V 
```
如果能正确输出whistle的版本信息，就表示安装成功了。之后可通过help命令查看帮助信息：

``` js
whistle help
```
## 启动whistle

whistle支持三种等价的命令whistle、w2、wproxy，本文使用w2命令。下面是常用的命令：

启动whistle:
``` js
w2 start
```

whistle的默认端口是8899，如果要指定端口号，执行下面的命令：
``` js
w2 start -p 8888
```

重启whistle（也支持指定端口）:
``` js
w2 restart
```

停止whistle:
``` js
w2 stop
```

# 配置代理
配置代理时有两个关键的参数：服务器IP和端口号。端口号比较简单，对应w2 start命令启动好的端口号即可。服务器IP的话需要分两种情况：一种是本地，对应127.0.0.1即可，另一种是远程，这时候需要填服务器的IP。配置所需要的信息在启动whistle时控制台会告诉我们，见下图：

![whistle启动界面](https://img13.360buyimg.com/imagetools/jfs/t1/154775/4/6340/81700/5fb50cb4Ec59755b9/5a2bed632ddd9f56.png)

从上面的图片可以看出，端口号为8899，IP有127.0.0.1、10.1.2.30、192.168.137.1和192.168.191.1这四个，如果是本地代理的话，这四个IP都可以（如果不想每次IP更换都要重新配置，选127.0.0.1呀），如果是远程代理，除了127.0.0.1之外，其他都可以的（仍然不建议填10.1.2.30这个IP，理由同上）。

下面来说下配置代理的几种方式：

## 浏览器代理
浏览器代理的话要使用浏览器的代理插件，这里介绍chrome和firefox两种：

1. chrome：使用[SwitchOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif)插件。
2. firefox：地址栏输入访问 about:preferences，找到 Network Proxy，选择 手动代理配置(Manual proxy configuration)，输入代理服务器地址、端口，保存即可。

## 全局代理
1. Windows: 菜单 > 设置 > 网络和Internet > 代理

![Windows全局代理](https://img12.360buyimg.com/imagetools/jfs/t1/137254/31/16451/53934/5fb65e59E4d64895c/1dd7cde6d531b03e.png)

2. Mac: 系统偏好设置 > 网络 > 高级 > 代理 > 网页代理(HTTP) 和 安全网页代理(HTTPS)

![Mac全局代理](https://img13.360buyimg.com/imagetools/jfs/t1/154826/14/6517/443129/5fb65f54E171875cf/a966123ef2007a17.jpg)

3. Linux: 工作中没有用过，此处略过。详见[whistle官网](https://wproxy.org/whistle/install.html)

## 手机代理

手机端要配置代理的话，需要保证所连wifi和启动了代理的主机处在同一网络。以IOS为例：
1. 连接网络（公司手机和电脑连接的不是同一网络，这个网络为电脑开的热点）：
![连接wifi](https://img14.360buyimg.com/imagetools/jfs/t1/150209/33/15612/33983/5fbc6ac5E57083b51/25b8d9225152042f.jpg)

2. 点击配置代理，修改为手动：
![配置代理](https://img14.360buyimg.com/imagetools/jfs/t1/150209/33/15612/33983/5fbc6ac5E57083b51/25b8d9225152042f.jpg)

# 访问whistle控制台

配置完代理后，我们需要先验证下代理是否配置成功。不同于Fiddler和Charles，whistle需要用浏览器访问配置页面（即whistle控制台），有以下两种方式：
1. 域名访问：http://local.whistlejs.com/
2. IP + 端口访问： 如http:192.168.191.1:8888

> 因为我们是要验证刚才配置的代理是否生效，所以访问控制台的IP应该和配置代理的IP保持一致。

如果配置正确，我们就可以看到whistle的控制台了，如下图：
![whistle页](https://img12.360buyimg.com/imagetools/jfs/t1/125525/19/19818/166034/5fbfa019E4b6928a9/355841e136dd3e81.jpg)

如果页面打不开，则可能是whistle所在的电脑防火墙限制了远程访问whistle的端口，需要关闭防火墙或者设置白名单。

设置白名单的步骤如下：
1. 控制面板 -> 系统和安全 -> Windows Defender防火墙 -> 高级设置 -> 入站规则 -> 新建规则
2. 选择端口
![选择端口](https://img14.360buyimg.com/imagetools/jfs/t1/145577/24/15737/27592/5fbfa3f9Ee31d94d1/7a49a3b0c2f3786e.png)
3. 输入wistle启动时设置的端口号
![输入端口号](https://img11.360buyimg.com/imagetools/jfs/t1/143009/14/15672/26988/5fbfa3f9E26bca6f3/7948166c78f175a0.png)
4. 选择允许连接
![允许连接](https://img12.360buyimg.com/imagetools/jfs/t1/134197/24/17457/28670/5fbfa3f9Ed82a1b51/a3c1bd1db9bccd52.png)
5. 输入设置的规格名称
![输入规则名称](https://img12.360buyimg.com/imagetools/jfs/t1/135720/28/17480/22628/5fbfa3f9Ef188fc6c/8d3854390d3657a8.png)
6. 在入站规则列表中查看是否新建成功以及是否开启，开启之后whistle提供的服务就可以在局域网中访问了
![查看入栈规则](https://img14.360buyimg.com/imagetools/jfs/t1/148633/37/18944/99950/5fdc21c6E2d1652c8/a1ccf9c2dfae880f.png)


# 安装https证书

关闭防火墙或者给whsitle设置了白名单之后，如果whistle的设置页面可以正常打开，这表示说我们可以代理http请求了。

如果你的页面和接口全部是http请求，就可以忽略安装https证书的这一步了。但现实是除了本地或者预发环境，我们很难找到不是https的了（很多预发环境也是https的），因此还是建议提前把证书装上。
如果你的环境中出现了以下情况（当然，没有装好证书的话这些情况基本都会出现的），就是https证书没有安装或者没装好：
1. whisle的配置页面可以打开，但是网页不能打开或者只加载了一部分页面
2. 京东App数据更新不了或展示不全，或者扫码提示“无法获取信息”
3. whistle配置页面中network中443端口的请求前面有小锁，或者抓不到请求
4. 浏览器提示“您的连接不是私密连接”

## 下载证书

我们可以通过下面的方式下载证书：
1. 在配置代理的设备上打开浏览器，在浏览器中输入rootca.pro即可下载，这种是最便捷的方式
2. 在启动了whistle的机器上用浏览器打开配置页面，点击https，会弹出一个带二维码的界面，点击Download RootCA 或者扫二维码下载
![https证书](https://img13.360buyimg.com/imagetools/jfs/t1/149838/6/18808/18953/5fdc4c49Efff2123b/80e0a6e0522277e1.png)

## 安装证书

证书的安装是不依赖于代理工具的，也就是说我们无论用的是whistle还是Fiddler或者是Charles，步骤都是一样的。我把安装步骤单独做了汇总，详见[【安装证书】](https://yolkpie.github.io/2020/12/22/%E5%AE%89%E8%A3%85%E8%AF%81%E4%B9%A6/)

## 开启拦截https

安装好证书之后，必须开启https拦截功能之后，whistle才能看到HTTPS、Websocket的请求。开启https拦截和安装并信任证书，这两个条件缺一不可。
我们需要通过下面的步骤开启https拦截：在启动了whistle的机器上用浏览器打开配置页面，点击https，会弹出一个带二维码的界面，在这个界面勾选Capture HTTPS CONNECTs选项（是不是很熟悉，在下载证书的第2种方法里我们见过的）
![开启https拦截](https://img13.360buyimg.com/imagetools/jfs/t1/149838/6/18808/18953/5fdc4c49Efff2123b/80e0a6e0522277e1.png)

> 上面的配置完成之后，如果https的请求还是不能正常访问或者还是出现安全提醒，可以重新打开浏览器访问或者重启下whistle。whistle官网给出的解释是如果之前访问过该页面，导致长链接已建立，所以我们之后的配置是不生效的。

# 基础
 虽然在[whistle使用实践（实例篇）]中我会按照具体的使用场景来详细介绍whistle的使用方法，但是在此之前，我们有必要对whistle控制台的功能划分和whisle的配置方式做一下简单的了解。

## 控制台

whistle控制台的打开方式上文我们已经说过了，这里不再重复。
whistle控制台核心部分的分区如下（[whistle界面详细列表点这里]（http://wproxy.org/whistle/webui/））：
![控制台划分](https://img10.360buyimg.com/imagetools/jfs/t1/148002/26/19653/283807/5fe1ad39E16b9900c/30e8246cd6654f05.jpg)

1. NetWork: 查看请求响应的详细信息及请求列表的Timeline
2. Rules: 匹配规则，whistle核心，详见下一节配置方式
3. Values: 配置key-value的数据，在Rules里面配置可以通过{key}获取
4. Plugins: 显示所有已安装的插件列表，开启关闭插件功能

## 配置方式

在文章的开头就说过，whistle的所有操作都可以通过配置实现，配置方式扩展于系统hosts配置方式(ip domain或组合方式ip domain1 domain2 domainN)，具有更丰富的匹配模式及更灵活的配置方式。

whistle默认的配置方式是将匹配模式(pattern)写在左边，操作uri(operatorURI)写在右边。这样，whistle会将请求的url与pattern进行匹配，如果匹配上就执行operatorURI对应的操作：

``` js
pattern operatorURI
```
> pattern和operatorURI也可以左右互换（[在这里](http://wproxy.org/whistle/mode.html)），为了行文的清晰，不造成新的混淆，这里只介绍我常用的配置方式，我认为只掌握一种就够了。

我们配置hosts时，如果一个IP要对应多个域名，会这样子写：

``` js
127.0.0.1  www.domain1.com www.domain2.com www.domainN.com
```
和系统hosts一样，如果一个pattern要对应多个操作，whsitle也支持组合方式的配置。使用组合方式时，whistle会按照从左到右的顺序执行operatorURI。

``` js
pattern operatorURI1 operatorURI2 operatorURIN
```

在简单了解了配置方式之后，我们就可以按照```pattern operatorURI```的模式为whsitle添加规则了。
还是再回到whsitle控制台的界面，选中Rules。我们可以像使用SwitchHosts软件管理hosts一样对规则进行分组管理。默认情况下，whistle只有一个Default的分组，如下：
![default分组](https://img14.360buyimg.com/imagetools/jfs/t1/138559/4/19993/221408/5fe313e6Edf676254/32de4c9528b8bba0.jpg)

我们可以点击Create按钮添加一个单品页的分组，在这个分组里可以加上所有与单品页相关的配置（如果要禁用某个配置，可以使用Ctrl + /的快捷键，或者直接在前面加#）
![添加分组](https://img13.360buyimg.com/imagetools/jfs/t1/141428/28/19593/327919/5fe313f4E0b32f1ba/7e624a7cbb37d9bf.jpg)

如果要配置的分组生效，需要双击左侧单品页的tab，出现对号就表示生效了，没有在使用的分组是没有对号的，也可以同时使用多个分组。
![分组生效](https://img13.360buyimg.com/imagetools/jfs/t1/155210/10/11166/324076/5fe313efE0b194f99/f694ed08e697dcb6.jpg)

### 匹配模式pattern

whistle的匹配模式分为以下几种：
1. 域名匹配：域名匹配不仅支持匹配某个域名，也可以限定端口号、协议

``` js
// 匹配www.domain.com域名下的所有请求，包括http、https、ws、wss，tunnel
www.domain.com operatorURI

// 匹配www.domain.com域名下的http请求

http://www.domain.com operatorURI

// 匹配www.domain.com域名下81端口的请求(http请求默认为80端口，https请求默认为443端口)
www.domain.com:81 operatorURI
```

2. 路径匹配：指定匹配某个路径，也可以限定端口号、协议
``` js
// 匹配www.domain.com:81/path路径及其子路径（如www.domain.com:81/path/child）的请求
www.domain.com:81/path operatorURI
```

3. 精确匹配：与上面的路径匹配不同，路径匹配不仅匹配对应的路径，而且还会匹配该路径下面的子路径，而精确匹配只能指定的路径，只要在路径前面加$即可变成精确匹配

``` js
// 匹配www.domain.com:81/path的路径，不包含子路径
$www.domain.com:81/path operatorURI
```

4. 正则匹配：正则的语法及写法跟js的正则表达式一致，支持两种模式：/reg/、/reg/i 忽略大小写，支持子匹配，但不支持/reg/g，且可以通过正则的子匹配把请求url里面的部分字符串传给operatorURI

``` js
// 匹配所有请求
* operatorURI

// 匹配url中包含keyword的请求，且忽略大小写
/keyword/i operatorURI

// 利用子匹配把url里面的参数带到匹配的操作uri
// 下面正则将把请求里面的文件名称，带到匹配的操作uri
// 最多支持10个子匹配 $0...9，其中$0表示整个请求url，其它跟正则的子匹配一样
/[^?#]\/([^\/]+)\.html/ protocol://...$1...
```
5. 通配符匹配：通常，域名匹配和路径匹配可以满足我们大部分的需要，不满足的部分也可以用正则匹配来补充，但正则对大部分人来说还是有门槛的，whistle
很贴心的为我们提供了更简单的通配符匹配方式。目前我还没用过通配符匹配，这里依然简单介绍下，完整通配符匹配：[在这里](http://wproxy.org/whistle/pattern.html)

- 通配符匹配
``` js
// 以 ^ 开头
^www.example.com/test/*** protocol://...$1...

// 限定结束位置
^www.example.com/test/***test$ protocol://...$1...
```
如果请求url为 https://www.example.com/test/abc?123test，这第一个配置 $1 = abc?123&test，第二个配置 $1 = abc?123，而 https://www.example.com/test/abc?123test2 只能匹配第一个。

- 通配域名匹配

``` js
// 匹配以 .com 结尾的所有url，如: test.com, abc.com，但不包含 *.xxx.com
*.com protocol://...$1...
// 匹配 test.com 的子域名，不包括 test.com
// 也不包括诸如 *.xxx.test.com 的四级域名，只能包含: a.test.com，www.test.com 等test.com的三级域名
*.test.com protocol://...$1...

// 如果要配置所有子域名生效，可以使用 **
**.com protocol://...$1...

```

- 通配路径匹配

``` js
// 对所有域名对应的路径 protocol://a.b.c/xxx[/yyy]都生效
*/ 127.0.0.1
```

### 操作值operatorURI

whistle官网将whsitle的操作值分为字符串和JSON对象两种。本文按照配置方式的不同，将whislte的操作值分为两种：带空格的和不带空格的。
- 带空格：带空格的字符串和保留缩进格式的JSON对象
- 不带空格：不带空格的字符串和序列化了的不带空格的JSON对象

不带空格的操作值可以直接在operatorURI中写入，模式为```pattern opProtocol://(strValue)```，注意字符串必须要用括号包裹：

``` js
// 将符合pattern的url的返回内容用helloworld代替
pattern resBody://(helloworld)
```

带空格的操作值需要将操作值保存在Values或者本地文件中。
1. 保存在Values中
在whsitle控制台中打开Values标签，点击Create，增加名称为test.json的操作值，并在右侧编辑test.json的内容，可按照```pattern opProtocol://{valueName}```来使用，注意value名称是用打括号包裹的，如下：

![添加values](https://img13.360buyimg.com/imagetools/jfs/t1/143992/4/19957/246593/5fe4336cEc5199118/8e955d43a329167f.jpg)

``` js
// 将符合pattern的url的返回内容用test.json文件中的内容代替
pattern resBody://{test.json}
```
> 增加的操作值的名称是按照自己的需求取的，后缀名也是非必填的。使用后缀名的话会按照对应的格式高亮展示，不使用的话默认文本格式展示

2. 保存在本地文件中
首先我们先在本地新建一个test1.json的文件，然后在whsitle控制台中点击Files标签
![点击Files](https://img11.360buyimg.com/imagetools/jfs/t1/156829/15/1850/248324/5fe447e0E52edb3cb/b36938896694268c.jpg)

按照步骤选中创建的test1.json文件，whsitle会生成一个path，我们可以按照这个路径来使用
![选中](https://img14.360buyimg.com/imagetools/jfs/t1/150822/27/12260/330508/5fe4483eEab5a4da3/0d2527e32c231d94.jpg)

``` js
// 将符合pattern的url的返回内容用test.json文件中的内容代替
pattern resBody:///$whistle/test1.json
```










 





























