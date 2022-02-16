---
title: 你不知道的JSON.stringify
date: 2021-12-23 17:15:00
tags:
- 
cover: 
top_img: 
---

# 你不知道的JSON.stringify
`JSON.stringify`是我们经常用到的的一个方法，它主要作用是将 JavaScript 值和对象转换为字符串。如：
```js
JSON.stringify({ foo: "bar" });
// => '{"foo":"bar"}'

JSON.stringify(123);
// => '123'
```

但是JS 的许多地方都有问题，这个函数也不例外。我们可能会想象一个叫做 "stringify "的函数总是返回一个字符串......但它并没有！
例如，如果你尝试 stringify undefined，它返回 undefined ，而不是一个字符串。
```js
JSON.stringify(undefined);
// => undefined
```

### 什么时候 JSON.stringify 不返回字符串?
`undefined`、任意的函数以及 `symbol` 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。函数、`undefined` 被单独转换时，会返回 `undefined`。
对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误
我认为 `JSON.stringify` 能够返回字符串以外的东西是挺惊讶的。但在6种情况下，它可以返回`undefined`:

1、试图在顶层对 `undefined` 进行序列化，会返回 `undefined`。
```js
JSON.stringify(undefined);
// => undefined
```
2、尝试序列化函数也会返回 `undefined`。对于常规函数、箭头函数、异步函数和生成器函数都是如此。
```js
JSON.stringify(function foo() {});
// => undefined

JSON.stringify(() => {});
// => undefined

function bar() {}
bar.someProperty = 123;
JSON.stringify(bar);
// => undefined

```
3、尝试序列化symbol 也会返回 `undefined`
```js
JSON.stringify(Symbol("computers were a mistake"));
// => undefined

```
4、在浏览器中，试图序列化被废弃的 `document.all` 也会返回 `undefined`
```js
JSON.stringify(document.all)
// => undefined

```
这只影响到浏览器，因为document.all在其他环境中是不可用的，比如Node
5、带有 toJSON 函数的对象将被运行，而不是试图正常地序列化它们。但是如果 toJSON 返回上面的一个值，试图在顶层序列化它将导致 JSON.stringify 返回undefined。
```js
JSON.stringify({ toJSON: () => undefined });
// => undefined

JSON.stringify({ ignored: true, toJSON: () => undefined });
// => undefined

JSON.stringify({ toJSON: () => Symbol("heya") });
// => undefined

```
6、你可以传递第二个参数，称为 "replacer"，它可以改变序列化的逻辑。如果这个函数为顶层返回上述值之一，JSON.stringify 将返回undefined
```js
JSON.stringify({ ignored: true }, () => undefined);
// => undefined

JSON.stringify(["ignored"], () => Symbol("hello"));
// => undefined

```
需要注意的是，其中的许多东西实际上只影响到顶层的序列化。例如，JSON.stringify({foo: undefined})，返回字符串"{}"，这并不令人惊讶。

另外 TypeScript的类型定义在这里是不正确的。例如，下面的代码类型的校验可以通过：
```ts
const result: string = JSON.stringify(undefined);
```

### JSON.stringify 也可能遇到问题，导致它抛出一个错误。
在正常情况下，有四种情况会发生：
- 1、循环引用会导致抛出一个类型错误。
```js
const b = { a };
a.b = b;

JSON.stringify(a);
// => TypeError: cyclic object value

```
注意，这些错误消息在不同浏览器可能提示是不样的，例如，Firefox 的错误信息与Chrome的不同。
- 2、BigInts不能用 JSON.stringify 进行序列化，这些也会导致一个TypeError
```js
JSON.stringify(12345678987654321n);
// => TypeError: BigInt value can't be serialized in JSON

JSON.stringify({ foo: 456n });
// => TypeError: BigInt value can't be serialized in JSON
```
- 3、带有 toJSON 函数的对象将被运行。如果这些函数抛出错误，它将冒泡到调用者
```js
const obj = {
  foo: "ignored",
  toJSON() {
    throw new Error("Oh no!");
  },
};

JSON.stringify(obj);
// => Error: Oh no!

```
- 4、你可以传递第二个参数，称为 replacer。如果这个函数抛出一个错误，它将冒泡。
```js
JSON.stringify({}, () => {
  throw new Error("Uh oh!");
});
// => Error: Uh oh!
```
现在我们已经看到了 JSON.stringify 不返回字符串的情况，接下来，我们来看看如何避免这些问题

## 如何避免这些问题
### 处理循环引用
JSON.stringify 在传递循环引用时最容易出错。如果这对你来说是一个常见的问题，我推荐 json-stringify-safe 包，它能很好地处理这种情况。
```js
const stringifySafe = require("json-stringify-safe");

const a = {};
const b = { a };
a.b = b;

JSON.stringify(a);
// => TypeError: cyclic object value

stringifySafe(a);
// => '{"b":{"a":"[Circular ~]"}}'

```
### 封装
你可能想用你自己的自定义函数来封装 JSON.stringify。你可以决定你想要它做什么。错误应该冒出来吗？如果 JSON.stringify 返回 undefined，应该怎么做？
例如，Signal Desktop有一个名为 reallyJsonStringify 的函数，它总是返回一个用于调试的字符串。就像这样
```js
function reallyJsonStringify(value) {
  let result;
  try {
    result = JSON.stringify(value);
  } catch (_err) {
    // If there's any error, treat it like `undefined`.
    result = undefined;
  }

  if (typeof result === "string") {
    // It's a string, so we're good.
    return result;
  } else {
    // Convert it to a string.
    return Object.prototype.toString.call(value);
  }
}
```
### 关于TypeScript类型
如果你已经在用 TypeScript，可能会惊讶地发现，TypeScript对 JSON.stringify的官方定义在这里并不正确。它们实际上看起来像这样:
```js
// Note: 这里面简化过
interface JSON {
  // ...
  stringify(value: any): string;
}

```
建议： 用自定义类型定义自己的包装器
