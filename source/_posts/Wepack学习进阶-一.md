---
title: Wepack学习进阶(一)
date: 2022-05-27 14:24:26
tags:
cover: https://static001.geekbang.org/infoq/97/979f6d8773003f91ddbfd0a59e1d5010.png
top_img: https://static001.geekbang.org/infoq/97/979f6d8773003f91ddbfd0a59e1d5010.png
---

## 前言

在当下的前端环境里，各种框架和工具层出不穷，比如 React、Vue、Angular 等，极大的提高了我们的开发效率，但是，他们都有一个共同点：源代码无法直接运行，必须经过转换之后才可执行。
Webpack 凭借强大的功能与良好的使用体验，已经成为目前最流行，社区最活跃的构建工具，是现代 Web 开发必须掌握的技能之一。


## Webpack 的基本概念

Webpack 是使用 NodeJs 开发出来的一个构建工具，本质上，它是一个用于现代 JavaScript 应用程序的 静态模块打包工具。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 依赖图(dependency graph)，然后将你项目中所需的每一个模块组合成一个或多个 bundles，它们均为静态资源，用于展示你的内容。

构建工具的常规作用：

- 代码转换：TypeScript 编译成 JavaScript、SCSS 编译成 CSS 等。

- 文件优化：压缩 JavaScript、CSS、HTML 代码，压缩合并图片等。

- 代码分割：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。

- 模块合并：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。

- 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器。

- 代码校验：在代码被提交到仓库前需要校验代码是否符合规范，以及单元测试是否通过。

- 自动发布：更新完代码后，自动构建出线上发布代码并传输给发布系统。



> 在 Webpack 里一切文件皆模块，通过 Loader 转换文件，通过 Plugin 注入钩子，最后输出由多个模块组合成的文件。

Webpack 专注于构建模块化项目。借用 Webpack 官网首页的图片来看一下它到底是什么

![](https://static001.geekbang.org/infoq/97/979f6d8773003f91ddbfd0a59e1d5010.png)

> 一切文件：JavaScript、CSS、SCSS、图片、模板，在 Webpack 眼中都是一个个模块，这样的好处是能清晰地描述出各个模块之间的依赖关系，以方便 Webpack 对模块进行组合和打包。 经过 Webpack 的处理，最终会输出浏览器能使用的静态资源。

## Webpack 的基本配置
以下是 Webpack的一些核心概念：

- 入口(entry)

- 输出(output)

- loader

- 插件(plugins)

- 模式(mode)

- 其他配置


### 入口(entry)

入口起点(entry point) 指示 webpack 应该使用哪个模块，来作为构建其内部 依赖图(dependency graph) 的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

默认值是 ./src/index.js，也可以通过在 webpack configuration 中配置 entry 属性，来指定一个（或多个）不同的入口起点。

- 用法：entry: string | [string] | object

1. entry 属性的单个入口语法，是以下形式的简写：

```
module.exports = {
    entry: "./app/entry",
}

```
等同于

```
module.exports = {
    entry: {
      main: "./app/entry"
    }
}

```
2. 也可以将一个文件路径数组传递给 entry 属性，这将创建一个 "multi-main entry"。在你想要一次注入多个依赖文件，并且将它们的依赖关系绘制在一个 "chunk" 中时，这种方式就很有用。

```
module.exports = {
    entry: ['./src/file_1.js', './src/file_2.js'],
    output: {
      filename: 'bundle.js',
    },
}

```
3. 对象写法

entry: { <entryChunkName> string | [string] } | {}

```
module.exports = {
  //多页面应用程序
    entry: {
      app: './src/app.js',
      adminApp: './src/adminApp.js',
    },
  //
}

```
用于描述入口的对象。你可以使用如下属性：

- dependOn: 当前入口所依赖的入口。它们必须在该入口被加载前被加载。
- filename: 指定要输出的文件名称。
- import: 启动时需加载的模块。
- publicPath: 当该入口的输出文件在浏览器中被引用时，为它们指定一个公共 URL 地址。

另外 dependOn 不能是循环引用的，下面的例子也会出现错误：

```
module.exports = {
    entry: {
    a3: {
      import: './a',
      dependOn: 'b3',
    },
    b3: {
      import: './b',
      dependOn: 'a3',
    },
  },
}

```

对象语法会比较繁琐。但这是应用程序中定义入口的最可扩展的方式。

**当你通过插件生成入口时，你可以传递空对象 {} 给 entry。**



### 输出(output)

output 属性告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件。主要输出文件的默认值是 ./dist/main.js，其他生成文件默认放置在 ./dist 文件夹中。
**注意，即使可以存在多个 entry 起点，但只能指定一个 output 配置。**

1.  基本用法
 - filename ：必要字段，指定输出文件名称
```
//此配置将一个单独的 bundle.js 文件输出到 dist 目录中。
module.exports = {
    output: {
      filename: 'bundle.js',
    },
}

```

2. 多个入口起点的output

如果配置中创建出多于一个 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用 占位符(substitutions) 来确保每个文件具有唯一的名称。

```
module.exports = {
    entry: {
      app: './src/app.js',
      search: './src/search.js',
    },
    output: {
      filename: '[name].js',
      path: __dirname + '/dist',
    },
}

```

3. 高级进阶（hush和cdn）

```
module.exports = {
    output: {
      path: '/home/proj/cdn/assets/[fullhash]',
      publicPath: 'https://cdn.example.com/assets/[fullhash]/',
    },
}

```
4. output的常用配置项：
（其余配置参考官方网站： https://webpack.docschina.org/configuration/output）

```
//此配置将一个单独的 bundle.js 文件输出到 dist 目录中。
module.exports = {
    output: {
      filename: 'js/[name].js',, // 必要字段，string 或者 function (pathData, assetInfo) => string
      path: path.resolve(__dirname, 'dist/assets'), // output目录对应一个绝对路径
      publicPath: 'https://cdn.example.com/assets/', //此选项指定在浏览器中所引用的「此输出目录对应的公开 URL」取值 string 或者function (pathData, assetInfo) => string，默认值为html路径
      asyncChunks: true, // 是否创建按需加载的异步 chunk
      charset: true, // 是否为 HTML 的 <script> 标签添加 charset="utf-8" 标识

      clean: true, // 在生成文件之前清空 output 目录
      clean: {
        keep: /ignored\/dir\//, // 保留 'ignored/dir' 下的静态资源
      },

      compareBeforeEmit: false, // 在写入到输出文件系统时检查输出的文件是否已经存在并且拥有相同内容,相同将不会重新写入

      library: 'MyLibrary', // string | string[] | object,输出一个库，为你的入口做导出

      chunkFilename: '[id].js', // 决定了非初始（non-initial）chunk 文件的名称(取值同filename)
      chunkFilename: (pathData) => {
        return pathData.chunk.name === 'main' ? '[name].js' : '[name]/[name].js';
      },
    },
}

```


### loader

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 模块，以供应用程序使用，以及被添加到依赖图中。

#### loader 有两个属性：

- test 属性，识别出哪些文件会被转换。
- use 属性，定义出在进行转换时，应该使用哪个 loader。

```
module.exports = {
    module: {
      rules: [
        { test: /\.txt$/, use: 'raw-loader' },
        { test: /\.css$/, use: 'css-loader' },
        { test: /\.ts$/, use: 'ts-loader' },
      ],
    },
}

```

以上配置中，对一个单独的 module 对象定义了 rules 属性，里面包含两个必须属性：test 和 use。这告诉 webpack 编译器(compiler) 当碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先 use(使用) raw-loader 转换一下。

#### 在你的应用程序中，有两种使用 loader 的方式：

- 配置方式（推荐）：在 webpack.config.js 文件中指定 loader。

  loader支持链式使用, loader ***从右到左（或从下到上）地取值(evaluate)/执行(execute)***。在下面的示例中，从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束。

  loader 可以被链式调用意味着不一定要输出 JavaScript。只要下一个 loader 可以处理这个输出，这个 loader 就可以返回任意类型的模块。

```
module.exports = {
    module: {
      rules: [
        {
          test: /\.css$/,
          include: [
            path.resolve(__dirname, "app")
          ],
          exclude: [
            path.resolve(__dirname, "app/demo-files")
          ],
          use: [
            { loader: 'style-loader' },
            { 
              loader: 'css-loader',
              options: {
                modules: true
              }
            },
            { loader: 'sass-loader' }
          ]
        }
      ]
    }
}

```

- 内联方式：在每个 import 语句中显式指定 loader。使用 ! 将资源中的 loader 分开。
```
import styles from 'style-loader!css-loader?modules!./styles.css'

```


使用 ! 前缀，将禁用所有已配置的 normal loader(普通 loader)
```
import styles from '!style-loader!css-loader?modules!./styles.css'

```


使用 !! 前缀，将禁用所有已配置的 loader（preLoader, loader, postLoader）
```
import styles from '!!style-loader!css-loader?modules!./styles.css'

```


**注意在 webpack v4 版本可以通过 CLI 使用 loader，但是在 webpack v5 中被弃用。**


#### 常用的几种loader

1. **样式类loader**
- style-loader 将模块导出的内容作为样式并添加到 DOM 中
- css-loader 加载 CSS 文件并解析 import 的 CSS 文件，最终返回 CSS 代码
- less-loader 加载并编译 LESS 文件
- sass-loader 加载并编译 SASS/SCSS 文件
- postcss-loader 使用 PostCSS 加载并转换 CSS/SSS 文件
- stylus-loader 加载并编译 Stylus 文件

2. **语法类loader**
- babel-loader 使用 Babel 加载 ES2015+ 代码并将其转换为 ES5
- buble-loader 使用 Bublé 加载 ES2015+ 代码并将其转换为 ES5
- traceur-loader 使用 Traceur 加载 ES2015+ 代码并将其转换为 ES5
- ts-loader 像加载 JavaScript 一样加载 TypeScript 2.0+
- coffee-loader 像加载 JavaScript 一样加载 CoffeeScript
- fengari-loader 使用 fengari 加载 Lua 代码
- elm-webpack-loader 像加载 JavaScript 一样加载 Elm

3. **架构类loader**
- vue-loader 加载并编译 Vue 组件
- angular2-template-loader 加载并编译 Angular 组件



#### loader特性

 - loader 可以是同步的，也可以是异步的
 - loader运行在node.js中，并且能执行任何操作
 - loader 可以通过options对象配置
 - 除了常见的通过 package.json 的 main 来将一个 npm 模块导出为 loader，还可以在 module.rules 中使用 loader 字段直接引用一个模块。
 - 插件(plugin)可以为 loader 带来更多特性。
 - loader 能够产生额外的任意文件。


### 插件（plugin）

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

1. 使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。

```
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
};

```
2. webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且在 整个 编译生命周期都可以访问 compiler 对象。

```

// ConsoleLogOnBuildWebpackPlugin.js 中

const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建正在启动！');
    });
  }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;

```


#### 用法
 
 由于插件可以携带参数/选项，你必须在 webpack 配置中，向 plugins 属性传入一个 new 实例。

1. 配置方式

```
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack'); // 访问内置的插件
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
      },
    ],
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' }),
  ],
};

```

2. Node API 方式

```
// some-node-script.js 中

const webpack = require('webpack'); // 访问 webpack 运行时(runtime)
const configuration = require('./webpack.config.js');

let compiler = webpack(configuration);

new webpack.ProgressPlugin().apply(compiler);

compiler.run(function (err, stats) {
  // ...
});


```

3. 常见的一些webpack 插件

- BannerPlugin                 给每个生成的chunk文件添加一个头部
- CommonsChunkPlugin           提炼出各个chunk文件中相同的模块，以减少文件大小和之前的依赖关系 
- CompressionWebpackPlugin     压缩文件并且输出编译好的文件
- ContextReplacementPlugin     处理require 的引入方式
- CopyWebpackPlugin            复制指定文件到build 目录
- DefinePlugin                 在编译好的文件中暴露出全局变量供外部使用
- DllPlugin                    拆分构建包，用来缩短构建时间
- HotModuleReplacementPlugin   启用热更新    
- HtmlWebpackPlugin            生成一个html文件插入到bundle.js中   
- LimitChunkCountPlugin        设置分块的最小/最大限制以更好地控制拆分模块   
- ProgressPlugin               报告编译进度
- ProvidePlugin                扩展可以不使用import 或 require 的方式引入模块
- SourceMapDevToolPlugin       生成map文件以实现更细致的文件分析功能
- MiniCssExtractPlugin         为每个需要 CSS 的 JS 文件创建一个 CSS 文件

### mode

提供 mode 配置选项，告知 webpack 使用相应模式的内置优化。默认 ‘production’

string = 'production': 'none' | 'development' | 'production'

- 配置：
1. 在config中配置

```
module.exports = {
  mode: 'development',
};

```

2. 在cli启动中配置

```

 "scripts": {
    "build:yufa": "webpack --mode=development"
  },

```


### 其他配置

```
resolve: {
    modules: ["node_modules",path.resolve(__dirname, "app")],
    extensions: [".js", ".json", ".jsx", ".css"],
    // 使用的扩展名
    alias: {
      // a list of module name aliases
      // aliases are imported relative to the current context
      "module": "new-module",
      // 别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"
      "only-module$": "new-module",
      // 别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"
      "module": path.resolve(__dirname, "app/third/module.js"),
      // alias "module" -> "./app/third/module.js" and "module/file" results in error
      "module": path.resolve(__dirname, "app/third"),
      // alias "module" -> "./app/third" and "module/file" -> "./app/third/file"
      [path.resolve(__dirname, "app/module.js")]: path.resolve(__dirname, "app/alternative-module.js"),
      // alias "./app/module.js" -> "./app/alternative-module.js"
    },
},
performance: {
    hints: "warning", // 枚举
    maxAssetSize: 200000, // 整数类型（以字节为单位）
    maxEntrypointSize: 400000, // 整数类型（以字节为单位）
    assetFilter: function(assetFilename) {
      // 提供资源文件名的断言函数
      return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
    }
},
devtool: "source-map", // enum
target: "web",
externals: ["react", /^@angular/],
optimization: {
    chunkIds: "size",
    // method of generating ids for chunks
    moduleIds: "size",
    // method of generating ids for modules
    mangleExports: "size",
    // rename export names to shorter names
    minimize: true,
    // minimize the output files
    minimizer: [new CssMinimizer(), "..."],
},
experiments: {
    asyncWebAssembly: true,
    // WebAssembly as async module (Proposal)
    syncWebAssembly: true,
    // WebAssembly as sync module (deprecated)
    outputModule: true,
    // Allow to output ESM
    topLevelAwait: true,
    // Allow to use await on module evaluation (Proposal)
},
devServer: {
    proxy: { // proxy URLs to backend development server
      '/api': 'http://localhost:3000'
    },
    static: path.join(__dirname, 'public'), // boolean | string | array | object, static file location
    compress: true, // enable gzip compression
    historyApiFallback: true, // true for index.html upon 404, object for multiple paths
    hot: true, // hot module replacement. Depends on HotModuleReplacementPlugin
    https: false, // true for self-signed, object for cert authority
    // ...
},
```



## Webpack 的优缺点
#### 优点
- 专注于处理模块化的项目，能做到开箱即用一步到位；

- 通过 Plugin 扩展，完整好用又不失灵活；

- 使用场景不仅限于 Web 开发；

- 社区庞大活跃，经常引入紧跟时代发展的新特性，能为大多数场景找到已有- 的开源扩展；

- 良好的开发体验。

#### 缺点：
- 只能采用模块化开发

## 选择 Webpack 的原因
- 大多数团队在开发新项目时会采用紧跟时代的技术，这些技术几乎都会采用“模块化+新语言+新框架”，Webpack 可以为这些新项目提供一站式的解决方案；

- Webpack 有良好的生态链和维护团队，能提供良好的开发体验和保证质量；

- Webpack 被全世界的大量 Web 开发者使用和验证，能找到各个层面所需的教程和经验分享。



##### Loader 和 Plugin 有哪些不同？请描述一下开发 Loader 和 Plugin 的思路。



- ader:  loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

- plugin； loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

- 开发一个 loader 同时也可能需要安装其他的插件，比如开发一个解析 markdown 文件的 loader，可能需要用到 marked 插件，总而言之最终 loader 无论如何输出的结果只有两种

  * 生成一段 js 代码供给 webpack 直接使用

  * 如果不能生成一段 js 代码就必须把输出的数据交给下一个 loader 进行处理，就比如 css-loader 后面必须使用 style-loader 才能正常打包 css 文件，因为 css-loader 生成的数据 webpack 无法直接使用

  * 开发一个 Plugin 需要将任务挂载在 webpack 生命周期的钩子（相当于事件监听）上才能实现

  * webpack 插件是一个具有 apply 属性的 JavaScript 对象。apply 属性会被 webpack compiler 调用，并且 compiler 对象可在整个编译生命周期访问。

  * 我们可以发现，几乎每一个插件使用的时候都要 new 一个新对象，所以我们可以使用创建一个构造函数（类）的方法创建一个新插件，内部添加一个 apply 方法

  * apply 接受一个 webpack 核心对象参数 compiler，使用 compiler 对象中的 compiler.hooks.钩子.tap()方法的方式实现插件的加载,例如：

```
    class MyPlugin {
      apply(compiler) {
        compiler.hooks.钩子.tap('MyPlugin', compilation =>{
        	...
        })
      }
    }
```
