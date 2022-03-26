---
title: 你不知道的TypeScript高级技巧
date: 2022-03-26 19:08:07
---

## 前言


TS 已经越来越火，不管是服务端（Node.js），还是前端框架（Angular、Vue3），都有越来越多的项目使用 TS 开发，作为前端程序员，TS 已经成为一项必不可少的技能。本文旨在介绍 TS 中的一些高级技巧，提高大家对这门语言更深层次的认知。

### Typescript 简介


- ECMAScript 的超集 (stage 3)

- 编译期的类型检查

- 不引入额外开销（零依赖，不扩展 js 语法，不侵入运行时）

- 编译出通用的、易读的 js 代码



``` Typescript = Type + ECMAScript + Babel-Lite ```

> Typescript 设计目标: https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals



### 为什么使用 Typescript


- 增加了代码的可读性和可维护性

- 减少运行时错误，写出的代码更加安全，减少 BUG

- 享受到代码提示带来的好处

- 重构神器

### 基础类型


- boolean

- number

- string

- array

- tuple

- enum

- void

- null & undefined

- any & unknown

- never
- ...

##### any 和 unknown 的区别


- any: 任意类型

- unknown: 未知的类型



任何类型都能分配给 unknown，但 unknown 不能分配给其他基本类型，而 any 啥都能分配和被分配。

  ```  

    let foo: unknown

    foo = true // ok
    foo = 123 //ok

    foo.toFixed(2) // error

    let foo1: string = foo // error

  ```

  ``` 

    let bar: any

    bar = true // ok
    bar = 123 //ok

    foo.toFixed(2) // ok

    let bar1:string  = bar // ok

  ```

可以看到，用了 any 就相当于完全丢失了类型检查，所以大家尽量少用 any，对于未知类型可以用 unknown。

##### unknown 的正确用法

我们可以通过不同的方式将 `unknown` 类型缩小为更具体的类型范围:

  ```
  function getLen(value: unknown): number {
  if (typeof value === 'string') {
    // 因为类型保护的原因，此处 value 被判断为 string 类型
   return value.length
  }
  
  return 0
}
  ```

#### never


`never` 一般表示哪些用户无法达到的类型。在最新的 typescript 3.7 中，下面代码会报错:  

  ```
    // never 用户控制流分析
    function neverReach (): never {
      throw new Error('an error')
    }

    const x = 2

    neverReach()

    x.toFixed(2)  // x is unreachable
  ```

never 还可以用于联合类型的 幺元：
  ```
    type T0 = string | number | never // T0 is string | number
  ```

##### 函数类型

几种函数类型的返回值类型写法

  ```
    function fn(): number {
      return 1
    }

    const fn = function (): number {
      return 1
    }

    const fn = (): number => {
      return 1
    }

    const obj = {
      fn (): number {
        return 1
      }
    }
  ```

在 () 后面添加返回值类型即可。


##### 函数类型


ts 中也有函数类型，用来描述一个函数：



`type FnType = (x: number, y: number) => number`


##### 完整的函数写法


  ```
    let myAdd: (x: number, y: number) => number = function(x: number, y: number): number {
      return x + y
    }

    // 使用 FnType 类型
    let myAdd: FnType = function(x: number, y: number): number {
      return x + y
    }

    // ts 自动推导参数类型
    let myAdd: FnType = function(x, y) {
      return x + y
    }
  ```


##### 函数重载？


js 因为是动态类型，本身不需要支持重载，ts 为了保证类型安全，支持了函数签名的类型重载。即：

多个重载签名和一个实现签名


  ```
    // 重载签名（函数类型定义）
    function toString(x: string): string;
    function toString(x: number): string;

    // 实现签名（函数体具体实现）
    function toString(x: string | number) {
      return String(x)
    }

    let a = toString('hello') // ok
    let b = toString(2) // ok
    let c = toString(true) // error
  ```


如果定义了重载签名，则实现签名对外不可见


  ```
    function toString(x: string): string;

    function toString(x: number): string {
      return String(x)
    }

    len(2) // error
  ```


实现签名必须兼容重载签名


  ```
    function toString(x: string): string;
    function toString(x: number): string; // error

    // 函数实现
    function toString(x: string) {
      return String(x)
    }
  ```


重载签名的类型不会合并


  ```
    // 重载签名（函数类型定义）
    function toString(x: string): string;
    function toString(x: number): string;

    // 实现签名（函数体具体实现）
    function toString(x: string | number) {
      return String(x)
    }

    function stringOrNumber(x): string | number {
      return x ? '' : 0
    }

    // input 是 string 和 number 的联合类型
    // 即 string | number
    const input = stringOrNumber(1)

    toString('hello') // ok
    toString(2) // ok
    toString(input) // error
  ```

##### 类型推断


ts 中的类型推断是非常强大，而且其内部实现也是非常复杂的。



基本类型推断：


  ```
    // ts 推导出 x 是 number 类型
    let x = 10
  ```


对象类型推断：


  ```
    // ts 推断出 myObj 的类型：myObj: { x: number; y: string; z: boolean; }
    const myObj = {
      x: 1,
      y: '2',
      z: true
    }
  ```


##### 函数类型推断：


  ```
    // ts 推导出函数返回值是 number 类型
    function len (str: string) {
      return str.length
    }
  ```


上下文类型推断：


  ```
    // ts 推导出 event 是 ProgressEvent 类型
    const xhr = new XMLHttpRequest()
    xhr.onload = function (event) {}
  ```
所以有时候对于一些简单的类型可以不用手动声明其类型，让 ts 自己去推断。


##### 类型兼容性


typescript 的子类型是基于 结构子类型 的，只要结构可以兼容，就是子类型。（Duck Type）


  ```
    class Point {
      x: number
    }

    function getPointX(point: Point) {
      return point.x
    }

    class Point2 {
      x: number
    }

    let point2 = new Point2()

    getPointX(point2) // OK
  ```


java、c++ 等传统静态类型语言是基于 名义子类型 的，必须显示声明子类型关系（继承），才可以兼容。


  ```
    public class Main {
      public static void main (String[] args) {
        getPointX(new Point()); // ok
        getPointX(new ChildPoint()); // ok
        getPointX(new Point1());  // error
      }

      public static void getPointX (Point point) {
        System.out.println(point.x);
      }

      static class Point {
        public int x = 1;
      }

      static class Point2 {
        public int x = 2;
      }
        
      static class ChildPoint extends Point {
        public int x = 3;
      }
    }
  ```

##### 对象子类型


子类型中必须包含源类型所有的属性和方法:


  ```
    function getPointX(point: { x: number }) {
      return point.x
    }

    const point = {
    x: 1,
      y: '2'
    }

    getPointX(point) // OK
  ```

注意: 如果直接传入一个对象字面量是会报错的：


  ```
    function getPointX(point: { x: number }) {
      return point.x
    }

    getPointX({ x: 1, y: '2' }) // error
  ```


这是 ts 中的另一个特性，叫做:  excess property check  ，当传入的参数是一个对象字面量时，会进行额外属性检查。



##### 函数子类型


介绍函数子类型前先介绍一下逆变与协变的概念，逆变与协变并不是 TS 中独有的概念，在其他静态语言中也有相关理念。



在介绍之前，先假设一个问题，约定如下标记：



- A ≼ B 表示 A 是 B 的子类型，A 包含 B 的所有属性和方法。

- A => B 表示以 A 为参数，B 为返回值的方法。(param: A) => B



如果我们现在有三个类型 Animal 、 Dog 、 WangCai(旺财) ，那么肯定存在下面的关系：



  `WangCai ≼ Dog ≼ Animal // 即旺财属于狗属于动物`


问题：以下哪种类型是 Dog => Dog 的子类呢?



- WangCai => WangCai

- WangCai => Animal

- Animal  => Animal

- Animal  => WangCai



从代码来看解答


  ```
    class Animal {
      sleep: Function
    }

    class Dog extends Animal {
      // 吠
      bark: Function
    }

    class WangCai extends Dog {
      dance: Function
    }


    function getDogName (cb: (dog: Dog) => Dog) {
      const dog = cb(new Dog())
      dog.bark()
    }

    // 对于入参来说，WangCai 是 Dog 的子类，Dog 类上没有 dance 方法, 产生异常。
    // 对于出参来说，WangCai 类继承了 Dog 类，肯定会有 bark 方法
    getDogName((wangcai: WangCai) => {
      wangcai.dance()
      return new WangCai()
    })

    // 对于入参来说，WangCai 是 Dog 的子类，Dog 类上没有 dance 方法, 产生异常。
    // 对于出参来说，Animal 类上没有 bark 方法, 产生异常。
    getDogName((wangcai: WangCai) => {
      wangcai.dance()
      return new Animal()
    })

    // 对于入参来说，Animal 类是 Dog 的父类，Dog 类肯定有 sleep 方法。
    // 对于出参来说，WangCai 类继承了 Dog 类，肯定会有 bark 方法
    getDogName((animal: Animal) => {
      animal.sleep()
      return new WangCai()
    })

    // 对于入参来说，Animal 类是 Dog 的父类，Dog 类肯定有 sleep 方法。
    // 对于出参来说，Animal 类上没有 bark 方法, 产生异常。
    getDogName((animal: Animal) => {
      animal.sleep()
      return new Animal()
    })
  ```

可以看到只有 `Animal => WangCai` 才是 `Dog => Dog` 的子类型，可以得到一个结论，对于函数类型来说，函数参数的类型兼容是反向的，我们称之为 逆变 ，返回值的类型兼容是正向的，称之为 协变 。

逆变与协变的例子只说明了函数参数只有一个时的情况，如果函数参数有多个时该如何区分？



其实函数的参数可以转化为 `Tuple` 的类型兼容性：


  ```
    type Tuple1 = [string, number]
    type Tuple2 = [string, number, boolean]

    let tuple1: Tuple1 = ['1', 1]
    let tuple2: Tuple2 = ['1', 1, true]

    let t1: Tuple1 = tuple2 // ok
    let t2: Tuple2 = tuple1 // error
  ```

可以看到 `Tuple2 => Tuple1` ，即长度大的是长度小的子类型，再由于函数参数的逆变特性，所以函数参数少的可以赋值给参数多的（参数从前往后需一一对应），从数组的 `forEach` 方法就可以看出来：


  ```
    [1, 2].forEach((item, index) => {
    console.log(item)
    }) // ok

    [1, 2].forEach((item, index, arr, other) => {
    console.log(other)
    }) // error
  ```

##### 高级类型
###### 联合类型与交叉类型


- 联合类型(union type)表示多种类型的 “或” 关系


  ```
    function genLen(x: string | any[]) {
      return x.length
    }

    genLen('') // ok
    genLen([]) // ok
    genLen(1) // error
  ```

- 交叉类型表示多种类型的 “与” 关系


  ```
  interface Person {
    name: string
    age: number
  }

  interface Animal {
    name: string
    color: string
  }

  const x: Person & Animal = {
    name: 'x',
    age: 1,
    color: 'red
  }
  ```

- 使用联合类型表示枚举


  ```
    type Position = 'UP' | 'DOWN' | 'LEFT' | 'RIGHT'

    const position: Position = 'UP'
  ```
> 可以避免使用 enum 侵入了运行时。



##### 类型保护


###### ts 初学者很容易写出下面的代码：


  ```
  function isString (value) {
    return Object.prototype.toString.call(value) === '[object String]'
  }

  function fn (x: string | number) {
    if (isString(x)) {
      return x.length // error 类型“string | number”上不存在属性“length”。
    } else {
      // .....
    }
  }
  ```

##### 如何让 ts 推断出来上下文的类型呢？



- 1. 使用 ts 的 is 关键词

````
function isString (value: unknown): value is string {
  return Object.prototype.toString.call(value) === '[object String]'
}

function fn (x: string | number) {
  if (isString(x)) {
    return x.length
  } else {
    // .....
  }
}
````
- 2. typeof 关键词

在 ts 中，代码实现中的 `typeof` 关键词能够帮助 ts 判断出变量的基本类型:

  ```
  function fn (x: string | number) {
    if (typeof x === 'string') { // x is string
      return x.length
    } else { // x is number
      // .....
    }
  }
  ```
- 3. instanceof 关键词

在 ts 中，`instanceof` 关键词能够帮助 ts 判断出构造函数的类型:
  ```
  function fn1 (x: XMLHttpRequest | string) {
    if (x instanceof XMLHttpRequest) { // x is XMLHttpRequest
      return x.getAllResponseHeaders()
    } else { // x is string
      return x.length
    }
  }
  ```
- 4. 针对 null 和 undefined 的类型保护

在条件判断中，ts 会自动对 `null` 和 `undefined` 进行类型保护:
  ```
  function fn2 (x?: string) {
    if (x) {
      return x.length
    }
  }
  ```
- 5. 针对 null 和 undefined 的类型断言

如果我们已经知道的参数不为空，可以使用 `!` 来手动标记:
  ```
  function fn2 (x?: string) {
    return x!.length
  }
  ```
##### typeof 关键词


`typeof` 关键词除了做类型保护，还可以从实现推出类型，。



> 注意：此时的 typeof 是一个类型关键词，只可以用在类型语法中。


  ```
  function fn(x: string) {
    return x.length
  }

  const obj = {
    x: 1,
    y: '2'
  }

  type T0 = typeof fn // (x: string) => number
  type T1 = typeof obj // {x: number; y: string }
  ```
##### keyof 关键词


`keyof` 也是一个 类型关键词 ，可以用来取得一个对象接口的所有 key 值:


  ```
  interface Person {
    name: string
    age: number
  }

  type PersonAttrs = keyof Person // 'name' | 'age'
  ```
##### in 关键词


`in` 也是一个 类型关键词, 可以对联合类型进行遍历，只可以用在 `type` 关键词下面。


  ```
  type Person = {
    [key in 'name' | 'age']: number
  }

  // { name: number; age: number; }
  ```
##### [ ] 操作符


使用 [] 操作符可以进行索引访问，也是一个 类型关键词


  ```
  interface Person {
    name: string
    age: number
  }

  type x = Person['name'] // x is string
  ```
##### 一个小栗子


写一个类型复制的类型工具:


  ```
  type Copy<T> = {
    [key in keyof T]: T[key]
  }

  interface Person {
    name: string
    age: number
  }

  type Person1 = Copy<Person>
  ```

##### 泛型


泛型相当于一个类型的参数，在 ts 中，泛型可以用在 类、接口、方法、类型别名 等实体中。



###### 小试牛刀

  ```
  function createList<T>(): T[] {
    return [] as T[]
  }

  const numberList = createList<number>() // number[]
  const stringList = createList<string>() // string[]
  ```
> 有了泛型的支持，createList 方法可以传入一个类型，返回有类型的数组，而不是一个 any[]。



##### 泛型约束


如果我们只希望 createList 函数只能生成指定的类型数组，该如何做，可以使用 `extends` 关键词来约束泛型的范围和形状。


  ```
  type Lengthwise = {
    length: number
  }

  function createList<T extends number | Lengthwise>(): T[] {
    return [] as T[]
  }

  const numberList = createList<number>() // ok
  const stringList = createList<string>() // ok
  const arrayList = createList<any[]>() // ok
  const boolList = createList<boolean>() // error
  ```
> any[] 是一个数组类型，数组类型是有 length 属性的，所以 ok。string 类型也是有 length 属性的，所以 ok。但是 boolean 就不能通过这个约束了。



##### 条件控制


`extends` 除了做约束类型，还可以做条件控制，相当于与一个三元运算符，只不过是针对 类型 的。

表达式：`T extends U ? X : Y`



>含义：如果 T 可以被分配给 U，则返回 X，否则返回 Y。一般条件下，如果 T 是 U 的子类型，则认为 T 可以分配给 U，例如：


  ```
  type IsNumber<T> = T extends number ? true : false

  type x = IsNumber<string>  // false
```


##### 映射类型


映射类型相当于一个类型的函数，可以做一些类型运算，输入一个类型，输出另一个类型，前文我们举了个 Copy 的例子。



- 几个内置的映射类型

  ```
  // 每一个属性都变成可选
  type Partial<T> = {
    [P in keyof T]?: T[P]
  }

  // 每一个属性都变成只读
  type Readonly<T> = {
    readonly [P in keyof T]: T[P]
  }

  // 选择对象中的某些属性
  type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
  }

  // ......
  ```

typescript 2.8 在 `lib.d.ts` 中内置了几个映射类型:



  + Partial<T> -- 将 T 中的所有属性变成可选。

  + Readonly<T> -- 将 T 中的所有属性变成只读。

  + Pick<T, U> -- 选择 T 中可以赋值给U的类型。

  + Exclude<T, U> -- 从T中剔除可以赋值给U的类型。

  + Extract<T, U> -- 提取T中可以赋值给U的类型。

  + NonNullable<T> -- 从T中剔除null和undefined。

  + ReturnType<T> -- 获取函数返回值类型。

  + InstanceType<T> -- 获取构造函数类型的实例类型。



所以我们平时写 TS 时可以直接使用这些类型工具：


  ```
  interface ApiRes {
    code: string;
    flag: string;
    message: string;
    data: object;
    success: boolean;
    error: boolean;
  }

  type IApiRes = Pick<ApiRes, 'code' | 'flag' | 'message' | 'data'>

  // {
  //   code: string;
  //   flag: string;
  //   message: string;
  //   data: object;
  // }
  ```
###### extends 条件分发


对于 `T extends U ? X : Y` 来说，还存在一个特性，当 T 是一个联合类型时，会进行条件分发。


  ```
  type Union = string | number
  type isNumber<T> = T extends number ? 'isNumber' : 'notNumber'

  type UnionType = isNumber<Union> // 'notNumber' | 'isNumber'
  ```

实际上，`extends` 运算会变成如下形式：


  ```
  (string extends number ? 'isNumber' : 'notNumber') | (number extends number ? 'isNumber' : 'notNumber')
  ```

Extract 就是基于此特性，再配合 never 幺元的特性实现的：


  ```
  type Exclude<T, K> = T extends K ? never : T

  type T1 = Exclude<string | number | boolean, string | boolean>  // number
  ```
##### infer 关键词


`infer` 可以对运算过程中的类型进行存储，内置的`ReturnType` 就是基于此特性实现的：


  ```
  type ReturnType<T> = 
    T extends (...args: any) => infer R ? R : never

  type Fn = (str: string) => number

  type FnReturn = ReturnType<Fn> // number
  ```

##### 模块
###### 全局模块 vs. 文件模块


默认情况下，我们所写的代码是位于全局模块下的：



  `const foo = 2`
此时，如果我们创建了另一个文件，并写下如下代码，ts 认为是正常的：

  `const bar = foo // ok`
如果要打破这种限制，只要文件中有 import 或者 export 表达式即可：

  `export const bar = foo // error`


###### 模块解析策略


Tpescript 有两种模块的解析策略：Node 和 Classic。当 tsconfig.json 中 module 设置成 AMD、System、ES2015 时，默认为 classic ，否则为 Node ，也可以使用 moduleResolution  手动指定模块解析策略。



两种模块解析策略的区别在于，对于下面模块引入来说：



  `import moduleB from 'moduleB'`



 <b> Classic 模式的路径寻址： </b>


  ```
  /root/src/folder/moduleB.ts
  /root/src/folder/moduleB.d.ts
  /root/src/moduleB.ts
  /root/src/moduleB.d.ts
  /root/moduleB.ts
  /root/moduleB.d.ts
  /moduleB.ts
  /moduleB.d.ts
  ```


 <b> Node 模式的路径寻址： </b>


  ```
  /root/src/node_modules/moduleB.ts
  /root/src/node_modules/moduleB.tsx
  /root/src/node_modules/moduleB.d.ts
  /root/src/node_modules/moduleB/package.json (如果指定了"types"属性)
  /root/src/node_modules/moduleB/index.ts
  /root/src/node_modules/moduleB/index.tsx
  /root/src/node_modules/moduleB/index.d.ts

  /root/node_modules/moduleB.ts
  /root/node_modules/moduleB.tsx
  /root/node_modules/moduleB.d.ts
  /root/node_modules/moduleB/package.json (如果指定了"types"属性)
  /root/node_modules/moduleB/index.ts
  /root/node_modules/moduleB/index.tsx
  /root/node_modules/moduleB/index.d.ts

  /node_modules/moduleB.ts
  /node_modules/moduleB.tsx
  /node_modules/moduleB.d.ts
  /node_modules/moduleB/package.json (如果指定了"types"属性)
  /node_modules/moduleB/index.ts
  /node_modules/moduleB/index.tsx
  /node_modules/moduleB/index.d.ts
  ```

