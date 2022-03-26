---
title: 带你了解Webpack
date: 2022-03-18 19:08:07
---

## 前言
Webpack 凭借强大的功能与良好的使用体验，已经成为目前最流行，社区最活跃的构建工具，是现代 Web 开发必须掌握的技能之一。

> 构建其实是工程化、自动化思想在前端开发中的体现，把一系列流程用代码去实现，让代码自动化地执行这一系列复杂的流程。 构建给前端开发注入了更大的活力，解放了我们的生产力。


## 背景

在当下的前端环境里，各种框架和工具层出不穷，比如 React、Vue、Angular 等，极大的提高了我们的开发效率，但是，他们都有一个共同点：源代码无法直接运行，必须经过转换之后才可执行。

而转换代码的这个过程我们可以称之为构建，被用来进行构建的工具我们叫做构建工具，而 Webpack 便是其中的佼佼者。

构建工具的常规作用：

- 代码转换：TypeScript 编译成 JavaScript、SCSS 编译成 CSS 等。

- 文件优化：压缩 JavaScript、CSS、HTML 代码，压缩合并图片等。

- 代码分割：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。

- 模块合并：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。

- 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器。

- 代码校验：在代码被提交到仓库前需要校验代码是否符合规范，以及单元测试是否通过。

- 自动发布：更新完代码后，自动构建出线上发布代码并传输给发布系统。

## Webpack 的基本概念

Webpack 是使用 NodeJs 开发出来的一个构建工具，本质上，它是一个现代 JavaScript 应用程序的静态模块打包器（module bundler）。

当 Webpack 处理应用程序时，它会递归地构建一个依赖关系图（dependency graph），其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

在 Webpack 里一切文件皆模块，通过 Loader 转换文件，通过 Plugin 注入钩子，最后输出由多个模块组合成的文件。

Webpack 专注于构建模块化项目。借用 Webpack 官网首页的图片来看一下它到底是什么

![](https://static001.geekbang.org/infoq/97/979f6d8773003f91ddbfd0a59e1d5010.png)

> 一切文件：JavaScript、CSS、SCSS、图片、模板，在 Webpack 眼中都是一个个模块，这样的好处是能清晰地描述出各个模块之间的依赖关系，以方便 Webpack 对模块进行组合和打包。 经过 Webpack 的处理，最终会输出浏览器能使用的静态资源。

## Webpack 的基本配置
以下是 Webpack 的基本配置，主要包含了 webpack 的四个核心概念：

- 入口(entry)

- 输出(output)

- loader

- 插件(plugins)

```
const path = require('path');

module.exports = {
    // 模式配置
    mode: "production", // "production" | "development" | "none"

    // 入口文件
    entry: "./app/entry", // string | object | array

    output: {
        // webpack 如何输出结果的相关选项
        path: path.resolve(__dirname,
            "dist"), // string
    },

    module: {
        // 关于模块配置
        rules: [
            // 模块规则（配置 loader、解析器等选项）
            {
                test: /\.jsx?$/,
            }
        ]
    },

    // 插件
    plugins: [
        // ...
    ],
}
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

## 前端工程化

##### Webpack 的构建流程主要有哪些环节？如果可以请尽可能详尽的描述 Webpack 打包的整个过程。



- entry-option(初始化 option)

- run(开始编译)

- make(从 entry 开始递归的分析依赖，对每个依赖模块进行 build)

- before-resolve(对模块位置进行解析)

- build-modue(开始构建某个模块)

- normal-modue-loader(将 load 而加载完成的 module 进行编译，生成 AST 树)

- program(遍历 AST，当遇到 require 等一些调用表达式时，收集依赖)

- seal(所有依赖 build 完成，开始优化)

- emit(输出到 dist 目录)



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