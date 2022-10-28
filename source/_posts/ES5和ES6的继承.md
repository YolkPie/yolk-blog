---
title: ES5和ES6的继承
date: 2022-06-28
author: 7
---
## ES5原型的几种常用继承方式
### 原型链继承（子类原型指向父类实例）
```js
function SuperType() {
    this.colors = ['red', 'blue', 'green'];
}

function SubType() {}

SubType.prototype = new SuperType();

let instance1 = new SubType();
instance1.colors.push('black');
console.log(instance1.colors);

let instance2 = new SubType();
console.log(instance2.colors);
```
缺点：
	- 属于引用类型传值，某一个副本实例属性的修改会引起其他副本实例属性的变化
	- 不能向父类构造函数传递参数，不够灵活

### 借用构造函数继承（执行子类构造函数、各得到一份父类实例副本）
```js
function Parent() {
    this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.say = function () {
    console.log('hello');
};

function Child() {
    Parent.call(this);
}

let instance3 = new Child();
instance3.colors.push('white');
console.log(instance3.colors);
console.log(instance3.prototype);
// instance3.say

let instance4 = new Child();
console.log(instance4.colors);
```
属于值传递，子类之间属性修改互不相干

缺点：没有父类prototype上的属性，而且instanceof操作无法判断出子类实例和父类的关系，因为子类的prototype和父类无关

### 组合式继承
```js
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.action = ['speak', 'run', 'eat'];
    console.log('我被调用了');
}
Person.prototype.say = function () {
    console.log(`my name is ${this.name} and I am ${this.age} years old!`);
};

function Student(name, age, score) {
    Person.call(this, name, age);  // 借用构造函数, 第一次调用父类构造函数
    this.score = score;
}

Student.prototype = new Person();  // 原型链继承, 第二次调用父类构造函数
Student.prototype.constructor = Student;  // 将实例的原型上的构造函数指定为当前子类的构造函数
Student.prototype.showScore = function () {
    console.log(`my score is ${this.score}`);
};

let xiaoming = new Student('xiaoming', 23, '88');
xiaoming.action.push('panio');
console.log(xiaoming.action);
xiaoming.say();
xiaoming.showScore();

let xiaohua = new Student('xiaohua', 24, '99');
console.log(xiaohua.action);
xiaohua.say();
xiaohua.showScore();
```
- Student.prototype = new Person();将子对象的prototype与父对象的实例关联，实现原型链连接
- Person.call(this, name, age); 和Student.prototype.constructor = Student; 在每一次new Student时，获得一个父实例副本，更改属性不会互相影响。

### 原型式继承
利用一个空对象作为媒介，将某对象直接赋值给空对象的构造函数的原型
```js
function object(obj){
    function F(){}
    F.prototype = obj;
    return new F();
}
```
object()函数创建了一个临时构造函数，将传入的对象赋值给这个构造函数的原型，并返回这个临时类型的一个实例。本质上，object()是对传入的对象执行了一次浅复制。
```js
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends);   //"Shelby,Court,Van,Rob,Barbie"
```
person的原型既有原始值属性又有引用值属性。这也意味着，当你通过anotherPerson 或者yetAnotherPerson 改变引用值属性的值时，person的引用值属性会被改变


[原文地址](https://www.jianshu.com/p/72fea052ed05)

## ES6继承
ES6引入了class语法糖，使继承操作看起来更像面向对象语言的写法。在JS中，class的本质还是一个function。
通过extends关键字，不仅可以继承一个类，也可以继承普通的构造函数。
```js
class A {}
class B extends A {
  constructor() {
    super();  // ES6 要求，子类的构造函数必须执行一次 super 函数
    		  //，否则会报错。
  }
}

function C() {}
class D extends C {}
```

constructor方法是类的默认构造函数，若未定义，则默认添加。在constructor中必须调用super方法，因为子类没有自己的this对象，而是继承父类的this对象，然后对其加工，super就代表了父类的构造函数，但在super内部的this指向B···
```js
A.prototype.constructor.call(this, props) // 以this参数的指向，props为参数执行A
```

```js
class A {
  constructor() {
    console.log(new.target.name); // new.target 指向当前正在执行的函数
  }
}

class B extends A {
  constructor() {
    super();
  }
}

new A(); // A
new B(); // B

// super()内部的this指向的是B
```
笔者知识有限，有理解不到位的地方，欢迎指正。