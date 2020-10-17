## 四、CLI 原理及不同端的运行机制

### 4.1 taro-cli 包

#### 4.1.1 Taro 命令

> taro-cli 包位于 Taro 工程的 Packages 目录下，通过 npm install -g @tarojs/cli 全局安装后，将会生成一个 Taro 命令。主要负责项目初始化、编译、构建等。直接在命令行输入 Taro ，会看到如下提示…

```js
➜ taro
👽 Taro v0.0.63


  Usage: taro <command> [options]

  Options:

    -V, --version       output the version number
    -h, --help          output usage information

  Commands:

    init [projectName]  Init a project with default templete
    build               Build a project with options
    update              Update packages of taro
    help [cmd]          display help for [cmd]...
```

里面包含了 Taro 所有命令用法及作用。

#### 4.1.2 包管理与发布

- 首先，我们需要了解 taro-cli 包与 Taro 工程的关系。
- 将 Taro 工程 Clone 之后，可以看到工程的目录结构如下，整体结构还是比较清晰的：

```js
.
├── CHANGELOG.md
├── LICENSE
├── README.md
├── build
├── docs
├── lerna-debug.log
├── lerna.json        // Lerna 配置文件
├── package.json
├── packages
│   ├── eslint-config-taro
│   ├── eslint-plugin-taro
│   ├── postcss-plugin-constparse
│   ├── postcss-pxtransform
│   ├── taro
│   ├── taro-async-await
│   ├── taro-cli
│   ├── taro-components
│   ├── taro-components-rn
│   ├── taro-h5
│   ├── taro-plugin-babel
│   ├── taro-plugin-csso
│   ├── taro-plugin-sass
│   ├── taro-plugin-uglifyjs
│   ├── taro-redux
│   ├── taro-redux-h5
│   ├── taro-rn
│   ├── taro-rn-runner
│   ├── taro-router
│   ├── taro-transformer-wx
│   ├── taro-weapp
│   └── taro-webpack-runner
└── yarn.lock...
```

> Taro 项目主要是由一系列 NPM 包组成，位于工程的 Packages 目录下。它的包管理方式和 Babel 项目一样，将整个项目作为一个 monorepo 来进行管理，并且同样使用了包管理工具 Lerna

Packages 目录下十几个包中，最常用的项目初始化与构建的命令行工具 Taro CLI 就是其中一个。在 Taro 工程根目录运行 lerna publish 命令之后，lerna.json 里面配置好的所有的包会被发布到 NPM 上

#### 4.1.3 taro-cli 包的目录结构如下

```js
./
├── bin        // 命令行
│   ├── taro              // taro 命令
│   ├── taro-build        // taro build 命令
│   ├── taro-update       // taro update 命令
│   └── taro-init         // taro init 命令
├── package.json
├── node_modules
├── src
│   ├── build.js        // taro build 命令调用，根据 type 类型调用不同的脚本
│   ├── config
│   │   ├── babel.js        // Babel 配置
│   │   ├── babylon.js      // JavaScript 解析器 babylon 配置
│   │   ├── browser_list.js // autoprefixer browsers 配置
│   │   ├── index.js        // 目录名及入口文件名相关配置
│   │   └── uglify.js
│   ├── creator.js
│   ├── h5.js       // 构建h5 平台代码
│   ├── project.js  // taro init 命令调用，初始化项目
│   ├── rn.js       // 构建React Native 平台代码
│   ├── util        // 一系列工具函数
│   │   ├── index.js
│   │   ├── npm.js
│   │   └── resolve_npm_files.js
│   └── weapp.js        // 构建小程序代码转换
├── templates           // 脚手架模版
│   └── default
│       ├── appjs
│       ├── config
│       │   ├── dev
│       │   ├── index
│       │   └── prod
│       ├── editorconfig
│       ├── eslintrc
│       ├── gitignor...
```

> 通过上面的目录树可以发现，taro-cli 工程的文件并不算多，主要目录有：/bin、/src、/template

### 4.2 用到的核心库

- tj/commander.js Node.js - 命令行接口全面的解决方案
- jprichardson/node-fs-extra - 在 Node.js 的 fs 基础上增加了一些新的方法，更好用，还可以拷贝模板。
- chalk/chalk - 可以用于控制终端输出字符串的样式
- SBoudrias/Inquirer.js - Node.js 命令行交互工具，通用的命令行用户界面集合，可以和用户进行交互
- sindresorhus/ora - 实现加载中的状态是一个 Loading 加前面转起来的小圈圈，成功了是一个 Success 加前面一个小钩钩
- SBoudrias/mem-fs-editor - 提供一系列 API，方便操作模板文件
- shelljs/shelljs - ShellJS 是 Node.js 扩展，用于实现 Unix shell 命令执行。

### 4.3 Taro Init

![](https://m.360buyimg.com/img/jfs/t1/88040/24/7314/51974/5dfb311bEef964024/24ef0e40d268b7fc.png)

> 当我们全局安装 taro-cli 包之后，我们的命令行里就有了 Taro 命令

- 那么 Taro 命令是怎样添加进去的呢？其原因在于 package.json 里面的 bin 字段：

```js
"bin": {
    "taro": "bin/taro"
}
```

上面代码指定，Taro 命令对应的可执行文件为 bin/taro 。NPM 会寻找这个文件，在 [prefix]/bin 目录下建立符号链接。在上面的例子中，Taro 会建立符号链接 [prefix]/bin/taro。由于 [prefix]/bin 目录会在运行时加入系统的 PATH 变量，因此在运行 NPM 时，就可以不带路径，直接通过命令来调用这些脚本。

- 关于 prefix，可以通过 npm config get prefix 获取。

```js
$ npm config get prefix
/usr/local
```

通过下列命令可以更加清晰的看到它们之间的符号链接…

```js
$ ls -al `which taro`
lrwxr-xr-x  1 chengshuai  admin  40  6 15 10:51 /usr/local/bin/taro -> ../lib/node_modules/@tarojs/cli/bin/taro...
```

#### 4.3.1 命令关联与参数解析

> 这里就不得不提到一个有用的包：tj/commander.js ，Node.js 命令行接口全面的解决方案，灵感来自于 Ruby’s commander。可以自动的解析命令和参数，合并多选项，处理短参等等，功能强大，上手简单

更主要的，commander 支持 Git 风格的子命令处理，可以根据子命令自动引导到以特定格式命名的命令执行文件，文件名的格式是 [command]-[subcommand]，例如

- taro init => taro-init
- taro build => taro-build
- /bin/taro 文件内容不多，核心代码也就那几行 .command() 命令：

```js
#! /usr/bin/env node

const program = require('commander')
const {getPkgVersion} = require('../src/util')

program
  .version(getPkgVersion())
  .usage('<command> [options]')
  .command('init [projectName]', 'Init a project with default templete')
  .command('build', 'Build a project with options')
  .command('update', 'Update packages of taro')
  .parse(process.argv)...
```

> 通过上面代码可以发现，init，build ，update 等命令都是通过.command(name, description)方法定义的，然后通过 .parse(arg) 方法解析参数

#### 4.3.2 参数解析及与用户交互

- commander 包可以自动解析命令和参数，在配置好命令之后，还能够自动生成 help（帮助）命令和 version（版本查看） 命令。并且通过 program.args 便可以获取命令行的参数，然后再根据参数来调用不同的脚本。
- 但当我们运行 taro init 命令后，如下所示的命令行交互又是怎么实现的呢？…

```js
$ taro init taroDemo
Taro 即将创建一个新项目!
Need help? Go and open issue: https://github.com/NervJS/taro/issues/new

Taro v0.0.50

? 请输入项目介绍！
? 请选择模板 默认模板...
```

这里使用的是 SBoudrias/Inquirer.js 来处理命令行交互。

用法其实很简单

```js
const inquirer = require('inquirer')  // npm i inquirer -D

if (typeof conf.description !== 'string') {
      prompts.push({
        type: 'input',
        name: 'description',
        message: '请输入项目介绍！'
      })
}...
```

- prompt()接受一个问题对象的数据，在用户与终端交互过程中，将用户的输入存放在一个答案对象中，然后返回一个 Promise，通过 then()获取到这个答案对象。
  借此，新项目的名称、版本号、描述等信息可以直接通过终端交互插入到项目模板中，完善交互流程。
- 当然，交互的问题不仅限于此，可以根据自己项目的情况，添加更多的交互问题。inquirer.js 强大的地方在于，支持很多种交互类型，除了简单的 input，还有 confirm、list、password、checkbox 等，具体可以参见项目的工程 README。
  此外，你在执行异步操作的过程中，还可以使用 sindresorhus/ora 来添加一下 Loading 效果。使用 chalk/chalk 给终端的输出添加各种样式…

#### 4.3.3 模版文件操作

最后就是模版文件操作了，主要分为两大块：

- 将输入的内容插入到模板中
- 根据命令创建对应目录结构，copy 文件
- 更新已存在文件内容

> 这些操作基本都是在 /template/index.js 文件里。
> 这里还用到了 shelljs/shelljs 执行 shell 脚本，如初始化 Git： git init，项目初始化之后安装依赖 npm install 等

拷贝模板文件

拷贝模版文件主要是使用 jprichardson/node-fs-extra 的 copyTpl()方法，此方法使用 ejs 模板语法，可以将输入的内容插入到模版的对应位置：

```js
this.fs.copyTpl(
  project,
  path.join(projectPath, 'project.config.json'),
  {description, projectName}
);...
```

### 4.4 Taro Build

- taro build 命令是整个 Taro 项目的灵魂和核心，主要负责多端代码编译（H5，小程序，React Native 等）。
- Taro 命令的关联，参数解析等和 taro init 其实是一模一样的，那么最关键的代码转换部分是怎样实现的呢？…

#### 4.4.1 编译工作流与抽象语法树（AST）

> Taro 的核心部分就是将代码编译成其他端（H5、小程序、React Native 等）代码。一般来说，将一种结构化语言的代码编译成另一种类似的结构化语言的代码包括以下几个步骤

![](https://m.360buyimg.com/img/jfs/t1/96906/16/7374/30011/5dfb338eE70cc9fd9/efe8147142d6aa89.png)

首先是 Parse，将代码解析（Parse）成抽象语法树（Abstract Syntex Tree），然后对 AST 进行遍历（traverse）和替换(replace)（这对于前端来说其实并不陌生，可以类比 DOM 树的操作），最后是生成（generate），根据新的 AST 生成编译后的代码…

#### 4.4.2 Babel 模块

Babel 是一个通用的多功能的 JavaScript 编译器，更确切地说是源码到源码的编译器，通常也叫做转换编译器（transpiler）。 意思是说你为 Babel 提供一些 JavaScript 代码，Babel 更改这些代码，然后返回给你新生成的代码…

#### 4.4.3 解析页面 Config 配置

> 在业务代码编译成小程序的代码过程中，有一步是将页面入口 JS 的 Config 属性解析出来，并写入 \*.json 文件，供小程序使用。那么这一步是怎么实现的呢？这里将这部分功能的关键代码抽取出来：

```js
// 1. babel-traverse方法， 遍历和更新节点
traverse(ast, {
  ClassProperty(astPath) { // 遍历类的属性声明
    const node = astPath.node
    if (node.key.name === 'config') { // 类的属性名为 config
      configObj = traverseObjectNode(node)
      astPath.remove() // 将该方法移除掉
    }
  }
})

// 2. 遍历，解析为 JSON 对象
function traverseObjectNode(node, obj) {
  if (node.type === 'ClassProperty' || node.type === 'ObjectProperty') {
    const properties = node.value.properties
      obj = {}
      properties.forEach((p, index) => {
        obj[p.key.name] = traverseObjectNode(p.value)
      })
      return obj
  }
  if (node.type === 'ObjectExpression') {
    const properties = node.properties
    obj = {}
    properties.forEach((p, index) => {
      // const t = require('babel-types')  AST 节点的 Lodash 式工具库
      const key = t.isIdentifier(p.key) ? p.key.name : p.key.value
      obj[key] = traverseObjectNode(p.value)
    })
    return obj
  }
  if (node.type === 'ArrayExpression') {
    return node.elements.map(item => traverseObjectNode(item))
 ...
```

## 五、Taro 组件库及 API 的设计与适配

### 5.1 多端差异

#### 5.1.1 组件差异

小程序、H5 以及快应用都可以划分为 XML 类，React Native 归为 JSX 类，两种语言风牛马不相及，给适配设置了非常大的障碍。XML 类有个明显的特点是关注点分离（Separation of Concerns），即语义层（XML）、视觉层（CSS）、交互层（JavaScript）三者分离的松耦合形式，JSX 类则要把三者混为一体，用脚本来包揽三者的工作…

不同端的组件的差异还体现在定制程度上

- H5 标签（组件）提供最基础的功能——布局、表单、媒体、图形等等；
- 小程序组件相对 H5 有了一定程度的定制，我们可以把小程序组件看作一套类似于 H5 的 UI 组件库；
- React Native 端组件也同样如此，而且基本是专“组”专用的，比如要触发点击事件就得用 Touchable 或者 Text 组件，要渲染文本就得用 Text 组件（虽然小程序也提供了 Text 组件，但它的文本仍然可以直接放到 view 之类的组件里）…

#### 5.1.2 API 差异

各端 API 的差异具有定制化、接口不一、能力限制的特点

- 定制化：各端所提供的 API 都是经过量身打造的，比如小程序的开放接口类 API，完全是针对小程序所处的微信环境打造的，其提供的功能以及外在表现都已由框架提供实现，用户上手可用，毋须关心内部实现。
- 接口不一：相同的功能，在不同端下的调用方式以及调用参数等也不一样，比如 socket，小程序中用 wx.connectSocket 来连接，H5 则用 new WebSocket() 来连接，这样的例子我们可以找到很多个。
- 能力限制：各端之间的差异可以进行定制适配，然而并不是所有的 API（此处特指小程序 API，因为多端适配是向小程序看齐的）在各个端都能通过定制适配来实现，因为不同端所能提供的端能力“大异小同”，这是在适配过程中不可抗拒、不可抹平的差异…

### 5.2 多端适配

#### 5.2.1 样式处理

H5 端使用官方提供的 WEUI 进行适配，React Native 端则在组件内添加样式，并通过脚本来控制一些状态类的样式，框架核心在编译的时候把源代码的 class 所指向的样式通过 css-to-react-native 进行转译，所得 StyleSheet 样式传入组件的 style 参数，组件内部会对样式进行二次处理，得到最终的样式…

![](https://m.360buyimg.com/img/jfs/t1/104718/12/7499/47989/5dfb3fedE9cfe19c7/d29edbfd298253d4.png)

为什么需要对样式进行二次处理？

> 部分组件是直接把传入 style 的样式赋给最外层的 React Native 原生组件，但部分经过层层封装的组件则不然，我们要把容器样式、内部样式和文本样式离析。为了方便解释，我们把这类组件简化为以下的形式：

```js
<View style={wrapperStyle}>
  <View style={containerStyle}>
    <Text style={textStyle}>Hello World</Text>
  </View>
</View>
```

> 假设组件有样式 margin-top、background-color 和 font-size，转译传入组件后，就要把分别把它们传到 wrapperStyle、containerStyle 和 textStyle，可参考 ScrollView 的 style 和 contentContainerStyle…

#### 5.2.2 组件封装

> 组件的封装则是一个“仿制”的过程，利用端提供的原材料，加工成通用的组件，暴露相对统一的调用方式。我们用 <Button /> 这个组件来举例，在小程序端它也许是长这样子的

```js
<button
  size="mini"
  plain={{ plain }}
  loading={{ loading }}
  hover-class="you-hover-me"
></button>
```

> 如果要实现 H5 端这么一个按钮，大概会像下面这样，在组件内部把小程序的按钮特性实现一遍，然后暴露跟小程序一致的调用方式，就完成了 H5 端一个组件的设计

```js
<button
  {...omit(this.props, ['hoverClass', 'onTouchStart', 'onTouchEnd'])}
  className={cls}
  style={style}
  onClick={onClick}
  disabled={disabled}
  onTouchStart={_onTouchStart}
  onTouchEnd={_onTouchEnd}
>
  {loading && <i class='weui-loading' />}
  {children}
</button>...
```

- 其他端的组件适配相对 H5 端来说会更曲折复杂一些，因为 H5 跟小程序的语言较为相似，而其他端需要整合特定端的各种组件，以及利用端组件的特性来实现，比如在 React Native 中实现这个按钮，则需要用到 <Touchable\* />、<View />、<Text />，要实现动画则需要用上 <Animated.View />，还有就是相对于 H5 和小程序比较容易实现的 touch 事件，在 React Native 中则需要用上 PanResponder 来进行“仿真”，总之就是，因“端”制宜，一切为了最后只需一行代码通行多端！
- 除了属性支持外，事件回调的参数也需要进行统一，为此，需要在内部进行处理，比如 Input 的 onInput 事件，需要给它造一个类似小程序相同事件的回调参数，比如 { target: { value: text }, detail: { value: text } }，这样，开发者们就可以像下面这样处理回调事件，无需关心中间发生了什么…

```js
function onInputHandler({ target, detail }) {
  console.log(target.value, detail.value);
}
```

## 六、JSX 转换微信小程序模板的实现

### 6.1 代码的本质

> 不管是任意语言的代码，其实它们都有两个共同点

- 它们都是由字符串构成的文本
- 它们都要遵循自己的语言规范

第一点很好理解，既然代码是字符串构成的，我们要修改/编译代码的最简单的方法就是使用字符串的各种正则表达式。例如我们要将 JSON 中一个键名 foo 改为 bar，只要写一个简单的正则表达式就能做到：

```js
jsonStr.replace(/(?<=")foo(?="\s*:)/i, 'bar')...
```

> 编译就是把一段字符串改成另外一段字符串

### 6.2 Babel

> JavaScript 社区其实有非常多 parser 实现，比如 Acorn、Esprima、Recast、Traceur、Cherow 等等。但我们还是选择使用 Babel，主要有以下几个原因

- Babel 可以解析还没有进入 ECMAScript 规范的语法。例如装饰器这样的提案，虽然现在没有进入标准但是已经广泛使用有一段时间了；
- Babel 提供插件机制解析 TypeScript、Flow、JSX 这样的 JavaScript 超集，不必单独处理这些语言；
- Babel 拥有庞大的生态，有非常多的文档和样例代码可供参考；
- 除去 parser 本身，Babel 还提供各种方便的工具库可以优化、生成、调试代码…

Babylon（ @babel/parser）

> Babylon 就是 Babel 的 parser。它可以把一段符合规范的 JavaScript 代码输出成一个符合 Esprima 规范的 AST。 大部分 parser 生成的 AST 数据结构都遵循 Esprima 规范，包括 ESLint 的 parser ESTree。这就意味着我们熟悉了 Esprima 规范的 AST 数据结构还能去写 ESLint 插件

我们可以尝试解析 n \* n 这句简单的表达式：

```js
import * as babylon from "babylon";
const code = `n * n`;
babylon.parse(code);...
```

最终 Babylon 会解析成这样的数据结构：
![](https://m.360buyimg.com/img/jfs/t1/85776/36/7301/115301/5dfb42c2Eccb38a5f/aa0004a25a7ee488.png)
你也可以使用 ASTExploroer 快速地查看代码的 AST

Babel-traverse (@babel/traverse)

> babel-traverse 可以遍历由 Babylon 生成的抽象语法树，并把抽象语法树的各个节点从拓扑数据结构转化成一颗路径（Path）树，Path 表示两个节点之间连接的响应式（Reactive）对象，它拥有添加、删除、替换节点等方法。当你调用这些修改树的方法之后，路径信息也会被更新。除此之外，Path 还提供了一些操作作用域（Scope） 和标识符绑定（Identifier Binding） 的方法可以去做处理一些更精细复杂的需求。可以说 babel-traverse 是使用 Babel 作为编译器最核心的模块…

让我们尝试一下把一段代码中的 n _ n 变为 x _ x

```js
import * as babylon from "@babel/parser";
import traverse from "babel-traverse";

const code = `function square(n) {
  return n * n;
}`;

const ast = babylon.parse(code);

traverse(ast, {
  enter(path) {
    if (
      path.node.type === "Identifier" &&
      path.node.name === "n"
    ) {
      path.node.name = "x";
    }
  }
});...
```

Babel-types（@babel/types）

> babel-types 是一个用于 AST 节点的 Lodash 式工具库，它包含了构造、验证以及变换 AST 节点的方法。 该工具库包含考虑周到的工具方法，对编写处理 AST 逻辑非常有用。例如我们之前在 babel-traverse 中改变标识符 n 的代码可以简写为：

```js
import traverse from "babel-traverse";
import * as t from "babel-types";

traverse(ast, {
  enter(path) {
    if (t.isIdentifier(path.node, { name: "n" })) {
      path.node.name = "x";
    }
  },
});
```

> 可以发现使用 babel-types 能提高我们转换代码的可读性，在配合 TypeScript 这样的静态类型语言后，babel-types 的方法还能提供类型校验的功能，能有效地提高我们转换代码的健壮性和可靠性…

### 6.3 实践例子

以一个简单 Page 页面为例：

```js
import Taro, { Component } from '@tarojs/taro'
import { View, Text } from '@tarojs/components'

class Home extends Component {

  config = {
    navigationBarTitleText: '首页'
  }

  state = {
    numbers: [1, 2, 3, 4, 5]
  }

  handleClick = () => {
    this.props.onTest()
  }

  render () {
    const oddNumbers = this.state.numbers.filter(number => number & 2)
    return (
      <ScrollView className='home' scrollTop={false}>
        奇数：
        {
          oddNumbers.map(number => <Text onClick={this.handleClick}>{number}</Text>)
        }
        偶数：
        {
          numbers.map(number => number % 2 === 0 && <Text onClick={this.handleClick}>{number}</Text>)
        }
      </ScrollView>
    )
  }
}...
```

#### 6.3.1 设计思路

- Taro 的结构主要分两个方面：运行时和编译时。运行时负责把编译后到代码运行在本不能运行的对应环境中，你可以把 Taro 运行时理解为前端开发当中 polyfill。举例来说，小程序新建一个页面是使用 Page 方法传入一个字面量对象，并不支持使用类。如果全部依赖编译时的话，那么我们要做到事情大概就是把类转化成对象，把 state 变为 data，把生命周期例如 componentDidMount 转化成 onReady，把事件由可能的类函数（Class method）和类属性函数(Class property function) 转化成字面量对象方法（Object property function）等等。
- 但这显然会让我们的编译时工作变得非常繁重，在一个类异常复杂时出错的概率也会变高。但我们有更好的办法：实现一个 createPage 方法，接受一个类作为参数，返回一个小程序 Page 方法所需要的字面量对象。这样不仅简化了编译时的工作，我们还可以在 createPage 对编译时产出的类做各种操作和优化。通过运行时把工作分离了之后，再编译时我们只需要在文件底部加上一行代码 Page(createPage(componentName)) 即可…

![](https://m.360buyimg.com/img/jfs/t1/85776/36/7301/115301/5dfb42c2Eccb38a5f/aa0004a25a7ee488.png)

- 回到一开始那段代码，我们定义了一个类属性 config，config 是一个对象表达式（Object Expression），这个对象表达式只接受键值为标识符（Identifier）或字符串，而键名只能是基本类型。这样简单的情况我们只需要把这个对象表达式转换为 JSON 即可。另外一个类属性 state 在 Page 当中有点像是小程序的 data，但它在多数情况不是完整的 data。这里我们不用做过多的操作，babel 的插件 transform-class-proerties 会把它编译到类的构造器中。函数 handleClick 我们交给运行时处理，有兴趣的同学可以跳到 Taro 运行时原理查看具体技术细节。
- 再来看我们的 render()函数，它的第一行代码通过 filter 把数字数组的所有偶数项都过滤掉，真正用来循环的是 oddNumbers，而 oddNumbers 并没有在 this.state 中，所以我们必须手动把它加入到 this.state。和 React 一样，Taro 每次更新都会调用 render 函数，但和 React 不同的是，React 的 render 是一个创建虚拟 DOM 的方法，而 Taro 的 render 会被重命名为 \_createData，它是一个创建数据的方法：在 JSX 使用过的数据都在这里被创建最后放到小程序 Page 或 Component 工厂方法中的 data。最终我们的 render 方法会被编译为…

```js
_createData() {
  this.__state = arguments[0] || this.state || {};
  this.__props = arguments[1] || this.props || {};

  const oddNumbers = this.__state.numbers.filter(number => number & 2);
  Object.assign(this.__state, {
    oddNumbers: oddNumbers
  });
  return this.__state;
}...
```

#### 6.3.2 WXML 和 JSX

在 Taro 里 render 的所有 JSX 元素都会在 JavaScript 文件中被移除，它们最终将会编译成小程序的 WXML。每个 WXML 元素和 HTML 元素一样，我们可以把它定义为三种类型：Element、Text、Comment。其中 Text 只有一个属性: 内容（content），它对应的 AST 类型是 JSXText，我们只需要将前文源码中对应字符串的奇数和偶数转换成 Text 即可。而对于 Comment 而言我们可以将它们全部清除，不参与 WXML 的编译。Element 类型有它的名字（tagName）、children、属性（attributes），其中 children 可能是任意 WXML 类型，属性是一个对象，键值和键名都是字符串。我们将把重点放在如何转换成为 WXML 的 Element 类型。

首先我们可以先看 <View className='home'>，它在 AST 中是一个 JSXElement，它的结构和我们定义 Element 类型差不多。我们先将 JSXElement 的 ScrollView 从驼峰式的 JSX 命名转化为短横线（kebab case）风格，className 和 scrollTop 的值分别代表了 JSXAttribute 值的两种类型：StringLiteral 和 JSXExpressionContainer，className 是简单的 StringLiteral 处理起来很方便，scrollTop 处理起来稍微麻烦点，我们需要用两个花括号{} 把内容包起来…

接下来我们再思考一下每一个 JSXElement 出现的位置，你可以发现其实它的父元素只有几种可能性：return、循环、条件（逻辑）表达式。而在上一篇文章中我们提到，babel-traverse 遍历的 AST 类型是响应式的——也就是说只要我们按照 JSXElement 父元素类型的顺序穷举处理这几种可能性，把各种可能性大结果应用到 JSX 元素之后删除掉原来的表达式，最后就可以把一个复杂的 JSX 表达式转换为一个简单的 WXML 数据结构。…

我们先看第一个循环：

```js
oddNumbers.map(number => <Text onClick={this.handleClick}>{number}</Text>);
```

Text 的父元素是一个 map 函数（CallExpression），我们可以把函数的 callee: oddNumbers 作为 wx:for 的值，并把它放到 state 中，匿名函数的第一个参数是 wx:for-item 的值，函数的第二个参数应该是 wx:for-index 的值，但代码中没有传所以我们可以不管它。然后我们把这两个 wx: 开头的参数作为 attribute 传入 Text 元素就完成了循环的处理。而对于 onClick 而言，在 Taro 中 on 开头的元素参数都是事件，所以我们只要把 this. 去掉即可。Text 元素的 children 是一个 JSXExpressionContainer，我们按照之前的处理方式处理即可。最后这行我们生成出来的数据结构应该是这样…

```js
{
  type: 'element',
  tagName: 'text',
  attributes: [
    { bindtap: 'handleClick' },
    { 'wx:for': '{{oddNumbers}}' },
    { 'wx:for-item': 'number' }
  ],
  children: [
    { type: 'text', content: '{{number}}' }
  ]
}...
```

有了这个数据结构生成一段 WXML 就非常简单了

再来看第二个循环表达式：

```js
numbers.map(number => number % 2 === 0 && <Text onClick={this.handleClick}>{number}</Text>)...
```

它比第一个循环表达式多了一个逻辑表达式（Logical Operators），我们知道 expr1 && expr2 意味着如果 expr1 能转换成 true 则返回 expr2，也就是说我们只要把 number % 2 === 0 作为值生成一个键名 wx:if 的 JSXAttribute 即可。但由于 wx:if 和 wx:for 同时作用于一个元素可能会出现问题，所以我们应该生成一个 block 元素，把 wx:if 挂载到 block 元素，原元素则全部作为 children 传入 block 元素中。这时 babel-traverse 会检测到新的元素 block，它的父元素是一个 map 循环函数，因此我们可以按照第一个循环表达式的处理方法来处理这个表达式。

这里我们可以思考一下 this.props.text || this.props.children 的解决方案。当用户在 JSX 中使用 || 作为逻辑表达式时很可能是 this.props.text 和 this.props.children 都有可能作为结果返回。这里 Taro 将它编译成了 this.props.text ? this.props.text: this.props.children，按照条件表达式（三元表达式）的逻辑，也就是说会生成两个 block，一个 wx:if 和一个 wx:else：

```js
<block wx:if="{{text}}">{{text}}</block>
<block wx:else>
    <slot></slot>
</block>
```