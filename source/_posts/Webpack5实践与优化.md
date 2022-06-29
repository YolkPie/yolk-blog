---
title: Webpack5实践与优化
date: 2022-07-01 10:00:00
---

# Webpack5简单介绍
Webpack 5 发布于 2020年10月10日，Webpack 5 对Node.js 的版本要求至少是10.13.0 (LTS)。
有以下特点：
- 尝试用持久性缓存来提高构建性能。
- 尝试用更好的算法和默认值来改进长期缓存。
- 尝试用更好的 Tree Shaking 和代码生成来改善包大小。
- 尝试改善与网络平台的兼容性。
- 尝试在不引入任何破坏性变化的情况下，清理那些在实现 v4 功能时处于奇怪状态的内部结构。
  
> 持久化缓存是 webpack5 所带来的非常强大的特性之一。一句话概括就是构建结果持久化缓存到本地的磁盘，二次构建(非 watch 模块)直接利用磁盘缓存的结果从而跳过构建过程当中的 resolve、build 等耗时的流程，从而大大提升编译构建的效率。




# 实践

## 基础配置
接下来一起配置一个的 Webpack5示例项目。
将支持以下功能：
- 分离开发环境、生产环境配置；
- 模块化开发；
- sourceMap 定位警告和错误；
- 动态生成引入 bundle.js 的 HTML5 文件；
- 实时编译；
- 封装编译、打包命令。

### 开始新建

```
// 初始化项目
npm init -y

// 创建 src 文件夹
mkdir src

// 创建 js文件
touch index.js
touch hello.js
```

index.js

```
import './hello.js'

console.log('index')
```

hello.js

```
console.log('hello webpack')
```

### 安装
#### node（版本有要求）

#### webpack

```
npm install webpack webpack-cli --save-dev
```
#### 新建配置文件

```
// 创建 config 目录
mkdir config

// 进入 config 目录
cd ./config

// 创建通用环境配置文件
touch webpack.common.js

// 创建开发环境配置文件
touch webpack.dev.js

// 创建生产环境配置文件
touch webpack.prod.js
```
##### webpack-merge

使用 webpack-marge 合并通用配置和特定环境配置。

```
// 安装
npm i webpack-merge -D

// webpack.common.js通用环境配置
module.exports = {} // 暂不添加配置

// webpack.dev.js开发环境配置
const { merge } = require('webpack-merge')
const common = require('./webpack.common')

module.exports = merge(common, {}) // 暂不添加配置

// webpack.prod.js生产环境配置
const { merge } = require('webpack-merge')
const common = require('./webpack.common')

module.exports = merge(common, {}) // 暂不添加配置
```
#### 入口（entry）
入口起点(entry point) 指示 webpack 应该使用哪个模块来作为构建其内部依赖图(dependency graph) 的开始。进入入口起点后，webpack会找出有哪些模块和库是入口起点（直接和间接）依赖的。

在此例中，使用 src/index.js 作为项目入口，webpack 以 src/index.js 为起点，查找所有依赖的模块。
修改 webpack.commom.js：

```
module.exports = merge(common, {
  // 入口
  entry: {
    index: './src/index.js',
  },
})
```
#### 输出（output)

输出（output)告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件。

生产环境的 output 需要通过 contenthash 值来区分版本和变动，可达到清缓存的效果，而本地环境为了构建效率，则不引人 contenthash。

新增 paths.js，封装路径方法：

```
const fs = require('fs')
const path = require('path')

const appDirectory = fs.realpathSync(process.cwd());
const resolveApp = relativePath => path.resolve(appDirectory, relativePath);

module.exports = {
  resolveApp
}
```

修改开发环境配置文件 webpack.dev.js：

```
module.exports =  merge(common, {
  // 输出
  output: {
    // bundle 文件名称
    filename: '[name].bundle.js',

    // bundle 文件路径
    path: resolveApp('dist'),

    // 编译前清除目录
    clean: true
  },
})
```

修改生产环境配置文件 webpack.prod.js：

```
module.exports =  merge(common, {
  // 输出
  output: {
    // bundle 文件名称 【只有这里和开发环境不一样】
    filename: '[name].[contenthash].bundle.js',

    // bundle 文件路径
    path: resolveApp('dist'),

    // 编译前清除目录
    clean: true
  },
})
```

> 上述 filename 的占位符解释如下
- [name] - chunk name（例如 [name].js -> app.js）。如果 chunk 没有名称，则会使用其 id 作为名称
- [contenthash] - 输出文件内容的 md4-hash（例如 [contenthash].js -> 4ea6ff1de66c537eb9b2.js）


#### 模式（mode）
通过 mode 配置选项，告知 webpack 使用相应模式的内置优化。
|选项|描述|
|-|-|
|development|会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 development. 为模块和 chunk 启用有效的名。|
|production|会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 production。为模块和 chunk 启用确定性的混淆名称。|

修改开发环境配置文件 webpack.dev.js：

```
module.exports =  merge(common, {
  // 开发模式
  mode: 'development',
})
```

修改开发环境配置文件 webpack.prod.js：

```
module.exports =  merge(common, {
  // 生产模式
  mode: 'production',
})
```

#### Source Map
当 webpack 打包源代码时，可能会很难追踪到 error 和 warning 在源代码中的原始位置。

为了更容易地追踪 error 和 warning， source map 可以将编译后的代码映射回原始源代码。

修改开发环境配置文件 webpack.dev.js：

```
module.exports =  merge(common, {
  // 开发工具，开启 source map，编译调试
  devtool: 'eval-cheap-module-source-map',
})
```

source map 还有许多其他 [可用选项](https://webpack.docschina.org/configuration/devtool) 

> 一般来说，为加快生产环境打包速度，不为生产环境配置 devtool。

#### HtmlWebpackPlugin

`npx webpack --config config/webpack.prod.js` 后生成了 bundle.js，我们需要一个 HTML5 文件，用来动态引入打包生成的 bundle 文件。

引入 HtmlWebpackPlugin 插件，生成一个 HTML5 文件， 其中包括使用 script 标签的 body 中的所有 webpack 包。

- 安装
  
```
npm install --save-dev html-webpack-plugin
```

- 修改通用环境配置文件 webpack.commom.js：
  
```
module.exports = {
  plugins: [
    // 生成html，自动引入所有bundle
    new HtmlWebpackPlugin({
      title: 'release_v0',
    }),
  ],
}
```

执行 `npx webpack --config config/webpack.prod.js`
生成了 index.html，动态引入了 bundle.js 文件：

```
<!DOCTYPE html>
<html>
 <head>
  <meta charset="utf-8" />
  <title>release_v0</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <script defer="defer" src="index.468d741515bfc390d1ac.bundle.js"></script>
 </head>
 <body></body>
</html>
```
#### DevServer
在每次编译代码时，手动运行 npx webpack --config config/webpack.prod.js 会显得很麻烦， webpack-dev-server 帮助我们在代码发生变化后自动编译代码。

webpack-dev-server 提供了一个基本的 web server，并且具有实时重新加载功能。
- 安装
  
```
npm install --save-dev webpack-dev-server
```

- 修改开发环境配置文件 webpack.dev.js：
  
```
module.exports = merge(common, {
  devServer: {
    // 告诉服务器从哪里提供内容，只有在你想要提供静态文件时才需要。
    contentBase: './dist',
  },
})
```

#### 执行命令
优化 webpack 的实时编译、打包编译指令。
通过 cross-env 配置环境变量，区分开发环境和生产环境。

- 安装
  
```
npm install --save-dev cross-env
```

- 修改 package.json：
  
```
{
    "scripts": {
        "dev": "cross-env NODE_ENV=development webpack serve --open --config config/webpack.dev.js",
        "build": "cross-env NODE_ENV=production webpack --config config/webpack.prod.js"
      },
}
```

-  npm run dev：本地构建
-  npm run build：生产打包

以上我们完成了一个基于 webpack5 编译的支持模块化开发的简单项目。下面开始进阶配置。

### 进阶配置
在上述配置基础上，继续配置，以实现如下功能：

- 加载图片；
- 加载字体；
- 加载 CSS；
- 使用 SASS；
- 使用 PostCSS，并自动为 CSS 规则添加前缀，解析最新的 CSS 语法，引入 css-modules 解决全局命名冲突问题；
- 使用 React；
- 使用 TypeScript。

#### 加载图片

在 webpack 5 中，可以使用内置的 [资源模块（Asset Modules）](https://webpack.docschina.org/guides/asset-modules/) ，将 images 图像混入我们的系统中。

修改通用环境配置文件 webpack.commom.js：

```
const { resolveApp } = require('./paths');
module.exports = {
    module: {
        rules: [
          {
            test: /\.(png|svg|jpg|jpeg|gif)$/i,
            include: [
              resolveApp('src'),
            ],
            type: 'asset/resource',
          },
        ],
      },
}
```

#### 加载字体（Font）
使用 [资源模块（Asset Modules）](https://webpack.docschina.org/guides/asset-modules/)  接收字体文件。

修改通用环境配置文件 webpack.commom.js：

```
module.exports = {
    module: {
        rules: [
            {
               test: /.(woff|woff2|eot|ttf|otf)$/i,
               include: [
                  resolveApp('src'),
                ],
               type: 'asset/resource',
             },
         ]
     }
 }
```

#### 加载 CSS

##### style-loader
style-loader 用于将 CSS 插入到 DOM 中，通过使用多个 <style></style> 自动把 styles 插入到 DOM 中.

##### css-loader
css-loader 对 @import 和 url() 进行处理，就像 js 解析 import/require() 一样，让 CSS 也能模块化开发。

- 安装相关依赖：
  
```
npm install --save-dev style-loader css-loader
```

- 修改通用环境配置文件 webpack.commom.js：
  
```
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                include: paths.appSrc,
                use: [
                  // 将 JS 字符串生成为 style 节点
                  'style-loader',
                  // 将 CSS 转化成 CommonJS 模块
                  'css-loader',
                ],
              },
          ]
      }
  }
```

#### 使用 SASS
##### Sass
Sass 是一款强化 CSS 的辅助工具，它在 CSS 语法的基础上增加了变量、嵌套、混合、导入等高级功能。
##### sass-loader
sass-loader 加载 Sass/SCSS 文件并将他们编译为 CSS。

- 安装相关依赖：
  
```
npm install --save-dev sass-loader sass
```

- 修改通用环境配置文件 webpack.commom.js：

```
module.exports = {
    module: {
        rules: [
        {
        test: /.(scss|sass)$/,
        include: paths.appSrc,
        use: [
          // 将 JS 字符串生成为 style 节点
          'style-loader',
          // 将 CSS 转化成 CommonJS 模块
          'css-loader',
          // 将 Sass 编译成 CSS
          'sass-loader',
        ],
      },
```

#### 使用 PostCSS
##### PostCSS
PostCSS 是一个用 JavaScript 工具和插件转换 CSS 代码的工具。
- 可以自动为 CSS 规则添加前缀；
- 将最新的 CSS 语法转换成大多数浏览器都能理解的语法；
- css-modules 解决全局命名冲突问题。

##### postcss-loader
postcss-loader 使用 PostCSS 处理 CSS 的 loader。

- 安装相关依赖
  
```
npm install --save-dev postcss-loader postcss postcss-preset-env
```

- 修改通用环境配置文件 webpack.commom.js：
  
```
const { resolveApp } = require('./paths');
module.exports = {
    module: {
        rules: [
          {
            test: /\.module\.(scss|sass)$/,
            include: paths.appSrc,
            use: [
              // 将 JS 字符串生成为 style 节点
              'style-loader',
              // 将 CSS 转化成 CommonJS 模块
              {
                loader: 'css-loader',
                options: {
                  // Enable CSS Modules features
                  modules: true,
                  importLoaders: 2,
                  // 0 => no loaders (default);
                  // 1 => postcss-loader;
                  // 2 => postcss-loader, sass-loader
                },
              },
              // 将 PostCSS 编译成 CSS
              {
                loader: 'postcss-loader',
                options: {
                  postcssOptions: {
                    plugins: [
                      [
                        // postcss-preset-env 包含 autoprefixer
                        'postcss-preset-env',
                      ],
                    ],
                  },
                },
              },
              // 将 Sass 编译成 CSS
              'sass-loader',
            ],
          },
        ],
      },
}
```

> 为提升构建效率，为 loader 指定 include，通过使用 include 字段，仅将 loader 应用在实际需要将其转换的模块。

#### 使用 React + TypeScript

- 安装 React 相关
  
```
npm i react react-dom @types/react @types/react-dom -D
```

- 安装 TypeScript 相关：
  
```
npm i -D typescript esbuild-loader
```

> 为提高性能，摒弃了传统的 ts-loader，选择最新的 esbuild-loader。

- 修改通用环境配置文件 webpack.commom.js：
  
```
module.exports = {
    resolve: {
        extensions: ['.tsx', '.ts', '.js'],
    },
    module: {
        rules: [
            {
                test: /\.(js|ts|jsx|tsx)$/,
                include: paths.appSrc,
                use: [
                  {
                    loader: 'esbuild-loader',
                    options: {
                      loader: 'tsx',
                      target: 'es2015',
                    },
                  }
                ]
              },
         ]
     }
 }
```

TypeScript 是 JavaScript 的超集，为其增加了类型系统，可以编译为普通 JavaScript 代码。

为兼容 TypeScript 文件，新增 typescript 配置文件 tsconfig.json：

```
{
    "compilerOptions": {
        "outDir": "./dist/",
        "noImplicitAny": true,
        "module": "es6",
        "target": "es5",
        "jsx": "react",
        "allowJs": true,
        "moduleResolution": "node",
        "allowSyntheticDefaultImports": true,
        "esModuleInterop": true,
      }
}
```



# 优化

- 开发体验
- 加快编译速度
- 减小打包体积

## 开发体验

### 自动更新

自动更新：在开发过程中，修改代码后，无需手动再次编译，可以自动编译代码更新编译后代码的功能。

webpack5 提供了以下几种可选方式，来实现自动更新功能：

- [webpack's Watch Mode](https://webpack.docschina.org/configuration/watch/#watch)
- [webpack-dev-server](https://github.com/webpack/webpack-dev-server)
- [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware)
官方推荐的方式是 webpack-dev-server，前面已经介绍了 webpack-dev-server 帮助我们在代码发生变化后自动编译代码实现自动更新的用法，在这里不重复赘述。

### 热更新

热更新：在开发过程中，修改代码后，仅更新修改部分的内容，无需刷新整个页面。

#### 修改 webpack-dev-server 配置

```
module.export = {
    devServer: {
        contentBase: './dist',
        hot: true, // 热更新
      },
}
```

#### 引入 react-refresh-webpack-plugin

- 安装
  
```
npm install -D @pmmmwh/react-refresh-webpack-plugin react-refresh
```

- 修改 webpack.dev.js 配置：
  
```
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

module.exports = {
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new ReactRefreshWebpackPlugin(),
    ]
}
```

### 加快构建速度

webpack5 较于 webpack4，新增了<font color="red">持久化缓存、改进缓存算法</font>等优化，较之前版本构建速度已有不错的提升。

#### cache

通过配置 [webpack 持久化缓存](https://webpack.docschina.org/configuration/cache/#root) cache: filesystem，来缓存生成的 webpack 模块和 chunk，改善构建速度。

简单来说，通过 cache: filesystem 可以将构建过程的 webpack 模板进行缓存，大幅提升二次构建速度、打包速度，当构建突然中断，二次进行构建时，可以直接从缓存中拉取，因而提升构建速度。

#### 减少 loader、plugins

为 loader 指定 include，减少 loader 应用范围，仅应用于最少数量的必要模块，来提升[webpack构建性能](https://webpack.docschina.org/guides/build-performance/)

webpack.common.js 配置方式如下：

```
module.exports = {
    rules: [
        {
            test: /\.(js|ts|jsx|tsx)$/,
            include: paths.appSrc,
            use: [
              {
                loader: 'esbuild-loader',
                options: {
                  loader: 'tsx',
                  target: 'es2015',
                },
              }
            ]
         }
    ]
}
```

> rule.exclude 可以排除模块范围，也可用于减少 loader 应用范围。


##### 管理资源

使用 webpack 资源模块 (asset module) 代替旧的 assets loader（如 file-loader/url-loader/raw-loader 等），减少 loader 配置数量。

```
module.exports = {
    rules: [
       {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        include: [
          paths.appSrc,
        ],
        type: 'asset/resource',
      },
    ]
}
```

#### 优化 resolve 配置
[resolve](https://webpack.docschina.org/configuration/resolve/#root) 用来配置 webpack 如何解析模块，可通过优化 resolve 配置来覆盖默认配置项，减少解析范围。

##### alias
alias 可以创建 import 或 require 的别名，用来简化模块引入。
webpack.common.js 配置方式如下：

```
module.exports = {
    resolve: {
        alias: {
          '@': paths.appSrc, // @ 代表 src 路径
        },
    }
}
```

##### extensions
extensions 表示需要解析的文件类型列表。

根据项目中的文件类型，定义 extensions，以覆盖 webpack 默认的 extensions，加快解析速度。

由于 <font color='red'>webpack 的解析顺序是从左到右</font>，因此要将使用频率高的文件类型放在左侧，如下我将 tsx 放在最左侧。

webpack.common.js 配置方式如下：

```
module.exports = {
    resolve: {
        extensions: ['.tsx', '.js'], // 这里只是举例，如果有其他类型，需要按顺序添加进去。
    }
}
```

#### modules
modules 表示 webpack 解析模块时需要解析的目录。

指定目录可缩小 webpack 解析范围，加快构建速度。
webpack.common.js 配置方式如下：

```
module.exports = {
    modules: [
      'node_modules', //这里只是举例说明
       paths.appSrc,
    ]
}
```

#### thread-loader
通过 [thread-loader](https://webpack.docschina.org/loaders/thread-loader/#root) 将耗时的 loader 放在一个独立的 worker 池中运行，加快 loader 构建速度。

- 安装：
  
```
npm i -D thread-loader
```

webpack.common.js 配置方式如下：

```
module.exports = {
    rules: [
        {
        test: /\.module\.(scss|sass)$/,
        include: paths.appSrc,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              importLoaders: 2,
            },
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  [
                    'postcss-preset-env',
                  ],
                ],
              },
            },
          },
          {
            loader: 'thread-loader',
            options: {
              workerParallelJobs: 2
            }
          },
          'sass-loader',
        ].filter(Boolean),
      },
    ]
}
```

> webpack 官网 提到 node-sass 中有个来自 Node.js 线程池的阻塞线程的 bug。 当使用 thread-loader 时，需要设置 workerParallelJobs: 2。

> 应该仅在非常耗时的 loader 前引入 thread-loader，如果代码量比较小，滥用thread-loader，反而会影响构建速度。

### 减小打包体积

#### 代码压缩
通过 webpack 插件，将 JS、CSS 等文件进行压缩。
##### JS 压缩

使用 [TerserWebpackPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/) 来压缩 JavaScript。

webpack5 自带最新的 terser-webpack-plugin，无需手动安装。

> terser-webpack-plugin 默认开启了 parallel: true 配置，并发运行的默认数量： os.cpus().length - 1 ，本文配置的 parallel 数量为 4，使用多进程并发运行压缩以提高构建速度。

webpack.prod.js 配置方式如下：

```
const TerserPlugin = require('terser-webpack-plugin');
module.exports = {
    optimization: {
        minimizer: [
            new TerserPlugin({
              parallel: 4,
              terserOptions: {
                parse: {
                  ecma: 8,
                },
                compress: {
                  ecma: 5,
                  warnings: false,
                  comparisons: false,
                  inline: 2,
                },
                mangle: {
                  safari10: true,
                },
                output: {
                  ecma: 5,
                  comments: false,
                  ascii_only: true,
                },
              },
            }),
        ]
    }
}
```

##### CSS 压缩

使用 [CssMinimizerWebpackPlugin](https://webpack.docschina.org/plugins/css-minimizer-webpack-plugin/#root) 压缩 CSS 文件。

- 安装
  
```
npm install -D css-minimizer-webpack-plugin
```

- webpack.prod.js 配置方式如下：
  
```
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

module.exports = {
  optimization: {
    minimizer: [
      new CssMinimizerPlugin({
          parallel: 4,
        }),
    ],
  }
}
```

#### 代码分离

代码分离能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。

##### 抽离重复代码

[SplitChunksPlugin](https://webpack.docschina.org/plugins/split-chunks-plugin) 插件，可以将公共的依赖模块提取到已有的入口 chunk 中，或者提取到一个新生成的 chunk。

webpack 将根据以下条件自动拆分 chunks：

- 新的 chunk 可以被共享，或者模块来自于 node_modules 文件夹；
- 新的 chunk 体积大于 20kb（在进行 min+gz 之前的体积）；
- 当按需加载 chunks 时，并行请求的最大数量小于或等于 30；
- 当加载初始化页面时，并发请求的最大数量小于或等于 30； 通过 splitChunks 把 react 等公共库抽离出来，不重复引入占用体积。

> 注意：切记不要为 cacheGroups 定义固定的 name，因为  cacheGroups.name  指定字符串或始终返回相同字符串的函数时，会将所有常见模块和 vendor 合并为一个 chunk。这会导致更大的初始下载量并减慢页面加载速度。

webpack.prod.js 配置方式如下：

```
module.exports = {
    splitChunks: {
      // include all types of chunks
      chunks: 'all',
      // 重复打包问题
      cacheGroups:{
        vendors:{ // node_modules里的代码
          test: /[\\/]node_modules[\\/]/,
          chunks: "all",
          // name: 'vendors', 一定不要定义固定的name
          priority: 10, // 优先级
          enforce: true 
        }
      }
    },
}
```

这样即将公共的模块单独打包，不再重复引入了。

##### CSS 文件分离
如果 CSS 是放在 JS 文件中，[MiniCssExtractPlugin](https://webpack.docschina.org/plugins/mini-css-extract-plugin/) 插件将 CSS 提取到单独的文件中，为每个包含 CSS 的 JS 文件创建一个 CSS 文件，并且支持 CSS 和 SourceMaps 的按需加载。

- 安装：
  
```
npm install -D mini-css-extract-plugin
```

- webpack.common.js 配置方式如下：
  
```
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  plugins: [new MiniCssExtractPlugin()],
  module: {
    rules: [
        {
        test: /\.module\.(scss|sass)$/,
        include: paths.appSrc,
        use: [
          'style-loader',
          isEnvProduction && MiniCssExtractPlugin.loader, // 仅生产环境
          {
            loader: 'css-loader',
            options: {
              modules: true,
              importLoaders: 2,
            },
          },
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  [
                    'postcss-preset-env',
                  ],
                ],
              },
            },
          },
          {
            loader: 'thread-loader',
            options: {
              workerParallelJobs: 2
            }
          },
          'sass-loader',
        ].filter(Boolean),
      },
    ]
  },
};
```

> 注意：MiniCssExtractPlugin.loader 一定要放在 style-loader 后面。

#### 最小化 entry chunk

通过配置 optimization.runtimeChunk = true，为运行时代码创建一个额外的 chunk，减少 entry chunk 体积，提高性能。

webpack.prod.js 配置方式如下：

```
module.exports = {
    optimization: {
        runtimeChunk: true,
      },
    };
}
```

#### Tree Shaking（摇树） 
摇树，就是将枯黄的落叶摇下来，只留下树上活的叶子。枯黄的落叶代表项目中未引用的无用代码，活的树叶代表项目中实际用到的源码。

##### JS

[JS Tree Shaking](https://webpack.docschina.org/guides/tree-shaking/) 将 JavaScript 上下文中的未引用代码（Dead Code）移除，通过 package.json 的 "sideEffects" 属性作为标记，向 compiler 提供提示，表明项目中的哪些文件是 "pure(纯正 ES2015 模块)"，由此可以安全地删除文件中未使用的部分。

###### webpack5 sideEffects
通过 package.json 的 "sideEffects" 属性，来实现这种方式。

```
{
  "name": "your-project",
  "sideEffects": false
}
```

需注意的是，当代码有副作用时，需要将 sideEffects 改为提供一个数组，添加有副作用代码的文件路径：

```
{
  "name": "your-project",
  "sideEffects": ["./src/some-side-effectful-file.js"]
}
```

##### CSS

使用 [purgecss-webpack-plugin](https://github.com/FullHuman/purgecss/tree/main/packages/purgecss-webpack-plugin) 对 CSS Tree Shaking。

- 安装：
  
```
npm i purgecss-webpack-plugin -D
```

因为打包时 CSS 默认放在 JS 文件内，因此要结合 webpack 分离 CSS 文件插件 mini-css-extract-plugin 一起使用，先将 CSS 文件分离，再进行 CSS Tree Shaking。

webpack.prod.js 配置方式如下：

```
const glob = require('glob')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const PurgeCSSPlugin = require('purgecss-webpack-plugin')
const paths = require('paths')

module.exports = {
  plugins: [
    // 打包体积分析
    new BundleAnalyzerPlugin(),
    // 提取 CSS
    new MiniCssExtractPlugin({
      filename: "[name].css",
    }),
    // CSS Tree Shaking
    new PurgeCSSPlugin({
      paths: glob.sync(`${paths.appSrc}/**/*`,  { nodir: true }),
    }),
  ]
}
```

# 自定义插件
插件向第三方开发者提供了 webpack 引擎中完整的能力。使用阶段式的构建回调，开发者可以在 webpack 构建流程中引入自定义的行为。

### 创建插件

webpack 插件由以下组成：

- 一个 JavaScript 命名函数或 JavaScript 类。
- 在插件函数的 prototype 上定义一个 apply 方法。
- 指定一个绑定到 webpack 自身的事件钩子。
- 处理 webpack 内部实例的特定数据。
- 功能完成后调用 webpack 提供的回调。

```
// 一个 JavaScript 类
class MyExampleWebpackPlugin {
  // 在插件函数的 prototype 上定义一个 `apply` 方法，以 compiler 为参数。
  apply(compiler) {
    // 指定一个挂载到 webpack 自身的事件钩子。
    compiler.hooks.emit.tapAsync(
      'MyExampleWebpackPlugin',
      (compilation, callback) => {
        console.log('这是一个示例插件！');
        console.log(
          '这里表示了资源的单次构建的 `compilation` 对象：',
          compilation
        );

        // 用 webpack 提供的插件 API 处理构建过程
        compilation.addModule(/* ... */);

        callback();
      }
    );
  }
}
```

### 基本插件架构
插件是由「具有 apply 方法的 prototype 对象」所实例化出来的。这个 apply 方法在安装插件时，会被 webpack compiler 调用一次。apply 方法可以接收一个 webpack compiler 对象的引用，从而可以在回调函数中访问到 compiler 对象。一个插件结构如下：

```
class HelloWorldPlugin {
  apply(compiler) {
    compiler.hooks.done.tap(
      'Hello World Plugin',
      (
        stats /* 绑定 done 钩子后，stats 会作为参数传入。 */
      ) => {
        console.log('Hello World!');
      }
    );
  }
}

module.exports = HelloWorldPlugin;
```

然后，要安装这个插件，只需要在你的 webpack 配置的 plugin 数组中添加一个实例：

```
// webpack.config.js
var HelloWorldPlugin = require('hello-world');

module.exports = {
  // ... 这里是其他配置 ...
  plugins: [new HelloWorldPlugin({ options: true })],
};
```

### Compiler 和 Compilation
在插件开发中最重要的两个资源就是 [compiler](https://blog.csdn.net/qq_40409143/article/details/123663207) 和 [compilation](https://blog.csdn.net/weixin_34293911/article/details/88675677) 对象。

```
class HelloCompilationPlugin {
  apply(compiler) {
    // 指定一个挂载到 compilation 的钩子，回调函数的参数为 compilation 。
    compiler.hooks.compilation.tap('HelloCompilationPlugin', (compilation) => {
      // 现在可以通过 compilation 对象绑定各种钩子
      compilation.hooks.optimize.tap('HelloCompilationPlugin', () => {
        console.log('资源已经优化完毕。');
      });
    });
  }
}

module.exports = HelloCompilationPlugin;
```

### 异步编译插件
有些插件钩子是异步的。我们可以像同步方式一样用 tap 方法来绑定，也可以用 tapAsync 或 tapPromise 这两个异步方法来绑定。
#### tapAsync

当我们用 tapAsync 方法来绑定插件时，_必须_调用函数的最后一个参数 callback 指定的回调函数。

```
class HelloAsyncPlugin {
  apply(compiler) {
    compiler.hooks.emit.tapAsync(
      'HelloAsyncPlugin',
      (compilation, callback) => {
        // 执行某些异步操作...
        setTimeout(function () {
          console.log('异步任务完成...');
          callback();
        }, 1000);
      }
    );
  }
}

module.exports = HelloAsyncPlugin;
```

#### tapPromise
当我们用 tapPromise 方法来绑定插件时，_必须_返回一个 pormise ，异步任务完成后 resolve 。

```
class HelloAsyncPlugin {
  apply(compiler) {
    compiler.hooks.emit.tapPromise('HelloAsyncPlugin', (compilation) => {
      // 返回一个 pormise ，异步任务完成后 resolve
      return new Promise((resolve, reject) => {
        setTimeout(function () {
          console.log('异步任务完成...');
          resolve();
        }, 1000);
      });
    });
  }
}

module.exports = HelloAsyncPlugin;
```

### 示例
一个简单的示例插件，生成一个叫做 assets.md 的新文件；文件内容是所有构建生成的文件的列表。这个插件大概像下面这样：

```
class FileListPlugin {
  static defaultOptions = {
    outputFile: 'assets.md',
  };

  // 需要传入自定义插件构造函数的任意选项
  //（这是自定义插件的公开API）
  constructor(options = {}) {
    // 在应用默认选项前，先应用用户指定选项
    // 合并后的选项暴露给插件方法
    // 记得在这里校验所有选项
    this.options = { ...FileListPlugin.defaultOptions, ...options };
  }

  apply(compiler) {
    const pluginName = FileListPlugin.name;

    // webpack 模块实例，可以通过 compiler 对象访问，
    // 这样确保使用的是模块的正确版本
    // （不要直接 require/import webpack）
    const { webpack } = compiler;

    // Compilation 对象提供了对一些有用常量的访问。
    const { Compilation } = webpack;

    // RawSource 是其中一种 “源码”("sources") 类型，
    // 用来在 compilation 中表示资源的源码
    const { RawSource } = webpack.sources;

    // 绑定到 “thisCompilation” 钩子，
    // 以便进一步绑定到 compilation 过程更早期的阶段
    compiler.hooks.thisCompilation.tap(pluginName, (compilation) => {
      // 绑定到资源处理流水线(assets processing pipeline)
      compilation.hooks.processAssets.tap(
        {
          name: pluginName,

          // 用某个靠后的资源处理阶段，
          // 确保所有资源已被插件添加到 compilation
          stage: Compilation.PROCESS_ASSETS_STAGE_SUMMARIZE,
        },
        (assets) => {
          // "assets" 是一个包含 compilation 中所有资源(assets)的对象。
          // 该对象的键是资源的路径，
          // 值是文件的源码

          // 遍历所有资源，
          // 生成 Markdown 文件的内容
          const content =
            '# In this build:\n\n' +
            Object.keys(assets)
              .map((filename) => `- ${filename}`)
              .join('\n');

          // 向 compilation 添加新的资源，
          // 这样 webpack 就会自动生成并输出到 output 目录
          compilation.emitAsset(
            this.options.outputFile,
            new RawSource(content)
          );
        }
      );
    });
  }
}

module.exports = { FileListPlugin };
```

```
//webpack.config.js
const { FileListPlugin } = require('./file-list-plugin.js');

// 在 webpack 配置中使用自定义的插件：
module.exports = {
  // …

  plugins: [
    // 添加插件，使用默认选项
    new FileListPlugin(),

    // 或者:

    // 使用任意支持的选项
    new FileListPlugin({
      outputFile: 'my-assets.md',
    }),
  ],
};
```
