---
title: 自定义webpack插件
date: 2021-03-17 14:04:14
tags:
- webpack
categories: webpack
author: 马金坤
keywords: webpack, 插件
description: 自定义webpack插件
cover: https://img13.360buyimg.com/imagetools/jfs/t1/156873/8/16203/7614/6051ba11Ede2bb8a0/3bb49e97a851255c.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/130200/34/17431/262954/5fc092d7E0b54491c/bb832c9742a8f536.png
---
### 前言
webpack 自身会提供一些基础插件，比如分析、压缩、html、 provider等。有时，我们还需要自定义一些插件，来满足我们日常开发需要。下面介绍如何自定义 webpack 插件。

### 插件基本结构
自定义插件大概包含以下几个步骤：
* webpack 插件其实就是一个构造函数，所以要先定义一个类函数；
* 在构造函数的原型上定义 apply 方法，传入 compiler 对象；
* 通过 compiler 对象，可以插入指定的事件钩子；
* 在钩子回调中，可以拿到 compilation 对象，使用 compilation 操纵修改webapack内部实例数据；
* 实现功能后，调用 Webpack 提供的 callback  

那 compiler 对象和 compilation 对象是什么呢？
* compiler 对象 包含webpack的所有配置信息（webpack.config.js），包括 options，loader 和 plugin，包括作为 webpack 的实例在启动时被初始化。
* compilation 对象 包含当前的模块资源、编译生成资源、变化的文件、以及被跟踪依赖的状态信息。当文件发生变化，就会有一个新的 compilation。compilation 对象也提供了许多事件回调钩子

#### compiler 钩子
##### 钩子的用法
```js
// someHook 钩子名称
// webpack 4
compiler.hooks.someHook.tap('MyPlugin', (res) => {
  /* ... */
})
// webpack 2
compiler.plugin(someHook, (res) => {
  /* ... */
})
```
#####  常用钩子介绍
* environment 在编译器准备环境时调用，时机就在配置文件中初始化插件之后
* afterEnvironment 当编译器环境设置完成后，在 environment hook 后直接调用
* entryOption 在 webpack 选项中的 entry 被处理过之后调用
* afterPlugins 在初始化内部插件集合完成设置之后调用，回调参数 context 和 entry
* afterCompile compilation 结束和封印之后执行。回调参数：compilation
* emit 输出 asset 到 output 目录之前执行。回调参数：compilation
* afterEmit 输出 asset 到 output 目录之后执行。回调参数：compilation
* done 在 compilation 完成时执行。回调参数：stats
* failed 在 compilation 失败时调用。回调参数：error  

全部 compiler 钩子用法请见 [compiler 钩子](https://webpack.docschina.org/api/compiler-hooks/)

#### compilation 钩子
##### 钩子的用法
```js
// someHook 钩子名称
// webpack 4
compilation.hooks.someHook.tap('MyPlugin', (res) => {
  /* ... */
})
// webpack 2
compilation.plugin(someHook, (res) => {
  /* ... */
})
```
#####  常用钩子介绍
* buildModule 在模块构建开始之前触发，可以用来修改模块。
* rebuildModule 在重新构建一个模块之前触发。
* failedModule 模块构建失败时执行。
* succeedModule 模块构建成功时执行。
* finishModules 所有模块都完成构建并且没有错误时执行。  

全部 compilation 钩子用法请见 [compilation 钩子](https://webpack.docschina.org/api/compilation-hooks/)

### 如何写插件
**插件代码**
```
class MyWebpackPlugin {
  constructor(options) {}
  apply(compiler) {
    // 插入钩子函数，里面加入
    compiler.hooks.emit.tap('MyWebpackPlugin', (compilation) => {
    console.log('Hello World!')
    })
  }
}
module.exports = MyWebpackPlugin;
```
**使用插件**  

在 webpack.config.js 中引入插件：
```js
module.exports = {
  plugins:[
    // 传入插件实例
    new MyWebpackPlugin()
  ]
}
```
### 插件示例
下面介绍的自定义的TestBuildPlugin插件，该插件是代码打包检查工具，通过一些简单的配置，在打包的过程中检查，检查webpack的配置项是否正确，打包后的代码是否符合要求。
**TestBuildPlugin 插件代码**
```js
const path = require('path')
const resolvePath = dir => path.join(path.resolve('./'), dir)
const testBuildConfig = require(resolvePath('.testBuildConfig.js')) || {}

class TestBuildPlugin {
  constructor (args) {}
  apply (compiler) {
    const { testBuild } = process.env || {}
    if (testBuild === '1') {
      // 判断webpack打包的配置项是否正确
      if (!this.testConfigValCorrect(compiler.options, testBuildConfig.options)) {
        throw new Error('options config err!')
      }
      // webpack4的写法
      if (compiler.hooks) {
        // 生成资源到output目录之前
        compiler.hooks.emit.tapAsync('TestBuildPlugin', this.emitFn.bind(this))
      } else { // 版本适配
        compiler.plugin('emit', this.emitFn.bind(this))
      }
      if (compiler.hooks) {
        // 生成资源到output目录之前
        compiler.hooks.done.tap('TestBuildPlugin', this.doneFn.bind(this))
      } else { // 版本适配
        compiler.plugin('done', this.doneFn.bind(this))
      }
    }
  }
  // emit 钩子的回调函数
  emitFn(compilation, callback) {
    // console.error(new Error(''));
    // process.exit(1)
    const assets = compilation.assets || {}
    const assetsArr = Object.keys(assets)
    if (assets && assetsArr && assetsArr.length) {
      const { mustHave, mustForbidden } = testBuildConfig.codeRules
      assetsArr.forEach((key) => {
        const asset = assets[key]
        const source = asset.source()
        const pathName = key
        // 判断必须禁止的代码
        const mustForbiddenRes = this.testCodeCorrect({
          codeRule: mustForbidden,
          type: 1,  // type为1，表示必须禁止的；type为2，表示必须包括的
          name: pathName, // 文件名
          source
        })
        if (!mustForbiddenRes.isCorrect) {
          callback(new Error(`${mustForbiddenRes.errItem} must be forbidden\nError detail: ${mustForbiddenRes.errItem} occurred in ${pathName} `))
        }
        // 判断必须包含的代码
        const mustHaveRes = this.testCodeCorrect({
          codeRule: mustHave,
          type: 2,  // type为1，表示必须禁止的；type为2，表示必须包括的
          name: pathName, // 文件名
          source
        })
        if (!mustHaveRes.isCorrect) {
          callback(new Error(`${mustHaveRes.errItem} must be have\nError detail: err occurred in ${pathName} `))
        }
      })
    } else {
      callback(new Error(`build err!`))
    }
    callback()
  }
  // done 钩子的回调函数
  doneFn() {
    console.log('test build passed!!')
  }
  // 判断配置是有一样
  testConfigValCorrect (buildVal, configVal) {
    if (!configVal) {
      return true
    }
    if (!buildVal) {
      return false
    }
    let correct = true
    if (this.getValType(configVal) === '[object Array]') {
      if (this.getValType(buildVal) !== '[object Array]') {
        return false
      }
      configVal.every((item, index) => {
        return correct = this.testConfigValCorrect(buildVal[index], configVal[index])
      })
    } else if (this.getValType(configVal) === '[object Object]') {
      if (this.getValType(buildVal) !== '[object Object]') {
        return false
      }
      Object.keys(configVal)
        .every((key) => {
          return correct = this.testConfigValCorrect(buildVal[key], configVal[key])
        })
    } else {
      return buildVal === configVal
    }
    return !!correct
  }

  // 判断代码是否正确
  testCodeCorrect ({
    codeRule,
    type,  // type为1，表示必须禁止的；type为2，表示必须包括的
    name, // 文件名
    source // 文件路径
  }) {
    let errItem = null
    let result = {
      isCorrect: true
    }
    if (codeRule && codeRule.length) {
      codeRule.every((codeRuleItem) => {
        if (codeRuleItem) {
          let { test, val } = codeRuleItem
          test = test ? this.getRegExpTypeVal(test) : /\.(js|html)$/
          val = val ? this.getArrayTypeVal(val) : null
          if (test.test(name)) {
            if (val && val.length) {
              val.every(valItem => {
                const itemRegExp = this.getRegExpTypeVal(valItem)
                if (itemRegExp) {
                  if ((itemRegExp.test(source) && type === 1)
                    || (!itemRegExp.test(source) && type === 2)
                  ) {
                    errItem = valItem
                    return false
                  }
                }
              })
            }
          }
        }
        return !errItem
      })
    }
    if (errItem) {
      result = {
        isCorrect: false,
        errItem
      }
    }
    return result
  }

  // 获取类型
  getValType (val) {
    if (!val) {
      return null
    }
    return Object.prototype.toString.call(val)
  }

  // 获取RegExp类型的值
  getRegExpTypeVal (val) {
    if (this.getValType(val) !== '[object RegExp]') {
      return new RegExp(val)
    }
    return val
  }

  // 获取数组类型的值
  getArrayTypeVal (val) {
    return this.getValType(val) === '[object Array]' ? val : [val]
  }
}

module.exports = TestBuildPlugin
```
**.testBuildConfig.js配置打包时的检查项**
```js
const path = require('path')
const resolvePath = dir => path.join(__dirname, dir)
module.exports = {
  // webpack 打包配置项
  options: {
    entry: {
      app: ''
    },
    output: {
      path: resolvePath(''),
      publicPath: ''
    }
  },
  codeRules: {
    // 用来配置必须有的代码，test: 检测的范围；val: 代码
    mustHave: [{
      test: /index\.html$/,
      val: ['abcdefg']
    }],
    // 用来配置必须禁止的代码，test: 检测的范围；val: 代码
    mustForbidden: [{
      test: /\.(js|html)$/,
      val: ['beta-api.m.jd.com']
    }]
  }
}
```
