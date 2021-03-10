---
title: 代码重构之重构API
date: 2021-03-10 11:08:19
cover: https://m.360buyimg.com/img/jfs/t1/143336/39/14133/31931/604835adEd45d1a89/12375373fd23c291.jpg
---


#### 将查询函数和修改函数分离

> 动机：任何有返回值的函数，都不应该有看得到的副作用。

一种常见的优化办法是：将查询所得记过缓存于某个字段中，后续重复查询可以大大加快速度。

1. 复制函数，以查询来命名
2. 移除有副作用的代码
3. 执行静态检查
4. 查找所有调用原函数的地方，替换为新函数，并在下面调用一次原函数
5. 从原函数中去掉返回值
6. 测试

Before:

```js
// Miscreant - 恶棍
function alertForMiscreant(people) {
  for (const p of people) {
    if (p === "Don") {
      setOffAlarms();
      return "Don";
    }
    if (p === "John") {
      setOffAlarms();
      return "John";
    }
  }
  return "";
}

const found = alertForMiscreant(people);
```

After:

```js
function findMiscreant(people) {
  for (const p of people) {
    if (p === "Don") {
      return "Don";
    }
    if (p === "John") {
      return "John";
    }
  }
  return "";
}

function alertForMiscreant(people) {
  if (findMiscreant(people) !== "") setOffAlarms(); // 替换算法
}

const found = findMiscreant(people);
alertForMiscreant(people);
```

#### 函数参数化

> 动机：两个函数逻辑非常相似，可以将其合并为一个函数，以参数形式传入不同值，从而消除重复。

1. 从一组相似函数选则一个
2. 把需要作为参数传入的字面量添加到参数列表
3. 修改该函数所有调用地方
4. 测试
5. 修改函数体，令其使用新传入的参数
6. 替换与其相似的函数，并测试

Before：

```js
function baseCharge(usage) {
  if (usage < 0) return usd(0);
  const amount =
    bottomBand(usage) * 0.03 + middleBand(usage) * 0.05 + topBand(usage) * 0.07;
  return usd(amount);
}
function bottomBand(usage) {
  return Math.min(usage, 100);
}
function middleBand(usage) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}
function topBand(usage) {
  return usage > 200 ? usage - 200 : 0;
}
```

After:

```js
function withinBand(usage, bottom, top) {
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}
function baseCharge(usage) {
  if (usage < 0) return usd(0);
  const amount =
    withinBand(usage, 0, 100) * 0.03 +
    withinBand(usage, 100, 200) * 0.05 +
    withinBand(usage, 200, Infinity) * 0.07;
  return usd(amount);
}
```

#### 移除标记参数

> 动机：用标记参数来指示被调函数应该执行哪一部分逻辑，影响了函数内部控制流。移除标记参数是代码更加整洁。

1. 针对参数的每一种可能值，新建一个明确函数
2. 修改调用函数的地方为新建明确函数

Before：

```js
function deliveryDate(anOrder, isRush) {
  if (isRush) {
    let deliveryTime;
    if (["MA", "CT"].includes(anOrder.deliveryState)) deliveryTime = 1;
    else if (["NY", "NH"].includes(anOrder.deliveryState)) deliveryTime = 2;
    else deliveryTime = 3;
    return anOrder.placedOn.plusDays(1 + deliveryTime);
  } else {
    let deliveryTime;
    if (["MA", "CT", "NY"].includes(anOrder.deliveryState)) deliveryTime = 2;
    else if (["ME", "NH"].includes(anOrder.deliveryState)) deliveryTime = 3;
    else deliveryTime = 4;
    return anOrder.placedOn.plusDays(2 + deliveryTime);
  }
}
```

After:

```js
function deliveryDate(anOrder, isRush) {
  if (isRush) return rushDeliveryDate(anOrder);
  else return regularDeliveryDate(anOrder);
}
function rushDeliveryDate(anOrder) {
  let deliveryTime;
  if (["MA", "CT"].includes(anOrder.deliveryState)) deliveryTime = 1;
  else if (["NY", "NH"].includes(anOrder.deliveryState)) deliveryTime = 2;
  else deliveryTime = 3;
  return anOrder.placedOn.plusDays(1 + deliveryTime);
}
function regularDeliveryDate(anOrder) {
  let deliveryTime;
  if (["MA", "CT", "NY"].includes(anOrder.deliveryState)) deliveryTime = 2;
  else if (["ME", "NH"].includes(anOrder.deliveryState)) deliveryTime = 3;
  else deliveryTime = 4;
  return anOrder.placedOn.plusDays(2 + deliveryTime);
}
```

#### 保持对象完整

> 动机：如果一个函数需要传入一个对象的多个属性值，传递对象本身是更好地方式。

1. 新建空函数，传入对象
2. 新函数中调用旧函数，并把新参数映射到就的参数列表
3. 执行静态检查
4. 修改调用地方为新函数
5. 把旧函数内联到新函数体内
6. 修改函数名为旧函数名，并修改所有调用的地方

Before:

```js
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (!aPlan.withinRange(low, high))
  alerts.push("room temperature went outside range");

class HeatingPlan {
  withinRange(bottom, top) {
    return (
      bottom >= this._temperatureRange.low && top <= this._temperatureRange.high
    );
  }
}
```

After:

```js
if (!aPlan.withinRange(aRoom.daysTemRange))
  alerts.push("room temperature went outside range");

class HeatingPlan {
  withinRange(aNumberRange) {
    return (
      aNumberRange.low >= this._temperatureRange.low &&
      aNumberRange.high <= this._temperatureRange.high
    );
  }
}
```

#### 以查询取代参数

> 动机：如果函数的一个参数只需要向另一个参数查询就能得到，则参数列表应避免重复。

1. 如有必要使用提炼函数将参数的查询过程提炼到一个独立函数中
2. 将函数体内参数饮用的地方改为调用新建的函数，并测试
3. 将参数去掉

Before:

```js
class Order {
  get finalPrice() {
    const basePrice = this.quantity * this.itemPrice;
    let discountLevel;
    if (this.quantity > 100) discountLevel = 2;
    else discountLevel = 1;
    return this.discountedPrice(basePrice, discountLevel);
  }
  discountedPrice(basePrice, discountLevel) {
    switch (discountLevel) {
      case 1:
        return basePrice * 0.95;
      case 2:
        return basePrice * 0.9;
    }
  }
}
```

After:

```js
class Order {
  get finalPrice() {
    const basePrice = this.quantity * this.itemPrice;
    return this.discountedPrice(basePrice);
  }
  get discountLevel() {
    return this.quantity > 100 ? 2 : 1;
  }
  discountedPrice(basePrice) {
    switch (discountLevel) {
      case 1:
        return basePrice * 0.95;
      case 2:
        return basePrice * 0.9;
    }
  }
}
```

#### 以参数取代查询

> 动机：在负责逻辑处理的模块中只有纯函数，其外再包裹处理 I/O 和其他可变元素的逻辑代码，使其更容易测试及理解。JavaScript 的类模型无法强制要求类的不可变形——始终有办法修改对象的内部数据，以参数取代查询是达成让类保持不可变的利器。

1. 对查询操作的代码提炼为变量，从函数体中分离出去
2. 提炼函数体内代码为新函数
3. 使用内联变量消除刚提炼出来的变量
4. 对原函数使用内联函数
5. 新函数该会原函数名字

Before:

```js
class HeatingPlan {
  get targetTemperature() {
    if (thermostat.selectedTemperature > this._max) return this._max;
    else if (thermostat.selectedTemperature < this._min) return this._min;
    else return thermostat.selectedTemperature;
  }
}

if (thePlan.targetTemperature > thermostat.currentTemperature) setToH;
else if (thePlan.targetTemperature < thermostat.currentTemperature) setToC;
else setOff();
```

After:

```js
class HeatingPlan {
  targetTemperature(selectedTemperature) {
    if (selectedTemperature > this._max) return this._max;
    else if (selectedTemperature < this._min) return this._min;
    else return selectedTemperature;
  }
}

if (
  thePlan.targetTemperature(thermostat.selectedTemperature) >
  thermostat.currentTemperature
)
  setToHeat();
else if (
  thePlan.targetTemperature(thermostat.selectedTemperature) <
  thermostat.currentTemperature
)
  setToCool();
else setOff();
```

#### 移除设值函数

> 动机：如果不希望在对象创建之后某个属性还有机会被改变，就不要为它提供 set 函数。

1. 在构造函数中调用设值函数，对字段设值
2. 移除所有在构造函数之外对设值函数的调用，改为使用新的构造函数，并测试
3. 使用内联函数消去设置函数
4. 测试

Before:

```js
class Person {
  get name() {
    return this._name;
  }
  set name(arg) {
    this._name = arg;
  }
  get id() {
    return this._id;
  }
  set id(arg) {
    this._id = arg;
  }
}

const martin = new Person();
martin.name = "martin";
martin.id = "1234";
```

After:

```js
class Person {
  constructor(id) {
    this._id = id;
  }
  get name() {
    return this._name;
  }
  set name(arg) {
    this._name = arg;
  }
  get id() {
    return this._id;
  }
}
const martin = new Person("1234");
martin.name = "martin";
```

#### 以工厂函数取代构造函数

> 动机：与一般函数相比，构造函数常有一些丑陋的局限性，只能返回当前所调用类的实例，构造函数名称是固定的类名，需要通过特殊操作符调用。工厂函数的实现内部可以调用构造函数，也可以换别的方式实现。

1. 新建一个工厂函数，让它调用现有的构造函数
2. 将调用构造函数的代码替换为工厂函数
3. 每次修改，执行测试
4. 尽量缩小构造函数可见范围

Before:

```js
class Employee {
  constructor(name, typeCode) {
    this._name = name;
    this._typeCode = typeCode;
  }
  get name() {
    return this._name;
  }
  get type() {
    return Employee.legalTypeCodes[this._typeCode];
  }
  static get legalTypeCodes() {
    return { E: "Engineer", M: "Manager", S: "Salesman" };
  }
}

const candidate = new Employee(document.name, document.empType);
const leadEngineer = new Employee(document.leadEngineer, "E");
```

After:

```js
class Employee {
  constructor(name, typeCode) {
    this._name = name;
    this._typeCode = typeCode;
  }
  get name() {
    return this._name;
  }
  get type() {
    return Employee.legalTypeCodes[this._typeCode];
  }
  static get legalTypeCodes() {
    return { E: "Engineer", M: "Manager", S: "Salesman" };
  }
}

function createEmployee(name, typeCode) {
  return new Employee(name, typeCode);
}
const candidate = createEmployee(document.name, document.empType);
function createEngineer(name) {
  return new Employee(name, "E");
}
const leadEngineer = createEngineer(document.leadEngineer);
```

#### 以命令取代函数

> 动机：将函数封装成自己的对象，称为“命令对象”，简称“命令”，只服务于单一函数，获得对该函数的请求，执行函数。

1. 为想要包装的函数创建一个空类，根据该函数名字命名
2. 把函数移动到空类里
3. 给每个参数创建一个字段，并在构造函数中添加对应的参数

Before:

```js
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  let highMedicalRiskFlag = false;
  if (medicalExam.isSmoker) {
    healthLevel += 10;
    highMedicalRiskFlag = true;
  }
  let certificationGrade = "regular";
  if (scoringGuide.stateWithLowCertification(candidate.originState)) {
    certificationGrade = "low";
    result -= 5;
  }
  // lots more code like this
  result -= Math.max(healthLevel - 5, 0);
  return result;
}
```

After:

```js
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer().execute(candidate, medicalExam, scoringGuide);
}
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }

  execute() {
    this._result = 0;
    this._healthLevel = 0;
    this._highMedicalRiskFlag = false;
    this.this.scoreSmoking();
    this._certificationGrade = "regular";
    if (
      this._scoringGuide.stateWithLowCertification(this._candidate.originState)
    ) {
      this._certificationGrade = "low";
      this._result -= 5;
    }
    // lots more code like this
    this._result -= Math.max(healthLevel - 5, 0);
    return this._result;
  }

  scoreSmoking() {
    if (this._medicalExam.isSmoker) {
      this._healthLevel += 10;
      this._highMedicalRiskFlag = true;
    }
  }
}
```

#### 以函数取代命令

> 动机：借助命令对象可以轻松地将原本复杂的函数拆解为多个方法，彼此间通过字段共享状态，拆解后的方法分别调用，开始调用前的数据状态也可以逐步构建。但如果这个函数不太复杂，可以考虑将其变回普通函数

1. 把“创建并执行命令对象”的代码单独提炼到一个函数中
2. 对命令对象在执行阶段调用到的函数，逐一使用内联函数
3. 把构造函数的参数转移到执行函数声明中
4. 执行函数中引用的所有字段改为使用参数，并测试
5. 把“调用构造函数”和“调用执行函数”都内联到调用方
6. 测试
7. 把命令类删除

Before:

```js
class ChargeCalculator {
  constructor(customer, usage, provider) {
    this._customer = customer;
    this._usage = usage;
    this._provider = provider;
  }
  get baseCharge() {
    return this._customer.baseRate * this._usage;
  }
  get charge() {
    return this.baseCharge + this._provider.connectionCharge;
  }
}
```

After:

```js
function charge(customer, usage, provider) {
  const baseCharge = customer.baseRate * usage;
  return baseCharge + provider.connectionCharge;
}
```
