---
title: "TypeScript分享"
date: "2021-09-09"
author: ZLF
---

# typescript 分享

![TypeScriptMap](https://tva1.sinaimg.cn/large/007S8ZIlly1gh8se06hddj30vq0nwgm3.jpg)

## 为什么要使用 TypeScript

微软推出 TypeScript 主要是为了实现两个目标。

1. 为 JavaScript 提供可选的类型系统：TypeScript 会在编译代码时进行严格的静态类型检查，这意味着你可以在编码阶段发现可能存在的隐患，而不必把它们带到线上。
2. 兼容当前及未来的 JavaScript 的特性：TypeScript 会包括来自 ES 6 和未来提案中的特性，比如异步操作和装饰器；也会从其他语言借鉴某些特性，比如接口和抽象类。

## 基础

### 类型基础

在了解 ts 的各种基本类型之前，首先先了解一下类型语言。

> 在强类型语言中，当一个对象从调用函数传递到被调用函数时，其类型必须与被调用函数中声明的类型兼容
> 强类型语言不允许改变变量的数据类型，除非进行强制类型转换
> 弱类型语言中，变量可以被赋予不同的数据类型

> 静态类型语言：在编译阶段确定所有变量的类型（如 c++）

    1. 对类型要求极其严格
    2. 立即发现错误
    3. 运行时性能较好
    4. 自文档化

> 动态类型语言：在执行阶段确定所有变量的类型（如 js）

    1. 对类型非常宽松
    2. Bug可能隐藏很久
    3. 运行性能较差
    4. 可读性差

注意：有人提出强类型语言在程序发生错误后不允许继续执行，但是 c++并没有对数组越界进行处理，所以认为它是弱类型语言。（归属特殊定义).

![语言象限图](https://tva1.sinaimg.cn/large/007S8ZIlly1ggabqdv9m6j30dy07bwef.jpg)

注意：如果把 ts 当做一门语言看待，那么它是强类型还是弱类型，是静态类型还是动态类型

我认为是静态+强类型语言

### 基本类型

![基本类型](https://tva1.sinaimg.cn/large/007S8ZIlly1gh5m0bv2obj30kk0rcglu.jpg)

#### 数组

TypeScript 像 JavaScript 一样可以操作数组元素。 有两种方式可以定义数组。 第一种，可以在元素类型后面接上 []，表示由**此类型元素**组成的一个数组：

`let list: number[] = [1, 2, 3];`

第二种方式是使用数组泛型，Array<元素类型>：

`let list: Array<number> = [1, 2, 3];`

#### 元组

元组为一个新的概念，元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为 string 和 number 类型的元组。

数组合并相同类型的对象，元组合并了不同类型的对象。

```ts
// 定义一个元组类型
let x: [string, number];
// 赋值
x = ["hello", 10]; // OK
// 不正确的赋值
x = [10, "hello"]; // Error
```

#### 函数

##### 函数定义方式

函数类型包含两部分：参数类型和返回值类型，

##### 函数声明

```ts
function sum(x: number, y: number): number {
  return x + y;
}
```

##### 函数表达式

```ts
let mySum = function(x: number, y: number): number {
  return x + y;
};
```

其中，ts 可以根据参数类型推断出返回值类型。

##### 函数参数

1. 可选参数
2. 默认参数
3. 剩余参数

```ts
// 1. 可选参数

function buildName(firstName: string, lastName?: string) {
  console.log("buildName -> firstName", firstName, lastName);
}
// 2. 默认参数
function buildName1(firstName: string, lastName = "Smith") {
  console.log("buildName -> firstName", firstName, lastName);
}
// 3. 剩余参数

function buildName2(firstName: string, ...restOfName: string[]) {
  console.log("buildName2", firstName + " " + restOfName.join(" "));
}

let employeeName = buildName2("Joseph", "Samuel", "Lucas", "MacKinzie");
```

##### 函数重载

typescript 可以 为同一个函数提供多个函数类型定义来进行函数重载

1. 函数重载的意义在于能够让你知道传入不同的参数得到不同的结果，如果传入的参数不同，但是得到的结果（类型）却相同，那么这里就不要使用函数重载（没有意义）。

```ts
// 4. 函数重载

let suits = ["hearts", "spades", "clubs", "diamonds"];

// 前两个为重载
function pickCard(x: { suit: string; card: number }[]): number;
function pickCard(x: number): { suit: string; card: number };
function pickCard(x: string | number | any[]): any {
  if (typeof x == "object") {
    let pickedCard = Math.floor(Math.random() * x.length);
    return pickedCard;
  } else if (typeof x == "number") {
    let pickedSuit = Math.floor(x / 13);
    return { suit: suits[pickedSuit], card: x % 13 };
  }
}

let myDeck = [
  { suit: "diamonds", card: 2 },
  { suit: "spades", card: 10 },
  { suit: "hearts", card: 4 },
];
let pickedCard1 = myDeck[pickCard(myDeck)];
console.log("card-Object: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
console.log("card-Number: " + pickedCard2.card + " of " + pickedCard2.suit);
```

#### 对象

object 表示非原始类型，也就是除 number，string，boolean，symbol，null 或 undefined 之外的类型。

```ts
let obje: { x: number; y: number } = { x: 1, y: 2 };
obje.x = 3;

console.log("functionpickCard -> obje", obje);
```

#### undefined 和 null

默认情况下 null 和 undefined 是所有类型的子类型。 就是说你可以把 null 和 undefined 赋值给 number 类型的变量。

```ts
let un: undefined = undefined;
let nu: null = null;

let num1;

num1 = null;
num1 = undefined;
num1 = 1;

console.log("undefined-null -> num", num1);
```

#### Void

某种程度上来说，void 类型像是与 any 类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是 void.

```ts
let noReturn = () => {};
```

#### Never

never 类型表示的是那些永不存在的值的类型。 例如，never 类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never 类型，当它们被永不为真的类型保护所约束时。

never 类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是 never 的子类型或可以赋值给 never 类型（除了 never 本身之外）。 即使 any 也不可以赋值给 never。

```ts
// 这里的never指的是会抛出、返回错误或者无限循环
let error = () => {
  throw new Error("error");
};

// 返回never的函数必须存在无法达到的终点
// while 循环会一直循环代码块，只要指定的条件为 true。
let endless = () => {
  while (true) {}
};
```

#### any

当一个变量为 any 类型，不作限制

```ts
let xabc;
xabc = 1;
xabc = [];
xabc = () => {};
```

#### 枚举

基本实现：

```ts
enum Role {
  Reporter = 1,
  Developer,
  Maintainer,
  Owner,
  Guest,
}
```

##### 反向映射

1. 枚举被编译为对象
2. 枚举成员的名称被作为 key， 枚举成员的值被作为 value， 表达式返回 value
3. 然后，value 又被作为 key，成员名称又被作为 value，返回枚举成员的名称。这种方法叫做反向映射

上述 ts 中的枚举转换为 JavaScript 代码如下，其中就是利用了反向映射的原理

```js
var Role;
(function(Role) {
  Role[(Role["Reporter"] = 1)] = "Reporter";
  Role[(Role["Developer"] = 2)] = "Developer";
  Role[(Role["Maintainer"] = 3)] = "Maintainer";
  Role[(Role["Owner"] = 4)] = "Owner";
  Role[(Role["Guest"] = 5)] = "Guest";
})(Role || (Role = {}));
```

这里可以使用 [tsPlayGround](https://www.tslang.cn/play/index.html) 查看编译之后的 js 代码

##### 数字枚举&字符串枚举

1. 使用枚举很简单：通过枚举的属性来访问枚举成员，和枚举的名字来访问枚举类型。
2. 除了创建一个以属性名做为对象成员的对象之外，数字枚举成员还具有了 反向映射，从枚举值到枚举名字
3. 正向映射（ name -> value）和反向映射（ value -> name）
4. 数字枚举的实现原理为反向映射
5. 在一个字符串枚举里，每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化。
6. 要注意的是 不会为字符串枚举成员生成反向映射。相比数字枚举,字符串枚举仅成员名称被作为 key,所以不支持反向映射

```ts
// 数字枚举
enum Role {
  Reporter = 1,
  Developer,
  Maintainer,
  Owner,
  Guest,
}

// 字符串枚举
enum Message {
  Success = "恭喜你，成功了",
  Fail = "抱歉，失败了",
}
```

##### 常量枚举

1. 常数枚举是使用 const enum 定义的枚举类型
2. 常数枚举与普通枚举的区别是，它会在编译阶段被删除，并且不能包含计算成员。
3. 假如包含了计算成员，则会在编译阶段报错。
4. "const" 枚举仅可在属性、索引访问表达式、导入声明的右侧或导出分配中使用。

```ts
const enum Month {
  Jan,
  Feb,
  Mar,
  Apr = Month.Mar + 1,
}
let month = [Month.Jan, Month.Feb, Month.Mar];
```

##### 常数项和计算所得项

1. 枚举项有两种类型：常数项（constant member）和计算所得项（computed member）。
2. 枚举成员的值在定义后变为只读类型，在定义之后是不能修改的

```ts
enum Char {
  // const member
  a,
  b = Char.a,
  c = 1 + 3,
  // computed member
  d = Math.random(),
  // g, // 如果紧接在计算所得项后面的是未手动赋值的项，那么它就会因为无法获得初始值而报错
  e = "123".length,
  f = 4,
}
```

##### 枚举类型

1. 在某些情况下, 枚举和枚举成员都可以作为一种单独的类型
2. 分为三种情况：
   1. 枚举成员没有任何初始值
   2. 所有枚举成员都是数字枚举
   3. 所有枚举成员都是字符串枚举
3. 可将任意 number 类型赋值给枚举类型,取值也可以超出枚举成员定义。
4. 两种不同类型的枚举,是不可以进行比较的,编辑器会报错

```ts
// 枚举成员没有任何初始值
enum E {
  a,
  b,
}
// 所有枚举成员都是数字枚举
enum D {
  a,
  b,
}
// 所有枚举成员都是数字枚举
enum F {
  a = 0,
  b = 1,
}
// 所有枚举成员都是字符串枚举
enum G {
  a = "apple",
  b = "banana",
}

let e: E = 3;
let dac: D = 3;
let fok: F = 3;
```

### 接口

TypeScript 的核心原则之一是对值所具有的结构进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 在 TypeScript 里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

在面向对象语言中，接口（Interfaces）是一个很重要的概念，它是对行为的抽象，而具体如何行动需要由类（classes）去实现（implement）。

TypeScript 中的接口是一个非常灵活的概念，除了可用于对类的一部分行为进行抽象以外，也常用于对「对象的形状（Shape）」进行描述。

##### 基本实现

**赋值**的时候，变量的形状必须和接口的形状保持一致。

```ts
interface List {
  readonly id: number; // 只读
  name: string; // 确定
  age?: number; // 可选
  // [x: string]: string; // 任意
  [x: string]: any; // 任意
}
interface Result {
  data: List[];
}
```

##### 关于属性

1. 属性了解：
   1. 可选属性（?:）
   2. 任意属性 ([x: string]: any)
      1. 一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集
      2. 一个接口中只能定义一个任意属性。如果接口中有多个类型的属性，则可以在任意属性中使用联合类型
   3. 只读属性 (readonly)
      1. 只读的约束存在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候

```ts
interface List {
  readonly id: number; // 只读
  name: string; // 确定
  age?: number; // 可选
  // [x: string]: string; // 任意
  [x: string]: any; // 任意
}
```

##### 类型检查跳过

为什么需要跳过类型检查？

对象类型接口直接验证有冗余字段的对象字面量时会报错，这种冗余字段有时是不可避免的存在的。

方法主要有三种：

1. 将对象字面量赋值给变量
2. 使用类型断言
3. 用字符串索引签名

第一种：将对象字面量赋值给变量

```ts
// 在外面声明变量 result ,然后把 result 传入 render 函数，避免传入对象字面量
function render(result: Result) {
  result.data.forEach(value => {
    console.log("result: id,name", { id: value.id, name: value.name });
    if (value.age) {
      console.log("result: age", { age: value.age });
    }
    // 只读属性不可进行操作
    // value.id++
  });
}
// 数据结构描述
// 赋值的时候，变量的形状必须和接口的形状保持一致。
let result = {
  data: [
    { id: 1, name: "A", sex: "male" },
    { id: 2, name: "B", age: 10 },
  ],
};
```

第二种：类型断言

```ts
let square = <Square1>{};
// let square = {} as Square1;
square.color = "blue";
square.sideLength = 10;
```

第三种：字符串索引签名》 `[x: string]: any; // 任意`

##### 可索引类型接口

1. 数字索引
2. 字符串索引

可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。 这是因为当使用 number 来索引时，JavaScript 会将它转换成 string 然后再去索引对象。 也就是说用 100（一个 number）去索引等同于使用"100"（一个 string）去索引，因此两者需要保持一致。

当接口中定义了一个索引后，例如设置了 【x:string】= string，就不能设置 y：number 了。
因为设置了【x:string】= string 相当于这个接口的字符串索引返回值都是字符串，而 y：number 违背这一原则，冲突了。反过来 如果定义了【x:string】=Number, 就不能设置 y:string 了。

```ts
interface StringArray {
  [index: number]: string;
}
let chars: StringArray = ["a", "b"];

// 混用时，数字索引签名的返回值必须是字符串索引签名返回值的子类型
interface Names {
  [x: string]: any;
  // y: number;
  [z: number]: number;
}
```

##### 类类型接口

1. 类必须实现接口中的所有属性
2. 接口只能约束类的公共成员，不能约束私有成员、受保护成员、静态成员和构造函数
3. 接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员

##### 接口继承接口

1. 一个接口可以继承多个接口，创建出多个接口的合成接口

##### 接口继承类

1. 当接口继承了一个类类型时，它会继承类的成员但不包括其实现。
2. 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。
3. 接口同样会继承到类的 private 和 protected 成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

##### 关于 interface 和 type

1. interface: 接口
2. type: 类型别名

接口就好比一个名字，用来描述对象。type 会给一个类型起个新名字。 type 有时和 interface 很像，但是可以作用于原始值（基本类型），联合类型，元组以及其它任何你需要手写的类型。

官方的文档说：

> "类型别名 可以和 interface 关键字一样，然而他们有一些细微的差别。"

不同点：

1. 扩展语法： interface 使用 extends，type 使用‘&’
2. 同名合并：interface 支持，type 不支持。
3. 描述类型：对象、函数两者都适用，但是 type 可以用于基础类型、联合类型、元祖。
4. 计算属性：type 支持计算属性，生成映射类型,interface 不支持。

相同点：

1. 两者都可以用来描述对象或函数的类型
2. 两者都可以实现继承

总的来说，公共的用 interface 实现，不能用 interface 实现的再用 type 实现。主要是一个项目最好保持一致。

[推荐：jsonToInterface 转换](https://static.jixun.moe/json2tsinterface.html)

```ts
// jsonToInterFace convert eg
{
  "data": [
    {
      "id": 1,
      "name": "A",
      "sex": "male"
    },
    {
      "id": 2,
      "name": "B",
      "age": 10
    }
  ]
}
```

### 类

传统方法中，JavaScript 通过构造函数实现类的概念，通过原型链实现继承。而在 ES6 中，我们终于迎来了 class。

TypeScript 除了实现了所有 ES6 中的类的功能以外，还添加了一些新的用法。

##### 基本实现

1. 类中定义的属性都是实例属性，类中定义的方法都是原型方法
2. 实例属性必须有初始值，或在函数中被赋值，或为可选成员

```ts
// 我们声明一个 Greeter类。这个类有3个成员：一个叫做 greeting的属性，一个构造函数和一个 greet方法。
// 你会注意到，我们在引用任何一个类成员的时候都用了 this。 它表示我们访问的是类的成员。
// 最后一行，我们使用 new构造了 Greeter类的一个实例。 它会调用之前定义的构造函数，创建一个 Greeter类型的新对象，并执行构造函数初始化它。

class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter = new Greeter("world");
```

##### 继承

基于类的程序设计中一种最基本的模式是允许使用继承来扩展现有的类。基本继承实现如下：

```ts
// 类从基类中继承了属性和方法。 这里， Dog是一个 派生类，它派生自 Animal 基类，通过 extends关键字。 派生类通常被称作 子类，基类通常被称作 超类。

class Animal {
  move(distanceInMeters: number = 0) {
    console.log(`Animal moved ${distanceInMeters}m.`);
  }
}

class DogC extends Animal {
  bark() {
    console.log("Woof! Woof!");
  }
}

const dog = new DogC();
dog.bark();
dog.move(10);
dog.bark();
```

更复杂的继承，子类的构造函数中必须含有 super 调用。这一次，我们使用 extends 关键字创建了 Animal 的两个子类： Horse 和 Snake。

与前一个例子的不同点是，派生类包含了一个构造函数，它 必须调用 super()，它会执行基类的构造函数。 而且，在构造函数里访问 this 的属性之前，我们 一定要调用 super()。 这个是 TypeScript 强制执行的一条重要规则。

```ts
class Animal {
  name: string;
  constructor(theName: string) {
    this.name = theName;
  }
  move(distanceInMeters: number = 0) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}

class Snake extends Animal {
  constructor(name: string) {
    super(name);
  }
  move(distanceInMeters = 5) {
    console.log("Slithering...");
    super.move(distanceInMeters);
  }
}

class Horse extends Animal {
  constructor(name: string) {
    super(name);
  }
  move(distanceInMeters = 45) {
    console.log("Galloping...");
    super.move(distanceInMeters);
  }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

##### 成员修饰符

1. public：对所有人可见，所有成员默认为 public
2. private：
   1. 只能在被定义的类中访问，不能通过实例或子类访问
   2. private constructor：不能被实例化，不能被继承
3. protected
   1. 只能在被定义的类和子类中访问，不能通过实例访问
   2. protected constructor：只能被实例化，不能被继承
4. readonly：必须有初始值，或在构造函数中被赋值
5. static：只能由类名调用，不能通过实例访问，可继承

##### 构造函数参数中的修饰符

1. 将参数变为实例属性

##### 抽象类

1. 不能被实例化，只能被继承
   1. 抽象方法包含具体实现，子类可以直接复用
   2. 抽象方法不包含具体实现，子类必须实现
2. 多态：多个子类对父抽象类的方法有不同实现，实现运行时绑定

##### this 类型

1. 实现实例方法的链式调用
2. 在继承时，具有多态性，保持父子类之间接口调用的连贯性

### 泛型

软件工程中，我们不仅要创建一致的定义良好的 API，同时也要考虑可重用性。 组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。

#### 泛型函数

设计泛型的关键目的是在成员之间提供有意义的约束，这些成员可以是：类的实例成员、类的方法、函数参数和函数返回值。

我们需要一种方法使返回值的类型与传入参数的类型是相同的，如果没有这种方法，在写代码时可能是这样的

```ts
function identity1(arg: number): number {
  return arg;
}

function identity(arg: any): any {
  return arg;
}
```

但是使用 any 可能会丢失一些信息：传入的类型与返回的类型应该是相同的

使用泛型则是下面所示：

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

我们把这个版本的 identity 函数叫做泛型，因为它可以适用于多个类型。 不同于使用 any，它不会丢失信息，像第一个例子那像保持准确性，传入数值类型并返回数值类型。

定义泛型函数之后，有两种使用方法：

```ts
function log<T>(value: T): T {
  console.log(value);
  return value;
}

// 第一种是，传入所有的参数，包含类型参数：
log<string[]>(["a", ",b", "c"]);

// 第二种是，利用了类型推论 -- 即编译器会根据传入的参数自动地帮助我们确定T的类型：
log(["a", ",b", "c"]);
```

#### 支持多种类型的方法

1. 函数重载
2. 联合类型
3. any 类型 》 丢失类型约束
4. 泛型 》不预先确定的类型，使用时才确定

#### 泛型接口

```ts
interface CreateArrayFunc<V> {
  (length: number, value: V): Array<V>;
}

function identity<T>(length: number, value: T): Array<T> {
  let result: T[] = [];
  for (let i = 0; i < length; i++) {
    result[i] = value;
  }
  console.log("泛型接口》result", result);
  return result;
}

let myIdentity: CreateArrayFunc<string> = identity;

myIdentity(3, "x"); // ['x', 'x', 'x']
```

#### 泛型类

<!-- 重点研究 -->

泛型类看上去与泛型接口差不多。 泛型类使用（ <>）括起泛型类型，跟在类名后面。

与接口一样，直接把泛型类型放在类后面，可以帮助我们确认类的所有属性都在使用相同的类型。

我们在类那节说过，类有两部分：静态部分和实例部分。 泛型类指的是实例部分的类型，所以类的静态属性不能使用这个泛型类型。

```ts
class GenericNumber<T> {
  // ! 非空断言. 联合类型，可选属性
  zeroValue!: T;
  add!: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) {
  return x + y;
};
```

#### 泛型约束

1. 确保属性存在
2. 检查对象上的键是否存在

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}

loggingIdentity("avc");
// loggingIdentity(23123)
```

现在这个泛型函数被定义了约束，因此它不再是适用于任意类型。

#### 泛型工具类型

1. `Partial<T>` 的作用就是将某个类型里的属性全部变为可选项 ?。
2. `Record<K extends keyof any, T>`的作用是将 K 中所有的属性的值转化为 T 类型。
3. `Pick<T, K extends keyof T>` 的作用是将某个类型中的子属性挑出来，变成包含这个类型部分属性的子类型。
4. `Exclude<T, U>` 的作用是将某个类型中属于另一个的类型移除掉。
5. `ReturnType<T>`的作用是用于获取函数 T 的返回类型。
6. `Readonly<T>`: 将 T 的所有属性变为只读

##### Partail

```ts
// Partail ts源码实现
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

它用来将 T 中的所有的属性都变成可选的。下面的示例中定义了一个类型 IFoo，它拥有两个必选的属性 a 和 b。

使用：

```ts
interface IFoo {
  a: number;
  b: number;
}

const foo: Partial<IFoo> = { a: 1 };
```

相当于

```ts
interface IFoo {
  a?: number;
  b?: number;
}
```

##### Record

```ts
// 源码实现
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

```ts
// 第一种场景
type Fooad = Record<"a", string>;
const fooad: Fooad = { a: "1" }; // 正确
```

可以用 Record 来处理另外一种场景。假如我本来已经有了两个类型：

```ts
interface Foo {
  a: string;
}
interface Bar {
  b: string;
}
```

我想把 Foo 和 Bar 两个类型的 key 合并到一起，并给它们重新指定成 number 类型，可以使用 Record 这样实现：

```ts
type Baz = Record<keyof Foo | keyof Bar, number>;
```

相当于

```ts
interface Baz {
  a: number;
  b: number;
}
```

使用

它用来生成一个属性为 K，类型为 T 的类型集合。如下所示，我用它生成了一个 Foo 类型，那么就表示所有指定为 Foo 类型的变量都必须包含一个 key 为 a，value 为 string 类型的字段。否则，TS 类型检查器就会报错。

```ts
type Foo = Record<"a", string>;

const foo: Foo = { a: "1" }; // 正确
const foo: Foo = { b: "1" }; // 错误，因为 key 不为 a
const foo: Foo = { a: 1 }; // 错误，因为 value 的值不是 str
```

##### Pick

```ts
// 源码实现
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

使用

它的作用是从 T 中将所有的 K 取出来，并生成一个新的类型。下面示例中定义的 IFoo 类型包含了两个必选属性 a 和 b。所以，将 foo 指定为 IFoo 类型之后，就肯定必须包含这两个属性，否则就会报类型检查错误：

```ts
interface IFoo {
  a: number;
  b: number;
}

const foo: IFoo = {
  a: 1,
  b: 2,
};
```

但是，如果我想让 foo 只包含 IFoo 类型的 a 属性，就可以用 Pick 这样来实现。它就是告诉 TS 仅仅将 a 属性从 IFoo 中提取出来即可。

```ts
// 正确，使用 Pick 生成的新类型确实只包含 a 属性
const foo: Pick<IFoo, "a"> = {
  a: 2,
};

// 错误，使用 Pick 生成的新类型中并不包含 b 属性
const foo: Pick<IFoo, "a"> = {
  b: 2,
};
```

注意，它和上面的 Partial 不一样的地方在于，Partial 是将类型中的所有的属性都变成了可选状态，而不能将某一个属性单独提取出来。

##### Exclude

```ts
// 源码
type Exclude<T, U> = T extends U ? never : T;
```

使用

它的作用是从 T 中排除掉所有包含的 U 属性。如果不明白这句话，就看下面示例。

代码运行之后，TFoo 只会包含一个 2。这是因为 Exclude 会从第一个类型参数中将其所有包含的第二个类型参数中的值给排除掉。我们可以看到在第一个类型参数中只包含第二个类型参数中的 1，因此，它就会被排除掉，只剩下 2 了。

```ts
type TFoo = Exclude<1 | 2, 1 | 3>;
```

所以，如果一个变量被指定为了 TFoo 类型，它就只能被赋值为 2 了，否则就会报类型检查错误：

```ts
const foo: TFoo = 2; // 正确
const foo: TFoo = 3; // 错误，因为 TFoo 中不包含 3
```

##### ReturnType

```ts
type ReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : any;
```

它用来得到一个函数的返回值类型。看下面的示例用 ReturnType 获取到 Func 的返回值类型为 string，所以，foo 也就只能被赋值为字符串了。

```ts
type Func = (value: number) => string;

const foo: ReturnType<Func> = "1";
```

##### Readonly

```ts
// Readonly在 TS 中的源码实现：

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

使用

```ts
interface IFoo {
  name: string;
  age: number;
}

const foo: Readonly<IFoo> = {
  name: "cxc",
  age: 22,
};

foo.name = "xiaoming"; // 错误，因为 name 仅是只读的
foo.age = 20; // 错误，因为 age 也仅是只读的
```

项目中不一定要强制使用泛型，还是应该用在对的场景。

### 类型检查机制

#### 类型推断

TypeScript 里，在有些没有明确指出类型的地方，类型推论会帮助提供类型。

```ts
let a15 = 1; // let a15: number
let b15 = [1, null, "a"]; // let b15: (string | number | null)[]
let c15 = { x: 1, y: "a" };
// let c15: {
//     x: number;
//     y: string;
// }
```

#### 类型断言

有时候你会遇到这样的情况，你会比 TypeScript 更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript 会假设你，程序员，已经进行了必须的检查。

两种写法, 当你在 TypeScript 里使用 JSX 时，只有 as 语法断言是被允许的。

1. 第一种：尖括号语法
2. 第二种：as 语法

```ts
// 尖括号
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
// as 语法
let someValueAs: any = "this is a string";
let strLengthAs: number = (someValue as string).length;
```

弊端是没有按照接口的约定赋值

#### 类型兼容性

typeScript 结构化类型系统的基本规则是，如果 x 要兼容 y，那么 y 至少具有与 x 相同的属性.

当一个类型 Y 可以被赋值给另一个类型 X 时，我们就可以说类型 X 兼容类型 Y。

X 兼容 Y：X（目标类型） = Y（源类型），源类型必须具备目标类型的所有属性。

1. 接口之间兼容：成员少的兼容成员多的
2. 函数之间兼容：
   1. 参数多的兼容参数少的
   2. 可选参数和剩余参数，遵循原则
      1. 固定参数兼容可选参数和剩余参数
      2. 可选参数不兼容固定参数和剩余参数（严格模式）
      3. 剩余参数兼容固定参数和可选参数
   3. 参数类型：必须匹配
   4. 参数为对象：
      1. 严格模式：成员多的的兼容成员少的
      2. 非严格模式：相互兼容（函数参数双向协变）
   5. 返回值类型：目标函数必须与源函数相同，或为其子类型
3. 枚举之间兼容：
   1. 枚举类型和数字类型相互兼容
   2. 枚举类型之间不兼容
4. 类兼容性：
   1. 静态成员和构造函数不在比较范围
   2. 两个类具有相同的实例成员，它们的示例相互兼容
   3. 类中包含私有成员或受保护成员，只有父类和子类的实例相互兼容
5. 泛型之间兼容：
   1. 泛型接口：只有类型参数 T 被接口成员使用时，才会影响兼容性
   2. 泛型函数：定义相同，没有指定类型参数时就兼容

```ts
// 简单的接口兼容
interface X {
  a: any;
  b: any;
}
interface Y {
  a: any;
  b: any;
  c: any;
}
let x: X = { a: 1, b: 2 };
let y: Y = { a: 1, b: 3, c: 3 };
x = y;

// x : {a: 1, b: 3, c: 3}
```

#### 类型保护

typescript 能够在特定的区块中保证变量属于某种确定的类型，可以在此区块中放心的引用此类型的属性，或者调用此类型的方法。

类型保护方法：

1. 使用 instanceof 可以判断一个实例是不是属于某个类
2. 使用 in 可以判断一个属性是不是属于某个对象
3. 使用 typeof 可以判断一个基本类型
4. 类型保护函数：某些判断可能不是一条语句能够搞定的，需要更多复杂的逻辑，适合封装到一个函数内

### 高级类型

#### 交叉类型（类型并集）

1. 含义：将多个类型合并为一个类型，新的类型将具有所有类型的特性
2. 应用场景：混入

```ts
let pet: DogInterface & CatInterface = {
  run() {},
  jump() {},
};
```

#### 联合类型（类型交集）

1. 含义：类型并不确定，可能为多个类型中的一个
2. 应用场景：多类型支持
3. 可区分的联合类型：结合联合类型和字面量类型的类型保护方法

```ts
let a18: number | string = 1; // let a18: number | string
let b18: "a" | "b" | "c"; //let b18: "a" | "b" | "c"
let c18: 1 | 2 | 3; // let c18: 1 | 2 | 3
```

#### 字面量类型

1. 字符串字面量
2. 数字字面量
3. 应用场景：限制变量取值范围

限制取值范围

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") {
      // ...
    } else if (easing === "ease-out") {
    } else if (easing === "ease-in-out") {
    } else {
      // error! should not pass null or undefined.
    }
  }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
// button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

区别函数重载

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
  // ... code goes here ...
}
```

#### 索引类型

1. 要点
   1. `keyof T` （索引查询操作符）：类型 T 公共属性名的字面量联合类型
   2. `T[K]` （索引访问操作符）：对象 T 的属性 K 所代表的类型
   3. 泛型约束
2. 应用场景：从一个对象中选取某些属性的值

使用索引类型，编译器就能够检查使用了动态属性名的代码。

keyof T， 索引类型查询操作符。 对于任何类型 T， keyof T 的结果为 T 上已知的公共属性名的联合。

```ts
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: "Jarid",
  age: 35,
};
let strings: string[] = pluck(person, ["name"]); // ok, string[]

// NOTE: keyof T
let personProps: keyof Person; // 'name' | 'age'
// pluck(person, ['age', 'unknown']); // error, 'unknown' is not in 'name' | 'age'
```

T[K]: 只要确保类型变量 K extends keyof T 就可以

```ts
// NOTE: T[K]

function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
  return o[name]; // o[name] is of type T[K]
}

let nameas: string = getProperty(person, "name");
console.log("nameas", nameas);
let age: number = getProperty(person, "age");
// let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```

#### 映射类型

1. 含义：从旧类型中创建出新的类型
2. 应用场景：
   1. `Readonly<T>`: 将 T 的所有属性变为只读
   2. `Partial<T>`: 将 T 的所有属性变为可选
   3. `Pick<T,K>`: 选取以 K 为属性的对象 T 的子集
   4. `Record<K,T>`: 创新属性为 K 的新对象，属性值的类型为 T

```ts
// 一种使用方式：在映射类型里，新类型以相同的形式去转换旧类型里每个属性。 例如，你可以令每个属性成为 readonly类型或可选的
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 另一种

type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;
```

最简单的使用方式

```ts
type Keys = "option1" | "option2";
type Flags = { [K in Keys]: boolean };
```

它的语法与索引签名的语法类型，内部使用了 for .. in。 具有三个部分：

1. 类型变量 K，它会依次绑定到每个属性。
2. 字符串字面量联合的 Keys，它包含了要迭代的属性名的集合。
3. 属性的结果类型。

最终转换为：

```ts
type Flags = {
  option1: boolean;
  option2: boolean;
};
```

#### 条件类型

1. 含义：T extend U ? X: Y（如果类型 T 可以赋值给类型 U，那么结果类型就是 X，否则就是 Y）
2. 应用场景：
   1. `Exclude<T,U>`: 从 T 中过滤掉可以赋值给 U 的类型
   2. `Extract<T,U>`: 从 T 中抽取出可以赋值给 U 的类型
   3. `NonNullable<T`>: 从 T 中除去 undefined 和 null
   4. `ReturnType<T>`: 获取函数的返回值类型

使用场景

```ts
// 源码
type Exclude<T, U> = T extends U ? never : T;
```

```ts
type ReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : any;
```

## typeScript 工程

### 代码检查

2019 年 1 月，TypeScirpt 官方决定全面采用 ESLint 作为代码检查的工具，并创建了一个新项目 typescript-eslint，提供了 TypeScript 文件的解析器 @typescript-eslint/parser 和相关的配置选项 @typescript-eslint/eslint-plugin 等。而之前的两个 lint 解决方案都将弃用：

1. typescript-eslint-parser 已停止维护
2. TSLint 将提供迁移工具，并在 typescript-eslint 的功能足够完整后停止维护 TSLint（Once we consider ESLint feature-complete w.r.t. TSLint, we will deprecate TSLint and help users migrate to ESLint1）

综上所述，目前以及将来的 TypeScript 的代码检查方案就是 typescript-eslint。

ts 已经能够在编译阶段检查出很多问题了，为什么 ts 项目中还需要代码检查？

这是因为 ts 关注的重心是类型的检查，而不是代码风格，当团队的成员越来越多时，同样的逻辑不同的人写出来可能会有很大的区别。比如说缩进应该是 4 个空格还是 2 个空格，这些问题 typescript 不会关注，但是却影响到多人协作开发时的效率、代码的可理解性以及可维护性。

### 声明文件

最容易理解的就是项目里引用 ui 库时，要为引入的组件编写声明文件，一般来说，我们目前在引用的库在社区中已经有编写好的声明文件，可以直接引用。

[查找包是否有 types 文件](https://microsoft.github.io/TypeSearch/)

```json
{
  "@types/classnames": "^2.2.7",
  "@types/enzyme": "^3.10.5",
  "@types/enzyme-adapter-react-16": "^1.0.6",
  "@types/enzyme-to-json": "^1.5.3",
  "@types/express": "^4.17.0",
  "@types/history": "^4.7.2",
  "@types/jest": "^24.0.13",
  "@types/lodash": "^4.14.133",
  "@types/qs": "^6.5.3",
  "@types/react": "^16.8.19",
  "@types/react-color": "^3.0.1",
  "@types/react-document-title": "^2.0.3",
  "@types/react-dom": "^16.8.4"
}
```

### 三斜线指令

三斜线指令是包含单个 XML 标签的单行注释。 注释的内容会做为编译器指令使用。

三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释，这包括其它的三斜线指令。 如果它们出现在一个语句或声明之后，那么它们会被当做普通的单行注释，并且不具有特殊的涵义。

```ts
/// <reference path="..." />
/// <reference path="..." />
```

指令是三斜线指令中最常见的一种。 它用于声明文件间的 依赖。

三斜线引用告诉编译器在编译过程中要引入的额外的文件.编译器会对输入文件进行预处理来解析所有三斜线引用指令

### tsconfig.json

如果一个目录下存在一个 tsconfig.json 文件，那么它意味着这个目录是 TypeScript 项目的根目录。 tsconfig.json 文件中指定了用来编译这个项目的根文件和编译选项。 一个项目可以通过以下方式之一来编译：

使用 tsconfig.json

1. 不带任何输入文件的情况下调用 tsc，编译器会从当前目录开始去查找 tsconfig.json 文件，逐级向上搜索父目录。
2. 不带任何输入文件的情况下调用 tsc，且使用命令行参数--project（或-p）指定一个包含 tsconfig.json 文件的目录。
3. 当命令行上指定了输入文件时，tsconfig.json 文件会被忽略。

```ts
{
  "compilerOptions": {
    "outDir": "build/dist",
    "module": "esnext", // 指定使用的模块，common.jd、amd、system、umd、或者es2015
    "target": "esnext", // 指定ECMAScript的目标版本, esnext指的是当前的ECMAScript版本
    "lib": ["esnext", "dom"], // 指定要包含在编译中的库文件
    "sourceMap": true, // 生成相应的 .map 文件
    "baseUrl": ".", // 用于解析非相对模块名称的基目录
    "jsx": "react", // 指定 JSX 代码的生成 .preserve、react-native、react文件
    "allowSyntheticDefaultImports": true, // 允许从没有设置默认导出的模块中默认导入
    "moduleResolution": "node", // 模块解析选项，选择模块解析策略、node或classic
    "resolveJsonModule": true, // 允许从 .json 中导入、导出其类型
    "noImplicitAny": false, // 在表达式和声明上有隐含的 any类型时报错。
    "forceConsistentCasingInFileNames": true, // 禁止对同一个文件的不一致的引用
    "noImplicitReturns": true, // 不是函数的所有返回路径都有返回值时报错。
    "suppressImplicitAnyIndexErrors": true, // 阻止 --noImplicitAny对缺少索引签名的索引对象报错。
    "noUnusedLocals": true, // 若有未使用的局部变量则抛错。
    "allowJs": true, // 允许编译javascript文件
    "experimentalDecorators": true, // 启用实验性的ES装饰器。
    "strict": true, // 启用所有严格类型检查选项。启用 --strict相当于启用 --noImplicitAny, --noImplicitThis, --alwaysStrict， --strictNullChecks和 --strictFunctionTypes和--strictPropertyInitialization。
    "strictNullChecks": false, // 在严格的 null检查模式下， null和 undefined值不包含在任何类型里，只允许用它们自己和 any来赋值（有个例外， undefined可以赋值到 void）。
    "paths": { // 模块名到基于 baseUrl的路径映射的列表。
      "@/*": ["./src/*"]
    }
  },
  "exclude": [ // 编译过滤
    "node_modules",
    "build",
    "scripts",
    "acceptance-tests",
    "webpack",
    "jest",
    "src/setupTests.ts",
    "tslint:latest",
    "tslint-config-prettier"
  ]
}

```

### 代码迁移

JavaScript 项目迁移到 TypeScript 一般采用渐进式迁移策略，目前有三种

1. 共存策略
   1. 如果我们用编译器选项 --allowJs，则 TypeScript 编译器支持 JavaScript 和 TypeScript 文件的混合，我们可以将文件一个一个的切换到 TypeScript
2. 宽松策略：将所有的 js 文件重命名为 ts 或者 tsx，然后使用最宽松的代码检查
3. 严格策略：开启最严格的类型检查规则

目前我们的项目基本上都是 通过 create-react-app 脚手架来搭建的，这里先介绍一下 关于这种搭建方式的迁移。

1. 第一步：安装 typescript 和一些包的声明文件，在我们的项目里用到了很多我们的自建库，注意需要为这些库编写声明文件。

```
$ npm install --save typescript @types/node @types/react @types/react-dom @types/jest
$ # 或者
$ yarn add typescript @types/node @types/react @types/react-dom @types/jest

```

2. 第二步：新建 tsconfig.json 文件

3. 第三步：将文件的 js 后缀改为 tsx 后缀，修改所产生的问题

### 新建一个 ts 项目

如果通过 create-react-app 脚手架搭建的话，可以直接 使用

```
npx create-react-app my-app --template typescript
# or
yarn create react-app my-app --template typescript
```

### 一些 tips

如果系统不是用 babel 来编译，ts 3.7 以上就已经支持了，但是如果项目使用 babel 编译的话，使用 babel 调用 ts 的时候，ts 只会被用来做类型检查，真正的编译还是 babel 亲自上场的。所以如果想在项目中使用可选链接操作符，需要安装 `@babel/plugin-proposal-optional-chaining`

```
npm install --save-dev @babel/plugin-proposal-optional-chaining
```

在 webpack.config.json 中配置

```
{
  "plugins": ["@babel/plugin-proposal-optional-chaining"]
}
```

就可以在项目中使用可选链接操作符了

完结撒花~
