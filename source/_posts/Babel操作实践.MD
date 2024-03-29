---
title: Babel 操作实践
date: 2023-03-17 17:00:00
tags:
- Babel
cover: https://img10.360buyimg.com/imagetools/jfs/t1/3527/34/19935/156961/6413c16eF119e2beb/fa40823024d09659.png
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/3527/34/19935/156961/6413c16eF119e2beb/fa40823024d09659.png
---

现在很多前端的脚手架都提供了开箱即用的配置，开发人员即使在不清楚配置的情况下，只要按照文档说明，也能一步步运行并使用。在这样的背景下，脚手架的实现细节和各种依赖逐渐被大家忽略，最终导致的结果是很多时候我们知道一个配置文件是做什么的，但对其中的配置项所知甚少，如果遇到对应的依赖项需要升级、或者按照官网文档运行时报错，就无从下手了。因此，了解常用依赖的作用、常见配置甚至实现原理，仍然是一件必要的事情。这不仅有助于我们解决遇到的配置问题，或许还可以为我们开发前端工具提供思路。

Babel 是我们开发中都会用到但都不太关注的一个工具，我们大概都知道，现在我们可以按照新的 js 语法写代码而不用担心适配问题，是因为 Babel 将我们的 js 转换成了可以适配低版本浏览器的代码，但如何在项目中配置 Babel、配置时该使用哪种类型的 Babel等类似的问题，仍旧会困扰我们。

# Babel 是什么

babel /ˈbeɪbl/ 是巴别塔（也叫通天塔）的意思，来自圣经中的典故：当时人类联合起来兴建希望能通往天堂的高塔，为了阻止人类的计划，上帝让人类说不同的语言，使人类相互之间不能沟通，计划因此失败，人类自此各散东西。这座塔就是巴别塔。

Babel 的前身是 6to5，发布于 2014年，作者是 Facebook 的工程师 Sebastian McKenzie。从名字就能看出，它的作用就是将 ES6 转换成 ES5 的语法。当时，6to5 的转换AST的引擎是 fork 的 acorn。后来在 2015 年，6to5 和 ESNext 库的团队决定一起开发 6to5，因为 6to5 看起来像是个临时的过渡方案，不能充分展示 6to5 会按照 ES 标准逐步完善、并反过来影响推进 ES 标准制定的项目目标，因此将 6to5 改名为 Babel。从意义上讲，Babel 就是需要解决不同 ES 语法和不同浏览器之间沟通问题的一座通天塔。

Babel 是什么？下面是 Babel 官网的说明：
- Babel 是一个 JavaScript 编译器。
- Babel 是一个工具链，主要用于将采用 ECMAScript 2015+ 语法编写的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。

注意：这里没有说转换成 ES5 的语法，是因为不同类型以及不同版本的浏览器对 ES6 新特性的支持程度都不一样，对于浏览器已经支持的部分，Babel 可以不转化。

下面举一个简单的例子。
我们常用的箭头函数在 IE 浏览器中是不支持的：
![arrow function 兼容性](https://img13.360buyimg.com/imagetools/jfs/t1/170159/30/35510/189721/6413cca1F096d4e0f/379b2194edaba841.png)

下面这段代码在 IE 11 中运行时会报语法错误：
``` js
    const sum = (a, b) => a + b
```
![箭头函数在 IE11 中报错](https://img14.360buyimg.com/imagetools/jfs/t1/171658/39/34153/6525/640f1683F212c6977/a72b9b2d4248b1e7.jpg)

经过 Babel 编译，箭头函数被转换成了可以在 IE 11 中正确运行的 ES5 的语法：
``` js
    var sum = function sum(a, b) {
        return a + b;
    };
```
# Babel 的版本
- 2014，6to5 发布
- 2015-02-15，6to5重命名为 babel
- 2015-03-31，babel 5.0发布
- 2015-10-30，babel 6.0发布
- 2018-08-27, babel 7.0发布，目前最新版本是 7.21.0

# Babel 的执行步骤

Babel 的执行分为三个步骤：
![babel 执行过程](https://img10.360buyimg.com/imagetools/jfs/t1/162023/30/36116/8517/6413d03fF1443b593/d95a3a2e61c2e339.png)

# Babel 可以做什么

1. 将代码转成 ES5 语法
- 转译 esnext、typescript、flow 等代码到目标环境支持的 js，并且还可以把目标环境不支持的 api 进行 polyfill。

2. 特定用途的转换，转成特定目标代码
babel 是一个转译器，暴露了很多 api，用这些 api 可以完成代码到 AST 的 parse，AST 的转换，以及目标代码的生成，开发者可以用它来来完成一些特定用途的转换
- 比如函数插桩（函数中自动插入一些代码，例如[埋点代码](https://blog.csdn.net/weixin_44800563/article/details/125726496)、代码覆盖率统计)
- [自动国际化](https://juejin.cn/post/7128724544717586468)
- default import 转 named import 等
- 现在比较流行的小程序转译工具 taro，就是基于 babel 的 api 来实现的
- prettier体系中，用于格式化 js 文件的功能，用的就是babel 的 api

3. 分析代码信息，做静态检查
对代码进行 parse 之后，能够进行转换，是因为通过 AST 的结构能够理解代码。理解了代码之后，除了进行转换然后生成目标代码之外，也同样可以用于分析代码的信息，进行一些检查。
- linter 工具：分析 AST 的结构并对代码规范进行检查（最常见的是 eslint）。
- api 文档自动生成工具：可以提取源码中的注释，然后生成文档（比如jsdoc）。
- type checker：会根据从 AST 中提取的或者推导的类型信息，对 AST 进行类型是否一致的检查，从而减少运行时因类型导致的错误。
- 压缩混淆工具：这个也是分析代码结构，进行删除死代码、变量名混淆、常量折叠等各种编译优化，生成体积更小、性能更优的代码。
- js 解释器：除了对 AST 进行各种信息的提取和检查以外，我们还可以直接解释执行 AST。也就是在 js 运行时，递归遍历ast节点，根据不同的节点类型，执行对应的操作。

# 使用 Babel

使用 Babel 时，有以下三种方式：

1. 直接使用（babel 提供的单个 script）
2. 使用命令行 (babel 提供的 @babel/cli)
3. 作为构建工具的插件 (webpack 的 babel-loader, rollup 的 rollup-plugin-babel)

第 2 种和第 3 种是工作中常见的，但其本质都是调用的 babel 的内核。我们先来了解下如何直接使用 babel 的核心库来实现代码编译以及如何使用 Babel 的插件等。

## 直接使用

1. parse: 通过 @babel/parser 解析代码，生成抽象语法树（即 AST）。
2. transform: 转换过程其实是对 AST 进行一系列的变换操作。事实上 Babel 中并没有 @babel/transform 这个包，Babel 的转换功能是通过一个个插件实现的。前面我们在介绍 Babel 是什么的时候说过，Babel 是一个工具链，这个工具链是依赖于插件的，当我们不配置任何插件时，经过 Babel 的代码和输入是相同的。Babel 中的插件可以通过 @babel/traverse 对节点进行遍历，并结合 @babel/types 对节点进行检查、转换等操作。 
3. generate: 通过 @babel/generator 将变换后的 AST 再转换为 js 代码。

注意：
- babel 7.x 把所有 babel-XXX 重命名为 @babel/XXX。以 @babel/generator 为例，Babel 7.x 之后，使用 @babel/generator，Babel 6.x 以及更早的版本需要使用 babel-generator。
- Babel 在 7.x 之前没有 babel-parser， 而是 babel/babylon（基于 acorn 和 acorn-jsx），在 7.x 移入 @babel/parser

还是以开头的箭头函数为例，我们使用 parse 和 generate 这两个方法对箭头函数进行转换：
``` js
    const { parse } = require('@babel/parser')
    const { default: generate } = require('@babel/generator')

    const sumString = `const sum = (a, b) => a + b`
    const parseAndGenerate = code => {
        const ast = parse(code)
        const targetCode = generate(ast)
        console.log(targetCode.code) 
    }
    parseAndGenerate(sumString)
```
结果如下：
``` js
    const sum = (a, b) => a + b;
```
和原始代码对比，转换后的代码只在末尾增加了一个分号，箭头函数并没有被转换。

下面我们暂时使用 traverse 方法代替 Babel 的插件，对 AST 进行遍历并转换：
``` js
const { parse } = require('@babel/parser')
const { default: traverse} = require('@babel/traverse')
const { default: generate } = require('@babel/generator')
const types = require('@babel/types')
const sumString = `const sum = (a, b) => a + b`

const parseAndTransAndGenerate = code => {
    const ast = parse(code)
    traverse(ast, {
        VariableDeclaration(path){
            const node = path.node;
            if(['const', 'let'].includes(node.kind)) {
                node.kind = 'var'
            }
        },
        ArrowFunctionExpression(path) {
            let {id, params, body, generator, async} = path.node
            if (!types.isBlockStatement(body)) {
                body = types.blockStatement([types.returnStatement(body)])
            }
            path.replaceWith(types.functionExpression(id, params,body, generator, async))
        }
    })
    const targetCode = generate(ast)
    console.log(targetCode.code)
}
parseAndTransAndGenerate(sumString)
```
在 traverse 方法中我们做了两步处理：在 VariableDeclaration 中将 const 处理为 var，在 ArrowFunctionExpression 中将箭头函数处理为普通的 functionExpression。从下面的结果可以看出，箭头函数被转换成了普通的 function：

``` js
    var sum = function (a, b) {
        return a + b;
    };
```

### Babel 的插件 Plugins

Babel 的插件是 Babel 转换的核心。上面的例子中，我们使用 Babel 了提供的四个 package： @babel/parse、@babel/traverse、@babel/types、@babel/generator 对代码进行解析和生成，如果每次转换都要引这四个 package 的话就太麻烦了，所以就有了 @babel/core，这个 package 是 Babel 的最核心的，它将底层的 package 进行封装，并另外加入了其他功能，比如读取、分析配置文件、生成 source map 等。下图是 @babel/core 安装的依赖，可以看到，我们上面代码中用到的四个 package 全部被包含进来了。

![@babel/core](https://img13.360buyimg.com/imagetools/jfs/t1/92859/35/25158/226320/640f2291F1745f7a3/dcd94a0d49a01c64.png)

我们可以用 @babel/core 中的 transfrom 方法来简化上面的逻辑，transform 方法接收三个参数，第一个参数是要转换的代码，第三个参数是转换后的回调方法，第三个比较重要，是转换过程的配置项（[配置项列表](https://babeljs.io/docs/options)），我们可以在配置项中定义 plugins字段来加载负责转换逻辑的插件。
![transform](https://img10.360buyimg.com/imagetools/jfs/t1/182226/8/32909/99902/640f25caF8e91a507/1e7831adaae9e77a.png)


Babel 的插件本质上是一个 function，接收三个参数：
- api：一个对象，包含了 types (@babel/types)、traverse (@babel/traverse)、template(@babel/template) 等方法，我们可以在这里访问到 @babel/core 中包含的依赖
- options：插件参数。
- dirname：目录名。

Babel 插件返回的对象如下：
- name：插件名字。
- inherits：指定继承某个插件，通过 Object.assign 的方式，和当前插件的 options 合并。
- visitor：指定 traverse 时调用的函数。
- pre 和 post 分别在遍历前后调用，可以做一些插件调用前后的逻辑，比如可以往 file（表示文件的对象，在插件里面通过 state.file 拿到）中放一些东西，在遍历的过程中取出来。
- manipulateOptions：用于修改 options，是在插件里面修改配置的方式。

注意：Babel 插件的返回对象必须要包含在上面列举的几个对象里，否则会报错：```Error: [BABEL] unknown file: .test is not a valid Plugin property```

我们先把上面例子中的 transform 的过程提取成插件（不清楚 AST 节点类型的话可以在这里查： https://astexplorer.net/）：
``` js
    const babel = require("@babel/core")

    const sumString = `const sum = (a, b) => a + b`
    babel.transform(sumString, { 
        plugins: [
            function ({ types }) {
                return {
                visitor: {
                    VariableDeclaration(path) {
                        const node = path.node;
                        if (['const', 'let'].includes(node.kind)) {
                            node.kind = 'var'
                        }
                    },
                    ArrowFunctionExpression(path) {
                        let { id, params, body, generator, async } = path.node
                        if (!types.isBlockStatement(body)) {
                            body = types.blockStatement([types.returnStatement(body)])
                        }
                        path.replaceWith(types.functionExpression(id, params, body, generator, async))
                    }
                }
                }
            }
        ]
    }, (err, result) => {
        if (err) {
            console.log(err)
        } else if (result) {
            console.log(result.code)
        }
    })
```

为了让插件能够得到复用，我们可以将插件单独提取到一个 js 文件中：

```js
    // transform-arrow-function-plugin.js
    module.exports = ({ types }) => {
        return {
            name: '@scope/babel-plugin-transform-arrow-function',
            visitor: {
                VariableDeclaration(path) {
                    const node = path.node;
                    if (['const', 'let'].includes(node.kind)) {
                    node.kind = 'var'
                    }
                },
                ArrowFunctionExpression(path) {
                    let { id, params, body, generator, async } = path.node
                    if (!types.isBlockStatement(body)) {
                    body = types.blockStatement([types.returnStatement(body)])
                    }
                    path.replaceWith(types.functionExpression(id, params, body, generator, async))
                }
            }
        }
    }
```
Babel 可以按照下面的方式引入插件：
- 按照相对路径引入：
``` js
    const babel = require("@babel/core")

    const sumString = `const sum = (a, b) => a + b`
    babel.transform(sumString, { 
        plugins: ['./transform-arrow-function-plugin.js']
    }, (err, result) => {
        if (err) {
            console.log(err)
        } else if (result) {
            console.log(result.code)
        }
    })

```
- npm包引入：如果插件是以 npm 包的形式引入的，需要将 name 传给 plugins，Babel 会去 node_modules 文件夹下检查是否安装了对应的插件。
``` js
    const babel = require("@babel/core")
    const sumString = `const sum = (a, b) => a + b`
    babel.transform(sumString, { 
        plugins: ['@scope/babel-plugin-transform-arrow-function']
    }, (err, result) => {
        if (err) {
            console.log(err)
        } else if (result) {
            console.log(result.code)
        }
    })
```

我们使用 npm 引入 Babel 插件的时候需要注意：Babel 在加载插件时会默认在名称前面加上 babel-plugin- 作为前缀，如果我们发布的 npm 包不是以 babel-plugin- 开头，Babel 加上前缀后就和我们发布时候的包名不一致，运行时就会提示 Cannot find module。在上面的例子中，@paimai/babel-plugin-transform-arrow-function 和 @paimai/transform-arrow-function 是等价的。

Babel 插件名称补全规则如下：
![插件名称规则](https://img13.360buyimg.com/imagetools/jfs/t1/158653/30/35979/86238/64140ea1F29d30109/4132f31bed25cc20.png)

### Babel 的预设 Presets

Presets 是一组插件的集合。如果我们按单个功能去引入插件，如果要把代码转换成 ES5 规范的，就需要引入很多个插件，直接使用 presets 就会方便许多。

和插件一样，Babel 的 preset 也是一个 function，返回一组 plugin 或者 preset。我们增加一个在 function 中插入 log 的插件: 

``` js
    module.exports = ({ template }) => {
        return {
            name: '@scope/babel-plugin-insert-log-function',
            visitor: {
                FunctionExpression(path) {
                    const ast = template.ast(`
                    console.log('log inserted in FunctionExpression')
                    `)
                    path.node.body.body.unshift(ast)
                },
                ArrowFunctionExpression(path) {
                    const ast = template.ast(`
                    console.log('log inserted ArrowFunctionExpression')
                    `)
                    path.node.body.body.unshift(ast)
                },
            }
        }
    }
```

之后再增加一个 preset，引入我们上面创建的两个插件（babel-plugin-transform-arrow-function、babel-plugin-insert-log-function）：

``` js
    // transform-arrow-log-presets
    module.exports = function () {
        return {
            plugins: ['./transform-arrow-function-plugin.js', './transform-insert-log-plugin.js']
        };
    };
    // 使用 preset
    const babel = require("@babel/core")

    const sumString = `const sum = (a, b) => a + b`
    babel.transform(sumString, { 
        presets: ['./transform-arrow-log-presets.js']
    }, (err, result) => {
        if (err) {
            console.log(err)
        } else if (result) {
            console.log(result.code)
        }
    })
```

目前 preset 可以分为下面几类：

1. stage-x：包含当年最新规范的草案，每年更新，但是在 Babel 从 7.0.0-beta.55 开始废弃了所有的 stage。如果想要使用最新的规范，可以自定义 preset 来引入对应的 plugin。
 - Stage 0 - 稻草人: 只是一个想法，经过 TC39 成员提出即可。
 - Stage 1 - 提案: 初步尝试。
 - Stage 2 - 初稿: 完成初步规范。
 - Stage 3 - 候选: 完成规范和浏览器初步实现。
 - Stage 4 - 完成: 将被添加到下一年度发布。
2. 官方 presets：Babel 官网给出了常用的几个 preset：
- @babel/preset-env：将 js 转换成目标环境所支持的语法，包含 polyfill（@babel/preset-env 不包含任何未进入 Stage 3 阶段的 JavaScript 语法提案，因为在 TC39 的流程中，未进入 Stage 3 阶段的提案是不会被任何浏览器所实现的。如果确有需要，请手动设置。通过设置 shippedProposals 参数可以包含进入 Stage 3 阶段并且已经被部分浏览器实现的提案）
- @babel/preset-react：转换 react 代码，把 jsx 编译成 React.createElement() 方法
- @babel/preset-typescript： 转换 typescript
- @babel/preset-flow：转换 flow（flow 是和 typescript 类似的静态类型检查器）
3. es201x, latest：包含 ES 的标准规范
我们在工作中常见的应该是 es2015，但是 es2015 / es2016/ es2017 等 presets 在 2017 年 9 月已经被宣布废弃，可以使用 @babel/preset-env 来代替。latest（每年更新，包含所有 es201x） 也是因为 env 的出现被废弃掉。

### Plugins 和 Presets 的顺序

如果使用多个 plugin 或者 preset，执行顺序如下：
- Babel 的 plugins 按照从前到后的顺序执行。
- Babel 的 presets 的执行顺序和 plugins 不同，按照从后到前的顺序执行（这主要是为了确保向后兼容，由于大多数用户将 "es2015" 放在 "stage-0" 之前）。 
- 如果同时配置了 plugins 和 presets，先执行 presets。

### Plugins 和 Presets 的参数

Babel 中的插件和预设都支持传入参数，需要传入参数时，可以把插件名字的和参数结合成一个数组，再放到 plugins 或 presets 中。

我们给上文中的 log 插件增加一个 text 的参数：
``` js
    module.exports = ({ template }, {text}) => {
        return {
            name: '@scope/babel-plugin-insert-log-function',
            visitor: {
                FunctionExpression(path) {
                    const ast = template.ast(`
                    console.log('${text || 'log inserted in FunctionExpression'}')
                    `)
                    path.node.body.body.unshift(ast)
                },
                ArrowFunctionExpression(path) {
                    const ast = template.ast(`
                    console.log('${text || 'log inserted ArrowFunctionExpression'}')
                    `)
                    path.node.body.body.unshift(ast)
                },
            }
        }
    }
```

使用时要把 babel-plugin-insert-log-function 定义为一个数组，在数组的第二位增加参数：

``` js
    module.exports = function () {
        return {
            plugins: [
                './transform-arrow-function-plugin.js', 
                ['./transform-insert-log-plugin.js', {
                    text: 'custom text'
                }]]
        };
    };

```

### 常用预设 @babel/preset-env

因为 preset-env 是我们常用的，这里着重介绍一下。

env 的核心是根据配置的目标信息做必要的转换。例如目标浏览器支持箭头函数，那么 Babel 就不会对箭头函数进行转换，这样可以减少转换的时间，也可以让生成的代码更小些。如果不写任何配置项，env 就会假定你需要兼容最旧版本的浏览器，就会将 ES2015-ES2020 的代码全部转换成 ES5。上文中的箭头函数经过 @babel/preset-env 的默认转换后如下（[use strict 严格模式](https://www.w3school.com.cn/js/js_strict.asp)）：
``` js
    "use strict";

    var sum = function sum(a, b) {
        return a + b;
    };
```

如果要指定浏览器版本，需要在配置 preset-env 的 target 选项。因为 @babel/preset-env 中 @babel/helper-compilation-targets 是在 browserlist 的基础上构建的，所以 target 可以参照 browserlist 的配置格式（[browserlist](https://github.com/browserslist/browserslist)、[查看 browserlist 对应的浏览器范围](https://browsersl.ist/#)）:
``` js
    {
        "targets": "last 2 versions, safari > 40"
    }
```
另外，也可以指定要支持的最小版本号：
``` js
    {
        "targets": {
            "chrome": "58",
            "ie": "11"
        }
    }
```
我们给 @babel/preset-env 配置支持 chrome 47 以后（和 caniuse 上的兼容性有出入，caniuse 是 45、46也支持），转换后的代码如下：
``` js
    "use strict";

    var sum = (a, b) => a + b;
```

补充：因为 js 生态中的其它工具也要用到 browserlist，例如 autoprefixer、stylelint、eslint-plugin-compat 等，建议将 browserlist 的配置单独提取到 .browserslistrc 文件或者在 package.json中配置 browserlist 字段。

.browserslistrc 配置示例：
```
    last 2 versions
    safari > 40
```

package.json 配置示例：
```
  "browserslist": ["last 2 versions", "safari > 40"],
```

### 常用预设 @babel/polyfill

Babel 把 ES6 的标准分为 syntax 和 built-in 两种类型。syntax 就是语法，我们开头示例中的箭头函数的转换就是 syntax 的类型。而对于那些可以通过改写覆盖的语法就认为是 built-in，像 includes 和 Promise 这些都属于 built-in。Babel 默认只转译 syntax 类型的，对于 built-in 类型的就需要通过 @babel/polyfill 来完成转译。@babel/polyfill 就是浏览器 API 的垫片，通过引入这些垫片，使得低版本的浏览器能模拟实现那些新的 API。

比如 Array.includes 方法在 IE 中不能使用：
![Array.includes 兼容性](https://img10.360buyimg.com/imagetools/jfs/t1/161351/12/29801/214035/64130406Fadccc9f7/c174f49e04ecdd08.png)

我们用 Babel 来转换 includes 方法：
``` js
    const babel = require("@babel/core")

    const sumString = `console.log([1, 2, 3].includes(1))`
    babel.transform(sumString, { 
        presets: ['@babel/preset-env']
    }, (err, result) => {
        if (err) {
            console.log(err)
        } else if (result) {
            console.log(result.code)
        }
    })
```
如果用 preset 默认配置，Array.includes 不会被转换，结果如下：
``` js
    "use strict";

    console.log([1, 2, 3].includes(1));
```

babel 6 安装的 babel-polyfill：

```
    "babel-runtime": "^6.26.0",
    "core-js": "^2.5.0",
    "regenerator-runtime": "^0.10.5"
```

babel 7 安装的 @babel/polyfill：
```
    "core-js": "^2.6.5",
    "regenerator-runtime": "^0.13.4"
```

@babel/ployfill 配合 @babel/preset-env 使用，并且要设置 @babel/preset-env 的 useBuiltIns 参数才行。useBuiltIns 参数的配置如下：
- false: 默认值，不注入任何垫片（上面示例的情况）
- entry: 需要在项目入口处引入们，缺点是无论项目中有没有用到，会将所有的垫片引入进来，增加文件的体积。我们在上面示例的基础上增加 useBuiltIns: 'entry'，执行后结果如下：
``` js
    "use strict";

    require("core-js/modules/es6.array.copy-within.js");
    require("core-js/modules/es6.array.every.js");
    require("core-js/modules/es6.array.fill.js");
    require("core-js/modules/es6.array.filter.js");
    require("core-js/modules/es6.array.find.js");
    require("core-js/modules/es6.array.find-index.js");
    require("core-js/modules/es7.array.flat-map.js");
    require("core-js/modules/es6.array.for-each.js");
    require("core-js/modules/es6.array.from.js");
    require("core-js/modules/es7.array.includes.js");
    require("core-js/modules/es6.array.index-of.js");
    require("core-js/modules/es6.array.is-array.js");
    require("core-js/modules/es6.array.iterator.js");
    require("core-js/modules/es6.array.last-index-of.js");
    require("core-js/modules/es6.array.map.js");
    require("core-js/modules/es6.array.of.js");
    require("core-js/modules/es6.array.reduce.js");
    require("core-js/modules/es6.array.reduce-right.js");
    require("core-js/modules/es6.array.slice.js");
    require("core-js/modules/es6.array.some.js");
    require("core-js/modules/es6.array.sort.js");
    require("core-js/modules/es6.array.species.js");
    require("core-js/modules/es6.date.now.js");
    require("core-js/modules/es6.date.to-iso-string.js");
    require("core-js/modules/es6.date.to-json.js");
    require("core-js/modules/es6.date.to-primitive.js");
    require("core-js/modules/es6.date.to-string.js");
    require("core-js/modules/es6.function.bind.js");
    require("core-js/modules/es6.function.has-instance.js");
    require("core-js/modules/es6.function.name.js");
    require("core-js/modules/es6.map.js");
    require("core-js/modules/es6.math.acosh.js");
    require("core-js/modules/es6.math.asinh.js");
    require("core-js/modules/es6.math.atanh.js");
    require("core-js/modules/es6.math.cbrt.js");
    require("core-js/modules/es6.math.clz32.js");
    require("core-js/modules/es6.math.cosh.js");
    require("core-js/modules/es6.math.expm1.js");
    require("core-js/modules/es6.math.fround.js");
    require("core-js/modules/es6.math.hypot.js");
    require("core-js/modules/es6.math.imul.js");
    require("core-js/modules/es6.math.log1p.js");
    require("core-js/modules/es6.math.log10.js");
    require("core-js/modules/es6.math.log2.js");
    require("core-js/modules/es6.math.sign.js");
    require("core-js/modules/es6.math.sinh.js");
    require("core-js/modules/es6.math.tanh.js");
    require("core-js/modules/es6.math.trunc.js");
    require("core-js/modules/es6.number.constructor.js");
    require("core-js/modules/es6.number.epsilon.js");
    require("core-js/modules/es6.number.is-finite.js");
    require("core-js/modules/es6.number.is-integer.js");
    require("core-js/modules/es6.number.is-nan.js");
    require("core-js/modules/es6.number.is-safe-integer.js");
    require("core-js/modules/es6.number.max-safe-integer.js");
    require("core-js/modules/es6.number.min-safe-integer.js");
    require("core-js/modules/es6.number.parse-float.js");
    require("core-js/modules/es6.number.parse-int.js");
    require("core-js/modules/es6.object.assign.js");
    require("core-js/modules/es6.object.create.js");
    require("core-js/modules/es7.object.define-getter.js");
    require("core-js/modules/es7.object.define-setter.js");
    require("core-js/modules/es6.object.define-property.js");
    require("core-js/modules/es6.object.define-properties.js");
    require("core-js/modules/es7.object.entries.js");
    require("core-js/modules/es6.object.freeze.js");
    require("core-js/modules/es6.object.get-own-property-descriptor.js");
    require("core-js/modules/es7.object.get-own-property-descriptors.js");
    require("core-js/modules/es6.object.get-own-property-names.js");
    require("core-js/modules/es6.object.get-prototype-of.js");
    require("core-js/modules/es7.object.lookup-getter.js");
    require("core-js/modules/es7.object.lookup-setter.js");
    require("core-js/modules/es6.object.prevent-extensions.js");
    require("core-js/modules/es6.object.to-string.js");
    require("core-js/modules/es6.object.is.js");
    require("core-js/modules/es6.object.is-frozen.js");
    require("core-js/modules/es6.object.is-sealed.js");
    require("core-js/modules/es6.object.is-extensible.js");
    require("core-js/modules/es6.object.keys.js");
    require("core-js/modules/es6.object.seal.js");
    require("core-js/modules/es6.object.set-prototype-of.js");
    require("core-js/modules/es7.object.values.js");
    require("core-js/modules/es6.promise.js");
    require("core-js/modules/es7.promise.finally.js");
    require("core-js/modules/es6.reflect.apply.js");
    require("core-js/modules/es6.reflect.construct.js");
    require("core-js/modules/es6.reflect.define-property.js");
    require("core-js/modules/es6.reflect.delete-property.js");
    require("core-js/modules/es6.reflect.get.js");
    require("core-js/modules/es6.reflect.get-own-property-descriptor.js");
    require("core-js/modules/es6.reflect.get-prototype-of.js");
    require("core-js/modules/es6.reflect.has.js");
    require("core-js/modules/es6.reflect.is-extensible.js");
    require("core-js/modules/es6.reflect.own-keys.js");
    require("core-js/modules/es6.reflect.prevent-extensions.js");
    require("core-js/modules/es6.reflect.set.js");
    require("core-js/modules/es6.reflect.set-prototype-of.js");
    require("core-js/modules/es6.regexp.constructor.js");
    require("core-js/modules/es6.regexp.flags.js");
    require("core-js/modules/es6.regexp.match.js");
    require("core-js/modules/es6.regexp.replace.js");
    require("core-js/modules/es6.regexp.split.js");
    require("core-js/modules/es6.regexp.search.js");
    require("core-js/modules/es6.regexp.to-string.js");
    require("core-js/modules/es6.set.js");
    require("core-js/modules/es6.symbol.js");
    require("core-js/modules/es7.symbol.async-iterator.js");
    require("core-js/modules/es6.string.anchor.js");
    require("core-js/modules/es6.string.big.js");
    require("core-js/modules/es6.string.blink.js");
    require("core-js/modules/es6.string.bold.js");
    require("core-js/modules/es6.string.code-point-at.js");
    require("core-js/modules/es6.string.ends-with.js");
    require("core-js/modules/es6.string.fixed.js");
    require("core-js/modules/es6.string.fontcolor.js");
    require("core-js/modules/es6.string.fontsize.js");
    require("core-js/modules/es6.string.from-code-point.js");
    require("core-js/modules/es6.string.includes.js");
    require("core-js/modules/es6.string.italics.js");
    require("core-js/modules/es6.string.iterator.js");
    require("core-js/modules/es6.string.link.js");
    require("core-js/modules/es7.string.pad-start.js");
    require("core-js/modules/es7.string.pad-end.js");
    require("core-js/modules/es6.string.raw.js");
    require("core-js/modules/es6.string.repeat.js");
    require("core-js/modules/es6.string.small.js");
    require("core-js/modules/es6.string.starts-with.js");
    require("core-js/modules/es6.string.strike.js");
    require("core-js/modules/es6.string.sub.js");
    require("core-js/modules/es6.string.sup.js");
    require("core-js/modules/es6.string.trim.js");
    require("core-js/modules/es7.string.trim-left.js");
    require("core-js/modules/es7.string.trim-right.js");
    require("core-js/modules/es6.typed.array-buffer.js");
    require("core-js/modules/es6.typed.data-view.js");
    require("core-js/modules/es6.typed.int8-array.js");
    require("core-js/modules/es6.typed.uint8-array.js");
    require("core-js/modules/es6.typed.uint8-clamped-array.js");
    require("core-js/modules/es6.typed.int16-array.js");
    require("core-js/modules/es6.typed.uint16-array.js");
    require("core-js/modules/es6.typed.int32-array.js");
    require("core-js/modules/es6.typed.uint32-array.js");
    require("core-js/modules/es6.typed.float32-array.js");
    require("core-js/modules/es6.typed.float64-array.js");
    require("core-js/modules/es6.weak-map.js");
    require("core-js/modules/es6.weak-set.js");
    require("core-js/modules/web.timers.js");
    require("core-js/modules/web.immediate.js");
    require("core-js/modules/web.dom.iterable.js");
    require("regenerator-runtime/runtime.js");
    console.log([1, 2, 3].includes(1));
```
- usage：不用在项目入口引入，Babel 会在编译的过程中根据使用情况来选择注入相应的实现。
``` js
    "use strict";

    require("core-js/modules/es7.array.includes.js");
    console.log([1, 2, 3].includes(1));
```

注意：当 useBuiltIns 设置为 entry 或 usage 时，还需要设置 @babel/preset-env 的 corejs 参数（不设置时会默认使用 core-js 2 的版本），用来指定注入 built-in 时使用 core-js 的版本（必须和我们已经安装的 core-js 的版本一致，@babel/polyfill 中 core-js 的版本为 2），否则 Babel 会有如下警告：
```
WARNING (@babel/preset-env): We noticed you're using the `useBuiltIns` option without declaring a core-js version. Currently, we assume version 2.x when no version is passed. Since this default version will likely change in future versions of Babel, we recommend explicitly setting the core-js version you are using via the `corejs` option.

You should also be sure that the version you pass to the `corejs` option matches the version specified in your `package.json`'s `dependencies` section. If it doesn't, you need to run one of the following commands:

  npm install --save core-js@2    npm install --save core-js@3
  yarn add core-js@2              yarn add core-js@3
```

另外，@babel/polyfill 从 Babel 7.4.0 开始，就被废弃了，我们示例中是通过手动安装的，@babel/polyfill 中的 core-js 只支持 2。（如果要 @babel/ployfill 支持 core-js 3，项目中应增加 core-js 3 的依赖，并在 @babel/preset-env 中配置 corejs 为3）

不使用 @babel/polyfill 的话， 我们需要直接引入 core-js。因为 core-js 3 的版本是持续维护的版本，建议大家直接使用 core-js 3。
``` js
    import "core-js/stable";
    import "regenerator-runtime/runtime";
```

### 常用预设 @babel/plugin-transform-runtime

在介绍这个预设之前，我们先来看一个例子：
``` js
    const babel = require("@babel/core")

    const sumString = `
        console.log([1, 2, 3].includes(1))
        class Person {
            name = 'XXX'
        }
    `
    babel.transform(sumString, { 
        presets: [['@babel/preset-env', {
            useBuiltIns: 'usage',
            corejs: 2
        }]]
    }, (err, result) => {
        if (err) {
            console.log(err)
        } else if (result) {
            console.log(result.code)
        }
    })

```

经过 Babel 编译后的结果：

``` js
    "use strict";

    require("core-js/modules/es6.symbol.js");
    require("core-js/modules/es6.number.constructor.js");
    require("core-js/modules/es6.object.define-property.js");
    require("core-js/modules/es6.string.iterator.js");
    require("core-js/modules/es6.object.to-string.js");
    require("core-js/modules/es6.array.iterator.js");
    require("core-js/modules/web.dom.iterable.js");
    function _typeof(obj) { "@babel/helpers - typeof"; return _typeof = "function" == typeof Symbol && "symbol" == typeof Symbol.iterator ? function (obj) { return typeof obj; } : function (obj) { return obj && "function" == typeof Symbol && obj.constructor === Symbol && obj !== Symbol.prototype ? "symbol" : typeof obj; }, _typeof(obj); }
    require("core-js/modules/es7.array.includes.js");
    function _defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, _toPropertyKey(descriptor.key), descriptor); } }
    function _createClass(Constructor, protoProps, staticProps) { if (protoProps) _defineProperties(Constructor.prototype, protoProps); if (staticProps) _defineProperties(Constructor, staticProps); Object.defineProperty(Constructor, "prototype", { writable: false }); return Constructor; }
    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    function _defineProperty(obj, key, value) { key = _toPropertyKey(key); if (key in obj) { Object.defineProperty(obj, key, { value: value, enumerable: true, configurable: true, writable: true }); } else { obj[key] = value; } return obj; }
    function _toPropertyKey(arg) { var key = _toPrimitive(arg, "string"); return _typeof(key) === "symbol" ? key : String(key); }
    function _toPrimitive(input, hint) { if (_typeof(input) !== "object" || input === null) return input; var prim = input[Symbol.toPrimitive]; if (prim !== undefined) { var res = prim.call(input, hint || "default"); if (_typeof(res) !== "object") return res; throw new TypeError("@@toPrimitive must return a primitive value."); } return (hint === "string" ? String : Number)(input); }
    console.log([1, 2, 3].includes(1));
    var Person = /*#__PURE__*/_createClass(function Person() {
        _classCallCheck(this, Person);
        _defineProperty(this, "name", 'cui');
    });
```

在 Babel 编译的过程中，对于 built-in 类型的语法通过 require 对应的 polyfill 模块来兼容，对于 syntax 类型的语法在转译的过程会在当前模块中注入类似 _classCallCheck 和 _defineProperty 的 helper 函数来实现兼容。不同项目如果都各自注入了对应的 helper 方法，整个项目文件就会很大。而且，使用 ployfill 还存在一个问题，那就是 polyfill 是通过重写方法来实现的，会造成全局变量的污染。使用 @babel/plugin-transform-runtime 可以解决这两个问题。

``` js
    npm install @babel/plugin-transform-runtime --save-dev
    npm install @babel/runtime --save 
```
> @babel/plugin-transform-runtime 是编译时使用的，安装为开发依赖，而 @babel/runtime 其实就是 helper 函数的集合，需要被引入到编译后代码中，所以安装为生产依赖

在 Babel 的配置中增加了 之后，编译后的结果如下，原来注入的方法也全部使用 require 的方式引入：
``` js
    "use strict";

    var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");
    var _createClass2 = _interopRequireDefault(require("@babel/runtime/helpers/createClass"));
    var _classCallCheck2 = _interopRequireDefault(require("@babel/runtime/helpers/classCallCheck"));
    var _defineProperty2 = _interopRequireDefault(require("@babel/runtime/helpers/defineProperty"));
    require("core-js/modules/es7.array.includes.js");
    console.log([1, 2, 3].includes(1));
    var Person = /*#__PURE__*/(0, _createClass2["default"])(function Person() {
    (0, _classCallCheck2["default"])(this, Person);
    (0, _defineProperty2["default"])(this, "name", 'cui');
    });
```
从上面的结果可以看出 includes 方法还是通过 require("core-js/modules/es7.array.includes.js") 的方式实现的，为了解决全局变量污染的问题，我们还需要按照下面的步骤去配置：
1. 去掉 preset-env 的 useBuiltIn 配置
2. 配置 @babel/plugin-transform-runtime 的参数 corejs，该参数默认为 false，可以设置为 2 或者 3，分别对应 @babel/runtime-corejs2 和 @babel/runtime-corejs3
3. 去掉 @babel/runtime, 使用 @babel/runtime-corejs3，去掉 core-js
```
    npm uninstall @babel/runtime
    npm install --save @babel/runtime-corejs3
    npm uninstall core-js
```
Babel 转换后的结果如下，编译后产生了一个 _includes 的局部变量：

``` js

    "use strict";

    var _interopRequireDefault = require("@babel/runtime-corejs3/helpers/interopRequireDefault");
    var _createClass2 = _interopRequireDefault(require("@babel/runtime-corejs3/helpers/createClass"));
    var _classCallCheck2 = _interopRequireDefault(require("@babel/runtime-corejs3/helpers/classCallCheck"));
    var _defineProperty2 = _interopRequireDefault(require("@babel/runtime-corejs3/helpers/defineProperty"));
    var _includes = _interopRequireDefault(require("@babel/runtime-corejs3/core-js-stable/instance/includes"));
    var _context;
    console.log((0, _includes["default"])(_context = [1, 2, 3]).call(_context, 1));
    var Person = /*#__PURE__*/(0, _createClass2["default"])(function Person() {
    (0, _classCallCheck2["default"])(this, Person);
    (0, _defineProperty2["default"])(this, "name", 'cui');
    });
```
注意：@babel/plugin-transform-runtime 目前不支持设置 targets。

## Babel 的配置文件

这里仅介绍 js 用到的配置，更多配置参考：[babel config files](https://babeljs.io/docs/config-files)
1. 项目维度: 在 babel 7.x 版本，babel 新增了一个 root 文件夹的概念，root 文件夹指的是当前的工作目录。babel 工作时，会首先在当前文件夹下查找 babel.config.js 或者 babel.config.json 文件作为全局的配置文件，该配置文件同时也会对路径下的 node_modules 下的文件生效。在 babel 6.x 中，如果要对 node_modules 生效，则需要注意下 include 中有没有 node_modules。 
- babel.config.js
- babel.config.json

兼容性：babel.config.json 需要 babel 7.8.0 及以上版本，如果是旧版本，使用 babel.config.js

2. 文件维度: babel 遇到要转换的文件时，会在要转换的这个文件的目录结构上查找是否有 .babelrc 或者 package.json 文件的 babel 字段，没有的话会使用全局的配置，有的话这个配置会和全局配置进行合并。我们可以理解为文件维度的配置为局部配置，可以实现对子目录中的文件进行单独配置。
- .babelrc
- package.json 中的 babel 字段

下面来看一个例子：
root   
├── .babelrc  
├── babel.config.js
├── package.json  
└── src  
&nbsp;&nbsp;&nbsp;&nbsp;├── index.js  
&nbsp;&nbsp;&nbsp;&nbsp;└── subdir  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── .babelrc  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── utils.js  

我们在顶层目录运行 babel src/**/*.js --out-dir dist 。此时会读取当前目录的 babel.config.js 作为全局配置，根据上面的规则：
- src/index.js 会应用 babel.config.js + .babelrc。
- src/subdir/utils.js 会应用 babel.config.js + src/subdir/.babelrc

如果我们在 src 目录执行 babel **/*.js --outdir ，此时则不会应用 babel.config.js 的规则，但会应用 src 外层 .babelrc 的配置文件：
- index.js 会应用 ../.babelrc
- subdir/index.js 会应用 subdir/.babelrc

> babel 决定一个 js 文件应用哪些配置文件时,会执行如下策略: 如果这个 js 文件在当前项目内，则会先递归向上搜索最近的一个 .babelrc 文件(直到遇到package.json目录)，将其与全局配置合并。如果这个 js 文件不在当前项目内，则只应用全局配置，并忽略与这个文件相关的 .babelrc 。


## 使用 @babel/cli
## 使用 webpack
### babel-loader

webpack 通过 babel-loader 使用 Babel 。

babel-loader 和 @babel/core 有版本对应关系，否则会提示 core 找不到或者有警告：

|  babel-loader   | babel  | webpack |
| ----------- | ----------- | ----------- |
| 6.x及以下  | 6.x | 1.x |
| 7.x（^6.2.10 也可以，但是会有版本警告）  | 6.x | 2.x |
| 7.1以上  | 6.x | 3.x |
| 8.x  | 7.x | 4.x or 5.x |
| 9.x  | ^7.12.0 | 5.x |



