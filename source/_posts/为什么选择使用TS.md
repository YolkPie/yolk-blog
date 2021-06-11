---
title: 为什么选择使用TS
date: 2021-06-05 15:26:47
tags:
- TS

author: YYY
---
# 为什么选择使用TS

一个非常有争议性的观点：由于 TypeScript 具有更强的可扩展性 并且 可以带给开发者更好的体验，开发者应该更愿意选择 TypeScript 而不是 JavaScript。


**开发者回避使用 TypeScript 的三个借口**

1. 它让 JavaScript 更像 Java 和 .NET

优秀的 JavaScript 开发者会提醒你避免使用 OOP 的风格。太多开发者一上来就写个class的原因通常是因为 他/她 是从OOP背景过来的 —— 在Java，你不能光秃秃地定义一个常量，一个函数或者一个表达式，你得先有个类，然后在类里定义一个静态不可变的属性 （public static final 三连） 才能产生一个常量，类似的，也只能在类里定义一个（静态或者非静态）的方法才能让函数有容身之地
[在大部分场景下不鼓励使用JavaScript class](https://zhuanlan.zhihu.com/p/158956514)

> 问题的关键是：TypeScript 不会改变 JavaScript，它只是被加到 JavaScript 中。

2. 它使代码变得不必要的冗长/复杂

TypeScript 意味着更多的代码，但这些东西是元数据ーー它有助于描述你正在操纵的数据，从而降低了整体的认知成本。

> TypeScript 通过减少记忆成本和消除打开不相关的文件手动检查类型 来提高开发速度。

3. 脚本不需要类型，应用程序需要

如果你想创建一个使用鼠标拖动 div 的程序，那么你只需要编写一个小型的 JS 脚本。添加类型是没有意义的。在编写大多数脚本时（就用途而言，简单的程序调用内置方法并处理很少的数据），JavaScript 通常就足够了，添加类型会不必要地增加冗长性。

> 对于较大的应用程序,TypeScript 允许我们描述数据结构，而不必记住或手动查阅它们。我们的 IDE 和编译器可以捕获我们所犯的任何错误


**TS是谁写给谁看/用的？**

TS是定义者给使用者写的。为了让使用者更方便（VSCode提示）以及更安全（约束）的使用他提供的方法或者类。

1. VSCode提示

定义者指定了一个方法

``` js
export function foo(name: string): number {
  return name.length
}
```

那么作为使用者可以通过VSCode准确了解到该函数的参数和返回值

![](./assets/vscode.webp)


2. 巧用注释

为了更完美的展示提示信息，我们还可以添加注释

``` js
/** foo function
 * @description count string size
 */
export function foo(name: string): number {
  return name.length
}
```

这时使用者会看到

![](./assets/640.webp)


3. 类型推断

TypeScript 强大的静态类型检查系统的一个基本特性是 类型推断。根据每个变量的初始值，重新赋值 或 依赖关系，TypeScript 可以推导出每个变量可赋的最具体的类型。

让我们假设我们有 users，一个 User 对象数组，然后我们可以取出一个有特定 ID 的用户：

``` js
const user = users.find(u => u.id === 1);
```

因为用户是 User[] 类型的，并且基于 Array.find() 的函数签名，所以 TypeScript 知道 user 会是 User 类型。因此，为它定义类型是多余的：

> 值得注意的一点，如果项目路径配置了别名，那么可能引入的方法没有正确提示，此时只需要在tsconfig.json这里的compilerOptions选项添加一个paths配置

``` json
{
    "compilerOptions": {
        "paths": {
          "@/*": ["src/*"]
        }
    }
}
```

**案例展示TypeScript是多么的有用**

1. Vue: 难以理解的 “payload”

VueX 为 Vue 应用程序提供了一个中央状态，帮开发者避免在组件之间杂乱的传递 props。

为了修改存储的状态，我们定义了一个变异函数，该函数接受两个参数: 当前的 state 和一个包含更改信息的 payload。

假设我们希望保持用 JavaScript 对象表示的待办事项的中心状态。我们可以使用空数组初始化状态，如下所示:

``` js
state = {
  todos: [],
}
```

现在我们可以定义一个 mutation，使我们能够更新这个待办事项列表：

``` js
updateTodos: (state, payload) => {
  state.todos = payload;
}
```

这足够简单了。但是我们必须从另一个模块中调用这个 mutation，比如一个组件。如果没有明确的类型，我们就需要猜测（或者回忆） updatetodo 需要一个 Todo 对象列表。这不是件好事。

事情不止于此。假设另一个开发人员加入了我们的团队，我们要求他们修改 updateTodos这个 mutation，以便它也更新状态的另一部分，比如一个跟踪已完成的待办事项数量的变量（由 someTodo.isComplete 得到）。

要做到这一点，开发人员必须首先确定 payload 的类型，包括以下步骤：

开发者必须假定当前版本的 updateTodos 是正确的（即 payload 和 state.todos是相同类型）

开发者必须一路滚动到初始化状态对象的代码位置，以检查 todos 的类型（或者，如果 updateTodos 已经被调用，在代码中搜索调用的位置）。

开发者必须确定每个 to-do 对象的结构，才能确定它是已经被完成了。

在这一切结束之后，由于一路上所做的所有假设，开发者除了测试一下，没有其他方法来检查解决方案的正确性。

通过定义 Todo 类型并将 payload 的类型指定为 Todo[] ，我们就解决了所有这些问题。

毫无疑问，至少在这种情况下，TypeScript 为开发团队提供了一个有重大价值的优势，提高准确性、减少开发者的头痛、加快开发速度并且降低类似 bug 的发生几率。

2. 为 JSON 响应 添加类型

许多 JavaScript 应用程序会向远程 API 发出网络请求来获取数据。通常，在使用这些 API 时，我们会在开发时了解期望收到的响应的结构。

例如，假设我们正在从后端获取数据。我们的后端团队编写了全面的文档，详细描述了每个请求和响应对象的结构。

文档提到，每个响应都有以下结构:

``` json
{
  "status": "success" or "failure",
  "data": {
    ...
  }
}
```
这可以生成如下的 TypeScript 数据定义：

``` js
interface ApiResponse {
  status: "success" | "failure";
  data: any;
}
```

然后我们可以参数化 ApiResponse 来指定它的数据字段的类型：

``` js
interface ApiResponse<T> {
  status: "success" | "failure";
  data: T;
}
```

现在，我们的 API 方法在返回类型方面可以更加具体:：例如，我们可以返回一个ApiResponse<User[]> 表示用户列表，而不是仅仅返回一个 ApiResponse。

让我们来看看这是如何提高开发速度的。假设你有一个从后端获取用户列表的方法：

``` js
async function getUsers(): Promise<ApiResponse<User[]>> { 
  ... 
}
```

我们在组件中使用它来获取用户信息：

``` js
const users = await getUsers();
```

然后我们映射用户信息得到他们的姓名:

``` js
const userNames = users.map(u => u.name);
```

对吧？

错。你发现错误了吗？可能没有，但静态类型检查会。我们忘记了响应包含我们 首先需要处理的 success 和 data 字段。多亏了 TypeScript，我们的 IDE 可以立即捕捉到这个错误。

对于新手 JavaScript 开发者来说，处理 API 响应是一个常见的 “问题”。

随着 JavaScript 经验的提升，你将养成每次都手动检查 API 客户端响应类型的习惯。但 TypeScript 帮你做了这些事之后，难道你还需要要为这些问题烦心吗？它不仅节省时间，还能防止疏忽引发的问题。

现在想象一下，你正在从一个体育 API 中获取数据。你使用 /upcoming 拉取即将开始的体育比赛数据。API 文档提供了以下响应结构：

``` json
{
  "id": 247283,
  "name": "New York Knicks at Atlanta Hawks",
  "date": "2021-05-23T02:00:00+00:00",
  "competitors": ["New York Knicks", "Atlanta Hawks"],
  "venue": "Madison Square Garden",
  ...
}
```

如果不使用 TypeScript，你将不得不频繁参考文档来了解响应中的字段及其类型。如果多个团队成员正在处理这段代码，那么你必须与您的团队共享这些文档。

这意味着更多的时间，更多的努力，更大的犯错几率。出现越来越多的坏事，情况越来越糟。

但是使用 TypeScript，只需添加一个类型定义：

``` js
interface SportsApiResponse {
  id: number;
  name: string;
  date: string;
  competitors: [string, string];
  venue: string;
}
```

现在，任何时候你使用 SportsApiResponse，你都会准确地知道哪些字段可用以及它们的类型。这样可以节省大量的时间，并最大限度地减少字段名拼写错误或将字符串错当成数字的可能性。




**参考**
[为什么要用那么复杂的TS](https://juejin.cn/post/6953500339425247246)
[应该在JavaScript中使用Class吗？](https://zhuanlan.zhihu.com/p/158956514)
[应当使用 TypeScript 的更有说服力的原因](https://betterprogramming.pub/the-bad-reasons-people-avoid-typescript-and-the-better-reasons-why-they-shouldnt-86f8d98534de)





















