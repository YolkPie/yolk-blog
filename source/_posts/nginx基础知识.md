---
title: Nginx基础知识
date: 2021-03-25 19:33:00
author: zjk537
cover: https://m.360buyimg.com/img/jfs/t1/156694/16/17976/4358/605c7d52Edbdb2a53/b95b369946c69565.webp
---
![Nginx](nginx.webp "Nginx")
## Nginx 学习手册
什么是Nginx
> 简单说： Nginx 是个神奇的 Web 服务器
> 专业说：Nginx 是一个采用主从架构的 Web 服务器，可用于反向代理、负载均衡器、邮件代理和 HTTP 缓存。

Nginx 是一个高性能的 HTTP 和反向代理服务器，特点是占用内存少，并发能力强，事实上 Nginx 的并发能力确实在同类型的网页服务器中表现较好。Nginx 专为性能优化而开发，性能是其最重要的要求，十分注重效率，有报告 Nginx 能支持高达 50000 个并发连接数。


### 1、Nginx 知识网结构图
![Nginx 知识网结构图](struct.jpg "Nginx 知识网结构图")

### 2、Nginx专业术语
![Nginx 专业术语](640.gif "Nginx 专业术语")
Nginx 的基本特性是代理，所以你一定要明白什么是代理和反向代理。
#### 正向代理与反向代理
##### 正向代理
局域网中的电脑用户想要直接访问网络是不可行的，只能通过代理服务器来访问，这种代理服务就被称为正向代理。

![Nginx 正向代理](proxy.jpg "Nginx 正向代理")
> 通俗讲：client1 和 client2 通过代理服务器向服务器发送请求 request1 和 request2，此时后端服务器不知道 request1 是由 client1 发送的还是 client2 发送的，但会执行（响应）操作。
**服务器**不知道请求**谁发起的**


##### 反向代理
客户端无法感知代理，因为客户端访问网络不需要配置，只要把请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据，然后再返回到客户端。
此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。

![Nginx 反向代理](r-proxy.jpg "Nginx 反向代理")
> 通俗讲：客户端将通过 Web 服务器发送请求. 而 Web 服务器会通过一个算法，当中最有意思的算法是轮询，直接将请求指向许多后端服务器中的一个，并通过 Web 服务器将响应返回给客户端。因此，在上面的例子中，客户端其实并不知道在与哪个后端服务器进行交互。
**客户端**不知道请求**是谁处理的**

#### 负载均衡
又是枯燥的一个名词：负载均衡，不过它很好理解，因为负载均衡本身就是反向代理的一个实例。

**有状态应用**
![有状态应用](has-status.webp "有状态应用")
> 如图所示，一个后端服务器 server1 存储了一些信息，服务器 server2 并不存储此信息，因此，客户端 (上图 Bob) 的交互可能会也可能不会得到想要的结果，因为它可能会与 server1 或 server2 交互。在本例中，server1 允许 Bob 查看数据文件，但 server2 不允许。因此，虽然有状态应用避免对数据库的多次 API 调用，并且（响应）速度更快，但它可能会在不同的服务器上导致这个（无法得到想要结果）问题。

**无状态应用**
![无状态应用](no-status.webp "无状态应用")
> 如果我通过 Web 服务器从客户端向后端服务器 server1 发送请求，它将向客户端返回一个令牌，用于任何进一步的访问请求。客户端可以使用令牌并向 Web 服务器发送请求。此时 Web 服务器将请求连同令牌一起发送到任意后端服务器，而每个后端服务器都能提供相同的所需结果。

**负载均衡与反向代理的区别**
>- 负载均衡: 必须有 2 个或者更多的后端服务器
>- 反向代理: 一台后端服务器也能运作

客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服务器处理完毕之后，再将结果返回给客户端。
- 普通请求和响应过程如下图：
![Nginx 反向代理](request.webp "Nginx 反向代理")

但是随着信息数量增长，访问量和数据量飞速增长，普通架构无法满足现在的需求。

我们首先想到的是升级服务器配置，可以由于`摩尔定律`的日益失效，单纯从硬件提升性能已经逐渐不可取了，怎么解决这种需求呢？

我们可以增加服务器的数量，构建集群，将请求分发到各个服务器上，将原来请求集中到单个服务器的情况改为请求分发到多个服务器，也就是我们说的负载均衡。

- 负载均衡
![Nginx 负载均衡](balance.jpg "Nginx 负载均衡")

假设有 15 个请求发送到代理服务器，那么由代理服务器根据服务器数量，平均分配，每个服务器处理 5 个请求，这个过程就叫做负载均衡。

#### 动静分离
为了加快网站的解析速度，可以把动态页面和静态页面交给不同的服务器来解析，加快解析的速度，降低由单个服务器的压力。
- 动静分离前
![Nginx 动静分离前](before-split.jpg "Nginx 动静分离前")
- 动静分离后
![Nginx 动静分离后](after-split.jpg "Nginx 动静分离后")


### 3、Nginx安装
#### Windows 安装
下载地址：http://nginx.org/en/download.html
下载后，解压，直接运行nginx.exe即可
#### Linux 安装
```bash
$ yum install nginx 
```

#### Mac 安装
```bash
# 安装
$ brew install Nginx


# 启动
$ nginx 
# OR 
$ sudo nginx
# 默认8080端口 http://localhost:8080/

# 查看帮助
$ nginx -h

# 设置nginx.conf 文件
$ nginx -c file name 
# brew 安装默认位置： /opt/homebrew/etc/nginx
```
### 4、Nginx配置
场景一： 我们将在一个公共端口上运行两个文件夹，并设置我们想要的规则
本地目录结构
```js
├── nginx-demo
│  ├── content
│  │  ├── first.txt
│  │  ├── index.html
│  │  └── index.md
│  └── main
│    └── index.html
└── temp-nginx
  └── outsider
    └── index.html
```
**1、添加配置的基本设置**
一定要添加 `events {}`，因为在 Nginx 架构中，它通常用来表示 worker 的数量。
用 http 告诉 Nginx 我们将在 OSI 模型 的第 7 层作业。

我们告诉 Nginx 监听 5000 端口，并指向 main 文件夹中的静态文件。
```js
events {
  worker_connections  1024;
}

http {

  server {
    listen 5000;
    root /path/to/nginx-demo/main/; 
  }

}
```
**2、添加其他规则**
接下来我们将为 `/content` 和 `/outsider` URL 添加其他的规则，其中 **outsider** 将指向第一步中提到的根目录之外的目录。

这里的 `location /content`  表示无论我在叶（leaf）目录中定义了什么根（root），**content** 子 URL 都会被添加到定义的根 URL 的末尾。因此，当我指定 root 为 `root /path/to/nginx-demo/`时，这仅仅意味着我告诉 Nginx 在 `http://localhost:5000/path/to/nginx-demo/content/` 文件夹中显示静态文件的内容。
```js
events {
  worker_connections  1024;
}
http {
  server {
    listen 5000;
    root /path/to/nginx-demo/main/; 

    location /content {
      root /path/to/nginx-demo/;
    }   
    location /outsider {
      root /path/temp-nginx/;
    }
  }
}
```
> **现在 Nginx 不仅能定义 URL 根路径，还可以设置规则，这样我们就能阻止客户端访问某个文件了。**

**3、限制访问**
接下来，我们在主服务器上编写一个规则来防止任意 .md 文件被访问。我们可以在 Nginx 中使用正则表达式，因此我们将这样定义规则：
```js
location ~ .md {
  return 403;
}
```
**4、来看看代理 proxy_pass**
我们已经在5000端口上运行了服务，再起一个在8888端口上的服务,这样我们就有两个服务了。

我们要做的是：当客户端通过 Nginx 访问 8888 端口时，将这个请求传到 5000 端口，并将响应返回给客户端！
```js
server {
  listen 8888;
  location / {
    proxy_pass http://localhost:5000/;
  }
  location /new {
    proxy_pass http://localhost:5000/outsider/;
  }
}
```
**5、负载均衡**
```bash
upstream myserver{
  #ip_hash
  server 127.0.0.1:5000 weight=1;
  server 127.0.0.1:8888 weight=2;
  #fair; 
}

server {
  listen 8000;
  location / {
    proxy_pass http://myserver;
  }
}
```
**负载均衡方式如下：**
> 轮询（默认）。
weight，代表权，权越高优先级越高。
fair，按后端服务器的响应时间来分配请求，相应时间短的优先分配。
ip_hash，每个请求按照访问 ip 的 hash 结果分配，这样每一个访客固定的访问一个后端服务器，可以解决 Session 的问题。

**6、动静分离实战**
![Nginx 动静分离后](after-split.webp "Nginx 动静分离后")
> 纯粹将静态文件独立成单独域名放在独立的服务器上，也是目前主流方案。
将动态跟静态文件混合在一起发布，通过 Nginx 分开
```bash
location /image/ {
  root /path/to/images/;
} 
```

看一下所有配置

```bash
events {
  worker_connections  1024;
}

http {
  server {
    listen 5000;
    root /path/to/nginx-demo/main/; 

    location /content {
      root /path/to/nginx-demo/;
    }   
    location /outsider {
      root /path/temp-nginx/;
    }
    location ~ .md {
      return 403;
    }
  }

  server {
    listen 8888;
    location / {
      proxy_pass http://localhost:5000/;
    }
    location /new {
      proxy_pass http://localhost:5000/outsider/;
    }
  }

  # 4、负载均衡
  server {
    listen 8000;
    location / {
      proxy_pass http://myserver;
    }
  }
}

upstream myserver{
  #ip_hash
  server 127.0.0.1:5000 weight=1;
  server 127.0.0.1:8888 weight=2;
  #fair; 
}
```
来把服务启起来
```bash
$ nginx
# OR
$ sudo nginx
```
**Nginx的其他命令**

- 1、首次启动 Nginx Web 服务器。
```bash
$ nginx 
# OR 
$ sudo nginx
```
- 2、重新加载正在运行的 Nginx Web 服务器。
```bash
$ nginx -s reload
# OR 
$ sudo nginx -s reload
```
- 3、停止正在运行中的 Nginx Web 服务器。
```bash
$ nginx -s stop
# OR 
$ sudo nginx -s stop
```
- 4、查看系统上运行的 Nginx 进程。
```bash
$ ps -ef | grep Nginx
```
第 4 条命令很重要，如果前 3 条命令产生了一些问题，通常你可以用第 4 条命令找到所有正在运行的 Nginx 进程并杀死进程，然后重新启动它们。

要杀死一个进程，你需要 PID，再用以下命令杀死它：
```bash
$ kill -9 <PID>
# OR 
$ sudo kill -9 <PID>
```

**location 指令说明，该语法用来匹配 url，语法如下：**
> =：用于不含正则表达式的 url 前，要求字符串与 url 严格匹配，匹配成功就停止向下搜索并处理请求。
~：用于表示 url 包含正则表达式，并且区分大小写。
~*：用于表示 url 包含正则表达式，并且不区分大小写。
^~：用于不含正则表达式的 url 前，要求 Nginx 服务器找到表示 url 和字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再匹配。
如果有 url 包含正则表达式，不需要有 ~ 开头标识。
本机演示命令
```js
sudo vim /opt/homebrew/etc/nginx/nginx.conf
```
摩尔定律
每18到24个月，集成电路上可容纳的元器件数目便会增加一倍，芯片的性能也会随之翻一番