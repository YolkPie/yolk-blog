---
title: Vue CLI3和TypeScript项目实践
date: 2020-06-19 14:45:44
tags:
- vue
- ts
categories: typescript
author: 马金坤
keywords: vue,TypeScript,Vue CLI3
description: vue,TypeScript,Vue CLI3
cover: https://img13.360buyimg.com/imagetools/jfs/t1/132074/32/2609/109500/5eec626dE785995ac/65e6c3a9122d6769.png
top_img: https://img14.360buyimg.com/imagetools/jfs/t1/110991/20/10424/10025/5eec626dE6bbbdd8e/e4013df3a17fb8fc.jpg
---
## typescript入门
### 基本数据类型
基本数据类型包括：boolean、number、string、null、undefined 以及 ES6 中的新类型 Symbol。
```
// 布尔值
let isShow: boolean = false

// 数字
let num: number = 6

// 字符串
let name: string = "bob"

// null 和 undefined
let u: undefined = undefined
let n: null = null
// undefined 和 null 是所有类型的子类型，可以赋值给其他类型
let num: number = undefined

// void 可以用 void 表示没有任何返回值的函数
function alertName(): void {
    alert('My name is Tom');
}
```
### 内置对象
* ECMAScript
```
let b: Boolean = new Boolean(1)
let e: Error = new Error('Error occurred')
let d: Date = new Date()
let r: RegExp = /[a-z]/
```
* DOM 和 BOM，有Document、HTMLElement、Event、NodeList 等
```
let body: HTMLElement = document.body
```
### 任意值
* any 类型的变量，允许被赋值为任意类型
```
let anything: any = 'seven'
anything = 7
```
* 变量如果在声明的时候，未指定其类型，没有赋值，会被推断成 any 类型，不做类型检查
```
// 被推断成 any 类型，不做类型检查
let anything;
anything = 'seven'
anything = 7
```
### 类型推论
若没有指定类型，ts会推断出一个类型
```
let anything = 'seven' // 推断为string类型
anything = 7 // 会报错

// 等价于
let anything: string = 'seven' // 推断为string类型
anything = 7 // 会报错
```
### 联合类型
使用 | 分隔每个类型。

当不确定联合类型的变量哪个类型的时：
* 只能访问该变量类型里共有的属性或方法：
```
function getLength(something: string | number): number {
    return something.length   // 报错
    return something.toString().length
}
```
* 可以使用**类型断言**来手动指定一个值的类型，通过 <类型>值 或 值 as 类型
```
function getLength(something: string | number): number {
    if ((<string>something).length) { // 或 something as string
        return (<string>something).length;
    } else {
        return something.toString().length;
    }
}
```
_注：类型断言不是类型转换，断言成一个联合类型中不存在的类型是不允许的，比如<boolean>something会报错。
* 联合类型的变量在被赋值时，会根据类型推论推断出一个类型
```
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
console.log(myFavoriteNumber.length); // 5
myFavoriteNumber = 7;
console.log(myFavoriteNumber.length); // 编译时报错
// index.ts(5,30): error TS2339: Property 'length' does not exist on type 'number'.
```
### type
* **类型别名**，用来给一个类型起个新名字
```
type Something = string | number
let something: Something = 'king'
```
* **字符串字面量类型**，用来约束取值只能是某几个字符串中的一个
```
type EventNames = 'click' | 'scroll' | 'mousemove'
function handleEvent(ele: Element, event: EventNames) { // event只能取这三种字符串，否则报错
    // do something
}
```
### 接口（对象类型）
使用接口（Interfaces）来定义对象的类型，接口一般首字母大写
```
// 定义的属性必须严格匹配，不允许增加或减少
interface Person {
    name: string;
    age: number;
}
// 可选属性
interface Person {
    name: string;
    age?: number;
}
// 定义任意属性，
interface Person {
    name: string;
    age?: number;
    readonly id: number;
    [propName: string]: any; // [propName: string] 定义了任意属性取 string 类型的值
}
```
> 一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集。例如，若定义[propName: string]: string，则确定属性和可选属性都必须是string类型

### 数组类型
* 类型+方括号
```
let fibonacci: number[] = [1, 2, 3]
```
* Array<elemType>
```
let fibonacci: Array<number> = [1, 2, 3]
```
* 用接口表示数组
```
interface NumberArray {
    [index: number]: number
}
let fibonacci: NumberArray = [1, 2, 3]
```
_注: 类数组，比如 arguments，其不是数组类型，不能用普通的数组的方式来描述，而应该用接口；类数组都有自己的接口定义，如 IArguments, NodeList, HTMLCollection 等
### 元组
数组合并了相同类型的对象，而元组合并了不同类型的对象
```
let tom: [string, number] = ['Tom', 25]

// 可以只赋值其中一项
let tom: [string, number]
tom[0] = 'Tom'

// 当添加越界的元素时，其类型会被限制为元组中每个类型的联合类型：
let tom: [string, number]
tom = ['Tom', 25]
tom.push('male')
tom.push(true) // 报错
```
### 函数类型
* 函数声明
```
function sum(x: number, y?: number): number {
    return x + y
}
```
* 函数表达式
```
const mySum = function (x: number, y: number): number {
    return x + y;
}
```
上面的代码虽然可以编译通过，但其并没有对左边的mySum进行了类型定义，只是通过赋值操作进行类型推论而推断出来的。正确的写法如下：
```
const mySum: (x: number, y: number) => number 
   = 
  (x: number, y: number) => {
    return x + y
  }
```
> 第一个 => 用来表示函数的定义，左边是输入类型(需要用括号括起来)，右边是输出类型；第二个=>是箭头函数
* 用接口或type定义函数的形状
```
// 1. 接口
interface SumFunc {
   (x: number, y: number): number
}
// 2. type，tslint更建议使用这种方式
type SumFunc = (x: number, y: number) => number
const mySum: SumFunc = (x: number, y: number) => {
   return x+y
}
```
### 类
TypeScript 有三种访问修饰符：
* public 修饰的属性或方法是公有的，可以在任何地方被访问到，默认是 public；
* private 修饰的属性或方法是私有的，外部和子类都不能访问；当构造函数修饰为 private 时，该类不允许被继承或者实例化；
* protected 修饰的属性或方法是受保护的，在子类中允许被访问；当构造函数修饰为 protected 时，该类只允许被继承，不允许被实例化
```
class Animal {
  protected name: string
  private age: number
  protected constructor (name, age) {
    this.name = name
    this.age = age
  }
}
class Cat extends Animal {
  private food: string
  constructor (name, age, food) {
    super(name, age)
    this.food = food
  }
  public sayHi() {
    console.log(`Meow, My name is ${this.name}`)
    // console.log(`Meow, My age is ${this.age}`) // 报错
  }
}
const animal = new Animal('Jack', 2) // 报错
const cat = new Cat('Jack', 2, 'fish')
console.log(cat.sayHi())
```
### 类与接口
接口（Interfaces）可以用于对「对象的形状」进行描述。若是不同类之间有一些共有的特性，可以把特性提取成接口，用 **implements** 关键字来实现：
```
interface Alarm {
    alert(): void
}
interface Light {
    lightOn(): void
}
class Car implements Alarm, Light {
    alert() {
        console.log('Car alert')
    }
    lightOn() {
        console.log('Car light on')
    }
}
```
* 接口继承接口：继承类型
```
interface Alarm {
    alert(): void
}
interface LightableAlarm extends Alarm {
    lightOn(): void;
}
```
* 接口继承类
```
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x
        this.y = y
    }
}
interface Point3d extends Point {
    z: number
}
let point3d: Point3d = {x: 1, y: 2, z: 3}

interface PointInstanceType {
    x: number
    y: number
}
function printPoint(p: Point) {
    console.log(p.x, p.y);
}
function printPoint(p: Point) {
    console.log(p.x, p.y);
}
```
> Point 当做一个类来用（使用 new Point 创建它的实例），也可以将 Point 当做一个类型来用。类型 PointInstanceType 和类型 Point 是等价的，只是缺少了构造函数、静态属性或静态方法
### 泛型
在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性
```
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = []
    for (let i = 0; i < length; i++) {
        result[i] = value
    }
    return result
}
createArray<string>(3, 'x')
```
* 使用泛型接口定义函数形状
```
interface CreateArrayFunc {
    <T>(length: number, value: T): Array<T>
}
let createArray: CreateArrayFunc
createArray = function<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value
    }
    return result
}
createArray <string> (3, 'x'); // ['x', 'x', 'x']
```
* 泛型类
```
class GenericNumber<T> {
    zeroValue: T
    add: (x: T, y: T) => T
}
let myGenericNumber = new GenericNumber<number>()
myGenericNumber.zeroValue = 0
myGenericNumber.add = function(x, y) { return x + y; }
```
## vue-cli3 + typescript
### 安装ts插件
* ts-loader 让webpack识别 .ts .tsx文件
* @typescript-eslint/parser ts文件解析器（不安装的话，eslint无法解析ts语法，会报Parsing error错误）
* @typescript-eslint/eslint-plugin 版本号需要与@typescript-eslint/parser的版本一致，解析器相关的配置选项
```
npm i ts-loader typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev
```
### webpack配置
找到./vue.config.js，配置如下：
```
configureWebpack: config => {
   // 更改入口文件
   config.entry.app = './src/main.ts'

   // 加上.ts 后缀（引入.ts的时候不写后缀）
   config.resolve.extensions.push('.ts')

   //  添加webpack对.ts的解析
   config.module.rules.push({
	test: /\.ts$/,
	exclude: /node_modules/,
	enforce: 'pre',
	loader: 'tslint-loader'
   })
   config.module.rules.push({
	test: /\.tsx?$/,
	loader: 'ts-loader',
	exclude: /node_modules/,
	options: {
		appendTsSuffixTo: [/\.vue$/],
	}
   })
}
```
### 添加 tsconfig.json
```
{
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ],
  "compilerOptions": {
    // 用来指定允许从没有默认导出的模块中默认导入，即允许import React from 'react'而不用只能import * as React from 'react'
    "allowSyntheticDefaultImports": true,
    // 用于指定是否启用装饰器特性
    "experimentalDecorators": true,
    // 允许编译javascript文件
    "allowJs": true,
    "module": "esnext",
    "target": "es5",
    // 如何处理模块
    "moduleResolution": "node",
    // 将每个文件作为单独的模块
    "isolatedModules": true,
    "lib": [
      "dom",
      "es5",
      "es2015.promise"
    ],
    // 是否包含可以用于 debug 的 sourceMap
    "sourceMap": true,
    "pretty": true,
    // 基准目录
    "baseUrl": "./",
    // 指定特殊模块的路径
    "paths": {
      "*": ["src/types/*"]
    }
  }
}
```
### 修改.eslintrc.js配置
```
module.exports = {
  root: true,
  env: {
    node: true,
    'browser': true,
    'commonjs': true,
    'es6': true,
  },
  parser: 'vue-eslint-parser',
  extends: [
    'plugin:vue/essential',
    '@vue/standard',
    // 新增配置项
    "plugin:@typescript-eslint/recommended"
  ],
  // 新增配置项
  plugins: ['@typescript-eslint'],
  parserOptions: {
    // 新增配置项
    parser: '@typescript-eslint/parser',
    ecmaFeatures: {
      'legacyDecorators': true
    }
  },
  rules: {
    'semi': ['error', 'never'],
    'no-extra-semi': 2,
		"space-before-function-paren": 0,
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    '@typescript-eslint/no-unused-vars': [0, { args: 'none' }], // 不限制定义的类型是否使用
    '@typescript-eslint/explicit-function-return-type': 0, // 不限制定义返回函数的类型
    '@typescript-eslint/no-empty-function': 0, // 不限制是空函数
    '@typescript-eslint/no-explicit-any': 0, // 不限制设置any类型
  }
}
```

### 修改main.js
把项目主文件 main.js 修改成 main.ts ，里面的写法基本不变，但是有一点需要注意： 引入Vue文件的时候，需要加上 .vue 后缀，因为 TypeScript 默认只识别 *.ts 文件，不识别 *.vue 文件。
```
import App from './App.vue'
```
### 让 ts 识别 .vue
由于 TypeScript 默认并不支持 *.vue 后缀的文件，在 vue 项目中引入的时候需要创建一个 vue-shim.d.ts 文件，例如 src/types/vue-shim.d.ts
```
declare module "*.vue" {
  import Vue from "vue";
  export default Vue;
}
```
### 开发vue项目
#### 安装vue相关插件
```
npm i vue-class-component vue-property-decorator vuex-class --save
```
#### vue-class-component

让 TypeScript 正确推断 Vue 组件选项中的类型，有两种方案，一种是Vue.extend（vue api上的用法），一种是使用vue-class-component

1）Vue.extend，使用基础 Vue 构造器，创建一个“子类”，其用法如下：
```
import Vue from 'vue'
export default Vue.extend({
   ... // 类型推断已启用
})
export default {
   // 这里不会有类型推断，
   // 因为 TypeScript 不能确认这是 Vue 组件的选项
}
```
2）vue-class-component的用法：
```
<template>
  <div></div>
</template>
<script>
import Vue from 'vue'
import Component from 'vue-class-component'
@Component({
  props: {
    propMessage: String
  }
})
export default class App extends Vue {
  // initial data
  msg = 123

  // use prop values for initial data
  helloMsg = 'Hello, ' + this.propMessage

  // lifecycle hook
  mounted () {
    this.greet()
  }

  // computed
  get computedMsg () {
    return 'computed ' + this.msg
  }

  // method
  greet () {
    alert('greeting: ' + this.msg)
  }
}
</script>
```
**Prop**

当从父组件传递数据到子组件时，通过 Prop 来实现；为了确保 Prop 的类型安全，我们会给 Prop 添加指定类型验证，之前的做法：
```
export default {
  props: {
    testProps: {
      type: Object,
      required: true,
      default: () => ({ message: 'test' })
    }
  }
}
```
上述定义了一个 testProps，它的类型是 Object。只定义Object，并不能获取更多的信息，在 TypeScript 看来，这将会是一个 any 类型。可以通过 TypeScript 添加更多的类型说明：
* 若是使用 Vue.extend() 或vue-class-component，给Prop添加类型注释时，需要以函数返回值的形式给 type 断言：
```
import Vue from 'vue'
interface User {
  name: string,
  age: number
}
export default Vue.extend({
  props: {
    testProps: {
      type: Object as () => User
    }
  }
})
```
* 若是使用 vue-propperty-decorator，给 prop 添加类型推荐时，就会变得简单：
```
import { Component, Vue, Prop } from 'vue-property-decorator'
interface User {
  name: string,
  age: number
}
@Component
export default class Test extends Vue {
  @Prop({ type: User })
  private test: { value: string }
}
```
#### vue-property-decorator
在 vue-class-component 上增强了更多的结合 Vue 特性的装饰器，简化书写，新增了这 7 个装饰器：
* @Component (完全继承于vue-class-component)
* @Emit
* @Inject
* @Provice
* @Prop
* @Watch
* @Model
```
import { Component, Emit, Inject, Model, Prop, Provide, Vue, Watch } from 'vue-property-decorator'

@Component
export class MyComponent extends Vue {
  @Prop({ default: 'default value' })
  propA: string

  @Prop([String, Boolean])
  propB: string | boolean
 
  count = 0

  @Watch('child')
  onChildChanged(val: string, oldVal: string) { }

  @Emit('reset')
  emitTodo(n: number){
    this.count += n
  }
  // 上面的代码等同于
  methods: {
    emitTodo(n) {
      this.count += n
      this.$emit('emitTodo', n)
    },
  }
  
  @Model('textInput', { type: String }) value
  // 上面的代码等同于
  model: {
    prop: 'value',
    event: 'textInput'
  },
  props: {
    value: String
  }
  
  // 在上一层级父组件里声明的provide，下一层级子组件无论多少级都可以通过inject来访问到provide的数据
  // 父组件
  @Provide()
  name = 'foo'
  // 子组件
  @Inject('name')
  // 上面的代码等同于
  // 父组件
  provide: {
     name: 'foo'
  }
  // 子组件
  inject: [name]
}
```
#### vuex-class
vuex-class 是基于 vue-class-component 对 Vuex 提供的装饰器。   
目录结构：
```
store  
├── modules                  
│   ├── home.ts    
│   └── xxx.ts  
├── index.ts  
└── type.ts   
``` 
index.ts
```
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
// 自动引入 modules 文件夹下的js文件，以文件名字作为对象的key
const modulesContext = require.context('./modules', false, /.*\.ts/)
const modules = modulesContext.keys().reduce((prev, cur) => {
  const key = cur.match(/(\w+)\.ts/)[1]
  prev[key] = modulesContext(cur).default
  return prev
}, {})

export default new Vuex.Store({
  modules
})
```
_注：会报“Property 'context' does not exist on type 'NodeRequire'.”的错误，这里需要安装webpack类型声明：
```
npm i @types/webpack-env --save-dev
```
home.ts 
```
import TYPES from '../types'
import {
  QueryParam,
  ResponseData
} from '@/types/index.d'
import * as Api from '@/utils/api.ts'

interface State {
  skuInfo: any,
  [propName: string]: any
}

const initState: State = {
  skuInfo: {}
}
export default {
  namespaced: true,
  state: initState,
  mutations: {
    [TYPES.SET_SKU_INFO] (state, skuInfo) {
      state.skuInfo = skuInfo
    }
  },
  actions: {
    getSkuInfo ({ dispatch, commit, getters, rootGetters }, { skuId }) {
      return new Promise((resolve, reject) => {
        Api.requestData({
          functionId: 'getSkuInfo',
          bodyParams: {
            skuId
          }
        }).then((resData: ResponseData) => {
          const { code, data } = resData
          if (code === '1' && data) {
            commit(TYPES.SET_SKU_INFO, data)
            resolve(data)
          } else {
            reject(resData)
          }
        }).catch(err => {
          console.log(err, 'getSkuInfo err!')
          reject(err)
        })
      })
    }
  }
}
```
在组件中使用 ‘home’ 模块中定义的 'skuInfo' State和 ‘getSkuInfo’ Action
* 第一种方式：  
```
import Vue from 'vue'
import Component from 'vue-class-component'
import { State, Action, namespace } from 'vuex-class'
const homeModule = namespace('home')

@Component
export class Home extends Vue {
  @homeModule.State('skuInfo') skuInfo
  @homeModule.Action('getSkuInfo') getSkuInfo: ({ skuId }) => Promise<any>
  created () {
    this.getSkuInfo({ skuId: 64967590189 })
  }
}
```
* 第二种方式：
```
import Vue from 'vue'
import Component from 'vue-class-component'
import { State, Action } from 'vuex-class'

@Component
export class Home extends Vue {
  @State('skuInfo', { namespace: 'home' }) skuInfo: any
  @Action('getSkuInfo', { namespace: 'home' }) getSkuInfo: ({ skuId }) => Promise<any>
  created () {
    this.getSkuInfo({ skuId: 64967590189 })
  }
}
```
上面的代码相当于：
```
import Vue from 'vue'
import { mapState, mapActions } from 'vuex'
export default Vue.extend({
   computed: {
     ...mapState('home', ['skuInfo'])
   },
   methods: {
     ...mapActions('home', ['getSkuInfo'])
   }
})
```
### 书写声明文件
#### 为本项目定义类型文件(可以重复引用)
```
// 1. 全局类型声明，无需import
// 1.1
declare type QueryParam = string | string[]

declare interface ResponseData {
  code: number | string;
  data?: any;
  msg?: string;
}

// 1.2 定义一个命名空间，声明这个拥有多个子属性的全局变量
// 用法：homeData.QueryParam 或 homeData.ResponseData
declare namespace homeData {
  type QueryParam = string | string[]

  interface ResponseData {
    code: number | string;
    data?: any;
    msg?: string;
  }
}

// 2.1 模块类型声明，需import引入类型文件
export type QueryParam = string | string[]

export interface ResponseData {
  code: number | string;
  data?: any;
  msg?: string;
}

```
#### 为第三方库定义类型声明文件

若是引用的第三方库已经定义好了类型声明文件，直接npm安装即可，例如webpack：
```
npm i @types/webpack-env --save-dev
```
尝试使用npm install @type/xxx 命令来安装声明文件，若是安装失败，则第三方库没有提供声明文件时，需要自己书写声明文件。

**全局库**

即通过 <script> 标签引入第三方库，能在全局命名空间下访问的（例如：不需要使用任何形式的import）。
假设通过script标签引入jQuery，类型声明如下：
创建一个 global.d.ts 文件，用来存放全局类型声明，例如 src/types/global.d.ts
```
declare var jQuery: (selector: string) => any
```
**模块化库**

有两种写法，一种是全局类型声明，另一种则是模块导出声明。以为第三方工具类库@yolkpie/utils定义类型声明文件为例：  
* **全局类型声明**  

创建一个 global.d.ts 文件，用来存放全局类型声明，例如 src/types/global.d.ts
```
// 只需要对引用到的方法进行类型声明
declare module '@yolkpie/utils' {
  export function isSupportWebp(): boolean
  export function rem(): void
  export function formatDate(date: Date|string|number, format?: string): string
}
declare module 'vue-awesome-swiper' {
  export const swiper: any
  export const swiperSlide: any
}
```
* **模块导出声明**  

创建一个 types 目录，专门用来管理自己写的声明文件，将 @yolkpie/utils 的声明文件放到 types/@yolkpie/utils/index.d.ts 中。这种方式需要配置下 tsconfig.json 中的 paths 和 baseUrl 字段。
目录结构：
```
src  
└── types                  
    └── @yolkpie    
        ├── utils 
        │   └── index.d.ts
        ├── global.d.ts 
        └── vue-shim.d.ts   
``` 
tsconfig.json 内容：
```
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "*": ["src/types/*"]
        }
    }
}
```
这样配置之后，通过 import 导入 @yolkpie/utils  的时候，也会去 types 目录下寻找对应的模块的声明文件了。
例如：src/types/@yolkpie/utils/index.d.ts
```
// 1.直接export
export function isSupportWebp(): boolean
export function rem(): void
export function formatDate(date: Date|string|number, format?: string): string

// 2.export和declare混合
declare function isSupportWebp(): boolean
declare function rem(): void
declare function formatDate(date: Date|string|number, format?: string): string
declare namespace jQuery {
  function ajax(url: string, settings?: any): void;
}
export {
  rem,
  isSupportWebp,
  formatDate
}
```
