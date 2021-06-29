
---
title: Nginx基础知识
date: 2021-06-29 19:33:00
author: zjk537
tags:
- npm npx
cover: https://m.360buyimg.com/img/jfs/t1/185728/28/11746/24505/60db1000Eec448cd5/5002f2567f7ced8a.jpg
---

## NPX的主要作用
### 调用项目安装的模块
npx主要解决的问题： 调用项目内部安装的模块; 如测试工具 Mocha
```bash
$ npm install -D mocha

# 未使用npx 项目的根目录下执行
$ node-modules/.bin/mocha --version

# 使用npx 就方便多了
$ npx mocha --version

```

npx 的原理很简单，就是运行的时候，会到node_modules/.bin路径和环境变量$PATH里面，检查命令是否存在。

由于 npx 会检查环境变量$PATH，所以系统命令也可以调用。

```bash
# 等同于 ls
$ npx ls
```
注意，Bash 内置的命令不在`$PATH`里面，所以不能用。比如，`cd`是 Bash 命令，因此就不能用`npx cd`
### 避免全局安装模块
```bash
# npx 将create-react-app下载到一个临时目录，使用以后再删除。所以，以后再次执行上面的命令，会重新下载create-react-app
$ npx create-react-app my-react-app

# 安装指定版本 
$ npx uglify-js@3.1.0 main.js -o ./dist/main.js

# 注意，只要 npx 后面的模块无法在本地发现，就会下载同名模块
# 本地没有安装http-server模块，下面的命令会自动下载该模块，在当前目录启动一个 Web 服务。
$ npx http-server

# --no-install 强制使用本地模块，不下载远程模块, 如果不存在就报错
$ npx --no-install http-server

# --ignore-existing 忽略本地的同名模块，强制安装使用远程模块
$ npx --ignore-existing create-react-app my-react-app
```

### 使用不同版本的node

```js
$ npx node@0.12.8 -v
v0.12.8
```
使用 0.12.8 版本的 Node 执行脚本。原理是从 npm 下载这个版本的 node，使用后再删掉。
某些场景下，这个方法用来切换 Node 版本，要比 nvm 那样的版本管理器方便一些。

#### -p 参数
`-p`参数用于指定 npx 所要安装的模块，所以上面的命令可以写成下面这样。
```bash
$ npx -p node@0.12.8 node -v 
v0.12.8
```
上面命令先指定安装`node@0.12.8`，然后再执行`node -v`命令。

#### -c 参数
如果 npx 安装多个模块，默认情况下，所执行的命令之中，只有第一个可执行项会使用 npx 安装的模块，后面的可执行项还是会交给 Shell 解释。
```bash
$ npx -p lolcatjs -p cowsay 'cowsay hello | lolcatjs'
# 报错
```
上面代码中，`cowsay hello | lolcatjs`执行时会报错，原因是第一项`cowsay`由 npx 解释，而第二项命令`localcatjs`由 Shell 解释，但是`lolcatjs`并没有全局安装，所以报错。
`-c`参数可以将所有命令都用 npx 解释。有了它，下面代码就可以正常执行了。
```bash
$ npx -p lolcatjs -p cowsay -c 'cowsay hello | lolcatjs'
```
-c参数的另一个作用，是将环境变量带入所要执行的命令。举例来说，npm 提供当前项目的一些环境变量，可以用下面的命令查看。
```bash
$ npm run env | grep npm_
```
-c参数可以把这些 npm 的环境变量带入 npx 命令。
```bash
$ npx -c 'echo "$npm_package_name"'
# 输出当前项目的项目名
```
