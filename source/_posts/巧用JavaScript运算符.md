---
title: 巧用JavaScript运算符
date: 2021-12-20 10:08:07
---


巧用运算符，事半功倍！

## ?? 非空运算符

在 JS 中，?? 运算符被称为非空运算符。如果第一个参数不是 null/undefined（译者注：这里只有两个假值，但是 JS 中假值包含：未定义 undefined、空对象 null、数值 0、空数字 NaN、布尔 false，空字符串’’，不要搞混了），将返回第一个参数，否则返回第二个参数。比如，

```
null ?? 5 // => 5
3 ?? 5 // => 3
```

给变量设置默认值时，以前常用 ||逻辑或运算符，例如，

```
var prevMoney = 1
var currMoney = 0
var noAccount = null
var futureMoney = -1
function moneyAmount(money) {
return money || `账户未开通`
}
console.log(moneyAmount(prevMoney)) // => 1
console.log(moneyAmount(currMoney)) // => 账户未开通
console.log(moneyAmount(noAccount)) // => 账户未开通
console.log(moneyAmount(futureMoney)) // => -1
```

上面我们创建了函数 moneyAmount，它返回当前用户余额。我们使用 || 运算符来识别没有帐户的用户。然而，当用户没有帐户时，这意味着什么？将无账户视为空而不是 0 更为准确，因为银行账户可能没有（或负）货币。在上面的例子中，|| 运算符将 0 视为一个虚假值，不应该包括用户有 0 美元的帐户。让我们使用?? 非空运算符来解决这个问题：

```
var currMoney = 0
var noAccount = null
function moneyAmount(money) {
  return money ?? `账户未开通`
}
moneyAmount(currMoney) // => 0
moneyAmount(noAccount) // => `账户未开通`
```

概括地说 ?? 运算符允许我们在忽略错误值（如 0 和空字符串）的同时指定默认值。

## ??= 空赋值运算符

??= 也被称为空赋值运算符，与上面的非空运算符相关。看看它们之间的联系：

```

var x = null
var y = 5
console.log(x ??= y) // => 5
console.log(x = (x ?? y)) // => 5
```

仅当值为 null 或 undefined 时，此赋值运算符才会赋值。上面的例子强调了这个运算符本质上是空赋值的语法糖（译者注，类似的语法糖：a = a + b 可写成 a += b ）。接下来，让我们看看这个前端培训运算符与默认参数（译者注，默认参数是 ES6 引入的新语法，仅当函数参数为 undefined 时，给它设置一个默认值）的区别：

```
function gameSettingsWithNullish(options) {
  options.gameSpeed ??= 1
  options.gameDiff ??= 'easy'
  return options
}
function gameSettingsWithDefaultParams(gameSpeed=1, gameDiff='easy') {
  return {gameSpeed, gameDiff}
}
gameSettingsWithNullish({gameSpeed: null, gameDiff: null}) // => {gameSpeed: 1, gameDiff: 'easy'}
gameSettingsWithDefaultParams(undefined, null) // => {gameSpeed: null, gameDiff: null}
```

上述函数处理空值的方式有一个值得注意的区别。默认参数将用空参数（译者注，这里的空参数，只能是 undefined）覆盖默认值，空赋值运算符将不会。默认参数和空赋值都不会覆盖未定义的值。

## ?. 链判断运算符

链判断运算符?. 允许开发人员读取深度嵌套在对象链中的属性值，而不必验证每个引用。当引用为空时，表达式停止计算并返回 undefined。比如：

```
var travelPlans = {
  destination: 'DC',
  monday: {
    location: 'National Mall',
    budget: 200
  }
}
console.log(travelPlans.tuesday?.location) // => undefined
```

现在，把我们刚刚学到的结合起来，把 tuesday 加入旅行计划中！

```
 
function addPlansWhenUndefined(plans, location, budget) {
  if (plans.tuesday?.location == undefined) {
    var newPlans = {
      plans,
      tuesday: {
        location: location ?? "公园",
        budget: budget ?? 200
      },
    }
  } else {
    newPlans ??= plans; // 只有 newPlans 是 undefined 时，才覆盖
    console.log("已安排计划")
  }
  return newPlans
}
// 译者注，对象 travelPlans 的初始值，来自上面一个例子
var newPlans = addPlansWhenUndefined(travelPlans, "Ford 剧院", null)
console.log(newPlans)
// => { plans:
// { destination: 'DC',
// monday: { location: '国家购物中心', budget: 200 } },
// tuesday: { location: 'Ford 剧院', budget: 200 } }
newPlans = addPlansWhenUndefined(newPlans, null, null)
// logs => 已安排计划
// returns => newPlans object
```

上面的例子包含了我们到目前为止所学的所有运算符。现在我们已经创建了一个函数，该函数将计划添加到当前没有嵌套属性的对象 tuesday.location 中。我们还使用了非空运算符来提供默认值。此函数将错误地接受像“0”这样的值作为有效参数。这意味着 budget 可以设置为零，没有任何错误。

## ?: 三元运算符

?: 又叫条件运算符，接受三个运算数：条件 ? 条件为真时要执行的表达式 : 条件为假时要执行的表达式。实际效果：

```

function checkCharge(charge) {
  return (charge > 0) ? '可用' : '需充值'
}
console.log(checkCharge(20)) // => 可用
console.log(checkCharge(0)) // => 需充值
```

如果你写过 JS，可能见过三元运算符。但是，你知道三元运算符可以用于变量赋值吗？

```
var budget = 0
var transportion = (budget > 0) ? '火车' : '步行'
console.log(transportion) // => '步行'
```

我们甚至可以用它来实现空赋值的行为：

```
var x = 6
var x = (x !== null || x !== undefined) ? x : 3
console.log(x) // => 6　
```

让我们通过创建一个函数来概括这个运算：

```
function nullishAssignment(x,y) {
  return (x == null || x == undefined) ? y : x
}
nullishAssignment(null, 8) // => 8
nullishAssignment(4, 8) // => 4　　
```

在结束之前，让我们使用三元运算符重构前面示例中的函数：

```
function addPlansWhenUndefined(plans, location, budget) {
 var newPlans =
   plans.tuesday?.location === undefined
     ? {
         plans,
         tuesday: {
           location: location ?? "公园",
           budget: budget ?? 200
         },
       }
     : console.log("已安排计划");
 newPlans ??= plans;
 return newPlans;
}
```