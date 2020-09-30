---
title: Jest单元测试（上）
date: 2020-06-19 15:57:21
tags:
- 单元测试
- Jest
categories: 测试
author: 崔梦林
keywords: Jest,单元测试
description: Jest,单元测试
cover: https://img14.360buyimg.com/imagetools/jfs/t1/112099/34/10664/24933/5eec71feE440de023/e1c745c1ec75aebb.jpg
top_img: https://img12.360buyimg.com/imagetools/jfs/t1/130581/29/2508/17816/5eec718bEadf4ddd7/5ac296f3d92ed386.png
---
## 为什么要进行单元测试

测试是开发工作的一个重要环节，但是往往由于编写和维护测试代码的成本问题，编写测试代码这个环节往往是被忽略的。首先说说前端单元测试为什么是重要的。  

仔细想一下，在我们的工作中是否经常遇到过下面的问题：
- 修改某个模块的功能，引起了其他模块的问题；
- 代码难以维护，新增需求时难以评估在整个项目的影响范围；
- 多人开发的代码难以维护，往往是新增加一份代码来替代原有功能；
- 由于测试资源和测试对项目熟悉程度的问题，代码无法重构；
- 代码质量差，每次更新都像在打补丁；
- 项目需要频繁更改上线，但因为某些模块的复杂扭曲的逻辑，实在快不起来；
- 等等等等。。。

如果你对这些问题感同身受，那么就有必要考虑是不是要使用单元测试了。增加自动化测试后，可以达到下面的目的：

1. 为核心功能编写测试后，可以保障项目的可靠性；
2. 强迫开发者编写更容易被测试的代码，提高代码质量；
3. 编写的测试有文档的作用，方便维护。

总的来说，如果你想保证代码质量，那么你需要写单元测试；如果你想随时整理重构代码，那么你需要写单元测试；如果你想有自动化的测试套件来帮你快速验证提交的完整性，那么你需要写单元测试。当然，如果你的项目不需要频繁上线，一旦变更可能就是要改版了，那可以抛开编写测试代码的环节，毕竟这个投入产出是不平衡的。

## 为什么选择Jest

1. Jest 是 Facebook 出品的一个测试框架，相对其他测试框架，其一大特点就是就是内置了常用的测试工具，比如自带断言、测试覆盖率工具，实现了开箱即用；
2. Jest 可以利用其特有的快照测试功能，通过比对 UI 代码生成的快照文件，实现对 React 等常见框架的自动测试；
3. Jest 的测试用例是并行执行的，而且只执行发生改变的文件所对应的测试，提升了测试速度；
4. React 内置了Jest，Vue CLI 也拥有开箱即用的通过 Jest 进行单元测试的内置选项

## Jest用法

### 安装

``` js
    npm install jest
    or
    yarn add jest
```

### 创建一个 Jest 测试的demo 

1. 创建 math.js 文件，添加 math 方法

``` js
    const sum = (a, b) => a + b
    module.exports = {
        sum
    }
``` 

2. 创建一个 math.test.js 文件，来测试 math.js 中定义的函数 

``` js
    const {
        sum
    } = require('./math.js')
    describe('Test Math Module', () => {
        test('should return sum value when one plus another', () => {
            const one = 1
            const another = 2
            const result = sum(one, another)
            expect(result).toBe(3)
        })
    })
```

3. 在 package.json 添加 scripts 命令 "test": "jest", 执行 npm run test 或者 yarn test， Jest 会自动搜索 __tests__ 文件夹下的文件、以 .test.js 或者 .spec.js 结尾的文件，运行后效果如下:

![ccccc](https://img10.360buyimg.com/imagetools/jfs/t1/121993/40/3077/234476/5eccef57E5269112c/9b1dae76a08f6027.jpg)

是不是看起来很简单，下面我们就来看看用 Jest 写测试代码的套路吧。 

### Given/When/Then 的套路

首先我们看到的是一个由 test 包裹的测试主体最小单元，采用了 Given When Then 的经典格式，我们常常称之为测试三部曲，也可以解释为 3A 即：

|  GWT   | 3A  | 说明 |  
|  ----  | ----  | ---- |   
| Given  | Arrange | 准备测试条件，如要测试的数据或者要渲染的组件 |   
| When  | Act | 采取行动，一般来说就是调用相应的模块执行对应的函数或方法 |
| Then  | Assert | 断言，这时需要借助的就是 Matchers 的能力，Jest 还可以扩展自己的 Matcher |

在 expect 后面的 toBe称之为 Matcher，是断言时的判断语句以验证正确性。插入下断言的概念：

> 在程序设计中，断言（assertion）是一种放在程序中的一阶逻辑（如一个结果为真或是假的逻辑判断式），目的是为了标示与验证程序开发者预期的结果－当程序运行到断言的位置时，对应的断言应该为真。若断言不为真时，程序会中止运行，并给出错误消息.

我们刚才创建的 demo 用 Given/When/Then 套路区分的话是这个样子的，当然实际编写测试用例的时候情况要复杂的多。

``` js
    const {
        sum
    } = require('./math.js')
    describe('Test Math Module', () => {
        test('should return sum value when one plus another', () => {
            // Given
            const one = 1
            const another = 2
            // When
            const result = sum(one, another)
            // Then
            expect(result).toBe(3)
        })
    })
```
### Matcher（匹配器）

1. 通用匹配

- toBe 精确匹配

``` js
    test('two plus two is four', () => {
        expect(2 + 2).toBe(4)
    })
```

- toEqual 会递归检查对象或数组的每一个字段

``` js
    test('object assignment', () => {
        const data = {
            one: 1
        }
        data['two'] = 2
        expect(data).toEqual({
            one: 1,
            two: 2
        })
    })
```

2. 真假值

- toBeNull 仅匹配 null
- toBeUndefined 仅匹配 undefined
- toBeDefined 与 toBeUndefined 相对
- toBeTruthy 匹配真值
- toBeFalsy 匹配假值

``` js 
    test('null', () => {
        const n = null
        expect(n).toBeNull()
        expect(n).toBeDefined()
        expect(n).not.toBeUndefined()
        expect(n).not.toBeTruthy()
        expect(n).toBeFalsy()
    })

    test('zero', () => {
        const z = 0
        expect(z).not.toBeNull()
        expect(z).toBeDefined()
        expect(z).not.toBeUndefined()
        expect(z).not.toBeTruthy()
        expect(z).toBeFalsy()
    })
```

3. 数字

- toBeGreaterThan 大于
- toBeGreaterThanOrEqual 大于等于
- toBeLessThan 小于
- toBeLessThanOrEqual 小于等于

``` js
    test('two plus two', () => {
        const value = 2 + 2
        expect(value).toBeGreaterThan(3)
        expect(value).toBeGreaterThanOrEqual(3.5)
        expect(value).toBeLessThan(5)
        expect(value).toBeLessThanOrEqual(4.5)

        // toBe 和 toEqual 在数字类型上作用等同
        expect(value).toBe(4)
        expect(value).toEqual(4)
    })
```
- toBeCloseTo 对于浮点数的计算  

``` js
    test('adding floating point numbers', () => {
        const value = 0.1 + 0.2
        // expect(value).toBe(0.3) 因为舍入问题的存在，这种判断不奏效
        expect(value).toBeCloseTo(0.3)  // 这种判断有用
    })
```

4. 字符串

- 字符串数据类型可使用正则表达式进行匹配判断

``` js
    test('there is no I in team', () => {
        expect('team').not.toMatch(/I/)
    })

    test('but there is a "stop" in Christoph', () => {
        expect('Christoph').toMatch(/stop/)
    })
```

5. 数组

- toContain 判断数组中是否存在某个特定的元素

``` js
    const shoppingList = ['diapers', 'beer']
    test('the shopping list has beer on it', () => {
        expect(shoppingList).toContain('beer')
    })
```

6. 异常

``` js
    function compileAndroidCode() {
        throw new ConfigError('you are using the wrong JDK')
    }

    test('compiling android goes as expected', () => {
        expect(compileAndroidCode).toThrow()
        expect(compileAndroidCode).toThrow(ConfigError)

        // 同样可以使用明确的错误消息或正则表达式
        expect(compileAndroidCode).toThrow('you are using the wrong JDK')
        expect(compileAndroidCode).toThrow(/JDK/)
    })
```
### 处理异步

在 Javascript 中，异步操作是很常见的。处理异步时，最重要的一点是告知Jest 当前它测试的代码是否已完成，然后它可以转移到另一个测试。

例如，假设有一个 fetchData(callback) 函数，获取一些数据并在完成时调用 callback(data)。 你期望返回的数据是一个字符串 'peanut butter'。有以下几种方式可以实现：

1. 回调 callback

``` js
    function fetchData (callback) {
        setTimeout(() => {
            callback('not peanut butter')
        }, 10000)
    }

    describe('Test fetchData Module', () => { 
        test('the data is peanut butter', () => {
            function callback(data) {
                expect(data).toBe('peanut butter')
            }
            fetchData(callback)
        })
    })
```

看起来很对是吧，把expect写在callbak里了，来看看执行结果：
![callback](https://img11.360buyimg.com/imagetools/jfs/t1/121723/34/3097/161064/5ecd1d84E8f58702e/5af19e5501423491.jpg)

fetchData 返回的是 not peanut butter， 明显是错误的，之所以可以通过测试，是因为默认情况下，Jest 测试一旦执行到末尾就会完成。fetchData 执行结束时，此测试就在没有调用回调函数前结束。

正确的回调应该使用单个参数调用 done，而不是将测试放在一个空参数的函数，Jest会等done回调函数执行结束后，结束测试， 如下：

``` js
    test('the data is peanut butter', done => {
        function callback(data) {
            try {
                expect(data).toBe('peanut butter')
                done()
            } catch (error) {
                done(error)
            }
        }
        fetchData(callback)
    })
```

2. Promises  

如果 fetchData 使用 Promise，可以使用 then（）/catch（） 或者 resolves / rejects。一定不要忘记把 promise 作为返回值,如果你忘了 return 语句的话，在 fetchData 返回的这个 promise 被 resolve、then() 有机会执行之前，测试就已经被视为已经完成了。

``` js
    function fetchData () {
        return new Promise (resolve => {
            setTimeout(() => {
                resolve('not peanut butter')
            }, 1000)
        })
    }
    describe('Test fetchData Module', () => { 
        test('the data is peanut butter', () => {
            return fetchData().then(data => {
                expect(data).toBe('peanut butter')
            })
        })
        test('the data is peanut butter', () => {
            return expect(fetchData()).resolves.toBe('peanut butter')
        })
    })
```

3. Async/Await

``` js
    test('the data is peanut butter', async () => {
        const data = await fetchData()
        expect(data).toBe('peanut butter')
        // await expect(fetchData()).resolves.toBe('peanut butter')
    })
```

补充： 如果使用catch 或者 rejects 时，需要添加 expect.assertions 来验证一定数量的断言被调用,否则不会让测试失败。

``` js
    function fetchData () {
        return new Promise(function(resolve, reject) {
        setTimeout(() => {
            reject('not peanut butter')
        }, 1000)
        })
    }
    
    
    test('the data is peanut butter', () => {
        // expect.assertions(1)
        return fetchData().then(data => {
            expect(data).toBe('peanut butter11')
        }, (err) => {
            console.log(err)
        })
    })
```

执行上述代码是不会报错的，但我们期望reject的时候可以报错提醒我们，在指定了断言的次数expect.assertions(1)之后，如果没有出现断言的时候，就会报错，如下图
![expect.assertions](https://img11.360buyimg.com/imagetools/jfs/t1/115148/20/8291/305438/5ecd2be6E97a0c53e/944a215811e4e31a.jpg)

额外的expect.assertions(number) 其实是验证在测试期间所调用的断言数量，这在测试多层异步代码时很有用，以确保实际调用回调中的断言次数。

### Mock Functions

在项目里，往往一个模块会调用外部一个或者多个模块的方法。比如你要测试一个 Order 模块 的 price() 方法，而 price() 方法需要在 Product 和 Customer 模块中调用一些函数。如果你希望单元测试所测试的 Order 模块是独立的，那么你就不想直接使用真正的 Product 或 Customer，因为 Customer 的错误会直接导致 Order 的单元测试失败。这种情况下，你就需要使用一个替身作为依赖的对象。在单元测试中，我们可能并不需要 Product 或者 Customer 内部的执行方法，只想知道它是否被正确调用或者返回指定值即可。

Mock函数提供的以下三种特性，在我们写测试代码时十分有用：
- 捕获函数调用情况
- 设置函数返回值
- 改变函数的内部实现

1. jest.fn()

jest.fn() 是创建 Mock 函数最简单的方式，如果没有定义函数内部的实现，jest.fn()会返回undefined作为返回值。

``` js
    test('test jest.fn()', () => {
        let mockFn = jest.fn()
        let result = mockFn(1, 2, 3)

        // 断言mockFn的执行后返回undefined
        expect(result).toBeUndefined()
        // 断言mockFn被调用
        expect(mockFn).toBeCalled()
        // 断言mockFn被调用了一次
        expect(mockFn).toBeCalledTimes(1)
        // 断言mockFn传入的参数为1, 2, 3
        expect(mockFn).toHaveBeenCalledWith(1, 2, 3)
    })
```
jest.fn()所创建的 Mock 函数还可以设置返回值，定义内部实现或返回Promise对象

``` js
    test('test jest.fn() return dafault value', () => {
        let mockFn = jest.fn().mockReturnValue('default')
        // 断言mockFn执行后返回值为default
        expect(mockFn()).toBe('default')
    })

    test('test jest.fn() inner function', () => {
        let mockFn = jest.fn((num1, num2) => {
            return num1 * num2
        })
        // 断言mockFn执行后返回100
        expect(mockFn(10, 10)).toBe(100)
    })

    test('test jest.fn() return promise', async () => {
        let mockFn = jest.fn().mockResolvedValue('default')
        let result = await mockFn()
        // 断言mockFn通过await关键字执行后返回值为default
        expect(result).toBe('default')
        // 断言mockFn调用后返回的是Promise对象
        expect(Object.prototype.toString.call(mockFn())).toBe('[object Promise]')
    })
```
2. jest.mock()  

如果我们要测试的模块调用的其他模块不需要实际的请求，这时候我们需要使用 jest.mock() 方法去 mock 整个模块。

``` js
    // foo.js
    module.exports = () => 40

    // test.js
    const foo = require('./foo')
    jest.mock('./foo')
    foo.mockImplementation(() => 42)

    describe('Test Mock Foo Module', () => {
        test('should return 42', () => {
            expect(foo()).toBe(42)
        })
    })
```

我们可以看到 jest.mock() 方法中的第二个参数是一个函数，那么我们就可以完全接管整个 foo 模块，被 Mock 之后我们的测试就可以使用 Mock 所返回的数据或方法，从而保证模块所返回的内容是我们所期望的。但这时需要注意的是，该模板的所有功能都已经被 Mock 掉，而不会再从原模块当中返回，所以我们就需要重新实现该模块中的所有功能。

3. jest.spyOn()  

spy 并不会影响到原有模块的功能代码，而只是充当一个监护人的作用。我们可以像下面这样创建并使用 spy：

``` js
    const bot = {
        sayHello: (name) => {
            console.log(`Hello ${name}!`)
        }
    }
    describe('bot', () => {
        it('should say hello', () => {
            const spy = jest.spyOn(bot, 'sayHello')
            bot.sayHello('Michael')
            expect(spy).toHaveBeenCalledWith('Michael')
            // 恢复 bot 对象原本的 sayHello 方法
            spy.mockRestore()
        })
    })
```
我们通过 jest.spyOn（） 监听了 bot 的 sayHello 方法，它就像间谍一样监听了所有对 bot 中 sayHello 方法的调用。由于创建 spy 时，Jest 实际上修改了 bot 对象的 sayHello 属性，所以在断言完成后，我们还要通过 mockRestore 来恢复 bot 对象原本的 sayHello 方法。

## 写在最后
测试只是一种工具和手段，代码质量是依靠设计和维护的。