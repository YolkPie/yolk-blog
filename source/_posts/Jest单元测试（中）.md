---
title: Jest单元测试（中）
date: 2020-09-29 21:00:00
tags:
- 单元测试
- Jest
categories: 测试
keywords: Jest,单元测试
description: Jest,单元测试
cover: https://img14.360buyimg.com/imagetools/jfs/t1/112099/34/10664/24933/5eec71feE440de023/e1c745c1ec75aebb.jpg
top_img: https://img12.360buyimg.com/imagetools/jfs/t1/130581/29/2508/17816/5eec718bEadf4ddd7/5ac296f3d92ed386.png
---

### 默认配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Jest 的目标是在大部分 JavaScript 项目中实现开箱即用，因此本身有一些默认配置，即使我们不做任何配置也可以正确运行。如果想了解这些默认配置项具体是什么，可以通过下面的方式获取：
- 查看 node_modules/jest-config/build/Default.js 文件中的 defaultOptions 对象
- 通过以下js获取。

    ``` js
        const { defaults } = require('jest-config')
        
        //下面是用第二种方法获取到的 Jest 默认配置
        {
            automock: false,
            bail: 0,
            cache: true,
            cacheDirectory: '/private/var/folders/3l/p3_nb2m13b9cgstcy0b3gx18ffrd_n/T/jest_80wf9x',
            changedFilesWithAncestor: false,
            clearMocks: false,
            collectCoverage: false,
            coveragePathIgnorePatterns: [ '/node_modules/' ],
            coverageProvider: 'babel',
            coverageReporters: [ 'json', 'text', 'lcov', 'clover' ],
            errorOnDeprecated: false,
            expand: false,
            forceCoverageMatch: [],
            globals: {},
            haste: { computeSha1: false, throwOnModuleCollision: false },
            maxConcurrency: 5,
            maxWorkers: '50%',
            moduleDirectories: [ 'node_modules' ],
            moduleFileExtensions: [ 'js', 'json', 'jsx', 'ts', 'tsx', 'node' ],
            moduleNameMapper: {},
            modulePathIgnorePatterns: [],
            noStackTrace: false,
            notify: false,
            notifyMode: 'failure-change',
            prettierPath: 'prettier',
            resetMocks: false,
            resetModules: false,
            restoreMocks: false,
            roots: [ '<rootDir>' ],
            runTestsByPath: false,
            runner: 'jest-runner',
            setupFiles: [],
            setupFilesAfterEnv: [],
            skipFilter: false,
            snapshotSerializers: [],
            testEnvironment: 'jest-environment-jsdom',
            testEnvironmentOptions: {},
            testFailureExitCode: 1,
            testLocationInResults: false,
            testMatch: [ '**/__tests__/**/*.[jt]s?(x)', '**/?(*.)+(spec|test).[tj]s?(x)' ],
            testPathIgnorePatterns: [ '/node_modules/' ],
            testRegex: [],
            testRunner: 'jasmine2',
            testSequencer: '@jest/test-sequencer',
            testURL: 'http://localhost',
            timers: 'real',
            transformIgnorePatterns: [ '/node_modules/' ],
            useStderr: false,
            watch: false,
            watchPathIgnorePatterns: [],
            watchman: true
        }
    ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很多，看起来也很头大。我们先将常用到的配置项做下归类，之后再配置对 Babel / webpack 等的支持，对这些配置项就会有大致的了解了。 Jest 配置详见： [Configuring Jest](https://jestjs.io/docs/en/configuration.html)

### Jest 配置项
1. mock
    - automock: 引入的模块会被自动mock
    - clearMocks: 测试前自动清除 mock.calls 和 mock.instances 属性，等价于 jest.clearAllMocks()
    - resetMocks: 测试前自动重置 mock 到初始状态，等价于 jest.resetAllMocks()
    - restoreMocks: 测试前自动恢复 mock（？？和restMock的区别），等价于 jest.restoreAllMocks()
2. 覆盖率
    - collectCoverage: 是否搜集测试时的覆盖率
    - collectCoverageFrom: 哪些文件测试时需要进行覆盖率统计（数组，通过[glob 模式](https://github.com/isaacs/node-glob#glob-primer)匹配）
    - coverageDirectory: 输出覆盖信息文件的目录
    - coveragePathIgnorePatterns: 哪些文件测试时不需要进行覆盖率统计（数组，通过正则匹配）
    - coverageReporters: 定义测试覆盖率报告的格式
    - coverageThreshold: 覆盖率的阈值，可按照 global 和 glob 模式分组（单位为百分比，正值表示允许的覆盖率的最小值，负值表示不通过的百分比的最大值）
    - forceCoverageMatch: 统计测试覆盖率时，测试文件默认被忽略，此项可以指定哪些文件需要被统计进来（数组，glob 模式）
3. 转换器
    - transform: 匹配文件和对应的转换器
    - transformIgnorePatterns: 转换时要排除的文件
4. bail
    - 停止运行测试代码时出现错误的次数
    

### Jest 配置方式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果我们想对 Jest 进行个性化配置，有以下方式：
1. 在项目中 package.json 文件中增加 “jest” 字段

    ``` js
        {
            "name": "my-project",
            "jest": {
                "verbose": true
            }
        }
    ```

2. 通过 jest.config.js  

    ``` js
        module.exports = {
            verbose: true,
        };
    ```
    
3. 如果以上两种方式不满足要求，想自定义配置文件的名称和格式，可在执行 jest 命令时增加 ```--config <path/to/file.js|cjs|mjs|json> ```，--config 选项可指定配置文件。需要注意的是如果采用json 文件，不能有 “jest” 键值。

    ``` js
        {
            "verbose": true
        }
    ```

### 生成一个基础配置文件

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了手动添加配置文件外，我们可以通过 Jest 提供的 Jest CLI，运行 ```jest --init``` 命令，运行完成后，会生成一个 jest.config.js，这个文件里每个选项都有简短的说明。

### 使用 Babel

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于我们配置的环境是 node，它的语法是CommonJS，因此使用ES6 语法 import 时会报错。

1. 安装依赖

    ``` js
        // babel 6
        npm install --save-dev babel-jest babel-core regenerator-runtime
        // babel 7
        npm install --save-dev babel-jest babel-core@^7.0.0-bridge.0 @babel/core regenerator-runtime
    ```

2. 在根目录下增加 .babelrc 文件
    ``` js
        {
        "presets": ["env"]
        }
    ```
3. 运行 ``` npm run test ``` 查看结果

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是，安装 Jest 时，babel-jest 是会被自动安装的。当运行jest 命令时，babel-jest 会检测当前环境是否安装了 babel-core，如果安装了，会去取 .babelrc 的配置并执行 babel 对代码进行转化。 如果要避免这个行为，比如需要添加额外的预处理器，需要显式的重置 transform 配置项，因为一旦添加了 transform 配置，babel-jest 就不会自动载入了。

``` js
    // package.json 重置 transform
    {
        "jest": {
            "transform": {}
        }
    }
```

### 添加 TypeScript 支持
1. 我们先创建一个 math.ts 文件：

    ``` js
        // math.ts
        export default function sum(a: number, b: number): number {
            return a + b
        }
    ```

2. 接着创建测试文件 math.test.ts:

    ``` js
        // math.test.ts
        import math from '../src/math'
        describe('Test Math Module', () => {
            test('should return sum value when one plus another', () => {
                expect(sum(1, 2)).toBe(3)
            })
        })
    ```

3. 创建 tsconfig.json 文件，添加 TypeScript 配置：

    ``` js
        // tsconfig.json
        {
            "compilerOptions": {
                "target": "es5",
                "strict": true,
            },
            "include": [
                "src/**/*",
                "__test__/**/*"
            ],
            "exclude": [
                "node_modules",
            ]
        }
    ```

4. 使用 Jest 测试 TypeScript 代码需要借助 ts-jest 解析器，安装依赖：

    ``` js
        npm install -save-dev ts-jest typescript @types/jest
    ```

5. 配置 Jest

    ``` js
        // jest.config.js
        module.exports = {
            collectCoverage: true,
            transform: {
                '^.+\\.tsx?$': 'ts-jest',
            },
        }
    ```

6. 运行 ``` npm run test ``` 查看结果

